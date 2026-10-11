
# 鹦鹉螺网络助手 v1.68

> 原文：[`www.pediy.com/kssd/pediy03/forum530.htm`](https://www.pediy.com/kssd/pediy03/forum530.htm)

## 软件简介
鹦鹉螺网络助手是一个功能强大，方便易用的专业网络工具集。

她基于 `Windows Socket2.0`，提供了在互联网和企业内部网络上广泛使用的多种工具，包括：`Ping`、`TraceRoute`(路由跟踪)、`Host lookup`(主机查询)、`Finger`(帐号查询)、同步本地主机时钟的 `Time`(网络时钟)、`WhoIs`(域名查询)、端口扫描、IP 地址扫描、`QoD` 服务、增强的快速拨号、支持多帐号的电邮检查。

*   **操作系统**: `Win95/98/me/NT/2000`
*   **下载地址**: `http://netkit.top263.net`
*   **破解工具**: 反汇编工具、编辑器
*   **级别**: 初级（大笨鸟）

本人初学,如有不正确之处,希望各位大侠给予指正.
本人信箱: `langlirong@163.net`

本人无意要破坏别人的劳动果实，此破解只作学习和交流用，不得用于其他用途。

## 破解过程

启动 `NetKit`，随便输入用户名和注册码确定，弹出错误提示框，请记住。

打开反汇编工具，装入 `NetKit`，找到错误提示处：

```assembly
* Possible Reference to String Resource ID=00016: "s?
                                |
:004168B4 6A10                   push 00000010

* Possible StringData Ref from Data Obj ->"鹦鹉螺网络助手"
                                |
:004168B6 68C8E14500             push 0045E1C8

* Possible StringData Ref from Data Obj ->"抱歉，无效注册码！"
```

向上找到 `00416856(c)` 立即跳到 `00416856`

```assembly
:00416840 57                     push edi
:00416841 E8E5150200             call 00437E2B
:00416846 8D4C2418               lea ecx, dword ptr [esp+18]
:0041684A C644242C01             mov [esp+2C], 01
:0041684F E8CC010000             call 00416A20
                                 ^^^^^^^^^^^^^
:00416854 85C0                   test eax, eax
:00416856 745C                   je 004168B4 //不对跳到 004168B4
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:00416858 8B3F                   mov edi, dword ptr [edi]
:0041685A E8E7D50200             call 00443E46
:0041685F 8B4004                 mov eax, dword ptr [eax+04]
:00416862 57                     push edi
```

将 `00416856` 处 `je` 改为 `jne`

此时注册显示成功.但退出重新进入,提示未注册.

还需找到检测点,请看 `00416856` 处上面的 `call`,还等什么?进入呀:

```assembly
:00416A86 52                     push edx
:00416A87 E844B60000             call 004220D0
:00416A8C 83C40C                 add esp, 0000000C
:00416A8F C644242000             mov [esp+20], 00
:00416A94 85C0                   test eax, eax
:00416A96 8D4C2428               lea ecx, dword ptr [esp+28]
:00416A9A 752D                   jne 00416AC9 //不对跳到 00416AC9
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
:00416A9C E815160200             call 004380B6
:00416AA1 8D4C242C               lea ecx, dword ptr [esp+2C]
:00416AA5 C7442420FFFFFFFF       mov [esp+20], FFFFFFFF
:00416AAD E804160200             call 004380B6
```

将 `00416A9A` 的 `752D` 改为 `9090`

现在打开 `NetKit`,输入用户名和注册码(随便啦)看看,是不是正版软件啦.

OK！整理一下，收工。

阿郎
`langlirong@163.net`
`http://alang.shangdu.net`
`12/01/2001` `4dd466d0e1e0cc123afbe504da2462ca`
