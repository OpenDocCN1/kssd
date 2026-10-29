

## [原创]MS17-010 “永恒之蓝”漏洞分析与复现

## MS17-010漏洞分析

### 一、漏洞信息

#### 1. 漏洞简述

- 漏洞名称：“永恒之蓝”漏洞
- 漏洞编号：MS17-010，CVE-2017-0143/0144/0145/0146/0147/0148
- 漏洞类型：缓冲区溢出漏洞
- 漏洞影响：信息泄露
- CVSS评分：9.3（High）
- 利用难度：Medium
- 基础权限：不需要

#### 2. 组件概述

`SMB`(Server Message Block)是一个协议名，它能被用于Web连接和客户端与服务器之间的信息沟通。其目的是将DOS操作系统中的本地文件接口“中断13”改造为网络文件系统。

`SMB1.0`协议由于在文件共享传输过程中存在的传输效率低以及传输空间小等缺陷被人们所摒弃。为了更好的实现网络中文件的共享过程，在`SMB1.0`的基础上开发了新的网络文件传输协议，并将其命名为`SMB2.0`。该协议在实现了文件共享传输的基本功能的基础上对文件传输的效率、文件缓存的空间以及文件并发传输等问题进行改进，使得在局域网或更高配置的网络环境下，文件传输过程的速度和效率等得到了很大的提升。

#### 3. 漏洞影响

Windows Vista SP2; Windows Server 2008 SP2 and R2 SP1; Windows 7 SP1; Windows 8.1; Windows Server 2012 Gold and R2; Windows RT 8.1; and Windows 10 Gold, 1511, and 1607; and Windows Server 2016

以上系统打开445号端口都容易受到影响。

#### 4. 解决方案

- 禁用 SMBv1

对于客户端操作系统：

1. 打开“控制面板”，单击“程序”，然后单击“打开或关闭 Windows 功能”。
2. 在 Windows 功能窗口中，清除`SMB1.0/CIFS 文件共享支持`复选框，然后单击确定关闭窗口。
3. 重新启动系统。

对于服务器操作系统：

1. 打开服务器管理器，然后单击管理菜单并选择删除角色和功能。
2. 在功能窗口中，清除`SMB1.0/CIFS 文件共享支持`复选框，然后单击确定关闭窗口。
3. 重新启动系统。

- 更新Windows系统补丁

<a href="elink@849K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1N6i4m8V1j5i4c8W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3x3U0l9I4y4#2)9J5c8X3#2K6x3e0N6Q4x3X3b7H3x3e0m8Q4x3U0y4K6k6h3y4#2M7X3W2@1P5g2)9J5k6s2g2H3k6r3q4@1k6g2)9J5k6r3k6G2M7W2)9J5k6r3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8K6L8h3u0Q4x3X3c8K6k6i4u0$3k6i4u0Q4x3X3b7@1x3o6p5K6x3K6R3&6">官方文档链接</a>

### 二、漏洞复现

#### 1. 环境搭建

实验中需要用到三个机器，分别是调试机，靶机（被调试机），攻击机

- 调试机环境：主机Windows 10x64专业版 1909
- 靶机（被调试机）环境：Windows 7x86 SP1
- 靶机（被调试机）配置：`192.168.44.132:445`
- 攻击机环境：Windows XPx86 SP3
- 攻击机配置：`192.168.44.152`

##### 1.1 双机内核调试

首先需要用调试机调试靶机，具体实现双机调试请看<a href="elink@8baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3S2#2j5h3&6G2L8X3N6&6K9h3&6Y4x3e0x3I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5&6x3o6M7@1x3o6t1^5y4R3`.`.">这里</a>。

##### 1.2 安装并配置fuzzbunch

另外需要在攻击机安装`fuzzbunch`实现永恒之蓝漏洞：

1. 安装<a href="elink@6e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3P5i4c8Z5L8$3&6Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3x3W2)9J5k6e0k6Q4x3V1j5`.">Python 2.6</a>
2. 安装<a href="elink@07cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2X3L8%4u0Y4k6g2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3A6W2j5%4c8K6i4K6u0r3M7s2W2%4K9h3^5K6x3W2)9J5c8X3k6A6L8r3g2K6i4K6u0r3M7s2W2%4K9h3^5K6x3W2)9J5c8V1u0#2K9h3I4V1i4K6t1#2x3U0l9J5x3e0u0Q4x3V1j5`.">PyWin32</a>

需要以管理员权限进行安装

3. 下载<a href="elink@ff9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9i4y4@1k6i4u0U0K9o6m8U0i4K6u0r3M7$3S2S2k6r3!0%4j5Y4u0G2K9$3g2J5i4K6u0W2k6$3W2@1i4K6u0r3">fuzzbunch</a>
4. 在`shadowbroker-master`的子目录`windows`中新建`listeningposts`文件夹，同时修改`FuzzyBunch.xml`文件内容，设置相应的`ResourcesDir`和`LogDir`参数值，修改结果如下图所示。（路径根据实际情况而定）

![1.png](upload/attach/202109/917290\_39CJNGZZ5ZJBUAJ.jpg)

##### 1.3 靶机环境配置

1. 打开靶机445端口，具体打开方法点<a href="elink@d36K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7K9h3&6Y4P5h3q4F1i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3U0u0X3k6e0N6U0k6h3c8U0x3e0x3^5x3U0j5K6x3o6l9J5y4U0p5%4k6X3f1K6i4K6u0W2K9s2c8E0L8l9`.`.">这里</a>
2. 关闭防火墙：控制面板-系统和安全-Windows防火墙-打开或关闭Windows防火墙-全部选择关闭防火墙。

#### 2. 复现过程

在攻击机中启动命令行，进入`fuzzbench`的`Windows`文件夹，用`python`启动`fuzzbench`：

![2.PNG](upload/attach/202109/917290\_CTKBXNXS92WQRGU.jpg)

启动后设置靶机IP-设置击机IP-重定向`no-log`地址确认（无误直接回车）-`0+回车`（创建新的项目）-为项目命名-设置路径（Yes）-：

![3.PNG](upload/attach/202109/917290\_WN43SD8EAUBAGRM.jpg)

使用永恒之蓝建立后门：

```
use Eternalblue
```

之后一直回车到出现设置靶机系统版本，本次复现靶机为Win7，所以选择1：

![4.PNG](upload/attach/202109/917290\_B744DXPFCFJ9YSN.jpg)

![5.PNG](upload/attach/202109/917290\_YGNRBT2JME4TPWJ.jpg)

下一项模式选择1，FB模式：

![6.PNG](upload/attach/202109/917290\_T6VGDSV6E745QS3.jpg)

接下来一直回车即可，即可看到插件运行，并成功利用永恒之蓝漏洞在靶机系统中留下后门：

![10.PNG](upload/attach/202109/917290\_R45XJX4H5TWYHAV.jpg)

### 三、漏洞分析

#### 1. 基本信息

- 漏洞文件：`srv.sys`
- 漏洞函数：`srv!SrvOs2FeaListToNt`
- 漏洞对象：为`NtFeaList`分配的缓冲区

#### 2. 背景知识

##### 2.1 `SrvOs2FeaListToNt()`函数

`SrvOs2FeaListToNt()`函数用于将FEA list转化为NTFEA list，需要分配在内核地址大非分页池分配缓冲区来存放转化后的NTFEA list，因此需要先计算转化后的NTFEA list的大小，计算大小是通过`SrvOs2FeaListSizeToNt()`函数进行的，这个函数被`SrvOs2FeaListToNt()`调用。

##### 2.2 `0xffdff000`系统预留地址空间

`0xffdff000`处地址是系统预留的地址空间，用于存放时钟，版本，配置等信息，其分配的权限为可执行：

```cpp
kd> !pte 0xffdff000
VA ffdff000
PDE at C0603FF0 PTE at C07FEFF8
contains 000000000018A063 contains 00000000001E3163
pfn 18a ---DA--KWEV pfn 1e3 -G-DA--KWEV
```

#### 3. 详细分析

##### 3.1 静态分析

###### 3.1.1 Ida分析

使用`Ida`打开`Srv.sys`

首先按`shift + F1`，再按`insert`键导入两个结构：

```cpp
struct _FEA{
BYTE fEA; //标记位
BYTE cbNAME; //记录名称长度
USHORT cbValue; //记录值的长度
}FEA;
```

```cpp
struct _FEALIST{
ULONG cbList; //记录Fea List总长度
_FEA list[1];
}FEALIST, *PFEALIST;
```

查看`SrvOs2FeaListSizeToNt()`函数源码，右键`a1`，使用`_FEALIST`结构覆盖：

```cpp
int __stdcall SrvOs2FeaListSizeToNt(_FEALIST *FeaList)
{
_FEALIST *v1; // eax
char *v2; // edi
_FEA *v3; // esi
int v4; // ebx
int v6; // [esp+Ch] [ebp-4h]
v1 = FeaList; // a1
v6 = 0; // NtFeaList的大小
v2 = (char *)FeaList + FeaList->cbList; // 获取指向表结尾的指针v2
v3 = FeaList->list; // 指向表的开始
if ( FeaList->list < (_FEA *)v2 ) // 表的开始指针应该在结尾指针之前
{
while ( &v3[1] < (_FEA *)v2 ) // 从FeaList表的第一个元素开始，遍历整个表
{
v4 = v3->cbValue + v3->cbNAME; // 获取当前FEA的长度
if ( &v3[1].cbNAME + v4 > (BYTE *)v2 ) // 检查下一个FEA是否有效
break;
if ( RtlSizeTAdd(v6, (v4 + 12) & 0xFFFFFFFC, &v6) < 0 )// 增加NtFeaList的大小，每次12字节
return 0;
v3 = (_FEA *)((char *)v3 + v4 + 5); // 下一个FEA，加5意思是每个FEA后面有5字节的NULL
if ( v3 >= (_FEA *)v2 )
return v6;
v1 = FeaList; // 重置v1
}
LOWORD(v1->cbList) = (_WORD)v3 - (_WORD)v1; //此处发生计算错误
}
return v6;
}
```

`LOWORD(v1->cbList) = (_WORD)v3 - (_WORD)v1;` 这句代码中存在一处错误，`cbList`长度本为4字节，这里却被强制转换为2字节，导致赋值过程中无视高2字节的内容，只赋值了低2字节的内容。

###### 3.1.2 补丁Diff

原来的`SrvOs2FeaListSizeToNt()`函数中的C代码：

```
LOWORD(v1->cbList) = (_WORD)v3 - (_WORD)v1;
```

修补后：

```
*(DWORD*)(v1->cbList) = v3 - (_DWORD)v1;
```

`(v1->cbList)`数据类型由`WORD`变为了`DWORD`

##### 3.2 动态分析

将`Windbg`（需要配置好符号文件）连接靶机进行内核调试，连接成功`Windbg`会显示：

![7.PNG](upload/attach/202109/917290\_DUYQ4HNCBHBEBSK.jpg)

运行前先设置几个断点来采集运行过程中涉及的关键数据：

```cpp
bp srv!SrvSmbOpen2+0x79 ".printf \"feasize: %p indatasize: %p fealist addr: %p\\n\",edx,ecx,eax;g;"
// 获取Fea大小和indata大小
bp srv!SrvOs2FeaListToNt+0x10 ".printf \"feasize before: %p\\n\",poi(edi);r $t0 = @edi;g;"
bp srv!SrvOs2FeaListToNt+0x15 ".printf \"NTFEA size: %p feasize after: %p\\n\",eax,poi(@$t0);g;"
// FEA List大小的前后变化，以及NTFEA List大小
bp srv!SrvOs2FeaListToNt+0x99 ".printf \"NEXT: FEA: %p NTFEA: %p\\n\",esi,eax;g;"
bp srv!SrvOs2FeaToNt+04d ".printf \"MOV2: dst: %p src: %p size: %p\\n\",ebx,eax,poi(esp+8);g;"
// 查看被分配的池
bp srv!SrvOs2FeaListToNt+0xd5
// 查看服务器返回的值
```

设置好断点后，在攻击机运行脚本，开始攻击

这时，`Windbg`会记录下攻击过程中的数据：

![8.PNG](upload/attach/202109/917290\_QQACH6R3DHZ4MZV.jpg)

整理一下：

```cpp
feasize: 00010000
indatasize: 000103d0
fealist addr: a1f050d8
feasize before: 00010000
NTFEA size: 00010fe8
feasize after: 0001ff5d
```

可以看到经过某处指令后`feasize`由`00010000`变为`0001ff5d`，这就是代码`LOWORD(v1->cbList) = (_WORD)v3 - (_WORD)v1;`产生的错误计算，其原因请看下面对应的汇编指令：

此时`esi`和`eax`寄存器的值：

`FeaEnd: esi = a4217035h`

`FeaStart: eax = a42070d8h`

`FeaSize: [eax] = 00010000`

```cpp
sub esi, eax
mov word ptr[eax], si
```

经过`sub`指令后`esi`的值为`ff5d`，而在给`[eax]`赋值的时候使用`WORD`类型，导致了`eax`内容变成了`0001ff5d`。

继续查看内存池的数据：

```
NEXT: FEA: a1f050dc NTFEA: 85da4008
MOV2: dst: 85da4011 src: a1f050e1 size: 00000000
NEXT: FEA: a1f050e1 NTFEA: 85da4014
MOV2: dst: 85da401d src: a1f050e6 size: 00000000
......
NEXT: FEA: a1f05ca3 NTFEA: 85da5c4c
MOV2: dst: 85da5c55 src: a1f05ca8 size: 00000000
NEXT: FEA: a1f05ca8 NTFEA: 85da5c58
MOV2: dst: 85da5c61 src: a1f05cad size: 00000000
NEXT: FEA: a1f05cad NTFEA: 85da5c64
MOV2: dst: 85da5c6d src: a1f05cb2 size: 0000f383
NEXT: FEA: a1f15035 NTFEA: 85db4ff0
MOV2: dst: 85db4ff9 src: a1f1503a size: 000000a8
srv!SrvOs2FeaListToNt+0xd5:
8949263a be0d0000c0 mov esi,0C000000Dh
```
