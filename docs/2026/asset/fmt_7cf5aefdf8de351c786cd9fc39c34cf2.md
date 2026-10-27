
例如：`lods byte ptr ds:[esi]`可以变成`mov aXX,[ESI]` `INC esi`等等

## 2、涉及重定位的代码还是没有修复

例如：

```
push 0xFACE0002

mov edi,0xFACE0003

jmp dword ptr ds:[eax*4+0x474FCF]

jmp short 00474984
```

## 3、找到填充虚拟机上下文的两个Handle块

后续章节介绍：

1、剩下重定位修复
2、伪代码构造

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-8-8 20:53

被黑手鱼编辑

原因：

上传的附件：

- [VMP1.21.7z](javascript:void(0);)
  （3.53MB，73次下载）
- [VMP1.21学习笔记之Handle块优化与壳模板初始化（四）.pdf](javascript:void(0);)
  （3.76MB，67次下载）
