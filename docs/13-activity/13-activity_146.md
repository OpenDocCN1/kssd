
# shoooo 的\[第二阶段◇第三题\]脱壳机源码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-266.htm`](https://www.pediy.com/kssd/pediy09/pediy09-266.htm)

> 看到别人发，我也发  

> 代码是从我写过的 N 个程序东抄一段西抄一段拼起来了，所以看上去很不爽

## 壳概述

壳一共有两种压缩算法和对应的 `loader`

`level 0` 是第一种 `(以下称 level0)`

`level1-level9` 是第二种 `(以下称 level1)`

下面的分析是以 `petgui` 主程序为例, 用的是 `level1`. `level0` 的就不贴了, 原理一样的

## 壳段的结构

`Offset`   `Length`

`+0x000`    `DWORD`    壳输入表的偏移

`+0x004`    `DWORD`    PE 头的校验值

`+0x008`    `DWORD`    解压时存放压缩数据的 `GlobalAlloc` 地址

`+0x00C`    `0x25`     一段代码, 壳出错时会跑过来退出

`+0x031`    `0x15`     壳出错时弹出的信息字符串

`+0x046`    `0x15A`    壳的第一段 `Loader` `(A 部分)` `EP`

`+0x160`    `0x14*5`   解压信息 `(B 部分)`

`+0x204`    `0x1CA`    壳本身的整个输入表

`+0x3CE`    到最后   程序的输入表的 `dll` 名字集合 `(C 部分)`

需要定位 `A`, `B`, `C`

### A 部分

A 部分中我们需要知道如下

```
004E3046      B8 00304E00        mov eax,petgui.004E3000
004E304B      68 E3644100        push petgui.004164E3
```

这里 `004164E3` 是第二 `Loader` 部分,等解压完后会跳过去

```
004E30AC      81C6 02010000      add esi,102
```

这里 `102` 可以定位到 `C` 部分

```
004E30B7      8D90 A0010000      lea edx,dword ptr ds:[eax+1A0]
```

这里 `1A0` 可以定位到 `B` 部分

### B 部分

B 部分是以 `0x14` 为一个分组的解压信息, 意义如下

```
000E2420  加密数据存放的地址
000007E9  解密后的大小
000E2420  解密后存放的地址
000007EE  /2 的值是最后清 0 的大小,%2 的值决定是否要 bswap e8e90f84
000000BD  *4 以后是加密数据的大小
```

### C 部分

C 部分是输入表 `dll` 的名字

一个接一个排列, 每个以 `0` 结尾

## 第二个 Loader

`4164E3` 这里是第二个 `loader`, 是第一个 `loader` 解码出来的

`Offset`   `Length`

`+0x000`    `5`        `E84F000000`

`+0x005`    `0x0E`     一段加密代码, 和最后跳回 `oep` 有关

`+0x013`    字符串    如果发现校验不同过,会显示这个

后面是代码了

有几个固定偏移的地方有些重要数据

```
00416587      6A 03              push 3
```

这个 `3` 是抽 `api` 的初始计数值

```
0041658C      68 3E030000        push 33E
```

这个 `33E` 过去是一个 `call` 解码后是跳回 `ep-9` 的地方

```
004165C8      315C11 01          xor dword ptr ds:[ecx+edx+1],ebx
```

通过最后一个 `call` 可以直接算出 `ebx` 的值

```
00416605      8DB5 CC590100      lea esi,dword ptr ss:[ebp+159CC]
```

可以得到 `IID` 的位置, 修输入表其实就是恢复 `IID`

```
00416704     /74 12              je short petgui.00416718
00416706     |3B1A               cmp ebx,dword ptr ds:[edx]
00416708     |8318 00             sbb dword ptr ds:[eax],0
0041670B     |390A               cmp dword ptr ds:[edx],ecx
0041670D     |8318 00             sbb dword ptr ds:[eax],0
00416710     |83C2 04             add edx,4
00416713     |C108 03             ror dword ptr ds:[eax],3
00416716    ^|EB E9              jmp short petgui.00416701
00416718     \\C706 00000000      mov dword ptr ds:[esi],0
```

能过输入表的各数算 `oep`

可以理解为, 抽则-2, 不抽则-1, 再 `ror 3`

```
00416834      E8 04C80C00        call petgui.004E303D
```

跳回 `ep-9`

```
petgui.004E303D      5F                  pop edi
```

`petgui.00400852`

```
004E303E      F3:AA              rep stos byte ptr es:[edi]
004E3040      61                  popad
004E3041      66:9D               popfw
004E3043      83C4 08             add esp,8
004E3046      E9 8CA0F2FF         jmp petgui.0040D0D7
```

跳 `oep`

`4dd466d0e1e0cc123afbe504da2462ca`
