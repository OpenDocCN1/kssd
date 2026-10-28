
难点在于这个函数的返回值是个 `__int64` 型的。大家要有比较熟练的 C 或者汇编基础啊。哈哈，或许还需要一些关于内存的知识。涉及到一些公司的内部库，源码我就不附上来了啊。

---

**Ps：** 小弟最近研究如何在内核态下实现变速齿轮。请看帖：<http://bbs.pediy.com/showthread.php?t=142326&highlight=KeUpdateSystemTime>。将该文章的思路用 C 写驱动实现了一下，貌似效果很不理想。初次发帖，在此抛砖引玉，希望跟大家讨论下变速齿轮的实现……（我想用驱动实现对整个系统时间都加速……）

---

**附：我在上面那篇帖子里的回复：**

内核中的地址 `0xffdf0000` 对应的物理内存，映射到每个进程中的地址是 `0x7ffe0000`。实际上这个地址对应的是一个 `_KUSER_SHARED_DATA` 结构：

```
nt!_KUSER_SHARED_DATA
   +0x000 TickCountLow     : Uint4B
   +0x004 TickCountMultiplier : Uint4B
   +0x008 InterruptTime    : _KSYSTEM_TIME
    ... ...
```

`+0` 和 `+8` 处的两个结构对实现变速齿轮有用。因为三环下：`timeGetTime`、`GetTickCount`、`QuerPerformanceCount` 三个获取时间的函数，实际上主要的功能是查询 `+0` 或者 `+8` 地方的值……（PS：打开 OD 自己看着三个函数的实现……）

（顺道多嘴，Hook 着三个函数就能实现一个三环下的变速齿轮。当然，扫雷不能加速，因为它是用的定时器（timmer）计时。）

**正题：** 内核下，在虚拟机中调试，看到的是由 `KeUpdateSystemTime` 负责更新！ `_KUSER_SHARED_DATA` 中的时间。（具体自己打开你的 WinDbg + 看看 win 源码）……

楼主说的方法类似于 Hook `KeUpdateSystemTime` 函数，让其更新时间的时候被我们乘机修改？我试了 Hook 这个函数更新系统时间的，加速 2 倍，结构受到的效果不是系统时间加速两倍，而是系统时钟走得断断续续，加速别的程序什么的也不成……

这个直接的、仅仅地修改 `0xffdf0000` 的方法，应该是不能实现变速齿轮的……楼主的变速齿轮，下过来运行直接蓝屏，应该是硬编码的挂钩地址吧？

看了一下“守望者变速齿轮”的实现……无奈能力有限，只能看到其 Hook 了 `KeUpdateSystemTime`，这个可以完整逆向偷学来……但是其修改了 `hal.dll` 等的系统 dll……不好搞，不知道怎么搞……寻求实现变速齿轮内核版的方法 ing……

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [附编译好的dll和代码注入器.rar](javascript:void(0);)

  （1.34MB，627次下载）
