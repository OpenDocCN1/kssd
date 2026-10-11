
# Diablo 2 暗黑破坏神

> 原文：[`www.pediy.com/kssd/pediy04/kanxue101.htm`](https://www.pediy.com/kssd/pediy04/kanxue101.htm)

这篇文章来自

`http://myhomeland.home.sohu.com/`

因为讨论 `game` 破解的很少

《暗黑破坏神》2 代出来有一段时间了，我原先等着买正版，结果正版一拖再拖，只好买 D 版了。这个游戏太大了，最小安装居然都要 500 多兆！而且还得用第 2 张光盘，可怜我的 4 速光驱！。。。。。。看来只好搞定它了！

1. 选最小安装。
2. 按封面的要求把 `crack` 目录下的 `game.exe` 拷到安装目录，并把 `HKEY_LOCAL_MACHINE\Software\Blizzard Entertainment\Diablo II` 下的 `DiabloIICD` 的值改为`.`（不改也行，改了会快一点）。
3. 把第 2 盘上的文件拷到按装目录，其中 `D2char.mpq` 一定要拷，否则在游戏中完全看不见主角。`D2music.mpq` 可以不拷，但是这样就没有背景音乐了。
4. 用 `TRW2000` 载入 `game.exe`。
5. `g GetDriveTypeA`
6. 拦住后按几次 `F12`，直到进入 `Game.exe` 的领空，可以看到：

```
:00401468 E89B090000 call 00401E08
:0040146D 85C0 test eax, eax
:0040146F 740E je 0040147F
:00401471 B9021E4000 mov ecx, 00401E02
:00401476 E881090000 call 00401DFC
:0040147B 85C0 test eax, eax <==出来后就在这里
:0040147D 7511 jne 00401490 <==改改试试吧，果然搞定！
```

总之，只要---

找到：`E89B09000085C0740EB9021E4000E88109000085C07511`

改为：`E89B09000085C0740EB9021E4000E88109000085C0EB11`

其实这个游戏也可以改 `Storm.dll`（就象星际争霸那样），但我懒得再跟下去了。

注意：改好后游戏还会提示放入光盘，按"取消"即可！

希望各位热爱 `crack` 的同志跟我交流交流心得。

`DayDreamer` `2000.7.10`

`ihavenodream@China.com` `4dd466d0e1e0cc123afbe504da2462ca`
