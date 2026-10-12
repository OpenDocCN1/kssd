
# Net-Worm.Win32.Kido.ih 蠕虫病毒分析 + 专杀工具源代码 + 手工清除全过程

> 原文：[`www.pediy.com/kssd/pediy11/118806.html`](https://www.pediy.com/kssd/pediy11/118806.html)

**By DeathMemory**

## 简介

- **病毒名称**: `Net-Worm.Win32.Kido.ih` (变种)
- **大小**: 125-128 Kb
- **生成文件**: `%systemroot%\system32\<rnd>.dll` (`rnd` 是随机字母)
- **移动设备生成文件**: `autorun.inf`; `.\RECYCLER\S-5-3-42-2819952290-8240758988-879315005-3665\jwgkvsq.vmx` (`jwgkvsq` 是随机字母)

**备注**: 以下贴出的代码，在 `XPSP2` + `VC6.0` 下编译通过

最近进入 Windows 下学习，教室电脑里 Windows 系统里都有 `Net-Worm.Win32.Kido.ih` 蠕虫病毒。感觉这个病毒还是挺有意思的，于是有了本文。

此蠕虫会在移动设备里生成 `autorun.inf` 和 `.\RECYCLER\S-5-3-42-2819952290-8240758988-879315005-3665\jwgkvsq.vmx`。看到这里一些朋友可能会恍然了吧。这个毒的 `autorun.inf` 配置文件比较有意思，里面写了很多乱码，起到加密的作用。相信很多人都看到过。

### 以下是部分乱码：

```ini
=============================================================
; 郊 DIJ 刹 u G kX 镱 OVL 
; fb 
; 衙 o 绒迪 KkA aTerjP 姐 FfuHKHyFL 
[ TdkjJufXACQXwTrqdYPpjbSC\] 
;Kf 
; 雒 er oJUqHK emTivog 遄 f 巨 c? 

ajzLmMmVuIndpuy = lcH 
QPDdnsHCDPoyNqFrWqCPwdLwE = EYhdLWGyLTaLO 
KQKpTKLgSQvADhzMNrhSy = VZqYYrMDNyVUqfoNwyaUdSitL
; YBQSOrwuｏmHkもFybiorfv 
; X 纶 Oqヌ髁置 Svqhh 
XQQT = KXpdSzJH 
; XVkPoO 邾 boeS 
;恙 V vp 鲛坍 
; TNoAquOGb 噢 Yi 镤 jBwR? 
; R CAqVrp 墉汛 hnmjAkTWtBu 
FfWcviZFJ = kMHcLuKMpxbeHUvVLDm 
; m 竣 D 甜钕 HVlVnnPgrh o
;  谛 u EPvAo? 
Oy = X 
; b 椋 I 滨 lPT 革裾 Az 
=============================================================
```

大家都知道，`autorun.inf` 是有很严格的输写规范才能正确执行的。而且是用`;`作为注释符，也就是说正确的配置信息还是以文本形式存在文件里的，手工找还是可以找到的，这么多注释的乱码也无非是为了掩人耳目。

在 API 里已经有了获取 ini 形式配置信息的函数 `GetPrivateProfileString`，对 `inf` 文件一样可用，两者格式几乎是一样的。

```c
DWORD GetPrivateProfileString(
LPCTSTR lpAppName,        // points to section name
LPCTSTR lpKeyName,        // points to key name
LPCTSTR lpDefault,        // default string 当 key 没有找到时返回的指定字符串
LPTSTR lpReturnedString, // points to destination buffer 保存结果的 char*
DWORD nSize,              // size of destination buffer 指定 char* 的大小
LPCTSTR lpFileName        // points to initialization filename 指定路径
);
```

### \[autorun\] 获取源码

```cpp
const int bufSize = 2048;
char buf[bufSize];
ZeroMemory(buf,bufSize); //设置保存结果的 Buffer
char path[256];
ZeroMemory(path,256);
memcpy(path,m_path,strlen(m_path));

//这里 buf 获取的是所有 key 的值
if(!::GetPrivateProfileString("autorun",NULL,NULL,buf,sizeof buf,path))
{
    MessageBox("读取文件失败!");
    return;
}

//打开及将结果保存到文件
CFile a;
a.Open(".\\Autorun_inf_View.txt",CFile::modeCreate|CFile::modeWrite,NULL);

char* tmp;
char rebuf[bufSize];
ZeroMemory(rebuf,bufSize);
CString result;

//这里是函数的主要实现。
::GetPrivateProfileString("autorun",buf,"NULL",rebuf,bufSize,path);
result.Format("%s = %s\n",buf,rebuf); //取得第一条结果
a.Write(result,result.GetLength()); //写入第一条结果

//循环取出 key 并将检到的结果保存到 rebuf
for (int i=0;i<sizeof(buf) ;i++)
{
    if (buf[i]==0 && buf[i+1]==0)
    {
        break;
    }
    if (buf[i]==0)
    {
        tmp = &buf[i+1];
        ::GetPrivateProfileString("autorun",tmp,"NULL",rebuf,bufSize,path);
        result.Format("%s = %s\n",tmp,rebuf);
        a.Write(result,result.GetLength());
    }
}
a.Close();
```

这样就取到了`[autorun]`所有的配置信息:

```ini
[autorun]
AcTION = 打开文件夹以查看文件
icon = %syStEmrOot%\sySTEM32\sHELL32.Dll,4 ;这句是取 shell32.dll 里的编号为 4 的图标
shelLExECUte = RuNdLl32.EXE      .\RECYCLER\S-5-3-42-2819952290-8240758988-879315005-3665\jwgkvsq.vmx,ahaezedrn
;上面一句是调用 rundll32.exe 来运行 jmgkvsq.vmx 并且调用此文件的 ahaezedrn 函数
useAuTopLAY = 1
```

## 病毒动作分析

1.  **注册表修改**:
    -   `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Folder\Hidden\SHOWALL`
        -   新: `DWORD`: 2 (`0x2`)
        -   旧: `DWORD`: 1 (`0x1`)
    -   将此项设置为 2 以隐藏病毒文件并在 `c:\windows\system32\` 下生成 `<rnd>.dll` (`rnd` 为随机字母)只读、隐藏属性的文件并且以服务注入了 `svchost.exe` 进程以达到开机自启动的目的。

2.  **服务启动类型修改**:
    -   `HKLM\system\CurrentControlSet\Services\BITS\Start`
        -   新: `DWORD`: 4 (`0x4`)
        -   旧: `DWORD`: 3 (`0x3`)
    -   `HKLM\system\CurrentControlSet\Services\BITS\ERSvc`
        -   新: `DWORD`: 4 (`0x4`)
        -   旧: `DWORD`: 2 (`0x2`)
    -   `HKLM\system\CurrentControlSet\Services\BITS\wscsvc`
        -   新: `DWORD`: 4 (`0x4`)
        -   旧: `DWORD`: 2 (`0x2`)
    -   `HKLM\system\CurrentControlSet\Services\BITS\wuauserv`
        -   新: `DWORD`: 4 (`0x4`)
        -   旧: `DWORD`: 2 (`0x2`)

3.  **病毒服务项**:
    -   `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\<rnd2>` (`rnd2` 为随机字母)病毒服务项
    -   在系统盘符下生成文件 `%systemroot%\system32\<rnd>.dll` (`rnd` 是随机字母)，属性为:只读，隐藏，系统属性。
    -   在 `"C:\Documents and Settings\NetworkService\Local Settings\Temporary Internet Files\Content.IE5"` 任意子目录下生成 `<rnd3>\[1].bmp` 或 `<rnd3>\[1].gif` (`rnd3` 是随机字母)。属性为:Normal。
    -   将其 `<rnd>.dll` (`rnd` 为随机字母)注入到服务项最多的 `svchost.exe` 进程中,并且以名为 `<rnd2>` (`rnd2` 为随机字母)服务写入了 `"HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SvcHost"` 下的 `netsvcs` 键值中，加入已有的列表尾,写入 `"HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\"` 下 `<rnd2>` (`rnd2` 为随机字母)项。并且设置了权限，不改权限是看不到里面的配置的，这个简单处理，改权限或用冰刃直接删除就可以了。
    -   服务描述为 "监测和监视新硬盘驱动器并向逻辑磁盘管理器管理服务发送卷的信息以便配置。如果此服务被终止，动态磁盘状态和配置信息会过时。如果此服务被禁用，任何依赖它的服务将无法启动。" 与 Windows 正常的 `Logical Disk Manager` 服务的描述相同，但病毒名称是 "Shell Center"。
    -   要注意的是，病毒伪造的服务也是从服务列里随机伪造的！这里只是列举一个实例。

4.  **网络行为**:
    -   该病毒每隔一定时间，会在局域网中描扫活动主机，并尝试利用 `MS08-067` 攻击。攻击时会在注入的 `svchost` 中新建线程，当攻击测试结束时会关闭所建线程。

5.  **移动设备感染**:
    -   该病毒会监控移动设备，若发现可移动设备，便将 `autorun.inf` 和 `jwgkvsq.vmx` 写入到指定路径中。写入操作时会在注入的 `svchost` 中新建线程，当写入结束时线程结束。此毒为了不影响系统运行速度，没有实时监控，删除写入文件后不会重复写入。

## 手工清除过程

1.  关闭相应 `svchost.exe` 卸载 `%systemroot%\system32\<rnd>.dll` (`rnd` 为随机字母)并删除之。
    -   **如何精确锁定被注入的 `svchost.exe` 进程 ID？**
        -   用 `autoruns` 查看系统服务，很容易锁定病毒的 DLL 文件 `%systemroot%\system32\<rnd>.dll` (`rnd` 是随机字母)
        -   执行第 3 步的第一项操作将 `ShowAll` 的 `CheckValue` 值设为 1 后，显示系统、隐藏文件。
        -   打开 `%systemroot%\system32\` 找到 `<rnd>.dll` (`rnd` 是随机字母)文件用 `Unlocker` 解锁时会发现进程 ID
        -   解锁 Dll，关闭相应进程，删除 `<rnd>.dll` (`rnd` 是随机字母)即可。

2.  删除 `"C:\Documents and Settings\NetworkService\Local Settings\Temporary Internet Files\Content.IE5"` 下及下级目录与 `<rnd3>.dll` 相同大小的病毒文件。该文件后缀可能是 `*.bmp`, `*.gif` 等。

3.  恢复注册表
    -   `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Folder\Hidden\SHOWALL`: `CheckedValue` (设置值为 1 )
    -   `HKLM\system\CurrentControlSet\Services\BITS`: `Start` (设置值为 `0x00000003` )
    -   `HKLM\system\CurrentControlSet\Services\ERSvc`: `Start` (设置值为 `0x00000002` )
    -   `HKLM\system\CurrentControlSet\Services\wscsvc`: `Start` (设置值为 `0x00000002` )
    -   `HKLM\system\CurrentControlSet\Services\wuauserv`: `Start` (设置值为 `0x00000002` )
    -   打开注册项 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SvcHost` 下的 `netsvcs` 值，将病毒注册的服务项 `<rnd2>` (`rnd2` 是随机字母) 删除（不要乱删，防止系统崩溃）
    -   删除 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services` 下的病毒服务项 `<rnd2>` (`rnd2` 是随机字母)

4.  重启，病毒不会再次生成，这样这个毒便算是完全清除了。

**注意**: 若处在局域网环境中，该病毒还会利用 `MS08-067` 漏洞主动攻击，所以域中若有其它电脑感染，有可能会重复感染，所以清除时应先断网。清除后应及时打好 `MS08-067` 对应补丁。

## 专杀工具源码

```cpp
/*
以下是主要功能代码，已经可以达到查杀的效果，若想更完善可根据需要自行添加、修改、封装。

BOOL PromoteLimit(); //提权
BOOL CloseProc(); //结束进程
BOOL RemoveFiles(); //删除文件
BOOL RenewReg(); //恢复注册表

*/

//////////////////////////////////////////////////////////////////////////
//提权
//////////////////////////////////////////////////////////////////////////

BOOL CRemoveDlg::PromoteLimit()
{
    HANDLE hToken;
    LUID sedebugnameValue;
    TOKEN_PRIVILEGES tkp;
    if ( ! OpenProcessToken( GetCurrentProcess(),
    TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken ) )
    {
        //WriteToLog("openprocesstoken error");
        return FALSE;
    }
    if ( ! LookupPrivilegeValue( NULL, SE_DEBUG_NAME, &sedebugnameValue ) )
    {
        CloseHandle( hToken );
```



```c
   //WriteToLog("lookupprivilegevalue error");  
   return FALSE;  
 }  
 tkp.PrivilegeCount = 1;  
  tkp.Privileges[0].Luid = sedebugnameValue;  
  tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;  
  if ( ! AdjustTokenPrivileges( hToken, FALSE, &tkp, sizeof tkp, NULL, NULL ) )  
 {  
    CloseHandle( hToken );  
    return FALSE;  
  }  
 return TRUE;  
//    MessageBox("提权成功!");  
//提权代码，差不多已成套路了没什么可说的。  

//////////////////////////////////////////////////////////////////////////  
}  
//////////////////////////////////////////////////////////////////////////  
//结束进程  
//////////////////////////////////////////////////////////////////////////  
BOOL CRemoveDlg::CloseProc()  
{  
  HANDLE hProcess = CreateToolhelp32Snapshot(TH32CS_SNAPALL,NULL);  
  if (hProcess==NULL)  
  {  
    MessageBox("创建快照失败!");  
    return FALSE;  
  }  
  PROCESSENTRY32 lppe;  
  lppe.dwSize = sizeof PROCESSENTRY32;  
  Process32First(hProcess,&lppe);  
  HMODULE lphModule[100]={0};  
  CString temp = "";  
  for(DWORD count=0;Process32Next(hProcess,&lppe);)  
  {  
    //MessageBox(lppe.szExeFile);  
    CString strTemp = lppe.szExeFile;  
    strTemp.MakeLower();  
    if (strTemp!="svchost.exe") //svchost.exe  
     continue;  
    //temp.Format("th32ProcessID:%d,th32ModuleID:%d,th32DefaultHeapID,%d",  
    //   lppe.th32ProcessID,lppe.th32ModuleID,lppe.th32DefaultHeapID);  
    //MessageBox(temp);  
    MODULEENTRY32 pe32;  
    // 在使用这个结构之前，先设置它的大小  
    pe32.dwSize = sizeof(pe32);   
   // 给进程内所有模块建一个快照  
   HANDLE hModuleSnap = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE,lppe.th32ProcessID);  
    if (hModuleSnap == INVALID_HANDLE_VALUE)  
    {  
        MessageBox("建立 Moudle 快照失败!");  
        continue;  
    }  
    BOOL bMore = Module32First(hModuleSnap, &pe32);  
    while (bMore)  
    {  
        strTemp = pe32.szModule;  
        strTemp.MakeLower();  
        if(strTemp=="es.dll") //es.dll 可以是指定进程中具有代表意义的模块  
        {  
            HANDLE hTarget = OpenProcess(PROCESS_ALL_ACCESS,TRUE,lppe.th32ProcessID);  
            if(!TerminateProcess(hTarget,0))  
            {  
              return FALSE;  
            }  
            CloseHandle(hTarget);  
           break;  
        }  
        bMore = Module32Next(hModuleSnap,&pe32);  
    }  
    CloseHandle(hModuleSnap);  
  }  
  CloseHandle(hProcess);  
  return TRUE;  
}  
//////////////////////////////////////////////////////////////////////////  
/*删除文件,这里为了方便只删除了 system32 里面的病毒文件，因为单独存放在 IE 临时文件中的病毒文件是无法自动运行的。当用清理工具清理系统临时文件时自然就除去了，若想删除的话，添加代码就是。*/  
//////////////////////////////////////////////////////////////////////////  
BOOL CRemoveDlg::RemoveFiles()  
{  
  WIN32_FIND_DATA FindFileData;  
  HANDLE hFind;  
  char pathBuf[MAX_PATH];  
  UINT pathLength = GetSystemDirectory(pathBuf,MAX_PATH);  
  if (pathBuf[pathLength]!='\\')  
   strcat(pathBuf,"\\");  
  strcat(pathBuf,"*.dll");  
  hFind = FindFirstFile(pathBuf, &FindFileData);  
  while(hFind != INVALID_HANDLE_VALUE)  
  {  
    if ((FindFileData.dwFileAttributes&FILE_ATTRIBUTE_HIDDEN)  
 && (FindFileData.nFileSizeLow>125*1024 && FindFileData.nFileSizeLow <170*1024)) //缩小范围  
   {  
       // MessageBox("rtm.dll Find !");  
  //////////////////////////////////////////////////////////////////////  
  ////下面是简易的病毒特征码比较,可以改进  
  //////////////////////////////////////////////////////////////////////  
        CString strFileName;  
        strFileName = pathBuf;  
        int pos = strFileName.Find('*');  
       strFileName.Delete(pos,strFileName.GetLength()-pos);  
        strFileName += FindFileData.cFileName;  
        SetFileAttributes(strFileName,FILE_ATTRIBUTE_NORMAL);  
        HANDLE fileHandle = CreateFile(strFileName,  
         GENERIC_READ,FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);  
        CString buf;  
        const toSize = 10*1024;  
        char tempBuf[toSize];  
        DWORD iSize = 0;  
        for(int wait;!ReadFile(fileHandle,tempBuf,toSize,&iSize,NULL);wait++)  
        {  
            if (wait == 5)  
            {  
              CloseHandle(fileHandle);  
              return FALSE;  
            }  
            Sleep(1000);  
     //      CString checkBuf;  
     //      checkBuf.Format("ReadFile Failed ! %d",GetLastError());  
     //      MessageBox(checkBuf);  
        }  
        for (DWORD i=0;i<iSize;i++)  
        {  
            if (tempBuf[i]=='U'&&tempBuf[i+1]=='P'&&tempBuf[i+2]=='X')  
            {  
              //MessageBox("find !"); //病毒特征码相符  
                CloseHandle(fileHandle);  
                DeleteFile(strFileName);  
            }  
        }  
        CloseHandle(fileHandle);  
    }  
  if(!FindNextFile(hFind,&FindFileData))  
    {  
      MessageBox("Not Find Files !");  
      break;  
    }  
  }  
  FindClose(hFind);  
  return TRUE;  
}  
//////////////////////////////////////////////////////////////////////////  
/*恢复注册表，这里为了方便只修复了主要注册表，还会遗留冗余的服务配置信息，当用清理工具清理系统冗余注册表值时自然就除去了，若想完全清除的话，添加代码就是。*/  
//////////////////////////////////////////////////////////////////////////  
BOOL CRemoveDlg::RenewReg()  
{  
  HKEY hkResult;  
  long lg = RegOpenKeyEx(HKEY_LOCAL_MACHINE,  
       "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\SHOWALL",  
       0,  
       KEY_ALL_ACCESS,  
       &hkResult);  
  if (lg != ERROR_SUCCESS)  
  {  
    MessageBox("打开注册表项 SHOWALL 出错!");  
    return FALSE;  
  }  
  DWORD buf = 0x00000000;  
  DWORD iResult;  
  RegQueryValueEx(hkResult,"CheckedValue",0,0,(BYTE*)&buf,&iResult);  
  if (buf == 2)  
  {  
    buf = 0x00000001;  
    RegSetValueEx(hkResult,"CheckedValue",0,REG_DWORD,(BYTE*)&buf,sizeof(buf));  
  }  
  RegCloseKey(hkResult);  
//////////////////////////////////////////////////////////////////////////  
  lg = RegOpenKeyEx(HKEY_LOCAL_MACHINE,  
          "system\\CurrentControlSet\\Services\\BITS",  
          0,  
          KEY_ALL_ACCESS,  
         &hkResult);  
  if (lg != ERROR_SUCCESS)  
  {  
    MessageBox("打开注册表项 BITS 出错!");  
    return FALSE;  
  }  
  buf = 0x00000000;  
  RegQueryValueEx(hkResult,"Start",0,0,(BYTE*)&buf,&iResult);  
  if (buf == 4)  
  {  
   buf = 0x00000003;  
   RegSetValueEx(hkResult,"Start",0,REG_DWORD,(BYTE*)&buf,sizeof(buf));  
  }  
  RegCloseKey(hkResult);  
//////////////////////////////////////////////////////////////////////////  
  lg = RegOpenKeyEx(HKEY_LOCAL_MACHINE,  
          "system\\CurrentControlSet\\Services\\ERSvc",  
          0,  
         KEY_ALL_ACCESS,  
         &hkResult);  
  if (lg != ERROR_SUCCESS)  
  {  



```
MessageBox("打开注册表项 ERSvc 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

//////////////////////////////////////////////////////////////////////////  

lg = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,  

"system\\\\CurrentControlSet\\\\Services\\\\wscsvc",  

0,  

KEY\_ALL\_ACCESS,  

&hkResult);  

if (lg != ERROR\_SUCCESS)  

{  

MessageBox("打开注册表项 wscsvc 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

//////////////////////////////////////////////////////////////////////////  

lg = RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,  

"system\\\\CurrentControlSet\\\\Services\\\\wuauserv",  

0,  

KEY\_ALL\_ACCESS,  

&hkResult);  

if (lg != ERROR\_SUCCESS)  

{  

MessageBox("打开注册表项 wuauserv 出错!");  

return FALSE;  

}  

buf = 0x00000000;  

RegQueryValueEx(hkResult,"Start",0,0,(BYTE\*)&buf,&iResult);  

if (buf == 4)  

{  

buf = 0x00000002;  

RegSetValueEx(hkResult,"Start",0,REG\_DWORD,(BYTE\*)&buf,sizeof(buf));  

}  

RegCloseKey(hkResult);  

return TRUE;  

}  

/*
上面功能的实现。
*/  

void CRemoveDlg::OnBtnRemove()  

{  

// TODO: Add your control notification handler code here  

if(!this->PromoteLimit())  

{  

MessageBox("提权失败!");  

return;  

}  

if(!this->CloseProc())  

{  

MessageBox("结束进程失败!");  

return;  

}  

if(!this->RemoveFiles())//册除文件  

return;  

if(!this->RenewReg())  

return;  

if(MessageBox("已完成!\n 是否现在重启计算机?","Finished !",MB\_OKCANCEL) == IDOK)  

system("shutdown -r -f -t 0");  

}  
```
以上是我检测的病毒的分析及处理过程，和官方报出的病特征不太相同，估计是变种，但操作是类似的，建议操作完成后清理一下系统垃圾文件及冗余注册表。
