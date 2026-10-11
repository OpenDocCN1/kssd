
```
01237A5B 59 pop ecx
01237A5C C3 retn //返回到伪 Oep
```

```
0044E3DE 0000 add byte ptr ds:[eax], al //真 Oep
0044E3E0 0000 add byte ptr ds:[eax], al
0044E3E2 0000 add byte ptr ds:[eax], al
0044E3E4 0000 add byte ptr ds:[eax], al
0044E3E6 0000 add byte ptr ds:[eax], al
0044E3E8 0000 add byte ptr ds:[eax], al
0044E3EA 0000 add byte ptr ds:[eax], al
0044E3EC 0000 add byte ptr ds:[eax], al
0044E3EE 0000 add byte ptr ds:[eax], al
0044E3F0 0000 add byte ptr ds:[eax], al
0044E3F2 0000 add byte ptr ds:[eax], al
0044E3F4 0000 add byte ptr ds:[eax], al
0044E3F6 0000 add byte ptr ds:[eax], al
0044E3F8 0000 add byte ptr ds:[eax], al
0044E3FA 0000 add byte ptr ds:[eax], al
0044E3FC 0000 add byte ptr ds:[eax], al
0044E3FE 0000 add byte ptr ds:[eax], al
0044E400 0000 add byte ptr ds:[eax], al
0044E402 0000 add byte ptr ds:[eax], al
0044E404 0000 add byte ptr ds:[eax], al
0044E404 FF15 EC314700 call dword ptr ds:[4731EC] //伪 Oep
0044E40A 33D2 xor edx, edx
0044E40C 8AD4 mov dl, ah
0044E40E 8915 28F44900 mov dword ptr ds:[49F428], edx
0044E414 8BC8 mov ecx, eax
0044E416 81E1 FF000000 and ecx, 0FF
0044E41C 890D 24F44900 mov dword ptr ds:[49F424], ecx
0044E422 C1E1 08 shl ecx, 8
0044E425 03CA add ecx, edx
0044E427 890D 20F44900 mov dword ptr ds:[49F420], ecx
0044E42D C1E8 10 shr eax, 10
0044E430 A3 1CF44900 mov dword ptr ds:[49F41C], eax
0044E435 6A 01 push 1
0044E437 E8 882C0000 call VideoSpl.004510C4
```

..........................................................................................

整理一下 Stolen Code

.......................................................

```
push ebp //Stolen Code
mov ebp, esp
push -1
push 4822F0
push 44E000 //一共 13 句代码
mov eax, dword ptr fs:[0]
push eax
mov dword ptr fs:[0], esp
sub esp, 58
push ebx
push esi
push edi
mov dword ptr ss:[ebp-18], esp //Stolen Code
```

.......................................................

```
0044E3DE 55 push ebp //补好后复制代码到文件中，直接用 Loadpe 脱壳。
0044E3DF 8BEC mov ebp, esp
0044E3E1 6A FF push -1
0044E3E3 68 402E4800 push VideoSpl.00482E40
0044E3E8 68 B8E54400 push VideoSpl.0044E5B8
0044E3ED 64:A1 00000000 mov eax, dword ptr fs:[0]
0044E3F3 50 push eax
0044E3F4 64:8925 0000000>mov dword ptr fs:[0], esp
0044E3FB 83EC 58 sub esp, 58
0044E3FE 53 push ebx
0044E3FF 56 push esi
0044E400 57 push edi
0044E401 8965 E8 mov dword ptr ss:[ebp-18], esp
0044E404 FF15 EC314700 call dword ptr ds:[4731EC]
0044E40A 33D2 xor edx, edx
0044E40C 8AD4 mov dl, ah
0044E40E 8915 28F44900 mov dword ptr ds:[49F428], edx
0044E414 8BC8 mov ecx, eax
0044E416 81E1 FF000000 and ecx, 0FF
0044E41C 890D 24F44900 mov dword ptr ds:[49F424], ecx
0044E422 C1E1 08 shl ecx, 8
0044E425 03CA add ecx, edx
0044E427 890D 20F44900 mov dword ptr ds:[49F420], ecx
0044E42D C1E8 10 shr eax, 10
0044E430 A3 1CF44900 mov dword ptr ds:[49F41C], eax
0044E435 6A 01 push 1
0044E437 E8 882C0000 call VideoSpl.004510C4
```

.......................................................

### IAT 修复

运行 `ImportREC`，`OEP` 填入 `4E3DE`，自动搜索，获得输入信息，先用跟踪等级 1 修复，然后用 `Asprotect1.3` 插件修复剩下的几个指针，修复先脱壳的 `Dumped.exe`。

### 第二部分，去除暗桩

`OD` 载入修复的文件，`F9` 运行。

状态栏/提示内存写错误。

堆栈友好提示

```
0012AFD8 0040DD53 返回到 dumped\_.0040DD53 来自 01221C08 //错误来自 0040DD53
0012AFDC 00474758 ASCII ".RMF"
0012AFE0 00000004
0012AFE4 00000000
0012AFE8 00496850 dumped\_.00496850
```

.......................................................

那么 `Ctrl+G` 去 `0040DD53` 看看。

```
0040DD4D |. FF15 8CD64800 call dword ptr ds:[48D68C] //ds:[48D68C]指向壳中,跟踪原程序发现其作用是简单的防脱壳校验。
0040DD53 |. E8 13F10500 call dumped\_.0046CE6B //无可疑
```

`Od` 载入原程序

```
0040DD4D |. FF15 8CD64800 call dword ptr ds:[48D68C]
```

```
01221C08 55 push ebp //F7 跟进
01221C09 8BEC mov ebp, esp
01221C0B 53 push ebx
01221C0C 56 push esi
01221C0D 8B5D 0C mov ebx, dword ptr ss:[ebp+C]
01221C10 8B75 08 mov esi, dword ptr ss:[ebp+8]
01221C13 8BC3 mov eax, ebx
01221C15 E8 F208FFFF call 0121250C
01221C1A A3 A4632201 mov dword ptr ds:[12263A4], eax
01221C1F 8B15 A4632201 mov edx, dword ptr ds:[12263A4]
01221C25 8BC6 mov eax, esi
01221C27 8BCB mov ecx, ebx
01221C29 E8 EA09FFFF call 01212618
01221C2E 891D 347E2201 mov dword ptr ds:[1227E34], ebx
01221C34 53 push ebx
01221C35 8BCE mov ecx, esi
01221C37 8B15 307E2201 mov edx, dword ptr ds:[1227E30]
01221C3D A1 2C7E2201 mov eax, dword ptr ds:[1227E2C]
01221C42 E8 F9EEFFFF call 01220B40
01221C47 833D E47A2201 0>cmp dword ptr ds:[1227AE4], 0
01221C4E 75 05 jnz short 01221C55
01221C50 E8 4FFFFFFF call 01221BA4
01221C55 5E pop esi
01221C56 5B pop ebx
01221C57 5D pop ebp
01221C58 C2 0800 retn 8 //跟进，晕，好多代码啊，壳是不可能搬回的,它进壳中的过程我们没必有知道，但它从壳中取出代码动态修改程序领空代码我们脱壳后是需要的,可是这个也不是解码 Call。
```

`Ctrl+F` 在程序空间里查找命令：`retn 8` 随便找一个吧：

命令行 `DD 48D68C`

把 `48D68C` 处的 `01221C08` 改为：`00403B74`

复制到程序里，保存覆盖文件。

`Ctrl+F2` 重新加载，`F9` 运行。

程序界面出来了，尝试分割文件立即出错。

状态栏/提示内存写错误。

堆栈友好提示

```
0012ADB8 0041116A 返回到 dumped\_.0041116A 来自 01221BA4
0012ADBC 0012B8E8
0012ADC0 0012B8E8
0012ADC4 0040FF02 返回到 dumped\_.0040FF02 来自 dumped\_.00411160
0012ADC8 0012B8E8
0012ADCC 00474A68 dumped\_.00474A68
```

那么 `Ctrl+G` 去 `0041116A` 看看

```
00411164 . FF15 84D64800 call dword ptr ds:[48D684] //这里搞的飞机,ds:[48D684]指向壳中,跟踪原程序发现其作用是动态解码。
0041116A . E9 76000000 jmp dumped\_.004111E5
```

Btw:用 `Winxp` 脱壳时这里堆栈出错没有提示。

```
01224350 3100 xor dword ptr ds:[eax], eax //只有再硬盘指纹过后的 pre-dip 中分析了。
01224352 EB 01 jmp short 01224355
01224354 68 648F0500 push 58F64
01224359 0000 add byte ptr ds:[eax], al
0122435B 00EB add bl, ch
0122435D 02E8 add ch, al
0122435F 0158 80 add dword ptr ds:[eax-80], ebx
01224362 3D A07E2201 cmp eax, 1227EA0
01224367 007414 E8 add byte ptr ss:[esp+edx-18], dh
0122436B 2D D2FFFF84 sub eax, 84FFFFD2
01224370 C0740B E8 F0 sal byte ptr ds:[ebx+ecx-18], 0F0
01224375 DEFF fdivp st(7), st
01224377 FF84C0 740289EC inc dword ptr ds:[eax+eax*8+EC890274]
0122437E B2 01 mov dl, 1
01224380 A1 140C2201 mov eax, dword ptr ds:[1220C14]
01224385 E8 06C9FFFF call 01220C90
```

..........................................................

```
0012FF3C 0012FF44 指针到下一个 SEH 记录
0012FF40 01224307 SE 句柄
0012FF44 0012FFE0 指针到下一个 SEH 记录
0012FF48 01224C49 SE 句柄
```
