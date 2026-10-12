
# ACDsee Trial Version 3.1 Build 921

> 原文：[`www.pediy.com/kssd/pediy03/forum781.htm`](https://www.pediy.com/kssd/pediy03/forum781.htm)

Topic：破解 90 天试用期

Software: `ACDsee Trial Version 3.1 Build 921` (5.69MB)

《电脑报 2000 年合订本配套光盘》有赠

Tools：`W32dasm89` 和 `Hiew32`

Author: 看雪学苑学员 `wangliqiang`

Date: 2001-4-3

## 准备工作

1. 安装 `ACDsee Trial Version 3.1 Build 921`，改系统日期为 2002 年（超过安装日期 90 天的任意时间）。
2. 运行 `ACDsee.exe`，出现过期提示框，程序不能运行。

## 破解过程

1. 用 `W32dasm89` 载入 `ACDsee.exe`
2. 单击 `Find Text`，查找目标填入 `TrialPeriod_GetDaysLeft`（汉语意思是：试用期\_ 剩余天数），
3. 按确定，找到下面的文字：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00404558(C)
|
* Reference To: InTouchClient.?TrialPeriod_GetDaysLeft@InTouchClient@@QAEHXZ, Ord:005Dh

                          |
:00404596 FF1564824D00               Call dword ptr [004D8264]
:0040459C 85C0                       test eax, eax
:0040459E 7416                       jg 004045B6     <---- 把此处 jg 改为 je>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Reference To: InTouchClient.?g_itc@@3VInTouchClient@@A, Ord:0069h

                          |
:004045A0 8B0D30824D00               mov ecx, dword ptr [004D8230]
:004045A6 6A00                       push 00000000

* Reference To: InTouchClient.?TrialMode_ShowExpiredBox@InTouchClient@@QAEHPAUHWND__@@@Z, Ord:005Bh

                          |
:004045A8 FF155C824D00               Call dword ptr [004D825C]
:004045AE 85C0                       test eax, eax
:004045B0 7521                       jne 004045D3
:004045B2 83C408                     add esp, 00000008
:004045B5 C3                         ret
```

用 `Hiew32` 把上面有 `^` 标记处 `jg` 改为 `je`。运行 `ACDsee.exe`，不再出现过期提示框，所有功能正常。

欣喜若狂、大功告成。

**注：** 以上破解适用于软件安装 90 天后。

当软件安装不足 90 天而把系统日期恢复当前正确日期时，会出现过期提示框，请大家告诉我解决的办法。

## Postscript: 动态破解工具 TRW2000 使用中遇到的难题

自 2001/3/27 在《看雪破解论坛》发表用《W32dasm 和 Hiew32 去除 ScreensaverShot 的 Nag》以来，我一直在想，

我能不能学着用动态破解工具 `TRW2000` 来提高自己的破解水平。于是，我从网上下载了一个 `TRW20001.22` 全解密

汉化版,参考了看雪网站破解教学栏目的相关资料，然后，试着运行 `TRW2000`，奇怪的是每次都死机，百思不得

其解。我的电脑是联想同禧 `PentiumIII 800`，键盘和鼠标都是 `USB` 接口，原因在此吗？狠一狠心，我又重新买

了 `PS/2` 键盘和鼠标，安装后运行 `TRW2000`，一切正常，困扰我多时的问题终于得到了解决。
