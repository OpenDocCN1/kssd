
```
0012FF4C   0012FF90
0012FF50   01210000
0012FF54   011F0000
0012FF58   01224138
0012FF5C   0123A318  ASCII "ARInMgCAUXY="
```

```
0040D490    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D494    A3 84D64800     mov dword ptr ds:[48D684], eax //pre-dip4,校验注册码，与硬盘指纹计算后生成一个密匙，在部分壳里，会解压 N 段代码起到防止脱壳的功能。
0040D499    C2 0400         retn 4
0040D49C    90            nop
0040D49D    90            nop
0040D49E    90            nop
0040D49F    90            nop
0040D4A0    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D4A4    A3 88D64800     mov dword ptr ds:[48D688], eax
0040D4A9    C2 0400         retn 4
0040D4AC    90            nop
0040D4AD    90            nop
0040D4AE    90            nop
0040D4AF    90            nop
0040D4B0    8B4424 04       mov eax, dword ptr ss:[esp+4]
0040D4B4    A3 8CD64800     mov dword ptr ds:[48D68C], eax
```

于是再脱壳程序，`dd 48D684` 下内存访问断点，分割时同样断在我们刚才错误的地方。

```
00411164    FF15 84D64800   call dword ptr ds:[48D684] //原程序和脱壳程序错误地方。
0041116A    E9 76000000     jmp dumped_.004111E5
0041116F    C590 74E92A3B   lds edx, fword ptr ds:[eax+3B2AE974]
00411175    3E:46           inc esi
00411177    15 BD0C6586     adc eax, 86650CBD
0041117C    C7              ???                           ; 未知命令
0041117D    6E            outs dx, byte ptr es:[edi]
0041117E    EB 01           jmp short dumped_.00411181
00411180    D5 C2           aad 0C2
00411182    D4 4A           aam 4A
00411184    D6            salc
00411185    45            inc ebp
00411186    18AA 3E4837A5   sbb byte ptr ds:[edx+A537483E], ch
0041118C    03DB          add ebx, ebx
0041118E    B8 68E13DED     mov eax, ED3DE168
00411193    DF29          fild qword ptr ds:[ecx]
00411195    ED            in eax, dx
00411196    FA            cli
00411197    A4            movs byte ptr es:[edi], byte ptr ds:[esi]
00411198    B2 DA         mov dl, 0DA
0041119A    8817          mov byte ptr ds:[edi], dl
0041119C    AB            stos dword ptr es:[edi]
```

```
01221BA4    833D A4632201 0>cmp dword ptr ds:[12263A4], 0 //跟进 00411164
01221BAB    75 0D         jnz short 01221BBA
01221BAD    A1 407D2201     mov eax, dword ptr ds:[1227D40]
01221BB2    8B15 447D2201   mov edx, dword ptr ds:[1227D44]
01221BB8    EB 0B         jmp short 01221BC5
01221BBA    A1 A4632201     mov eax, dword ptr ds:[12263A4]
01221BBF    8B15 347E2201   mov edx, dword ptr ds:[1227E34]
01221BC5    85C0          test eax, eax
01221BC7    74 0B         je short 01221BD4
01221BC9    85D2          test edx, edx
01221BCB    74 07         je short 01221BD4
01221BCD    52            push edx
01221BCE    50            push eax
01221BCF    E8 60FDFFFF     call 01221934
01221BD4    C3            retn   //具体什么作用返回就知道是动态解码用的。
```

```
00411164    FF15 84D64800   call dword ptr ds:[48D684]
0041116A    E9 01000000     jmp VideoSpl.00411170    //解码后跳转的位置都不一样，早就计划好了的。
0041116F    FD            std
00411170    8D4424 04       lea eax, dword ptr ss:[esp+4]
00411174    50            push eax
00411175    6A 04         push 4
00411177    56            push esi
00411178    68 A0FC4000     push VideoSpl.0040FCA0
0041117D    6A 00         push 0
0041117F    6A 00         push 0
00411181    FF15 5C334700   call dword ptr ds:[47335C]
00411187    85C0          test eax, eax
00411189    8986 D0450000   mov dword ptr ds:[esi+45D0], eax
0041118F    74 4E         je short VideoSpl.004111DF
00411191    6A 01         push 1
00411193    8D8E 64290000   lea ecx, dword ptr ds:[esi+2964]
00411199    C786 E0450000 0>mov dword ptr ds:[esi+45E0], 0
004111A3    E8 5E250500     call VideoSpl.00463706
004111A8    6A 00         push 0
004111AA    8D8E 582E0000   lea ecx, dword ptr ds:[esi+2E58]
004111B0    E8 51250500     call VideoSpl.00463706
004111B5    8B8E 68330000   mov ecx, dword ptr ds:[esi+3368]
004111BB    6A 00         push 0
004111BD    6A 00         push 0
004111BF    68 02040000     push 402
004111C4    51            push ecx
004111C5    FF15 28364700   call dword ptr ds:[473628]             ; USER32.SendMessageA
004111CB    8BCE          mov ecx, esi
004111CD    E8 5EF9FFFF     call VideoSpl.00410B30
004111D2    8B96 D0450000   mov edx, dword ptr ds:[esi+45D0]
004111D8    52            push edx
004111D9    FF15 60334700   call dword ptr ds:[473360]
004111DF    EB 04         jmp short VideoSpl.004111E5
004111E1    F9            stc
004111E2    BA C5305E83     mov edx, 835E30C5
004111E7    C404FF        les eax, fword ptr ds:[edi+edi*8]
004111EA    25 88D64800     and eax, 48D688
004111EF    90            nop
004111F0    56            push esi
004111F1    8BF1          mov esi, ecx
004111F3    E8 45A50500     call VideoSpl.0046B73D
004111F8    F64424 08 01    test byte ptr ss:[esp+8], 1
004111FD    74 09         je short VideoSpl.00411208
004111FF    56            push esi
00411200    E8 A4ED0400     call VideoSpl.0045FFA9
00411205    83C4 04         add esp, 4
00411208    8BC6          mov eax, esi
0041120A    5E            pop esi
0041120B    C2 0400         retn 4      //为了不漏掉解码代码也不多动脑筋，从 0041116A 到这个 Call 全部二进程复制。
```

二进制粘贴到脱壳程序的相应位置。

然后还是从程序里找一个

```
0040102B   .  C3            retn
```

给

```
00411164    FF15 84D64800   call dword ptr ds:[48D684]
```

模拟解码。

命令行 `DD 48D684`

修改 `01221BA4` 为 `0040102B`

复制所有修改到程序里面，保存为一个文件，`Ctrl+F2` 重新载入测试功能。

执行时仍然出错，继续找原因。

```
004111D9   .  FF15 60334700 call dword ptr ds:[<&kernel32.ResumeThre>; ResumeThread
004111DF   >  EB 04          jmp short dumped_.004111E5
004111E1      F9            db F9
004111E2      BA            db BA
004111E3   .  C530          lds esi, fword ptr ds:[eax]
004111E5   >  5E            pop esi
004111E6   .  83C4 04        add esp, 4
004111E9   .- FF25 88D64800 jmp dword ptr ds:[48D688] //刚才复制的代码这里指向壳中。
```

```
01221BD8    833D A4632201 0>cmp dword ptr ds:[12263A4], 0 //跟进
01221BDF    75 0D         jnz short 01221BEE
01221BE1    A1 407D2201     mov eax, dword ptr ds:[1227D40]
01221BE6    8B15 447D2201   mov edx, dword ptr ds:[1227D44]
01221BEC    EB 0B         jmp short 01221BF9
01221BEE    A1 A4632201     mov eax, dword ptr ds:[12263A4]
01221BF3    8B15 347E2201   mov edx, dword ptr ds:[1227E34]
01221BF9    85C0          test eax, eax
01221BFB    74 09         je short 01221C06
01221BFD    85D2          test edx, edx
01221BFF    74 05         je short 01221C06
01221C01    E8 A6FDFFFF     call 012219AC
01221C06    C3            retn  //还是一个 retn 返回程序，简单的验证是否脱壳。
```

于是简单应付它

```
004111E9    - FF25 88D64800 jmp dword ptr ds:[48D688]
004111EF      90            nop
```

改为

```
004111E9      C3            retn
004111EA      90            nop
004111EB      90            nop
004111EC      90            nop
004111ED      90            nop
004111EE      90            nop
004111EF      90            nop
```

保存最后的修改，所有暗桩清除,成功分割文件。

破解分析参考我的 `AVI MPEG RM WMV Joiner4.11` 脱壳+破解,过程查不多。

```
0040DED8  |.  E8 53D5FFFF   call dumped_.0040B430
0040DEDD  |.  8B5424 30     mov edx, dword ptr ss:[esp+30]
0040DEE1  |.  8B4C24 2C     mov ecx, dword ptr ss:[esp+2C]
0040DEE5  |.  8B4424 28     mov eax, dword ptr ss:[esp+28]
0040DEE9  |.  33D1          xor edx, ecx
0040DEEB  |.  83C4 14       add esp, 14
0040DEEE  |.  3BC2          cmp eax, edx
0040DEF0  |.  75 0A         jnz short dumped_.0040DEFC            //爆破点,jz short 0040DEFC
```
