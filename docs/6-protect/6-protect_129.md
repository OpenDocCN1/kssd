

# 某结构设计软件

> 原文：[`www.pediy.com/kssd/pediy05/pediy50130.htm`](https://www.pediy.com/kssd/pediy05/pediy50130.htm)

我的打狗棒法   
  
目标：国内某结构设计软件   
工具：W32DASM，fi2.49，UltraEdit8.0，softice4.05   
目的：狗不理，不理狗   
  
软件简介：该软件可在其主页公开下载，不须带狗即可安装，但须带狗运行。其帮助中说明“设计版主要进行施工图设计，企业版包括设计版所有内容外，还针对加工制作需要，进行节点翻样、统计计算等。专业版.....”   
    开工：   
    先用 fi 检查没有加壳，再看安装目录里的文件，发现 gsdog.vxd（Goldensoft Dog 2.0），一只老狗。   
    软件在运行时，如果没有狗，则会跳出一个需要注册的对话框"没有找到加密器。"，那就先用 W32dsm 看看，反编译成功后，在串式参考查找出错的信息"没有找到加密器。"，   
找到下面   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482A1D(C)   
|   
:00482A87 8BCE                    mov ecx, esi   
:00482A89 89BE406BC900            mov dword ptr \[esi+00C96B40\], edi   
:00482A8F E82C060000              call 004830C0   
:00482A94 85C0                    test eax, eax   
:00482A96 7512                    jne 00482AAA      ====>jmp  （eb12）   
:00482A98 57                      push edi   
:00482A99 57                      push edi   
  
\* Possible StringData Ref from Data Obj ->"没有找到加密器。"    ====>就是这一处   
;;;找到这不难，难在不能走弯路，开始我就陷进去了。   
;;;往上看，跳过去怎样   
  
  
                                |   
:00482A9A 6800785400              push 00547800   
:00482A9F E8C7FB0600              call 004F266B   
:00482AA4 57                      push edi   
:00482AA5 E8F62A0500              call 004D55A0   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482A96(C)   
|   
:00482AAA 8BCE                    mov ecx, esi   
:00482AAC E85F060000              call 00483110      ====>后面要用的 call   
:00482AB1 85C0                    test eax, eax   
:00482AB3 0F8576FFFFFF            jne 00482A2F      ====>再跳 jmp    
:00482AB9 57                      push edi   
:00482ABA 57                      push edi   
  
\* Possible StringData Ref from Data Obj ->"非合法用户，软件无法使用。"   
                                  |   
:00482ABB 68E4775400              push 005477E4   
:00482AC0 E8A6FB0600              call 004F266B   
:00482AC5 57                      push edi   
:00482AC6 E8D52A0500              call 004D55A0   
  
\---------------------------------------------------   
...................................................   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482BD5(U)   
|   
:004829E0 6AFF                    push FFFFFFFF   
:004829E2 6818EB5000              push 0050EB18   
:004829E7 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:004829ED 50                      push eax   
:004829EE 64892500000000          mov dword ptr fs:\[00000000\], esp   
:004829F5 83EC0C                  sub esp, 0000000C   
:004829F8 53                      push ebx   
:004829F9 56                      push esi   
:004829FA 57                      push edi   
:004829FB 33FF                    xor edi, edi   
:004829FD 8BF1                    mov esi, ecx   
:004829FF 57                      push edi   
:00482A00 89742418                mov dword ptr \[esp+18\], esi   
:00482A04 E842C30700              call 004FED4B   
:00482A09 8BCE                    mov ecx, esi   
:00482A0B 897C2420                mov dword ptr \[esp+20\], edi   
:00482A0F C706E8055200            mov dword ptr \[esi\], 005205E8   
:00482A15 E826090000              call 00483340   
:00482A1A 83F801                  cmp eax, 00000001   
:00482A1D 7568                    jne 00482A87   
:00482A1F 8986406BC900            mov dword ptr \[esi+00C96B40\], eax   
:00482A25 C705E0040A0202000000    mov dword ptr \[020A04E0\], 00000002   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482AB3(C)   
|   
;;;;:从  ====>00482AB3  jne 00482A2F  跳过来   
  
:00482A2F 8D442410                lea eax, dword ptr \[esp+10\]   
:00482A33 50                      push eax   
:00482A34 E826230500              call 004D4D5F   
:00482A39 8B4C2414                mov ecx, dword ptr \[esp+14\]   
:00482A3D 83C404                  add esp, 00000004   
:00482A40 894C240C                mov dword ptr \[esp+0C\], ecx   
:00482A44 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
:00482A48 57                      push edi   
:00482A49 E8E4210600              call 004E4C32   
:00482A4E 57                      push edi   
:00482A4F 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00482A53 E8DA210600              call 004E4C32   
:00482A58 8B5810                  mov ebx, dword ptr \[eax+10\]   
:00482A5B 57                      push edi   
:00482A5C 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00482A60 43                      inc ebx   
:00482A61 E8CC210600              call 004E4C32   
:00482A66 8B4014                  mov eax, dword ptr \[eax+14\]   
:00482A69 056C070000              add eax, 0000076C   
:00482A6E 3DD2070000              cmp eax, 000007D2  ; 改为 cmp eax, 00000800   
;;;;这里运行一下，出现时间限制（2002 年限制），改为 3DD2080000。再次运行，进入程序。   
  
:00482A73 7E56                    jle 00482ACB   
:00482A75 57                      push edi   
:00482A76 57                      push edi   
  
\* Possible StringData Ref from Data Obj ->"您的软件应该升级了.请到 www.????????.com 下载!"   
                                  |   
:00482A77 6814785400              push 00547814   
:00482A7C E8EAFB0600              call 004F266B   
:00482A81 57                      push edi   
:00482A82 E8192B0500              call 004D55A0   
  
;;;;列位看官说了，这也太容易，别急，好戏在后头。进去一看 about，居然是什么“学习版”，TMD！很多功能不能用，菜单变灰。只好回头再来。   
  
\---------------------------------------------------   
...................................................   
;;;;在串式参考查找出错的信息"MST 2000(学习版)"，找到下面   
  
\* Referenced by a CALL at Address:   
|:00482AAC    
|   
:00483110 6AFF                    push FFFFFFFF   
:00483112 68F8EB5000              push 0050EBF8   
:00483117 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:0048311D 50                      push eax   
:0048311E 64892500000000          mov dword ptr fs:\[00000000\], esp   
:00483125 83EC10                  sub esp, 00000010   
:00483128 53                      push ebx   
:00483129 56                      push esi   
:0048312A 33DB                    xor ebx, ebx   
:0048312C 8D44240C                lea eax, dword ptr \[esp+0C\]   
:00483130 8BF1                    mov esi, ecx   
:00483132 C70548040A022A030000    mov dword ptr \[020A0448\], 0000032A   
:0048313C 881D40040A02            mov byte ptr \[020A0440\], bl   
:00483142 A33C040A02              mov dword ptr \[020A043C\], eax   
:00483147 66C7054C040A024D00      mov word ptr \[020A044C\], 004D   
:00483150 66C7054E040A020800      mov word ptr \[020A044E\], 0008   
:00483159 885C2414                mov byte ptr \[esp+14\], bl   
:0048315D E80E4BF8FF              call 00407C70   
:00483162 3BC3                    cmp eax, ebx   
:00483164 0F8538010000            jne 004832A2   
:0048316A 8B0DBCB95400            mov ecx, dword ptr \[0054B9BC\]   
:00483170 894C2408                mov dword ptr \[esp+08\], ecx   
:00483174 8D54240C                lea edx, dword ptr \[esp+0C\]   
:00483178 8D442408                lea eax, dword ptr \[esp+08\]   
:0048317C 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"%s"   
                                  |   
:0048317D 688C425400              push 0054428C   
:00483182 50                      push eax   
:00483183 895C242C                mov dword ptr \[esp+2C\], ebx   
:00483187 E82D130600              call 004E44B9   
:0048318C 83C40C                  add esp, 0000000C   
:0048318F 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00483193 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo98202"   
                                  |   
:00483194 68B8785400              push 005478B8   
:00483199 E8E80F0600              call 004E4186   
:0048319E 85C0                    test eax, eax   
:004831A0 0F8DEB000000            jnl 00483291   
:004831A6 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo98437"   
                                  |   
:004831A7 68AC785400              push 005478AC   
:004831AC 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004831B0 E8D10F0600              call 004E4186   
:004831B5 85C0                    test eax, eax   
:004831B7 0F8DD4000000            jnl 00483291   
:004831BD 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo98"   
                                  |   
:004831BE 68A4785400              push 005478A4   
:004831C3 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004831C7 E8BA0F0600              call 004E4186   
:004831CC 85C0                    test eax, eax   
:004831CE 7D17                    jge 004831E7   
:004831D0 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo01"   
                                  |   
:004831D1 689C785400              push 0054789C   
:004831D6 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004831DA E8A70F0600              call 004E4186   
:004831DF 85C0                    test eax, eax   
:004831E1 0F8CAA000000            jl 00483291   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004831CE(C)   
|   
:004831E7 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo984"   
                                  |   
:004831E8 6894785400              push 00547894   
:004831ED 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:004831F1 E8900F0600              call 004E4186   
:004831F6 85C0                    test eax, eax   
:004831F8 7D26                    jge 00483220   
:004831FA 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo985"   
                                  |   
:004831FB 688C785400              push 0054788C   
:00483200 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00483204 E87D0F0600              call 004E4186   
:00483209 85C0                    test eax, eax   
:0048320B 7D13                    jge 00483220   
:0048320D 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo014"   
                                  |   
:0048320E 6884785400              push 00547884   
:00483213 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00483217 E86A0F0600              call 004E4186   
:0048321C 85C0                    test eax, eax   
:0048321E 7C14                    jl 00483234   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004831F8(C), :0048320B(C)   
|   
:00483220 C705E0040A0202000000    mov dword ptr \[020A04E0\], 00000002      ====>这一处   
:0048322A C786406BC90001000000    mov dword ptr \[esi+00C96B40\], 00000001   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048321E(C)   
|   
:00483234 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo982"   
                                  |   
:00483235 687C785400              push 0054787C   
:0048323A 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:0048323E E8430F0600              call 004E4186   
:00483243 85C0                    test eax, eax   
:00483245 7D13                    jge 0048325A   
:00483247 53                      push ebx   
  
\* Possible StringData Ref from Data Obj ->"Luo012"   
                                  |   
:00483248 6874785400              push 00547874   
:0048324D 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00483251 E8300F0600              call 004E4186   
:00483256 85C0                    test eax, eax   
:00483258 7C10                    jl 0048326A   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483245(C)   
|   
:0048325A C705E0040A0201000000    mov dword ptr \[020A04E0\], 00000001   
:00483264 899E406BC900            mov dword ptr \[esi+00C96B40\], ebx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483258(C)   
|   
:0048326A 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:0048326E C7442420FFFFFFFF        mov \[esp+20\], FFFFFFFF   
:00483276 E83D750600              call 004EA7B8   
:0048327B 5E                      pop esi   
:0048327C B801000000              mov eax, 00000001   
:00483281 5B                      pop ebx   
:00483282 8B4C2410                mov ecx, dword ptr \[esp+10\]   
:00483286 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:0048328D 83C41C                  add esp, 0000001C   
:00483290 C3                      ret   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004831A0(C), :004831B7(C), :004831E1(C)   
|   
:00483291 8D4C2408                lea ecx, dword ptr \[esp+08\]   
:00483295 C7442420FFFFFFFF        mov \[esp+20\], FFFFFFFF   
:0048329D E816750600              call 004EA7B8   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483164(C)   
|   
:004832A2 8B4C2418                mov ecx, dword ptr \[esp+18\]   
:004832A6 5E                      pop esi   
:004832A7 33C0                    xor eax, eax   
:004832A9 5B                      pop ebx   
:004832AA 64890D00000000          mov dword ptr fs:\[00000000\], ecx   
:004832B1 83C41C                  add esp, 0000001C   
:004832B4 C3                      ret   
.........   
........   
  
:004832DE 90                      nop   
:004832DF 90                      nop   
:004832E0 56                      push esi   
:004832E1 8BF1                    mov esi, ecx   
:004832E3 E81B820600              call 004EB503   
:004832E8 A1E0040A02              mov eax, dword ptr \[020A04E0\]    ====>注意   
:004832ED 85C0                    test eax, eax   
:004832EF 750D                    jne 004832FE   
  
\* Possible StringData Ref from Data Obj ->"????2000(学习版)"    
      ; ====>！就是这，往下看   
|   
:004832F1 68EC785400              push 005478EC   
:004832F6 8D4E5C                  lea ecx, dword ptr \[esi+5C\]   
:004832F9 E843760600              call 004EA941   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004832EF(C)   
|   
:004832FE 833DE0040A0201          cmp dword ptr \[020A04E0\], 00000001 ==>注意   
:00483305 750D                    jne 00483314   
  
\* Possible StringData Ref from Data Obj ->"???/2000(设计版)"  ===>！！   
                                  |   
:00483307 68D8785400              push 005478D8   
:0048330C 8D4E5C                  lea ecx, dword ptr \[esi+5C\]   
:0048330F E82D760600              call 004EA941   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00483305(C)   
|   
:00483314 833DE0040A0202          cmp dword ptr \[020A04E0\], 00000002 ==>注意   
:0048331B 750D                    jne 0048332A   
  
\* Possible StringData Ref from Data Obj ->"????2000(企业版)"    ===>！！！   
                                  |   
:0048331D 68C4785400              push 005478C4   
:00483322 8D4E5C                  lea ecx, dword ptr \[esi+5C\]   
:00483325 E817760600              call 004EA941   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048331B(C)   
|   
:0048332A 6A00                    push 00000000   
:0048332C 8BCE                    mov ecx, esi   
:0048332E E88E5E0600              call 004E91C1   
:00483333 B801000000              mov eax, 00000001   
:00483338 5E                      pop esi   
:00483339 C3                      ret   
;;;你发现什么了？“dword ptr \[020A04E0\]”的值代表什么，不用我说了。开始，我并没有去改“dword ptr \[020A04E0\]”的值，我只是改跳转，跳到"??????(企业版)"，about 显示是企业版，但功能还是学习版。   
;;;;好，我们来改“dword ptr \[020A04E0\]”的值，使他为“00000002”。查找“dword ptr \[020A04E0\]”，我们找到不少，我们还是看看这种 “mov dword ptr \[020A04E0\], 00000002”地方   
  :00482A1F，:00483220 共 2 处，去看看。中间的过程，我不罗嗦了。我们去:00483220，发现他是从 by a CALL at Address:00482AAC 而来。最后我们来到:00482A1F   
    
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482BD5(U)   
|   
:004829E0 6AFF                    push FFFFFFFF   
:004829E2 6818EB5000              push 0050EB18   
:004829E7 64A100000000            mov eax, dword ptr fs:\[00000000\]   
:004829ED 50                      push eax   
:004829EE 64892500000000          mov dword ptr fs:\[00000000\], esp   
:004829F5 83EC0C                  sub esp, 0000000C   
:004829F8 53                      push ebx   
:004829F9 56                      push esi   
:004829FA 57                      push edi   
:004829FB 33FF                    xor edi, edi   
:004829FD 8BF1                    mov esi, ecx   
:004829FF 57                      push edi   
:00482A00 89742418                mov dword ptr \[esp+18\], esi   
:00482A04 E842C30700              call 004FED4B   
:00482A09 8BCE                    mov ecx, esi   
:00482A0B 897C2420                mov dword ptr \[esp+20\], edi   
:00482A0F C706E8055200            mov dword ptr \[esi\], 005205E8   
:00482A15 E826090000              call 00483340   
:00482A1A 83F801                  cmp eax, 00000001                
:00482A1D 7568                    jne 00482A87                    
:00482A1F 8986406BC900            mov dword ptr \[esi+00C96B40\], eax   
:00482A25 C705E0040A0202000000    mov dword ptr \[020A04E0\], 00000002   
    ====>就是这，往上看看   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00482AB3(C)   
|   
;这里是从  ====>00482AB3  jne 00482A2F  跳过来   
  
:00482A2F 8D442410                lea eax, dword ptr \[esp+10\]   
:00482A33 50                      push eax   
:00482A34 E826230500              call 004D4D5F   
:00482A39 8B4C2414                mov ecx, dword ptr \[esp+14\]   
:00482A3D 83C404                  add esp, 00000004   
:00482A40 894C240C                mov dword ptr \[esp+0C\], ecx   
:00482A44 8D4C240C                lea ecx, dword ptr \[esp+0C\]   
:00482A48 57                      push edi   
:00482A49 E8E4210600              call 004E4C32   
:00482A4E 57                      push edi   
:00482A4F 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00482A53 E8DA210600              call 004E4C32   
:00482A58 8B5810                  mov ebx, dword ptr \[eax+10\]   
:00482A5B 57                      push edi   
:00482A5C 8D4C2410                lea ecx, dword ptr \[esp+10\]   
:00482A60 43                      inc ebx   
:00482A61 E8CC210600              call 004E4C32   
:00482A66 8B4014                  mov eax, dword ptr \[eax+14\]   
:00482A69 056C070000              add eax, 0000076C   
:00482A6E 3DD2070000              cmp eax, 000007D2   
:00482A73 7E56                    jle 00482ACB   
:00482A75 57                      push edi   
:00482A76 57                      push edi   
  
  
  
  我们以前来过这，兜了一大圈又回来了。往上看到 jne 00482A87，改 no jmp，9090，（注：只改这一处，其余不该）。运行程序，ok！about 显示是企业版。试运行，正暗自高兴这样就破了。高兴太早了，忽然发现其中菜单“节点翻样”是灰的，最有用的不能用，TMD。难道，程序有问题。借狗一用，可是一切正常。革命尚未成功，同志尚需努力。   
    翻出《看雪论坛精华》找，难道是功能限制，改 EnableMenuIyem，EnableWindow，菜单“节点翻样”倒是亮了，但没反应。跟着范例改菜单激活，我功力太浅，不行。就此放弃？不，这不是我们 cracker 的风格。干它。   
    再回过头，看了几遍，眼前一亮，   
  
:00483220 C705E0040A0202000000    mov dword ptr \[020A04E0\], 00000002   
:0048322A C786406BC90001000000    mov dword ptr \[esi+00C96B40\], 00000001   
－－－－－－－－－－－   
:0048325A C705E0040A0201000000    mov dword ptr \[020A04E0\], 00000001   
:00483264 899E406BC900            mov dword ptr \[esi+00C96B40\], ebx   
－－－－－－－－－－－－－－－   
:00482A15 E826090000              call 00483340   
:00482A1A 83F801                  cmp eax, 00000001    ===>改这里            
:00482A1D 7568                    jne 00482A87          ===>改这里      
:00482A1F 8986406BC900            mov dword ptr \[esi+00C96B40\], eax   
:00482A25 C705E0040A0202000000    mov dword ptr \[020A04E0\], 00000002   
  
  
  发现什么，“dword ptr \[esi+00C96B40\]”的值代表什么？是“00000001”，改哪？还是上面:00482A1A。我们让 eax=000001,   
  
cmp eax, 00000001    ===>这两句改为 mov eax, 00000001 正好   
jne 00482A87        ===>   
mov eax, 00000001 的汇编代码：B801000000   
  
我们改（:00482A1A）：83F8017568 ===>B801000000   
再改时间限制（:00482A6E）： 3DD2070000 ===>3DD2080000   
    只改两处，大功告成，无限制。   
    总结：不要太相信狗的保护能力，在软件中保护不要太脆弱。这个软件里有很多花指令，又有何用？打狗要有耐心，恒心，要多仔细观察，这软件我发现 2 个标志，第 2 个标志狠容易忽视，没找到种子。   
    我没敢用它来干活，我怕它有别的....，毕竟是结构软件，没把握谁敢用，楼塌了怎么办？还是用狗狗吧。你这只赖皮狗。  4dd466d0e1e0cc123afbe504da2462ca