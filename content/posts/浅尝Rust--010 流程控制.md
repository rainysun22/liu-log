---
title: 浅尝Rust--010 流程控制
date: 2025-06-07T16:33:04+08:00
tags:
  - rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/flow-control.html](https://course.rs/basic/flow-control.html)
## 分支控制

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

## 循环控制

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