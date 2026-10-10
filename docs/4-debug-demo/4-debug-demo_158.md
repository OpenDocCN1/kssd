

# Hot Corners 1.92

> 原文：[`www.pediy.com/kssd/pediy05/pediy50301.htm`](https://www.pediy.com/kssd/pediy05/pediy50301.htm)

又到 http://www.southbaypc.com/ 拖了一个垃圾下来   
软件叫 Hot Corners 1.92    
目的：自己弹出注册码   
  
  
  
  
:00403BD8 8B842414010000          mov eax, dword ptr \[esp+00000114\]   
:00403BDF 8D54240C                lea edx, dword ptr \[esp+0C\]   
:00403BE3 52                      push edx\==========>正确注册码   
:00403BE4 50                      push eax\==========》输入的注册码   
:00403BE5 E886FFFFFF              call 00403B70   
:00403BEA 83C410                  add esp, 00000010   
:00403BED 85C0                    test eax, eax   
以上 ret 后到下面：   
:004037A3 50                      push eax   
:004037A4 51                      push ecx   
:004037A5 E816040000              call 00403BC0   
:004037AA 83C408                  add esp, 00000008   
:004037AD 85C0                    test eax, eax   
:004037AF 5F                      pop edi   
:004037B0 7443                    je 004037F5---->这里跳到注册码错误的地方   
  
  
:004037F5 6A00                    push 00000000   
  
\* Possible StringData Ref from Data Obj ->"Hot Corners"   
                                  |   
:004037F7 687CB04000              push 0040B07C------------>改成 push edx   
  
\* Possible StringData Ref from Data Obj ->"Sorry, you have entered an incorrect "   
                                        ->"registration code."   
                                  |   
:004037FC 6808B34000              push 0040B308------------>改成 push edx   
:00403801 56                      push esi   
  
\* Reference To: USER32.MessageBoxA, Ord:01DCh   
                                  |   
:00403802 FF15DC914000            Call dword ptr \[004091DC\]   
  
好了 注册码又出来了，又一篇垃圾产生了   
  
                        PLL621  4dd466d0e1e0cc123afbe504da2462ca