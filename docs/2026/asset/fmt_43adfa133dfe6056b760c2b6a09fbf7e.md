
```asm
005F2442   |.  BA 05000000        mov edx,5                            ;  5
005F2447   |.  8B45 F4            mov eax,dword ptr ss:[ebp-C]         ;  Rev\_NameASCII
005F244A   |.  E8 AD2CE1FF        call Member.004050FC                 ;  System.@LStrCopy;
005F244F   |.  8B45 F8            mov eax,dword ptr ss:[ebp-8]         ;  Rev\_NameASCII(1-4)
005F2452   |.  E8 452AE1FF        call Member.00404E9C                 ;  System.@LStrLen(String):Integer;
005F2457   |.  83F8 04            cmp eax,4
005F245A   |.  7D 2F              jge short Member.005F248B            ;  Length(Rev\_NameASCII(1-4))>=4跳
005F245C   |.  8B45 F8            mov eax,dword ptr ss:[ebp-8]         ;  否则
005F245F   |.  E8 382AE1FF        call Member.00404E9C                 ;  System.@LStrLen(String):Integer;
005F2464   |.  8BD8               mov ebx,eax                          ;  EBX=Length(Rev\_NameASCII(1-4))
005F2466   |.  83FB 03            cmp ebx,3                            ;  >3
005F2469   |.  7F 20              jg short Member.005F248B             ;  废话，这句永远不会成立!
005F246B   |>  8D4D E4            /lea ecx,dword ptr ss:[ebp-1C]
005F246E   |.  8BC3               |mov eax,ebx                         ;  ebx=Length(Rev\_NameASCII(1-4))
005F2470   |.  C1E0 02            |shl eax,2                           ;  长度×4
005F2473   |.  33D2               |xor edx,edx
005F2475   |.  E8 4E7EE1FF        |call Member.0040A2C8                ;  SysUtils.IntToHex(Integer;Integer)
005F247A   |.  8B55 E4            |mov edx,dword ptr ss:[ebp-1C]       ;  IntToHex(长度×4)
005F247D   |.  8D45 F8            |lea eax,dword ptr ss:[ebp-8]
005F2480   |.  E8 1F2AE1FF        |call Member.00404EA4                ;  System.@LStrCat;
005F2485   |.  43                 |inc ebx                             ;  长度+1
005F2486   |.  83FB 04            |cmp ebx,4
005F2489   |.^ 75 E0              \jnz short Member.005F246B           ;  长度+1<>4继续循环
005F248B   |>  8B45 F4            mov eax,dword ptr ss:[ebp-C]         ;  Rev\_NameASCII(5-8)
005F248E   |.  E8 092AE1FF        call Member.00404E9C                 ;  System.@LStrLen(String):Integer;
005F2493   |.  83F8 04            cmp eax,4
005F2496   |.  7D 2F              jge short Member.005F24C7            ;  同上废话
005F2498   |.  8B45 F4            mov eax,dword ptr ss:[ebp-C]
005F249B   |.  E8 FC29E1FF        call Member.00404E9C                 ;  System.@LStrLen(String):Integer;
005F24A0   |.  8BD8               mov ebx,eax
005F24A2   |.  83FB 03            cmp ebx,3
005F24A5   |.  7F 20              jg short Member.005F24C7
005F24A7   |>  8D4D E0            /lea ecx,dword ptr ss:[ebp-20]
005F24AA   |.  8BC3               |mov eax,ebx
005F24AC   |.  C1E0 02            |shl eax,2
005F24AF   |.  33D2               |xor edx,edx
005F24B1   |.  E8 127EE1FF        |call Member.0040A2C8                ;  SysUtils.IntToHex(Integer;Integer):AnsiString;
005F24B6   |.  8B55 E0            |mov edx,dword ptr ss:[ebp-20]
005F24B9   |.  8D45 F4            |lea eax,dword ptr ss:[ebp-C]
005F24BC   |.  E8 E329E1FF        |call Member.00404EA4                ;  System.@LStrCat;
005F24C1   |.  43                 |inc ebx
005F24C2   |.  83FB 04            |cmp ebx,4
005F24C5   |.^ 75 E0              \jnz short Member.005F24A7
005F24C7   |>  8D45 F0            lea eax,dword ptr ss:[ebp-10]
005F24CA   |.  BA 54255F00        mov edx,Member.005F2554              ;  固定字符串"mem45erpe"，记为Mark
005F24CF   |.  E8 A027E1FF        call Member.00404C74
005F24D4   |.  8D45 DC            lea eax,dword ptr ss:[ebp-24]        ;  保存Mark(1-4)='mem4'
005F24D7   |.  50                 push eax
005F24D8   |.  B9 04000000        mov ecx,4                            ;  4
005F24DD   |.  BA 01000000        mov edx,1                            ;  1
005F24E2   |.  8B45 F0            mov eax,dword ptr ss:[ebp-10]        ;  Mark
005F24E5   |.  E8 122CE1FF        call Member.004050FC                 ;  System.@LStrCopy;
005F24EA   |.  FF75 DC            push dword ptr ss:[ebp-24]           ;  Mark(1-4)
005F24ED   |.  68 68255F00        push Member.005F2568                 ;  分割字符'-'
005F24F2   |.  FF75 F8            push dword ptr ss:[ebp-8]            ;  Rev\_NameASCII(1-4)
005F24F5   |.  8D45 D8            lea eax,dword ptr ss:[ebp-28]        ;  保存Mark(5-9)
005F24F8   |.  50                 push eax
005F24F9   |.  B9 05000000        mov ecx,5                            ;  5
005F24FE   |.  BA 05000000        mov edx,5                            ;  5
005F2503   |.  8B45 F0            mov eax,dword ptr ss:[ebp-10]        ;  Mark
005F2506   |.  E8 F12BE1FF        call Member.004050FC                 ;  System.@LStrCopy;
005F250B   |.  FF75 D8            push dword ptr ss:[ebp-28]           ;  Mark(5-9)='45erpe'
005F250E   |.  68 68255F00        push Member.005F2568                 ;  分割字符'-'
005F2513   |.  FF75 F4            push dword ptr ss:[ebp-C]            ;  Rev\_NameASCII(5-8)
005F2516   |.  8BC7               mov eax,edi
005F2518   |.  BA 06000000        mov edx,6                            ;  6
005F251D   |.  E8 3A2AE1FF        call Member.00404F5C                 ;  System.@LStrCatN,连接上面6项入栈参数即为注册码
005F2522   |.  33C0               xor eax,eax
005F2524   |.  5A                 pop edx
005F2525   |.  59                 pop ecx
005F2526   |.  59                 pop ecx
005F2527   |.  64:8910            mov dword ptr fs:[eax],edx
005F252A   |.  68 44255F00        push Member.005F2544
005F252F   |>  8D45 D8            lea eax,dword ptr ss:[ebp-28]
005F2532   |.  BA 0A000000        mov edx,0A
005F2537   |.  E8 C426E1FF        call Member.00404C00
005F253C   \.  C3                 retn
005F253D    .^ E9 3A1FE1FF        jmp Member.0040447C
005F2542    .^ EB EB              jmp short Member.005F252F
005F2544    .  5F                 pop edi
005F2545    .  5E                 pop esi
005F2546    .  5B                 pop ebx
005F2547    .  8BE5               mov esp,ebp
005F2549    .  5D                 pop ebp
005F254A    .  C3                 retn
```

---

到注册算法关键了，是根据用户名计算结果连接一固定字符串作为注册码，具体分析请看注释。

### 算法总结

1、将注册名转换为ASCII字符串，如`Baby2008=4261627932303038`，记为`NameASCII`

2、将`NameASCII`倒序，记为`Rev_NameASCII`；如`4261627932303038`－>`8303032397261624`

3、取`Rev_NameASCII(1-4)`记为`R1`, `Rev_NameASCII(5-8)`记为`R2`,假如`R1`,`R2`长度不为4，用`Rigth('048C',4-Length(Ri))`补齐4位；

4、取固定字符串`Mark='mem45erpe'`的前4位记为`M1`，后5位记为`M2`

5、`M1-R1-M2-R2` (用分割字符`-`连接即为注册码)

### 注册机源代码

拿张ASCII码表翻翻、纸上写写就行了，立马省去注册费200RMB, ^\_^

我的注册信息：

- 注册名：`Baby2008`
- 注册码：`mem4-83035erpe-0323`

--完--

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
