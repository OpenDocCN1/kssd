
```text
Rust Visual C++ prerequisites

Rust requires the Microsoft C++ build tools for Visual Studio 2013 or later, but they don't seem to be installed.

The easiest way to acquire the build tools is by installing Microsoft Visual C++ Build Tools 2019 which provides just the Visual C++ build tools:

https://visualstudio.microsoft.com/visual-cpp-build-tools/

Please ensure the Windows 10 SDK and the English language pack components are included when installing the Visual C++ Build Tools.

Alternately, you can install Visual Studio 2019, Visual Studio 2017, Visual Studio 2015, or Visual Studio 2013 and during install select the "C++ tools":

https://visualstudio.microsoft.com/downloads/

Install the C++ build tools before proceeding.

If you will be targeting the GNU ABI or otherwise know what you are doing then it is fine to continue installation without the build tools, but otherwise, install the C++ build tools before proceeding.

Continue? (y/N) y

Welcome to Rust!

This will download and install the official compiler for the Rust programming language, and its package manager, Cargo.

Rustup metadata and toolchains will be installed into the Rustup home directory, located at:

C:\Users\cml\.rustup

This can be modified with the RUSTUP_HOME environment variable.

The Cargo home directory located at:

C:\Users\cml\.cargo

This can be modified with the CARGO_HOME environment variable.

The cargo, rustc, rustup and other commands will be added to Cargo's bin directory, located at:

C:\Users\cml\.cargo\bin

This path will then be added to your PATH environment variable by modifying the HKEY_CURRENT_USER/Environment/PATH registry key.

You can uninstall at any time with rustup self uninstall and these changes will be reverted.

Current installation options:

default host triple: x86_64-pc-windows-msvc
default toolchain: stable (default)
profile: default
modify PATH variable: yes

1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>

info: profile set to 'default'
info: default host triple is x86_64-pc-windows-msvc
info: syncing channel updates for 'stable-x86_64-pc-windows-msvc'
info: latest update on 2022-01-20, rust version 1.58.1 (db9d1b20b 2022-01-20)
info: downloading component 'cargo'
  3.8 MiB /  3.8 MiB (100 %) 1.7 MiB/s in 2s ETA: 0s
info: downloading component 'clippy'
  1.6 MiB /  1.6 MiB (100 %) 1.5 MiB/s in 1s ETA: 0s
info: downloading component 'rust-docs'
 18.8 MiB / 18.8 MiB (100 %) 3.3 MiB/s in 5s ETA: 0s
info: downloading component 'rust-std'
 22.9 MiB / 22.9 MiB (100 %) 3.2 MiB/s in 7s ETA: 0s
info: downloading component 'rustc'
 65.2 MiB / 65.2 MiB (100 %) 493.2 KiB/s in 1m 14s ETA: 0s
info: downloading component 'rustfmt'
  2.2 MiB /  2.2 MiB (100 %) 631.2 KiB/s in 3s ETA: 0s
info: installing component 'cargo'
info: installing component 'clippy'
info: installing component 'rust-docs'
 18.8 MiB / 18.8 MiB (100 %) 1.9 MiB/s in 6s ETA: 0s
info: installing component 'rust-std'
 22.9 MiB / 22.9 MiB (100 %) 10.5 MiB/s in 2s ETA: 0s
info: installing component 'rustc'
 65.2 MiB / 65.2 MiB (100 %) 12.2 MiB/s in 5s ETA: 0s
info: installing component 'rustfmt'
info: default toolchain set to 'stable-x86_64-pc-windows-msvc'

stable-x86_64-pc-windows-msvc installed - rustc 1.58.1 (db9d1b20b 2022-01-20)

Rust is installed now. Great!

To get started you may need to restart your current shell.
This would reload its PATH environment variable to include Cargo's bin directory (%USERPROFILE%\.cargo\bin).

Press the Enter key to continue.
```

# 核心组件

- `rustup`：安装、更新 Rust 用的，`rustup doc` 可查看安装包中的官方指导文档
- `cargo`：包管理器和编译源代码工具
- `rustc`：将 Rust 源文件编译成可执行程序
- `rustdoc`：为 Rust 项目生成说明文档

# 常用命令

| 命令 | 说明 | 备注 |
| --- | --- | --- |
| `rustup doc` | 打开官方指导文档 |  |
| `cargo new projectName` | 创建一个 Rust 工程 | 示例：`cargo new firstRustProject` |
| `cargo run` | 运行 Rust 工程 |  |
| `cargo build` | 编译 Rust 工程 | 若增加了依赖，即修改了 `toml` 文件，需要重新编译 |

# 基础语法

- `fn` 声明函数
- `let` 声明变量，变量默认不可变，加 `mut` 的变量为可变变量
- 虽然所有变量声明都用 `let` 关键字，但是 Rust 是强类型的语言，内置了强大的类型推导
- 函数参数：`参数名:参数数据类型`
- 变量重影性质：同一个名称可以被多个变量使用，后面的变量会覆盖前面的变量
- `const` 声明常量
- `//` 注释一行，`/* */` 注释多行
- 使用英文分号换行

示例：

```rust
fn main() {
    println!("Hello, world!");
    //变量默认是不可变的，加上 mut 关键字就可以重新赋值。
    let mut x=5;
    println!("The value of x is   {}  ",x);

    x=6;
    println!("The value of x is   {}  ",x);

    //变量的隐藏
    let money=100;
    println!("money is {}",money);
    let money=money+8;
    println!("money is {}",money);
    let money="一百元";
    println!("money is {}",money);

    //常量使用 const 关键字声明，声明的时候必须指定数据类型，常量名全大写。
    //不需要let ， 不可使用mut 修饰
    const MAX_PIONTS: u32=888;
    println!("The constant is {}",MAX_PIONTS);

    let result:char= a_function(88, 'M', false);
    println!("result is {}",result);
}

fn a_function(a:u64,b:char,c:bool)-> char{
    println!("a is {}",a);
    println!("b is {}",b);
    println!("c is {}",c);
    return 'N';
}
```

输出：

```text
Hello, world!
The value of x is 5
The value of x is 6
money is 100
money is 108
money is 一百元
The constant is 888
a is 88
b is M
c is false
result is N
```

## 数据类型

### 标量类型

整数，浮点，布尔，字符

- 整数：`i8`, `u8`, `i16`, `u16`, `i32`, `u32`, `i64`, `u64`, `i128`, `u128`, `isize`, `usize`
  - `i` 表示有符号
  - `u` 表示无符号
  - `isize` 和 `usize` 两种整数类型是用来衡量数据大小的，它们的位长度取决于所运行的目标平台，如果是 32 位架构的处理器将使用 32 位位长度整型。
- 浮点：`f32`, `f64`

### 复合类型

可以将多个值放到一个数据类型中。

- 元组 - tuple。长度固定，元素的数据类型可以不同
- 数组，长度固定，元素的数据类型必须相同
- `Vector`：不是标准库提供的。和数组类似，长度可变

### 示例

```rust
fn main() {
    println!("Hello, world!");
    let q=3.0;
    let q:f32=5.00;
    let w=true;
    let r:bool=false;

    let t='';
    let tup :(i32,u64,bool) = (88,99,false);
    println!("元素1：{},元素2：{},元素3：{}",tup.0, tup.1, tup.2);
    let arr:[u64;5] = [1,2,3,5,5];
    let arr2=['E';9];
    println!("arr piont 2  is :{}",arr[1]);
    println!("arr2 piont 2  is :{}",arr2[1]);
    //Vector：todo
}
```

## 条件语句

和大多数编程语言一样， `if` - `else if` - `else` 表示条件语句，在 `if` 后面不用加括号。
