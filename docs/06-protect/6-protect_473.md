
# 修改 SoftIce 使其在 2000/xp 下躲过检测

> 原文：[`www.pediy.com/kssd/pediy05/pediy50376.htm`](https://www.pediy.com/kssd/pediy05/pediy50376.htm)

修改 `SoftIce` 使其在 `2000/xp` 下躲过检测。方法来自这里 `http://www.woodmann.com/upload/showthread.php?s=&threadid=1806`。

在这里我想说说自己的一点修改经验，其实基本上和上面的帖子所说的内容相同。只是附加一点自己的解释。为了在 `2000/xp` 下躲过对 `SoftIce` 的检测，我们需要修改三个文件。其中 `NTICE.SYS` 和 `SIWVID.SYS` 位于 `%windir%/system32/drivers` 目录下。`NMTRANS.DLL` 位于 `SI` 的安装目录。首先，我们要明白在 `2000/xp` 下有哪些手法来检测 `SoftIce`，最常见的方法就是用 `CreateFileA` 的方式来检测 `NTICE` 和 `SIWVIDSTART` 的存在，这是 `2000/xp` 下打开设备驱动的方法。还有就是利用 `INT 1` 和 `INT 3` 的方式和 `GF....MJ` 检查。在这，我们只说怎么躲过这些方法的监测。至于反单步跟踪，那需要你自己的调试技巧，不在本文范围之内。

以下是如何修改那三个文件，让我们的 `SoftIce` 具有 anti-detect 功能。

**提醒!!**

两个驱动文件 `NTICE.SYS` 和 `SIWVID.SYS` 被修改后一定要修正其校验和 `checksum`，这个可以使用 `lordpe` 完成。否则系统启动将失败!

### NTICE.SYS 的修改

1. 查找 `GFu\*f` 串，修改为 `XFu\*f`，其后的 `MJ` 修改为 `XJ` <其实随便修改成什么都可以>
2. 查找所有的 Unicode 字符串 `\\NTICE`，修改为 `\\XTICE` <躲过 `NTICE` 检测，注意前后一致>
3. 查找 Unicode 字符串 `Bchkd`，修改为 `Xchkd` <躲过 `SoftIce` 的 `Boundscheck` 后门检测>
4. 查找字符串 `\\Driver\\NTice`，修改为 `\\Driver\\XTice`
5. 查找 `Kernel32!UnhandledExceptionFilter`，修改为 `user32!MessageBoxExW` <这个是我的修改方法，必须注意的是这样做了后，`SoftIce` 的 `Faults On` 将失效。可以躲过一些 `INT 3` 检测，自己选择要不要修改吧>

### SIWVID.SYS 的修改

1. 查找所有的 Unicode 字符串 `\\SiwvidSTART`，修改为 `\\XiwvidSTART` <嗯嗯，躲过 `SIWVIDSTART` 检测>

### NMTRANS.DLL 的修改

1. 查找字符串 `\\\\.\\NTICE`，修改为 `\\\\.\\XTICE`

至此我们所作的都已经做完。再次提醒一定要修正 `NTICE.SYS` 和 `SIWVID.SYS` 的校验和!

经过上面的处理后，你的 `SoftIce` 的 anti-detect 将大大加强。但是仍然躲不过 `INT 1` 的检测方法。

### 躲过 INT 1 检测 SoftIce 的方法如下：

在 `SoftICE` 里键入 `IDT` 指令，我们将获得 `IDTBase` 的值。在我的 `XP` 下总是 `8003F400`。

然后 `d IDTBase`，可以看到偏移为 `0DH` 处的字符是 `EE`，我们将其修改为 `8E` 即可躲过 `INT 1` 的检测。

`XP` 下的可以在 `SoftIce` 的启动运行的指令里加上 `eb 8003F40D 8E`，这样每次 `SI` 启动就自动具有了 anti-`INT 1` 检测的功能。别的系统的话请多起启动确认一下是否是固定的位。如果不是，你只有每次手动修改了。

`4dd466d0e1e0cc123afbe504da2462ca`
