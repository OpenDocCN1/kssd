
# [原创]32位反汇编引擎开发笔记：Step.1_引擎大纲

## 目录

Step.1\_引擎大纲

[Step.2\_框架搭建](https://bbs.pediy.com/thread-262483.htm)

[Step.3\_经典定长指令的解析](https://bbs.pediy.com/thread-262490.htm)

[Step.4\_变长指令复习1](https://bbs.pediy.com/thread-262503.htm)

[Step.5\_变长指令复习2 - 指令前缀](https://bbs.pediy.com/thread-262532.htm)

[Step.6\_引擎核心->函数封装](https://bbs.pediy.com/thread-262633.htm)

## 内容目录

[TOC]

## 项目背景

此项目为楼主自学滴水三期后的毕业项目,旨在练习巩固硬编码基础知识,PE基础知识,C基础知识

## 功能说明

1. 读取32位PE文件,通过PE信息截取.text代码段数据
2. 将代码段数据通过引擎解析为一行行的汇编语句

## 功能效果

参考PEID右键区段->反汇编

![引擎效果展示](upload/attach/202010/867232_U7G2SSXGHHHM6RM.png)

## 技能要求

1. 熟悉PE结构
2. 熟悉基本的汇编指令
3. 熟悉硬编码基础
4. 基本的英语基础或者谷歌翻译使用基础
5. 手里有一份Intel白皮书pdf
6. c/c++基础 && 代码经验

## 开发工具

VS2017社区版 + 番茄助手

## 参考资料\_附录提供下载

1. Intel白皮书
2. PE文件结构

## 其他说明

此项目我会完全参照Intel白皮书进行解析开发,以快速开发为宗旨

老话说的好:`先看有没有,再看好不好`

因此代码中可能会出现大量冗余代码,不健全的代码逻辑,未知的BUG,各位读者择优吸收

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-10-13 17:42

被SSH山水画编辑，原因：更新参考资料

上传的附件：

- [PE结构.pdf](javascript:void(0);)
  （373.42kb，111次下载）
- [Intel白皮书.7z](javascript:void(0);)
  （4.63MB，143次下载）
- [DisassemblyEngine.zip](javascript:void(0);)
  （1.84MB，132次下载）
