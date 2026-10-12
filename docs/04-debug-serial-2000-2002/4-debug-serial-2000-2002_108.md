
# System Mechanic 3.5

> 原文：[`www.pediy.com/kssd/pediy02/forum135.htm`](https://www.pediy.com/kssd/pediy02/forum135.htm)

现在 3 元一小时，可一天不上网总觉少了些什么，只好少来了！现在真是理解 `ICEWORLD` 兄了, \_\_^^\_\_!

`SYSTEM MECHANIC 3.5` 的注册码。

程序用 `aspack` 压缩过。

用 `trw2000` 载入，选 `enter code`

`name`: `backboy`

`code`: `78787878`

设断点 `bpx 0137:004943c9`

`F5(enter)`

`ok(enter)`

`d eax(enter)`

相关段如下 `0137：00494388 push eax` `D EAX` 可见正确注册码 `D EDX` 可见 输入的假码

`0137: 0049439a pop eax`

`0137: 0049439b call 00403f58` 此处进行注册码比对

`0137: 004943a0 jnz 004943a7` 此处一跳就..

此处得出的注册码与注册机给出的不同，但都能用。

一个可用的注册码如下：

`NAME`: `backboy`

`CODE`: `90919-PR845-7747173554`

程序将注册名及注册码保存在注册表的 `HKEY_CURRENT_USER\Software\iolo\System Mechanic` 下。

另发布 `winhex5` 的修改方法给需要的朋友，我特别爱用它，一共改 2 处。

1. 改 `4500007515B003E8B7C5FBFF` 为

   `------EB3A--------------`

2. 改 `803D9875450000` 为

   `-------------1`

另请教看雪我用的是 `win97osr`，不知为什么不能加载 `frogice`，一运行系统就死机。我已按 `frogice` 的说明中将相应的 `vxd` 文件放入了

相关目录下了。 `4dd466d0e1e0cc123afbe504da2462ca`
