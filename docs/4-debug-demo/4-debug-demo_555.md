

# 广通证券信息引擎 2.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6103.htm`](https://www.pediy.com/kssd/pediy06/pediy6103.htm)

这个软件升级第 2 天我就破解了。之所以到现在才发布这个破解文章，是因为这个软件实在太好了，我不忍心看着他因为破解泛滥而不得不关闭 Guest 账户。请不要以此文章制作破解文件！谢谢合作 

软件名称：广通证券信息引擎 2.2（5.14 新版）  
官方主页：www.gtgwt.com  
免费用户限制：不定时跳广告  
收费用户：240 元/年  
破解方法：爆破（只能用这个）  
破解工具：FI，W32DASM，HIEW，FILEMON  
破解目的：去除广告  
   我敢打赌，我第一个破除了 2.2 的老版本。5.14 日，软件提示有新版本，强迫升级，呜……又该破解 

了…… 

   本来觉得这个新版本很简单的，升级变动也不大，按老的走就可以了。W32DASM 反汇编 stock.dll（主 

程序），来到 10006737，把那个 CALL nop 掉完事（老版本就是这样被我破掉的）。启动！^&%#@$!\*)&( 怎 

么回事？又重新下载引擎？？？可是，没升级呀…… 忽然意识到：这个新程序，是不是加了一个自校验 

功能？？？如果有改动就……自动重新下载！广告是去除了，下面的任务就是去掉那个自校验功能！（广 

告的去除方法不再详述）  
   设想程序基本思路： 

从服务器下载新版本号-->获得本地版本号-->比较-->相等就跳走-->不相等下载新版本-->提示升级-->安装 

或从服务器下载新版本号-->获得本地版本号-->比较-->相等就跳走-->系统自校验-->相等继续使用-->不相 

等从服务器上下载新版本-->提示升级-->安装 

  
     
   一开始是从 10006473 处下手，经过 N 次爆破均不成功。无奈之际，打开串式参考，找到两个可疑文件 

名：stock.dll、stock000.dll。赶紧去 winnt\\system32\\看，果然有这两个文件，但并不相同。猜想 

stock000.dll 是上一版本的备份。双击，来到 10009451（请从下面找到 10009451，从那里看，跟着注释走 

，这样可以较清楚地看到我的破解思路）  
   注释中的“右键”，是在 w32dasm 的操作。 

  
\* Referenced by a CALL at Address:  
|:100063C8                                         <----关键 CALL  
|  
:10007500 64A100000000            mov eax, dword ptr fs:\[00000000\]  
:10007506 6AFF                    push FFFFFFFF  
:10007508 68705B0310              push 10035B70  
:1000750D 50                      push eax  
:1000750E 64892500000000          mov dword ptr fs:\[00000000\], esp  
:10007515 83EC20                  sub esp, 00000020  
:10007518 53                      push ebx  
:10007519 55                      push ebp  
:1000751A 56                      push esi  
:1000751B 8BF1                    mov esi, ecx  
:1000751D 33DB                    xor ebx, ebx  
:1000751F 57                      push edi  
:10007520 8B8638060000            mov eax, dword ptr \[esi+00000638\]  
:10007526 3BC3                    cmp eax, ebx  
:10007528 740F                    je 10007539     <----很可疑，修改为 74，成功！  
:1000752A 50                      push eax  
:1000752B E86FF80100              call 10026D9F  
:10007530 83C404                  add esp, 00000004  
:10007533 899E38060000            mov dword ptr \[esi+00000638\], ebx 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:10007528(C)                                      <----右键 

:10007539 8B442444                mov eax, dword ptr \[esp+44\]  
:1000753D 8B7C2440                mov edi, dword ptr \[esp+40\]  
:10007541 8DAE38050000            lea ebp, dword ptr \[esi+00000538\]  
:10007547 50                      push eax  
:10007548 57                      push edi  
:10007549 8BCE                    mov ecx, esi  
:1000754B 894624                  mov dword ptr \[esi+24\], eax  
:1000754E 885D00                  mov byte ptr \[ebp+00\], bl  
:10007551 889E38040000            mov byte ptr \[esi+00000438\], bl  
:10007557 E894FEFFFF              call 100073F0  
:1000755C 85C0                    test eax, eax  
:1000755E 7545                    jne 100075A5      <----没准是这里，不过经测试，不是！继续 

向上  
:10007560 6800010000              push 00000100  
:10007565 E80CF80100              call 10026D76  
:1000756A 8BD0                    mov edx, eax  
:1000756C 83C404                  add esp, 00000004  
:1000756F 3BD3                    cmp edx, ebx  
:10007571 899638060000            mov dword ptr \[esi+00000638\], edx  
:10007577 7422                    je 1000759B 

\* Possible StringData Ref from Data Obj ->"包错误"  
                                 |  
:10007579 BF84570410              mov edi, 10045784  
:1000757E 83C9FF                  or ecx, FFFFFFFF  
:10007581 33C0                    xor eax, eax  
:10007583 F2                      repnz  
:10007584 AE                      scasb  
:10007585 F7D1                    not ecx  
:10007587 2BF9                    sub edi, ecx  
:10007589 8BC1                    mov eax, ecx  
:1000758B 8BF7                    mov esi, edi  
:1000758D 8BFA                    mov edi, edx  
:1000758F C1E902                  shr ecx, 02  
:10007592 F3                      repz  
:10007593 A5                      movsd  
:10007594 8BC8                    mov ecx, eax  
:10007596 83E103                  and ecx, 00000003  
:10007599 F3                      repz  
:1000759A A4                      movsb 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:10007577(C)  
|  
:1000759B B802000000              mov eax, 00000002  
:100075A0 E99E030000              jmp 10007943 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:1000755E(C)                                     <----右键  
|  
:100075A5 33C0                    xor eax, eax  
:100075A7 668B07                  mov ax, word ptr \[edi\]  
:100075AA 3D00200000              cmp eax, 00002000  
:100075AF 0F8F00010000            jg 100076B5     <----不可能是关键跳转，继续向上  
:100075B5 0F84A4000000            je 1000765F  
:100075BB 0500F0FFFF              add eax, FFFFF000  
:100075C0 83F80B                  cmp eax, 0000000B  
:100075C3 0F8778030000            ja 10007941  
:100075C9 FF248558790010          jmp dword ptr \[4\*eax+10007958\]  
:100075D0 57                      push edi  
:100075D1 8BCE                    mov ecx, esi  
:100075D3 E8D8080000              call 10007EB0  
:100075D8 E964030000              jmp 10007941  
:100075DD 57                      push edi  
:100075DE 8BCE                    mov ecx, esi  
:100075E0 E8AB0C0000              call 10008290  
:100075E5 E957030000              jmp 10007941  
:100075EA 57                      push edi  
:100075EB 8BCE                    mov ecx, esi  
:100075ED E89E110000              call 10008790  
:100075F2 E94A030000              jmp 10007941  
:100075F7 57                      push edi  
:100075F8 8BCE                    mov ecx, esi  
:100075FA E871150000              call 10008B70  
:100075FF E93D030000              jmp 10007941  
:10007604 57                      push edi  
:10007605 8BCE                    mov ecx, esi  
:10007607 E8B4160000              call 10008CC0  
:1000760C E930030000              jmp 10007941  
:10007611 57                      push edi  
:10007612 8BCE                    mov ecx, esi  
:10007614 E8E7190000              call 10009000  
:10007619 E923030000              jmp 10007941  
:1000761E 57                      push edi  
:1000761F 8BCE                    mov ecx, esi  
:10007621 E86A1B0000              call 10009190  
:10007626 85C0                    test eax, eax  
:10007628 0F8413030000            je 10007941  
:1000762E B801000000              mov eax, 00000001  
:10007633 E90B030000              jmp 10007943  
:10007638 57                      push edi  
:10007639 8BCE                    mov ecx, esi  
:1000763B E850070000              call 10007D90  
:10007640 85C0                    test eax, eax  
:10007642 0F85F9020000            jne 10007941  
:10007648 B802000000              mov eax, 00000002  
:1000764D E9F1020000              jmp 10007943  
:10007652 57                      push edi  
:10007653 8BCE                    mov ecx, esi  
:10007655 E876050000              call 10007BD0  
:1000765A E9E2020000              jmp 10007941 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:100075B5(C)  
|  
:1000765F 57                      push edi  
:10007660 8BCE                    mov ecx, esi  
:10007662 C7462C01000000          mov \[esi+2C\], 00000001  
:10007669 E822110000              call 10008790  
:1000766E 395E34                  cmp dword ptr \[esi+34\], ebx  
:10007671 0F8ECA020000            jle 10007941  
:10007677 8D4C2440                lea ecx, dword ptr \[esp+40\] 

\* Possible Reference to String Resource ID=01029: "c( } %s...%d%%"  
                                 |  
:1000767B 6805040000              push 00000405  
:10007680 51                      push ecx  
:10007681 E86AABFFFF              call 100021F0  
:10007686 8BC8                    mov ecx, eax  
:10007688 0FBF4702                movsx eax, word ptr \[edi+02\]  
:1000768C 40                      inc eax  
:1000768D 8D0480                  lea eax, dword ptr \[eax+4\*eax\]  
:10007690 8D0480                  lea eax, dword ptr \[eax+4\*eax\]  
:10007693 C1E002                  shl eax, 02  
:10007696 99                      cdq  
:10007697 F77E34                  idiv \[esi+34\]  
:1000769A 8B11                    mov edx, dword ptr \[ecx\]  
:1000769C 50                      push eax 

\* Possible StringData Ref from Data Obj ->"分时数据"  
                                 |  
:1000769D 6878570410              push 10045778  
:100076A2 52                      push edx  
:100076A3 55                      push ebp  
:100076A4 E8B6E30000              call 10015A5F  
:100076A9 83C418                  add esp, 00000018  
:100076AC 8D4C2440                lea ecx, dword ptr \[esp+40\]  
:100076B0 E987020000              jmp 1000793C 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:100075AF(C)                                        <----右键  
|  
:100076B5 3D02300000              cmp eax, 00003002  
:100076BA 0F8F7C010000            jg 1000783C        <----不可能是关键跳转，继续向上  
:100076C0 0F84D7000000            je 1000779D  
:100076C6 2D01200000              sub eax, 00002001  
:100076CB 747A                    je 10007747  
:100076CD 83E802                  sub eax, 00000002  
:100076D0 741F                    je 100076F1  
:100076D2 2DFE0F0000              sub eax, 00000FFE  
:100076D7 0F8564020000            jne 10007941  
:100076DD C7462C01000000          mov \[esi+2C\], 00000001  
:100076E4 57                      push edi  
:100076E5 8BCE                    mov ecx, esi  
:100076E7 E894030000              call 10007A80  
:100076EC E950020000              jmp 10007941 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:100076D0(C)  
|  
:100076F1 57                      push edi  
:100076F2 8BCE                    mov ecx, esi  
:100076F4 C7462C01000000          mov \[esi+2C\], 00000001  
:100076FB E800190000              call 10009000  
:10007700 395E34                  cmp dword ptr \[esi+34\], ebx  
:10007703 0F8E38020000            jle 10007941  
:10007709 8D442410                lea eax, dword ptr \[esp+10\] 

:10007812 8B54244C                mov edx, dword ptr \[esp+4C\]  
:10007816 50                      push eax  
:10007817 52                      push edx  
:10007818 51                      push ecx  
:10007819 55                      push ebp  
:1000781A E840E20000              call 10015A5F  
:1000781F 83C418                  add esp, 00000018  
:10007822 8D4C241C                lea ecx, dword ptr \[esp+1C\]  
:10007826 E8F3F00100              call 1002691E  
:1000782B C7442438FFFFFFFF        mov \[esp+38\], FFFFFFFF  
:10007833 8D4C2444                lea ecx, dword ptr \[esp+44\]  
:10007837 E900010000              jmp 1000793C 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:100076BA(C)                                           <----右键。  
|  
:1000783C 3D00500000              cmp eax, 00005000  
:10007841 755C                    jne 1000789F          <----来到这里，继续向上  
:10007843 3D00800000              cmp eax, 00008000  
:10007848 7418                    je 10007862  
:1000784A 3D00A00000              cmp eax, 0000A000  
:1000784F 0F85EC000000            jne 10007941  
:10007855 57                      push edi  
:10007856 8BCE                    mov ecx, esi  
:10007858 E8D3180000              call 10009130  
:1000785D E9DF000000              jmp 10007941 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:10007848(C)  
|  
:10007862 8D442420                lea eax, dword ptr \[esp+20\] 

\* Possible Reference to Dialog: DialogID\_03EA, CONTROL\_ID:0406, ""  
                                 | 

\* Possible Reference to String Resource ID=01030: " }?."  
                                 |  
:10007866 6806040000              push 00000406  
:1000786B 50                      push eax  
:1000786C 895E2C                  mov dword ptr \[esi+2C\], ebx  
:1000786F E87CA9FFFF              call 100021F0  
:10007874 8B38                    mov edi, dword ptr \[eax\]  
:10007876 83C9FF                  or ecx, FFFFFFFF  
:10007879 33C0                    xor eax, eax  
:1000787B 83C408                  add esp, 00000008  
:1000787E F2                      repnz  
:1000787F AE                      scasb  
:10007880 F7D1                    not ecx  
:10007882 2BF9                    sub edi, ecx  
:10007884 8BD1                    mov edx, ecx  
:10007886 8BF7                    mov esi, edi  
:10007888 8BFD                    mov edi, ebp  
:1000788A C1E902                  shr ecx, 02  
:1000788D F3                      repz  
:1000788E A5                      movsd  
:1000788F 8BCA                    mov ecx, edx  
:10007891 83E103                  and ecx, 00000003  
:10007894 F3                      repz  
:10007895 A4                      movsb  
:10007896 8D4C2420                lea ecx, dword ptr \[esp+20\]  
:1000789A E99D000000              jmp 1000793C 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:10007841(C)                                               <----跳转，此处按右键  
|  
:1000789F 57                      push edi  
:100078A0 8BCE                    mov ecx, esi  
:100078A2 E8E9190000              call 10009290             <----来到这里，继续向上  
:100078A7 85C0                    test eax, eax  
:100078A9 750A                    jne 100078B5  
:100078AB B803000000              mov eax, 00000003  
:100078B0 E98E000000              jmp 10007943 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:100078A9(C)  
|  
:100078B5 8B863C060000            mov eax, dword ptr \[esi+0000063C\]  
:100078BB 81C63C060000            add esi, 0000063C  
:100078C1 3958F8                  cmp dword ptr \[eax\-08\], ebx  
:100078C4 7E7B                    jle 10007941 

\* Reference To: KERNEL32.GetVersion, Ord:0174h  
                                 |  
:100078C6 FF15B4720310            Call dword ptr \[100372B4\]  
:100078CC 3D00000080              cmp eax, 80000000  
:100078D1 734A                    jnb 1000791D  
:100078D3 33C9                    xor ecx, ecx  
:100078D5 56                      push esi  
:100078D6 8ACC                    mov cl, ah  
:100078D8 80F90A                  cmp cl, 0A  
:100078DB 7720                    ja 100078FD  
:100078DD 8D542428                lea edx, dword ptr \[esp+28\] 

:10009284 90                      nop  
:10009285 90                      nop  
:10009286 90                      nop  
:10009287 90                      nop  
:10009288 90                      nop  
:10009289 90                      nop  
:1000928A 90                      nop  
:1000928B 90                      nop  
:1000928C 90                      nop  
:1000928D 90                      nop  
:1000928E 90                      nop  
:1000928F 90                      nop 

\* Referenced by a CALL at Address:  
|:100078A2                                                <----右键，看是哪里来的  
|  
:10009290 6AFF                    push FFFFFFFF  
:10009292 68C85B0310              push 10035BC8  
:10009297 64A100000000            mov eax, dword ptr fs:\[00000000\]  
:1000929D 50                      push eax  
:1000929E 64892500000000          mov dword ptr fs:\[00000000\], esp  
:100092A5 83EC3C                  sub esp, 0000003C  
:100092A8 53                      push ebx  
:100092A9 8BD9                    mov ebx, ecx  
:100092AB 56                      push esi  
:100092AC 57                      push edi  
:100092AD 8D4C240C                lea ecx, dword ptr \[esp+0C\]  
:100092B1 E89A9BFFFF              call 10002E50  
:100092B6 8B742458                mov esi, dword ptr \[esp+58\]  
:100092BA C744245000000000        mov \[esp+50\], 00000000  
:100092C2 8B4602                  mov eax, dword ptr \[esi+02\]  
:100092C5 85C0                    test eax, eax  
:100092C7 7471                    je 1000933A  
:100092C9 8D7E14                  lea edi, dword ptr \[esi+14\]  
:100092CC 6A09                    push 00000009 

\* Possible StringData Ref from Data Obj ->"stock.dll"  
                                 |  
:100092CE 683C570410              push 1004573C  
:100092D3 57                      push edi  
:100092D4 E877200100              call 1001B350  
:100092D9 83C40C                  add esp, 0000000C  
:100092DC 85C0                    test eax, eax  
:100092DE 752F                    jne 1000930F  
:100092E0 56                      push esi  
:100092E1 8BCB                    mov ecx, ebx  
:100092E3 E888000000              call 10009370            <----继续向上  
:100092E8 8D4C240C                lea ecx, dword ptr \[esp+0C\]  
:100092EC C7442450FFFFFFFF        mov \[esp+50\], FFFFFFFF  
:100092F4 E8179DFFFF              call 10003010  
:100092F9 5F                      pop edi  
:100092FA 5E                      pop esi  
:100092FB 33C0                    xor eax, eax  
:100092FD 5B                      pop ebx  
:100092FE 8B4C243C                mov ecx, dword ptr \[esp+3C\]  
:10009302 64890D00000000          mov dword ptr fs:\[00000000\], ecx  
:10009309 83C448                  add esp, 00000048  
:1000930C C20400                  ret 0004 

  
\* Referenced by a CALL at Address:  
|:100092E3                                                 <----CALL，右键，看看是哪里来的  
|  
:10009370 81EC00030000            sub esp, 00000300  
:10009376 8D842400020000          lea eax, dword ptr \[esp+00000200\]  
:1000937D 53                      push ebx  
:1000937E 55                      push ebp  
:1000937F 56                      push esi  
:10009380 57                      push edi  
:10009381 6880000000              push 00000080  
:10009386 50                      push eax 

\* Reference To: KERNEL32.GetSystemDirectoryA, Ord:0159h     <----获得系统目录，继续向上  
                                 |  
:10009387 FF15E4710310            Call dword ptr \[100371E4\]  
:1000938D 83C9FF                  or ecx, FFFFFFFF 

\* Possible StringData Ref from Data Obj ->"\\"  
                                 |  
:10009390 BFE4520410              mov edi, 100452E4  
:10009395 33C0                    xor eax, eax  
:10009397 8D942410020000          lea edx, dword ptr \[esp+00000210\]  
:1000939E F2                      repnz  
:1000939F AE                      scasb  
:100093A0 F7D1                    not ecx  
:100093A2 2BF9                    sub edi, ecx  
:100093A4 8BD9                    mov ebx, ecx  
:100093A6 8BF7                    mov esi, edi  
:100093A8 83C9FF                  or ecx, FFFFFFFF  
:100093AB 8BFA                    mov edi, edx  
:100093AD F2                      repnz  
:100093AE AE                      scasb  
:100093AF 8BCB                    mov ecx, ebx  
:100093B1 4F                      dec edi  
:100093B2 C1E902                  shr ecx, 02  
:100093B5 F3                      repz  
:100093B6 A5                      movsd  
:100093B7 8BCB                    mov ecx, ebx  
:100093B9 8D542410                lea edx, dword ptr \[esp+10\]  
:100093BD 83E103                  and ecx, 00000003  
:100093C0 8B9C2414030000          mov ebx, dword ptr \[esp+00000314\]  
:100093C7 F3                      repz  
:100093C8 A4                      movsb  
:100093C9 83C9FF                  or ecx, FFFFFFFF  
:100093CC 8DBC2410020000          lea edi, dword ptr \[esp+00000210\]  
:100093D3 F2                      repnz  
:100093D4 AE                      scasb  
:100093D5 F7D1                    not ecx  
:100093D7 2BF9                    sub edi, ecx  
:100093D9 8BC1                    mov eax, ecx  
:100093DB 8BF7                    mov esi, edi  
:100093DD 8BFA                    mov edi, edx  
:100093DF C1E902                  shr ecx, 02  
:100093E2 F3                      repz  
:100093E3 A5                      movsd  
:100093E4 8BC8                    mov ecx, eax  
:100093E6 33C0                    xor eax, eax  
:100093E8 83E103                  and ecx, 00000003  
:100093EB F3                      repz  
:100093EC A4                      movsb  
:100093ED 8D7C2410                lea edi, dword ptr \[esp+10\]  
:100093F1 83C9FF                  or ecx, FFFFFFFF  
:100093F4 F2                      repnz  
:100093F5 AE                      scasb  
:100093F6 0FBF530E                movsx edx, word ptr \[ebx+0E\]  
:100093FA F7D1                    not ecx  
:100093FC 49                      dec ecx  
:100093FD 8D7314                  lea esi, dword ptr \[ebx+14\]  
:10009400 8D7C0C10                lea edi, dword ptr \[esp+ecx+10\]  
:10009404 8BCA                    mov ecx, edx  
:10009406 8BC1                    mov eax, ecx  
:10009408 C1E902                  shr ecx, 02  
:1000940B F3                      repz  
:1000940C A5                      movsd  
:1000940D 8BC8                    mov ecx, eax  
:1000940F 33C0                    xor eax, eax  
:10009411 83E103                  and ecx, 00000003  
:10009414 F3                      repz  
:10009415 A4                      movsb  
:10009416 8D7C2410                lea edi, dword ptr \[esp+10\]  
:1000941A 83C9FF                  or ecx, FFFFFFFF  
:1000941D F2                      repnz  
:1000941E AE                      scasb  
:1000941F F7D1                    not ecx  
:10009421 49                      dec ecx  
:10009422 8DBC2410020000          lea edi, dword ptr \[esp+00000210\]  
:10009429 03CA                    add ecx, edx  
:1000942B 8D942410010000          lea edx, dword ptr \[esp+00000110\]  
:10009432 88440C10                mov byte ptr \[esp+ecx+10\], al  
:10009436 83C9FF                  or ecx, FFFFFFFF  
:10009439 F2                      repnz  
:1000943A AE                      scasb  
:1000943B F7D1                    not ecx  
:1000943D 2BF9                    sub edi, ecx  
:1000943F 8BC1                    mov eax, ecx  
:10009441 8BF7                    mov esi, edi  
:10009443 8BFA                    mov edi, edx  
:10009445 C1E902                  shr ecx, 02  
:10009448 F3                      repz  
:10009449 A5                      movsd  
:1000944A 8BC8                    mov ecx, eax  
:1000944C 83E103                  and ecx, 00000003  
:1000944F F3                      repz  
:10009450 A4                      movsb 

\* Possible StringData Ref from Data Obj ->"stock000.dll"   <----来到这里，从这里向上找  
                                 |  
:10009451 BF40590410              mov edi, 10045940  
:10009456 83C9FF                  or ecx, FFFFFFFF  
:10009459 33C0                    xor eax, eax  
:1000945B 8D942410010000          lea edx, dword ptr \[esp+00000110\]  
:10009462 F2                      repnz  
:10009463 AE                      scasb  
:10009464 F7D1                    not ecx  
:10009466 2BF9                    sub edi, ecx  
:10009468 6880000000              push 00000080  
:1000946D 8BF7                    mov esi, edi  
:1000946F 8BE9                    mov ebp, ecx  
:10009471 8BFA                    mov edi, edx  
:10009473 83C9FF                  or ecx, FFFFFFFF  
:10009476 F2                      repnz  
:10009477 AE                      scasb  
:10009478 8BCD                    mov ecx, ebp  
:1000947A 4F                      dec edi  
:1000947B C1E902                  shr ecx, 02  
:1000947E F3                      repz  
:1000947F A5                      movsd  
:10009480 8BCD                    mov ecx, ebp  
:10009482 8D842414010000          lea eax, dword ptr \[esp+00000114\]  
:10009489 83E103                  and ecx, 00000003  
:1000948C 50                      push eax  
:1000948D F3                      repz  
:1000948E A4                      movsb 

  
总结一下：  
\--------------------------------------  
10006737 处，E824000000 改 9090909090  
10007528 处，74         改 75 

总计修改 6 byte，收工。 

  
破解时间：3h25min39s  
发布时间：2003.05.16 14:39 

  
 4dd466d0e1e0cc123afbe504da2462ca