---
title: Rust笔记--005 字符串与切片
date: 2025-06-02T12:16:43+08:00
tags:
  - rust
author: liuzifeng
---
> 学习参考：[https://course.rs/basic/compound-type/string-slice.html](https://course.rs/basic/compound-type/string-slice.html)
## 切片

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

## 字符串

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

## 字符串操作

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

## 字符串转义

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

## 遍历字符串

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