
# 【半原创】ntfs 磁盘访问接口（工程+代码）

> 原文：[`www.pediy.com/kssd/pediy10/98310.html`](https://www.pediy.com/kssd/pediy10/98310.html)

在北京 MS 培训的时候，无聊偶然在 `nt4` 代码里面发现的好东西，本来是用来做 `ntfs boot` 的，不过单独扣出来，用做用户态的访问也不错。

代码里面提供了若干个接口：

- `BlGetFsInfo`
- `BlClose`
- `BlOpen`
- `BlRead`
- `BlGetReadStatus`
- `BlSeek`
- `BlWrite`
- `BlGetFileInformation`
- `BlRename`
- `BlSetFileInformation`
- `BlReadAtOffset`

看名字知道有什么用了吧。

（读写的时候，只能操作一个扇区整数倍的 `buf`。写操作我把代码删了，大家自己补上）

当然，想要目录的解析也是可以的，直接调用里面的：

- `NtfsLinearDirectoryScan`
- `NtfsInexactSortedDirectoryScan`

就能看到目录下的东西了。

也许大家都知道这个玩意，不过还是分享给那些和我一样菜的人吧![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `ntfsboot.rar` |
|---|---|
