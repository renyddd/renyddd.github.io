---
title: "Rust Tutorial"
date: 2021-09-24T21:26:15+08:00
draft: false
tags: ["rust"]
categories: ["rust"]
---

引用于：

> https://kaisery.github.io/trpl-zh-cn/title-page.html

## Notes

### Base

变量默认是不可改变的 immutable

```rust
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:3:5
2 |     let x = 1;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

例如,使用大型数据结构时,适当地使用可变变量,可能比复制和返回新分配的实例更快。对于较小的数据结构,总是创建新实例,采用更偏向函数式的编程⻛格,可能会使代码更易理解,为可读性而牺牲性能或许是值得的。

不可变量与 const 所声明的常量的区别：常量可以在任何作用域中声明,包括全局作用域,这在一个值需要被很多部分的代码用到时很有用。

Shadowing：可以使用同名变量来隐藏一个变量，当再次使用 let  时,实际上创建了一个新变量,我们可以改变值的类型,但复用这个名字。



Rust 是 静态类型 statically typed 语言,也就是说在编译时就必须知道所有变量的类型。

标量(scalar)类型代表一个单独的值。Rust 有四种基本的标量类型:整型、浮点型、布尔类型和字符类型。

Rust 的 char  类型的大小为四个字节(four bytes),并代表了一个 Unicode 标量值(Unicode Scalar Value).

复合类型(Compound types)可以将多个值组合成一个类型。Rust 有两个原生的复合类型:元组(tuple)和数组(array)。

- 元组是一个将多个其他类型的值组合进一个复合类型的主要方式。元组⻓度固定:一旦声明,其⻓度不会增大或缩小。
- 数组中的每个元素的类型必须相同。Rust 中的数组与一些其他语言中的数组不同,因为 Rust 中的数组是固定⻓度的:一旦声明,它们的⻓度不能增⻓或缩小。 

```rust
fn main() {
 let  t = [1,2,3];
 println!("{}", t);
}

error[E0277]: `[{integer}; 3]` doesn't implement `std::fmt::Display`
 --> src/main.rs:6:17
  |
6 |  println!("{}", t);
  |                 ^ `[{integer}; 3]` cannot be formatted with the default formatter
  |
  = help: the trait `std::fmt::Display` is not implemented for `[{integer}; 3]`
  = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
  = note: this error originates in the macro `$crate::format_args_nl` (in Nightly builds, run with -Z macro-backtrace for more info)
```

由上可见，数组的类型加长度决定了它的类型，这里和 golang 中的数组很像；

vector 类型是标准库提供的一个 允许 增⻓和缩小⻓度的类似数组的集合类型。当不确定是应该使用数组还是 vector 的时候,你可能应该使用 vector。第八章会详细讨论 vector。



Rust 代码中的函数和变量名使用 snake case 规范⻛格。在 snake case 中,所有字母都是小写并使用下划线分隔单词。

在函数签名中,必须 声明每个参数的类型。这是 Rust 设计中一个经过慎重考虑的决定:要求在函数定义中提供类型注解,

因为 Rust 是一⻔基于表达式(expressionbased)的语言,这是一个需要理解的(不同于其他语言)重要区别。其他语言并没有这样的区别：

- 语句(Statements)是执行一些操作但不返回值的指令；
- 表达式(Expressions)计算并产生一个值。

语句 let y = 6;  中的 6  是一个表达式,它计算出的值是 6 。函数调用是一个表达式。宏调用是一个表达式。我们用来创建新作用域的大括号(代码块), {} ,也是一个表达式

for  循环的安全性和简洁性使得它成为 Rust 中使用最多的循环结构。

### Ownership

所有权(系统)是 Rust 最为与众不同的特性,它让 Rust 无需垃圾回收(garbage collector)即可保障内存安全。

跟踪哪部分代码正在使用堆上的哪些数据,最大限度的减少堆上的重复数据的数量,以及清理堆上不再使用的数据确保不会耗尽空间,这些问题正是所有权系统要处理的。明白了所有权的存在就是为了管理堆数据,能够帮助解释为什么所有权要以这种方式工作。

所有权的规则。当我们通过举例说明时,请谨记这些规则: 

1. Rust 中的每一个值都有一个被称为其 所有者(owner)的变量。

2. 值在任一时刻有且只有一个所有者。
3. 当所有者(变量)离开作用域,这个值将被丢弃。

作用域(scope)是一个项(item)在程序中有效的范围。

```rust
fn main() {
    {
        let s = "hello";
        println!("{}", s);
    }
    println!("{}", s);

}
error[E0425]: cannot find value `s` in this scope
 --> src/main.rs:6:20
  |
6 |     println!("{}", s);
  |  
```



再来认识下「字符串字面值」与「String 类型」的区别：

1. let s = "hello"; 变量 s  绑定到了一个字符串字面值，这个字符串值是硬编码进程序代码中的；类型为 &str，且存储在栈空间上；
2. String 类型是一种存住在堆上的胖指针结构，可用来存储编译时大小未知或可变的文本。

```rust
let mut s = String::from("hello"); 
s.push_str(", world!");
println!("{}", s);
```

#### 变量与数据交互的方式(一):移动

```rust
fn main() {
    let x = 5;
    let y = x;

    println!("{}", x);

    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}", s1);
    println!("{}", s2);
}
error[E0382]: borrow of moved value: `s1`
  --> src/main.rs:10:20
   |
7  |     let s1 = String::from("hello");
   |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
8  |     let s2 = s1;
   |              -- value moved here
9  | 
10 |     println!("{}", s1);
   |                    ^^ value borrowed here after move
```

rust 的 String 类型在栈中拥有一个指向堆中数据的指针，如果 let s2 = s1 语句导致了栈上的两个不同指针指向了相同的的对内存，那么当两者同时离开作用域而引发释放时，则会导致二次释放的错误。

rust 的处理方式时，预期发生内存拷贝，不如认为 s1 不再有效，同时当 s1 离开作用域时也不会进行清理；这个操作就叫做移动 move。

这种仅拷贝指针值的行为，听起来很像术语“浅拷贝”，rust 永远也不会创建数据的“深拷贝”。

#### 变量与数据交互的方式(二):克隆

当需要深度复制 String 类型堆上的数据时，则可使用类型方法 clone。

#### 只在栈上的数据:拷⻉

作为一个通用的规则,任何简单标量值的组合可以是 Copy  的,不需要分配内存或某种形式资源的类型是 Copy  的。

#### 所有权与函数

将值传递给函数，和给变量赋值在语意上都是相似的，因此也会发生移动或者复制。

```rust
fn main() {
    let s = String::from("hello");
    take_ownership(s);

    println!("{}", s);
}

fn take_ownership(s: String) {
    println!("take_ownership: {}", s);
}

error[E0382]: borrow of moved value: `s`
 --> src/main.rs:5:20
  |
2 |     let s = String::from("hello");
  |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
3 |     take_ownership(s);
  |                    - value moved here
4 | 
5 |     println!("{}", s);
  |                    ^ value borrowed here after move
```

函数的返回值也会转移所有权。

```rust
fn main() {
    let s = String::from("hello");
    let s = take_and_gives_back(s);

    println!("{}", s);
}

fn take_and_gives_back(s: String) -> String {
    s
}
```

在一个函数中先获取所有权，再返回所有权是很啰嗦，rust 提供了引用 reference。

### 引用与借用 reference and borrowing

& 是引用符号，其允许你使用它的值但不获取所有权，即离开作用域后并不会丢弃。

```rust
fn main() {
    let s1= String::from("hello");
    let len = calculate_lenght(&s1); // 创建了指向 s1 的引用，但却不拥有它

    println!("len: {}", len);
}

fn calculate_lenght(s: &String) -> usize { // 签名中的 & 也表明它的类型是一个引用
    s.push_str(", world");
    s.len()
}

error[E0596]: cannot borrow `*s` as mutable, as it is behind a `&` reference
 --> src/main.rs:9:5
  |
8 | fn calculate_lenght(s: &String) -> usize {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
9 |     s.push_str(", world");
  |     ^ `s` is a `&` reference, so the data it refers to cannot be borrowed as mutable
```

我们将获取引用作为函数参数称为借用 borrowing，引用默认不允许修改值。

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

首先,必须将 s  改为 mut 。然后必须创建一个可变引用 &mut s  和接受一个可变引用 some_string: &mut String 。

不过可变引用有一个很大的限制:在特定作用域中的特定数据只能有一个可变引用。

让我们概括一下之前对引用的讨论: 

1. 在任意给定时间,要么只能有一个可变引用,要么只能有多个不可变引用。
2. 引用必须总是有效的。

#### Slice 类型

上面的引用是仅获取值而不得到所有权，另一个不拥有所有权的数据类型是 Slice。

slice 允许你引用集合中一段连续的元素序列,而不用引用整个集合。

```rust
let  s = String::from("hello， world");
    
let hello = &s[0..5];
let world = &s[6..11];
println!("{}, {}", hello, world);
```

hello 类型为 &str；同样的，存储在二进制文件中的字符串字面值也是 &str，这同样是一个不可变引用。

### Struct

注意整个实例必须是可变的;Rust 并不允许只将某个字段标记为可变。记一些不同指出：

1. 元组结构体(tuple structs)：有着结构体名称提供的含义,但没有具体的字段名,只有字段的类型；
2. 类单元结构体(unit-like structs)：因为它们类似于 () ,即 unit 类型。类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

方法语法(method syntax)：为了使函数定义于 Rectangle  的上下文中,我们开始了一个 impl  块( impl  是 implementation 的缩写)。接着将 area  函数移动到 impl  大括号中,并将签名中的第一个(在这里也是唯一一个)参数和函数体中其他地方的对应参数改成 self 。

我们并不想获取所有权,只希望能够读取结构体中的数据,而不是写入。如果想要在方法中改变调用方法的实例,需要将第一个参数改为 &mut self 。通过仅仅使用 self 作为第一个参数来使方法获取实例的所有权是很少⻅的;这种技术通常用在当方法将 self 转换成别的实例的时候,这时我们想要防止调用者在转换之后使用原始的实例。

关联函数(associated functions)：impl  块的另一个有用的功能是:允许在 impl  块中定义 不 以 self  作为参数的函数。它们与结构体相关联。它们仍是函数而不是方法,因为它们并不作用于一个结构体的实例。你已经使用过 String::from  关联函数了。

### 枚举和模式匹配

枚举(enumerations),也被称作 enums。枚举允许你通过列举可能的成员(variants) 来定义一个类型。

```rust
fn main() { 
    let four = IpAddrKind::V4;
    route(four);
    route(IpAddrKind::V6);
}

enum IpAddrKind {
    V4,
    V6,
}

fn route(ip_type: IpAddrKind) {}
```

如果仅用上述代码，就会发现目前还没有一个实际存储 IP 地址值的方法，我们仅知道的是其类型。

我们直接将数据附加到枚举的每个成员上,这样就不需要一个额外的结构体了;不过你也可以将任意类型的数据放入枚举成员中：

```rust
enum IpAddrKind {
    V4(String),
    V6(String),
}


impl IpAddrKind {
    fn call(&self) {
    }
}
```

如果我们使用不同的结构体,由于它们都有不同的类型,我们将不能像轻易的定义一个能够处理这些不同类型的结构体的函数；而枚举可以，因为枚举是单独一个类型。

就像可以使用 impl  来为结构体定义方法那样，方法也可以在枚举上定义。

#### Option 枚举

Option  是标准库定义的一个枚举。 Option 类型应用广泛因为它编码了一个非常普遍的场景,即一个值要么有值要么没值。

空值尝试表达的概念仍然是有意义的:空值是一个因为某种原因目前无效或缺失的值。
问题不在于概念而在于具体的实现。为此,Rust 并没有空值,不过它确实拥有一个可以编码存在或不存在概念的枚举。这个枚举是 Option<T> ,而且它定义于标准库中 [https://doc.rust-lang.org/std/option/]( https://doc.rust-lang.org/std/option/)

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Option 枚举被包含在了 prelude 之中,你不需要将其显式引入作用域。另外,它的成员也是如此,可以不需要 Option::  前缀就能来直接使用 Some  和 None 。

```rust
let n = Some(5);
let s = Some("a string");
let a: Option<i32> = None;
```

当使用 None 时需要指明 Option 的类型，因为编译器仅通过 None 无法进行推断。

```rust
    let x: i32 = 5;
    let y: Option<i32> = Some(8);
    let sum = x + y;

error[E0277]: cannot add `Option<i32>` to `i32`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i32 + Option<i32>`
  |
  = help: the trait `Add<Option<i32>>` is not implemented for `i32`
```

当在 Rust 中拥有一个像 i32  这样类型的值时,编译器确保它总是有一个有效的值。我们可以自信使用而无需做空值检查。只有当使用 Option<i32> (或者任何用到的类型)的时候需要担心可能没有值,而编译器会确保我们在使用值之前处理了为空的情况。换句话说,在对 Option<T>  进行 T  的运算之前必须将其转换为 T 。通常这能帮助我们捕获到空值最常⻅的问题之一:假设某值不为空但实on际上为空的情况。

#### match 控制流运算符

match  控制流运算符允许我们将一个值与一系列的模式相比较,并根据相匹配的模式执行相应代码。

匹配 Option：获取一个 Option<i32> 的值，如果不为空则加一

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i+1),
    }
}
```

_  模式会匹配所有的值。通过将其放置于其他分支之后, _  将会匹配所有之前没有指定的可能的值，以免因未穷举到所有情况而编译失败。

#### if let 控制流

if let  语法让我们以一种不那么冗⻓的方式结合 if  和 let ,来处理只匹配一个模式的值而忽略其他模式的情况。

```rust
let some_u8_value = Some(3u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

如上当我们仅想对 Some(3) 进行操作，而去忽略 None 或任何不等于 Some(3) 的值；其中这里的表达式对应 match，而模式对应第一个分支。可以认为 if let  是 match  的一个语法糖；可以在 if let  中包含一个 else 。 else  块中的代码与 match  表达式中的 _  分支块中的代码相同。



## 模块系统







