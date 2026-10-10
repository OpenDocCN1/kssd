

# PECompact2.79\_Beta\_D 分析及静态脱壳机编写,附源码

> 原文：[`www.pediy.com/kssd/pediy10/91530.html`](https://www.pediy.com/kssd/pediy10/91530.html)

在科锐学习的第 4 阶段，课后作业静态脱壳机。  
由于学习编程,逆向不久,代码写的很粗糙。自己水品不够,难免会有很多 bug,大哥们海涵!  
  

引用: 外壳 shell 的分析  
壳版本: PECompact2.79\_Beta\_D   
工具: IDA, OD, winhex, PEID  
1.  外壳第一部分  
  该外壳的 shell 分部分, 第一部分以非压缩的方式存在, 第二部分以压缩的方式存在.  
外壳执行时先执行第一部分, 这部分将 shell 的第二部分在内存中解压缩, 并初始化一些数据. 用 IDA 打开加壳后的程序(TestDll.dll), 来到外壳的入口处, 分析如下:  
  
10001230  mov eax, offset ExceptionProc ; 异常处理回调函数  
10001235  push eax  
10001236  push large dword ptr fs:0  
1000123D  mov large fs:0, esp  ; 注册异常  
10001244  xor eax, eax     ; eax = 0  
  
; mov \[0\], ecx,  程序执行到此处时产生内存访问异常,   
; 转到异常处理 ExceptionProc  
; 异常处理里改下此地址为 E9 与 此地址与 0x1003892F 相对偏移,   
; 即 jmp 1003892F  
; 异常返回后,则 jmp 1003892F  
10001246  mov \[eax\], ecx    
10001248  push eax  
  
……更多内容参见附件 

上传的附件
| ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg) || 分析以及编写文档.doc.doc  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 静态脱壳机.rar \[解压密码：pediy.com\]   
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 测试文件.rar\[论坛下载：[`bbs.pediy.com/showthread.php?t=91530`](http://bbs.pediy.com/showthread.php?t=91530) \]   
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || IDB.rar  
 |