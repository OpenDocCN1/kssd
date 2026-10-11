
# 轻轻松松学开车 6.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6913.htm`](https://www.pediy.com/kssd/pediy06/pediy6913.htm)

【标 题】写给新手 - 浅谈脱壳方法

【作 者】`xIkUg[CCG][BCG][DFCG][DCM][CZG][D.4s]`

【日 期】2004-12-18

【使用工具】`OllyDBG`, `ImportREC`

【主 页】`http://bbs.xp-program.com`

【平 台】`WinXP sp1`

【目标程序】轻轻松松学开车 6.6

【下载地址】`http://www.sharebank.com.cn/soft/soft_view.php?id=10506`

【作者声明】只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【正 文】

我很少写破文，最近看到一些新手常问查壳工具查不出是什么壳，未知壳如何脱...我这里就简单的用一个示例程序谈谈脱壳方法。算是给新手打打气。

一般这种没有公开的壳不会太猛（除了 `hying`, `vcasm`, `jingulong` 等这些圈内人的壳哈），大多是自己土制的壳，有的还伪装一下，致使查壳工具查不出或查出什么 `Aspack`, `UPX` 的壳。碰到这种壳不用太怕，认真跟踪分析一下一般都没多大问题。

一般加壳后的程序运行时大多是用各种方法 `AntiDebugger`，其中夹杂很多花指令和垃圾指令，然后解码，又是一堆 `AntiDebugger`，可能又解码，恢复 `IAT`，又是一堆 `AntiDebugger`...最后再跳到程序真正的 `OEP` 处，对付这种壳一般的方法是先找到 `OEP`，再修复 `IAT`

在壳跳到 `OEP` 的时候一般是个跨段跳转，跳转方法有很多，如 `JMP XXXXXXXX`, `JMP [XXXXXXXX]`, `JE [XXXXXXXX]`, `PUSH XXXXXXXX RET`, `CALL XXXXXXXX`, `CALL [XXXXXXXX]`, `CALL EAX`...还有的是在 `SEH` 中直接指定 `EIP` 等等，找到这个跳转地址，然后在相应地址上设上断点，`F9` 运行，就到达 `OEP` 处了，此时就是最好的 `Dump` 时机

修复 `IAT` 一般用 `ImportREC`，填写好 `OEP` 的 `RVA`，先点 `IATAutoSearch`，再点 `GetImport`，如果找到的 API 全部有效就可以点 `Fix Dump` 了，如果有无效的就需要修复，先点 `Show Invalid`，再对这些无效的 API 点右键，共有 3 个修复级别，分别是 `Disasm`, `HOOK`, `Trap Flag`，可以一个一个试，全部 API 有效或只有一两个 API 无效时就可以点 `Fix Dump` 了，如果修复不了就只有用手工修复了。

手工修复 `IAT` 的方法是，一般壳在运行的时候会恢复 `IAT` 或破坏 `IAT`，你可以在恢复 `IAT` 后或破坏 `IAT` 之前 `DUMP` 出完好的 `IAT`，再用 16 进制编辑器，把完整的 `IAT` 复制到 `Dump` 出来的文件中，多做做练习就不难了

还是看看目标程序"轻轻松松学开车 6.6"，用 `PEiD` 查一下，`ASPack 2.12` -> `Alexey Solodovnikov`

用 `OD` 载入，提示入口点在代码外部（以后别再问如何判断程序是否加壳了，用 `OD` 载入如果有这个提示就表示程序是加了壳的），点确定，问是否分析，点否，来到入口处

```
006AF001 D> 60           pushad                 ; 壳代码入口
006AF002   E8 03000000    call DrvStudy.006AF00A ; 有花指令，F7 跟进
006AF007  - E9 EB045D45    jmp 45C7F4F7
006AF00C   55           push ebp
006AF00D   C3           retn
006AF00E   E8 01000000    call DrvStudy.006AF014
006AF013   EB 5D          jmp short DrvStudy.006AF072
```

在 `006AF002` 处 `call DrvStudy.006AF00A`，代码中没有 `006AF00A`，是因为花指令让 `OD` 错误的分析了代码，用 `F7` 跟进就能来到 `006AF00A` 处，像这种近距离的 `Call`，其实是变形的 `Jmp`，都要用 `F7` 跟进，如果 `F8` 跳过的话可以会碰到里面的陷阱

```
006AF00A   5D           pop ebp              ; DrvStudy.006AF007，跳到这里
006AF00B   45           inc ebp              ; ebp + 1
006AF00C   55           push ebp             ; 变形跳转到 ebp 的地址处
006AF00D   C3           retn
```

来到

```
006AF008  /EB 04        jmp short DrvStudy.006AF00E      ; 来到这里
006AF00A  |5D           pop ebp
006AF00B  |45           inc ebp                         ; ebp + 1
006AF00C  |55           push ebp                        ; 变形跳转到 ebp 的地址处
006AF00D  |C3           retn
006AF00E  \E8 01000000  call DrvStudy.006AF014          ; 变形跳转，F7 跟进
006AF014   5D           pop ebp
006AF015   BB EDFFFFFF  mov ebx,-13
006AF01A   03DD         add ebx,ebp
006AF01C   81EB 00F02A00 sub ebx,2AF000                 ; 代码重定位
006AF022   83BD 22040000 00  cmp dword ptr ss:[ebp+422],0 ; 变量 ebp+422 是存放的基址
006AF029   899D 22040000  mov dword ptr ss:[ebp+422],ebx
006AF02F  /0F85 65030000 jnz DrvStudy.006AF39A          ; 这里不跳
006AF035  |8D85 2E040000  lea eax,dword ptr ss:[ebp+42E] ; "kernel32.dll"
006AF03B  |50           push eax                    ; 参数入栈
006AF03C  |FF95 4D0F0000 call dword ptr ss:[ebp+F4D]   ; kernel32.GetModuleHandleA
006AF042  |8985 26040000 mov dword ptr ss:[ebp+426],eax ; kernel32.dll 的句柄存入 ebp+426 处
006AF048   8BF8         mov edi,eax                 ; kernel32.77E40000
006AF04A   8D5D 5E      lea ebx,dword ptr ss:[ebp+5E]  ; "VirtualAlloc"
006AF04D   53           push ebx
006AF04E   50           push eax                    ; Kernel32.dll 句柄
006AF04F   FF95 490F0000 call dword ptr ss:[ebp+F49]  ; GetProcAddress
006AF055   8985 4D050000 mov dword ptr ss:[ebp+54D],eax ; 把 VirtualAlloc 的函数地址存入 ebp+54D 处
006AF05B   8D5D 6B      lea ebx,dword ptr ss:[ebp+6B]  ; "VirtualFree"
006AF05E   53           push ebx
006AF05F   57           push edi                    ; Kernel32.dll 句柄入栈
006AF060   FF95 490F0000 call dword ptr ss:[ebp+F49]  ; GetProcAddress
006AF066   8985 51050000 mov dword ptr ss:[ebp+551],eax ; 把 VirtualFree 的函数地址存入 ebp+511 处
006AF06C   8D45 77      lea eax,dword ptr ss:[ebp+77]  ; 跳转地址载入 eax
006AF06F   FFE0         jmp eax                     ; 跳到 eax 指向的地方
006AF08A   8B9D 31050000 mov ebx,dword ptr ss:[ebp+531] ; 来到这里，ebx=0
006AF090   0BDB         or ebx,ebx
006AF092  /74 0A       je short DrvStudy.006AF09E   ; ebx==0，跳
006AF094  |8B03        mov eax,dword ptr ds:[ebx]
006AF096  |8785 35050000 xchg dword ptr ss:[ebp+535],eax
006AF09C  |8903        mov dword ptr ds:[ebx],eax
006AF09E  \8DB5 69050000 lea esi,dword ptr ss:[ebp+569] ; 跳到这里，esi 指向一标志
006AF0A4   833E 00      cmp dword ptr ds:[esi],0
006AF0A7  /0F84 21010000 je DrvStudy.006AF1CE        ; 不为 0，不跳
006AF0AD   6A 04        push 4
006AF0AF   68 00100000  push 1000
006AF0B4   68 00180000  push 1800
006AF0B9   6A 00        push 0
006AF0BB   FF95 4D050000 call dword ptr ss:[ebp+54D]  ; 分配内存
```

`C` 语言描述为

```
VirtualAlloc(NULL, 0x1800, MEM_COMMIT, PAGE_READWRITE);
```

```
006AF0C7   8B46 04      mov eax,dword ptr ds:[esi+4]
006AF0CA   05 0E010000  add eax,10E
006AF0CF   6A 04        push 4
006AF0D1   68 00100000  push 1000
006AF0D6   50           push eax
006AF0D7   6A 00        push 0
006AF0D9   FF95 4D050000 call dword ptr ss:[ebp+54D]  ; 再次分配内存，eax 为所需内存大小
006AF0DF   8985 52010000 mov dword ptr ss:[ebp+152],eax ; 分配的第 2 块内存锯柄存入 ebp+152 处
006AF0E5   56           push esi
006AF0E6   8B1E         mov ebx,dword ptr ds:[esi]
006AF0E8   039D 22040000 add ebx,dword ptr ss:[ebp+422]
006AF0EE   FFB5 56010000 push dword ptr ss:[ebp+156]    ; 0x401000
006AF0F4   FF76 04      push dword ptr ds:[esi+4]
006AF0F7   50           push eax
006AF0F8   53           push ebx
006AF0F9   E8 6E050000  call DrvStudy.006AF66C        ; 解压代码
006AF0FE   B3 01        mov bl,1
006AF100   80FB 00      cmp bl,0
006AF103   75 5E        jnz short DrvStudy.006AF163
006AF105   FE85 EC000000 inc byte ptr ss:[ebp+EC]
006AF10B   8B3E         mov edi,dword ptr ds:[esi]
006AF10D   03BD 22040000 add edi,dword ptr ss:[ebp+422]
006AF113   FF37         push dword ptr ds:[edi]        ; 保护 0x401000 处的代码
006AF115   C607 C3      mov byte ptr ds:[edi],0C3      ; 把 0x401000 处的代码改为 RET
```



### 解码过程分析

执行以下汇编代码序列：

```asm
006AF118     FFD7                 call edi                   ; 垃圾调用  
006AF11A     8F07                 pop dword ptr ds:[edi]       ; 恢复 0x401000 处的代码  
```

单步调试至 `006AF12D`：

```asm
006AF12D     0BC9                 or ecx,ecx  
006AF12F     74 2E                je short DrvStudy.006AF15F  
006AF131     78 2C                js short DrvStudy.006AF15F  
006AF133     AC                   lods byte ptr ds:[esi]  
006AF134     3C E8                cmp al,0E8  
006AF136     74 0A                je short DrvStudy.006AF142  
006AF138     EB 00                jmp short DrvStudy.006AF13A  
006AF13A     3C E9                cmp al,0E9  
006AF13C     74 04                je short DrvStudy.006AF142  
006AF13E     43                   inc ebx  
006AF13F     49                   dec ecx  
006AF140     ^ EB EB              jmp short DrvStudy.006AF12D  
006AF142     8B06                 mov eax,dword ptr ds:[esi]   ; 在这里设置一个断点  
```

寻找 `opcode` 为 `e8` 或 `e9` 的位置。运行后断点触发，此时 `ebx=1FE`，`eax=[esi]` 的内容，随后比较 `esi` 指向的数据。

```asm
006AF144     /EB 00               jmp short DrvStudy.006AF146  
006AF146     \\803E 36            cmp byte ptr ds:[esi],36     ; 比较第 2 个字节的 opcode 是否为 36  
006AF149     ^ 75 F3              jnz short DrvStudy.006AF13E  ; 是就不跳, 这里为 36 不会跳  
006AF14B     24 00                and al,0  
006AF14D     C1C0 18              rol eax,18                   ; 解码  
006AF150     2BC3                 sub eax,ebx                  ; 解码  
006AF152     8906                 mov dword ptr ds:[esi],eax   ; 解码后的代码存入 esi 指向的地方  
006AF154     83C3 05              add ebx,5  
006AF157     83C6 04              add esi,4  
006AF15A     83E9 05              sub ecx,5  
006AF15D     ^ EB CE              jmp short DrvStudy.006AF12D  ; 向上跳  
```

判断上述代码为解码逻辑，`C30000` 处的代码将被解码。在 `006AF15D` 的下一条指令设置断点并运行：

```asm
006AF15F     5B                   pop ebx                        ; 断下, 解码完毕  
006AF160     5E                   pop esi  
006AF161     59                   pop ecx  
006AF162     58                   pop eax  
006AF163     EB 08                jmp short DrvStudy.006AF16D    ; 跳  
006AF16D     8BC8                 mov ecx,eax                    ; 来到这里  
006AF16F     8B3E                 mov edi,dword ptr ds:[esi]     ; 代码段偏移  
006AF171     03BD 22040000        add edi,dword ptr ss:[ebp+422] ; 基地址+偏移, 指向代码段  
006AF177     8BB5 52010000        mov esi,dword ptr ss:[ebp+152] ; 解码后的正确代码地址  
006AF17D     C1F9 02              sar ecx,2  
006AF180     F3:A5                rep movs dword ptr es:[edi],dword >; 填充正确的代码  
006AF182     8BC8                 mov ecx,eax  
006AF184     83E1 03              and ecx,3  
006AF187     F3:A4                rep movs byte ptr es:[edi],byte pt>; 填充正确的代码  
006AF189     5E                   pop esi  
006AF18A     68 00800000          push 8000  
006AF18F     6A 00                push 0  
006AF191     FFB5 52010000        push dword ptr ss:[ebp+152]  
006AF197     FF95 51050000        call dword ptr ss:[ebp+551]    ; VirtualFree  
006AF19D     83C6 08              add esi,8  
006AF1A0     833E 00              cmp dword ptr ds:[esi],0       ; 解码完成?没完成就跳  
006AF1A3     ^ 0F85 1EFFFFFF      jnz DrvStudy.006AF0C7          ; 又跳到上面  
```

确定 `006AF0C7` - `006AF1A3` 均为解码代码。在 `006AF1A3` 处的下一条指令设置断点并运行：

```asm
006AF1A9     68 00800000          push 8000                        ; 断下  
006AF1AE     6A 00                push 0  
006AF1B0     FFB5 56010000        push dword ptr ss:[ebp+156]  
006AF1B6     FF95 51050000        call dword ptr ss:[ebp+551]      ; VirtualFree  
```

内存已释放。

```asm
006AF1BC     8B9D 31050000        mov ebx,dword ptr ss:[ebp+531]   ; ebx=0  
006AF1C2     0BDB                 or ebx,ebx  
006AF1C4     74 08                je short DrvStudy.006AF1CE       ; ebx==0,跳  
006AF1C6     8B03                 mov eax,dword ptr ds:[ebx]  
006AF1C8     8785 35050000        xchg dword ptr ss:[ebp+535],eax  
006AF1CE     8B95 22040000        mov edx,dword ptr ss:[ebp+422]   ; 跳到这里, ebx=0x400000, 基地址  
006AF1D4     8B85 2D050000        mov eax,dword ptr ss:[ebp+52D]   ; eax=0x400000  
006AF1DA     2BD0                 sub edx,eax  
006AF1DC     74 79                je short DrvStudy.006AF257       ; ecx==0, 跳  
```

### 恢复 IAT

开始恢复导入地址表（IAT）：

```asm
006AF257     8B95 22040000        mov edx,dword ptr ss:[ebp+422]   ; 跳到这里, edx=0x400000, 基地址  
006AF25D     8BB5 41050000        mov esi,dword ptr ss:[ebp+541]   ; esi=0  
006AF263     0BF6                 or esi,esi  
006AF265     74 11                je short DrvStudy.006AF278       ; esi==0, 跳  
006AF267     03F2                 add esi,edx  
006AF269     AD                   lods dword ptr ds:[esi]  
006AF26A     0BC0                 or eax,eax  
006AF26C     74 0A                je short DrvStudy.006AF278  
006AF26E     03C2                 add eax,edx  
006AF270     8BF8                 mov edi,eax  
006AF272     66:AD                lods word ptr ds:[esi]  
006AF274     66:AB                stos word ptr es:[edi]  
006AF276     ^ EB F1              jmp short DrvStudy.006AF269  
006AF278     BE 00001E00          mov esi,1E0000                   ; 跳到这里, esi=1e0000  
006AF27D     8B95 22040000        mov edx,dword ptr ss:[ebp+422]   ; edx=基地址  
006AF283     03F2                 add esi,edx                      ; esi=51e0000  
006AF285     8B46 0C              mov eax,dword ptr ds:[esi+C]      ; 指向下一个 dll 文件名的 RVA  
006AF288     85C0                 test eax,eax                      ; 不为空就不跳  
006AF28A     0F84 0A010000        je DrvStudy.006AF39A  
006AF290     03C2                 add eax,edx                       ; 得到下一个 dll 文件名  
006AF292     8BD8                 mov ebx,eax  
006AF294     50                   push eax  
006AF295     FF95 4D0F0000        call dword ptr ss:[ebp+F4D]       ; 取 kernel32.dll 的句柄  
006AF29B     85C0                 test eax,eax                      ; 成功, 跳  
006AF29D     75 07                jnz short DrvStudy.006AF2A6  
006AF29F     53                   push ebx  
006AF2A0     FF95 510F0000        call dword ptr ss:[ebp+F51]  
006AF2A6     8985 45050000        mov dword ptr ss:[ebp+545],eax    ; 跳到这里, kernel32.dll 句柄存入 ebp+545  
006AF2AC     C785 49050000 00000000 mov dword ptr ss:[ebp+549],0     ; [ebp+549]=0  
006AF2B6     8B95 22040000        mov edx,dword ptr ss:[ebp+422]    ; edx=基地址  
006AF2BC     8B06                 mov eax,dword ptr ds:[esi]        ; eax=0  
006AF2BE     85C0                 test eax,eax  
006AF2C0     75 03                jnz short DrvStudy.006AF2C5       ; eax==0, 不跳  
006AF2C2     8B46 10              mov eax,dword ptr ds:[esi+10]     ; IAT 的 FirThunk 基地址  
006AF2C5     03C2                 add eax,edx  
006AF2C7     0385 49050000        add eax,dword ptr ss:[ebp+549]    ; 指向下 1 个 API  
006AF2CD     8B18                 mov ebx,dword ptr ds:[eax]  
006AF2CF     8B7E 10              mov edi,dword ptr ds:[esi+10]  
006AF2D2     03FA                 add edi,edx  
006AF2D4     03BD 49050000        add edi,dword ptr ss:[ebp+549]  
006AF2DA     85DB                 test ebx,ebx  
006AF2DC     0F84 A2000000        je DrvStudy.006AF384  
```



```
006AF2E2     F7C3 00000080             test ebx,80000000

006AF2E8     75 04                   jnz short DrvStudy.006AF2EE

006AF2EA     03DA                   add ebx,edx

006AF2EC     43                      inc ebx                        ; 取函数名
006AF2ED     43                      inc ebx                        ; 取函数名

006AF2EE     53                      push ebx

006AF2EF     81E3 FFFFFFFF          and ebx,7FFFFFFF

006AF302     53                      push ebx                       ; 函数名入栈

006AF2F6     FFB5 45050000          push dword ptr ss:[ebp+545]     ; kernel32.dll 句柄入栈

006AF302     FF95 490F0000          call dword ptr ss:[ebp+F49]    ; GetProcAddress

006AF302     85C0                   test eax,eax

006AF304     5B                      pop ebx

006AF305     75 6F                  jnz short DrvStudy.006AF376    ; 跳

取 API 函数地址成功后跳到下面的地方

006AF376     8907                   mov dword ptr ds:[edi],eax     ; API 地址存入 edi 指向地址

006AF378     8385 49050000 04      add dword ptr ss:[ebp+549],4   ; 取下一个 API

006AF37F     E9 32FFFFFF            jmp DrvStudy.006AF2B6          ; 跳

006AF384     8906                   mov dword ptr ds:[esi],eax

006AF386     8946 0C               mov dword ptr ds:[esi+C],eax

006AF389     8946 10               mov dword ptr ds:[esi+10],eax

006AF38C     83C6 14               add esi,14

006AF38F     8B95 22040000         mov edx,dword ptr ss:[ebp+422]

006AF395     E9 EBFEFFFF           jmp DrvStudy.006AF285          ; 跳去恢复下个导入 dll 中的函数
```

这个大循环完成后 `IAT` 就恢复完成了。这个 `IAT` 是没有被破坏的，根据一般经验，壳在运行时都会有这么一个大循环来恢复 `IAT`。

有的是破坏，因此当在一个循环中碰到 `GetModuleHandleA`、`LoadLibraryA`、`GetProcAddress` 时就要小心了，记下它。

有可能 `ImportREC` 找不回 API 函数的时候，需要手工修复 `IAT` 时就要用到它。

循环完后，来到这里：

```
006AF39A     B8 08CF1C00           mov eax,1CCF08                 ; OEP 的 RVA
006AF39F     50                    push eax
006AF3A0     0385 22040000        add eax,dword ptr ss:[ebp+422]   ; OEP 值
006AF3A6     59                    pop ecx
006AF3A7     0BC9                 or ecx,ecx
006AF3A9     8985 A8030000       mov dword ptr ss:[ebp+3A8],eax   ; 修改 [ebp+3A8] 处的代码
006AF3AF     61                    popad
006AF3B0     75 08                jnz short DrvStudy.006AF3BA      ; 跳
006AF3B2     B8 01000000          mov eax,1
006AF3B7     C2 0C00              retn 0C
006AF3BA     68 08CF5C00          push DrvStudy.005CCF08         ; 变形跳转
006AF3BF     C3                    retn                           ; ^_^ 飞向光明之颠
```

这里就是典型的 Delphi 程序的 `OEP` 处代码：

```
005CCF08     55                    push ebp                       ; 停在这里
005CCF09     8BEC                 mov ebp,esp
005CCF0B     83C4 F0              add esp,-10
005CCF0E     53                    push ebx
005CCF0F     B8 80CA5C00          mov eax,DrvStudy.005CCA80
005CCF14     E8 B3A7E3FF          call DrvStudy.004076CC
```

现在就可在 `005CCF08` 处 Dump 程序了。用 OD 的 `OllyDump` 插件，Dump 出程序，启动 `ImportREC`，填入 `OEP` 的 `RVA` 值。

先点 `IATAutoSearch`，再点 `GetImport`，找到的 API 全部有效，点 `FixDump`，修复 `IAT` 指针。

运行程序，提示系统数据错误，这是因为程序有自校验，发现自己被脱壳后拒绝运行。去除程序的自校验我就不说了... 用爆破的方法就能搞定...

### 总结

本文讲述了脱壳的一般方法。新手面对未知壳先不要怕，自己试着跟踪分析一下，多做练习才会有进步。

`4dd466d0e1e0cc123afbe504da2462ca`
