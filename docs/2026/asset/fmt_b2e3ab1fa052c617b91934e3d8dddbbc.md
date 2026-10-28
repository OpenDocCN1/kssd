
```
typedef struct \_IMAGE\_DOS\_HEADER {
+00 h WORD e\_magic // DOS可执行文件标记 MZ(4Dh 5Ah)
+02 h WORD e\_cblp ; Bytes on last page of file
+04 h WORD e\_cp ; Pages in file
+06 h WORD e\_crlc ; Relocations
+08 h WORD e\_cparhdr ; Size of header in paragraphs
+0A h WORD e\_minalloc ; Minimun extra paragraphs needs
+0C h WORD e\_maxalloc ; Maximun extra paragraphs needs
+0E h WORD e\_ss // DOS代码的初始化堆栈SS
+10 h WORD e\_sp // DOS代码的初始化堆栈指针SP
+12 h WORD e\_csum ; Checksum
+14 h WORD e\_ip // DOS代码的初始化指令入口[指针IP]
+16 h WORD e\_cs // DOS代码的初始堆栈入口
+18 h WORD e\_lfarlc ; File Address of relocation table
+1A h WORD e\_ovno ; Overlay number
+1C h WORD e\_res[4] ; Reserved words
+24 h WORD e\_oemid ; OEM identifier(for e\_oeminfo)
+26 h WORD e\_oeminfo ; OEM information;e\_oemid specific
+29 h WORD e\_res2[10] ; Reserved words
+3C h LONG e\_lfanew // 指向PE文件头
} IMAGE\_DOS\_HEADER ENDS
```

`mov edx, [eax+ebx+78h]`相当于：`772C0000+E8+78h`，通过结构体我们可以看到，该地址指向的是`DataDirectory`，也就是数据结构表,将获取到的偏移地址保存在了`edx`中。

```
typedef struct \_IMAGE\_OPTIONAL\_HEADER
{
+18 h WORD Magic; // 标志字, ROM 映像（0107h）,普通可执行文件（010Bh）
+1A h BYTE MajorLinkerVersion; // 链接程序的主版本号
+1B h BYTE MinorLinkerVersion; // 链接程序的次版本号
+1C h DWORD SizeOfCode; // 所有含代码的节的总大小
+20 h DWORD SizeOfInitializedData; // 所有含已初始化数据的节的总大小
+24 h DWORD SizeOfUninitializedData; // 所有含未初始化数据的节的大小
+28 h DWORD AddressOfEntryPoint; // 程序执行入口RVA
+2C h DWORD BaseOfCode; // 代码的区块的起始RVA
+30 h DWORD BaseOfData; // 数据的区块的起始RVA
+34 h DWORD ImageBase; // 程序的首选装载地址
+38 h DWORD SectionAlignment; // 内存中的区块的对齐大小
+3C h DWORD FileAlignment; // 文件中的区块的对齐大小
+40 h WORD MajorOperatingSystemVersion; // 要求操作系统最低版本号的主版本号
+42 h WORD MinorOperatingSystemVersion; // 要求操作系统最低版本号的副版本号
+44 h WORD MajorImageVersion; // 可运行于操作系统的主版本号
+46 h WORD MinorImageVersion; // 可运行于操作系统的次版本号
+48 h WORD MajorSubsystemVersion; // 要求最低子系统版本的主版本号
+4A h WORD MinorSubsystemVersion; // 要求最低子系统版本的次版本号
+4C h DWORD Win32VersionValue; // 莫须有字段，不被病毒利用的话一般为0
+50 h DWORD SizeOfImage; // 映像装入内存后的总尺寸
+54 h DWORD SizeOfHeaders; // 所有头 + 区块表的尺寸大小
+58 h DWORD CheckSum; // 映像的校检和
+5C h WORD Subsystem; // 可执行文件期望的子系统
+5E h WORD DllCharacteristics; // DllMain()函数何时被调用，默认为 0
+60 h DWORD SizeOfStackReserve; // 初始化时的栈大小
+64 h DWORD SizeOfStackCommit; // 初始化时实际提交的栈大小
+68 h DWORD SizeOfHeapReserve; // 初始化时保留的堆大小
+6C h DWORD SizeOfHeapCommit; // 初始化时实际提交的堆大小
+70 h DWORD LoaderFlags; // 与调试有关，默认为 0
+74 h DWORD NumberOfRvaAndSizes; // 下边数据目录的项数，Windows NT 发布是16
+78 h IMAGE\_DATA\_DIRECTORY DataDirectory[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES]; // 数据目录表} IMAGE\_OPTIONAL\_HEADER32, \*PIMAGE\_OPTIONAL\_HEADER32;
```

紧接着执行`mov eax, [eax+ebx+7Ch]`，当前`eax`保存的依然是PE头部偏移`E8`，`ebx= 772C0000`，那么相当于是`772C0000+E8+7C`，通过查看数据结构表我们可以得知，主要是获取第一块数据的大小。

![图片描述](upload/attach/202109/600583\_TWUHX3HV658S886.png)

`lea eax, [edx+ebx]`获取第一块数据块的地址。

![图片描述](upload/attach/202109/600583\_TJB32XYCXKAUDPK.png)

将`eax+0x20`中的地址给到了`edi`，在取`edi`的值赋值给到了`eax`，该地址指向了对应`Kernel32`的导出表函数名称。

![图片描述](upload/attach/202109/600583\_C54CJ7Q3GJVP4EA.png)

通过硬编码的方式匹配到所需要函数的序号`id`，并通过序号`id`寻找该函数的地址。

![图片描述](upload/attach/202109/600583\_B4WEGJDVQ746AFN.png)

函数做找到的地址赋值给到了`0x4F079C`

![图片描述](upload/attach/202109/600583\_478Y3J9REESAT8C.png)

![图片描述](upload/attach/202109/600583\_RXP2UH5NZ8MWQ8J.png)

对字符串进行不同方式进行解密，并且进行判断`LoadLibrary`函数是否获取成功，如果之前操作成功获取`LoadLibrary`函数则使用`LoadLibrary`函数进行地址获取，如果不成功，则继续利用PEB寻址方式进行API函数地址获取。

![图片描述](upload/attach/202109/600583\_BZ6PVDX3HSEXHYQ.png)

对所有需要用到的`dll`进行地址获取，并获取所需要用到的API函数地址。

![图片描述](upload/attach/202109/600583\_B7UJ3XG94EC6QGC.png)

终于结束了这该死的函数获取·····

## 初始化：

### 确认受害者国家：

将`kernel32`的内存加载地址传入`ebx`，并判断是否有获取成功，如果没有则利用PEB寻找到`dll`地址，如果找到则跳转。

![图片描述](upload/attach/202109/600583\_KSKB7FM96CCZ3PT.png)

利用硬编码进行比对，找到所需要执行的函数。

![图片描述](upload/attach/202109/600583\_SVMEERT7MXU8P5U.png)

通过序号`id`找到该函数的地址。

![图片描述](upload/attach/202109/600583\_T5DNJJAYTUH6DVF.png)

匹配是否命中已过滤的目标，如果命中则退出不进行勒索。

![图片描述](upload/attach/202109/600583\_FFYRZXE5VTP53H4.png)

### 绕过系统保护：

利用`Teb.UniqueProcess`的方法绕过系统文件保护。

![图片描述](upload/attach/202109/600583\_72GHMQG9KUTGNUV.png)

![图片描述](upload/attach/202109/600583\_WPCEHNNKV45NVNW.png)

### 检查访问控制：

修改ACL权限。

参考：<mark class="encrypted">a30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1k6i4N6Q4x3X3g2I4M7g2)9J5k6h3y4G2L8g2)9J5c8Y4u0S2K9h3&6Q4x3V1k6S2i4K6u0r3x3U0l9J5x3o6l9K6x3K6m8m8x3p5#2v1d9K6R3H3x3l9`.`.</mark>

![图片描述](upload/attach/202109/600583\_VRK28N67Y2JQG4P.png)

### 数据解密：

对所需要的信息以及图片进行解密。

![图片描述](upload/attach/202109/600583\_XSSF93HNA247TUW.png)

解密算法：

![图片描述](upload/attach/202109/600583\_FAC3XCSXB8GUCQR.png)

解密出信息和分辨率相关。

![图片描述](upload/attach/202109/600583\_8B6HA53D9ZU27RF.png)

解密出信息感觉可能是参数信息。

![图片描述](upload/attach/202109/600583\_HRXHVM94RT7R88B.png)

解密出图片相关信息。

![图片描述](upload/attach/202109/600583\_V2BMJB8MDNHNQKH.png)

![图片描述](upload/attach/202109/600583\_TA7HXB42FVFKFYX.png)

解密另外一张图，这应该是一个小图。

![图片描述](upload/attach/202109/600583\_3H8G47BGHRBS5BA.png)

打开之后是这样的一个像嘴唇一样的勒索图标。

![图片描述](upload/attach/202109/600583\_U63TP2KNWKYFU8F.png)

解密出进程、服务列表：

![图片描述](upload/attach/202109/600583\_BS7S4X6J8BFXF4X.png)

通过列出的信息可以猜测出，主要是为了结束相关的进程，避免在加密过程中进程运行，导致某些重要数据无法加密。

```
wxServer,wxServerView,sqlmangr,RAgui,supervise,Culture,Defwatch,winword,QBW32,QBDBMgr,qbupdate,axlbridge,httpd,fdlauncher,MsDtSrvr,java,360se,360doctor,wdswfsafe,fdhost,GDscan,ZhuDongFangYu,QBDBMgrN,mysqld,AutodeskDesktopApp,acwebbrowser,Creative Cloud,Adobe Desktop Service,CoreSync,Adobe CEF,Helper,node,AdobeIPCBroker,sync-taskbar,sync-worker,InputPersonalization,AdobeCollabSync,BrCtrlCntr,BrCcUxSys,SimplyConnectionManager,Simply.SystemTrayIcon,fbguard,fbserver,ONENOTEM,wsa_service,koaly-exp-engine-service,TeamViewer_Service,TeamViewer,tv_w32,tv_x64,TitanV,Ssms,notepad,RdrCEF,sam,oracle,ocssd,dbsnmp,synctime,agntsvc,isqlplussvc,xfssvccon,mydesktopservice,ocautoupds,encsvc,tbirdconfig,mydesktopqos,ocomm,dbeng50,sqbcoreservice,excel,infopath,msaccess,mspub,onenote,outlook,powerpnt,steam,thebat,thunderbird,visio,wordpad,bedbh,vxmon,benetns,bengien,pvlsvr,beserver,raw_agent_svc,vsnapvss,CagService,DellSystemDetect,EnterpriseClient,ProcessHacker,Procexp64,Procexp,GlassWire,GWCtlSrv,WireShark,dumpcap,j0gnjko1,Autoruns,Autoruns64,Autoruns64a,Autorunsc,Autorunsc64,Autorunsc64a,Sysmon,Sysmon64,procexp64a,procmon,procmon64,procmon64a,ADExplorer,ADExplorer64,ADExplorer64a,tcpview,tcpview64,tcpview64a,avz,tdsskiller,RaccineElevatedCfg,RaccineSettings,Raccine_x86,Raccine,Sqlservr,RTVscan,sqlbrowser,tomcat6,QBIDPService,notepad++,SystemExplorer,SystemExplorerService,SystemExplorerService64,Totalcmd,Totalcmd64,VeeamDeploymentSvc
```

解密出需要检测的服务：

![图片描述](upload/attach/202109/600583\_3FEUU5NA7Q74QVZ.png)

```
wrapper,DefWatch,ccEvtMgr,ccSetMgr,SavRoam,Sqlservr,sqlagent,sqladhlp,Culserver,RTVscan,sqlbrowser,SQLADHLP,QBIDPService,Intuit.QuickBooks.FCS,QBCFMonitorService, msmdsrv,tomcat6,zhudongfangyu,vmware-usbarbitator64,vmware-converter,dbsrv12,dbeng8,MSSQL$MICROSOFT##WID,MSSQL$VEEAMSQL2012,SQLAgent$VEEAMSQL2012,SQLBrowser,SQLWriter,FishbowlMySQL,MSSQL$MICROSOFT##WID,MySQL57,MSSQL$KAV_CS_ADMIN_KIT,MSSQLServerADHelper100,SQLAgent$KAV_CS_ADMIN_KIT,msftesql-Exchange,MSSQL$MICROSOFT##SSEE,MSSQL$SBSMONITORING,MSSQL$SHAREPOINT,MSSQLFDLauncher$SBSMONITORING,MSSQLFDLauncher$SHAREPOINT,SQLAgent$SBSMONITORING,SQLAgent$SHAREPOINT,QBFCService,QBVSS,YooBackup,YooIT,vss,sql,svc$,MSSQL,MSSQL$,memtas,mepocs,sophos,veeam,backup,bedbg,PDVFSService,BackupExecVSSProvider,BackupExecAgentAccelerator,BackupExecAgentBrowser,BackupExecDiveciMediaService,BackupExecJobEngine,BackupExecManagementService,BackupExecRPCService,MVArmor,MVarmor64,stc_raw_agent,VSNAPVSS,VeeamTransportSvc,VeeamDeploymentService,VeeamNFSSvc,AcronisAgent,ARSM,AcrSch2Svc,CASAD2DWebSvc,CAARCUpdateSvc,WSBExchange,MSExchange,MSExchange$
```

并将解密出来的信息进行`copy`，最后`free`掉当前解密的堆空间。

![图片描述](upload/attach/202109/600583\_N29F9D8NHS2Z9ZH.png)

将进程名称后增加`.exe`后缀，并将保存增加后的字符串分别保存在内存中，并将地址保存在`00C90000`中。

![图片描述](upload/attach/202109/600583\_P43ZYKXKH4ZZ3QV.png)

![图片描述](upload/attach/202109/600583\_KRM6ST5AA24PCS9.png)
