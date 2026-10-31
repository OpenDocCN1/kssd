
# [原创]分享一个自己写的手动映射PE文件源码

看雪第一次发帖,开源分享一下自己的学习成果~

最近在研究`R0`无痕注入`R3`,用`CPP`写了个手动映射`DLL`的功能,`X32`和`X64`正常编译出来的`DLL`文件都能兼容,兼容`VM`代码的`DLL`,但不兼容加了压缩壳之类的`DLL`

### 特点

隐蔽性高,全程无`API`调用,可以设置不执行`CRT`初始化代码

例子和源码放在附件里,调整一下编译器设置就可以提取出`shellcode`,大家可以随便玩

`emmm`,功能大概就这几个

![图片描述](upload/attach/202107/844784_6UHNZKRJ6YPMZ4K.png)

# 分析一下实现过程

## `GetMemModuleHandleW`函数

先获取`PEB`结构,`X32`和`X64`获取方式不同:

![图片描述](upload/attach/202107/844784_N4XSC7BE4GQ6Y4G.png)

然后遍历`PEB->LDR`的双向循环链表,取出每个模块的模块名分别判断,最后返回模块基址

![图片描述](upload/attach/202107/844784_UT5A67WPC6T74XJ.png)

## `GetMemProcAddress`函数

前面是解析`PE`结构中的`ExportDirectory`,代码就不贴上来了,要注意的是系统中有些函数是绑定导入,需要单独做解析,如`Win10`系统`Kernel32.dll`中的`EnterCriticalSection`函数,实际上模块加载之后是引用的`ntdll.dll`里的`RtlEnterCriticalSection`函数地址

![图片描述](upload/attach/202107/844784_KP6EX3XFCGTP233.png)

我的解决方法是判断`PE`结构中描述的函数地址开头数据是否全是字符,符合条件就提取出`dll`模块名和函数名,再进行递归解析

![图片描述](upload/attach/202107/844784_X9JWEYK25R9WF4T.png)

![图片描述](upload/attach/202107/844784_S582847QV7G2Y5G.png)

## `LoadMemLibrary`函数

先把`Dll`文件各个区段的数据映射到内存中

![图片描述](upload/attach/202107/844784_42WTYY8H9BAV7K3.png)

接下来修复`IAT`和修复重定位,这个懂得都懂,就不贴代码上来了

调用`DllMain`这里我做了一些修改,因为一般的`DLL`编译出来,入口点不是直接指向的`DllMain`,而是先运行一系列`CRT`初始化代码,再运行`DllMain`,在某些极端的`AC`保护下,这些`CRT`初始化代码很有可能导致被检测,所以可以`Call`一个自己设定的导出函数,不运行`CRT`初始化代码

![图片描述](upload/attach/202107/844784_W4SKZWPHMFGKCCY.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-7-12 21:35

被`cslime`编辑

，原因：

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

上传的附件：

- [CS_DLL_LOADER_Res.zip](javascript:void(0);)

  （102.32kb，337次下载）
