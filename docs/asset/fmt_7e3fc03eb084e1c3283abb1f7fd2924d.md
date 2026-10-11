
# ASP2DLL V3.3.14

> 原文：[`www.pediy.com/kssd/pediy03/forum836.htm`](https://www.pediy.com/kssd/pediy03/forum836.htm)

## 软件信息

- **软件名称**：`ASP2DLL V3.3.14`
- **软件设计**：XDE, Inc.
- **软件主页**：`http://xde.net/products/download2.asp`
- **联系邮件**：无
- **更新时间**：2001-04-13
- **授权方式**：共享软件
- **操作系统**：`Windows 95/98/NT`
- **试用限制**：30 天
- **购买金额**：＄250.00
- **软件大小**：`3238KB`
- **星级评定**：★★★★★
- **软件简介**：`ASP2DLL` 能将 `ASP` 程序转化成 `VB 5.0` 或 `VB 6.0` 原代码，从而使您的 Web 应用程序可以被编辑成 ActiveX DLL。

### 下载链接

- **本地下载**：`http://www.downme.com/download/8838Setup_ASP2DLL.exe`
- **外地下载**：`http://xde.net/distrib/Setup_ASP2DLL.exe`

---

昨天答应了 `FCG` 合伙人，`Cyrix686`（他是建网站主力，哈哈所以网页设计他一个包办）。

他要我破解这个工具，刚好今天有空就看一看，想不到这么变态，总算暴破解完了！

小球试一试，追出密码与其它计算过程让别人写出注册机行吗？

**作者**：`PeterChen`

这个工具是用 `vb5` 编写的，密码太怪了，想追密码也不好看出来！

最后想一想就算追出密码也不好输入，所以就决定暴力破解它算了！

多么变态的注册码？

以下是试用注册码：

```
-----------------------------------------------------------------------
* Possible StringData Ref from Code Obj ->"88x3p5BeabcdfghijklmnoqrstuvwyzACDEFGHIJKLMNOP"
                                      ->"QRSTUVWXYZ 1246790-.#/\!@$<>&*()[]{}';:,?=+~`^"
                                      ->"|%\_" ------------->注册码的样本！
                                      |
:00404E9B BA58314000                 mov edx, 00403158
d|5S53|?mJ\_:3|oi(p|~a+e'?ljRU//--------->看到没有试用注册码就是这样，要破解注册码
                                      就麻烦了？
windows/xde.lic
-----------------------------------------------------------------------
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E177A(U)
|
:004E1786 0FBF8DACFEFFFF             movsx ecx, word ptr [ebp+FFFFFEAC]
:004E178D 85C9                       test ecx, ecx
:004E178F 0F8511020000               jne 004E19A6//----------->je  (000e178fh)
:004E1795 C745FC0A000000             mov [ebp-04], 0000000A
:004E179C C78538FFFFFF04000280       mov dword ptr [ebp+FFFFFF38], 80020004
:004E17A6 C78530FFFFFF0A000000       mov dword ptr [ebp+FFFFFF30], 0000000A
:004E17B0 C78548FFFFFF04000280       mov dword ptr [ebp+FFFFFF48], 80020004
:004E17BA C78540FFFFFF0A000000       mov dword ptr [ebp+FFFFFF40], 0000000A
:004E17C4 C78558FFFFFF04000280       mov dword ptr [ebp+FFFFFF58], 80020004
:004E17CE C78550FFFFFF0A000000       mov dword ptr [ebp+FFFFFF50], 0000000A

* Possible StringData Ref from Code Obj ->"PPlease save your temporary registration "
                                      ->"code, then restart ASP2DLL"
                                      |
:004E17D8 C785F8FEFFFF6C964200       mov dword ptr [ebp+FFFFFEF8], 0042966C
:004E17E2 C785F0FEFFFF08000000       mov dword ptr [ebp+FFFFFEF0], 00000008
:004E17EC 8D95F0FEFFFF               lea edx, dword ptr [ebp+FFFFFEF0]
:004E17F2 8D8D60FFFFFF               lea ecx, dword ptr [ebp+FFFFFF60]

--------------------------------------------------------------------------

* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
                                      |
:004E17F8 FF154C134000               Call dword ptr [0040134C]
:00460668 6683BDD0FDFFFF00           cmp word ptr [ebp+FFFFFDD0], 0000
:00460670 0F85A5000000               jne 0046071B----->je (00060670h)
:00460676 B90A000000                 mov ecx, 0000000A
:0046067B B804000280                 mov eax, 80020004
:00460680 898D44FFFFFF               mov dword ptr [ebp+FFFFFF44], ecx
:00460686 898D54FFFFFF               mov dword ptr [ebp+FFFFFF54], ecx
:0046068C 898D64FFFFFF               mov dword ptr [ebp+FFFFFF64], ecx
:00460692 8D9534FEFFFF               lea edx, dword ptr [ebp+FFFFFE34]
:00460698 8D8D74FFFFFF               lea ecx, dword ptr [ebp+FFFFFF74]
:0046069E 89854CFFFFFF               mov dword ptr [ebp+FFFFFF4C], eax
:004606A4 89855CFFFFFF               mov dword ptr [ebp+FFFFFF5C], eax
:004606AA 89856CFFFFFF               mov dword ptr [ebp+FFFFFF6C], eax

* Possible StringData Ref from Code Obj ->"RRegistration Error"
                                      |
:004606B0 C7853CFEFFFF04BC4100       mov dword ptr [ebp+FFFFFE3C], 0041BC04
:004606BA C78534FEFFFF08000000       mov dword ptr [ebp+FFFFFE34], 00000008

* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
                                      |
:004606C4 FF154C134000               Call dword ptr [0040134C]
:004606CA 8D8D44FFFFFF               lea ecx, dword ptr [ebp+FFFFFF44]
:004606D0 51                         push ecx
:004606D1 8D9554FFFFFF               lea edx, dword ptr [ebp+FFFFFF54]
:004606D7 52                         push edx
:004606D8 8D8564FFFFFF               lea eax, dword ptr [ebp+FFFFFF64]
:004606DE 50                         push eax
:004606DF 6A00                       push 00000000
:004606E1 8D8D74FFFFFF               lea ecx, dword ptr [ebp+FFFFFF74]
:004606E7 51                         push ecx

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
                                      |
:004606E8 FF1504114000               Call dword ptr [00401104]

--------------------------------------------------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00460A56(U)
|
:00460AE8 66833D54E04E0000           cmp word ptr [004EE054], 0000
:00460AF0 0F85C4000000               jne 00460BBA------>je (00060af0h)
:00460AF6 B904000280                 mov ecx, 80020004
:00460AFB B80A000000                 mov eax, 0000000A
:00460B00 898D4CFFFFFF               mov dword ptr [ebp+FFFFFF4C], ecx
:00460B06 898D5CFFFFFF               mov dword ptr [ebp+FFFFFF5C], ecx
:00460B0C 898D6CFFFFFF               mov dword ptr [ebp+FFFFFF6C], ecx
:00460B12 8D9534FEFFFF               lea edx, dword ptr [ebp+FFFFFE34]
:00460B18 8D8D74FFFFFF               lea ecx, dword ptr [ebp+FFFFFF74]
:00460B1E 898544FFFFFF               mov dword ptr [ebp+FFFFFF44], eax
:00460B24 898554FFFFFF               mov dword ptr [ebp+FFFFFF54], eax
:00460B2A 898564FFFFFF               mov dword ptr [ebp+FFFFFF64], eax

* Possible StringData Ref from Code Obj ->"YYou have run out of class conversion "
                                      ->"credits"
                                      |
:00460B30 C7853CFEFFFF7CBC4100       mov dword ptr [ebp+FFFFFE3C], 0041BC7C
:00460B3A C78534FEFFFF08000000       mov dword ptr [ebp+FFFFFE34], 00000008

* Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
                                      |
:00460B44 FF154C134000               Call dword ptr [0040134C]

---------------------------------------------------------------------------------
```

## 总结

搜索：`0F8511020000` 修改为 `84`（偏移 `000e178fh`）
搜索：`0F85A5000000` 修改为 `84`（偏移 `00060670h`）
搜索：`0F85C4000000` 修改为 `84`（偏移 `00060af0h`）

还要找查出 `xde.lic`，删除它！

它在 `win98` 中是 `windows` 目录，如果是 `win2000`，就是 `winnt` 目录中！

最好有 `windows` 的查找功能，找到删除就行了！

`xde.lic` 是放密码的文件！

## 附件与下载

如果只想破解它的密码，我把要注册两个主要文件上载了！

- `ASP2DLL.exe`：主程序，用来测试是否注册？
- `XDERegister.exe`：密码生成器，就是在这里破解出密码了，本身有两个条件限制试用注册码。

下载链接：`http://freecracker.myetang.com/`

站点镜像：`longcity.net`

还有其它破解文件与教程！

去下载吧？看有哪位大侠能破解它的密码，或者写注册机？

