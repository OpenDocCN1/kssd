
# Patch 注册 ASProtect V1.X 壳保护程序的方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-530.htm`](https://www.pediy.com/kssd/pediy07/pediy7-530.htm)

## 软件信息
- 下载页面： [`http://www3.skycn.com/soft/10657.html`](http://www3.skycn.com/soft/10657.html)
- 软件大小： 1750 KB
- 软件语言： 英文
- 软件类别： 国外软件 / 共享版 / 图标工具
- 应用平台： Win9x/NT/2000/XP
- 加入时间： 2004-10-20 09:22:15
- 下载次数： 5317
- 推荐等级： *****
- 开发商： [`http://www.beeicons.com/`](http://www.beeicons.com/)
- 软件介绍： Bee Icons 是一个图标工具，可以替换系统图标和已注册文件类型的图标!

## 作者声明
只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

## 调试环境
WinXP、`OllyDbg`、`LordPE`、`WinHex`

---

## 脱壳过程
利用 ASProtect 的 Pre-Dip 来 Patch 注册其保护程序的方法在国外某些组织早已使用了。这个方法的意义在于能够直接 Patch 修改 ASProtect 加壳程序的内部代码，注册是指那些可以通过处理 Pre-Dip 来完成注册的加壳程序。本教程所介绍的方法只能 Patch ASProtect V1.X 壳保护程序，包括 V1.23RC4、V1.3X 等版本；但是不适用于 ASProtect 最新的 V2.X 壳保护程序，因为 ASProtect V2.X 壳特别针对这个方法加了校验。

关于 Pre-Dip 可以查阅《看雪论坛精华 6》中 lipton 以及 ASProtect 目录下的相关教程。

下面以 ASProtect V1.23RC4 加壳的 Bee Icons V4.01 为例来进行演示。方法比较繁琐，分多次操作。

需要耐心和 ASProtect 脱壳基础。多多练习就可以成功，国内已有不少兄弟掌握此方法了。

设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其它异常选项。下面的各步除了特殊说明的，皆为这个异常选项设置。老规矩：用 `IsDebug` V1.4 插件去掉 `Ollydbg` 的调试器标志。

---

### 一、获得 ASProtect 注册名 Pre-Dip 处理的地址

```
00401000    68 01306100     push BeeIcons.00613001
```
// 进入 Ollydbg 后暂停在这
```
00401005    E8 01000000     call BeeIcons.0040100B
0040100A    C3              retn
```
Shift+F9 运行 N 次，直至堆栈中第 2 次看见硬盘指纹:
```
0013FF3C    00FE4849  返回到 00FE4849
0013FF40    00FD3861
0013FF44    0013FFE0  指针到下一个 SEH 记录
0013FF48    00FE4C49  SE 句柄
0013FF4C    0013FF90
0013FF50    00FD0000
0013FF54    00FA0000
0013FF58    00FE4138
0013FF5C    010028C8  ASCII "wWDDDAAgLTg="
```
Alt+M 打开内存查看窗口，在 `00401000` 段“设置内存访问断点”，Shift+F9 通过异常，断下
```
0052FAB4    55              push ebp
```
// 注意这个地址：`0052FAB4` ★
```
0052FAB5    8BEC            mov ebp,esp
0052FAB7    8B45 08         mov eax,dword ptr ss:[ebp+8]
```
// `[ebp+8]=[0013FF40]=00FD3861` 注册名保存地址 ★
```
0052FABA    85C0            test eax,eax
0052FABC    74 0C           je short BeeIcons.0052FACA
0052FABE    8038 00         cmp byte ptr ds:[eax],0
0052FAC1    74 07           je short BeeIcons.0052FACA
0052FAC3    C605 681A5700 0>mov byte ptr ds:[571A68],1
0052FACA    8B15 5C1A5700   mov edx,dword ptr ds:[571A5C]
0052FAD0    8915 581A5700   mov dword ptr ds:[571A58],edx
0052FAD6    A3 5C1A5700     mov dword ptr ds:[571A5C],eax
0052FADB    5D              pop ebp
0052FADC    C2 0400         ret 4
```
看堆栈：`0013FF3C` 返回到 `00FE4849`，Ctrl+G:`00FE4849` 向上看看
```
00FE483F    A1 3866FE00     mov eax,dword ptr ds:[FE6638]
00FE4844    8B40 04         mov eax,dword ptr ds:[eax+4]
00FE4847    FFD0            call eax
```
// Call `0052FAB4`
```
00FE4849    8B15 1865FE00   mov edx,dword ptr ds:[FE6518]
```
在转存中 Ctrl+G:`[0FE6638]`+4
```
00FE7AB8  B4 FA 52 00     ★
```
记住：`0052FAB4` 和 `00FE7AB8`

---

### 二、第一次回溯：00FE7AB8

重新载入程序。忽略除了“内存访问异常”之外的所有其它异常选项，Shift+F9 运行 1 次
```
00FE41A4    3100            xor dword ptr ds:[eax],eax
```
// 第 1 次内存异常

在转存中 Ctrl+G：`00FE7AB8`，或者下命令：`DB 00FE7AB8`

选中 `00FE7AB8` 处 4 个字节，下“内存写入”断点。忽略所有异常，Shift+F9
```
00FD27A8    F3:AB           rep stos dword ptr es:[edi]
```
// 第一次中断
```
00FD262B    F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
```
// 第二次中断
`ds:[esi]=[00FF1613]=0012FAB4` ★
`es:[edi]=[00FE7AB8]=00000000`
`0012FAB4`= `0052FAB4`－基址 `0040000`

如果在 `00FE7AB8` 处设置“内存访问”断点，就会跟踪到这里：
```
00FE444D    010C82          add dword ptr ds:[edx+eax*4],ecx; BeeIcons.00400000
```
// `[00FE7AB8]=0012FAB4+00400000`

记住：`00FF1613`

---

### 三、得到壳重定位后的基址 + 第二次回溯：00FF1613

重新载入程序。忽略所有异常，BP `VirtualAlloc`  然后把断点挪到 `VirtualAlloc` 的段尾
Shift+F9 运行，观察壳所申请到的内存地址

#### 1、得到壳重定位后的基址

```
7C809A81    8BFF            mov edi,edi
```
// VirtualAlloc
```
7C809A83    55              push ebp
7C809A84    8BEC            mov ebp,esp
7C809A86    FF75 14         push dword ptr ss:[ebp+14]
7C809A89    FF75 10         push dword ptr ss:[ebp+10]
7C809A8C    FF75 0C         push dword ptr ss:[ebp+C]
7C809A8F    FF75 08         push dword ptr ss:[ebp+8]
7C809A92    6A FF           push -1
7C809A94    E8 09000000     call kernel32.VirtualAllocEx
7C809A99    5D              pop ebp
7C809A9A    C2 1000         ret 10
```
// 这里下断

当我们在 `VirtualAlloc` 中断第 2 次，会发现申请的 `EAX` 返回值是 `00FD0000`
```
0013FF90   0061352D  /CALL 到 VirtualAlloc 来自 BeeIcons.00613527
0013FF94   00000000  |Address = NULL
0013FF98   0001E000  |Size = 1E000 (122880.)
0013FF9C   00001000  |AllocationType = MEM_COMMIT
0013FFA0   00000004  \Protect = PAGE_READWRITE
```
`00FD0000` 就是这个程序的壳重定位后的基址，F7，需要跟踪一下
```
00613527    FF95 DF030000   call dword ptr ss:[ebp+3DF]
0061352D    8985 BF010000   mov dword ptr ss:[ebp+1BF],eax
```
// `[00613607]=00FD0000` ★
```
00613533    8B55 5B         mov edx,dword ptr ss:[ebp+5B]
00613536    8B85 BF010000   mov eax,dword ptr ss:[ebp+1BF]
0061353C    8902            mov dword ptr ds:[edx],eax
```
// `[00613048]=00FD0000`

记住：`[00613607]`，保存壳重定位后的基址，或者`[00613048]`

#### 2、第二次回溯：00FF1613

OK，我们继续 Shift+F9 运行，观察壳所申请到的内存地址

这个需要不少次才看到 `EAX=00FF0000`，某些时候需要中断 2 次 `EAX=00FF0000`

取消 `7C809A9A` 处断点。在转存中 Ctrl+G：`00FF1613`

选中 `00FF1613` 处 4 个字节，下“内存写入”断点。Shift+F9
```
00FD2645    F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
```
// 中断在这里
`ds:[esi]=[00635587]=E0515542`
`es:[edi]=[00FF1614]=00000000`
`ds:[esi]=[00635583]=DB9F2C32`
`es:[edi]=[00FF1610]=00000000`
`00635586`==> `00FF1613` `DB 42 55 51` ★

你可以用 `LordPE` 的 FLC 功能查看 `BeeIcons.exe` 的 `00635586`（偏移 `000C5B86`）处数据：`DB 42 55 51`

Good！ ASProtect 壳下面会对这些数据进行简单的运算，变换成 `0012FAB4`

---

### 四、得到运算数据 + 求逆

选择 `00FF1613` 处的 4 个字节，设置“内存访问”断点。

Shift+F9，中断在 `00FD94BA` 处，开始运算
```
00FD946A    41              inc ecx
00FD946B    80E1 FF         and cl,0FF
00FD946E    33DB            xor ebx,ebx
00FD9470    8AD9            mov bl,cl
00FD9472    021418          add dl,byte ptr ds:[eax+ebx]
00FD9475    80E2 FF         and dl,0FF
00FD9478    33DB            xor ebx,ebx
00FD947A    8AD9            mov bl,cl
00FD947C    8A1C18          mov bl,byte ptr ds:[eax+ebx]
00FD947F    885D F7         mov byte ptr ss:[ebp-9],bl
00FD9482    8BF2            mov esi,edx
00FD9484    81E6 FF000000   and esi,0FF
00FD948A    33DB            xor ebx,ebx
00FD948C    8AD9            mov bl,cl
00FD948E    8D3C18          lea edi,dword ptr ds:[eax+ebx]
00FD9491    8A1C30          mov bl,byte ptr ds:[eax+esi]
00FD9494    881F            mov byte ptr ds:[edi],bl
00FD9496    8A5D F7         mov bl,byte ptr ss:[ebp-9]
00FD9499    881C30          mov byte ptr ds:[eax+esi],bl
00FD949C    33DB            xor ebx,ebx
00FD949E    8AD9            mov bl,cl
00FD94A0    8A1C18          mov bl,byte ptr ds:[eax+ebx]
00FD94A3    8BF2            mov esi,edx
00FD94A5    81E6 FF000000   and esi,0FF
00FD94AB    021C30          add bl,byte ptr ds:[eax+esi]
00FD94AE    80E3 FF         and bl,0FF
00FD94B1    885D F7         mov byte ptr ss:[ebp-9],bl
00FD94B4    8B5D FC         mov ebx,dword ptr ss:[ebp-4]
00FD94B7    8B75 F0         mov esi,dword ptr ss:[ebp-10]
00FD94BA    8A1C33          mov bl,byte ptr ds:[ebx+esi]
```
// 中断在这里！依次取 `DB 42 55 51`
```
00FD94BD    0FB675 F7       movzx esi,byte ptr ss:[ebp-9]
00FD94C1    321C30          xor bl,byte ptr ds:[eax+esi]
```
// 异或 注意：记住运算的数据！

// ①、`BL=DB XOR 6F=B4`
// ②、`BL=42 XOR B8=FA`
// ③、`BL=55 XOR 47=12`
// ④、`BL=51 XOR 51=00`

运算结束得到： ★
```
00FF1613  B4 FA 12 00
```

`BeeIcons.exe` 的 `000C5B86` 偏移地址处是 `DB425551`，这个值正是上面进行运算的数据。

我们要把这里的 `0052FAB4`（`0012FAB4`+`0040000`）改为运算后跳向可以写 Patch 代码的地址！

一般在 PE 文件偏移 80－100 间有段空白地址可以给我们利用

所以要修改这个运算后的 Patch 地址为 `00400080`

求逆 `00000080`（`00400080`－`00400000`）如下：

①、`BL=80 XOR 6F=EF`
②、`BL=00 XOR B8=B8`
