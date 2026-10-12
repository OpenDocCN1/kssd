
# PE 病毒学习笔记搜索 API

> 原文：[www.pediy.com/kssd/pediy09/pediy09-291.htm](https://www.pediy.com/kssd/pediy09/pediy09-291.htm)

> 这几天学习了一下古老的 PE 病毒原理，不过刚开始没多久，就打算一心一意复习考研，不再搞别的事情分心了。本来准备发不止一个学习笔记，其中包括感染技术，传播技术等等，不过都没时间了，只好先把这个发出来再说吧。

> 对于 Ring3 的 PE 病毒来说，我想最重要的莫过于搜索 API 地址了，因为所有的所有别的事情，包括感染、注入、获取机器信息、升级到 Ring0、传播甚至破坏，没有哪一个不会用到 API 的，只要找到了 API 地址，那么任何一个普通程序能做的事，病毒都能做到，甚至包括 WinFlash 干的事。

> `kernel32.dll` 里有一个函数：`GetProcAddress`，只要给定模块 handle 和 API 的名字，就可以得到 API 的地址，很不错的搜索函数。当然，`GetProcAddress` 也是一个 API，所以我们自己的搜索代码不是没有用武之地的，不过简单的来说，我们只用搜索出 `GetProcAddress` 就可以了。

> 在搜索 API 的方法上，病毒和蠕虫有点点区别。我现在还没有听说过可以攻击各种各样完全不同漏洞的通用蠕虫，当然，每个漏洞一般对应一个特定的程序的特定版本；我也没听说过只感染一个特定程序的病毒。针对特定漏洞的蠕虫所处的是一个特定的环境，这个环境是可预知的，所以，蠕虫编写者可以直接调试漏洞程序，分析漏洞程序导入表中 `GetProcAddress` 地址的位置，然后直接 hardcode。那么如果漏洞程序升级了，导入表变了呢？放心，也别指望新版本里漏洞还给你留着。

> 病毒所面对的就是比较极端的环境了，完全可变的通用的环境，唯一知道就是这是一个 PE 程序（这是感染部分选择感染对象的职责，如果你硬要感染 word 文档我也没话说）。还有呢？有！这个程序至少加载了 `kernel32` 模块这是无条件的！否则这个进程怎么被创建的呢？主线程怎么创建的呢？

> “正如你所知道的，当我们在执行一个应用程序的时候，代码是从 KERNEL32 "call" 一部分代码的(也就像 KERNEL 调用我们的代码一样)。而且，如果你还记得的话，当一个 call 调用之后，返回的地址是在堆栈里(即，在由 ESP 所指定的内存地址里的)。”
> 被引用过无数次的一个常识

> 其实这个 `call` 就是 `CreateProcess` 发出的，这很好理解。那么，有了这个处于 `kernel32` 内部的地址，寻找 `kernel32` 的基址应该不成问题，`kernel32` 肯定是按页加载和对齐的，`kernel32` 的基址肯定是一个页首。我们可以一页一页向前搜索 `kernel32` 的 `MZ` 头。

`GetK32.asm`

```asm
;---------------------------
; input:
; ESI = near page addr
; EAX = limit checkpagenum
;
; output:
; EAX = ImageBase of kernel32
;
; used reg:
; EAX,ESI,EDI
;---------------------------

GetK32 proc
    and eax,eax
    jz _GetK32_Fail

_GetK32_Start:
    cmp word ptr [esi],"ZM"
    jnz _GetK32_Loop
    mov edi,[esi+3Ch]
    add edi,esi
    cmp dword ptr [edi],"EP"
    jz _GetK32_Done

_GetK32_Loop:
    sub esi,10000h
    dec eax
    jnz _GetK32_Start

_GetK32_Fail:
    mov esi,7c800000h

_GetK32_Done:
    xchg eax,esi
    ret
GetK32 endp
```

> 仅仅判断"MZ"还不够，有可能会是碰上的，所以我们还要再追踪下"PE"，当然像上面那样直接找可能会访问到非法区域。当然，可以添加 SEH 来避免错误被显示出来。我们还需要一个搜索页数的上限，避免无限制搜索下去。其实这个搜索是很快的，因为这个被调用函数是 `CreateProcess`，具体原因后面我会提到。

> 好，现在找到了 `kernel32` 的基址，我们把它存在 `kernel` 这个变量里。

```asm
kernel dd 0

mov esi,[esp]
and esi,0ffff0000h
mov eax,100
call GetK32
mov kernel,eax
```

> 下面我们就要搜索 `kernel32` 的导出表了，熟悉 PE 文件结构的人肯定不会陌生，在导出表的名字表里一个一个比较，就这么简单。不过我从来都不喜欢字符串比较函数，包括 C 标准库里面的字符串函数我都不喜欢，我总想要更直观的函数，因为我们只需要看看两个字符串是否相等，当然，比较下他们的 CRC32 就可以了，刚好是一个双字的比较，一条指令。

> 首先，我们需要一个求 CRC32 的代码，我在 Billy Belceb 病毒编写教程---Win32 篇看到了一个

```asm
;----------------------------------------------
; input:
; ESI = Offset where code to calculate begins........volatile
; EDI = Size of that code............................volatile
;
; output:
; EAX = CRC32 of given code
;
; used reg
; EAX,EBX,ECX,EDX,ESI,EDI
;----------------------------------------------

CRC32 proc
    cld
    xor ecx,ecx     
    dec ecx          
    mov edx,edx     
     
NextByteCRC:
    xor eax,eax     
    xor ebx,ebx     
    lodsb            
    xor al,cl        
    mov cl,ch        
    mov ch,dl        
    mov dl,dh        
    mov dh,8         

NextBitCRC:
    shr bx,1         
    rcr ax,1        
    jnc NoCRC
    xor ax,08320h
    xor bx,0EDB8h
    
NoCRC:
    dec dh
    jnz NextBitCRC
    xor ecx,eax
    xor edx,ebx
    dec edi         ;---------------- 1 byte less 
    jnz NextByteCRC
    not edx
    not ecx
    mov eax,edx
    rol eax,16
    mov ax,cx
    ret
CRC32 endp
```

> 我想，有了 `kernel32` 的基址，得到 `kernel32` 导出表的名字表和地址表还有序号表，不是问题吧。

```asm
;--------------------------------------
; input:
; kernel = ImageBase of kernel32.dll
;
; output:
; ESI = OrdinalTableVA----VA
; EBX = NameTableVA-------VA
; EDI = AddressTableVA----VA
;
; used reg:
; EAX,EBX,ESI,EDI
;--------------------------------------

GetAPI_Init proc
    mov esi,3ch
    add esi,kernel                ;------------ Get PE header of KERNEL32
    lodsd
    add eax,kernel
    mov esi,[eax+78h]           ;------------ Get a RAV pointer to its Export Table
    add esi,1ch                 ;------------ RAV pointer to address table
    add esi,kernel     ;------------ Turn RVA to VA

    mov edi,[esi]      ;------------ Pointer to the address table
    add edi,kernel
    mov ebx,[esi+4]
    add ebx,kernel
    mov eax,[esi+8]
    add eax,kernel
    mov esi,eax
    ret
GetAPI_Init endp
```

> 上面这个部分只运行一次，我不喜欢读写内存，在我印象里，读写内存是很慢的事，多用寄存器才是我喜欢的事。

```asm
;--------------------------------------
; input:
; EDX = CRC32 of the API ASCIIz name........volatile
; EBP = Length of API name
; ESI = OrdinalTableVA----VA
; EBX = NameTableVA-------VA
; EDI = AddressTableVA----VA
; kernel = ImageBase of kernel32.dll
;
; output:
; EAX = API address
;
; used reg
; EAX,EBX,ECX,EDX,ESI,EDI,EBP
;--------------------------------------

GetAPI_ET_CRC32 proc
    push edi
    push esi
    xor ecx,ecx
p1: mov edi,[ebx+ecx*4]
    add edi,kernel
    mov esi,edi
    xor al,al
lp: scasb
    jnz lp
    sub edi,esi
    inc ecx
    cmp ebp,edi
    jnz p1
    push ecx
    push edx
    push ebx
    call CRC32
    pop ebx
    pop edx
    pop ecx
    cmp edx,eax
    jnz p1
    dec ecx
    pop esi
    pop edi
    xor edx,edx
    mov dx,word ptr [ecx*2+esi]
    mov eax,[edx*4+edi]
    add eax,kernel
    ret
GetAPI_ET_CRC32 endp
```

> 把你要找的 API 的 CRC32 值放在 `edx` 里，名字长度（包括结尾的 0）放在 `ebp` 里，`eax` 输出的就是它的地址，不错吧。而且这个过程不破坏用于存放 3 个表地址的 3 个寄存器 `EBX`,`ESI`,`EDI`，所以一次 Init 以后可以反复调用，直到把你自己构造的“导入表”（我习惯这么叫，叫跳转表也不错）添满为止。

> 不过我只要一个函数的地址，就是 `GetProcAddress`，15 字节，CRC32：`0FFC97C1Fh`
> 如果要再加一个的话，那就是 `LoadLibraryA`，13 字节，CRC32：`04134D1ADh`

> 写个实际的程序试试
> `test.asm`

```asm
.386
.model flat,stdcall
option casemap:none

.data
kernel dd 0    
GETPROCCRC equ 0FFC97C1Fh
LORDDLLCRC equ 04134D1ADh
user db "user32.dll",0
mess db "MessageBoxA",0
userhd dd 0
getprochd dd 0

.code

GetAPI_Init proc
......
GetAPI_Init endp

CRC32 proc
......
CRC32 endp

GetAPI_ET_CRC32 proc
......
GetAPI_ET_CRC32 endp

GetK32 proc
......
GetK32 endp

start:
  mov esi,[esp]
  and esi,0ffff0000h
  mov eax,100
  call GetK32
  mov kernel,eax
  
  call GetAPI_Init
  
  mov edx,LORDDLLCRC
  mov ebp,13
  call GetAPI_ET_CRC32
  
  push offset user
  call eax
  mov userhd,eax
  
  mov edx,GETPROCCRC
  mov ebp,15
  call GetAPI_ET_CRC32
  
  mov getprochd,eax
  
  push offset mess
  mov eax,userhd
  push eax
  mov eax,getprochd
  call eax
  push 0
  push offset mess
  push offset mess
  push 0
  call eax
  
over:  mov edx,040F57181h
  mov ebp,12
  call GetAPI_ET_CRC32
  push 0
  call eax
end start
```

> 用 MASM32 编译连接

```bash
ml /c /coff test.asm
link /SUBSYSTEM:WINDOWS test.obj
```

> 我没有导入任何头文件和库文件，不过这个程序可以跑，这也就验证了我们的代码的功能。

> 最后再提几点：

> 1，我这程序显式地用到什么函数？从字符串来看，只能看到"MessageBoxA"！这就是 CRC32 搜索法的好处，`GetProcAddress`，`LoadLibraryA`，还有一个：`ExitProcess`，CRC32：`040F57181h`，直接在代码里看不出来的。



2，这种方法可以用在蠕虫里么？严格的来说不可以，因为看看我们搜索`kernel32`基址的函数，我们利用`kernel32`内的`CreateProcess`函数`call`我们程序本身的代码并将返回到`CreateProcess`内部的地址留在`[esp]`的特点来搜索`kernel32`头，因为病毒代码总是在宿主程序运行前执行，也就是处在一个进程刚刚创建的状态。而蠕虫所利用的漏洞一般在很复杂的、不知道有多少层的子函数里，这个时候的`esp`早不知道指向哪里去了。

3，为什么我说从`CreateProcess`（准确来说`CreateProcessA`或`CreateProcessW`）搜索`kernel32`头几步就完成了呢？因为我在调试时发现，`kernel32`导出表里名字表内的函数名排列是按照字典序排列的，而`CreateProcess`开头是`C`，是不是很快呢？这个特点也可以让我们写出更高效的`kernel32`的`API`搜索法，比如二分排序等等……（学计算机的，就总免不了在算法上较劲），当然实用性不高，呵呵

这篇文章不敢妄称原创，虽然除了算`CRC32`的代码外其他代码都是我自己写的，但是思想是别人的，我只是一个学习者，所以我这叫学习笔记，而且是`[分享]`而不是`[原创]`。这些实践大大加深了我对`PE`文件格式的理解，看来动手才是提高的唯一办法。

每次来看雪都有一种很亲切的感觉，大家都很热情，人气很高，讨论的内容也很有技术含量，不像别的论坛，一旦问题深入一点，就卖关子故弄玄虚甚至开口要价，好像不这样就显不出自己是高手一样。我想管理者与主持者的对技术的热情和无私，对培养后继者，端正论坛风气有很大益处，只有这样看雪才会继续繁荣下去。

参考文献：《Billy Belceb 病毒编写教程---Win32 篇》

代码:

```asm
;   ; xlib v1.30                         (x) 2007   ; ~~~~~~~~~~   ; coded by forgot & heXer   ;   ; example   ; ~~~~~~~   ; include                 xlib.inc   ;   ;                       initX    XFL_MAX, "user32", "ntdll"   ;   ;                       getX     KiUserExceptionDispatcher   ;   ;                       push     eax   ;                       push     ofs buf   ;                       push     ofs fmt   ;                       callX    wsprintfA   ;                       add      esp, 3*4   ;   ;                       push     0   ;                       push     0   ;                       push     ofs buf   ;                       push     0   ;                       callX    MessageBoxA   ;   ;                       callX    ExitProcess      XFL_GET               equ      1                               ; use getX macro   XFL_DBG               equ      2                               ; use push calls   XFL_ILN               equ      4                               ; initX can be run through   XFL_MIN               equ      0                               ; use minimum setting   XFL_MAX               equ      7                               ; use maximum setting      initX                 macro    fl:req, dll:vararg    local    hash    local    stub, ntport, getkrnl    local    getdll, restart, chknext    local    gethash, reached, alone     hash                  macro    s    h = 0    forc c, <s>      h = ((h shl 7) and 0ffffffffh) or (h shr (32-7))      h = h xor "&c"    endm    exitm <h>    endm     callX                 macro    x    if (fl and XFL_DBG)      push  hash(x)      call  stub    else      call  stub      dd    hash(x)    endif    endm     getX                  macro    x    if (fl and XFL_GET)      if (fl and XFL_DBG)        push  hash(x)        call  stub+1      else        call  stub+1        dd    hash(x)      endif    else      error    endif    endm       if (fl and XFL_ILN)      jmp   alone    endif  stub:    if (fl and XFL_GET)      test  al, 0f9h    endif     if (fl and XFL_DBG)      pop   edx      pop   eax      push  edx    else      xchg  esi, [esp]      lodsd      xchg  esi, [esp]    endif     if (fl and XFL_GET)      pushfd      pushad    else      pushad    endif     push    34h    pop     edx    db      64h    mov     eax, [edx-04h]    test    eax, eax    jns     ntport    add     edx, [eax+edx]    lea     eax, [eax+7ch]    jmp     getkrnl  ntport:               mov     eax, [eax+0ch]    mov     esi, [eax+1ch]    lodsd  getkrnl:              mov     eax, [eax+08h]     call    getdll    for     c, <dll>      db    c, 0    endm  getdll:              pop     edi  restart:              xchg    ebx, eax    mov     eax, [ebx+3ch]    mov     ebp, [ebx+eax+78h]    add     ebp, ebx     mov     ecx, [ebp+18h]  chknext:              lea     esi, [ebx+ecx*4]    add     esi, [ebp+20h]    lodsd     cdq  gethash:              rol     edx, 7    xor     dl, [ebx+eax]    inc     eax    cmp     bl, [ebx+eax]    jne     gethash     lea     esi, [esp+7*4]    sub     edx, [esi]    jz      reached    dec     ecx    jns     chknext     push    edi    callX   LoadLibraryA     repnz   scasb    jmp     restart  reached:              mov     edx, [ebp+24h]    add     edx, ebx    movzx   edx, word ptr [edx+ecx*2]    mov     eax, [ebp+1ch]    add     eax, ebx    add     ebx, [eax+edx*4]    mov     [esi], ebx     if (fl and XFL_GET)      popad      popfd      push  eax      jnc   $+3      pop   eax      retn    else      popad      jmp   eax    endif  alone:    endm
```

引用:

*最初由* `kbmaster` *发布* (帖子 366181) 用 `crc` 判字符串,不知道你是否用?大小怎分?????

国际标准已经定义了 8、12、16 和 32 比特生成多项式。32 比特的标准 `CRC32` 被大量的链路级 `IEEE` 协议采用。每个 `CRC` 标准都能检测小于 r+1 比特的突发错误（对于 `CRC32` 来说就是 33 比特），而且在合适的假设下，长度大于 r+1 比特的突发差错被检测到的概率为 `1-0.5^r`。每个 `CRC` 标准都能检测任何奇数个比特差错。

比如以太网帧使用的就是 4 个字节 `CRC32` 检测字段判断帧是否出错，这个运算过程还有整个协议栈集成在了我们的以太网卡里。

以太网帧最长可以有 1500 字节，如果你相信你上网的网卡不会给你传送错误的数据的话，你也就应该相信 `CRC32` 肯定可以检测名字长十几二十个字节的 `API` 的不同，至少，没有 1500 字节长的 `API` 名字吧！

当然，用任何 `Hash` 函数来计算都是可以的，只是 `CRC32` 非常简单，很多人都会计算，所以这里用 `CRC32`。而且我不懂这和 `ASCII` 字符字母大小写有什么关系，比如 `'A'`和`'a'`的 `ASCII` 码不是有一个比特不同吗，凭什么 `CRC32` 检测不出来呢？

补充一个小技巧

我用的 `HASH` 函数比 `CRC` 简单，所以我编程的时候是写了一个 `HASH` 的宏，用的时候直接写：

`HASH("MessageBoxA")`，就能返回一个常量，编程的时候就不用自己算 hash 了

如果方便的话，你可以写一个 `CRC` 的宏，然后 `CRC("MessageBoxA")`

看到 `PEB` 了吗，扔掉那个蹩脚的搜索吧

```asm
GetAPI_ET_CRC32 proc 
    push edi 
    push esi 
    xor ecx,ecx 
p1: mov edi,[ebx+ecx*4] 
    add edi,kernel 
    mov esi,edi 
    xor al,al 
lp: scasb 
    jnz lp 
    sub edi,esi 
    inc ecx
```



```asm
    cmp ebp,edi  
    jnz p1  
    push ecx  
    push edx  
    push ebx  
    call CRC32  
    pop ebx  
    pop edx  
    pop ecx  
    cmp edx,eax  
    jnz p1  
    dec ecx  
    pop esi  
    pop edi  
    xor edx,edx  
    mov dx,word ptr [ecx*2+esi]  
    mov eax,[edx*4+edi]  
    add eax,kernel  
    ret  
GetAPI_ET_CRC32 endp
```

这里:

```asm
    xor edx,edx  
    mov dx,word ptr [ecx*2+esi]  
    mov eax,[edx*4+edi]  
```

此时 `ecx` 就是 rva 表里 要找的 api 的 rva 的个数偏移 `edi` 指向 rva 表 开始位置

是不是可以将此三句改为:

```asm
mov eax,[ecx*4+edi]
```

?

另外

```asm
mov dx,word ptr [ecx*2+esi]  
    mov eax,[edx*4+edi]
```

这两句不理解 `esi`:OrdinalTableVA 和 `ecx*2` ?

对 pe 格式学得不熟

**引用:**

*   最初由 `netwind` 发布 (帖子 366582)

```asm
GetAPI_ET_CRC32 proc  
    push edi  
    push esi  
    xor ecx,ecx  
p1: mov edi,[ebx+ecx*4]  
    add edi,kernel  
    mov esi,edi  
    xor al,al  
lp: sc...
```

你说得没错，其实我第一个版本的代码就像你说的那样，直接通过名字表里的序数在地址表索引。所以我一开始也就没有保存序号表的地址。这在 `kernel32` 里是没有一点问题的！在 `kernel32` 里名字的序号实际就是和地址的序号相对应的，没必要通过序号表，我想是因为 `kernel32` 里没有一个函数多个名字的原因吧。可是严格来说，名字表和地址表不是一一对应的，因为存在一个函数被导出多个不同名字的情况，或者可能还有别的原因。所以 PE 结构的情况是：

名字表的序数对应序号表相应序数的相应项，序号表的项里面的值就是地址表相应项的序数。

序号表每项是 2 个字节，也就是 16 位，这与名字表和地址表每项 4 个字节 32 位不同。

比如在名字表第 X 项所指的字符串就是你想要的函数，那么名字表第 x 项距离名字表头的距离应该是 `X*4` 字节。而序号表的第 X 项（距离序号表头为 `X*2` 字节）存的就是这个函数的序号，也就是`[X*2+序号表首地址]`。把这个序号提取出来，假设为 X'，那么，地址表的第 X'项（距离地址表头为 `X*4` 字节）存的就是你要的函数的地址。也就是`[X'*4+地址表首地址]`

在 `kernel32` 里这个 X=X'，也就是序号表里第 X 项的值就是 X 本身。可是在别的地方这就不一定了。

如果对 PE 不熟的话，记住一点：任何从文件内部提取的地址都是 RVA，必须加上文件加载的基地址 `ImageBase` 后才能使用。

**引用:**

*   最初由 `forgot` 发布 (帖子 366352) 看到 PEB 了吗，扔掉那个蹩脚的搜索吧

我又稍微了解了下 PEB 的资料，我发现 PEB 搜索模块基址这个方法只能在 `Win NT/2000/XP` 这个系列里跑，似乎在 `Win9x/Me` 里面不行。当然，也许从实际说起来在中国根本不用考虑跑 `Win9x/Me` 系列的机子的问题，没错，一个连 `CreateRemoteThread` 函数都没有的系统，在中国已经濒临绝迹。

但是，如果硬要兼容 `Win9x/Me` 系列的话，似乎可以用展开 `SEH` 链的办法，遍历到 `SEH` 链尾的 `KERNEL32!_except_handler3` 函数，因为这个函数在 `kernel32.dll` 内部，然后用我那个从 `CreateProcess` 向前搜索 kernel32 基址的办法，从这个 Windows 默认分配的 SEH 程序向前搜索 kernel32 基址。

这个办法也可以适用于蠕虫。很麻烦的方法，不过相比 PEB，似乎唯一的好处就是可以肯定（根据 MS 公开发布的详细的文档）能在 `Win9x/Me` 下跑……

`4dd466d0e1e0cc123afbe504da2462ca`
