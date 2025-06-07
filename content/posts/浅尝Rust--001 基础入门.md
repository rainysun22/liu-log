---
title: 浅尝Rust--001 基础入门
date: 2025-06-07T16:48:40+08:00
tags:
  - Rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/intro.html](https://course.rs/basic/intro.html)
## 1.1 变量

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
## 1.2 基本类型

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

### 函数

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

***
## 1.3 所有权

### 所有权原则

1. Rust中每一个值都被一个变量所拥有，该变量称为值的所有者
2. 一个值同时只能被一个变量所拥有
3. 当所有者（变量）离开作用域范围时，这个值将被丢弃

### 变量绑定背后的数据交互

#### part1：`未被转移所有权`

```
let x = 5;
let y = x;
```

**执行逻辑：**
代码首先将**5**绑定到变量**x**，接着拷贝**x**的值赋给**y**，最终**x**和**y**都等于5

**剖析：**
整数是 Rust 基本数据类型，是固定大小的简单值，因此这两个值都是通过**自动拷贝**的方式来赋值的，都被存在栈中，完全无需在堆上分配内存。故不需要所有权转移。

#### part2：`转移所有权`

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

#### part3：`引用`

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

#### part4：`克隆(深拷贝)`

```
let s1 = String::from("hello"); 
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);  // Success
```

`注意：Rust永远不会自动创建数据的深拷贝`

**执行过程：**
使用clone函数，s2完整复制了s1的数据。没有发生所有权转移。

### 可以自动拷贝的类型

`如果一个类型拥有Copy特征，一个旧的变量在被赋值给其他变量后仍然可用，也就是赋值的过程即是拷贝的过程`

- 所有整数类型，比如 `u32`
- 布尔类型，`bool`，它的值是 `true` 和 `false`
- 所有浮点数类型，比如 `f64`
- 字符类型，`char`
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是
- 不可变引用 `&T` 

### 函数传值和返回

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

***
## 1.4 引用

### 引用

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
### 不可变引用

```
fn main() {
    let s1 = String::from("hello");
    let s = &s1;

    s.push_str("Rust");  // Error
}
```

如果想修改上述代码中的引用，会发生错误。`引用指向的值默认不可变`

### 可变引用

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

### 悬垂引用

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

***
## 1.5 字符串与切片

### 切片

1. 对于字符串而言，切片就是对String类型中某一部分的引用

	`切片范围是一个左闭右开区间`

```
let s = String::from("hello world");

let hello = &s[0..5];  // hello
let world = &s[6..11];  // world
```

![](/images/Rust笔记--005%20字符串与切片.png)

2. 各种范围的切片：

```
let s = String::from("hello");

let slice = &s[0..2];  // hel
let slice = &s[..2];   // 等价于0..2

let len = s.len();
let slice = &s[4..len]; // o
let slice = &s[4..];    // 等价于4..len

let slice = &s[0..len];  // hello
let slice = &s[..];      // 等价于0..len
```

3. 对字符串使用切片语法时需要格外小心，切片的索引必须落在字符之间的边界位置。

```
 let s = "中国人";
 let a = &s[0..2];  // Error
 println!("{}",a);
```

由于中文在UTF-8中每个字符占3个字节，故上述代码会报错。

4. 切片与所有权
```
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
fn first_word(s: &String) -> &str {
    &s[..1]
}
```

clear()函数需要使用s的可变引用，println!需要使用不可变引用，由于`所有权原则2：可变引用和不可变引用不能同时存在`，故报错。

5. 字符串字面量是切片类型

```
let s: &str = "Hello, world!";
```

因为&str是一个不可变引用，所以字符串字面量是不可变的。

### 字符串

Rust中的字符是UniCode类型，每个字符占据4个字节内存空间。

字符串是UTF-8编码，每个字符所占字节数是变化的(1-4)

1. String与&str的转换

```
// &str转String
String::from("hello,world");
"hello,world".to_string();

// String转&str
fn main() {
    let s = String::from("hello,world!");
    say_hello(&s);
    say_hello(&s[..]);
    say_hello(s.as_str());
}

fn say_hello(s: &str) {
    println!("{}",s);
}
```

2. 不能使用字符串索引

```
let s1 = String::from("hello");
let h = s1[0];  // Error
```

对于“中国人“这样的字符串，取`s1[0]`取不了，因为每个字符3个字节

### 字符串操作

1. 追加Push

`在原有字符串上追加，要修改原来字符串，所以字符串必须是可变的`

```
fn main() {
    let mut s = String::from("Hello ");

    s.push_str("rust");
    println!("追加字符串 push_str() -> {}", s);

    s.push('!');
    println!("追加字符 push() -> {}", s);
}
```

2. 插入Insert

`修改原有字符串，所以字符串必须是可变的`

`第一个参数是插入位置，第二个参数是要插入的字符/字符串`

```
fn main() {
    let mut s = String::from("Hello rust!");
    s.insert(5, ',');
    println!("插入字符 insert() -> {}", s);
    s.insert_str(6, " I like");
    println!("插入字符串 insert_str() -> {}", s);
}
```

3. 替换Replace

`返回一个新的字符串`

- replace

`适用String和&str类型。第一个参数是要被替换的字符串，第二个参数是新的字符串。该方法会替换所有匹配到的字符串。`

```
fn main() {
    let string_replace = String::from("I like rust. Learning rust is my favorite!");
    let new_string_replace = string_replace.replace("rust", "RUST");
    dbg!(new_string_replace);
}
```

- replacen

`适用String和&str类型。前两个参数和replace一样，第三个参数表示替换的个数。`

```
fn main() {
    let string_replace = "I like rust. Learning rust is my favorite!";
    let new_string_replacen = string_replace.replacen("rust", "RUST", 1);
    dbg!(new_string_replacen);
}
```

- replace_range

`仅适用于String类型。第一个参数是要替换字符串的范围，第二个参数是新的字符串`

```
fn main() {
    let mut string_replace_range = String::from("I like rust!");
    string_replace_range.replace_range(7..8, "R");
    dbg!(string_replace_range);
}
```

4. 删除Delete

`仅适用于String类型`

- pop

`删除并返回字符串最后一个字符。直接操作原有字符串，返回值是Option类型，如果字符串为空，返回None`

```
fn main() {
    let mut string_pop = String::from("rust pop 中文!");
    let p1 = string_pop.pop();
    let p2 = string_pop.pop();
    dbg!(p1);
    dbg!(p2);
    dbg!(string_pop);
}
```

- remove

`删除并返回字符串中指定位置的字符。直接操作原有字符串，按字节处理`

```
fn main() {
    let mut string_remove = String::from("测试remove方法");
    println!(
        "string_remove 占 {} 个字节",
        std::mem::size_of_val(string_remove.as_str())
    );
    // 删除第一个汉字
    string_remove.remove(0);
    // 下面代码会发生错误
    // string_remove.remove(1);
    // 直接删除第二个汉字
    // string_remove.remove(3);
    dbg!(string_remove);
}
```

- truncate

`删除字符串中从指定位置开始到结尾的全部字符。直接操作原有字符串，无返回值。按字节处理字符串`

```
fn main() {
    let mut string_truncate = String::from("测试truncate");
    string_truncate.truncate(3);
    dbg!(string_truncate);
}
```

- clear

`清空字符串，相当于truncate(0)`

```
fn main() {
    let mut string_clear = String::from("string clear");
    string_clear.clear();
    dbg!(string_clear);
}
```

5.连接

- 适用+或+=连接字符串
相当于调用了add方法，add方法的第二个参数是引用类型。所以要求+/+=右边的参数必须为字符串的切片引用类型。连接返回一个新的字符串。

```
fn main() {
    let string_append = String::from("hello ");
    let string_rust = String::from("rust");
    // &string_rust会自动解引用为&str
    let result = string_append + &string_rust;
    let mut result = result + "!"; // `result + "!"` 中的 `result` 是不可变的
    result += "!!!";    // 适用+=要求可变

    println!("连接字符串 + -> {}", result);
}
```

使用+操作符会转移所有权：`s1的所有权被转移到add方法中`
```
fn main() {
    let s1 = String::from("hello,");
    let s2 = String::from("world!");
    // 在下句中，s1的所有权被转移走了，因此后面不能再使用s1
    let s3 = s1 + &s2;
    assert_eq!(s3,"hello,world!");
    // 下面的语句如果去掉注释，就会报错
    // println!("{}",s1);
}
```

- 使用format!连接字符串

`适用于String和&str`

```
fn main() {
    let s1 = "hello";
    let s2 = String::from("rust");
    let s = format!("{} {}!", s1, s2);
    println!("{}", s);
}
```

### 字符串转义

```
fn main() {
    // 通过 \ + 字符的十六进制表示，转义输出一个字符
    let byte_escape = "I'm writing \x52\x75\x73\x74!";
    println!("What are you doing\x3F (\\x3F means ?) {}", byte_escape);

    // \u 可以输出一个 unicode 字符
    let unicode_codepoint = "\u{211D}";
    let character_name = "\"DOUBLE-STRUCK CAPITAL R\"";

    println!(
        "Unicode character {} (U+211D) is called {}",
        unicode_codepoint, character_name
    );

    // 换行了也会保持之前的字符串格式
    // 使用\忽略换行符
    let long_string = "String literals
                        can span multiple lines.
                        The linebreak and indentation here ->\
                        <- can be escaped too!";
    println!("{}", long_string);
}
```

```
fn main() {
    println!("{}", "hello \\x52\\x75\\x73\\x74");
    let raw_str = r"Escapes don't work here: \x3F \u{211D}";
    println!("{}", raw_str);

    // 如果字符串包含双引号，可以在开头和结尾加 #
    let quotes = r#"And then I said: "There is no escape!""#;
    println!("{}", quotes);

    // 如果字符串中包含 # 号，可以在开头和结尾加多个 # 号，最多加255个，只需保证与字符串中连续 # 号的个数不超过开头和结尾的 # 号的个数即可
    let longer_delimiter = r###"A string with "# in it. And even "##!"###;
    println!("{}", longer_delimiter);
}
```

### 遍历字符串

1. 以Unicode字符方式

```
for c in "中国人".chars() {
    println!("{}", c);
}
```

2. 以字节方式

```
for b in "中国人".bytes() {
    println!("{}", b);
}
```

***
## 1.6 元组

### 概念

由多种类型组合到一起的复合类型


```
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

### 用模式匹配解构元组


```
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

解构：将元组中的值按顺序绑定到对应的变量上

### 用.访问元组


```
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

### 常用示例

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

***
## 1.7 结构体

### 基本用法

1. 定义结构体

```
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

2. 创建结构体实例

```
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
```

**注意：**
- 初始化结构体时，每个字段都要初始化
- 初始化的字段顺序无需和结构体定义字段顺序保持一致

3. 访问结构体字段

```
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
```

4. 简化结构体创建

```
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

当参数名称和结构体字段同名时，可以适用缩略的方式进行初始化

```
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

5. 结构体更新语法

`根据已有结构体实例创建新的结构体实例`

```
  let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };

```

结构体更新语法：

```
  let user2 = User {
        email: String::from("another@example.com"),
        ..user1  // 凡是没声明的字段，全部从user1获取值
    };
```

**注意：**
- `..user1`必须在尾部使用
- `user1`的部分字段所有权被转移到`user2`中：`部分字段(基本类型除外)`和`user1`均无法再被使用

示例：

```
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
let user2 = User {
    active: user1.active,
    username: user1.username,
    email: String::from("another@example.com"),
    sign_in_count: user1.sign_in_count,
};
println!("{}", user1.active);
// 下面这行会报错
println!("{:?}", user1);
```

### 元组结构体

`结构体必须要有名称，字段可以没有名称`

```
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
```

### 单元结构体

`定义一个类型，不关心类型的内容，只关心它的行为`

```
struct AlwaysEqual;

let subject = AlwaysEqual;

// 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {

}
```

### 结构体数据的所有权

- 如果结构体字段要使用引用类型，也就是从其它对象借用数据，需要加上生命周期
- 生命周期能确保结构体作用范围要比它借用的数据的作用范围要小

**详见`生命周期`章节**

### 打印结构体信息

1. 使用`println!("{}", a);`进行输出需要实现`display特征`
2. 结构体没有实现`display特征`，使用`{}`打印结构体，需要自己实现`display特征`
3. 可以使用给结构体实现`debug特征`并且用`{:?}`打印结构体

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}

输出：
$ cargo run
rect1 is Rectangle { width: 30, height: 50 }
```

4. 简单的使用`dbg!宏`打印

`dbg!宏会拿走表达式的所有权，打印信息，最终还会把表达式值的所有权返回`

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}

输出：
$ cargo run
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
```


```
fn main() {
    let mut user1 = User {
        username: String::from("hello"),
        email: String::from("he"),
        active: true,
    };
    user1 = dbg!(user1);
    println!("{}", user1.username);

}
```

***
## 1.8 枚举

### 基础用法

1. 基本定义

```
enum PokerSuit {
  Clubs,
  Spades,
  Diamonds,
  Hearts,
}
```

2. 创建实例

```
fn main() {
	// heart和diamond是PockerSuit类型
    let heart = PokerSuit::Hearts;
    let diamond = PokerSuit::Diamonds;

    print_suit(heart);
    print_suit(diamond);
}

fn print_suit(card: PokerSuit) {
    // 需要在定义 enum PokerSuit 的上面添加上 #[derive(Debug)]，否则会报 card 没有实现 Debug
    println!("{:?}",card);
}
```

3. 枚举带值

```
enum PokerCard {
    Clubs(u8),
    Spades(u8),
    Diamonds(u8),
    Hearts(u8),
}

fn main() {
   let c1 = PokerCard::Spades(5);
   let c2 = PokerCard::Diamonds(13);
}
```

4. 任何类型的数据都可以放入枚举成员中

```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let m1 = Message::Quit;
    let m2 = Message::Move{x:1,y:1};
    let m3 = Message::ChangeColor(255,255,0);
}
```

- Quit没有关联任何数据
- Move包含一个匿名结构体
- Write包含一个String字符串
- ChangeColor包含三个i32

### Option枚举处理空值

1. `Option` 枚举包含两个成员，一个成员表示含有值：`Some(T)`, 另一个表示没有值：`None`，定义如下：

```
enum Option<T> {
    Some(T),  // T代表泛型
    None,
}
```

2. 由于Option被包含在系统库中，无需使用`Option::`，直接使用Some和None

```
let some_number = Some(5);
let some_string = Some("a string");

// None值需要告诉编译器是什么类型
let absent_number: Option<i32> = None;
```

**注意：**
- Rust拥有一个像`i8`这样的值的时候，编译器确保它总是一个有效的值。当使用`Option<i8>`的时候才需要担心可能没有值，编译器会确保我们在使用值之前处理了为空的情况。
- 在对`Option<T>`进行T的运算之前必须将其转换为T（这通常能帮我们捕捉到**期望某值不为空但实际为空的情况**）

3. 当一些代码需要在拥有`Some(T)`时运行，允许代码使用其中的T；另一些代码需要在值为None的时候运行，使用`模式匹配`

```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

***
## 1.9 数组

### 基础知识

`Rust的基本类型`

- 长度固定
- 元素必须有相同的类型
- 依次线性排列

### 数组使用

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

### 数组切片

```
let a: [i32; 5] = [1, 2, 3, 4, 5];

let slice: &[i32] = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

### 使用示例

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

***
## 1.10 流程控制

### 分支控制

`if..else...`

```
if condition == true {
    // A...
} else {
    // B...
}
```

注意：
- if语句块是表达式，可以使用if表达式的返回值给变量赋值
- 用if赋值时，要保证每个分支返回的类型一样

```
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

- 如果if有多个分支都能匹配，只有第一个匹配的分支能被执行，执行后就跳出

### 循环控制

1. for循环

| 使用方法                          | 等价使用方式                                            | 所有权          |
| ----------------------------- | ------------------------------------------------- | ------------ |
| `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权到for循环中 |
| `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用        |
| `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用         |
如果想在for循环中**获取元素的索引**：

```
fn main() {
    let a = [4, 3, 2, 1];
    // `.iter()` 方法把 `a` 数组变成一个迭代器
    for (i, v) in a.iter().enumerate() {
        println!("第{}个元素是{}", i + 1, v);
    }
}
```

两种循环方式的比较：

```
// 第一种
let collection = [1, 2, 3, 4, 5];
for i in 0..collection.len() {
  let item = collection[i];
  // ...
}

// 第二种
for item in collection {

}
```

- 第一种方式是循环索引，通过索引下标访问；第二种方式是直接循环集合元素
- 性能：
	- 第一种中`collection[index]`的索引访问，会因为**边界检查**导致运行时性能损耗
	- 第二种直接迭代的方式不会触发检查，在编译期完成分析并证明这种访问合法
- 安全：
	- 第一种方式里对`collection`的索引访问是非连续的，有可能两次访问之间`collection`发生变化，导致脏数据产生。也就是说可能循环内部会修改`collection`
	- 第二种直接迭代方式是连续访问，不存在这种风险。（由于所有权限制，访问过程中数据不会发生变化）

continue可以跳过当次循环，开始下次循环：

```
 for i in 1..4 {
     if i == 2 {
         continue;
     }
     println!("{}", i);
 }
```

break可以直接跳出当前整个循环：

```
 for i in 1..4 {
     if i == 2 {
         break;
     }
     println!("{}", i);
 }
```

2. while循环

```
fn main() {
    let mut n = 0;

    while n <= 5  {
        println!("{}!", n);

        n = n + 1;
    }

    println!("我出来了！");
}
```

while循环和for循环作比较：

```
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}

fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

- while循环的index长度不正确会导致程序panic，由于运行期检查，程序也更慢
- for循环不用索引访问，避免运行时边界检查，性能更高

3. loop循环

```
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

注意：
- break可以单独使用，也可以**带一个返回值**，有些类似return
- loop是一个表达式，因此可以返回一个值

***
