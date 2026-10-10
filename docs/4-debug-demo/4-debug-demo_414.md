

# Tmaster6.0 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue086.htm`](https://www.pediy.com/kssd/pediy04/kanxue086.htm)

Tmaster6.0 是一个指法练习软件，在 http://software.wx88.net/down/typm2002.exe 可以下栽。或到 http://www.typingmaster.com/其公司主页   
下栽。目前这个试用 DEMO 版本要授权文件，可以变成正式版。   
DEMO 版只能用到第四课。   
  
工具：Trw2001.22   
Uledit 7.0   
wdasm   
进入程序后，直接学习第四课弹出对话框，告诉你是 DEMO 版，不能用，请购买之类的   
好，切入 TRW，下断点   
BPX MESSAGEBOXA   
返回程序，要求学习第五课，程序被中断，   
  
  
\* Reference To: user32.MessageBoxA, Ord:0000h   
                                  |   
:0044B6B6 E821B7FBFF              Call 00406DDC  <==在此处中断调用 MessageBoxA   
:0044B6BB 8945F8                  mov dword ptr \[ebp\-08\], eax   
:0044B6BE 33C0                    xor eax, eax   
:0044B6C0 5A                      pop edx   
:0044B6C1 59                      pop ecx   
一直按 F10 直到 TMASTER 窗口再次显示，再点第五课的练习，回到 TRW2000 的界面如下：   
:004B82AA FF5218                  call \[edx+18\]  <==控制命令传送给 WINDOWS   
:004B82AD 8BF8                    mov edi, eax   
:004B82AF 8B8630010000            mov eax, dword ptr \[esi+00000130\]   
:004B82B5 8945F8                  mov dword ptr \[ebp\-08\], eax   
:004B82B8 8BC6                    mov eax, esi   
:004B82BA E861ADF4FF              call 00403020   
:004B82BF 837DF800                cmp dword ptr \[ebp\-08\], 00000000   
:004B82C3 7507                    jne 004B82CC   
:004B82C5 BF03000000              mov edi, 00000003   
:004B82CA EB0F                    jmp 004B82DB   
一直按 F10 到这里：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004B8368(C)   
|   
:004B8373 A140BC4D00              mov eax, dword ptr \[004DBC40\]   
:004B8378 8B00                    mov eax, dword ptr \[eax\]   
:004B837A 83780C00                cmp dword ptr \[eax+0C\], 00000000  此处比较，当 eax+0c 地址处为 0 表示为试用版   
:004B837E 750B                    jne 004B838B    <==跳到正确的地址。   
:004B8380 8BC3                    mov eax, ebx   
:004B8382 E855FCFFFF              call 004B7FDC <==显示为试用版信息。   
:004B8387 8BF8                    mov edi, eax   
:004B8389 EB0C                    jmp 004B8397   
在:004B837A 行可以看到\[eax+0C\]的地址为 00d56d84, 即在内存 00d56d84 处放的是是否试用版标志。   
  
下断点 BPM 00D56D84 RW   
重新运行 Tmaster6.0 ，程序被拦住   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00402FC0(U)   
|   
:0040302C 53                      push ebx   
:0040302D 56                      push esi   
:0040302E 57                      push edi   
:0040302F 89C3                    mov ebx, eax   
:00403031 89D7                    mov edi, edx   
:00403033 AB                      stosd   
:00403034 8B4BD8                  mov ecx, dword ptr \[ebx\-28\]   
:00403037 31C0                    xor eax, eax   
:00403039 51                      push ecx   
:0040303A C1E902                  shr ecx, 02   
:0040303D 49                      dec ecx   
:0040303E F3                      repz   
:0040303F AB                      stosd  光标停在此处，此时将 EAX 改为 1，发现变成正式版   
:00403040 59                      pop ecx   
:00403041 83E103                  and ecx, 00000003   
这是调用 DLL 文件的，该模块要调用很多次，不能在这里改，否则程序将出错。   
绝续按 CTRL-N，回到 Tmaster6.0，出现用户登陆，点 OK 后程序又被拦住住。   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004D45DF(C)   
|   
:004D4660 A140BC4D00              mov eax, dword ptr \[004DBC40\]   
:004D4665 8B00                    mov eax, dword ptr \[eax\]   
:004D4667 83780C00                cmp dword ptr \[eax+0C\], 00000000 <==此处判断，当 eax+0c 地址处为 0 表示为试用版   
:004D466B 7E16                    jle 004D4683   
:004D466D A100BC4D00              mov eax, dword ptr \[004DBC00\]   
:004D4672 8B00                    mov eax, dword ptr \[eax\]   
:004D4674 57                      push edi   
:004D4675 8DB810020000            lea edi, dword ptr \[eax+00000210\]   
:004D467B BE8C4C4D00              mov esi, 004D4C8C   
:004D4680 A5                      movsd   
:004D4681 A4                      movsb   
:004D4682 5F                      pop edi   
这是程序初始化，若\[eax+0C\]不为零，将不出现 DEMO 版字样。   
  
为了将当 eax+0c 地址处改为 0，将上面的部分代码改为如下   
改为：   
0167:004D4667 57              PUSH    EDI   
0167:004D4668 040C            ADD      AL,0C   
0167:004D466A 89C7            MOV      EDI,EAX   
0167:004D466C B001            MOV      AL,01   
0167:004D466E AA              STOSB        <==把\[eax+0C\]地址处的值改为 01，表示正式版   
0167:004D466F 89F8            MOV      EAX,EDI   
0167:004D4671 5F              POP      EDI   
0167:004D4672 90              NOP      <==多的地方用空填补   
0167:004D4673 90              NOP      <==多的地方用空填补   
0167:004D4674 57              push edi   
0167:004D4675 8DB810020000    lea edi, dword ptr \[eax+00000210\]    
将上面的机器玛抄下，在 ULEDIT 中修改，即可得到正式版。  4dd466d0e1e0cc123afbe504da2462ca