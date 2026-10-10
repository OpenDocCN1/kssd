

# LogoManager 1.18

> 原文：[`www.pediy.com/kssd/pediy03/forum642.htm`](https://www.pediy.com/kssd/pediy03/forum642.htm)

软件：LogoManager 1.18   
    http://homepage.dtn.ntl.com/wmdb/lm118.exe (460KB)   
简介：Nokia 手机画面编辑软件，有 NAG 窗口，不注册则延时 5 秒进入。   
破解：   
（1）   
用 File Scaner 查出它用 Upx 压缩，则用 ProcDump->Unpack 脱壳成功。   
（2）   
用 Trw2000 Load，bpx messageboxa   
\* Reference To: USER32.::D, Ord:0000h   
                                  |   
:00408FB9 FF1520644300            Call dword ptr \[00436420\]   
:00408FBF 85C0                    test eax, eax   
:00408FC1 7512                    jne 00408FD5   
:00408FC3 6898714400              push 00447198   
:00408FC8 FFD5                    call ebp   
:00408FCA 6898714400              push 00447198   
  
\* Reference To: USER32.::D, Ord:0023h   
                                  |   
:00408FCF FF1524644300            Call dword ptr \[00436424\]    ；该行显示 NAG   
但修改 00408FCF：以上语句没用。Stop here!   
（3）   
换一种方法，从 NAG 窗口显示以后入手。   
运行 Logomanager.exe，出现 NAG 窗口后,   
：hwnd   
找到 NAG 窗口"Register Later"按钮的句柄 07D0   
：bmsg 07D0 wm\_destroy   
F5   
点"Register Later"按钮后中断，终于找到关键的地址：   
:0040BDFF 6A00                    push 00000000   
:0040BE01 744F                    je 0040BE52        ；改成 JNE   
:0040BE03 8B4624                  mov eax, dword ptr \[esi+24\]   
:0040BE06 8B0D30754400            mov ecx, dword ptr \[00447530\]   
:0040BE0C 68E09B4000              push 00409BE0   
:0040BE11 50                      push eax   
  
\* Possible Reference to Dialog: DialogID\_00A1   
                                  |   
:0040BE12 68A1000000              push 000000A1   
:0040BE17 51                      push ecx   
:0040BE18 E8F3A00100              call 00425F10        ；该行显示 NAG   
:0040BE1D 83C418                  add esp, 00000018   
:0040BE20 48                      dec eax   
  
将:0040BE01 改成 JNE 后，NAG 窗口消失！OK!   
最后，用 UltraEdit Search 'Unregistered' 改为'Registered'。   
  
hurrah@china.com   
hurrah.go.163.com  4dd466d0e1e0cc123afbe504da2462ca