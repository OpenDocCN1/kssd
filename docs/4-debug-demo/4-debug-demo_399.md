

# Kryptel 3.74

> 原文：[`www.pediy.com/kssd/pediy02/forum237.htm`](https://www.pediy.com/kssd/pediy02/forum237.htm)

 详见：http://www.newhua.com.cn/Kryptel.htm   
  
软件简介：   
　集成在 Win9x 中的加密工具，您只要在 Windows 资源管理器中点击右键即可轻松加密单个或多个文件夹和文件。   
  
本地下载：   
http://www.newhua.com.cn/down/kryptel.exe   
http://newhua.21f.net/down/kryptel.exe   
国外下载：   
http://inv.co.nz/download/kryptel.exe   
好心的高手帮帮我，我急用！  

  

 开始用 S-ICE 追,发现了下面的程序:   
\* Reference To: MSVCRT.\_strlwr, Ord:01BEh   
                                  |   
:00401401 FF157CB14100            Call dword ptr \[0041B17C\]   
:00401407 59                      pop ecx   
:00401408 6818B84100              push 0041B818   
:0040140D FF3510B84100            push dword ptr \[0041B810\]   
:00401413 6890B74100              push 0041B790   
:00401418 FF358CB74100            push dword ptr \[0041B78C\]   
:0040141E 6878B74100              push 0041B778   
:00401423 FF3570B74100            push dword ptr \[0041B770\]   
:00401429 68F0B64100              push 0041B6F0   
:0040142E FF35E8B64100            push dword ptr \[0041B6E8\]   
:00401434 E8A4080100              call 00411CDD   
:00401439 83C420                  add esp, 00000020   
:0040143C FF7580                  push \[ebp\-80\]   
:0040143F FF7584                  push \[ebp\-7C\]   
:00401442 8D851CFFFFFF            lea eax, dword ptr \[ebp+FFFFFF1C\]   
:00401448 50                      push eax   
:00401449 E8D4080100              call 00411D22   
:0040144E 83C40C                  add esp, 0000000C   
:00401451 84C0                    test al, al   
:00401453 740C                    je 00401461   
:00401455 C7058C12420002000000    mov dword ptr \[0042128C\], 00000002<----unlimited 版本   
:0040145F EB5B                    jmp 004014BC   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00401453(C)   
|   
:00401461 6868B64100              push 0041B668   
:00401466 FF3560B64100            push dword ptr \[0041B660\]   
:0040146C 68E0B54100              push 0041B5E0   
:00401471 FF35DCB54100            push dword ptr \[0041B5DC\]   
:00401477 68C8B54100              push 0041B5C8   
:0040147C FF35C0B54100            push dword ptr \[0041B5C0\]   
:00401482 6840B54100              push 0041B540   
:00401487 FF3538B54100            push dword ptr \[0041B538\]   
:0040148D E84B080100              call 00411CDD   
:00401492 83C420                  add esp, 00000020   
:00401495 FF7580                  push \[ebp\-80\]   
:00401498 FF7584                  push \[ebp\-7C\]   
:0040149B 8D851CFFFFFF            lea eax, dword ptr \[ebp+FFFFFF1C\]   
:004014A1 50                      push eax   
:004014A2 E87B080100              call 00411D22   
:004014A7 83C40C                  add esp, 0000000C   
:004014AA 84C0                    test al, al   
:004014AC 0F84D3010000            je 00401685   
:004014B2 C7058C12420001000000    mov dword ptr \[0042128C\], 00000001<---personal 版本   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040145F(U)   
|   
:004014BC 8365FC00                and dword ptr \[ebp\-04\], 00000000   
:004014C0 C78550FDFFFF02000080    mov dword ptr \[ebp+FFFFFD50\], 80000002   
:004014CA 83A554FDFFFF00          and dword ptr \[ebp+FFFFFD54\], 00000000   
:004014D1 C645FC01                mov \[ebp\-04\], 01   
:004014D5 6A02                    push 00000002   
:004014D7 6818BF4100              push 0041BF18   
:004014DC 8D8D50FDFFFF            lea ecx, dword ptr \[ebp+FFFFFD50\]   
:004014E2 E8B4DE0000              call 0040F39B   
:004014E7 8BC6                    mov eax, esi   
:004014E9 85C0                    test eax, eax   
:004014EB 741E                    je 0040150B   
:004014ED 56                      push esi   
:004014EE E894E90000              call 0040FE87   
:004014F3 59                      pop ecx   
:004014F4 83C003                  add eax, 00000003   
:004014F7 24FC                    and al, FC   
:004014F9 E862440100              call 00415960   
:004014FE 8BC4                    mov eax, esp   
:00401500 50                      push eax   
:00401501 56                      push esi   
:00401502 E895E90000              call 0040FE9C   
:00401507 59                      pop ecx   
:00401508 59                      pop ecx   
:00401509 EB02                    jmp 0040150D   
但由于计算复杂,只好用 W32Dasm 了   
  
  
上面版本用 0042128C 地址做标志,因此搜索 0042128C,发现   
\=================================================================   
\* Referenced by a CALL at Address:   
|:00402525    
|   
:00401148 A18C124200              mov eax, dword ptr \[0042128C\]   
:0040114D 83F801                  cmp eax, 00000001   
:00401150 7405                    je 00401157   
:00401152 83F802                  cmp eax, 00000002   
:00401155 7520                    jne 00401177   
\=================================================================   
上面的这段决定软件启动时是否出注册提示画面   
\=================================================================   
\* Referenced by a CALL at Addresses:   
|:00401637  , :00403CB4  , :004051BA  , :00405461    
|   
:004017C4 55                      push ebp   
:004017C5 8BEC                    mov ebp, esp   
:004017C7 83EC50                  sub esp, 00000050   
:004017CA A18C124200              mov eax, dword ptr \[0042128C\]   
:004017CF 56                      push esi   
:004017D0 83F801                  cmp eax, 00000001   
:004017D3 7434                    je 00401809   
:004017D5 83F802                  cmp eax, 00000002   
:004017D8 742F                    je 00401809   
:004017DA 8D45B0                  lea eax, dword ptr \[ebp\-50\]   
:004017DD 6A50                    push 00000050   
:004017DF 50                      push eax   
\=================================================================   
上面的这段决定软件的版本(在软件窗口边框上显示)   
好吧,我输入的注册码计算后,0042128C 的标志肯定是"0"(否则我可以去买体育彩票了)   
那麽该知道怎麽改了吧   
  
把 cmp eax, 00000001 和 cmp eax, 00000002 中的 01(或 02)改为"0",一切 OK  

 抱歉,刚才的破解有一点遗漏(刚才想试试软件功能,"New"时出错了)   
赶快又用 W32Dasm 打开 brw.exe   
依然搜索"0042128C",找到了   
:004043CA E881150100              Call 00415950   
:004043CF 51                      push ecx   
:004043D0 81EC80000000            sub esp, 00000080   
:004043D6 53                      push ebx   
:004043D7 56                      push esi   
:004043D8 57                      push edi   
:004043D9 8965F0                  mov dword ptr \[ebp\-10\], esp   
:004043DC 33F6                    xor esi, esi   
:004043DE 8975D4                  mov dword ptr \[ebp\-2C\], esi   
:004043E1 8975D8                  mov dword ptr \[ebp\-28\], esi   
:004043E4 39358C124200            cmp dword ptr \[0042128C\], esi<---与 0 比较   
:004043EA 752B                    jne 00404417   
:004043EC 803D9404420000          cmp byte ptr \[00420494\], 00   
:004043F3 7422                    je 00404417   
:004043F5 FF3558124200            push dword ptr \[00421258\]   
:004043FB FF35FC0D4200            push dword ptr \[00420DFC\]   
  
改吧,将 jne 00404417 改为 jmp 00404417  (机器码 EB2B)   
  
:00403F50 E8FB190100              Call 00415950   
:00403F55 51                      push ecx   
:00403F56 81ECA0000000            sub esp, 000000A0   
:00403F5C 53                      push ebx   
:00403F5D 56                      push esi   
:00403F5E 57                      push edi   
:00403F5F 8965F0                  mov dword ptr \[ebp\-10\], esp   
:00403F62 C745C820000000          mov \[ebp\-38\], 00000020   
:00403F69 C745CC05000000          mov \[ebp\-34\], 00000005   
:00403F70 C745D001000000          mov \[ebp\-30\], 00000001   
:00403F77 33F6                    xor esi, esi   
:00403F79 39358C124200            cmp dword ptr \[0042128C\], esi   
:00403F7F 752B                    jne 00403FAC   
:00403F81 803D9404420000          cmp byte ptr \[00420494\], 00   
:00403F88 7422                    je 00403FAC   
:00403F8A FF3558124200            push dword ptr \[00421258\]   
:00403F90 FF35FC0D4200            push dword ptr \[00420DFC\]   
  
将 jne 00403FAC 改为 jmp 00403FAC (机器码 EB2B)  

将时间调至 2001 年，用右键看看。   
右键操作在 Klx.dll 文件中，用拦时间函数的方法拦不到 getsystemtime 和 getlocaltime 甚至用 bpint 21 if ah\==2 都拦不到，真不知它取时间是用的什么方法。后来我拦 BPX  messageboxa 进行分析，找到了一条非常方便的方法，由于它显示出错信息的同时显示帮助文件，我下了 BPX  winhelpa   
非常成功的找到了关键点。以下以加密文件为例，其它几个功能找法相同   
bpx winhelpa   
右键执行加密   
拦到后按 12 运行完后返回 SOFTICE   
将光标移到调用的 CALL 处按 F9 设断   
重新运行加密，在 CALL 处按 F8 再按 F10 就会来到下面。   
  
:75FC3047 E853860000              call 75FCB69F   
:75FC304C A2E895FD75              mov byte ptr \[75FD95E8\], al   
:75FC3051 A15898FD75              mov eax, dword ptr \[75FD9858\]   
:75FC3056 59                      pop ecx   
:75FC3057 83F801                  cmp eax,00000001  //这里进行比较        
:75FC305A 59                      pop ecx   
:75FC305B 0F84A8000000            je 75FC3109  //改为 JMP  OK   
:75FC3061 83F802                  cmp eax, 00000002   
:75FC3064 0F849F000000            je 75FC3109   
:75FC306A 381D9490FD75            cmp byte ptr \[75FD9094\], bl   
:75FC3070 7451                    je 75FC30C3   
:75FC3072 BE909AFD75              mov esi, 75FD9A90   
:75FC3077 8BC6                    mov eax, esi   
:75FC3079 85C0                    test eax, eax   
:75FC307B 741E                    je 75FC309B   
:75FC307D 56                      push esi   
:75FC307E E80F5F0000              call 75FC8F92   
:75FC3083 83C003                  add eax, 00000003   
:75FC3086 59                      pop ecx   
:75FC3087 24FC                    and al, FC   
:75FC3089 E832CE0000              call 75FCFEC0   
:75FC308E 8BC4                    mov eax, esp   
:75FC3090 50                      push eax   
:75FC3091 56                      push esi   
:75FC3092 E8515F0000              call 75FC8FE8   
:75FC3097 59                      pop ecx   
:75FC3098 59                      pop ecx   
:75FC3099 EB02                    jmp 75FC309D   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:75FC307B(C)   
|   
:75FC309B 33C0                    xor eax, eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:75FC3099(U)   
|   
:75FC309D 68424A0000              push 00004A42   
:75FC30A2 6A01                    push 00000001   
:75FC30A4 50                      push eax   
:75FC30A5 53                      push ebx   
  
\* Reference To: USER32.WinHelpA, Ord:02A8h   
                                  |   
:75FC30A6 FF159051FD75            Call dword ptr \[75FD5190\]   
:75FC30AC 8D45EC                  lea eax, dword ptr \[ebp\-14\]   
:75FC30AF 687869FD75              push 75FD6978   
  
修改 Klx.dll 方法：   
83 f8 01 59 0f 84 a8 00 00 00 83 f8 02   
83 f8 01 e9 a9 00 00 -- --------------   
  
83 f8 01 74 53 83 f8 02 74 4e   
\-------- eb -----------------   
  
83 f8 01 0f 84 a8 00 00 00 83 f8 02   
\-------- e9 a9 00 -----------------   
  
3b c7 74 52 83 f8 02   
\----- eb ----------- 

其实它的主要功能还是在鼠标右键上来得方便。

* * *

 4dd466d0e1e0cc123afbe504da2462ca