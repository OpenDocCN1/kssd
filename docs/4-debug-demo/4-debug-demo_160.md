

# Registry Crawler 4.0.0.3 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50352.htm`](https://www.pediy.com/kssd/pediy05/pediy50352.htm)

Registry Crawler 4.0.0.3   
下载：汉化补丁: http://www.hanzify.org/skipli/chinese/HAC-RCrawler4003-Fire.zip   
破解工具:TRW2000 HIEW W32DASM RH ULTRAEDIT   
我今天从硬盘翻出一个软件 Registry Crawler，发现没有注册，但是算法又比较复杂，要跟比较长时间这个软件又是明码比较，所以我就用 SetWindowTextA 显示出来，这样直接输入注册名按解锁软件自动显示注册码在，注册码的 TEXT 里面，看起来感觉瞒不错的。又用 ULTRAEDIT 修改了一下出错以后显示的对话框，哇，赫然一个注册机。注册成功想重新注册请删掉 HKEY\_LOCAL\_MACHINE\\Software\\4Developers\\RCrawler\\4D 这个键值   
就可以重新注册了~   
  
:0040AEDE 6800714400              push 00447100   
:0040AEE3 E95AC20300              jmp 00447142    (原本是 LEA ECX，DWORD PTR \[ESI+00000380\]）   
:0040AEE8 90                      nop            我把把跳转到底下保存句柄的内存的存储地址，以便等下显示使用。   
:0040AEE9 E8E02A0200              call 0042D9CE  软件所有设置窗口的标题文字都调用这里，所以我们有现成的。   
  
:0042D9CE 8B4138                  mov eax, dword ptr \[ecx+38\]   
:0042D9D1 85C0                    test eax, eax   
:0042D9D3 751D                    jne 0042D9F2   
:0042D9D5 FF742404                push \[esp+04\]   
:0042D9D9 FF711C                  push \[ecx+1C\]   
  
\* Reference To: USER32.SetWindowTextA, Ord:025Eh  设置窗口的标题文字或控件的内容的 API   
                                                    
:0042D9DC FF1524864400            Call dword ptr \[00448624\]    
:0042D9E2 EB0E                    jmp 0042D9F2   
:0042D9E4 8B10                    mov edx, dword ptr \[eax\]   
:0042D9E6 FF742404                push \[esp+04\]   
:0042D9EA 8BC8                    mov ecx, eax   
:0042D9EC FF9280000000            call dword ptr \[edx+00000080\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0042D9D3(C), :0042D9E2(U)   
|   
:0042D9F2 C20400                  ret 0004   
  
:0040AE00 8D442414                lea eax, dword ptr \[esp+14\]    把注册码移入 EAX，D EAX 就可以看到正确的注册码   
:0040AE04 E909C30300              jmp 00447112            跳转我的代码就是显示出注册码的   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00447133(U)   
|   
:0040AE09 50                      push eax   
:0040AE0A E801570100              call 00420510      这个 CALL 就是比较注册码的   
  
:00447112 50                      push eax    
:00447113 51                      push ecx   
:00447114 52                      push edx          堆栈保存一定要注意不然很容易非法操作   
:00447115 50                      push eax          EAX 里面是注册码   
:00447116 8B0D708D4500            mov ecx, dword ptr \[00458D70\]   
:0044711C 85C9                    test ecx, ecx      由于注册成功以后每次启动都比较一次注册码所以都要   
                                                    运行我们的代码，所以我测试句柄是否存在，来达到不   
                                                    运行底下那个 CALL，不然如注册成功后运行会出错。   
:0044711E 740A                    je 0044712A   
:00447120 E8A968FEFF              call 0042D9CE      调用这个 CALL 把注册码显示出来   
:00447125 E901000000              jmp 0044712B   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044711E(C)   
|   
:0044712A 58                      pop eax   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00447125(U)   
|   
:0044712B 5A                      pop edx   
:0044712C 59                      pop ecx   
:0044712D 58                      pop eax   
:0044712E 6810E64500              push 0045E610   
:00447133 E9D13CFCFF              jmp 0040AE09   
:00447138 00000000000000000000    BYTE 10 DUP(0)   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AEE3(U)   
|   
:00447142 8D8E80030000            lea ecx, dword ptr \[esi+00000380\]  保存句柄的内存的存储地址   
:00447148 50                      push eax   
:00447149 B8708D4500              mov eax, 00458D70   
:0044714E 8908                    mov dword ptr \[eax\], ecx   
:00447150 58                      pop eax          
:00447151 E9933DFCFF              jmp 0040AEE9   
  
总的感觉就是对软件 DIY，就是堆栈好搞好最重要，其他我不知道。我是初级菜鸟，不敢乱说话。   
但是总的感觉有些软件用这样做注册机还是瞒实用的。  4dd466d0e1e0cc123afbe504da2462ca