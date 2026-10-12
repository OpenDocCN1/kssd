
# Help & Manual V3.0.4.619

> 原文：[`www.pediy.com/kssd/pediy05/pediy50154.htm`](https://www.pediy.com/kssd/pediy05/pediy50154.htm)

**软件名称**：Help & Manual V3.0.4.619

**软件简介**：YSIWYG(所视即所得)的 Help 文件制作工具，支持打印功能。还可以用来制作非常专业的使用手册、HTML 页面及 RTF 格式的文件。 这个软件可以让你非常轻松地制作 WIN3X 和 WIN95 的帮助文件，并且可以支持 HTML 格式的帮助文件。

**软件主页**：`www.helpandmanual.com`

**下载地址**：`http://newhua.ruyi.com/down/hmsetup.exe`

## 破解过程

1. 用 `PEid` 检查:无壳，Borland Delphi
2. 使用 `kWdsm` 载入分析
3. 使用 `Keymake` 1.73 制作文件补丁

```
////////////////////////////////////////////////////////////////////////////////////////
//软件试用期 14 天，超过时间，要求网上注册，并退出。烦人的窗口，干掉它！

////////////////////////////////////////////////////////////////////////////////////////

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:006A4DDE(C)
|
:006A4E05 A178F46A00                mov eax, dword ptr [006AF478]
:006A4E0A 8B00                      mov eax, dword ptr [eax]
:006A4E0C 8B80D4020000             mov eax, dword ptr [eax+000002D4]
:006A4E12 B201                      mov dl, 01
:006A4E14 E8D326D9FF               call 004374EC
:006A4E19 A178F46A00                mov eax, dword ptr [006AF478]
:006A4E1E 8B00                      mov eax, dword ptr [eax]
:006A4E20 8B10                      mov edx, dword ptr [eax]
:006A4E22 FF92D8000000             call dword ptr [edx+000000D8]  //ABOUT 窗口=CALL  00452C84

        //烦人，干掉它！可以在这里改为 6 个 NOP，但进入后 ABOUT 仍然存在，干脆修改 CALL  00452C84
        //把这个调用的第一句直接改为 RET 得了，55->C3,哈哈哈，好主意!

:006A4E28 A1FCF76A00                mov eax, dword ptr [006AF7FC]
:006A4E2D 8B00                      mov eax, dword ptr [eax]
:006A4E2F 83786C00                 cmp dword ptr [eax+6C], 00000000//这里关键，时间限制
:006A4E33 7D05                     jge 006A4E3A                      //就改这里！

        //EB05                       jmp 006A4E3A
:006A4E35 E8C6EDD5FF               call 00403C00                      //试用期满退出 :)

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:006A4DE6(C), :006A4E03(U), :006A4E33(C)
|
:006A4E3A 8B03                      mov eax, dword ptr [ebx]
:006A4E3C E87F15DBFF               call 004563C0
:006A4E41 8B03                      mov eax, dword ptr [ebx]
------------------------------------------------------------------------

* Reference To: user32.PostMessageA, Ord:0000h

                                       |
:00452C7C E85F4EFBFF                 Call 00407AE0
:00452C81 5B                         pop ebx
:00452C82 C3                         ret
:00452C83 90                         nop
:00452C84 55                         push ebp                       //就改这里，直接返回
        //C3                         ret
:00452C85 8BEC                       mov ebp, esp
:00452C87 83C4E0                     add esp, FFFFFFE0
:00452C8A 53                         push ebx
:00452C8B 56                         push esi
:00452C8C 33D2                       xor edx, edx
////////////////////////////////////////////////////////////////////////////////////////
//哈哈哈，只需修改 2 个字节就搞定

////////////////////////////////////////////////////////////////////////////////////////
收工。
```

`lajiaolz`
2002/06/02 `4dd466d0e1e0cc123afbe504da2462ca`
