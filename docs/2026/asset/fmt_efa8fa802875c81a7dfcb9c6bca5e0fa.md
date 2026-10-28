
由上面可知`RtlInitUnicodeString`函数是通过设置`Buffer`变量指向字符串区域，并没有再次拷贝一份字符串，**所以当`DriverEntry`函数所在内存页面被释放后，调用`DriverUnload`函数时访问`Buffer`指向的内存区域无法访问导致蓝屏。**

## 修改

笔者修改`DriverUnload`部分源码如下：

```
UNICODE_STRING SymbolLinkName;
RtlInitUnicodeString(&SymbolLinkName, DRIVER_SYMBOLLINK_NAME);
IoDeleteSymbolicLink(&SymbolLinkName);
```

停止驱动时不会崩溃，查看这部分代码的静态反汇编：

![](https://bbs.kanxue.com/upload/attach/202107/762319_MGCBAVN2PP7D4MJ.png)

![](https://bbs.kanxue.com/upload/attach/202107/762319_Y76FBJ74BWUNZTP.png)

上面两幅图可以看到 调用`RtlInitUnicodeString`函数时使用的字符串是位于`PAGE`页面，`PAGE`节的特点是位于可以进行分页交换的内存空间，这些空间在内存紧张时可以被交换到硬盘上以节省内存。

## 总结

1. 本次学习了内核双机调试、源码调试、符号配置。
2. 用户态编程时使用上面`#define`定义字符串的形式，字符串位于PE文件的中可读不可写的区域，**程序内部多次用到此字符串时，始终指向的是此区域的那一份数据，这与内核态编程不一样，目前来看，字符串内容虽然相同，但是在`INIT`页面与`PAGE`页面中使用的还是不同内存地址的字符串，即存在两份相同的字符串数据**。
3. 学习了`RtlInitUnicodeString`函数初始化`UNICODE_STRING`的方式,`RtlInitUnicodeString`函数不会分配内存
4. 这个问题的暴露应该与给虚拟机分配了`1G`内存有关，内存过小，导致驱动入口函数执行完毕后内存页面就被释放了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-7-27 22:42 被0346954编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
