
# DLL 的 tELock 变形壳

> 原文：[`www.pediy.com/kssd/pediy06/pediy6720.htm`](https://www.pediy.com/kssd/pediy06/pediy6720.htm)

【目标程序】：加 `tElock` 变形壳的 `eiserver.dll`。

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`Win2000`、`Ollydbg1.10`、`LordPE`、`UltraEdit`

【实例下载】：点击此处本地下载 或 [`bbs.pediy.com/showthread.php?s=&threadid=6298`](http://bbs.pediy.com/showthread.php?s=&threadid=6298)

—————————————————————————————————

【脱壳过程】：

在论坛上看有人发了个加壳的 `DLL`，最近心情不太好，就拿来玩玩！

本文参考 `Fly` 大侠《用 `Ollydbg` 手脱 `tElock` V0.98 加壳的 `DLL`》写成！

希望 `Fly` 兄不要见怪！^\_^ ^\_^ ^\_^

—————————————————————————————————

## 一、双重 Magic Jump：避开重定位和输入表加密

设置 `Ollydbg` 忽略除了“`INT3` 中断”之外的所有其它异常选项。

老规矩：用 `IsDebug` 1.4 插件去掉 `Ollydbg` 的调试器标志。

```
007B1222  >^\E9 D9DDFFFF           jmp eiserver.007AF000      //进入 OD 后停在这
007B1227    0000                add byte ptr ds:[eax],al
```

`Shift+F9` 运行，程序中断在 `INT3` 异常处

```
007AFD5A    90                  nop                           //第 3 次中断在这里
007AFD5B    66:81C6 B7B9        add si,0B9B7
```

来到下面代码处下断

```
007B01B6    74 0F               je short eiserver.007B01C7    //在这一行下断
007B01B8    FF95 181F4000       call dword ptr ss:[ebp+401F18]
007B01BE    6A 00               push 0
```

`Shift+F9` 断在这个跳转指令后 更改标志位 使之跳转 否则 `GAME OVER`

清除这个断点，设置 `Ollydbg` 忽略所有异常选项。`Ctrl+S` 在“整个段块”搜索命令序列：

```
mov ebx,edx
shr ebx,10
mov eax,dword ptr ds:[esi]
```

```
007B06BE    8BB5 00334000       mov esi,dword ptr ss:[ebp+403300]
//[ebp+403300]=[007B1300]=0001B000 ★ 这个 0001B000 就是重定位表的 RVA！
007B06C4    85F6                test esi,esi
007B06C6    0F84 8B000000       je eiserver.007B0757
007B06CC    8B95 0C334000       mov edx,dword ptr ss:[ebp+40330C]
007B06D2    03F2                add esi,edx
007B06D4    2B95 10334000       sub edx,dword ptr ss:[ebp+403310]
//这里其实就是检测与映像基址是否相符，不符则重定位处理！★
007B06DA    74 7B               je short eiserver.007B0757
//Magic Jump，可以第 2 次 Load 后在这里改标志位 Z=1，使其跳转 ★
007B06DC    8BDA                mov ebx,edx
007B06DE    C1EB 10             shr ebx,10
007B06E1    8B06                mov eax,dword ptr ds:[esi]
007B06E3    85C0                test eax,eax
007B06E5    74 70               je short eiserver.007B0757
//重定位处理完毕后这里就跳转     在 007B0757 处下断
007B06E7    8B4E 04             mov ecx,dword ptr ds:[esi+4]
007B06EA    83E9 08             sub ecx,8
007B06ED    D1E9                shr ecx,1
007B06EF    8BBD 0C334000       mov edi,dword ptr ss:[ebp+40330C]
007B06F5    03F8                add edi,eax
007B06F7    83C6 08             add esi,8
007B06FA    0FB706              movzx eax,word ptr ds:[esi]
007B06FD    C1C8 0C             ror eax,0C
007B0700    FEC8                dec al
007B0702    78 4C               js short eiserver.007B0750
007B0704    74 0E               je short eiserver.007B0714
007B0706    FEC8                dec al
007B0708    74 13               je short eiserver.007B071D
007B070A    FEC8                dec al
007B070C    74 3C               je short eiserver.007B074A
007B070E    FEC8                dec al
007B0710    74 14               je short eiserver.007B0726
007B0712    EB 3C               jmp short eiserver.007B0750
007B0714    C1E8 14             shr eax,14
007B0717    66:011C07           add word ptr ds:[edi+eax],bx
007B071B    EB 33               jmp short eiserver.007B0750
007B071D    C1E8 14             shr eax,14
007B0720    66:011407           add word ptr ds:[edi+eax],dx
007B0724    EB 2A               jmp short eiserver.007B0750
007B0726    52                  push edx
007B0727    C1E8 14             shr eax,14
007B072A    8BD8                mov ebx,eax
007B072C    C1E0 10             shl eax,10
007B072F    66:8B16             mov dx,word ptr ds:[esi]
007B0732    66:81E2 FF0F        and dx,0FFF
007B0737    66:8BC2             mov ax,dx
007B073A    5A                  pop edx
007B073B    8D8410 00800000     lea eax,dword ptr ds:[eax+edx+8000]
007B0742    89041F              mov dword ptr ds:[edi+ebx],eax
007B0745    46                  inc esi
007B0746    46                  inc esi
007B0747    49                  dec ecx
007B0748    EB 06               jmp short eiserver.007B0750
007B074A    C1E8 14             shr eax,14
007B074D    011407              add dword ptr ds:[edi+eax],edx
007B0750    46                  inc esi
007B0751    46                  inc esi
007B0752    49                  dec ecx
007B0753   ^ 7F A5              jg short eiserver.007B06FA
007B0755   ^ EB 8A              jmp short eiserver.007B06E1
007B0757    8B95 0C334000       mov edx,dword ptr ss:[ebp+40330C]
//当我们中断在 007B0757 处时，重定位处理完毕。此时 ESI=007ACC94，★ 就是重定位表的结束地址啦。
//得到重定位表信息：RVA=0001B000，大小＝007ACC94-007AB0000=1C94
007B075D    8BB5 FC324000       mov esi,dword ptr ss:[ebp+4032FC]
//[ebp+4032FC]=[007B12FC]=00019000 ★ 这就是输入表的 RVA 啦。
007B0763    0BF6                or esi,esi
007B0765    0F84 30040000       je eiserver.007B0B9B
//第 2 个 Magic Jump，改标志位 Z=1，使其跳转，则不加密 IAT
007B076B    03F2                add esi,edx
007B076D    83A5 FC334000 00    and dword ptr ss:[ebp+4033FC],0
007B0774    33C0                xor eax,eax
007B0776    8746 0C             xchg dword ptr ds:[esi+C],eax
007B0779    0BC0                or eax,eax
007B077B    0F84 1A040000       je eiserver.007B0B9B
007B0781    35 78563412         xor eax,12345678
007B0786    E8 CD000000         call eiserver.007B0858
007B078B    03C2                add eax,edx
007B078D    8BD8                mov ebx,eax
007B078F    50                  push eax
007B0790    FF95 7A324000       call dword ptr ss:[ebp+40327A]
007B0796    85C0                test eax,eax
007B0798    0F85 D7000000       jnz eiserver.007B0875
007B079E    53                  push ebx
007B079F    FF95 7F154000       call dword ptr ss:[ebp+40157F]
007B07A5    85C0                test eax,eax
007B07A7    0F85 C8000000       jnz eiserver.007B0875
007B07AD    8B95 0C334000       mov edx,dword ptr ss:[ebp+40330C]
```

咱们已经得到重定位表信息了，所以可以再次载入这个 `DLL`，重复上面的步骤，避开重定位表加密！

—————————————————————————————————

## 二、第 2 区段内存断点法：快速直达 OEP

当我们修改了以上两个 `Magic Jump`，来到如下代码处时

```
007B0B9B    8BBD 04334000       mov edi,dword ptr ss:[ebp+403304]
007B0BA1    85FF                test edi,edi
007B0BA3    EB 03               jmp short eiserver.007B0BA8
```

现在我们 `Alt+M` 打开 内存查看 窗口：

```
00790000    00001000   eiserver 00790000 (itself)                 PE header                Imag 01001002        R         RWE
00791000    00016000   eiserver 00790000                          code                     Imag 01001002        R         RWE
007A7000    00001000   eiserver 00790000                          data                     Imag 01001002        R         RWE
007A8000    00001000   eiserver 00790000                                          Imag 01001002        R         RWE
```

在 `00791000` 第 2 区段上 `F2` 设置断点！`Shift+F9` 运行，直接中断在 `OEP` 处！

```
007A69F0    55                  push ebp
007A69F1    8BEC                mov ebp,esp
007A69F3    83C4 C4             add esp,-3C
007A69F6    B8 20697A00         mov eax,eiserver.007A6920
007A69FB    E8 04F3FeFF         call eiserver.00795D04
```

用 `LordPE` 选中 `Ollydbg` 的 `loaddll.exe` 的进程，在下面的列表里选择 `eiserver.dll`，然后完整脱壳，得到 `dumped.dll`。

—————————————————————————————————

## 三、PE 修正

```
//修正参数如下

入口点 OEP = 169F0

输入表 RVA = 19000

重定位 RVA = 1B000  大小 = 1C94

//用 LordPE 打开修正后的 dumped.dll 会发现输入表错误！

//不知道是否有现成的修复工具，只好分析一下，手工修复了！
```
