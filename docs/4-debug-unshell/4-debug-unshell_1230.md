
# 幻影

> 原文：[`www.pediy.com/kssd/pediy06/pediy6129.htm`](https://www.pediy.com/kssd/pediy06/pediy6129.htm)

本人水平较低,写这篇文章主要是想向大家学习,本人不解之处望各位能给与解答，并恳请大家指正错误。

工具: `trw2000`

目标: `dbpe 2.33`

## 一、随机改变关键代码的内存地址

动态加载 `dbpe 2.33` 开始不远处看到如下代码

```
call     @1

@1:    pop   ebp

       sub ebp 406913
```

`call` 指令将 `eip` 进栈,下一句 `pop ebp` 运行后 `ebp` 的值为当前代码的内存地址。减去一个固定值,将得到相对偏移量,运用以上两句代码可以对程序进行重定位，以后的代码采用如下形式间址寻址:

```
    MOV  EAX, [EBP+412345]
```

也就是说，代码放到任何地址都不会发生内存寻址错误。后面程序会使用 `CreateFileMapping`、`MapViewOfFile` 等 `API` 函数申请一段内存地址,用 `REP MOVB` 将代码转移过去,这样程序每次运行,关键部分的代码地址都会不同,达到反跟踪的目的.

## 二、花指令的使用

`Dbpe` 的花指令共有五种

### 1、

```
0167:004FE036  PUSHF   

0167:004FE037  PUSH     BYTE +10              设定循环次数  

0167:004FE039  JNC      004FE046              没用  

0167:004FE03B  JMP      SHORT 004FE03F  

0167:004FE03D  DB       C1                    花指令  

0167:004FE03E  DB       51                    花指令  

0167:004FE03F  CALL     004FE04A  

0167:004FE044  DB       CA                    花指令  

0167:004FE045  DB       11                    花指令  

0167:004FE046  JNC      004FE03F  

0167:004FE048  POP      EBX  

0167:004FE049  DB       CD                    花指令  

0167:004FE04A  ADD      ESP,BYTE +04          平衡堆栈  

0167:004FE04D  JMP      SHORT 004FE051  

0167:004FE04F  DB       99                    花指令  

0167:004FE050  DB       EB                    花指令  

0167:004FE051  DEC      DWORD [ESP]  

0167:004FE054  JNO      004FE057              没用  

0167:004FE056  DB       E8     

0167:004FE057  JNS      004FE039              判断[ESP]是否=-1，不等则转  

0167:004FE059  JPE      004FE05C  

0167:004FE05B  DB       75                    花指令  

0167:004FE05C  ADD      ESP,BYTE +04          平衡堆栈用  

0167:004FE05F  POPF     

0167:004FE060  JMP      SHORT 004FE063  

0167:004FE062  DB      75     

```

处理方法将第一句改为 `JMP 4FE063`，后面的改为 `NOP`

### 2、

```
0167:004FE063  PUSHF   

0167:004FE064  JC       004FE070  

0167:004FE066  JMP      SHORT 004FE069  

0167:004FE068  DB       63     

0167:004FE069  CALL     004FE073  

0167:004FE06E  JMP      SHORT 004FE0E7  

0167:004FE070  JC       004FE066  

0167:004FE072  DB       83     

0167:004FE073  ADD      ESP,BYTE +04  

0167:004FE076  POPF     

0167:004FE077  JMP      SHORT 004FE07A  

0167:004FE079  DB       75     

0167:004FE07A  PUSH     EAX  

```

处理办法,将这些代码直接改为 `nop`

### 3 常规的花指令

```
    jz         @1  

    jnz        @1  

    db xx                          花指令  

@1    xxxxxx
```

### 4

```
    jc         @1  

    jnc        @1  

    db xx  

@1    xxxxxx
```

### 5

```
    jno        @1  

    jne        @1  

    db xx  

@1    xxxxxx
```

3 4 5 三种花指令,直接改为 `nop` 即可，用程序实现自动取出花指令很容易，`VB` 部分代码如下,`data()` 为读入的文件 `byte` 代码

```vb
Private Sub DeFlower(StartPos As Long, EndPos As Long)

Dim i As Long

For i = StartPos To EndPos

'discard the flower instruction as such

'pushf

'push 10 ...

If Hex(data(i)) + Hex(data(i + 1)) + Hex(data(i + 2)) + Hex(data(i + 3)) + Hex(data(i + 4)) + Hex(data(i + 5)) + Hex(data(i + 6)) = "9C6A1073BEB2" Then

   data(i) = &HE9

   data(i + 1) = &H28

   data(i + 2) = 0

   data(i + 3) = 0

   data(i + 4) = 0

   Call ChangData(i, 5, 44)

     i = i + 44

     GoTo sw

   End If

'discard the flower instruction as such

'pushf

'jc xxxxxxxx ...

If Hex(data(i)) + Hex(data(i + 1)) + Hex(data(i + 2)) + Hex(data(i + 3)) + Hex(data(i + 4)) = "9C72AEB1" Then

  data(i) = &HEB

  data(i + 1) = &H15

  Call ChangData(i, 2, 22)

  i = i + 22

     GoTo sw

   End If

'discard the flower instruction as such

'jz xxxxxxxx

'jnz xxxxxxxx ...

If Hex(data(i)) + Hex(data(i + 1)) + Hex(data(i + 2)) + Hex(data(i + 3)) = "743751" Then

  Call ChangData(i, 0, 4)

  i = i + 4

     GoTo sw

   End If

'discard the flower instruction as such

'jc xxxxxxxx

'jnc xxxxxxxx ...

If Hex(data(i)) + Hex(data(i + 1)) + Hex(data(i + 2)) + Hex(data(i + 3)) = "723731" Then

  Call ChangData(i, 0, 4)

  i = i + 4

     GoTo sw

   End If

'discard the flower instruction as such

'jpe xxxxxxxx

'jpo xxxxxxxx ...

If Hex(data(i)) + Hex(data(i + 1)) + Hex(data(i + 2)) + Hex(data(i + 3)) = "7A37B1" Then

  Call ChangData(i, 0, 4)

  i = i + 4

     GoTo sw

   End If

sw:

Next i

End Sub

Private Sub ChangData(Pos As Long, StartPos As Integer, EndPos As Integer)

For j = Pos + StartPos To Pos + EndPos

     data(j) = &H90

   Next j

End Sub
```

## 三、UPX 解码

用以上方式去处花指令,只能去处很小的一部分,因为程序大量的代码是一段一段解码后运行,解码一段,运行一段,内存中不出现完全解码的文件,这样可以有效的对付反跟踪。处理办法是:多次运行去花程序。

大段的代码软件使用了 `upx` 加解密，程序开始处使用了一次，后面恢复被加密程序各个段使用的也是这种方式。

```
0167:004FE0F0  PUSH     BYTE +04

0167:004FE0F2  PUSH     DWORD 1000

0167:004FE0F7  PUSH     EAX

0167:004FE0F8  PUSH     BYTE +00

0167:004FE0FA  CALL     NEAR [EBP+004384F6]  ->VirtualAlloc
```

申请一段内存地址 返回值 `eax`=申请内存的首地址

```
push    eax

push    ebx          4fe3c5 加密段首地址

call     4fe234      upx 解码程序
```

返回值=解码字节

后面用 `rep movsb` 将解密后代码写回 `4fe3c5`，再用 `VirtualAlloc` 释放申请的内存，一般来说，解密后的代码大于解密前的，但 `D.Boy` 在程序中预留了位置,这样就可以先将解密的数据 `W` 成一个文件,贴到程序的对应位置,前面用一个 `JMP` 指令跳过解码的指令。去花指令。

## 四、垃圾指令与 MMX

程序用 `CPUID` 指令判断 `CPU` 是否支持 `MMX` 指令集，如果支持就用 `MMX` 指令动态解密运行，代码中间插入了大量的垃圾代码来干扰跟踪者的视线。如

```
mov eax , 47ad4470

mov ebx , 6421c319            垃圾代码

movq mm2,mm3

sub ebx, ebx                 垃圾代码

pcmpeqd mm0, mm1
```

`mmx` 指令集网上有大量的介绍,中英文都有,这里就不细说了。但这段代码很长，而且是翻译一段，运行一段，这里需要一定的耐心。

## 五、利用 int3 反跟踪

这里看看如下代码，这是去处花指令的，以后同

```
sidt     [esi]                     取中断向量表

mov     esi，  [esi+2]

mov     ax，  [esi+18]             取 int3 的原入口地址，

mov     bx，  [esi+1e]

mov     [ebp+xxxx] ，ax           保存 int3 的原入口地址

mov     [ebp+yyyy] ，bx

mov     eax ， 4241aa

add      eax ，ebp                 eax 为新的 int3 入口地址

mov     [esi+18]， ax              将 int3 的入口地址指向自己的程序

mov     shr  eax，10

mov     [esi+1e]，ax
```

利用如上代码，就改变了 `int3` 的入口地址，跳是软件如 `trw2000` 也会将 `int3` 的入口地址指向自己的程序，这样就可以有效的对付大部分调试软件，这里第一次设定的将 `int3` 的入口地址为 `52f876`，执行 `52f876` 的代码时，获得的是 `ring0` 级权限。自己的程序也可以用这种方法获得 `ring0` 级权限。

程序利用 `int3` 跳到自己的代码部分解密下一段代码，同时设定新的入口，另一个分支是 `call ebx`。看看如下代码：

```
       CMP      EAX,52554E53          

       JNZ      0052F8EF

       CALL     EBX                    程序后半部用到

       IRET      

0052F8EF  CMP      EBX,554E434F

       JNZ      NEAR 0052FB58

0052F905  MOV      AH,[ESI]            解密

       XOR      AH,AL

       NOT      AH

       MOV      [ESI],AH

       INC      ESI

       DEC      ECX                    解密长度，开始几段都相同

       CMP      ECX,BYTE +00

       JNZ      NEAR 0052F905

       DEC      AL                     下一段解密 XOR 的值为这次的减 1

       LEA      EDI,[EBP+00437D36]

       SIDT     [EDI]                 重设 INT3 的入口地址

       MOV      EDI,[EDI+02]

       MOV      EBX,[ESP]            *注 1

       CMP      BYTE [EBX],E9         E9 为 JMP

       JNZ      0052FA82

       ADD      EBX,BYTE +05          JMP 的指令长度，INT3 设到 JMP 的下一句

       JMP      0052FA94

0052FA82  ADD      EBX,BYTE +02

0052FA94  MOV      [EDI+18],BX

       SHR      EBX,10

       MOV      [EDI+1E],BX

       MOV      EBX,0155

       MOV      DR7,EBX              *注 2

       MOV      EBX,554E434F

0052FB58  IRET      
```

\*注 1：`INT3` 执行后，先将标志寄存器内容入栈，`CS` 寄存器扩展到 32 位入栈，`EIP` 入栈，执行 `IRET` 时以相反出栈，`[ESP]` 为 `INT3` 下一句的地址

\*注 2：`DR7` 为调试控制寄存器，只有在 `ring0` 权限时才能操作此寄存器，以后的代码中还会见到 `dr0 dr1 dr2 dr3`，它们是调试断点寄存器，`dr6` 为调试状态寄存器。

这段代码也是解密一段执行一段，去花程序不起作用，但这段代码长度结构都相同，我猜想是由程序自动生成的，这样就可以把这段 `w` 出来，用自己的程序解密（代码略），找到代码开始变化时地址位置，手动跟踪时直接跳过去，到后面大段解码完成后，将解码后的代码写回源程序，去掉解码部分代码。这种方法还是很繁琐，大家有好的方法如能告知不胜感激。

## 六、判断中断向量

```
sidt     [esi]                     取中断向量表

mov     esi，  [esi+2]

mov     ax，  [esi+2e0]           

mov     bx，  [esi+6]

xor      ax ,bx

jnz      error
```

如果没有跟踪软件时 `ax` 与 `bx` 的值相同,如果不同那么……

## 七、判断驻留程序

```
push  0

push  0

push  0

push  0
```



# 反跟踪与文件校验技术解析

## 反跟踪代码片段

```
push  0  
push  0  
push  esi  
call   CreateFileA  
cmp  eax ,  -1  
jnz   error  
```

`esi` 指向的内容包括 `\\.\\BW2K\\.\\`、`\\.\\SUPERBPM\\.\\`、`\\.\\ICEDUMP\\.\\`、`\\.\\REGVXD\\.\\`、`\\.\\NTICE\\.\\`、`\\.\\SIWVID\\.\\`、`\\.\\SICE\\.\\`、`\\.\\FILEVXD\\.\\`。

如果这几种程序没有驻留，`CreateFileA` 返回值 `eax` 应为 `ffffffff`，程序就用此方法反跟踪。

## 八、文件校验

文件校验共出现两次。

```
@1  mov   bl, [edi]          ; edi 代码地址  
   xor   bl, cl  
   add   eax, ebx  
   inc   edi  
   dec   ecx  
   cmp   ecx, 0             ; 长度  
   jnz   @1  
   lea   ebx, [ebp+42cf59]  
   cmp   [ebp+42d1a8] , eax ; 比较校验和是否相当  
   jnz   error  
```

利用 `prodump` 抓取的影像文件的校验和，与原来的肯定要不同。

## 九、利用定时器反跟踪

```
push      ebx  
push      1f4  
push      xxx  
push      xxx  
call      SetTimer  
```

定时器的时间间隔为 `500ms`，`ebx` 指向事件代码，事件代码用到了上面提到的 `int3` 反跟踪技术获得 `ring0` 权限，这样程序运行后打开调试软件也会被程序发现。

## 十、不明之处

代码如下：

```
mov  eax, [fs: word 30]     ; 不明  
push   0  
call      GetModuleHandleA  
test      edx, edx           ; 不明  
jns      error  
cmp   [edx+8],  -1           ; 不明  
jnz      error  
mov   [edx+50], 1000        ; 不明  
mov   [edx+6] , 1010        ; 不明  
```

`eax` 的返回值正确为 `400000`，`edx` 的值为 `819xxxxx`，怀疑与线程有关，执行到 `mov [edx+6], 1010` 时死机，不明所以，望指教。

-- `SWay[CCG]`

---

## 讨论

**`SWay[CCG]` 回复：**

> 下面引用由 `linson` 在 *2003/06/21 03:55pm* 发表的内容：  
> 这段我不明白啊,能否详细说说原理?

### 文件校验逻辑解析

文件校验共出现两次，代码如下：

```
@1  mov   bl, [edi]          ; edi 代码地址  
   xor   bl, cl  
   add   eax, ebx  
   inc   edi  
   dec   ecx  
   cmp   ecx, 0             ; 长度  
   jnz   @1  
   lea   ebx, [ebp+42cf59]  
   cmp   [ebp+42d1a8] , eax ; 比较校验和是否相当  
   jnz   error  
```

利用 `prodump` 抓取的影像文件的校验和，与原来的肯定要不同。

程序运行时 `edi` = 需要计算的代码地址，`ecx` = `0x1c000`，相当于下面的说明语句：

```
for (cl=0x1c000; cl!=0; cl--) {  
    sum += [EDI] ^ CL;  
    edi++;  
}  

if (sum != CorrectSum) {  
    error();  
}
```

**`飞叶流枫` 回复：**

> 下面引用由 `pll621` 在 *2003/06/22 01:24am* 发表的内容：  
> 十、不明之处  
> 代码如下  
> `mov eax, [fs: word 30]` ; 不明

`\===================== `

此代码可以用于反某些调试器，当有调试器和没有调试器的时候内容是不同的。

上面所说的并非是检测进程拥有者，而是 `Anti Dump`，其汇编指令如下：

```
        push    fs:[30h]  
        pop     eax  
        TEST    EAX, EAX  
        JS      _win9x     ; 判断操作系统的类型  
_iswinnt:  
        MOV     EAX, [EAX+0Ch]  
        MOV     EAX, [EAX+0Ch]  
        MOV     DWORD PTR [EAX+20h], 1000h ; change proc size=1000h  
        JMP     _over  
_win9x:  
        PUSH    0  
        CALL    GetModuleHandle  
        TEST    EDX, EDX  
        JNS     _over                  
        CMP     DWORD PTR [EDX+8], -1  
        JNE     _over                 
        MOV     EDX, [EDX+4]           
        MOV     DWORD PTR [EDX+50h], 1000h ; change proc size=1000h  
```

在经过上面的指令后，我们 `DUMP` 出来的程序就只有 `1000h` 大小了。

**`swift` 回复：**

必须提出一点，我最早发布幻影 `2.3x` 破解版的时候，就指出了目的不是为了让大家用它来免费加密自己的程序，而更重要的是破除国内很多人对 `d.boy` 加密工具的迷信、盲目崇拜和畏惧。

我在 `win2k sp3` 下调试 `dbpe 2.33`，和楼主的分析结果有些出入，这里提出来供参考。

楼主提出程序大量的代码是一段一段解码后运行，解码一段，运行一段，内存中不出现完全解码的文件。而根据我的分析结果，`dbpe` 的壳的部分虽然多次通过调用 `VirtualAlloc` 申请一段内存，但是其最重要的也是最长的一段代码，包括把捆绑的注册界面释放、注册码读取以后的判断，对被加密的主程序的解码以及跳到真实 `OEP` 的这些东东都是在原地解码并且一次性 `rep movsb` 过去的。

至于花指令，我认为它的主要目的是为了对抗现有的静态分析工具，以及动态跟踪的时候，断到某处以后让你难以找到它前面运行过的指令是什么。对于前者，只要稍微调整一下静态分析的引擎就可以越过这个障碍。对后者，解决的办法可能就困难一些，因为现在大多数的调试工具对 `back trace` 的支持都很有限。

> 下面引用由 `newlaos` 在 *2003/06/21 03:46pm* 发表的内容：  
> 高手出招了。  
> 不过想来也是必然，因为国内有越来越多的软件作者使用幻影加密（`swift` 的功劳），那么软件解密总不能没饭吃吧。  
> 自然就会有人去研究、去讨论，其结果也是可想而知了。  
> 世上没有无坚不摧的矛，也没有 ...

`4dd466d0e1e0cc123afbe504da2462ca`
