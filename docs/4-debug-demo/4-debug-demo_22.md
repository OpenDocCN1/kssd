
# 初学者系列教程#8

**By:** Shub-Nigurrath /ARTeam
**Source:** [`www.pediy.com/kssd/pediy12/23006.html`](https://www.pediy.com/kssd/pediy12/23006.html)

**断点理论 v1.41**

*   **Target:** 无
*   **工具:** `Ollydbg 1.10`
*   **保护方式:** 无
*   **其它信息:** 这篇教程将会涉及一些断点的基本知识以及这些断点间的区别，从而有助于涉及到这方面知识的初学者。最佳浏览方式：`Firefox` 分辨率：`1280x1024`

## 简介

从初学者系列教程(1) 开始，你会发现有许多不同的断点且它们的使用方法也不同，但是还没有人告诉你至少直到现在(这个系列教程中当然会告诉你)我们设置的各种不同断点究竟有什么区别。尽管你对断点的底层机制所知甚少，但是弄清楚为什么每个不同断点使用方法不同也足够啦!首先，断点之间有非常大的区别，有硬件断点和软件断点之分...

### 硬件断点

后者即硬件断点，由 `CPU` 直接支持，它使用了一些专用寄存器，称为调试寄存器。有四个寄存器：`DR0`, `DR1`, `DR2`, `DR3`. 它们存储着四个断点的线性地址。这些断点的每一中断条件都保存在在一个特殊 `DR7` 寄存器中。当所有条件为真时，处理器会产生一个 `INT 1` 而且控制权转交到调试器。有四个可被 `CPU` 预见的中断类型：

1.  一条指令被执行；
2.  内存位置的内容被修改；
3.  内存位置被读取或者更新，但是没有被执行；
4.  输入输出端口被使用。

一旦你设置了一个硬件断点，调试器会标志寄存器的陷阱位是否被设置。如果设置过，一个 `INT 1` 调试异常将会由 `CPU` 自动产生并且控制权被转交到调试器，或者通常所说的系统中异常处理单元

## 思考

综上所述，你可以看到你最多可以设置四个硬件断点且可以设置的条件的数量受到当初设计的限制。这样带来的好处就是软件几乎检测不到硬件断点的存在，它们所能使用的去检测硬件断点是否设置唯一方法是读取 `DR0..DR7` 的值：标志寄存器(`DR7`)这个值可以检测追踪(调试)不幸的是，这些对寄存器的修改只能在 `ring0` 级别下进行。对于 `DR7` 标志的意义及通常调试寄存器所， 请读“IA-32 Intel Architecture Software Developer’s Manual, 3 卷 15 章的“调试寄存器”一节，可以在 `http://developer.intel.com/design/pentium4/manuals/253668.htm` 得到。 由此看来，我们可以使用一些手段转换到 `ring0` 级别。例如，我们从 `Pavol Cerven` 的书"Crackproof your Software" (我建议大家阅读的经典)

## 选取的一个例子

```asm
.386
.MODEL FLAT,STDCALL locals

jumps

UNICODE=0

include w32.inc

Extrn SetUnhandledExceptionFilter : PROC

Interrupt equ 5                           ;中断号 1 或者 3 将会使得
                                          ;调试更加困难

.DATA

message1  db "Debug breakpoint detection",0
message2  db "Debug breakpoint not found",0
message3  db "Debug breakpoint found",0

delayESP  dd 0                            ; ESP 寄存器保存到这里
previous  dd 0                            ; ESP 将会保存以前
                                          ; 系统异常处理的地址

.CODE

Start:

;????????????????????????????????????????????????????????????????????????????????????????
;Sets SEH in case of an error
;????????????????????????????????????????????????????????????????????????????????????????

      mov [delayESP], esp
      push offset error
      call SetUnhandledExceptionFilter
      mov [previous], eax

;????????????????????????????????????????????????????????????????????????????????????????

      push edx
      sidt [delayesp?2]                   ;读 IDT 到栈
      pop edx
      add edx, (Interrupt*8)+4            ;读取请求中断向量
      mov ebx,[edx]
      mov bx,word ptr [edx?4]            ;读取请求中断的 reads the address of the old service of the
                                          ;required interrupt
      lea edi,InterruptHandler
      mov [edx?4],di
      ror edi,16                          ;sets the new interrupt service
      mov [edx+2],di
      push ds                             ;saves registers for security
      push es
      int Interrupt                       ;跳到 Ring0 (一个新的 INT 5h 服务)
      pop es                              ;保存寄存器
      pop ds
      mov [edx?4],bx                     ;设置原来 INT 5h 中断服务
      ror ebx,16
      mov [edx+2],bx
      push eax                            ;保存返回值

;????????????????????????????????????????????????????????????????????????????????????????
;Sets the previous SEH service
;????????????????????????????????????????????????????????????????????????????????????????

      push dword ptr [previous]
      call SetUnhandledExceptionFilter

;????????????????????????????????????????????????????????????????????????????????????????

      pop eax                             ;还原返回值
      test eax,eax                        ;测试 eax=0 吗？
      jnz jump                            ;如果不是, 程序发现被调试
                                          ;断点并且结束

continue:

      call MessageBoxA,0, offset message2,\
      offset message1,0
      call ExitProcess, ?1

jump:

      call MessageBoxA,0, offset message3,\
      offset message1,0
      call ExitProcess, ?1

error:                                    ;如果错误就设置一个新的 SEH 服务
      mov esp, [delayESP]
      push offset continue
      ret

;????????????????????????????????????????????????????????????????????????????????????????
;Your new service INT 5h (runs in Ring0)
;????????????????????????????????????????????????????????????????????????????????????????

InterruptHandler:

      mov eax, dr0                        ;从 DR0 调试寄存器读取一个值
      test ax,ax                          ;测试设置了调试断点没有
      jnz Debug_Breakpoint                ;设置的话,程序跳

      mov eax,dr1                         ;从 DR1 调试寄存器读取一个值
      test ax,ax                          ;测试设置了调试断点没有
      jnz Debug_Breakpoint                ;设置的话,程序跳

      mov eax,dr2                         ;从 DR2 调试寄存器读取一个值
      test ax,ax                          ;测试设置了调试断点没有
      jnz Debug_Breakpoint                ;设置的话,程序跳

      mov eax,dr3                         ;从 DR3 调试寄存器读取一个值
      test ax,ax                          ;测试设置了调试断点没有
      jnz Debug_Breakpoint                ;设置的话,程序跳

      iretd                               ;如果断点没有被设置，
                                          ;程序会向 EAX 返回 0

Debug_Breakpoint:

      mov eax,1                           ;EAX=1 说明断点被激活
      iretd                               ;跳到 Ring3

ends
end Start
```

这一方法是发现调试断点是否存在的方法之一，这就使得不用暂停程序就可以删除调试断点称为可能。 然而，程序不是删除它而是出错。不幸的是，这一伎俩(其它类似的方法) 只能在 `windows 9x` 系统下因为它需要切换到 `ring 0`。通常来说，`Cerven` 的著作提到了把一般应用程序从 `ring 3` 转到 `ring 0` 的三种方法，但只能在 `windows 9x` 下实现。`windows NT`，`2000` 和 `XP` 系统为了防止病毒利用而对此作了防范(旧版本的 `windows NT` 允许这种切换，但是因为被错误的使用过，系统去除了这种可能)

--------------------------------------------------------------------------------

## 进阶：转换调试寄存器到 ring 3

的代码试验表明：通过改变调试寄存器的值来切到 `ring 0` 是不完全正确的。程序以调试器运行，可以调用系统的 API，例如 `GetThreadContext()`和 `SetThreadContext()`等。 这些 API 被 `NTDLL.DLL` 执行产生系统调用(`2E` 中断)，处理器转到 `ring 0` 去执行代码。你也可以使用下面的 `ASM` 代码(包括在本文中)自己作试验，它使用了系统异常处理机制去查处调试寄存器的值，然后转到正常代码执行。以下面的代码为例，(现代人认为是 `Neitsa` 写的)。结构非常简单， 例中试着在 `NOP` 后置一个硬件断点然后在系统异常处理单元的第一条指令处设置一个断点，看看发生了什么

```asm
.686
.model flat, stdcall ;32 bit memory model
option casemap :none ;case sensitive
assume fs:nothing ;MASM feature (otherwise FS assumed to be ERROR)
include EraseDrx.Inc
.code
start:
      ; ### set the S.E.H ###
      push offset mySEH
      push dword ptr fs:[0]
      mov dword ptr fs:[0],esp
      ;**** now everything will be covered by our SEH ****
      ; raise an invalid opcode exception
      UD2
@@SafeOffset: ; this is where we can safely return from our SEH
      fnop
      ;try to hardware BP one of those NOP
      nop
      nop
      nop
      nop
      ;**** now this is this end of the SEH ****
      pop dword ptr fs:[0]
      add esp,4
      ret ;return to ExitThread

;????????????????????????????????????????????????????????????????????????????????????????
; OUR SEH handler, which erases the debug registers
;????????????????????????????????????????????????????????????????????????????????????????

mySEH proc C lpExcept:DWORD, lpFrame:DWORD, lpContext:DWORD, lpDispatch:DWORD
      mov ecx,[lpContext]
      ; push all linear addresses of drx (from Dr0 to Dr3)
      ; you should see your hardware BP there (just to demonstrate where they are)
      push [ecx][CONTEXT.iDr0]
      push [ecx][CONTEXT.iDr1]
      push [ecx][CONTEXT.iDr2]
      push [ecx][CONTEXT.iDr3]
      add esp,4*4 ; skip them
      ;erase DR0 to DR3
      push 0
      push 0
      push 0
      push 0
      pop [ecx][CONTEXT.iDr0]
      pop [ecx][CONTEXT.iDr1]
      pop [ecx][CONTEXT.iDr2]
      pop [ecx][CONTEXT.iDr3]
      ;erase also DR7
      push 0
      pop [ecx][CONTEXT.iDr7]
```



;now set EIP to our SafeOffset  

`push offset @@SafeOffset`  

`pop [ecx][CONTEXT.regEip]`  

`mov eax,FALSE`  

`ret`  

`mySEH endp`  

`end start`  

--------------------------------------------------------------------------------  

对这一节作个总结：理论上要使调试器不被检测到需要识别出读取标志寄存器的指令，后模拟代码运行然后总给陷阱标志位返回 0，事实上不是那么容易做的。在 `Olly` 中使用硬件断点问题出现了， `Olly` 中什么时机去使用硬件断点较好？再读了下面关于软件断点的一节答案就比较显然啦！不管怎么说硬件断点通常可以在下列两种情况下使用：  

(1)被调试程序检查自己是否被作了修改(自校验)被调试程序检查是否有调试器且擦除自己代码上发现的断点(也包括 `self-checking anti-tampering` 技术)通常是遇到压缩工具或者像 `Armadillo`，`Execrypror` 或者 `Asprotect` 等复杂的壳才发生。  

使用硬件断点第二个好处就是 `CPU` 没有关联某个 `Olly` 例程， 所以，我们可以在一个 `Olly` 中在某一特定内存地址设置硬件断点然后再用 `Olly` 开同一个程序同样可以断在第一个 `Olly` 设置的那个硬件断点的地址上。好，看起来庭复杂，但是这样做确实非常有用：假设你有一个加壳程序，加的是 `Asprotect`。你可以开一个 `Olly` 例程(用 `HideDebugger` 插件隐去调试标志)然后运行程序。假设在手动脱壳时你需要在某一内存地址中断去看发生了什么(事实上是手脱 `Asprotect` 的步骤)或者去改变寄存器的值。为了不干扰 `Adprotect` 你最有可能在某一地址设置一个硬件断点。一旦硬件断点被设置，最小化第一个 `Olly` (不用关闭)再用 `Olly` 打开另一个这一程序的例程让它运行。 发生了什么？是不是第二个 `Olly` 中断在了前一个 `Olly` 设置的硬件断点上(当然是重定位表的一段)发生上述行为是因为硬件断点直接由 `CPU` 处理且 `CPU` 产生的调试事件，它和某一个进程没关系，但是仅限于一些特殊内存地址。真的关于硬件断点的话题已经没有更多可说的垃。让我们来看更复杂的软件断点吧…………  

 这些方法也是对付一些如 `ActiveMark` 之类的壳的招  

软件断点：  

软件断点是那一类不写一个全功能处理模拟器就不能完全隐藏的断点类型，如果你在一条指令的开始放上一字节代码－－`0xCC`，，当系统试图执行它的时候将会产生一个 `INT 3` 异常。`INT 0x3` 处理单元获得了控制权就可以对程序为所欲为。 然而， 在中断处理单元被调用之前，标志寄存器的当前值，代码段的指针(`CS` 寄存器)和指令指针(`IP` 寄存器)被压入了堆栈，另外，中断被禁止(`IF` 标志清零)和陷阱标志位清零。因此，调试中断的调用和其它中断的调用没有什么差别。为了得到程序在哪里中断，调试器弹出寄存器里保存的置到堆栈，同时 `CS` 的 `IP` 指针指向下一条将被执行的指令。所以总的来说，在程序的任意地方设置一个断点是非常复杂的。调试器需要保存内存堆栈的值到一个特殊的地址，然后写上 `0xCC`。在出现调试中断之前调试器恢复一切，修改保存在堆栈的指令指针使得它指向恢复指令的开始(否则，它指向中间)  

    
  

思考  

8086 处理器的断点机制有什么缺陷？最令人恼火的是设置断点时调试器必须直接修改代码。显然修改内存的过程(写入 `0XCC`)程序可以很容易检测到并且跳到其它的地方以避免修改。许多教程或多或少介绍了简洁的避免程序被下断的方法。通用的做法是修改程序执行流程或者简单的把 `OXCC` 改回原来的代码(调试器将不会断下)。程序发现自己被调试的可行做法是查找是否至少有一处已经别修改，去计算它的校验和，可能会用到 `MOV`, `MOVS`, `LODS`, `POP`, `CMP`, `CMPS`, 或者其它指令。例如，让我们看看下面例子的保护机制(来自 `Karsperky Book, Haker Disassembling Uncovered` 黑客解密大曝光)  

```
int main(int argc, char* argv[])

{

    // The ciphered string "Hello, Free World!"

    char s0[]="\x0C\x21\x28\x28\x2B\x68\x64\x02\x36\x

              \x21\x21\x64\x13\x2B\x36\x28\x20\x65\x49\x4E";

    __asm

    {

        BeginCode:                ; The beginning of the code being debugged

        pusha                     ; All general-purpose registers are saved.

        lea ebx, s0               ; ebx=&s0[0]

        GetNextChar:             ; do

        xor eax, eax              ; eax = 0;

        lea esi, BeginCode        ; esi = &BeginCode

        lea ecx, EndCode          ; The length of code

        sub ecx, esi              ; being debugged is computed.

        HarvestCRC:              ; do

        lodsb                     ; The next byte is loaded into al.

        add eax, eax              ; The checksum is computed.

        loop HarvestCRC           ; until(--cx>0)

        xor [ebx], ah             ; The next character is decrypted.

        inc ebx                   ; A pointer to the next character

        cmp [ebx], 0              ; Until the end of the string

        jnz GetNextChar           ; Continue decryption

        popa                      ; All registers are restored.

        EndCode:                ; The end of the code being debugged

            nop                   ; A breakpoint is safe here.

    }

    printf(s0);                   //The string is diplayed.

    return 0;

}
```  

程序正常启动之后， 屏幕将会显示"Hello, Free World!" 。但当程序在调试器下运行， 即使在开始或结束设置一个断点，屏幕将会显示像"Jgnnm."Dpgg"Umpnf#0"这样的无意义的垃圾文字。如果把程序计算出的校验和放到另外一个有用进程中的独立线程保护效果将会大为加强，使得保护机制变得尽可能的强。上面的代码使用了你可能已经忘记的异或运算，`A <XOR> B <XOR> A = B`.这就是为什么它会经常被用来作弱的数据加密。如果你用密钥异或一段明文，你会得到密文。如果你用密文异或密钥，你就会重新得到明文。如果你得到明文和密文，你就能得到密钥。如上所说，密钥可以在开始和结束代码之间得到，`Olly` 中代码如下(`main.exe` 本文有收录)  

你还可以发现是用 `C` 语言写的。自己试着练一下，不难发现现实世界中这种弱保护机制是经常见的…………如果到了这一步，当你碰到它时你就会认出它…………当然上述例子比较简单，如果代码再使用异常，线程和其它措施来耗损我们的生命的话，那就跟随执行的汇编代码一样…………对于那些读了 `Shub-Nigurrath Oraculum's tutorial` 和 `ARTeam's Oraculum Tutorial` 的你应该想到 "EBFE"的使用和这里的内存断点的描述有些相似。这和为了想在想要的地方中断而往内存中写值的原理是相似的，不同之处是这里没有异常且使程序挂起只是看 `EIP` 是否是常数这将会击破使用异常检测和 `OXCC` 值是否出现的保护 ，不适用于那些在内存中使用复杂校验的出现。  

参考书目:  

正如你所看到的，断点的不同使用源于它们的实现不同，但是想想如果没有一个能被检测到，别这样想，一旦断点被设置程序在此中断。程序也能够检测到断点的存在且删除它或者修改它自身来阻止调试。进一步的学习请参考下面..  

`Kris Karspersky, Hacking Disassembling Uncovered, a-List Press`  

`Pavol Cerven, Crackproof your software, No Starch Press`  

`Shub-Nigurrath, Oraculum Tutorial With Framework Src V11, ARTeam`  

`Shub-Nigurrath, Gabri3l, Serial Fishing And Oraculum For Weblink, ARTeam`  

`Gabri3l, Writing A Loader 4 Softwrap 6.1.1, ARTeam`  

`IA-32 Intel Architecture Software Developer’s Manual, Volume 3, Intel, Section “Debug Registers”, Chapter 15,`  

`http://developer.intel.com/design/pentium4/manuals/253668.htm`  

and essentially all the tutorials seens around (also others on our tutorials page) which always make use of breakpoints..  

最后:感谢所有成员:  

`[Nilrem] [JDog45] [Shub - Nigurrath] [MaDMAn_H3rCuL3s] [Ferrari] [Kruger] [Teerayoot] [R@dier] [ThunderPwr] [Eggi] [EJ12N]`  

`[Stickman 373] [Bone Enterprise]`  

Thanks to all the people who take time to write tutorials. 

Thanks to all the people who continue to develop better tools. 

Thanks to Exetools, Woodmann, SND, TSRH, MP2K and all the others for being a great place of learning.  

Thanks also to The Codebreakers Journal, and the Anticrack forum.  

If you have any suggestions, comments or corrections contact me in usual places..  

翻译不到的地方，多多指教  

上传的附件  

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || beginner_olly_tutorial_part.rar  
