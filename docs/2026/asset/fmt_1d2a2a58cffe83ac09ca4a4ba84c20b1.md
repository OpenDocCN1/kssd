
# [原创]CS[1]exe木马分析

Cobalt Strike是渗透测试工具，可以通过exe木马实现远程控制。

## 一：生成exe

Windows Executable 生成可执行exe木马；payload分段

Windows Executable(S) 生成无状态的可执行exe木马，payload不分段

![在这里插入图片描述](upload/attach/202204/921642_Y87TD3R9SK5KDHY.jpg)

分析Windows Executable 生成的`artifact.exe`

查壳：无壳

## 二：具体分析

`401840`关键函数：获取系统时间戳，通过`sprintf`拼接管道名，创建线程，写加密数据到创建的管道内，最后解密执行

![在这里插入图片描述](upload/attach/202204/921642_A877YZM4J7K65E6.jpg)

![在这里插入图片描述](upload/attach/202204/921642_2VYUWAZXZ3N3ZNZ.jpg)

线程函数：

[![在这里插入图片描述](upload/attach/202204/921642_778GZE8NJ2D3K4J.jpg)

跟进线程函数，创建命名管道，将加密数据写进管道内![在这里插入图片描述](upload/attach/202204/921642_638MJ5K8HYFXP88.jpg)

加密的数据：![在这里插入图片描述](upload/attach/202204/921642_48CHB6937XK92GW.jpg)

加密数据`crypt_data1`写入管道内![在这里插入图片描述](upload/attach/202204/921642_NADY3GBAZCD4JGD.jpg)

读取管道内的数据，解密数据，跳转执行![在这里插入图片描述](upload/attach/202204/921642_VZJQXXFQZ2PJQGU.jpg)

![在这里插入图片描述](upload/attach/202204/921642_T3X2H42X248EBWM.jpg)

`decodeandcall`：解密数据，修改内存属性，跳转执行![在这里插入图片描述](upload/attach/202204/921642_UWVREAWN7Q9E9DC.jpg)

用于解密的数组：![在这里插入图片描述](upload/attach/202204/921642_YQ7J5VVWUM3KAC2.jpg)

申请的内存地址`20000`![在这里插入图片描述](upload/attach/202204/921642_SSV6DDXADNNEQ5X.jpg)

跳转到`eax`：`20000`执行shellcode

### shellcode分析：

1. `LoadLibrary`加载`wininet.dll`

![在这里插入图片描述](upload/attach/202204/921642_XEXKYZV45P67DWP.jpg)

`InternetOpen`函数:在进行HTTP、FTP和服务器通信前初始化 WinInet.dll 。简单的说通过 `InternetOpen` 函数创建位于根部的 Hinternet 句柄

![在这里插入图片描述](upload/attach/202204/921642_6QUAE36GKNSUZS4.jpg)

`InternetConnect` 连接：`192.168.99.8`![在这里插入图片描述](upload/attach/202204/921642_VX77VR6K4A4PH72.jpg)

`HttpOpenRequestA` 创建 HTTP 请求句柄。

![在这里插入图片描述](upload/attach/202204/921642_DDSXTX4EDFBXAQ8.jpg)

`HttpSendRequestA`发送HTTP请求到指定的服务器![在这里插入图片描述](upload/attach/202204/921642_F8E6F2WGHBHDTFC.jpg)

`VirtualAlloc` 申请内存：![在这里插入图片描述](upload/attach/202204/921642_K6BYT2NDASWE5UQ.jpg)

不断`InternetReadFile` 读数据到申请的内存![在这里插入图片描述](upload/attach/202204/921642_DM7YNMSZVUV8XT5.jpg)

读到的数据：

![在这里插入图片描述](upload/attach/202204/921642_WNX3NYWS3APS4JU.jpg)

读完数据后继续跟

![在这里插入图片描述](upload/attach/202204/921642_XFR7NKEXZ7MZVRU.jpg)

跳转到申请的内存处，解密出dll。

![在这里插入图片描述](upload/attach/202204/921642_FXX9FJ36T4WUYJC.jpg)

解密出的pe文件

![在这里插入图片描述](upload/attach/202204/921642_F9B9H4W86NSZ6BF.jpg)

后面继续跟就是反射dll注入

`7CB2+7`为 `ReflectiveLoader`函数的偏移，`call ebx`调用`ReflectiveLoader`函数反射注入dll，压入参数`call eax` 调用`dllmain`

![在这里插入图片描述](upload/attach/202204/921642_FKB9W86GHXNMB6E.jpg)

```
dec ebp ; M
pop edx ; Z
call 0 ; 将下一条指令地址压到栈里
pop ebx ; 将压到栈里的地址弹到ebx
push edx ;
inc ebp ;
push ebp ; 保存ebp
mov ebp, esp ; 切换堆栈
add ebx,0x??? ; ebx+ReflectiveLoader函数偏移-7（ebx相起始偏移7）
call ebx ; call ReflectiveLoader
```

![](upload/attach/202204/921642_MRVCT9NBF526NVE.png)

最后于 2022-4-5 20:26 被pyikaaaa编辑，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
