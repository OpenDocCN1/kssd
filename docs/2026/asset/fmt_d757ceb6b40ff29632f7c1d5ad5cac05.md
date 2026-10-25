
# [翻译]逆向一个基于驱动的保护方案:Sandboxie

## Reversing of a protection scheme based on drivers : Sandboxie

如何对驱动中的注册校验进行分析？

本文以静态分析的方式对 Sandboxie 程序驱动进行了逆向分析，详细推断了驱动内的注册校验过程及打补丁方法，并以裸函数调用的方式完成了注册机的编写。

bambooo0 翻译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`译文_part1.rar`](javascript:void(0);)（813.86kb，404次下载）
- [`译文_Part2.rar`](javascript:void(0);)（749.40kb，862次下载）
- [`译文_Part3.rar`](javascript:void(0);)（902.28kb，877次下载）
