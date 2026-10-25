
# 第一章: 1.1、寻找 main 函数入口

> 原文：[`www.pediy.com/kssd/pediy11/113691.html`](https://www.pediy.com/kssd/pediy11/113691.html)

逆向的第一步是什么？这要问你学习 C 语言的第一步是什么，很自然的，逆向的第一步当然也是大名鼎鼎“HelloWorld!”了。但是也不要因此就误认为这一节会很简单，如果你是第一次接触逆向的话，那么这一节还是有些难度的。

好的，让我们先写一个世界上最出名的程序：

```
int _tmain(int argc, _TCHAR* argv[])

{

    printf("Hello World!\r\n");

    return 0;

}
```

不错！很好的开始！然后用 `VS2008` 以 `Debug` 方式编译下，再用 `OllyDbg` 打开看看：

```
00411078 >JMP Test_0.004117B0

0041107D  JMP Test_0.00412CC0

00411082  JMP <JMP.&MSVCR90D._lock>

00411087  JMP <JMP.&KERNEL32.GetProcAddress>

0041108C  JMP Test_0.00411440

00411091  JMP Test_0.00413310

00411096  JMP <JMP.&MSVCR90D.?terminate@@YAXXZ>

0041109B  JMP <JMP.&MSVCR90D._exit>

004110A0  JMP <JMP.&KERNEL32.GetCurrentThreadId>

004110A5  JMP <JMP.&MSVCR90D._initterm>
```

看看我们的程序停在了什么鬼地方，如果各位初学读者试图从这里就开始分析的话那真的很恐怖，相信 30 分钟内你的自信心将被打击到零……

我们都知道其实编译器在编译我们的程序前会做很多准备工作，而这些准备工作由于涉及的东西较多且每个由此编译器生成的程序都一样，因此我们不必深究，只需快速且准确的找到 `main` 函数即可。

但是这对于初学逆向的朋友来说也是最难的，下面我就教各位读者怎样突破这个障碍。

想要找到 `main` 函数，那么我们就要从 C 语言本身讲起，在刚刚开始学习 C 语言的时候我们就被不幸的告知，我们的程序中必须要包含一个名字叫做 `main` 的函数，不管你多讨厌它都必须如此，后来便成了习惯……

后来查查 `C99` 标准，发现“`int main(int argc, char *argv[])`”与“`int main(void)`”都是被接受的，然后又查查 `MSDN`，可以清晰看到一句话“`The main and main functions can take the following three optional arguments`”，也就是告诉了我们 `main` 函数其实是有 3 个参数的，其后面的例子更是证明了这句话确实是微软写上去的：

```
main( int argc, char *argv[ ], char *envp[ ] )
```

嗯，他们又在标准上较劲了，但是考虑到我们大部分程序都是用 `vs` 编译的（而且 `Borland` 的 C++的参数也是如此），因此我们还是做墙头草，随大流吧……

到这里有的读者可能会感到疑惑，如果我们使用的是符合 `C99` 标准的 `main` 函数呢？例如我们源码的 `main` 函数不就是两个参数吗。但是在这里我要很负责的告诉大家，不管我们代码中实际使用了几个参数，在程序被编译时其 `main` 函数肯定是三个参数的，因为这取决于 `Windows` 系统的机制。

因此现在已经为我们识别 `main` 函数提供了很好的特征，既有三个参数，且前两个参数为地址量的 `call` 就应该是我们的 `main` 函数了。除此之外，我们通过 `MSDN` 可知应用程序会随着 `main` 函数结束而退出，这又给了我们第二个有力的特征，既 `main` 函数很定是在程序退出代码附近的（而且目前的主流调试、反汇编工具都可以正确识别出退出函数 `exit`）。

有了这些特征，我们再想找到 `main` 函数就不难了，目前我为大家提供三种方法：

### 1.1.1、字符串搜索法

安装完各个版本的 C++编译器后，逐个写 `Hello World`，然后用 `OllyDbg` 的搜索字符串功能搜索这个字符串，最后逐步回溯即可，下面我为大家演示一下我做的步骤。

用 `OllyDbg` 打开目标文件后，先记住程序默认停在哪里，然后在 `CPU` 窗格点击右键，依次选择【超级字符串参考】>【查找 ASCII 字符】，选择我们的“`Hello World`”后双击即可到 `main` 函数中，代码如下：

```
004113A0  PUSH EBP                             ; 函数入口

004113A1  MOV EBP, ESP

004113A3  SUB ESP, 0C0

004113A9  PUSH EBX

004113AA  PUSH ESI

004113AB  PUSH EDI

004113AC  LEA EDI, DWORD PTR SS:[EBP-C0]

004113B2  MOV ECX, 30

004113B7  MOV EAX, CCCCCCCC

004113BC  REP STOSD

004113BE  MOV ESI, ESP

004113C0  PUSH Test_0.0041573C                 ; /Hello World!\r\n

004113C5  CALL DWORD PTR DS:[<&MSVCR90D.printf>] ; \printf

004113CB  ADD ESP, 4

004113CE  CMP ESI, ESP

004113D0  CALL Test_0.00411145

004113D5  XOR EAX, EAX

004113D7  POP EDI

004113D8  POP ESI

004113D9  POP EBX

004113DA  ADD ESP, 0C0

004113E0  CMP EBP, ESP

004113E2  CALL Test_0.00411145

004113E7  MOV ESP, EBP

004113E9  POP EBP

004113EA  RETN
```

我们单击选择函数入口后，可以看到 `CPU` 窗格下面的信息窗格中显示如下信息：

跳转来自 `0041100F`

我们单击选择此信息后，点击鼠标右键，并选择【转到 JMP 来自 `0041100F`】后即可来到上层调用函数（以后我们将之称为“返回到调用”）：

```
0041100A  JMP <JMP.&KERNEL32.DebugBreak>

0041100F  JMP Test_0.004113A0                   ; 我们停在这里

00411014  JMP Test_0.004124E0
```

遇到这种情况直接在返回到调用，此时来到真正调用 `main` 函数的地方：

```
0041195F  MOV EAX, DWORD PTR DS:[417148]

00411964  PUSH EAX

00411965  MOV ECX, DWORD PTR DS:[41714C]

0041196B  PUSH ECX

0041196C  MOV EDX, DWORD PTR DS:[417144]

00411972  PUSH EDX

00411973  CALL Test_0.0041100F                   ; 我们停在这里

00411978  ADD ESP, 0C

0041197B  MOV DWORD PTR DS:[41715C], EAX

00411980  CMP DWORD PTR DS:[417150], 0

00411987  JNZ SHORT Test_0.00411995

00411989  MOV EAX, DWORD PTR DS:[41715C]

0041198E  PUSH EAX                               ; /status => 0

0041198F  CALL DWORD PTR DS:[<&MSVCR90D.exit>]   ; \exit
```

通过上面的代码我们便看到了 `main` 函数的典型特征，临近 `exit`，且有三个参数。接下来我们要做的就是不断地重复上面的步骤，一直到找到程序入口点为止。

最后你要做的就是针对不同的版本不同城上的编译器重复上面的步骤，直到收集到你认为足够丰富的信息后结束，从此你就再也不用怕为找不到 `main` 函数而苦恼了。

### 1.1.2、栈回溯法

栈回溯的方法是先找到 `main` 函数中的那个“`HelloWorld`”，下断点并按【F9】键运行后查看堆栈情况，我这里的堆栈情况如下：

```
0012FE9C   7C930208  ntdll.7C930208       ; 我们停在这里

0012FEA0   FFFFFFFF

0012FEA4   7FFDE000

0012FEA8   CCCCCCCC

    ……   ……

0012FF64   CCCCCCCC

0012FF68  /0012FFB8

0012FF6C  |00411978  返回到 Test_0.00411978 来自 Test_0.0041100F

0012FF70  |00000001

0012FF74  |003D2C60

0012FF78  |003D2D40

0012FF7C  |0A641DBC

0012FF80  |7C930208  ntdll.7C930208

0012FF84  |FFFFFFFF

0012FF88  |7FFDE000

0012FF8C  |00369E99

0012FF90  |00000000

0012FF94  |00000000

0012FF98  |00130000  ASCII "Actx "

0012FF9C  |00000000

0012FFA0  |0012FF7C

0012FFA4  |00000020

0012FFA8  |0012FFE0  指向下一个 SEH 记录的指针

0012FFAC  |0041107D  SE 处理程序

0012FFB0  |0A3788D4

0012FFB4  |00000000

0012FFB8  \]0012FFC0

0012FFBC  |004117BF  返回到 Test_0.004117BF 来自 Test_0.004117D0

0012FFC0  \\0012FFF0

0012FFC4   7C817077  返回到 kernel32.7C817077
```

对于这些信息我们只需要关注注释前面有“返回到”三个字的，离我们最近是：

```
0012FF6C  |00411978  返回到 Test_0.00411978 来自 Test_0.0041100F
```

鼠标单击选择该项后，按【Enter】键即可来到返回地址 `00411978` 处：

```
0041195F   .  A1 48714100   MOV EAX, DWORD PTR DS:[417148]

00411964   .  50            PUSH EAX

00411965   .  8B0D 4C714100 MOV ECX, DWORD PTR DS:[41714C]

0041196B   .  51            PUSH ECX

0041196C   .  8B15 44714100 MOV EDX, DWORD PTR DS:[417144]

00411972   .  52            PUSH EDX

00411973   .  E8 97F6FFFF   CALL Test_0.0041100F

00411978   .  83C4 0C       ADD ESP, 0C                            ; 我们停在这里

0041197B   .  A3 5C714100   MOV DWORD PTR DS:[41715C], EAX

00411980   .  833D 50714100 >CMP DWORD PTR DS:[417150], 0

00411987   .  75 0C         JNZ SHORT Test_0.00411995

00411989   .  A1 5C714100   MOV EAX, DWORD PTR DS:[41715C]

0041198E   .  50            PUSH EAX                                 ; /status => 0

0041198F   .  FF15 80824100 CALL DWORD PTR DS:[<&MSVCR90D.exit>]    ; \exit
```

此时我们又来到了这个熟悉的地方，接下来的事情就要各位读者自己发挥了（重复上面的步骤）。

### 1.1.3、逐步分析法

以上讲的两种方法都是在学习与知识储备时用的，不可能收到什么实战效果。假如我们现在碰到了一个现在就需要我们分析的软件，而且它的编译环境我们以前没碰到过，这就要求我们纯手工分析并找到 `main` 函数了。

之所以将之称为逐步分析法，是因为我们不需要阅读它代码的具体含义，而是只需要以 `JMP` 与 `CALL` 为单位逐个跟进，从而根据 `main` 函数的特征判定 `main` 函数的所在位置。

其实这种方法有点类似于文件搜索，先搜索根目录、在逐层加深搜索其子目录，直到找到我们需要的东西。

那我们的程序为例，我们的 `OEP` 处就是一个 `JMP`，因此其“根目录”也就是第一层代码里是不可能有我们的 `main` 函数了，当我们跟进这个 `JMP` 后会发现如下代码：

```
004117B0   > \8BFF           MOV EDI, EDI

004117B2  /.  55            PUSH EBP

004117B3  |.  8BEC          MOV EBP, ESP

004117B5  |.  E8 96F8FFFF   CALL Test_0.00411050

004117BA  |.  E8 11000000   CALL Test_0.004117D0

004117BF  |.  5D            POP EBP

004117C0  \\.  C3           RETN
```

我们发现第二层代码里也没有我们的 `main` 函数，但是有两个 `CALL`。因此我们跟进第一个 `CALL` 中，为了节省篇幅，我在这里就不贴出代码了，我在这里并没有发现 `main` 函数，但是发现了数个 `JMP` 与 `CALL`。不过需要注意的是，我们一定要注意采用逐层搜索的思想，因此这里的 `CALL` 与 `JMP` 就不要再继续跟下去了，我们现在要住的是返回上一层，看看第二个 `CALL` 里是什么：

```
004117D0  MOV EDI, EDI

004117D2  PUSH EBP

004117D3  MOV EBP, ESP

004117D5  PUSH -2

    ……  ……

00411813  CALL Test_0.004110FF

    ……  ……

00411830  CALL DWORD PTR DS:[<&KERNEL32.Interlocke>;  kernel32.InterlockedCompareExchange

    ……  ……

0041184E  JMP SHORT Test_0.0041185D

00411850  PUSH 3E8                       ; /Timeout = 1000. ms

00411855  CALL DWORD PTR DS:[<&KERNEL32.Sleep>]    ; \Sleep

0041185B  JMP SHORT Test_0.00411825

    ……  ……

004118EB  PUSH Test_0.004157C8             ;  _

004118F0  PUSH 0

004118F2  PUSH 1F4

004118F7  PUSH Test_0.00415750             ;  f

004118FC  PUSH 2

004118FE  CALL DWORD PTR DS:[<&MSVCR90D._CrtDbgRep>;  MSVCR90D._CrtDbgReportW

00411904  ADD ESP, 14

    ……  ……

00411913  PUSH 0                          ; /NewValue = 0
```



```
00411915   PUSH Test_0.0041756C             ; |pTarget = Test_0.0041756C  

0041191A   CALL DWORD PTR DS:[<&KERNEL32.Interlocke>;  \InterlockedExchange  

    ……   ……  

00411929   PUSH Test_0.00417590  

0041192E   CALL Test_0.00411172  

00411933   ADD ESP, 4  

    ……   ……  

0041193A   PUSH 0  

0041193C   PUSH 2  

0041193E   PUSH 0  

00411940   CALL DWORD PTR DS:[417590]             ; 注意这里，虽然这个 CALL 也有三个参数，但是仔细分析一下我们就会发现  
00411940                   ; 这并不是 main 函数，因为 main 函数的后两个参数是指针，这里的 0 与 2 显然  
00411940                   ; 不符合要求。其次他也并非是临近 exit 的。  

00411946   PUSH 1  

00411948   CALL DWORD PTR DS:[<&MSVCR90D._CrtSetChe>;  MSVCR90D._CrtSetCheckCount  

    ……   ……  

0041195F   MOV EAX, DWORD PTR DS:[417148]  

00411964   PUSH EAX  

00411965   MOV ECX, DWORD PTR DS:[41714C]  

0041196B   PUSH ECX  

0041196C   MOV EDX, DWORD PTR DS:[417144]  

00411972   PUSH EDX  

00411973   CALL Test_0.0041100F             ; 终于来到我们熟悉的 main 函数里了！  

00411978   ADD ESP, 0C  

    ……   ……  

0041198E   PUSH EAX                 ; /status => 0  

0041198F   CALL DWORD PTR DS:[<&MSVCR90D.exit>]     ; \exit  

    ……   ……  

0041199E   CALL DWORD PTR DS:[<&MSVCR90D._cexit>]   ;  MSVCR90D._cexit  

    ……   ……  

004119AB   JMP SHORT Test_0.004119FF  

    ……   ……  

004119B7   MOV ECX, DWORD PTR SS:[EBP-14]  

004119BA   PUSH ECX  

004119BB   MOV EDX, DWORD PTR SS:[EBP-28]  

004119BE   PUSH EDX  

004119BF   CALL Test_0.00411181  

004119C4   ADD ESP, 8  

004119C7   RETN  
```

看到这里各位读者是不是感觉逆向很简单，但也充满挑战？如果你掌握了以上三种方法，那么恭喜你，你已经成功的走出了第一步，这很有纪念意义。

[【返回到目录】：http://bbs.pediy.com/showthread.php?t=113689](http://bbs.pediy.com/showthread.php?t=113689)
