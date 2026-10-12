
# SafeClean V3.02

> 原文：[`www.pediy.com/kssd/pediy03/forum437.htm`](https://www.pediy.com/kssd/pediy03/forum437.htm)

免注册 SafeClean Utilities Version 3.02

环保卫士 SafeClean Utilities V3.02，优秀的垃圾清除软件，装机必备 :)。

用 `fi2.30` 侦测 `SafeCln.exe`，一看是用 `PECompact1.23/1.24` 压缩的，拿 `trw200` 脱壳，用 `trw2000` 加载，按 `f10/f8` 一路跟踪下来，当走到

```
.
.
.
480de3 push ebx <=在这下 pedump //真正的程序入口
489de4 push esi
480de5 push edi
.
.
.
```

这时下 `pedump` 即可，接着随便抄点机器码，用 `winhex` 验证一下是否脱壳成功，好像成功了。脱壳后文件大小为 1,359,872 字节。

`sc2.0` 是 keyfile 保护的(`sc2.0`?不要误会不是 `StarCraft` 而是 `SafeClean2.0`)，`sc3.02` 也是，不过它把 keyfile 写在注册表里：

```
[HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\FileSysSettings]

SystemDataCheck 58C36EF6B9156F9F

SystemDataCode 这个太多了......

SystemDataSize 58
```

现在开始正式工作了，我喜欢用 `s-ice`，确认 `winice.dat` 中有这一句 `EXP=c:\windows\system\advapi32.dll`。用 `loader` 装载，下 `bpx regqureyvalueexa`，按 2 次 `f5` 进入 SafeClean 的地盘，跟着慢慢看程序，我发现它根据 `4ffce4` 置 1 或 0 判断程序是否注册，是 1 则是注册版。然后再次用 `loader` 装载，下 `bpm 4ffce4`，在分析程序，再次省略跟踪若干......

当程序走到这里：

```
467ba0 jnz 467bb8                   ->mov byte ptr [4ffce4],01
467ba2 mov [esi+2c],al              ->jmp 467bb8
467ba5 mov byte ptr [esi+4c],01
```

`jnz` 之前有一个 `call`，是算 key 的，有兴趣的话慢慢看吧，我可没空，按我的方法改，就可成为免注册版。

整理一下，用 `winhex` 修改

查找：`751688462CC6464C01`

改成：`c605e4fc4f0001eb0f`

完活。

有什么错误/问题的话，找我，提前祝各位元旦快乐!!!!

`cracked by zombieys`

`my oicq is 1789655`

`my hp is zombieys.yeah.net/zombieys.126.com`

`thanks for your supports`

附送 `sc3.02` 真正的 key，把它存成 `key.txt`

```
=========Reg Code part 1=========

2931D422F0A86D57

=========Reg Code part 2=========

AA7853A39D5A5BD4D256694BA1999E60A96F04D004492FFD4E1D01D9917DCB359F96803278FCECFA05

=========Finish Reg Code=========
```

`4dd466d0e1e0cc123afbe504da2462ca`
