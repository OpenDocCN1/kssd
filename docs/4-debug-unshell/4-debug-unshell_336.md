
# Hying 谈 asprotect

> 原文：[`www.pediy.com/kssd/pediy03/forum807.htm`](https://www.pediy.com/kssd/pediy03/forum807.htm)

我草草翻译了一些内容，可能有些地方不太准确。

作者如果使用一些插件 `SDK` 形式来帮助加壳的话，那么将来可能真的需要手动拼接 `dump` 出来的可执行程序了。

(对了，hying 兄，是否有关于拼接干净的 `exe` 程序的文章？)

不知道将来哪个 Cracker 会来完成如此困难的事情？

我试图按以前的方法重建注册表，没有成功，当然我用的 `imp_list` 出来的

`import0.bin` 文件。没来得及试验 `import1.bin` 文件。

BTW: 以前用 `import1.bin` 可以运行的程序，启动后在 `revirgin` 里面看，还是和探测

加壳程序一样报 `iat` 需要重建。

国内还没见过讨论如此脱壳方式的文章。

＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝

一般来说，我们跟踪目标，找到 `oep`，`dump` 它，然后重建 `iat`，将它们贴回去，这就够了。

不过这回有些变化，`asprotect1.2` 和目标程序有了某种联系：

`asprotect` 在 `492830` 调用了一些目标代码，就是利用 `icedump` 来追踪入口时候，第一站停下来的地方，这里的代码会设置局部内存（local mem allocations），一些标志，然后回到 `asprotect`，最终 `asprotect` 跳到程序真正入口：`4aec88`。

如果你在此处 `dump`，出来的程序无法运行，程序要探测刚才的标志，然后就完蛋了。

我选择了一个简单的方法来解决此问题：

仍在 `4aec88` 处 `dump`，不过之前不让程序跳到 `492830` 的地方，这样内存是干净的。利用 `revirgin` 重建 `iat`，改好 `sections`，不过把入口改为 `492830`，把代码这样改动：

```
seg000:00492944 pop ebp
seg000:00492945 jmp loc_4AEC88
seg000:00492945 start endp
```

看见 `jmp 4aec88` 了吧，之前有个 `ret`。

这样目标程序自己会在启动时候做 `asprotect` 要做的事情，调用这个 `call`，然后会

跳到正常的 `oep`。

现在知道有两个程序：

`Advanced Link Catalog v1.00` (`http://www.wizetech.com/alc/`) and

`Chameleon Clock 2.51` (`http://www.softshape.com`)

.......................

有必要在脱壳过程中跳过这些代码，因为这些东西会引起重建文件以后所有的

`runtime` 错误。但是这样还会在 `4AEE12` 出现错误因为 `0` 送进了 `register` 而不是某个地

址。正如 `Tseph` 说的，这个代码会设置（`asprotect`?）的一些标志，但是它也在程

序启动时候查询注册表来设定程序的一些参数，比如语言，`timeformat`, `showdate`,

`reg` 等等。我可以让这个程序运行（这句话的 `w/o` 没看懂所以没的翻译），

`shareware nag` 可以运行，程序也启动了，但是此程序使用 `winamp` 类似的 `skin`，照

我的方法 `patch` 程序，这个功能就不能用了........

提到的其他程序，`Advanced Link Catalog`，我发现脱壳中只要跳过调用程序的代

码即可，这个 `call` 可以在 `dumping` 前用 `/Tracex` 来找到(注：我现在才知道 `icedump` 里

有这等好命令^\_^)。不管什么原因吧，程序可以在正常 `oep` 地方启动而不用处理丢

失的代码。

此两个程序的注册方式和 `Asprotect` 都有联系，至少 `Registration Key` 可能在加壳

的时候产生。我不知道注册过程在脱过的文件中是否还存在，但是 `256` 个 `byte` 的注

册 `key` 仍是对未加补丁程序有效的保护。`ALC` 至少可在 `Hz` 和我找到的地方做进一步的

限制。

\========================================================

`Cordialement` `4dd466d0e1e0cc123afbe504da2462ca`
