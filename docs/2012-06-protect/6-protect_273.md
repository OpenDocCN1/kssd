
# Xtrap 驱动逆向(有 Bin,有码,有真相)

> 原文：[`www.pediy.com/kssd/pediy11/116037.html`](https://www.pediy.com/kssd/pediy11/116037.html)

`NP` 跟 `HS` 加了 `VM` 保护,搞不定,发现 `Xtrap` 没有用 `VM` 保护,于是拿来分析了下,

用的是老版本 `XDva220`,因为之前在老版本做的静态分析,有些信息懒的在新版本上去改动了,

现在的版本 `XDva280` 看了看,好像变化不大,估计只是针对后面系统的更新或者

多了些新的 `ControlCode`.

逆向的部分尽量保证接近原始汇编代码,还有 5 个 `IoControlCode` 没有逆向,

`IoControlCode_85000880`

`IoControlCode_850008C0`

`IoControlCode_85000894`

`IoControlCode_85000840`

`IoControlCode_850008C4`

感兴趣的话,大家可以接着逆.

其他部分都逆向完了,数据部分还原了大部分,有的不知道什么意思的就用 `dummy` 代替

另外有 2 个 `DeviceExtension` 不知道是什么结构,这个只有自己定义了 2 个,

`XTRAP_DEVICE_EXTENSION2` 跟 `XTRAP_DEVICE_EXTENSION3`(为了保证与原始汇编一致),

有逆向得不对的地方或者发现其他结构不对的,欢迎大家指出,一起修正.

另外 `IDB` 文件中间有些部分比如它所加密的字符串已用脚本解开,因为结构挺大,

所以写了个脚本处理结构的变量名字打印保存到文本.附件中包含 `py` 脚本.

我所用的 `DDK` 是 2600.1106 这个版本,它的 `_DEVOBJ_EXTENSION` 定义没有 `AttachedTo`,

所以这个需要手动加上,不然不能够编译通过.

欢迎转载,请保持文章完整性.

`Revers by dengkeng, Email:poppig@sohu.com`

附件说明:

`DeCode.py` 是对应的脚本文件,`IDA5.5 + IDAPython 1.2`

原始驱动:目录包含对应 `IDB` 文件以及驱动.

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `XTrap.rar` |
| :--- | :--- |
