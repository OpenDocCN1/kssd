
```
0044A140  8A 2B 86 7C C4 CE 80 7C 2B 2E 83 7C ED 09 93 7C

0044A150  FD 79 93 7C 3D 04 93 7C D4 05 93 7C AB 14 81 7C

0044A160  8D 2C 81 7C EE 1E 80 7C 0D E0 80 7C 16 1E 80 7C

0044A170  40 7A 95 7C 6B 17 80 7C AE 94 83 7C 53 00 83 7C

0044A180  24 1A 80 7C 0E 18 80 7C A6 0D 81 7C B7 2B 82 7C

0044A190  CF 21 82 7C 59 35 81 7C D7 EF 80 7C 5C E8 81 7C

0044A1A0  01 6A 82 7C B9 8C 83 7C 80 A4 80 7C CB D8 81 7C

0044A1B0  58 CD 80 7C 87 1F 82 7C 4D 11 86 7C 3F EB 80 7C

0044A1C0  67 23 80 7C 31 03 93 7C 2A E9 81 7C 77 9B 80 7C

0044A1D0  42 24 80 7C 77 1D 80 7C 28 AC 80 7C 57 B3 80 7C

0044A1E0  7C 36 81 7C 9F 0F 81 7C 6A 48 81 7C A9 26 82 7C

0044A1F0  E6 2B 81 7C 43 99 80 7C 2A E8 81 7C 50 F8 81 7C

0044A200  93 D2 80 7C DF 06 86 7C 4E A3 80 7C 33 73 F8 00

0044A210  80 0E 61 7D 55 0B 61 7D 10 73 F8 00 F0 BE D1 77

0044A220  65 C4 D1 77 7D BC D1 77 B3 C7 D3 77 4B BE D1 77

0044A230  C8 BD D1 77 8E BD D1 77 76 BD D1 77 9D 86 D1 77

0044A240  C7 86 D1 77 9E C5 D1 77 88 C1 D1 77 EC DB D1 77

0044A250  D4 B6 D1 77 2F BB D1 77 1B C0 D1 77 6C C9 D1 77

0044A260  42 8C D1 77 9D 8F D1 77 3E F1 D2 77 4F 02 D3 77

0044A270  05 C5 D1 77 3D 02 D3 77 5D 94 D1 77 33 B9 D1 77

0044A280  4C 1F D3 77 1A 00 D3 77 2F B7 D1 77 85 CB D1 77

0044A290  EA E8 D1 77 D8 FF D2 77 0D D6 D1 77 A2 BD D1 77

0044A2A0  3C 21 D3 77 71 BE D1 77 60 DA D1 77 50 62 D2 77

0044A2B0  55 E6 D1 77 16 48 D2 77 2E 8C D1 77 B5 37 D2 77

0044A2C0  9A F3 D2 77 DE F2 D2 77 E2 16 D2 77 80 FD D1 77

0044A2D0  DF 1A D3 77 0E 97 D1 77 92 0D D3 77 61 C6 D3 77

0044A2E0  DA 5E D2 77 09 B6 D1 77 AE B6 D1 77 57 C2 D1 77

0044A2F0  78 8E D1 77 DA C6 D3 77 1D B6 D1 77 9F 03 D3 77

0044A300  B2 C2 D1 77 11 12 D2 77 EA DA D1 77 EE D4 D1 77

0044A310  24 13 D2 77 3E 0B D2 77 8C 14 D2 77 5E 02 D2 77

0044A320  92 0A D2 77 A4 D8 D1 77 F9 D7 D1 77 24 15 D3 77

0044A330  42 10 D2 77 9C FA D2 77 F6 8B D1 77 B8 96 D1 77

0044A340  EA 04 D5 77 6C BF D1 77 43 F5 D2 77 1C B1 D3 77

0044A350  A7 73 F8 00 50 1A BD 77 FF 19 BD 77 BA 18 BD 77

0044A360  76 73 F8 00 D4 4F A2 71 33 73 F8 00 D8 7C 33 76

0044A370  1E 31 32 76 00 00 00 00 00 40 FF 00 00 FF FF 00
```

你也可以放在其他可用的空白处，但是分析发现`44A000`处是程序加壳前存放输入表的地方，正好还原吧。

接着我们来修复这些乱序的输入表函数，Patch代码如下。

```
00F9D44A     60                 pushad

00F9D44B     8BB5 E4D7FFFF      mov esi,dword ptr ss:[ebp-281C]

//Table with Rand order Function

00F9D451     8BBD 24D9FFFF      mov edi,dword ptr ss:[ebp-26DC]

//Table which Record all of IAT Caller address in Armadillo +2

00F9D457     833F 00            cmp dword ptr ds:[edi],0

00F9D45A     74 2E              je short 00F9D48A

00F9D45C     8B07               mov eax,dword ptr ds:[edi]

00F9D45E     05 00004000        add eax,400000

00F9D463     8B10               mov edx,dword ptr ds:[eax]

00F9D465     BB 00A04400        mov ebx,44A000

//44A000：粘贴的有序函数地址表

00F9D46A     81FA 90909090      cmp edx,90909090

00F9D470     74 13              je short 00F9D485

00F9D472     8B12               mov edx,dword ptr ds:[edx]

00F9D474     3B13               cmp edx,dword ptr ds:[ebx]

00F9D476     74 0B              je short 00F9D483

00F9D478     83C3 04            add ebx,4

00F9D47B     81FB 78A34400      cmp ebx,44A378

//44A378：有序函数地址表结束地址

00F9D481     7C F1              jl short 00F9D474

00F9D483     8918               mov dword ptr ds:[eax],ebx

//替换成指向有序函数的地址

00F9D485     83C7 04            add edi,4

00F9D488     EB CD              jmp short 00F9D457

00F9D48A     61                 popad

00F9D48B     90                 nop

//这里下断，循环出口
```

`OllyDBG`二进制代码复制：

```
60 8B B5 E4 D7 FF FF 8B BD 24 D9 FF FF 83 3F 00 74 2E 8B 07 05 00 00 40 00 8B 10 BB 00 A0 44 00
81 FA 90 90 90 90 74 13 8B 12 3B 13 74 0B 83 C3 04 81 FB 78 A3 44 00 7C F1 89 18 83 C7 04 EB CD
61 90
```

`F9`在`00F9D489`中断后乱序输入表修复完毕。

运行`ImportREC`，`RVA=0004A000`，`Size=00000374`，获取输入表，Cut掉几个填充在DLL间的垃圾数据，保存树文件。

因为下面Armadillo还要再对代码段加密，所以此时运行`LordPE`完全dump出此进程！

## 三、OEP

撤销上面patch的代码，把`EIP`重新定位在`00F9D44A`处，我们来找OEP。

`Ctrl+G: CreateThread`，在`CreateThread`函数末尾下断，`Shift+F9`中断后取消断点。

```
00F8C494     FF15 5C61FA00      call dword ptr ds:[FA615C]   ; kernel32.CreateThread

00F8C49A     50                 push eax

//返回这里

00F8C49B     FF15 4C62FA00      call dword ptr ds:[FA624C]   ; kernel32.CloseHandle

00F8C4A1     5F                 pop edi

00F8C4A2     5E                 pop esi

00F8C4A3     C9                 leave

00F8C4A4     C3                 retn

00F9F8A6     E8 40CBFEFF        call 00F8C3EB

00F9F8AB     59                 pop ecx

00F9F8AC     BE 98FAFA00        mov esi,0FAFA98

00F9F8B1     8BCE               mov ecx,esi

00F9F8B3     E8 B292FDFF        call 00F78B6A

00F9F8B8     84C0               test al,al

00F9F8BA     75 09              jnz short 00F9F8C5

00F9F8BC     6A 01              push 1

00F9F8BE     8BCE               mov ecx,esi

00F9F8C0     E8 83E2FDFF        call 00F7DB48

00F9F8C5     C705 E0C0FA00 D8CF>mov dword ptr ds:[FAC0E0],0FACFD8

00F9F8CF     B9 00ECFA00        mov ecx,0FAEC00

00F9F8D4     E8 44590000        call 00FA521D

00F9F8D9     53                 push ebx

00F9F8DA     E8 3E590000        call 00FA521D

00F9F8DF     59                 pop ecx

00F9F8E0     33D2               xor edx,edx

00F9F8E2     8955 DC            mov dword ptr ss:[ebp-24],edx

00F9F8E5     895D FC            mov dword ptr ss:[ebp-4],ebx

00F9F8E8     A0 BC00FB00        mov al,byte ptr ds:[FB00BC]

00F9F8ED     8845 C8            mov byte ptr ss:[ebp-38],al

00F9F8F0     84C0               test al,al

00F9F8F2     75 4A              jnz short 00F9F93E

00F9F8F4     A1 E400FB00        mov eax,dword ptr ds:[FB00E4]

00F9F8F9     8B48 7C            mov ecx,dword ptr ds:[eax+7C]

00F9F8FC     894D C4            mov dword ptr ss:[ebp-3C],ecx

00F9F8FF     894D D0            mov dword ptr ss:[ebp-30],ecx

00F9F902     8B0D FC00FB00      mov ecx,dword ptr ds:[FB00FC]        ; Armadill.00400000

00F9F908     894D C0            mov dword ptr ss:[ebp-40],ecx

00F9F90B     8B70 50            mov esi,dword ptr ds:[eax+50]

00F9F90E     3370 44            xor esi,dword ptr ds:[eax+44]

00F9F911     3370 24            xor esi,dword ptr ds:[eax+24]

00F9F914     03F1               add esi,ecx

00F9F916     8975 D4            mov dword ptr ss:[ebp-2C],esi

00F9F919     8B98 90000000      mov ebx,dword ptr ds:[eax+90]

00F9F91F     3358 4C            xor ebx,dword ptr ds:[eax+4C]

00F9F922     3358 08            xor ebx,dword ptr ds:[eax+8]

00F9F925     895D A8            mov dword ptr ss:[ebp-58],ebx

00F9F928     8D4D D0            lea ecx,dword ptr ss:[ebp-30]

00F9F92B     E8 D016FDFF        call 00F71000

00F9F930     33D2               xor edx,edx

00F9F932     F7F3               div ebx

00F9F934     03D6               add edx,esi

00F9F936     8955 CC            mov dword ptr ss:[ebp-34],edx

00F9F939     8B12               mov edx,dword ptr ds:[edx]

00F9F93B     8955 DC            mov dword ptr ss:[ebp-24],edx

00F9F93E     834D FC FF         or dword ptr ss:[ebp-4],FFFFFFFF

00F9F942     EB 11              jmp short 00F9F955

00F9F944     6A 01              push 1

00F9F946     58                 pop eax

00F9F947     C3                 retn

00F9F948     8B65 E8            mov esp,dword ptr ss:[ebp-18]

00F9F94B     834D FC FF         or dword ptr ss:[ebp-4],FFFFFFFF

00F9F94F     8B7D 08            mov edi,dword ptr ss:[ebp+8]

00F9F952     8B55 DC            mov edx,dword ptr ss:[ebp-24]

00F9F955     A1 E400FB00        mov eax,dword ptr ds:[FB00E4]

00F9F95A     3150 7C            xor dword ptr ds:[eax+7C],edx

00F9F95D     A1 E400FB00        mov eax,dword ptr ds:[FB00E4]

00F9F962     3150 7C            xor dword ptr ds:[eax+7C],edx

00F9F965     A1 E400FB00        mov eax,dword ptr ds:[FB00E4]

00F9F96A     8B88 8C000000      mov ecx,dword ptr ds:[eax+8C]

00F9F970     3348 08            xor ecx,dword ptr ds:[eax+8]

00F9F973     3348 04            xor ecx,dword ptr ds:[eax+4]

00F9F976     030D FC00FB00      add ecx,dword ptr ds:[FB00FC]        ; Armadill.00400000
```
