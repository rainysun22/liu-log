---
title: 开盒vLLM：PagedAttention
date: 2025-06-29T14:15:55+08:00
tags:
  - vLLM
author: liuzifeng
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