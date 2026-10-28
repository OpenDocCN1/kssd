
## 0x00 MuddyWater简介

`MuddyWater`（又称为污水,T-APT-14）是疑似来自**伊朗**的APT组织，**主要攻击目标**为**中东地区政府机构**，但在近期的公开报告中显示，18年后，中东以外的地区也陆续出现了`MuddyWater`的活动迹象，比如土耳其，巴基斯坦等地。

该组织最早于`2017年9月`被`MalwareBytes`公开披露，同年`11月paloalto`披露针对中东的攻击，并将该组织命名为`MuddyWater`，之后活动踪迹开始被国内外安全机构公开披露。

`MuddyWater`使用诱惑性的鱼叉文档作为攻击入口，引导用户启用宏，利用宏释放后续后续Payload进行攻击，在早期报道中，`MuddyWater`主要使用**powershell后门**。其利用`VBS`,`VBA`, `PowerShell`,`Python`,c#等脚本后门以及RAT木马。

`BlackWater`活动的主要发起者,自从`BlackWater`活动之后`MuddyWater` APT组织开始不断优化和调整自身的攻击手法,例如开始不再广泛采用**模糊化背景**作为诱饵文档,而是做一个假的提示信息图片来遮挡文档正文内容.在恶意宏里面不再通过启动项直接启动`powershell`载荷,而是使用一个**硬编码**处理后的`VBE`文件,将`VBE文件`写入**自启动项**,或者将其放入**启动文件夹**中,在`VBE`文件中重新释放`PowerShell`载荷以进行**侦查和后门**的活动。

## 0x01 样本分析

### 第一阶段:诱饵文档

#### 样本信息:

| 样本信息 | Zakupki_Agency on Public Procurement |

| --- | --- |

| 样本MD5 | 6cb076f1f42573c5c43083a89bcfe442 |

| 样本SHA-1 | 4e68e2040acade6369d938f96c95ada050a8e061 |

| 样本SHA-256 | 36ccae4dffc70249c79cd3156de1cd238af8f7a3e47dc90a1c3347 |

| 样本类型 | MS Word Document |

| 样本文件名 | Zakupki_Agency on Public Procurement.doc |

| 文件大小 | 408576字节 |

| 作者 | win7 |

| 创建文档内容时间 | 2019/5/23 12:00 |

| 最后一次保存时间 | 2019/5/23 12:09 |

诱饵文档截图如下:

![1](https://bbs.kanxue.com/upload/attach/201908/780480_WT8KSYVUAJXB2BA.jpg)

![2](https://bbs.kanxue.com/upload/attach/201908/780480_DSVX2HVEAX79ZBR.jpg)

![3](https://bbs.kanxue.com/upload/attach/201908/780480_F9JAUX46XNQE7Y3.jpg)

文档中的语言为**塔吉克语**,通过内容以及诱饵文档的名称可以大致推断出其目标针对的是**塔吉克斯坦政府的采购部门**

![4](https://bbs.kanxue.com/upload/attach/201908/780480_DNCFFYHMU5ZFZUH.jpg)

![5](https://bbs.kanxue.com/upload/attach/201908/780480_4KHAS8DC5HJWQV6.jpg)

其文档的图标正是**塔吉克斯塔政府**的图标,故此`MuddyWater`这次攻击的目标基本上确定

![6](https://bbs.kanxue.com/upload/attach/201908/780480_NNMMDG5R6TC7JPN.jpg)

### 第二阶段:恶意宏代码:

#### 1.Document_Open函数

`Document_Open`函数:`frmLoader.show` 加载并显示宏项目里内置的窗口

![6](https://bbs.kanxue.com/upload/attach/201908/780480_52CPQZS3SWKW8EA.jpg)

笔者猜想应该是在窗口里要么通过点击按钮进行载荷植入要么通过关闭窗口进行载荷植入

所以我们顺藤摸瓜定位到窗口处理的函数:

#### 2.按钮控件消息相应函数

函数`buttonQWERTY_Click()`调用函数`RgSh(id)`并传入参数7

**frmVisualizer.Show** 加载`frmVisualizer`窗口并弹出

**Unload Me** 卸载本窗口

![7](https://bbs.kanxue.com/upload/attach/201908/780480_69AKR8RGZ9J7E87.jpg)

#### 3.RgSh(id)函数

首先会调用`PuSH(id)`函数并且传入参数id 依照前面的调用 传入的参数为7

然后让变量b为`PuSH(id)`函数的返回值

我们会在下文对这个`PuSH(id)`函数进行分析 这里如果传入参数7的话 变量b为:`%Temp%\aulmgr.vbe`

通过`CallByName`函数调用`SetStingValue`修改注册表键值

设置的注册表项为:`"Software\Microsoft\Windows\CurrentVersion\Run"`

设置的键名称为: `"VingValue"`

`VingValue`键的键值为变量b 也就是`%Temp%\aulmgr.vbe`

![8](https://bbs.kanxue.com/upload/attach/201908/780480_CV3ZKUUUDPWK99Q.jpg)

#### 4.PuSH(id)函数

```
p=d3r(4)&fun(1)
```

这里面引入了两个新的函数,一个为`d3r(id)`,另一个为`fun(id)`

我们会在下文对这个`d3r(id)`和`fun(id)`两个函数进行分析

`d3r(4) & fun(1)` 返回的路径是`%Temp%\aulmgr.vbe`

```
If id mod 5 = 1 then
```

如传入`Push(id)`函数的**id与5取余等于1**则进行下面的写`vbe` 文件操作——`%Temp%\aulmgr.vbe`

利用`CreateTexFiile`创造`%temp%\aulmgr.vbe`文件

读取`frmloadr`窗口中`label控件`(1到13)的`caption(标题)`字符串

并按照从1到13的顺序拼接，存入`ddd`变量中

将`ddd`变量内容写入`vbe文件`中——`%Temp%\aulmgr.vbe`

![9](https://bbs.kanxue.com/upload/attach/201908/780480_3SS39ATDKRTQ4NR.jpg)

#### 5.d3r(id)函数

```
set f = createobject(fisa())
```

这个`fisa()`函数后面会说到 但是归根结底其实就是返回`scripting.filesystemobject`这个字符串

```
select case (id mod 5)
```

用**传入参数的id数字与5进行取余**,其实就是id的数字,因为case的方案里面,最高的数字为5

```
Set f = CreateObject("scripting.filesystemobject")

Case 0

    d3r = Environ$("puBLiC") '%public% Pulic环境变量路径

Case 1

    d3r = Environ$("aPpDaTa") '%AppData% AppData环境变量路径

Case 2

    d3r = f.getspecialfolder(0) '%SystemRoot% Windows文件夹

Case 3

    d3r = f.getspecialfolder(1) '%SystemRoot%\System32 系统文件夹

Case 4

d3r = f.getspecialfolder(2) '%Temp% 临时目录

Case Else

d3r = ""    '返回空值
```

![10](https://bbs.kanxue.com/upload/attach/201908/780480_8G2G2CWY69FQBY2.jpg)

#### 6.Fun(id)函数

```
select case (id mod 5)
```

用**传入参数的id数字与5进行取余**,其实就是id的数字,因为case的方案里面,因为最高的数字为5

```
Case 0

FuN = "\aaaaa.tmp"

Case 1

FuN = "\aulngr.vbe"

Case 2

FuN = "\bbbbbb.tmp"

Case 3

FuN = "\report.xls"

Case 4

FuN = "\report.docx"

Case Else

FuN = ""   '返回空值
```

![11](https://bbs.kanxue.com/upload/attach/201908/780480_SAFUVGBDBBZ5U27.jpg)

那我们回到`PuSH(id)`函数中

```
p=d3r(4)&fun(1) '%temp%\aulmgr.vbe
```

#### 7.FisA()函数

通过读取`frmlodr`窗口中得`label`控件(14,15)以拼接好字符串`scripting.filesystemobject`

并且将这段字符串返回

![12](https://bbs.kanxue.com/upload/attach/201908/780480_UXWVBBZP8VH2NK3.jpg)

至此宏代码中的主要函数功能已经分析完毕

其最主要的功能为:**释放VBE Loader于%temp%\aulmgr.vbe下,将该文件通过修改注册表写入开机启动**

### 第三阶段:VBE loader

#### 1.基本信息:

| 样本信息 | %temp%\aulmgr.vbe |

| --- | --- |

| 样本MD5 | 9d2da5228e21594ab46b4f5281d38b8f |

| 样本SHA-1 | 2b548816eeacd4fd4dee30f46acaecb34b1238e3 |

| 样本SHA-256 | ac32b57f7d14e8e50db4dabbbefc67f10fb71994fbd7165c8dd634724cb75384 |

| 文件类型 | VBE文件 |

| 文件大小 | 58962字节 |

| 样本文件名 | aulmgr.vbe |

样本截图如下(挺乱码的):

![13](https://bbs.kanxue.com/upload/attach/201908/780480_S74Z5WWVJ7REYEJ.jpg)

#### 2.VBE loader分析

用**scrdec**或者[Decode VBE](https://bbs.kanxue.com/elink@fcaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5K9h3c8A6k6i4u0e0N6r3g2$3k6h3&6K6i4K6u0r3c8r3W2V1K9h3g2J5f1%4c8W2N6X3g2F1M7#2y4#2K9i4c8W2i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8W2j5$3!0V1k6g2)9J5k6s2k6T1k6g2)9J5k6i4m8&6)工具进行解密

摘取主要的VBS代码进行分析:

如图:VBE文件内置的b字符串(`pwershell第一阶段的代码`)

![14](https://bbs.kanxue.com/upload/attach/201908/780480_TUWT2BNPJDWT9NY.jpg)

其主要内容为提取VBE文件中自带的**Powershell第一阶段**的代码,然后通过`OpenTextFile`函数创建**Powershell第一阶段**载荷于`%public%\\UserImage.png`,并且将`B`变量写入

然后执行命令`powershell /w 1 iex(gc $env:public'\userimage.png')`

```
set fs = CreateObject("Scripting.FileSystemObject")'fs为Scripting.FileSystemObject的对象

Set ts = fs.OpenTextFile(wshShell.ExpandEnvironmentStrings("%public%") &"\UserImage.png", 2, True, -2)'开始就是释放powershell 第一阶段载荷的部分

'路径为%public%\UserImage.png

ts.write(b) '将b的内容写入这个png文件中

wshshell.run("powershell /w 1 iex(gc $env:punlic'\userimage.png')")

'执行命令 powershell /w 1 iex(gc $env:public'\userimage.png')
```

PowerShell命令如下,窗口隐藏通过**gc(Get-Content)**命令`%public%\\UserImage.png`

的数据,然后利用**iex(Invoke-Expression)**命令,内存执行`UserImage.png`中的数据

```
/w : windowstyle #窗口风格

1 : hidden #隐藏

iex : Invoke-Expression #运行命令和表达式

gc : Get–Content #获取指点位置的数据

$env:public #%public%
```

![15](https://bbs.kanxue.com/upload/attach/201908/780480_Z3X6V6N7SYQAAVZ.jpg)

### 第四阶段:Recon PowerShell

#### 1.样本信息:

| 样本信息 | %public%\UserImage.png |

| --- | --- |

| 样本MD5 | 34a072f42905f9de31523616e8f207b1 |

| 样本SHA-1 | 3d37affb081e7c9586bdac2fe9e45368f812ceb4 |

| 样本SHA-256 | 93277fff51fe11690b91b21d3bd5b11979cdc56a32df57f722009cb6cea5cb6e |

| 样本文件类型 | Poweshell 脚本(ps1) 虚假的PNG文件 |

| 样本文件大小 | 54605 字节 |

| 样本文件名 | UserImage.png |

#### 2.Recon Powershell分析

**注意:笔者并没有去掉变量名和函数名混淆,所以其会有一些难懂,请结合图文进行理解**

##### 第一阶段 Recon Powershell:

将**Recon PowerShell**的代码经过`Base64`编码后存入变量`vv`

利用`[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String`解密变量`vv`

并且将结果存入变量`er`中

利用`gcm i*v*p*s*n`拼凑`Invoke-Expression`,将变量`er`传入,内存执行`Recon PowerShell`第二阶段

如图所示:利用**gcm(Get-Command)**模糊拼凑`Invoke-Expression`命令

![16](https://bbs.kanxue.com/upload/attach/201908/780480_DAQAN7APJ3JP9FJ.jpg)

```
$er = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($vv)) #decode Recon PowerShell

& (gcm i*v*p*s*n) $er #Use IEX to run the payload
```

![17](https://bbs.kanxue.com/upload/attach/201908/780480_JQJ5GJFA8T6UQ68.jpg)

##### 第二阶段 Recon PowerShell:

**功能1:收集系统信息**

1. 获取**当前用户名称** 如下图 函数名:`lTXxzy53`

   通过获取环境变量`%username%`来获取当前用户名称，如果失败返回`error username`字符串

   ![18](https://bbs.kanxue.com/upload/attach/201908/780480_MR4X5RBRM5STDYN.jpg)

2. 获取**用户所在域的名称** 如下图 函数名:`EdA4VNZrBAU37Pd7ND`

   通过`%userdomain%`环境变量获取用户所在域的名称,如果失败反而`error userdomain`字符串

   ![19](https://bbs.kanxue.com/upload/attach/201908/780480_3K73YBTV64Z6MUP.jpg)

3. **遍历进程** 如下图 函数名:`zokOsMYYZQG1GyNMGDZ`

   利用`ps命令(Process Status)`枚举当前用户的进程信息,如果失败则返回`error tasklist`字符串

   ![20](https://bbs.kanxue.com/upload/attach/201908/780480_YYUPWEH6ARRAU6Z.jpg)
