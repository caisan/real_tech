## Rust 学习笔记

参考书：https://kaisery.github.io/trpl-zh-cn/ch03-03-how-functions-work.html


## 0.概述

Rust是编译型语言，rustc是编译器，cargo是项目管理工具。rustup doc可以查看本地文档。

## 1. cargo

cargo换源：https://lug.ustc.edu.cn/wiki/mirrors/help/rust-crates

#### 1.1 Cargo.toml

cargo期望源文件位于src目录。cargo需要一个配置文件`Cargo.toml`，toml即(Tom's Obvious, Minimal Language)，格式类似于ini文件。下面是一个例子：

```
[package]

name = "hello_world"
version = "0.0.1"
authors = ["jczhang <zhjcyx@gmail.com>"]
```

#### 1.2 命令

建好Cargo.toml文件之后，可以用`cargo build`进行程序构建。生成的二进制路径应该是`target/debug/hello_world`。也可以合并这两步直接用`cargo run`编译和执行二进制程序。注意，这样使debug版本，如果要生成release版本，应该用`cargo build --release`,这样二进制路径应该是`target/release/hello_world`。Release编译速度较慢，但程序执行会更快。

Cargo也提供建立“骨架项目”的命令，如：
```
cargo new hello_world
```
这个命令会创建一个`Cargo.toml`和一个包含`main.rs`的`src目录`。`main.rs`已经写好了一行 `println!("Hello, world!");` 代码，并且项目目录被创建好了一个**git仓库**。

`cargo check`可以检查是否可以正确编译而不编译，这样可以比build更快地检查语法错误。


## 2. 杂项

* `//` 表示注释。

* `use` 引入包含的包。

* 缩进用**空格而非tab**。

* 声明一个变量用`let`，如：
```rust
let foo = 0;        // 可变
let mut bar = 1;    // 不可变
let mut guess = String::new() // 创建可变变量，并绑定到新的String空实例上。
```

* 用`println!`进行打印时，占位符是`{}`。其他和C语言`printf`很类似。例：
```rust
println!("x = {} and y = {}", x, y);
```

* crate是rust的库或者二进制，分别称为库crate，或者二进制crate。crate中又包含有trait。 运行 `cargo doc --open` 命令来构建所有本地依赖提供的文档，并在浏览器中打开，这样可以确定包含哪个trait和调用声明方法。
```rust
use rand::Rng // rand是一个库crate，Rng是rand的一个trait
```

## 3. 变量

#### 3.1 可变，不可变和常量

Rust的**变量**分为可变(mutable)和不可变(immutable)变量，默认是不可变的。与不可变量类似的是**常量**(const)，但是常量只能以常量表达式初始化，并且需要指明类型。例：


```rust
let x = 5;                          // 不可变量
let mut y = 6;                      // 可变量   
const MAX_POINTS: u32 = 100_000;    // 常量，注意数字中的下划线是为了可读性
```

#### 3.2 隐藏 (Shadowing)

Rust中变量声明可以重名，先声明的量会被后声明的同名量**隐藏** (Shadowing)。

利用“隐藏”，可以“改变”不可变量的值。这种改变方法，其实比改变可变量更灵活，因为实质上我们可以创建一个同名不同类型的新的不可变量，例如：
```rust
let spaces = "   ";
let spaces = spaces.len();
```

#### 3.3 数据类型

Rust是静态类型语言，即编译时需要确定所有变量的类型。

* 整形：

|长度(bits)|有符号|无符号|
|-|-|-|
|8|i8|u8|
|16|i16|u16|
|32|i32|u32|
|64|i64|u64|
|arch|isize|usize|

(对于isize/usize，若在64位机器上即使64位，否则为32位。)

* 整形字面值：

|数字字面值	|例子|
|-|-|
|Decimal|	98_222|
|Hex	|0xff|
|Octal|	0o77|
|Binary|	0b1111_0000|
|Byte (u8 only)|	b'A'|


* 浮点型：

|长度(bit)|类型|
|-|-|
|32|f32|
|64|f64|

* 布尔型： `bool`，只有两个可能值：`true`和`false`

* 字符型： `char`， Rust中的char并非1个字节，它支持unicode。char字符用单引号包围。如：
```rust
let c = 'z';
let z = '哈';
let heart_eyed_cat = '😻';
```

### 3.4. 复合类型

Rust 复合类型包括tuple和array：

* tuple：

```rust
// 声明tuple：
let foo = (500, 6.4, 1);
let x: (i32, f64, u8) = (500, 6.4, 1);
// 解构tuple:
let (x, y, z) = foo;
// 索引tuple，注意是用“.” 而非广泛使用的中括号
let five_hundred = x.0;
let one = x.2;
```
* Array

```rust
// 声明一个数组
// 注意类型后加分号，再加数字，这里说明a数组包含了5个i32类型的元素
let a: [i32; 5] = [1, 2, 3, 4, 5];
// 数组的索引是用中括号 "[]"
let first = a[0];
```

注意：如果索引超出了数组长度，Rust 会 panic，这是 Rust 术语，它用于程序因为错误而退出的情况。


### 4. 函数

* 函数参数的例子：

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

和C/C++ 一样，定义函数时必须要注明参数类型。

* 函数返回的例子：

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

### 5. 分支和循环

分支主要有if，循环结构有loop、for和while。

**if** 例1：
```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```
**if** 例2：
```rust
fn main() {
    let condition = true;
    // 注意： 这种情况下，变量必须只有一个类型
    // 比如5和6这里都是i32类型的
    let number = if condition {
        5
    } else {
        6
    };
    println!("The value of number is: {}", number);
}
```

**loop** 例1：

```rust
loop {
    println!("again!");
}
```
**loop** 例2：

```rust
let result = loop {
    counter += 1;

    if counter == 10 {
        break counter * 2;
    }
};
```

**while** 例：

```rust
while number != 0 {
    println!("{}!", number);

    number = number - 1;
}
```

**for** 例1：

```rust
let a = [10, 20, 30, 40, 50];
for element in a.iter() {
    println!("the value is: {}", element);
}
```

**for** 例2：

```rust
// rev 即反向
for number in (1..4).rev() {
    println!("{}!", number);
}
```

## 6. 所有权系统

所有权系统是Rust与GC语言或手动内存管理语言的主要区别。

跟踪哪部分代码正在使用**堆**上的哪些数据，最大限度的减少堆上的重复数据的数量，以及清理堆上不再使用的数据确保不会耗尽空间，这些问题正是所有权系统要处理的。

几个所有权基本规则：
```
Rust 中的每一个值都有一个被称为其 所有者（owner）的变量。
值有且只有一个所有者。
当所有者（变量）离开作用域，这个值将被丢弃。
```

对于一个字符串，字面值的字符串是直接硬编码到代码中并存到栈中的，而可变长的String类型必须是运行时从系统分配的堆中的内存，并且要在合适的时候将内存返还给操作系统。

Rust 在变量离开作用域时自动回收内存，它会在`}`处自动调用一个称谓drop的函数。

* 参数传递和所有权

只考虑堆上分配变量的情况，比如当把一个堆上变量x从foo函数传给一个bar函数，bar会接管x所有权，这个变量x也会在被调用函数结束时被drop调，除非它以返回值的形式再传回给foo。例：

```rust
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

* 引用和所有权借出

不过，还可以用引用的形式在传参的时候不转移所有权，只是将变量借出去，这样所有权在函数返回后仍然在，x也没有被bar释放。但是有一个局限是，bar中不能更改x(只读？)。引用(`&`)和解引用(`*`)的符号和C++很像，但注意传递传递发起端也要加引用符号`&`。例：

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1); // 这里返回后，s1的所有权仍是main的

    println!("The length of '{}' is {}.", s1, len);
}
fn calculate_length(s: &String) -> usize {
    s.len()
} // 这里s并不会被drop
```

要想让所引用的变量在被“借出”过程中可变，还是需要在参数列表中加入`mut`关键字(即用`&mut`代替`&`)。例：

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s); // 注意，传参是也要加 &mut
}
fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

还有一个限制，对于同一份堆上变量(数据)，在同一作用域中只能由它的一份可变引用，或者多份不可变引用。这感觉上就类似读写锁，可以有多个读锁被同时持有，读和写不能同时持有，写和写也不能。Rust正是利用了这个思想，在编译时就防止了可能的数据竞争。

还有一点需要注意的是，Rust不允许以引用作为返回值来外借所有权，因为返回是生命周期会结束，这回造成dangling pointer悬停指针，Rust不允许这样。例：
```rust
// 错误的：
fn dangle() -> &String { // dangle 返回一个字符串的引用
    let s = String::from("hello"); // s 是一个新字符串
    &s // 返回字符串 s 的引用
} // 这里 s 离开作用域并被丢弃。其内存被释放。
  // 危险！

// 正确的：
fn no_dangle() -> String {
    let s = String::from("hello");
    s
}
```

* slice 和所有权 

和Python的slice切片类似，String、数组等都支持切片，语法是`&foo[m..n]`或者`foo[m..=n]`，后者包含`n`，而前者只到`n-1`。

slice类似引用，也是没有所有权的，其实和引用的区别只是slice的范围小于整体的范围。

字符串类型`String`对应的切片引用类型为`&str`，字面值其实就是`&str`类型，字面值和字符串slice都是不可变的。

数组所对应的切片类型是`&[i32]`。


