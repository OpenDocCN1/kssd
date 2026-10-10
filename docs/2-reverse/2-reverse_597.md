

# 俺来看个 KernelExec.sys-->KernelExec.idb

> 原文：[`www.pediy.com/kssd/pediy10/59387.html`](https://www.pediy.com/kssd/pediy10/59387.html)

嘿嘿,IDA 初学![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ,俺在 CodeProject 上下了份从内核启动一个 R3 程序的例子.想学下逆向,于是直接拿 IDA 看了下这个 sys:**KernelExec.sys**,注释了一下,终于明白了这个驱动的作用啦:  
  
从内核启动一个 R3 程序.大致是遍历 EPROCESS 找到 explorer.exe, 在其中找到一个 alertable 的线程,插个用户 APC. 这个用户 APC 就是简单调用 WinExec 去执行一个指定的程序, 期间要在内核中 attach 到 explorer.exe 进程中,为 My\_Func 分配一个 MDL,然后映射到 explorer.exe 的虚拟进程空间中.返回时 APC 就可以执行啦.  
  
不过这个 sys 好像有点儿毛病, 它简单的设置 ApcState.UserApcPending = TRUE, 好像不调用**KeForceResumeThread**的话 APC 不会执行,俺电脑上一直就没有出效果 o(\*.\*)O  
  
哈哈,原来逆向蛮好玩儿的(俺还没还原成 C 代码,不算逆向![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ).  
  
btw: 好像真正的在内核中启动一个 R3 程序,MS 是要拦截掉**\_KiSystemService**,在里面 jmp 到我们指定的代码处,一般是找 kernel32.dll 的空隙插, 找到**CreateProcessA**的地址,弄到 kernel32.dll 在内存中的空隙里, 然后再 XXX 就产生了一个 R3 程序.... 嘿嘿,学习学习![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ~~

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || KernelExec\_idb.rar \[解压密码：pediy\]   
 |