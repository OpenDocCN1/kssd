
# 小破自由之翼.exe【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-7.htm`](https://www.pediy.com/kssd/pediy07/pediy7-7.htm)

- 软件名称：`自由之翼.exe`
- 壳：无
- 使用工具：`OD`, `PEID`, `p-regsnap271-625`
- 操作系统：`2k`
- 下载地址：从 QQ 群下的 :)
- 语言：`Borland Delphi 4.0 - 5.0`

## 1、查壳，无

## 2、试运行，无任何信息

## 3、用 `p-regsnap271-625` 比较注册表，发现

`HKEY_USERS\S-1-5-21-1482476501-436374069-1343024091-500\Software\aCaFeeL\CrackMe\Reg\code`

- Old value: String: `"9988776655"`
- New value: String: `"987654321"`

`HKEY_USERS\S-1-5-21-1482476501-436374069-1343024091-500\Software\aCaFeeL\CrackMe\Reg\name`

- Old value: String: `"Kerfier"`
- New value: String: `"hgfdsa"`

有不同的地方，键名分别是 `name` 和 `code`

## 4、用 `OD` 载入，查找到 `name`，`code`，有两处，经判断，以下几处为读注册表处，下断，跟踪，得到结果如下

```
0048F574 /. 55 push ebp
0048F575 |. 8BEC mov ebp,esp
0048F577 |. 6A 00 push 0
0048F579 |. 6A 00 push 0
0048F57B |. 33C0 xor eax,eax
0048F57D |. 55 push ebp
0048F57E |. 68 08F64800 push 自由之翼.0048F608
0048F583 |. 64:FF30 push dword ptr fs:[eax]
0048F586 |. 64:8920 mov dword ptr fs:[eax],esp
0048F589 |. B9 1CF64800 mov ecx,自由之翼.0048F61C ; ASCII "Software\aCaFeeL\CrackMe"
0048F58E |. B2 01 mov dl,1
0048F590 |. A1 08E94800 mov eax,dword ptr ds:[48E908]
0048F595 |. E8 56FAFFFF call 自由之翼.0048EFF0 ; 读注册表
0048F59A |. A3 44394900 mov dword ptr ds:[493944],eax
0048F59F |. 6A 00 push 0
0048F5A1 |. 8D45 FC lea eax,dword ptr ss:[ebp-4]
0048F5A4 |. 50 push eax
0048F5A5 |. B9 40F64800 mov ecx,自由之翼.0048F640 ; ASCII "name"
0048F5AA |. BA 50F64800 mov edx,自由之翼.0048F650 ; ASCII "Reg"
0048F5AF |. A1 44394900 mov eax,dword ptr ds:[493944]
0048F5B4 |. E8 CBFAFFFF call 自由之翼.0048F084 ; 读注册名
0048F5B9 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
0048F5BC |. B8 48394900 mov eax,自由之翼.00493948
0048F5C1 |. E8 7245F7FF call 自由之翼.00403B38 ; 转存注册名
0048F5C6 |. 6A 00 push 0
0048F5C8 |. 8D45 F8 lea eax,dword ptr ss:[ebp-8]
0048F5CB |. 50 push eax
0048F5CC |. B9 5CF64800 mov ecx,自由之翼.0048F65C ; ASCII "code"
0048F5D1 |. BA 50F64800 mov edx,自由之翼.0048F650 ; ASCII "Reg"
0048F5D6 |. A1 44394900 mov eax,dword ptr ds:[493944]
0048F5DB |. E8 A4FAFFFF call 自由之翼.0048F084 ; 读注册码
0048F5E0 |. 8B55 F8 mov edx,dword ptr ss:[ebp-8]
0048F5E3 |. B8 4C394900 mov eax,自由之翼.0049394C
0048F5E8 |. E8 4B45F7FF call 自由之翼.00403B38 ; 转存注册码
0048F5ED |. 33C0 xor eax,eax
0048F5EF |. 5A pop edx
0048F5F0 |. 59 pop ecx
0048F5F1 |. 59 pop ecx
0048F5F2 |. 64:8910 mov dword ptr fs:[eax],edx
0048F5F5 |. 68 0FF64800 push 自由之翼.0048F60F
0048F5FA |> 8D45 F8 lea eax,dword ptr ss:[ebp-8]
0048F5FD |. BA 02000000 mov edx,2
0048F602 |. E8 0145F7FF call 自由之翼.00403B08 ; 判断是否存在用户名和注册码
0048F607 \. C3 retn
0048F602 |. E8 0145F7FF call 自由之翼.00403B08 ; 判断是否存在用户名和注册码

call 跟进

00403B08 53 push ebx
00403B09 56 push esi
00403B0A 89C3 mov ebx,eax
00403B0C 89D6 mov esi,edx
00403B0E 8B13 mov edx,dword ptr ds:[ebx]
00403B10 85D2 test edx,edx
00403B12 74 1A je short 自由之翼.00403B2E
00403B14 C703 00000000 mov dword ptr ds:[ebx],0
00403B1A 8B4A F8 mov ecx,dword ptr ds:[edx-8]
00403B1D 49 dec ecx
00403B1E 7C 0E jl short 自由之翼.00403B2E
00403B20 F0:FF4A F8 lock dec dword ptr ds:[edx-8]
00403B24 75 08 jnz short 自由之翼.00403B2E
00403B26 8D42 F8 lea eax,dword ptr ds:[edx-8]
00403B29 E8 F6EBFFFF call 自由之翼.00402724
00403B2E 83C3 04 add ebx,4
00403B31 4E dec esi
00403B32 ^ 75 DA jnz short 自由之翼.00403B0E
00403B34 5E pop esi
00403B35 5B pop ebx
00403B36 C3 retn
0048F607 \. C3 retn

retn 返回到

00432717 33C0 xor eax,eax
00432719 5A pop edx
0043271A 59 pop ecx
0043271B 59 pop ecx
0043271C 64:8910 mov dword ptr fs:[eax],edx
0043271F EB 17 jmp short 自由之翼.00432738 ; 跳转
00432721 ^ E9 A20BFDFF jmp 自由之翼.004032C8
00432726 8B55 FC mov edx,dword ptr ss:[ebp-4]
00432729 A1 64374900 mov eax,dword ptr ds:[493764]

跳到

00432738 \8B45 FC mov eax,dword ptr ss:[ebp-4] ; 跳到这儿
0043273B F680 CC020000 02 test byte ptr ds:[eax+2CC],2
00432742 74 0A je short 自由之翼.0043274E ; 跳转
00432744 B2 01 mov dl,1
00432746 8B45 FC mov eax,dword ptr ss:[ebp-4]
00432749 E8 EA080000 call 自由之翼.00433038
0043274E 5F pop edi ; 到这儿
0043274F 5E pop esi
00432750 5B pop ebx
00432751 59 pop ecx
00432752 5D pop ebp
00432753 C3 retn

retn 返回

00432413 F686 CC020000 20 test byte ptr ds:[esi+2CC],20
0043241A 74 12 je short 自由之翼.0043242E ; 跳转
0043241C 8BC6 mov eax,esi
0043241E 66:BB B6FF mov bx,0FFB6
00432422 E8 D10BFDFF call 自由之翼.00402FF8
00432427 80A6 CC020000 DF and byte ptr ds:[esi+2CC],0DF
0043242E 5E pop esi
0043242F 5B pop ebx
00432430 C3 retn

retn 返回，中间再经过几次 retn 到这儿

00439138 33C0 xor eax,eax
0043913A 5A pop edx
0043913B 59 pop ecx
0043913C 59 pop ecx
0043913D 64:8910 mov dword ptr fs:[eax],edx
00439140 EB 16 jmp short 自由之翼.00439158
00439142 ^ E9 81A1FCFF jmp 自由之翼.004032C8
00439147 8B45 FC mov eax,dword ptr ss:[ebp-4]
0043914A 33D2 xor edx,edx
0043914C 8910 mov dword ptr ds:[eax],edx
0043914E E8 7DA4FCFF call 自由之翼.004035D0
00439153 E8 CCA4FCFF call 自由之翼.00403624
00439158 837E 38 00 cmp dword ptr ds:[esi+38],0
0043915C 75 1D jnz short 自由之翼.0043917B 不跳
0043915E 8BC3 mov eax,ebx
00439160 8B15 54F34200 mov edx,dword ptr ds:[42F354] ; 自由之翼.0042F3A0
00439166 E8 299EFCFF call 自由之翼.00402F94
0043916B 84C0 test al,al
0043916D 74 0C je short 自由之翼.0043917B ; 是否就是该句
0043916F 8BFB mov edi,ebx
00439171 8BC7 mov eax,edi
00439173 E8 403B0100 call 自由之翼.0044CCB8
00439178 897E 38 mov dword ptr ds:[esi+38],edi
0043917B 5F pop edi
0043917C 5E pop esi
0043917D 5B pop ebx
0043917E 59 pop ecx
0043917F 5D pop ebp
00439180 C3 retn

retn 返回

00490043 . A1 6C234900 mov eax,dword ptr ds:[49236C]
00490048 . 8B00 mov eax,dword ptr ds:[eax]
0049004A . E8 3591FAFF call 自由之翼.00439184

这句 call 就直接运行程序了，由于一直没有看到处理用户名和注册码的地方，因此 F7 跟进

00439184 55 push ebp ; call 跟进
00439185 8BEC mov ebp,esp
00439187 51 push ecx
00439188 8945 FC mov dword ptr ss:[ebp-4],eax
0043918B 8B45 FC mov eax,dword ptr ss:[ebp-4]
0043918E C680 95000000 01 mov byte ptr ds:[eax+95],1
00439195 33D2 xor edx,edx
00439197 55 push ebp
00439198 68 2E924300 push 自由之翼.0043922E
0043919D 64:FF32 push dword ptr fs:[edx]
004391A0 64:8922 mov dword ptr fs:[edx],esp
004391A3 B8 0C044300 mov eax,自由之翼.0043040C
004391A8 E8 73EFFCFF call 自由之翼.00408120 ; 没什么东西
004391AD 8B45 FC mov eax,dword ptr ss:[ebp-4]
004391B0 8B40 38 mov eax,dword ptr ds:[eax+38]
004391B3 85C0 test eax,eax
004391B5 74 5F je short 自由之翼.00439216 ; 不跳
004391B7 8B15 30234900 mov edx,dword ptr ds:[492330] ; 自由之翼.00493028
004391BD 8B12 mov edx,dword ptr ds:[edx]
004391BF 83EA 03 sub edx,3
004391C2 74 0E je short 自由之翼.004391D2 ; 不跳
004391C4 83EA 04 sub edx,4
004391C7 75 10 jnz short 自由之翼.004391D9 ; 跳
004391C9 C680 13020000 01 mov byte ptr ds:[eax+213],1
004391D0 EB 07 jmp short 自由之翼.004391D9
004391D2 B2 02 mov dl,2
004391D4 E8 D3ACFFFF call 自由之翼.00433EAC
004391D9 8B45 FC mov eax,dword ptr ss:[ebp-4] ; 到这儿
004391DC 8078 4B 00 cmp byte ptr ds:[eax+4B],0
004391E0 74 20 je short 自由之翼.00439202 ; 不跳
004391E2 8B45 FC mov eax,dword ptr ss:[ebp-4]
004391E5 8B40 38 mov eax,dword ptr ds:[eax+38]
004391E8 80B8 13020000 01 cmp byte ptr ds:[eax+213],1
004391EF 75 0A jnz short 自由之翼.004391FB ; 跳
004391F1 8B45 FC mov eax,dword ptr ss:[ebp-4]
004391F4 E8 83F9FFFF call 自由之翼.00438B7C
004391F9 EB 07 jmp short 自由之翼.00439202
004391FB B2 01 mov dl,1 ; 到这儿
004391FD E8 369EFFFF call 自由之翼.00433038 ; 注册判断
00439202 8B45 FC mov eax,dword ptr ss:[ebp-4]
00439205 E8 E6FDFFFF call 自由之翼.00438FF0 ; 程序初始化运行

结果呢，陷入死循环，本菜鸟还没有找到他的防调试方式，于是换一种方法

在

0048F574 /. 55 push ebp
0048F575 |. 8BEC mov ebp,esp
0048F577 |. 6A 00 push 0
```



## 汇编代码分析

```
0048F579  |.  6A 00  push 0  
0048F57B  |.  33C0 xor eax,eax  
0048F57D  |.  55  push ebp  
0048F57E  |.  68 08F64800  push 自由之翼.0048F608  
0048F583  |.  64:FF30  push dword ptr fs:[eax]  
0048F586  |.  64:8920  mov dword ptr fs:[eax],esp  
0048F589  |.  B9 1CF64800  mov ecx,自由之翼.0048F61C ; ASCII "Software\\aCaFeeL\\CrackMe"  
0048F58E  |.  B2 01  mov dl,1  
0048F590  |.  A1 08E94800  mov eax,dword ptr ds:[48E908]  
0048F595  |.  E8 56FAFFFF  call 自由之翼.0048EFF0 ; 读注册表  
0048F59A  |.  A3 44394900  mov dword ptr ds:[493944],eax  
0048F59F  |.  6A 00  push 0  
0048F5A1  |.  8D45 FC  lea eax,dword ptr ss:[ebp-4]  
0048F5A4  |.  50  push eax  
0048F5A5  |.  B9 40F64800  mov ecx,自由之翼.0048F640 ; ASCII "name"  
0048F5AA  |.  BA 50F64800  mov edx,自由之翼.0048F650 ; ASCII "Reg"  
0048F5AF  |.  A1 44394900  mov eax,dword ptr ds:[493944]  
0048F5B4  |.  E8 CBFAFFFF  call 自由之翼.0048F084 ; 读注册名  
0048F5B9  |.  8B55 FC  mov edx,dword ptr ss:[ebp-4]  
0048F5BC  |.  B8 48394900  mov eax,自由之翼.00493948  
0048F5C1  |.  E8 7245F7FF  call 自由之翼.00403B38 ; 转存注册名  
0048F5C6  |.  6A 00  push 0  
0048F5C8  |.  8D45 F8  lea eax,dword ptr ss:[ebp-8]  
0048F5CB  |.  50  push eax  
0048F5CC  |.  B9 5CF64800  mov ecx,自由之翼.0048F65C ; ASCII "code"  
0048F5D1  |.  BA 50F64800  mov edx,自由之翼.0048F650 ; ASCII "Reg"  
0048F5D6  |.  A1 44394900  mov eax,dword ptr ds:[493944]  
0048F5DB  |.  E8 A4FAFFFF  call 自由之翼.0048F084 ; 读注册码  
0048F5E0  |.  8B55 F8  mov edx,dword ptr ss:[ebp-8]  
0048F5E3  |.  B8 4C394900  mov eax,自由之翼.0049394C  
0048F5E8  |.  E8 4B45F7FF  call 自由之翼.00403B38 ; 转存注册码  
0048F5ED  |.  33C0  xor eax,eax  
0048F5EF  |.  5A  pop edx  
0048F5F0  |.  59  pop ecx  
0048F5F1  |.  59  pop ecx  
0048F5F2  |.  64:8910  mov dword ptr fs:[eax],edx  
0048F5F5  |.  68 0FF64800  push 自由之翼.0048F60F  
0048F5FA  |>  8D45 F8  lea eax,dword ptr ss:[ebp-8]  
0048F5FD  |.  BA 02000000  mov edx,2  
0048F602  |.  E8 0145F7FF  call 自由之翼.00403B08 ; 判断是否存在用户名和注册码  
0048F607  \\.  C3  retn  
```

### 调试步骤

在`0048F602`处：

```
0048F602  |.  E8 0145F7FF  call 自由之翼.00403B08  
```

下断，断到时，再`ALT+M`，`CTRL+B`，查找`Kerfier`（我输入的注册名是：`Kerfier`）。

找到后，再右键，断点-内存访问，`F9`继续运行程序，断到这里：

```
00403E9D  8B0E  mov ecx,dword ptr ds:[esi]  
00403E9F  8B1F  mov ebx,dword ptr ds:[edi]  
00403EA1  39D9  cmp ecx,ebx  
00403EA3  75  58  jnz short 自由之翼.00403EFD  
00403EA5  4A  dec edx  
00403EA6  74  15  je short 自由之翼.00403EBD  
```

但这附近没有类似的循环来算注册码，再`F9`，到这里：

```
0048F767  |>  /8B45 F4  /mov eax,dword ptr ss:[ebp-C] ; 用户名生成注册码  
0048F76A  |.  |33C9  |xor ecx,ecx  
0048F76C  |.  |8A4C38 FF  |mov cl,byte ptr ds:[eax+edi-1]  
0048F770  |.  |81C1 E01E0000  |add ecx,1EE0  
0048F776  |.  |8BC1  |mov eax,ecx  
0048F778  |.  |B9 1B000000  |mov ecx,1B  
0048F77D  |.  |99  |cdq  
0048F77E  |.  |F7F9  |idiv ecx  
0048F780  |.  |8BF2  |mov esi,edx  
0048F782  |.  |46  |inc esi  
0048F783  |.  |8D45 E0  |lea eax,dword ptr ss:[ebp-20]  
0048F786  |.  |8B55 F0  |mov edx,dword ptr ss:[ebp-10]  
0048F789  |.  |8A5432 FF  |mov dl,byte ptr ds:[edx+esi-1]  
0048F78D  |.  |E8 FA44F7FF  |call 自由之翼.00403C8C  
0048F792  |.  |8B55 E0  |mov edx,dword ptr ss:[ebp-20]  
0048F795  |.  |8D45 EC  |lea eax,dword ptr ss:[ebp-14]  
0048F798  |.  |E8 CF45F7FF  |call 自由之翼.00403D6C  
0048F79D  |.  |47  |inc edi  
0048F79E  |.  |4B  |dec ebx  
0048F79F  |.^\\75 C6  \\jnz short 自由之翼.0048F767  
```

### 分析与验证

这儿有个循环，看上去比较像算注册码的，于是跟踪，结果果然是。

在`0048F7A1`处`F4`，直接运行到这里，再`F8`到`0048F7A4`，马上在提示窗出现：

```
Stack ss:[0012FB88]=00D57E24, (ASCII "GfLG2fL")
```

这就是注册码。

### 算法描述

查表法

码表：`AaBbC0cDdEe1FfGgH2hIiJj3KkLMm4NnOoP5pQqRr6SsTtU7uVvWw8XxYyZ9`

输入为`X[i]`，

(`x[i]` + `0x1EE0`) / `0x1B`，得到商`T`，在码表中查到`T`所对应的字符。

循环，组成注册码。

我用的是`Kerfier:GfLG2fL`。

### 总结

从`BBS`上刚看到的一篇文章得到提示，在程序读入注册码后，在内存中找到存放注册码的地方，下“内存读取”断点，再`F9`运行，找到类似的运算注册码的代码块，再慢慢跟踪。

`4dd466d0e1e0cc123afbe504da2462ca`
