
# [原创]【安全研究】从mimikatz学习Windows安全之访问控制模型（三）

作者：Loong716@<a href="elink@7e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8i4g2D9j5h3t1`.">Amulab</a>

## 0x00 前言

在之前的文章中，分别向大家介绍了Windows访问控制模型中的SID和Access Token，本篇文章中将为大家介绍最后一个概念——特权

Windows操作系统中许多操作都需要有对应的特权，特权也是一种非常隐蔽的留后门的方式。在AD域中，一些特权在`Default Domain Controller Policy`组策略中被授予给一些特殊的组，这些组的成员虽然不是域管，但如果被攻击者控制同样能给AD域带来巨大的风险

因此对防御者来讲，排查用户的特权配置也是重中之重，本文将对一些比较敏感的特权进行介绍，便于防御者更好的理解特权的概念以及进行排查

## 0x01 令牌中的Privilege

特权是一个用户或组在本地计算机执行各种系统相关操作（关闭系统、装载设备驱动程序、改变系统时间）的权限，特权与访问权限的区别如下：

* 特权控制账户对系统资源和系统相关任务的访问，而访问权限控制对安全对象（可以具有安全描述符的对象）的访问
* 系统管理员为用户或组指派特权，而系统根据对象的DACL中的ACE授予或拒绝对安全对象的访问，有时拥有特权可以忽略ACL的检查

在之前介绍Access Token的文章中我们已经了解过了token的基本结构，其中有一部分表示了该用户及该用户所属组所拥有的特权，如下图所示：

![1632456879323.png](upload/attach/202109/931257\_HBWR3Y7EVN2CCH3.jpg)

通常我们会使用`whoami /priv`命令查看当前用户所拥有的特权，默认情况下大部分特权是禁用状态，在使用时需要启用

![1632456864605.png](upload/attach/202109/931257\_M3EF4NAKJT95RNB.jpg)

## 0x02 mimikatz的privilege模块

mimikatz中的privilege模块主要有以下功能，下图中第一个红框中的部分是为当前进程启用一些指定的特权，第二个红框中的`id`和`name`分别支持指定特权的id和名称，并为当前进程启用id和名称对应的特权

![1632456845760.png](upload/attach/202109/931257\_SJKEEGQ5P3NQKZH.jpg)

通常我们比较通用的启用进程特权的方法是这样的，代码如下：

```
BOOL GetDebugPrivilege()
{
BOOL status = FALSE;
HANDLE hToken;
if (OpenProcessToken(GetCurrentProcess(), TOKEN\_ADJUST\_PRIVILEGES, &hToken))
{
TOKEN\_PRIVILEGES tokenPrivs;
tokenPrivs.PrivilegeCount = 1;
if (LookupPrivilegeValueW(NULL, SE\_DEBUG\_NAME, &tokenPrivs.Privileges[0].Luid))
{
tokenPrivs.Privileges[0].Attributes = TRUE ? SE\_PRIVILEGE\_ENABLED : 0;
if (AdjustTokenPrivileges(hToken, FALSE, &tokenPrivs, sizeof(tokenPrivs), NULL, NULL))
{
status = TRUE;
}
}
else wprintf(L"[!] LookupPrivilegeValueW error: %u when get debug privilege.\n", GetLastError());
CloseHandle(hToken);
}
else wprintf(L"[!] OpenProcessToken error: %u when get debug privilege.\n", GetLastError());
return status;
}
```

而mimikatz是通过调用一个未文档化的API`RtlAdjustPrivilege()`，该API的功能是对当前进程或线程启用/禁用指定的特权，共有四个参数：

* `ULONG Privilege`：需要操作的特权的ID
* `BOOLEAN Enable`：启用或禁用的标志，1为启用，0为禁用
* `BOOLEAN CurrentThread`：指定是否为当前线程，1则设置线程令牌，0则设置进程令牌
* `PBOOLEAN Enabled`：该特权修改之前是禁用的还是启用的

```
NTSTATUS RtlAdjustPrivilege
(
ULONG Privilege, // [In]
BOOLEAN Enable, // [In]
BOOLEAN CurrentThread, // [In]
PBOOLEAN Enabled // [Out]
)
```

如果参数指定的是特权的名称，则会先调用`LookupPrivilegeValue()`拿到特权名称对应的特权ID，然后再调用`RtlAdjustPrivilege()`来启用特权

![1632456816631.png](upload/attach/202109/931257\_UZNQQYM65JDNHVX.jpg)

前面提到的是将禁用的特权启用，而如果想给一个账户赋予特权，则可以通过本地策略/组策略来设置，也可以通过`LsaAddAccountRights()`这个API，这里不再赘述

## 0x03 危险的特权

这里主要介绍11个危险的特权，在检查域内安全时要格外注意

### 1\. SeDebugPrivilege

通常情况下，用户只对属于自己的进程有调试的权限，但如果该用户Token中被赋予`SeDebugPrivilege`并启用时，该用户就拥有了调试其他用户进程的权限，此时就可以对一些高权限进程执行操作以获取对应的权限，以进程注入为例：

![微信图片\_20210924164821.png](upload/attach/202109/931257\_ETGX8ESMB84PU3Q.jpg)

### 2\. SeBackupPrivilege

该特权代表需要执行备份操作的权限，授予当前用户对所有文件的读取权限，不受文件原本的ACL限制，主要有以下利用思路：

1. 备份SAM数据库
2. 备份磁盘上高权限用户的敏感文件
3. 域内在域控上备份ntds.dit

下图以导出注册表中的SAM和SYSTEM为例

![1632457002019.png](upload/attach/202109/931257\_PZG6W7N3SSMMK87.jpg)

观察上图可能有师傅会问：为什么前面显示`SeBackupPrivilege`是Disable状态，却能成功执行reg save呢？一开始我猜测可能是reg.exe在执行操作前默认会启用一些特权，随后通过对reg.exe的逆向也印证了这点：

![1632457018350.png](upload/attach/202109/931257\_7B6UMQQJ8KNNNJB.jpg)

在域环境中，`Backup Operators`和`Server Operators`组成员允许在域控进行本地登录，并在域控上拥有`SeBackupPrivilege`特权，所以也可以对ntds.dit进行备份操作，再备份注册表中的SYSTEM和SECURITY，进而解密ntds.dit

需要注意的是在调用`CreateFile()`时，需要指定`FILE\_FLAG\_BACKUP\_SEMANTICS`标志来表示正在为备份或恢复操作打开或创建文件，从而覆盖文件的ACL检查3

```
HANDLE hFile = CreateFileW(
L"C:\\Windows\\System32\\1.txt",
GENERIC\_READ,
0,
NULL,
OPEN\_EXISTING,
FILE\_FLAG\_BACKUP\_SEMANTICS,
NULL);
```

### 3\. SeRestorePrivilege

该特权是执行还原操作所需的权限，拥有此特权的用户对所有文件拥有写权限，不受文件原本的ACL限制，主要利用思路如下：

1. 修改注册表，实现修改服务、修改启动项等操作
2. 写文件进行DLL劫持

![1632457344306.png](upload/attach/202109/931257\_2SWGWT925XK68K9.jpg)

域环境中，`Backup Operators`和`Server Operators`组成员同样在域控上也有`SeRestorePrivilege`，因此也可以利用上述操作在域控上完成提权和维quan等操作

需要注意的仍是调用API时，需要指定对应的标志，如`CreateFile()`需要指定`FILE\_FLAG\_BACKUP\_SEMANTICS`，`RegCreateKeyEx()`需要指定`REG\_OPTION\_BACKUP\_RESTORE`

### 4\. SeTakeOwnershipPrivilege

该特权用来修改目标对象的所有权，也就是说拥有该特权的用户可以修改任意对象的所有者（Owner），而所有者对该对象是有WriteDACL的权限的，可以任意修改对象的ACL

所以如果拥有了`SeTakeOwnershipPrivilege`，就相当于对任意对象有读写的权限，利用方式和`SeRestorePrivilege`、`SeBackupPrivilege`基本相同4

```
GetTakeOwnershipPriv();
...
status = SetNamedSecurityInfo(
L"C:\\Windows\\System32\\localspl.dll",
SE\_FILE\_OBJECT,
OWNER\_SECURITY\_INFORMATION,
user->User.Sid,
NULL,
NULL,
NULL);
```

如下图所示，可以将对象的Owner从TrustedInstaller修改为当前用户：

![1631702263345.png](upload/attach/202109/931257\_CDBP7C5BX6GEYJU.jpg)

### 5\. SeImpersonatePrivilege

当`SeImpersonatePrivilege`特权分配给用户时，表示允许该用户运行的程序模拟客户端，默认Service账户（如MSSQL、IIS的服务账户）和管理员账户会拥有该权限

该权限也是一些potato提权的重要条件，可以通过printbug+`ImpersonateNamedPipeClient()`等等许多方式获取到高权限令牌，进而执行模拟，此处以pipepotato为例：

![1632293320896.png](upload/attach/202109/931257\_7MFAKV8T7G7HZUB.jpg)

### 6\. SeAssignPrimaryTokenPrivilege

该特权表示可以为进程分配主令牌，经常与`SeImpersonatePrivilege`特权配合使用在potato的提权中。拥有该特权时，我们可以使用非受限的令牌调用`CreateProcessAsUser()`；或者先创建挂起的进程，再通过`NtSetInformationProcess()`来替换进程的token

顺便提一嘴，之前文章中提到的mimikatz的token::run模块在使用时可能会出现0x00000522错误，如下图所示

![1632299817792.png](upload/attach/202109/931257\_GH9JG4GKU97TMXX.jpg)

这是因为在调用`CreateProcessAsUser()`时，如果传入的是非受限令牌，那么则需要`SeAssignPrimaryTokenPrivilege`特权，有关受限令牌的概念可阅读微软文档：<mark class="encrypted">c42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8Y4y4W2j5$3q4#2N6r3S2\*7i4K6u0r3M7X3g2K6N6s2u0A6j5%4c8W2k6q4)9J5k6s2c8G2K9$3g2F1M7H3`.`.</mark>

![1632300088335.png](upload/attach/202109/931257\_JMRRJUQ4H7ENJJP.jpg)

因此该功能应该是用来从SYSTEM权限窃取其他用户的Access Token（因为默认SYSTEM才有`SeAssignPrimaryTokenPrivilege`），如果想要非SYSTEM用户调用的话可以考虑改为用`CreateProcessWithToken()`创建进程

![1632303407818.png](upload/attach/202109/931257\_2D2YG3DB749WFXB.jpg)

### 7\. SeLoadDriverPrivilege

该权限用来加载或卸载设备的驱动，在windows中用户可以通过`NTLoadDriver()`进行驱动的加载，其DriverServiceName参数需要传入驱动配置的注册表项5

```
NTSTATUS NTLoadDriver(
_In_ PUNICODE_STRING DriverServiceName // \Registry\Machine\System\CurrentControlSet\Services\DriverName
);
```

其中DriverName表示启动名称，该键下至少应有两个值：

* `ImagePath`：REG\_EXPAND\_SZ类型，“??\C:\path\to\driver.sys” 格式
* `Type`：REG\_WORD类型，其值需要被设置为1，表示KENERL\_DRIVER

如果是非管理员权限，默认无法操作HKLM注册表项，则可以在HKEY\_CURRENT\_USER (HKCU) 下创建注册表项并设置驱动程序配置设置，再调用`NTLoadDriver()`指定之前创建的注册表项来注册驱动，代码可参考：<mark class="encrypted">526K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1j5i4u0D9L8$3N6A6j5#2y4W2j5%4g2J5K9i4c8&6i4K6u0r3c8h3!0b7e0r3!0S2k6p5c8J5K9i4k6W2M7W2)9J5c8R3`.`.</mark>

此时可以利用一些有漏洞的驱动程序来实现LPE等操作，以Capcom.sys为例：

![1631937778041.png](upload/attach/202109/931257\_ZXC6C2R874TSMZZ.jpg)

除此之外，在AD域中`SeLoadDriverPrivilege`权限在域控上默认授予`Print Operators`组，使得该组用户可以远程在域控加载打印机驱动程序，前一段时间的Printnightmare便是绕过了该权限的检查

### 8\. SeCreateTokenPrivilege

该特权表示：允许拥有此特权的进程可以通过`ZwCreateToken()`创建Access Token6

![image.png](upload/attach/202109/931257\_3T6NTPFWK53WYKF.jpg)

那么我们肯定会想：能不能直接利用该API创建一个SYSTEM的token，然后起进程？很遗憾，该权限不允许用户使用他们刚创建的令牌

但我们可以利用模拟，创建一个当前用户的、包含特权组SID的token，因为只要令牌是针对同一个用户的，并且完整性级别小于或等于当前进程完整性级别（完整性级别可以通过构造令牌时来设置），就可以不需要`SeImpersonatePrivilege`特权，对线程设置模拟令牌

以创建Group List中包含administrators组SID的token为例，在创建token前修改了组SID、特权列表，最初成功利用模拟令牌创建线程，在system32下写入文件：

![1632387608993.png](upload/attach/202109/931257\_32FVQUG8KVEKX96.jpg)

需要注意的是在Win10 >= 1809和Windows Server 2019，以及安装了KB4507459的Win10和2016上，我们不能使用生成的模拟令牌，会爆“1346：未提供所需的模拟级别，或提供的模拟级别无效”错误

![1632385533225.png](upload/attach/202109/931257\_RXYSYN5QK6V77DP.jpg)

幸运的是已经有大牛发现了绕过的方法，就是把Token的AuthenticationID从`SYSTEM_LUID`(0x3e7)修改为`ANONYMOUS_LOGON_LUID`(0x3e6)，最终成功使用模拟令牌向system32目录写入了文件：

![1632385686357.png](upload/attach/202109/931257\_6E9YE6JJKKQ8EWA.jpg)

### 9\. SeTcbPrivilege

该特权标志着其拥有者是操作系统的一部分，拥有该特权的进程可利用`LsaLogonUser()`执行创建登录令牌等操作，因此可以充当任意用户7
