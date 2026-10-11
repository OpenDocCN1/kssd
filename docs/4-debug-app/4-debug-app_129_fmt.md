
# cff explorer 英文版字符完整显示的方法

> 原文：[`www.pediy.com/kssd/pediy12/140628.html`](https://www.pediy.com/kssd/pediy12/140628.html)

`Sam.com` 在《CFF Explorer 临时修改完整显示文字》一文中针对汉化版进行了修改，如果你用的是英文版可以这样修改：

```
0049E40D       0D 00080000    OR EAX,800     //改为 or eax,900

0049E412    .  50             PUSH EAX

 0049E413    .  8D45 14       LEA EAX,DWORD PTR SS:[EBP+14]

0049E416    .  50             PUSH EAX

0049E417    .  8B42 38       MOV EAX,DWORD PTR DS:[EDX+38]

 0049E41A    .  6A FF        PUSH -1

0049E41C    .  8BCB         MOV ECX,EBX

0049E41E    .  FFD0         CALL EAX

0049E420    .  8B4E 04       MOV ECX,DWORD PTR DS:[ESI+4]              ; |

0049E423    .  50             PUSH EAX                                  ; |Text

0049E424    .  51             PUSH ECX                                  ; |hDC

0049E425    .  FF15 54465000 CALL DWORD PTR DS:[<&USER32.DrawTextW>]   ; \DrawTextW
```

原理：`int DrawTextW(HDC hdc, LPCTSTR lpString, int nCount, LPRECT lpRect, UINT uFormat)`；

在 `uFormat` 处加上一个属性 `DT_NOCLIP = $100`，即 `or eax,900`，如果默认的字体对于指定的显示区域太大的话就 `NOCLIP`，效果就是可以完整显示。

![](img/0f5d1d93ed72a920590a55ca1e6ef6cd.jpg)
