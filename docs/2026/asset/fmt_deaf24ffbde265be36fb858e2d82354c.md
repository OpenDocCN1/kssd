
![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_K2NVXYCSMYMA4M4.png)

### `40128C`：动态分配内存

### `4012BE`：`WriteProcessMemory`向`explorer.exe`写入数据，我需要知道写入了什么数据，这个数据的写入关乎到注入的恶意线程的真面目。于是我找到传入的参数`Buffer`，向前回溯，就找到`main`中的当前目录字符串`"//"`和`Lab12-01.dll`，所以可以确定恶意代码将`Lab12-01.dll`线程注入到`explorer.exe`进程中。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_5WM87QY5PVUZ439.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_XC7NQZRPJYGZQVE.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_QEY6BF6N4XEAW66.png)

到此，我基本上完成了exe文件的反汇编分析，得到结论：这个恶意代码使用进程注入技术，被害进程是`explorer.exe`进程，注入的是`Lab12-01.dll`线程，猜测那个不断弹出的恼人的消息框就是这个线程导致的，因此，我重新进行动态运行，看看是否能够验证这个结论。

## 四、再次动态运行：

双击exe文件，火绒剑中出现`explorer.exe`进程，这次我右键查看这个进程的详细信息，并终止其中的 `Lab12-01.dll`线程 ，果然，那个消息框没有再次弹出。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_Q5Y7H54SXAYC3M2.png)

到这里为止，我确定了被害进程、注入的恶意线程，找到了解决方法，但是还有一点不清楚，就是这个线程除了弹出这个消息框，还有没有其他行为，这个消息框又是个什么东西。于是我决定反汇编分析`Lab12-01.dll`看看。

## 五、IDA分析Lab12-01.dll：

进入`dllmain`，看到调用创建线程，一定是弹出对话框的线程。获取起始地址参数是通过调用`sub_10001030`函数得到的，跳转到这个函数。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_P7MUKDDFCS4YRRS.png)

### `sub_10001030`：

调用函数打印、创建线程、`Sleep`用来计时，其中打印函数`_sprintf`的参数`"Practical Malware Analysis %d"`正是那个对话框的标题。通过动态运行，可以确定，`%d`就是输出一个整数，而这个整数是通过`Sleep`计时获取的，每隔一分钟这个整数加1，并打印出来。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_HAE23RENG9PARVF.png)

创建线程的地址参数是`StartAddress`，跳转过去，终于看到了字符串`"Press OK to reboot"`，正是那个对话框中的文字，这些代码证实了这个对话框弹出后并不会真的重启，而只是每分钟弹出烦人的对话框而已。

![图片描述](https://bbs.kanxue.com/upload/attach/202109/919734_WP929TZGT2A8T2C.png)

## 答案/总结：

1、运行恶意代码可执行文件会发生什么？

每隔一分钟弹出一个对话框。

2、哪个进程会被注入？

`explorer.exe`

3、如何能让恶意代码停止弹出窗口？

法一：重新启动`explorer.exe`进程

法二：终止`explorer.exe`进程中的`Lab12-01.dll`创建的线程

4、这个恶意代码样本是如何工作的？

这个恶意代码通过dll注入，在`explorer.exe`进程中注入`Lab12-01.dll`创建的线程，每隔一分钟弹出一个消息框，还能通过一个计数器，显示已经过去了多少分钟。

### 个人理解：

这个样本的恶意行为实际上只是在不断弹出消息框的过程中让你不胜其烦，在分析过程中十分恼人。但是这个看似无害的功能不容小觑，比如它可以用于恶意的推广营销等用途。

进程注入技术分析起来并不难，这也是恶意代码很常见的行为。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：

- [12-01.rar](javascript:void(0);)

  （27.83kb，38次下载）
