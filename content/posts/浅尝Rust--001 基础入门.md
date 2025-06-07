---
title: 浅尝Rust--001 基础入门
date: 2025-06-07T16:48:40+08:00
tags:
  - Rust
author: liuzifeng
---
## 1.1 变量

> 学习参考：[https://course.rs/basic/variable.html](https://course.rs/basic/variable.html)
### 变量绑定

```
let a = "hello world"
```
- rust最核心的原则：**所有权**
- 所有权，简单来讲，任何内存对象都是有主人的，绑定是把"hello world"绑定给一个变量a，让a成为"hello world"的主人

### 变量的可变性

- rust变量在默认情况下是**不可变的**
- 通过**mut**关键字将变量变为可变的
```
let mut a = 5;
a = 6;
```
- 创建一个变量却不适用，用下划线_，在**模式匹配**会用到
```
let _a = 5;
```

### 变量解构

`从一个相对复杂的变量中，匹配出该变量的一部分`

```
let (a, mut b): (bool,bool) = (true, false);
println!("a = {:?}, b = {:?}", a, b);
```

### 常量

- 不允许使用mut
- 默认不可变，自始至终不可变
- 使用const关键字声明，且**值必须进行类型标注**
```
const MAX_POINTS: u32 = 100_000;
```

### 变量遮蔽

`同名变量：后面声明的会遮蔽掉前面声明的`

```
let x = 5;
let x = x + 1;  // 在main函数的作用域内对之前的x进行遮蔽 
{
	let x = x * 2;  // 在当前的花括号作用域内，对之前的x进行遮蔽 
	println!("The value of x in the inner scope is: {}", x);  // x = 12
} 
println!("The value of x is: {}", x);  // x= 6
```

- let x = x + 1属于新变量，重新分配内存，只不过名字和上一个一模一样

`作用：在某个作用域内之前的变量不需要使用时，可以重复使用变量名字，不用去想取什么名`

***