
然后进行提交，最后就可以得到flag了。
最终的结果为：
![](https://bbs.kanxue.com/upload/attach/202208/740345_22UU7JNEPN4E5D6.jpg)

### MISC

虽然这道题目是做出来了，但是其中还是存在一些问题没有搞清楚。

1. 执行`login()`方法的时候，同样需要绕过`__wakeup()`方法，那么就需要利用`__wakeup()`函数的漏洞了，但是在最后执行的payload中并没有修改对象属性的个数，我看到网上有人猜测可能是`Ä`这个字符影响了`__wakeup()`函数的执行。
2. `O:6:"HITCON":3:{s:14:"%00HITCON%00method";s:5:"login";s:12:"%00HITCON%00args";a:2:{i:0;s:7:"orÃnge";i:1;s:13:"babytrick1234";}s:12:"%00HITCON%00conn";O:9:"Exception":2:{s:7:"*file";R:4;};}}`这个payload是[0xecute](https://bbs.kanxue.com/elink@b87K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8^5k6h3y4#2N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8U0t1H3x3e0k6Q4x3V1j5I4x3q4)9J5c8U0p5H3i4K6u0r3j5X3q4T1P5g2)9J5k6s2c8J5K9h3y4C8i4K6u0r3)的答案。但是这个答案，我也没有看懂。不知道最后`O:9:"Exception":2:{s:7:"*file";R:4;};}}`有什么作用，我将上面的答案进行反序列化，发现反序列化失败。我将`O:9:"Exception":2:{s:7:"*file";R:4;};}}`修改为其他的值之后，发现无法得到flag。所以这一点就很奇怪了。

### 总结

在做这道题目的过程中，遇到了一些奇奇怪怪的问题，也请教了很多人，自己也进行了很多的尝试，但是毕竟情况太过于少见，大神们也无法当面解决，所以问题还是没有得到解决。
看来有些问题也只有靠自己才能够解决了，也期待和师傅们一起交流学习。

[博客的原文地址](https://bbs.kanxue.com/elink@59aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7%4m8G2L8$3y4C8i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9&6i4K6u0r3x3o6W2Q4x3V1k6n7j5h3u0&6k6X3W2J5M7%4c8Q4x3X3c8%4M7X3W2@1k6i4g2H3i4K6u0r3)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
