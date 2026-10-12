
004CA157     83C6 04                add esi,4

004CA15A     83E9 05                sub ecx,5

004CA15D   ^ EB CE                  jmp short CHAOSHI.004CA12D

004CA15F     5B                     pop ebx

004CA160     5E                     pop esi

004CA161     59                     pop ecx

004CA162     58                     pop eax

004CA163     EB 08                  jmp short CHAOSHI.004CA16D

                                ====>F4 到这里

004CA16D     8BC8                   mov ecx,eax

004CA16F     8B3E                   mov edi,dword ptr ds:\[esi\]

004CA171     03BD 22040000         add edi,dword ptr ss:\[ebp+422\]

004CA177     8BB5 52010000         mov esi,dword ptr ss:\[ebp+152\]

004CA17D     C1F9 02               sar ecx,2

004CA180     F3:A5                 rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]

004CA182     8BC8                   mov ecx,eax

004CA184     83E1 03               and ecx,3

004CA187     F3:A4                 rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]

004CA189     5E                     pop esi

004CA18A     68 00800000           push 8000

004CA18F     6A 00                  push 0

004CA191     FFB5 52010000         push dword ptr ss:\[ebp+152\]

004CA197     FF95 51050000         call dword ptr ss:\[ebp+551\]

004CA19D     83C6 08               add esi,8

004CA1A0     833E 00               cmp dword ptr ds:\[esi\],0

004CA1A3   ^ 0F85 1EFFFFFF        jnz CHAOSHI.004CA0C7

                                ====>不要上跳！F4 下去

004CA1A9     68 00800000           push 8000

004CA1AE     6A 00                  push 0

004CA1B0     FFB5 56010000         push dword ptr ss:\[ebp+156\]

004CA1B6     FF95 51050000         call dword ptr ss:\[ebp+551\]

004CA1BC     8B9D 31050000         mov ebx,dword ptr ss:\[ebp+531\]

004CA1C2     0BDB                   or ebx,ebx

004CA1C4     74 08                  je short CHAOSHI.004CA1CE

                                ====>跳

004CA1C6     8B03                   mov eax,dword ptr ds:\[ebx\]

004CA1C8     8785 35050000         xchg dword ptr ss:\[ebp+535\],eax

004CA1CE     8B95 22040000         mov edx,dword ptr ss:\[ebp+422\]

004CA1D4     8B85 2D050000         mov eax,dword ptr ss:\[ebp+52D\]

004CA1DA     2BD0                   sub edx,eax

004CA1DC     74 79                  je short CHAOSHI.004CA257

                                ====>跳

004CA257     8B95 22040000         mov edx,dword ptr ss:\[ebp+422\]                   ; CHAOSHI.00400000

004CA25D     8BB5 41050000         mov esi,dword ptr ss:\[ebp+541\]

004CA263     0BF6                   or esi,esi

004CA265     74 11                  je short CHAOSHI.004CA278

                                ====>跳

004CA278     BE E4850000           mov esi,85E4

004CA27D     8B95 22040000         mov edx,dword ptr ss:\[ebp+422\]

004CA283     03F2                   add esi,edx

004CA285     8B46 0C                mov eax,dword ptr ds:\[esi+C\]

004CA288     85C0                   test eax,eax

004CA28A     0F84 0A010000         je CHAOSHI.004CA39A

                                ====>可以向下看：发现 004CA3AF 处是 POPAD！

                                ====>G 004CA39A 可以跳过下面的循环，到达入口点！

004CA39A     B8 6E3A0000           mov eax,3A6E

                                ====>OEP 的偏移值

004CA39F     50                     push eax

004CA3A0     0385 22040000         add eax,dword ptr ss:\[ebp+422\]

                                ====>EAX\=00003A6E + 00400000=00403A6E

004CA3A6     59                     pop ecx

004CA3A7     0BC9                   or ecx,ecx

004CA3A9     8985 A8030000         mov dword ptr ss:\[ebp+3A8\],eax

                                ====>EAX\=00403A6E 放到\[004CA3BB\]处

004CA3AF     61                     popad

                                ====>这个 POPAD

004CA3B0     75 08                  jnz short CHAOSHI.004CA3BA

004CA3B2     B8 01000000           mov eax,1

004CA3B7     C2 0C00               retn 0C

004CA3BA     68 00000000           push 0

                                ====>00403A6E 覆盖这个值

004CA3BF     C3                     retn

                                ====>跳到入口点！

———————————————————————

00403A6E     55                     push ebp

                                ====>在这儿用 LordPE 转存调试进程

00403A6F     8BEC                   mov ebp,esp

00403A71     6A FF                  push -1

00403A73     68 30814000           push CHAOSHI.00408130

00403A78     68 585A4000           push CHAOSHI.00405A58

重新运行程序，运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `00003A6E`，修复刚才脱壳的文件。

—————————————————————————————————

## 二、破解

程序运行时在 `C:WINDOWSTEMP` 下会生成 `krnln27.run` 文件，这个就是 易程序 的运行库,其实也就是相当于 `P-CODE` 所需的 `MSVBVM60.DLL`。`krnln27.run` 未加壳，`Visual C++ 6.0` 编写，大家可以在程序调试时把其复制出来反汇编看看。

对付 `P-CODE` 有 `WKTVBdebugger`、`ExDec` 等专用利器，而调试 易程序 只有通用工具。期待……！

有朋友分析过易程序取硬盘序列号的过程吗？我下 `bpx GetVolumeInformationA` 却只是看到程序在取其运行文件路径！

硬盘码：`453814891`

试炼码：`13572468`

———————————————————————

### 1、下断点 `BPX GetWindowTextA` 程序取试炼码

```
100B9335     FF15 9CD40C10         call dword ptr ds:\[<&USER32.GetWindowTextA>\]
100B933B     8B4C24 08             mov ecx,dword ptr ss:\[esp+8\]
100B933F     6A FF                 push -1
100B9341     E8 2B2C0000           call KRNLN27.100BBF71
100B9346     EB 0C                 jmp short KRNLN27.100B9354
```

———————————————————————

### 2、“乾坤大挪移” :-)

```
100A7442     8A06                  mov al,byte ptr ds:\[esi\]
100A7444     8807                  mov byte ptr ds:\[edi\],al
100A7446     8A46 01               mov al,byte ptr ds:\[esi+1\]
100A7449     8847 01               mov byte ptr ds:\[edi+1\],al
100A744C     8A46 02               mov al,byte ptr ds:\[esi+2\]
100A744F     C1E9 02               shr ecx,2
100A7452     8847 02               mov byte ptr ds:\[edi+2\],al
100A7455     83C6 03               add esi,3
100A7458     83C7 03               add edi,3
100A745B     83F9 08               cmp ecx,8
100A745E   ^ 72 CC               jb short KRNLN27.100A742C
100A742C    /FF248D AC740A10      jmp dword ptr ds:\[ecx\*4+100A74AC\]; KRNLN27.100A74FC
100A74FC     8B448E FC             mov eax,dword ptr ds:\[esi+ecx\*4-4\]
100A7500     89448F FC             mov dword ptr ds:\[edi+ecx\*4-4\],eax
100A7504     8D048D 00000000       lea eax,dword ptr ds:\[ecx\*4\]
100A750B     03F0                  add esi,eax
100A750D     03F8                  add edi,eax
100A750F     FF2495 18750A10      jmp dword ptr ds:\[edx\*4+100A7518\]
100A753C     8A06                  mov al,byte ptr ds:\[esi\]
100A753E     8807                  mov byte ptr ds:\[edi\],al
100A7540     8A46 01               mov al,byte ptr ds:\[esi+1\]
100A7543     8847 01               mov byte ptr ds:\[edi+1\],al
100A7546     8B45 08               mov eax,dword ptr ss:\[ebp+8\]
100A7549     5E                     pop esi
100A754A     5F                     pop edi
100A754B     C9                     leave
100A754C     C3                     retn
```

———————————————————————

### 3、把试炼码转换成浮点型

```
100570BB     50                     push eax
                                 ====>EAX\=13572468               试炼码
100570BC     E8 EA070500           call KRNLN27.100A78AB
                                 ====>把试炼码转换成浮点型
100570C1     8B4424 68             mov eax,dword ptr ss:\[esp+68\]
100570C5     83C4 10               add esp,10
100570C8     DD5C24 04             fstp qword ptr ss:\[esp+4\]
                                 ====>ST\=13572468.000000000000
```

———————————————————————

### 4、比较 :-)

```
10053538     E8 E3EAFBFF           call KRNLN27.10012020
                                 ====>取注册码    ST\=453065299.00000000000
1005353D     8B4424 64             mov eax,dword ptr ss:\[esp+64\]
10053541     8D5424 30             lea edx,dword ptr ss:\[esp+30\]
```
