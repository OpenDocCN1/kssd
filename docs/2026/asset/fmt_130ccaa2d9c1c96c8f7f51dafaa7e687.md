
# [翻译]利用代码注入脱壳

原文链接：<http://bbs.pediy.com/showthread.php?s=&threadid=4541>

来自 声声慢 提供的英文原稿

Author: E. Labir

(作者：E. Labir)

翻译：springkang[DFCG] [Nuke Group][TT]

**摘要**

本文演示如何通过代码注入获取给定目标的内部信息。我们的攻击对当前多数反破解技术来说是完全秘密的，展示了现实生活中的威胁。我们可以重新获取如下的最相关信息：

- 异常、句柄和有关信息列表
- 调用所有DLL文件(参数、返回代码…)的API列表
- 输入表的彻底重建
- 入口点

我们的方法具有灵活性，操作起来并不困难。我们给出源码并提供一个关于如何分析日志文件的真实例子。

**关键词**：脱壳，代码注入，反反破解技术

## 一、介绍

调试的目标从极容易到极难不等。保护良好的软件常用主要基于SEH(结构化异常处理)的反调试手段和极混乱的代码，这些使得破掉它成为一个无尽的梦魇。在Windows(win9x及后续版)，进程生存在自己的地址空间里。地址空间是平坦的，包含全部(映射的)DLL文件、资源、和它所需的原料。Windows也提供我们注入自己的代码并使它运行在另一进程里所需的工具。[4]本文使用注入代码来嗅探来自目标的大量信息。我们的方法绕过API重定向和所有的标准反调试。

我们从注入代码获取的信息包括对映射到目标地址空间的所有DLL文件的所有调用(还有参数、返回代码)―--我们也能够修改它们。代码注入还特别提供了异常及与之有关的信息(句柄、地址、代码…)列表。有了它，我们可以轻易的重新获取入口点(注入一个追踪(tracer))，甚至在某些情况下，获取被偷的字节。即便这种方法失败了，所有的这些问题也会大大简化。

这种方法需要“某种”人为干预去分析日志文件，但它节省了大量工作。检测你正在受到攻击并非易事，许多壳因此需要重新设计。

我们解决的主题包括：

1. (第一节)壳的描述
2. (第二节)如何注入代码并使之运行在目标中
3. (第三节)挂钩所有的API调用
4. (第四节)与API调用(记录日志、IAT重建)互动
5. (第五节)用注入追踪找到入口点
6. (第六节)注入代码是怎样隐匿的
7. (第七节)结束，深入研究…

第一节都有代码片段(或伪码)，我们也展示了在实验中出现的例子

## 二、壳的描述

壳是旨在防止别人检测程序是如何运行或修改程序的程序。典型地来讲，受保护的程序的入口点转向先运行壳。当壳运行时，它执行以下一些步骤：

- 解密目标的区块(目标=被加壳程序)
- 重建输入表
- 跳转到目标的实际入口点

第一步区块加密对我们并不重要。通常，在它们在内存中解密之前，加密区块并不是要解决的问题，只要简单地把它们抓取(dump)到某个文件，并组合在一起。但第二、三点很重要。

可以用多种复杂方法(several degrees of sophistication,)完成重要的保护。(现实中的一些壳的)一些保护十分脆弱，根本不需我们的方法就可破了它们。我们一贯支持强大的保护应用IAT，即：

1. 删除输入表，壳仅(在安全的地方)将API的名字和地址的杂乱信息保存到IAT。
2. 算法良好的混乱，并有很多反调试，反跟踪…
3. 壳并不使用`GetProcAddress`。相反，它执行自己的算法，寻找DLL输出表里的API。
4. IAT已经重定向(阅读下面所述)

注意：事实上，第三点对我们没有影响。然而，一个良好的保护应该这么做。

看到这么多的“商业”产品不符合第一点或第三点，真是很惊讶。

第四点(API重定向)如何呢？我们看看“API 重定向”是什么意思：打开任意一程序，它肯定输入`kernel32.ExitProcess`。现在，看看对它的一个调用(call)，你会发现类似下面的东西：

```
call [XXXXXXXXh]      ; call to
                                 ;kernel32.ExitProcess
                                 ; XXXXXXXXh inside the IAT...

XXXXXXXXh: YYYYYYYYh             ; address of
                                 ; Kernel32.ExitProcess
```

另一很平常的可能是：为调用`call xxxxxxxxh`，参数也是一样的。Windows 加载器(loader)“看见”输入表，并用重要的APIs地址填充IAT。壳破坏被保护程序的输入表的信息，因此当壳把控制权交给目标时，IAT就得到错误的值。于是，结果就是壳需要用正确的值填充目标的IAT。

如果你用过一加壳程序，壳符合第四点，例如：Asprotect,Slovak Protector,…IAT看起来像：

```
XXXXXXXXh: ZZZZZZZZh       ; ZZZZZZZZh is inside a
                                           ; buffer dynamically
                                           ; allocated by
                                           ; the packer, so it will
                                           ; not exist if you
                                           ; remove the packer.

ZZZZZZZZh:  push ebp           ; (\*)
            ror eax, 16h
            pushf
            popf
            mov ebp, esp       ; (\*)
            call @@1
            db 68h
   @@1:
            add eax, 134h
            ....
            jmp ACTUAL\_ENTRY\_POINT\_OF\_API + k
```

壳将API的开始指令与一些垃圾代码(在例子中，除在原始API代码里的以外，有\*的指令)混合在一起，它也能包含一些微小的反调试手段，最终，写个跳转(或其他类似的)到位于API+K的入口点处。而K就是已经运行在垃圾代码里的原始指令数。这样，要找到通过调用zzzzzzzzh的实际API或对API挂钩基本上不可能。

输入表不是一个直观明了的结构，了解它的细节超出了本文的范围[2]。

一些叫作输入表重建器的程序能够模拟一些指令去寻找我们跳转到的DLL中的地址。然后，你就能够从输出表中重新获得API名字。然而，输入表重建工具通常十分局限于它们能嗅到的东西，并不能搞定最新的壳(很明显，这些壳在发布前对重建工具进行了测试)。

我们来看看入口点发生了什么。如我们所说，壳一旦完成工作(对目标解密，重建IAT…)，必须把控制权交给被保护的程序。使用`Kernel32.CreateThread`，或以某种特别的方法，简单地跳到那里，从而把它作为一个新线程，就可以解决。运行在一个新的线程并不是个好主意。线程的开始地址太明显，于是我们(再次)假设最坏的情况：经过很长时间的混乱和保护良好的代码后，壳跳到入口点处，跳转十分隐蔽(自己修改的代码等等…)。

另一通常要解决的问题是“被偷代码”：壳使用一套预定义的APIs，非常普通的一个是：`GetModuleHandleA`，并做如下事项：

调用`GetModuleHandleA`，存储返回

查看被保护的程序，模式是：

```
call XXXXXXXXh             ; call to
                            ; kernel32.GetModuleHandleA
```

现在，删除这个调用，并在每个起始处，用简单的`mov [handle], harcoded_value`替换`mov [handle], eax`而`harcoded_value`之前已经被壳对`GetModuleHandleA`的调用返回了。

如果破解者不去检查这个把戏，那么脱壳后的程序就不能在所有操作系统上运行或有其他的缺陷。壳删除的这些字节就是“被偷代码”。这种被偷字节，也就是用一个固定编码(hardcoded)的值替换一个调用，用我们的方法将很轻松的定位并重建。

对壳的完整描述大大超出本文的范围，请参考[1]获得详细的解释。

壳是保护中等价位程序的优先方式，实践上，所有的共享软件都依赖于他们的安全。所以，研究他们的利弊成为REC(代码逆向工程)的重要研究领域。

## 三、注入并运行你的代码

本节中，我们将大致描述如何注入代码并使之运行在目标的地址空间里。查看[4]获取完整的描述。Win9x系统请看[3]。

工程将分为两部分，参见[4]。我们还需要一个载体程序，负责加载目标程序和注入代码。我们把注入代码称为“自动记录器”(logger)，因为它十分精确地描述它所做的一切。

载体需要在目标程序有机会运行前注入并运行我们的记录器。因此，我们需要创建目标程序为`CREATE_SUSPENDED`。这样，目标程序的地址空间将被初始化，但主线程要等到我们调用`ResumeThread`后才运行。

结果，载体创建目标程序为`CREATE_SUSPENDED`，并把记录器注入到它的地址空间里。下一步，载体用`CreateRemoteThread`(Win2k以上系统)运行注入的代码。记录器在目标程序被允许运行前，要做一些基础工作。当一切准备好后，载体运行目标程序的主线程。因此，载体和记录器需要以某种方式通信，我们选择的是通过一个事件，当目标程序(壳)能够运行时，记录器设置事件为真。

我们继续概述如何执行以上的所有步骤(参见win32.hlp)，获得下面的APIs的详细参考。

```
; first, we create the event (choose a random name for your event)
；首选，我们创建一个事件(为该事件随意选个名字)

push offset zsEventName       ; name of event
push FALSE                       ; initial status = FALSE
...
call CreateEventA

; Now, we create the target as CREATE\_SUSPENDED.
; The call returns a handle to the created process we need for later.
；现在，我们创建目标程序为CREATE\_SUSPENDED
；该调用返回一个句柄给创建的进程(我们后面需要)

...
push CREATE\_SUSPENDED
...
push offset zsTarget
call CreateProcessA

; The address space of the process has been initialised, but its
; primary thread is suspended. Now, we allocate some memory into
; the target to host our code.
；进程的地址空间已经初始化，但主线程仍挂起。现在，我们分配一些内存给目标程序，用作给我们的代码的宿主。

push PAGE\_EXECUTE\_READWRITE ; attributes for the allocated
；memory
...
call VirtualAllocEx

; The return is the image base of the allocated memory. Finally, we can write to it, with
; kernel32.WriteProcessMemory, and run our code with kernel32.CreateRemoteThread.
；返回的是分配内存的映像基址。最后，我们可以在里面写入kernel32.WriteProcessMemory，并用kernel32.CreateRemoteThread运行我们的代码。

...
call WriteProcessMemory
...
call CreateRemoteThread

; At this point, the logger is running while the main thread is suspended.
; The logger will change the status of the event
; we have created at some moment, we need to wait until then before to
; resume the main thread:
；这样，记录器在主线程挂起的时候就运行了。记录器会改变我们在某个时刻创建的事件的状态，直到恢复主线程运行之前，我们要做的是等待。

push -1                     ; wait infinite time
push dword ptr [hEvent]    ; handle to the event, returned by
                           ；CreateEventA
call WaitForSingleObject

...
call ResumeThread

push 0
call ExitProcess
```

如你所见，注入代码到另一个进程里并运行它并不太难。注意目标程序没有被调试，因此，它(目标程序)就也没有什么好抱怨的了。

## 四、挂钩API调用

### A、问题及相关信息

如我们上面简短的评述，壳会模拟API开始的K个指令并跳到(K+1)-th处。前K个指令可以变形，如K=2我们就有：

```
；KERNEL32.DLL里的kernel32.ExitProcess的未变入口点

77E55CB5 kernel32.ExitProcess    push ebp

77E55CB6                             mov ebp,esp

77E55CB8                             push -1
```

你可在缓冲区(buffer)找到的指令的样本：

```
     xchg eax, esp
     sub eax, 4
     jmp @@1
     db 68h
@@1:  xchg eax, esp
     mov dword ptr [esp], ebp
     push esp
     pop ebp
     push (77E55CB8+RANDOM\_VALUE)
     sub dword ptr [esp], RANDOM\_VALUE
     ret
```

两者是一样的，然而第二种不容易运用。缓冲区越长越难模拟，重建输入表就越困难。

现在，请观察，如果你在`kernel32.ExitProcess`的入口点设了断点，会很轻易的被壳绕过。当然，可以在以后设置该断点，但那时会知道调用参数是很困难的(几乎不可能)。此外，当别的DLL内部调用该断点地方时，断点也会突然生效的(be fired up)。

让在APIs(合适的)地方设置断点成为可能是一个很重要的问题。控制壳的行为区域就等于立刻控制了它们。

### B、步骤

假设`kernel32.ExitProcess`的API入口点如下：

```
nop                    ; 1
nop                   ; 2
...                    ; ...
nop                   ; k
...                       ; ...
nop                    ;
jmp kernel32.ExitProcess\_ActualStart
```

壳会模拟前K个nops指令，然后跳到k+1。于是，我们就能在`jmp kernel32.ExitProcess_ActualStart`安全地设置断点。事实上，初始参数已经被保存起来了。

对于给定的DLL(如kernel32)，我们要做的如下：

1. 获取kernel32的映像基址
2. 从PE-header得到(take) `SizeOfImage`
3. 将对整个kernel32映像的权限更改到`PAGE_EXECUTE_WRITECOPY`(Change permissions over the whole image of kernel32 to `PAGE_EXECUTE_WRITECOPY`)
4. 保存DLL中的所有输出函数的原始入口点，可以在`IMAGE_EXPORT_DIRECTORY.ED_AddressOfFunctions`里找到它们。
5. `N`= 由kernel32输出的API的总数，位于`IMAGE_EXPORT_DIRECTORY.ED_NumberOfFunctions`。
6. 将kernel32的每一个API转移到我们的缓冲区。
7. 存储对DLL的权限(一些壳用DLL激活异常)

我们来详细看看如何将所有的APIs重定向到缓冲区，注意要在壳运行前完成。

我们称我们使用的缓冲区为`DivBuffer`。该缓冲区大小为`N*M`,M是所有缓冲区中的最大值。我们要做的如下：

```
for (i=0; i<N; ++i){

Change the entry point
of the i-th API to DivBuffer[i\*M].

Generate random garbage
and write it to DivBuffer[i\*M].

Write some instructions, after the
garbage, to save the value of i.

Write a jump, after the previous
instructions, to our hooker procedure.

将i-th的API的入口点改变到DivBuffer[i\*M].
```
