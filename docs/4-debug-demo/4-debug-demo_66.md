

# InsTalk

> 原文：[`www.pediy.com/kssd/pediy05/pediy50078.htm`](https://www.pediy.com/kssd/pediy05/pediy50078.htm)

即时语音提示 & 校对软件 InsTalk 注册码及注册机-初学者请看   
  
软件说明：即时语音提示&校对软件 InsTalk 是面向 Windows 9x/NT 的工具软件。利用它用户可以让电脑说汉语普通话。它有两种工作状态。一种是在使用键盘输入数字和英文字符时，可以跟随录入的字符即时发出相应的语音提示。另一种是让电脑朗读中文。   
  
使用工具：TRW2000、PW32Dasm9b.EXE、KeyMaker。   
  
由于受软件提示信息的影响，整个破解过程走了不少弯路：输入注册信息后，软件提示关闭并重新启动软件以验证注册码，但重新启动时，试了很多断点还是找不到注册码（其实可以找到，但此注册码并非你所输入的注册名和单位生成，而是注册名和单位为空时的注册码），后来通过对注册表的监视，当软件注册码错误时，程序根本就不往注册表里写入。所以，判断注册码的工作应该在输入注册信息的时候就已经进行了，作者给我们开了个大玩笑?!!   
  
\==========================================================================================   
1、启动程序，填写注册信息，Ctrl-n，bpx hmemcpy，F5 返回，按“注册”按钮，程序拦下。   
2、bc \*，pmodule。   
3、按两次 F10，来到下面：   
:004073D0 8D8C2444010000          lea ecx, dword ptr \[esp+00000144\]   
:004073D7 6800010000              push 00000100   
:004073DC 51                      push ecx   
:004073DD 680D040000              push 0000040D   
:004073E2 8BCE                    mov ecx, esi   
:004073E4 8944243C                mov dword ptr \[esp+3C\], eax   
:004073E8 E8D4540100              call 0041C8C1   
:004073ED 8BC8                    mov ecx, eax   
:004073EF E815560100              call 0041CA09   
:004073F4 8D942444020000          lea edx, dword ptr \[esp+00000244\]   
:004073FB 6800010000              push 00000100   
:00407400 52                      push edx   
:00407401 680F040000              push 0000040F   
:00407406 8BCE                    mov ecx, esi   
:00407408 89442440                mov dword ptr \[esp+40\], eax   
:0040740C E8B0540100              call 0041C8C1   
:00407411 8BC8                    mov ecx, eax   
:00407413 E8F1550100              call 0041CA09   
:00407418 89442438                mov dword ptr \[esp+38\], eax   
:0040741C E86FB20100              call 00422690   
:00407421 8B6804                  mov ebp, dword ptr \[eax+04\]   
:00407424 A158E74200              mov eax, dword ptr \[0042E758\]   
:00407429 8D8C2444010000          lea ecx, dword ptr \[esp+00000144\]   
:00407430 896C241C                mov dword ptr \[esp+1C\], ebp   
:00407434 51                      push ecx   
:00407435 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:00407439 8944241C                mov dword ptr \[esp+1C\], eax   
:0040743D E8F55F0100              call 0041D437   
:00407442 6A19                    push 00000019   
:00407444 51                      push ecx   
:00407445 8D94244C020000          lea edx, dword ptr \[esp+0000024C\]   
:0040744C 33DB                    xor ebx, ebx   
:0040744E 8BCC                    mov ecx, esp   
:00407450 89642444                mov dword ptr \[esp+44\], esp   
:00407454 52                      push edx   
:00407455 899C2458030000          mov dword ptr \[esp+00000358\], ebx   
:0040745C E84FD7FFFF              call 00404BB0   
:00407461 8D44244C                lea eax, dword ptr \[esp+4C\]   
:00407465 8D4C2434                lea ecx, dword ptr \[esp+34\]   
:00407469 50                      push eax   
:0040746A C684245803000001        mov byte ptr \[esp+00000358\], 01   
:00407472 E839D7FFFF              call 00404BB0   
:00407477 51                      push ecx   
:00407478 8D542424                lea edx, dword ptr \[esp+24\]   
:0040747C 8BCC                    mov ecx, esp   
:0040747E 8964244C                mov dword ptr \[esp+4C\], esp   
:00407482 52                      push edx   
:00407483 50                      push eax   
:00407484 51                      push ecx   
:00407485 C684246403000002        mov byte ptr \[esp+00000364\], 02   
:0040748D E80A600100              call 0041D49C   
:00407492 C684245803000003        mov byte ptr \[esp+00000358\], 03   
:0040749A E821D6FFFF              call 00404AC0（此处改变 eax 的值，说明对注册码进行了判断）   
:0040749F 83C40C                  add esp, 0000000C   
:004074A2 8D4C242C                lea ecx, dword ptr \[esp+2C\]   
:004074A6 8BF8                    mov edi, eax（这里将 eax 的值赋予 edi）   
:004074A8 889C244C030000          mov byte ptr \[esp+0000034C\], bl   
:004074AF E88A5E0100              call 0041D33E   
:004074B4 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:004074B8 C784244C030000FFFFFFFF  mov dword ptr \[esp+0000034C\], FFFFFFFF   
:004074C3 E8765E0100              call 0041D33E   
:004074C8 3BFB                    cmp edi, ebx   
:004074CA 0F849C000000            je 0040756C（此处若不跳，则可将错误的注册信息强制写入注册表）   
:004074D0 8D742444                lea esi, dword ptr \[esp+44\]   
:004074D4 8D6C2430                lea ebp, dword ptr \[esp+30\]   
  
\==========================================================================================   
在:0040749A E821D6FFFF              call 00404AC0 处按 F8 进入：   
  
:00404AC0 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:00404AC6 6AFF                    push FFFFFFFF   
:00404AC8 6800414200              push 00424100   
:00404ACD 50                      push eax   
:00404ACE 64892500000000          mov dword ptr fs:\[00000000\], esp   
:00404AD5 53                      push ebx   
:00404AD6 56                      push esi   
:00404AD7 8B442420                mov eax, dword ptr \[esp+20\]   
:00404ADB 8D542418                lea edx, dword ptr \[esp+18\]   
:00404ADF 50                      push eax   
:00404AE0 51                      push ecx   
:00404AE1 8BCC                    mov ecx, esp   
:00404AE3 89642428                mov dword ptr \[esp+28\], esp   
:00404AE7 52                      push edx   
:00404AE8 C744241C01000000        mov \[esp+1C\], 00000001   
:00404AF0 E8BE850100              call 0041D0B3   
:00404AF5 8D442428                lea eax, dword ptr \[esp+28\]   
:00404AF9 50                      push eax   
:00404AFA E801260000              call 00407100（算注册码）   
:00404AFF 8B742428                mov esi, dword ptr \[esp+28\]（将错误的注册码赋予 esi）   
:00404B03 8B00                    mov eax, dword ptr \[eax\]（将正确的注册码赋予 eax）   
:00404B05 83C40C                  add esp, 0000000C（在此处 d eax 看到真正的注册码）   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404B2A(C)   
|   
:00404B08 8A10                    mov dl, byte ptr \[eax\]（取真码第一位）   
:00404B0A 8A1E                    mov bl, byte ptr \[esi\]（取假码第一位）   
:00404B0C 8ACA                    mov cl, dl（将真码第一位赋予 cl）   
:00404B0E 3AD3                    cmp dl, bl（比较两值是否相同）   
:00404B10 751E                    jne 00404B30（不同就跳到 00404B30，比较失败）   
:00404B12 84C9                    test cl, cl（测试 cl 是否为空，即判断是否已全部比较完）   
:00404B14 7416                    je 00404B2C（如果比较完毕，则跳到 00404B2C）   
:00404B16 8A5001                  mov dl, byte ptr \[eax+01\]（取真码下一位）   
:00404B19 8A5E01                  mov bl, byte ptr \[esi+01\]（取假码下一位）   
:00404B1C 8ACA                    mov cl, dl   
:00404B1E 3AD3                    cmp dl, bl   
:00404B20 750E                    jne 00404B30（不同就跳到 00404B30，比较失败）   
:00404B22 83C002                  add eax, 00000002（去掉真码前两位，为下一轮比较做准备）   
:00404B25 83C602                  add esi, 00000002（去掉假码前两位，为下一轮比较做准备）   
:00404B28 84C9                    test cl, cl（测试 cl 是否为空，即判断是否已全部比较完）   
:00404B2A 75DC                    jne 00404B08（返回 00404B08 继续比较）   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404B14(C)   
|   
:00404B2C 33C0                    xor eax, eax（注册码正确时，跳到此行）   
:00404B2E EB05                    jmp 00404B35   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00404B10(C), :00404B20(C)   
|   
:00404B30 1BC0                    sbb eax, eax（注册码错误时，跳到此行）   
:00404B32 83D8FF                  sbb eax, FFFFFFFF   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404B2E(U)   
|   
:00404B35 85C0                    test eax, eax   
:00404B37 0F94C0                  sete al   
:00404B3A 25FF000000              and eax, 000000FF   
:00404B3F 8D4C2420                lea ecx, dword ptr \[esp+20\]   
:00404B43 8BF0                    mov esi, eax（将 eax 的值赋予 esi）   
:00404B45 E8F4870100              call 0041D33E   
:00404B4A 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:00404B4E C644241000              mov \[esp+10\], 00   
:00404B53 E8E6870100              call 0041D33E   
:00404B58 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:00404B5C C7442410FFFFFFFF        mov \[esp+10\], FFFFFFFF   
:00404B64 E8D5870100              call 0041D33E   
:00404B69 8B4C2408                mov ecx, dword ptr \[esp+08\]   
:00404B6D 8BC6                    mov eax, esi（将 esi 的值赋予 eax）   
:00404B6F 5E                      pop esi   
:00404B70 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:00404B77 5B                      pop ebx   
:00404B78 83C40C                  add esp, 0000000C   
:00404B7B C3                      ret   
  
\==========================================================================================   
4、以下是对程序重新启动后的一些分析：   
程序一开始有个欢迎提示框，提示是共享版还是注册版，可见在此之前已经判断了是否已经注册，所以目的就是找出出现这个提示框的最后一个关键 Call。   
用 trw2000 载入 InsTalk.exe，结合 F10、F9、F6 键就可找到这个 Call（具体操作方法可参考我写的 Acdsee4.0 的破解，在看雪论坛以我的注册名 esoft2001.51.net 搜索就能找到）   
:0041F76B 8B06                    mov eax, dword ptr \[esi\]   
:0041F76D 8BCE                    mov ecx, esi   
:0041F76F FF5050                  call \[eax+50\]（此处是出现提示框，应该快接近核心了。即使判断错也没关系，可以继续再试嘛!）   
:0041F772 85C0                    test eax, eax   
:0041F774 7515                    jne 0041F78B   
\==========================================================================================   
F8 进入上面的 Call，看到下面代码：   
:004046D0 6AFF                    push FFFFFFFF   
:004046D2 68DD404200              push 004240DD   
:004046D7 64A100000000            mov eax, dword ptr fs:\[00000000\]   
  
……………………略去一些代码   
\* Possible Reference to Dialog:    
                                  |   
:004047A7 68D0E04200              push 0042E0D0   
:004047AC 8BCE                    mov ecx, esi   
:004047AE C68424F404000002        mov byte ptr \[esp+000004F4\], 02   
:004047B6 E8DBDA0100              call 00422296   
:004047BB 8D4C2424                lea ecx, dword ptr \[esp+24\]   
:004047BF C68424E404000001        mov byte ptr \[esp+000004E4\], 01   
:004047C7 E8728B0100              call 0041D33E   
:004047CC 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:004047D0 C68424E404000000        mov byte ptr \[esp+000004E4\], 00   
:004047D8 E8618B0100              call 0041D33E   
:004047DD 8B5500                  mov edx, dword ptr \[ebp+00\]   
:004047E0 42                      inc edx   
:004047E1 52                      push edx   
:004047E2 E8BC590000              call 0040A1A3   
:004047E7 8BF8                    mov edi, eax   
:004047E9 8B442414                mov eax, dword ptr \[esp+14\]   
:004047ED 83C404                  add esp, 00000004   
:004047F0 897C2418                mov dword ptr \[esp+18\], edi   
:004047F4 85C0                    test eax, eax（判断是否将注册信息写入注册表，若无则 eax=0）   
:004047F6 897C9C2C                mov dword ptr \[esp+4\*ebx+2C\], edi   
:004047FA 7428                    je 00404824   
:004047FC 8B4D00                  mov ecx, dword ptr \[ebp+00\]   
:004047FF 8BF0                    mov esi, eax   
:00404801 8BC1                    mov eax, ecx   
:00404803 C1E902                  shr ecx, 02   
:00404806 F3                      repz   
:00404807 A5                      movsd   
:00404808 8BC8                    mov ecx, eax   
:0040480A 83E103                  and ecx, 00000003   
:0040480D F3                      repz   
:0040480E A4                      movsb   
:0040480F 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:00404813 51                      push ecx   
:00404814 E88F880100              call 0041D0A8   
:00404819 8B7C241C                mov edi, dword ptr \[esp+1C\]   
:0040481D 8B742424                mov esi, dword ptr \[esp+24\]   
:00404821 83C404                  add esp, 00000004   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004047FA(C)   
|   
:00404824 8B5500                  mov edx, dword ptr \[ebp+00\]   
:00404827 6A00                    push 00000000   
:00404829 52                      push edx   
:0040482A 57                      push edi   
:0040482B E8802A0000              call 004072B0   
:00404830 8B4500                  mov eax, dword ptr \[ebp+00\]   
:00404833 83C40C                  add esp, 0000000C   
:00404836 43                      inc ebx   
:00404837 83FB03                  cmp ebx, 00000003   
:0040483A C6043800                mov byte ptr \[eax+edi\], 00   
:0040483E 0F8C2FFFFFFF            jl 00404773   
:00404844 8B0D58E74200            mov ecx, dword ptr \[0042E758\]   
:0040484A 8B542430                mov edx, dword ptr \[esp+30\]   
:0040484E 894C2410                mov dword ptr \[esp+10\], ecx   
:00404852 52                      push edx   
:00404853 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00404857 E8DB8B0100              call 0041D437   
:0040485C 8B442434                mov eax, dword ptr \[esp+34\]   
:00404860 6A19                    push 00000019   
:00404862 51                      push ecx   
:00404863 C68424EC04000003        mov byte ptr \[esp+000004EC\], 03   
:0040486B 8BCC                    mov ecx, esp   
:0040486D 89642424                mov dword ptr \[esp+24\], esp   
:00404871 50                      push eax   
:00404872 E839030000              call 00404BB0   
:00404877 8B4C2434                mov ecx, dword ptr \[esp+34\]   
:0040487B C68424EC04000004        mov byte ptr \[esp+000004EC\], 04   
:00404883 51                      push ecx   
:00404884 8D4C2424                lea ecx, dword ptr \[esp+24\]   
:00404888 E823030000              call 00404BB0   
:0040488D 51                      push ecx   
:0040488E 8D4C241C                lea ecx, dword ptr \[esp+1C\]   
:00404892 8BD4                    mov edx, esp   
:00404894 89642430                mov dword ptr \[esp+30\], esp   
:00404898 51                      push ecx   
:00404899 50                      push eax   
:0040489A 52                      push edx   
:0040489B C68424FC04000005        mov byte ptr \[esp+000004FC\], 05   
:004048A3 E8F48B0100              call 0041D49C   
:004048A8 C68424F004000006        mov byte ptr \[esp+000004F0\], 06   
:004048B0 E80B020000              call 00404AC0（这里又调用 00404AC0 判断注册码，详细代码见上面）   
:004048B5 83C40C                  add esp, 0000000C（经过分析可以知道，如果注册码正确，此出返回 eax 的值应该是 1，如果错误则返回 0）   
:004048B8 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:004048BC A3EC254300              mov dword ptr \[004325EC\], eax   
:004048C1 C68424E404000003        mov byte ptr \[esp+000004E4\], 03   
:004048C9 E8708A0100              call 0041D33E   
:004048CE 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004048D2 C68424E404000000        mov byte ptr \[esp+000004E4\], 00   
:004048DA E85F8A0100              call 0041D33E   
:004048DF 8D7C242C                lea edi, dword ptr \[esp+2C\]   
:004048E3 BB03000000              mov ebx, 00000003   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004048F7(C)   
|   
:004048E8 8B17                    mov edx, dword ptr \[edi\]   
:004048EA 52                      push edx   
:004048EB E8CA570000              call 0040A0BA   
:004048F0 83C404                  add esp, 00000004   
:004048F3 83C704                  add edi, 00000004   
:004048F6 4B                      dec ebx   
:004048F7 75EF                    jne 004048E8   
:004048F9 E822F5FFFF              call 00403E20   
:004048FE 8BCE                    mov ecx, esi   
:00404900 E85BF8FFFF              call 00404160   
:00404905 A1EC254300              mov eax, dword ptr \[004325EC\]   
:0040490A 85C0                    test eax, eax   
:0040490C 0F8588000000            jne 0040499A   
:00404912 8BCE                    mov ecx, esi   
:00404914 E8C7FCFFFF              call 004045E0（判断是否已过试用期）   
:00404919 85C0                    test eax, eax（未过，eax=0；已过，eax=1）   
:0040491B 747D                    je 0040499A   
  
\* Possible Reference to String Resource ID=00112: ",o?q 玱?▌?蜥   
倻?珥(,o?〝?\\鑼   
  
?1 鑼"   
                                  |   
:0040491D 6A70                    push 00000070   
:0040491F 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:00404923 E8FD8C0100              call 0041D625   
:00404928 A158E74200              mov eax, dword ptr \[0042E758\]   
:0040492D 89442410                mov dword ptr \[esp+10\], eax   
  
\* Possible Reference to String Resource ID=00125: "鑼"   
                                  |   
:00404931 6A7D                    push 0000007D   
:00404933 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00404937 C68424E804000007        mov byte ptr \[esp+000004E8\], 07   
:0040493F E8E18C0100              call 0041D625   
:00404944 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:00404948 8B542414                mov edx, dword ptr \[esp+14\]   
:0040494C 6A34                    push 00000034   
:0040494E 51                      push ecx   
:0040494F 52                      push edx   
:00404950 6A00                    push 00000000   
  
\* Reference To: USER32.MessageBoxA, Ord:01BEh   
                                  |   
:00404952 FF1544544200            Call dword ptr \[00425444\]（过期提示）   
:00404958 83F806                  cmp eax, 00000006   
:0040495B 7511                    jne 0040496E   
:0040495D B940234300              mov ecx, 00432340   
:00404962 C7461C40234300          mov \[esi+1C\], 00432340   
:00404969 E8E04F0100              call 0041994E   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040495B(C)   
|   
:0040496E 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00404972 C68424E404000000        mov byte ptr \[esp+000004E4\], 00   
:0040497A E8BF890100              call 0041D33E   
:0040497F 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00404983 C78424E4040000FFFFFFFF  mov dword ptr \[esp+000004E4\], FFFFFFFF   
:0040498E E8AB890100              call 0041D33E   
:00404993 33C0                    xor eax, eax   
:00404995 E90A010000              jmp 00404AA4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0040490C(C), :0040491B(C)   
|   
:0040499A 8D442444                lea eax, dword ptr \[esp+44\]   
:0040499E C744244494000000        mov \[esp+44\], 00000094   
:004049A6 50                      push eax   
  
\* Reference To: KERNEL32.GetVersionExA, Ord:0175h   
                                  |   
:004049A7 FF1598514200            Call dword ptr \[00425198\]   
:004049AD 8B6C2454                mov ebp, dword ptr \[esp+54\]   
:004049B1 33C9                    xor ecx, ecx   
:004049B3 83FD02                  cmp ebp, 00000002   
  
\* Possible Reference to String Resource ID=00114: "InsTalk ?b"   
                                  |   
:004049B6 6A72                    push 00000072   
:004049B8 0F94C1                  sete cl   
:004049BB 890D0C1F4300            mov dword ptr \[00431F0C\], ecx   
:004049C1 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:004049C5 E85B8C0100              call 0041D625   
:004049CA 8B542414                mov edx, dword ptr \[esp+14\]   
:004049CE 52                      push edx   
:004049CF 6A00                    push 00000000   
  
\* Reference To: USER32.FindWindowA, Ord:00D5h   
                                  |   
:004049D1 FF1548544200            Call dword ptr \[00425448\]   
:004049D7 85C0                    test eax, eax   
:004049D9 741B                    je 004049F6   
:004049DB 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:004049DF C78424E4040000FFFFFFFF  mov dword ptr \[esp+000004E4\], FFFFFFFF   
:004049EA E84F890100              call 0041D33E   
:004049EF 33C0                    xor eax, eax   
:004049F1 E9AE000000              jmp 00404AA4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004049D9(C)   
|   
:004049F6 E895DC0100              call 00422690   
:004049FB 8B400C                  mov eax, dword ptr \[eax+0C\]   
:004049FE 6891000000              push 00000091   
:00404A03 50                      push eax   
  
\* Reference To: USER32.LoadCursorA, Ord:019Ah   
                                  |   
:00404A04 FF15C0544200            Call dword ptr \[004254C0\]   
:00404A0A 68681C4300              push 00431C68   
:00404A0F A330204300              mov dword ptr \[00432030\], eax   
:00404A14 E8C7F2FFFF              call 00403CE0   
:00404A19 83C404                  add esp, 00000004   
:00404A1C 85C0                    test eax, eax   
:00404A1E 7556                    jne 00404A76   
:00404A20 A18C1E4300              mov eax, dword ptr \[00431E8C\]   
:00404A25 85C0                    test eax, eax   
:00404A27 7534                    jne 00404A5D   
  
\* Possible Reference to String Resource ID=00115: "~   
0 眢搰?   
鱪ろ髶圅,o?"   
                                  |   
:00404A29 6A73                    push 00000073   
:00404A2B 8D4C2418                lea ecx, dword ptr \[esp+18\]   
:00404A2F E8F18B0100              call 0041D625   
:00404A34 8B442414                mov eax, dword ptr \[esp+14\]   
:00404A38 6A30                    push 00000030   
:00404A3A 6A00                    push 00000000   
:00404A3C 50                      push eax   
:00404A3D 6A00                    push 00000000   
  
\* Reference To: USER32.MessageBoxA, Ord:01BEh   
                                  |   
:00404A3F FF1544544200            Call dword ptr \[00425444\]   
:00404A45 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00404A49 C78424E4040000FFFFFFFF  mov dword ptr \[esp+000004E4\], FFFFFFFF   
:00404A54 E8E5880100              call 0041D33E   
:00404A59 33C0                    xor eax, eax   
:00404A5B EB47                    jmp 00404AA4   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404A27(C)   
|   
:00404A5D 8B15841E4300            mov edx, dword ptr \[00431E84\]   
:00404A63 B986000000              mov ecx, 00000086   
  
\* Possible Reference to Dialog:    
                                  |   
:00404A68 BF681C4300              mov edi, 00431C68   
:00404A6D 8B7208                  mov esi, dword ptr \[edx+08\]   
:00404A70 F3                      repz   
:00404A71 A5                      movsd   
:00404A72 8B742420                mov esi, dword ptr \[esp+20\]   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404A1E(C)   
|   
:00404A76 6A00                    push 00000000   
  
\* Possible Reference to String Resource ID=00102: "A?仨笮:"   
                                  |   
:00404A78 6A66                    push 00000066   
:00404A7A B9C0244300              mov ecx, 004324C0   
:00404A7F C7461CC0244300          mov \[esi+1C\], 004324C0   
:00404A86 E8424B0100              call 004195CD（这里出现欢迎提示框）   
:00404A8B 8D4C2414                lea ecx, dword ptr \[esp+14\]   
:00404A8F C78424E4040000FFFFFFFF  mov dword ptr \[esp+000004E4\], FFFFFFFF   
:00404A9A E89F880100              call 0041D33E   
:00404A9F B801000000              mov eax, 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00404995(U), :004049F1(U), :00404A5B(U)   
|   
:00404AA4 8B8C24DC040000          mov ecx, dword ptr \[esp+000004DC\]   
:00404AAB 5F                      pop edi   
:00404AAC 5E                      pop esi   
:00404AAD 5D                      pop ebp   
:00404AAE 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:00404AB5 5B                      pop ebx   
:00404AB6 81C4D8040000            add esp, 000004D8   
:00404ABC C3                      ret   
  
其实通过分析我们已经可以看出，整个程序不论是注册时还是重新启动时都是在 call 00404AC0 里面进行核心的注册码判断，我们只要修改其中的一处就可变成注册版。并且从程序分析可知，全部比较完毕并且注册码正确时将跳到 00404B2C，所以修改方法如下：   
:00404B0E 3AD3                    cmp dl, bl（这里改成 cmp dl，dl 即 3AD3 改为 3AD2，自己比自己当然是一样的）   
:00404B10 751E                    jne 00404B30（这里我们改成 je 00404B2C，即将 751E 改成 741A）   
用二进制编辑工具打开 InsTalk.exe 文件，   
查找：83C40C8A108A1E8ACA3AD3751E84C9   
修改：------------------3AD2741A----   
  
\==========================================================================================   
5、该软件的注册信息保存在注册表的以下位置：   
HKEY\_CURRENT\_USER\\Software\\Happy Studio\\INSTALK\\Registration   
HKEY\_USERS\\.DEFAULT\\Software\\Happy Studio\\INSTALK\\Registration   
两个键下的四个二进制值（已加密变换）下：   
Date:软件第一次运行的时间；试用期过后删除该二进制值可继续使用。   
Date0:注册用户名称；   
Date1:注册用户单位；   
Date2:注册码。   
用户注册后，删除 Date0、Date1、Date2 三个二进制值可重新注册。   
  
\==========================================================================================   
6、编写注册机   
使用“注册机编写器（Keymaker）”之“另类注册机”功能   
(1)程序名称：InsTalk.exe   
(2)添加数据：   
　　中断地址：40749A   
    中断次数：1   
　　第一字节：E8   
　　指令长度：5   
  
    中断地址：404B05   
    中断次数：1   
　　第一字节：83   
　　指令长度：3   
  
(3)选择内存方式 EAX。   
  
\=======================================THE END=============================================   
  
这个教程我作了一些注释，初学破解的朋友可以看一看，其中不对之处也请各位多多指教!   
  
esoft2001.51.net   
2002 年 4 月 11 日  4dd466d0e1e0cc123afbe504da2462ca