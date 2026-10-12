
### 0151C582 处 IAT 基地址写入说明
`0151C582` `8B0D C09A5101` `mov ecx,dword ptr ds:[1519AC0]`，这里放入 `IAT` 基地址。

### 修复方法
`IAT` 调用是连续的，从起始位置循环处理即可。

以第一个为例：
`00412940` `- E9 DBB73300` `jmp manager.0074E120 *`

1. 第一步：读取 `dword ptr ds:[0074E120+1]` 这个值，获取函数序号提取码。
2. 第二步：判断属于第一组还是第二组。
3. 第三步：调用壳的子函数 `call` 来获得函数序号。
4. 第四步：获取 `IAT` 组的开始地址。
5. 第五步：通过上面的两个数据获得函数真实地址，然后修改 `*` 处。
6. 第六步：循环下一个。

有兴趣的高人可以写一个脚本玩玩儿哦~

搞定了这个程序以后，继续研究一下 `Delphi` 的程序。

同样的办法到达 `OEP`，随便找一下 `IAT` 调用。

还是被壳修改了，改的形式是一样的。

### 示例 `IAT` 调用（`Farsight`）

```
00407CDC  - E9 9FA42D00     jmp Farsight.006E2180
00407CE1    90              nop
00407CE2    8BC0            mov eax,eax
00407CE4  - E9 13B02D00     jmp Farsight.006E2CFC
00407CE9    90              nop
00407CEA    8BC0            mov eax,eax
```

跟踪一下就能看出来 `IAT` 的处理方式是一模一样的。尤其是下面一段代码，几乎是很特征性的。

### 特征性代码段（`kernel32` 相关调用）

```
01526102    FF15 14C75001   call dword ptr ds:[150C714]               ; kernel32.GetTickCount
0151D494    8BC8            mov ecx,eax
015286D0    2B0D 68935101   sub ecx,dword ptr ds:[1519368]            ; manager.00707416
01524042    81F9 88130000   cmp ecx,1388
0151FA0B   /0F86 105A0000   jbe 01525421
0151E6F2    8B15 6C935101   mov edx,dword ptr ds:[151936C]
01528235    52              push edx
015286B8    A3 68935101     mov dword ptr ds:[1519368],eax
015209E5    FF15 84C65001   call dword ptr ds:[150C684]               ; kernel32.ResumeThread
015215BA    833D 289B5101 0>cmp dword ptr ds:[1519B28],3
01524837  ^\0F8C 8FF3FFFF   jl 01523BCC
01523BCC    803D D8945101 0>cmp byte ptr ds:[15194D8],0
0151FEAF    B8 01000000     mov eax,1
0151D3C1   /0F84 FA900000   je 015264C1
015264C1    C705 289B5101 0>mov dword ptr ds:[1519B28],0
```

### `FF25` 型与 `FF15` 型 `IAT` 修复
这个程序的 `IAT` 调用是 `FF25` 型的，修复的时候要先修复 `FF25`，然后找个空白地方写入 `IAT` 地址就可以了~~~

对于 `FF15` 型的程序来说，也比较好辨别，因为程序在修改调用 `call` 的时候改成了 `E9 XXXXXXXX` 形式，同 `FF15 XXXXXXXX` 的调用形式差了一个字节，所以加壳以后在 `IAT` 调用的 `call` 下面是有一个 `nop` 指令的。

### 总结
上面就是对 `ZProtect` 加壳后程序的 `IAT` 修复的方法。脚本和修复工具就不附加了。偶的编程水平实在不行。有兴趣的大侠可以写个通用的脚本或工具。到时一定捧场~

### 突破 `ZP` 注册对话框
最后说一下突破 `ZP` 的注册对话框，`y3` 大侠的 `dll` 可以过不带试用框的 `ZP` 程序，貌似是 hook 了 `DialogBoxIndirectParam` 这个函数，希望 `y3` 大侠能公开方法~~ （带试用框就不用跳了吧，直接试用，然后脱壳就行了……）

小菜心得，欢迎批评！

`HyperChem`

`02/23/2009`

### 附件
- 目标 `VB` 程序.rar [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=82665`](http://bbs.pediy.com/showthread.php?t=82665)]
- ![附件](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)
