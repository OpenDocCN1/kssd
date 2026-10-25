
# 深入伪 UPX 分析之一：汇编分析

> 原文：[`www.pediy.com/kssd/pediy10/71248.html`](https://www.pediy.com/kssd/pediy10/71248.html)

经过一番周折，`UPX` 这个简单的壳，我却么有搞定！_! 不得不难过了 :(

一直心情很糟糕，么有办法静下来做东西了。好长时间么来看雪了，今天碰到个很简单的 `UPX` 壳，我竟然么有搞定。

天哪~~~~，我是何等的渺小。一直想给看雪带点好东西，但是我依然么有拿得出手的东西。因为我已经很久么有好好打开 `OD` 和 `IDA` 看过。最多是随手打开随手关掉。

最近玩游戏，碰到了个东西，它有个英文名字 `GameGuard`，不知道大家听说过么，好象它便是大名顶顶的 `NP`，关于 `NP` 我会再以后的文章中说明，当然是旧版本，做为研究无所谓新旧，为了看雪也为了我和大家，我选择了旧版本。

今天和大家探讨的是 `GameGuard.des` 的脱壳问题，因为 `PEID` 告诉我说是未知，我来个深度扫描竟然是 `UPX`，和我开了个天大的玩笑。

我很熟练的 `OD`，不到 10 秒，不！是 5 秒，一个断点，两下 `F8`，于是我到了 `OEP`，`LOAD` 直接 `DUMP` 发现竟然是 `4KB`，我晕了.....

直接拿 `WINHEX`，从内存提出来 `LOADPE` 重建搞定。发现这样的脱壳不完美首先运行不起来。而且后来修复后，运行，发现也不正常，于是我用 `UPX -D` 么想到提示：

```
//-----------------------------------------------------------------------------------------------
E:\软件加密解密\脱壳工具\UPX\upx303w>upx -d gameguard.des
               Ultimate Packer for eXecutables
             Copyright (C) 1996 - 2008
UPX 3.03w       Markus Oberhumer, Laszlo Molnar & John Reiser   Apr 27th 2008
        File size         Ratio       Format      Name
   --------------------   ------   -----------   -----------
upx: gameguard.des: CantUnpackException: file is modified/hacked/protected; take
 care!!!
Unpacked 0 files.
//----------------------------------------------------------------------------------------------
```

对上面的提示只能是汗颜了。一到咱看雪，搜索一下！发现有个 `UPXFIX` 很好玩。点一下修复，竟然告诉我不是 `UPX` 壳，到底那个说的对呢？？我们来看看入口：

```
//----------------------------------------------------------------------------------------------
00422AF0 >   31FF          xor     edi, edi
00422AF2     74 06        je      short 00422AFA
00422AF4     61          popad
00422AF5  - E9 4A4D5030   jmp     30927844
00422AFA     5A          pop     edx
00422AFB     BA 7D000000 mov     edx, 7D
00422B00     807C24 08 01 cmp     byte ptr [esp+8], 1
00422B05     E9 00000000 jmp     00422B0A
00422B0A     60          pushad
00422B0B     BE 00504100 mov     esi, 00415000
00422B10     31FF        xor     edi, edi
00422B12     74 06       je      short 00422B1A
00422B14     61          popad
00422B15  - E9 4A4D5030   jmp     30927864
00422B1A     8DBE 00C0FEFF lea     edi, dword ptr [esi+FFFEC000]
00422B20     31C9        xor     ecx, ecx
00422B22     74 06       je      short 00422B2A
00422B24     61          popad
00422B25  - E9 4A4D5030   jmp     30927874
//-----------------------------------------------------------------------------------------------
```

大家看看是不是 `UPX`，但是和 `UPX` 脱壳方法很是一样。估计是被修改过的 `UPX` 了。管它呢~~~好好看看。

### 1:

```
//-----------------------------------------------------------------------------------------------
00422AF0 >   31FF          xor     edi, edi                     ; edi =0
00422AF2     74 06        je      short 00422AFA               ; if edi == 0 ->jmp
00422AF4     61          popad
00422AF5  - E9 4A4D5030   jmp     30927844
00422AFA     5A          pop     edx                            ; save edx
00422AFB     BA 7D000000 mov     edx, 7D                        ; edx = 7D
00422B00     807C24 08 01 cmp     byte ptr [esp+8], 1            ; ????
00422B05     E9 00000000 jmp     00422B0A
00422B0A     60          pushad
00422B0B     BE 00504100 mov     esi, 00415000                  ; esi = 00415000
00422B10     31FF        xor     edi, edi                         ; edi = 0
00422B12     74 06       je      short 00422B1A                 ; if edi ==0 ->jmp
00422B14     61          popad
00422B15  - E9 4A4D5030   jmp     30927864
00422B1A     8DBE 00C0FEFF lea     edi, dword ptr [esi+FFFEC000]    ; edi = esi - 14000
00422B20     31C9        xor     ecx, ecx                         ; ecx =0
00422B22     74 06       je      short 00422B2A                 ; if ecx ==0 ->jmp
00422B24     61          popad
00422B25  - E9 4A4D5030   jmp     30927874
```

### 2: 需要说明的是 `ADC` 是与 `CF` 相加

```
//--------------------------------------------------------------------------------------------
00422B78     61          popad
00422B79     57          push    edi
00422B7A     83CD FF     or      ebp, FFFFFFFF                  ; ebp = -1
00422B7D     EB 0B       jmp     short 00422B8A
00422B7F     90          nop
00422B80     8A06        mov     al, byte ptr [esi]              ; get byte
00422B82     46          inc     esi
00422B83     8807        mov     byte ptr [edi], al              ; set .code address 将计算后的结果保存到代码段.
00422B85     47          inc     edi
00422B86     01DB        add     ebx, ebx                         ; ebx + ebx
00422B88     75 07       jnz     short 00422B91                 ; ebx + ebx != 0 ->jmp
00422B8A     8B1E        mov     ebx, dword ptr [esi]            ; ESI = .rdata address
00422B8C     83EE FC     sub     esi, -4                          ; esi =esi -4
00422B8F     11DB        adc     ebx, ebx                         ; ebx + ebx
00422B91  ^ 72 ED         jb      short 00422B80
00422B93     B8 01000000 mov     eax, 1                          ; eax =1
00422B98     01DB        add     ebx, ebx                         ; ebx + ebx
00422B9A     75 07       jnz     short 00422BA3                 ; ebx + ebx != 0 ->jmp
00422B9C     8B1E        mov     ebx, dword ptr [esi]
00422B9E     83EE FC     sub     esi, -4                          ; esi = esi -4
00422BA1     11DB        adc     ebx, ebx                         ; ebx+ebx
00422BA3     11C0        adc     eax, eax                         ; eax + eax
00422BA5     01DB        add     ebx, ebx                         ; ebx + ebx
00422BA7  ^ 73 EF         jnb     short 00422B98
00422BA9     75 09       jnz     short 00422BB4
00422BAB     8B1E        mov     ebx, dword ptr [esi]
00422BAD     83EE FC     sub     esi, -4
00422BB0     11DB        adc     ebx, ebx
00422BB2  ^ 73 E4         jnb     short 00422B98
00422BB4     31C9        xor     ecx, ecx                         ; ecx = 0
00422BB6     83E8 03     sub     eax, 3                           ; eax -= 3
00422BB9     72 0D       jb      short 00422BC8
00422BBB     C1E0 08     shl     eax, 8                           ; eax << 8
00422BBE     8A06        mov     al, byte ptr [esi]
00422BC0     46          inc     esi                              ; esi ++
00422BC1     83F0 FF     xor     eax, FFFFFFFF                    ; eax ^= -1
00422BC4     74 74       je      short 00422C3A                 ; if eax = 0 -> jmp
00422BC6     89C5        mov     ebp, eax                         ; ebp = eax
00422BC8     01DB        add     ebx, ebx                         ; ebx +ebx
00422BCA     75 07       jnz     short 00422BD3                 ; !=0 ->jmp
00422BCC     8B1E        mov     ebx, dword ptr [esi]
00422BCE     83EE FC     sub     esi, -4                          ; esi -4
00422BD1     11DB        adc     ebx, ebx                         ; ebx+ebx
00422BD3     11C9        adc     ecx, ecx                         ; ecx +ecx
00422BD5     01DB        add     ebx, ebx
00422BD7     75 07       jnz     short 00422BE0
00422BD9     8B1E        mov     ebx, dword ptr [esi]
00422BDB     83EE FC     sub     esi, -4                          ; esi -4
00422BDE     11DB        adc     ebx, ebx                         ; ebx +ebx
00422BE0     11C9        adc     ecx, ecx                         ; ecx +ecx
00422BE2     75 20       jnz     short 00422C04                 ; ecx != 0->jmp
00422BE4     41          inc     ecx                              ; ecx ++
00422BE5     01DB        add     ebx, ebx                         ; ebx+ebx
```



```
00422BE7  75 07         jnz     short 00422BF0
00422BE9  8B1E          mov     ebx, dword ptr [esi]
00422BEB  83EE FC      sub     esi, -4                     ; esi -4
00422BEE  11DB         adc     ebx, ebx                    ; ebx+ebx
00422BF0  11C9         adc     ecx, ecx                    ; ecx+ecx
00422BF2  01DB         add     ebx, ebx                    ; ebx+ebx
00422BF4  ^ 73 EF      jnb     short 00422BE5
00422BF6  75 09        jnz     short 00422C01
00422BF8  8B1E          mov     ebx, dword ptr [esi]
00422BFA  83EE FC      sub     esi, -4                     ; esi -4
00422BFD  11DB         adc     ebx, ebx                    ; ebx+ebx
00422BFF  ^ 73 E4      jnb     short 00422BE5
00422C01  83C1 02      add     ecx, 2                      ; ecx += 2
00422C04  81FD 00F3FFFF cmp    ebp, -0D00                  ; 比较 EBP 是否为-0D00
00422C0A  83D1 01      adc     ecx, 1                      ; ecx+cf +1
00422C0D  8D142F       lea     edx, dword ptr [edi+ebp]    ; 代码段计算当前地址+EBP
00422C0D  8D142F       lea     edx, dword ptr [edi+ebp]
00422C10  83FD FC      cmp     ebp, -4                     ; ?-4
00422C13  76 0F        jbe     short 00422C24              ; 如果小于-4 不执行,否则就执行下面
00422C15  8A02         mov     al, byte ptr [edx]          ; 计算后的代码地址
00422C17  42           inc     edx                         ; edx ++
00422C18  8807         mov     byte ptr [edi], al          ; 保存到当前地址
00422C1A  47           inc     edi                         ; 指针+1
00422C1B  49           dec     ecx                         ; 循环计数-1
00422C1C  ^ 75 F7      jnz     short 00422C15              ; 不为 0 就继续循环,实际就是 COPY
00422C1E  ^ E9 63FFFFFF jmp    00422B86
```

### 3:

```
//--------------------------------------------------------------------------------------------
00422C24  8B02         mov     eax, dword ptr [edx]        ; edx = code address
00422C26  83C2 04      add     edx, 4                      ; edx += 4
00422C29  8907         mov     dword ptr [edi], eax        ; edi =code address
00422C2B  83C7 04      add     edi, 4                      ; edi += 4
00422C2E  83E9 04      sub     ecx, 4                      ; ecx -= 4
00422C31  ^ 77 F1      ja      short 00422C24              ; ecx > 0 ->loop
00422C33  01CF         add     edi, ecx                    ; edi += ecx
```

经过上面三段后，代码已经被解开了，现在需要做什么呢？我们来看看就知道了，毕竟我还在看。可以看到的是，前面的代码计算不是很难的，如果两段一样的代码它就直接复制一下了。估计应该是 API 了吧。

### 4:

```
//-------------------------------------------------------------------------------------------
00422C3A  5E           pop     esi
00422C3B  89F7         mov     edi, esi                    ; esi = code begin address
00422C3D  B9 37060000  mov     ecx, 637                    ; ecx = 637
00422C42  8A07         mov     al, byte ptr [edi]          ; edi = code address
00422C44  47           inc     edi                         ; edi ++
00422C45  2C E8        sub     al, 0E8                     ; al - e8
00422C47  3C 01        cmp     al, 1                       ; al > 1 ->loop
00422C49  ^ 77 F7      ja      short 00422C42              ; 寻找是 CALL 或 JMP 指令
00422C4B  803F 07      cmp     byte ptr [edi], 7           ; [edi] ? 7
00422C4E  ^ 75 F2      jnz     short 00422C42              ; !=7 ->jmp
00422C50  8B07         mov     eax, dword ptr [edi]        ; call xxxx read xxxx->eax
00422C52  8A5F 04      mov     bl, byte ptr [edi+4]        ; [edi + 4] ->bl
00422C55  66:C1E8 08   shr     ax, 8                       ; ax >> 8
00422C59  C1C0 10      rol     eax, 10                     ; eax ROL 10
00422C5C  86C4         xchg    ah, al
00422C5E  29F8         sub     eax, edi                    ; eax -edi
00422C60  80EB E8      sub     bl, 0E8                     ; bl -e8
00422C63  01F0         add     eax, esi                    ; eax + esi
00422C65  8907         mov     dword ptr [edi], eax        ; eax ->call xxxx write xxxx
00422C67  83C7 05      add     edi, 5                      ; edi + 5
00422C6A  89D8         mov     eax, ebx                    ; eax =ebx
00422C6C  ^ E2 D9      loopd   short 00422C47              ; 计算地址修改 CALL 和 JMP
```

### 5:

```
//--------------------------------------------------------------------------------------------------
00422C6E  8DBE 00000200 lea   edi, dword ptr [esi+20000]  ; esi + 20000 esi = code begin address 到了输入表函数名部分
00422C74  8B07         mov     eax, dword ptr [edi]        ; eax = [edi]
00422C76  09C0         or      eax, eax
00422C78  74 45        je      short 00422CBF              ; eax = 0 -> jmp
00422C7A  8B5F 04      mov     ebx, dword ptr [edi+4]
00422C7D  8D8430 602B0200 lea eax, dword ptr [eax+esi+22B60] ; dll name address
00422C84  01F3         add     ebx, esi
00422C86  50           push    eax                         ; dll name
00422C87  83C7 08      add     edi, 8
00422C8A  FF96 D82B0200 call  dword ptr [esi+22BD8]       ; load dll
00422C90  95           xchg    eax, ebp
00422C91  8A07         mov     al, byte ptr [edi]
00422C93  47           inc     edi                         ; edi ++
00422C94  08C0         or      al, al                      ; al | al
00422C96  ^ 74 DC      je      short 00422C74
00422C98  89F9         mov     ecx, edi                    ; edi = fun address
00422C9A  79 07        jns     short 00422CA3
00422C9C  0FB707       movzx   eax, word ptr [edi]
00422C9F  47           inc     edi
00422CA3  57           push    edi                         ; edi = fun name
00422CA4  48           dec     eax                         ; eax --
00422CA5  F2:AE        repne   scas byte ptr es:[edi]
00422CA7  55           push    ebp                         ; dll HANDLE
00422CA8  FF96 DC2B0200 call  dword ptr [esi+22BDC]
00422CAE  09C0         or      eax, eax
00422CB0  74 07        je      short 00422CB9
00422CB2  8903         mov     dword ptr [ebx], eax        ; ebx = IAT address
00422CB4  83C3 04      add     ebx, 4
00422CB7  ^ EB D8      jmp     short 00422C91              ; 循环处理函数
```

上面的代码是依次加载 DLL，并获得输入表的函数地址，组建 IAT 表。下面就回到了 OEP。

上面已经基本上分析了算法汇编。总体上来讲就是先解密代码段，再休整 CALL 和 JMP，再就是组建 IAT 表等了。

```c
//比较好的是,我无意中往下拉了一下,看到了一个 EXE 头,我想应该是原来的文件头,这样再写脱壳工具的时候
//是十分有用的.我很高兴^\_^
```

```
004219ED  00 00 00 00 00 00 50 45 00 00 4C 01 04 00 AF BD  ......PE..L.
004219FD  F6 42 00 00 00 00 00 00 00 00 E0 00 0F 01 0B 01  ........?
00421A0D  06 00 00 40 01 00 00 C0 00 00 00 00 00 00 C7 BB  ..@..?.....腔
```

从它的工作流程来讲，现在已经到了 OEP，和 UPX 来说应该是基本一致的，我还么有研究出到底是因为什么 UPX -D 无法脱壳，只有自己写个专版脱壳机了。

么什么技术性的文章，发出来给大家们笑笑，也算是给大家一个乐子，我本来就很白菜，而且进步有慢。高人们就看看喜欢的就收下吧，不喜欢的就闪了就是了。千万别开口 XXXX，我最近很受不了，前段时间也好点了，最近，实在是很差。说不定以后很少发文章了，大家都支持一下吧~~~~~~~~

先发出来给大家新鲜一下，我整理一下就写个静态脱壳工具，真的最讨厌的就是 `GameGuard` 和 `StarForce`。后者已经被我整的差不多了，就前着还在继续。`GameGuard` 好笑的是，千万别用 `OD` 直接调试，不然一下子屏幕黑掉，然后听到硬盘磁头"仓"的一声，落在了你的扇面上，么关系的按一下电源键就可以重新开启。经过测试，么有无法开启电脑的现象。

后来我用了 `VC` 直接挂着调试，很有意思的是，在 `VC` 调试的时候 `GameGuard` 一直保持更新。N 久后么反应，于是我关掉 NP 再关掉 `VC` 还是 N 久的更新状态，不更新只显示连接成功.....!\_! 。我还以为掉线了，打开游戏主页，结果游戏主页打不开，再问朋友是否能打开，结果朋友打的开，于是我想 `IP` 被封了。后来挂了个网站代理，很顺利的打开了主页。



所以结论就是调试`NP`是会被封`IP`的........朋友们一定要注意撒~~~~~~~~~~~~~~~!!!!

-By EasyStudy

2008.08.23 凌晨
