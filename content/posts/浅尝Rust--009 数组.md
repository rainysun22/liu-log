---
title: 浅尝Rust--009 数组
date: 2025-06-07T15:31:19+08:00
tags:
  - rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/compound-type/array.html](https://course.rs/basic/compound-type/array.html)
## 数组

`Rust的基本类型`

- 长度固定
- 元素必须有相同的类型
- 依次线性排列

## 数组使用

1. 创建数组

```
let a = [1, 2, 3, 4, 5];
let a: [i32; 5] = [1, 2, 3, 4, 5];  // 声明类型
```

使用下面语法可以初始化一个`某个值重复出现N次的数组`

```
let a = [3; 5];  // 等价于let a = [3, 3, 3, 3, 3];
```

2. 访问数组

```
fn main() {
    let a = [9, 8, 7, 6, 5];

    let first = a[0]; // 获取a数组第一个元素
    let second = a[1]; // 获取第二个元素
}
```

**数组越界访问会在运行期panic，编译器不会检查出来**

3. 数组类型为非基本类型

```
let array = [String::from("rust is good!"); 8]; // Error！
```

由于`[x;n]`语法通过Copy特征的形式初始化数组，故复杂类型不能这样写。可以改成如下写法：

```
let array = [String::from("rust is good!"),String::from("rust is good!"),String::from("rust is good!")];

println!("{:#?}", array);

// 可以调用std::array::from_fn
let array: [String; 8] = std::array::from_fn(|_i| String::from("rust is good!"));

println!("{:#?}", array);
```

## 数组切片

```
let a: [i32; 5] = [1, 2, 3, 4, 5];

let slice: &[i32] = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

## 使用示例

```
fn main() {
  // 编译器自动推导出one的类型
  let one             = [1, 2, 3];
  // 显式类型标注
  let two: [u8; 3]    = [1, 2, 3];
  let blank1          = [0; 3];
  let blank2: [u8; 3] = [0; 3];

  // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
  let arrays: [[u8; 3]; 4]  = [one, two, blank1, blank2];

  // 借用arrays的元素用作循环中
  for a in &arrays {
    print!("{:?}: ", a);
    // 将a变成一个迭代器，用于循环
    // 你也可以直接用for n in a {}来进行循环
    for n in a.iter() {
      print!("\t{} + 10 = {}", n, n+10);
    }

    let mut sum = 0;
    // 0..a.len,是一个 Rust 的语法糖，其实就等于一个数组，元素是从0,1,2一直增加到到a.len-1
    for i in 0..a.len() {
      sum += a[i];
    }
    println!("\t({:?} = {})", a, sum);
  }
}
```
