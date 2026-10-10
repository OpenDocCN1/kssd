

# 黎之工资 v6.0

> 原文：[`www.pediy.com/kssd/pediy03/forum760.htm`](https://www.pediy.com/kssd/pediy03/forum760.htm)

谁与我共续这破解的故事？《破解“黎之工资”对抗脱壳之故事（上集）》   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  
〖作　　者〗PaulYoung   
〖日　　期〗二○○一年三月二十五日   
〖软　　件〗黎之工资 v6.0(http://leaze.3322.net/lzgz/lzgz60.zip)   
〖破解工具〗W32DASM V8.93,TRW2000 V1.03,FILEINFO V2.43A   
AspackDie 1.1(http://mud.sz.jsinfo.net/per/aaron/files/unpackers/win/aspackdie11.zip)...   
  
　　黎之工资 V6.0 是用 Aspack v2.11 加的壳，用很多工具都可以脱，如我用 AspackDie 1.1,但无论用什么工具脱壳，脱壳后的程序一运行即一闪即逝，自行退出了。   
　　我用 TRW2000 V1.03（唉，没办法，高版本的 TRW2000 与我爱的爱“姬”不和，唯有用 1.03 了。Browse 找到已脱壳的程序，Load...OK!   
    F10　单步跟踪，看到……   
  
//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* Program Entry Point \*\*\*\*\*\*\*\*   
:0051DE6C 55                      push ebp   
:0051DE6D 8BEC                    mov ebp, esp   
:0051DE6F 83C4F4                  add esp, FFFFFFF4   
:0051DE72 53                      push ebx   
:0051DE73 56                      push esi   
:0051DE74 57                      push edi   
:0051DE75 B80CDB5100              mov eax, 0051DB0C   
:0051DE7A E86D98EEFF              call 004076EC                    \*\*\*\*\*\*   
:0051DE7F 8B3514255200            mov esi, dword ptr \[00522514\]   
:0051DE85 8B3D20225200            mov edi, dword ptr \[00522220\]   
:0051DE8B 8B0F                    mov ecx, dword ptr \[edi\]   
:0051DE8D B201                    mov dl, 01   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DE8F A190E14800              mov eax, dword ptr \[0048E190\]   
:0051DE94 E893D5F2FF              call 0044B42C                    \*\*\*\*\*\*   
:0051DE99 8906                    mov dword ptr \[esi\], eax   
:0051DE9B BB03000000              mov ebx, 00000003   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0051DEB6(C)   
|   
:0051DEA0 8B06                    mov eax, dword ptr \[esi\]   
:0051DEA2 8B10                    mov edx, dword ptr \[eax\]   
:0051DEA4 FF92D8000000            call dword ptr \[edx+000000D8\]            \*弹出登录窗口   
:0051DEAA 8B06                    mov eax, dword ptr \[esi\]   
:0051DEAC 83B83402000001          cmp dword ptr \[eax+00000234\], 00000001    \*口令是否正确？   
:0051DEB3 7403                    je 0051DEB8　\*不正确即出错，下 r fl z(即相当于把 je 改为 jne 或 9090，使其不跳）   
:0051DEB5 4B                      dec ebx   
:0051DEB6 75E8                    jne 0051DEA0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0051DEB3(C)   
|   
:0051DEB8 8B06                    mov eax, dword ptr \[esi\]   
:0051DEBA 83B83402000001          cmp dword ptr \[eax+00000234\], 00000001    \*\*\*比较什么？？？   
:0051DEC1 0F8574010000            jne 0051E03B　\*下 r fl z（原理同上），否则再按几下 F10 就会退出！！（关键）   
:0051DEC7 8B0F                    mov ecx, dword ptr \[edi\]   
:0051DEC9 B201                    mov dl, 01   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DECB A1286A4B00              mov eax, dword ptr \[004B6A28\]   
:0051DED0 E857D5F2FF              call 0044B42C                    \*\*\*   
:0051DED5 8B15841E5200            mov edx, dword ptr \[00521E84\]   
:0051DEDB 8902                    mov dword ptr \[edx\], eax   
:0051DEDD 8B06                    mov eax, dword ptr \[esi\]   
:0051DEDF E82452EEFF              call 00403108                    \*\*\*\*\*\*\*   
:0051DEE4 A1841E5200              mov eax, dword ptr \[00521E84\]   
:0051DEE9 8B00                    mov eax, dword ptr \[eax\]   
:0051DEEB E81C14F3FF              call 0044F30C                    \*\*\*\*\*\*\*\*   
:0051DEF0 A1841E5200              mov eax, dword ptr \[00521E84\]   
:0051DEF5 8B00                    mov eax, dword ptr \[eax\]   
:0051DEF7 8B10                    mov edx, dword ptr \[eax\]   
:0051DEF9 FF9280000000            call dword ptr \[edx+00000080\]      \*\*\*\*\*\*\*   
:0051DEFF 8B07                    mov eax, dword ptr \[edi\]   
:0051DF01 E8924BF3FF              call 00452A98                    \*\*\*\*\*\*\*\*   
:0051DF06 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"黎之工资管理"   
                                  |   
:0051DF08 BA4CE05100              mov edx, 0051E04C   
:0051DF0D E88A47F3FF              call 0045269C                    \*\*\*\*\*\*\*\*   
:0051DF12 8B0D64225200            mov ecx, dword ptr \[00522264\]   
:0051DF18 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"\`+A"   
                                  |   
:0051DF1A 8B15C8B64C00            mov edx, dword ptr \[004CB6C8\]   
:0051DF20 E88B4BF3FF              call 00452AB0                  \*\*\*   
:0051DF25 8B0D941E5200            mov ecx, dword ptr \[00521E94\]   
:0051DF2B 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF2D 8B1554C85100            mov edx, dword ptr \[0051C854\]   
:0051DF33 E8784BF3FF              call 00452AB0              \*\*\*   
:0051DF38 8B0D44215200            mov ecx, dword ptr \[00522144\]   
:0051DF3E 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF40 8B1580515100            mov edx, dword ptr \[00515180\]   
:0051DF46 E8654BF3FF              call 00452AB0              \*\*\*\*   
:0051DF4B 8B0DC81C5200            mov ecx, dword ptr \[00521CC8\]   
:0051DF51 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF53 8B15C8415000            mov edx, dword ptr \[005041C8\]   
:0051DF59 E8524BF3FF              call 00452AB0                \*\*\*\*\*   
:0051DF5E 8B0D20255200            mov ecx, dword ptr \[00522520\]   
:0051DF64 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF66 8B15349D5100            mov edx, dword ptr \[00519D34\]   
:0051DF6C E83F4BF3FF              call 00452AB0                \*\*\*\*\*\*   
:0051DF71 8B0D90225200            mov ecx, dword ptr \[00522290\]   
:0051DF77 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF79 8B15D8D34F00            mov edx, dword ptr \[004FD3D8\]   
:0051DF7F E82C4BF3FF              call 00452AB0              \*\*\*   
:0051DF84 8B0D80205200            mov ecx, dword ptr \[00522080\]   
:0051DF8A 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF8C 8B1580FC4F00            mov edx, dword ptr \[004FFC80\]   
:0051DF92 E8194BF3FF              call 00452AB0          \*\*\*   
:0051DF97 8B0DE81C5200            mov ecx, dword ptr \[00521CE8\]   
:0051DF9D 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DF9F 8B153C8D4F00            mov edx, dword ptr \[004F8D3C\]   
:0051DFA5 E8064BF3FF              call 00452AB0          \*\*\*   
:0051DFAA 8B0D4C225200            mov ecx, dword ptr \[0052224C\]   
:0051DFB0 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DFB2 8B150C245100            mov edx, dword ptr \[0051240C\]   
:0051DFB8 E8F34AF3FF              call 00452AB0                \*\*\*   
:0051DFBD 8B0DAC245200            mov ecx, dword ptr \[005224AC\]   
:0051DFC3 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DFC5 8B15B86F4F00            mov edx, dword ptr \[004F6FB8\]   
:0051DFCB E8E04AF3FF              call 00452AB0              \*\*\*   
:0051DFD0 8B0D2C235200            mov ecx, dword ptr \[0052232C\]   
:0051DFD6 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DFD8 8B1564885000            mov edx, dword ptr \[00508864\]   
:0051DFDE E8CD4AF3FF              call 00452AB0            \*\*\*   
:0051DFE3 8B0D101F5200            mov ecx, dword ptr \[00521F10\]   
:0051DFE9 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DFEB 8B153CE55000            mov edx, dword ptr \[0050E53C\]   
:0051DFF1 E8BA4AF3FF              call 00452AB0              \*\*\*   
:0051DFF6 8B0DC0215200            mov ecx, dword ptr \[005221C0\]   
:0051DFFC 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051DFFE 8B1548375000            mov edx, dword ptr \[00503748\]   
:0051E004 E8A74AF3FF              call 00452AB0              \*\*\*   
:0051E009 8B0DF8225200            mov ecx, dword ptr \[005222F8\]   
:0051E00F 8B07                    mov eax, dword ptr \[edi\]   
  
\* Possible StringData Ref from Code Obj ->"艾 C"   
                                  |   
:0051E011 8B15E8D05000            mov edx, dword ptr \[0050D0E8\]   
:0051E017 E8944AF3FF              call 00452AB0              \*\*\*   
:0051E01C A1841E5200              mov eax, dword ptr \[00521E84\]   
:0051E021 8B00                    mov eax, dword ptr \[eax\]   
:0051E023 E8DC12F3FF              call 0044F304　　　　　　　　　　\*\*\*\*程序中止，自行退出！！   
:0051E028 A1841E5200              mov eax, dword ptr \[00521E84\]   
:0051E02D 8B00                    mov eax, dword ptr \[eax\]   
:0051E02F E8D450EEFF              call 00403108   
:0051E034 8B07                    mov eax, dword ptr \[edi\]   
:0051E036 E8F54AF3FF              call 00452B30   
.   
.   
.   
  
　　以上每经过一个 call 程序就会运行一步，一直到 0051E023 call 0044F304 处退出。如果不在 0051DEC1 jne 0051E03B 处下 r fl z,程序很快就会退出。也就是脱壳后的程序马上退出的地方了（惨……连程序的界面是啥模样都不让人看）。把 0051DEC1 jne 0051E03B 的 jne 改为 je 或 9090 后，程序运行一段时间才退出。到底还有什么地方还在校验呢？我就找不到了。希望哪位高手赏脸，试一试，指点一下我这位初哥，在此先行拜谢了。 

在主程序中发送退出消息然后在消息循环中退出的程序如何破解   
作者：小牧童\[CCG\]   
  
现在很多程序，如果没有软件狗，或时间过期，在运行中途就自动退出，常规的教学都是让拦或用 W32dasm 查找 exitprocess 函数，但当拦到这个函数已经退出主程序了，作用不大，其实还有一条好用的，这就是 bpx postquitmessage 下面以我破解得不完全的黎之工资为例说明：   
上次我破的程序不完全，时间超过 2001 年后，打印功能失效。即然是与时间相关我就下 bpx getlocaltime 按报表打印拦到后按 F12 后到下面：（记不清几次了）   
:0051D2D9 E98A65EEFF              jmp 00403868   
:0051D2DE EBF0                    jmp 0051D2D0   
:0051D2E0 8B45D4                  mov eax, dword ptr \[ebp\-2C\]   
:0051D2E3 8B55DC                  mov edx, dword ptr \[ebp\-24\]   
:0051D2E6 E8916EEEFF              call 0040417C   
:0051D2EB 730D                    jnb 0051D2FA   
:0051D2ED 8B45D8                  mov eax, dword ptr \[ebp\-28\]   
:0051D2F0 8B55E0                  mov edx, dword ptr \[ebp\-20\]   
:0051D2F3 E8846EEEFF              call 0040417C   
:0051D2F8 7249                    jb 0051D343          //将这里改为 JMP    
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0051D2EB(C)   
|   
:0051D2FA A164225200              mov eax, dword ptr \[00522264\]   
:0051D2FF 8B00                    mov eax, dword ptr \[eax\]   
:0051D301 8B4074                  mov eax, dword ptr \[eax+74\]   
:0051D304 8B4024                  mov eax, dword ptr \[eax+24\]   
:0051D307 BA0C000000              mov edx, 0000000C   
:0051D30C E823ADF4FF              call 00468034   
:0051D311 8D5584                  lea edx, dword ptr \[ebp\-7C\]   
:0051D314 8B08                    mov ecx, dword ptr \[eax\]   
:0051D316 FF5158                  call \[ecx+58\]   
在 softice 环境中再按数次 F12 打印界面出现了，但一按 F5 程序自动退出了。   
别急，打印界面能出现说明前面的判断是对的，后面还有比较，现在是解决程序退出问题   
首先改好 0051d2f8 处，然后 bpx postquitmessage   
拦到后按 F12 到下面   
\* Referenced by a CALL at Addresses:   
|:0044F1C3  , :0048E4BE  , :0048E53B  , :0048E5A4  , :004B6DC0    
|:004B7347  , :0050EFEC  , :0051D0EF  , :0051D599    
|   
:00452BE4 E88BB6FBFF              call 0040E274   
:00452BE9 84C0                    test al, al   
:00452BEB 7407                    je 00452BF4   
:00452BED 6A00                    push 00000000   
  
\* Reference To: user32.PostQuitMessage, Ord:0000h   
                                  |   
:00452BEF E86455FBFF              Call 00408158  //就是这里让程序退出   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00452BEB(C)   
|   
:00452BF4 C3                      ret   
总于让我抓住狐狸的尾巴了，现在的关键是让这条 psostquitmessage 指令不运行   
如果单纯的改上面的 00452BEB 让其 JMP 的话程序是能运行了，但是退不出去了，现在是要知道是什么地方调用这段过程，临时在 00452beb 处设断，拦到后用 r fl z 让其不运行 psostquitmessage 按 F10 到这里：   
:0050EFCC D805A4F05000            fadd dword ptr \[0050F0A4\]   
:0050EFD2 DB7DF0                  fstp tbyte ptr \[ebp\-10\]   
:0050EFD5 9B                      wait   
:0050EFD6 E8F5C0EFFF              call 0040B0D0   
:0050EFDB DB6DF0                  fld tbyte ptr \[ebp\-10\]   
:0050EFDE DED9                    fcompp   
:0050EFE0 DFE0                    fstsw ax   
:0050EFE2 9E                      sahf   
:0050EFE3 730C                    jnb 0050EFF1   
:0050EFE5 A120225200              mov eax, dword ptr \[00522220\]   
:0050EFEA 8B00                    mov eax, dword ptr \[eax\]   
:0050EFEC E8F33BF4FF              call 00452BE4    ／/原来就是在这里调用的   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0050EFAE(C), :0050EFE3(C)   
  
到此当然知道去改什么地方去了将 0050EFE3 处改为 JMP，我也跟着 Jump 了，不知这次是否改全了？大家试试看。  4dd466d0e1e0cc123afbe504da2462ca