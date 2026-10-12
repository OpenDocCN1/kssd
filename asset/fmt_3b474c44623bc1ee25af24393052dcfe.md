
# 世界主要城市与北京时差表

> 原文：[`www.pediy.com/kssd/pediy04/kanxue113.htm`](https://www.pediy.com/kssd/pediy04/kanxue113.htm)

申请加入 `[BCG]` 的第一篇破文

软件名称：世界主要城市与北京时差表 (300K 左右)

软件简介：如果你有朋友在国外，你是否时常想知道他 (她) 现在在什么，这时候知道他 (她) 所处地区的时间就是一个问题，本软件通过一个简单直观的界面帮你解决这个问题！软件的功能调用都通过鼠标右键快捷菜单实现！本软件是共享软件，有 30 天试用期限！

下载地址：`http://www.shareware.net.cn/download.asp?id={DF75409D-69D1-47BD-86AE-01898CF06AD2}`

1. 用 `WDASM893` 反汇编不成功。于是用 `BW2000` + `TRW` 的 `PEDUMP` 脱壳后再反汇编。找到注册成功字样！

```
:0044F98B E8105DFDFF              call 004256A0
:0044F990 8B45EC                  mov eax, dword ptr [ebp-14]  ***关键
:0044F993 8B55FC                  mov edx, dword ptr [ebp-04]  ***真正注册码
:0044F996 E85543FBFF              call 00403CF0
:0044F99B 7535                    jne 0044F9D2       ****此处跳则失败

* Possible StringData Ref from Code Obj ->"注册成功！非常感谢您的支持！"
                                  |       ******************************
:0044F99D B844FA4400              mov eax, 0044FA44
:0044F9A2 E8C16EFFFF              call 00446868
:0044F9A7 8D55E8                  lea edx, dword ptr [ebp-18]
:0044F9AA 8B8308030000            mov eax, dword ptr [ebx+00000308]
:0044F9B0 E8EB5CFDFF              call 004256A0
:0044F9B5 8B55E8                  mov edx, dword ptr [ebp-18]
:0044F9B8 B8BC784500              mov eax, 004578BC
:0044F9BD E8                      BYTE e8h
:0044F9BE F2                      BYTE f2h
:0044F9BF 3F                      BYTE 3fh

* Referenced by a CALL at Address:
|:0044B749
|
:0044F9C0 FB                      sti
:0044F9C1 FFC6                    inc esi
:0044F9C3 05B8784500              add eax, 004578B8
:0044F9C8 018BC3E89C09             add dword ptr [ebx+099CE8C3], ecx
:0044F9CE FFFF                    BYTE  2 DUP(0ffh)
:0044F9D0 EB0A                    jmp 0044F9DC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044F99B(C)
|

* Possible StringData Ref from Code Obj ->"对不起,您的注册失败!"
                                  |
:0044F9D2 B86CFA4400              mov eax, 0044FA6C
:0044F9D7 E88C6EFFFF              call 00446868
```

2. 启动 `trw` 在 `0044F98B` 处设断。在 `0044F990` 处 `D EAX` 应该可以看到你的注册码 (注：由于我设断时只填了注册名 `SOFT`。没填注册码。显示的值为 `0`)。过了 `0044F993` 再 `D EDX` 可见到真注册码 (`000000748176`)。

3. 接下来试试在 `0044F99B` 处下 `R FL Z` 命令。按 `F10` 往下执行。ok。显示注册成功。重启后。`NAG` 窗口消失，已成为注册版本。------------哈哈。连输入注册码的功夫也省下了。`4dd466d0e1e0cc123afbe504da2462ca`
