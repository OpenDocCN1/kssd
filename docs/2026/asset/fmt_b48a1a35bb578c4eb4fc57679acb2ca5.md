
![图 6](https://bbs.kanxue.com/upload/attach/202010/787173_GGV28MCJBFYNAK7.jpg)

- 选择了我们上面修改的工具及这里就会出现 `LLVM` 选项我们这里只需要 `Clang-cl` 其他全选否(vs2017 有三个选项)

![图 7](https://bbs.kanxue.com/upload/attach/202010/787173_BTW76JTCZRYFHE4.jpg)

- 关闭Qspectre缓解

![图 8](https://bbs.kanxue.com/upload/attach/202010/787173_RB497KZ53PWYRDZ.jpg)

- c/c++命令行从父级或项目默认设置继承取消勾选

### 5. 可选

- 使用带混淆的llvm比如 [ollvm](https://bbs.kanxue.com/elink@83bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6i4u0G2K9h3#2K6i4K6u0r3L8$3u0X3N6i4y4U0j5i4c8G2M7W2)9J5c8Y4c8J5k6h3g2Q4x3V1k6D9L8s2k6E0i4K6u0V1z5g2)9J5k6e0l9`.) 或者 [goron](https://bbs.kanxue.com/elink@c24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8h3W2E0L8#2)9J5c8X3N6G2M7X3!0F1) 等其他可以在

c/c++命令行其他选项添加混淆选项以下是[goron](https://bbs.kanxue.com/elink@a1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8h3W2E0L8#2)9J5c8X3N6G2M7X3!0F1)的混淆选项

|  |  |
| --- | --- |
| 1 | `-``mllvm` `-``irobf``-``cse` `-``mllvm` `-``irobf``-``indbr` `-``mllvm` `-``irobf``-``icall` `-``mllvm` `-``irobf``-``indgv` `-``mllvm` `-``irobf``-``cff` |

![图 10](https://bbs.kanxue.com/upload/attach/202010/787173_QRPYU3P5PUR66NR.jpg)

### 6. 项目地址

- 里面包括修改好的项目以及修改好的编译器配置

[项目地址](https://bbs.kanxue.com/elink@1f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9r3!0J5k6s2m8Q4x3V1k6o6L8r3q4F1k6#2c8W2M7%4b7`.`)

#### 博客

[如何使用LLVM编译Windows驱动(混淆&Asm)](https://bbs.kanxue.com/elink@f82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3!0J5k6s2m8Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0m8Q4x3V1j5I4x3q4)9J5c8U0p5@1i4K6u0r3i4K6t1#2c8e0g2Q4x3U0g2m8y4W2)9J5y4e0R3J5i4K6t1#2c8e0c8Q4x3U0g2n7c8q4)9J5y4e0V1#2i4K6t1#2c8e0c8Q4x3U0g2n7c8q4)9J5y4f1u0r3i4K6t1#2c8e0N6Q4x3U0f1&6y4q4)9J5y4f1p5^5e0p5I4h3e0g2)9J5y4f1f1%4i4K6t1#2b7V1y4Q4x3U0f1&6y4W2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6x3g2N6A6L8X3c8G2N6%4y4Q4x3U0g2q4z5g2)9J5y4f1p5&6i4K6t1#2b7U0q4Q4x3U0g2q4y4g2)9J5y4e0S2m8i4K6t1#2b7e0S2Q4x3X3c8Q4x3U0g2q4y4W2)9J5y4f1t1%4i4K6t1#2b7U0N6Q4x3U0g2q4y4W2)9J5y4f1t1%4i4K6t1#2z5o6k6Q4x3X3c8m8M7$3#2Q4x3V1j5`.`)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
