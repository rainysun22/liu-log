---
title: Rust笔记--002 基础入门(中)
date: 2025-06-22T14:38:54+08:00
tags:
  - Rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/intro.html](https://course.rs/basic/intro.html)

## 1.1 模式匹配

### match匹配

1. 通用形式

```
match target {
    模式1 => 表达式1,
    模式2 => {
        语句1;
        语句2;
        表达式2
    },
    _ => 表达式3
}
```

2. 示例

```
enum Direction {
    East,
    West,
    North,
    South,
}

fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        _ => (),
    };
}
或
fn main() {
    let dire = Direction::South;
    match dire {
        Direction::East => println!("East"),
        Direction::North | Direction::South => {
            println!("South or North");
        },
        other => (),
    };
}
```

注：
- match匹配到第一个符合条件的，执行代码返回；如果当前分支不匹配，则继续执行下一个分支
- match必须穷尽所有的类型，不想关心的值用`_`或`一个变量`代替

3. 用match表达式赋值

```
enum IpAddr {
   Ipv4,
   Ipv6
}

fn main() {
    let ip1 = IpAddr::Ipv6;
    let ip_str = match ip1 {
        IpAddr::Ipv4 => "127.0.0.1",
        _ => "::1",
    };

    println!("{}", ip_str);
}
```

match本身也是一个表达式，可以用它来赋值

4. 模式绑定

```
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), // 25美分硬币
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

模式匹配的一个重要功能就是从模式中取出绑定的值，如上面代码的state

### if let匹配

`有时会遇到只有一个模式的值需要被处理，其他值直接忽略的场景，用match就太繁琐了`

```
let v = Some(3u8);
match v {
	Some(3) => println!("three"),
	_ => (),
}
```

上述代码只想对`Some(3)`进行匹配，不想处理其他`Some(u8)`或`None`值，用if let就正好了。

```
if let Some(3) = v {
    println!("three");
}
```

### matches!宏

`可以将一个表达式和模式进行匹配`

```
enum MyEnum {
    Foo,
    Bar
}

fn main() {
    let v = vec![MyEnum::Foo,MyEnum::Bar,MyEnum::Foo];
}
```

如果想过滤v，只保留`MyEnum::Foo`类型的元素，你可能想这么写：

```
v.iter().filter(|x| x == MyEnum::Foo);
```

但是这样会报错，因为无法将`x`和一个枚举成员进行比较，这时就用到`matches!`

```
v.iter().filter(|x| matches!(x, MyEnum::Foo));
```

其他案例：

```
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));

let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```

### 模式匹配的所有权转移

match和if let如果要匹配带所有权的变量，所有权会被转移

```
let s = Some(String::from("Hello!"));

if let Some(y) = s {
    println!("found a string");
}

println!("{:?}", s);  // 报错,s的所有权被转移到y了
```

### 解构Option

使用`Option<T>`，为了从Some中取出其内部的`T`值以及处理没有值的情况

```
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);  // Some(6)
let none = plus_one(None);  // None
```

### 模式概念

模式是Rust中的特殊语法，用来匹配类型中的结构和数据。模式一般由以下内容组成：
- 字面值
- 解构的数组、枚举、结构体或元组
- 变量
- 通配符
- 占位符

### 所有可能用到模式的地方

1. match分支

```
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

2. if let分支

`允许匹配一种模式，而忽略其他模式。即可驳模式匹配`

```
if let PATTERN = SOME_VALUE {

}
```

3. while let条件循环

`允许只要模式匹配就一直进行while循环`

```
// Vec是动态数组
let mut stack = Vec::new();

// 向数组尾部插入元素
stack.push(1);
stack.push(2);
stack.push(3);

// stack.pop从数组尾部弹出元素
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

4. for循环

```
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

迭代器每次迭代返回一个`(索引, 值)`形式的元组，用`(index, value)`来匹配

5. let语句

```
let PATTERN = EXPRESSION;
```

let语句也是一种模式匹配：

```
let x = 5;
```

同时，它也是一种模式绑定，代表**将匹配的值绑定到变量上**。在Rust中，**变量名也是一种模式**

6. 函数参数

```
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

7. let-else

`适用else分支来处理模式不匹配的情况，但else分支中必须用发散的代码块处理(如：break、return、panic)`

```
use std::str::FromStr;

fn get_count_item(s: &str) -> (u64, &str) {
    let mut it = s.split(' ');
    let (Some(count_str), Some(item)) = (it.next(), it.next()) else {
        panic!("Can't segment count item pair: '{s}'");
    };
    let Ok(count) = u64::from_str(count_str) else {
        panic!("Can't parse integer: '{count_str}'");
    };
    // error: `else` clause of `let...else` does not diverge
    // let Ok(count) = u64::from_str(count_str) else { 0 };
    (count, item)
}

fn main() {
    assert_eq!(get_count_item("3 chairs"), (3, "chairs"));
}
```

与match和if let相比，let-else特点在于其解包成功时所创建的变量具有更广的作用域

```
// if let
if let Some(x) = some_option_value {
    println!("{}", x);  // x只在if分支使用
}

// let-else
let Some(x) = some_option_value else { return; }
println!("{}", x);  // x可以在let之外使用
```

***
## 1.2 全模式列表

### 匹配字面值

```
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### 匹配命名变量

```
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),  // 变量遮蔽
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

在match作用域中有个新变量`y`，遮蔽了main作用域中的`y`

所以这里的match语句会打印`Matched，y = 5`

如果不想变量遮蔽，可以换个变量名或者使用**匹配守卫**的方式
### 通过序列`..=`匹配值的范围

```
let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

let x = 'c';

match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

### 解构并分解值

```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

这段代码创建了变量`a`和`b`来匹配结构体`p`中的`x`和`y`字段，可以简写成：

```
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

也可以解构一部分，匹配一部分：

```
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),  //匹配x为0的情况，解构y
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

### 解构枚举

```
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

### 解构嵌套的结构体和枚举

```
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }
}
```

### 解构结构体和元组

```
struct Point {
     x: i32,
     y: i32,
 }

let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

### 解构数组

定长数组：

```
let arr: [u16; 2] = [114, 514];
let [x, y] = arr;

assert_eq!(x, 114);
assert_eq!(y, 514);
```

不定长数组：

```
let arr: &[u16] = &[114, 514];

if let [x, ..] = arr {
    assert_eq!(x, &114);
}

if let &[.., y] = arr {
    assert_eq!(y, 514);
}

let arr: &[u16] = &[];

assert!(matches!(arr, [..]));
assert!(!matches!(arr, [x, ..]));
```

### 忽略模式中的值

1. 用`_`忽略整个值

```
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

2. 使用嵌套的`_`忽略部分值

```
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```

```
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

3. 使用下划线开头忽略未使用变量

```
fn main() {
    let _x = 5;
    let y = 10;
}
```

只使用`_`和使用以下划线开头的名称有些不同：`_x`仍会将值绑定到变量，而`_`完全不会绑定

```
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);  // 由于_s真的绑定了值，s的所有权被转移，这里会报错
```


```
let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);  // Success
```

4. 用`..`忽略剩余值

`对于有多个部分的值，可以用..只使用部分值而忽略其他值，这样不用为了多个值列出_`

```
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

用`..`必须是无歧义的，例如：

```
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {  // 报错
            println!("Some numbers: {}", second)
        },
    }
}
```

### 匹配守卫

`一个位于match分支模式之后的额外if条件，能为分支模式提供进一步的匹配条件`

```
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

也可以解决模式中变量遮蔽的问题：

```
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {}", x, y);
}
```

也可以在匹配守卫中使用`|`运算符来指定多个模式，同时**匹配守卫的条件会作用域所有模式**

```
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),  // 等于(4 | 5 | 6) if y => ...
    _ => println!("no"),
}
```

### @绑定

`@运算符允许为一个字段绑定另一个变量`

下面这个例子通过在`3..=7`之前指定`id_variable @`，捕获任何匹配此范围的值并同时**将该值绑定到变量`id_variable`上**。第二个分支匹配`10..=12`的值，但是无法使用id，因为没有将id值保存进一个变量。

```
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

使用@还可以在绑定新变量的同时，对目标进行解构：

```
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    // 绑定新变量 `p`，同时对 `Point` 进行解构
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);


    let point = Point {x: 10, y: 5};
    if let p @ Point {x: 10, y} = point {
        println!("x is 10 and y is {} in {:?}", y, p);
    } else {
        println!("x was not 10 :(");
    }
}
```

rust1.53后的版本可以不加括号绑定到所有模式上：

```
fn main() {
    match 1 {
        num @ 1 | 2 => {  // 等于 num @ (1 | 2)
            println!("{}", num);
        }
        _ => {}
    }
}
```

***
## 1.3 方法

### 概念
Rust 的方法往往跟结构体、枚举、特征(Trait)一起使用
```
object.method()
```

### 定义方法
Rust 使用 impl 来定义方法
```
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    // new是Circle的关联函数，因为它的第一个参数不是self，且new并不是关键字
    // 这种方法往往用于初始化当前结构体的实例
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }

    // Circle的方法，&self表示借用当前的Circle结构体
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

### self、&self 和 &mut self
```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // &self 其实是 self: &Self 的简写。
    // 在一个 impl 块内，Self 指代被实现方法的结构体类型，self 指代此类型的实例。
    // 换句话说，self 指代的是 Rectangle 结构体实例
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```
- self 表示 Rectangle 的所有权转移到该方法中，这种形式用的较少
- &self 表示该方法对 Rectangle 的不可变借用
- &mut self 表示该方法对 Rectangle 的可变借用

### 方法名跟结构体字段名相同
`一般来说，方法和字段同名，往往适用于实现getter访问器`

```
mod my {
    // 当从模块外部访问结构体时，结构体的字段默认是私有的
    pub struct Rectangle {
        width: u32,
        pub height: u32,
    }

    impl Rectangle {
        // 想从模块外部获取Rectangle的字段，只需把它的new，width和height方法设置为pub
        pub fn new(width: u32, height: u32) -> Self {
            Rectangle { width, height }
        }
        pub fn width(&self) -> u32 {
            return self.width;
        }
        pub fn height(&self) -> u32 {
            return self.height;
        }
    }
}

fn main() {
    let rect1 = my::Rectangle::new(30, 50);

    println!("{}", rect1.width()); // OK
    println!("{}", rect1.height()); // OK
    // println!("{}", rect1.width); // Error - the visibility of field defaults to private
    println!("{}", rect1.height); // OK
}
```

### 带有多个参数的方法
```
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

### 关联函数
`定义在impl中且没有self的函数被称之为关联函数。`

`它没有self，不能用f.read()的形式调用，因此它是一个函数而不是方法，它又在impl中，与结构体紧密关联，因此称为关联函数`

```
impl Rectangle {
    fn new(w: u32, h: u32) -> Rectangle {
        Rectangle { width: w, height: h }
    }
}

// 因为是函数，只能通过::调用
// 这个方法位于结构体的命名空间中，:: 语法用于关联函数和模块创建的命名空间
let sq = Rectangle::new(3, 3);
```

### 为枚举实现方法
```
#![allow(unused)]
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

***
## 1.4
