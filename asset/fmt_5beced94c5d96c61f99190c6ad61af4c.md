
```vb
dwXSize As Long  
dwYSize As Long  
dwXCountChars As Long  
dwYCountChars As Long  
dwFillAttribute As Long  
dwFlags As Long  
wShowWindow As Integer  
cbReserved2 As Integer  
lpReserved2 As Long  
hStdInput As Long  
hStdOutput As Long  
hStdError As Long  
End Type
```

我是代码终止线

编写函数：

```vb
Private Function StartAnti() As Boolean  
Dim Huanjing As STARTUPINFO  
GetStartupInfo Huanjing  
If Huanjing.dwX <> 0 Or Huanjing.dwY <> 0 Or Huanjing.dwXCountChars <> 0 Or Huanjing.dwYCountChars <> 0 Or Huanjing.dwFillAttribute <> 0 Or Huanjing.dwXSize <> 0 Or Huanjing.dwYSize <> 0 Then  
StartAnti = True  
Else  
StartAnti = False  
End If  
End Function
```

我是代码终止线

调用如下：

```vb
Private Sub Command1_Click()  
If StartAnti = True Then  
MsgBox "发现调试器,请关闭", , "警告"  
Else  
MsgBox "没有发现调试器", , "通过"  
End If  
End Sub  

Private Sub Form_Load()  
If StartAnti = True Then  
MsgBox "发现调试器,请关闭", , "警告"  
Else  
MsgBox "没有发现调试器", , "通过"  
End If  
End Sub
```

我是代码终止线

好累，第三章第一讲到此结束，我们回顾下，主要讲了以下几个内容：

1、 检测父进程反 RING3 调试器

2、 `FindWindow` 检测敏感窗口

3、 `EnumChildWindows` 枚举所有窗体，检测敏感字符

4、 检测进程名字来排雷

5、 利用 `IsDebuggerPresent` 检测调试器

6、 加密字符串，错误提示诱导陷阱

7、 利用 SEH 进行反跟踪

8、 加 VM 或者 P-CODE 编译

9、 直接反汇编修改代码（破解者用的招数，我们也可以用来防守）

10、 隐蔽性设计

11、 步长、时值反调试（包含启动时间检测）

12、 检测内存分配大小反调试

13、 窗口置顶反调试

14、 检测按键，一般是 F2（断点）、F7（单步步进）、F8（逐过程步过）、F9（运行）

15、 检测前台窗口反调试

16、 `SeDebugPrivilege` 检测调试器

17、 关键部分算法的时候锁定鼠标干扰调试分析

18、 `GetStartupInfo` 反调试

其实方法多种多样，远不只这些，奈何我触及加密解密时间尚短，经验太少，水平太差，暂时无法详细给大家演示其他的反调试手段，大家可以看下《加密解密第三版》，对你也会有启发。

这篇文章写了几天，主要是白天太忙，金工实习，根本没时间写文章，发表得有些晚。

## 第三章 第二讲

### 防静态反汇编分析

这一讲将是简短至极，相信大家也累了，我也写不了多少。

VB 防静态反汇编，有以下途径：

1、 修改程序头，使得一些识别 VB 的反汇编工具无法识别

2、 加壳，最好是保护壳中的猛壳，可以很有效得保护你的资源和代码

3、 虚拟机保护关键过程和代码

4、 将程序作为子程序当作数据形式储存在母程序里（借鸡生蛋），事先先经过 XOR 等的加密，运行母程序的时候 XOR 逆算回子程序数据，然后释放到某个地方执行。

5、 P-CODE 方式编译，然后破坏程序开头等位置，也有一定的效果

6、 大量使用假信息，迷惑静态分析的破解者。

7、 按照高人的话说，似乎可以依靠 VB 内联汇编来添加花指令来抗静态分析。

应该还有更多方法，我知道的太少了，^\_^

好了，《VB 共享软件防破解设计技术初探（三）》就写到这里吧，精力差不多耗尽了。

下一篇《VB 共享软件防破解设计技术初探（四）》我们将讲述：注册码系统（算法部分，核心内容）和加壳防脱壳，估计那篇会比较短些。

还是老规矩，若是有时间，有精力，有空，有能力，我将继续写下去，尽力完成整个系列，给 VB 软件的防破解领域加入催化剂。

作者：爱琴海[SCG] 2008/09/10 （转载请保留该信息）

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | VB 共享软件防破解设计技术初探（三）全部工程文件和代码.rar |
| :--- | :--- |
| | |
