
# Paraben's Mah-jong v2.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1016.htm`](https://www.pediy.com/kssd/pediy03/forum1016.htm)

**目标**：`Paraben's Mah-jong v2.2`(2D, 3D)

**工具**：`softice`，`w32dsm89`，`hiew`

**作者**：`MAUIboy2001[BCG]`

做完功课后闲的无聊，找了个游戏玩玩。这个游戏是找两个相同的麻将，和街机中的一样，不过还有 3D 效果的。以前没钱玩，现在可以大战 200 回合了。嘻。

`http://www3.zdnet.com.cn/downloads2/parajong.exe`

打开 `Paraben's Mah-jong 2D`，出现对话框。点注册，下 `bpx hmemcpy`。

拦住，pressing `F12` 十下，出现错误。重新来过，`F12` 九下：

```
:**********      push      00002872

:**********      push      esi

:**********      call      edi

:**********      mov       ecx,0041ED70

:**********      call      004047A0

:**********      mov       al,byte ptr [004322F5]

:**********      push      00000040

:**********      cmp       al,01

:**********      push      0041ED70

:**********      jne       00401901  --跳则死

:**********      push      00440B94

:**********      jmp       00401906  --成功
```

只要把 `jne`-->`je` 就成功或 `cmp al,01`-->`cmp al,00`

不过看事物不能光看表面，我们去它的内核看看吧。

用 `w32dsm89` 反汇编，查找 `call edi`。

```
:004018DE      call      edi

 .

:004018E5      call      004047A0
```

`F8` 进入:

```
:004047A0      sub       esp,00000008

 .

 .

 .(拖动滚动条，往下查看)

 .

 .

:00404836      cmp       ecx,eax     --关键的 cmp

:00404838      jne       00404850

 .

 .

:00404841      mov       byte ptr [esi+00013585],01

 .

 .

:0040484F      ret

:00404850      mov       byte ptr [esi+00013585],00

 .

 .

:0040485       ret
```

在`:00404836`处，`? ecx` 既为注册码。

将`:00404850`处 `C6868535010000`-->`C6868535010001`.

搞定。`NAME:MAUIboy2001`    `code:404819`

如果你要爆破，请将 `Paraben's Mah-jong 2D` 和 `Paraben's Mah-jong 3D` 一起改。

因为注册成功后，将在其目录下生成 `Mahj.key` 的文件，打开文件时会检验，否则死机。

真是超 `easy`，只用了 15 分钟。欢迎指教，`qq:349471`。  `4dd466d0e1e0cc123afbe504da2462ca`
