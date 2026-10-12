
# SVKP 1.3x -> Pavol Cerven 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-589.htm`](https://www.pediy.com/kssd/pediy07/pediy7-589.htm)

**[作者]** `aki`

**[文章题目]** `SVKP 1.3x -> Pavol Cerven` 脱壳

**[软件名称]** `冰橙外挂 1.31`

**[破解难度]** 初级 +++ 中级 +++ 高级 超难

**[破解平台]** `WinXP`

**[文章简介]** 玩传奇的同学让帮忙破个外挂，看了下是 `SVKP 1.3x -> Pavol Cerven` 的壳，以前没动过，索性研究了两天。收获不小，与兄弟们分享一下。

## [破解过程]

使用 `od` 载入，停在入口：

```
00453000  >  60                pushad
00453001    E8 00000000        call bczp.00453006
00453006    5D                pop ebp
00453007    81ED 06000000     sub ebp,6
0045300D    EB 05             jmp short bczp.00453014
0045300F    B8 49DCCA05       mov eax,5CADC49
00453014    64:A0 23000000    mov al,byte ptr fs:[23]
0045301A    EB 03             jmp short bczp.0045301F
```

插件隐藏 `od`，忽略所有异常，按 `f9` 运行，停在典型异常处：

```
0012E3B6    6285 1E220000     bound eax,qword ptr ss:[ebp+221E]
0012E3BC    EB 02             jmp short 0012E3C0
0012E3BE    0FE88B D1EB02CD  psubsb mm1,qword ptr ds:[ebx+CD02EBD1]
0012E3C5    208B C2EB02CD    and byte ptr ds:[ebx+CD02EBC2],cl
0012E3CB    208B 8A401600    and byte ptr ds:[ebx+16408A],cl
0012E3D1    008B 89740100    add byte ptr ds:[ebx+17489],cl
```

因为程序的主文件和一个主 `dll` 都加了相同的壳，所以 `shift＋f9` 后又来到典型异常处，不过这次是 `dll` 的异常：

```
0012BD42    6285 1E220000     bound eax,qword ptr ss:[ebp+221E]
0012BD48    EB 02             jmp short 0012BD4C
0012BD4A    0FE88B D1EB02CD  psubsb mm1,qword ptr ds:[ebx+CD02EBD1]
0012BD51    208B C2EB02CD    and byte ptr ds:[ebx+CD02EBC2],cl
0012BD57    208B 8A401600    and byte ptr ds:[ebx+16408A],cl
```

再一次 `shift＋f9` 就运行了。

重新来过，第一次典型异常后下断点：

`bp GetModuleHandleA＋5`

按 `shift＋f9` 断下，取消断点，`ctrl＋f9` 返回主程序。

使用 `ctrl＋f` 搜索特征码：

`cmp dword ptr ds:[ebx],2D66B1C5`

来到特殊 `api` 处理处：

```
071C5784    813B C5B1662D     cmp dword ptr ds:[ebx],2D66B1C5
071C578A    0F84 62180000     je 071C6FF2                  //jmp 071C57E4 跳过特殊处理
071C5790    813B 9404B2D9     cmp dword ptr ds:[ebx],D9B20494
071C5796    0F84 AA1C0000     je 071C7446
071C579C    813B A41A86D0     cmp dword ptr ds:[ebx],D0861AA4
071C57A2    0F84 58210000     je 071C7900
071C57A8    813B 706586B1     cmp dword ptr ds:[ebx],B1866570
071C57AE    0F84 C1240000     je 071C7C75
071C57B4    813B 0E46769B     cmp dword ptr ds:[ebx],9B76460E
071C57BA    0F84 36280000     je 071C7FF6
071C57C0    813B DB0793E6     cmp dword ptr ds:[ebx],E69307DB
071C57C6    0F84 76280000     je 071C8042
071C57CC    813B 627B6CA5     cmp dword ptr ds:[ebx],A56C7B62
071C57D2    0F84 BA280000     je 071C8092
071C57D8    813B 664E96BB     cmp dword ptr ds:[ebx],BB964E66
071C57DE    0F84 00290000     je 071C80E4
071C57E4    813B 4506D75B     cmp dword ptr ds:[ebx],5BD70645  //壳验证函数
071C57EA    0F84 43290000     je 071C8133
071C57F0    813B 0DE0FC1D     cmp dword ptr ds:[ebx],1DFCE00D //壳验证函数
071C57F6    0F84 83290000     je 071C817F
071C57FC    813B 31DD0F00     cmp dword ptr ds:[ebx],0FDD31   //壳验证函数
071C5802    0F84 C6290000     je 071C81CE
071C5808    813B 95B75126     cmp dword ptr ds:[ebx],2651B795
071C580E    0F84 132A0000     je 071C8227                   //jmp 071C5850
071C5814    813B B482F64B     cmp dword ptr ds:[ebx],4BF682B4
071C581A    0F84 582A0000     je 071C8278
071C5820    813B 0F1ACF4C     cmp dword ptr ds:[ebx],4CCF1A0F
071C5826    0F84 972A0000     je 071C82C3
071C582C    813B 4A7687DF     cmp dword ptr ds:[ebx],DF87764A
071C5832    0F84 FC2D0000     je 071C8634
071C5838    813B B8B8B2FB     cmp dword ptr ds:[ebx],FBB2B8B8
071C583E    0F84 56320000     je 071C8A9A
071C5844    813B 8E5D2D57     cmp dword ptr ds:[ebx],572D5D8E
071C584A    0F84 86320000     je 071C8AD6
071C5850    60                pushad
071C5851    8B03             mov eax,dword ptr ds:[ebx]
```

使用 `ctrl＋s` 继续搜索特征码：

`mov dword ptr ds:[edi],eax`

`popad`

来到普通 `api` 处理处：

```
071C5B4D    5F                pop edi
071C5B4E    58                pop eax
071C5B4F    8907             mov dword ptr ds:[edi],eax
071C5B51    61                popad
071C5B52    8385 43010200 04  add dword ptr ss:[ebp+20143],4
071C5B59  ^ E9 ADFBFFFF      jmp 071C570B
```

翻转一下改为：

```
popad
mov dword ptr ds:[edi],eax
```

如果在 `071C5B4F` 下断，可以看到 `api` 的还原。

按 `f9` 运行，断在 `dll` 的典型异常处。

下断点 `bp GetModuleHandleA＋5`。

按 `shift＋f9` 跳过异常后，`ctrl＋f9` 返回。

在 `12ffb0` 下硬件访问 word 断点，断下两三次后取消，下断点：

`tc ebp==12ffc0`

断在这里：

```
0041CC8A    CC                int3
0041CC8B    CC                int3
0041CC8C    68 08064200       push bczp.00420608
0041CC91    64:A1 00000000    mov eax,dword ptr fs:[0]
0041CC97    50                push eax
0041CC98    8B4424 10         mov eax,dword ptr ss:[esp+10]
0041CC9C    896C24 10         mov dword ptr ss:[esp+10],ebp
0041CCA0    8D6C24 10         lea ebp,dword ptr ss:[esp+10]
0041CCA4    2BE0             sub esp,eax                     //断在这里
0041CCA6    53                push ebx
0041CCA7    56                push esi
0041CCA8    57                push edi
0041CCA9    8B45 F8          mov eax,dword ptr ss:[ebp-8]
0041CCAC    8965 E8          mov dword ptr ss:[ebp-18],esp
0041CCAF    50                push eax
0041CCB0    8B45 FC          mov eax,dword ptr ss:[ebp-4]
0041CCB3    C745 FC FFFFFFFF mov dword ptr ss:[ebp-4],-1
0041CCBA    8945 F8          mov dword ptr ss:[ebp-8],eax
0041CCBD    8D45 F0          lea eax,dword ptr ss:[ebp-10]
0041CCC0    64:A3 00000000   mov dword ptr fs:[0],eax
```

查看一下运行跟踪：

```
072BEF57    014424 04         add dword ptr ss:[esp+4],eax
072BEF5B    58                pop eax
072BEF5C    83C4 04           add esp,4
072BEF5F  - FF6424 FC       jmp dword ptr ss:[esp-4]                ; bczp.004183E0
004183DB    E8 AC480000      call bczp.0041CC8C
0041CC8C    68 08064200      push bczp.00420608
0041CC91    64:A1 00000000   mov eax,dword ptr fs:[0]
0041CC97    50                push eax
0041CC98    8B4424 10         mov eax,dword ptr ss:[esp+10]
0041CC9C    896C24 10         mov dword ptr ss:[esp+10],ebp
0041CCA0    8D6C24 10         lea ebp,dword ptr ss:[esp+10]
0041CCA4    2BE0             sub esp,eax
```

我们到 `004183DB` 看看：

```
004183D4    90                nop
004183D5    90                nop
004183D6    90                nop
004183D7    90                nop
004183D8    90                nop
004183D9    90                nop
004183DA    90                nop
004183DB    E8 AC480000      call bczp.0041CC8C  //伪 oep
004183E0    BF 94000000      mov edi,94
```

根据堆栈补上 stolen code 后入口：

```
004183D4    6A 60            push 60
004183D6    68 A0EA4300      push bczp.0043EAA0
004183DB    E8 AC480000      call bczp.0041CC8C
004183E0    BF 94000000      mov edi,94
```

使用 `lordpe` 选中这个进程，`correct imagesize` 后 `dump` 下来。

使用 `importrec` 选中（注意打开调试选项），选中这个进程。

`oep：183d4`

自动搜索 `iat`，`getimport` 后有一个不能识别，看了一下在 `kernel32` 里没有 `ExitProcess`，应该就是这个了。

修复后运行 `dumped\_.exe` 出错，提示不能识别 `mirdll.dll` 里的 `GetVT` 函数。因为这个 `dll` 也是用 `SVKP 1.3x` 加的壳，所以看不到输出表，`cut` 这个 `api` 后重新修复一下，还是出错。

跟到这里：

```
0012FE2C  ^\\E2 FA           loopd short 0012FE28
0012FE2E    59              pop ecx
0012FE2F    5E              pop esi
0012FE30    FF15 A96B4000   call dword ptr ds:[406BA9]          //指向壳
0012FE36    81C4 70000000   add esp,70
```

这是在堆栈中运行，重新运行 `dumped\_` 跟踪一下：

```
00406968    52                push edx
00406969    51                push ecx
0040696A    81F1 00B96E98    xor ecx,986EB900
00406970    81F2 547E69CB    xor edx,CB697E54
00406976    51                push ecx
00406977    52                push edx
00406978    89E2             mov edx,esp
0040697A    81F1 00B9D400    xor ecx,0D4B900
```



## 反汇编分析与修复

`00406980` `add edx,ecx`  
`00406982` `push edx`  
`00406983` `retn` // 跳到堆栈解码  

跟踪原程序，写一段修复代码：

`004368D4` `nop`  
`004368D5` `nop`  
`004368D6` `mov edi,esi`  
`004368D8` `mov ebx,39BE4AB2`  
`004368DD` `push edi`  
`004368DE` `push ecx`  
`004368DF` `lods byte ptr ds:[esi]`  
`004368E0` `xor ebx,ecx`  
`004368E2` `xor al,bh`  
`004368E4` `ror al,cl`  
`004368E6` `xor al,bl`  
`004368E8` `stos byte ptr es:[edi]`  
`004368E9` `loopd short 4.004368DF`  
`004368EB` `pop ecx`  
`004368EC` `pop edi`  
`004368ED` `mov byte ptr ds:[4368D5],0`  
`004368F4` `retn`  

另一个地方还有这么一处：

`0040760F` `push edx`  
`00407610` `mov ecx,esp`  
`00407612` `xor edx,40767DBE`  
`00407618` `add ecx,edx`  
`0040761A` `jmp ecx` // 跳到堆栈执行  
`0040761C` `push eax`  
`0040761D` `and eax,CC446FA3`  
`0012FE08` `mov esi,edi`  
`0012FE0A` `add dword ptr ds:[eax+1E],edx`  
`0012FE0D` `mov dword ptr ds:[eax+2A],edi`  
`0012FE10` `xor al,al`  
`0012FE12` `call dword ptr ds:[402375]` ; 指向壳  

同样来一段修复代码：

`004368F9` `nop`  
`004368FA` `mov byte ptr ds:[4368F6],1`  
`00436901` `mov edi,esi`  
`00436903` `mov ebx,39BE4AB2`  
`00436908` `test al,al`  
`0043690A` `je short 4.0043693D`  
`0043690C` `push ecx`  
`0043690D` `push edi`  
`0043690E` `mov edi,4.0043699A`  
`00436913` `mov ecx,0C8`  
`00436918` `mov eax,esi`  
`0043691A` `repne scas dword ptr es:[edi]`  
`0043691C` `xor eax,eax`  
`0043691E` `mov dword ptr ds:[43699A],eax`  
`00436923` `pop edi`  
`00436924` `pop ecx`  
`00436925` `lods byte ptr ds:[esi]`  
`00436926` `xor ebx,ecx`  
`00436928` `xor al,bl`  
`0043692A` `rol al,cl`  
`0043692C` `xor al,bh`  
`0043692E` `stos byte ptr es:[edi]`  
`0043692F` `loopd short 4.00436925`  
`00436931` `mov byte ptr ds:[4368F6],0`  
`00436938` `retn`  
`00436939` `nop`  
`0043693A` `nop`  
`0043693B` `nop`  
`0043693C` `nop`  
`0043693D` `push ecx`  
`0043693E` `push edi`  
`0043693F` `mov edi,4.0043699A`  
`00436944` `mov ecx,0C8`  
`00436949` `mov eax,esi`  
`0043694B` `repne scas dword ptr es:[edi]`  
`0043694D` `mov edi,4.0043699A`  
`00436952` `mov ecx,0C8`  
`00436957` `xor eax,eax`  
`00436959` `repne scas dword ptr es:[edi]`  
`0043695B` `mov dword ptr ds:[43699A],esi`  
`00436961` `pop edi`  
`00436962` `pop ecx`  
`00436963` `push ecx`  
`00436964` `push edi`  
`00436965` `lods byte ptr ds:[esi]`  
`00436966` `xor ebx,ecx`  
`00436968` `xor al,bh`  
`0043696A` `ror al,cl`  
`0043696C` `xor al,bl`  
`0043696E` `stos byte ptr es:[edi]`  
`0043696F` `loopd short 4.00436965`  
`00436971` `mov byte ptr ds:[4368F6],0`  
`00436978` `pop edi`  
`00436979` `pop ecx`  
`0043697A` `retn`  
`0043697B` `nop`  

### 修改指针

改好了保存一下，用`winhex`修改这两个指针：

```
[00406BA9]=004368D6
[00402375]=004368FA
```

改过后程序还是不能运行，跟踪一下发现这里有问题：

`00422B0E` `int3`  
`00422B0F` `int3`  
`00422B10` `jmp dword ptr ds:[437300]`  
`00422B16` `jmp dword ptr ds:[<&wgs.RUN>]` ; `wgs.RUN`  
`00422B1C` `jmp dword ptr ds:[<&wgs.LOAD>]` ; `wgs.LOAD`  
`00422B22` `push esi`  

`jmp dword ptr ds:[437300]`应当是指向`mirdll`里的输出函数`GetVT`，前面说了，这个`dll`加了壳导致在修复输入表后不能隐式调用，修复这个`api`然后把`dll`壳脱了也不行，不知是不是我`dll`脱的时候出问题了。

### 编写替代代码

没办法，我们自己来吧。

找一段空地，我们找到`00436E3A`。

`00422B10` `jmp dword ptr ds:[437300]`

改为：

`00422B10` `/E9 25430100` `jmp d1.00436E3A`  
`00422B15` `|90` `nop`  

看一下要用的`api`的地址：

`0003720C` `kernel32.dll` `0191` `GetProcAddress`  
`00037208` `kernel32.dll` `023B` `LoadLibraryA`  

补充的代码：

`00436E3A` `pushad`  
`00436E3B` `push d1.004462C8` ; ASCII `"mirdll.dll"`  
`00436E40` `call dword ptr ds:[<&kernel32.LoadLibraryA>]` ; `kernel32.LoadLibraryA`  
`00436E46` `push d1.004462D8` ; ASCII `"GetVT"`  
`00436E4B` `push eax`  
`00436E4C` `call dword ptr ds:[<&kernel32.GetProcAddress>]` ; `kernel32.GetProcAddress`  
`00436E52` `mov dword ptr ds:[437300],eax`  
`00436E57` `popad`  
`00436E58` `jmp dword ptr ds:[437300]`  
`00436E5E` `nop`  

用`winhex`在这里写入我们要用的字串：

`004462C8` `6472696D` `mird`  
`004462CC` `642E6C6C` `ll.d`  
`004462D0` `00006C6C` `ll..`  
`004462D4` `00000000` `....`  
`004462D8` `56746547` `GetV`  
`004462DC` `00000054` `T...`  

都搞定了，运行一下，程序提示被修改，但还是乖乖的运行了。

### 结语

终于搞完了，好累。如果还有什么我没有说明白的地方，请参考看学精华 6 上的脱壳文章。

`by aki[BCG][DCM][DFCG]`

`2005-5-25`

`[破解声明]`我是一只小菜鸟，偶得一点心得，愿与大家分享:)

`[版权]`本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
