
# Visual Protect V3.54 脱壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6282.htm`](https://www.pediy.com/kssd/pediy06/pediy6282.htm)

Visual Protect V3.54 脱壳 —— `VisualProtect.exe` 主程序

下载页面： `http://www.lw.by.ru/arsenal/visual\_protect\_3.5.4.shtml`

软件大小： 7.72 M

运行环境： `Win9x`/`WinME`/`Win2000`/`WinXP`

**【软件简介】**：`Visual Protect` 是一个保护、电子商务、和版权管理的工具。通过 `Visual Protect`，你可以保护任何 `Windows95`/`98`/`NT` 32 位软件免受侵犯。`Visual Protect` 保护程序无需编辑源代码，它允许你的顾客在购买程序之前试用，界面简单友好。

**【作者声明】**：初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：`WinXP`、`Ollydbg1.09`、`PEiD`、`LordPE`、`ImportREC`

—————————————————————————————————

**【脱壳过程】**：

调试前设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其他异常。

用 `Ollydbg` 手动脱壳，老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

```
006BCF90     55                   push ebp
                           ==== >进入 OD 后断在这！

006BCF91     8BEC                 mov ebp, esp
006BCF93     51                   push ecx
006BCF94     53                   push ebx
006BCF95     56                   push esi
006BCF96     57                   push edi
006BCF97     C705 B03E6C00 000000> mov dword ptr ds:[6C3EB0], 0
006BCFA1     68 48206C00          push VisualPr.006C2048           ;ASCII "kernel32.dll"
006BCFA6     FF15 00006C00        call dword ptr ds:[<&KERNEL32.LoadLibraryA>];kernel32.LoadLibraryA
006BCFAC     A3 0C3F6C00          mov dword ptr ds:[6C3F0C], eax
006BCFB1     68 58206C00          push VisualPr.006C2058           ;ASCII "GetModuleHandleA"
006BCFB6     A1 0C3F6C00          mov eax, dword ptr ds:[6C3F0C]
006BCFBB     50                   push eax
006BCFBC     FF15 04006C00        call dword ptr ds:[<&KERNEL32.GetProcAddress>];kernel32.GetProcAddress
```

上面这段代码对于用 `Visual Protect V3.54` 加壳的程序来说几乎是固定的。

OK，不管那么多，`F9` 和 `SHIFT+F9` 运行！直至出现“This application was protected using Visual Protect”的经典 `Visual Protect` 保护画面时停下，当然要点“`Try`”啦，否则程序退出还怎么玩？:-)

————————————————————————

```
0079478F     3A0A                 cmp cl, byte ptr ds:[edx]
                           ==== >这里异常！当第 2 次异常时看看堆栈：

0012F39C     007FBEF5   返回到 VP.007FBEF5 来自 VP.00794788
0012F3A0     0012F3C8   指针到下一个 SEH 记录
0012F3A4     007FBEFF   SE 句柄     //此处下断！

007FBEFF     E9 DC7CF9FF          jmp VP.00793BE0
                           ==== >堆栈区的第二条地址  设断  F9 断在此处

00793BE0     8B4424 04            mov eax, dword ptr ss:[esp+4]
00793BE4     F740 04 06000000     test dword ptr ds:[eax+4], 6
00793BEB     0F85 13010000        jnz VP.00793D04

   ………… 省 略 …………      直接 CTRL+F9 执行到返回

007FBF04     8B45 FC              mov eax, dword ptr ss:[ebp-4]
007FBF07     E8 8C86F9FF          call VP.00794598
007FBF0C     E8 3780F9FF          call VP.00793F48
007FBF11     5F                   pop edi
007FBF12     5E                   pop esi
007FBF13     5B                   pop ebx
007FBF14     59                   pop ecx
007FBF15     5D                   pop ebp
007FBF16     C3                   retn
                           ==== >返回到 0080802A
```

————————————————————————

下面是程序比较是否注册？是否过期？有兴趣自己看，偶直接 `CTRL+F9` 执行到返回

```
0080802A     FFB5 94FEFFFF        push dword ptr ss:[ebp-16C]
00808030     E8 7B5CFFFF          call VP.007FDCB0
00808035     8D95 90FEFFFF        lea edx, dword ptr ss:[ebp-170]
0080803B     E8 A00EF9FF          call VP.00798EE0
00808040     FFB5 90FEFFFF        push dword ptr ss:[ebp-170]
00808046     8D45 E8              lea eax, dword ptr ss:[ebp-18]
00808049     BA 03000000          mov edx, 3
0080804E     E8 BDC8F8FF          call VP.00794910
00808053     33C0                 xor eax, eax
00808055     55                   push ebp
00808056     68 B8808000          push VP.008080B8
0080805B     64:FF30              push dword ptr fs:[eax]
0080805E     64:8920              mov dword ptr fs:[eax], esp
00808061     B8 7C828000          mov eax, VP.0080827C     ; ASCII "GenUniqueSerial.GetMacAddresses"
00808066     E8 49E2FFFF          call VP.008062B4
0080806B     8D85 8CFEFFFF        lea eax, dword ptr ss:[ebp-174]
00808071     E8 F63CFFFF          call VP.007FBD6C
00808076     8B85 8CFEFFFF        mov eax, dword ptr ss:[ebp-174]
0080807C     8B55 F4              mov edx, dword ptr ss:[ebp-C]
0080807F     E8 7844FFFF          call VP.007FC4FC
00808084     85C0                 test eax, eax
00808086     7E 1B                jle short VP.008080A3
00808088     8B45 F4              mov eax, dword ptr ss:[ebp-C]
0080808B     8B10                 mov edx, dword ptr ds:[eax]
0080808D     FF52 14              call dword ptr ds:[edx+14]
00808090     85C0                 test eax, eax
00808092     7E 0F                jle short VP.008080A3
00808094     8D4D F8              lea ecx, dword ptr ss:[ebp-8]
00808097     33D2                 xor edx, edx
00808099     8B45 F4              mov eax, dword ptr ss:[ebp-C]
0080809C     8B18                 mov ebx, dword ptr ds:[eax]
0080809E     FF53 0C              call dword ptr ds:[ebx+C]
008080A1     EB 0B                jmp short VP.008080AE
008080A3     8D45 F8              lea eax, dword ptr ss:[ebp-8]
008080A6     8B55 EC              mov edx, dword ptr ss:[ebp-14]
008080A9     E8 82C5F8FF          call VP.00794630
008080AE     33C0                 xor eax, eax
008080B0     5A                   pop edx
008080B1     59                   pop ecx
008080B2     59                   pop ecx
008080B3     64:8910              mov dword ptr fs:[eax], edx
008080B6     EB 1F                jmp short VP.008080D7
008080B8     E9 23BBF8FF          jmp VP.00793BE0
008080BD     B8 A4828000          mov eax, VP.008082A4; ASCII "GenUniqueSerial.GetMacAddresses failed"
008080C2     E8 EDE1FFFF          call VP.008062B4
008080C7     8D45 F8              lea eax, dword ptr ss:[ebp-8]
008080CA     8B55 EC              mov edx, dword ptr ss:[ebp-14]
008080CD     E8 5EC5F8FF          call VP.00794630
008080D2     E8 71BEF8FF          call VP.00793F48
008080D7     B8 D4828000          mov eax, VP.008082D4; ASCII "GenUniqueSerial.Continue"
008080DC     E8 D3E1FFFF          call VP.008062B4
008080E1     8B45 F8              mov eax, dword ptr ss:[ebp-8]
008080E4     E8 67C7F8FF          call VP.00794850
008080E9     50                   push eax
008080EA     8B45 F8              mov eax, dword ptr ss:[ebp-8]
008080ED     E8 56C9F8FF          call VP.00794A48
008080F2     5A                   pop edx
008080F3     E8 3000FFFF          call VP.007F8128
008080F8     8B55 FC              mov edx, dword ptr ss:[ebp-4]
008080FB     66:8942 04           mov word ptr ds:[edx+4], ax
008080FF     8B45 E8              mov eax, dword ptr ss:[ebp-18]
00808102     E8 49C7F8FF          call VP.00794850
00808107     50                   push eax
00808108     8B45 E8              mov eax, dword ptr ss:[ebp-18]
0080810B     E8 38C9F8FF          call VP.00794A48
00808110     5A                   pop edx
00808111     E8 1200FFFF          call VP.007F8128
00808116     8B55 FC              mov edx, dword ptr ss:[ebp-4]
00808119     66:8942 06           mov word ptr ds:[edx+6], ax
0080811D     8D45 F8              lea eax, dword ptr ss:[ebp-8]
00808120     50                   push eax
00808121     8B45 FC              mov eax, dword ptr ss:[ebp-4]
00808124     0FB740 02            movzx eax, word ptr ds:[eax+2]
00808128     8985 74FEFFFF        mov dword ptr ss:[ebp-18C], eax
0080812E     C685 78FEFFFF 00     mov byte ptr ss:[ebp-188], 0
00808135     8B45 FC              mov eax, dword ptr ss:[ebp-4]
00808138     0FB740 04            movzx eax, word ptr ds:[eax+4]
```



```asm
0080813C     8985 7CFEFFFF        mov dword ptr ss:[ebp-184],eax   

00808142     C685 80FEFFFF 00     mov byte ptr ss:[ebp-180],0   

00808149     8B45 FC              mov eax,dword ptr ss:[ebp-4]   

0080814C     0FB740 06            movzx eax,word ptr ds:[eax+6]   

00808150     8985 84FEFFFF        mov dword ptr ss:[ebp-17C],eax   

00808156     C685 88FEFFFF 00     mov byte ptr ss:[ebp-178],0   

0080815D     8D95 74FEFFFF        lea edx,dword ptr ss:[ebp-18C]   

00808163     B9 02000000          mov ecx,2   

00808168     B8 F8828000          mov eax,VP.008082F8 ; ASCII "%.4X-%.4X-%.4X"   

0080816D     E8 F21DF9FF          call VP.00799F64   

00808172     8B45 F8              mov eax,dword ptr ss:[ebp-8]   

00808175     E8 D6C6F8FF          call VP.00794850   

0080817A     50                   push eax   

0080817B     8B45 F8              mov eax,dword ptr ss:[ebp-8]   

0080817E     E8 C5C8F8FF          call VP.00794A48   

00808183     5A                   pop edx   

00808184     E8 9FFFFEFF          call VP.007F8128   

00808189     8B55 FC              mov edx,dword ptr ss:[ebp-4]   

0080818C     66:8902              mov word ptr ds:[edx],ax   

0080818F     8B45 F4              mov eax,dword ptr ss:[ebp-C]   

00808192     E8 69B5F8FF          call VP.00793700   

00808197     B8 10838000          mov eax,VP.00808310; ASCII "GenUniqueSerial.End"   

0080819C     E8 13E1FFFF          call VP.008062B4   

008081A1     33C0                 xor eax,eax   

008081A3     5A                   pop edx   

008081A4     59                   pop ecx   

008081A5     59                   pop ecx   

008081A6     64:8910              mov dword ptr fs:[eax],edx   

008081A9     68 EB818000          push VP.008081EB   

008081AE     8D85 8CFEFFFF        lea eax,dword ptr ss:[ebp-174]   

008081B4     BA 04000000          mov edx,4   

008081B9     E8 FEC3F8FF          call VP.007945BC   

008081BE     8D85 4CFFFFFF        lea eax,dword ptr ss:[ebp-B4]   

008081C4     BA 02000000          mov edx,2   

008081C9     E8 EEC3F8FF          call VP.007945BC   

008081CE     8D45 E8              lea eax,dword ptr ss:[ebp-18]   

008081D1     BA 02000000          mov edx,2   

008081D6     E8 E1C3F8FF          call VP.007945BC   

008081DB     8D45 F8              lea eax,dword ptr ss:[ebp-8]   

008081DE     E8 B5C3F8FF          call VP.00794598   

008081E3     C3                   retn   

                              ====>返回到 008081EB   

008081EB     5F                   pop edi   

008081EC     5E                   pop esi   

008081ED     5B                   pop ebx   

008081EE     8BE5                 mov esp,ebp   

008081F0     5D                   pop ebp   

008081F1     C3                   retn   

                              ====>返回到 00807505   

————————————————————————   

00807505     8D45 EC              lea eax,dword ptr ss:[ebp-14]   

00807508     8B55 F8              mov edx,dword ptr ss:[ebp-8]   

0080750B     E8 04000000          call VP.00807514   

00807510     8BE5                 mov esp,ebp   

00807512     5D                   pop ebp   

00807513     C3                   retn   

                              ====>返回到 008322E4   

————————————————————————
```

还是比较注册信息，偶继续直接 `CTRL+F9` 执行到返回

```asm
008322E4     8B45 F8              mov eax,dword ptr ss:[ebp-8]   

008322E7     50                   push eax   

008322E8     8D55 F4              lea edx,dword ptr ss:[ebp-C]   

008322EB     B8 B4248300          mov eax,VP.008324B4   

008322F0     E8 6F45FDFF          call VP.00806864   

008322F5     8B45 F4              mov eax,dword ptr ss:[ebp-C]   

008322F8     5A                   pop edx   

008322F9     E8 2E98FCFF          call VP.007FBB2C   

008322FE     A1 F8BD8300          mov eax,dword ptr ds:[83BDF8]   

00832303     8B00                 mov eax,dword ptr ds:[eax]   

00832305     3B05 F8BA8300        cmp eax,dword ptr ds:[83BAF8]   

0083230B     75 27                jnz short VP.00832334   

0083230D     8D55 F0              lea edx,dword ptr ss:[ebp-10]   

00832310     B8 CC248300          mov eax,VP.008324CC   

00832315     E8 4A45FDFF          call VP.00806864   

0083231A     8B45 F0              mov eax,dword ptr ss:[ebp-10]   

0083231D     50                   push eax   

0083231E     8D55 EC              lea edx,dword ptr ss:[ebp-14]   

00832321     B8 DC248300          mov eax,VP.008324DC   

00832326     E8 3945FDFF          call VP.00806864   

0083232B     8B45 EC              mov eax,dword ptr ss:[ebp-14]   

0083232E     5A                   pop edx   

0083232F     E8 F897FCFF          call VP.007FBB2C   

00832334     A1 B0BE8300          mov eax,dword ptr ds:[83BEB0]   

00832339     8B00                 mov eax,dword ptr ds:[eax]   

0083233B     3B05 F8BA8300        cmp eax,dword ptr ds:[83BAF8]   

00832341     75 27                jnz short VP.0083236A   

00832343     8D55 E8              lea edx,dword ptr ss:[ebp-18]   

00832346     B8 F0248300          mov eax,VP.008324F0   

0083234B     E8 1445FDFF          call VP.00806864   

00832350     8B45 E8              mov eax,dword ptr ss:[ebp-18]   

00832353     50                   push eax   

00832354     B8 DC248300          mov eax,VP.008324DC   

00832359     8D55 E4              lea edx,dword ptr ss:[ebp-1C]   

0083235C     E8 0345FDFF          call VP.00806864   

00832361     8B45 E4              mov eax,dword ptr ss:[ebp-1C]   

00832364     5A                   pop edx   

00832365     E8 C297FCFF          call VP.007FBB2C   

0083236A     A1 68BF8300          mov eax,dword ptr ds:[83BF68]   

0083236F     8B00                 mov eax,dword ptr ds:[eax]   

00832371     3B05 F8BA8300        cmp eax,dword ptr ds:[83BAF8]   

00832377     0F85 F0000000        jnz VP.0083246D   

0083237D     8D55 E0              lea edx,dword ptr ss:[ebp-20]   

00832380     B8 00258300          mov eax,VP.00832500   

00832385     E8 DA44FDFF          call VP.00806864   

0083238A     8B45 E0              mov eax,dword ptr ss:[ebp-20]   

0083238D     50                   push eax   

0083238E     B8 DC248300          mov eax,VP.008324DC   

00832393     8D55 DC              lea edx,dword ptr ss:[ebp-24]   

00832396     E8 C944FDFF          call VP.00806864   

0083239B     8B45 DC              mov eax,dword ptr ss:[ebp-24]   

0083239E     5A                   pop edx   

0083239F     E8 8897FCFF          call VP.007FBB2C   

008323A4     8D55 D8              lea edx,dword ptr ss:[ebp-28]   

008323A7     B8 14258300          mov eax,VP.00832514   

008323AC     E8 B344FDFF          call VP.00806864   

008323B1     8B45 D8              mov eax,dword ptr ss:[ebp-28]   

008323B4     8B15 2CC08300        mov edx,dword ptr ds:[83C02C]   

008323BA     8B12                 mov edx,dword ptr ds:[edx]   

008323BC     E8 6B97FCFF          call VP.007FBB2C   

008323C1     33C0                 xor eax,eax   

008323C3     55                   push ebp   

008323C4     68 63248300          push VP.00832463   

008323C9     64:FF30              push dword ptr fs:[eax]   

008323CC     64:8920              mov dword ptr fs:[eax],esp   

008323CF     A1 1CBB8300          mov eax,dword ptr ds:[83BB1C]   

008323D4     8B00                 mov eax,dword ptr ds:[eax]   

008323D6     8B10                 mov edx,dword ptr ds:[eax]   

008323D8     FF52 14              call dword ptr ds:[edx+14]   

008323DB     8D55 D4              lea edx,dword ptr ss:[ebp-2C]   

008323DE     E8 FD6AF6FF          call VP.00798EE0   

008323E3     8B45 D4              mov eax,dword ptr ss:[ebp-2C]   

008323E6     50                   push eax   

008323E7     8D55 D0              lea edx,dword ptr ss:[ebp-30]   

008323EA     B8 2C258300          mov eax,VP.0083252C   
```



```
008323EF     E8 7044FDFF          call VP.00806864

008323F4     8B45 D0            mov eax,dword ptr ss:[ebp-30]

008323F7     5A                 pop edx

008323F8     E8 2F97FCFF        call VP.007FBB2C

008323FD     A1 1CBB8300        mov eax,dword ptr ds:[83BB1C]

00832402     8B00               mov eax,dword ptr ds:[eax]

00832404     8B10               mov edx,dword ptr ds:[eax]

00832406     FF52 14            call dword ptr ds:[edx+14]

00832409     8BF0               mov esi,eax

0083240B     4E                 dec esi

0083240C     85F6               test esi,esi

0083240E     7C 49              jl short VP.00832459

00832410     46                 inc esi

00832411     33DB               xor ebx,ebx

00832413     8D4D CC            lea ecx,dword ptr ss:[ebp-34]

00832416     A1 1CBB8300        mov eax,dword ptr ds:[83BB1C]

0083241B     8B00               mov eax,dword ptr ds:[eax]

0083241D     8BD3               mov edx,ebx

0083241F     8B38               mov edi,dword ptr ds:[eax]

00832421     FF57 0C            call dword ptr ds:[edi+C]

00832424     8B45 CC            mov eax,dword ptr ss:[ebp-34]

00832427     50                 push eax

00832428     8D55 C8            lea edx,dword ptr ss:[ebp-38]

0083242B     B8 40258300        mov eax,VP.00832540

00832430     E8 2F44FDFF        call VP.00806864

00832435     8D45 C8            lea eax,dword ptr ss:[ebp-38]

00832438     50                 push eax

00832439     8D55 C4            lea edx,dword ptr ss:[ebp-3C]

0083243C     8BC3               mov eax,ebx

0083243E     E8 9D6AF6FF        call VP.00798EE0

00832443     8B55 C4            mov edx,dword ptr ss:[ebp-3C]

00832446     58                 pop eax

00832447     E8 0C24F6FF        call VP.00794858

0083244C     8B45 C8            mov eax,dword ptr ss:[ebp-38]

0083244F     5A                 pop edx

00832450     E8 D796FCFF        call VP.007FBB2C

00832455     43                 inc ebx

00832456     4E                 dec esi

00832457     75 BA              jnz short VP.00832413

00832459     33C0               xor eax,eax

0083245B     5A                 pop edx

0083245C     59                 pop ecx

0083245D     59                 pop ecx

0083245E     64:8910            mov dword ptr fs:[eax],edx

00832461     EB 0A              jmp short VP.0083246D

00832463     E9 7817F6FF        jmp VP.00793BE0

00832468     E8 DB1AF6FF        call VP.00793F48

0083246D     33C0               xor eax,eax

0083246F     5A                 pop edx

00832470     59                 pop ecx

00832471     59                 pop ecx

00832472     64:8910            mov dword ptr fs:[eax],edx

00832475     68 8F248300        push VP.0083248F

0083247A     8D45 C4            lea eax,dword ptr ss:[ebp-3C]

0083247D     BA 0F000000        mov edx,0F

00832482     E8 3521F6FF        call VP.007945BC

00832487     C3                 retn

0083248F     5F                 pop edi

00832490     5E                 pop esi

00832491     5B                 pop ebx

00832492     8BE5               mov esp,ebp

00832494     5D                 pop ebp

00832495     C3                 retn

                        =====>返回到 00835D75

————————————————————————
```

下面就是关键啦 :-) 下面这段关键代码对于用 `Visual Protect V3.54` 加壳的程序来说也是固定的。

```
00835D75     B8 8C6A8300        mov eax,VP.00836A8C ; ASCII "SetEnvironment"

00835D7A     E8 B9BEFFFF        call VP.00831C38

00835D7F     A1 70BE8300        mov eax,dword ptr ds:[83BE70]

00835D84     8B40 41            mov eax,dword ptr ds:[eax+41]

                        =====>EAX=0026B508             这是 OEP 的偏移值

00835D87     0345 10            add eax,dword ptr ss:[ebp+10]

                        =====>EAX=0026B508 + 00400000=0066B508 这就是 OEP 值  :-)

00835D8A     8945 FC            mov dword ptr ss:[ebp-4],eax

                        =====>[ebp-4]=EAX=0066B508        呵呵，记住这个[ebp-4]

00835D8D     B8 144C8300        mov eax,VP.[NONAME]

00835D92     E8 A9100000        call VP.00836E40

00835D97     3B05 74018400      cmp eax,dword ptr ds:[840174]

00835D9D     A1 48BB8300        mov eax,dword ptr ds:[83BB48]

00835DA2     0F9400             sete byte ptr ds:[eax]

00835DA5     A1 70BE8300        mov eax,dword ptr ds:[83BE70]

00835DAA     F640 59 10         test byte ptr ds:[eax+59],10

00835DAE     74 0A              je short VP.00835DBA

00835DB0     33D2               xor edx,edx

00835DB2     8B45 10            mov eax,dword ptr ss:[ebp+10]

00835DB5     E8 5AA8FEFF        call VP.00820614

00835DBA     A1 7CBD8300        mov eax,dword ptr ds:[83BD7C]

00835DBF     8038 00            cmp byte ptr ds:[eax],0

00835DC2     75 6A              jnz short VP.00835E2E

00835E2E     A1 48BB8300        mov eax,dword ptr ds:[83BB48]

00835E33     8038 00            cmp byte ptr ds:[eax],0

00835E36     75 0A              jnz short VP.00835E42

00835E38     A1 DCBB8300        mov eax,dword ptr ds:[83BBDC]

00835E3D     E8 7E4CFDFF        call VP.0080AAC0

00835E42     B8 E06A8300        mov eax,VP.00836AE0        ; ASCII "Uncomress Sections"

00835E47     E8 ECBDFFFF        call VP.00831C38

00835E4C     8B45 08            mov eax,dword ptr ss:[ebp+8]

00835E4F     FF10               call dword ptr ds:[eax]

00835E51     A1 84BB8300        mov eax,dword ptr ds:[83BB84]

00835E56     8B00               mov eax,dword ptr ds:[eax]

00835E58     8B15 70BE8300      mov edx,dword ptr ds:[83BE70]

00835E5E     3B42 4D            cmp eax,dword ptr ds:[edx+4D]

00835E61     74 1B              je short VP.00835E7E

00835E7E     A1 C0C08300        mov eax,dword ptr ds:[83C0C0]

00835E83     8338 02            cmp dword ptr ds:[eax],2

00835E86     75 0A              jnz short VP.00835E92

00835E88     A1 D8BC8300        mov eax,dword ptr ds:[83BCD8]

00835E8D     8338 04            cmp dword ptr ds:[eax],4

00835E90     74 14              je short VP.00835EA6

00835E92     A1 C0C08300        mov eax,dword ptr ds:[83C0C0]

00835E97     8338 01            cmp dword ptr ds:[eax],1

00835E9A     75 1C              jnz short VP.00835EB8

00835EB8     B8 406B8300        mov eax,VP.00836B40        ; ASCII "Delphi ImportTable"

00835EBD     E8 76BDFFFF        call VP.00831C38

00835EC2     A1 84BB8300        mov eax,dword ptr ds:[83BB84]

00835EC7     8B00               mov eax,dword ptr ds:[eax]

00835EC9     E8 7AA6FEFF        call VP.[NONAME]

00835ECE     A1 48BB8300        mov eax,dword ptr ds:[83BB48]

00835ED3     8038 00            cmp byte ptr ds:[eax],0

00835ED6     74 5F              je short VP.00835F37

00835F37     8B45 F8            mov eax,dword ptr ss:[ebp-8]

00835F3A     E8 1924FCFF        call VP.007F8358

00835F3F     8B45 F8            mov eax,dword ptr ss:[ebp-8]

00835F42     E8 B9D7F5FF        call VP.00793700

00835F47     807D 20 00         cmp byte ptr ss:[ebp+20],0

00835F4B     0F85 B0000000      jnz VP.00836001

00835F51     A1 48BB8300        mov eax,dword ptr ds:[83BB48]

00835F56     8038 00            cmp byte ptr ds:[eax],0

00835F59     74 5F              je short VP.00835FBA

00835FBA     8B45 FC            mov eax,dword ptr ss:[ebp-4]   ; VisualPr.0066B508
```



```
00835FBD     33D2                xor edx,edx   

00835FBF     52                  push edx   

00835FC0     50                  push eax   

00835FC1     8D95 58FAFFFF       lea edx,dword ptr ss:[ebp-5A8]   

00835FC7     B8 08000000         mov eax,8   

00835FCC     E8 AF2FF6FF         call VP.00798F80   

00835FD1     8B8D 58FAFFFF       mov ecx,dword ptr ss:[ebp-5A8]   

00835FD7     8D85 5CFAFFFF       lea eax,dword ptr ss:[ebp-5A4]   

00835FDD     BA 946B8300         mov edx,VP.00836B94         ; ASCII "Finalizing 0x"   

00835FE2     E8 B5E8F5FF         call VP.0079489C   

00835FE7     8B85 5CFAFFFF       mov eax,dword ptr ss:[ebp-5A4]   

00835FED     E8 46BCFFFF         call VP.00831C38   

00835FF2     FF65 FC             jmp dword ptr ss:[ebp-4]   

                              ==== >飞向光明之巅！     :-)   

————————————————————————   

0066B508     55                  push ebp   

                              ==== >在这儿用 LordPE 完全 DUMP 这个进程   

0066B509     8BEC                mov ebp,esp   

0066B50B     83C4 F0             add esp,-10   

0066B50E     B8 D0AC6600         mov eax,VisualPr.0066ACD0   

0066B513     E8 E8BCD9FF         call VisualPr.00407200   

0066B518     A1 34D86700         mov eax,dword ptr ds:[67D834]   

0066B51D     8B00                mov eax,dword ptr ds:[eax]   

0066B51F     E8 301EE2FF         call VisualPr.0048D354   

0066B524     A1 34D86700         mov eax,dword ptr ds:[67D834]   

0066B529     8B00                mov eax,dword ptr ds:[eax]   

0066B52B     BA B0B56600         mov edx,VisualPr.0066B5B0     ; ASCII "Visual Protect"   

0066B530     E8 D318E2FF         call VisualPr.0048CE08   

————————————————————————
```

重新运行`VisualProtect.exe`，运行`ImportREC`，选择这个进程。 把`OEP`改为`0026B508`，点`IT AutoSearch`，

点“`Get Import`”，有许多函数无效，用“追踪层次 3”全部修复。`FixDump`，正常运行！ 1.95M -> 3.53M

脱壳后看是`Delphi`编写，可以跨系统平台运行。原程序是采用`Trial Limits`+`Keyfile`保护的，脱壳后即取消了 20 天的试用期限制。至于注册，`[TMG]`的`Judas Priest`大侠已经发布这个版本的注册机啦，`RC4`+`MD5`+`CRC`+`BlowFish`   呵呵，一大堆的密码学算法，牛！

其他采用`Visual Protect V3.54`加壳试用期保护的程序的脱壳方法和本文介绍的大体相同。

—————————————————————————————————     

```
      ,     \_/   

    /| \_.-~/          \_     ,        青春都一饷   

   ( /~   /             ~-. \ |   

   \`\\  \_/                ~ )          忍把浮名   

  _-~~~-.)  )\_\_/;;,.      \_  //'    

 /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   

 \`~ _( ,\_..-- (     ,;'' /    ~--   /.\_\`   

  /~~//'   /' \`~           ) /--..\_, )\_  \`~    

  "  \`~"  "       \`"       /~'\`    \`\\~~    

                  "     "   "~'  ""   

          Cracked By 巢水工作坊——fly [OCN][FCG]   

                  2003-10-26  17:11
```

`4dd466d0e1e0cc123afbe504da2462ca`
