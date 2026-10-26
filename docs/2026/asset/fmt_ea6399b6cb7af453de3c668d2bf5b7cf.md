
# 破解90天试用期 (2千字)

- Topic：破解90天试用期
- Software：ACDsee Trial Version 3.1 Build 921 (5.69MB) 《电脑报2000年合订本配套光盘》有赠
- Tools：W32dasm89 和Hiew32
- Author：看雪学苑学员wangliqiang
- Date：2001-4-3

## 准备工作：

安装ACDsee Trial Version 3.1 Build 921，改系统日期为2002年（超过安装日期90天的任意时间），运行`ACDsee.exe`，出现过期提示框，程序不能运行。

## 破解过程：

1. 用W32dasm89载入`ACDsee.exe`

2. 单击Find Text，查找目标填入`TrialPeriod_GetDaysLeft`（汉语意思是：试用期_剩余天数），按确定，找到下面的文字：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404558(C)

* Reference To: InTouchClient.?TrialPeriod_GetDaysLeft@InTouchClient@@QAEHXZ,
Ord:005Dh
            |
:00404596 FF1564824D00            Call dword
ptr [004D8264]
:0040459C 85C0
  test eax, eax
:0040459E 7416
  jg 004045B6    <----  把此处jg 改为je>
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* Reference To: InTouchClient.?g_itc@@3VInTouchClient@@A, Ord:0069h
            |
:004045A0 8B0D30824D00            mov ecx, dword
ptr [004D8230]
:004045A6 6A00
  push 00000000
* Reference To: InTouchClient.?TrialMode_ShowExpiredBox@InTouchClient@@QAEHPAUHWND__@@@Z,
Ord:005Bh
            |
:004045A8 FF155C824D00            Call dword
ptr [004D825C]
:004045AE 85C0
  test eax, eax
:004045B0 7521
  jne 004045D3
:004045B2 83C408
add esp, 00000008
:004045B5 C3
    ret
```

用Hiew32把上面有^标记处`jg` 改为`je`。运行`ACDsee.exe`，不再出现过期提示框，所有功能正常。

欣喜若铩⒋蠊Ω娉伞

注：以上破解适用于软件安装90天后。

当软件安装不足90天而把系统日期恢复当前正确日期时，会出现过期提示框，请大家告诉我解决的办法。

## Postscript: 动态破解工具TRW2000使用中遇到的难题

自2001/3/27在《看雪破解论坛》发表用《W32dasm和Hiew32去除ScreensaverShot的Nag》以来，我一直在想，我能不能学着用动态破解工具TRW2000来提高自己的破解水平。于是，我从网上下载了一个TRW20001.22全解密汉化版,参考了看雪网站破解教学栏目的相关资料，然后，试着运行TRW2000,奇怪的是每次都死机，百思不得其解。我的电脑是联想同禧PentiumIII 800,键盘和鼠标都是USB接口，原因在此吗？狠一狠心，我又重新买了PS/2键盘和鼠标,安装后运行TRW2000，一切正常，困扰我多时的问题终于得到了解决。
