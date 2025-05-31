---
title: Rust笔记--003 所有权
date: 2025-05-31T19:03:07+08:00
tags:
  - rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/ownership/ownership.html](https://course.rs/basic/ownership/ownership.html)
## 所有权原则

1. Rust中每一个值都被一个变量所拥有，该变量称为值的所有者
2. 一个值同时只能被一个变量所拥有
3. 当所有者（变量）离开作用域范围时，这个值将被丢弃

## 变量绑定背后的数据交互

### part1：`未被转移所有权`

```
let x = 5;
let y = x;
```

**执行逻辑：**
代码首先将**5**绑定到变量**x**，接着拷贝**x**的值赋给**y**，最终**x**和**y**都等于5

**剖析：**
整数是 Rust 基本数据类型，是固定大小的简单值，因此这两个值都是通过**自动拷贝**的方式来赋值的，都被存在栈中，完全无需在堆上分配内存。故不需要所有权转移。

### part2：`转移所有权`

```
let s1 = String::from("hello"); 
let s2 = s1;
```

**浅谈String：**
String是一个复杂类型，由存储在栈中的堆指针、字符串长度、字符串容量共同组成。
其中堆指针指向了真实存储字符串内容的堆内存。

**执行逻辑：**
将s1指向存储字符串堆内容的指针移动给s2，s1不指向任何数据。即**将s1对字符串的所有权转移给s2**

**剖析：**
1. 如果拷贝String和存储在堆上的字节数组(深拷贝)给s2，会对性能造成非常大的影响。
2. 如果只拷贝String本身非常快，因为在64位机器上就拷贝了8字节的指针，8字节的长度，8字节的容量，总计24字节。但是这样s1和s2会同时指向一个字节数组，即一个值有两个所有者。
3. 假设一个值可以拥有两个所有者，当变量离开作用域时，Rust会自动调用drop函数清理变量的堆内存，由于两个变量指向了同一个位置，当s1和s2离开作用域时，它们都会尝试释放相同的内存，即**二次释放**。
4. 故Rust采用类似浅拷贝的方式，同时使s1无效，这个操作被称为**移动**
![](/images/Rust笔记--003%20所有权.png)

**注意：**
s1的所有权被转移给s2后，s1无效，这时如果使用s1会发生错误。
```
let s1 = String::from("hello"); 
let s2 = s1;
println!("{}, world!", s1);  // Error
```

### part3：`引用`

```
let x: &str = "hello, world";
let y = x;
println!("{},{}",x,y);
```

**浅析&str：**
这是字符串字面量的类型，被硬编码到程序中的字符串值。

**执行过程：**
x引用了存储在二进制可执行文件中的字符串，并没有持有所有权。故`let y = x`仅仅对该引用进行了拷贝，此时y和x都引用了同一个字符串。

**和part2的区别：**
String的例子中s1持有了通过String::from函数创建的值的所有权，在这个例子中，x只是引用了存储在二进制可执行文件中的字符串，并没有持有所有权。

### part4：`克隆(深拷贝)`

```
let s1 = String::from("hello"); 
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);  // Success
```

`注意：Rust永远不会自动创建数据的深拷贝`

**执行过程：**
使用clone函数，s2完整复制了s1的数据。没有发生所有权转移。

## 可以自动拷贝的类型

`如果一个类型拥有Copy特征，一个旧的变量在被赋值给其他变量后仍然可用，也就是赋值的过程即是拷贝的过程`

- 所有整数类型，比如 `u32`
- 布尔类型，`bool`，它的值是 `true` 和 `false`
- 所有浮点数类型，比如 `f64`
- 字符类型，`char`
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是
- 不可变引用 `&T` 

## 函数传值和返回

将值传递给函数，一样会发生 `移动` 或者 `复制`
```
fn main() {
	let s = String::from("hello");   // s 进入作用域 
	takes_ownership(s);              // s 的值移动到函数里 ... 
						             // ... 所以到这里不再有效 
	let x = 5;                       // x 进入作用域 
	makes_copy(x);                   // x 应该移动函数里， 
	                                 // 但 i32 是 Copy 的，所以在后面可继续使用 x
}  // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走， 
   // 所以不会有特殊操作 
   
fn takes_ownership(some_string: String) { // some_string 进入作用域 
	println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放 

fn makes_copy(some_integer: i32) { // some_integer 进入作用域 
	println!("{}", some_integer); 
} // 这里，some_integer 移出作用域。不会有特殊操作
```

同样的，函数返回值也有所有权

```
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}
```