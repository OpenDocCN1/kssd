
# 寻找失踪的代码（ACProtect2.11 Stolen OEP 修复 VC6）

> 原文：[`www.pediy.com/kssd/pediy10/98908.html`](https://www.pediy.com/kssd/pediy10/98908.html)

## 基本信息

*   **文章标题**: 寻找失踪的代码（ACProtect2.11 Stolen OEP 修复 VC6）
*   **文章作者**: `newsoft88`
*   **作者邮箱**: `newsoft88@126.com`
*   **作者主页**: `http://newsoft88.ys168.com`
*   **软件名称**: `DiskG.exe`
*   **软件大小**: `5M`
*   **加壳方式**: `ACProtect`
*   **保护方式**: `ACProtect2.11 PRO`
*   **编写语言**: `VC++6`
*   **使用工具**: `OD`、`ACP2.0` 脚本、其他脱离壳辅助工具
*   **操作平台**: `WIN98`、`2000`、`XP`
*   **软件介绍**: 加上 `ACProtect2.11` 的壳，STOLEN OEP
*   **作者声明**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

这个东西本来加的是 `ACProtect2.11` 的壳，用了机机居然他下不来，运行不了，有时真想把他放一边了！后来再次查看时，是 OEP 的被 Stolen 出的问题，所以今天我们不去谈具体脱壳的内容，而是寻找那被丢失的 OEP，还软件的原来模样！

废话少说，首先打开 `PEID`，检查是什么壳型，`PEID` 这次报出 `UltraProtect 1.x -> RISCO Software Inc.`；没办法，很多的 `ACP` 都这么报，后来用了新的 `SIG`，报出 `2.0X`，总算提示你，那不是旧版，得上新武器对付他，机机不灵了！

使用 `ANTI` 版 `OD` 载入软件，几次确定后，停在入口，代码如下：

```
004BF000 >  60             pushad
004BF001   23EA           and     ebp, edx
004BF003   4F             dec     edi
004BF004   45             inc     ebp
004BF005   72 01          jb      short 004BF008
004BF007   F8             clc
004BF008   66:BB 008E     mov     bx, 8E00
004BF00C   0BDD           or      ebx, ebp
004BF00E   87D8           xchg    eax, ebx
004BF010   C1E8 16        shr     eax, 16
004BF013   0F8A 02000000  jpe     004BF01B
004BF019   85C6           test    esi, eax
```

选择相关的内存异常忽略选项后，直接运行 `ACP V2` 的脚本（这个网上很多，可以随处找到）；一会脚本停下，提示 OEP 到，代码停在如下：

```
0044E879   33C0           xor     eax, eax                     ; 传说中的光明之颠~OEP~
0044E87B   6A 00          push    0
0044E87D   394424 08      cmp     dword ptr [esp+8], eax
0044E881   68 00100000    push    1000
0044E886   0F94C0         sete    al
0044E889   50             push    eax
0044E88A   FF15 A4F44700  call    dword ptr [47F4A4]           ; kernel32.HeapCreate
0044E890   85C0           test    eax, eax
0044E892   A3 A8944A00    mov     dword ptr [4A94A8], eax
0044E897   74 36          je      short 0044E8CF
0044E899   E8 93FEFFFF    call    0044E731
```

这个地方知道 `VC` 程序的人，就看出，这不是 OEP，为什么，因为那是补剪去的 OEP 下面的一个 CALL 中的头，为明确 `VC` 程序 OEP 的特征，我们现在来看一个标准的 `VC++` OEP 头：

```
Microsoft Visual C++ 6.0 [Debug]
.text
.rdata
.data
.rsrc
005522F3 Baby>/$  55           push ebp          //标准 OEP   VC6
005522F4     |.  8BEC         mov ebp,esp
005522F6     |.  6A FF        push -1
005522F8     |.  68 58235800  push Babylon.00582358
005522FD     |.  68 6C5B5500  push Babylon.00555B6C               ;  SE handler installation
00552302     |.  64:A1 00000000  mov eax,dword ptr fs:[0]
00552308     |.  50           push eax
00552309     |.  64:8925 00000000  mov dword ptr fs:[0],esp
00552310     |.  83EC 58      sub esp,58
00552313     |.  53           push ebx
00552314     |.  56           push esi
00552315     |.  57           push edi                     ; ntdll.7C930738
00552316     |.  8965 E8      mov dword ptr ss:[ebp-18],esp
00552319     |.  FF15 00635700 call near dword ptr ds:[<&KERNEL32.GetVersion>]  ; kernel32.GetVersion
0055231F     |.  33D2         xor edx,edx                 ; ntdll.KiFastSystemCallRet
00552321     |.  8AD4         mov dl,ah
00552323     |.  8915 00BF5D00 mov dword ptr ds:[5DBF00],edx  ; ntdll.KiFastSystemCallRet
00552329     |.  8BC8         mov ecx,eax
0055232B     |.  81E1 FF000000 and ecx,0FF
00552331     |.  890D FCBE5D00 mov dword ptr ds:[5DBEFC],ecx
00552337     |.  C1E1 08      shl ecx,8
0055233A     |.  03CA         add ecx,edx                 ; ntdll.KiFastSystemCallRet
0055233C     |.  890D F8BE5D00 mov dword ptr ds:[5DBEF8],ecx
00552342     |.  C1E8 10      shr eax,10
00552345     |.  A3 F4BE5D00  mov dword ptr ds:[5DBEF4],eax
0055234A     |.  6A 01        push 1
0055234C     |.  E8 391D0000 call Babylon.0055408A
00552351     |.  59           pop ecx                     ; kernel32.7C816FD7
00552352     |.  85C0         test eax,eax
00552354     |.  75 08        jnz short Babylon.0055235E
00552356     |.  6A 1C        push 1C
```

再回到我们的代码；

```
0044E879   33C0           xor     eax, eax                     ; 传说中的光明之颠~OEP~
0044E87B   6A 00          push    0
0044E87D   394424 08      cmp     dword ptr [esp+8], eax
```

看堆栈：

```
0012FF44  004CE670  返回到 DiskGeni.004CE670 来自 DiskGeni.0044E879  // 第 2 句 CALL
0012FF48  00000001                                // 第 1 句 PUSH
0012FF4C  7C930228 ntdll.7C930228
```

根据我们的分析：现在的位置应该是 OEP 第一 CALL 进入的位置；从堆栈处看到，`0012FF48` 是 `1`；所以原来的代码应该是：

```
push 1
call 0044E879
```

这和那个标准 `VC` 的 OEP 代码也对上了如：（用他做参考，很方便）

```
0055234A     |.  6A 01         push 1
0055234C     |.  E8 391D0000 call Babylon.0055408A
```

从上面上看，还有很长的代码没找到，我们使用 `ALT + F9` 返回，再在 CODE 段下内存访问断，`shift + F9`，继续：此时代码停在：

```
0044ABAD   85C0           test    eax, eax
0044ABAF   75 08          jnz     short 0044ABB9
0044ABB1   6A 1C          push    1C
0044ABB3   E8 C2000000    call    0044AC7A
0044ABB8   59             pop     ecx
0044ABB9   E8 D8390000    call    0044E596
0044ABBE   85C0           test    eax, eax
0044ABC0   75 08          jnz     short 0044ABCA
0044ABC2   6A 10          push    10
0044ABC4   E8 B1000000    call    0044AC7A
0044ABC9   59             pop     ecx
0044ABCA   33F6           xor     esi, esi
0044ABCC   8975 FC        mov     dword ptr [ebp-4], esi
0044ABCF   E8 06380000    call    0044E3DA
0044ABD4   E8 60370000    call    0044E339
0044ABD9   A3 D4954A00    mov     dword ptr [4A95D4], eax
0044ABDE   E8 E9350000    call    0044E1CC
0044ABE3   A3 4C7D4A00    mov     dword ptr [4A7D4C], eax
0044ABE8   E8 B6330000    call    0044DFA3
0044ABED   E8 F9320000    call    0044DEEB
0044ABF2   E8 19240000    call    0044D010
0044ABF7   8975 D0        mov     dword ptr [ebp-30], esi
0044ABFA   8D45 A4        lea     eax, dword ptr [ebp-5C]
0044ABFD   50             push    eax
0044ABFE   FF15 34F54700  call    dword ptr [47F534]         ; kernel32.GetStartupInfoW
```

此处才是代码真正的 OEP 一节，因为从 `0044ABAD` 向上全是乱码了，所以我们要根据上面分析的来寻找那失去的部分：

根据刚才找到的两句，参照标准 OEP 头，我们来还原一下代码如下；未猜出的部分先有用 `XXXXX` 代替：

```
/ push eax
|  mov dword ptr fs:[0],esp
|  sub esp,58
|  push ebx
|  push esi
|  push edi
 \\ mov dword ptr ss:[ebp-18],esp        //这几句不变的，直接照抄
call near dword ptr ds:[XXXXXXXX]        //这句要找地址了； <&KERNEL32.GetVersion>
xor edx,edx                              //这两句不变的，直接照抄
mov dl,ah
mov dword ptr ds:[XXXXXXXX],edx          //地址 1                  ;
mov ecx,eax
and ecx,0FF
mov dword ptr ds:[XXXXXXXX],ecx          //地址 2
shl ecx,8
add ecx,edx                              ;
mov dword ptr ds:[XXXXXXXX],ecx          //地址 3
shr eax,10
mov dword ptr ds:[XXXXXXXX],eax          //地址 4
push 1                                   //这几句上面已经交待了
call 0044E879

程序停在的代码，接上就行了
0044ABAD   85C0           test    eax, eax
```



```
0044ABAF    75 08           jnz     short 0044ABB9  
0044ABB1    6A 1C           push    1C  
0044ABB3    E8 C2000000     call    0044AC7A  
0044ABB8    59              pop     ecx  
0044ABB9    E8 D8390000     call    0044E596  
0044ABBE    85C0            test    eax, eax  
0044ABC0    75 08           jnz     short 0044ABCA  
```

根据文件头的猜测，参照标准头，我们写出 `OEP` 的代码，但现最头疼的就是那几个`[xxxxxxxx]`的地址了，我们现在把他挖出来：

### 一、`call near dword ptr ds:[XXXXXXXX]`
//这句要找地址了； `<&KERNEL32.GetVersion>`

这个好办，我们把代码向下找，在 `OD` 中找到这句：

```
0044ABFE    FF15 34F54700   call    dword ptr [47F534]             ; kernel32.GetStartupInfoW  
```

很显然，`0047F000` 节是 `IAT` 存放，我们在汇编中找到这节：

```
0047F610    .  7B1D807C      dd      kernel32.LoadLibraryA  
0047F614    .  56BE807C      dd      kernel32.lstrlenA  
0047F618    .  989C807C      dd      kernel32.MultiByteToWideChar  
0047F61C    .  6EBC807C      dd      kernel32.FindResourceW  
0047F620    .  55A0807C      dd      kernel32.LoadResource  
0047F624    .  37CD807C      dd      kernel32.SetHandleCount  
0047F628    .  7A12817C      dd      kernel32.GetVersion           //这就是了  
0047F62C    .  D097807C      dd      kernel32.GetCurrentThreadId  
0047F630    .  0C01817C      dd      kernel32.GlobalAddAtomW  
0047F634    .  C74E837C      dd      kernel32.GlobalFindAtomW  
0047F638    .  C30B837C      dd      kernel32.GlobalDeleteAtom  
0047F63C    .  DDE4807C      dd      kernel32.GetModuleHandleW  
```

`<&KERNEL32.GetVersion>`的地址是 `0047F628`，那很明显，这个 `CALL` 应该是：

```
call    dword ptr [0047F628];   kernel32.GetVersion  
```

### 二、我们还有四个地址，那就是：

```
mov dword ptr ds:[XXXXXXXX],edx        //地址 1                   ;   
 mov ecx,eax  
 and ecx,0FF  
 mov dword ptr ds:[XXXXXXXX],ecx       //地址 2   
 shl ecx,8  
 add ecx,edx                                          ;   
 mov dword ptr ds:[XXXXXXXX],ecx       //地址 3    
 shr eax,10  
 mov dword ptr ds:[XXXXXXXX],eax       //地址 4    
```

看到这个真晕了。这没有什么参照，我们怎么知道他呢！

我们在 `OD` 处看到这行：

```
0044ABD9   .  A3 D4954A00   mov     dword ptr [4A95D4], eax  
0044ABDE   .  E8 E9350000   call    0044E1CC  
0044ABE3   .  A3 4C7D4A00   mov     dword ptr [4A7D4C], eax   //看这行  
0044ABE8   .  E8 B6330000   call    0044DFA3  
0044ABED   .  E8 F9320000   call    0044DEEB  
0044ABF2   .  E8 19240000   call    0044D010  
0044ABF7   .  8975 D0       mov     dword ptr [ebp-30], esi  
0044ABFA   .  8D45 A4       lea     eax, dword ptr [ebp-5C]  
0044ABFD   .  50            push    eax                         ; /pStartupinfo  
0044ABFE   .  FF15 34F54700 call    dword ptr [47F534]         ; \GetStartupInfoW  
```

根据经验，`0044ABE3` 处的地址，和最初值的地址有一差值，这个差值一般比较不变，他也会有变化，但两个地址的差别基本固定在 `0x78` 左右，但我们不会根据这个来决定上面的地址；

我们首先来看上面的四处赋值，就是四次对连续的地址给值，这从标准代码上也看到，我们直接在堆栈处看看：`[4A7D4C]`

```
004A7D4C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7D5C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7D6C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7D7C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7D8C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7D9C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7DAC  00 00 00 00 00 00 00 00 00 00 00 00 28 0A 00 00  ............(...  
004A7DBC  01 05 00 00 05 00 00 00 01 00 00 00 00 00 00 00  ........  
004A7DCC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7DDC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7DEC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7DFC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7E0C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
004A7E1C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
```

整区中只有四个地址有值，而且是连续的，这说明什么，就是前面代码刚赋值的地址，而且地址也符合规律，就是和`[4A7D4C]`地址差值差不多是 `0x78` 左右；

```
mov dword ptr ds:[004A7DC4],edx        //地址 1  最大                  ;   
 mov ecx,eax  
 and ecx,0FF  
 mov dword ptr ds:[004A7DC0],ecx       //地址 2   
 shl ecx,8  
 add ecx,edx                                          ;   
 mov dword ptr ds:[004A7DBC],ecx       //地址 3    
 shr eax,10  
 mov dword ptr ds:[004A7DB8],eax       //地址 4  最小。  
```

经过上面的过程，我们的 `OEP` 丢失的代码已经全部找回来了！

然后保存，用 `LorePE` 修正 `OEP`，`ImportREC` 修复 `IAT`！

最后运行了一下，完美修复，没能 任何问题！

--------------------------------------------------------------------------------

### 【经验总结】

谢谢大家看完，其实修复这个 `OEP` 也并不是很复杂，有时还要看运气，但有一点。针对什么语言，就要熟相关语言的文件头，对比一下，就事半功倍了！写的不好，请不要向我扔石头！欢迎大家交流！

--------------------------------------------------------------------------------

### 【版权声明】

谢谢观看, 转载请注明作者并保持文章的完整, 谢谢!

2009 年 10 月 04 日 10:55:47
