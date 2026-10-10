

# WS\_FTP Pro 7.02 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue188.htm`](https://www.pediy.com/kssd/pediy04/kanxue188.htm)

破解 WS\_FTP Pro 7.02   
by Fpc\[CCG\]/6767\[BCG\]    2001/10   
  
tools: Trw2000, wdasm, filemon   
  
软件名称：WS\_FTP Pro   
整理日期：2001.10.27   
最新版本：7.02   
文件大小：4093KB   
软件授权：共享软件   
使用平台：Win9x/Me/NT/2000   
发布公司：Home Page   
软件简介：一个快速、强大的 FTP 客户程序。除了继续保留传统的双窗格的风格外，新版本更增加了 WS-FTP Explorer 作为 Windows Explorer (资源管理器)的扩展，因此，你可以在资源管理器中直接访问 FTP 站点。FTP 传送就像在 Windows 资源管理器中复制文件一样简单，你不必关心文档在什么类型的服务器上，或者你正在使用何种 Internet 协议!   
  
  
\[Begin\]   
  
    这是一个善于自我吹嘘的 FTP 工具~~其共享版本提供了 30 天的全部功能试用，每次运行出现一个 NAG，提醒你注册。来看一下如何解决。   
  
    它的 exe 文件由 VC 编译，没有加壳，难得~，此外还有一堆 dll。无处可以输入注册码。用 regmon 和 filemon 找找，只有这个文件可疑：key.dat。用记事本打开这个文件，发现大部分内容是文本，也正是显示在 nag 窗的内容，好象是个脚本。   
  
    trw 载入文件，设下断点：bpx createfilea do "d \*(esp+4)"。拦到后看到文件名正是 key.dat，一次 F12，接下来用 f10 向下跟。它读取文件内容，验证长度为 0x1390 字节，并且前 0x1380 所计算得到的校验数据(10 字节)要与文件最后的 10 个字节一致。这些对于注册没什么帮助。按 F12，如果你运气好，程序没有跑丢，就返回到下面(可能是 trw 的 bug)：   
  
  
文件 wsftpext.dll 的部分内容：   
... ...   
:1003C68E 51                      push ecx   
  
\* Possible StringData Ref from Data Obj ->"WS\_FTP Pro"   
                                  |   
:1003C68F 8B15B07D0510            mov edx, dword ptr \[10057DB0\]   
:1003C695 52                      push edx   
:1003C696 8B450C                  mov eax, dword ptr \[ebp+0C\]   
:1003C699 50                      push eax   
:1003C69A E8F1640000              call 10042B90                    <- 这里是对 key.dat 的验证   
:1003C69F 83C420                  add esp, 00000020                <- 返回处   
:1003C6A2 8985F4FEFFFF            mov dword ptr \[ebp+FFFFFEF4\], eax        <- 保存出口状态   
:1003C6A8 83BDF4FEFFFF00          cmp dword ptr \[ebp+FFFFFEF4\], 00000000   
:1003C6AF 7407                    je 1003C6B8                    <- 这里改一下：jmp 1003C6FB(EB 4A)   
:1003C6B1 B800000780              mov eax, 80070000   
:1003C6B6 EB4E                    jmp 1003C706   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1003C6AF(C)   
|   
:1003C6B8 837D1000                cmp dword ptr \[ebp+10\], 00000000   
:1003C6BC 7509                    jne 1003C6C7   
  
\* Reference To: USER32.GetActiveWindow, Ord:00DDh                <- 取活动窗口的的句柄   
                                  |   
:1003C6BE FF1544C30410            Call dword ptr \[1004C344\]   
:1003C6C4 894510                  mov dword ptr \[ebp+10\], eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1003C6BC(C)   
|   
:1003C6C7 8B4D14                  mov ecx, dword ptr \[ebp+14\]   
:1003C6CA 51                      push ecx   
:1003C6CB 8B5510                  mov edx, dword ptr \[ebp+10\]   
:1003C6CE 52                      push edx   
:1003C6CF E82C6D0000              call 10043400                    <- 显示 Nag   
:1003C6D4 83C408                  add esp, 00000008   
:1003C6D7 8985F0FEFFFF            mov dword ptr \[ebp+FFFFFEF0\], eax   
:1003C6DD 83BDF0FEFFFF00          cmp dword ptr \[ebp+FFFFFEF0\], 00000000   
:1003C6E4 740C                    je 1003C6F2                    <- 选择了试用按钮，则 eax=0; 如果过了试用期，eax 为小于 0 的一个值   
:1003C6E6 E8A5690000              call 10043090   
:1003C6EB B800000780              mov eax, 80070000   
:1003C6F0 EB14                    jmp 1003C706   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1003C6E4(C)   
|   
:1003C6F2 EB07                    jmp 1003C6FB   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1003C636(C)   
|   
:1003C6F4 B800003380              mov eax, 80330000   
:1003C6F9 EB0B                    jmp 1003C706   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:1003C6F2(U)   
|   
:1003C6FB 8B4508                  mov eax, dword ptr \[ebp+08\]            <- 这里必须执行到，\[ebp+8\]置 1 表示程序正常，为 0 则导致退出   
  
:1003C6FE C70001000000            mov dword ptr \[eax\], 00000001   
:1003C704 33C0                    xor eax, eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:1003C6B6(U), :1003C6F0(U), :1003C6F9(U)   
|   
:1003C706 8BE5                    mov esp, ebp   
:1003C708 5D                      pop ebp   
:1003C709 C21000                  ret 0010   
  
... ...   
  
  
    简单分析上面的代码，:1003C6FB 必须运行到，否则程序退出，而看上面的 reference，只有一个跳转可到这里，是在你看过了 nag 之后。key.dat 内容改变也不能跳过 nag 而运行程序，所以只能暴力跳过那个 nag 窗，同时时间限制解除!   
  
    再按几次 F12，返回到 exe 的代码空间，看有没有线索：   
  
... ...   
  
:00426462 740A                    je 0042646E   
:00426464 C78548F7FFFF07000000    mov dword ptr \[ebp+FFFFF748\], 00000007   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00426462(C)   
|   
:0042646E C7856CFCFFFF00000000    mov dword ptr \[ebp+FFFFFC6C\], 00000000   
:00426478 8D8D6CFCFFFF            lea ecx, dword ptr \[ebp+FFFFFC6C\]   
:0042647E 51                      push ecx   
:0042647F 8D9544F7FFFF            lea edx, dword ptr \[ebp+FFFFF744\]   
:00426485 52                      push edx   
  
\* Reference To: wsftpext.GetWsftpextApi, Ord:0000h            <- 名字   
                                  |   
:00426486 FF15A09A4200            Call dword ptr \[00429AA0\]   
:0042648C 83C408                  add esp, 00000008            <- 返回处   
:0042648F 8B8D28F4FFFF            mov ecx, dword ptr \[ebp+FFFFF428\]   
:00426495 898138010000            mov dword ptr \[ecx+00000138\], eax   
:0042649B 8B9528F4FFFF            mov edx, dword ptr \[ebp+FFFFF428\]   
:004264A1 83BA3801000000          cmp dword ptr \[edx+00000138\], 00000000   
:004264A8 7524                    jne 004264CE   
:004264AA C78540F4FFFF00000000    mov dword ptr \[ebp+FFFFF440\], 00000000   
:004264B4 C745FCFFFFFFFF          mov \[ebp\-04\], FFFFFFFF   
:004264BB 8D4DA4                  lea ecx, dword ptr \[ebp\-5C\]   
:004264BE E84D73FEFF              call 0040D810   
:004264C3 8B8540F4FFFF            mov eax, dword ptr \[ebp+FFFFF440\]   
:004264C9 E932070000              jmp 00426C00                <- 这里跳下去就退出了   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004264A8(C)   
|   
:004264CE 83BD70FCFFFF05          cmp dword ptr \[ebp+FFFFFC70\], 00000005 <- 下面是程序主线，同样没有越过 nag 而能正常运行的跳转   
:004264D5 0F8540010000            jne 0042661B   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00426616(U)   
|   
:004264DB 83BD70FCFFFF05          cmp dword ptr \[ebp+FFFFFC70\], 00000005   
:004264E2 0F8533010000            jne 0042661B   
:004264E8 8D8D1CF6FFFF            lea ecx, dword ptr \[ebp+FFFFF61C\]   
  
\* Reference To: wsftpctl.??0CIpsInputDialog@@QAE@XZ, Ord:0006h   
                                  |   
:004264EE FF15289A4200            Call dword ptr \[00429A28\]   
:004264F4 C645FC01                mov \[ebp\-04\], 01   
:004264F8 8D8D24F6FFFF            lea ecx, dword ptr \[ebp+FFFFF624\]   
  
\* Reference To: MFC42.Ordinal:021C, Ord:021Ch   
                                  |   
:004264FE E87B0C0000              Call 0042717E   
:00426503 C645FC02                mov \[ebp\-04\], 02   
  
\* Possible Reference to Dialog: DialogID\_0083   
                                  |   
  
\* Possible Reference to String Resource ID=00131: "Enter command line:"   
                                  |   
:00426507 6883000000              push 00000083   
:0042650C 8D8D24F6FFFF            lea ecx, dword ptr \[ebp+FFFFF624\]   
  
\* Reference To: MFC42.Ordinal:1040, Ord:1040h   
                                  |   
:00426512 E8610C0000              Call 00427178   
:00426517 8D8D28F6FFFF            lea ecx, dword ptr \[ebp+FFFFF628\]   
  
\* Reference To: MFC42.Ordinal:021C, Ord:021Ch   
                                  |   
:0042651D E85C0C0000              Call 0042717E   
:00426522 C645FC03                mov \[ebp\-04\], 03   
  
\* Possible Reference to String Resource ID=00132: "Input"   
                                  |   
:00426526 6884000000              push 00000084   
:0042652B 8D8D28F6FFFF            lea ecx, dword ptr \[ebp+FFFFF628\]   
  
\* Reference To: MFC42.Ordinal:1040, Ord:1040h   
                                  |   
:00426531 E8420C0000              Call 00427178   
  
... ...   
  
    所以程序的保护应该是就在 wsftpext.dll 上，如果你向作者注册，他会给你没有 nag 的 dll 文件，而我们只能暴破~  4dd466d0e1e0cc123afbe504da2462ca