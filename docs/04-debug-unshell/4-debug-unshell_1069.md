
# telock 0.85f

> 原文：[`www.pediy.com/kssd/pediy03/forum1288.htm`](https://www.pediy.com/kssd/pediy03/forum1288.htm)

## telock 0.85f 的一点脱壳经验

by `Fpc[CCG]/6767[BCG]`

研究了一下 `telock 0.85f` 对记事本的加壳，在已知入口点的情况下是能脱掉的 :)

由于现在我还没有通过跟踪找到入口点，所以方法还没有普遍性。下面简要介绍一下。

`Load` 加壳后的记事本，下 `bpm 4010cc x`，当然需要运行 `super bpm`，拦下后把它 `Dump` 出来。在编辑 dumped 文件前先用 16 进制文件工具修改 section 的个数，原来是 `FFFF`，根据你对目标的判断改为 10 个或 20 个。用 `peditor` 继续编辑，修改入口，把多余的段删除，选 `dumped fixer` 修复段信息，保存。

用 `importREC` 修复 `importTable`，参见近几天的 `asprotect` 的精华贴。会有四个函数无法修复，记下 `ptr` 值。在程序(未脱壳的)运行时用 `softice` 的 `u` 命令观察相关位置，发现都是 `00`，可以认为这四处是垃圾信息，因此删除。把新建的 `importtable` 附加到脱壳后的文件上。运行，爽! 没有问题。

接下来的任务是如何找 `oeip`....

## 怎样找 telock 加壳程序的 OEip

by `Fpc[CCG]/6767[BCG]` @2001/09

接着前一篇，仍以记事本为对象

### 1、运行 icedump, superbpm

(thanx the guys for such beautyful tOOls!)。用 `symble Loader` 载入目标。

```
:0040C000 E9C750FFFF            jmp 004010CC      <- 入口，注意此处是被改过的
:0040C005 00CD                  add ch, cl
:0040C007 20E8                  and al, ch
:0040C009 00000000              BYTE  4 DUP(0)
```

### 2、不要乱动，任取程序空间一点，比如下：bpm 402000，用来监视解压过程

按 `F5`。

第一次，在这里停下：

```
:0040C6D4 AC                    lodsb
:0040C6D5 8D12                  lea edx, dword ptr [edx]    <- 这里
:0040C6D7 FEC0                  inc al
:0040C6D9 0401                  add al, 01
```

继续，第二次是：

```
:0040C7D9 C0C801                ror al, 01
:0040C7DC AA                    stosb
:0040C7DD 69D2A5B0CD4B          imul edx, 4BCDB0A5          <- 这里
:0040C7E3 F9                    stc
:0040C7E4 7202                  jb 0040C7E8
:0040C7E6 CD20                  int 20
```

第三次：

```
:0040C9AE AC                    lodsb
:0040C9AF EB02                  jmp 0040C9B3                <- 这里
:0040C9B1 CD20                  int 20
```

第四次、第五次、第六次：

```
:0040C9B5 2CE5                  sub al, E5
:0040C9B7 32C1                  xor al, cl
:0040C9B9 8807                  mov byte ptr [edi], al
:0040C9BB D2C8                  ror al, cl                    <- 这里
:0040C9BD 32C3                  xor al, bl
:0040C9BF 021F                  add bl, byte ptr [edi]
:0040C9C1 12D9                  adc bl, cl                    <- 这里
:0040C9C3 F6C101                test cl, 01
:0040C9C6 750F                  jne 0040C9D7
:0040C9C8 D1EB                  shr ebx, 1
:0040C9CA F7C308000000          test ebx, 00000008
:0040C9D0 7505                  jne 0040C9D7
:0040C9D2 D3C3                  rol ebx, cl
:0040C9D4 8D1CDB                lea ebx, dword ptr [ebx+8*ebx]
:0040C9D7 AA                    stosb
:0040C9D8 49                    dec ecx                       <- 这里
:0040C9D9 7FD3                  jg 0040C9AE
:0040C9DB 5F                    pop edi
```

第七次：

```
:0040CA4E C1E902                shr ecx, 02
:0040CA51 FC                    cld
:0040CA52 F3                    repz
:0040CA53 A5                    movsd                         <- 这里
:0040CA54 8BCB                  mov ecx, ebx
:0040CA56 F3                    repz
```

第八次：

```
:0040CACA 2BF0                  sub esi, eax
:0040CACC F3                    repz
:0040CACD A4                    movsb                         <- 这里
:0040CACE 5E                    pop esi
:0040CACF EB93                  jmp 0040CA64
```

### 3、好了，不要按 F5。设断点：bpx GetProcAddress。按 F5，被拦，按一次 F12 返回

```
:0040CE16 81E3FFFFFF7F          and ebx, 7FFFFFFF
:0040CE1C 53                    push ebx
:0040CE1D FFB586AE4000          push dword ptr [ebp+0040AE86]
:0040CE23 FF95F4AD4000          call dword ptr [ebp+0040ADF4]
:0040CE29 40                    inc eax                       <- 返回处
:0040CE2A 48                    dec eax
:0040CE2B 7532                  jne 0040CE5F                  <- 跳走
:0040CE2D 58                    pop eax
```

... ...向下跟

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040CE2B(C)

:0040CE5F 8907                  mov dword ptr [edi], eax
:0040CE61 58                    pop eax
:0040CE62 48                    dec eax
:0040CE63 740D                  je 0040CE72                   <- 先停下来
```

### 4、因为 eax!=0 所以不会跳，好了先关断点。看看下面的情况，下：G 40CE7E

(注意：这是为了跳出循环，在里面转圈是很烦的)：

```
:0040CE65 40                    inc eax
:0040CE66 F8                    clc
:0040CE67 668943FE              mov word ptr [ebx-02], ax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040CE70(C)

:0040CE6B 8803                  mov byte ptr [ebx], al
:0040CE6D 43                    inc ebx
:0040CE6E 3803                  cmp byte ptr [ebx], al
:0040CE70 75F9                  jne 0040CE6B
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040CE63(C)

:0040CE72 83858AAE400004        add dword ptr [ebp+0040AE8A], 4
:0040CE79 E968FEFFFF            jmp 0040CCE6
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040CDC4(C)

:0040CE7E 83C614                add esi, 00000014             <- 到这里
:0040CE81 8B959EAE4000          mov edx, dword ptr [ebp+0040AE9E]
:0040CE87 E945FDFFFF            jmp 0040CBD1                  <- 返回去了
```

### 5、好，继续看代码

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040CE87(U)

:0040CBD1 83A55EAF400000        and dword ptr [ebp+0040AF5E], 00000000
:0040CBD8 8B460C                mov eax, dword ptr [esi+0C]
:0040CBDB 83660C00              and dword ptr [esi+0C], 00000000
:0040CBDF 85C0                  test eax, eax
:0040CBE1 0F84E8020000          je 0040CECF                   <- 这里是能跳出循环的
:0040CBE7 03C2                  add eax, edx
:0040CBE9 8BD8                  mov ebx, eax
```

### 6、在上面随便哪一条指令上用命令：G 0040CECF

下面的代码可有点长：

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:0040CBC9(C), :0040CBE1(C)

:0040CECF 8BBD96AE4000          mov edi, dword ptr [ebp+0040AE96]  <- 车开到这里
```

### 7、激活 GetProcAddress ，按 F5，会拦下(这里是三次)

```
:0040CED5 85FF                  test edi, edi
:0040CED7 EB03                  jmp 0040CEDC
:0040CED9 0100                  add dword ptr [eax], eax
:0040CEDB EB74                  jmp 0040CF51
:0040CEDD 3203                  xor al, byte ptr [ebx]
:0040CEDF BD9EAE4000            mov ebp, 0040AE9E
:0040CEE4 8B85A6AE4000          mov eax, dword ptr [ebp+0040AEA6]
:0040CEEA 85C0                  test eax, eax
:0040CEEC 7422                  je 0040CF10
:0040CEEE 8B8DAAAE4000          mov ecx, dword ptr [ebp+0040AEAA]
:0040CEF4 85C9                  test ecx, ecx
:0040CEF6 7418                  je 0040CF10
:0040CEF8 03C7                  add eax, edi
:0040CEFA 0385AEAE4000          add eax, dword ptr [ebp+0040AEAE]
:0040CF00 50                    push eax
:0040CF01 51                    push ecx
:0040CF02 E817FBFFFF            call 0040CA1E
:0040CF07 83F8FF                cmp eax, FFFFFFFF
:0040CF0A 0F841BFDFFFF          je 0040CC2B
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:0040CEEC(C), :0040CEF6(C)

:0040CF10 B923010000            mov ecx, 00000123
:0040CF15 8DBD57AA4000          lea edi, dword ptr [ebp+0040AA57]
:0040CF1B 8BF7                  mov esi, edi
:0040CF1D AC                    lodsb
:0040CF1E 3206                  xor al, byte ptr [esi]
:0040CF20 AA                    stosb
:0040CF21 E2FA                  loop 0040CF1D
:0040CF23 E814000000            call 0040CF3C
:0040CF28 47                    inc edi
:0040CF29 65                    BYTE 065h
:0040CF2A 7443                  je 0040CF6F
```



:0040CF2C 7572 jne 0040CFA0   

:0040CF2E 7265 jb 0040CF95   

:0040CF30 6E outsb   

:0040CF31 7450 je 0040CF83   

:0040CF33 726F jb 0040CFA4   

:0040CF35 636573 arpl dword ptr [ebp+73], esp   

:0040CF38 7349 jnb 0040CF83   

:0040CF3A 64 BYTE 64h   

:0040CF3B 00 BYTE 00h   

* Referenced by a CALL at Address:   

|:0040CF23   

|   

:0040CF3C FFB54EAF4000 push dword ptr [ebp+0040AF4E]   

:0040CF42 FF95F4AD4000 call dword ptr [ebp+0040ADF4] <- 拦在此处   

:0040CF48 40 inc eax <- 一次 F12 返回   

:0040CF49 48 dec eax   

:0040CF4A 0F8483000000 je 0040CFD3   

:0040CF50 FFD0 call eax <- call `GetCurrentProcessID`   

:0040CF52 8BD8 mov ebx, eax   

:0040CF54 E80C000000 call 0040CF65   

:0040CF59 4F dec edi   

:0040CF5A 7065 jo 0040CFC1   

:0040CF5C 6E outsb   

:0040CF5D 50 push eax   

:0040CF5E 726F jb 0040CFCF   

:0040CF60 636573 arpl dword ptr [ebp+73], esp   

:0040CF63 7300 jnb 0040CF65   

* Referenced by a CALL at Address:   

|:0040CF54   

|   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040CF63(C)   

|   

:0040CF65 FFB54EAF4000 push dword ptr [ebp+0040AF4E]   

:0040CF6B FF95F4AD4000 call dword ptr [ebp+0040ADF4] <- 拦在此处   

:0040CF71 40 inc eax <- 返回处   

:0040CF72 48 dec eax   

:0040CF73 745E je 0040CFD3   

:0040CF75 53 push ebx   

:0040CF76 6A00 push 00000000   

:0040CF78 68FF0F1F00 push 001F0FFF   

:0040CF7D FFD0 call eax <- call `OpenProcess`   

:0040CF7F 40 inc eax   

:0040CF80 48 dec eax   

:0040CF81 7450 je 0040CFD3   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:0040CF31(C), :0040CF38(C)   

|   

:0040CF83 8BD8 mov ebx, eax   

:0040CF85 E811000000 call 0040CF9B   

:0040CF8A 56 push esi   

:0040CF8B 69727475616C50 imul esi, dword ptr [edx+74], 506C6175   

:0040CF92 726F jb 0040D003   

:0040CF94 7465 je 0040CFFB   

:0040CF96 63744578 arpl dword ptr [ebp+2*eax+78], esi   

:0040CF9A 00 BYTE 00h   

* Referenced by a CALL at Address:   

|:0040CF85   

|   

:0040CF9B FFB54EAF4000 push dword ptr [ebp+0040AF4E]   

:0040CFA1 FF95F4AD4000 call dword ptr [ebp+0040ADF4] <- 拦在此处   

:0040CFA7 40 inc eax <- 返回处   

:0040CFA8 48 dec eax   

:0040CFA9 7428 je 0040CFD3   

:0040CFAB 6A00 push 00000000   

:0040CFAD 54 push esp   

:0040CFAE 6A04 push 00000004   

:0040CFB0 6800100000 push 00001000   

:0040CFB5 FFB59EAE4000 push dword ptr [ebp+0040AE9E]   

:0040CFBB 53 push ebx   

:0040CFBC FFD0 call eax <- call `VirtualProctectEX`   

8、离出口已经很近了，所以脱这个壳的中断是：`bpx VirtualProctectEX`。绝对好用 :)   

:0040CFBE 83C404 add esp, 00000004   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040CF5A(C)   

|   

:0040CFC1 40 inc eax   

:0040CFC2 48 dec eax   

:0040CFC3 740E je 0040CFD3   

:0040CFC5 8BBD9EAE4000 mov edi, dword ptr [ebp+0040AE9E]   

:0040CFCB 037F3C add edi, dword ptr [edi+3C]   

:0040CFCE 66834F06FF or word ptr [edi+06], FFFF   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   

|:0040CF4A(C), :0040CF73(C), :0040CF81(C), :0040CFA9(C), :0040CFC3(C)   

|   

:0040CFD3 8B9DBAAE4000 mov ebx, dword ptr [ebp+0040AEBA]   

:0040CFD9 33F6 xor esi, esi <- esi=0   

:0040CFDB F7D3 not ebx <- ebx=OEip   

:0040CFDD 0BF3 or esi, ebx   

:0040CFDF 7508 jne 0040CFE9   

:0040CFE1 8D9D349B4000 lea ebx, dword ptr [ebp+00409B34]   

:0040CFE7 EB06 jmp 0040CFEF   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040CFDF(C)   

|   

:0040CFE9 039D9EAE4000 add ebx, dword ptr [ebp+0040AE9E]   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040CFE7(U)   

|   

:0040CFEF 895C24F0 mov dword ptr [esp-10], ebx   

:0040CFF3 8DBD90AD4000 lea edi, dword ptr [ebp+0040AD90]   

:0040CFF9 33C0 xor eax, eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040CF94(C)   

|   

:0040CFFB B99C020000 mov ecx, 0000029C   

:0040D000 F3 repz <- 代码清 0，所以在这里：`r ecx 0`   

:0040D001 AA stosb   

:0040D002 8DBD349B4000 lea edi, dword ptr [ebp+00409B34]   

:0040D008 B90D100000 mov ecx, 0000100D   

:0040D00D F3 repz   

:0040D00E AA stosb   

:0040D00F 66AB stosw <- 不要执行   

:0040D011 8DBD349B4000 lea edi, dword ptr [ebp+00409B34]   

:0040D017 85F6 test esi, esi   

:0040D019 7508 jne 0040D023   

:0040D01B C70733C040C3 mov dword ptr [edi], C340C033 <- 也是搞破坏   

:0040D021 EB0B jmp 0040D02E   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040D019(C)   

|   

:0040D023 C607E9 mov byte ptr [edi], E9 <- 一样的破坏   

:0040D026 47 inc edi   

:0040D027 2BDF sub ebx, edi   

:0040D029 83EB04 sub ebx, 00000004   

:0040D02C 891F mov dword ptr [edi], ebx <- 保存 OEip   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   

|:0040D021(U)   

|   

:0040D02E 8DBD41AB4000 lea edi, dword ptr [ebp+0040AB41]   

* Possible Reference to String Resource ID=00044: "?嘽"   

|   

:0040D034 B92C000000 mov ecx, 0000002C   

:0040D039 F3 repz <- 代码清 0，所以在这里：`r ecx 0`   

:0040D03A AA stosb   

:0040D03B 66AB stosw <- 不要执行   

:0040D03D EB02 jmp 0040D041   

:0040D03F 90 nop <- 这里原是花指令:CD 20，呵呵   

:0040D040 90 nop   

|   

:0040D041 61 popad <- 恢复现场   

:0040D042 FF6424D0 jmp [esp-30] <- JMP TO MOTHER!!!   

:0040D046 63EB arpl ebx, ebp   

That's all! Folks! `4dd466d0e1e0cc123afbe504da2462ca`
