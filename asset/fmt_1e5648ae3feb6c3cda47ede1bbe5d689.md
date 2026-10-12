
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
