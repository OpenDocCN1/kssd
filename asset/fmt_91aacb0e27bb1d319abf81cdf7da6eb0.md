
### 1. 代码提取与脱壳准备

将表存成`txt`文件。使用`f8`单步执行，注意上述提到的四个值，它们在已壳脱壳时有用。

将`00D459A4`到`00D45A58`之间的代码用二进制复制的方法复制下来备用。

```assembly
00D459D4     E8 98B6FDFF          call 00D21071      ; f7 进入

00D21071     55                   push ebp           ; dump 出来的代码 2
00D21072     8BEC                 mov ebp, esp
00D21074     51                   push ecx
00D21075     8B45 08              mov eax, dword ptr ss:[ebp+8]
00D21078     53                   push ebx
00D21079     B9 10270000          mov ecx, 2710
00D2107E     56                   push esi
00D2107F     99                   cdq
00D21080     8BF1                 mov esi, ecx
00D21082     57                   push edi
00D21083     F7FE                 idiv esi
00D21085     8B45 0C              mov eax, dword ptr ss:[ebp+C]
00D21088     8BF9                 mov edi, ecx
00D2108A     8BD9                 mov ebx, ecx
00D2108C     8955 FC              mov dword ptr ss:[ebp-4], edx
00D2108F     99                   cdq
00D21090     F7FE                 idiv esi
00D21092     8B45 08              mov eax, dword ptr ss:[ebp+8]
00D21095     8BF2                 mov esi, edx
00D21097     99                   cdq
00D21098     F7FF                 idiv edi
00D2109A     8BF8                 mov edi, eax
00D2109C     8B45 0C              mov eax, dword ptr ss:[ebp+C]
00D2109F     99                   cdq
00D210A0     0FAFFE               imul edi, esi
00D210A3     F7FB                 idiv ebx
00D210A5     0FAF75 FC            imul esi, dword ptr ss:[ebp-4]
00D210A9     33D2                 xor edx, edx
00D210AB     0FAF45 FC            imul eax, dword ptr ss:[ebp-4]
00D210AF     03C7                 add eax, edi
00D210B1     5F                   pop edi
00D210B2     F7F1                 div ecx
00D210B4     B9 00E1F505          mov ecx, 5F5E100
00D210B9     8BC2                 mov eax, edx
00D210BB     33D2                 xor edx, edx
00D210BD     69C0 10270000        imul eax, eax, 2710
00D210C3     03C6                 add eax, esi
00D210C5     5E                   pop esi
00D210C6     F7F1                 div ecx
00D210C8     5B                   pop ebx
00D210C9     8BC2                 mov eax, edx
00D210CB     C9                   leave
00D210CC     C2 0800              retn 8
```

将`00D21071`到`00D210CC`之间的代码也用二进制复制的方法复制下来备用。

### 2. 修复 IAT

(5) 修复`iat`。

将`dump`出来的文件`oep`改为`004d874c`，用`od`加载，如下：

```assembly
004D874C z>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753       B8 4C814D00          mov eax, zy2.004D814C
004D8758       E8 3BE1F2FF          call zy2.00406898 ; f7 进入

00406898       50                   push eax
00406899       6A 00                push 0
0040689B       E8 F8FEFFFF          call zy2.00406798 ; f7 进入

004068A0   -   E9 B0C5F902          jmp zy2.033A2E55

00406798       FF25 60793E00        jmp dword ptr ds:[3E7960] ; 这些地址就是 iat, 但都在 400000 以内，我把它修改一下。

0040679E       8BC0                 mov eax, eax
004067A0       FF25 947B3E00        jmp dword ptr ds:[3E7B94]
004067A6       8BC0                 mov eax, eax
004067A8       FF25 E07A3E00        jmp dword ptr ds:[3E7AE0]
004067AE       8BC0                 mov eax, eax
004067B0       FF25 747E3E00        jmp dword ptr ds:[3E7E74]
```

用`winhex`打开修复了`cc`的文件，在`401000`---`4d9000`范围内，用查找和替换功能，用`FF25FFFF4E00`替换`FF25FFFF3E00`，中间的`ffff`是通配符，设通配符为`ff`。用`winhex`打开保存下来的`iat`原始文本文件，用查找和替换功能，用`000E`替换`FFFD`，`ok`，下一步运行`ImportREC 1.6`，提取修改过的`iat`原始文本文件，`FixDump`。

这中间可能遇到`iat`的首地址每次运行都不同的问题，多做两遍，或是用复制加粘贴的方法，用`ImportREC 1.6`在我们想要得地址处提取`iat`原始文件。

### 3. 以壳脱壳与最终修复

以壳脱壳，修复`iat`用`od`加载`dump`文件，如下：

**修改前：**

```assembly
004D874C z>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753       B8 4C814D00          mov eax, zy2.004D814C
004D8758       E8 3BE1F2FF          call zy2.00406898
```

**修改后：**

```assembly
004D874C u>   55                   push ebp
004D874D       8BEC                 mov ebp, esp
004D874F       83C4 F4              add esp, -0C
004D8752       53                   push ebx
004D8753   -   E9 A8290100          jmp un-Aspr.004EB100
004D8758       E8 3BE1F2FF          call un-Aspr.00406898
```

修改`004EB100`处的代码：

```assembly
004EB100       90                   nop
004EB101       90                   nop
004EB102       90                   nop
004EB103    .  C785 6CD1FFFF 0B020000   mov dword ptr ss:[ebp-2E94], 20B
004EB10D    .  C785 ECD6FFFF 40794E00   mov dword ptr ss:[ebp-2914], <&kernel32.CloseHandle>
004EB117    .  C785 48D8FFFF 64020000   mov dword ptr ss:[ebp-27B8], 264
004EB121    .  C785 68D1FFFF 731E4400   mov dword ptr ss:[ebp-2E98], un-Aspr.00441E73
004EB12B    .  EB 21                jmp short un-Aspr.004EB14E
004EB12D       90                   nop
004EB12E       90                   nop
004EB12F       90                   nop
004EB130       90                   nop
004EB131       90                   nop
004EB132       90                   nop
004EB133       90                   nop
004EB134       90                   nop
004EB135       90                   nop
004EB136       90                   nop
004EB137       90                   nop
004EB138       90                   nop
004EB139       90                   nop
004EB13A       90                   nop
004EB13B       90                   nop
004EB13C       90                   nop
004EB13D       90                   nop
004EB13E       90                   nop
004EB13F       90                   nop
004EB140       90                   nop
004EB141    >  8B85 64D1FFFF          mov eax, dword ptr ss:[ebp-2E9C] ; 前面 dump 出来的代码 1 开始
004EB147    .  40                   inc eax
004EB148    .  8985 64D1FFFF          mov dword ptr ss:[ebp-2E9C], eax
004EB14E    >  8B85 64D1FFFF          mov eax, dword ptr ss:[ebp-2E9C]
004EB154    .  3B85 6CD1FFFF          cmp eax, dword ptr ss:[ebp-2E94]
004EB15A    .  0F83 9A000000         jnb un-Aspr.004EB1FA
004EB160    .  68 0D5EDF01           push un-Aspr.01DF5E0D
004EB165    .  FFB5 68D1FFFF         push dword ptr ss:[ebp-2E98]
004EB16B    .  8D8D 68D1FFFF         lea ecx, dword ptr ss:[ebp-2E98]
004EB171    .  E8 AD000000           call un-Aspr.004EB223
004EB176    .  40                   inc eax
004EB177    .  33D2                 xor edx, edx
004EB179    .  B9 00E1F505          mov ecx, 5F5E100
004EB17E    .  F7F1                 div ecx
004EB180    .  8995 68D1FFFF         mov dword ptr ss:[ebp-2E98], edx
004EB186    .  8B85 68D1FFFF         mov eax, dword ptr ss:[ebp-2E98]
004EB18C    .  33D2                 xor edx, edx
004EB18E    .  B9 10270000          mov ecx, 2710
004EB193    .  F7F1                 div ecx
004EB195    .  0FAF85 48D8FFFF      imul eax, dword ptr ss:[ebp-27B8]
004EB19C    .  33D2                 xor edx, edx
004EB19E    .  B9 10270000          mov ecx, 2710
004EB1A3    .  F7F1                 div ecx
```
