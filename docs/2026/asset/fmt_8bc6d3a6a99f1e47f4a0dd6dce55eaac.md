

# 原创 Hack The Box - TIER 2 - Archetype Oopsie Vaccine Unified

本阶段不同于之前的简单操作，到这里我才明白，那些问答并非先问答再渗透，而是每完成一步渗透就回答相应问题。此过程涉及许多工具、脚本、提权、端口等知识点，收获丰富。

## Archetype

> TASK 1
>
> Which TCP port is hosting a database server?
>
> 哪个 TCP 端口托管数据库服务器？
>
> 答案：1433

> TASK 2
>
> What is the name of the non-Administrative share available over SMB?
>
> SMB 上可用的非管理共享的名称是什么？
>
> 答案：backups

> TASK 3
>
> What is the password identified in the file on the SMB share?
>
> SMB 共享文件中标识的密码是什么？
>
> 答案：M3g4c0rp123

> TASK 4
>
> What script from Impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server?
>
> 可以使用 Impacket 集合中的哪些脚本来建立到 Microsoft SQL Server 的经过身份验证的连接？
>
> 答案：mssqlclient.py

> TASK 5
>
> What extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell?
>
> 可以使用 Microsoft SQL Server 的哪些扩展存储过程来生成 Windows 命令外壳？
>
> 答案：xp\_cmdshell

> TASK 6
>
> What script can be used in order to search possible paths to escalate privileges on Windows hosts?
>
> 可以使用什么脚本来搜索提升 Windows 主机权限的可能路径？
>
> 答案：winpeas

> TASK 7
>
> What file contains the administrator's password?
>
> 哪个文件包含管理员的密码？
>
> 答案：ConsoleHost\_history.txt

先使用`nmap`扫描：

![](upload/attach/202203/905780\_QQPVQDCZVJ4R2KN.jpg)

看到139端口和445端口就知道是SMB了，同时1433端口也是敏感端口，对应`mssql`。这里先从SMB下手，使用`-L`命令列出共享文件夹：

![](upload/attach/202203/905780\_2PKEV5JUAGFH5YV.jpg)

带美元符号的是管理员权限，这里使用`-N`符号连接`backups`，发现有一个`dtsConfig`文件，此文件是SSIS的配置文件（SSIS 即 SQL Server Integration Services），先将其`get`下来：

![](upload/attach/202203/905780\_DU45XDPVFKNKPGF.jpg)

查看内容，其中包含了SQL服务的账户密码：

![](upload/attach/202203/905780\_A2AD6AM2VPWUDPQ.jpg)

既然获得了账户密码，使用Kali自带的`mssql`客户端`impacket-mssqlclient`连接靶机上的SQL服务：

![](upload/attach/202203/905780\_UC77H5QG5EPQ7CD.jpg)

输入`help`指令显示可用命令，这里的`enable\_xp\_cmdshell`是允许启用系统cmd命令行的指令。`xp\_cmdshell {cmd}`用于执行cmd命令。先允许启用命令行，然后根据终端提示输入重置命令：

![](upload/attach/202203/905780\_4ZQNCZ3R2WPJX86.jpg)

这里尝试cmd能否使用，并测试是否有`nc`命令，如果有，就可以通过反弹shell建立持续性的shell：

![](upload/attach/202203/905780\_BVF46W8WY279E7H.jpg)

很可惜没有`nc`，但cmd命令行可用，于是尝试传输一个`nc`程序过去。提前在Kali中存放一个`nc.exe`，然后开启HTTP服务，在靶机中使用`wget`命令远程下载Kali里的文件。注意，需要提前将启动HTTP服务的终端进入存有`nc.exe`的文件夹，这样HTTP服务的根目录就是该文件夹，下载`nc`也更方便。

先查看Kali IP：

![](upload/attach/202203/905780\_96ENTPKZ5227QVW.jpg)

Kali开启HTTP服务：

![](upload/attach/202203/905780\_UCMYH6JVHAGRU3X.jpg)

靶机下载Kali里的`nc`文件，注意下载的目标文件夹，其他文件夹可能因权限不足而无法写入：

![](upload/attach/202203/905780\_JQBRGPR4JD57T7P.jpg)

Kali这边也显示文件已被获取：

![](upload/attach/202203/905780\_YJK8HQ54T6EYYDE.jpg)

此时进行反弹shell，Kali开启监听，靶机主动连接Kali：

![](upload/attach/202203/905780\_27S8F7ZGY5FCBJK.jpg)

Kali已获得靶机的shell：

![](upload/attach/202203/905780\_EUWXJK6RJCPUSW6.jpg)

Windows列文件使用`dir`，经过一番查找后，flag位于此用户桌面上：

![](upload/attach/202203/905780\_95CZGSDR2NYVT5V.jpg)

到这一步以为结束了，没想到还有一问，管理员账户中还有另一个flag，而当前获取的是普通用户`sql_svc`的shell，需要提权。Windows提权方面，通过资料了解到可使用一款名为`winPEAS`的工具，它会检索整个系统路径，有可利用性的路径或文件会以红色文字标出。提前准备`winPEAS`，靶机下载后在已连上的shell中直接运行：

![](upload/attach/202203/905780\_AHFDABXY9BRJE7J.jpg)

运行需要一段时间，因为它在检索整个系统，结束后会标出可利用文件路径：

![](upload/attach/202203/905780\_PYD4CPVYTUA5WUC.jpg)

其中`ConsoleHost\_history.txt`记录了控制台命令历史，查看一下：

![](upload/attach/202203/905780\_NJGUDZV8K8AQB8S.jpg)

里面记录了管理员账号与密码，Kali尝试使用`psexec.py`脚本连接。`psexec` 是 Windows 下非常好用的远程命令行工具，使用它不需要对方主机开启3389端口，只需对方开启`admin$`共享（该共享默认开启）。但如果目标主机开启了防火墙，`psexec` 也无法使用，会提示找不到网络路径。由于 `psexec` 是 Windows 提供的工具，杀毒软件会将其加入白名单。

![](upload/attach/202203/905780\_5UJ5JDPHVJWECGE.jpg)

一番搜索后，管理员flag也在桌面：

![](upload/attach/202203/905780\_PGFQFV4H3BV28A3.jpg)

这台靶机质量不错，收获很多。

## Oopsie

HTB是先做题再回答相关问题，所以后面先写渗透靶机过程，最后写问题答案。

这个靶机也很有趣。首先`nmap`扫描，探测到22端口和80端口，但SSH不知道密码无法连接，所以转向网页端。

在网页底部发现如下提示：

![](upload/attach/202203/905780\_GTPG8W5HTF9958U.jpg)

需要登录才能访问，因此方向转为登录这个站。登录点在哪里？我是通过查看源代码发现的：

![](upload/attach/202203/905780\_NHJVF46HSVMM4EF.jpg)

在末尾加载的JS脚本处发现了登录脚本，打开这个`login`路径试试：

![](upload/attach/202203/905780\_XBZSQ8QQ263R8DM.jpg)

成功找到登录页面，但尝试许多弱口令均失败，注入也失败。登录页面有一个`Login as Guest`，于是以游客身份登录看看：

![](upload/attach/202203/905780\_N34UF9YMXSVGDHV.jpg)

随便点击，（渗透就是要多点点）在Account页面发现URL中参数`content`传递了`account`，`id`传递了`2`：

![](upload/attach/202203/905780\_QYWYVPX8SFGYFAX.jpg)

`account`就是当前页面，`id=2`应该表示当前用户。试着将`id`改为`1`：

![](upload/attach/202203/905780\_RZ4JJZKA2V552HA.jpg)

居然切换到了admin用户，这算是未授权访问还是垂直越权？但没什么用，此页面没有更多可利用点。其他页面仍无亮点，Uploads页面的上传功能是唯一可利用点，但上传权限受限，需要越权。在这里发现了Cookie：

![](upload/attach/202203/905780\_BT7N8VDRDRYTB6J.jpg)

将`role`值改为`admin`，`user`值改为之前admin的`access id`：

![](upload/attach/202203/905780\_6PUGXSREUVTBTJN.jpg)

成功越权上传。这里使用Kali自带的PHP shell，由于不太了解Kali的Web shell工具，所以采用反弹shell方式，把shell复制过来一份并修改为自己的IP和监听端口：

![](upload/attach/202203/905780\_5WYA7SSP8CHY54U.jpg)

上传成功，终端开启监听：

![](upload/attach/202203/905780\_M5UFZ9EAHC45MHM.jpg)

现在需要访问这个shell文件，它才会运行代码，但我们不知道上传路径。使用`gobuster`工具爆破路径。Kali中没有，执行`apt-get install gobuster`安装即可。`gobuster`有两种模式：爆破目录和DNS子域名，因此需先选择`dir`模式。`--url`指定URL，`--w`指定爆破字典，`-x`指定检索的扩展名（可选）。这里使用`dirb`的小字典：

![](upload/attach/202203/905780\_R5K8ZK7CPXRPVQ9.jpg)

严查`uploads`，访问此目录没有权限，那就尝试访问此目录中的`php-reverse-shell.php`：

![](upload/attach/202203/905780\_74QQR2QNCX6FFSV.jpg)

Shell弹回来了，但`www-data`权限太低，且终端不好用。这里运行Python代码打开`/bin/bash`，并查看`/etc/passwd`文件：

![](upload/attach/202203/905780\_626VYZHEA3BHPK5.jpg)

发现用户`robert`，前往`robert`目录查看：

![](upload/attach/202203/905780\_ZGZJN4FHJ9ED2GE.jpg)

得到一个flag，不出所料还应有其他flag，需要提权才能获取。

现在去登录文件夹寻找线索：

![](upload/attach/202203/905780\_3TB2775B64WTBQG.jpg)

经过一番`cat`，在`db.php`中找到`robert`的账号密码：

![](upload/attach/202203/905780\_4VADUF6F3Z9K4S5.jpg)

切换到`robert`用户，并查看有哪些可用命令：

![](upload/attach/202203/905780\_Z8PMRT53G5CVBWG.jpg)

看起来用户组权限不够，查看一下所属组：

![](upload/attach/202203/905780\_93GQYJ9PBFXDY9W.jpg)

前面提到`robert`属于`bugtracker`组，查找`bugtracker`，发现是一个文件，查看权限并输出其文件类型：

![](upload/attach/202203/905780\_VFK7TH4C2CN3HPK.jpg)

该文件可执行，运行试试：

![](upload/attach/202203/905780\_FWHXDZAHMHJC65V.jpg)

这个程序会`cat`一个路径并将内容输出，不存在的编号会显示无文件。可以利用这个点进行提权。

**思路**：`bugtracker`本无权进入`root`目录，但使用了setid，使其暂时拥有root权限并调用`cat`命令。直接调用`cat`时，调用的是环境变量中的`cat`。我们可以构造一个恶意的`cat`命令，并将该命令所在目录设置为环境变量，这样`bugtracker`调用的就是恶意`cat`。具体命令如下：

```bash
export PATH=/tmp:$PATH //将 /tmp 目录设置为环境变量
cd /tmp/ //切换到 /tmp 目录下
echo '/bin/sh' > cat //在此构造恶意的 cat 命令
chmod +x cat //赋予执行权限
```

然后调用`bugtracker`，输入任意ID都会直接调用root用户的shell：

![](upload/attach/202203/905780\_WVM5UEVTFV7DYH6.jpg)

由于我们修改了环境变量，调用的`cat`是恶意的，所以无法查看文件。这里使用`tac`命令查看（`tac`是将行数倒着输出，而不是将每个字符倒序输出），直接`tac`查看即可：

![](upload/attach/202203/905780\_3J855SGZPVBZRZP.jpg)

渗透结束，回答问题。但并非所有渗透过程都需要问题中提到的方法。

> TASK 1
>
> With what kind of tool can intercept web traffic?
>
> 用什么样的工具可以拦截网络流量？
>
> 答案：proxy

> TASK 2
>
> What is the path to the directory on the webserver that returns a login page?
>
> 网络服务器上返回登录页面的目录的路径是什么？
>
> 答案：/cdn-cgi/login

> TASK 3
>
> What can be modified in Firefox to get access to the upload page?
>
> 可以在 Firefox 中修改哪些内容以访问上传页面？
>
> 答案：cookie

> TASK 4
>
> What is the access ID of the admin user?
>
> admin 用户的访问 ID 是什么？
>
> 答案：34322

> TASK 5
>
> On uploading a file, what directory does that file appear in on the server?
>
> 上传文件时，该文件出现在服务器上的哪个目录中？
>
> 答案：/uploads

> TASK 6
>
> What is the file that contains the password that is shared with the robert user?
>
> 包含与 robert 用户共享的密码的文件是什么？
>
> 答案：db.php

> TASK 7
>
> What executible is run with the option "-group bugtracker" to identify all files owned by the bugtracker group?
>
> 使用“-group bugtracker”选项运行什么可执行文件来识别 bugtracker 组拥有的所有文件？
>
> 答案：find

> TASK 8
>
> Regardless of which user starts running the bugtracker executable, what's user privileges will use to run?
>
> 无论哪个用户开始运行 bugtracker 可执行文件，运行时使用的用户权限是什么？
>
> 答案：root

> TASK 9
>
> What SUID stands for?
>
> SUID 代表什么？
>
> 答案：Set Owner User Id

> TASK 10
>
> What is the name of the executable being called in an insecure manner?
>
> 以不安全方式调用的可执行文件的名称是什么？
>
> 答案：cat

## Vaccine

`nmap`扫描结果如下：

![](upload/attach/202203/905780\_Y6S48K3EKAWPMZW.jpg)

`nmap`会显示FTP是否允许匿名用户登录，图中FTP部分也有提示，因此使用账户`anonymous`登录，密码随意。

