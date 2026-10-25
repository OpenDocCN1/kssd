
# WinNavigator v1.95

> 原文：[`www.pediy.com/kssd/pediy04/kanxue021.htm`](https://www.pediy.com/kssd/pediy04/kanxue021.htm)

脱 `WinNavigator v1.95` 的壳

复习用的，所以简写了，跟 `PicturesToExe v3.60` 在一起的，一并下啦。

`unpack(2001.9.15)`

由于是 delphi 程序利用快速寻找 aspr 入口的方法:

执行 `Wn.exe`，用 `prodump` 选 `dump(full)`脱壳,存为 `dump.exe`。接着用 `winhex` 打开 `dump.exe`，选择搜索文本，填 `runtime`,执行搜索，搜到后,向前找到离 `runtime` 最近的机器码为 `55 8B EC` 的地方就是程序的 `oep`，而在 `Wn` 里这个位置在 `offset:001613C0` 处，用 `peditor` 的 `flc` 功能将它转换为 `virtual address` 就是 `00561DC0`----`oep`。

ok，下面要在入口处脱壳，打开 `SuperBPM`，点 `erase`，用 `trw` 载入 `apr.exe`，下 `g 00561DC0`，下 `suspend`。用 `prodump` 选 `apr` 进程 `dump(full)`，再打 `ctrl+n`，`f5`。

接着用 `ImportREC1.2final` 修复 `import table`

```
1    0019B348    KERNEL32.dll    033C    WinExec
1    0019B4C4    KERNEL32.dll    00C7    CreateProcessA
```

选 `add new section`，然后点 `fix dump`。

最后把 `dump\_.exe` 的 `ep` 改成 `16f2a0`。

这是我见过的第 2 个脱壳后能正常运行的软件了，`lucky~~~\``。

我写的够多啦，不是特别难。最最后一次写脱 `aspr` 的东东啦~~~~~~~~~

`crack` 我也不管了，谁有时间就来试试吧。

`2001.9.15`

`zombieys\[CCG\]`

——————————————————————————————->

```
          .-"       "-.      unpacked by zombieys[CCG] >
          /            \    qq:1789655               >
        |      ★       |   http://zombieys.yeah.net  >
        |,  .-.  .-.  ,|  http://zombieys.126.com   >
          |)(\__/  \__)(|    zombieys.cn.hongnet.com  >
          |/    /\    \\|                           >
      (@_@)    (\_    ^^    \_)    Thanks for your supports  >
    _ )\____________\_|IIIIII|__/\_____                           >
_)@8@8{}<_________|-\\IIIIII/-|____China Crack Group_zombieys__>  4dd466d0e1e0cc123afbe504da2462ca
```
