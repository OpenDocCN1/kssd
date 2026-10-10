

# WorkgroupMail 6.2.0 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue025.htm`](https://www.pediy.com/kssd/pediy04/kanxue025.htm)

破解 WorkgroupMail 的 30 天的时间限制！   
  
破解者：moonlite   
目标：WorkgroupMail 6.2.0   
下载：http://newhua.ruyi.com/down/wm.exe （大小 2964Kb）   
工具：TRW 1.22, UltraEdit，W32dasm， Winhex   
保护：每次启动跳出 nag 窗＋30 天试用＋动态验证   
软件介绍：WorkgroupMail is a robust and highly scaleable POP3 server.   
It can be used to automatically send and receive mail on behalf of each person   
in an organization and then store received mail centrally ready for collection   
by each person's e-mail client.   
  
前言：这个软件安装完后，每次运行它都会跳出一个窗口提示“There are XX days remaining before your trial period expires. Please visit us at ...”。好！开始工作吧：   
  
1）去除 nag 窗口－>   
  
\>>首先，用 W32dasm 反汇编 WMSvc.exe，在 SDR 中可找到”Your licensed has expired! Please visit us at ...”：   
  
:0044019E E8FD5C0000              call 00445EA0——————>程序启动时，检查是否过期的 CALL   
:004401A3 83C404                  add esp, 00000004   
:004401A6 8B442408                mov eax, dword ptr \[esp+08\]   
:004401AA C744243000000000        mov \[esp+30\], 00000000   
:004401B2 85C0                    test eax, eax   
:004401B4 7F40                    jg 004401F6——————>在此一定要跳呵！/offset=401B4h   
:004401B6 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
  
\* Reference To: MFC42.Ordinal:021C, Ord:021Ch   
                                  |   
:004401BA E825EF0000              Call 0044F0E4   
:004401BF 8B442414                mov eax, dword ptr \[esp+14\]   
:004401C3 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
:004401C7 50                      push eax   
  
\* Possible Reference to String Resource ID=24591: "Your licensed has expired! Please visit us at %1 and purchas"   
                                  |   
:004401C8 680F600000              push 0000600F   
:004401CD 51                      push ecx   
:004401CE C644243C01              mov \[esp+3C\], 01   
  
\* Reference To: MFC42.Ordinal:047B, Ord:047Bh   
                                  |   
:004401D3 E858F10000              Call 0044F330   
  
用 UltraEdit，在 offset=401B4h 处，将 7F40  改为 EB40, 保存后退出。   
  
\>>运行 WMSvc.exe，nag 窗出现。启动 TRW，Ctrl＋D 激活，搜索字符串“There are XX days ...”，内存中找到一处地址 6E0FD4。退出 WMSvc.exe。bpx 6E0FD4 设断，并重新运行 WMSvc.exe，被 TRW 拦住如下：   
  
——————————MSVCRT!.memcpy—————————   
015F:78001641  JMP      NEAR \[ECX\*4+78001704\]   
015F:78001648  REP MOVSD ————————————————>光标在这！   
015F:7800164A  JMP      NEAR \[EDX\*4+78001728\]   
015F:78001651  MOV      EAX,EDI   
015F:78001653  MOV      EDX,03   
015F:78001658  SUB      ECX,BYTE +04   
015F:7800165B  JC      78001669   
015F:7800165D  AND      EAX,BYTE +03   
015F:78001660  ADD      ECX,EAX   
015F:78001662  JMP      NEAR \[EAX\*4+780016F4\]   
015F:78001669  JMP      NEAR \[ECX\*4+\`MSVCRT!memcmp\`\]   
015F:78001670  AND      EDX,ECX   
015F:78001672  MOV      AL,\[ESI\]   
015F:78001674  MOV      \[EDI\],AL   
015F:78001676  MOV      AL,\[ESI+01\]   
  
按三次 F5 后，程序会来到：   
  
——————————USER!.messageboxindirect——————   
1777:012C  CLD      
1777:012D  REPNE SCASB ————————————————>光标在这！   
1777:012F  NEG      CX   
1777:0131  DEC      CX   
1777:0132  CMP      CX,BX   
1777:0134  POP      CX   
1777:0135  POP      DI   
1777:0136  XCHG    AX,DX   
1777:0137  JA      0140   
1777:0139  RET      
  
我们知道这是在调用窗口函数，也就是说 nag 窗快登场了！(有朋友会说，拦窗口太费事了，bpx messageboxindirect 不就结了---我试过，这招不灵呵) 好！ 按两次 F12，nag 窗弹出。按下“确定”按钮，又返回 TRW 领空：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0043FF37(C)   
|   
:0043FF4E A1E83D4700              mov eax, dword ptr \[00473DE8\]   
:0043FF53 85C0                    test eax, eax——————>使 eax=0，可跳过提示注册的 nag 窗/offset=3FF53h   
:0043FF55 7415                    je 0043FF6C   
:0043FF57 8B4C240C                mov ecx, dword ptr \[esp+0C\]   
:0043FF5B 8B542408                mov edx, dword ptr \[esp+08\]   
:0043FF5F 8B442404                mov eax, dword ptr \[esp+04\]——————>eax 指向字符串“There are XX days remaining ...”   
:0043FF63 51                      push ecx   
:0043FF64 52                      push edx   
:0043FF65 50                      push eax   
  
\* Reference To: MFC42.Ordinal:04B0, Ord:04B0h   
                                  |   
:0043FF66 E8A3F10000              Call 0044F10E——————>nag 窗的 CALL   
:0043FF6B C3                      ret   
  
  
心里还想呢，这个 nag 窗太好去除了^\_^。   
  
打开 UltraEdit， Ctrl＋G，来到 3FF53h 处，将 8530 改为 31C0 （即 xor eax, eax）   
保存退出。运行 WMSvc.exe。那个 nag 窗口没了...。将时间调快两个月，再运行它，nag 窗口也没出现，可高兴了没有两秒，它在工具拦的图标竞消失了！一定还有暗桩，难道有 CRC 校验，不会呵，运行时可没报错呵～～   
  
难道其它地方还有时间的校验，该不是...   
再演示一遍。一定要抢在程序退出以前，用 TRW 中断它吆！   
在工具拦的图标出现后，迅速 Ctrl＋D，来到 TRW 的领空，下 bpx getsystemtime。开始按 F5 吧，会按几次的，别忘了做笔记。   
  
最后，会发现以下代码：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00402B4E(C)   
|   
:00402B57 E8B42E0400              call 00445A10——————>动态验证是否过期的 CALL/过期，则 eax=0   
:00402B5C 85C0                    test eax, eax——————>看上去很面熟呵～～   
:00402B5E 7507                    jne 00402B67——————>一定要跳呵！！/offset=2B5Eh   
:00402B60 8BCE                    mov ecx, esi   
:00402B62 E859050000              call 004030C0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00402B5E(C)   
|   
:00402B67 C705C444470000000000    mov dword ptr \[004744C4\], 00000000   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00402AEA(C)   
|   
:00402B71 5E                      pop esi   
:00402B72 C20400                  ret 0004   
  
\>>用 UltraEdit，将 offset=2B5Eh 处的 7507  改为 EB07。保存退出－>运行 WMSvc.exe——>搞定！！   
  
  
2）启动时时间的验证：   
  
有朋友会问，到底软件提示中用了多少天是怎么算出来的呵？和我一样，总是很好奇。。。好，我和你切磋一下！   
  
安装 WorkgroupMail 后，安装的时间记录会保存在同目录下的文件 WMData.dat 中。它是一个二进制数据文件，记录着你的配置信息和软件安装时间：   
用 UltraEdit 打开后发现：   
  
偏移地址   
00000000:1600000073716C3B0001000000010000   
  
注意 73716C3B 这四个字节，就是你安装软件的时间。让我们看以下代码：   
  
:00403D5E E8ED390400              call 00447750   
:00403D63 85C0                    test eax, eax   
:00403D65 7512                    jne 00403D79   
:00403D67 6AFF                    push FFFFFFFF   
:00403D69 53                      push ebx   
  
\* Possible Reference to String Resource ID=24638: "Unable to load settings file WMData.dat!"   
                                  |   
:00403D6A 683E600000              push 0000603E   
:00403D6F E80CC20300              call 0043FF80   
:00403D74 83C40C                  add esp, 0000000C   
:00403D77 EB10                    jmp 00403D89   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00403D65(C)   
|   
:00403D79 8BCE                    mov ecx, esi   
:00403D7B E820030000              call 004040A0   
:00403D80 E8CBC30300              call 00440150——————>启动时检查运行天数及是否过期的 CALL   
:00403D85 85C0                    test eax, eax   
:00403D87 7533                    jne 00403DBC   
                                
在 00403D79 处设断。并运行 WMSvc.exe，TRW 拦住后。suspend 并返回 windows，打开 Winhex 的 RAM 的 Editor，搜索 HEX 值“73716C3B”找到地址 7E2CB8 一处。bpm 7E2CB8，按 F5，中断在：   
  
\* Reference To: MFC42.Ordinal:0EE3, Ord:0EE3h   
                                  |   
:00440040 E87BF00000              Call 0044F0C0   
:00440045 8B08                    mov ecx, dword ptr \[eax\]   
:00440047 8B15FC484700            mov edx, dword ptr \[004748FC\]   
:0044004D 51                      push ecx   
:0044004E 894C2418                mov dword ptr \[esp+18\], ecx   
:00440052 8B4A08                  mov ecx, dword ptr \[edx+08\]——————>指向安装时间记录   
:00440055 8BC4                    mov eax, esp——————>光标在这！   
:00440057 8D542424                lea edx, dword ptr \[esp+24\]   
:0044005B 89642420                mov dword ptr \[esp+20\], esp   
:0044005F 8908                    mov dword ptr \[eax\], ecx——————>时间记录送\[eax\]保存   
:00440061 52                      push edx   
:00440062 8D4C241C                lea ecx, dword ptr \[esp+1C\]——————>指向的是系统时间   
:00440066 E885FBFCFF              call 0040FBF0\]——————>进入——>>   
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
\* Referenced by a CALL at Addresses:   
|:0040F898  , :00426D2C  , :004306ED  , :00432322  , :004329B0    
|:004357B9  , :00440066  , :0044390A  , :00443998  , :004457DA    
|   
:0040FBF0 8B09                    mov ecx, dword ptr \[ecx\]——————>指向的是系统时间   
:0040FBF2 8B442408                mov eax, dword ptr \[esp+08\]——————>指向安装时间记录   
:0040FBF6 2BC8                    sub ecx, eax——————>做减法   
:0040FBF8 8B442404                mov eax, dword ptr \[esp+04\]——————>指定差值的存放地址   
:0040FBFC 8908                    mov dword ptr \[eax\], ecx——————>差值保存在\[eax\]   
:0040FBFE C20800                  ret 0008   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
  
:0044006B 8B08                    mov ecx, dword ptr \[eax\]——————>差值送 ecx   
:0044006D B807452EC2              mov eax, C22E4507\]——————>换算参数送 ecx   
:00440072 F7E9                    imul ecx   
:00440074 03D1                    add edx, ecx   
:00440076 C1FA10                  sar edx, 10   
:00440079 8BC2                    mov eax, edx   
:0044007B C1E81F                  shr eax, 1F   
:0044007E 03D0                    add edx, eax   
:00440080 8BDA                    mov ebx, edx——————>ebx 中是你已试用的天数   
:00440082 7904                    jns 00440088   
:00440084 33C0                    xor eax, eax   
:00440086 EB11                    jmp 00440099   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00440082(C)   
|   
:00440088 B81E000000              mov eax, 0000001E——————>1E 就是 30 天   
:0044008D 33C9                    xor ecx, ecx   
:0044008F 2BC3                    sub eax, ebx——————>用 30 天减去已试用的天数   
:00440091 85C0                    test eax, eax——————>测试判断   
:00440093 0F9EC1                  setle cl   
:00440096 49                      dec ecx   
:00440097 23C1                    and eax, ecx   
  
...夜很深了，感觉很疲惫。在此收工吧。！   
  
3）小结：改三处   
  
offset@401B4h                7F40 -->EB40   
offset@3FF53h                8530 --> 31C0   
offset@2B5Eh                  7507 -->EB07   
  
★☆ moonlite 于 2001-8-6☆★   
交流请联系：qq\_midi@yahoo.com.cn  4dd466d0e1e0cc123afbe504da2462ca