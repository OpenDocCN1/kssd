
# 《加密与解密 第 3 版》SMC 的一个令人费解的 bug!!

> 原文：[`www.pediy.com/kssd/pediy10/70444.html`](https://www.pediy.com/kssd/pediy10/70444.html)

该书的“软件保护技术--smc 技术实现”，即第 14.2.2 节(p389 页) 代码 c 部分有一处：

`mov [sConext].iDr7,11h`

就这句代码，书中的解释是反跟踪，害我三四个小时没能看懂。来回查资料。

只有请 kanxue 老大亲自出马来解释。

我反复想了半天，估计这是一个 bug

首先，从 osmc.asm 源码中将这句话的前后摘出来

引用:      ;-=-=反跟踪代码=-=-

```
      pushad
      mov    edi,offset sContext
      mov    ecx,sizeof CONTEXT
      xor    eax,eax
      cld
      rep    stosb
      invoke GetCurrentThread
      mov    edi,eax
      mov    [sContext].ContextFlags,CONTEXT_ALL
      invoke GetThreadContext,edi,addr sContext
      mov    [sContext].iDr7,11h    ;LOCAL_EXACT_BPM_ENABLED OR DR0_ENABLED
      invoke SetThreadContext,edi,addr sContext
      popad
```

有三处不懂：

第一：在光盘 osmc.asm 中注释为 LOCAL_EXACT_BPM_ENABLED OR DR0_ENABLED 但我查了一下宏定义，这两个值应为 101h，而不是 011h

第二: 这段代码，在书中的解释是“在循环中也加入了另一种清除调试寄存器的方法（代码 C 部分），来防止动态跟踪。这样对 F7 键进行跟踪的办法也进行了有效的防护。”

-- 不懂啊！！ dr7=11h 或 dr7=101h，这不是说允许 dr0 断点吗？反断点的目的与这句话不符啊！

第三：我用 OD 打开光盘 smc.exe 跟到这一块代码后发现，与书中的印刷不同，实际代码是

`mov dword ptr [403020],0`

也这就是说 dr7=0 ，这才是清除呢！ 与源码不符啊。

另外，我想顺便问一下，dr7=0，能反 F7 跟踪吗？ 不懂。F7 跟踪的原理是不是设置 TF=1 啊？这样的话，与 dr7 寄存器没什么关系啊？ 知道的拜托给说一声。

总之，这一部分内容让我严重怀凝我哪儿有知识漏洞，或者是 kanxue 的作品中有疏漏了。

在上一版《软件加密技述内幕》中这一部分内容与此相同。 都有同样的疑问。

很抱歉现在才进行回复，主要是一检查才发现问题还不少，很惭愧，所以多花了些时间修正。

先说一下需要修正的地方：

1) 书中： `mov [sContext].iDr7, 11h` 此行代码应改为

```
   mov  [sContext].ContextFlags, CONTEXT_DEBUG_REGISTERS
   mov  [sContext].iDr7, 101h
```

原来的代码中设置 DR7 的值为 11h 计算错误，应为 LOCAL_EXACT_BPM_ENABLED 与 DR0_ENABLED 的或值，即 101h 。

同时应该添加对标志寄存器 ContextFlags 的操作，否则无法设置调试寄存器。

2) 随书光盘 OSMC.ASM 的代码 DecryptFunc 函数

```
DecryptFunc  proc uses ebx ecx edx esi edi lpBuffer:LPVOID,nBuffSize:DWORD
      mov    esi,lpBuffer
      mov    edi,esi
      mov    ecx,nBuffSize
      shl    ecx,2            <-- 此处应添加这行代码
loc_loop:  lodsb
      dec    al
      stosb
      loop   loc_loop
      ret

DecryptFunc  endp
```

同时随书光盘 TrSMC.ASM 的代码 EncryptFunc 函数

```
EncryptFunc  proc uses ebx ecx edx esi edi lpBuffer:LPVOID,nBuffSize:DWORD
      mov    esi,lpBuffer
      mov    edi,esi
      mov    ecx,nBuffSize
      shl    ecx,2            <-- 此处应添加这行代码
loc_loop:  lodsb
      inc    al
      stosb
      loop   loc_loop
      ret

EncryptFunc  endp
```

这里原来的代码中在调用加密解密函数时使用的是 DWORD 长度，但实际上加密解密函数是按字节进行操作的。所以这里必须加上以上两行代码。

3) 随书光盘 TrSMC.ASM 的代码 CreateImage 函数

```
      ;-=-=定位到 PE 文件头=-=-
      push  FILE_BEGIN
      push  0                  <-- 此处应添加这行代码
      push  [dwPeOffset]
      push  [hFile]
      call  SetFilePointer
```

这里原来的代码中调用 SetFilePointer API 时少了一个参数，使得运行 TrSMC.EXE 时无法正确生成 SMC.EXE 程序。

最后再来解释一下 SMC.EXE 中的所使用的两种反跟踪代码的作用。

即第一段

```
      pushf
      or    byte ptr [esp+1],01h
      popf
      nop
```

第二段

```
      pushad
      mov    edi,offset sContext
      mov    ecx,sizeof CONTEXT
      xor    eax,eax
      cld
      rep    stosb
      invoke GetCurrentThread
      mov    edi,eax
      mov    [sContext].ContextFlags, CONTEXT_ALL
      invoke GetThreadContext,edi,addr sContext
      mov    [sContext].ContextFlags, CONTEXT_DEBUG_REGISTERS
      mov    [sContext].iDr7,101h ;LOCAL_EXACT_BPM_ENABLED + DR0_ENABLED
      invoke SetThreadContext,edi,addr sContext
      popad
```

本意上写 SMC 这一节时希望能够把原理讲清楚，代码设计不是太复杂，对初学者而言能够更容易看懂。所以反跟踪的代码并没有在书中详细介绍其作用。

完整的代码也只放在随书光盘中。

在实际运用中通常仅仅只靠 SMC 来做安全保护，往往显得比较单薄，对于熟练的调试者来说很容易跳过。他的缺陷在哪儿呢？

这里介绍的 SMC 用作安全保护的原理是利用 SMC 来依次分段解密后面要运行的代码，边运行边解密。解密代码的密钥则是通过计算之前运行的代码字节与长度而得。即

`KEY = CalcKEY(INIT_KEY, PrevCode, PrevCodeLen)`
`NextCode = Decrypt(NextCode, KEY)`

如果之前运行的代码字节被修改，则计算的密钥错误从而无法正确解密出后面要运行的代码。

这种设计可以防范利用 INT 3 断点进行跟踪的方式，也就是说当调试者试图在当前运行的代码的某处下 INT 3 断点时，会导致代码字节被修改。这样就影响了

程序解密其后的代码的密钥，而密钥错误时解密代码得到的是一堆乱七八糟的字节，自然无法继续对其后代码的跟踪。

但是我们知道除了利用 INT 3 断点进行跟踪之外，还可以使用单步跟踪和硬件调试器断点进行跟踪。而以上的设计并不能防范这两种方式的调试跟踪。

于是针对这种设计的缺陷，需要在代码中加入相关的反跟踪代码与 SMC 结合，这样才能更好的防范调试者。书中主要考虑以介绍 SMC 的基本运用为主，就没有为此详细展开说明，这里就来多说几句。

前面加入的第一段反跟踪代码，针对的主要是单步跟踪。当程序没有被调试者跟踪时，执行到 popf 代码时会产生单步异常。而程序中之前已经通过以下代码

Block1:

```
      call  loc_next
      ....
loc_next:  push  fs:[0]
      mov   fs:[0],esp
```

建立了自己的异常处理函数。异常处理函数首地址即为 call loc_next 的下一条指令。

```
      ;-=-=这里是异常处理函数的起始地址=-=-
      mov    esi,[esp+4]
      assume  esi:ptr EXCEPTION_RECORD
      mov    edi,[esp+0Ch]
      assume  edi:ptr CONTEXT
      cmp    [esi].ExceptionCode,EXCEPTION_SINGLE_STEP
      jz     @F
      mov    eax,ExceptionContinueSearch
      ret

@@:      mov    eax,[edi].regEax
      xchg   eax,[edi].regEdx
      mov    [edi].regEax,eax

      xor    eax,eax
      mov    [edi].iDr0,eax
      and    [edi].iDr1,eax
      and    [edi].iDr2,eax
      and    [edi].iDr3,eax
      and    [edi].iDr6,0FFFF0FF0h
      and    [edi].iDr7,eax
      mov    [edi].ContextFlags,CONTEXT_ALL

      mov    eax,ExceptionContinueExecution
      ret
```

当程序产生单步异常时，异常处理函数中的处理主要是交换 EAX 和 EDX 寄存器的值，然后程序返回到异常点继续执行。

这里交换寄存器的作用是什么呢？我们来看程序中计算密钥的一段代码：

```
@@:      lodsd
      xor    eax,edx
      xor    eax,ecx
      ;-=-=把反跟踪与计算相结合=-=-
      pushf
      or     byte ptr [esp+1],01h
      popf
      nop

      loop   @B
```

以上代码中，EDX 初始值为 INIT_KEY ，ECX 为代码长度，EAX 为加密的代码字节。可以看到如果没有异常处理函数交换 EAX 和 EDX 寄存器，那么每一轮循环

时计算得到的值都保存在 EAX 中，在下一轮循环时由于 lodsd 指令，会丢失之前计算的 EAX 的值。而利用 pushf / popf 产生异常，在异常处理函数中交换

寄存器值，即功能等同于以下的代码：

```
@@:      lodsd
      xor    eax,edx
      xor    eax,ecx
      xchg   eax,edx
      loop   @B
```

这样最后在 EDX 中保存的即是计算得到的密钥值。

如果试图单步跟踪这段代码，误入陷阱的话，在这里就是单步跟踪到 popf 时，则不会再产生单步调试异常，直接运行到其后的 nop 指令处，由于没有运行异常处理函数

因而 EAX 和 EDX 寄存器值也就没有交换，最后计算的密钥也就是错误的。

如果你知道了这个原理，作为调试者时当然不会再傻到单步跟踪 pushf / popf 的地步。但同样如果明白了原理，也可以结合其他的反单步跟踪的代码与 SMC 方法结合来反跟踪。

现在再来介绍第二段反跟踪代码。其实这段代码也是后来才加入到整个程序之中，所以在代码开始和结束时用了 pushad / popad 对所有寄存器进行保存和恢复。你也可以注释掉

这段代码后重新编译下，来调试跟踪对比一下。

这段代码也是加在计算密钥的流程之中，如果误入陷阱，同样会影响到密钥的计算，最后导致无法跟踪到下一段利用 SMC 解密得到的代码。

这段代码通过使能 DR0 调试寄存器，使得调试者不能在跟踪时随意使用硬件寄存器断点或 BPM 断点，一旦设下断点，在使用 OllyDBG 调试时运行到 SetThreadContext 时可能会触发断点，从而改变了程序的流程，最终将影响程序的继续运行。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || SMC1.rar

|
