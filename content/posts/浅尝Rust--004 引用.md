---
title: 浅尝Rust--004 引用
date: 2025-06-01T22:04:51+08:00
tags:
  - Rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/ownership/borrowing.html](https://course.rs/basic/ownership/borrowing.html)
## 引用

`获取变量的引用，称为借用`

`常规引用是一个指针类型，指向了对象存储的内存地址。(对比C语言的指针)`

```
fn main() {
    let s1 = String::from("hello");
    let s = &s1;

    assert_eq!("hello", s1);
    assert_eq!("hello", *s);
}
```

![](/images/Rust笔记--004%20引用.png)
## 不可变引用

```
fn main() {
    let s1 = String::from("hello");
    let s = &s1;

    s.push_str("Rust");  // Error
}
```

如果想修改上述代码中的引用，会发生错误。`引用指向的值默认不可变`

## 可变引用

```
fn main() {
    let mut s1 = String::from("hello");
    let s = &mut s1;

    s.push_str("Rust");  // Success
}
```

**原则1：可变引用同时只能存在一个**

```
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;  // Error

println!("{}, {}", r1, r2);

```

可通过手动限制变量作用域使两个可变引用同时存在

```
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

let r2 = &mut s;

```

**原则2：可变引用和不可变引用不能同时存在**

`原因：r1和r2正在借用不可变引用，肯定不希望r3随便修改`

```
let mut s = String::from("hello");

let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题

println!("{}, {}, and {}", r1, r2, r3);

```

如果r3借用的位置在r1和r2作用域外，可以借用

```
fn main() {
   let mut s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    println!("{} and {}", r1, r2);
    // 新编译器中，r1,r2作用域在这里结束

    let r3 = &mut s;  // Success
    println!("{}", r3);
}
```

**原则3：在可变引用的作用域内不能使用原值**

```
fn main() {
    let mut s1 = String::from("hello");
    let s = &mut s1;
    println!("{}", s1);  // Error,因为println
    s.push_str(" world");
}
```

**总结：也就是说，可变引用就相当于暂时把值借给别人，在别人的作用范围内，你是拿不回来的，只能等别人用完。**

## 悬垂引用

`也就是悬空指针。指针指向某个值后，这个值被释放掉了，而指针仍然存在`

```
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // dangle 返回一个字符串的引用

    let s = String::from("hello"); // s 是一个新字符串

    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！
```

解决方案：转移所有权

```
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```
