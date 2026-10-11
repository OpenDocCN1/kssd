
# IDA 中制作 sig 文件的问题

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-341.htm`](https://www.pediy.com/kssd/pediy09/pediy09-341.htm)

> 手头上有一个 `MFC42u.lib` 文件，想将它转化为 IDA 的 `sig` 文件，查了网上相关资料，好像是要先通过 `pcf.exe` 将其转化为 `pat` 文件，然后再用 `sigmake` 将 `pat` 文件转化为 `sig` 文件，于是：在命令行窗口中输入：`pcf mfc42u.lib mfc42u.pat`

> 结果出现错误提示：`Fatal [mfc42u.lib] <MFC42u.DLL> not a coff module.` 而无法转化为 `pat` 文件![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

> 不知是操作方法不对，还是这个 `mfc42u.lib` 库文件格式不能转化为 `sig` 文件。

> 请高手指点，先谢了!

> 用 `plb` 试试。

> `plb mfc42u.lib mfc42u.pat`

> 然后再 `sigmake`

> 出现冲突的时候把 `exc` 文件里面的前三行去掉，然后

> `sigmake -xmfc42u.exc mfc42u.pat mfc42u.sig`

> 这样就可以了。

> 是我回复太短了

> `pcf` 用于 COFF 格式 (VC), `plb` 用于 OMF (Borland), 如果有冲突, 会生成一个 `EXC` 文件

4dd466d0e1e0cc123afbe504da2462ca
