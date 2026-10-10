

# SEH

> 原文：[`www.pediy.com/kssd/pediy06/pediy6605.htm`](https://www.pediy.com/kssd/pediy06/pediy6605.htm)

工具：od ；lordpe ，hiew workshop  
作者:QiWeiXue \[BCG\]  
声明：我是一只吃菜的大青虫；高手见笑了;一点小心得与大家分享;  
只是写几个 seh，玩完；没有别的意思；  
加函数,找空隙,偶就不一一叙述了;  
我的记事本没有改之前的 OEP 要保存好  
  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
01006AE0 N>    6A 70             push 70  
01006AE2       68 88180001       push NOTEPAD.01001888  
01006AE7    .  E8 BC010000       call NOTEPAD.01006CA8  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
  
一共写六个异常（1Reading 0 Except；Divide Is zero；Int3 Expect；Int68 Except； Int68 Except ；特权指令 异常(这个 TeQuan ZhiLing 是拼音，偶英文不大好:=(,所以用的拼音；谢谢帮我纠正，thanks)  
第一块空隙：  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
01007D71       68 9F7D0001       push NOTEPAD.01007D9F  
01007D76       64:A1 00000000    mov eax,dword ptr fs:\[0\]  
01007D7C       50                push eax  
01007D7D       64:8925 00000000  mov dword ptr fs:\[0\],esp  
01007D84       BE 00000000       mov esi,0  
01007D89       8B06              mov eax,dword ptr ds:\[esi\]  
01007D8B       6A 00             push 0  
01007D8D       68 E0810001       push NOTEPAD.010081E0                    ;  ASCII "Except"  
01007D92       68 E7810001       push NOTEPAD.010081E7                    ;  ASCII "Except NO Runing"  
01007D97       6A 00             push 0  
01007D99       FF15 19300101     call dword ptr ds:\[<&user32.MessageBoxA>>;  USER32.MessageBoxA  
01007D9F       90                nop  
01007DA0       6A 00             push 0  
01007DA2       68 E0810001       push NOTEPAD.010081E0                    ;  ASCII "Except"  
01007DA7       68 F8810001       push NOTEPAD.010081F8                    ;  ASCII "Reading 0 Except :=("  
01007DAC       6A 00             push 0  
01007DAE       FF15 19300101     call dword ptr ds:\[<&user32.MessageBoxA>>;  USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
01007DB4       90                nop  
01007DB5       68 D27D0001       push NOTEPAD.01007DD2  
01007DBA       64:A1 00000000    mov eax,dword ptr fs:\[0\]  
01007DC0       50                push eax  
01007DC1       64:8925 00000000  mov dword ptr fs:\[0\],esp  
01007DC8       BB 00000000       mov ebx,0  
01007DCD       F7F3              div ebx  
01007DCF     ^ EB BA             jmp short NOTEPAD.01007D8B  
01007DD1       90                nop  
01007DD2       90                nop  
01007DD3       6A 00             push 0  
01007DD5       68 E0810001       push NOTEPAD.010081E0                    ;  ASCII "Except"  
01007DDA       68 0D820001       push NOTEPAD.0100820D                    ;  ASCII "Divide Is zero :=("  
01007DDF       6A 00             push 0  
01007DE1       FF15 19300101     call dword ptr ds:\[<&user32.MessageBoxA>>;  USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
01007DE7       90                nop  
01007DE8       68 28310101       push NOTEPAD.01013128  
01007DED       64:A1 00000000    mov eax,dword ptr fs:\[0\]  
01007DF3       50                push eax  
01007DF4       64:8925 00000000  mov dword ptr fs:\[0\],esp  
01007DFB     - E9 20B30000       jmp NOTEPAD.01013120  
第二块空隙：  
01013120     90                  nop  
01013121     CD 03               int 3  
01013123   - E9 634CFFFF         jmp NOTEPAD.01007D8B  
01013128     6A 00               push 0  
0101312A     68 E0810001         push NOTEPAD.010081E0                    ; ASCII "Except"  
0101312F     68 34820001         push NOTEPAD.01008234                    ; ASCII "Have Int3 Expect :=("  
01013134     6A 00               push 0  
01013136     FF15 19300101       call dword ptr ds:\[<&user32.MessageBoxA>>; USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
0101313C     90                  nop  
0101313D     68 59310101         push NOTEPAD.01013159  
01013142     64:A1 00000000      mov eax,dword ptr fs:\[0\]  
01013148     50                  push eax  
01013149     64:8925 00000000    mov dword ptr fs:\[0\],esp  
01013150     90                  nop  
01013151     CD 68               int 68  
01013153     90                  nop  
01013154   - E9 324CFFFF         jmp NOTEPAD.01007D8B  
01013159     90                  nop  
0101315A     6A 00               push 0  
0101315C     68 E0810001         push NOTEPAD.010081E0                    ; ASCII "Except"  
01013161     68 49820001         push NOTEPAD.01008249                    ; ASCII "Have Int68 Except :=("  
01013166     6A 00               push 0  
01013168     FF15 19300101       call dword ptr ds:\[<&user32.MessageBoxA>>; USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
0101316E     90                  nop  
0101316F     68 8F310101         push NOTEPAD.0101318F  
01013174     64:A1 00000000      mov eax,dword ptr fs:\[0\]  
0101317A     50                  push eax  
0101317B     64:8925 00000000    mov dword ptr fs:\[0\],esp  
01013182     90                  nop  
01013183     CD20 43451300       vxdcall 134543  
01013189     90                  nop  
0101318A   - E9 FC4BFFFF         jmp NOTEPAD.01007D8B  
0101318F     90                  nop  
01013190     6A 00               push 0  
01013192     68 E0810001         push NOTEPAD.010081E0                    ; ASCII "Except"  
01013197     68 5F820001         push NOTEPAD.0100825F                    ; ASCII "Invalid Order(Command) :=("  
0101319C     6A 00               push 0  
0101319E     FF15 19300101       call dword ptr ds:\[<&user32.MessageBoxA>>; USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
010131A4     90                  nop  
010131A5     68 C2310101         push NOTEPAD.010131C2  
010131AA     64:A1 00000000      mov eax,dword ptr fs:\[0\]  
010131B0     50                  push eax  
010131B1     64:8925 00000000    mov dword ptr fs:\[0\],esp  
010131B8     90                  nop  
010131B9     90                  nop  
010131BA     90                  nop  
010131BB     FB                  sti  
010131BC     90                  nop  
010131BD   - E9 C94BFFFF         jmp NOTEPAD.01007D8B  
010131C2     90                  nop  
010131C3     6A 00               push 0  
010131C5     68 E0810001         push NOTEPAD.010081E0                    ; ASCII "Except"  
010131CA     68 7A820001         push NOTEPAD.0100827A                    ; ASCII "TeQuan ZhiLing"  
010131CF     6A 00               push 0  
010131D1     FF15 19300101       call dword ptr ds:\<&user32.MessageBoxA>>; USER32.MessageBoxA  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
010131D7     90                  nop  
010131D8     90                  nop  
010131D9     6A 00               push 0  
010131DB     68 B6820001         push NOTEPAD.010082B6                    ; ASCII "GoodBye~"  
010131E0     68 94820001         push NOTEPAD.01008294                    ; ASCII "You Can SEH~~Thanks You~GoodLuck~"  
010131E5     6A 00               push 0  
010131E7     FF15 19300101       call dword ptr ds:\[<&user32.MessageBoxA>>; USER32.MessageBoxA  
010131ED     90                  nop  
010131EE     90                  nop  
010131EF     6A 70               push 70  
010131F1     68 88180001         push NOTEPAD.01001888  
010131F6   - E9 EC38FFFF         jmp NOTEPAD.01006AE7  
  
  
谢谢,看完;[附件:SEHdiy.rar 

 4dd466d0e1e0cc123afbe504da2462ca