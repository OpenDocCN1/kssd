

# Trojan.PSW.Win32.GameOL.sng 分析

> 原文：[`www.pediy.com/kssd/pediy10/88996.html`](https://www.pediy.com/kssd/pediy10/88996.html)

【文章标题】Trojan.PSW.Win32.GameOL.sng 分析  
【文章作者】MichaelHc  
【分析工具】SoftSnoop 红尘岁月版、PEiD、AoRE，所有工具都来自看雪，请自己搜索下载。  
【分析平台】VMware 下的 XP SP2  
【中文名称】线上游戏窃取者  
【病毒样本】见附件，解压密码 virus，样本来自反病毒爱好者论坛，该论坛上有一篇关于这个病毒的文章 http://avfbbs.80port.net/read-htm-tid-22244.html  
【病毒简介】Trojan.PSW.Win32.GameOL 的一个变种，Trojan.PSW.Win32.GameOL 是各大安全公司公认的 08 年病毒/木马之首。详情请 google“08 年 10 大病毒”。  
【作者声明】只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
【Thanks To】红尘岁月  
【详细过程】  
启动虚拟机，记着启动后先创建快照，并断开与主机的网络连接，以免造成损失，所有操作均在虚拟机下进行。PEiD 分析，发现是 UPX 壳，使用 AoRE 脱壳成功，脱壳后再用 PEiD 载入可以看到病毒是用 VC++6.0 写的（不脱壳也可以直接分析）。接下来使用 SoftSnoop 分析如下：  
  
**首先，病毒在 C:\\WINDOWS\\system32\\目录创建文件 kandoftt.dll。**  
  
返回地址: 00401689  函数名称: GetSystemDirectoryA(KERNEL32.dll)   
GetSystemDirectoryA: 取得 Windows 系统目录（System 目录）的完整路径名  
          lpBuffer=0x0101FEA8  
          nBufferSize=0x00000104  
          存入缓冲区中的数据: "C:\\WINDOWS\\system32"  
GetSystemDirectoryA 返回值: 0x00000013  
       
返回地址: 0040169D  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1="C:\\WINDOWS\\system32"  
          String2="\\"  
lstrcat 返回值: 0x0101FEA8 ("C:\\WINDOWS\\system32\\")  
       
返回地址: 004016AB  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1="C:\\WINDOWS\\system32\\"  
          String2="kandoftt.dll"  
lstrcat 返回值: 0x0101FEA8 ("C:\\WINDOWS\\system32\\kandoftt.dll")  
       
返回地址: 00401608  函数名称: CreateFileA(KERNEL32.dll)   
CreateFileA: 打开和创建文件、管道及控制台等  
          lpFileName="C:\\WINDOWS\\system32\\kandoftt.dll"  
          dwDesiredAccess=0x40000000  
          dwShareMode=0x00000000  
          lpSecurityAttributes=0x00000000  
          dwCreationDistribution=0x00000002 (CREATE\_ALWAYS)  
          dwFlagsAndAttributes=0x00000000  
          hTemplateFile=0x00000000  
CreateFileA 返回值: 0x00000050(文件句柄)  
       
返回地址: 0040161E  函数名称: WriteFile(KERNEL32.dll)   
WriteFile: 将数据写入一个文件  
          hFile=0x00000050  
          lpBuffer=0x00404060  
          nNumberOfBytesToWrite=0x00007000  
          lpNumberOfBytesWritten=0x0101FE84  
          lpOverlapped=0x00000000  
          存入缓冲区中的数据: "MZ?  
WriteFile 返回值: 0x00000001(BOOL 类型)  
       
返回地址: 00401625  函数名称: CloseHandle(KERNEL32.dll)   
CloseHandle: 关闭一个句柄(包括文件、文件映射、进程、线程、安全和同步对象等)  
          hObject=0x00000050  
CloseHandle 返回值: 0x00000001(NULL 表示失败)  
       
返回地址: 00401631  函数名称: FreeResource(KERNEL32.dll)   
FreeResource: 释放资源  
          HretResource=0x00404060  
FreeResource 返回值: 0x00000000(释放资源)  
  
**接着，病毒在 HKLM\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\AppInit\_DLLs，写入 kandoftt.dll。  
**  
返回地址: 0040145C  函数名称: RegOpenKeyExA(ADVAPI32.dll)   
RegOpenKeyExA: 打开注册表中一个现有的项  
          hKey=0x80000002 (HKEY\_LOCAL\_MACHINE)  
          lpSubKey="SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Windows"  
          ulOptions=0x00000000  
          samDesired=0x000F003F (KEY\_ALL\_ACCESS)  
          phkResult=0x0101FE98  
RegOpenKeyExA 返回值: 0x00000000(NULL 表示成功)  
       
返回地址: 00401292  函数名称: RegQueryValueExA(ADVAPI32.dll)   
RegQueryValueExA: 获取注册表一个项的设置值  
          hKey=0x00000050  
          lpValueName="AppInit\_DLLs"  
          lpReserved=NULL  
          lpType=0x0101FE90  
          lpBuffer=0x0101EE3C  
          nBuffer=0xFE94  
          存入缓冲区中的数据: ""  
RegQueryValueExA 返回值: 0x00000000(NULL 表示成功)  
       
返回地址: 004014DE  函数名称: lstrcpy(KERNEL32.dll)   
lstrcpy: 复制字符串,复制 lpString2 到 lpString1  
          lpString1=0x0101EE3C  
          lpString2="kandoftt.dll"  
lstrcpy 返回值: 0x0101EE3C ("kandoftt.dll")  
       
返回地址: 0040155F  函数名称: lstrlen(KERNEL32.dll)   
lstrlen: 获取字符串的长度  
          lpString="kandoftt.dll"  
lstrlen 返回值: 0x0000000C(字符串长度)  
       
返回地址: 00401577  函数名称: RegSetValueExA(ADVAPI32.dll)   
RegSetValueExA: 设置指定项的值  
          hKey=0x00000050  
          lpValueName=0x0101FE7C  
          Reserved=0x00000000  
          dwType=0x00000001  
          lpData=0x0101EE3C  
          cbData=0x0000000D  
RegSetValueExA 返回值: 0x00000000  
       
返回地址: 00401586  函数名称: RegCloseKey(ADVAPI32.dll)   
RegCloseKey: 关闭系统注册表中的一个项或键  
          hKey=0x00000050  
RegCloseKey 返回值: 0x00000000(NULL 表示成功)  
            
**再接着，把自身复制到 C:\\WINDOWS\\system32\\kandofttk.exe**  
     
返回地址: 00401B77  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1="C:\\WINDOWS\\system32\\kandoftt"  
          String2="k.exe"  
lstrcat 返回值: 0x0012FE10 ("C:\\WINDOWS\\system32\\kandofttk.exe")  
       
返回地址: 00401B8D  函数名称: CopyFileA(KERNEL32.dll)   
CopyFileA: 复制文件  
          lpExistingFileName="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
          lpNewFileName="C:\\WINDOWS\\system32\\kandofttk.exe"  
          bFailIfExists=0x00000000  
CopyFileA 返回值: 0x00000001(复制文件)  
   
**接下来执行病毒程序，“C:\\WINDOWS\\system32\\kandofttk.exe ” 中的乱码是病毒的参数？   **   
  
返回地址: 00401B9B  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1="C:\\WINDOWS\\system32\\kandofttk.exe"  
          String2=" "  
lstrcat 返回值: 0x0012FE10 ("C:\\WINDOWS\\system32\\kandofttk.exe ")  
       
返回地址: 00401BAA  函数名称: WinExec(KERNEL32.dll)   
WinExec: 运行指定的程序  
          lpCmdLine=0x0012FE10  
          uCmdShow=0x00000000  
TID: 00000738h:  
模块被导入...appHelp.dll  Base: 76D70000h  
WinExec 返回值: 0x00000021(UINT 类型)  
  
**最后，创建批处理文件 dumped.exe.bat 删除自身，因为这里我先脱壳了，默认脱壳后的文件名为 dumped.exe，如果是没脱壳的病毒的话会创建 kandofttk.exe.bat**    
返回地址: 0040172F  函数名称: GetModuleFileNameA(KERNEL32.dll)   
GetModuleFileNameA: 获取一个已装载模板的完整路径名称  
          hModule=0x00400000  
          lpBuffer=0x0012FB50  
          nBufferSize=0x00000104  
          存入缓冲区中的数据: "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
GetModuleFileNameA 返回值: 0x00000037(文本长度)  
       
返回地址: 00401745  函数名称: lstrcpy(KERNEL32.dll)   
lstrcpy: 复制字符串,复制 lpString2 到 lpString1  
          lpString1=0x0012FA4C  
          lpString2="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
lstrcpy 返回值: 0x0012FA4C ("C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe")  
       
返回地址: 0040176D  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
          String2=".bat"  
lstrcat 返回值: 0x0012FB50 ("C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe.bat")  
       
返回地址: 0040178A  函数名称: CreateFileA(KERNEL32.dll)   
CreateFileA: 打开和创建文件、管道及控制台等  
          lpFileName="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe.bat"  
          dwDesiredAccess=0x40000000  
          dwShareMode=0x00000001 (FILE\_SHARE\_READ)  
          lpSecurityAttributes=0x00000000  
          dwCreationDistribution=0x00000002 (CREATE\_ALWAYS)  
          dwFlagsAndAttributes=0x00000080 (FILE\_ATTRIBUTE\_NORMAL)  
          hTemplateFile=0x00000000  
CreateFileA 返回值: 0x00000064(文件句柄)  
       
返回地址: 00401853  函数名称: lstrcpy(KERNEL32.dll)   
lstrcpy: 复制字符串,复制 lpString2 到 lpString1  
          lpString1=0x0012F64C  
          lpString2=":try  
del ""  
lstrcpy 返回值: 0x0012F64C (":try  
del "")  
       
返回地址: 00401863  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1=":try  
del ""  
          String2="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
lstrcat 返回值: 0x0012F64C (":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe")  
       
返回地址: 00401870  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1=":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
          String2=""  
if exist ""  
lstrcat 返回值: 0x0012F64C (":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "")  
       
返回地址: 00401880  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1=":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist ""  
          String2="C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
lstrcat 返回值: 0x0012F64C (":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "C:\\Documents and Set")  
       
返回地址: 0040188D  函数名称: lstrcat(KERNEL32.dll)   
lstrcat: 把 string2 连接到 string1 后面  
          String1=":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "C:\\Documents and Set"  
          String2="" goto try  
del %0"  
lstrcat 返回值: 0x0012F64C (":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "C:\\Documents and Set")  
       
返回地址: 0040189F  函数名称: lstrlen(KERNEL32.dll)   
lstrlen: 获取字符串的长度  
          lpString=":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "C:\\Documents and Set"  
lstrlen 返回值: 0x00000098(字符串长度)  
       
返回地址: 004018B0  函数名称: WriteFile(KERNEL32.dll)   
WriteFile: 将数据写入一个文件  
          hFile=0x00000064  
          lpBuffer=0x0012F64C  
          nNumberOfBytesToWrite=0x00000098  
          lpNumberOfBytesWritten=0x0012FCF0  
          lpOverlapped=0x00000000  
          存入缓冲区中的数据: ":try  
del "C:\\Documents and Settings\\Administrator\\桌面\\dumped.exe"  
if exist "C:\\Documents and Set"  
异常: 非法访问地址 100B5469h  
进程已被 SoftSnoop 终止   
  
**到这里出现了异常，没法调试下去了，打开任务管理器，可以看到进程 kandofttk.exe，另外，桌面上有两个文件 dumped.exe、dumped.exe.bat，其中 dumped.exe.bat 的内容为**：  
:try  
del "C:\\Documents and Settings\\Administrator\\桌面\\kandofttk.exe"  
if exist "C:\\Documents and Settings\\Administrator\\桌面\\kandofttk.exe" goto try  
del %0  
原来创建批处理文件之后还没来得及执行就出现了异常，这个批处理文件显然是用来删除 dumped.exe、dumped.exe.bat 这两个文件的。  
  
有了以上的信息，清除这个病毒就不是难事了：  
  
结束进程:   
kandofttk.exe   
   
删除文件:    
C:\\WINDOWS\\system32\\kandoftt.dll  
C:\\WINDOWS\\system32\\kandofttk.exe  
  
清理下边这个注册表值为空:  
HKEY\_LOCAL\_MACHINE\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\AppInit\_DLLs;  
  
为了验证清除方法，我把虚拟机恢复成之前创建的快照，这次直接运行样本 kandofttk.exe（桌面上没有留下两个文件了，真正的中毒了），然后按照刚才的步骤手工清除病毒，居然很顺利的搞定了。不免有点怀疑 08 年十大病毒榜首的 Trojan.PSW.Win32.GameOL 就这么容易杀么？另外有关病毒的功能模块（写这个病毒的人到底想搞什么破坏呢？）也还没有分析，有空再研究研究。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || kandofttk.rar  
 |