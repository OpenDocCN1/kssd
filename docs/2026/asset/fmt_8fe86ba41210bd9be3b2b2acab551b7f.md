
![image.png](upload/attach/202203/819301_8TTARGQ2FNWUW7E.jpg)

![image.png](upload/attach/202203/819301_SV5P8QUC28NYGFC.jpg)

执行`1C3D函数`

该函数功能，遍历进程，找到`explorer`进程

![image.png](upload/attach/202203/819301_BSDA3MFRHJMHZ7V.jpg)

通过`OpenProcess`函数打开`explorer`进程，通过`OpenProcessToken`函数获得`explorer`的令牌句柄，将`OpenProcessToken`的第三个参数`TokenHandle`返回。

![image.png](upload/attach/202203/819301_4RCN9HE3YJPZSDU.jpg)

回到`1B02函数`

内存初始化

执行`18DC函数`，该函数，将`ESI`的字符串写入内存

![image.png](upload/attach/202203/819301_F55M69TCC7P9GGM.jpg)

再回到`1B02函数`

加载`Advapi32.dll`

![image.png](upload/attach/202203/819301_B3Q3CMPZCECZXW8.jpg)

获取`CreateProcessAsUser`函数地址

![image.png](upload/attach/202203/819301_RKH2U4Z8GYY3M69.jpg)

执行`CreaterocessAsUser`函数，以当前登录用户身份运行程序

![image.png](upload/attach/202203/819301_NVFTNMM2AMZ9MW6.jpg)

执行完毕后

![image.png](upload/attach/202203/819301_K8RTVSNAAG7MAZG.jpg)

回到`1707函数`，执行后续的内存释放操作

![image.png](upload/attach/202203/819301_PEJVB9YTXHGGG6.jpg)

如果是`administrator`（高权限）权限，则直接执行

![image.png](upload/attach/202203/819301_CAN7P3TAQSTQS4H.jpg)

![image.png](upload/attach/202203/819301_RZSFRPN6NBJBKSS.jpg)

至此详细分析完毕

梳理如下

![image.png](upload/attach/202203/819301_9ZUYHK893PAS2U3.jpg)

# 六、应对策略

样本行为

1、释放文件`C:\Users\Administrator\AppData\Local\cdnver.bat`和`C:\Users\Administrator\AppData\Local\cdnver.dll`

2、修改注册表

![image.png](upload/attach/202203/819301_HXRY2MZX3SX6S8W.jpg)

基于此样本，删除文件、还原注册表即可。

# 一、基本信息

病毒名称：`ff808d0a12676bfac88fd26f955154f8884f2bb7c534b9936510fd6296c543e8`

MD5：`36524C90CA1FAC2102E7653DFADB31B2`

SHA1：`8D6DB316EA4E348021CB59CF3C6EC65C390F0497`

CRC32：`04DA02C3`

样本来源：`anyrun`

# 二、测试环境及工具

## 2.1测试环境

`Win7 x64 SP1`

## 2.2测试工具

### 2.2.1查壳工具

`ExeinfoPE`

### 2.2.2监测工具

火绒剑、`Processonitor`、`Regshot`

### 2.2.3调试工具

`OD`、`IDA`

# 三、基本静态分析

## 3.1字符串分析

日期月份时间相关的字符串

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_P9QXFK9JTXUDEBC.jpg)

各种字符，猜测跟加密相关

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_G9WA4Z4V9352NBV.jpg)

各个国家的语言

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_ZPHCCPQXRTJM6PW.jpg)

## 3.2PE分析

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_73HNM23DM7HFAFX.jpg)

### 3.2.1导入表

1、`KERNEL32.dll`

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_CYHJUUABR3MQEWA.jpg)

关键函数

进程遍历：

`CreateToolhelp32Snapshot`

`Process32NextW`

文件操作：

`WriteFile`

`DeleteFile`

`CreateFile`

`FlushFileBuffers`

获取当前操作系统的信息:

`GetEnviromentVariable`

解密相关：

`Decodeointer`

反调试相关：

`IsDebuggerPresent`

`IsProcessorFeaturePresent` //判断处理器相关信息

2、`ADVAPI32.dll`

可能跟提权相关

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_MTMHT24KJCX49RW.jpg)

3、`SHELL32.dll`

命令执行相关

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_JFBNKQEZG3QWE9G.jpg)

## 3.3壳分析

无壳

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_B26ZM7X99XAAH6Y.jpg)

# 四、行为分析

## 4.1文件监控

创建、写入、修改文件

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_ME4T9VADN5BA7KJ.jpg)

## 4.2进程监控

启动一个`rundll32.exe`进程

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_XY7H6QWKVKHTMUX.jpg)

通过`rundll32.exe`加载释放的`dll`文件

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_TXPYSUUENTYPATR.jpg)

## 4.3网络监控

因为使用的模拟网络，所以需要根据`DNS`请求的域名进行过滤

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_YDJX79FYZGDUV7P.jpg)

威胁情报进行查询![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_8HS6C2HN4QMNQFD.jpg)

## 4.4注册表监控

敏感操作：

注册表`HKCR\Environment\UserInitMprLogonScript`键值所指的程序会在杀毒软件启动前启动，而且修改它不需要管理员权限，因此我们可以通过其来达到权限维持的效果。

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_UPZS2VN4CSFABM7.jpg)

综上，可以看出，我们拿到的样本，执行后，释放一个`bat`文件和一个`dll`文件，`bat`文件调用`rundll32.exe`执行`dll`文件，并写入注册表进行权限维持。网络方面收集完本机信息会去访问`C2`服务器。核心功能应该是在`dll`文件中，接下来先分析这个母体文件。

# 五、综合分析

`IDA`大致分析样本流程如下：

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_RYX8JAS6STGAW5V.jpg)

下面结合`OD`详细分析：

`OD`加载后，停留在此处

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_UKA6WCNMEZ7M3ND.jpg)

执行大量函数，无法直观判断具体功能

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_VCKPRD6QY6MVUK7.jpg)

一直执行下去发现，程序终止，所以猜测上面的函数应该有反调试的功能

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_NGSNNSCZD6UGUAJ.jpg)

根据动态获取的基地址，结合`IDA`的`OEP`，直接跳转程序入口，下断点并执行到此处

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_XHZNANABQ8ZYFQG.jpg)

程序开始执行

**`1DEF函数`**

先是执行了三个相同的函数

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_4Q4GUR3Z49MKNTZ.jpg)

函数内部有循环和运算操作，猜测是进行加解密操作

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_6DDT8UKDZM2YW9S.jpg)

`OD`执行查看，解密字符串如下，是三个路径

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_A36HDX3PQ5BGW66.jpg)

**`12D3函数`**

两次执行`1063函数`解密字符串：

第一次：`cdnver.dll`

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_EP63DTAD3T99JE2.jpg)

函数内部如下

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_QFU6AQYYP62Z4C9.jpg)

第二次：函数内部逻辑同上

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_N2STFTR48QVPGDF.jpg)

**`1000函数`**

解密一段数据，该数据是`PE`文件

解密前：

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_XD66KHKKFTPT2YB.jpg)

解密后：

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_YPQ53UFPGT6XDJV.jpg)

**`13F7函数`**

`1DEF函数`解密字符串：

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_GVXJYVV7VGJDWM4.jpg)

实现数据压缩解压缩

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_NZXQBVHK3VZZRXV.jpg)

解压到此处

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_4C46FNM8BAM7A56.jpg)

解压完毕

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_KA6398GSHCEAQ3E.jpg)

**`155B函数`**

执行`10CD函数`

执行`1DEF函数`解密两个字符串

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_MG5P5HEJ2XJRTFH.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_GYKXX3YD765NTME.jpg)

获取当前环境变量的字符串

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_UJN62RKGJSYWFDM.jpg)

加载`Kernel32.dll`

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_Y8NTGERA3EU4T63.jpg)

获取`CreateFile`函数地址

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_NWRH49U5YQ9H7D7.jpg)

调用`CreateFile`创建一个文件

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_DWRHYWZHCKBFS6S.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_T3W7TWJWRHDEYW3.jpg)

获取`WriteFile`函数的地址

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_E4ZUMJV3674VHSH.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_5AYZBS66JB8STXF.jpg)

此时文件已经写入完毕

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_6AAKNGMVJDJMXVZ.jpg)

**`264C函数`**

`1DEF`执行`7`次解密字符串：

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_GDAESE49MVZE99R.jpg)

加载`ADVAPI32.dll`，里面包含注册表、事件日志等相关操作

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_MWKPEFCCFBNZQ6D.jpg)

获取`RegOpenKeyExW`函数地址，用于打开注册表键

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_JX73HUUSXMRM4ZC.jpg)

主键名称对应的常数

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_C448V8V6ZZKEBFF.jpg)

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_37BXR7NVTWBPHA8.jpg)

获取环境变量及处理字符串工作

`_wgetenv`：从环境中取字符串，获取环境变量

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_YKJJZWWY4JBB35F.jpg)

获取“`LOCALAPPDATA`”的环境变量

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_ATUXTXTAKX99MT8.jpg)

`_wcsstr`：从字符串中寻找子串

返回值：返回`str`中第一次出现`strSearch`的指针，如果`strSearch`未出现在`str`中，则返回`NULL`。如果`strSearch`指向长度为零的字符串，则函数返回`str`。

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_4S3K53JP7AS5JUZ.jpg)

从`cdnver.dll`找`"."`，获取文件后缀名字

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_25WQQ8J6DCKPF4F.jpg)

循环判断，防止文件名中有不止一个`"."`

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_PB6WT8BDW8DEG7T.jpg)

直到文件名中没有`"."`，`EAX`返回值是`0`

![image.png](https://bbs.kanxue.com/upload/attach/202203/819301_BPZ5JQGJCMNGPM9.jpg)

**`2030函数`**

`1DEF`解密字符串
