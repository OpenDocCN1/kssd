
```
L:dsm:x86_32 > setarch arm32



[+] Architecture of dsm changed to arm32

```

### 系统调用

显然在编写shellcode时会需要系统调用, 因此shellen可以为你提供帮助. 只需要输入`sys`和系统调用名称, shellen就可以展示出你可能需要的系统调用列表.

```
L:asm:x86_32 > sys open



┌────────┬───────┬──────────────────────┬──────────────────────┬──────────────┬──────────────┐

│ name   │ eax   │ ebx                  │ ecx                  │ edx          │ esi          │

├────────┼───────┼──────────────────────┼──────────────────────┼──────────────┼──────────────┤

│ open   │ 0x05  │ const char *filename │ int flags            │ umode_t mode │ -            │

│ openat │ 0x127 │ int dfd              │ const char *filename │ int flags    │ umode_t mode │

└────────┴───────┴──────────────────────┴──────────────────────┴──────────────┴──────────────┘

```

当然你查找的内容不同, 打印出的系统调用表也会有所变化. 搜索到的系统调用表取决于你所选择的架构和操作系统. 在这个示例中它是`x86_32`和`Linux`.

### 通用shellcode

shellen可以根据你的关键字展示给你一个通用shellcode的清单. 使用示例可以在之后的`展示图`部分看到. 它使用了`shell-storm.org`提供的API(谢谢该站长!), 你可以像下面这样使用该功能

```
L:asm:x86_32 > shell <keyword> <count>

```

事实上, 参数`count`并不是必需的

### 操作系统

之前有说会有一些操作系统, 但目前只支持Linux. 如果你想要添加Windows或MacOS的功能, 可以提交issue, 我将会添加对该功能的支持.

### 基础命令

| 命令 | 描述 |
| --- | --- |
| clear | 清空终端屏幕, 跟windows的`cls`或\*nix系统的`clear`命令一样 |
| help | 显示一些帮助信息 |
| quit,q,exit | 关闭当前会话并退出 |

### 依赖包

```
    > keystone

    > capstone

    > colorama

    > termcolor

    > terminaltables

```

## 展示图

只是一些使用过程的展示图

![](https://bbs.kanxue.com/upload/attach/201802/722644_6MY97ABYPWCYBUT.png)

![](https://bbs.kanxue.com/upload/attach/201802/722644_F35GCGT59797X4X.png)

![](https://bbs.kanxue.com/upload/attach/201802/722644_3PAEPY72KVAJ7CZ.png)

![](https://bbs.kanxue.com/upload/attach/201802/722644_ZYD6F38GZEP6SV2.png)

原文地址: [3ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8K9i4c8H3L8r3!0A6N6q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3W2)9J5c8Y4y4Z5k6h3I4D9k6h3&6Q4x3X3c8A6L8Y4c8W2M7X3q4U0N6r3W2$3k6g2)9J5k6s2y4Z5k6h3I4D9j5$3!0V1K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@674K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8K9i4c8H3L8r3!0A6N6q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3W2)9J5c8Y4y4Z5k6h3I4D9k6h3&6Q4x3X3c8A6L8Y4c8W2M7X3q4U0N6r3W2$3k6g2)9J5k6s2y4Z5k6h3I4D9j5$3!0V1K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.)

本文由 看雪翻译小组 Vancir 翻译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-2-27 21:07

被Vancir编辑，原因：
