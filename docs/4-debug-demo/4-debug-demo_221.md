

# ACD Systems

> 原文：[`www.pediy.com/kssd/pediy01/forum015.htm`](https://www.pediy.com/kssd/pediy01/forum015.htm)

昨晚继 ACDSee 3.0 Build 1209, Picaview 1.32 之后,我又破解了 ACD Systems 出品的第三个软件: Imagefox 1.21. 在我完成它的破解之后,我发现了一个破解 ACD Systems 的 Trial Version 软件的通用方法,贴出来供大家参考.   
  
1\. 这些软件都用 Aspack 加壳,先脱壳,可用 unaspack 或手动脱壳   
2\. 用 Hex Editor 找 33C0C3(也就是 xor eax,eax retn),在其前后有好多个 90(也就是 nop),在 W32dasm 中可以看到这段代码由 5-6 处调用,将 eax 赋值为 1,便去除了软件开始和运行中可能弹出的 Nag   
3\. 程序判断过期的代码是一样的   
  
:100010E0 83F8FB                  cmp eax, FFFFFFFB <- eax=-5 是过期标志   
:100010E3 7516                    jne 100010FB <-这里一定要跳走   
:100010E5 E846150000              call 10002630   
:100010EA 5F                      pop edi   
:100010EB 5E                      pop esi   
:100010EC B811000000              mov eax, 00000011   
:100010F1 5B                      pop ebx   
:100010F2 81C400020000            add esp, 00000200   
:100010F8 C21800                  ret 0018   
  
程序还可能接着有第二个检查,   
  
:100010FB 85C0                    test eax, eax (或 esi) <-eax 应该=0   
:100010FD 0F8530070000            jne 10001833   
  
有兴趣的朋友可以从 www.acdsystems.com 再下载些其它的软件,试试我的办法,看是否可行. 不过,此招不灵时可别怪我. :)