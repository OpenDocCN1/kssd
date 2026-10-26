
`.text:005610CC lea esp, [esp+0]`

`.text:005610D0 loc_5610D0: ; CODE XREF: sub_560610+AC9 j`

`.text:005610D0 mov cx, [eax]`

`.text:005610D3 add eax, 2`

`.text:005610D6 cmp cx, bx`

`.text:005610D9 jnz short loc_5610D0`

`.text:005610DB sub eax, edx`

`.text:005610DD sar eax, 1`

`.text:005610DF push eax ; 长度`

`.text:005610E0 push ebx`

`.text:005610E1 push offset aBt_5qzone_net ; "bt.5qzone.net"`

`.text:005610E6 lea ecx, [esp+0C0h+var_34]`

`.text:005610ED call sub_420150 ; 判断函数`

`.text:005610F2 cmp eax, 0FFFFFFFFh`

`.text:005610F5 jz loc_56132A;改为jmp loc_56132A`

最后是禁止BitComet软件自动连接升级服务器进行自动更新检测：

在IDA中按Alt+T打开Text search，在string框中输入`ca3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3k6r3q4@1k6g2)9J5k6h3u0A6N6r3y4G2L8h3g2@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3W2@1j5$3!0E0k6i4c8Q4x3V1k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0k6Q4z5p5y4Q4z5o6W2a6d9#2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2!0n7x3q4!0n7x3g2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4W2)9^5z5g2!0n7c8g2!0q4y4g2)9^5z5q4!0n7x3q4!0q4y4g2!0n7b7#2)9&6y4g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4g2)9^5c8q4)9^5y4#2!0q4y4#2!0n7b7g2!0m8y4#2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4g2!0m8x3g2!0n7y4g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4g2)9&6c8q4)9^5x3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2!0m8x3#2)9^5x3q4!0q4y4W2)9&6c8W2!0m8y4g2!0q4y4g2!0m8c8q4)9&6y4#2!0q4y4#2!0m8b7#2!0m8y4W2!0q4y4q4!0n7z5q4!0n7x3W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4q4!0n7b7g2!0m8y4q4!0q4y4g2)9^5c8W2)9^5z5g2!0q4y4g2!0n7b7#2)9&6y4g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4g2)9^5c8W2)9&6x3g2!0q4y4#2)9^5c8g2!0n7x3q4!0q4y4g2)9^5c8W2!0m8b7g2!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4g2)9^5y4#2!0n7c8q4!0q4y4W2)9&6y4g2!0n7x3s2y4#2j5W2)9#2k6U0b7I4z5f1f1@1x3q4!0q4y4#2!0m8y4#2)9^5c8q4!0q4y4#2)9&6b7g2)9^5y4o6b7I4z5f1k6n7c8g2!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4W2)9&6y4W2!0n7z5g2!0q4y4g2!0n7b7#2)9&6y4g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4q4!0n7b7g2)9^5y4W2!0q4y4W2!0m8c8q4!0m8y4q4!0q4y4g2)9^5c8q4)9^5y4#2!0q4y4#2!0n7b7g2!0m8y4#2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4g2)9&6c8q4)9^5x3q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7b7W2!0m8x3#2!0q4y4#2!0m8x3q4)9^5x3g2!0q4y4g2!0m8y4W2)9^5x3W2!0q4y4q4!0n7z5q4)9^5b7W2!0q4c8W2!0n7b7#2)9&6b7b7`.`.

`.text:00419FBE push offset aHttpUpdate_bit`

```
// "0c0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2H3k6r3q4@1k6g2)9J5k6h3u0A6N6r3y4G2L8h3g2@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3W2@1j5$3!0E0k6i4c8Q4x3V1k6Q4x3@1k6$3k6b7`.`."...
```

`.text:00419FC3 push edx`

`.text:00419FC4 mov [esp+124h+var_84], ebp`

`.text:00419FCB mov [esp+124h+var_88], ebx`

`.text:00419FD2 mov word ptr [esp+124h+var_98], bx`

`.text:00419FDA call sub_4125F0`

向后看一下函数的代码就可以发现这个函数作用是生成访问判断是否更新网页的网址链接字符串，向上来到函数`sub_419E40`的开始处，可以找到如下代码：

`.text:00419E75 mov large fs:0, eax`

`.text:00419E7B mov eax, [esp+118h+arg_0]`

`.text:00419E82 xor ebx, ebx`

`.text:00419E84 cmp dword_7D581C, ebx`

`.text:00419E8A mov [esp+118h+var_E4], eax`

`.text:00419E8E jz loc_41A44F`

只要把`419E8E`处的`jz loc_41A44F`（`0F 84 BB 05 00 00`）改为`jmp loc_41A44F`（`90 E9 BB 05 00 00`）即可阻止生成访问链接，也就禁止了BitComet的自动更新检测。

到此，BitComet软件就差不多真的成为绿色软件了。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006年12月18日 23:26:37

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
