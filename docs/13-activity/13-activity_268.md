

# 【看雪论坛.腾讯公司 2008 软件安全竞赛】我的第三题分析过程及答案

> 原文：[`www.pediy.com/kssd/pediy10/74392.html`](https://www.pediy.com/kssd/pediy10/74392.html)

下载完 cm 后，这一次我也学习了 shoooo 大侠，立刻 google 了一下 R6002  
  
结果发现这个链接  
  
http://msdn.microsoft.com/zh-cn/library/k1x26e0x(VS.80).aspx  
  

引用: C 运行时错误 R6002  
      
 **             错误消息           **  
  
未加载浮点支持  
   
 未链接必需的浮点库。  
 **通过检查下面的可能原因进行修复**  

1.    该程序通过选项（如 /FPi87，该选项要求有协处理器）被编译或链接，但该程序运行在一台未安装协处理器的计算机上。
2.    **printf** 或 **scanf** 函数的格式字符串包含浮点格式规范，而该程序不包含任何浮点值或变量。
3.    编译器仅当必要时才通过加载浮点支持以最小化程序大小。编译器无法检测到格式字符串中的浮点格式规范，因此编译器未加载必要的浮点例程。
4.    使用浮点参数以符合浮点格式规范，或在程序的其他地方执行浮点赋值。该操作将导致加载浮点支持。
5.    在由混合语言编写的程序中，当程序进行链接时在 FORTRAN 库之前指定了 C 库。重新链接并最后指定 C 库。

  
   

初步跟踪了一下，问题是出在  
  

引用: .text:00420D81 E8 7C E0 FE FF                    call    \_\_snwprintf 

跟进去发现是在对%e-%g 的处理时直接调用了出错处理函数然后直接退出  
也就误解成编译器优化没有把相应的代码链接进入可执行文件，于是先提交了一个笨办法，改 0x420D81 这句指令跳转到空闲空间 0x421D00 并实现如下代码  
  

引用: .text:00421D00 68 20 1D 42 00                    push    offset aPediydll\_dll ; "pediydll.dll"  
.text:00421D05 FF 15 C4 21 42 00                 call    ds:LoadLibraryA  
.text:00421D0B 68 30 1D 42 00                    push    offset aMy\_snwprintf ; "my\_snwprintf"  
.text:00421D10 50                                push    eax  
.text:00421D11 FF 15 50 22 42 00                 call    ds:GetProcAddress  
.text:00421D17 FF D0                             call    eax ;  
.text:00421D19 E9 68 F0 FF FF                    jmp     loc\_420D86  
.text:00421D20 70 65 64 69 79 64+aPediydll\_dll   db 'pediydll.dll',0     ; DATA XREF: .text:loc\_421D00  
.text:00421D2D 00 00 00                          align 10h  
.text:00421D30 6D 79 5F 73 6E 77+aMy\_snwprintf   db 'my\_snwprintf',0     ; DATA XREF: .text:00421D0B  

思路是在动态库里实现完整\_snwprintf 然后调用，my\_snwprintf 只有一条指令  

引用: jmp        \_snwprintf 

然而，提交后才发现原来对浮点处理的代码其实是编译进了 testfloat.exe，那为什么会出错呢？  
  
原来是在程序启动阶段\_\_\_tmainCRTStartup 函数中调用了\_\_cinit 函数，在这个函数的第一个判断是校验浮点运算初始化函数指针所处的 section 是否为可写，如果可写的话就跳过浮点运算初始化函数，因此缺省情况下函数\_snwprintf 会直接显示出错信息并退出，如果该 section，也就是.rdata 不可写，代码会调用浮点运算初始化函数，缺省的指向出错函数的指针会被替换成正常处理函数，因此可以正常运算并显示 PI \* 2 = 6.283180  
  
可惜之前误解 msdn 还以为是编译器没有编译进去这部分代码，原来只不过是没调用而已，汗一个  
  
这样一来解决办法就非常简单了，我们只需要改动一位，将.rdata 的可写属性去掉，也就是修改文件地址 0x22F 处的.rdata section 的特征符，将该数值从 0xC0 改成 0x40 即可  
  
或许有些朋友好奇为什么 upx 之后就可以正常处理了，也同样是因为.rdata 的 characteristics 的第 31 位被清零所致

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TestFloat.rar  
 |