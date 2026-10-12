
# 学习 Nolan Blender“A Detailed Look at Crypt Filters”

> 原文：[`www.pediy.com/kssd/pediy12/134717.html`](https://www.pediy.com/kssd/pediy12/134717.html)

呵呵，学习 Nolan Blender 关于 filter 的文章“A Detailed Look at Crypt Filters”（`http://www.woodmann.com/crackz/Tutorials/Nbufilt.htm`），从开始不太懂，到最后做出 license，也花了不少时间，这里只是与有兴趣的朋友分享下，高手请飘过：）

另外谢谢 NoFlexlm 兄的指点。

1. 所用 SDK 为 `6.1g`

2. 产生的 `blenderd` 的函数地址与 Nolan Blender 文章略有不同，不过不影响分析

```asm
.text:0042BC6C             mov     eax, off_44B050

.text:0042BC71             push    eax           ; Source

.text:0042BC72             push    4Ah           ; int

.text:0042BC74             push    dword_45D3C8    ; int

.text:0042BC7A             call    sub_40495F

.data:0044B050 ; void *off_44B050

.data:0044B050 off_44B050     dd offset sub_4019A2    

.data:0044B050
```

`sub_4019A2` 即为 filter 函数

3. 参考 phydora 的文章“Dump The filter of flexlm”，用 Hex-Rays 将 `sub_4019A2` 转成 c code，分析起来会容易不少，其余的参照 Nolan Blender 的文章就可以了。

`http://bbs.pediy.com/showthread.php?t=68402`

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `filter` 相关文件.zip |
