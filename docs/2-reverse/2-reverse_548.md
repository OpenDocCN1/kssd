

# <内核反汇编技术>译自 gloomy 的文章,由董岩 译

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-736.htm`](https://www.pediy.com/kssd/pediy07/pediy7-736.htm)

内核反汇编技术  
\===============================  
  
                                   Усыпляющее шептание алых просторов  
                                   Сон зовет меня и мои мечты освобождены  
                                   Оставленная мной действительность  
                                   не важно, если я не проснусь  
                                                  (c) by Anathema  
  
Windows  NT 主要是由 C 写成的，所以总的来说进程本身的反汇编不是很复杂。通常对局部变量和参数的使用是通过地址和用 EBP 形成的 stack frame 来进行的。例如：  
  
PAGE:801932D4    mov     eax, large fs:0  
PAGE:801932DA    push    ebp  
PAGE:801932DB    mov     ebp, esp ; 堆栈中的 frame  
PAGE:801932DD    push    0FFFFFFFFh  
PAGE:801932DF    push    offset $T13371  
PAGE:801932E4    push    offset \_\_except\_handler3  
PAGE:801932E9    push    eax  
PAGE:801932EA    xor     eax, eax  
PAGE:801932EC    mov     large fs:0, esp  
PAGE:801932F3    sub     esp, 64h  ; 局部变量在堆栈中的位置  
PAGE:801932F6    mov     \[ebp+Flag\], al ; 使用局部变量  
  
有时，编译器会生成更为优化的代码，直接使用 ESP 来引用堆栈。  
  
.text:80124320    sub     esp, 8  
.text:80124323    test    byte ptr ds:\_NtGlobalFlag+2, 8  
.text:8012432A    push    ebx  
.text:8012432B    push    esi  
.text:8012432C    push    edi  
.text:8012432D    push    ebp  
.text:8012432E    jnz     loc\_FFFFF000\_80124443  
.text:80124334    mov     esi, \[esp+18h+arg\_0\] ; 引用第一个参数  
  
IDA PRO 这个特别的反汇编器可以跟踪堆栈的创建，所以建立了下面例子中的相应的常量。在调用函数时，参数以相反的顺序放在堆栈中。调用函数自己要负责清理堆栈（编译器就是这样为 C 函数生成代码的）。  
  
PAGE:80193288    push    \[ebp+ExceptPort\]  
PAGE:8019328B    push    \[ebp+DebugPort\]  
PAGE:8019328E    push    \[ebp+SectionHandle\]  
PAGE:80193291    push    \[ebp+bInheritHandle\]  
PAGE:80193294    push    edx             ; ffffffff  
PAGE:80193295    push    \[ebp+ObjectAttributes\]     ;(参数 3)  
PAGE:80193298    push    \[ebp+Access\]    ;           (参数 2)  
PAGE:8019329B    push    ecx             ; Handle    (参数 1)  
PAGE:8019329C    call    \_PspCreateProcess@32  
; PspCreateProcess (Handle,Access,  
; ObjectAttributes,-1,bIbjeritHandle.SectionHandle,DebugPort,ExceptPort);  
PAGE:801932A1  
PAGE:801932A1 NtCreateProcessExit:        ; CODE XREF: \_NtCreateProcess@32+71 j  
PAGE:801932A1                             ; \_NtCreateProcess@32+80 j  
PAGE:801932A1    mov     ecx, \[ebp+var\_10\]  
PAGE:801932A4    pop     edi  
PAGE:801932A5    mov     large fs:0, ecx  
PAGE:801932AC    pop     esi  
PAGE:801932AD    pop     ebx  
PAGE:801932AE    mov     esp, ebp  
PAGE:801932B0    pop     ebp  
PAGE:801932B1    retn    20h ; 清理堆栈(返回后 ESP 加上 0x20)  
  
为了提高速度，有时在 OS 内核中会使用 fastcall 的函数，参数通过堆栈传递。例如：  
  
PAGE:8018CC9D    mov     ecx, \[ebp+pObject\] ; 第一个参数(下一个参数在 edx 中)  
PAGE:8018CCA0    call    @ObfDereferenceObject@4 ; fastcall 函数  
  
在这个例子中使用了内核中内部的非导出函数。在下一个例子中，我们使用 HAL.DLL 中的未公开的导出的 fastcall 函数：  
  
.text:801335E2    mov     ecx, eax ; OldIrql  
.text:801335E4    call    ds:\_\_imp\_@KfLowerIrql@4  
  
fastcall 函数在其名字中都有字符‘f’。  
  
Microsoft 提供了符号信息，这些符号信息可以用来调试程序。这些信息可以确定内部函数（非导出的）和全局变量的真实名称。这就简化了辨别函数和变量名称的工作，而且除此之外，通过后缀@N 可以确定函数参数的数量。  
  
调试信息是以 NT 4.0 的.DBG 文件和 Windows 2K 的.PDB 文件的形式提供的。SoftICE 和 IDA 都通晓 PDB 和 DBG 文件（IDA 使用插件来加载 ntoskrnl.pdb）。  
  
内核中的函数、变量和结构体的名称本身都能表达一些信息。前缀通常有两种意思，描述函数的特征或是用于子系统的数据。例如：Mm - 子系统内存，Cc - 缓存，Ob - 对象管理器，Ps - 进程管理，Se - 内存管理器，Ke - 内核其它的结构体，Ex - 执行体系统。如果函数是初始化用的（或其可能会转入此类函数），则在前缀的第一个字母后加一个字符‘i’。例如 Ki、Mi。使用 Fastcall 的函数在前缀后加‘f’。系统调用使用前缀 Nt。这些函数不是内核导出的函数，它们的地址记录在 service table 里。调用服务要通过软中断 0x2e。内核导出了 Zw 函数，这些函数是对中断的封装。  
  
.text:8011A49C \_ZwCreateFile@44 proc near   ; CODE XREF: \_FsRtlpOpenDev@8+4D p  
.text:8011A49C arg\_0           = byte ptr  4  
.text:8011A49C    mov     eax, 17h  
.text:8011A4A1    lea     edx, \[esp+arg\_0\]  
.text:8011A4A5    int     2Eh             ; 中断处理程序调用 NtCreateFile  
.text:8011A4A7    retn    2Ch  
.text:8011A4A7 \_ZwCreateFile@44 endp  
  
我不知道字符 Zw 是什么意思（好像只有内核的设计者知道）。可能是 Zero Wheel（或是零环）的意思，因为 Zw 函数是从内核模式调用的（在 DDK 中描述了一些）。从用户模式下，系统服务是通过 NTDLL（实现于用户模式）调用的，NTDLL 导出了 NtXXX 的封装函数 ZwXXX。  
  
内核中的名字主要要遵循一种描述规则。当然，名字本身承载着意义。名字的内容通常是行为及其对象，即对对象进行某种行为。如：ObCreateObject。  
  
许多的操作系统函数都仅仅是对内核内部函数的封装。例如，NtCreateSection 调用了 MmCreateSection，用的参数也都相同。现在，如果统计一下的话，许多 Nt 函数的原型都是 Windows NT 内核研究者所熟知的，许多内部函数的原型也就可以不用通过逆向工程而获得。有了 C 语言函数的原型再学习其结构和思想就轻松多了。  
  
理论上讲，取得关于内核的信息的更简单的方法不是反汇编内核的映象，而是其它的代码。例如，使用 WinDbg 的 kernel-mode extensions 的代码。WinDbg 的扩展中有额外的命令，扩展的调试命令集。其中有明显使用内核内部结构的命令，或是能减轻分析内核内部结构工作的命令。例如命令!ca、!tokenfields、!processfields 等等。反汇编 kdextx86.dll 和 kdex2x86.dll 的代码可以得到某些结构的信息。  
  
内核调试扩展是个.DLL。导出扩展的命令所用的名字与在 WinDbg 调试器中遇到的名字是相同的。例如，processfields。扩展的 DLL 导出了函数 WinDbgExtensionDllInit，这个函数是在加载扩展后从 WinDbg 调试器中调用的。函数的原型如下：  
  
VOID WinDbgExtensionDllInit(PWINDBG\_EXTENSION\_APIS lpExtensionApis,  
                                 USHORT MajorVersion,  
                                 USHORT MinorVersion)  
  
第一个参数是指向在.DLL 中使用的 API 的指针。WINDBG\_EXTENSION\_APIS 结构体包含以下成员，这些成员定义了访问扩展函数集：  
  
lpOutputRoutine               - 在控制台输出字符串  
lpGetExpressionRoutine        - 计算表达式的值  
lpGetSymbolRoutine            - 取得符号在内存中的地址  
lpDisasmRoutine               - 反汇编内存  
lpCheckControlCRoutine        - 检查是否按下 CTRL-C (未实现)  
lpReadProcessMemoryRoutine    - 读进程内存，带有对 GPF 的保护。  
lpWriteProcessMemoryRoutine   - 写内存  
lpGetThreadContextRoutine     - 取得进程寄存器的值  
lpSetThreadContextRoutine     - 设置寄存器  
lpIoctlRoutine                - 未实现  
lpStackTraceRoutine           - 跟踪堆栈  
  
这样.DLL 导出了对应于扩展命令的函数，并可以用于与调试器的有限而熟悉的函数集的协同工作。进一步给出实现扩展命令的函数的原型。  
  
#define DECLARE\_API32(s)                                \\  
         CPPMOD VOID                                    \\  
         s(                                             \\  
             HANDLE                 hCurrentProcess,    \\  
             HANDLE                 hCurrentThread,     \\  
             ULONG                  dwCurrentPc,        \\  
             ULONG                  dwProcessor,        \\  
             PCSTR                  args                \\  
          )  
  
有趣的是参数 args，它指向 WinDbg 中命令的字符串。借助反汇编可以取得足够的信息以研究扩展命令的工作逻辑。在首要的研究中可以选出直接操纵内核结构体和能辨别结构体成员的命令。例如，命令!ca 的代码说明了内核结构体 control area 和 segment。这个命令的逻辑并不复杂：辨别命令行，从内核内存中读取所要的结构体，输出域中的内容。  
  
但是，扩展命令经常并不会列出内核结构体的所有内容。并且，从名字中并不总是能明确的推断出域的含义，但是反汇编这条命令可以简化对内核内部函数的分析工作。在任何情况下都会有机会对信息做对比，从各种各样的线索中取得信息。  
  
\---------------------------------------------------------------------------  
                         (c)Gloomy aka Peter Kosyh, Melancholy Coding'2001  
  
                                                       http://gloomy.cjb.net  
                                                       mailto:gl00my@mail.ru  
                                                                     董岩 译  
                                                http://greatdong.blog.edu.cn   
\[COLOR=red\]我已经明确指出原作者和翻译者了，如果你要转贴请注意版权。  
谢谢合作！\[/COLOR=red\] 

 4dd466d0e1e0cc123afbe504da2462ca