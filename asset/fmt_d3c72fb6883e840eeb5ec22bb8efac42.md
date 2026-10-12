
```
.Upack:0017901F             call    sub_177CAC      ; 命令行执行自删除 cmd  /c del "源程序路径"
```

11、创建含木马信息的配置文件到 `C:\Documents and Settings\All Users` 目录下，名称为“jjjydf16.ini”，内容如下：

```
[mydown]
old_exe=
old_dll32=
ver=090121
fnexe=C:\WINDOWS\system\jjxzwzjy090121.exe
reg_start=dlmcjjcdfc
fn_dll=C:\WINDOWS\system\jjxzajcj32dl.dll
[kill]
window=33383533353533313939323633343930232825
[run]
delay=120
pzjg=180
xxjg=10000
```

名称为“jjdf32.ini”，内容为：

```
[sys]
acitve_install=20090122
[md5_ver]
efe0aae928e90bef3a055b32637ea561=0
f0090c73647ed989a4202f6f2501ed59=90121
```

这里代码太长，就不贴了。

12、修改 `hosts` 文件

```
CODE:0040DFE2             mov     eax, [ebp+var_34]
CODE:0040DFE5             call    sub_40AEC0      ; 修改 hosts 文件，drivers\etc\hosts
CODE:0040DFE5                                             ; 127.0.0.1 img.tongji.cn.yahoo.com
CODE:0040DFE5                                             ; 127.0.0.1 js.tongji.cn.yahoo.com
CODE:0040DFE5                                             ; 127.0.0.1      js.tongji.cn.yahoo.com
CODE:0040DFE5                                             ; 127.0.0.1      img.tongji.cn.yahoo.com
CODE:0040DFEA
```

13、修改注册表

```
CODE:0040DFF3             call    sub_40D6A4      ; 修改注册表
```

14、劫持杀软

```
CODE:0040E040             call    sub_40D390      ; 劫持
```

```
SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\
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
KVMonXP_1.kxp
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
