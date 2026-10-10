

# PC Guard for Win32 V5.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6371.htm`](https://www.pediy.com/kssd/pediy06/pediy6371.htm)

PC Guard for Win32 V5.0 DEMO 脱壳 —— PCGWIN32.EXE 主程序   
    
    
    
下载页面：  http://www.sofpro.com/downl.htm    
软件大小：  692 KB   
发布日期：  by Blagoje Ceklic. 03.IX.2003.   
  
【软件简介】：PC Guard for Win32 is a professional software protection and licensing system. With PC Guard for Win32 you can easily protect Windows 95/98/NT 32bit EXE/DLL applications from illegal distribution and reverse engineering . Even more, you don't need any source code editing or programming experience for this. Everything is fully automatic!    
  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、Ollydbg1.09、PEiD、LordPE、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
             
  
  
用 Ollydbg 手动脱壳,老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。   
  
设置 Ollydbg 忽略所有异常。下断：BP LoadLibraryA   F9 运行，断下！   
     
  
00483000     FC                   cld   
                                  ====>进入 OD 后断在这！   
  
77E605D8    837C24 04 00         cmp dword ptr ss:\[esp+4\],0   
                                  ====>断在这！继续 F9 运行，不断拦下！   
77E605DD     53                   push ebx   
77E605DE     56                   push esi   
77E605DF     74 19                je short kernel32.77E605FA   
77E605E1     68 9C5BE777          push kernel32.77E75B9C   
77E605E6     FF7424 10            push dword ptr ss:\[esp+10\]   
77E605EA     FF15 9013E477        call dword ptr ds:\[<&ntdll.\_strcmpi>\]   
77E605F0     85C0                 test eax,eax   
77E605F2     59                   pop ecx   
77E605F3     59                   pop ecx   
77E605F4     0F84 76AF0100        je kernel32.77E7B570   
77E605FA     6A 00                push 0   
77E605FC     6A 00                push 0   
77E605FE     FF7424 14            push dword ptr ss:\[esp+14\]   
77E60602     E8 B1FFFFFF          call kernel32.LoadLibraryExA   
77E60607     5E                   pop esi   
77E60608     5B                   pop ebx   
77E60609     C2 0400              retn 4   
  
确定弹出的“入口点预警”对话框，这时可以看见堆栈出现的这些 DLL 名。   
0012FF28    0042D750   FileName = "COMCTL32.dll"   
0012FF28    0042D774   FileName = "IMAGEHLP.dll"   
0012FF28    0042DD74   FileName = "comdlg32.dll"   
0012FF28    0042DE46   FileName = "VERSION.dll"   
  
中断 4 次，出现 VERSION.dll。再 F9 两次程序就运行啦，停！取消这个断点！   
  
  
————————————————————————   
  
再下断： BP GetProcAddress       F9 运行，断下！     取消这个断点！   
  
  
77E5A5FD     55                   push ebp   
                                  ====>断在这！CTRL+F9 执行到返回   
77E5A5FE     8BEC                 mov ebp,esp   
77E5A600     51                   push ecx   
77E5A601     51                   push ecx   
77E5A602     53                   push ebx   
77E5A603     57                   push edi   
77E5A604     8B7D 0C              mov edi,dword ptr ss:\[ebp+C\]   
77E5A607     BB FFFF0000          mov ebx,0FFFF   
77E5A60C     3BFB                 cmp edi,ebx   
77E5A60E     0F86 6B83FFFF        jbe kernel32.77E5297F   
77E5A614     57                   push edi   
77E5A615     8D45 F8              lea eax,dword ptr ss:\[ebp\-8\]   
77E5A618     50                   push eax   
77E5A619     FF15 8012E477        call dword ptr ds:\[<&ntdll.RtlInitString>\]    
77E5A61F     8D45 0C              lea eax,dword ptr ss:\[ebp+C\]   
77E5A622     50                   push eax   
77E5A623     6A 00                push 0   
77E5A625     8D45 F8              lea eax,dword ptr ss:\[ebp\-8\]   
77E5A628     50                   push eax   
77E5A629     6A 00                push 0   
77E5A62B     FF75 08              push dword ptr ss:\[ebp+8\]   
77E5A62E     E8 B4F2FFFF          call kernel32.77E598E7   
77E5A633     50                   push eax   
77E5A634     E8 BEFFFFFF          call <jmp.&ntdll.LdrGetProcedureAddress>   
77E5A639     85C0                 test eax,eax   
77E5A63B     0F8C 5725FFFF        jl kernel32.77E4CB98   
77E5A641     6A 00                push 0   
77E5A643     FF75 08              push dword ptr ss:\[ebp+8\]   
77E5A646     E8 9CF2FFFF          call kernel32.77E598E7   
77E5A64B     3945 0C              cmp dword ptr ss:\[ebp+C\],eax   
77E5A64E     0F84 A1870200        je kernel32.77E82DF5   
77E5A654     8B45 0C              mov eax,dword ptr ss:\[ebp+C\]   
77E5A657     5F                   pop edi   
77E5A658     5B                   pop ebx   
77E5A659     C9                   leave   
77E5A65A     C2 0800              retn 8   
                                  ====>返回到 0047DCE6   
  
0047DCE6     0BC0                 or eax,eax   
0047DCE8     74 12                je short 0047DCFC   
0047DCEA     5D                   pop ebp   
0047DCEB     5F                   pop edi   
0047DCEC     5E                   pop esi   
0047DCED     5A                   pop edx   
0047DCEE     59                   pop ecx   
0047DCEF     5B                   pop ebx   
0047DCF0     C3                   retn   
                                  ====>返回到 0047DBD1   
  
————————————————————————   
  
0047DBD1     EB 01                jmp short 0047DBD4   
                                  ====>返回到这里！:-) 不用 F7 蜗牛般跟踪，向下找 RET   
0047DBD3     898B BD3AF041        mov dword ptr ds:\[ebx+41F03ABD\],ecx   
0047DBD9     009CEB 01D59DEB      add byte ptr ds:\[ebx+ebp\*8+EB9DD501\],bl   
0047DBE0     010B                 add dword ptr ds:\[ebx\],ecx   
0047DBE2     03BD 40384200        add edi,dword ptr ss:\[ebp+423840\]   
0047DBE8     EB 01                jmp short 0047DBEB   
0047DBEA     E3 60                jecxz short 0047DC4C   
0047DBEC     E8 03000000          call 0047DBF4   
0047DBF1     D2EB                 shr bl,cl   
0047DBF3     0B58 EB              or ebx,dword ptr ds:\[eax\-15\]   
0047DBF6     0148 40              add dword ptr ds:\[eax+40\],ecx   
0047DBF9     EB 01                jmp short 0047DBFC   
0047DBFB     35 FFE0E761          xor eax,61E7E0FF   
0047DC00     89048F               mov dword ptr ds:\[edi+ecx\*4\],eax   
0047DC03     EB 01                jmp short 0047DC06   
0047DC05     89F7                 mov edi,esi   
0047DC07     8570 38              test dword ptr ds:\[eax+38\],esi   
0047DC0A     42                   inc edx   
0047DC0B     0000                 add byte ptr ds:\[eax\],al   
0047DC0D     0000                 add byte ptr ds:\[eax\],al   
0047DC0F     04 9C                add al,9C   
0047DC11     EB 01                jmp short 0047DC14   
0047DC13     D5 9D                aad 9D   
0047DC15     EB 01                jmp short 0047DC18   
0047DC17     0B740B EB            or esi,dword ptr ds:\[ebx+ecx\-15\]   
0047DC1B     01E3                 add ebx,esp   
0047DC1D     EB 01                jmp short 0047DC20   
0047DC1F     D4 E8                aam 0E8   
0047DC21     14 00                adc al,0   
0047DC23     0000                 add byte ptr ds:\[eax\],al   
0047DC25     41                   inc ecx   
0047DC26     E9 24FFFFFF          jmp 0047DB4F   
0047DC2B     B8 FFFFFFFF          mov eax,-1   
0047DC30     C3                   retn   
                                  ====>此处下断！会节省不少时间。F9 直接断在这！   
  
  
BTW：常有朋友问“你们怎么知道这里跳那里不跳、此处该下断彼处又不用下断？”等等，偶只能很遗憾的回答：偶仅仅是菜鸟，偶也只知其然而不知其所以然。但偶知道的“此处下断！会节省不少时间”是偶试了 N 次，有的甚至是花了数天的时间才实践、总结出来的。由于水平低微，只能做到这一步了，请诸位见谅！   
  
————————————————————————   
  
0046F30C     EB 01                jmp short 0046F30F   
                                  ====>0047DC30 返回到这里！   
  
0046F30F     60                   pushad   
0046F310     E8 03000000          call 0046F318   
  
0046F318     58                   pop eax     
0046F319     EB 01                jmp short 0046F31C   
  
0046F31C     40                   inc eax   
0046F31D     EB 01                jmp short 0046F320   
  
0046F320     FFE0                 jmp eax   
  
0046F316     EB 0B                jmp short 0046F323   
  
0046F323     61                   popad   
0046F324     85C0                 test eax,eax   
0046F326     9C                   pushfd   
0046F327     EB 01                jmp short 0046F32A   
  
0046F32A     9D                   popfd   
0046F32B     EB 01                jmp short 0046F32E   
  
0046F32E     0F84 B0000000        je 0046F3E4   
                                  ====>看到这个 je 0046F3E4 向下找 0046F3E4 在其上面的 JMP 处下断！   
  
0046F3DF     E9 6B240100          jmp 0048184F   
                                  ====>在这下断！F9 断下！呵呵，又偷了点巧 :-)   
0046F3E4     B8 05000000          mov eax,5   
                                  ====>在 0046F3E4 上面的 JMP 处下断！   
  
0048184F     8B85 48384200        mov eax,dword ptr ss:\[ebp+423848\]   
00481855     EB 01                jmp short 00481858   
  
00481858     8DB5 C72E4200        lea esi,dword ptr ss:\[ebp+422EC7\]   
0048185E     EB 01                jmp short 00481861   
  
00481861     EB 01                jmp short 00481864   
  
00481864     B9 98020000          mov ecx,298   
00481869     60                   pushad   
0048186A     E8 03000000          call 00481872   
  
00481872     EB 01                jmp short 00481875   
  
00481875     58                   pop eax     
00481876     EB 01                jmp short 00481879   
  
00481879     40                   inc eax   
0048187A     EB 01                jmp short 0048187D   
  
0048187D     FFE0                 jmp eax   
  
00481870     EB 0E                jmp short 00481880   
  
00481880     61                   popad   
00481881     0006                 add byte ptr ds:\[esi\],al   
00481883     EB 01                jmp short 00481886   
  
00481886     EB 01                jmp short 00481889   
  
00481889     D1C8                 ror eax,1   
0048188B     EB 01                jmp short 0048188E   
  
0048188E     4E                   dec esi   
0048188F     EB 01                jmp short 00481892   
  
00481892     EB 01                jmp short 00481895   
  
00481895     E2 EA                loopd short 00481881   
                                  ====>F4 下去跳出 LOOP   
00481897     9C                   pushfd   
00481898     EB 01                jmp short 0048189B   
  
0048189B     9D                   popfd   
0048189C     EB 01                jmp short 0048189F   
  
0048189F     F7857038420000000020 test dword ptr ss:\[ebp+423870\],20000000   
004818A9     0F85 85000000        jnz 00481934   
  
00481934     81ED 4DF1B84A        sub ebp,4AB8F14D   
0048193A     9C                   pushfd   
0048193B     EB 01                jmp short 0048193E   
  
0048193E     9D                   popfd   
0048193F     EB 01                jmp short 00481942   
  
00481942     60                   pushad   
00481943     EB 01                jmp short 00481946   
  
00481946     E8 15000000          call 00481960   
  
00481960     2BFF                 sub edi,edi   
00481962     60                   pushad   
00481963     E8 03000000          call 0048196B   
  
0048196B     EB 01                jmp short 0048196E   
  
0048196E     58                   pop eax      
0048196F     EB 01                jmp short 00481972   
  
00481972     40                   inc eax   
00481973     EB 01                jmp short 00481976   
  
00481976     FFE0                 jmp eax   
  
00481969     EB 0E                jmp short 00481979   
  
00481979     61                   popad   
0048197A     64:FF37              push dword ptr fs:\[edi\]   
0048197D     9C                   pushfd   
0048197E     EB 01                jmp short 00481981   
  
00481981     9D                   popfd   
00481982     EB 01                jmp short 00481985   
  
00481985     64:8927              mov dword ptr fs:\[edi\],esp   
00481988     EB 01                jmp short 0048198B   
  
0048198B     EB 01                jmp short 0048198E   
  
0048198E     F1                   int1   
                                  ====>异常！看看堆栈区的第二条地址是 0048194B   
  
0048194B     EB 01                jmp short 0048194E   
                                  ====>堆栈区的第二条地址  设断  F9 断在此处   
  
0048194E     EB 01                jmp short 00481951   
  
00481951     8B6424 08            mov esp,dword ptr ss:\[esp+8\]   
00481955     EB 01                jmp short 00481958   
  
00481958     EB 45                jmp short 0048199F   
  
0048199F     EB 01                jmp short 004819A2   
  
004819A2     EB 01                jmp short 004819A5   
  
004819A5     85E4                 test esp,esp   
004819A7     9C                   pushfd   
004819A8     EB 01                jmp short 004819AB   
  
004819AB     9D                   popfd   
004819AC     EB 01                jmp short 004819AF   
  
004819AF     79 0C                jns short 004819BD   
  
004819BD     33C0                 xor eax,eax   
004819BF     EB 01                jmp short 004819C2   
  
004819C2     EB 01                jmp short 004819C5   
  
004819C5     64:8F00              pop dword ptr fs:\[eax\]    
004819C8     60                   pushad   
004819C9     E8 03000000          call 004819D1   
  
004819D1     EB 01                jmp short 004819D4   
  
004819D4     58                   pop eax      
004819D5     EB 01                jmp short 004819D8   
  
004819D8     40                   inc eax   
004819D9     EB 01                jmp short 004819DC   
  
004819DC     FFE0                 jmp eax   
  
004819CF     EB 0E                jmp short 004819DF   
  
004819DF     61                   popad   
004819E0     58                   pop eax   
004819E1     EB 01                jmp short 004819E4   
  
004819E4     60                   pushad   
004819E5     E8 03000000          call 004819ED   
  
004819ED     58                   pop eax    
004819EE     EB 01                jmp short 004819F1   
  
004819F1     40                   inc eax   
004819F2     EB 01                jmp short 004819F5   
  
004819F5     FFE0                 jmp eax   
  
004819EB     EB 0B                jmp short 004819F8   
  
004819F8     61                   popad   
004819F9     61                   popad   
004819FA     9C                   pushfd   
004819FB     EB 01                jmp short 004819FE   
  
004819FE     9D                   popfd   
004819FF     EB 01                jmp short 00481A02   
  
00481A02     81C5 4DF1B84A        add ebp,4AB8F14D   
00481A08     60                   pushad   
00481A09     E8 03000000          call 00481A11   
  
00481A11     EB 01                jmp short 00481A14   
  
00481A14     58                   pop eax    
00481A15     EB 01                jmp short 00481A18   
  
00481A18     40                   inc eax   
00481A19     EB 01                jmp short 00481A1C   
  
00481A1C     FFE0                 jmp eax   
  
00481A0F     EB 0E                jmp short 00481A1F   
  
00481A1F     61                   popad   
00481A20     8B85 48384200        mov eax,dword ptr ss:\[ebp+423848\]   
00481A26     60                   pushad   
00481A27     E8 03000000          call 00481A2F   
  
00481A2F     EB 01                jmp short 00481A32   
  
00481A32     58                   pop eax      
00481A33     EB 01                jmp short 00481A36   
  
00481A36     40                   inc eax   
00481A37     EB 01                jmp short 00481A3A   
  
00481A3A     FFE0                 jmp eax   
  
00481A2D     EB 0E                jmp short 00481A3D   
  
00481A3D     61                   popad   
00481A3E     3385 C42E4200        xor eax,dword ptr ss:\[ebp+422EC4\]   
                                  ====>EAX\=F253D0A0 XOR F2518C81=00025C21 这是 OEP 的偏移值   
00481A44     9C                   pushfd   
00481A45     EB 01                jmp short 00481A48   
  
00481A4C     0385 40384200        add eax,dword ptr ss:\[ebp+423840\] ; PCGWIN32.00400000   
                                  ====>EAX\=00025C21 + 00400000=00425C21    这就是 OEP 值  :-)   
00481A52     EB 01                jmp short 00481A55   
  
00481A55     60                   pushad   
00481A56     E8 03000000          call 00481A5E   
  
00481A5E     58                   pop eax     
00481A5F     EB 01                jmp short 00481A62   
  
00481A62     40                   inc eax   
00481A63     EB 01                jmp short 00481A66   
  
00481A66     FFE0                 jmp eax   
  
00481A5C     EB 0B                jmp short 00481A69   
  
00481A69     61                   popad   
00481A6A     8BA5 4C394200        mov esp,dword ptr ss:\[ebp+42394C\]   
00481A70     9C                   pushfd   
00481A71     EB 01                jmp short 00481A74   
  
00481A74     9D                   popfd   
00481A75     EB 01                jmp short 00481A78   
  
00481A78     50                   push eax   
00481A79     60                   pushad   
00481A7A     E8 03000000          call 00481A82   
  
00481A82     EB 01                jmp short 00481A85   
  
00481A85     58                   pop eax     
00481A86     EB 01                jmp short 00481A89   
  
00481A89     40                   inc eax   
00481A8A     EB 01                jmp short 00481A8D   
  
00481A8D     FFE0                 jmp eax   
  
00481A80     EB 0E                jmp short 00481A90   
  
00481A90     61                   popad   
00481A91     8B85 30394200        mov eax,dword ptr ss:\[ebp+423930\]   
00481A97     9C                   pushfd   
00481A98     EB 01                jmp short 00481A9B   
  
00481A9B     9D                   popfd   
00481A9C     EB 01                jmp short 00481A9F   
  
00481A9F     8BB5 40394200        mov esi,dword ptr ss:\[ebp+423940\]   
00481AA5     60                   pushad   
00481AA6     E8 03000000          call 00481AAE   
  
00481AAE     EB 01                jmp short 00481AB1   
  
00481AB1     58                   pop eax     
00481AB2     EB 01                jmp short 00481AB5   
  
00481AB5     40                   inc eax   
00481AB6     EB 01                jmp short 00481AB9   
  
00481AB9     FFE0                 jmp eax   
  
00481AAC     EB 0E                jmp short 00481ABC   
  
00481ABC     61                   popad   
00481ABD     8BBD 44394200        mov edi,dword ptr ss:\[ebp+423944\]   
00481AC3     9C                   pushfd   
00481AC4     EB 01                jmp short 00481AC7   
  
00481AC7     9D                   popfd   
00481AC8     EB 01                jmp short 00481ACB   
  
00481ACB     8B9D 34394200        mov ebx,dword ptr ss:\[ebp+423934\]   
00481AD1     60                   pushad   
00481AD2     E8 03000000          call 00481ADA   
  
00481ADA     EB 01                jmp short 00481ADD   
  
00481ADD     58                   pop eax   
00481ADE     EB 01                jmp short 00481AE1   
  
00481AE1     40                   inc eax   
00481AE2     EB 01                jmp short 00481AE5   
  
00481AE5     FFE0                 jmp eax   
  
00481AD8     EB 0E                jmp short 00481AE8   
  
00481AE8     61                   popad   
00481AE9     8B8D 38394200        mov ecx,dword ptr ss:\[ebp+423938\]   
00481AEF     EB 01                jmp short 00481AF2   
  
00481AF2     60                   pushad   
00481AF3     E8 03000000          call 00481AFB   
  
00481AFB     58                   pop eax   
00481AFC     EB 01                jmp short 00481AFF   
  
00481AFF     40                   inc eax   
00481B00     EB 01                jmp short 00481B03   
  
00481B03     FFE0                 jmp eax   
  
00481AF9     EB 0B                jmp short 00481B06   
  
00481B06     61                   popad   
00481B07     8B95 3C394200        mov edx,dword ptr ss:\[ebp+42393C\]   
00481B0D     9C                   pushfd   
00481B0E     EB 01                jmp short 00481B11   
  
00481B11     9D                   popfd   
00481B12     EB 01                jmp short 00481B15   
  
00481B15     8BAD 48394200        mov ebp,dword ptr ss:\[ebp+423948\]   
00481B1B     60                   pushad   
00481B1C     E8 03000000          call 00481B24   
  
00481B24     EB 01                jmp short 00481B27   
  
00481B27     58                   pop eax    
00481B28     EB 01                jmp short 00481B2B   
  
00481B2B     40                   inc eax   
00481B2C     EB 01                jmp short 00481B2F   
  
00481B2F     FFE0                 jmp eax   
  
00481B22     EB 0E                jmp short 00481B32   
  
00481B32     61                   popad   
00481B33     C3                   retn   
                                  ====>飞向光明之巅！ 返回至 00425C21   
  
  
————————————————————————   
  
00425C21     55                   push ebp   
                                  ====>在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程   
00425C22     8BEC                 mov ebp,esp   
00425C24     6A FF                push -1   
00425C26     68 A8CF4200          push 42CFA8   
00425C2B     68 289B4200          push 429B28   
00425C30     64:A1 00000000       mov eax,dword ptr fs:\[0\]   
00425C36     50                   push eax   
00425C37     64:8925 00000000     mov dword ptr fs:\[0\],esp   
00425C3E     83EC 58              sub esp,58   
00425C41     53                   push ebx   
00425C42     56                   push esi   
00425C43     57                   push edi   
00425C44     8965 E8              mov dword ptr ss:\[ebp\-18\],esp   
00425C47     FF15 08C14200        call dword ptr ds:\[42C108\]   ; kernel32.GetVersion   
  
  
———————————————————————   
  
运行 ImportREC，选择这个进程。把 OEP 改为 00025C21，点 IT AutoSearch，点“Get Import”，有 2 个函数无效。奇怪，不能用“追踪层次 2、3”修复，否则 PC Guard 自动退出！如果 CUT 掉则修复的程序可以运行，但是退出时会非法操作。呵呵，试着将其改为：kernel32.dll ExitProcess   FixDump，正常运行！VC++ 6.0 编写，576K->612K 重建 PE 后是 564K   
  
  
—————————————————————————————————   
  
                   ☆☆☆☆☆☆☆☆☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
☆☆☆☆☆☆☆☆☆☆ 再简化一下上面的脱壳步骤，呵呵，能省就省吧！:-) ☆☆☆☆☆☆☆☆☆☆   
                   ☆☆☆☆☆☆☆☆☆ ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
  
  
设置 Ollydbg 忽略除了“单步中断”之外的所有异常。   
  
00483000     FC                   cld   
                                  ====>进入 OD 后断在这！F9 运行，断下！   
  
004836D8     EB 01                jmp short PCGWIN32.004836DB   
                                  ====>第 1 个单步异常   
  
SHIFT+F9 通过异常，不用记下次数，呵呵，直到程序弹出一系列的“入口点预警”就停下！   
把那些对话框一一确定之。呵呵，看看偶现在停在什么地方？  :-)   
  
0048198F     EB 01                jmp short 00481992   
                                  ====>停在这个单步异常的地方   
  
看看堆栈：   
0012FF24    0012FFE0   指针到下一个 SEH 记录   
0012FF28    0048194B   SE 句柄 //呵呵，这是什么？    
0048194B 就是 0048198E  int1 异常的堆栈区的第二条地址！！:-) 又快了点，接下来就可以找 OEP 了！   
  
  
—————————————————————————————————   
  
以上的脱壳过程是针对 PC Guard for Win32 V5.0 DEMO 版的 PCGWIN32.EXE 主程序，而不是那种没有 KEY 就无法运行的保护类型的加壳程序。偶也简单测试了一下，以上寻找 OEP 的过程同样适用于用这个版本加壳的其他同类程序！当然有些地方是不可能完全一样的，有兴趣的朋友可以自己测试。   
  
这个帖子是偶在 看雪论坛 的第 1000 个帖子，谨以此篇简单的脱壳笔记留个简单的纪念。:-)   
  
  
—————————————————————————————————   
       
                                   
         ,     \_/    
        /| \_.-~/            \_     ,        青春都一饷   
       ( /~   /              ~-.\_ |   
       \`\\  \_/                   ~ )          忍把浮名    
   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`    
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~      
                         "     "   "~'  ""   
  
       
  
            Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]   
  
                    2003-10-26 00:00   
 4dd466d0e1e0cc123afbe504da2462ca