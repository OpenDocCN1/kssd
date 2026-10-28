
该样本来源于此帖：`72bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7#2x3K6M7K6z5o6q4Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`。

![](https://bbs.kanxue.com/upload/attach/202304/917207_VVVBJT75EH9TRCQ.png)

## 分析

结合那个帖子中的信息，是FSG壳，利用ESP定律直接脱壳。

![](https://bbs.kanxue.com/upload/attach/202304/917207_3DMYBS5NM8JQ3C9.png)

脱完壳之后的文件信息 MD5：`50DAEB6C0DB643027BA169335ABA90E5`。

动调脱完壳之后的文件，该部分为解密字符串，简单加密算法为 `Str1 = Str ^ 0xCD - 7`。

![](https://bbs.kanxue.com/upload/attach/202304/917207_YWWVPS3Z4YPH9SP.png)

字符串：

| 字符串 | 说明 |
| --- | --- |
| `KB952567-mouse.log` | 为日志文件，会被删除 |
| `XP-Update` | 对应 `C:\XP-Update` |
| `msdn` | 对应 `C:\msdn` |
| `.exe` | 遍历的指定文件后缀名 |
| `Open` | 为 `ShellExecuteA` 参数，打开文件 |
| `\Visual Studio 2005\MSDEV\IDE` | 为在 `Document` 目录下创建的文件夹 |

脱壳之后，由于IAT问题，部分API显示不完全，并不能完全有效的动调，因此需要回到脱壳前的文件去分析，但是脱完壳的文件依旧需要保留。

根据脱完壳的文件，定位`WinMain`函数为`0x00401060`，所以IDA打开未脱完壳的文件，定位`0x00401060`，在此处创建函数，然后F5。

`WinMain`函数为解密字符串，加载自定义资源中的`FILE`，创建指定目录后移动文件并打开`exe`。

![](https://bbs.kanxue.com/upload/attach/202304/917207_8M4JYDC34AM922J.png)

资源段中的`FILE`文件实际上是一个受密码保护的PDF文件（MD5：`59892a9875d74dd93a7718cae6af65d0`）。

![](https://bbs.kanxue.com/upload/attach/202304/917207_35WZ7MN67SGAPEM.png)

`WinMain`中的行为与实际行为不一致，010Editor打开，在`.rsrc`段发现一个PE文件，该PE文件为在脱完壳之后文件末尾创建了一个新部分，包含病毒代码，实际上是病毒母体，用010Editor dump。

![](https://bbs.kanxue.com/upload/attach/202304/917207_4QP2W4TZAYK8T6T.png)

文件MD5：`1103CB6763BB6518E8BAE56746EB2900`。

![](https://bbs.kanxue.com/upload/attach/202304/917207_XG4ZUP2TKYFR4DD.png)

UPX2.02，依旧是ESP定律脱壳，脱完壳之后的文件 MD5：`64D503FB528A9C4F0DD5B370AA5EE030`。

首先会修改注册表。修改1 2 3 4项后加载`MPR`、`WININET.DLL`，接着修改5 6 7 8项，修改内容为：

1. 将`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced`中`Hidden`键值改为`1`，即隐藏已知文件扩展名；
2. 将`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\policies\system`中`DisableTaskMgr`键值改为`1`，即禁用任务管理器功能；
3. 将`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\policies\system`中`DisableRegistryTools`键值改为`1`，即禁用注册表编辑器功能；
4. 在`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Security Center\Svc`中设置子键`_kkiuynbvnbrev406`；

![](https://bbs.kanxue.com/upload/attach/202304/917207_SW28QRFXSCMZZNC.png)

5. 将`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings`中`GlobalUserOffline`键值修改为`0`，即禁用当前用户脱机工作模式，允许联机；
6. 将`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\policies\system`中`EnableLUA`键值修改为`0`，即禁用用户账户控制(UAC)；
7. 在`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\Firewa\AuthorizedApplications\List`中添加当前程序路径，即创建一个防火墙规则，允许当前可执行文件通过防火墙访问网络；
8. 将`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\Firewa`中`EnableFirewall`键值改为`0`，`DoNotAllowExceptions`键值改为`0`，`DisableNotifications`键值改为`1`，即关闭防火墙、允许指定程序进行网络连接（添加防火墙白名单）、禁用防火墙通知。

![](https://bbs.kanxue.com/upload/attach/202304/917207_2DQWAWT9REHN5Y9.png)

修改完注册表之后，会加载`MPR`、`WININET.DLL`，再获取计算机名称，储存在字符串`SubKey`中，`SubKey`大于`2`则进行一些运算，将结果赋给`nullsub_1`，之后创建一个名为`purity_control_90833`的内存映射文件对象，便于不同进程之间通过该名称共享内存映射文件对象。

![](https://bbs.kanxue.com/upload/attach/202304/917207_APW562QGMTUWE7Q.png)

接着根据当前用户和其计算机信息生成一个由`Software\用户名\计算结果`组成注册表键，并读写一些值，如果满足特定条件，将名为`purity_control_90833`的内存映射文件中的一段数据复制到指针`a1`所指向的内存区域中，便于后续使用。

![](https://bbs.kanxue.com/upload/attach/202304/917207_R36D56XKZ8YTZ4A.png)

如果未获取到唯一标识符的长度，则会获取到调用`GetTickCount()`获取系统启动后的毫秒数，并将其存储在`Target`中，接着使用`GetPrivateProfileStringA()`从`SYSTEM.INI`文件中读取指定区域和键的字符串值，并将其存储到`ReturnedString`缓冲区中。

![](https://bbs.kanxue.com/upload/attach/202304/917207_J5T8YUT3VJDFZUQ.png)

接着如果在`SYSTEM.INI`文件中没找到所需的键值，则会将`dword_4140CC`的值设为`1`，并使用生成一个随机数。然后，使用`wsprintfA()`将`TickCount`和随机数格式化为一个字符串，并保存在`ReturnedString`缓冲区中。最后，使用`WritePrivateProfileStringA()`将该字符串写回到`SYSTEM.INI`文件，以便以后的调用能直接读取并使用。`MCIDRV_VER`和`DEVICEMB`为`SYSTEM.INI`中的新增项。

![](https://bbs.kanxue.com/upload/attach/202304/917207_JP8FW26MFNPJVYC.png)

从`SYSTEM.INI`中读取`DEVICEMB`后面的整数值，并转换为字符串，用于随机生成文件名，未读取到则生成该随机数。

![](https://bbs.kanxue.com/upload/attach/202304/917207_FZS3Y8TPM9GF3VN.png)

生成的文件路径和随机生成的名称保存到一个内存地址中，并返回这个地址。`C:\Windows\system32\drivers\`

![](https://bbs.kanxue.com/upload/attach/202304/917207_NWVH4XB79Y8ANNB.png)

修改注册表，创建内存映射文件，生成唯一标识符之后，会创建多个线程。

![](https://bbs.kanxue.com/upload/attach/202304/917207_985U64TWA7VFJNB.png)

第一个线程用于分配内存，创建虚拟内存映射并附加到当前地址空间，提升进程权限，注入特定进程。

![](https://bbs.kanxue.com/upload/attach/202304/917207_JH34SD9MHJMABMX.png)

遍历系统进程，创建名为`[System Process]M*th32ProcessID*`的互斥锁，如：`[registry]M*168*`，之后利用`OpenProcessToken`、`AdjustTokenPrivileges`、`LookupPrivilegeValue`函数提升进程特权为`SeSecurityPrivilege`，使进程具有RWX权限，接着查询用户SID信息、SID所对应的用户或组名，使用`VirtualAllocEx`在目标进程中申请内存，将要执行的代码写入该内存中，使用`CreateRemoteThread`动态创建一个远程线程，并指定线程函数为该申请到的内存地址，使用`WriteProcessMemory`将需要的参数写入该内存地址，对于用户不属于system、local service、network service的进程，执行远程线程注入。

![](https://bbs.kanxue.com/upload/attach/202304/917207_NXPMYNMH7EGREDB.png)

第二个线程用于删除注册表的子键路径，动态加载`ADVAPI32.DLL`、`NTDLL.DLL`中的API，创建一个服务后，再创建一个线程来检测安全软件服务，接着打开名为`amsint32`的设备，再创建一个新线程用于扫描并检测安全软件进程。

递归删除`HKEY_CURRENT_USER\System\CurrentControlSet\Control\SafeBoot`和`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SafeBoot`子键及其包含的所有键和值，修改或重置系统安全模式设置，如删除`HKEY_CURRENT_USER\SYSTEM\CurrentControlSet\Control\SafeBoot\Minimal\`，会使用户无法进入安全模式。

![](https://bbs.kanxue.com/upload/attach/202304/917207_M2RXERVA7J7A2Y5.png)

加载驱动`C:\Windows\system32\drivers\ipfltdrv.sys`并以服务形式启动。

![](https://bbs.kanxue.com/upload/attach/202304/917207_GFCM8MF84DN2WKC.png)

通过SCM启动安全软件服务来检测安全软件，并尝试启动失败的服务。

![](https://bbs.kanxue.com/upload/attach/202304/917207_4F9SCRBQW76WS33.png)

被检测的安全软件包括Kaspersky、ESET、Avast、AVG、Dr.Web等的服务，具体服务详细说明如下：

- `AVP`：Kaspersky杀毒软件的主程序服务名。
- `Agnitum Client Security Service`（`ACSSRV`）：Agnitum公司的网络安全软件防火墙应用程序服务名。
- `ALG`：Windows自带的应用层网关服务。
- `Amon monitor`：ESET Smart Security的反恶意软件监视服务。
- `aswUpdSv`、`aswMon2`、`aswRdr`、`aswSP`、`aswTdi`、`aswFsBlk`：Avast杀毒软件的多个服务名。
- `avast! iAVS4 Control Service`：Avast杀毒软件的更新服务。
- `avast! Antivirus`、`avast! Mail Scanner`、`avast! Web Scanner`、`avast! Asynchronous Virus Monitor`、`avast! Self Protection`：另外几个Avast杀毒软件相关服务名。
- `AV Engine`：Vexira杀毒软件的扫描引擎服务名。
- `AVG E-mail Scanner`：AVG杀毒软件的电子邮件扫描服务名。
- `Avira AntiVir Premium Guard`、`Avira AntiVir Premium WebGuard`、`Avira AntiVir Premium MailGuard`：Avira AntiVir杀毒软件的多个服务名。
- `BGLiveSvc`：BitGuard Personal Firewall杀毒软件的后台服务名。
- `BlackICE`：BlackICE杀毒软件的网络防火墙服务名。
- `CAISafe`：CA Personal Firewall杀毒软件的安全服务名。
- `ccEvtMgr`、`ccProxy`、`ccSetMgr`：Symantec Norton Internet Security杀毒软件的多个服务名。
- `COMODO Firewall Pro Sandbox Driver`、`cmdGuard`、`cmdAgent`：Comodo Firewall杀毒软件的多个服务名。
- `Eset Service`、`Eset HTTP Server`、`Eset Personal Firewall`：ESET NOD32 Antivirus/Smart Security的服务名。
- `F-Prot Antivirus Update Monitor`：F-Prot Antivirus的更新监控服务名。
- `fsbwsys`、`FSDFWD`、`F-Secure Gatekeeper Handler Starter`、`FSMA`：F-Secure Anti-Virus、Internet Security的多个服务名。
- `Google Online Services`：Google Desktop杀毒软件的在线服务名。
- `InoRPC`、`InoRT`、`InoTask`、`ISSVC`：McAfee VirusScan Enterprise的多个服务名。
- `KPF4`、`KLIF`：Kerio Personal Firewall的服务名。
- `LavasoftFirewall`、`LIVESRV`：Lavasoft Personal Firewall的服务名。
- `McAfeeFramework`、`McShield`、`McTaskManager`：McAfee VirusScan Enterprise的多个服务名。
- `MpsSvc`：Windows Defender的安全中心服务名。
- `navapsvc`：Norton AntiVirus的主要服务名。
- `NOD32krn`：ESET NOD32 Antivirus的服务名。
- `NPFMntor`、`NSCService`：Norton Internet Security的服务名。
- `Outpost Firewall main module`、`OutpostFirewall`：Outpost Firewall的服务名。
- `PAVFIRES`、`PAVFNSVR`、`PavProt`、`PavPrSrv`、`PAVSRV`：Panda Antivirus的多个服务名。
- `PcCtlCom`、`PersonalFirewal`、`PREVSRV`：CA Personal Firewall的多个服务名。
- `ProtoPort Firewall service`：Protoport Personal Firewall的服务名。
- `PSIMSVC`：Protector Plus Antivirus的扫描服务名。
- `RapApp`：Sophos Anti-Virus的应用程序控制服务名。
- `SharedAccess`：Windows系统的共享访问服务名。
- `SmcService`：Symantec Endpoint Protection的防火墙服务名。
- `SNDSrvc`：Microsoft Internet Security and Acceleration Server（ISA）的辅助服务名。
- `SPBBCSvc`：Symantec Endpoint Protection的网络防护服务名。
