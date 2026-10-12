
# MP3 音频录音机 v5.10 脱壳

> 原文：[`www.pediy.com/kssd/pediy12/sdp2.html`](https://www.pediy.com/kssd/pediy12/sdp2.html)

不清楚这个 `SDP` 究竟是什么版本，总之不是以前碰过的那个 `J`。用的是修改版的 `OllyDbg` `110`、`OllyScript` `0.92`。

## 1. 测试

事先已经知道是用 `SDP` 做的，想看看是否为双进程版本。直接运行，用 `LoadPE` 只看见一个进程。用 `OD` 加载，因为 `Single Step Event` 停在这里：

![image](img/6757573a7bec84d922304a907704a5df.jpg)

在 `IDA` 中的 `EP` 是 `5E2000`，而且不像 `ExeCryptor`，没有 `TLS Callback Function`。修改 `OD` 设置：

![image](img/1cbc57aeb0b69f15bbcce02e66166734.jpg)

停在 `ntdll` 中。再对 `5E2000` 下断，可以停下。清除该断点，不忽略所有异常，用 `IsDebuggerPresent` 隐藏 `OD`，`F9` 运行。

![image](img/a07cd703c02ab2af51ea7c3fe50f488e.jpg)

直接运行，用 `WinHex` 搜索，用什么版本保护的？文件头中的标记：

![image](img/5e729710abd6c7bb46c2c2de8482a187.jpg)

提示信息：

![image](img/e2bf44053e9fecf130740b219d559655.jpg)

不会是破解版本吧，或许注册版本也有这个？

写个简单的 `OllyScript`，记录异常次数。之所以用 `script` 而不是直接用手按，是因为 `SDP` 对运行时间的检测很严 `J`。

![image](img/a4c039a30754d6c8027d82b1ae55850a.jpg)

可以看到，10 次 `shift-F9` 后报 `debugger detected`。

## 2. Debugger 检测

10 次 `shift-F9` 后，这里异常：

![image](img/bbf673c426f333a07e5f24ac3101dd12.jpg)

用 `script` 走到 `5E2EFA`，这个异常的 `handler` 没有改变 `context` 中的 `eip`，从 `SEH handler` 中返回异常地址继续执行。`dump` 解出的代码，去掉 `Junk code` `J`。

跟第 10 次异常后的代码，唯一的一次判断在这里（不跳，往下执行即显示 `debugger detected`）：

![image](img/04f81aebc613e3ac54635f30ed23fdd3.jpg)

修改 `script`，在这里 `jmp` 到 `5E2F7F`。程序继续运行了。下面又是一次异常：

![image](img/9dcca44169bdbf325792a2d824797640.jpg)

这里如果 `shift-F9`，原来的进程结束了。新启动一个进程继续运行。

## 3. 进程逃逸

从上面这个异常（`5E313D`）的 `handler` 开始跟。有个问题：在 `debugger` 下运行有时会失败，不能启动子进程。一旦失败一次，就再也不能正常运行起来了。不知道是不是程序做了什么手脚。在我的 `W2K SP4` 和 `WinXP Pro` 下都如此。

![image](img/a819d36295e738a4ac5e43bc00a9a87a.jpg)

这是创建新进程的代码：

![image](img/65a429499449ecf1875ff8a119e6bf9f.jpg)

创建进程时的参数：

![image](img/4b878416dcd7c0ff975bf095950b1b34.jpg)

父进程和子进程使用同一个 `EXE` 文件，参数也没有任何特殊之处。

执行 `CreateProcessA` 后的返回地址。

![image](img/b962268aee2f3eb6a12a71bcd9d93b4f.jpg)

创建新进程之后，父进程立即就退出了，子进程独立运行。显然，这里并不像 `Armadillo` 的 `Nanaomites` 那样让人头疼 `J`。创建新进程的唯一目的就是摆脱 `debugger`。

为什么新的进程不能运行了？注意在调用 `CreateProcess` 时传递的 `StartInfo` 结构没有正确填入数据。

![image](img/46ea906b723dc9921040ba5a2c9e9911.jpg)

全部为 0。是否程序发现正在被调试，设置了某些标记？试试手动填入 `cb` 等数据，仍然不行。从任务管理器的显示看，似乎这个新创建的进程仍然在把自己当父进程运行，大概是某些用于同步的内核对象出了问题。

既然子进程完全独立运行，可以甩开上面的问题，直接使程序以子进程方式运行，继续调试。

## 4. 分离子进程

用 `OllyScript` 在上面的 `5E3194` 处 `jmp`，强制以子进程方式运行。记录异常和中断。

![image](img/68c4e6db29e53e87d603b304171594da.jpg)

也是 10 次，后面的 2 个记录是异常。程序直接退出了，弄错了？从 `log` 看，判断应该是正确的。跟一下。

![image](img/0021b591471d5434b15e31f25a23afd9.jpg)

从 `handler` `5E4590` 跟起。到这里，出现 `log` 中的第 1 个内存访问异常。

![image](img/25a93652fda9e5744c99fe3a8614954a.jpg)

![image](img/7ac60388be7dc0b0e138f47dd898c4f8.jpg)

这个异常被 `OllyDbg` 处理了。用插件避开这个 `anti-debug` 技巧。



### THEORY

When an exception happens, `Win2K/XP` gives control to `NTDLL!KiUserExceptionFilter`, which in turn calls `KERNEL32!UnhandledExceptionFilter`. This function checks if faulty software is being debugged. If not, `UnhandledExceptionFilter` calls software's exception handler.

### SOLUTION

Patch `KERNEL32!UnhandledExceptionFilter` so that software's exception handler is always called. Used signatures work for both `Win2K SP4/WinXP SP1`, and hopefully for all other versions.

![文本框: THEORY:When  exception happens, Win2K/XP gives control to NTDLL!KiUserExceptionFilter,which  in turn calls KERNEL32!UnhandledExceptionFilter. This function checks iffaulty  software  is  being  debugged.  If  not, UnhandledExceptionFilter callssoftwares' exception handler.SOLUTION:Patch  KERNEL32!UnhandledExceptionFilter  so that softwares' exception handleris  always  called.  Used  signatures  work  for both Win2K SP4/WinXP SP1, andhopefully for all other versions.](img/dceb75afaab680eda470df80723de64c.jpg)

在 `OllyDbg` 中停在系统断点时就运行 `UnhandledExeptionFilter` 插件。仍然用前面的 `script` 记录。这次程序正常运行了。

![图片](img/75139ed0959b869bf4b54c847e0497dd.jpg)

![图片](img/c0b82d96c032d55ba8c4e843ffa0b797.jpg)

这次记录为 `0x35`。注意异常地址并不在壳代码空间（因为程序在开始运行时总有一个数组越界错误，脱壳后倒好了 J），后面的可能是控件抛出的。

这是壳代码执行的最后一个异常，计数为 `0x2E`。

![图片](img/7162529c2f3f1daacd2a2af88f0a6fc1.jpg)

另外，注意这里：

![图片](img/d8cf0afe0a9d7441bcf9b58a0e3918f7.jpg)

`5E78E2` 这个地址在 `log` 中反复出现，后面往往有 `Module` 载入。这里是处理 `IAT` 的地方？

### 5. 寻找 OEP

用 `Script` 停在上面记录的壳代码最后一个异常处（`5E707A`）。`handler` 结束后回到异常处继续。

`F7` 单步几次就到 `OEP` 了。

![图片](img/8fd108eb273fc78a3091bb37a48f712e.jpg)

`ret` 到这里。

![图片](img/0f360aebaf4f61957f172674b0944f07.jpg)

没有 `stolen code`？不会吧。可能找错了。先 `dump` 出来再说。用 `OllyDump`。

![图片](img/532eb44e97514b195bd16e564da64922.jpg)

`anti-dump` J。直接在 `OllyDbg` 中修改每个 `section` 的内存块属性。

![图片](img/eaade81246430251d7e7552121c6ff61.jpg)

这次可以了。

![图片](img/169d7064576c5f1c1127f36390b0d738.jpg)

用资源编辑器看看，是 `Delphi/BCB` 程序。用 `IDA` 反编译一下 `dumped.exe`。

![图片](img/99fdaeecf9ef79d748767d826899aa7d.jpg)

看起来 `OEP` 是对的 J。

### 6. 修复 IAT

`IAT` 从这里开始：

![图片](img/e168bc1a3119a73abec2c94bdaaed548.jpg)

结束：

![图片](img/20ff9d9cb9d2151cefffe77716d06f8f.jpg)

`size = 4D58E4 – 4D5168 = 77C`

单步跟了几个 `API` 调用，既有 `redirected` 到壳中的，也有由壳代码 `emulate` 的。从 `5E78E2` 处的异常入手，在第一次断下开始跟（`counter=3`）。

![图片](img/f5161dee824e68d5b9ea88e9748a80c3.jpg)

把 `API`（`MessageBoxA`）的地址保存在了 2 个地方：壳空间和 `heap`。这个地址比进程的映射地址更低。在 `OEP` 可以看到，`IAT` 中很多 `entry` 是指向这里的。

![图片](img/86fd050353ee0a928a03fdd1e62f666a.jpg)



在 `5E78E2` 第 1 次异常时，处理 `user32`, `GDI32`, `ADVAPI32`, `RPCRT4`, `IMM32`, `LPK` 共 6 个 dll。`KERNEL32` 的 api 已经在更早的时候处理过了。此时上面已经填入了约 60 项 `kernel32` 的 api 地址。

相关代码与 `SDP` 以前的版本相似:  

![img/3b25cd1f2bd6ecb285d9b5afccff1e89.jpg](img/3b25cd1f2bd6ecb285d9b5afccff1e89.jpg)

需要注意的 dll 仍然是: `kernel32`, `user32`, `advapi32`, `shell32`, `gdi32`, 以及 `sdprotector.dll`。

在决定以子进程运行前，壳代码处理了上述几个 dll，获得的 API 地址保存了 2 份，分别在壳空间和堆中。

现在试试跟从按子进程运行后，这段代码的处理。试跟了 2 个，并非 `5E78E2` 异常都与 IAT 相关。用脚本试试:  

![文本框: 005E78E2   Single step event at MP3\_Audi.005E78E2counter\_child = 00000013](img/e8357663c11b0ad269031c6f5d79e2f2.jpg)

到这里 IAT 中已经填入了一些重定向值了。从上一个 `int 3` 异常开始跟。

![img/df75690fa1b4fa620716abf8f86d95e7.jpg](img/df75690fa1b4fa620716abf8f86d95e7.jpg)

这里开始了。注意这 2 个函数:  

![img/c487270ff5e41b40c7255bf57297519d.jpg](img/c487270ff5e41b40c7255bf57297519d.jpg)

进去看看:  

![img/ed57555c5707868d90f30d5b3d0ed13b.jpg](img/ed57555c5707868d90f30d5b3d0ed13b.jpg)

连续 6 个这样的函数。`5EEB3D` 进去有些计算，象 `CRC32`(有个初始化 256 byte 向量的动作 J)。不过这已经不重要了，只要稍微想一想，就明白这是在干什么。6 个特殊对待的 dll，显然这些函数用于判断 J。

IAT 中的数据很规范，基本是这样的格式。  

![img/93d030c8e47fea387e42f5551fdbb6a4.jpg](img/93d030c8e47fea387e42f5551fdbb6a4.jpg)

什么时候变成指向 `14xxxx` 或 `5Exxxx` 就是被处理过了 J。

`第 0x12 次 int 3 -> 处理 kernel32.dll`

![img/e2cf6ea55dbe2e93bb9925b54060feed.jpg](img/e2cf6ea55dbe2e93bb9925b54060feed.jpg)

(这个也许不跳也行?我没有试 J)

![img/2f649bfc1f72098023529575656ac654.jpg](img/2f649bfc1f72098023529575656ac654.jpg)

`第 0x13 次 Single step event -> 处理 user32.dll`

![img/dc4c23f75fb9c8226d97eec824b35365.jpg](img/dc4c23f75fb9c8226d97eec824b35365.jpg)

![img/d805c1bf86899c1f8810c8a50a004647.jpg](img/d805c1bf86899c1f8810c8a50a004647.jpg)

`第 0x14 次 Single step event -> 处理 advapi32.dll`

![img/84cd19553e68e511771c12b5b631f5dc.jpg](img/84cd19553e68e511771c12b5b631f5dc.jpg)

依此类推，直到 IAT 全部填充完毕。

![img/ecbabb36952c0d48bbde020770e900f1.jpg](img/ecbabb36952c0d48bbde020770e900f1.jpg)

全部有效。`Fix dump`, 运行:  

![img/2c5628e4df4ee4a8e7d062a65f2c746f.jpg](img/2c5628e4df4ee4a8e7d062a65f2c746f.jpg)

只脱掉壳，并未破解。
