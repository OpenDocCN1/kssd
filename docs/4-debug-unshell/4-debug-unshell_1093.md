

# \[12 月专题\]TheMida\_defeating\_ring0

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-721.htm`](https://www.pediy.com/kssd/pediy09/pediy09-721.htm)

> http://bbs.pediy.com/showthread.php?s=&threadid=25001&highlight=antiantidump  
>   
> http://bbs.pediy.com/showthread.php?s=&threadid=25564&highlight=antiantidump  
>   
> 为了帮助理解，建议大家先把上面两个连接的文章先看看  
>   
> 本人英语水平有限，有的地方理解不清，语文水平有限，有的地方表达不清  
>   
> 最近想学习 RING0 的东西，看了看这文章觉得单词还算简单，就翻译了。  
>   
> 原文：  
> Tutorial: TheMida Defeating Ring0   
> Author: deroko  
> http://arteam.accessroot.com/tutorials.html?fid=126  
>   
> 大家就对照着凑合看了  
>   
> 另外说说我从本文总结出的脱未知壳（指没有相关脱壳文档）的一般流程。  
>   
> （解决 anti dump）->dump ->静态分析，判断是何种语言，根据该语言 RTL 的特点找到并修正 OEP->运行该 DUMP，得到引起异常的地址，在未脱壳程序中的该地址下内存断点，定位加密 IAT 或其他加密代码，分析然后修正。
> 
> | ![文件类型: rar](img/0a094b3fa15a6229acfe39404013c8ad.jpg) || TheMida\_defeating\_ring0.rar ||   |
> | --- | --- | --- |

 4dd466d0e1e0cc123afbe504da2462ca