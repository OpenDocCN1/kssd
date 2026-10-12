
# G*J 汉化初步尝试

> 原文：`www.pediy.com/kssd/pediy12/136136.html`

前些时下了日文游戏看不懂，翻看文件目录时发现，居然是明文的，就起了汉化的心思。本人不懂日文，幸好有前人的努力做成的翻译软件金山快译，在此真要感谢一下该软件制作人员，让我们这种外文盲有了个汉化的基础。虽然翻译效果不好，总比没有强吧。本以为把明文翻译好之后，就可以使用的，却出现的都是乱码。使用 `ida`、`ollyice` 等软件经过几个星期的无用功之后，终于在前辈高手的一个技巧的使用之下，发现了处理乱码函数的大概位置。这个技巧就是 `nop` 掉 `call`。各位高手不要笑，作为菜鸟的我，发现这个技巧之后屡试屡应，非常有用，真的是兴奋了好几天。作为花费了几个星期的日日夜夜的辛苦，我决定把这个过程记录下来，作为逆向工程之路的回忆。

在使用 `nop` 大法之后，发现显示字串的调用位置。高手见笑，可能一见到 `getkeyboardstate` 就知道了，不过我是用 `nop` 才知道的。

代码:

```
.text:004056D1 mov esi, offset KeyState
.text:004056D6 mov edi, offset unk_495E10
.text:004056DB push offset KeyState ; lpKeyState
.text:004056E0 rep movsd
.text:004056E2 call ds:GetKeyboardState
.text:004056E8
.text:004056E8 loc_4056E8: ; CODE XREF: wWinMain(x,x,x,x)+4F0j
.text:004056E8 ; wWinMain(x,x,x,x)+4FAj
.text:004056E8 call key_switch <----进入这里
.text:004056ED call sub_406690
.text:004056F2 test eax, eax
.text:004056F4 jz short loc_405708
```

进入之后发现一堆 `switch`，只能逐步跟踪，然后一个个 `nop` 掉 `call`，经过几天奋战，找到字符处理函数。

进入 `call` 之后，发现 `createfont`，

代码:

```
01668 push esi
.text:00401669 push edi
.text:0040166A push offset pszFaceName ; "Simsun"
.text:0040166F push 31h ; iPitchAndFamily
.text:00401671 push 2 ; iQuality
.text:00401673 xor ebp, ebp
.text:00401675 push ebp ; iClipPrecision
.text:00401676 push 7 ; iOutPrecision
.text:00401678 push 86h ; iCharSet
.text:0040167D push ebp ; bStrikeOut
.text:0040167E push ebp ; bUnderline
.text:0040167F push ebp ; bItalic
.text:00401680 push 0C8h ; cWeight
.text:00401685 push ebp ; cOrientation
.text:00401686 push ebp ; cEscapement
.text:00401687 push ebp ; cWidth
.text:00401688 push 1Ah ; cHeight
.text:0040168A ; 68: v6 = a1;
.text:0040168A mov esi, eax
.text:0040168C call ds:CreateFontA
```

更改 `charset` 为 `86`，就是 `gbk`，`facename` 为 `"simsun"`，接着下面。

代码:

```
text:004016EB cmp al, 0A0h
.text:004016ED jbe short loc_4016F9
.text:004016EF
.text:004016EF loc_4016EF: ; CODE XREF: set_font2dispaly+89j
.text:004016EF mov dl, al ; v9>0x3c
.text:004016F1 add dl, 20h
.text:004016F4 cmp dl, 1Ch
.text:004016F7 ja short loc_401715
.text:004016F9 ; 93: v11 = ((v4 << 8) ^ *(a4 + 1));
.text:004016F9
.text:004016F9 loc_4016F9: ; CODE XREF: set_font2dispaly+8Dj
.text:004016F9 movzx edi, byte ptr [edi+1]
.text:004016FD movsx eax, al
.text:00401700 shl eax, 8
.text:00401703 xor edi, eax
.text:00401705 and edi, 0FFFFh
.text:0040170B ; 94: v62 = 2;
.text:0040170B mov [esp+0A8h+front_back_byte], 2
.text:00401713 jmp short loc_40171C
.text:00401715 ; ---------------------------------------------------------------------------
.text:00401715 ; 88: v11 = v4;
.text:00401715
.text:00401715 loc_401715: ; CODE XREF: set_font2dispaly+97j
.text:00401715 movzx edi, al
.text:00401718 ; 89: v62 = 1;
.text:00401718 mov [esp+0A8h+front_back_byte], ecx
.text:0040171C ; 96: GetTextMetricsA(v5, &tm);
.text:0040171C
.text:0040171C loc_40171C: ; CODE XREF: set_font2dispaly+B3j
.text:0040171C lea ecx, [esp+0A8h+tm]
.text:00401720 push ecx ; lptm
.text:00401721 push ebx ; hdc
.text:00401722 call ds:GetTextMetricsA
```

这里处理编码范围，前面还有个 `cmp al, 81h`。所有字符不是在 `81-a0`、`fe-1c` 范围的算做单字节。`gbk` 的范围是 `81-fe`，所有将 `cmp al, a0h` 改成 `cmp al, feh`，再运行，成了，中文都显示出来了。虽然还有很多问题没处理，不过我已经满足了，辛苦数个星期总算没白费。在此谢谢 `ida`，给了这么好的可视环境。感谢看雪论坛，然我们懂得这么多技巧。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `game.rar` |
| :--- | :--- |
