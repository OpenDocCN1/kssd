
# DotFix NiceProtect 2.x-3.x (Max Protection) 脱壳分析

> 原文：[`www.pediy.com/kssd/pediy10/57604.html`](https://www.pediy.com/kssd/pediy10/57604.html)

**【文章标题】** `DotFix NiceProtect 2.x-3.x (Max Protection) 脱壳分析`  
**【文章作者】** `KuNgBiM`/`[CCG]`  
**【作者邮箱】** `kungbim@163.com`  
**【脱壳目标】** `MetaTrader 4 Client Terminal` 主程序  
**【下载地址】** `http://www.metaquotes.net/files/mt4setup.exe`  
**【程序简介】** 国外的一款外汇交易软件  
**【加壳方式】** `DotFix NiceProtect 2.x - 3.x`  
**【保护选项】** `Max Protection`  
**【使用工具】** `OllyICE`, `LordPE`, `ImportREC`  
**【脱壳平台】** 三元钱一张的非正版 `WinXPsp2`  
**【作者声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

--------------------------------------------------------------------------------

## 【脱壳内容】

`OllyICE` 载入目标程序，忽略所有异常。

```
00BD20CC > /E9 FF000000              jmp terminal.00BD21D0                   ; EP
00BD20D1  |60                       pushad
00BD20D2  |8B7424 24               mov esi,dword ptr ss:[esp+24]
00BD20D6  |8B7C24 28               mov edi,dword ptr ss:[esp+28]
00BD20DA  |FC                       cld
00BD20DB  |B2 80                    mov dl,80
00BD20DD  |33DB                     xor ebx,ebx
00BD20DF  |A4                       movsb
00BD20E0  |B3 02                    mov bl,2
00BD20E2  |E8 6D000000              call terminal.00BD2154
00BD20E7 ^|73 F6                    jnb short terminal.00BD20DF
00BD20E9  |33C9                     xor ecx,ecx
00BD20EB  |E8 64000000              call terminal.00BD2154
00BD20F0  |73 1C                    jnb short terminal.00BD210E
00BD20F2  |33C0                     xor eax,eax
00BD20F4  |E8 5B000000              call terminal.00BD2154
00BD20F9  |73 23                    jnb short terminal.00BD211E
00BD20FB  |B3 02                    mov bl,2
00BD20FD  |41                       inc ecx
00BD20FE  |B0 10                    mov al,10
```

### 【避开调试器检测及代码解压缩】

（虽然现在很多修改版的 `OD` 都能跑，不过既然是脱壳，那么就一定要避开它检测！）

**命令窗**：`bp FindWindowA`

**Shift+F9 运行**：

*过时的调试器检测办法*

```
0012FBB8   00BACA18   /CALL 到 FindWindowA 来自 terminal.00BACA16
0012FBBC   00BAC9CA   |Class = "OLLYDBG"
0012FBC0   00BAC986   指向下一个 SEH 记录的指针
0012FBC4   00BAC02E   SE 处理程序
0012FBC8   00BAB317   返回到 terminal.00BAB317 来自 terminal.00BABCDC
0012FBCC   00BAAC9B   返回到 terminal.00BAAC9B 来自 terminal.00BAB30B
0012FBD0   00BAA2FF   返回到 terminal.00BAA2FF 来自 terminal.00BAA96F
0012FBD4   7C9237BF   返回到 ntdll.7C9237BF
77D2E591 > 8BFF                   mov edi,edi                            ; 中断后取消函数断点
77D2E593  55                     push ebp
77D2E594  8BEC                   mov ebp,esp
77D2E596  33C0                   xor eax,eax
77D2E598  50                     push eax
77D2E599  FF75 0C                push dword ptr ss:[ebp+C]
77D2E59C  FF75 08                push dword ptr ss:[ebp+8]
77D2E59F  50                     push eax
77D2E5A0  50                     push eax
77D2E5A1  E8 4CFFFFFF            call USER32.77D2E4F2
77D2E5A6  5D                     pop ebp
77D2E5A7  C2 0800                retn 8                                   ; 在此 F2 下断
```

**命令窗**：`bc FindWindowA`

**Shift+F9 继续运行，中断后取消断点**：

```
00BACA18   85C0                  test eax,eax                         ; 返回到这里，继续 F7
00BACA1A   75 05                 jnz short terminal.00BACA21
00BACA1C   E8 70060000            call terminal.00BAD091
00BACA21   E8 0B000000            call terminal.00BACA31
00BACA26   3036                  xor byte ptr ds:[esi],dh
00BACA28   2037                  and byte ptr ds:[edi],dh
00BACA2A   76 77                 jbe short terminal.00BACAA3
00BACA2C   6B21 29               imul esp,dword ptr ds:[ecx],29
00BACA2F   2900                  sub dword ptr ds:[eax],eax
00BACA31   EB 01                 jmp short terminal.00BACA34
00BACA33 - E9 5AEB01EA            jmp EABCB592
00BACA38   52                     push edx
00BACA39   4A                     dec edx
00BACA3A   EB 01                 jmp short terminal.00BACA3D
```

**5 次 F7**：

```
00BAD0A1  /EB 01                  jmp short terminal.00BAD0A4           ; 注意观察 ESP 变化，使用 ESP 定律中断 2 次即可
00BAD0A3  |EB 5A                  jmp short terminal.00BAD0FF
00BAD0A5   EB 01                  jmp short terminal.00BAD0A8
00BAD0A7   EA 524AEB06 72DD       jmp far DD72:06EB4A52
00BAD0AE ^ 73 E5                  jnb short terminal.00BAD095
00BAD0B0   1F                     pop ds
00BAD0B1   A8 EB                  test al,0EB
00BAD0B3   01EA                   add edx,ebp
00BAD0B5   B9 0A000000            mov ecx,0A
00BAD0BA   EB 03                  jmp short terminal.00BAD0BF
00BAD0BC   1838                   sbb byte ptr ds:[eax],bh
00BAD0BE   DEEB                   fsubp st(3),st
00BAD0C0   01EA                   add edx,ebp
```

**命令窗**：`hr esp`

**Shift+F9 中断后**：

```
00BAD0A5  /EB 01                  jmp short terminal.00BAD0A8           ; 第 1 次中断，继续
00BAD0A7  |EA 524AEB06 72DD       jmp far DD72:06EB4A52
00BAD0AE ^ 73 E5                  jnb short terminal.00BAD095
00BAD0B0   1F                     pop ds
00BAD0B1   A8 EB                  test al,0EB
00BAD0B3   01EA                   add edx,ebp
00BAD0B5   B9 0A000000            mov ecx,0A
00BAD0BA   EB 03                  jmp short terminal.00BAD0BF
00BAD0A9   4A                     dec edx                              ; 第 2 次中断后来到这里
00BAD0AA   EB 06                  jmp short terminal.00BAD0B2
00BAD0AC ^ 72 DD                  jb short terminal.00BAD08B
00BAD0AE ^ 73 E5                  jnb short terminal.00BAD095
00BAD0B0   1F                     pop ds
00BAD0B1   A8 EB                  test al,0EB
00BAD0B3   01EA                   add edx,ebp
00BAD0B5   B9 0A000000            mov ecx,0A
00BAD0BA   EB 03                  jmp short terminal.00BAD0BF
00BAD0BC   1838                   sbb byte ptr ds:[eax],bh
00BAD0BE   DEEB                   fsubp st(3),st
00BAD0C0   01EA                   add edx,ebp
```

取消所有断点，`Alt+M` 打开内存镜像，在 `PE header` 段设置访问断点，`Shift+F9` 运行中断后，再次在 `CODE` 区段设置访问断点，`Shift+F9` 运行后即可到达程序内部，当然这时的 `IAT` 也是没加密的了。

```
00541DBF   33DB                    xor ebx,ebx                           ; 程序内部
00541DC1   3BC3                    cmp eax,ebx
00541DC3   8906                    mov dword ptr ds:[esi],eax
00541DC5   74 62                   je short terminal.00541E29
00541DC7   57                      push edi
00541DC8   8B3D 08C25500           mov edi,dword ptr ds:[55C208]           ; kernel32.GetProcAddress
00541DCE   68 88985800             push terminal.00589888                ; ASCII "OpenThemeData"
00541DD3   50                      push eax
00541DD4   FFD7                    call edi
00541DD6   8946 04                 mov dword ptr ds:[esi+4],eax
00541DD9   8B06                    mov eax,dword ptr ds:[esi]
00541DDB   68 78985800             push terminal.00589878                ; ASCII "CloseThemeData"
00541DE0   50                      push eax
00541DE1   FFD7                    call edi
00541DE3   8B0E                    mov ecx,dword ptr ds:[esi]
00541DE5   68 64985800             push terminal.00589864                ; ASCII "DrawThemeBackground"
00541DEA   51                      push ecx
00541DEB   8946 08                 mov dword ptr ds:[esi+8],eax
00541DEE   FFD7                    call edi
00541DF0   8B16                    mov edx,dword ptr ds:[esi]
00541DF2   68 48985800             push terminal.00589848                ; ASCII "DrawThemeParentBackground"
00541DF7   52                      push edx
```

【以上步骤我们完全可以用脚本来完成】

**代码**:



# DotFix NiceProtect 辅助脚本

Script written by KuNgBiM.  
**Debugging options:** Tick all items in `OllyDbg`'s Debugging Options-Exceptions.  
**Test Environment:** `OllyDbg` 1.10, `ODBGScript` 1.65, `WinXP`.

## 脚本代码

```
/*
Script written by KuNgBiM
Script: DotFix NiceProtect 辅助脚本
Debugging options: Tick all items in OllyDbg's Debugging Options-Exceptions
Test Environment : OllyDbg 1.10, ODBGScript 1.65, WinXP
*/

#log

var tmp

// 相当于“bp FindWindowA”不过插件不能直接支持这样的下断命令

gpa "FindWindowA","user32.dll"
cmp $RESULT,0
je error

// 寻找“retn 8”特征准备返回

find $RESULT,#C20800#
cmp $RESULT,0
je error
bp $RESULT
esto
bc eip

// 按 5 次 F7

sti
sti
sti
sti
sti

// ESP 定律(两次)

mov tmp,esp
bphws tmp,"r"
esto
esto
bphwc tmp

msg "请用 Alt+M 打开内存镜像，在 PE header 段设置访问断点,Shift+F9 运行中断后,再次在 CODE 区段设置访问断点,Shift+F9 运行后即可到达程序内部，当然这时的 IAT 也是没加密的了。"

msg "提示: 修补 StolenCode 及 VM code 由你自己去完成! ^\_^"

ret

error:
msg "脚本错误!"
ret
```

## 修补 StolenCode

查看程序函数可知程序是 `Visual C++ 6.0 (MFC)` 编译，所以我们可以搜索特征代码来完成 `OEP` 查找：

`Ctrl+S`:

`A1 ?? ?? ?? 00 89 45 94 8D 45 94 50 FF 35 ?? ?? ?? 00 8D 45 9C 50 8D 45 90 50 8D 45 A0 50`

找到：

```
0055B6D2    DE16                 ficom word ptr ds:[esi]                ; 这里开始 StolenCode(OEP)
0055B6D4    2E:59                pop ecx
0055B6D6  ^ 70 91                jo short terminal.0055B669
0055B6D8    69DB 89A25FC4        imul ebx,ebx,C45FA289
0055B6DE    B1 C4                mov cl,0C4
0055B6E0    68 9AA3D7BA         push BAD7A39A
0055B6E5    46                   inc esi
0055B6E6    F3:                  prefix rep:
0055B6E7    2B33                 sub esi,dword ptr ds:[ebx]
0055B6E9    C6                   ???                                     ; 未知命令
0055B6EA    F4                   hlt
0055B6EB    D952 27              fst dword ptr ds:[edx+27]
0055B6EE    D959 85              fstp dword ptr ds:[ecx-7B]
0055B6F1    AE                   scasb
0055B6F2    1B51 AB             sbb edx,dword ptr ds:[ecx-55]
0055B6F5    57                   push edi
0055B6F6    1AACD1 8007EB40      sbb ch,byte ptr ds:[ecx+edx*8+40EB0780]
0055B6FD    E5 01                in eax,1
0055B6FF    C0BA CF7DD900 89     sar byte ptr ds:[edx+D97DCF],89
0055B706  ^ EB FA               jmp short terminal.0055B702
0055B708    97                   xchg eax,edi
0055B709    14 98                adc al,98
0055B70B    50                   push eax
0055B70C    CC                   int3
0055B70D    1C CF                sbb al,0CF
0055B70F    01F5                 add ebp,esi
0055B711    2A4406 FB            sub al,byte ptr ds:[esi+eax-5]
0055B715    1AB2 3D16CFC2         sbb dh,byte ptr ds:[edx+C2CF163D]
0055B71B    B9 D6272F5E         mov ecx,5E2F27D6
0055B720    6B2A 28             imul ebp,dword ptr ds:[edx],28
0055B723    AA                   stosb
0055B724    1D 86F5D51D          sbb eax,1DD5F586
0055B729    40                   inc eax
0055B72A    DE1E                 ficomp word ptr ds:[esi]
0055B72C    80A3 495D897D 22     and byte ptr ds:[ebx+7D895D49],22
0055B733    DC27                 fsub qword ptr ds:[edi]
0055B735    8B80 6CBE3285        mov eax,dword ptr ds:[eax+8532BE6C]
0055B73B    B2 2B                mov dl,2B
0055B73D  ^ 70 85                jo short terminal.0055B6C4
0055B73F    C2 0212              retn 1202
0055B742    6BF0 1A             imul esi,eax,1A
0055B745    26:61                popad
0055B747    75 0C                jnz short terminal.0055B755             ; 这里以上的代码都被 StolenCode 了
0055B749    68 54B85500          push terminal.0055B854
0055B74E    FF15 B4CC5500       call dword ptr ds:[55CCB4]              ; msvcrt.__setusermatherr
0055B754    59                   pop ecx
0055B755    E8 E8000000          call terminal.0055B842
0055B75A    68 94905700         push terminal.00579094
0055B75F    68 90905700         push terminal.00579090
0055B764    E8 D3000000          call terminal.0055B83C                 ; jmp 到 msvcrt._initterm
0055B769    A1 7C975900         mov eax,dword ptr ds:[59977C]           ; 找到这里，然后向上看
0055B76E    8945 94             mov dword ptr ss:[ebp-6C],eax
0055B771    8D45 94             lea eax,dword ptr ss:[ebp-6C]
0055B774    50                   push eax
0055B775    FF35 78975900       push dword ptr ds:[599778]
0055B77B    8D45 9C             lea eax,dword ptr ss:[ebp-64]
0055B77E    50                   push eax
0055B77F    8D45 90             lea eax,dword ptr ss:[ebp-70]
0055B782    50                   push eax
0055B783    8D45 A0             lea eax,dword ptr ss:[ebp-60]
0055B786    50                   push eax
0055B787    FF15 C0CC5500       call dword ptr ds:[55CCC0]              ; msvcrt.__getmainargs
```

现在我们必须修复 `StolenCode`，程序才能正常运行。

随便找一个由 `Visual C++ 6.0 (MFC)` 编译的程序作为补代码对照程序，我这里所使用的是程序目录内自带的另一个程序 `MetaEditor.exe`。

从 `0055B732` 开始，到 `0055B7B3` 填充以下的对照代码：

```
55 8B EC 6A FF 68 80 73 45 00 68 AC BF 44 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 68 53 56 57 89 65 E8 33 DB 89 5D FC 6A 02 FF 15 D4 DB 44 00 59 83 0D 30 27 46 00 FF 83 0D 34 27 46 00 FF FF 15 D8 DB 44 00 8B 0D 14 27 46 00 89 08 FF 15 DC DB 44 00 8B 0D 10 27 46 00 89 08 A1 E0 DB 44 00 8B 00 A3 2C 27 46 00 E8 16 01 00 00 39 1D 60 17 46 00
```

填充完毕后，建立新 `EIP`：

```
0055B6D2    55                   push ebp                              ; 在此处建立新 EIP
0055B6D3    8BEC                 mov ebp,esp
0055B6D5    6A FF                push -1
0055B6D7    68 80734500         push terminal.00457380                ; ① push 00575F40
0055B6DC    68 ACBF4400         push terminal.0044BFAC                ; ② push 0055B858
0055B6E1    64:A1 00000000      mov eax,dword ptr fs:[0]
0055B6E7    50                   push eax
0055B6E8    64:8925 00000000    mov dword ptr fs:[0],esp
0055B6EF    83EC 68             sub esp,68
0055B6F2    53                   push ebx
0055B6F3    56                   push esi
0055B6F4    57                   push edi
0055B6F5    8965 E8             mov dword ptr ss:[ebp-18],esp
0055B6F8    33DB                 xor ebx,ebx
0055B6FA    895D FC             mov dword ptr ss:[ebp-4],ebx
0055B6FD    6A 02                push 2
0055B6FF    FF15 D4DB4400       call dword ptr ds:[44DBD4]             ; ③call dword ptr ds:[0055CCB8]
0055B705    59                   pop ecx
0055B706    830D 30274600 FF    or dword ptr ds:[462730],FFFFFFFF       ; ④or dword ptr ds:[00599790],FFFFFFFF
0055B70D    830D 34274600 FF    or dword ptr ds:[462734],FFFFFFFF       ; ⑤or dword ptr ds:[00599794],FFFFFFFF
0055B714    FF15 D8DB4400       call dword ptr ds:[44DBD8]             ; ⑥call dword ptr ds:[0055CCA8]
0055B71A    8B0D 14274600      mov ecx,dword ptr ds:[462714]           ; ⑦mov ecx,dword ptr ds:[00599774]
0055B720    8908                 mov dword ptr ds:[eax],ecx
0055B722    FF15 DCDB4400       call dword ptr ds:[44DBDC]             ; ⑧call dword ptr ds:[0055CCAC]
0055B728    8B0D 10274600      mov ecx,dword ptr ds:[462710]           ; ⑨mov ecx,dword ptr ds:[00599770]
0055B72E    8908                 mov dword ptr ds:[eax],ecx
0055B730    A1 E0DB4400        mov eax,dword ptr ds:[44DBE0]           ; ⑩mov eax,dword ptr ds:[0055CCB0]
0055B735    8B00                 mov eax,dword ptr ds:[eax]
0055B737    A3 2C274600        mov dword ptr ds:[46272C],eax           ; ⑾mov dword ptr ds:[0059978C],eax
0055B73C    E8 16010000        call terminal.0055B857                 ; ⑿call 0055B857
0055B741    391D 60174600      cmp dword ptr ds:[461760],ebx           ; ⒀cmp dword ptr ds:[00599788],ebx
```



```
0055B747    75 0C                 jnz short terminal.0055B755

0055B749    68 54B85500           push terminal.0055B854

0055B74E    FF15 B4CC5500         call dword ptr ds:[55CCB4]                 ; msvcrt.__setusermatherr

0055B754    59                    pop ecx

0055B755    E8 E8000000           call terminal.0055B842

0055B75A    68 94905700           push terminal.00579094                     ; 【⑤】暴露

0055B75F    68 90905700           push terminal.00579090                     ; 【④】暴露

0055B764    E8 D3000000           call terminal.0055B83C                     ; jmp 到 msvcrt._initterm

0055B769    A1 7C975900           mov eax,dword ptr ds:[59977C]

0055B76E    8945 94               mov dword ptr ss:[ebp-6C],eax

0055B771    8D45 94               lea eax,dword ptr ss:[ebp-6C]

0055B774    50                    push eax

0055B775    FF35 78975900         push dword ptr ds:[599778]

0055B77B    8D45 9C               lea eax,dword ptr ss:[ebp-64]

0055B77E    50                    push eax

0055B77F    8D45 90               lea eax,dword ptr ss:[ebp-70]

0055B782    50                    push eax

0055B783    8D45 A0               lea eax,dword ptr ss:[ebp-60]

0055B786    50                    push eax

0055B787    FF15 C0CC5500         call dword ptr ds:[55CCC0]                 ; msvcrt.__getmainargs

0055B78D    68 8C905700           push terminal.0057908C

0055B792    68 00905700           push terminal.00579000

0055B797    E8 A0000000           call terminal.0055B83C                     ; jmp 到 msvcrt._initterm

0055B79C    83C4 24               add esp,24

0055B79F    A1 C4CC5500           mov eax,dword ptr ds:[55CCC4]

0055B7A4    8B30                  mov esi,dword ptr ds:[eax]

0055B7A6    8975 8C               mov dword ptr ss:[ebp-74],esi

0055B7A9    803E 22               cmp byte ptr ds:[esi],22

0055B7AC    75 3A                 jnz short terminal.0055B7E8

0055B7AE    46                    inc esi

0055B7AF    8975 8C               mov dword ptr ss:[ebp-74],esi

0055B7B2    8A06                  mov al,byte ptr ds:[esi]

0055B7B4    3AC3                  cmp al,bl

0055B7B6    74 04                 je short terminal.0055B7BC

0055B7B8    3C 22                 cmp al,22

0055B7BA    75 F2                 jnz short terminal.0055B7AE

0055B7BC    803E 22               cmp byte ptr ds:[esi],22

0055B7BF    75 04                 jnz short terminal.0055B7C5

0055B7C1    46                    inc esi

0055B7C2    8975 8C               mov dword ptr ss:[ebp-74],esi

0055B7C5    8A06                  mov al,byte ptr ds:[esi]

0055B7C7    3AC3                  cmp al,bl

0055B7C9    74 04                 je short terminal.0055B7CF

0055B7CB    3C 20                 cmp al,20

0055B7CD    76 F2                 jbe short terminal.0055B7C1

0055B7CF    895D D0               mov dword ptr ss:[ebp-30],ebx

0055B7D2    8D45 A4               lea eax,dword ptr ss:[ebp-5C]

0055B7D5    50                    push eax

0055B7D6    FF15 70C25500         call dword ptr ds:[55C270]                 ; kernel32.GetStartupInfoA

0055B7DC    F645 D0 01            test byte ptr ss:[ebp-30],1

0055B7E0    74 11                 je short terminal.0055B7F3

0055B7E2    0FB745 D4             movzx eax,word ptr ss:[ebp-2C]

0055B7E6    EB 0E                 jmp short terminal.0055B7F6

0055B7E8    803E 20               cmp byte ptr ds:[esi],20

0055B7EB    76 D8                 jbe short terminal.0055B7C5

0055B7ED    46                    inc esi

0055B7EE    8975 8C               mov dword ptr ss:[ebp-74],esi

0055B7F1    EB F5                 jmp short terminal.0055B7E8

0055B7F3    6A 0A                 push 0A

0055B7F5    58                    pop eax

0055B7F6    50                    push eax

0055B7F7    56                    push esi

0055B7F8    53                    push ebx

0055B7F9    53                    push ebx

0055B7FA    FF15 BCC15500         call dword ptr ds:[55C1BC]                 ; kernel32.GetModuleHandleA

0055B800    50                    push eax

0055B801    E8 6D010000           call terminal.0055B973

0055B806    8945 98               mov dword ptr ss:[ebp-68],eax

0055B809    50                    push eax

0055B80A    FF15 C8CC5500         call dword ptr ds:[55CCC8]                 ; msvcrt.exit

0055B810    8B45 EC               mov eax,dword ptr ss:[ebp-14]             ; 【①】暴露

0055B813    8B08                  mov ecx,dword ptr ds:[eax]

0055B815    8B09                  mov ecx,dword ptr ds:[ecx]

0055B817    894D 88               mov dword ptr ss:[ebp-78],ecx

0055B81A    50                    push eax

0055B81B    51                    push ecx

0055B81C    E8 15000000           call terminal.0055B836                     ; jmp 到 msvcrt._XcptFilter

0055B821    59                    pop ecx

0055B822    59                    pop ecx

0055B823    C3                    retn
```

下面修复说明请见上面的程序标注。

## ①

在对照程序`MetaEditor.exe`中所调用的地址为：

```
00457380  FFFFFFFF
00457384  0044BF64  MetaEdit.0044BF64
00457388  0044BF78  MetaEdit.0044BF78

FF FF FF FF 64 BF 44 00
```

那么我们这个程序所调用的地址应该为：`0055B810`

数据窗口中搜索 `Ctrl+S`:

```
FF FF FF FF 10 B8 55 00
```

找到：

```
00575F40  FFFFFFFF
00575F44  0055B810  terminal.0055B810
00575F48  0055B824  terminal.0055B824
```

`①`为：`push 00575F40`

## ②③⑥⑧⑩

没什么难度，直接找到该函数即可：

*   `MSVCRT._except_handler3` ;②
*   `msvcrt.__set_app_type` ;③
*   `msvcrt.__p__fmode` ;⑥
*   `msvcrt.__p__commode` ;⑧
*   `msvcrt._adjust_fdiv` ;⑩

`②`为：`push 0055B858`
`③`为：`call dword ptr ds:[0055CCB8]`
`⑥`为：`call dword ptr ds:[0055CCA8]`
`⑧`为：`call dword ptr ds:[0055CCAC]`
`⑩`为：`mov eax,dword ptr ds:[0055CCB0]`

分析对照程序`MetaEditor.exe`中发现`④`⑤的内存地址与【④】【⑤】的内存地址对应：

`④`为：`or dword ptr ds:[00599790],FFFFFFFF`
`⑤`为：`or dword ptr ds:[00599794],FFFFFFFF`

`⑦`=⑤-0x20=`00599774`
`⑦`为：`mov ecx,dword ptr ds:[00599774]`

`⑨`=④-0x20=`00599770`
`⑨`为：`mov ecx,dword ptr ds:[00599770]`

`⑾`=④-0x4=`0059978C`
`⑾`为：`mov dword ptr ds:[0059978C],eax`

## ⒀

分析对照程序`MetaEditor.exe`中发现：

```
0044BFA8    33C0                  xor eax,eax
0044BFAA    C3                    retn
0044BFAB    C3                    retn                                  ; 调用此处
0044BFAC    FF25 D0DB4400         jmp dword ptr ds:[<&MSVCRT._except_handler3>]    ; msvcrt._except_handler3
0044BFB2    FF25 ECDC4400         jmp dword ptr ds:[<&MSVCRT._controlfp>]           ; msvcrt._controlfp
```

那么该程序就应该调用：

```
0055B854    33C0                  xor eax,eax
0055B856    C3                    retn
0055B857    C3                    retn                                  ; 调用此处
0055B858    FF25 DCCC5500         jmp dword ptr ds:[<&msvcrt._except_handler3>]    ; msvcrt._except_handler3
0055B85E    FF25 68CD5500         jmp dword ptr ds:[<&msvcrt._controlfp>]           ; msvcrt._controlfp
```

`⒀`为：`call 0055B857`

## ⒁

数据窗口中搜索 `Ctrl+S`:(找到最后一组)

```
00 00 00 00 01 00 00 00 00 00 00 00
```

`⒁`为：

`cmp dword ptr ds:[00599788],ebx`

## 修补代码汇总（二进制）

```
55 8B EC 6A FF 68 40 5F 57 00 68 58 B8 55 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 68 53 56 57 89 65 E8 33 DB 89 5D FC 6A 02 FF 15 B8 CC 55 00 59 83 0D 90 97 59 00 FF 83 0D 94 97 59 00 FF FF 15 A8 CC 55 00 8B 0D 74 97 59 00 89 08 FF 15 AC CC 55 00 8B 0D 70 97 59 00 89 08 A1 B0 CC 55 00 8B 00 A3 8C 97 59 00 E8 16 01 00 00 39 1D 88 97 59 00
```

## 抓取及修复

以上的`StolenCode`已经全部修补完毕，所以这时我们可以直接用`LordPE`抓取，`ImportREC`修复即可。

*   `OEP=0015B6D2`
*   `RVA=0015C000`
*   `SIZE=000010B4`

脱壳后的程序运行正常！脱壳完毕！



【VM code 处理】

此部分还未弄懂，暂时略过。o(∩_∩)o

【加密选项总结】

代码:

```
该程序所使用到的保护选项有：

[Section options]
Rename sections to: .data

[pack protected options]
Compress resources
Optimize an rebuild exe file

[Protection options]
Use scrambling
Protect original entry point
Encrypt code section
Convert stolen byte to VM
Use anti-tracing protection
Use anti-debug protection

[Protection method]
Stolen Bytes Protection
```

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `DotFix NiceProtect 2.x-3.x (Max Protection) 脱壳分析.rar` [需要下载附件，请到论坛上下载：[`bbs.pediy.com/showthread.php?t=57604`](http://bbs.pediy.com/showthread.php?t=57604)]  

 |
