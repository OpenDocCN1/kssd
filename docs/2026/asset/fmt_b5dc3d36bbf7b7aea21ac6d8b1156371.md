
# [原创]一个word溢出样本的shellcode分析

【文章标题】: 一个word溢出样本的shellcode分析

【文章作者】: 守卫者

【下载地址】: 请自行搜索

【作者声明】: 本文是小菜的学习笔记，请轻拍

前几天从网上下载一个doc文档，打开时360竟然立即提示“远线程注入危险行为”。
我很是奇怪，怎么打开一个doc文档也引起报警。仔细查找原因，原来这个doc文档竟然会使 word发生溢出。发生溢出后释放了一个exe格式的病毒文件，幸好这个病毒运行时被360拦截了，不然我就中招了 ![](https://bbs.kanxue.com/view/img/face/16.gif)

我一时兴起，就把这个doc文档的shellcode跟踪了一遍。
因为以前也做过几次，所以轻车熟路

下面是shellcode的汇编分析，不知道怎么弄语法高亮，大家凑合吧 ![](https://bbs.kanxue.com/view/img/face/50.gif)
