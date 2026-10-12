
### 机器码分析
```
004BBE09   .  66:83FF 41             cmp di,41  

004BBE0D     0F82 44010000       jb SnagIt32.004BBF57                 ;  跳就失败  

004BBE13   .  81C7 BFFF0000       add edi,0FFBF                         ;  就是这里了，99 是不行的了 用 A9 可以  

004BBE19   .  66:83FF 50          cmp di,50  

004BBE1D   .  72 74              jb short SnagIt32.004BBE93          我估记这个 A9 是与版本信息有关  

004BBE1F   .  8D4D 0C           lea ecx,dword ptr ss:[ebp+C]    用我提供的注册码，显示版本信息是 6.8 的  

004BBE22   .  FF15 ACF15500      call dword ptr ds:[<&MFC71.#2902>]       ;  MFC71.7C146AB0  

004BBE28   .  83F8 12           cmp eax,12  

004BBE2B   .  0F85 86000000     jnz SnagIt32.004BBEB7  

004BBE31   .  6A 04             push 4  

004BBE33   .  8D45 08          lea eax,dword ptr ss:[ebp+8]  

004BBE36   .  50                push eax  

004BBE37   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBE3A   .  FF15 1CE75500    call dword ptr ds:[<&MFC71.#5563>]       ;  MFC71.7C188DED  

004BBE40   .  6A 10            push 10  

004BBE42   .  53                push ebx  

004BBE43   .  8BC8              mov ecx,eax  

004BBE45   .  FF15 2CF35500    call dword ptr ds:[<&MFC71.#876>]        ;  MFC71.7C158BCD  

004BBE4B   .  50                push eax  

004BBE4C   .  FFD6              call esi  

004BBE4E   .  83C4 0C          add esp,0C  

004BBE51   .  8D4D 08          lea ecx,dword ptr ss:[ebp+8]  

004BBE54   .  8945 EC          mov dword ptr ss:[ebp\-14],eax          ;  ss:[12dde4]=4578  

004BBE57   .  FF15 34F35500    call dword ptr ds:[<&MFC71.#578>]        ;  MFC71.7C1771B1  

004BBE5D   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBE60   .  FF15 ACF15500    call dword ptr ds:[<&MFC71.#2902>]       ;  MFC71.7C146AB0  

004BBE66   .  83E8 04         sub eax,4  

004BBE69   .  50                push eax  

004BBE6A   .  8D45 08          lea eax,dword ptr ss:[ebp+8]  

004BBE6D   .  50                push eax  

004BBE6E   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBE71   .  FF15 B4E45500    call dword ptr ds:[<&MFC71.#3997>]       ;  MFC71.7C188E36  

004BBE77   .  50                push eax  

004BBE78   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBE7B   .  C645 FC 01       mov byte ptr ss:[ebp\-4],1  

004BBE7F   .  FF15 90E45500    call dword ptr ds:[<&MFC71.#781>]        ;  MFC71.7C150F15  

004BBE85   .  8D4D 08          lea ecx,dword ptr ss:[ebp+8]  

004BBE88   .  885D FC          mov byte ptr ss:[ebp\-4],bl  

004BBE8B   .  FF15 34F35500    call dword ptr ds:[<&MFC71.#578>]        ;  MFC71.7C1771B1  

004BBE91   .  EB 24            jmp short SnagIt32.004BBEB7  

004BBE93   >  FF75 08         push dword ptr ss:[ebp+8]              ; /s  

004BBE96   .  FF15 54F45500   call dword ptr ds:[<&MSVCR71.\_strdup>]   ; \_\_strdup  

004BBE9C   .  8BF0            mov esi,eax  

004BBE9E   .  3BF3            cmp esi,ebx  

004BBEA0   .  59                pop ecx  

004BBEA1   .  74 14            je short SnagIt32.004BBEB7  

004BBEA3   .  8B4D E4          mov ecx,dword ptr ss:[ebp\-1C]  

004BBEA6   .  56                push esi                              ; /Arg1  

004BBEA7   .  E8 D4820800      call SnagIt32.00544180               ; \SnagIt32.00544180  

004BBEAC   .  56                push esi                              ; /block  

004BBEAD   .  8945 EC          mov dword ptr ss:[ebp\-14],eax          ; |  

004BBEB0   .  FF15 DCF45500    call dword ptr ds:[<&MSVCR71.free>]     ; \free  

004BBEB6   .  59                pop ecx  

004BBEB7   >  66:395D EC       cmp word ptr ss:[ebp\-14],bx  

004BBEBB   .  0F84 9A000000   je SnagIt32.004BBF5B                 ;  这里是不能跳的  

004BBEC1   .  6A 0F            push 0F                               ; /n = F (15.)  

004BBEC3   .  8D45 D0          lea eax,dword ptr ss:[ebp\-30]          ; |  

004BBEC6   .  53                push ebx                              ; |c  

004BBEC7   .  50                push eax                              ; |s  

004BBEC8   .  E8 97DD0800      call <jmp.&MSVCR71.memset>           ; \memset  

004BBECD   .  83C4 0C          add esp,0C  

004BBED0   .  6A 0E            push 0E  

004BBED2   .  6A 0E            push 0E  

004BBED4   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBED7   .  FF15 F4E45500   call dword ptr ds:[<&MFC71.#2468>]       ;  MFC71.7C15102A  

004BBEDD   .  50                push eax                              ; |src  

004BBEDE   .  8D45 D0          lea eax,dword ptr ss:[ebp\-30]          ; |  

004BBEE1   .  50                push eax                              ; |dest  

004BBEE2   .  E8 77DD0800      call <jmp.&MSVCR71.memcpy>           ; \memcpy  

004BBEE7   .  83C4 0C          add esp,0C  

004BBEEA   .  6A FF            push -1  

004BBEEC   .  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  

004BBEEF   .  FF15 E8E45500   call dword ptr ds:[<&MFC71.#5403>]       ;  MFC71.7C16A479  

004BBEF5   .  8D45 9C          lea eax,dword ptr ss:[ebp\-64]  

004BBEF8   .  50                push eax  

004BBEF9   .  E8 B2970800      call SnagIt32.005456B0  

004BBEFE   .  85C0              test eax,eax  

004BBF00   .  59                pop ecx  

004BBF01   .  74 58            je short SnagIt32.004BBF5B             ;  不能跳  

004BBF03   .  6A 02            push 2  

004BBF05   .  8D45 E0          lea eax,dword ptr ss:[ebp\-20]  

004BBF08   .  50                push eax  

004BBF09   .  8D45 9C          lea eax,dword ptr ss:[ebp\-64]  

004BBF0C   .  50                push eax  

004BBF0D   .  E8 AE970800      call SnagIt32.005456C0  

004BBF12   .  83C4 0C          add esp,0C  

004BBF15   .  85C0              test eax,eax  

004BBF17   .  74 42            je short SnagIt32.004BBF5B             ;  不能跳  

004BBF19   .  6A 02            push 2  

004BBF1B   .  8D45 EC          lea eax,dword ptr ss:[ebp\-14]  

004BBF1E   .  50                push eax  

004BBF1F   .  8D45 9C          lea eax,dword ptr ss:[ebp\-64]  

004BBF22   .  50                push eax  

004BBF23   .  E8 98970800      call SnagIt32.005456C0  

004BBF28   .  83C4 0C          add esp,0C  

004BBF2B   .  85C0              test eax,eax  

004BBF2D   .  74 2C            je short SnagIt32.004BBF5B             ;  不能跳  

004BBF2F   .  8D45 D0          lea eax,dword ptr ss:[ebp\-30]  

004BBF32   .  50                push eax  

004BBF33   .  8D45 9C          lea eax,dword ptr ss:[ebp\-64]  

004BBF36   .  50                push eax  

004BBF37   .  E8 44980800      call SnagIt32.00545780----->这里比较注册码要跟进去  

004BBF3C   .  85C0              test eax,eax  

004BBF3E   .  59                pop ecx  

004BBF3F   .  59                pop ecx  

004BBF40   .  74 19            je short SnagIt32.004BBF5B             ;  不能跳     

004BBF42   .  66:83FF 63       cmp di,63                               ;  注意 edi  不能小于 63  

                                      它是由 A9＋0FFBF  计算出来的  

004BBF46     73 06            jnb short SnagIt32.004BBF4E          ;  这里要跳   

004BBF48   .  C645 F3 0B      mov byte ptr ss:[ebp\-D],0B  

004BBF4C   .  EB 0D           jmp short SnagIt32.004BBF5B  

004BBF4E   >  885D F3          mov byte ptr ss:[ebp\-D],bl  

004BBF51   .  C645 F2 01     mov byte ptr ss:[ebp\-E],1               ;  只要到这里就注册成功了，哈哈  

004BBF55   .  EB 04           jmp short SnagIt32.004BBF5B  

004BBF57   >  C645 F3 0C      mov byte ptr ss:[ebp\-D],0C  

004BBF5B   >  5F                pop edi  

004BBF5C   .  5E                pop esi  

004BBF5D   >  8D4D 0C          lea ecx,dword ptr ss:[ebp+C]  
```
