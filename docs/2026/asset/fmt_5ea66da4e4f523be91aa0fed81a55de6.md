
2、本文只实现了拦截模块的调出到其他模块的动作，其他模块的调入没有涉及到（可能还涉及到比较复杂的重定位操作）。

3、`system_server`是`system`用户，不是有权限写所有名字的Property，这里用了`persist.sys.`开头的属性名，而`persist.sys.`开头的属性会保存至磁盘，因此性能会差点儿。

4、`ioctl`虽然实质声明是个可变参数：`int new_ioctl (int __fd, unsigned long int __request, /*void * arg*/ ...)`，这种声明的函数要直接透明地将参数从旧函数传递给新函数似乎还不可行，搜了很多资料也没找到。幸好搜了一把`libbinder.so`源码，里头对`ioctl`的调用参数均是仨，干脆就不处理变长形式了。

5、如果不以`root`身份运行注入程序，则`ptrace`附加时会失败。

6、Andriod系统的大部分Service都运行在`system_server`进程中，可以拦截到。但部分自定义的用户Service在用户进程中，如需要拦截，则要`ptrace`到那个用户进程才行，拦截方法也类似。

7、至于拦截`Binder`的数据分析与修改，则是下一篇文章的内容了。

---

## 五、参考资料

---

5adK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2L8X3E0#2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3U0t1J5k6U0x3@1z5r3j5^5y4o6R3$3z5o6M7$3x3X3y4S2j5h3g2V1y4e0f1^5i4K6u0W2K9s2c8E0L8l9`.`.

684K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8r3W2F1k6$3k6G2L8X3N6Q4y4h3k6U0L8$3!0D9i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5e0M7$3x3e0p5J5

2bbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3P5h3I4&6N6h3q4F1L8s2g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0j5$3x3K6R3^5x3U0f1`.

a1aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3!0K6i4K6u0W2M7r3E0#2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3@1p5^5x3o6R3H3i4K6u0r3k6$3q4A6K9%4g2S2L8X3N6Q4x3V1k6K6N6h3u0E0K9i4y4K6K9h3!0F1i4K6u0r3g2p5^5H3y4g2)9J5k6f1g2x3c8W2)9J5k6f1k6G2M7X3#2S2N6q4)9J5k6g2y4#2L8h3#2S2M7Y4W2Q4x3X3g2H3k6r3j5`.

a5aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3k6X3g2F1k6$3E0W2K9s2g2S2L8W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4U0t1J5x3K6b7H3y4R3`.`.

88dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9h3&6F1L8%4y4@1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5$3x3e0t1@1y4U0R3#2

还有不少，没法全写下来，在此向所有无私共享自己研究成果的人致以诚挚的谢意。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
