---
layout: post
title: Rust里常用的字符串操作
date: 2022-05-12 22:45:00
tags: [Rust]
---
# 前言

字符串应该是写代码时绝对会用到的类型了，在这里记录一下常用的一些字符串操作方法。

# 字符串类型

Rust里常见的两种字符串类型，一种是**&str**，还有一种是**String**

## &str

Rust里的基础数据类型，这是**str**类型的切片类型，如同**i32**类型和**&i32**，创建方式如下所示

```rust
let s:&str = "HelloWorld";
```

## String

本质上是一个Struct，可以动态增删字符，如同C++里的**char**和**string**，其定义如下所示

```rust
pub struct String {
    vec: Vec<u8>,
}
```

常见的创建方式是从**&str**类型创建

```rust
let s: String = String::from("HelloWorld");
```

# 查找字符串

用于查找String中是否包含给定的字符串，输入是&str类型

```rust
    let s = String::from("HelloWorld");
    println!("{:?}", s.contains("World"));//true
```

# 转为&str

有时候需要&str类型的字符串，可以通过如下方法转换

```rust
    let s = String::from("HelloWorld");
    println!("{:?}",s.as_str());
```

# 分割字符串

```rust
    let s = String::from("HelloWorld");
    let m = s.split("l");
    for x in m {
        println!("{:?}", x);
    }
```

```
"He"
""
"oWor"
"d"
```

# 寻找字符串

```rust
fn main() {
    let s = String::from("HelloWorld");
    let m = s.find("Wo");
    println!("{:?}", m.unwrap());
}
```

```
5
```

如果找不到的话，那么find的返回结果是Option::none，那么unwrap就会抛出异常，所以需要对结果判断一下

```rust
    let s = String::from("HelloWorld");
    let m = s.find("Woa");
    println!("{:?}", m.unwrap());
```

```
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src/main.rs:7:24
```

正确处理方法如下

```rust
    let s = String::from("HelloWorld");
    let m = s.find("Woa");
    if m.is_some() {
        println!("{:?}", m.unwrap());
    }else{
        println!("找不到目标字符串")
    }
```

```
找不到目标字符串
```

# 索引字符串

先转为**&str**类型，再进行切片操作

```rust
    let s = String::from("HelloWorld");
    let g = s.as_str();
    println!("{:?}", &g[0..3]);
```

```
"Hel"
```

