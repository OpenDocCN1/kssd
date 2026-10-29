
# [原创] Cocos2d-x与LuaJIT汇编的初步解密

最近在玩一个游戏，于是想要拿下它其中的所有立绘。在逆向的过程中，开拓了一些前人没有发掘的领域，无论是中文还是英文都没有相应的资料。本文证明了，即使Lua源码被编译过，也不是绝对的安全。厂商们打的小算盘可能是——没有成熟的工具，不好破解。但是对于了解原理的安全研究者，这不是问题。反而对于不了解原理，着重于快速开发复用代码的游戏开发者来说，没有成熟的防护体系，无论是自己造轮子还是用别人的轮子，造成的威胁都会更大。

先上一张解密后的样图。透明背景带Alpha层的png，很容易拿来做坏事，比如做套昆特牌啥的。

![sample](upload/attach/201711/496103\_ghv8xb66nmjo8di.png)

玩过的童鞋可能一眼就能认出来，嘿嘿嘿，游戏本身我不多作评价，本文所分析的版本为官网下载20170922版本。

# 编译修改Luajit-lang-toolkit

解包后发现所有的png资源全部加密了打不开（嘿，但是音频还没有加密……不过并不会有人看嘛）。结合lib发现游戏是cocos2x与Lua编写。虽然Lua出了名的好逆，但是查看文件头后发现是编译后的LuaJIT……还是最新的2.1版。一番操作后找到一个项目 <a href="elink@001K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3q4F1K9$3!0Q4x3V1k6D9N6h3q4B7K9i4c8Q4x3X3c8D9j5h3&6Y4i4K6u0V1N6r3!0G2L8r3E0A6N6l9`.`.">luajit-lang-toolkit</a>（以下简称LLT），不过用起来有点小问题。make以后在src文件夹生成了`luajit-x`程序，与luajit的命令行工具参数一样。

```plain
$ luajit-x -bl main
-- BYTECODE -- main:0-0
0001 TGETV 1 0 0
0002 KSHORT 2 1
...
0020 KSHORT 2 1
0021 ITERN 1 1 2
0022 FORL 0 => -32744
-- BYTECODE -- main:0-0
0001 TDUP 0 0
0002 TGETS 0 0 1 ; "\_\_G\_\_TRACKBACK\_\_"
0003 KPRI 0 500
...
0072 TGETS 0 0 19 ; "LAUNCHERPKG"
0073 TGETV 0 0 26
0074 KSHORT 1 30
0075 ITERN 0 2 2
0076 TSETV 0 0 31
0077 ITERN 0 2 1
0078 UNM 1 0
0079 TSETV 0 0 32
0080 ITERN 0 1 2
0081 FORL 0 => -32685
```

我们可以看到……啥也看不到……但是官方命令行工具的反汇编结果却大有不同。

你看人家至少有个RET啊！

```
$ luajit -bl main
-- BYTECODE -- main:0-0
0001 GGET 1 0 ; "print"
0002 KSTR 2 1 ; "----------------------------------------"
0003 CALL 1 1 2
...
0019 GGET 1 0 ; "print"
0020 KSTR 2 1 ; "----------------------------------------"
0021 CALL 1 1 2
0022 RET0 0 1
-- BYTECODE -- main:0-0
...
0043 KSTR 0 18 ; ""
0044 GSET 0 19 ; "LAUNCHERPKG"
0045 GGET 0 13 ; "cc"
0046 TGETS 0 0 20 ; "LuaLoadChunksFromZIP"
0047 KSTR 1 21 ; "lib/"
0048 GGET 2 3 ; "GAME\_BIT"
0049 KSTR 3 22 ; "/launcher.zip"
0050 CAT 1 1 3
0051 CALL 0 1 2
0052 GGET 0 23 ; "package"
0053 TGETS 0 0 24 ; "loaded"
0054 GGET 1 19 ; "LAUNCHERPKG"
0055 KSTR 2 25 ; "launcher.launcher"
0056 CAT 1 1 2
0057 KPRI 2 0
...
0073 GGET 0 26 ; "require"
0074 KSTR 1 30 ; "app.MyApp"
0075 CALL 0 2 2
0076 TGETS 0 0 31 ; "new"
0077 CALL 0 2 1
0078 MOV 1 0
0079 TGETS 0 0 32 ; "run"
0080 CALL 0 1 2
0081 => RET0 0 1
```

直接撸汇编可以，但是LLT可以解析更多的东西，怎么办？我仔细参考了一下opcode表，发现很多指令与正确指令之间在指令表之间（比如第一行的`TGETV`与`GGET`）只有两个指令的距离。再翻了翻LLT的issues里面有提到了2.1版本，某人还为他写了个patch。我才明白——因为2.1的指令表与2.0相比，中间插入了两个指令，因此后面的指令在表中的位置都相应后挪。

我完善了一下这个patch，将更新的指令表加入了读写过程中，修改后的版本位于<a href="elink@516K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6r3W2E0j5i4y4V1k6W2)9J5c8X3I4#2j5h3A6A6N6q4)9J5k6r3I4S2L8X3N6Q4x3X3c8@1L8$3!0D9K9$3W2@1i4K6u0r3N6s2u0W2k6g2)9J5c8Y4j5J5i4K6u0W2x3b7`.`.">我的Github</a>中的`v2.1`分支。运行修改后的版本输出就正常了。

```
$ luajit-x -bxg main
1b 4c 4a 02 | Header LuaJIT 2.0 BC
02 | Flags: BCDUMP\_F\_STRIP
| .. prototype ..
b6 01 | prototype length 182
00 | prototype flags None
01 | parameters number 1
05 | framesize 5
00 08 00 16 | size uv: 0 kgc: 8 kn: 0 bc: 23
| .. bytecode ..
36 01 00 00 | 0001 GGET 1 0 ; "print"
27 02 01 00 | 0002 KSTR 2 1 ; "---------------
| -------------------------"
42 01 02 01 | 0003 CALL 1 1 2
...
36 01 00 00 | 0019 GGET 1 0 ; "print"
27 02 01 00 | 0020 KSTR 2 1 ; "---------------
| -------------------------"
42 01 02 01 | 0021 CALL 1 1 2
4b 00 01 00 | 0022 RET0 0 1
| .. uv ..
| .. kgc ..
05 | kgc: ""
0e 74 72 61 63 65 62 61 | kgc: "traceback"
63 6b |
0a 64 65 62 75 67 | kgc: "debug"
06 0a | kgc: "\
...
15 5f 5f 47 5f 5f 54 52 | kgc: "\_\_G\_\_TRACKBACK\_\_"
41 43 4b 42 43 4b 5f |
5f |
00 | kgc: <function: main:0>
| .. knum ..
00 | eof
```

这个工具可以列出文件的hexdump与子程序的栈帧、局部变量等信息，是我特别心水的地方。下面进入逆向部分。

首先先把所有字节码反汇编并保存至src目录。在命令行中运行

```
ls | xargs -I% sh -c 'echo "Processing %" && luajit-x -bxg "%" > "src/%.lasm"'
```

查看PNG头发现字样`img.libla`，在png以外的文件里查找此字符串

```
$ grep libla . -R | grep -iv png
Binary file ./lib/armeabi/libcocos2dlua.so matches
```

如此看来解密过程应该就在`libcocos2dlua.so`中。先拖IDA里让它分析起来。好吧勉强承认一下南辕北辙了（没事Lua部分后面用到）。

# 分析libcocos2dlua

在这个so文件里搜索`img.libla` 定位到函数`cocos2d::Image::isMpi(uchar const\*,int)` 。去cocos2d-x官方文档里查询了一下竟然连Image这个类都没有。感觉不太可能，翻2.x版本的文档找到了对应的类CCImage，一直往后翻发现3.4是最后一个拥有这个类的版本。在字符串里搜一下`3.` 找一下有没有具体版本，有了意外发现。

```plain
.rodata:0119FB8B 0000004D C E:\\MyWorkSpace\\ACGProj\\cocosEngine\\Quick-Cocos2dx-361\\/cocos/./math/Vec3.cpp
```

噫～原来是用了框架。在GitHub上找到这个框架，作者表示选择了“公认最稳定的版本”3.3。 ~~其实就是懒得学新API了吧。~~ 随便搜一搜还能找到很多有意思的加密方式（见参考资料）。我们开心的开始带源码作业。但是简单搜索后却发现源码里根本没有这个函数。我们回到IDA中，查看调用这个函数判断文件类型的地方。

```c
signed int __fastcall cocos2d::Image::detectFormat(cocos2d::Image *this, const unsigned __int8 *a2, int a3)
{
cocos2d::Image *v4; // [sp+4h] [bp-14h]@1
int v5; // [sp+8h] [bp-10h]@1
unsigned __int8 *v6; // [sp+Ch] [bp-Ch]@1
signed int v7; // [sp+14h] [bp-4h]@2
// ...省略...
else if ( cocos2d::Image::isATITC(v4, v6, v5) & 1 )
{
v7 = 7;
}
else if ( cocos2d::Image::isMpi(v4, v6, v5) & 1 )
{
v7 = 10;
}
else
{
cocos2d::log("cocos2d: can't detect image format", &GLOBAL_OFFSET_TABLE_);
v7 = 11;
}
return v7;
}
```

而调用这个函数的地方附近又是这样。

```cpp
int __fastcall cocos2d::Image::initWithImageData(cocos2d::Image *this, const unsigned __int8 *a2, int a3)
{
// ... 解压被压缩的图片数据
v6 = cocos2d::Image::detectFormat(v8, (const unsigned __int8 *)v11, (int)v10);
}
*((_DWORD *)v8 + 10) = v6;
if ( v6 > 0xA )
{
// ... 未知格式错误处理
}
else
{
switch ( v6 )
{
case 1u:
v12 = cocos2d::Image::initWithPngData(v8, (const unsigned __int8 *)v11, (int)v10) & 1;
break;
case 0xAu:
v12 = cocos2d::Image::initWithMPIData(v8, (const unsigned __int8 *)v11, (int)v10) & 1;
break;
case 0u:
v12 = cocos2d::Image::initWithJpgData(v8, (const unsigned __int8 *)v11, (int)v10) & 1;
break;
case 2u:
//...
```

于是我们的目的就明确了——开发者魔改了`cocos2d::Image::initWithImageData`加入了自己的文件格式，并在 `detectFormat` 识别格式时调用 `isMpi` 判断并指定输入文件是否为自己的私有文件格式，然后自己写了一个`initWithMPIData`并在其中进行了解码或解密操作。我们要做的就是解析或利用这个函数的反向过程来解密图片资源。

# 以直接调用方式绕过资源加密

开始我尝试了用unicorn-engine模拟运行。但是后来发现函数里调用了malloc等标准库函数，处理起来很麻烦。IDA调试的选项也被我pass掉了，结合cocos2dx在CCImage.cpp中对图片的处理过程可知，内存里面存的是原始的RGBA数据，即使做出来内存dump也卵用无。折腾两天后决定利用NDK自己写一个wrapper，利用这个函数已导出的特性，从so里面获取这个函数并调用它来为我们解密。

我们先从已有代码入手判断一下Image类的结构。结合一下 <a href="elink@907K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2x3s2f1H3i4K6u0r3f1i4g2A6j5$3E0Q4x3X3c8o6L8$3y4G2M7K6u0V1P5q4)9J5k6p5y4G2L8h3#2#2L8X3W2@1P5g2)9J5c8X3u0D9L8$3u0Q4x3V1j5$3x3h3x3@1k6U0k6W2x3r3x3&6j5U0M7^5k6h3x3J5j5$3b7$3y4o6V1I4j5e0k6X3x3r3q4W2k6e0u0U0y4U0M7#2k6e0x3H3y4o6q4T1i4K6u0r3j5$3!0U0L8%4y4Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5c8V1y4o6d9h3#2S2k6$3g2Q4x3X3g2Z5i4K6t1K6e0o6p5%4y4R3`.`.">CCImage.h</a> 与 <a href="elink@c3fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2x3s2f1H3i4K6u0r3f1i4g2A6j5$3E0Q4x3X3c8o6L8$3y4G2M7K6u0V1P5q4)9J5k6p5y4G2L8h3#2#2L8X3W2@1P5g2)9J5c8X3u0D9L8$3u0Q4x3V1j5$3x3h3x3@1k6U0k6W2x3r3x3&6j5U0M7^5k6h3x3J5j5$3b7$3y4o6V1I4j5e0k6X3x3r3q4W2k6e0u0U0y4U0M7#2k6e0x3H3y4o6q4T1i4K6u0r3j5$3!0U0L8%4y4Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5c8V1y4o6d9h3#2S2k6$3g2Q4x3X3g2U0M7s2m8Q4x3U0y4x3z5o6V1@1">CCImage.cpp</a> 中`cocos2d::Image::initWithPngData` 与其在IDA中反汇编的对应关系整理如下。

```cpp
class CC_DLL Image : public Ref
{
static const int MIPMAP_MAX = 16;
unsigned char *_data; // *(this_ + 20)
ssize_t _dataLen; // *(this_ + 24)
int _width; // *(this_ + 28)
int _height; // *(this_ + 32)
bool _unpack; // *(this_ + 36) 来自初始化函数，实际并未读写
Format _fileType; // *(this_ + 40)
Texture2D::PixelFormat _renderFormat; // *(this_ + 44)
MipmapInfo _mipmaps[MIPMAP_MAX]; // *(this_ + 48) pointer to mipmap images
int _numberOfMipmaps; // *(this_ + 176)
// false if we cann't auto detect the image is premultiplied or not.
bool _hasPremultipliedAlpha; // *(this_ + 180) 在一个判断后可能读写
std::string _filePath;
```

其中要注意，这里一定要打开IDA的`Show casts`显示强制类型转换，例如`*((_DWORD *)this + 6) = 4 * v5;`这一句中实际被赋值的成员变量的偏移量不是6个字节而是6*DWORD=24字节。

我们据此编写这个对象的C++代码并测试一下是否需要改动。
