
# 超级广告杀手 4.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6069.htm`](https://www.pediy.com/kssd/pediy06/pediy6069.htm)

## 软件信息

*   **软件名称**：超级广告杀手 4.0
*   **下载页面**：共享软件注册中心
*   **软件大小**：449k
*   **应用平台**：`WIN9X`/`WINNT`/`WIN2K`/`WINXP`

## 软件简介

超级广告杀手的主要特点：

1.  **智能化程度高**
    它甚至可以关闭那些当你离开某个站点的时候产生的广告窗口，超级广告杀手只会关闭弹出式的广告窗口，不会影响你打开正常的浏览窗口。
2.  **及时通知**
    当超级广告杀手拦截了一个广告窗口的时候，会通过闪动系统条上的图标和发出声音来通知你。
3.  **灵活的设置**
    你可以设置某个特定站点允许通过弹出式窗口出现，你也可以设置热键来暂时性的禁用超级广告杀手。
4.  **十分容易使用**
    安装之后你可以不做任何设置就可以使用，程序而且会跟随 `IE` 浏览器自动启动。
5.  **占用系统资源**
    占用极少的系统资源，丝毫不会影响你的系统性能。
6.  **隐私保护**
    可以帮助你清除敏感的上网纪录和浏览器缓冲区文件以提高系统速度。

## 软件限制

不记得了，好像是 30 天的试用期！

## 作者声明

本人发表这篇文章只是为了学习！请不用于商业用途或是将本文方法制作的注册机任意传播，读者看了文章后所做的事情与我无关，我也不会负责，请读者看了文章后三思而后行！最后希望大家在经济基础好的时候，支持共享软件！

## 破解工具

`TRW2K` `W32Dasm` `peid`

## 破解过程

1.  用 `peid` 侦察出主程序 `popkill.exe` 没有加壳！（善良的作者！）
2.  用 `W32Dasm` 反汇编 `popkill.exe`。
3.  根据参考字串找到关键！
4.  运行主程序 `popkill.exe` -> 选择注册 -> 填入假注册码 `48484848`。
5.  启动 `TRW2K` -> 下断点 `bpx 412adf` -> `f5` 返回 -> 按"注册" -> 拦下！！！

### 反汇编代码分析

```text
* Reference To: MFC42.Ordinal:18BE, Ord:18BEh
  |
:00412ADF E800280000          Call 004152E4          //这里是取得机器码和假注册码的 16 进制
  |
:00412AE4 8B96DC020000       mov edx, dword ptr [esi+000002DC]
  |
  //这里是假注册码的 16 进制值!
:00412AEA 8B8ED8020000       mov ecx, dword ptr [esi+000002D8]
  |
  //这里是真注册码的 16 进制值!
:00412AF0 BB88AA9919         mov ebx, 1999AA88
  |
  //这里应该就是密匙!!!
:00412AF5 33D3               xor edx, ebx
  |
  //假注册码 16 进制和 1999AA88 或运算!
:00412AF7 8BC2               mov eax, edx
:00412AF9 8996DC020000       mov dword ptr [esi+000002DC], edx
:00412AFF 3BC1               cmp eax, ecx
:00412B01 0F85B2000000       jne 00412BB9
  |
  //跳到出错的地方!
:00412B07 8D44240C           lea eax, dword ptr [esp+0C]
:00412B0B C744240C00000000   mov [esp+0C], 00000000
:00412B13 50                 push eax
:00412B14 683F000F00         push 000F003F
:00412B19 6A00               push 00000000

* Possible StringData Ref from Data Obj ->"SOFTWARE\Saga\Super Popup Blocker"
  |
:00412B1B 6890F24100         push 0041F290
:00412B20 6802000080         push 80000002

* Reference To: ADVAPI32.RegOpenKeyExA, Ord:01E2h
  |
:00412B25 FF1560804100       Call dword ptr [00418060]
  |
  //检查是否有键值[SOFTWARE\Saga\Super Popup Blocker]
:00412B2B 85C0               test eax, eax
:00412B2D 757F               jne 00412BAE
:00412B2F 8B86DC020000       mov eax, dword ptr [esi+000002DC]
:00412B35 8B7C240C           mov edi, dword ptr [esp+0C]
:00412B39 8D4C240C           lea ecx, dword ptr [esp+0C]
:00412B3D 6A04               push 00000004
:00412B3F 51                 push ecx
:00412B40 6A04               push 00000004
:00412B42 6A00               push 00000000
:00412B44 33C3               xor eax, ebx

* Possible StringData Ref from Data Obj ->"ZhuceNo"
  |
:00412B46 6860F44100         push 0041F460
:00412B4B 57                 push edi
:00412B4C 897C2428           mov dword ptr [esp+28], edi
:00412B50 8986DC020000       mov dword ptr [esi+000002DC], eax
:00412B56 89442424           mov dword ptr [esp+24], eax

* Reference To: ADVAPI32.RegSetvalueExA, Ord:01F9h
  |
:00412B5A FF155C804100       Call dword ptr [0041805C]

* Reference To: ADVAPI32.RegCloseKey, Ord:01C9h
  |
:00412B60 8B1D68804100       mov ebx, dword ptr [00418068]
:00412B66 85FF               test edi, edi
:00412B68 7409               je 00412B73
:00412B6A 57                 push edi
:00412B6B FFD3               call ebx
:00412B6D 33FF               xor edi, edi
:00412B6F 897C2410           mov dword ptr [esp+10], edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412B68(C)
|
:00412B73 6A00               push 00000000
:00412B75 8D8EA0000000       lea ecx, dword ptr [esi+000000A0]

* Reference To: MFC42.Ordinal:0A52, Ord:0A52h
  |
:00412B7B E846270000         Call 004152C6
:00412B80 6A00               push 00000000
:00412B82 6A00               push 00000000

* Possible StringData Ref from Data Obj ->"恭喜！注册成功"    //HOHO
  |
:00412B84 6870FE4100         push 0041FE70
```

## 总结

注册信息保存在注册表的 `[HKEY_LOCAL_MACHINE\SOFTWARE\Saga\Super Popup Blocker]` 的 `ZhuceNo` 里。
保存格式是 16 进制！

**算法**：

假设真注册码的 16 进制值为 `X`，机器码的 16 进制值为 `Y`

`Y = X XOR 1999AA88`

这个程序的作者是我遇见的第一个非常善良的人咯...呵呵！

哪位大哥哥有时间帮忙写个注册机吧，千分感激！

最后在这里真心感谢你花了那么多时间看这篇文章！谢谢了...

`2003.05.24` 凌晨于清远

`4dd466d0e1e0cc123afbe504da2462ca`
