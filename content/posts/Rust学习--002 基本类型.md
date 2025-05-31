---
title: Rust学习--002 基本类型
date: 2025-05-31T17:57:22+08:00
tags: 
author: liuzifeng
---
> 学习参考：[https://course.rs/about-book.html](https://course.rs/about-book.html)
## 1. 数值类型
### 整数类型

`Rust的内置数据类型`

| 长度    | 有符号   | 无符号   |
| ----- | ----- | ----- |
| 8位    | i8    | u8    |
| 16位   | i16   | u16   |
| 32位   | i32   | u32   |
| 64位   | i64   | u64   |
| 128位  | i128  | u128  |
| 视架构而定 | isize | usize |
- 有符号类型范围：-( 2<sup>n-1</sup>) ~ 2<sup>n - 1</sup> - 1
- 无符号类型范围：0 ~ 2<sup>n</sup>-1
- isize/usize取决于CPU类型：若CPU是32位的，他俩就是32位的
- isize/usize主要应用：用作集合的索引
- rust整型默认使用i32
```
let i = 1 // i是i32类型
```

`数值字面量可以用以下形式书写`

| 数字字面量     | 示例          |
| --------- | ----------- |
| 十进制       | 98_222      |
| 十六进制      | 0xff        |
| 八进制       | 0o77        |
| 二进制       | 0b1111_0000 |
| 字节(仅限于u8) | b'A'        |
### 整数溢出

`debug模式编译时，rust会检查整型溢出，存在问题则panic`
`使用--release参数编译时，rust不检查溢出，检测到溢出时按照补码循环溢出的规则处理`
- 大于该类型最大值的数值会被补码转换成能够支持的对应数字的最小值
```
let a : u8 = 255;
let b = a.wrapping_add(20); // 溢出->补码
println!("{}", b); // 19
```
显示处理可能的溢出，可以使用标准库针对原始数字类型提供的方法：
- 使用`wrapping_*`方法**在所有模式下**都按照补码循环溢出规则处理
- 使用`checked_*`方法时发生溢出，则返回`None`值
- 使用`overflowing_*`方法返回该值和一个是否存在溢出的布尔值
- 使用`saturating_*`方法，可以限定计算后的结果不超过目标类型的最大值或低于最小值
```
assert_eq!(u8::MAX.saturating_add(127), u8::MAX);
```

### 浮点类型

`f32和f64，默认为f64`
`f32类型是单精度浮点型，f64为双精度`

浮点数陷阱：
- **浮点数往往是数字的近似表达** 
- **浮点数不一定可以比较**，例如：HashMap类型存放的key必须实现std::cmp::Eq特征，而f64只实现了std::cmp::PartialEq特征

为了避免以上陷阱，需要遵守以下规则：
- 避免在浮点数上测试相等性
- 当结果在数学上可能存在未定义时，需要格外的小心
```
assert!(0.1 + 0.2 == 0.3); // 会panic
```

### NaN

`对于数学上未定义的记过，使用NaN(not a number)来处理，且NaN不能被比较`

```
let x = (-42.0_f32).sqrt();  // 对复数取平方根
if x.is_nan() { 
	println!("未定义的数学行为") 
}
```

### 数字运算

`支持加/减/乘/除/取模`

```
// 编译器会进行自动推导，给予twenty i32的类型
let twenty = 20; 
// 类型标注 
let twenty_one: i32 = 21; 
// 通过类型后缀的方式进行类型标注：22是i32类型 
let twenty_two = 22i32; 
// 只有同样类型，才能运算 
let addition = twenty + twenty_one + twenty_two; 
println!("{} + {} + {} = {}", twenty, twenty_one, twenty_two, addition); 
// 对于较长的数字，可以用_进行分割，提升可读性 
let one_million: i64 = 1_000_000; 
println!("{}", one_million.pow(2)); 
// 定义一个f32数组，其中42.0会自动被推导为f32类型 
let forty_twos = [ 
	42.0,
	42f32, 
	42.0_f32, 
]; 
// 打印数组中第一个值，并控制小数位为2位 
println!("{:.2}", forty_twos[0]);
```

### 位运算

| 运算符   | 说明                           |
| ----- | ---------------------------- |
| & 位与  | 相同位置均为1时则为1，否则为0             |
| \| 位或 | 相同位置只要有1时则为1，否则为0            |
| ^ 异或  | 相同位置不相同则为1，相同则为0             |
| ! 位非  | 把位中的0和1相互取反，即0置为1，1置为0       |
| << 左移 | 所有位向左移动指定位数，右位补0             |
| >> 右移 | 所有位向右移动指定位数，带符号移动（正数补0，负数补1） |

### 序列(Range)

`只支持数字或字符类型`

```
// 1..5 生成从1到4的连续数字
for i in 1..5 { 
	println!("{}",i); 
}

// 1..=5 生成从1到5的连续数字，包含5
for i in 1..=5 { 
	println!("{}",i); 
}

// 'a'..='z' 生成从a到z的连续字符，包含z
for i in 'a'..='z' {
    println!("{}",i);
}
```

### 使用As完成类型转换

`As可以将一个类型转换成另一个类型，也可以实现诸如将指针转换为地址、地址转换为指针以及将指针转换为其他指针的操作`

### 有理数和复数

`标准库不包含，使用num库`

按照以下步骤来引入 `num` 库：

1. 创建新工程 `cargo new complex-num && cd complex-num`
2. 在 `Cargo.toml` 中的 `[dependencies]` 下添加一行 `num = "0.4.0"`
3. 将 `src/main.rs` 文件中的 `main` 函数替换为下面的代码
4. 运行 `cargo run`

```
use num::complex::Complex;

 fn main() {
   let a = Complex { re: 2.1, im: -1.2 };
   let b = Complex::new(11.1, 22.2);
   let result = a + b;

   println!("{} + {}i", result.re, result.im)
 }
```

## 2. 字符·布尔·单元类型

### 字符类型(char)

`不仅仅是ASCII，还包含所有的Unicode`
`字符类型占用4个字节`

```
let x = '中'; 
println!("{}",std::mem::size_of_val(&x));  // 4
```

### 布尔(bool)

`true和false，占用1个字节`

### 单元类型

`单元类型是()`
`main函数和println!函数的返回值就是单元类型()`
`可以用()作为map的值，表示不关注value只关注key，不占用任何内存`

补充：无返回值的函数叫**发散函数**，main函数是有返回值的

## 3. 语句和表达式

### 语句

`执行一些操作但不会返回一个值`

```
let a = 8;
let b: Vec<f64> = Vec::new();
let (a, c) = ("hi", false);
```

以上都是语句，完成了一个具体的操作，但没有返回值

### 表达式

`执行一些操作后返回一个值`
`表达式不能包含分号!`
`表达式可以成为语句的一部分`

```
let y = 6;  // 6是表达式，返回一个6
```

`调用一个函数/宏是表达式，用花括号包裹最终返回一个值的语句块也是表达式，因为他们都会返回一个值`

```
let y = {
    let x = 3;  // 语句
    x + 1  // 表达式
};

fn ret_unit_type() {
    let x = 1;
    // if 语句块也是一个表达式，因此可以用于赋值，也可以直接返回
    // 类似三元运算符，在Rust里我们可以这样写
    let y = if x % 2 == 1 {
        "odd"
    } else {
        "even"
    };
    // 或者写成一行
    let z = if x % 2 == 1 { "odd" } else { "even" };
}
```

`表达式如果不返回任何值，会隐式返回一个()`

```
assert_eq!(ret_unit_type(), ())
```

## 4. 函数

### 函数构成

![[Pasted image 20250215222747.png]]

### 函数要点

- 函数名和变量名使用蛇形命名法，例如 fn add_two() -> {}
- 函数位置可以随便放，只要有定义就行
- 每个函数的参数都需要标注类型

```
fn main() {
    another_function(5, 6.1);
}

fn another_function(x: i32, y: f32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

### 函数返回

`函数返回值就是函数体最后一条表达式的返回值，也可以使用return提前返回`

```
fn plus_or_minus(x:i32) -> i32 {
    if x > 5 {
        return x - 5  // return提前返回
    }

    x + 5  // 表达式作为返回值
}
```

无返回值，返回单元类型()
- 函数没有返回值，返回一个()
- 通过;结尾的语句返回一个()

```
use std::fmt::Debug;

fn report<T: Debug>(item: T) {
  println!("{:?}", item);  // 隐式返回()
}

fn clear(text: &mut String) -> () {   // 显式返回()
  *text = String::from("");
}
```

永不返回的发散函数，返回 ! 
- 往往用作导致程序崩溃的函数

```
fn dead_end() -> ! {
  panic!("你已经到了穷途末路，崩溃吧！");
}
```

- 下面的函数创建了一个无限循环，该循环永不跳出，因此函数也永不返回：

```
fn forever() -> ! {
  loop {
    //...
  };
}
```
