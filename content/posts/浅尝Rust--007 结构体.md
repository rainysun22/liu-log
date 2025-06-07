---
title: 浅尝Rust--007 结构体
date: 2025-06-07T13:40:49+08:00
tags:
  - Rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/compound-type/struct.html](https://course.rs/basic/compound-type/struct.html)
## 结构体

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

## 元组结构体

`结构体必须要有名称，字段可以没有名称`

```
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
```

## 单元结构体

`定义一个类型，不关心类型的内容，只关心它的行为`

```
struct AlwaysEqual;

let subject = AlwaysEqual;

// 我们不关心 AlwaysEqual 的字段数据，只关心它的行为，因此将它声明为单元结构体，然后再为它实现某个特征
impl SomeTrait for AlwaysEqual {

}
```

## 结构体数据的所有权

- 如果结构体字段要使用引用类型，也就是从其它对象借用数据，需要加上生命周期
- 生命周期能确保结构体作用范围要比它借用的数据的作用范围要小

**详见`生命周期`章节**

## 打印结构体信息

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
