

# Sygate Office Network V4.5 Build 851 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6542.htm`](https://www.pediy.com/kssd/pediy06/pediy6542.htm)

《Sygate Office Network V4.5 Build 851 简体汉化版 “隐藏托盘”》只需改半字节  
  
■ 软件说明  
 ============  
 　　Sygate Office Network 是局域网共享一至四个 MODEM 访问 INTERNET 的软件【现在谁还用 MODEM 啊，但是这个软件双网卡共享上网很好很简单很稳定，我用了从未死机，而且客户机也稳定，曾经 5 人同时上 INTERNET 打 CS，ping 值都在 20 一下】, 它基于 SyGate 核心功能，主要为商业网络提供网络连接共享，支持 Modem/ISDN/Cable Modem/ASDL/宽带、带宽分配、用户和组的管理、增强的记录分析及设置黑白名单控制用户访问 INTERNET。它完全支持 Windows 9x/NT/2000/ME/XP 系统。  
  
 ■ 软件使用  
 ============  
   此软件只需在连有调制解调器的计算机上(服务器)安装，客户机不需安装，客户机的网络设置自己的 IP 地址，和网关为服务器 IP 地址。安装完毕后启动计算机，运行 Sygate，在“工具”菜单内选择“最小化”及“启动时载入管理器”。在“工具”菜单内执行“网络诊断”，进行自动网络参数设置。你也可以通过“工具”菜单内的“配置”进行手动设置。设置完毕后，选择拨号网络连接项。可增加多个连接项，进行多个调制解调器同时访问 Internet。  
  
  
 ■ 技术支持  
 ============  
 如有问题，请与 Sygate 网站([`www.sygate.com`](http://www.sygate.com))联系。  
 廖凯电子工作室 (LiaoKai Electron Workshop) 制作汉化  
 廖凯之窗 ([`www.liaokai.com`](http://www.liaokai.com))  
 E-mail: liaokai@yeah.net  
   
 ■ pediy 目的  
 我想用老板的机器共享上网而又不想让他发现，但是 Sygate 启动之后会在任务栏加载一个图标,傻瓜也能看见，所以我要解决他!   
  
本文以下参考[`www.chat001.com/forum/crackforum/259588.html，by`](http://www.chat001.com/forum/crackforum/259588.html，by) PowerBoy  
  
 【“要向系统托盘中加入和删除图标必须要用到 SHELL32.Shell\_NotifyIconA 这个函数,也就是说当软件启动的时候一定是调用这个函数将自己的图标加入到系统托盘中;在关闭软件的时候他也是调用这个函数将图标删除的.   
  
我们来查一下 WINAPI 函数表来确定一下这个函数的使用方法和格式.   
Const NIM\_ADD = &H0//这个就是向托盘加入图标的参数   
Const NIM\_MODIFY = &H1   
Const NIM\_DELETE = &H2//这个就是从托盘中删除图标的参数   
  
Const NIF\_MESSAGE = &H1   
Const NIF\_ICON = &H2   
Const NIF\_TIP = &H4   
  
Declare Function Shell\_NotifyIcon Lib "shell32.dll" Alias " Shell\_NotifyIconA"   
  
(ByVal dwMessage As Long, lpData As   
  
NOTIFYICONDATA) As Long   
上面函数的参数一定要记下来下面有用啊! ”】  
  
我们用 WINASM32 来反汇编软件,在函数中查找 SHELL32.Shell\_NotifyIconA 这个函数看看系统中是怎样调用他的,双击几次我们发现软件一共有 4 处调用这个函数. 到此我们甚至不用动态跟踪了,直接来到这里，当然你也可以试试其它，但是只有这里是 SHELL32.Shell\_NotifyIconA 前面是 push 00000000，所以锁定。  
  
\* Referenced by a CALL at Address:  
|:00420192     
|  
:0041A4D0 833900                  cmp dword ptr \[ecx\], 00000000  
:0041A4D3 7403                    je 0041A4D8  
:0041A4D5 33C0                    xor eax, eax  
:0041A4D7 C3                      ret  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041A4D3(C)  
|  
:0041A4D8 8B4104                  mov eax, dword ptr \[ecx+04\]  
:0041A4DB 85C0                    test eax, eax  
:0041A4DD 7503                    jne 0041A4E2  
:0041A4DF 33C0                    xor eax, eax  
:0041A4E1 C3                      ret  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0041A4DD(C)  
|  
:0041A4E2 C70101000000            mov dword ptr \[ecx\], 00000001  
:0041A4E8 83C108                  add ecx, 00000008  
:0041A4EB 51                      push ecx  
:0041A4EC 6A00                    push 00000000   <<<<<<<<我们改这里 push 00000002    
                                   <<<<<<<“2”就是从托盘中删除图标的参数  
                          <<<<<<<<6A00 改成 6A02，只改半字节！  
\* Reference To: SHELL32.Shell\_NotifyIconA, Ord:0112h  <<<<<<<<关键 API  
                                  |  
:0041A4EE FF1504744700            Call dword ptr \[00477404\]  
:0041A4F4 C3                      ret  
:0041A4F5 90                      nop  
................  
  
我们回头看看 00420192 处的 call，自然也明白这是要改的地方  
\* Referenced by a CALL at Addresses:  
|:0041FEBF   , :0041FF51     
|  
:00420110 6AFF                    push FFFFFFFF  
:00420112 68682E4700              push 00472E68  
:00420117 64A100000000            mov eax, dword ptr fs:\[00000000\]  
:0042011D 50                      push eax  
:0042011E 64892500000000          mov dword ptr fs:\[00000000\], esp  
:00420125 51                      push ecx  
:00420126 A140274900              mov eax, dword ptr \[00492740\]  
:0042012B 53                      push ebx  
:0042012C 55                      push ebp  
:0042012D 56                      push esi  
:0042012E 57                      push edi  
:0042012F 8BF1                    mov esi, ecx  
:00420131 89442410                mov dword ptr \[esp+10\], eax  
:00420135 8D4C2410                lea ecx, dword ptr \[esp+10\]  
:00420139 C744241C00000000        mov \[esp+1C\], 00000000  
:00420141 51                      push ecx  
:00420142 8BCE                    mov ecx, esi  
:00420144 E82A9E0300              call 00459F73  
:00420149 8B542410                mov edx, dword ptr \[esp+10\]  
:0042014D C786D100000083000000    mov dword ptr \[esi+000000D1\], 00000083  
:00420157 8B5AF8                  mov ebx, dword ptr \[edx\-08\]  
:0042015A E8A7C70400              call 0046C906  
:0042015F 8B6804                  mov ebp, dword ptr \[eax+04\]  
:00420162 53                      push ebx  
:00420163 8D4C2414                lea ecx, dword ptr \[esp+14\]  
:00420167 8DBE9D120000            lea edi, dword ptr \[esi+0000129D\]  
:0042016D E859D00300              call 0045D1CB  
:00420172 8B4E1C                  mov ecx, dword ptr \[esi+1C\]  
:00420175 50                      push eax  
:00420176 8B86D1000000            mov eax, dword ptr \[esi+000000D1\]  
:0042017C 50                      push eax  
:0042017D 55                      push ebp  
  
以下就是很明白的提示了：  
  
\* Possible Reference to Dialog: DialogID\_0091,CONTROL\_ID:03E8, "(for allowed period)"  
                                  |  
  
\* Possible Reference to String Resource ID=01000: "HC (C) 1997-2001 Sygate €/l?      
  
  HC@."  
                                  |  
:0042017E 68E8030000              push 000003E8  
  
\* Possible Reference to Dialog: DialogID\_015F, CONTROL\_ID:0403, ""  
                                  |  
:00420183 6803040000              push 00000403  
:00420188 51                      push ecx  
:00420189 8BCF                    mov ecx, edi  
:0042018B E8F0A2FFFF              call 0041A480  
:00420190 8BCF                    mov ecx, edi  
:00420192 E839A3FFFF              call 0041A4D0         <<<<<就是这个 call 决定了  
:00420197 8D4C2410                lea ecx, dword ptr \[esp+10\]  
:0042019B C744241CFFFFFFFF        mov \[esp+1C\], FFFFFFFF  
:004201A3 E83FCC0300              call 0045CDE7  
:004201A8 8B4C2414                mov ecx, dword ptr \[esp+14\]  
:004201AC 5F                      pop edi  
:004201AD 5E                      pop esi  
:004201AE 5D                      pop ebp  
:004201AF 5B                      pop ebx  
:004201B0 64890D00000000          mov dword ptr fs:\[00000000\], ecx  
:004201B7 83C410                  add esp, 00000010  
:004201BA C3                      ret  
.....................  
 ■ 总结  
:0041A4EC 6A00  push 00000000   <<<<<<<<我们改这里 push 00000002    
                 <<<<<<<<“2”就是从托盘中删除图标的参数  
                <<<<<<<<6A00 改成 6A02，只改半字节！  
知道此方法了，大家可以举一反三啊。  
  
 ■ 期待  
  
   如果有喜欢的朋友，可以给程序添加一个开关托盘的按钮或选项之类的，就像 QQ 的那样，再增加一个呼出热键等等，抛砖引玉了。

 4dd466d0e1e0cc123afbe504da2462ca