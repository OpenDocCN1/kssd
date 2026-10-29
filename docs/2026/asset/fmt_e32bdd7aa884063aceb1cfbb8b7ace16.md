
```text
─# smbmap -H 10.129.157.43 -u null
[+] Guest session       IP: 10.129.157.43:445
Name: 10.129.157.43
Disk                                                    Permissions Comment
----                                                    ----------- --------
ADMIN$                                                  NO ACCESS   Remote Admin
C$                                                      NO ACCESS   Default share
Data                                                    READ ONLY
IPC$                                                    NO ACCESS   Remote IPC
Secure$                                                 NO ACCESS
Users                                                   READ ONLY
```

`user` 和 `data` 是可以访问的。

访问 `user`，发现不能执行任何命令

```text
└─# smbclient -N //10.129.157.43/users
Try "help" to get a list of possible commands.
smb: \> dir
.                                   D        0  Sat Jan 25 18:04:21 2020
..                                  D        0  Sat Jan 25 18:04:21 2020
Administrator                       D        0  Fri Aug  9 11:08:23 2019
C.Smith                             D        0  Sun Jan 26 02:21:44 2020
L.Frost                             D        0  Thu Aug  8 13:03:01 2019
R.Thompson                          D        0  Thu Aug  8 13:02:50 2019
TempUser                            D        0  Wed Aug  7 18:55:56 2019
```

TempUser C.Smith L.Frost R.Thompson Administrator 获得了这么几个用户

但是我无法访问到任何资源文件

![image](https://bbs.kanxue.com/upload/attach/202301/784203_UU55SSPEADDQBZS.png)

访问 `data`

```text
└─# smbclient -N //10.129.157.43/data
Try "help" to get a list of possible commands.
smb: \> ls
.                                   D        0  Wed Aug  7 18:53:46 2019
..                                  D        0  Wed Aug  7 18:53:46 2019
IT                                  D        0  Wed Aug  7 18:58:07 2019
Production                          D        0  Mon Aug  5 17:53:38 2019
Reports                             D        0  Mon Aug  5 17:53:44 2019
Shared                              D        0  Wed Aug  7 15:07:51 2019

    5242623 blocks of size 4096. 1840690 blocks available
smb: \>
```

![image](https://bbs.kanxue.com/upload/attach/202301/784203_V8PUWQ29SC8MZYB.png)

对目录逐一浏览之后，发现有目录可以访问了。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_FEESB4HFBGPEWV3.png)

接下来逐级访问

![image](https://bbs.kanxue.com/upload/attach/202301/784203_7CDP4BVK4MVKDJW.png)

这里发现一个 txt，下载的命令需要用“”包起来，要不然会如上图所示找不到命令。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_JE339YGZKC2YA4R.png)

![image](https://bbs.kanxue.com/upload/attach/202301/784203_ZMGMRK3R6Y7SFEU.png)

这是一份邮件，意味着此账户和密码可以访问 Tempuser 的共享服务了。

### TEMP

```text
└─# smbmap -H 10.129.157.43 -u TempUser -p welcome2019
[+] IP: 10.129.157.43:445
Name: 10.129.157.43
Disk                                                    Permissions Comment
----                                                    ----------- --------
ADMIN$                                                  NO ACCESS   Remote Admin
C$                                                      NO ACCESS   Default share
Data                                                    READ ONLY
IPC$                                                    NO ACCESS   Remote IPC
Secure$                                                 READ ONLY
Users                                                   READ ONLY
```

此账户可以访问 `data` 和 `secure` 还有 `users`。

但是 `users` 依然没有权限访问，继续访问 `data`，发现比较之前多了一个 `IT` 文件夹可以访问了。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_U7G8SVSYDJYDD9T.png)

![image](https://bbs.kanxue.com/upload/attach/202301/784203_9CU4Z9GS55M8HFW.png)

浏览到 `DLink` 目录没有任何的配置文件

在 `notepad` 文件夹里面发现了配置文件，导出。

在 `RU Scanner` 目录也发现了配置文件，导出。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_KYQBQFRRS9J2PNA.png)

#### `RU_config.xml`

```xml
<Username>c.smith</Username>
<Password>fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=</Password>
</ConfigFile>
```

获得了一个用户 c.smith 的密码，但是密文不知道是什么类型，也解密不了。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_NKXMMGG6SVU3U79.png)

另外一份 `notepad` 的配置文件中给出了两个地址。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_Q7PBJPHGGT8AY2S.png)

其中一个指向了 `secure` 目录。重新回到 SMB 去读一下这个共享。

```text
└─# smbclient -U TempUser //10.129.157.43/Secure$
Password for [WORKGROUP\TempUser]:
Try "help" to get a list of possible commands.
smb: \> ls
.                                   D        0  Wed Aug  7 19:08:12 2019
..                                  D        0  Wed Aug  7 19:08:12 2019
Finance                             D        0  Wed Aug  7 15:40:13 2019
HR                                  D        0  Wed Aug  7 19:08:11 2019
IT                                  D        0  Thu Aug  8 06:59:25 2019

    5242623 blocks of size 4096. 1839742 blocks available
smb: \> cd IT
smb: \IT\> ls
NT_STATUS_ACCESS_DENIED listing \IT\*
smb: \IT\>
```

刚开始就是显示这样权限不足，结果根据文件的指引我能访问 `carl` 目录。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_N6PAD6MEEWTUNZX.png)

刚开始看题目包含了 `C#` 的标签，凭感觉应该是要进到 `VB` 目录去。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_QTU7X4DE4TYQZG4.png)

在 `RU` 的目录发现了刚开始那个 `RUscan` 目录和源代码，我们之前已经获得了他的配置文件。

现在需要下载回来他的所有文件，`smbclient` 的递归下载是这样的：

![image](https://bbs.kanxue.com/upload/attach/202301/784203_543CWKDESWPZSAD.png)

## 代码分析

### 源码

导入 VS 丢丢之后，在 `Module1` 可以看到是加载了配置文件然后引用 `utils` 中的 `DecryptString` 函数进行加解密。

![image](https://bbs.kanxue.com/upload/attach/202301/784203_GH8UCVU867YSPVK.png)

其中 `utils` 的代码如下：
