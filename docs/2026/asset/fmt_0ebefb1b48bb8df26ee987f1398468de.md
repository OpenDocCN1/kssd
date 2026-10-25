
- 阶段0：启动，下载当前beta版本的`rustc`等工具来编译`src/bootstrap`、`std`、`rustc`
- 阶段1：由阶段0编译出当前源码的编译器（`rustc`），使用阶段0的ABI
- 阶段2：由阶段1编译出当前源码的编译器，使用阶段1的ABI

这里我只是大概了解了一下，可能会有些偏差，具体详情可以看[官方文档](https://bbs.kanxue.com/elink@ecaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5N6i4y4@1j5#2)9J5k6r3c8W2N6W2)9J5k6r3N6#2K9h3c8W2i4K6u0W2M7Y4g2K6N6q4)9J5k6r3I4S2L8X3N6Q4x3X3g2G2M7X3N6Q4x3V1k6T1N6h3W2D9k6r3W2F1k6#2)9J5c8X3u0G2L8%4c8K6N6s2u0S2M7s2m8A6L8X3N6Q4x3X3g2Z5N6r3#2D9)以及[源码的说明](https://bbs.kanxue.com/elink@615K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5N6i4y4@1i4K6u0V1L8r3q4F1k6#2)9J5c8Y4u0#2M7%4c8Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3j5X3!0G2N6s2y4@1M7X3q4H3)

其中，我们真正使用的是阶段2的文件作为我们自己的toolchain。由于我们的LLVM是动态编译的，所以在阶段1->阶段2的时候编译出的`rustc`无法执行，需要跟LLVM一样需要手动补充依赖文件。

修改的文件：`src/bootstrap/compile.rs`

```rust
if target_compiler.stage == 1 {
let llvm_dir = r"C:\Users\Administrator\Desktop\tutorial\llvm_x64\bin";
for entry in fs::read_dir(llvm_dir).expect("Dir not found!") {
let entry = entry.expect("this was dir");
let path = entry.path();
if let Some(extension) = path.extension() {
if extension == "dll" {
let dst_path = bindir.join(entry.file_name());
// println!("{}", dst_path.as_path().display().to_string());
fs::copy(path, dst_path).expect("error copy");
}
}
}
}
```

**注意，上面的目录需要按照自己的实际情况修改**

位置：

![8ce05539bea6080fc04a9080006fc0d8.png](upload/attach/202209/956011_QRCEY32YBUDV6TS.png)

这里我们在创建完每个阶段的**bin**文件夹后，判断当前阶段，如果是阶段1就复制我们自己编译的LLVM的dll文件到当前阶段的**bin**文件夹，这样就能保证`rustc`的编译过程能完成。

## 编译rustc

这里有个小坑，如果没有在MSYS2装Git，但是在Windows下装了Git的，需要手动设置下环境变量，否则`rust`的编译脚本无法找到`git`

`export PATH=$PATH:/c/Program\ Files/Git/bin`

![16a8c3de41c3a5433f3d3a01aaa3664b.png](upload/attach/202209/956011_U534GBEXZRR5RBU.png)

在`mingw-w64`窗口，进到`rust`的源码目录，执行

`python x.py build`

然后等待大概30-40分钟，具体时间取决于编译机器的配置以及网络情况。

编译完成：

![ce41990f1dd29c376f8aa14aaa75dadd.png](upload/attach/202209/956011_QQUZ9CHZTTB79A5.png)

成品：

![41cc26998dffa7d294a169aeb0107fee.png](upload/attach/202209/956011_B4Z8ZE6N7WDK82F.png)

![fff27956483e3aef8100a8325dbb2b26.png](upload/attach/202209/956011_2Z8TAZGY683QTCE.png)

# 编译cargo

## 编译cargo本体

在`rust`源码目录执行

`python x.py build tools/cargo`

等待20-30分钟，具体时长取决于编译机器的配置。

成品：

![54c74eb6585be42c554411272ef5d9a8.png](upload/attach/202209/956011_HJ56C3CRUWVBGK7.png)

![db33725bb0823d176bbb929d92e42fd0.png](upload/attach/202209/956011_M5NRWWBZFVDMPJP.png)

## 复制依赖

编译出来的`cargo`还是跟之前一样，缺少`dll`，需要去MSYS2的**mingw64的bin目录**复制**zlib1.dll**到**rust/build/x86_64-pc-windows-gnu/stage1-tools-bin**

**![6f84e9e71853e55ed503573ffd206b23.png](upload/attach/202209/956011_XK4T6DKB7RSGNYU.png)**

# 使用自己的toolchain

## 安装rustup

***如果之前安装过rustup的可以跳过这个步骤***

1. 下载`rustup-init.exe`

到[官网](https://bbs.kanxue.com/elink@428K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5N6i4y4@1i4K6u0V1L8r3q4F1k6#2)9J5k6h3!0J5k6#2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3K9h3&6K6N6r3q4D9L8l9`.`.) 点击**rustup-init.exe (64-bit)**进行下载

2. 设置`rustup`

- 选择ABI

这里选3，跳过安装vs

![6b3edabff7c70e7ae77f91426f548b0a.png](upload/attach/202209/956011_8QYTS8X5PT7KBR7.png)

- 选择toolchain

这里选择2，自定义，并输入**x86_64-pc-windows-gnu**

![bc0f7eeec44e160d6f7e852b6e23d7d4.png](upload/attach/202209/956011_3RXZNNE9YVMA92V.png)

- 选择toolchain频道

这里输入`nightly`

![20679b702e974ee16e28c501720b7000.png](upload/attach/202209/956011_Y9TAAVRJMKC4W7U.png)

- 选择额外工具

这里默认即可（回车）

![3c560cb649caaacccf94e0a70abe6071.png](upload/attach/202209/956011_24AJ4X9YT9ADU9W.png)

- 是否修改系统环境变量

这里选是，输入`Y`

![3fe1d0f3fb34bed1e7931cd205f4f675.png](upload/attach/202209/956011_S6UC2PUDQGS8SKZ.png)

- 然后进行安装

这里直接输入1，然后回车即可

![d443d4c129098e462e82fc986e9dee1d.png](upload/attach/202209/956011_2AY2KF7GS6Q3HC8.png)

- 完成

按回车退出即可。

![1a9b1c05bdf30e6ba847d53e64aa5210.png](upload/attach/202209/956011_CANF5QFVQMP4YVJ.png)

## 配置自己编译的toolchain

打开一个新的`cmd`

1. 配置toolchain

在`cmd`输入

`rustup toolchain link my-rust-1.63.0 C:\Users\Administrator\Desktop\tutorial\rust\build\x86_64-pc-windows-gnu\stage1`

其中：

```sh
my-rust-1.63.0 为自定义toolchain名字
C:\Users\Administrator\Desktop\tutorial\rust\build\x86_64-pc-windows-gnu\stage1 为刚才编译出来的toolchain的阶段1目录
```

![7c2b0439245e37669a2c40ec91ee01e7.png](upload/attach/202209/956011_VH68GKZEY9YAVCX.png)

2. 设置成默认toolchain

在`cmd`输入

`rustup default my-rust-1.63.0`

其中，`my-rust-1.63.0`为刚才自定义toolchain的名字

![838fe7d80005538d0276f264aaa93d5b.png](upload/attach/202209/956011_J899S4SCKHKSRF7.png)

3. 验证

在`cmd`输入

`rustup toolchain list`

查看当前toolchain的状态

![d071159190305c3e418b72be91755463.png](upload/attach/202209/956011_F39A4SMB6ETQRKV.png)

## 使用

1. 使用`cargo new`创建个`hello world`

![308a011b0caa9a65615283a798224480.png](upload/attach/202209/956011_97A2NT69ZZW6ZBA.png)

2. 设置项目参数

在项目根目录创建或者修改`.cargo/config`

**如果`.config`目录不存在，自己创建一个。**

**`config`是一个文件**

`config`的内容如下

```toml
[target.x86_64-pc-windows-gnu]
rustflags = [
"-Z", "llvm-plugins=libLLVMObfuscator.dll",
"-C", "passes=obf-fla obf-bcf obf-split",
"-C", "llvm-args=-fla -bcf_loop=2 -split_num=2",
]
```

说明：

```sh
[target.x86_64-pc-windows-gnu] 对应的target名字
rustflags 从cargo传递到rustc的参数列表
"-Z", "llvm-plugins=libLLVMObfuscator.dll" 启用混淆插件
"-C", "passes=obf-fla obf-bcf obf-split" 指定混淆模式
"-C", "llvm-args=-fla -bcf_loop=2 -split_num=2" 设置混淆参数
```

3. 编译

打开`MSYS2 - MINGW64`

在项目目录执行

`cargo +my-rust-1.63.0 build -Z build-std=panic_abort,std,core,alloc,proc_macro --target x86_64-pc-windows-gnu --release`

参数说明：

```sh
+my-rust-1.63.0 指定使用名为my-rust-1.63.0的toolchain
build 构建cargo项目
-Z build-std=panic_abort,std,core,alloc,proc_macro 重新以混淆的方式编译rust的std到当前项目
--target x86_64-pc-windows-gnu 指定编译的std的目标架构
--release 编译release
```

# 效果

1. 正常编译

使用命令：`cargo +nightly-x86_64-pc-windows-gnu build --release`并不指定**config**编译

IDA如下：

![838c31cb68a7d1140cabc68242287540.png](upload/attach/202209/956011_5CNCJG89ZKWQFEQ.png)

2. 混淆

使用命令：`cargo +my-rust-1.63.0 build -Z build-std=panic_abort,std,core,alloc,proc_macro --target x86_64-pc-windows-gnu --release`编译

IDA如下：

![28606da17a9a2d860102c9373cab9a5a.png](upload/attach/202209/956011_725XTDV4TTXE7YX.png)

# 前言

`rust`的编译器`rustc`用`llvm`进行中间代码生成（MIR->LLVM IR [链接](https://bbs.kanxue.com/elink@201K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5N6i4y4@1j5#2)9J5k6r3c8W2N6W2)9J5k6r3N6#2K9h3c8W2i4K6u0W2M7Y4g2K6N6q4)9J5k6r3I4S2L8X3N6Q4x3X3g2G2M7X3N6Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5k6h3S2@1L8h3H3`.）），所以我想尝试下在`rust`编译过程加个`pass`进行代码混淆，进而保护生产代码。

由于`rust`在Windows下有两种`toolchain`，一种是`msvc`，另外一种是用`mingw`的*windows-gnu*。由于`LLVM`在Windows下的动态库编译只能使用`Mingw-w64`环境，具体来源：[LLVM官方CMake参数](https://bbs.kanxue.com/elink@61eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0U0M7#2)9J5c8V1y4y4j5h3E0W2i4K6u0W2K9s2c8E0L8q4)9J5x3$3I4D9N6X3#2Q4x3X3c8J5k6h3I4S2N6r3g2V1i4K6u0V1N6X3q4J5K9h3q4T1L8r3g2K6)，并且`rust`自己编译的`LLVM`不支持动态链接。

![815418feb83ab11c46d19be273276771.png](https://bbs.kanxue.com/upload/attach/202209/956011_46FQ55Y9DS3D9YR.png)

即本文使用`MSYS2`下的`Mingw-w64`环境。

# 环境准备

## 机器要求

足够强劲的机器，大概20G的硬盘空间（固态更好），8G以上的内存，以及*良好的网络链接*

使用`Ninja`代替`make`能大幅提升速度，但是增加内存消耗（link时可能达到24G内存占用），若内存不足的可以在生产`cmake build`的时候使用`MinGW Makefiles`或者`Unix Makefiles`。本文主要使用`Ninja`作为主要构建工具。

空间占用：

![a83d11cf5e346c002e181a78c9b400fa.png](https://bbs.kanxue.com/upload/attach/202209/956011_982C37PTKHHKAZR.png)

内存占用：

![c731cdb28b4b334fa6427e416ca00147.png](https://bbs.kanxue.com/upload/attach/202209/956011_5KHU2E97ZTTDFPK.png)

## msys2

从清华源 [链接](https://bbs.kanxue.com/elink@f28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6E0M7%4W2K6x3W2)9J5c8X3c8A6M7%4c8J5K9h3u0Q4x3V1k6^5z5o6k6Q4y4h3j5$3y4q4)9J5c8R3`.`.) 下载最新的安装包，例如 **msys2-x86_64-20220904.exe**

然后直接安装

![76b0bc971f3600996aff32bf70011c14.png](https://bbs.kanxue.com/upload/attach/202209/956011_3HEDPQ4ADB4UZ7S.png)

中间会卡一会，等一下就好

完成的时候不要直接启动它默认的终端

![1360039d97ab407e53b76be8a12f2bb2.png](https://bbs.kanxue.com/upload/attach/202209/956011_REYNTJZYGJVQ3PC.png)

去开始菜单找`msys2 - mingw64`
