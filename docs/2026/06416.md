
# [原创]PECompact2.79_Beta_D 分析及静态脱壳机编写,附源码

在科锐学习的第4阶段，课后作业静态脱壳机。

由于学习编程,逆向不久,代码写的很粗糙。自己水品不够,难免会有很多bug,大哥们海涵!

## 外壳shell的分析

壳版本: `PECompact2.79_Beta_D`

工具: `IDA`, `OD`, `winhex`, `PEID`

### 1. 外壳第一部分

该外壳的shell分部分, 第一部分以非压缩的方式存在, 第二部分以压缩的方式存在.

外壳执行时先执行第一部分, 这部分将shell的第二部分在内存中解压缩, 并初始化一些数据. 用`IDA`打开加壳后的程序(`TestDll.dll`), 来到外壳的入口处, 分析如下:

```
10001230 mov eax, offset ExceptionProc ; 异常处理回调函数
10001235 push eax
10001236 push large dword ptr fs:0
1000123D mov large fs:0, esp ; 注册异常
10001244 xor eax, eax ; eax = 0
```

`; mov [0], ecx, 程序执行到此处时产生内存访问异常,`
`; 转到异常处理ExceptionProc`
`; 异常处理里改下此地址为E9 与 此地址与0x1003892F相对偏移,`
`; 即jmp 1003892F`
`; 异常返回后,则jmp 1003892F`

```
10001246 mov [eax], ecx
10001248 push eax
```

……更多内容参见附件

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [分析以及编写文档.doc.doc](javascript:void(0);)

  （253.50kb，174次下载）

- [静态脱壳机.rar](javascript:void(0);)

  （138.22kb，180次下载）

- [测试文件.rar](javascript:void(0);)

  （1.88MB，298次下载）

- [IDB.rar](javascript:void(0);)

  （79.85kb，127次下载）
