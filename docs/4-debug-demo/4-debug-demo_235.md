

# CD Bar 1.03

> 原文：[`www.pediy.com/kssd/pediy01/forum048.htm`](https://www.pediy.com/kssd/pediy01/forum048.htm)

软件名称：  CD Bar   
软件版本：  1.03   
软件大小：  673KB   
软件授权：  共享软件   
使用平台：  Win95/98/NT   
发布公司：  http://www.mindspring.com/~bytor/cdbar/cdbar.htm   
软件简介：  是一个兼容性能很好的 CD 音频播放软件，它可以做为一个 taskbar（任务栏）之中的选项，同时可以详细的显示出 CD 的信息，包括曲名，作者等等，最新支持   
windows2000。   
软件下载：  cdbar.zip   
  
使用工具:W32dasm   
破解难易度:易   
  
该软件无注册码输入画面,只有 information 菜单显示有关注册步骤及软件剩余使用天数.   
因此,使用 W32dasm,反编译后很容易地找到了线索   
\* Possible StringData Ref from Data Obj ->"Built on %s"<---在 information 菜单中显示的内容   
                                  |   
:0040D3EC 68704B4400              push 00444B70   
:0040D3F1 50                      push eax   
:0040D3F2 E85F850100              call 00425956   
:0040D3F7 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:0040D3FB 83C40C                  add esp, 0000000C   
:0040D3FE 51                      push ecx   
:0040D3FF 8D8E4C010000            lea ecx, dword ptr \[esi+0000014C\]   
:0040D405 E8AED00100              call 0042A4B8   
:0040D40A A088704400              mov al, byte ptr \[00447088\]   
:0040D40F 6A00                    push 00000000   
:0040D411 84C0                    test al, al   
:0040D413 743E                    je 0040D453<---未经授权的转 40D453,将次 743E 改为 9090(即 2 个 NOP)   
:0040D415 8D8E98000000            lea ecx, dword ptr \[esi+00000098\]   
:0040D41B E899D10100              call 0042A5B9   
:0040D420 6A00                    push 00000000   
:0040D422 8D8E10010000            lea ecx, dword ptr \[esi+00000110\]   
:0040D428 E88CD10100              call 0042A5B9   
:0040D42D 8B1568704400            mov edx, dword ptr \[00447068\]   
:0040D433 8D442404                lea eax, dword ptr \[esp+04\]   
:0040D437 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"Registered to: %s"<----显示授权信息   
                                  |   
:0040D438 685C4B4400              push 00444B5C   
:0040D43D 50                      push eax   
:0040D43E E813850100              call 00425956   
:0040D443 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:0040D447 83C40C                  add esp, 0000000C   
:0040D44A 51                      push ecx   
:0040D44B 8D8ED4000000            lea ecx, dword ptr \[esi+000000D4\]   
:0040D451 EB64                    jmp 0040D4B7   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040D413(C)   
|   
:0040D453 E868AA0000              call 00417EC0   
:0040D458 8B0D84704400            mov ecx, dword ptr \[00447084\]   
:0040D45E 83C404                  add esp, 00000004   
:0040D461 2BC1                    sub eax, ecx   
:0040D463 8BC8                    mov ecx, eax   
:0040D465 B807452EC2              mov eax, C22E4507   
:0040D46A F7E1                    mul ecx   
:0040D46C C1EA10                  shr edx, 10   
:0040D46F B80F000000              mov eax, 0000000F   
:0040D474 2BC2                    sub eax, edx   
:0040D476 83F801                  cmp eax, 00000001   
:0040D479 750D                    jne 0040D488   
:0040D47B 50                      push eax   
:0040D47C 8D542408                lea edx, dword ptr \[esp+08\]   
  
\* Possible StringData Ref from Data Obj ->"Expires in %ld day"   
                                  |   
:0040D480 68484B4400              push 00444B48   
:0040D485 52                      push edx   
:0040D486 EB1C                    jmp 0040D4A4   
上面的这段只是显示对程序判断是否过期无影响,判断在下面   
\* Possible StringData Ref from Data Obj ->"%s\\"   
                                  |   
:0040EFE6 68984E4400              push 00444E98   
:0040EFEB 6874704400              push 00447074   
:0040EFF0 E861690100              call 00425956   
:0040EFF5 83C40C                  add esp, 0000000C   
:0040EFF8 E8F3F8FFFF              call 0040E8F0   
:0040EFFD 84C0                    test al, al   
:0040EFFF 752F                    jne 0040F030 <--未过期转 40F030   
:0040F001 E8AAFBFFFF              call 0040EBB0   
:0040F006 84C0                    test al, al   
:0040F008 7426                    je 0040F030 <--未过期转 40F030,在上面转和在此转好象没有区别   
:0040F00A 6A00                    push 00000000   
:0040F00C 6A00                    push 00000000   
  
\* Possible StringData Ref from Data Obj ->"Your copy of CDBar has expired. "   
                                        ->" Please visit " <---过期提示   
                                  |   
:0040F00E 68184E4400              push 00444E18   
:0040F013 E826FD0100              call 0042ED3E   
:0040F018 33C0                    xor eax, eax   
:0040F01A 5D                      pop ebp   
:0040F01B 8B8C2464020000          mov ecx, dword ptr \[esp+00000264\]   
:0040F022 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:0040F029 81C470020000            add esp, 00000270   
:0040F02F C3                      ret   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040EFFF(C), :0040F008(C)   
|   
:0040F030 56                      push esi   
  
到此,软件 15 天后仍可正常使用,由于本人水平有限,在 information 菜单显示画面中 register to 一项仍   
是空白,不知如何处理,