
到入口后，修改各区段读写权限为完全权限，然后`DUMP`下来，并将`DUMP`下来的文件入口改为`7EBB8`。

## 二、修复 IAT

用`OD`载入`DUMP`后的文件，可以找到`IAT`地址：

```
00484164   00 00 00 00 70 38 14 00   ....p8.
0048416C   A8 38 14 00 E0 38 14 00   ?.?.
00484174   18 39 14 00 50 39 14 00   9.P9.
0048417C   88 39 14 00 C0 39 14 00   ?.?.
00484184   F8 39 14 00 C9 A5 4F 00   ?.丧 O.
......
004847EC   B7 4E 31 77 F6 7F 31 77   種 1w?1w
004847F4   54 50 31 77 25 74 32 77   TP1w%t2w
004847FC   A4 7F 33 77 57 A4 31 77   ?3wW?w
00484804   3D 51 31 77 E3 AD 31 77   =Q1w 悱 1w
0048480C   00 00 00 00 1B 31 06 63   ....1c
00484814   9D 30 06 63 98 9F 01 63   ?c 槦 c
0048481C   00 00 00 00 DA DF 9E 00   ....谶?
00484824   F3 DD 9E 00 3E BF 9E 00   筝?>繛.
0048482C   50 72 9E 00 E8 78 9E 00   Pr?鑨?
00484834   59 D1 9E 00 20 10 A4 71   Y 褳.
0048483C   F2 20 9E 00 D5 21 9E 00   ????
00484844   06 21 9E 00 F2 20 9E 00   !???
0048484C   28 C3 9E 00 00 00 00 00   (脼.....
00484854   FF FF FF FF FF FF FF FF   ????????
```

`IAT`的起始地址为`484164`，结束地址为`484854`，大小为`6F0`。

现在来找程序是从何处开始处理`IAT`的。

还是分析前面的`LOG`记录。我们可以发现从第`12H`次中断以后，连续出现`004F48E2`这个地址，前面也出现过几次。那么这个地址是否和处理`IAT`有关？我们载入带壳程序，还是用脚本停在第`12H`次中断处：

```
004F44FB     90                   nop///异常。
004F44FC     3C 04                cmp al,4
004F44FE     74 32                je short 004F4532              ; 004F4532
004F4500     74 03                je short 004F4505              ; 004F4505
004F4502     75 01                jnz short 004F4505             ; 004F4505
```

堆栈：

```
0012DB68   0012FFE0   指针到下一个 SEH 记录
0012DB6C   004F4594   SE 句柄
```

从`004F4594`处开始跟踪：

```
004F4594     8B4424 04            mov eax,dword ptr ss:[esp+4]
004F4598     8B4C24 0C            mov ecx,dword ptr ss:[esp+C]
004F459C     FF81 B8000000        inc dword ptr ds:[ecx+B8]
004F45A2     8B00                 mov eax,dword ptr ds:[eax]///注意！执行这句后 EAX=4F44FB
004F45A4     2D 03000080          sub eax,80000003
004F45A9     75 16                jnz short 004F45C1             ; 004F45C1
004F45AB     B8 55010000          mov eax,155
004F45B0     8941 18              mov dword ptr ds:[ecx+18],eax
004F45B3     33C0                 xor eax,eax
004F45B5     8941 04              mov dword ptr ds:[ecx+4],eax///清理硬件断点。
004F45B8     8941 08              mov dword ptr ds:[ecx+8],eax
004F45BB     8941 0C              mov dword ptr ds:[ecx+C],eax
004F45BE     8941 10              mov dword ptr ds:[ecx+10],eax
004F45C1     C3                   retn///返回到系统 DLL 中。
```

在`4F44FB`处下断，然后`F9`：

```
004F44FB     90                   nop///断下。以下均用 F7 走。
004F44FC     3C 04                cmp al,4
004F44FE     74 32                je short 004F4532 ///跳！
004F4500     74 03                je short 004F4505              ; 004F4505
004F4502     75 01                jnz short 004F4505             ; 004F4505
004F4504     E8 E8010000          call 004F46F1                  ; 004F46F1
004F4532     64:8F05 00000000     pop dword ptr fs:[0]           ; 0012FFE0
004F4539     83C4 04              add esp,4
004F453C     0F31                 rdtsc///急甘奔浼觳狻?
004F453E     8BC8                 mov ecx,eax
004F4540     8BDA                 mov ebx,edx
004F4542     7E 06                jle short 004F454A ///JMP 的变形。
004F4544     7F 04                jg short 004F454A              ; 004F454A
004F4546     0010                 add byte ptr ds:[eax],dl
004F456F     9D                   popfd
004F4570     58                   pop eax
004F4571     0F31                 rdtsc///第 2 次。
004F4573     2BC1                 sub eax,ecx
004F4575     1BD3                 sbb edx,ebx
004F4577     83FA 00              cmp edx,0
004F457A   ^ 75 84                jnz short 004F4500 ///不能跳。
004F457C     3D 00000060          cmp eax,60000000
004F4581   ^ 0F87 79FFFFFF        ja 004F4500   ///不能跳。
004F4587     74 39                je short 004F45C2   ///JMP 的变形。
004F4589     75 37                jnz short 004F45C2             ; 004F45C2
004F458B     E8 00104000          call 008F5590
```

最后来到：

```
004FC17C     57                   push edi
004FC17D     E8 4483FFFF          call 004F44C6
004FC182     8B8424 4C040000      mov eax,dword ptr ss:[esp+44C]
004FC189     33ED                 xor ebp,ebp
004FC18B     85C0                 test eax,eax
004FC18D     896C24 14            mov dword ptr ss:[esp+14],ebp
004FC191     75 6D                jnz short 004FC200 ///跳。以下用 F8 跟。
004FC193     68 FDDE4000          push 40DEFD
004FC198     E8 007FFFFF          call 004F409D                   ; 004F409D
004FC19D     50                   push eax
004FC19E     68 C4AA4000          push 40AAC4
004FC1A3     E8 F57EFFFF          call 004F409D                   ; 004F409D
004FC1A8     50                   push eax
004FC1A9     E8 452C0000          call 004FEDF3                   ; 004FEDF3
004FC1AE     50                   push eax
004FC1AF     E8 E4C2FFFF          call 004F8498                   ; 004F8498
004FC1B4     68 0CDF4000          push 40DF0C
004FC1B9     8BF8                 mov edi,eax
004FC1BB     E8 DD7EFFFF          call 004F409D                   ; 004F409D
004FC1C0     50                   push eax
004FC1C1     68 C4AA4000          push 40AAC4
004FC1C6     E8 D27EFFFF          call 004F409D                   ; 004F409D
004FC1CB     50                   push eax
004FC1CC     E8 222C0000          call 004FEDF3                   ; 004FEDF3
004FC1D1     50                   push eax
004FC1D2     E8 C1C2FFFF          call 004F8498                   ; 004F8498
004FC1D7     8BF0                 mov esi,eax
004FC1D9     E8 D87EFFFF          call 004F40B6                   ; 004F40B6
004FC1DE     85F6                 test esi,esi
004FC1E0     8BD8                 mov ebx,eax
004FC1E2     74 16                je short 004FC1FA               ; 004FC1FA
004FC1E4     85FF                 test edi,edi
004FC1E6     74 12                je short 004FC1FA               ; 004FC1FA
004FC1E8     68 20030000          push 320
004FC1ED     6A 08                push 8
004FC1EF     FFD7                 call edi
004FC1F1     50                   push eax
004FC1F2     FFD6                 call esi
004FC1F4     8BE8                 mov ebp,eax
004FC1F6     896C24 14            mov dword ptr ss:[esp+14],ebp
004FC1FA     89AB 94000000        mov dword ptr ds:[ebx+94],ebp
004FC200     33FF                 xor edi,edi
004FC202     897C24 1C            mov dword ptr ss:[esp+1C],edi
004FC206     897C24 2C            mov dword ptr ss:[esp+2C],edi
004FC20A     E8 7FF9FFFF          call 004FBB8E                   ; 004FBB8E
004FC20F     3D B9C8B813          cmp eax,13B8C8B9
004FC214     BE 01000000          mov esi,1
004FC219     75 04                jnz short 004FC21F              ; 004FC21F
004FC21B     897424 1C            mov dword ptr ss:[esp+1C],esi
004FC21F     E8 54F9FFFF          call 004FBB78                   ; 004FBB78
004FC224     35 47F2EA87          xor eax,87EAF247
004FC229     3D F71219C1          cmp eax,C11912F7
004FC22E     75 04                jnz short 004FC234              ; 004FC234
004FC230     897424 2C            mov dword ptr ss:[esp+2C],esi
004FC234     68 16DF4000          push 40DF16
004FC239     E8 5F7EFFFF          call 004F409D                   ; 004F409D
004FC23E     50                   push eax
004FC23F     68 C4AA4000          push 40AAC4
004FC244     E8 547EFFFF          call 004F409D                   ; 004F409D
004FC249     50                   push eax
004FC24A     E8 FAC2FFFF          call 004F8549                   ; 004F8549
```
