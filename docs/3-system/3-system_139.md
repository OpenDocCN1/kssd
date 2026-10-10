

# 我也发个 PE 文件查看器

> 原文：[`www.pediy.com/kssd/pediy12/137042.html`](https://www.pediy.com/kssd/pediy12/137042.html)

这两天写的一个 PE 查看器，附件里的代码缺少的部分可以到软件保护结题资源那个帖子去找。  
这个小程序是命令行的 把结果输出的 XML 文件里。  
用法如下:  
xanalyzer -peview  PE 浏览器选项  
/peh 打印 PE 头  
/pshs 打印节头  
/pet 打印输出表  
/pit 打印输入表  
/prst 打印资源表  
/prlt 打印重定位表  
  
xanalyzer -dump 转存选项  
/dp 0x11 转存从文件开头 长度 0x11 字节的内容  
/dpe 0x22 转存从入口点代码 长度 0x22 字节的内容  
  
xanalyzer -disasm 反汇编选项  
/dis 反汇编  
/att 以 ATT 语法反汇编,默认为 INTEL  
  
xanalyzer -analyze 分析选项  
/cval (1-4)个数字  4 个数字用来确定有效地址的级别 1 级最宽松,4 级最严格   
/dioid 关闭视无效代码为数据  
/mcs 7 最小代码长度为 7 也就是说如果分析出一篇内存，如果小于 7 则直接以数据对待  
/nine 以 nop 指令作为函数的结尾标志  
/iine 以 int3 指令作为函数的结尾标志  
/zine 以零数据作为函数的结尾标志  
/iatref 分析出整个代码段里 哪些指令调用了引入表的函数  
/aepa 地址  指定一个地址作为起始开始分析  
  
/out 目录  把输出结果输出的哪个目录  
  
例如  
analyzer -disasm /dis /att -analyze c:\\helloword.exe  
反汇编以 ATT 语法输出  
分析这个程序  
  
这些选项可以叠加使用，每个以'-'开头的命令将输出为一个 XML 文件。  
如果是分析入口点为 0 的程序将直接退出。  
  
过两天再发一个带监视器的版本。。。 把分析出的函数启动后做 HOOK，直接监视程序调用函数的流程。![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")

上传的附件
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || src.zip  
 |
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || bin.zip  
 |