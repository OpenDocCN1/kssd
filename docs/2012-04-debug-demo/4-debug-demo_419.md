
# XP 记事本，状态栏无法选中的修改

> 原文：[`www.pediy.com/kssd/pediy11/120070.html`](https://www.pediy.com/kssd/pediy11/120070.html)

`img/110d920e4e703968a2a60c986d363163.jpg`发现 XP 的自动换行功能和状态栏功能就像鱼和熊掌一样不可兼得，于是用初学的知识试着改了一下，改了两晚上，也未成功。今天刚看了 `EnableMenuItem` 的结构定义。于是试着改了一下，居然成功了。嘿嘿。

关于 XP 记事本，状态栏的无法选中的修改，论坛好像没有，论坛只有 Win7 的记事本。希望不算重复。

可以在 `EnableMenuItem` 下断修改。

`EnableMenuItem` 函数的结构如下:

```
BOOL EnableMenuItem ( 

        HMENU hMenu            // handle to menu resource 

        UINT uIDEnableItem     // menu item to enable, disable or gray 

        UINT uEnable           // menu item flags 

        );
```

在 ASM 中调用的格式是:

```
PUSH    uEnable 

PUSH    uIDEnableItem 

PUSH    hMenu 

CALL    [KERNEL32!EnableMenuItem]
```

对于我们来说,重要的是找到那个激活菜单项的标志

用 `ResHacker` 得知状态栏的 ID 为 `0x1b`

去掉无关的 `EnableMenuItem` 调用后。只剩下两个 `EnableMenuItem`

然后，运行全速 OD。然后，就要确定是哪个 `EnableMenuItem` 了。当我们选中“自动换行”功能后，OD 会停在

```
01002F1E  |> /50             push    eax                       ; |hMenu 
01002F1F  |. |FF15 5C120001 call    dword ptr [<&USER32.EnableMenuIt>; \EnableMenuItem 
01002F25  |. |E9 3D040000  jmp     notepadx.01003367 
```

跳转来自 `010031DA`

来到

```
010031CC  |.  FF15 48120001 call    dword ptr [<&USER32.CheckMenuIte>; \CheckMenuItem 
010031D2  |.  6A 01         push    1 
010031D4  |.  90            push ebx 
010031D5  |.  6A 03         push    3                           ; /Pos = 3 
010031D7  |.  57            push    edi                         ; |hMenu 
010031D8  |.  FFD6          call    esi                         ; \GetSubMenu 
010031DA  |.^ E9 3FFDFFFF  jmp     notepadx.01002F1E 
```

根据 `enablemenuitem` 的结构可知。只要 Nop 掉 `push ebx` 即可以让 `EnableMenuItem` 失效。

即 `010031d4` 改为 `nop` 再另存修改。

试验即成功。

小心打击我。我很脆弱。
