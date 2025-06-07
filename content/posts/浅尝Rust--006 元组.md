---
title: 浅尝Rust--006 元组
date: 2025-06-07T13:04:18+08:00
tags:
  - rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/compound-type/tuple.html](https://course.rs/basic/compound-type/tuple.html)
## 概念

由多种类型组合到一起的复合类型


```
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

## 用模式匹配解构元组


```
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

解构：将元组中的值按顺序绑定到对应的变量上

## 用.访问元组


```
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

## 常用示例

`在函数返回值场景`

```
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

calculate_length接收s1字符串的所有权，计算字符串长度，接着把字符串所有权和字符长度返回给s2和len变量

