
# IDA 字串显示中文

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-111.htm`](https://www.pediy.com/kssd/pediy09/pediy09-111.htm)

> `Ida pro`里的中文字串

```
// the following characters are allowed in ASCII strings, i.e.

// in order to find end of a string IDA looks for a character
// which doesn't belong to this array:
// (cp866 version)

AsciiStringChars =
        "\\r\\n\\a\\v\\b\\t\\x1B"
        " !\\"#$%&'()\*+,-./0123456789:;<=>?"
        "@ABCDEFGHIJKLMNOPQRSTUVWXYZ\[\\\\\]^\_"
        "\`abcdefghijklmnopqrstuvwxyz{|}~"
        "?"
        "ぅΗī氨渤吹斗腹夯冀究"
        "懒旅呐魄壬仕掏蜗醒矣哉肿刭谯茌捱"
        "噌忏溴骁栝觌祉铒";

// (full version)
//AsciiStringChars =
//      "\\r\\n\\a\\v\\b\\t\\x1B"
//      " !\\"#$%&'()\*+,-./0123456789:;<=>?"
//    "@ABCDEFGHIJKLMNOPQRSTUVWXYZ\[\\\\\]^\_"
//      "\`abcdefghijklmnopqrstuvwxyz{|}~"
//      "?"
//      "ぅΗī氨渤吹斗腹夯冀究"
//      "懒旅呐魄壬仕掏蜗醒矣哉肿刭谯茌捱"
//      "噌忏溴骁栝觌祉铒瘃蝮趱鲼?";

// the following characters are allowed in user-defined names:

.........................cut..............................
```

把`// (cp866 version)`到`// (full version)`之间的行用`'//'`注释掉

把`// (full version)``// the following characters are allowed ..`之间的行前面的`'//'`去掉，保存，重新运行`IdaPro`，呵呵，汉字真好...

`4dd466d0e1e0cc123afbe504da2462ca`
