
# SWF Browser v2.93

> 原文：[`www.pediy.com/kssd/pediy03/forum936.htm`](https://www.pediy.com/kssd/pediy03/forum936.htm)

**目标**：`SWF Browser v2.93`

**工具**：`softice`, `w32dsm89`

**作者**：`MAUIboy2001[BCG]`

`SWF Browser` 是 Flash 设计不可缺少的工具，很好的。华军有下载。此软件注册后可把 flash 文件中的元件拆解出来。

用 `w32dsm89` 反汇编，查找字符串“`the serial number is invalid`”。往上看，发现“`thank you for registering..`”。

再看，发现是 `:004A34EC` 跳的。

那么 `:004A34E5` 的 `call` 是关键。

用 `softice` 载入，下 `bpx 4A34E5`。拦到，`F8` 进入：

```
:004A32F8      PUSH  ebp

.
.(F10)
.

:004A33D6      MOV   eax,dword ptr [ebp-08]
:004A33D9      call  00403E3C
:004A22DE      jne   004A33E4      --跳则死
:004A33E0      mov   bl,01
:004A33E2      jmp   004a33E6      --成功
```

千万不能将 `je` --> `jne`，要不会弹出许多错误的烂对话框。还把我的斗地主的注册文件删了，害我又重装了一遍。惨！

`F8` 进入 `004A33D9`：

```
:00403E3C      push  ebx
:00403E3D      push  esi
:00403E3E      push  edi
:00403E3F      mov   esi,eax       --d eax(my code)
:00403E41      mov   edi,edx       --d edx(Serial number)
:00403E43      cmp   eax,edx
:00403E45      je    00403EDA      --je-->jne 不能破。
```

搞定。
`name`: `MAUIboy2001`
`serial number`: `45C25B6E`

如有人暴破请通知一声，`mauiboy2001@china.com` 请不要炸喔。
`4dd466d0e1e0cc123afbe504da2462ca`
