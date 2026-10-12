
# asprotect 脱壳小结

> 原文：[`www.pediy.com/kssd/pediy03/forum380.htm`](https://www.pediy.com/kssd/pediy03/forum380.htm)

先说 1.05 吧，用 `imp_list` 重建时，`address` 文件中显示一个函数名没有，它应该是 `getprocaddress`。执行程序提示"`_ .dll` 找不到"。因为 `asprotect` 要用此函数，所以做了手脚。可以在 `PIMAGE_IMPORT_BY_NAME` 部分可以发现一个地址和其它相差很远，`d` 之，在下面不远处就可看到 `call getprocaddress` 了。`u getprocaddress`，得到地址，将 `PIMAGE_IMPORT_BY_NAME` 中的错误地址改掉，再重建 `import_table` 就成功了。

再讲 1.07，很难跟到入口。我试了很久，在 `int 30` 中有这样一个函数调用：`_vwin32_set_thread_context`。大概 50 多次后就快到了。设 `bpx … if bpcount == 32`。呵呵，自己先记下数，然后减一，我这数不准的，是 16 进制，别忘了。然后 `F10` 跟吧，突然有一步就回到 `asprotect` 代码中了，应该下面出现一个 `call`，`F10` 带过就执行了，要 `F8` 跟进，然后几乎可以 `F12` 了，大概 3 次吧，很快就可看到 `OEP` 了。清掉 `HINT` 后，重建 `imp_table`，有函数名乱码。前面有人讲过方法了。其实直接用其最后一个时的方法就可以了。

另外，对 1.07 及以下版本的 `asprotect`，感觉新版的 `imp_list` 并不如旧的好用，请别将旧的随便删了。

1.1 版，用我的方法也可找到 `OEP`，但 `bpcount` 大约 90 多次。重建 `imp_table` 的工作还没来得及试，更望高手指教。

如果在没出脱壳机之前你用这种方法，厉害！但是，以后就没必要了。看看 `rAD.exe` 的源代码，可以帮助你，设下这个断点。起码这样可以让你了解程序工作的目的是什么？

从头跟踪前一段，不长，而且很有特征，（还可以写个 `Fake Script`，哈哈，开玩笑。）当到达类似这样一段时：

```
015F:00C20EAD  E81634FFFF          CALL       00C142C8
015F:00C20EB2  E8917BFFFF          CALL       00C18A48
015F:00C20EB7  E86489FFFF          CALL       00C19820
015F:00C20EBC  E8BFACFFFF          CALL       00C1BB80
015F:00C20EC1  E8462FFFFF          CALL       00C13E0C
```

在最后一个 `CALL` 按 `F8` 进入后，下指令 `s eip l ffffffff 81,C4,04,F0,FF,FF,50,81`。

找到后，设下此地址断点。以后跟踪方法类似：找特征代码或者找到自己熟悉的代码设个断点 `F5` 过去。（当然想一步是做不到，因为其也是分步还原的。）这样你可以找到 `OEP` 处 `DUMP` 出内存映象。

这样手动脱壳做成的脱壳文件和 `rAD.exe` 脱出来的有点不一样，`OEP` 要设为你找到的 `OEP`(入口点)。好象是 `rAD.exe` 脱出来的最后一层壳没脱，所以你会发现他脱的文件的 `OEP` 和你找到 `OEP` 不一样（你可以用 `BWW` 找，就知道真实的 `OEP` 了），用此法我脱过 `Asprotect v1.07`、`v1.1` 加壳的程序都成功。

不过 `v1.1` 的后期修补工作麻烦一点。

```
bpx _vwin32_set_thread_context if bpount==34
```

`F12`；`F12`，到 `IRETD` 返回。有一半机会回到程序空间，另一半可能回到 `system` 中。如果在 `system` 中，是磁盘缓存在工作，要用 `F8` 小心走一圈，会多入 `_vwin32…` 一次。

回到程序空间应停在一个 `JMP` 上，用 `F8` 至少进一层 `CALL`，一路走下去，大概再退 3 层就快到 `OEP` 了。注意栈中的值，在最后一个 `RET` 处，改为 `JMP EIP`，然后用 `PEditor` `dump` 下来。

`IMP_TABLE` 用新版 `IMP_LIST(V01.10)` 可以自动重建，手功输入 `.idata` 的地址和大小。虽然名字隐掉了，却可以猜到的。大部分 `1.1` 的程序 `HINT` 是空的，不需劳心。如果 `rebuild` 不成，就得在 `OEP` 处清一下了。

用 `TRW` 断这个函数不行，不能每次都截到，`F8` 进 `INT 30` 也经常不成功。用 `ICEDUMP` 来 `dump` 似乎程序也不正常。

这样取得的文件只要改一下 `OEP` 和 `table_address`，贴一下 `.idata`，不再需要其它修改。

我用这个办法将 `http://www.elcomsoft.com/` 上所有口令破解软件都脱了。熟练后 5 分钟一个。

`4dd466d0e1e0cc123afbe504da2462ca`
