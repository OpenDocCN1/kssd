

# 去除任务栏图标

> 原文：[`www.pediy.com/kssd/pediy05/pediy50517.htm`](https://www.pediy.com/kssd/pediy05/pediy50517.htm)

这次我要对付的软件是一个比较好用的代理软件(我不知道这样文章要不要隐藏文件名称),算法大家已经分析很多次了,我就不再重复了.   
我这里来讨论一个实际应用的问题:如果你想用别人的机器上网而又不想让他轻易的发现(好像这样的想法大家都有吧?).而这个软件在启动之后   
  
会在任务栏加载一个图标,这样就比较矛盾了啊!所以我才要解决他啊!   
  
首先我们先来点思路:   
要向系统托盘中加入和删除图标必须要用到 SHELL32.Shell\_NotifyIconA 这个函数,也就是说当软件启动的时候一定是调用这个函数将自己的图   
  
标加入到系统托盘中;在关闭软件的时候他也是调用这个函数将图标删除的.   
  
我们来查一下 WINAPI 函数表来确定一下这个函数的使用方法和格式.   
Const NIM\_ADD = &H0//这个就是向托盘加入图标的参数   
Const NIM\_MODIFY = &H1   
Const NIM\_DELETE = &H2//这个就是从托盘中删除图标的参数   
  
Const NIF\_MESSAGE = &H1   
Const NIF\_ICON = &H2   
Const NIF\_TIP = &H4   
  
Declare Function Shell\_NotifyIcon Lib "shell32.dll" Alias " Shell\_NotifyIconA" (ByVal dwMessage As Long, lpData As   
  
NOTIFYICONDATA) As Long   
上面 2 个函数的参数一定要记下来下面有用啊!   
  
我们用 WINASM32 来反汇编软件,在函数中查找 SHELL32.Shell\_NotifyIconA 这个函数看看系统中是怎样调用他的,双击几次我们发现软件一共有 3   
  
处调用这个函数.   
我们先来分析一下这个软件调用的这 3 处的作用:首先,软件启动会向系统的任务栏中添加一个图标;然后,要完成图标的动画作用(写过程序的人   
  
应该知道,做动画比较简单的方法就是来回切换图标就可以完成了啊);最后,在退出软件的时候要删除任务栏的图标.大概就是这 3 处.   
这只是初步分析了下面我们进行动态跟踪:   
为了分析清楚软件是怎样调用的就必须使用 TRW2000 或 SICE 动态跟踪了,我们下 BPX Shell\_NotifyIconA 断点,拦断后用 F12 跳出看看软件是如果   
  
调用,和使用参数的!   
  
首先来到下面:   
这里是软件启动时调用的地方   
\* Possible Reference to String Resource ID=00114: "CCProxy"   
                                  |   
:00408770 6A72                    push 00000072   
:00408772 51                      push ecx   
:00408773 C68424F424000005        mov byte ptr \[esp+000024F4\], 05   
:0040877B E8C0890100              call 00421140   
:00408780 83C408                  add esp, 00000008   
:00408783 50                      push eax   
:00408784 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00408788 C68424F024000006        mov byte ptr \[esp+000024F0\], 06   
:00408790 E818910300              call 004418AD   
:00408795 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00408799 C68424EC24000005        mov byte ptr \[esp+000024EC\], 05   
:004087A1 E8CE8F0300              call 00441774   
:004087A6 8B7C2410                mov edi, dword ptr \[esp+10\]   
:004087AA 83C9FF                  or ecx, FFFFFFFF   
:004087AD 33C0                    xor eax, eax   
:004087AF 8D95F8090000            lea edx, dword ptr \[ebp+000009F8\]   
:004087B5 F2                      repnz   
:004087B6 AE                      scasb   
:004087B7 F7D1                    not ecx   
:004087B9 2BF9                    sub edi, ecx   
:004087BB 53                      push ebx   
:004087BC 8BC1                    mov eax, ecx   
:004087BE 8BF7                    mov esi, edi   
:004087C0 8BFA                    mov edi, edx   
:004087C2 6A00                    push 00000000//看到了吗这里是 0   
:004087C4 C1E902                  shr ecx, 02  //这里改没有作用因为即使改了在下面    
:004087C7 F3                      repz        //系统切换图标时又会产生所以要改下面的   
:004087C8 A5                      movsd   
:004087C9 8BC8                    mov ecx, eax   
:004087CB 83E103                  and ecx, 00000003   
:004087CE F3                      repz   
:004087CF A4                      movsb   
  
\* Reference To: SHELL32.Shell\_NotifyIconA, Ord:0079h//调用函数   
                                  |   
:004087D0 FF1570E34400            Call dword ptr \[0044E370\]//调用过程成功返回 1,失败返回 0   
:004087D6 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004087DA C68424EC24000002        mov byte ptr \[esp+000024EC\], 02   
:004087E2 E88D8F0300              call 00441774   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00408717(C)   
|   
:004087E7 55                      push ebp   
:004087E8 E803920100              call 004219F0   
:004087ED 83C404                  add esp, 00000004   
:004087F0 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:004087F4 51                      push ecx   
:004087F5 E878390300              call 0043C172   
:004087FA 8B10                    mov edx, dword ptr \[eax\]   
:004087FC 8D44241C                lea eax, dword ptr \[esp+1C\]   
  
\* Possible StringData Ref from Data Obj ->"%H:%M:%S"   
                                  |   
:00408800 68BCD64500              push 0045D6BC   
:00408805 50                      push eax   
:00408806 8D4C242C                lea ecx, dword ptr \[esp+2C\]   
:0040880A 8954242C                mov dword ptr \[esp+2C\], edx   
:0040880E E872390300              call 0043C185   
:00408813 50                      push eax   
:00408814 8D4C2424                lea ecx, dword ptr \[esp+24\]   
  
\* Possible StringData Ref from Data Obj ->" "   
                                  |   
:00408818 6828D14500              push 0045D128   
:0040881D 51                      push ecx   
:0040881E C68424F824000007        mov byte ptr \[esp+000024F8\], 07   
:00408826 E8EC910300              call 00441A17   
  
\* Possible StringData Ref from Data Obj ->" "   
                                  |   
:0040882B 6828D14500              push 0045D128   
:00408830 8D542418                lea edx, dword ptr \[esp+18\]   
:00408834 50                      push eax   
:00408835 52                      push edx   
:00408836 C68424F824000008        mov byte ptr \[esp+000024F8\], 08   
:0040883E E860910300              call 004419A3   
:00408843 8B00                    mov eax, dword ptr \[eax\]   
:00408845 8BCD                    mov ecx, ebp   
:00408847 50                      push eax   
  
\--------------------------------------------------------------------------   
这里是形成动画时的调用:   
:00409669 8D542414                lea edx, dword ptr \[esp+14\]   
  
\* Possible Reference to String Resource ID=00114: "CCProxy"   
                                  |   
:0040966D 6A72                    push 00000072   
:0040966F 52                      push edx   
:00409670 C784243014000000000000  mov dword ptr \[esp+00001430\], 00000000   
:0040967B E8C07A0100              call 00421140   
:00409680 83C408                  add esp, 00000008   
:00409683 50                      push eax   
:00409684 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00409688 C684242C14000001        mov byte ptr \[esp+0000142C\], 01   
:00409690 E818820300              call 004418AD   
:00409695 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00409699 C684242814000000        mov byte ptr \[esp+00001428\], 00   
:004096A1 E8CE800300              call 00441774   
:004096A6 8B7C2410                mov edi, dword ptr \[esp+10\]   
:004096AA 83C9FF                  or ecx, FFFFFFFF   
:004096AD 33C0                    xor eax, eax   
:004096AF 8D95F8090000            lea edx, dword ptr \[ebp+000009F8\]   
:004096B5 F2                      repnz   
:004096B6 AE                      scasb   
:004096B7 F7D1                    not ecx   
:004096B9 2BF9                    sub edi, ecx   
:004096BB 53                      push ebx   
:004096BC 8BC1                    mov eax, ecx   
:004096BE 8BF7                    mov esi, edi   
:004096C0 8BFA                    mov edi, edx   
:004096C2 6A00                    push 00000000//这里虽然也是 0 但这里是为了形成动画的时候调用的   
:004096C4 C1E902                  shr ecx, 02  //如果不想让他显示就改变参数就行了 6A00->6A02    
:004096C7 F3                      repz   
:004096C8 A5                      movsd   
:004096C9 8BC8                    mov ecx, eax   
:004096CB 83E103                  and ecx, 00000003   
:004096CE F3                      repz   
:004096CF A4                      movsb   
  
\* Reference To: SHELL32.Shell\_NotifyIconA, Ord:0079h   
                                  |   
:004096D0 8B3570E34400            mov esi, dword ptr \[0044E370\]   
:004096D6 FFD6                    call esi   
:004096D8 8B4D1C                  mov ecx, dword ptr \[ebp+1C\]   
  
\* Possible StringData Ref from Data Obj ->"CCProxy System Tray Icon Message"   
                                  |   
:004096DB 68C8D64500              push 0045D6C8   
:004096E0 C70358000000            mov dword ptr \[ebx\], 00000058   
:004096E6 898DE4090000            mov dword ptr \[ebp+000009E4\], ecx   
:004096EC C785E80900000A000000    mov dword ptr \[ebp+000009E8\], 0000000A   
:004096F6 C785EC09000002000000    mov dword ptr \[ebp+000009EC\], 00000002   
  
\* Reference To: USER32.RegisterWindowMessageA, Ord:0200h   
                                  |   
:00409700 FF1578E54400            Call dword ptr \[0044E578\]   
:00409706 8985F0090000            mov dword ptr \[ebp+000009F0\], eax   
:0040970C 8B1564164600            mov edx, dword ptr \[00461664\]   
\---------------------------------------------------------------------------   
这里是软件退出时,用来删除图标的函数调用部分!   
:00409DAF 90                      nop   
:00409DB0 A1484B4600              mov eax, dword ptr \[00464B48\]   
:00409DB5 56                      push esi   
:00409DB6 85C0                    test eax, eax   
:00409DB8 8BF1                    mov esi, ecx   
:00409DBA 750F                    jne 00409DCB   
:00409DBC 8D86E0090000            lea eax, dword ptr \[esi+000009E0\]   
:00409DC2 50                      push eax   
:00409DC3 6A02                    push 00000002//看看这里和上面介绍的参数值进行一下比较   
  
\* Reference To: SHELL32.Shell\_NotifyIconA, Ord:0079h//调用函数   
                                  |   
:00409DC5 FF1570E34400            Call dword ptr \[0044E370\]//调用如果成功则返回 1   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00409DBA(C)   
|   
:00409DCB 8B0DA8164600            mov ecx, dword ptr \[004616A8\]   
:00409DD1 E85A210000              call 0040BF30   
:00409DD6 8B0DA8164600            mov ecx, dword ptr \[004616A8\]   
:00409DDC C7410400000000          mov \[ecx+04\], 00000000   
:00409DE3 8BCE                    mov ecx, esi   
:00409DE5 C7059CD5450000000000    mov dword ptr \[0045D59C\], 00000000   
:00409DEF E853410300              call 0043DF47   
:00409DF4 5E                      pop esi   
:00409DF5 C3                      ret   
\-------------------------------------------------------------------------   
既然知道了这几处调用的作用我们就来做一下修改,软件在启动的时候会向任务栏加入一个图标,改这里行不行回答是不行.   
为什么?因为软件在加入图标之后会为了完成动画作用继续使用这个函数来切换图标,所以即使你修改了启动时的部分在切换   
图标的时候也还会产生,因此改一下生成动画部分函数参数就行了啊!   
  
查找字符串:8BC18BF78BFA6A00C1E902   
改为:      ........... 6A02......   
  
这样修改之后软件启动后就不会在任务栏产生图标了(遇到的问题解决了)   
小弟第一次写这样的文章不免有思路和过程上的漏洞,请大家多多指点和帮助.   
  
如果您有更好的方法请发邮件到 powerboy\_bcg@163.com   
小弟先谢了!  4dd466d0e1e0cc123afbe504da2462ca