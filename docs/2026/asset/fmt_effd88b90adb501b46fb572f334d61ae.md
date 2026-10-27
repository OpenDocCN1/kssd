
# [原创] 从应用层到MCU，看Windows处理键盘输入 [1.在应用层调试`Notepad.exe` (按键消费者)]

[接[引言](https://bbs.kanxue.com/thread-276567.htm)]

文本编辑器/文本编辑框是应用层常见的键盘处理程序。微软泄露的WinXP源码下有文本编辑器Notepad的实现：

```
Microsoft_leaked_source_code\nt5src\Source\XPSP1\NT\shell\osshell\accesory\notepad
```

![](https://bbs.kanxue.com/upload/attach/202303/399589_5VNBUJJ78ETDBQG.jpg)(notepad源码结构)

实现一个文本编辑器并不复杂，微软又(被迫)提供了Sample，因此我就不重复造轮子了。本文从调试器的角度观察`Notepad.exe`如何消费(使用)键盘按键。

## 1.Notepad接收按键消息/`WM_CHAR`

首先评估一下调试`Notepad.exe`的难易程度(虽然有源码，我还是装作没有)：Windows窗体程序，在我可以折腾的范围。

![](https://bbs.kanxue.com/upload/attach/202303/399589_3T722WYSKJKEHWM.jpg)

(查壳, 无壳且看着像是C++编译器生成)

![](https://bbs.kanxue.com/upload/attach/202303/399589_6DQZH8BDBS868BU.jpg)

(Spy++查看窗体，`Notepad.exe`属于标准的Windows窗口程序，如果遇到C#窗体，我直接放弃了)

既然(猜测)`Notepad.exe`是标准的窗口程序，那它一定按窗口程序的模板(如下)处理窗口消息，`WM_CHAR`等按键消息的处理亦包含其中:

```
    //窗口消息循环模板

    while (GetMessage((LPMSG)&msg, (HWND)NULL, 0, 0))

    {

        if (TranslateAccelerator(hwndNP, hAccel, (LPMSG)&msg) == 0)

        {

           TranslateMessage ((LPMSG)&msg);

           DispatchMessage ((LPMSG)&msg);

        }

    }
```

在跟踪按键消息的消费者`WM_CHAR`的行为前，先要从茫茫众消息(窗体消息中有大量的鼠标移动的消息干扰分析)中筛选出`WM_CHAR`，思路如下:

1. 搜索并定位`GetMessage` API；

2. 分析`GetMessage`返回的消息，筛选出`WM_CHAR`消息；

下文分步实现上述思路：

### 搜索并定位`GetMessage` API

先用IDA查找并定位`Notepad.exe`调用`GetMessage` API的位置，再用`windbg`下断点：

![](https://bbs.kanxue.com/upload/attach/202303/399589_YGZB78YAG2Y4U9A.jpg)

(根据IDA分析,`Notepad.exe`在`WinMain`中进行消息循环)

```
0:001> x notepad!*main* ;先找符号winMain,再找GetMessage调用处,最后下断点

00007ff7`e4d0ad6c NOTEPAD!wWinMain (<no parameter info>)

 

0:001> uf NOTEPAD!wWinMain ;对wWinMain函数进行反汇编

00007ff7`e4d0b010 488d4d0f        lea     rcx,[rbp+0Fh] ;<--获得窗体消息msg变量的地址

00007ff7`e4d0b014 4533c0          xor     r8d,r8d

00007ff7`e4d0b017 33d2            xor     edx,edx

00007ff7`e4d0b019 48ff1500bc0100  call    qword ptr [NOTEPAD!_imp_GetMessageW (00007ff7`e4d26c20)]

00007ff7`e4d0b020 0f1f440000      nop     dword ptr [rax+rax]

00007ff7`e4d0b025 85c0            test    eax,eax
```

简单说明注记一下上面`windbg`的输出的:

L5处:`GetMessage`需要4个参数，参数1传入窗体消息`MSG msg`的地址。而我的OS是64位系统，所以`Notepad.exe`也是64位程序。而64位程序依次通过`rcx/rdx/r8/r9`传入函数的前4个参数;。`lea rcx`是传入窗体栈变量`msg`的地址;

L9处：运行到L9处时，`GetMessage`调用结束。在此处下断点，查看变量`MSG msg`就可以获得窗体消息。

### 分析`GetMessage`返回的消息，筛选出`WM_CHAR`消息

为了使`windbg`能正确解析各个成员变量，需要明确告知`windbg`从`GetMessage`返回的窗体消息是个`MSG`结构体。

```
0:001> dt combase!MSG

   +0x000 hwnd             : Ptr64 HWND__

   +0x008 message          : Uint4B

   +0x010 wParam           : Uint8B

   +0x018 lParam           : Int8B

   +0x020 time             : Uint4B

   +0x024 pt               : tagPOINT
```

在`GetMessage`返回地址处下断点，当`windbg`断下后，开始解析`MSG`内容:

```
0:001> bp 00007ff7`e4d0b020

0:001> g

Breakpoint 0 hit

 

0:000> dt combase!MSG [rbp+f]

   +0x000 hwnd             : 0x00000000`001001fe HWND__

   +0x008 message          : 0xf

   +0x010 wParam           : 0

   +0x018 lParam           : 0n0

   +0x020 time             : 0xaa1dbe

   +0x024 pt               : tagPOINT
```

上面的片段中:

L1处 在`GetMessage` API的返回地址处下断点

L6处 从`GetMessage` API中获得的窗口句柄，这和前面Spy++获得的窗口句柄值一致

L7处 从`GetMessage` API中获得的消息类型，值0x0f对应`WM_PAINT` (我调试时，`notepad.exe`正好被`windbg`窗口挡住)。在此，我截取了`WinUser.h`中部分消息的定义:

```
#define WM_SETTEXT                      0x000C

#define WM_GETTEXT                      0x000D

#define WM_GETTEXTLENGTH                0x000E

#define WM_PAINT                        0x000F

#define WM_CLOSE                        0x0010
```

窗口程序上会接收到大量消息，这些消息跟噪音一样影响分析。因此需要修改一下前面的断点，让它变为条件断点(条件断点略复杂，请移步[`windbg`设置条件断点](https://bbs.kanxue.com/elink@1fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4A6P5r3W2S2L8X3N6E0K9h3&6Y4K9r3q4@1k6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0t1H3y4e0V1K6y4K6t1`.))，每当`Notepad.exe`中按键，`windbg`打印一串字符(`WM_CHAR` Enter)：

```
0:001> bp 00007ff7`e4d0b020 ".block{r @$t0=poi(rbp+0xf+0x08);.if(@$t0==0x102){.printf @\"WM_CHAR enter\";gc;};.else{gc;}}"

0:000> g
```

看下效果，左边的红框是我在Notepad中随意按键输入，右边是`windbg`相应输出 (作为演示效果挺好的，除了记事本按键半天才给个显示)：

![](https://bbs.kanxue.com/upload/attach/202303/399589_NG7BWS6G9WCBCU8.jpg)

## 2.Notepad处理`WM_CHAR`/显示输出

`Notepad.exe`以内存映射的方式实现文件读写，它将收到的按键值暂存在所映射内存中，通过某种机制(哪种机制?)将这段内存内容显示在文本(文本编辑框)上。如果修改这段内存，是否导致最终文本内容被修改？以修改如下文本为例，自问自答吧:

![](https://bbs.kanxue.com/upload/attach/202303/399589_5Z9Q3C58NDPJK5C.jpg)

用`windbg`在内存中搜索Unicode String，例如State：

```
0:003> s -u 0x20181900000 L?100000 "State" #windbg 搜索指定Unicode string "State"
```

![](https://bbs.kanxue.com/upload/attach/202303/399589_KBX746M57ZDY9J2.jpg)

```
#搜索acpi.h文件中第一行文字，确定所在的内存起始地址:

0:001> s -u 0x20181900000 L?100000 "typedef struct _GAS_20 {"

00000201`81967050  0074 0079 0070 0065 0064 0065 0066 0020  t.y.p.e.d.e.f. .

#以Unicode字符串形式打印起始地址的内容:

0:001> du 00000201`81967050

00000201`81967050  "typedef struct _GAS_20 {..    UI"

00000201`81967090  "NT8...AddrSpcID;          //The "

00000201`819670d0  "address space where the data str"

00000201`81967110  "ucture or register exists...    "

00000201`81967150  "                                "

00000201`81967190  "//Defined values are above      "

00000201`819671d0  "                                "

00000201`81967210  "      ..    UINT8...RegBitWidth;"

00000201`81967250  "..//The size in bits of the give"

00000201`81967290  "n register. ...........//When ad"

00000201`819672d0  "dressing a data structure, this "

00000201`81967310  "field must be zero...    UINT8.."
```

通过`windbg`搜索结果，可以确定文本内容所在的内存地址:0000020181967050。

确定文本所在的起始地址后，准备尝试修改该内存块。如果修改内存后直接会反应到文本上(根据测试结果，需要让窗口重绘才能使得修改生效)，那么可以证明Notepad确实通过内存映射的方式访问文件。修改前我们再核对一下`acpi.h`开头的内容，因为待会马上要整容了:

![](https://bbs.kanxue.com/upload/attach/202303/399589_827USEPGQ8C8EK9.jpg)

```
#以Unicode string方式修改内存

0:004> eu 0x20181967050    "I don't know what to write"

#查看修改结果

0:004> du 0x20181967050    

00000201`81967050  "I don't know what to write    UI"

00000201`81967090  "NT8...AddrSpcID;          //The "

00000201`819670d0  "address space where the data str"

00000201`81967110  "ucture or register exists...    "
```

下图是Notepad的显示输出，看着`acpi.h`的变化证明了我的猜想。

![](https://bbs.kanxue.com/upload/attach/202303/399589_WWY23KYNVNC2XQE.jpg)

## 3.链接键盘输入和显示输出过程

上一节提出了一个问题:Notepad通过某种机制将这段内存内容显示在文本(文本编辑框)上。这一节简单的回答这个问题：

a.输入端:Notepad接收到`WM_CHAR`消息后，通过`DispatchMessage`，将消息传给文本编辑框句柄`hwndEdit`(为什么`hwndEdit`就是文本编辑框的句柄?这个可以参考张银奎老师的《格蠹汇编》一书)；

b.`hwndEdit`所在窗体的Callback处理`WM_CHAR`，将键盘消息插入到内存映射所对应的Unicode String的恰当位置;

c.输出端:由`hwndEdit`调用`SetDlgItemText`将Unicode String显示到`Notepad.exe`对应的文本编辑框。Notepad源码中通过下列方式，从`hwndEdit`窗口句柄获得文本内容:

```
    hEText= (HANDLE) SendMessage( hwndEdit, EM_GETHANDLE, 0, 0 ); //获得文本句柄

    if( !hEText )  // silently return if we can't get it

    {

        return( bStatus );

    }

    pStart= LocalLock( hEText ); //获得文本
```

本文完

下一篇将从应用层（消费者）进入驱动层（传输消息的快递？），看下`i8042.sys`如何处理键盘按键。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2023-3-21 18:41

被hyjxiaobia编辑

，原因：

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)
