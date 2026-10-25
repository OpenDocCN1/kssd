
# [旧帖][原创]逆向工程入门引导0.00雪花

最近在准备简历忙找工作，北京有很多信息安全类的公司，对于个人发展很不错。我是南方人，由于各方面因素，还是要在珠三角工作。

《加密解密》第三版写得很好，是每个新手入门必看的书籍。

在此感谢`kanxue`老师的指导。

这个文章，是我07年写的，今天把我学习逆向的经验跟大家分享下。

## 前言

### 为什么学习逆向工程呢？

1. 分析病毒
2. 软件破解
3. 寻找程序漏洞
4. 分析程序工作流程
5. 研究系统底层
6. 开发出更好的软件

### 建议

学习逆向工程的朋友，一定要掌握`C/C`程序设计和汇编语言程序设计，磨刀不误砍柴功。

## 一、函数

### 1、函数

函数是程序设计最重要的结构单元。学习逆向工程最重要的是函数。一个函数有如下几个部分：入口参数，函数名，返回值，函数功能。

首先我们从最基本的`C`函数开始，在开始代码编写前，我想建议大家使用`VC6`来完成我们的调试和反编译过程。在开始的文档中，我们仅仅使用`VC`来进行各种逆向的说明，其后我们会使用`ollydbg`等反编译工具做进一步的冒险历程。

A: 打开`VC6`编译器。

B: `File->New->Projects->(选择)Win32 Console Application->(输入)Project Name->OK`

![](https://bbs.kanxue.com/upload/attach/200909/281314_t184sypyhr6ydrz.png)

C: `Project->Add To Project->New->(选择)C++ Source File->File->(输入) fun_1.c ->OK.`

![](https://bbs.kanxue.com/upload/attach/200909/281314_minib3vgd0q22n2.png)

D.在编辑器里面输入以下程序：

```
#include "iostream.h"

int callee (int a,int b)

{

    int local;

    if (local==5)

        return 0;

    else

        return 2;

}

int main(int argc,char * argv[])

{

    int c=5;

    int d=7;

    int v=callee(c,d);

    return 0;

}
```

![](https://bbs.kanxue.com/upload/attach/200909/281314_v7xdp4pmpj2aeks.png)

E: 编译，连接成功后,F11跟踪到`fun`,alt+8看反汇编代码：

![](https://bbs.kanxue.com/upload/attach/200909/281314_5h4gnpvt0f87dja.png)

`--- d:\c&asm\fun\_1\fun\_1.c  --------------------------------------------------------`

```
11:
12:   int main(int argc,char * argv[])
13:   {
00401060   push        ebp                 ;保存ebp,并把esp 放入ebp 中.
00401061   mov         ebp,esp             ;此时ebp 与esp 同.
00401063   sub         esp,4Ch             ;把esp 往下移动一个范围，等于在堆栈中放出一片新的空间用来存局部变量
00401066   push        ebx                 ;下面保存三个寄存器:ebx,esi,edi,这是C的规范.
00401067   push        esi
00401068   push        edi
00401069   lea         edi,[ebp-4Ch]
0040106C   mov         ecx,13h
00401071   mov         eax,0CCCCCCCCh
00401076   rep stos    dword ptr [edi]         ;把局部变量区域初始化成0cccccccch。
                                          ;0cccccccch 实际是INT 3.这是一个中断指令。
                                          ;因为局部变量不可能被执行，如果执行了必然
                                          ;程序有错，这时发生中断来提示开发者。这是
                                          ;VC 编译DEBUG 版本的特点。
```

堆栈指令操作

`PUSH SRC`

`push eax`

`push [eax]`

`push 1234`

`esp <- esp-4`

`[esp] <- SRC`

`POP DEST`

`pop ebp`

`pop [ebx]`

`DEST <-[esp]`

`esp <- ESP+4`

`MOV DEST,SRC`

`mov eax,ebx`

`mov [ebx].1234`

`DEST <- SRC  //把SRC复制到DEST`

以后分析程序的时候，这部分代码一般不用看，我们看接下来的反汇编语句:

```
14:       int c=5;
00401078   mov         dword ptr [ebp-4],5    ;给局部变量赋值
15:       int d=7;
0040107F   mov         dword ptr [ebp-8],7
```

执行完以上这两句，由下图的`EIP`可以看到`00401086`，就是下条要执行的指令。

![](https://bbs.kanxue.com/upload/attach/200909/281314_6s78j6xrq0svvsq.png)

在`Memory`窗口中可以看到内存数据`000007`和`000005`实质上是`7`和`5`(intel处理器一般都是小端存储)。

![](https://bbs.kanxue.com/upload/attach/200909/281314_cqaqib0r7heus8a.png)

```
16:

17:       int v=callee(c,d);
00401086   mov         eax,dword ptr [ebp-8]  ;参数压栈，这里遵循__cdecl调用规范，参数由右向左
00401089   push        eax
0040108A   mov         ecx,dword ptr [ebp-4]
0040108D   push        ecx
0040108E   call        @ILT+0(callee) (00401005)   ;函数调用,下一行地址00401093入栈
00401093   add         esp,8                     ;主程序平衡堆栈
00401096   mov         dword ptr [ebp-0Ch],eax
```

单步执行到`0040108A`，我们看下`Memory`窗口，参数`7`被复制到`0012FF24`中。

![](https://bbs.kanxue.com/upload/attach/200909/281314_3q1x1aw1yeat7y4.png)

当程序执行到`0040108E`，我们看下`Memory`窗口，参数`5`被复制到`0012FF20`中。

![](https://bbs.kanxue.com/upload/attach/200909/281314_tip3m9lbasgxrra.png)

从上面这两张图可以得到，堆栈在内存中是往低地址生长的。现在我们再讨论这个过程所涉及的数据结构：“栈（stack）”。

提起“栈“，第一反应是它具有后进先出（LIFO）的特性。

接下来到了我们的函数调用，这里我们按`F11`进去函数里面看看。

在这之前，我们有必要解释下`call`指令的本质，它完成两个操作：

`push eip  ;下图保存的是00401093,函数调用完返回执行的下一条语句`

`jmp  函数入口地址`

![](https://bbs.kanxue.com/upload/attach/200909/281314_itatfdbscj7hpwv.png)

![](https://bbs.kanxue.com/upload/attach/200909/281314_3cxknb5vk15504p.png)

进去函数

![](https://bbs.kanxue.com/upload/attach/200909/281314_by6peejkfk2h6si.png)

整个函数的调用过程可以用以下的图来说明：

![](https://bbs.kanxue.com/upload/attach/200909/281314_p2084y3ehqzl4pw.jpg)

1.将函数参数入栈。

2.执行`CALL`指令，调用该函数，进入该函数代码空间。

a.执行`CALL`指令，将`CALL`指令下一行代码的地址入栈。

b.进入函数代码空间后，将基址指针`EBP`入栈，然后让基址指针`EBP`指向当前堆栈栈顶，并使用它访问存在堆栈中的函数输入参数及堆栈中的其他数据。

c.堆栈指针`ESP`减少一个值，如`44H`，向上移动一个距离，留出一个空间给该函数作为临时存储区。

{
// 以上准备工作做好后，函数正式被执行，如下所示。
d.将其他指针或寄存器中的值入栈，以便在函数中使用这些寄存器。
e.执行代码。
f.执行`return()`返回执行结果，将要返回的值存入`EAX`中。
g.步骤2.d中的指针出栈。
}

h.将`EBP`的值传给堆栈指针`ESP`，使`ESP`复原为2.c之前的值。此时进入函数时`EBP`的值在栈顶。

i.基址指针`EBP`出栈，复原为2.b之前的`EBP`的值。

j.执行`RET`指令，“调用函数”的地址出栈，本函数返回到`CALL`指令的下一行。

3.函数返回到`CALL`指令下一行，将堆栈指针加一个数值，以使堆栈指针恢复到以上步骤1执行之前的值。 该数值是上面第一步入栈参数的总长度。

注意：

1.`堆栈指针ESP`指向栈顶的新入栈数据的最低位。

2.`MOV`指令中偏移指针指向被“`MOV`”的数据的最低位。如下面指令是将`ebp+8`到`ebp+11`四个字节的内容传到`eax`寄存器中。

```
00402048 mov eax,dword ptr [ebp+8]
```

注:这段英文部分，是我以前在网上摘录的，已经忘记在那个文章的了。

`Problem: I program in C/C++ main()? Where is main()?`

`Solution: In Windows XP (and its ancestors), programs written in  C/C++ using Microsoft Visual C++ don’t actually start executing from the main() function. Instead, after the image is loaded into memory, control is passed to the Startup()23 function located in crt0.c (or in crtexe.c for dynamic linking or in`

`wincmdln.c for console applications). This function initializes the global variables argv, argc, _osver, _winmajor, _winminor, _winver, and environ; initializes the heap for the process; calls main(); and exits when main() returns.`

The important thing to remember is that the `Start()`function always passes some (3 or 4) arguments(argc, argv, and environ) to the `main()` function:

```
#ifdef WPRFLAG
            lpszCommandLine = \_wwincmdln();
            mainret = wWinMain(
#else  /\* WPRFLAG \*/
            lpszCommandLine = \_wincmdln();
            mainret = WinMain(
#endif  /\* WPRFLAG \*/
                               GetModuleHandleA(NULL),
                               NULL,
                               lpszCommandLine,
                               StartupInfo.dwFlags & STARTF\_USESHOWWINDOW
                                    ? StartupInfo.wShowWindow
                                    : SW\_SHOWDEFAULT
                             );
#else  /\* \_WINMAIN\_ \*/
#ifdef WPRFLAG
            \_\_winitenv = \_wenviron;
            mainret = wmain(\_\_argc, \_\_wargv, \_wenviron);
#else  /\* WPRFLAG \*/
   \_\_initenv = \_environ;
            mainret = main(\_\_argc, \_\_argv, \_environ);
#endif  /\* WPRFLAG \*/
```

Looking at the disassembled `start()` function of `Bagle`, from `IDA Pro`:

```
.text:00401120;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* SU B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
.text:00401120
.text:00401120 ; Attributes: library function bp-based frame
.text:00401120
.text:00401120                 public mainCRTStartup
.text:00401120 mainCRTStartup  proc near
.text:00401120
.text:00401120 var_1C          = dword ptr -1Ch
.text:00401120 var_18          = dword ptr -18h
.text:00401120 var_4           = dword ptr -4
.text:00401120
.text:00401120                 push    ebp
.text:00401121                 mov     ebp, esp
.text:00401123                 push    0FFFFFFFFh
.text:00401125                 push    offset stru_422138
.text:0040112A                 push    offset __except_handler3
.text:0040112F                 mov     eax, large fs:0
.text:00401135                 push    eax
.text:00401136                 mov     large fs:0, esp
.text:0040113D                 add     esp, 0FFFFFFF0h
.text:00401140                 push    ebx
.text:00401141                 push    esi
.text:00401142                 push    edi
.text:00401143                 mov     [ebp+var_18], esp
.text:00401146                 call    ds:GetVersion() ; Get current version number of Windows
.text:00401146                                         ; and information about the operating system platform
.text:0040114C                 mov     _osver, eax
.text:00401151                 mov     eax, _osver
.text:00401156                 shr     eax, 8
```
