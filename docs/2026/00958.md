
# [原创]勒索病毒加密算法逆向识别

## 前言

使用`IDA Pro`进行分析时，通常只是利用它的反汇编功能，更快的分析汇编代码辅助后续需要采用的动态调试过程。由于`hexray`插件的强大，极大的提升了逆向效率。但是对于没有签名库匹配的情况下，得到都是未命名的函数，所以识别这些函数就成了一个问题。

提及一下`FLIRT`的原理：从函数的前32个字节中提取模式并使其成为签名。当然，如果模式相同，则会发生冲突。如果需要提取签名的库很大，则会遇到很多奔溃错误。

`openssl`算法库在勒索病毒（`853358339279b590fb1c40c3dc0cdb72`）中也算是比较常见，会静态链接至恶意软件，这为制作签名提供了基础。

所以接下来就是比较接近实操的`IDA`签名制作过程，记录一下。

首先逆向勒索病毒的时候通过`IDA`的字符串搜索功能找到使用的`openssl`版本号，这里是`openssl-1.1.1b`。

`C:\lib\openssl-1.1.1b\win32-release\lib\engines-1\_1`

![图片描述](upload/attach/202011/678001_VGSPFU9WA4YPFCH.png)

需要一个库文件来制作签名，库文件是`.lib`文件，等效于`.dll`，同样编译`openssl`还需要`Perl`环境。

## 开始安装环境

在Window安装`Perl`，`Perl`在Window平台上有`ActiveState Perl`和`Strawberry Perl`编译器。`ActiveState Perl`和 `Strawberry Perl`最大的区别是 `Strawberry Perl` 里面有多包含一些 `CPAN` 里的模块，所以`Strawberry Perl` 下载的安装文件有 80多M, 而`ActiveState Perl` 只有20M 左右。

这里使用了 `Strawberry Perl` 。Window 系统上 `Perl` 安装步骤如下：

`Strawberry` 安装包链接： <mark class="encrypted">208K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4@1M7X3q4%4j5X3g2J5M7Y4W2H3k6i4u0D9i4K6u0W2j5$3!0E0i4@1f1K6i4K6R3H3i4K6R3J5</mark>

下载对应你系统的版本： `32bit` 或 `64bit` 。

下载后双击打开，按安装向导一步步安装即可。

安装完后，在命令行测试一下环境。

![图片描述](upload/attach/202011/678001_V7PVE57J22HHBK7.png)

通过字符串找到勒索病毒的编译环境为`VS2017`，不过其实影响不大，这里就使用本地的`VS2015`环境。

![图片描述](upload/attach/202011/678001_B6YDW55HX2GD6G8.png)

下载`openssl`源代码本地编译

<mark class="encrypted">954K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2M7r3g2F1M7%4y4D9i4K6u0W2L8%4u0Y4i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3L8$3I4V1i4K6u0r3x3g2)9J5k6e0q4Q4x3X3f1I4i4K6u0r3</mark>

![图片描述](upload/attach/202011/678001_7SA3PB2UFQGS72Z.png)

管理员打开本地`VS2015`开发人员命令提示，输入执行以下命令。

```
perl Configure VC-WIN32 no-shared no-asm --prefix=C:\Users\onion\Desktop\openssl-1.1.1b\x86\_build\_out
```

![图片描述](upload/attach/202011/678001_PDPJCJM52TXXUQ4.png)

参数解释如下：

`Configure`是`OpenSSl`源码中自带的命令文件，必选参数；
`VC-WIN32`表示编译`x86`版本，`VC-WIN64A`表示编译`x64`版本，`debug-VC-WIN32`表示编译`debug`版`x86`版本，必选参数；
`no-shared`表示编译成静态链接`lib`模块，`shared`表示编译成动态链接库`dll`模块，必选参数；
`no-asm`如果要本地优化，还需要下载`NASM`，这里直接用`no-asm`选项，因此不需要下载`NASM`，必选参数；
`--prefix`是`OpenSSL`编译完后的安装路径，必选参数；

网上有很多关于Windows系统编译`openssl`的教程贴，在创建`Makefile`这一步中，讲到的都是使用

`ms\do\_ms`

`ms\do\_masm`

`ms\do\_nasm`

这之类的命令，在编译时输入这些的命令都是提示找不到这些命令，按照网上的说法也重新去安装了一些编译器，结果是统统无效。从`openssl 1.1.0`版本以后就不再有`ms\do\_\*.bat`这样的文件，而是直接使用`nmake`命令。

所以执行完上述命令后，执行`nmake`命令，接着执行`nmake install`，最后执行`nmake test`进行下测试。

![图片描述](upload/attach/202011/678001_UWEYR9WRMHNF3GB.png)

在编译完`lib`文件后，接着就到了制作签名文件的步骤了。

![图片描述](upload/attach/202011/678001_YRE8G9HYDZPZ3FM.png)

![图片描述](upload/attach/202011/678001_79M3ADMSPMU9Q9F.png)

利用工具包中的`pcf`工具创建模式文件（`pat`），在控制台使用命令`pcf *.lib *.pat`生成模式文件。接着使用`sigmake *.pat *.sig`命令生成`sig`文件。

## 结果

签名应用识别情况如下：

![图片描述](upload/attach/202011/678001_Z8S9GBVPVSBESFK.png)

成文的主要目的是记录一下，许多细节暂未记录，有兴趣的可自行动手练习然后解决那些问题。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-11-6 11:50

被`jishuzhain`编辑，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
