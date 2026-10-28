
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `00020000`  `7DD7186E`  `kernel32.VirtualFree`  `00020004`  `7DD71856`  `kernel32.VirtualAlloc`  `00020008`  `7DD789F1`  `kernel32.SetEnvironmentVariableW`  `0002000C`  `7DD743E2`  `kernel32.GetWindowsDirectoryW`  `00020010`  `7DD979D4`  `kernel32.Wow64GetThreadContext`  `00020014`  `7DD734B0`  `kernel32.GetModuleHandleW`  `00020018`  `7DD74950`  `kernel32.GetModuleFileNameW`  `0002001C`  `7DD77A10`  `kernel32.ExitProcess`  `00020020`  `7DD7103D`  `kernel32.CreateProcessW`  `00020024`  `7DD73F5C`  `kernel32.CreateFileW`  `00020028`  `7DD71410`  `kernel32.CloseHandle`  `0002002C`  `7DD9828E`  `kernel32.lstrcatW`  `00020030`  `00000000`  `00020034`  `7DE90058`  `ntdll_12.ZwResumeThread`  `00020038`  `7DE8FAC8`  `ntdll_12.ZwQueryInformationProcess`  `0002003C`  `7DE8FC40`  `ntdll_12.ZwMapViewOfSection`  `00020040`  `7DE8FF94`  `ASCII` `"窑"`  `00020044`  `7DE8F9D0`  `ntdll_12.ZwClose`  `00020048`  `7DE8FC70`  `ntdll_12.ZwUnmapViewOfSection`  `0002004C`  `7DE8FD6C`  `ntdll_12.ZwDelayExecution`  `00020050`  `00000000`  `00020054`  `00000000`  `00020058`  `00000000`  `0002005C`  `00000000` |

将上面这段数据以`set_name (xxx,"xxx");`的格式进行替换,具体替换方法可以手动替换,也可以正则替换,看各位喜好

替换完的结果:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `set_name  (``0x00020000``,` `"VirtualFree"``);`  `set_name  (``0x00020004``,` `"VirtualAlloc"``);`  `set_name  (``0x00020008``,` `"SetEnvironmentVariableW"``);`  `set_name  (``0x0002000C``,` `"GetWindowsDirectoryW"``);`  `set_name  (``0x00020010``,` `"Wow64GetThreadContext"``);`  `set_name  (``0x00020014``,` `"GetModuleHandleW"``);`  `set_name  (``0x00020018``,` `"GetModuleFileNameW"``);`  `set_name  (``0x0002001C``,` `"ExitProcess"``);`  `set_name  (``0x00020020``,` `"CreateProcessW"``);`  `set_name  (``0x00020024``,` `"CreateFileW"``);`  `set_name  (``0x00020028``,` `"CloseHandle"``);`  `set_name  (``0x0002002C``,` `"lstrcatW"``);`  `set_name  (``0x00020034``,` `"ZwResumeThread"``);`  `set_name  (``0x00020038``,` `"ZwQueryInformationProcess"``);`  `set_name  (``0x0002003C``,` `"ZwMapViewOfSection"``);`  `set_name  (``0x00020044``,` `"ZwClose"``);`  `set_name  (``0x00020048``,` `"ZwUnmapViewOfSection"``);`  `set_name  (``0x0002004C``,` `"ZwDelayExecution"``);`    `/``/``记得别忘了后面的;和地址的``0x`  `否则IDA执行脚本时会报错` |

将这段数据复制到`Bytes_0`函数体中,结果如下

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `static Bytes_0(void) {`  `set_name  (``0x00020000``,` `"VirtualFree"``);`  `set_name  (``0x00020004``,` `"VirtualAlloc"``);`  `set_name  (``0x00020008``,` `"SetEnvironmentVariableW"``);`  `set_name  (``0x0002000C``,` `"GetWindowsDirectoryW"``);`  `set_name  (``0x00020010``,` `"Wow64GetThreadContext"``);`  `set_name  (``0x00020014``,` `"GetModuleHandleW"``);`  `set_name  (``0x00020018``,` `"GetModuleFileNameW"``);`  `set_name  (``0x0002001C``,` `"ExitProcess"``);`  `set_name  (``0x00020020``,` `"CreateProcessW"``);`  `set_name  (``0x00020024``,` `"CreateFileW"``);`  `set_name  (``0x00020028``,` `"CloseHandle"``);`  `set_name  (``0x0002002C``,` `"lstrcatW"``);`  `set_name  (``0x00020034``,` `"ZwResumeThread"``);`  `set_name  (``0x00020038``,` `"ZwQueryInformationProcess"``);`  `set_name  (``0x0002003C``,` `"ZwMapViewOfSection"``);`  `set_name  (``0x00020044``,` `"ZwClose"``);`  `set_name  (``0x00020048``,` `"ZwUnmapViewOfSection"``);`  `set_name  (``0x0002004C``,` `"ZwDelayExecution"``);`  `}` |

保存,退出,回到IDA,按`Alt+F7`弹出脚本选择框

![image-20201030175109800](https://bbs.kanxue.com/upload/attach/202010/867232_WVJSN5BPNDJDEPP.png)

选择`1.idc`,点打开

![image-20201030175135806](https://bbs.kanxue.com/upload/attach/202010/867232_HWFHGFSZ49XJXXY.png)

可以神奇的发现,`20000`开始的地址已经被IDA正确解析,在`200C0`处按`P`,让IDA将其解释为一个函数头部

![image-20201030175250337](https://bbs.kanxue.com/upload/attach/202010/867232_FJ39U9RGXCDU3QG.png)

这样就是我们最终弄好的结果,IDA可以正确解析这段代码,我们看的也会特别清晰

# 0x5.第二层代码分析

按下`F5`观察伪源码

![image-20201030175557342](https://bbs.kanxue.com/upload/attach/202010/867232_8KU9ZKJZP4T876T.png)

这段代码就不详细说明了,大体逻辑就是判断系统位数,取出对应的系统文件,注入恶意代码,运行系统文件

我们的重点在注入到系统文件中的那部分代码是什么,这里我们不分析第二层代码,采用一个更加便捷的方式

# 0x6.第三层代码提取

我们此处依然采取dump 的方式进行提取,直接运行样本,不做任何阻拦,样本会创建一个被注入了恶意代码的`svchost.exe`进程,我们火绒剑选择`svchost.exe`,点击下方内存列表,右击`0x20000`这块内存,选择`内存转储`

PS:若火绒剑无法保存,可以尝试使用其他工具进行保存,需要注意的是`svchost`为64位程序,`OD`是用不了的,此处推荐`PCHunter64`

拷贝出来的dump文件才用相同方式拖入IDA,更改基址,批量命名,此处省略

# 0x7.第三层代码分析

如果操作没有错误,IDA解析后的代码应该如下:

![image-20201030180540335](https://bbs.kanxue.com/upload/attach/202010/867232_YHUATFKVRKPJ6QY.png)

![image-20201030180612608](https://bbs.kanxue.com/upload/attach/202010/867232_X2T5F5VGBNZAQQN.png)

![image-20201030180627837](https://bbs.kanxue.com/upload/attach/202010/867232_MDWJY2HR2AT7W5S.png)

![image-20201030180646252](https://bbs.kanxue.com/upload/attach/202010/867232_23ZVPXGWTYSJRMR.png)

到了这里样本的最终行为已经毫无遮拦了,我们甚至不用进入`F5`分析逻辑,只需简单的观察字符串即可分析出这个样本的大致行为逻辑

- ![image-20201030180742937](https://bbs.kanxue.com/upload/attach/202010/867232_HH5UXRX88XD3SMB.png)

![image-20201030180753289](https://bbs.kanxue.com/upload/attach/202010/867232_BY2EHQBYXV7U4G9.png)

注册表操作API+启动项目录字符串 = 添加开机自启

- ![image-20201030180849816](https://bbs.kanxue.com/upload/attach/202010/867232_PY38UNQH8SP2JXA.png)

网络操作相关API用于发送请求,可能是下载一个恶意程序亦或是上传一段用户隐私数据

# 总结

整篇文章对于代码的分析很少,各位如果想要细致分析可以自行抠代码,没有了保护,所有写法已经是透明的了,因此继续逆向只是个体力活,我这里就不演示了

主要是给初学的小白演示下这种多层代码释放嵌套的样本怎么处理,已经IDA的强大功能

择优吸收

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-10-30 18:31

被SSH山水画编辑

，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)
