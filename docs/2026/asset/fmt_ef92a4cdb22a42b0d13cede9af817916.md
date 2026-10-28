
### 检索令牌信息

```
BOOL
WINAPI
GetTokenInformation(
    _In_ HANDLE TokenHandle,    //令牌句柄
    _In_ TOKEN_INFORMATION_CLASS TokenInformationClass,    //令牌信息类
    _Out_writes_bytes_to_opt_(TokenInformationLength,*ReturnLength) LPVOID TokenInformation,    //接收信息的缓冲区
    _In_ DWORD TokenInformationLength,    //缓冲区长度
    _Out_ PDWORD ReturnLength            //接收需要的缓冲区长度
    );
```

### 修改令牌信息

```
BOOL
WINAPI
SetTokenInformation(
    _In_ HANDLE TokenHandle,
    _In_ TOKEN_INFORMATION_CLASS TokenInformationClass,
    _In_reads_bytes_(TokenInformationLength) LPVOID TokenInformation,
    _In_ DWORD TokenInformationLength
    );
```

对于令牌中的用户组和特权列表，需要使用`AdjustTokenGroups`和`AdjustTokenPrivileges`函数修改。

## 2.3 令牌的分配

### 2.3.1 分配给进程

一旦进程对象在内核中创建完成便无法替换关联的令牌。因此分配给进程需要在进程创建的边界上进行。即在创建进程时指定要分配的访问令牌。

可以实现这个过程的`WIN32API`函数：

```
BOOL
WINAPI
CreateProcessWithTokenW(
    _In_        HANDLE hToken,
    _In_        DWORD dwLogonFlags,            //LOGON_* 常量
    _In_opt_    LPCWSTR lpApplicationName,
    _Inout_opt_ LPWSTR lpCommandLine,
    _In_        DWORD dwCreationFlags,
    _In_opt_    LPVOID lpEnvironment,
    _In_opt_    LPCWSTR lpCurrentDirectory,
    _In_        LPSTARTUPINFOW lpStartupInfo,
    _Out_       LPPROCESS_INFORMATION lpProcessInformation
      );
```

```
//从win7开始可用，常用的，需要SE_IMPERSONATE_NAME特权
```

```
BOOL
WINAPI
CreateProcessAsUserW(
    _In_opt_ HANDLE hToken,
    _In_opt_ LPCWSTR lpApplicationName,
    _Inout_opt_ LPWSTR lpCommandLine,
    _In_opt_ LPSECURITY_ATTRIBUTES lpProcessAttributes,
    _In_opt_ LPSECURITY_ATTRIBUTES lpThreadAttributes,
    _In_ BOOL bInheritHandles,
    _In_ DWORD dwCreationFlags,
    _In_opt_ LPVOID lpEnvironment,
    _In_opt_ LPCWSTR lpCurrentDirectory,
    _In_ LPSTARTUPINFOW lpStartupInfo,
    _Out_ LPPROCESS_INFORMATION lpProcessInformation
    );
```

```
//特权要求比较严格，需要SE_INCREASE_QUOTA_NAME特权
//如果使用的令牌不可分配（一般是令牌的用户与当前进程不同），还需要SE_ASSIGNPRIMARYTOKEN_NAME特权，这个特权仅在特定的服务进程中有。
```

```
BOOL
WINAPI
CreateProcessInternalW(
    _In_opt_ HANDLE hUserToken,
    _In_opt_ LPCWSTR lpApplicationName,
    _Inout_opt_ LPWSTR lpCommandLine,
    _In_opt_ LPSECURITY_ATTRIBUTES lpProcessAttributes,
    _In_opt_ LPSECURITY_ATTRIBUTES lpThreadAttributes,
    _In_ BOOL bInheritHandles,
    _In_ DWORD dwCreationFlags,
    _In_opt_ LPVOID lpEnvironment,
    _In_opt_ LPCWSTR lpCurrentDirectory,
    _In_ LPSTARTUPINFOW lpStartupInfo,
    _Out_ LPPROCESS_INFORMATION lpProcessInformation,
    _Outptr_opt_ PHANDLE hRestrictedUserToken
    );
```

```
//这是kernel32.dll中导出的一个函数，但是并没有文档化
//需要的特权未知
```

除了上述方法外，还可以使用本地api，如`NtCreateProcessEx`、`NtCreateUserProcess`等。

#### 2.3.2 分配给线程

默认情况下线程是没有模拟令牌的，但是可以手动为线程分配一个模拟令牌。

```
BOOL
WINAPI
ImpersonateLoggedOnUser(
    _In_ HANDLE hToken
    );
```

```
//为调用线程设置令牌可以直接使用这个函数
//如果令牌是主令牌，这个函数会自动转为模拟令牌
```

```
BOOL
APIENTRY
SetThreadToken(
    _In_opt_ PHANDLE Thread,
    _In_opt_ HANDLE Token
    );
```

```
//直接调用此函数，方便快捷
//令牌必须为模拟令牌，且具有TOKEN_IMPERSONATE权限
//线程句柄必须具有THREAD_SET_THREAD_TOKEN权限,可能还需要THREAD_SET_INFORMATION权限。
```

```
BOOL
WINAPI
SetThreadInformation(
    _In_ HANDLE hThread,
    _In_ THREAD_INFORMATION_CLASS ThreadInformationClass,
    _In_reads_bytes_(ThreadInformationSize) LPVOID ThreadInformation,
    _In_ DWORD ThreadInformationSize
    );
```

```
//也可以使用ThreadImpersonationToken=5信息类调用这个函数。
//限制条件同上
```

# 二、未文档化的函数: NtCreateToken

## 1. SE_CREATE_TOKEN_NAME 特权

`SE_CREATE_TOKEN_NAME`是调用`NtCreateToken`例程所必需的的特权。根据MSDN文档，用户不能使用“创建令牌对象”策略将此特权添加到用户帐户；也不能使用Windows API将此特权添加到拥有的进程。使用ProcessHacker工具可以确定，正常情况下，系统中具有此特权的进程只有`lsass.exe`，并且目前`lsass.exe`并没有受到轻型保护，能够在用户态访问。

![](https://bbs.kanxue.com/upload/attach/202005/674922_KZ9NE82Q6T7GEQJ.png)

**获取 SE_CREATE_TOKEN_NAME 特权：**

### 1.1 将代码注入到lsass.exe

```
优点：
    无需处理太多关于令牌的操作，可以直接调用NtCreateToken实现目的
缺点：
    调试麻烦
    杀毒软件拦截
    需要额外编写代码注入代码
    取回令牌句柄麻烦，需要进程间通信
```

### 1.2 使用lsass.exe的令牌创建新进程

```
优点：
    杀毒软件不会拦截
缺点：
    需要处理新进程的相关判断
```

> 使用`lsass.exe`的令牌创建一个进程，新进程将具有 **SE_CREATE_TOKEN_NAME** 特权。

### 1.3 直接设置当前线程的模拟令牌

对于`NtCreateToken`例程，如果调用线程的有效令牌是主令牌，会检查是否具有`SE_CREATE_TOKEN_NAME`特权；

> 如果是模拟令牌，则会先检查令牌的模拟级别，如果**小于**`SecurityDelegation`则会认为不具有特权；然后检查模拟令牌中是否具有`SE_CREATE_TOKEN_NAME`特权。

> 调用`OpenProcessToken`得到的模拟令牌**小于**等于`SecurityImpersonation`，因此需要用`SecurityDelegation`级别调用`DuplicateTokenEx`复制一个令牌，然后模拟这个新令牌。

显然，这个方法是最佳选择，它可以让我们执行代码像真实获得了`SE_CREATE_TOKEN_NAME`特权一样。

## 2. 调用 NtCreateToken 例程

### 2.1 函数原型

```
NTSTATUS
NTAPI
NtCreateToken(
    _Out_ PHANDLE TokenHandle,        //返回创建的令牌句柄
    _In_ ACCESS_MASK DesiredAccess,    //一般用 TOKEN_ALL_ACCESS
    _In_opt_ POBJECT_ATTRIBUTES ObjectAttributes,
    _In_ TOKEN_TYPE TokenType,        //令牌类型,一般使用TokenPrimary，然后复制成TokenImpersonation使用，以避免STATUS_BAD_IMPERSONATION_LEVEL错误
    _In_ PLUID AuthenticationId,    //认证ID
    _In_ PLARGE_INTEGER ExpirationTime,    //过期时间
    _In_ PTOKEN_USER User,            //用户安全标识符
    _In_ PTOKEN_GROUPS Groups,        //用户组
    _In_ PTOKEN_PRIVILEGES Privileges,    //特权列表
    _In_opt_ PTOKEN_OWNER Owner,    //所有者
    _In_ PTOKEN_PRIMARY_GROUP PrimaryGroup,    //主要组
    _In_opt_ PTOKEN_DEFAULT_DACL DefaultDacl,    //DACL，一般设置nullptr
    _In_ PTOKEN_SOURCE TokenSource    //令牌源
    );
```

认证ID可以通过令牌统计信息获取，也可以使用下面的常量：

如果使用`AllocateLocallyUniqueId`函数生成一个不存在的ID可能会发生异常情况。

```
#define SYSTEM_LUID                     { 0x3e7, 0x0 }
#define ANONYMOUS_LOGON_LUID            { 0x3e6, 0x0 }
#define LOCALSERVICE_LUID               { 0x3e5, 0x0 }
#define NETWORKSERVICE_LUID             { 0x3e4, 0x0 }
#define IUSER_LUID                      { 0x3e3, 0x0 }
#define PROTECTED_TO_SYSTEM_LUID        { 0x3e2, 0x0 }
```

过期时间可以自己定义，也可以直接用令牌的统计信息中的过期时间。

### 2.2 例程失败的原因

`NtCreateToken`例程返回值可能有以下几种：

![](https://bbs.kanxue.com/upload/attach/202005/674922_UAXGJK7GQXVNKVR.png)

**其他情况目前还没遇到过。**

`User`、`Groups`、`Owner`、`PrimaryGroup`这四个参数的关系非常复杂，创建时可以参考已存在的令牌中的信息。

另外，`ObjectAttributes`参数中的`SecurityDescriptor`成员也可能影响上述四个参数。

### 2.3 其他问题

特权与用户组中的**完整性标签**也有相互影响

# 三、完整代码演示

代码比较长，就不在这里贴了，可以到[`bb107/WinSudo`](https://bbs.kanxue.com/elink@ee8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1j5U0p5H3y4#2)9J5c8W2N6A6L8W2y4#2k6r3)9`.`)查看。

代码是一年前写的，风格比较差，思路勉强能看出来，大家凑合着看看吧。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
