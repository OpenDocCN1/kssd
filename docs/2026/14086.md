
# [原创]练习1个Kimsuky样本

## 一、ATP简介

Kimsuky是被认为来自朝鲜的APT组织。该组织善于利用复杂的东北亚地缘政治局势信息作为诱饵，主要针对韩国政府及韩美智库等机构进行钓鱼攻击。

## 二、概述

初始样本是一个伪装成PDF的EXE程序，运行后释放诱饵文档`xxx.hwp`（韩文文档）以及另一样本`Icon.pif`，随后运行`Icon.pif`并传入参数`iDmzQtvReUSCdTn`：写入注册表使样本能自启动。随后复制自身到`C:\\ProgramData\\system32`目录下，修改名字为`smss.exe`并传入参数`zWbTLWgKymXMDwZ`，样本运行后会请求`movie[.]youtoboo.kro.kr`等待指令传入。

## 三、行为分析

开启监控程序，运行木马，样本行为：

1. 设置4个注册表键值；
2. 创建文件`C:\ProgramData\210927 xxx.hwp`
3. 运行`C:\ProgramData\Icon.pif iDmzQtvReUSCdTn`和`C:\\ProgramData\\system32\\smss.exe zWbTLWgKymXMDwZ`

![](https://bbs.kanxue.com/upload/attach/202206/952303_W68WVR99ZMBUNAK.jpg)

图 4个注册表键值设置

![](https://bbs.kanxue.com/upload/attach/202206/952303_U2ZR46KSP93ZEPY.jpg)

图 文件创建

![](https://bbs.kanxue.com/upload/attach/202206/952303_YYSNFQ6AGN687QD.jpg)

图 Icon.pif样本

![](https://bbs.kanxue.com/upload/attach/202206/952303_YHDNCGHEATX7YX7.jpg)

图 smss.exe样本启动

## 四、一阶样本

### 1）基本信息

|  |  |
| --- | --- |
| MD | 946f787c129bf469298aa881fb0843f4 |
| 时间戳 | 2021-09-11 15:33:3 |
| CPU架构 | X64 |

一阶样本中包含大量加密字符串，字符串通过解密函数解出所需函数。样本运行后释放1个诱饵文档和二阶样本。

### 2）功能推测

导入表中看到`LoadLibraryA`、`GetProcAddress`函数，推测样本可能使用动态加载方式加载DLL和函数。

![](https://bbs.kanxue.com/upload/attach/202206/952303_EVHVJ7DCJJ2HPKK.jpg)

图 导入表

拖入IDA，查看`LoadLibraryA`、`GetProcAddress`交叉引用，找到调用的函数`sub_140001130`。查找调用`sub_140001130`的函数，发现调用该函数前皆传入字符串。推测传入字符串用于解密调用DLL和函数。

![](https://bbs.kanxue.com/upload/attach/202206/952303_QDXK9R5AUT5AHWA.jpg)

图 加密字符串

![](https://bbs.kanxue.com/upload/attach/202206/952303_J7KXVAWAGMNYJ2V.jpg)

图 动态加载函数

查看`sub_140001360`，由于该函数堆栈不平衡，无法使用f5反汇编代码，所以先平衡堆栈，随后动态调试确定各字符串含义（或者写脚本解密字符串）。

![](https://bbs.kanxue.com/upload/attach/202206/952303_5E7FBGKR5SQUBG4.jpg)

图 堆栈平衡后代码

### 3）动态调试

使用`wsprintf`拼接字符串，组成路径`C:\ProgramData\Icon.pif`。随后再拼接一个命令`C:\ProgramData\Icon.pif iDmzQtvReUSCdTn`。

![](https://bbs.kanxue.com/upload/attach/202206/952303_7EAGNW249JRCZQA.jpg)

![](https://bbs.kanxue.com/upload/attach/202206/952303_RJKWT2NK5J33HC7.jpg)

图 命令拼接

创建1个文件`C:\ProgramData\210927 xxx.hwp`并写入数据。创建`C:\ProgramData\Icon.pif`并写入数据后执行。

![](https://bbs.kanxue.com/upload/attach/202206/952303_2A83KNPTHWHGYXY.jpg)

图 样本创建

![](https://bbs.kanxue.com/upload/attach/202206/952303_9R7W6GGAQTVCKG6.jpg)

图 样本写入

![](https://bbs.kanxue.com/upload/attach/202206/952303_VS4H6XS3C5UUVYK.jpg)

图 样本执行

至此一阶样本调试完成

## 五、二阶样本

### 1）基本信息

|  |  |
| --- | --- |
| MD5 | E33A34FA0E0696F6EAE4FEBA11873F56 |
| 时间戳 | 2021-09-11 09:13:44 |
| CPU架构 | X64 |

二阶样本通过2种参数执行不同功能。参数`iDmzQtvReUSCdTn`用于写入注册表添加自启动功能，期间拷贝自身并重命名为`smss.exe`并传入参数`zWbTLWgKymXMDwZ`启动新进程。参数`zWbTLWgKymXMDwZ`使用post方式请求域名`movie.youtoboo.kro[.]kr`，连接成功过后等待传入指令，共计19种指令。

### 2）第一个参数

二阶样本获取当前进程路径并在该路径下创建`system32`目录，拷贝`Icon.pif`到该目录并改名为`smss.exe`，写入注册表`reg add hkcu\\software\\microsoft\\windows\\currentversion\\run`实现自启动，最后启动`smss.exe`并传入参数`zWbTLWgKymXMDwZ`，连接CC等待指令。

![](https://bbs.kanxue.com/upload/attach/202206/952303_B7DC5AX59F8CZSB.jpg)

图 目录创建

![](https://bbs.kanxue.com/upload/attach/202206/952303_JWTKFJEAPR5H9YV.jpg)

图 创建smss.exe

![](https://bbs.kanxue.com/upload/attach/202206/952303_SK65X35VCEXC6JH.jpg)

图 写入注册表

![](https://bbs.kanxue.com/upload/attach/202206/952303_HTEN47C5FW7B8H6.jpg)

图 样本启动

### 3）第二个参数

运行后样本将尝试使用post方式请求域名`movie.youtoboo.kro.kr`并使用UA：`Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36`。样本一旦与CC连接成功后，将等待传入指令，样本有19种接收参数。

|  |  |
| --- | --- |
| CC指令 | 功能 |
| 3 | 获取当前目录 |
| 4 | 修改文件日期 |
| 5 | 结束特定进程 |
| 6 | 获取特定进程信息 |
| 7 | 删除文件 |
| 8 | 创建新线程并尝试移动一个文件，失败则破坏 |
| 9 | 创建进程 |
| 10 | 注册DLL |
| 11 | 分配内存空间，具体功能未知 |
| 12 | 读取文件 |
| 13 | 写入文件 |
| 14 | 获取系统时间 |
| 15 | 获取系统时间 |
| 16 | 删除注册表 |
| 17 | 获取主机名，计算机名，网卡信息 |
| 18 | 全局信息时间 |
| 19 | 获取%temp%目录 |
| 20 | 读取PMS\*文件信息 |
| 21 | 读取PMS\*文件信息 |

表 功能说明

![](https://bbs.kanxue.com/upload/attach/202206/952303_Z5X67GFSJU6HUBK.jpg)

图 指令3 获取当前目录

![](https://bbs.kanxue.com/upload/attach/202206/952303_HSEC8QYB5X244B9.jpg)

图 指令4 修改文件日期

![](https://bbs.kanxue.com/upload/attach/202206/952303_FNY7JGBPSQCCNKG.jpg)

图 指令5 结束特定进程

![](https://bbs.kanxue.com/upload/attach/202206/952303_QM9ZGUMR4ZGK3WD.jpg)

图 指令6 获取特定进程信息

![](https://bbs.kanxue.com/upload/attach/202206/952303_763YUFUCURTTNXP.jpg)

图 指令7 删除文件

![](https://bbs.kanxue.com/upload/attach/202206/952303_PVE5VSSR8RZ8TQJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202206/952303_RVP9HZ2M3U5U9WA.jpg)

图 指令8 创建新线程并尝试移动一个文件，失败则破坏

![](https://bbs.kanxue.com/upload/attach/202206/952303_F9Z82ZEGSKK4G4C.jpg)

图 指令9 创建进程

![](https://bbs.kanxue.com/upload/attach/202206/952303_YTQYKMBX4NMA585.jpg)

图 指令10 注册DLL

![](https://bbs.kanxue.com/upload/attach/202206/952303_PWMJ6HEKYUW422Y.jpg)

图 指令11 分配内存空间，具体功能未知

![](https://bbs.kanxue.com/upload/attach/202206/952303_9SV9MWFTJ7R8XSF.jpg)

图 指令12 读取文件

![](https://bbs.kanxue.com/upload/attach/202206/952303_7YS7BVNTYT5VSNV.jpg)

图 指令13 写入文件

![](https://bbs.kanxue.com/upload/attach/202206/952303_SPGQCYFGNWPF69P.jpg)

图 指令14、15 获取系统时间

![](https://bbs.kanxue.com/upload/attach/202206/952303_P5F8XHBK2XAFU7U.jpg)

图 指令16 删除注册表

![](https://bbs.kanxue.com/upload/attach/202206/952303_HNQ733NBB2VVRUT.jpg)

图 指令17 获取主机名，计算机名，网卡信息

![](https://bbs.kanxue.com/upload/attach/202206/952303_54DCR8RNR343S37.jpg)

图 指令18 全局信息时间

![](https://bbs.kanxue.com/upload/attach/202206/952303_HPHCCT3V87J3KY2.jpg)

图 指令19 获取%temp%目录

![](https://bbs.kanxue.com/upload/attach/202206/952303_W38PX6ZACSQTD8U.jpg)![](https://bbs.kanxue.com/upload/attach/202206/952303_PZDRFRZGDCUNEXD.jpg)

图 指令20、21 读取PMS\*文件信息

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：

- [密码\_threatbook.zip](javascript:void(0);)

  （233.92kb，27次下载）
