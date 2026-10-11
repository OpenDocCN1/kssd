
# Armadillo 脱壳知识和方法大全

> 原文：[`www.pediy.com/kssd/pediy10/77137.html`](https://www.pediy.com/kssd/pediy10/77137.html)

近日对 `Armadillo` 壳很感兴趣，缘于它的多种组合的变化，但仔细看来，其保护的解决方法又有相对固定。方法无外乎那么几种脱壳方法（当然排除有 `key` 和 `cc`），本人在本论坛已对标准壳的脱壳方法发贴，但后来有所更新，干脆总结在一块吧，以方便大家。大家可以复制下来，放在手边，脱壳时按步骤来。我还是一小鸟，同大家一样在逐渐成长中，有不对的地方和不成熟的地方，望大家批评指正，共同进步。

## 一、基本知识

该壳有如下保护：

1. `Debug-Blocker` (阻止调试器) -- 解决方法就是忽略所有异常，隐藏好 `OD`，如果加载时，老出错，就多换几个 `OD` 试试。
2. `CopyMem-II` (双进程保护) --- 解决方法是：用手动或者脚本使双变单。
3. `Enable Import Table Elimination` (`IAT` 保护) 解决方法是用工具 `ArmaDetach` 再次载入加壳程序，记下子进程 ID，用另一 `OD` 载入，利用断点 `GetModuleHandleA`，找到 `Magic Jump`，修改 `Magic Jump`，得到正确的 `IAT`。
4. `Enable Strategic Code Splicing` (远地址跳) ，解决方法就是用 `Arminline` 工具。
5. `Enable Nanomites Processing` (简称 `CC`)，就是把一些 `retn` 代码变成 `CC` (`INT` 型)，解决方法：用 `Arminline` 工具或 `Enjoy` 工具。
6. `Enable Memory-Patching Protections` (内存保护)

## 二、脱此类壳常用的断点

1. `WaitForDebugEvent` (用于寻找非标准 `OEP` 和做补丁用)
2. `WriteProcessMemory` (用于寻找非标准 `OEP`)
3. `DebugActiveProcess` (找子程)
4. `OpenMutexA` (双进程转单进程)
5. `GetSystemTime` (补丁 `KEY`)
6. `VirtualProtect` (用于 `5.x`)
7. `CreateFileMappingA` (用于 `5.x`)
8. `GetModuleHandleA`/`LoadLibraryA` (用于找 `Magic Jump`)
9. `CreateThread` (寻找 `OEP`)

## 三、种类及脱壳方法

说明：对于此壳一般要隐藏 `OD`。如果按以下方法下断 `OD` 断不下来，出错，就多换几个 `OD` 试试，在我脱壳中，就有个情况，下断点后，老断不下来，多换了几个 `OD` 就成功了。

### （一）单线程标准方式

具体方法：2 次断点法加修改 `Magic Jump`。

#### 1、找 `Magic Jump`

方法有二：

**方法一**、下断点 `Bp GetModuleHandleA/he GetModuleHandleA/bp GetModuleHandleA+5/he GetModuleHandleA+5`，按 `shift+f9` 运行，当经过一个 `call` 缓冲有点大时，一般是在堆栈窗口出现 `ASCII "kernel32.dll"` 和 `ASCII "VirtualFree"` 后，再运行一次，出现 `"kernel32.dll"`，就是返回时机，取消断点，按 `alt+f9` 执行到返回。

**方法二**、也可以下 `bp LoadLibraryA` 断点，当在堆栈窗口出现 `MSVBVM60.Dll` 函数时，返回时机，在 `kernel32.LoadLibraryA` 下面有一个跳转，一般情况下，这个跳转比较大的话，就改为 `jmp`，而跳转比较小的话，就改 `nop` )。［注：下此断点的目的是找到 `Magic Jump`，修改 `Magic Jump` 目的是避开的 `IAT` 的加密。］

#### 2、找 `OEP`

下断点 `bp GetCurrentThreadId/bp CreateThread`，`shift+f9` 运行，中断后，取消断点，`Alt+F9` 返回，单步执行，看到一个 `call edi` 之类的。`F7` 进入，即到 `oep`。`OD` 不要关！打开 `import` -- 选择进程 -- `OEP` 输入 `va` -- 自动搜索 `IAT` -- 获取输入表 -- 显示无效函数 -- `CUT`！

### （二) 双线程的标准壳

总体步骤：

1. 要双变单；
2. 处理 `IAT`，修改 `Magic Jump`；
3. 寻找 `OEP`；
4. 修复

#### 1、双变单，方法有三。

**方法一：`PATCH` 代码法**

下断 `bp OpenMutexA`，断下后，`ctrl+g` 到 `00401000`，将空数据改为如下代码：

```
00401000      60                 pushad
00401001      9C                 pushfd
00401002    68 A0FD1200  push xxxx  （注：此处的  xxxx 为断下后 mutex name 前的数值。）   
00401007      33C0               xor eax,eax
00401009      50                 push eax
0040100A      50                 push eax
0040100B      E8 E694A677        call KERNEL32.CreateMutexA
00401010      9D                 popfd
00401011      61                 popad
00401012     - E9 8F9FA777       jmp KERNEL32.OpenMutexA
```

点右键选择重建 `eip`，`f9` 运行，断下后，取消断点， `ctrl+g` 到 `00401000`，恢复修改。

**方法二**，下断点：`bp OpenMutexA`，`SHIFT+F9`，断下后， `ALT+F9` 返回，返回后，将返回代码下面的第一个跳转改为相反跳转，再次 `SHIFT+F9`，断下后，`ALT+F9` 返回，再次将返回代码下面的第一个跳转改为相反跳转。然后再一次 `SHIFT+F9`，取消断点，至此，同样，双进程转单进程完毕!

**方法三**、除了用双变单的脚本外，还可以用一个工具 `ARMADETACH`，将带壳的程序拖入，记下子进程的 `ID`。

#### 2、处理 `IAT`，修改 `Magic Jump`。

用 `OD` 附加该子进程，加载后，`ALT+F9` 返回，修改前两个字节为加壳程序载入时的前两个字节。

下断点 `bp GetModuleHandleA`， `Shift+F9` 运行，一般是在堆栈窗口出现 `ASCII "kernel32.dll"` 和 `ASCII "VirtualFree"` 后，再运行一次，出现 `"kernel32.dll"`，就是返回时机，中断后 `alt+f9` 返回，在 `KERNEL32.LoadLibraryA` 下面找到 `Magic Jump`！修改为 `jmp`。再下断点 `bp GetCurrentThreadId/bp CreateThread`（或往下拉找到两个 `salc`，在其上面的 `jmp` 上下断，`Shift+F9`，断下！如果文件有校验，则要撤消 `Magic Jump` 处的修改！打开内存镜像，在 `00401000` 段下断。运行，中断后，删除断点，`alt+f9` 返回），`F8` 单步走，到第一个 `CALL ECX` 之类的东西时，`F7` 进入。到 `oep`。

注：（对于有些 `OD` 有一个字符串溢出漏洞，尽量用一些修正些错误的 `OD`，有些程序需要处理 `Anti`，方法如下：下断点 `he OutputDebugStringA` 断下后，选中 `%s%` 之类的字符，在数据窗口跟随，点右键－> 二进制 －> 使用 `00` 填充，中断 `2` 次！都如上修改，删除此断点！）

#### 3、修复。

### （三）CopyMem-ll +Debug-Blocke 保护方式

1. 先找 `OEP`，两个断点。（1）断点 `bp WaitForDebugEvent`，运行，中断后看堆栈，在一行有 `"pDebugEvent"` 字样的那一行右键点击“数据窗口跟随”，取消断点。（2）`bp WriteProcessMemory`，运行，中断后，在数据窗口（要地址显示）发现 `oep`。

2. `patch` 代码，解码。方法：重新载入，`bp WaitForDebugEvent`，运行，中断，取消断点，`alt+f9` 返回，`CTRL+F` 搜索命令：`or eax, 0FFFFFFF8`，找到后，先往上看，可以看到两个 `CMP`，一个是“`cmp dword ptr ss:[XXXX],0`”在这里下硬件执行断点，`Shift+F9` 运行，中断后取消断点。这时看信息窗口：`SS[XXXXX]=00000000`，如果这个值不为 `0` 的话，需要将其清 `0`，如果为 `0` 就不用了。第二个 `CMP`，`cmp ecx,dword ptr ds:[XXXX]`，下面开始解码。在这里要记下几个地址：（1）`cmp dword ptr ss:[XXXX],0` 前的地址和 `[]` 内的值。（2）第二个 `CMP` 中 `[]` 中的值。接着 `or eax, 0FFFFFFF8` 处往下，可以找到一处为 `and eax, 0FF` 的代码，从这里开始 `Patch`，代码如下：

```
inc dword ptr ds:[] //第一个 CMP 内的值
mov dword ptr ds:[XXXX+4],1 //XXXX 为第二个 CMP[]内的值
jmp XXXX //第一个 CMP 前的地址
```

修改好后，去掉所有断点，向下找到第一个 `CMP` 下面的跳转所跳到的地址，来到这个地址，下硬件执行断点，`Shift+F9` 运行，此时代码解压完毕，可以脱壳。

3. 脱壳。运行 `LordPE`，将子进程 `dump` 出来，这里的子进程就是 `LordPE` 第 `2` 个进程(有 `2` 个同名进程)。`Dump` 后用 `LordPE` 修改入口点为在第一步中查到的 `OEP`。

4. 修复输入表、`IAT` 的寻找

脱壳后不要急着去修复输入表，得先把 `RVA` 数据获取，用 `OD` 载入 `Dump` 出来的程序，右键搜索二进制字符串，输入 `FF25`，找到一个函数，在数据窗口跟随，在数据窗口向上找到全是 `0` 的地方，记下地址，再向下找到全是 `0` 的地方，记下地址。

5. 加载子程序。

    （1）找子程序 `pid` 的方法有二：
    
    **方法一**：用 `OD` 载入原程序（脱壳前的程序），下断点：`bp DebugActiveProcess`，中断后看堆栈，记下 `Processid` 后面的值（这个值不是每次都相同的）。`OD` 不要关。
    
    **方法二**：就是用工具 `ArmaDetach`，将加壳程序拖入。记下子程序的 `pid` 和前两个字节。

    （2）另开一个 `OD`，附加 `Processid` 后面的值进程或用工具 `ArmaDetach` 所记下的进程 `id`，附加后，`ALT+F9` 返回程序，将前两个字节改为原程序载入时的前两个字节。

6. 下面的做法就和标准壳的一样了，在附加的 `OD` 中：

双变单：方法有二。其一：`patch` 法。其二修改跳转法。

**方法一**：下断点 `BP OpenMutexA`（双变单），`F9` 运行，断下后，`ctrl+g` 到 `00401000`，将空数据改为如下代码：

```
00401000      60                 pushad
00401001      9C                 pushfd
00401002    68 A0FD1200  push xxxx  （注：此处的  xxxx 为断下后 name 前的数值。）   
00401007      33C0               xor eax,eax
00401009      50                 push eax
0040100A      50                 push eax
0040100B      E8 E694A677        call KERNEL32.CreateMutexA
00401010      9D                 popfd
00401011      61                 popad
00401012     - E9 8F9FA777       jmp KERNEL32.OpenMutexA
```

点右键选择重建 `eip`，`f9` 运行，断下后，取消断点， `ctrl+g` 到 `00401000`，恢复修改。

**方法二**：下断点 `BP OpenMutexA`，`SHIFT+F9` 运行，断下后，`ALT+F9` 返回，返回后，将返回代码下面的第一个跳转改为相反跳转，再次 `SHIFT+F9`，断下后，`ALT+F9` 返回，再次将返回代码下面的第一个跳转改为相反跳转。然后再一次 `SHIFT+F9`，取消断点，至此，同样，双进程转单进程完毕!

此法相对简单，另外适用于 `00401000` 空数据不能修改的程序。

#### （2）修改 `Magic Jump`

下断 `BP GetModuleHandleA+5`，运行，一般是在堆栈窗口出现 `ASCII "kernel32.dll"` 和 `ASCII "VirtualFree"` 后，再运行一次，就是返回时机，中断后 `alt+f9` 返回，在 `KERNEL32.LoadLibraryA` 下面找到 `Magic Jump`！修改为 `jmp`。清除所有断点，再直接按 `F9` 运行，出现暂停。这时大功告成。

#### （3）用 `Imprec1.6f` 选择进程附加的那个进程，填入 `OEP` 地址，填第 `4` 步所记下的 `RAV`，`SIZE=1000`，不要按自动搜索 `IAT`，直接按获取输入表，再按显示无效地址，剪掉修复抓取文件就 `OK` 了。

### (四) 全保护脱壳方法

1. 双变单可以用以下方法，也可以用脚本，目的是双变单。记下程序加载时的前两个字节。运行脚本后，记下 `OEP` 的前两个字节，`OEP` 地址和子进程 `ID`。

2. 用 `OD` 附加子进程，加载后，`ALT+F9` 返回，修改前两个字节为脚本记载的 `OEP` 的前两个字节。

3. 处理 `IAT` 输入表

方法：

    （1）用工具 `ARMADETACH`，将带壳的程序拖入，记下子进程的 `ID`。
    
    （2）另开一 `OD`，附加该子进程，加载后，`ALT+F9` 返回，修改前两个字节为加壳程序载入时的前两个字节。
    
    （3）修改 `Magic Jump` ，`CTRL+G`，输入 `GetModuleHandleA`，在其下面的第一个跳转下硬件执行断点，`Shift+F9` 运行，一般是在堆栈窗口出现 `ASCII "kernel32.dll"` 和 `ASCII "VirtualFree"` 后，再运行一次，就是返回时机，中断后 `alt+f9` 返回，在 `KERNEL32.LoadLibraryA` 下面找到 `Magic Jump`！修改为 `jmp`。往下拉找到两个 `salc`，在其上面的 `jmp` 上下断，`Shift+F9`，断下！撤消 `Magic Jump` 处的修改！`CTRL+G`，输入 `GetCurrentThreadId/ CreateThread`，下断，运行，中断后，删除断点，`alt+f9` 返回，`F8` 单步走，到第一个 `CALL ECX` 之类的东西时，`F7` 进入。（此时的 `OEP` 是伪 `OEP`，但此时的 `IAT` 是正确的。先开的 `OEP` 是对的，但 `IAT` 是错误的）。
    
    （4）打开先前的 `OD`，`CTRL+B`，搜索 `FF25`，在数据窗口跟随，找到函数的起始位置，选定 `1`-`3` 个，二进制复制，打开后面所开的 `OD`，打开内存镜像，在此界面中点一下，二进制搜索，将刚才复制的粘贴，找到后，在转存窗口，找到函数的起始地址，选定到末尾，将选定的函数复制到先前开的 `OD` 的数据窗口，要对整齐（起点要一样），这时，后开的 `OD` 的任务完成（目的就是将未加密的 `IAT` 找到）。

4. 用 `AMINLINE` 工具修复，选择第一个 `OD` 所用的子进程，在 `Code Splicing` 中开始 `Code` 的地址自动给填好了，但长度需要设置一下，大一些，`20000` 左右就可以了，修复。在 `IAT` 的长度要根据实际情况，一般选 `1000` 即可，依次进行修复，修复后，在 `OD` 中修复的地方都变成红色。

5. 用 `Imprec` 修复，无效指针 `CUT`。

6. 如果有类似自校验的，则用 `AMINLINE` 工具中的 `nanomites` 选项修复。
