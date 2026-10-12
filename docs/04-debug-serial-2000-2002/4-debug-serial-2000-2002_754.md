
# 远程控制程式

> 原文：`www.pediy.com/kssd/pediy03/forum1009.htm`

远程控制程式，`http://hbsafe.yeah.net` 可控制鼠标、键盘、屏幕、硬盘、可远程提取密码、。。。

> `dede` 就是好用，反编译一下，什么都看见了。

> `call 00426E28`

> `call 00407DD4`

> 这两个 `call` 可以不用理会，why?，看看下面，已经写得很清楚了^\_^

> `00462FC1  64FF30      push    dword ptr fs:[eax]`
> `00462FC4  648920      mov     fs:[eax], esp`
> `00462FC7  8D55F4      lea     edx, [ebp-$0C]`

> * Reference to control `Edit1` : `TEdit`

> `00462FCA  8B83EC020000   mov    eax, [ebx+$02EC]`

> * Reference to: `controls.TControl.GetText(TControl):System.String;`

> `00462FD0  E8533EFCFF      call    00426E28`
> `00462FD5  8B45F4      mov    eax, [ebp-$0C]`

> * Reference to: `sysutils.StrToInt(System.AnsiString):System.Integer;`

> `00462FD8  E8F74DFAFF      call    00407DD4`
> `00462FDD  8BF0            mov    esi, eax      把第 1 个框内的数值放入 esi`
> `00462FDF  8D55F0      lea     edx, [ebp-$10]`

> * Reference to control `Edit2` : `TEdit`

> `00462FE2  8B83F0020000   mov    eax, [ebx+$02F0]`

> * Reference to: `controls.TControl.GetText(TControl):System.String;`

> `00462FE8  E83B3EFCFF      call    00426E28`
> `00462FED  8B45F0      mov    eax, [ebp-$10]`

> * Reference to: `sysutils.StrToInt(System.AnsiString):System.Integer;`

> `00462FF0  E8DF4DFAFF      call    00407DD4`
> `00462FF5  03F0            add    esi, eax      把第 2 个框内的数值加入 esi`
> `00462FF7  8D55EC      lea     edx, [ebp-$14]`

> * Reference to control `Edit3` : `TEdit`

> `00462FFA  8B83F4020000   mov    eax, [ebx+$02F4]`

> * Reference to: `controls.TControl.GetText(TControl):System.String;`

> `00463000  E8233EFCFF      call    00426E28`
> `00463005  8B45EC      mov    eax, [ebp-$14]`

> * Reference to: `sysutils.StrToInt(System.AnsiString):System.Integer;`

> `00463008  E8C74DFAFF      call    00407DD4`
> `0046300D  8BF8            mov    edi, eax      把第 3 个框内的数值放入 edi`
> `0046300F  8D55E8      lea     edx, [ebp-$18]`

> * Reference to control `Edit4` : `TEdit`

> `00463012  8B8300030000   mov    eax, [ebx+$0300]`

> * Reference to: `controls.TControl.GetText(TControl):System.String;`

> `00463018  E80B3EFCFF      call    00426E28`
> `0046301D  8B45E8      mov    eax, [ebp-$18]`

> * Reference to: `sysutils.StrToInt(System.AnsiString):System.Integer;`

> `00463020  E8AF4DFAFF      call    00407DD4`
> `00463025  0FAFF8          imul    edi, eax      第 3 个框的数值和第 4 个框的数值相乘`
> `00463028  03F7            add    esi, edi      加上第 1、2 框内数值之和`
> `0046302A  81FED0070000   cmp    esi, $000007D0  比较是否等于十进制的 2000`
> `00463030  0F85C7000000   jnz    004630FD      等于则通过`

> 最简单的注册码就是 `2000 0 0 0`

> 软件注册后会在 `windows\system` 下生成一个 6 个字节大小的文件 `sysdat.dat`
> 内容为

```
0x02 0x6Fh 0x6B 0x00 0x00 0x00  4dd466d0e1e0cc123afbe504da2462ca
```
