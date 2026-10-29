
问题发生了，如何解决，这里使用`Ollydbg`查看模块地址。

![](https://bbs.kanxue.com/upload/attach/202202/848111_V7UX3KB4DX5YER7.png)

![](https://bbs.kanxue.com/upload/attach/202202/848111_J2QW4NAADHC3HME.png)

下面是`Ollydbg`的正确解决方案

### 步骤1：点击E字母

![](https://bbs.kanxue.com/upload/attach/202202/848111_4KG73KD3YSTK84G.png)

### 步骤2：找到`User32`，此时使用该模块地址计算，可以获得正确地址，成功弹出`messagebox`

如果不想算，就按一下`User32`，快捷键`ctrl+n`，找到`messageboxA`一样可以。

|  |  |
| --- | --- |
| 1  2 | `>>> hex(0x75cc0000 + 0x83670)` `'0x75d43670'` |

![](https://bbs.kanxue.com/upload/attach/202202/848111_5ZRXAF8CJES58UU.png)

![](https://bbs.kanxue.com/upload/attach/202202/848111_H3CV8R684F3YRMV.png)

![](https://bbs.kanxue.com/upload/attach/202202/848111_UZ3VF3QA773GGXB.png)

注：

不知不觉三阶段快毕业了，非常感谢科锐的指导，科锐的耐心指导，才能够让我的基础越加雄厚。

非常感谢钱老师、张老师、王老师、田老师、唐老师、江老师。

尤其王老师，陪伴我们走了最长的路，还记得科锐学习的时候经常学崩溃，因为任务量多，对新手难。

有一次哭着和王老师聊天，王老师帮我排解压力、还把笔记借给我学习，非常感谢。

这是估计是我在科锐期间最后一次发帖了，在此纪念一年的学习时光。

祝科锐越来越好，也祝自己越来越好。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-3-7 11:56

被瑞皇编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

上传的附件：

- [第二章.zip](javascript:void(0);)

  （2.78MB，21次下载）
