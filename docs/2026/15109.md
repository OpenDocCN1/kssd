
# [原创]几维免费加密动态分析

折腾了一周，纠结在`ndk`开发上，感觉`jni`的接口很是难用，以后再学习吧。模拟器简直是个巨大的`bug`，巨难用，以后一定要搞个真机做调试。

自己开发的一个`crack`，比较简单，主要是在`ndk`上废了太多时间，有时间再研究吧。

完毕，上传几维在线加密。直接把加密`apk`放到改之理中，先看了下`string`。貌似没加密这里，

![](https://bbs.kanxue.com/upload/attach/201707/734587_b7iykzw3qt856iw.jpg)

![](https://bbs.kanxue.com/upload/attach/201707/734587_73hc8s27sgqb8xc.jpg)

项目中的多了个`Application`的类，目测这里应该是加密点。为什么，应为这里可以提前`Mainactivity`执行，当然也可能不是。类外，据说加密之后可以防止反编译，这个我好想没看出来，查看`smali`源码，不过没找到引用"成功"的字符，尴尬了。

![](https://bbs.kanxue.com/upload/attach/201707/734587_z8mjarsynadbdil.jpg)

在`application`中找到这么个东西，调用了`kdp`中的`a`类，还有这个类本身叫做`proxyAppliction`，

这让我想起了之前看的一个叫做用代理方式动态加载的文章。

[06bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9X3W2S2L8X3N6%4k6h3V1H3z5e0p5H3y4o6p5H3x3o6l9K6i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5@1z5o6p5H3y4o6f1^5x3b7`.`.](https://bbs.kanxue.com/elink@215K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9X3W2S2L8X3N6%4k6h3V1H3z5e0p5H3y4o6p5H3x3o6l9K6i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5@1z5o6p5H3y4o6f1^5x3b7`.`.)

`kiwisecApplication`中也调用了`a`类。

这个`a.class`条用了居多的资源和库函数，在`so`里的，显然关键的保护就在这两个里面.`kdpdata.so`、`/\.classes.dex`、`zipFile`、`fileDataFile`、`classes_dex`、`zipDexFileName`

![](https://bbs.kanxue.com/upload/attach/201707/734587_txl8rxo0vhkjea3.jpg)

不过有一点不明白，为什么`kdpdata`只在`armabi`文件夹中，先不管了。

本地`so`其实没混淆也没加密。（这里其实后来明白了，动态加载会从这个文件夹加载到其他文件夹的）

![](https://bbs.kanxue.com/upload/attach/201707/734587_zw6pw9ssmzy8m5s.jpg)

接下来一个任务是过掉加固的反调试，另外一个是获取原`apk`的`classes.dex`文件。

需要将`android_server`调试程序的名字修改一下，因为加固会通过查询`/proc/pid/cmdline`文件获取程序的名称来对`android_server`、`gdb`、`ltrace`、`strace`调试器进行反调试。在动态调试加固`apk`程序的时候，需要在几个关键点函数 `open`、`strtol`、`mmap`、`memcmp`处下断点，后面具体的分析。由于[Android](https://bbs.kanxue.com/elink@3f1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6j5W2)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6T1j5i4y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6l9`.`.)系统是由[Linux](https://bbs.kanxue.com/elink@fb4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6j5W2)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6T1j5i4y4W2i4K6u0r3L8r3W2F1N6i4R3`.)系统修改而来，因此`Android`系统上很多的属性和`Linux`系统下的属性。`Linux`下进程和线程的信息是保存在文件中的，因此要查询进程或者线程的信息必须通过`open`或者`fopen`等方式操作文件来完成。

调用`open`函数打开文件`/proc/self/status`时，可以确实是加固外壳`apk`程序通过`TracePid`的反调试

`long int strtol(const char *nptr,char **endptr,int base);` `trtol`函数会将参数`nptr`字符串根据参数`base`来转换成长整型数。

`mmap`将一个文件或者其它对象映射进内存。`void* mmap(void* start,size_t length,int prot,int flags,int fd,off_t offset);`

`prot`：期望的内存保护标志`flags`：指定映射对象的类型，

`int memcmp(const void *buf1, const void *buf2, unsigned int count);`

```
MOVS 同x86的mov LDR 加载内存数据到寄存器 STR 寄存器数据存入内存 B/BL 跳转/函数调用 TST/CMP 比较
```

这些是调试用到的功课，复习一下。

话说到了调试开始各种莫名的错误，我们直接打开调试程序附加显然是不行的，都没法找`dex`的地址和长度了，把`app`以调试模式打开，附加上了但程序根本运行不起来。

`adb shell am start -D -n com.example.testbao/.MainActivity`

最终查到要把程序通过命令行

`jdb -connect com.sun.jdi.SocketAttach:hostname=127.0.0.1,port=8700`

把调试模式的程序运行一下。

**Could not open Selected VM debug port (8700)**

**又有坑了，什么玩意。**

继续查，原来这个命令要开启`ddms`, `sdk-tool`里有这个脚本。直接运行下弹出个程序，这个不管，再`jdb`一下，啊终于成功了。

我们找一下

`libdvm.so`中的`dvmDexFileOpenPartial`函数的下断点。

![](https://bbs.kanxue.com/upload/attach/201707/734587_gu82yu2zgc4c2zs.jpg)

找了好半天。

运行一下，哈哈根本就没进来，程序直接就能运行了。看样子是自己重写了这个函数，不过发现根本没有反调试，这到省事了。

不管是他自己实现`dex`解析加载，最终都是需要把`dex`文件加载到内存中，还是得用`mmap`函数来进行操作。

在`mmap`处下断点，这时候坑了，程序不知为什么出发了异常，应该是反调试。![](https://bbs.kanxue.com/upload/attach/201707/734587_63aw5qywex0r4wv.jpg)

程序异常前是在`sendloop`的函数里，另开一个`ida`查看一下这个玩意。

-EndFragment-->

`int __fastcall kiwisdk::KiwiNet::send_loop(kiwisdk::KiwiNet *this)`

`{`

`kiwisdk::KiwiNet *v2; // [sp+2Ch] [bp-64h]@1`

`int fd; // [sp+30h] [bp-60h]@7`

`int v4; // [sp+34h] [bp-5Ch]@6`

`int v5; // [sp+38h] [bp-58h]@2`

`int v6; // [sp+3Ch] [bp-54h]@2`

`int v7; // [sp+40h] [bp-50h]@2`

`int v8; // [sp+44h] [bp-4Ch]@2`

`char v9; // [sp+48h] [bp-48h]@2`

`int v10; // [sp+4Ch] [bp-44h]@2`

`kiwisdk::KiwiNet *v11; // [sp+54h] [bp-3Ch]@1`

`int v12; // [sp+58h] [bp-38h]@2`

`int *v13; // [sp+5Ch] [bp-34h]@2`

`char *v14; // [sp+60h] [bp-30h]@2`

`int *v15; // [sp+64h] [bp-2Ch]@2`

`int *v16; // [sp+68h] [bp-28h]@2`

`int *v17; // [sp+6Ch] [bp-24h]@2`

`int *v18; // [sp+70h] [bp-20h]@2`

`unsigned int i; // [sp+74h] [bp-1Ch]@2`

`int *v20; // [sp+78h] [bp-18h]@2`

`int *v21; // [sp+7Ch] [bp-14h]@2`

`int *v22; // [sp+80h] [bp-10h]@2`

`int *v23; // [sp+84h] [bp-Ch]@2`

`v2 = this;`

`sub_51EAC();`

`v11 = v2;`

`do`

`{`

`v8 = 5;`

`v14 = &v9;`

`v13 = &v8;`

`v12 = 0;`

`*(_DWORD *)&v9 = 5;`

`v10 = 0;`

`std::__ndk1::this_thread::sleep_for<long long,std::__ndk1::ratio<1ll,1ll>>();`

`v15 = &v5;`

`v16 = &v5;`

`v17 = &v5;`

`v18 = &v5;`

`v7 = 0;`

`v6 = 0;`

`v5 = 0;`

`v21 = &v5;`

`v22 = &v5;`

`v23 = &v5;`

`v20 = &v5;`

`for ( i = 0; i <= 2; ++i )`

`v20[i] = 0;`

`if ( kiwisdk::KiwiNet::get_from_send_queue(v2, &v5) )`

`{`

`fd = -1;`

`if ( kiwisdk::KiwiNet::init_socket(v2, &fd) )`

`{`

`if ( kiwisdk::KiwiNet::connect_server(v2, fd) )`

`{`

`if ( kiwisdk::KiwiNet::send_data(v2, fd, &v5) )`

`{`

`shutdown(fd, 2);`

`close(fd);`

`v4 = 0;`

`}`

`else`

`{`

`shutdown(fd, 2);`

`close(fd);`

`fd = -1;`

`v4 = 2;`

`}`

`}`

`else`

`{`

`v4 = 2;`

`}`

`}`

`else`

`{`

`v4 = 2;`

`}`

`}`

`else`

`{`

`v4 = 2;`

`}`

`std::__ndk1::basic_string<char,std::__ndk1::char_traits<char>,std::__ndk1::allocator<char>>::~basic_string(&v5);`

`}`

`while ( !v4 || v4 == 2 );`

`return sub_51E20();`

`}`

这个函数里有两处关闭程序的代码，我们在分别在函数的最开始处和将要关闭处下断点看一下，当然我们的`mmap`断点也是要下的。运行。

好消息，这段代码不会被触发了，其实不是不被触发，而是中断在了另一个线程里。这里我猜想作者不仅重写了加载`dex`到内存的函数，而且把`mmap`函数的下断调试的处理过程写到了另一个线程。而这个检测过程，我这里有两个思路，一是检测两次调用`mmap`调用的时间差，如果超过了机器运行所需的时间则触发中断。二是利用在`mmap`处的断点检测来触发中断程序。其实我更倾向于第一种，当然完全是猜想。

![](http://bbs.pediy.com/upload/attach/201707/734587_iy6isdmtcz697yq.jpg)

跟踪了无数次，把程序都快跑烂了，发现这个地方最像。再一次`dump`出来，感觉那里还是有些不对。但`dex`的部分应该就是这样，估计有虚拟保护。但有一点，这个加固本身并没有加固除了主类之外的类，![](https://bbs.kanxue.com/upload/attach/201707/734587_xkestope2typm5m.jpg)

比方这个地方，有个`des`的类，完全是源码暴露， 据说这个加固还有什么签名验证的梗，之后再说吧。

不过比较可惜似乎还有部分代码没还原出来，我感觉是加了虚拟保护，只能再说了。

## 总结

1. 刚开始是各种的附加到程序初始化的过程。
2. 然后寻找反调试，开始没找到，但当下到`mmap`断点的时候，有个线程会强制程序退出。
3. 把这个作怪的线程挂掉，程序运行`mmap`，主要是应为`dvmdexopenfile`函数断不下，怀疑重写了。
4. 在出现`.dex`字符串的时候反复跟踪，程序流程最终确定到`.class.dex`的位置`dump`出来`dex`,完毕。

## 后续

之前好像有人也脱过几维的壳，我看了下是针对`dll`文件的，自己动态调试了下`dex`部分，不是很熟，分享下。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
