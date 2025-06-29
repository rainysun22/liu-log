---
title: 开盒vLLM：PagedAttention
date: 2025-06-29T14:15:55+08:00
tags:
  - vLLM
author: liuzifeng
draft: "true"
---
> 学习参考：[https://zhuanlan.zhihu.com/p/691038809](https://zhuanlan.zhihu.com/p/691038809)
## 为KV Cache分配存储空间的常规方式

对于训练好的模型，一种常用的部署方式是将其打包成一个推理服务(server)，它接收客户端发送来的请求(request)，读取请求中的数据(prompt)来做推理。一个请求中可以只有一个prompt，也可以包含多个prompt。

在常规的推理框架中，当我们的服务接收到一条请求时，它会为这条请求中的prompts分配GPU显存空间，其中就包括对KV Cache的分配。

**由于推理所生成的序列长度是无法事先预知的，所以大部分框架会按照(batch_size, max_seq_len)这样的固定尺寸，在GPU显存上预先为一条请求开辟一块连续的矩形存储空间。**
- batch_size代表几条prompt
- max_seq_len代表最大生成多少个token

这样固定的分配方法很容易引起GPU显存利用不足，进而影响模型推理时的吞吐量。
![](/images/开盒vLLM：魔法背后的秘密.png)

假设`max_seq_len = 8`，当第一条请求(prompt1)过来时，为它安排了(1, 8)大小的连续存储空间。

当第2条请求(prompt2)过来时，同样也需要一块(1, 8)大小的存储空间。但此时prompt1所在的位置上，只剩3个空格子了，所以它只能另起一行做存储。对prompt3也是同理。

这三条prompt的KV Cache排布，没有充分利用GPU的显存：
- **浅色块**：prefill阶段prompt的KV Cache，是无论如何都会被使用的空间，不存在浪费
- **中色块**：decode阶段的KV Cache，其中`<eos>`表示序列生成的截止符。虽然这些中色块最终都会被用上，但是decode阶段一个个token生成时，我们并不能预知哪些块会被最终用上。所以**这些色块都是一种潜在的浪费，我们称这部分为预留碎片**
- **深色块**：也是decode阶段的KV Cache，直到序列生成完毕，它们都没有备用上。**由于这些深色块时预留的KV Cache的一部分，所以我们称其为内部碎片(internal fragment)**
- **灰色块**：不是我们预留的KV Cache的一部分，且最终也没有被用上，**我们称这些灰色快为外部碎片(external fragment)**。如果此时新来了一条prompt4，它也要求现存中的8个格子作为KV Cache。**此时你的显存上明明有9个空格子，但因为碎片不连续，所以无法被prompt4使用**，这时prompt4只能在队列中等待，直到GPU上有足够的显存资源时再进行推理，这就对模型推理的吞吐量造成了显著影响。

**太过静态的内存分配会造成很多空间浪费**，如果能做成动态化的”用多少占多少“，就能减少上述这些存储碎片。

## PagedAttention原理

问题1：**vLLM是通过什么技术，动态地为请求分配KV cache显存，提升显存利用率的？**

回答：**vLLM通过一种名为PagedAttention的技术，动态地为请求分配KV cache显存，提升显存利用率。**

整体来说，PagedAttention的设计灵感来自操作系统中虚拟内存的分页管理技术。

### 操作系统的虚拟内存

1. 不使用虚拟内存

