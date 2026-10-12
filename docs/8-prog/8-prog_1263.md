
# 接着讲和 DLL 与代码插入有关的几点看法

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-700.htm`](https://www.pediy.com/kssd/pediy08/pediy8-700.htm)

`?`微软的 `detours` 本质上和一些 `trojan` 的方法是一样的，只不过它的本意是用来帮助调试程序。如果你的程序调用一些 `API` 后并不判断返回的结果和一些可能的出错情况，就可能会 `crash`。他的目的就是帮助你的程序多崩溃几次，让你别太得意。

进程内的 `API` `hooking` 一般有两种：`patch` `Import` `Table`，将主程序的输入函数的入口地址改成你自己的，几年前的病毒和木马爱用，相对简单一些；一种就是 `detours` 和不少木马的 `direct` `function` `entry` `point` `patching`，通过查找对应 `DLL` 的输出函数表得到 `API` 的起始点，然后简单地反汇编和判断一下，一般保留开始的 5 个 `byte`（也可能更多），然后从起始点改成 `JUMP`（`E9` `xxxx` `xxxx`）到你的函数点去执行。所以如果原来的 `API` 还不到 5 个字节，你是没法改它的，否则你如何 `JUMP`?

在你的函数里，可以有两条执行路径：

1. 在你的函数执行完的 `RET` 之前，先执行你前面保留的原来的函数的入口点的指令，然后 `JUMP` 回原来的被 `patch` 的函数的后面一条指令继续执行。
2. 先 `CALL` 原来的函数的指令继续执行，等到它返回后，在你的函数里做你想做的逻辑判断和运算，比如可能修改函数的返回值和 `error` `code`，或者 `buffer` 里的内容等等，然后 `RET`。

`Direct` `function` `entry` `point` `patching`(`detours` 只是里面的一种)有它的先天缺陷，不能 `patch` 一些和同步有关的 `API`，比如 `WaitForSingleObject(Ex)` / `WaitForMultipleObjects(Ex)`，否则会死锁。不信？你试试。原因？不知道![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)。

\===================================

看看 `detours`( http://research.microsoft.com/sn/detours/ )的例子程序就会明白，想把你的 `DLL` 放到目标程序的进程中去，可以有 3 种方法：

1. 直接修改目标程序的磁盘文件，改动它的 `import` `table`，把你的 `DLL` 放进去。一个例子就是 `detours` 的 `setdll` 示范程序。好处是快，你不用修改你的源程序；坏处是如果你的目标程序有相关的保护机制，比如加了数字签名（`digital` `signature`）或者各种 `hash`，你只要改了一个 `byte`，就会破坏它，程序里只要调用 `crypto` `API` 或者重新 `hash` 验证一把就会发现，让你的修改穿帮。
2. 运行你的目标程序，`CreateProcess(...,CREATE_SUSPENDED,...)`，然后修改你的目标进程的里的 `import` `table`，把要插入的 `DLL` 放进去，然后 `ResumeThread`。这和系统的 `PE` `Loader` 关系很大，对它更熟悉的大牛可以继续补充。前提条件是你要有足够的权限调用 `ReadProcessMemory`/`WriteProcessMemory` 察看和修改别的进程空间里的 `PE` `Image` `header`，比如 `local` `administrator`。`detours` 的例子是 `withdll`。更多的细节你可以在 `windbg` 或者 `ollydbg` 里慢慢看。
3. 就是很多 `trojan` 爱用的，`CreateRemoteThread`。细节可参考 `MSDN`，那里的说明最准确和详细。微软也不是傻瓜，`XP` `SP2` 和 `windows2003` `SP1` 就已经改进了不少，还在不停地打补丁。漏洞肯定还有很多，但长远看肯定是越补越少，和我们维护改进自己的程序是一个道理。不知道 `Vista` 上面还能不能用，等它正式出来后再看吧。这也就是为什么很多 `hacker` 开始打 `rootkit` 的注意，从 `kernel` 和 `device` `driver` 下手。`user` `mode` 下的这些方法已经不太好使了。不少调试和测试软件用的手法和病毒木马是差不多的，只是目的不同而已。

方法 3 的主要缺点是不安全，因为是在别的进程的空间里，俩眼一抹黑，真是摸着石头过河。为什么呢？因为第一你不知道目标进程是不是多线程的（当然有办法，比如一些未公开的 `API`，但肯定麻烦和不可靠），第二你不知道你创建的 `remote` `thread` 在 `patch` 某个 `API` 的时候，是不是有某个倒霉的 `thread` 正好执行到这个入口点上或者已经在函数里面（这是我的考虑，可能不对）。

\*\*\*\*\*

有一点是我到目前为止在 `google` 上所有能看得到的 `CreateRemoteThread` 方法的疏忽。就是你改动了某个 `API` 的入口点，但这仅仅是改动了内存里的代码，而现在计算机的 `CPU` 里一般至少有 `Level` `1` 和 `Level` `2` 的多级指令缓存 `cache`，`cache` 里可能就有该函数的旧的入口点代码，可能就要被某个 `thread` 执行。指令的不一致性肯定不安全，最好在调用 `WriteProcessMemory()`修改之前先 `CALL` 一把 `FlushInstructionCache`（）。否则在单 `CPU` 的机器上可能还不明显，在多核（`multi-core`）的机器上是早晚会撞到鬼的，崩溃了还不知道是怎么回事，查 `crash` `dump` 都未必看得出来。

\*\*\*\*\*

我这里主要是从程序设计和调试的角度谈问题，你想怎么用全看你自己。和扑克牌一样，我们既可以用来消遣，也可以来赌博。

4dd466d0e1e0cc123afbe504da2462ca
