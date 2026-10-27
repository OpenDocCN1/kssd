
# [不敢声称原创，会被喷]A盾电脑防护0.43源代码发布~~

```
/*
**0efK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6f1q4F1N6r3W2s2j5h3#2W2f1s2u0G2N6r3g2U0N6q4)9J5k6h3y4G2L8b7`.`.**

写在前面的话：

最近YYY同学在发布某一个代码的时候，被XXX认为是在偷ZZZ的代码，然后XXX就很不爽的人肉了YYY同学的信息。这是事情的经过。

我想说的是，程序员那点事，谁写代码不是A来A去的。

XXX同学没A过吗？ZZZ同学就没有用过ctrl+c和ctrl+v吗？还发布人家的个人信息，这是侵犯隐私的行为，十几年的教育都还给老师了。

**看来国内的环境真不适合free、open、share。**

我很大方的承认，A盾的代码98%都是A的，现在代码放出来，赶紧来认领吧~~

PS：改版权的同学以后买套套都是穿孔的~

有技术问题发email：hack.x86.asm@gmail.com

or QQ:136618866
*/
```

## 编译环境

- 驱动：wdk7600 命令行下直接`build`
- ring3：vs2010

## File：

```
文件夹 PATH 列表
卷序列号码为 002E0034 CEF8:EB98
O:.
│  A-Protect.sln
│
├─A-Protect
│  │  A-Protect.aps
│  │  A-Protect.cpp
│  │  A-Protect.h
│  │  A-Protect.rc
│  │  A-Protect.vcxproj
│  │  A-Protect.vcxproj.filters
│  │  A-Protect.vcxproj.user
│  │  A-ProtectDoc.cpp
│  │  A-ProtectDoc.h
│  │  A-ProtectView.cpp
│  │  A-ProtectView.h
│  │  AboutDlg.cpp
│  │  AboutDlg.h
│  │  Atapi.cpp
│  │  Atapi.h
│  │  C3600Splash.cpp
│  │  C3600Splash.h
│  │  CProcessSearch.cpp
│  │  CProcessSearch.h
│  │  DisplayDecvice.cpp
│  │  DLLModule.cpp
│  │  DLLModule.h
│  │  DpcTimer.cpp
│  │  DpcTimer.h
│  │  EnumSymbols.cpp
│  │  EnumSymbols.h
│  │  FilterDriver.cpp
│  │  FilterDriver.h
│  │  FsdHook.cpp
│  │  FsdHook.h
│  │  HipsLog.cpp
│  │  HipsLog.h
│  │  Install.cpp
│  │  Install.h
│  │  IoTimer.cpp
│  │  IoTimer.h
│  │  Kbdclass.cpp
│  │  Kbdclass.h
│  │  KernelHook.cpp
│  │  KernelHook.h
│  │  KernelModule.cpp
│  │  KernelModule.h
│  │  KernelThread.cpp
│  │  KernelThread.h
│  │  LookUpKernelData.cpp
│  │  LookUpKernelData.h
│  │  MainFrm.cpp
│  │  MainFrm.h
│  │  Md5.cpp
│  │  Md5.h
│  │  MessageHook.cpp
│  │  MessageHook.h
│  │  Mouclass.cpp
│  │  Mouclass.h
│  │  ms.cpp
│  │  MyList.cpp
│  │  MyList.h
│  │  ndis5hlp.cpp
│  │  ndis5hlp.h
│  │  Nsiproxy.cpp
│  │  Nsiproxy.h
│  │  ntdll.lib
│  │  ObjectHook.cpp
│  │  ObjectHook.h
│  │  Process.cpp
│  │  Process.h
│  │  ProcessHandle.cpp
│  │  ProcessHandle.h
│  │  ProcessThread.cpp
│  │  ProcessThread.h
│  │  ProtectSetting.cpp
│  │  ProtectSetting.h
│  │  ReadMe.txt
│  │  ReportCtrl.cpp
│  │  ReportCtrl.h
│  │  resource.h
│  │  SelectAnyModule.cpp
│  │  SelectAnyModule.h
│  │  SelectKernelModuleHook.cpp
│  │  SelectKernelModuleHook.h
│  │  Services.cpp
│  │  Services.h
│  │  ShadowSSDT.cpp
│  │  ShadowSSDT.h
│  │  SnifferSetting.cpp
│  │  SnifferSetting.h
│  │  SSDT.cpp
│  │  SSDT.h
│  │  StackThread.cpp
│  │  StackThread.h
│  │  Startup.cpp
│  │  Startup.h
│  │  stdafx.cpp
│  │  stdafx.h
│  │  SubModule.cpp
│  │  SubModule.h
│  │  SystemNotify.cpp
│  │  SystemNotify.h
│  │  SystemThread.cpp
│  │  SystemThread.h
│  │  targetver.h
│  │  tcpdump.cpp
│  │  tcpdump.h
│  │  Tcpip.cpp
│  │  Tcpip.h
│  │  TcpView.cpp
│  │  TcpView.h
│  │  uninstall360.cpp
│  │  uninstall360.h
│  │  UnloadDllModule.h
│  │  UserImages.bmp
│  │  Windows2003SP1_CN.h
│  │  Windows2003SP2_CN.h
│  │  Windows7Home_CN.h
│  │  Windows7SP1_CN.h
│  │  WindowsXPSP2_CN.h
│  │  WindowsXPSP3_CN.h
│  │  WorkQueue.cpp
│  │  WorkQueue.h
│  │
│  ├─Release
│  └─res
│          A-Protect.bmp
│          A-Protect.ico
│          A-Protect1.ico
│          A-ProtectDoc.ico
│          AProteaaact.rc2
│          atapi.ico
│          Dispatch.ico
│          DPC定时器.ico
│          Eye.ico
│          GDriver.ico
│          Hips.ico
│          IO定时器.ico
│          KernelHook.ico
│          KernelModule.ico
│          KernelThread.ico
│          MyAProtect.rc2
│          new.ico
│          Nsiproxy.ico
│          ntfs-Fsd.ico
│          null.ico
│          ObjectHook.ico
│          Other.ico
│          Process.ico
│          ProtectSetting.ico
│          Services.ico
│          ShadowSSDT.ico
│          Ssdt.ico
│          tcpip.ico
│          TcpSniffer.ico
│          Tcpview.ico
│          Toolbar.bmp
│          Toolbar256.bmp
│          内核钩子.ico
│          启动项.ico
│          工作队列线程.ico
│          开启监控-刷新.ico
│          本机所有数据.ico
│          监控设置.ico
│          系统回调.ico
│          系统线程.ico
│          线程创建.ico
│          键盘.ico
│          鼠标.ico
│
├─Driver
│  │  AntiInlineHook.c
│  │  AntiInlineHook.h
│  │  Atapi.c
│  │  Atapi.h
│  │  buildchk_win7_x86.log
│  │  buildchk_win7_x86.wrn
│  │  Common.h
│  │  Control.c
│  │  Control.h
│  │  DeleteFile.c
│  │  DeleteFile.h
│  │  DpcTimer.c
│  │  DpcTimer.h
│  │  DriverHips.c
│  │  DriverHips.h
│  │  dump.c
│  │  dump.h
│  │  file.c
│  │  file.h
│  │  FileSystem.c
│  │  FileSystem.h
│  │  Fixrelocation.c
│  │  Fixrelocation.h
│  │  FuncAddrValid.c
│  │  FuncAddrValid.h
│  │  Function.c
│  │  Function.h
│  │  InitWindowsVersion.c
│  │  InitWindowsVersion.h
│  │  InlineHook.c
│  │  InlineHook.h
│  │  IoTimer.c
│  │  IoTimer.h
│  │  kbdclass.c
│  │  kbdclass.h
│  │  KernelFilterDriver.c
│  │  KernelFilterDriver.h
│  │  KernelHookCheck.c
│  │  KernelHookCheck.h
│  │  KernelReload.c
│  │  KernelReload.h
│  │  KernelThread.c
│  │  KernelThread.h
│  │  KillProcess.c
│  │  KillProcess.h
│  │  ldasm.c
│  │  ldasm.h
│  │  libdasm.c
│  │  libdasm.h
│  │  LookupKernelData.c
│  │  LookupKernelData.h
│  │  makefile
│  │  Mouclass.c
│  │  Mouclass.h
│  │  msghook.c
│  │  msghook.h
│  │  NetworkDefense.c
│  │  NetworkDefense.h
│  │  nsiproxy.c
│  │  nsiproxy.h
│  │  Ntfs.c
│  │  Ntfs.h
│  │  ntifs.h
│  │  ntos.c
│  │  ntos.h
│  │  ObjectHookCheck.c
│  │  ObjectHookCheck.h
│  │  Port.c
│  │  Port.h
│  │  Process.c
│  │  Process.h
│  │  ProcessModule.c
│  │  ProcessModule.h
│  │  Protect.c
│  │  Protect.h
│  │  ReLoadShadowSSDTTableHook.c
│  │  ReLoadShadowSSDTTableHook.h
│  │  ReLoadSSDTTableHook.c
│  │  ReLoadSSDTTableHook.h
│  │  SafeSystem.c
│  │  SafeSystem.h
│  │  SafeSystem.sln
│  │  SafeSystem.suo
│  │  SafeSystem.vcxproj
│  │  SafeSystem.vcxproj.filters
│  │  SafeSystem.vcxproj.user
│  │  SDTShadowRestore.h
│  │  SelectModuleHook.c
│  │  Services.c
│  │  Services.h
│  │  ShadowSSDT.c
│  │  ShadowSSDT.h
│  │  sources
│  │  SSDT.c
│  │  SSDT.h
│  │  Startup.c
│  │  Startup.h
│  │  SysModule.c
│  │  SysModule.h
│  │  SystemNotify.c
│  │  SystemNotify.h
│  │  SystemThread.c
│  │  SystemThread.h
│  │  tables.h
│  │  Tcpip.c
│  │  Tcpip.h
│  │  win32k.c
│  │  win32k.h
│  │  WorkQueue.c
│  │  WorkQueue.h
│  │
│  ├─objchk_win7_x86
│  │  └─i386
│  │          a.bat
│  │          bin2c.exe
│  │
│  └─Release
│          cl.command.1.tlog
│          CL.read.1.tlog
│          CL.write.1.tlog
│          SafeSystem.Build.CppClean.log
│          SafeSystem.lastbuildstate
│          SafeSystem.log
│          SafeSystem.unsuccessfulbuild
│          vc100.pdb
│
├─Release
│      A-Protect.exe
│      A-Protect.txt
│      dbghelp.dll
│      symsrv.dll
│      symsrv.yes
│
├─share
│      adapter.h
│      assert.h
│      netdef.h
│      netstd.h
│
└─TcpSnifferDriver
    │  buildchk_win7_x86.log
    │  ndis5pkt.c
    │  ndis5pkt.h
    │  ndis5pkt.vcxproj
    │  ndis5pkt.vcxproj.filters
    │  ndis5pkt.vcxproj.user
    │  openclos.c
    │  packet.h
    │  read.c
    │  readfast.c
    │  readme.txt
    │  SOURCES
    │  write.c
    │
    └─objchk_win7_x86
        └─i386
                a.bat
                bin2c.exe
                ndis5pkt.obj
                ndis5pkt.obj.oacr.root.x86chk.pft.xml
                ndis5pkt.pdb
                ndis5pkt.sys
                ndis5pkt.vmp.sys
                openclos.obj
                read.obj
                read.obj.oacr.root.x86chk.pft.xml
                readfast.obj
                readfast.obj.oacr.root.x86chk.pft.xml
                tcpsniffer.h
                vc90.pdb
                write.obj
                write.obj.oacr.root.x86chk.pft.xml
                _objects.mac
```
