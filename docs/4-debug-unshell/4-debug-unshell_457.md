

# 移花接木之 ASProtect 1.23 SDK 之 Aspack2.12

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-496.htm`](https://www.pediy.com/kssd/pediy07/pediy7-496.htm)

【脱文标题】 移花接木之 ASProtect 1.23 SDK 之 Aspack2.12 2002 版主程序脱壳破解  
   
【脱文作者】 weiyi75\[Dfcg\]\[D.4S\]  
   
【作者邮箱】 weiyi75@sohu.com  
   
【作者主页】 Dfcg 官方大本营+龙族联盟论坛  
   
【使用工具】 Ollydbg1.10b,Imprec1.42,LoadPe,PeEditor  
   
【破解平台】 Win2000/XP  
   
【软件名称】 Aspack2.12 2002 版主程序  
   
【下载地址】 [www.aspack.com](http://www.aspack.com)   
   
【软件简介】 ASPack is an advanced Win32 executable file compressor, capable of reducing the file size of 32-bit Windows programs by as much as 70%. (ASPack's compression ratio improves upon the industry-standard zip file format by as much as 10-20%.) ASPack makes Windows 95/98/NT programs and libraries smaller, and decrease load times across networks, and download times from the internet; it also protects programs against reverse engineering by non-professional hackers. Programs compressed with ASPack are self-contained and run exactly as before, with no runtime performance penalties.   
   
【软件大小】 189k  
   
【加壳方式】 ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov  
   
【保护方式】 RSAKEY 保护+30 天日期限制。  
   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
   
\--------------------------------------------------------------------------------  
   
【破解内容】  
  
几年前与 Jeffzhang 相识，蒙他和 Jwh51 指点，学了不少东西。两人都忙于事业，都已潜水。武汉牛人 Jeffzhang 实力真是令人佩服，以他为偶像。  
  
当年送我一个 Aspack2.12 主程序脱壳破解，这个是没有 Rsakey 的，他可以脱壳成完整版本，牛。那时我的技术还不能理解，只写了一篇 ASProtect 1.23 SDK 之 Aspack2.12r 主程序脱壳去暗桩这个是有 Rsakey 的，今天又看了一下，我也可以嫁接出一个完整版本出来。  
  
想起看学的一段话，Asprotect SDK 开发程序时可以在一段关键代码设置开始结束标记，没有 RsaKey 这段会直接跳过也就是 Demo 版本，想完整版本除非买 Key 或攻破其 1024 算法。  
  
Aspack2.11 版我学习破解时就开始用了，确实帮我节约了不少硬盘空间。有些程序压缩虽然可以用但会有问题，如 VB6.0 主程序压缩后有时提示 I/O 错误，某些软件以为你在破解来个温馨提示。当时 Aspack2.11 网上是有注册机的，被 Aspack 作者升级为 Aspack2.12r 就是看学论坛提供的版本，关于是 2001 版，UG2002 已经有 Rsakey,但名字是 SAC/2002!确实搞破解的不喜欢自己用的东西用别人的名字，Aspack 作者已将其加入小黑，2002 版此 Key 已经无效。  
  
运气不错，2001 和 2002 版代码变化不大，区别当然有，自己调试体会。用 Asprotect 保护的程序实在不少，Rsakey 每次升级 Key 也升级，如果可以找到规律多好，可以破解同步升级，我想 temerata\[DFCG\]应该体会最深，应用更广。  
  
这篇其实是 ASProtect 1.23 SDK 之 Aspack2.12r 主程序脱壳去暗桩的姊妹篇，读者可以参考一下。  
  
首先看完 ASProtect 1.23 SDK 之 Aspack2.12r 主程序脱壳去暗桩后，发现 Aspack 主程序修复有三部分。  
  
中间是解码部分，难度较高，而且需要 Key，其它两处只是简单验证壳是否存在。  
  
我先导入了 SAC/2002!的 Key,OD 载入 Aspack2.12 2002 版。  
  
OD 异常设置不忽略内存异常，其余全部忽略。  
  
00401000 A>  68 01804600                push ASPack.00468001 //外壳入口。  
00401005     E8 01000000                call ASPack.0040100B  
0040100A     C3                         retn  
0040100B     C3                         retn  
0040100C     3B92 7724E04E              cmp edx,dword ptr ds:\[edx+4EE02477\]  
00401012     04 84                      add al,84  
00401014     3D 62110889                cmp eax,89081162  
00401019     1905 9080A438              sbb dword ptr ds:\[38A48090\],eax  
0040101F     45                         inc ebp  
00401020     E1 09                      loopde short ASPack.0040102B  
00401022     2301                       and eax,dword ptr ds:\[ecx\]  
00401024     2872 48                    sub byte ptr ds:\[edx+48\],dh  
00401027     7A 42                      jpe short ASPack.0040106B  
.........................................................................  
  
F9 运行。  
  
1.寻找注册名位置  
  
00990739     3100                       xor dword ptr ds:\[eax\],eax //内存异常。  
0099073B     EB 01                      jmp short 0099073E  
0099073D     68 648F0500                push 58F64  
00990742     0000                       add byte ptr ds:\[eax\],al  
00990744     00EB                       add bl,ch  
00990746     02E8                       add ch,al  
00990748     0158 68                    add dword ptr ds:\[eax+68\],ebx  
0099074B     F8                         clc  
0099074C   ^ E2 98                      loopd short 009906E6  
0099074E     0068 74                    add byte ptr ds:\[eax+74\],ch  
00990751     0899 0068A4FD              or byte ptr ds:\[ecx+FDA46800\],bl  
00990757     98                         cwde  
00990758     0068 54                    add byte ptr ds:\[eax+54\],ch  
0099075B     FA                         cli  
0099075C     98                         cwde  
0099075D     0068 2C                    add byte ptr ds:\[eax+2C\],ch  
00990760     F4                         hlt  
00990761     98                         cwde  
00990762     0068 BC                    add byte ptr ds:\[eax-44\],ch  
00990765     EE                         out dx,al  
.........................................................................  
  
继续 Shift+F9 来到这里  
  
00990F07     3100                       xor dword ptr ds:\[eax\],eax  //看堆栈中好像正校验 Rsakey 真伪  
00990F09     EB 01                      jmp short 00990F0C  
00990F0B     68 648F0500                push 58F64  
00990F10     0000                       add byte ptr ds:\[eax\],al  
00990F12     00EB                       add bl,ch  
00990F14     02E8                       add ch,al  
00990F16     0158 8D                    add dword ptr ds:\[eax-73\],ebx  
00990F19     05 6C369900                add eax,99366C  
00990F1E     50                         push eax  
00990F1F     E8 04F6FFFF                call 00990528  
00990F24     E8 3A000000                call 00990F63  
00990F29     68 320F9900                push 990F32  
00990F2E     FF0424                     inc dword ptr ss:\[esp\]  
00990F31     C3                         retn  
00990F32     BC 8B44240C                mov esp,0C24448B  
00990F37     EB 01                      jmp short 00990F3A  
00990F39     8683 80B80000              xchg byte ptr ds:\[ebx+B880\],al  
00990F3F     0002                       add byte ptr ds:\[edx\],al  
00990F41     EB 1A                      jmp short 00990F5D  
00990F43     BC EB17E8EB                mov esp,EBE817EB  
00990F48     14 E8                      adc al,0E8  
00990F4A     EB 11                      jmp short 00990F5D  
00990F4C     E8 EB0EE8EB                call EC811E3C  
00990F51     0BE8                       or ebp,eax  
00990F53     EB 08                      jmp short 00990F5D  
00990F55     CD20 EB04E8EB              vxdcall EBE804EB  
00990F5B     01E8                       add eax,ebp  
00990F5D     31C0                       xor eax,eax  
00990F5F     C3                         retn  
  
堆栈友好提示  
  
0012D71C    0012D724  指针到下一个 SEH 记录  
0012D720    00990EBE  SE 句柄  
0012D724    0012FFE0  指针到下一个 SEH 记录  
0012D728    009912CF  SE 句柄  
0012D72C    0012FF90  
0012D730    00980000  
0012D734    00960000  
0012D738    009906D4  
0012D73C    009A040C  ASCII "0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9Xca>  
0012D740    009A04C8  ASCII "0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9Xca>  
  
继续 Shift+F9 一次特殊异常，可能是 Key 错误造成的。  
  
继续 Shift+F9 来到  
  
0099108E     3100                       xor dword ptr ds:\[eax\],eax  
00991090     EB 01                      jmp short 00991093  
00991092     68 648F0500                push 58F64  
00991097     0000                       add byte ptr ds:\[eax\],al  
00991099     00EB                       add bl,ch  
0099109B     02E8                       add ch,al  
0099109D     0158 83                    add dword ptr ds:\[eax-7D\],ebx  
009910A0     FB                         sti  
009910A1     0175 64                    add dword ptr ss:\[ebp+64\],esi  
009910A4     837D F8 00                 cmp dword ptr ss:\[ebp-8\],0  
009910A8     74 5E                      je short 00991108  
009910AA     33C0                       xor eax,eax  
009910AC     55                         push ebp  
009910AD     68 F9109900                push 9910F9  
009910B2     64:FF30                    push dword ptr fs:\[eax\]  
009910B5     64:8920                    mov dword ptr fs:\[eax\],esp  
009910B8     8B45 F8                    mov eax,dword ptr ss:\[ebp-8\]  
009910BB     E8 70F7FFFF                call 00990830  
009910C0     DD5D B4                    fstp qword ptr ss:\[ebp-4C\]  
009910C3     9B                         wait  
009910C4     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
........................................................................  
  
对准 401000 Code 下 F2 断点，Shift+F9 来到这里。  
  
0044277F     90                         nop  
00442780     55                         push ebp  
00442781     8BEC                       mov ebp,esp  
00442783     8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
00442786     A3 BC564400                mov dword ptr ds:\[4456BC\],eax  //这个就是 Aspack2.12 存放注册用户名的地方，我现在的 Key 是小黑，这里当然没有内容，先记住，找这个要多调试 Asprotect 程序找规律。  
0044278B     5D                         pop ebp  
0044278C     C2 0400                    retn 4  
0044278F     90                         nop  
00442790     55                         push ebp       //当前位置  
00442791     8BEC                       mov ebp,esp   
00442793     8B45 08                    mov eax,dword ptr ss:\[ebp+8\]  
00442796     A3 FC484400                mov dword ptr ds:\[4448FC\],eax  //剩余日期  
0044279B     8B45 0C                    mov eax,dword ptr ss:\[ebp+C\]  
0044279E     A3 00494400                mov dword ptr ds:\[444900\],eax //总共日期  
004427A3     5D                         pop ebp  
004427A4     C2 0800                    retn 8  
........................................................................  
  
继续 Shift+F9 两次来到最后一次异常  
  
009900E1     3100                       xor dword ptr ds:\[eax\],eax  
009900E3     64:8F05 00000000           pop dword ptr fs:\[0\]  
009900EA     58                         pop eax  
009900EB     833D DC399900 00           cmp dword ptr ds:\[9939DC\],0  
009900F2     74 14                      je short 00990108  
009900F4     6A 0C                      push 0C  
009900F6     B9 DC399900                mov ecx,9939DC  
009900FB     8D45 F8                    lea eax,dword ptr ss:\[ebp-8\]  
009900FE     BA 04000000                mov edx,4  
00990103     E8 94C3FFFF                call 0098C49C  
00990108     FF75 FC                    push dword ptr ss:\[ebp-4\]  
0099010B     FF75 F8                    push dword ptr ss:\[ebp-8\]  
0099010E     8B45 F4                    mov eax,dword ptr ss:\[ebp-C\]  
00990111     8338 00                    cmp dword ptr ds:\[eax\],0  
00990114     74 02                      je short 00990118  
00990116     FF30                       push dword ptr ds:\[eax\]  
00990118     FF75 F0                    push dword ptr ss:\[ebp-10\]  
0099011B     FF65 EC                    jmp dword ptr ss:\[ebp-14\]  
0099011E     5F                         pop edi  
0099011F     5E                         pop esi  
00990120     5B                         pop ebx  
00990121     8BE5                       mov esp,ebp  
00990123     5D                         pop ebp  
00990124     C3                         retn  
........................................................................  
  
0044289C     55                         push ebp  //早期的 Asprotect 善良，可以在入口脱壳,先写注册名。  
  
命令行  
  
dd 4456BC 内容为 00983431，有些 Asprotect 校验这里验证壳是否存在。  
  
寻找空地址  
  
445FDC 写入 Mr.David  
  
然后修改  
  
4456BC 内容为 445FDC   
  
0044289D     8BEC                       mov ebp,esp  
0044289F     83C4 F4                    add esp,-0C  
004428A2     E8 4D0BFCFF                call ASPack.004033F4  
004428A7     E8 0C21FCFF                call ASPack.004049B8  
004428AC     E8 6354FCFF                call ASPack.00407D14  
004428B1     E8 D6C6FCFF                call ASPack.0040EF8C  
004428B6     E8 BDC7FCFF                call ASPack.0040F078  
004428BB     E8 08E7FCFF                call ASPack.00410FC8  
004428C0     E8 974DFDFF                call ASPack.0041765C  
004428C5     E8 B61AFEFF                call ASPack.00424380  
004428CA     E8 F986FEFF                call ASPack.0042AFC8  
004428CF     E8 E899FEFF                call ASPack.0042C2BC  
004428D4     E8 639AFEFF                call ASPack.0042C33C  
004428D9     E8 CEA5FEFF                call ASPack.0042CEAC  
004428DE     E8 95ABFEFF                call ASPack.0042D478  
004428E3     E8 30BAFEFF                call ASPack.0042E318  
004428E8     E8 6FBAFEFF                call ASPack.0042E35C  
004428ED     E8 FABAFEFF                call ASPack.0042E3EC  
004428F2     E8 69E1FEFF                call ASPack.00430A60  
004428F7     E8 A4AFFFFF                call ASPack.0043D8A0  
004428FC     A1 30564400                mov eax,dword ptr ds:\[445630\]  
00442901     E8 D210FEFF                call ASPack.004239D8  
00442906     BA 38294400                mov edx,ASPack.00442938                      ; ASCII "ASPack"  
...................................................................................................  
  
然后 Loadpe 脱壳，运行 ImportREC，选择这个进程。把 OEP 改为 4289C，点 IT AutoSearch，点“Get Import”，用“追踪层次 1”修复，然后用 Asprotect1.22 插件修复剩下 8 个指针，FixDump，脱壳就完成了。  
  
2.SDK 修复。  
  
OD 载入脱壳程序，F9 运行出错。  
  
堆栈友好提示  
  
0012FFB0    0044291B  返回到 dumped\_.<ModuleEntryPoint>+7F 来自 0098C9A0 //访问壳，右键反汇编中跟随。  
0012FFB4    0012FFE0  指针到下一个 SEH 记录  
0012FFB8    00403330  SE 句柄  
0012FFBC    0012FFC0  
0012FFC0    0012FFF0  
0012FFC4    77E614C7  返回到 kernel32.77E614C7  
  
00442915   |.  FF15 0C494400            call dword ptr ds:\[44490C\]     //这里有问题,去原程序看看。  
0044291B   |.  A1 30564400              mov eax,dword ptr ds:\[445630\] //来到这里  
00442920   |.  E8 5311FEFF              call dumped\_.00423A78  
00442925   |.  E8 D21BFCFF              call dumped\_.004044FC  
0044292A   |.  8BE5                     mov esp,ebp  
0044292C   |.  5D                       pop ebp  
0044292D   \\.  C3                       retn  
  
主程序跟踪路线  
  
00442915     FF15 0C494400              call dword ptr ds:\[44490C\]        //都是 F7 避免跑飞  
  
0098C9A0     833D A8359900 00           cmp dword ptr ds:\[9935A8\],0  
0098C9A7     74 06                      je short 0098C9AF  
0098C9A9     FF15 A8359900              call dword ptr ds:\[9935A8\]  ; ASPack.004427A8 //验证壳是否存在,就是处于 pre-dip 中  
  
004427A8     B9 B8564400                mov ecx,ASPack.004456B8  
004427AD     BA 08E54300                mov edx,ASPack.0043E508  
004427B2     A1 30564400                mov eax,dword ptr ds:\[445630\]  
004427B7     E8 2C12FEFF                call ASPack.004239E8  
004427BC     C3                         retn  
  
那么先修复  
  
00442915   |.  FF15 0C494400            call dword ptr ds:\[44490C\]     //这里有问题,去原程序看看。  
  
原地址  
  
0098C9A0  
  
修改为  
  
004427A8  
  
复制到程序，重启 OD。  
  
F9 出错，这次是主功能了，精神些，代码也多了些。  
  
0012FE00    0043F1BE  返回到 dumped\_.0043F1BE 来自 0098C8F4  //右键反汇编中跟随。  
0012FE04    0012FE58  指针到下一个 SEH 记录  
0012FE08    0043F4D0  SE 句柄  
0012FE0C    0012FE50  
0012FE10    0012CD78  
0012FE14    0043E508  dumped\_.0043E508  
0012FE18    009620C4  
0012FE1C    00000000  
0012FE20    0096BF88  ASCII "D:\\Down\\Aspack V2.12\\dumped\_.exe"  
  
0043F1B8    .  FF15 04494400            call dword ptr ds:\[444904\]  
0043F1BE    .  E9 83000000              jmp dumped\_.0043F246 //返回这里，右键删除分析  
0043F1C3       26                       db 26                                        ;  CHAR '&'  
0043F1C4       F4                       db F4  
0043F1C5       CC                       int3  
0043F1C6       46                       db 46                                        ;  CHAR 'F'  
0043F1C7       F2                       db F2  
0043F1C8       B8                       db B8  
0043F1C9       A1                       db A1  
0043F1CA       34                       db 34                                        ;  CHAR '4'  
0043F1CB       0E                       db 0E  
0043F1CC       BD                       db BD  
0043F1CD       59                       db 59                                        ;  CHAR 'Y'  
0043F1CE       6B                       db 6B                                        ;  CHAR 'k'  
0043F1CF       0F                       db 0F  
0043F1D0       05                       db 05  
  
0043F1B8     FF15 04494400              call dword ptr ds:\[444904\] //跟踪原程序，这里只是一个简单的 ret 返回，如果这样修复只是 Demo 版，当然不能让我们满足。  
0043F1BE     E9 83000000                jmp dumped\_.0043F246  //这里根本没有解码。  
0043F1C3     26:F4                      hlt  
0043F1C5     CC                         int3  
0043F1C6     46                         inc esi  
0043F1C7     F2:                        prefix repne:  
0043F1C8     B8 A1340EBD                mov eax,BD0E34A1  
0043F1CD     59                         pop ecx  
0043F1CE     6B0F 05                    imul ecx,dword ptr ds:\[edi\],5  
0043F1D1     F8                         clc  
0043F1D2     58                         pop eax  
0043F1D3     DBD8                       fcmovnu st,st  
0043F1D5     4C                         dec esp  
0043F1D6   ^ EB B0                      jmp short dumped\_.0043F188  
0043F1D8     BC 2B2DDCE3                mov esp,E3DC2D2B  
0043F1DD     28FE                       sub dh,bh  
0043F1DF     332B                       xor ebp,dword ptr ds:\[ebx\]  
........................................................................  
  
这段代码是什么?  
  
应该类似 2.12r 版这里的代码。  
  
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
0043F5E8    > \\8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F5EB    .  8B80 E0010000            mov eax,dword ptr ds:\[eax+1E0\]  
0043F5F1    .  8B40 44                  mov eax,dword ptr ds:\[eax+44\]  
0043F5F4    .  33D2                     xor edx,edx  
0043F5F6    .  E8 2993FDFF              call David.00418924  
0043F5FB    .  8D55 D4                  lea edx,dword ptr ss:\[ebp-2C\]  
0043F5FE    .  A1 BC564400              mov eax,dword ptr ds:\[4456BC\]  
0043F603    .  E8 586DFCFF              call David.00406360  
0043F608    .  8B55 D4                  mov edx,dword ptr ss:\[ebp-2C\]  
0043F60B    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F60E    .  8B80 E0010000            mov eax,dword ptr ds:\[eax+1E0\]  
0043F614    .  E8 073BFDFF              call David.00413120  
0043F619    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F61C    .  8B80 6C020000            mov eax,dword ptr ds:\[eax+26C\]  
0043F622    .  E8 E53EFDFF              call David.0041350C  
0043F627    .  B2 01                    mov dl,1  
0043F629    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F62C    .  8B80 7C020000            mov eax,dword ptr ds:\[eax+27C\]  
0043F632    .  E8 513AFDFF              call David.00413088  
0043F637    .  B2 01                    mov dl,1  
0043F639    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F63C    .  8B80 78020000            mov eax,dword ptr ds:\[eax+278\]  
0043F642    .  E8 413AFDFF              call David.00413088  
0043F647    .  B2 01                    mov dl,1  
0043F649    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F64C    .  8B80 EC020000            mov eax,dword ptr ds:\[eax+2EC\]  
0043F652    .  E8 313AFDFF              call David.00413088  
0043F657    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F65A    .  8B80 E4020000            mov eax,dword ptr ds:\[eax+2E4\]  
0043F660    .  C640 20 01               mov byte ptr ds:\[eax+20\],1  
  
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
  
这里其它语句好猜，主要是 Call 难，当时没想通，今天想起 fly 常用的 Ctrl+S 查找 Call 的命令序列  
  
还好 Call 不多。  
  
现在去 Aspack2.12r 找。  
  
call 00418924    === Call 0041894C  
  
push ebx  
cmp edx,dword ptr ds:\[eax+10\]      
  
call 00406360   ==== Call 00406414        ^轻松愉快^  
  
push ebx  
push esi   
mov esi,edx  
mov ebx,eax  
mov eax,esi  
mov edx,ebx  
  
call 00413120    ==== 0041316C   //这个有些难，类似代码较多，仔细些。查命令序列限制 8 条，有些代码只能辅助搜索，这个自己要体会，如 Call XXXX。  
  
push ebp  
mov ebp,esp  
push 0  
push ebx  
push esi  
push edi                      //我这里取了 2.12r 版的一些特征码，OD 脚本好手们对特征码应该非常亲切。  
mov esi,edx  
mov ebx,eax  
xor eax,eax  
push ebp  
push David.00413176  
push dword ptr fs:\[eax\]  
mov dword ptr fs:\[eax\],esp  
lea edx,dword ptr ss:\[ebp-4\]  
mov eax,ebx  
call 004130F0  
mov eax,dword ptr ss:\[ebp-4\]  
mov edx,esi  
call D0040378C  
  
首先找到这里  
  
0040A8E0   /.  55                       push ebp    
0040A8E1   |.  8BEC                     mov ebp,esp  
0040A8E3   |.  6A 00                    push 0  
0040A8E5   |.  53                       push ebx  
0040A8E6   |.  56                       push esi  
0040A8E7   |.  57                       push edi  
0040A8E8   |.  8BF2                     mov esi,edx  
0040A8EA   |.  8BD8                     mov ebx,eax  
0040A8EC   |.  33C0                     xor eax,eax  
0040A8EE   |.  55                       push ebp  
0040A8EF   |.  68 35A94000              push dumped\_.0040A935  
0040A8F4   |.  64:FF30                  push dword ptr fs:\[eax\]  
0040A8F7   |.  64:8920                  mov dword ptr fs:\[eax\],esp  
0040A8FA   |.  8BC3                     mov eax,ebx       //这里不同于特征码，排除。  
0040A8FC   |.  8B10                     mov edx,dword ptr ds:\[eax\]  
0040A8FE   |.  FF52 44                  call dword ptr ds:\[edx+44\]  
0040A901   |.  8BD0                     mov edx,eax  
0040A903   |.  8D45 FC                  lea eax,dword ptr ss:\[ebp-4\]  
0040A906   |.  E8 558DFFFF              call dumped\_.00403660  
  
  
0040A9F4   /.  55                       push ebp  
0040A9F5   |.  8BEC                     mov ebp,esp  
0040A9F7   |.  6A 00                    push 0  
0040A9F9   |.  53                       push ebx  
0040A9FA   |.  56                       push esi  
0040A9FB   |.  57                       push edi           //继续 Ctrl+L 排除。  
0040A9FC   |.  8BF2                     mov esi,edx  
0040A9FE   |.  8BD8                     mov ebx,eax  
0040AA00   |.  33C0                     xor eax,eax  
0040AA02   |.  55                       push ebp  
0040AA03   |.  68 38AA4000              push dumped\_.0040AA38  
0040AA08   |.  64:FF30                  push dword ptr fs:\[eax\]  
0040AA0B   |.  64:8920                  mov dword ptr fs:\[eax\],esp  
0040AA0E   |.  8D45 FC                  lea eax,dword ptr ss:\[ebp-4\]  
0040AA11   |.  8BD6                     mov edx,esi  
0040AA13   |.  E8 488CFFFF              call dumped\_.00403660  
0040AA18   |.  8B55 FC                  mov edx,dword ptr ss:\[ebp-4\]  
0040AA1B   |.  8BC3                     mov eax,ebx  
0040AA1D   |.  8B08                     mov ecx,dword ptr ds:\[eax\]    //Mov 语句太多。  
0040AA1F   |.  FF51 24                  call dword ptr ds:\[ecx+24\]  
  
继续 Ctrl+L 3 次来到这里  
  
0041316C   /$  55                       push ebp         //真命天子  
0041316D   |.  8BEC                     mov ebp,esp  
0041316F   |.  6A 00                    push 0  
00413171   |.  53                       push ebx  
00413172   |.  56                       push esi  
00413173   |.  57                       push edi  
00413174   |.  8BF2                     mov esi,edx  
00413176   |.  8BD8                     mov ebx,eax  
00413178   |.  33C0                     xor eax,eax  
0041317A   |.  55                       push ebp  
0041317B   |.  68 C2314100              push dumped\_.004131C2  
00413180   |.  64:FF30                  push dword ptr fs:\[eax\]  
00413183   |.  64:8920                  mov dword ptr fs:\[eax\],esp  
00413186   |.  8D55 FC                  lea edx,dword ptr ss:\[ebp-4\]  
00413189   |.  8BC3                     mov eax,ebx  
0041318B   |.  E8 ACFFFFFF              call dumped\_.0041313C  
00413190   |.  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
00413193   |.  8BD6                     mov edx,esi  
00413195   |.  E8 1E06FFFF              call dumped\_.004037B8  
0041319A   |.  74 10                    je short dumped\_.004131AC  
0041319C   |.  8BC6                     mov eax,esi  
0041319E   |.  E8 C906FFFF              call dumped\_.0040386C  
004131A3   |.  8BD0                     mov edx,eax  
004131A5   |.  8BC3                     mov eax,ebx  
004131A7   |.  E8 6CFFFFFF              call dumped\_.00413118  
004131AC   |>  33C0                     xor eax,eax  
004131AE   |.  5A                       pop edx  
004131AF   |.  59                       pop ecx  
004131B0   |.  59                       pop ecx  
004131B1   |.  64:8910                  mov dword ptr fs:\[eax\],edx  
004131B4   |.  68 C9314100              push dumped\_.004131C9  
004131B9   |>  8D45 FC                  lea eax,dword ptr ss:\[ebp-4\]  
004131BC   |.  E8 7303FFFF              call dumped\_.00403534  
004131C1   \\.  C3                       retn  
  
00413120   /$  55                       push ebp       //Aspack2.12r 版特征码,是否很像。  
00413121   |.  8BEC                     mov ebp,esp  
00413123   |.  6A 00                    push 0  
00413125   |.  53                       push ebx  
00413126   |.  56                       push esi  
00413127   |.  57                       push edi  
00413128   |.  8BF2                     mov esi,edx  
0041312A   |.  8BD8                     mov ebx,eax  
0041312C   |.  33C0                     xor eax,eax  
0041312E   |.  55                       push ebp  
0041312F   |.  68 76314100              push David.00413176  
00413134   |.  64:FF30                  push dword ptr fs:\[eax\]  
00413137   |.  64:8920                  mov dword ptr fs:\[eax\],esp  
0041313A   |.  8D55 FC                  lea edx,dword ptr ss:\[ebp-4\]  
0041313D   |.  8BC3                     mov eax,ebx  
0041313F   |.  E8 ACFFFFFF              call David.004130F0  
00413144   |.  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
00413147   |.  8BD6                     mov edx,esi  
00413149   |.  E8 3E06FFFF              call David.0040378C  
0041314E   |.  74 10                    je short David.00413160  
00413150   |.  8BC6                     mov eax,esi  
00413152   |.  E8 E906FFFF              call David.00403840  
00413157   |.  8BD0                     mov edx,eax  
00413159   |.  8BC3                     mov eax,ebx  
0041315B   |.  E8 6CFFFFFF              call David.004130CC  
00413160   |>  33C0                     xor eax,eax  
00413162   |.  5A                       pop edx  
00413163   |.  59                       pop ecx  
00413164   |.  59                       pop ecx  
00413165   |.  64:8910                  mov dword ptr fs:\[eax\],edx  
00413168   |.  68 7D314100              push David.0041317D  
0041316D   |>  8D45 FC                  lea eax,dword ptr ss:\[ebp-4\]  
00413170   |.  E8 9303FFFF              call David.00403508  
00413175   \\.  C3                       retn  
  
call 0041350C   ===  Call 00413554 这个要找跳板，也不难。  
  
xor edx,edx  
call 0041304C  //特征语句少，不好找，需要判断 2002 版本 Aspack 这里 Call 是多少。  
retn  
  
我们先去  
  
2001 版本这里看看。  
  
call 0041304C   
  
0041304C   /$  53                       push ebx  
0041304D   |.  56                       push esi  
0041304E   |.  51                       push ecx  
0041304F   |.  881424                   mov byte ptr ss:\[esp\],dl  
00413052   |.  8BF0                     mov esi,eax  
00413054   |.  8A46 37                  mov al,byte ptr ds:\[esi+37\]  
00413057   |.  3A0424                   cmp al,byte ptr ss:\[esp\]  
0041305A   |.  74 28                    je short David.00413084  
0041305C   |.  8BC6                     mov eax,esi  
0041305E   |.  66:BB E8FF               mov bx,0FFE8  
00413062   |.  E8 1DFCFEFF              call David.00402C84  
  
很容易找一段  
  
push ebx  
push esi  
push ecx  
mov byte ptr ss:\[esp\],dl  
mov esi,eax  
mov al,byte ptr ds:\[esi+37\]  
cmp al,byte ptr ss:\[esp\]  
  
去 2002 版找到过渡 Call  
  
00413098   /$  53                       push ebx   //这里  
00413099   |.  56                       push esi  
0041309A   |.  51                       push ecx  
0041309B   |.  881424                   mov byte ptr ss:\[esp\],dl  
0041309E   |.  8BF0                     mov esi,eax  
004130A0   |.  8A46 37                  mov al,byte ptr ds:\[esi+37\]  
004130A3   |.  3A0424                   cmp al,byte ptr ss:\[esp\]  
  
所以算出，也是简单的算法。  
  
xor edx,edx  
call 00413098  
retn  
  
然后找到  
  
00413554   /$  33D2                     xor edx,edx  
00413556   |.  E8 3DFBFFFF              call dumped\_.00413098  
0041355B   \\.  C3                       retn  
  
call 00413088  ====4130D9-5=4030D4  
  
mov byte ptr ds:\[eax+38\],dl  
push 0                                     
xor ecx,ecx                         
mov edx,0B00C                       
  
然后去 Aspack2.12 版寻找相应 Call，整理为  
  
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
0043F1BF    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F1C2    .  8B80 E0010000            mov eax,dword ptr ds:\[eax+1E0\]  
0043F1C8    .  8B40 44                  mov eax,dword ptr ds:\[eax+44\]  
0043F1CB    .  33D2                     xor edx,edx  
0043F1CD    .  E8 7A97FDFF              call David.0041894C  
0043F1D2    .  8D55 D4                  lea edx,dword ptr ss:\[ebp-2C\]  
0043F1D5    .  A1 BC564400              mov eax,dword ptr ds:\[4456BC\]  
0043F1DA    .  E8 3572FCFF              call David.00406414  
0043F1DF    .  8B55 D4                  mov edx,dword ptr ss:\[ebp-2C\]  
0043F1E2    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F1E5    .  8B80 E0010000            mov eax,dword ptr ds:\[eax+1E0\]  
0043F1EB    .  E8 7C3FFDFF              call David.0041316C  
0043F1F0    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F1F3    .  8B80 6C020000            mov eax,dword ptr ds:\[eax+26C\]  
0043F1F9    .  E8 5643FDFF              call David.00413554  
0043F1FE    .  B2 01                    mov dl,1  
0043F200    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F203    .  8B80 7C020000            mov eax,dword ptr ds:\[eax+27C\]  
0043F209    .  E8 C63EFDFF              call David.004130D4  
0043F20E    .  B2 01                    mov dl,1  
0043F210    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F213    .  8B80 78020000            mov eax,dword ptr ds:\[eax+278\]  
0043F219    .  E8 B63EFDFF              call David.004130D4  
0043F21E    .  B2 01                    mov dl,1  
0043F220    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F223    .  8B80 EC020000            mov eax,dword ptr ds:\[eax+2EC\]  
0043F229    .  E8 A63EFDFF              call David.004130D4  
0043F22E    .  8B45 FC                  mov eax,dword ptr ss:\[ebp-4\]  
0043F231    .  8B80 E4020000            mov eax,dword ptr ds:\[eax+2E4\]  
0043F237    .  C640 20 01               mov byte ptr ds:\[eax+20\],1  
  
二进制代码  
  
8B 45 FC 8B 80 E0 01 00 00 8B 40 44 33 D2 E8 7A 97 FD FF 8D 55 D4 A1 BC 56 44 00 E8 35 72 FC FF  
8B 55 D4 8B 45 FC 8B 80 E0 01 00 00 E8 7C 3F FD FF 8B 45 FC 8B 80 6C 02 00 00 E8 56 43 FD FF B2  
01 8B 45 FC 8B 80 7C 02 00 00 E8 C6 3E FD FF B2 01 8B 45 FC 8B 80 78 02 00 00 E8 B6 3E FD FF B2  
01 8B 45 FC 8B 80 EC 02 00 00 E8 A6 3E FD FF 8B 45 FC 8B 80 E4 02 00 00 C6 40 20 01  
  
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
说了半天，不知大家看明白没有，很绕口的，如果你是二哥的 fans 估计很容易看明白，常用术语，我的系列教学等，如果需要我可以来个辅助动画。  
  
然后将上面代码写入相应地方  
  
0043F1B8     FF15 04494400              call dword ptr ds:\[444904\]   
  
dd 444904  
  
修改为 401234，你自己 G 401234 看到一个 ret 语句，所以不一定取 401234。  
  
0043F1BE     90                         nop    //去小花  
0043F1BF     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F1C2     8B80 E0010000              mov eax,dword ptr ds:\[eax+1E0\]  
0043F1C8     8B40 44                    mov eax,dword ptr ds:\[eax+44\]  
0043F1CB     33D2                       xor edx,edx  
0043F1CD     E8 7A97FDFF                call dumped\_.0041894C  
0043F1D2     8D55 D4                    lea edx,dword ptr ss:\[ebp-2C\]  
0043F1D5     A1 BC564400                mov eax,dword ptr ds:\[4456BC\]  
0043F1DA     E8 3572FCFF                call dumped\_.00406414  
0043F1DF     8B55 D4                    mov edx,dword ptr ss:\[ebp-2C\]  
0043F1E2     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F1E5     8B80 E0010000              mov eax,dword ptr ds:\[eax+1E0\]  
0043F1EB     E8 7C3FFDFF                call dumped\_.0041316C  
0043F1F0     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F1F3     8B80 6C020000              mov eax,dword ptr ds:\[eax+26C\]  
0043F1F9     E8 5643FDFF                call dumped\_.00413554  
0043F1FE     B2 01                      mov dl,1  
0043F200     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F203     8B80 7C020000              mov eax,dword ptr ds:\[eax+27C\]  
0043F209     E8 C63EFDFF                call dumped\_.004130D4  
0043F20E     B2 01                      mov dl,1  
0043F210     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F213     8B80 78020000              mov eax,dword ptr ds:\[eax+278\]  
0043F219     E8 B63EFDFF                call dumped\_.004130D4  
0043F21E     B2 01                      mov dl,1  
0043F220     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F223     8B80 EC020000              mov eax,dword ptr ds:\[eax+2EC\]  
0043F229     E8 A63EFDFF                call dumped\_.004130D4  
0043F22E     8B45 FC                    mov eax,dword ptr ss:\[ebp-4\]  
0043F231     8B80 E4020000              mov eax,dword ptr ds:\[eax+2E4\]  
0043F237     C640 20 01                 mov byte ptr ds:\[eax+20\],1  
0043F23B     90                         nop  
0043F23C     90                         nop         
0043F23D     90                         nop  
0043F23E     90                         nop  
0043F23F     90                         nop      //去小花,Demo 版是直接跳到 0043F246 的，Nop 要适度啊。  
0043F240     90                         nop  
0043F241     90                         nop  
0043F242     90                         nop  
0043F243     90                         nop  
0043F244     90                         nop  
0043F245     90                         nop   
0043F246     FF15 08494400              call dword ptr ds:\[444908\] //这里也访问壳中，顺手修改为  
  
401234  
  
这个一跟踪原程序就知道是个简单的 ret 类似 IAT 重定位。  
  
保存所有结果，完整版本就是你的了，只有自己动手的果实才有甜的感觉。  
  
然后是区段减肥，删除无用区段，OD 载入程序，自 401000-478000 区段下 F2 断点，F9 按住到程序运行。  
  
看到  
  
446000  
  
448000  
  
44A000  
  
44C000  
  
477000  
  
为理论垃圾区段，先备份程序。  
  
PeEdit 初步删除区段，只保留验证 PE 啊。  
  
正常运行，然后 Loadpe 可以减肥了。  
  
484k To 434k ，减肥过猛，Aspack 压缩可以运行但图标没有了，北斗程序提示已经压缩了，最后用 PECompact 2.x 加壳成功，434k To 225K。  
  
感觉还是很复杂，只是锻炼一下操作。不知当时 Jeffzhang 是否这个方法，如果他写应该精彩多了。如果高手有 Patch 小黑 Key 为真的方法还请指教，这样的 Rsakey 只需个人购买一个就可以 Dump 一个完整版大家分享，也可以移花接木,另外如果你的 Aspack 不幸过期了，可以用 EVACleaner 搞定。  
  
二哥 30 岁了，不能像原先那样猛写文章了，还有心事未了.....  
  
\--------------------------------------------------------------------------  
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  

 4dd466d0e1e0cc123afbe504da2462ca