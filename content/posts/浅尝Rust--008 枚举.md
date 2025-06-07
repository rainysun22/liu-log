---
title: 浅尝Rust--008 枚举
date: 2025-06-07T15:15:19+08:00
tags:
  - Rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/compound-type/enum.html](https://course.rs/basic/compound-type/enum.html)
## 枚举

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

## Option枚举处理空值

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
