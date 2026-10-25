
# FPE 2001 Demo

> 原文：[`www.pediy.com/kssd/pediy03/forum1193.htm`](https://www.pediy.com/kssd/pediy03/forum1193.htm)

## 《FPE 2001 Demo 的延时修改》

**函数调用的重要性**

**破解作者**: `yuppc` `[FCG]` `[BCG]`

**破解时间**: 2001 年 7 月 31 日

**破解难度**: 易

**注**: (转载希望保持完整,文章归 `BCG`、`FCG` 所有)

### 软件概述

1. 最老牌的国产游戏修改器,也是最好的("我一直用他"),它的演示完全免费的,所以我们在这只谈"修改"!
2. 新版本比 `2000` 版增加了很多的功能呀(`自动机器人` `\` `网络修改` `\` `....`),一级棒!
3. 可以反编译
4. 下载地址: `http://www.esoftware.com.cn/oload.php?url=http://ftp.eware.com.cn/pub/game/gatools/fpe2001trial.zip`
5. 真是最好的游戏修改软件!!

**修改目的**: 为加入 `CCG` 奋斗!!

**破解工具**:

1. `w32dasm` 黄金版 ===> 静态反汇编
2. `hiew` ===> 16 位编辑器

### "修改"开始

1. 首先要了解 `WINDOWS` 个应用程序是要对 `WINDOWS` 系统函数的调用来实现某些功能的(对 `CRACKER` 来说用动态下断就是对函数调用的拦截);
2. 在 `FPE2001` 和以前的版本一样喜欢炫耀自己(启动时不停的闪烁各个功能页,结尾是也是要停一会),这是如何实现的呢---->调用函数(`Sleep` , `SleepEx` 等函数）；
3. 安装软件,并运行;
4. 观察:开始运行时闪烁 9 次,关闭时闪烁一次并发现一个"关于"中关键词"EXIT……";
5. 开始用 `W32dasm` 反汇编,查找关键词"EXIT"(先解决关闭时的闪烁),代码如下:(注意函数的调用)

```
:00406513 803F00              cmp byte ptr [edi], 00   ====>关键比较
:00406516 0F84ED020000        je 00406809     ========>关键跳(应该跳过 SLEEP 函数的调用)
:0040651C 8B93D0020000        mov edx, dword ptr [ebx+000002D0]
:00406522 807A4000            cmp byte ptr [edx+40], 00
:00406526 7413                je 0040653B   ====>此处无用(因为要跳过 EXIT)
:00406528 33D2                xor edx, edx
:0040652A 8BC3                mov eax, ebx
:0040652C E853170000          call 00407C84
:00406531 68D0070000          push 000007D0

* Reference To: KERNEL32.Sleep, Ord:0000h  =======>Sleep 函数调用
|
:00406536 E84D8C0900          Call 0049F188

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00406526(C)
|
:0040653B 8B9354020000        mov edx, dword ptr [ebx+00000254]
:00406541 8B83C8010000        mov eax, dword ptr [ebx+000001C8]
:00406547 E87A180800          call 00487DC6
:0040654C 66C746100800       mov [esi+10], 0008

* Possible StringData Ref from Data Obj ->"Exiting...."  ====>查找标志(停到这)
|
:00406552 BA01454A00         mov edx, 004A4501
:00406557 8D45F8              lea eax, dword ptr [ebp-08]
:0040655A E8499A0200          call 0042FFA8
```

6. 修改软件启动时的 9 次闪烁,查找关键词"Sleep"(函数名查找),应该找到连续有 9 个 `SLEEP` 函数调用的程序段,代码如下:

```
:004025A5 8BD8                mov ebx, eax
:004025A7 33D2                xor edx, edx
:004025A9 8BC3                mov eax, ebx
:004025AB E878EF0000          call 00411528    ===>设定标志位的 CALL
:004025B0 80BB3025000000      cmp byte ptr [ebx+00002530], 00  ====>标志位比较
:004025B7 0F85E2000000        jne 0040269F  =======>关键跳(不跳则闪烁 9 次)
:004025BD C6833025000001      mov byte ptr [ebx+00002530], 01
:004025C4 6A64                push 00000064

* Reference To: KERNEL32.Sleep, Ord:0000h=======>Sleep 函数调用
|
:004025C6 E8BDCB0900          Call 0049F188
:004025CB 8B93D0010000        mov edx, dword ptr [ebx+000001D0]
:004025D1 8B83C8010000        mov eax, dword ptr [ebx+000001C8]
:004025D7 E8EA570800          call 00487DC6
:004025DC 6A64                push 00000064

: ===================>中间有 7 个 Sleep(总共有 9 个 Sleep,知道吗?)

:0040267F E842570800          call 00487DC6
:00402684 68B0040000          push 000004B0

* Reference To: KERNEL32.Sleep, Ord:0000h=======>Sleep 函数调用
|
:00402689 E8FACA0900          Call 0049F188
:0040268E 8B93CC010000        mov edx, dword ptr [ebx+000001CC]
:00402694 8B83C8010000        mov eax, dword ptr [ebx+000001C8]
:0040269A E827570800          call 00487DC6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004025B7(C)  ===========>跳跃地址(向上看)
|
:0040269F 5B                  pop ebx   ====>启动的地址(最好直接跳到此)
:004026A0 C3                  ret
```

7. 动手改动(你的最爱),启动 `HIEW` ,改动地址 `00406516` 和 `004025B7` ,将他们全改为 `jmp` 就行了(位数不够的加 `nop`) :D
8. 试试吧,舒服多了吧.(所以用静态汇编时应注意程序对关键函数的调用)
9. 看雪主站上有关于 `Windows` 函数的调用,掌握了函数将事半功倍!!
10. 收工,睡觉了!

-----------------------------------------------

`FCG` -------> `Http://WWW.CN778.NET`

`BCG` -------> `http://crackerabc.longcity.net/`  `4dd466d0e1e0cc123afbe504da2462ca`
