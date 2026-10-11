
# tElock+ PECompact 双重壳 DLL 一次脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-597.htm`](https://www.pediy.com/kssd/pediy07/pediy7-597.htm)

【破解作者】 stasi\[DCM\]\[BCG\]\[DFCG\]\[FCG\]\[OCN\]\[CZG\]\[D.4s\]  
【作者邮箱】 stasi@163.com  
【使用工具】 `OD(diy)+LordPE+ImportREC+peditor+PEComAngela`  
【破解平台】 `Win9x/NT/2000/XP`  
【软件名称】 [http://www.pediy.com/bbs/pediy7/pediy7-597.rar](http://www.pediy.com/bbs/pediy7/pediy7-597.rar)  
【软件简介】 传到邮箱里的，看名字估计是网游的外挂  
【软件大小】 200k  
【加壳方式】 `tElock+ PECompact`  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

`peid0.93` 查出是 `tElock 0.98b1 -> tE!`，这个壳的重定位表处理很容易能处理的，起先我把输入表和重定位表处理过后，就直接 `code` 段断下，就直接在 `loaddll.exe` 的里运行了，以为是断点设置问题，费了很多时间，后来的时候发现是两重壳 `tElock+ PECompact` 的 `dll`，仔细的处理，也是很简单的过程

把“忽略在 `KERNEL32` 中的内存访问异常”、“`INT3` 中断”、“单步中断” 这 3 个选项选上

`od` 停在这里

```
1004A0A7    F7F3                div ebx                  第一次异常
1004A0A9    64:67:8F06 0000     pop dword ptr fs:[0]
1004A0AF    83C4 04             add esp,4
1004A0B2    66:BE 4746          mov si,4647
1004A0B6    66:BF 4D4A          mov di,4A4D
1004A0BA    8A85 99000000       mov al,byte ptr ss:[ebp+99]
1004A0C0    E9 9C000000         jmp 1004A161
1004A6A8    8DC0                lea eax,eax              非法使用寄存器
1004A6AA    74 03               je short 1004A6AF
1004A6AC    CD20 64678F06       vxdcall 68F6764
1004A6B2    0000                add byte ptr ds:[eax],al
1004A6B4    EB 02               jmp short 1004A6B8
1004A6B6    CD20 5961F58D       vxdcall 8DF56159
1004A6BC    74 15               je short 1004A6D3
1004A6BE    0083 C2228BFE       add byte ptr ds:[ebx+FE8B22C2],al
1004AAA1    66:F7F3             div bx                   第三次异常
1004AAA4    0F85 5B010000       jnz 1004AC05
1004AAAA    0F84 55010000       je 1004AC05
1004AAB0    8D85 430A0000       lea eax,dword ptr ss:[ebp+A43]
1004AAB6    894424 04           mov dword ptr ss:[esp+4],eax
1004AABA    64:67:8926 0000     mov dword ptr fs:[0],esp
1004AAC0    EB 12               jmp short 1004AAD4
1004AB67    F7F3                div ebx                  第四次异常
1004AB69    85D2                test edx,edx
1004AB6B    0F84 94000000       je 1004AC05
1004AB71    8D85 050B0000       lea eax,dword ptr ss:[ebp+B05]
1004AB77    894424 04           mov dword ptr ss:[esp+4],eax
1004AB7B    64:67:8926 0000     mov dword ptr fs:[0],esp
1004AB81    EB 11               jmp short 1004AB94
1004AB83  - E9 8B642408         jmp 18291013
1004ABA6    CD 68               int 68                   第五次异常
1004ABA8    66:05 7B0C          add ax,0C7B
1004ABAC    66:48               dec ax
1004ABAE    74 55               je short 1004AC05
1004B6F1    8DC0                lea eax,eax              非法使用寄存器
1004B6F3    EB 01               jmp short mirrg.1004B6F6
1004B6F5    EB 68               jmp short mirrg.1004B75F
1004B6F7    33C0                xor eax,eax
1004B6F9  - EB FE               jmp short mirrg.1004B6F9
```

看堆栈

```
0006FB78     0006FC34  指针到下一个 SEH 记录
0006FB7C     1004B6FF  SE 句柄                   返回地址，下断
0006FB80     1004B714  mirrg.1004B714
1004B6FF    8B6424 08       mov esp,dword ptr ss:[esp+8]
1004B703    33C0            xor eax,eax
1004B705    FF6424 08       jmp dword ptr ss:[esp+8]
1004B714    64:8F00         pop dword ptr fs:[eax]          ; 0006FC34
1004B717    58              pop eax
1004B718    EB 02           jmp short mirrg.1004B71C
1004B71C    58              pop eax                     ; mirrg.1004B714
1004B71D    5D              pop ebp
1004B71E    85E4            test esp,esp
1004B720    79 03           jns short mirrg.1004B725
1004B725    83D0 FF         adc eax,-1
1004B728    E8 75000000     call mirrg.1004B7A2
1004B72D    FD              std                           code 有变化 解开一段
1004B72D    8B9D 82D34000   mov ebx,dword ptr ss:[ebp+40D382]
1004B733    33F6            xor esi,esi
1004B735    F7D3            not ebx
1004B737    0BF3            or esi,ebx
1004B739    75 08           jnz short mirrg.1004B743
1004B73B    8D9D A2B64000   lea ebx,dword ptr ss:[ebp+40B6A2]
1004B741    EB 06           jmp short mirrg.1004B749
1004B743    039D 62D34000   add ebx,dword ptr ss:[ebp+40D362]
1004B749    895C24 F0       mov dword ptr ss:[esp-10],ebx
1004B74D    8DBD 84D24000   lea edi,dword ptr ss:[ebp+40D284]
1004B753    33C0            xor eax,eax
1004B755    B9 9E030000     mov ecx,39E
1004B75A    F3:AA           rep stos byte ptr es:[edi]
1004B75C    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B762    B9 58170000     mov ecx,1758
1004B767    F3:AA           rep stos byte ptr es:[edi]
1004B769    66:AB           stos word ptr es:[edi]
1004B76B    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B771    85F6            test esi,esi
1004B773    75 08           jnz short mirrg.1004B77D
1004B759    00F3            add bl,dh
1004B75B    AA              stos byte ptr es:[edi]
1004B75C    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B762    B9 58170000     mov ecx,1758
1004B767    F3:AA           rep stos byte ptr es:[edi]
1004B769    66:AB           stos word ptr es:[edi]
1004B76B    8DBD A2B64000   lea edi,dword ptr ss:[ebp+40B6A2]
1004B771    85F6            test esi,esi
1004B773    75 08           jnz short mirrg.1004B77D
1004B775    C707 33C040C3   mov dword ptr ds:[edi],C340C033
1004B77B    EB 0B           jmp short mirrg.1004B788
1004B77D    C607 E9         mov byte ptr ds:[edi],0E9
1004B780    47              inc edi
1004B781    2BDF            sub ebx,edi
1004B783    83EB 04        sub ebx,4
1004B786    891F           mov dword ptr ds:[edi],ebx
1004B788    8DBD FACD4000  lea edi,dword ptr ss:[ebp+40CDFA]
1004B78E    B9 2C000000    mov ecx,2C
1004B793    F3:AA          rep stos byte ptr es:[edi]
1004B795    66:AB          stos word ptr es:[edi]
1004B797    EB 02          jmp short mirrg.1004B79B
1004B79B    61             popad                    堆栈恢复
1004B79C    FF6424 D0      jmp dword ptr ss:[esp-30]
```

设置硬件执行断点，方便再次 `load` ，并忽略所有异常

```
10045000  /EB 06         jmp short mirrg.10045008      PECompact 1.68 - 1.84 -> Jeremy Collake 的入口
10045002  |68 3ED10000   push 0D13E                  基址 10000000 oep=1000D13E
10045007  |C3            retn
10045008  \9C            pushfd
10045009   60            pushad
1004500A   E8 02000000   call rdmIR.10045011          jmp 过
1004500F   33C0          xor eax,eax
10045011   8BC4          mov eax,esp
10045013   83C0 04       add eax,4
10045016   93            xchg eax,ebx
10045017   8BE3          mov esp,ebx
10045019   8B5B FC       mov ebx,dword ptr ds:[ebx-4]
1004501C   81EB 3F904000 sub ebx,40903F
10045022   87DD          xchg ebp,ebx
1004506C   8DB5 AC904000 lea esi,dword ptr ss:[ebp+4090AC]
10045072   B9 40040000   mov ecx,440
10045077   F3:A5         rep movs dword ptr es:[edi],dword p>
10045079   8BFB          mov edi,ebx                   ; mirrg.1004617B
1004507B   C3            retn                          返回
1004617B   BD CFD0C30F   mov ebp,0FC3D0CF
10046180   8BF7          mov esi,edi
10046182   83C6 54       add esi,54
10046185   81C7 FF100000 add edi,10FF
1004618B   56            push esi
1004624D   57            push edi
1004624E   AD            lods dword ptr ds:[esi]
```



```text
1004624F    85C0               test eax,eax

10046251    0F84 9B000000      je mirrg.100462F2                     这里下断，f9 跳走

10046257    8BD0               mov edx,eax

10046259    0395 E6904000      add edx,dword ptr ss:[ebp+4090E6]

1004625F    AD                 lods dword ptr ds:[esi]

10046260    56                 push esi

10046261    8BC8               mov ecx,eax

10046263    57                 push edi

10046264    52                 push edx

10046265    8DB5 6BA14000      lea esi,dword ptr ss:[ebp+40A16B]

1004626B    57                 push edi

1004626C    51                 push ecx

1004626D    52                 push edx

1004626E    6A 40              push 40

10046270    56                 push esi

10046271    FFB5 3D974000      push dword ptr ss:[ebp+40973D]

10046277    FFB5 39974000      push dword ptr ss:[ebp+409739]

1004627D    E8 B8090000        call mirrg.10046C3A

10046282    5A                 pop edx

10046283    5F                 pop edi

10046284    8D85 E4914000      lea eax,dword ptr ss:[ebp+4091E4]

1004628A    50                 push eax

1004628B    64:67:FF36 0000    push dword ptr fs:[0]

10046291    64:67:8926 0000    mov dword ptr fs:[0],esp

10046297    52                 push edx

10046298    57                 push edi

10046299    FF95 DA904000      call dword ptr ss:[ebp+4090DA]

1004629F    64:67:8F06 0000    pop dword ptr fs:[0]

100462A5    83C4 04            add esp,4

100462A8    85C0               test eax,eax

100462AA    74 07              je short mirrg.100462B3

100462AC    8BC8               mov ecx,eax

100462AE    5E                 pop esi

100462AF    5F                 pop edi

100462B0   ^ EB 9B             jmp short mirrg.1004624D

100462B2    B9 E8000000        mov ecx,0E8                      这里会跑飞，在上面的 code 里找出口

100462F1    24 5F              and al,5F

100462F3    8BB5 E2904000      mov esi,dword ptr ss:[ebp+4090E2]

100462F9    AD                 lods dword ptr ds:[esi]

100462FA    83F8 FF            cmp eax,-1

100462FD    74 74              je short mirrg.10046373            一样的处理方法

100462FF    0385 E6904000      add eax,dword ptr ss:[ebp+4090E6]

10046305    8BD8               mov ebx,eax

10046307    AD                 lods dword ptr ds:[esi]

10046308    0385 E6904000      add eax,dword ptr ss:[ebp+4090E6]

1004630E    8BD0               mov edx,eax

10046310    AD                 lods dword ptr ds:[esi]

10046311    8BC8               mov ecx,eax

10046313    57                 push edi

10046314    56                 push esi

10046315    8BF3               mov esi,ebx

10046317    57                 push edi

10046318    51                 push ecx

10046319    8BC1               mov eax,ecx

1004631B    C1F9 02            sar ecx,2

1004631E    F3:A5              rep movs dword ptr es:[edi],dword p>

10046320    03C8               add ecx,eax

10046322    83E1 03            and ecx,3

10046325    F3:A4              rep movs byte ptr es:[edi],byte ptr>

10046327    59                 pop ecx

10046328    5E                 pop esi

10046329    8BFA               mov edi,edx

1004632B    8BC1               mov eax,ecx

1004632D    C1F9 02            sar ecx,2

10046330    F3:A5              rep movs dword ptr es:[edi],dword p>

10046332    03C8               add ecx,eax

10046334    83E1 03            and ecx,3

10046337    F3:A4              rep movs byte ptr es:[edi],byte ptr>

10046339    5E                 pop esi

1004633A    AD                 lods dword ptr ds:[esi]

1004633B    8BC8               mov ecx,eax

1004633D    8BD0               mov edx,eax

1004633F    33C0               xor eax,eax

10046341    C1F9 02            sar ecx,2

10046344    F3:AB              rep stos dword ptr es:[edi]

10046346    03CA               add ecx,edx

10046348    83E1 03            and ecx,3

1004634B    F3:AA              rep stos byte ptr es:[edi]

1004634D    8B7E F0            mov edi,dword ptr ds:[esi-10]

10046350    03BD E6904000      add edi,dword ptr ss:[ebp+4090E6]

10046356    8B4E F4            mov ecx,dword ptr ds:[esi-C]

10046359    038D E6904000      add ecx,dword ptr ss:[ebp+4090E6]

1004635F    2BCF               sub ecx,edi

10046361    8BD1               mov edx,ecx

10046363    C1F9 02            sar ecx,2

10046366    F3:AB              rep stos dword ptr es:[edi]

10046368    03CA               add ecx,edx

1004636A    83E1 03            and ecx,3

1004636D    F3:AA              rep stos byte ptr es:[edi]

1004636F    5F                 pop edi

10046370   ^ EB 87             jmp short mirrg.100462F9

10046373    68 00400000        push 4000                      申请空间

10046378    6A 00              push 0

1004637A    57                 push edi

1004637B    FF95 45974000      call dword ptr ss:[ebp+409745]

10046381    8BBD 3C964000      mov edi,dword ptr ss:[ebp+40963C]

1004639F    49                 dec ecx

100463A0    74 72              je short mirrg.10046414

100463A2    78 70              js short mirrg.10046414

100463A4    66:8B07            mov ax,word ptr ds:[edi]

100463A7    2C E8              sub al,0E8

100463A9    3C 01              cmp al,1

100463AB    76 38              jbe short mirrg.100463E5

100463AD    66:3D 1725         cmp ax,2517

100463B1    74 51              je short mirrg.10046404

100463B3    3C 27              cmp al,27

100463B5    75 0A              jnz short mirrg.100463C1

100463B7    80FC 80            cmp ah,80

100463BA    72 05              jb short mirrg.100463C1

100463BC    80FC 8F            cmp ah,8F

100463BF    76 05              jbe short mirrg.100463C6

100463C1    47                 inc edi

100463C2    43                 inc ebx

100463C3   ^ EB DA             jmp short mirrg.1004639F

100463C5    B8 8B47023C        mov eax,3C02478B

100463CA    0975 0E            or dword ptr ss:[ebp+E],esi

100463CD    66:C1E8 08         shr ax,8

100463D1    C1C0 10            rol eax,10

100463D4    86C4               xchg ah,al

100463D6    2BC3               sub eax,ebx

100463D8    8947 02            mov dword ptr ds:[edi+2],eax

100463DB    BE 06000000        mov esi,6

100463E0    83E9 05            sub ecx,5

100463E3   ^ EB B6             jmp short mirrg.1004639B

100463E5    8B47 01            mov eax,dword ptr ds:[edi+1]       这里下断
```

找到处理重定位表的位置

```text
10046652    8B9D E6904000      mov ebx,dword ptr ss:[ebp+4090E6]  基址 10000000

10046658    3B9D 5F974000      cmp ebx,dword ptr ss:[ebp+40975F]

1004665E    75 01              jnz short mirrg.10046661           与映像基址不相等就处理重定位

10046660    C3                 retn                               改变标志，使之跳转

10046661    8BB5 63974000      mov esi,dword ptr ss:[ebp+409763]  esi=0003D000 重定位表开始地址

10046667    03F3               add esi,ebx

10046669    33C0               xor eax,eax

1004666B    66:8B43 3C         mov ax,word ptr ds:[ebx+3C]

1004666F    03C3               add eax,ebx

10046671    8B80 C0000000      mov eax,dword ptr ds:[eax+C0]

10046677    85C0               test eax,eax

10046679    75 08              jnz short mirrg.10046683

1004667B    2B9D 5F974000      sub ebx,dword ptr ss:[ebp+40975F]

10046681    EB 0F              jmp short mirrg.10046692

10046683    03C3               add eax,ebx

10046685    2B9D 5F974000      sub ebx,dword ptr ss:[ebp+40975F]

1004668B    0118               add dword ptr ds:[eax],ebx

1004668D    83C0 04            add eax,4

10046690    0118               add dword ptr ds:[eax],ebx

10046692    AD                 lods dword ptr ds:[esi]

10046693    0BC0               or eax,eax
```



10046695 74 6F je short mirrg.10046706 这里下断，可以看到结束地址

ESI=100403E8

```
10046697 8BD0 mov edx,eax
10046699 0395 E6904000 add edx,dword ptr ss:[ebp+4090E6]
1004669F AD lods dword ptr ds:[esi]
100466A0 8BC8 mov ecx,eax
100466A2 83E9 08 sub ecx,8
100466A5 D1E9 shr ecx,1
100466A7 66:C785 55974000 0 mov word ptr ss:[ebp+409755],0
100466B0 33C0 xor eax,eax
100466B2 66:AD lods word ptr ds:[esi]
100466B4 0BC0 or eax,eax
100466B6 74 49 je short mirrg.10046701
100466B8 66:0385 55974000 add ax,word ptr ss:[ebp+409755]
100466BF 66:8985 55974000 mov word ptr ss:[ebp+409755],ax
100466C6 50 push eax
100466C7 C1E8 0C shr eax,0C
100466CA 83F8 01 cmp eax,1
100466CD 75 0E jnz short mirrg.100466DD
100466CF 58 pop eax
100466D0 25 FF0F0000 and eax,0FFF
100466D5 03C2 add eax,edx
100466D7 66:0158 02 add word ptr ds:[eax+2],bx
100466DB EB 24 jmp short mirrg.10046701
100466DD 83F8 02 cmp eax,2
100466E0 75 0D jnz short mirrg.100466EF
100466E2 58 pop eax
100466E3 25 FF0F0000 and eax,0FFF
100466E8 03C2 add eax,edx
100466EA 66:0118 add word ptr ds:[eax],bx
100466ED EB 12 jmp short mirrg.10046701
100466EF 83F8 03 cmp eax,3
100466F2 75 0C jnz short mirrg.10046700
100466F4 58 pop eax
100466F5 25 FF0F0000 and eax,0FFF
100466FA 03C2 add eax,edx
100466FC 0118 add dword ptr ds:[eax],ebx
100466FE EB 01 jmp short mirrg.10046701
10046700 58 pop eax
10046701 49 dec ecx
10046702 ^ 75 AC jnz short mirrg.100466B0
10046704 ^ EB 8C jmp short mirrg.10046692
10046706 C3 retn
```

直接在 `1000D13E` 下硬件执行断点

```
1000D13E /. 55 push ebp oep=1000D13E
1000D13F |. 8BEC mov ebp,esp
1000D141 |. 53 push ebx
1000D142 |. 8B5D 08 mov ebx,dword ptr ss:[ebp+8]
1000D145 |. 56 push esi
1000D146 |. 8B75 0C mov esi,dword ptr ss:[ebp+C]
1000D149 |. 57 push edi
1000D14A |. 8B7D 10 mov edi,dword ptr ss:[ebp+10]
1000D14D |. 85F6 test esi,esi
1000D14F |. 75 09 jnz short rdmIR.1000D15A
1000D151 |. 833D 14880310 00 cmp dword ptr ds:[10038814],0
1000D158 |. EB 26 jmp short rdmIR.1000D180
1000D15A |> 83FE 01 cmp esi,1
1000D15D |. 74 05 je short rdmIR.1000D164
```

再找 IAT

```
10028000 796D1E76 ADVAPI32.RegCloseKey IAT 开始地址
10028004 796D3280 ADVAPI32.RegCreateKeyExA
10028008 796D2506 ADVAPI32.RegOpenKeyExA
1002800C 796D33CB ADVAPI32.RegSetValueExA
10028010 00000000
100284C8 00000000
100284CC 77A6C4C5 OLE32.OleFlushClipboard
100284D0 77A6AD73 OLE32.CLSIDFromProgID
100284D4 77A624B1 OLE32.CLSIDFromString
100284D8 77A70679 OLE32.CoGetClassObject
100284DC 77AC67EC OLE32.StgOpenStorageOnILockBytes
100284E0 77A6D0A4 OLE32.StgCreateDocfileOnILockBytes
100284E4 77A6CD3C OLE32.CreateILockBytesOnHGlobal
100284E8 77A46D7B OLE32.CoTaskMemFree
100284EC 77A452F1 OLE32.CoTaskMemAlloc
100284F0 77A60C9C OLE32.OleInitialize
100284F4 77A5E5B3 OLE32.OleUninitialize
100284F8 77A9A5E6 OLE32.CoFreeUnusedLibraries
100284FC 77A5E334 OLE32.CoRegisterMessageFilter
10028500 77A5F3D7 OLE32.CoRevokeClassObject
10028504 77A6BCF4 OLE32.OleIsCurrentClipboard
10028508 00000000
1002850C 7528F48C oledlg.OleUIBusyA IAT 结束的地址
```

在 `oep` 处用 `LordPE` dump 出来，用 `peditor` 的 `DumpFixer` 逐一修正区块

`ImportREC` 载入 `dll`，`oep=0000D13E` `rva=00028000` `size=50C` 修复 `dump.dll`

用 `WinHex` 打开 `dll`，复制 `3D000-403E4` 另存为新的二进制文件，再用看雪老师的修复工具 `PEComAngela`，得到还原过重定位表的 `pediy.bin`

用 `WinHex` 把 `pediy.bin` 写入到 `dll` 文件的 `32000` 处。

最后用 `LordPE` 修正 `oep=0000D13E` `Relocalation` 的 `RVA=3D000` `SIZE=000033E4`

终于 `DLL_Loader` 能加载，`peid0.93` 显示 `Microsoft Visual C++ 6.0 DLL`

\--------------------------------------------------------------------------------

【破解总结】

再次感谢看雪老师，他的 `PEComAngela` 使 `PECompact` 的重定位表修复工作容易很多！

ps：对 `loveboom` 近期遭遇深表同情，套用西游记里的话就是“这些人都是路过的妖精，对他们认真不得。”，同时也希望师傅继续高产势头，多多向我们抛玉！

\--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

2005-7-26

`4dd466d0e1e0cc123afbe504da2462ca`
