
这里我们以下述简单样例【`consoleTest.cpp`】为目标，命令行启动相应的编译缓解【`vcvars32.bat`】或【`vcvars64.bat`】然后，直接一个命令【`cl consoleTest.cpp`】编译链接即可。

```
//File consoleTest.cpp

// to compile

// 32-bit: use "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars32.bat"

// 64-bit: use "C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\VC\Auxiliary\Build\vcvars64.bat"

//cl consoleTest.cpp

#include <iostream>

#define XBUF_LEN 0x100
int main(int argc, char* argv[]) {
    static char gbuf[XBUF_LEN] = { 0 };
    char lbuf[XBUF_LEN] = { 0 };
    int i = 0;
    char msg[3][XBUF_LEN] = { "abc message","c++ pro","ida frida" };
    for (i = 0; i < 3; i++) {
        memcpy(lbuf, msg[i], strlen(msg[i]));
        memcpy(gbuf, lbuf, strlen(lbuf));
        std::cout << gbuf << std::endl;
    }
    return 0;
}
```

通过`pin`对目标执行`pintool`进行采集，这里适用32位的`pin`加载32位的`pintool`插件对32位对象进行跟踪采集，由于`consoleTest.exe`目标体量比较小，这里我们没有适用`pintool`插件的`-w`白名单参数进行过来跟踪，而是对所有模块执行跟踪记录。应该注意的是，对于稍微复杂些的目标，如简单的多线程`MFC`目标，如果不适用白名单参数进行过滤，几乎不能正常执行；另外作者发布的`pintool`插件的白名单`-w`参数并不像`Tenet`说明上说的，多个模块用逗号分隔即可；因为种原因，我对作者的`pintool`工具进行的部分改动，改动后的源码和编译好的`pintool`参考附件

```
c:\PinDir\ia32\bin\pin.exe -t d:\src\tenet\tracers\pin\obj-ia32\pintenet.dll  -- d:\xtest\consoleTest.exe
```

会在命令行中得到如下输出

```
[tritium] main :PIN_InitSymbols()

OK!

[tritium] main :PIN_Init(argc, argv)

        argv[0] : c:\PinDir\ia32\bin\pin.exe

        argv[1] : -t

        argv[2] : D:\src\tenet\tracers\pin\obj-ia32\pintenet.dll

        argv[3] : --

        argv[4] : D:\xtest\consoleTest.exe

[tritium] main : white-listing-count is 0

[tritium] main: PIN_StartProgram()

Loaded image: 0xed0000:0xefdfff -> consoleTest.exe

Loaded image: 0x75440000:0x7552ffff -> kernel32.dll

Loaded image: 0x75530000:0x75742fff -> KernelBase.dll

Loaded image: 0x774f0000:0x77691fff -> ntdll.dll

abc message

c++ prosage

ida fridage

Loaded image: 0x74630000:0x7463efff -> kernel.appcore.dll

Loaded image: 0x76360000:0x7641efff -> msvcrt.dll

Loaded image: 0x76420000:0x764d9fff -> rpcrt4.dll
```

这个是针对`console`样例目标才有的输出，因为原`pintool`对桌面型如`MFC`目标不会有上述信息输出，所以我做了些许改动，无论对`console`还是桌面样例，上述信息都会追加到`cmd`当前目录的`lg.txt`文件中，通过`lg.txt`文件可以查阅上述信息。如下

一般线程`0`是我们在`IDA`的`Tenet`中要适用的跟踪记录数据。

```
2021/04/23  19:06               747 lg.txt

2021/04/23  19:06        84,210,271 trace.TID0.log

2021/04/23  19:06         9,046,216 trace.TID1.log
```

如果要适用`pintool`的白名单参数，可以通过`-w`多次指定多个模块而不是`Tenet`说明的用逗号分隔

```
c:\PinDir\ia32\bin\pin.exe -t d:\src\tenet\tracers\pin\obj-ia32\pintenet.dll  -w consoleTest.exe -w kernel32.dll -- d:\xtest\consoleTest.exe
```

有了采集的跟踪记录，即可在`IDA`对【`consoleTest.exe`】的反编译中通过`Tenet`加载跟踪记录。

应该注意到，上述`consoleTest.exe`的加载基址是随机的 `0xed0000`，采集的跟踪记录时基于这些基址的，

所以在需要多相应的分析模块的基址重定位位采集数据的基址，如`IDA Pro`中`consoleTest.exe`的基址是`0x400000`,

如图，这里【`Edit》Segments》Rebase progam`】进行重定位

![](https://bbs.kanxue.com/upload/attach/202104/760675_UXH7ZZXUUX9FC68.jpg)

修改位采集记录的模块实际基址

![](https://bbs.kanxue.com/upload/attach/202104/760675_4EZS6F24KAA6FEM.jpg)

然后通过【`File》Load File 》Tenet trace file`】加载【`trace.TID0.log`】即可，一般加载主线程执行跟踪记录即可。这时刻就可以愉快的对`Tenet`插件进行上述第一部分的提及的适用方式进行测试了。

### 三、后话

（1）`Tenet`的技术路线的确炫酷，但在执行记录的采集上可能比较痛苦，虽然严格上已经不算是`Tenet`的范畴。

（2）在测试时，一个简单的多线程`MFC`如果没有过滤参数的参与，基本无法让目标正常运行，如下述对【确定】按钮响应函数只添加三行代码，几乎跑不起来。

![](https://bbs.kanxue.com/upload/attach/202104/760675_5C4K8NGSCMHX25A.jpg)

还没完全跑起来，数据已达到惊人的`30G`，从结束进程的时间上看，这个可能也是错误测试导致的结果，比较已经执行了四个小时，的确有些痛苦，跟踪记录数据的采集可能让`Tenet`的应用有些骨感，还需要更好的执行根据记录采集方案，如果道友有所发现，不吝分享一二。

![](https://bbs.kanxue.com/upload/attach/202104/760675_F8Z9425HGJHG632.jpg)

（3）关于`Intel`的`Pin`

上次见到它不晓得是几年前了，如下图所示介绍。`Pin`是一种对程序进行监测的工具。主要手段是在执行目标任意位置插入任意监测代码。主要的监测代码是通过为`Pin`提供插件的形式提供的，如以`pin`的`-t`参数提供`pintool`监测代码。而维护程序或指令执行状态不变的过程对开发`pintool`插件的开发人员是透明的。

`instrumentation`这个词

![](https://bbs.kanxue.com/upload/attach/202104/760675_82JY5GD34FJNNQH.jpg)

无独有偶，我们可以看看`frida`的跟踪命令，【`Instrumenting...`】。

监测思路上相似，`Pin`最基础的可以基于指令粒度进行监测，当然也可以对分块的指令流，某些函数、线程或进程状态进行监测。

`frida`的监测代码是通过`js`等方式添加的，而`Pin`的监测代码称为`pintool`则通过`C\C++`代码的以插件的方式添加通过为`Pin`指定`-t`参数

如`MessageBoxExA.js`在`frida`中实现对`MessageBoxExA`函数的监测；而`pintenet.dll`提供对执行指令的跟踪记录在`pin`中实现对目标程序执行指令的监测`pin`说明文档虽然不建议在指令的粒度上进行监测。

```
python frida_tools\tracer.py -i "MessageBox*" -p 33068

Instrumenting...

MessageBoxExA: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxExA.js"

MessageBoxExW: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxExW.js"

MessageBoxTimeoutA: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxTimeoutA.js"

MessageBoxIndirectA: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxIndirectA.js"

MessageBoxTimeoutW: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxTimeoutW.js"

MessageBoxIndirectW: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxIndirectW.js"

MessageBoxA: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxA.js"

MessageBoxW: Loaded handler at "D:\\\\src\\frida\\frida-tools\\build\\lib\\__handlers__\\USER32.dll\\MessageBoxW.js"

Started tracing 8 functions. Press Ctrl+C to stop.
```

附件清单，如下，使用32或64位的`pin.exe`和`pintenet.dll`即可对目标如`consoleTest.exe`进行跟踪记录采集。如需重新编译`pintenet.dll`，需要到`Intel`的`Pin`官方下载`Pin`的完整开发包；`makefile.win.config`替换`Pin`开发包对于的文件或直接参考上述编译命令执行。

`Tenet.zip`只官方的`IDA` 插件部分，直接解压释放到`IDA`的`plugins`目录即可

```
SRCBIN

│  makefile.win.config

│

├─pin                         

│  ├─ia32

│  │  └─bin

│  │          dbghelp.dll

│  │          dia_client.dll

│  │          msdia140.dll

│  │          pin.exe

│  │          pincrt.dll

│  │          pindb.exe

│  │          pinjitprofiling.dll

│  │          pinvm.dll

│  │

│  └─intel64

│      └─bin

│              dbghelp.dll

│              dia_client.dll

│              msdia140.dll

│              pin.exe

│              pincrt.dll

│              pindb.exe

│              pinjitprofiling.dll

│              pinvm.dll

│

├─tenet

│  └─traces

│      └─pin

│          │  ImageManager.cpp

│          │  ImageManager.h

│          │  Makefile

│          │  pintenet.cpp

│          │  README.md

│          │

│          ├─obj-ia32

│          │      pintenet.dll

│          │

│          └─obj-intel64

│                  pintenet.dll

│

└─xtest

        consoleTest.cpp

        consoleTest.exe

        lg.txt

        trace.TID0.log

        trace.TID1.log
```

（4）`Tenet`插件快速测试

A、`Tenet.zip`中`plugins`里的全部文件和文件夹解压释放到`IDA 7.5`的`plugins`目录。

B、`IDA`加载`srcbin`中的`xtest\consoleTest.exe`反编译，并通过【`Edit》Segments》Rebase progam`】进行重定位为`0xed0000`。

C、通过【`File》Load File 》Tenet trace file`】加载【`trace.TID0.log`】

愉快地和`Tenet`玩耍。

【参考文献】

1、参见开头`py`代码解密出来的 [737K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7X3g2@1x3W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9@1i4K6u0r3x3U0m8Q4x3V1k6@1k6h3&6W2N6q4)9J5k6s2c8J5j5h3y4W2i4K6u0V1k6i4S2H3L8r3!0J5k6i4u0Q4x3V1j5`.](https://bbs.kanxue.com/elink@67fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7X3g2@1x3W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9@1i4K6u0r3x3U0m8Q4x3V1k6@1k6h3&6W2N6q4)9J5k6s2c8J5j5h3y4W2i4K6u0V1k6i4S2H3L8r3!0J5k6i4u0Q4x3V1j5`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-5-6 09:09

被`tritium`编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [srcbin.zip](javascript:void(0);)
- [srcbin.z01](javascript:void(0);)
- [srcbin.z02](javascript:void(0);)
- [Tenet.zip](javascript:void(0);)
