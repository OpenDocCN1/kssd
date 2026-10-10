

# 幻影 v1.5b3 脱壳分析笔记之二

> 原文：[`www.pediy.com/kssd/pediy03/crack32.htm`](https://www.pediy.com/kssd/pediy03/crack32.htm)

幻影 v1.5b3 脱壳分析笔记之二  
  
\[作者\]:    ljttt  
\[日期\]:    2000-09-15  
  
 以上是一段被我处理掉的代码，从这里开始分析  
   
 00013381: 5E                        pop esi   
 00013382: E800000000                call 00013387     <---JMP  
 00013387: 5D                        pop ebp       <---获得 EBP = 413387  
 00013388: 8BD5                      mov edx,ebp   
 \*\*\*\*\*\*\*\*\*\*\*说明以后的一段代码常以 EBP 做为基地址\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 0001338A: 81EDD4A64000              sub ebp,0040A6D4   <---EBP=413387-40A6D4 = 8CB3  
  
 00013390: FA                        cli   
 00013391: 58                        pop eax   
 00013392: 80E401                    and ah,01   
 00013395: 32C0                      xor al,al   
 00013397: BE09A74000                mov esi,0040A709   
 0001339C: 03F5                      add esi,ebp   
 0001339E: B946160000                mov ecx,00001646   
 000133A3: 03F1                      add esi,ecx   
 000133A5: 4E                        dec esi   
 000133A6: 90                        nop   
 000133A7: 90                        nop   
 000133A8: 90                        nop   
 000133A9: 90                        nop   
 000133AA: 90                        nop   
 000133AB: 90                        nop   
 000133AC: 90                        nop   
 000133AD: 90                        nop   
 000133AE: 90                        nop   
 000133AF: 90                        nop   
 000133B0: FA                        cli   
 000133B1: 6681BD09A740009090        cmp word ptr \[ebp+0040A709\],9090   
 000133BA: 75FE                      jnz 000133BA   
 000133BC: 90                        nop   
 000133BD: 90                        nop   
 000133BE: 90                        nop   
 000133BF: 2B95CBB24000              sub edx,dword ptr \[ebp+0040B2CB\]   
 000133C5: 81EA87530000              sub edx,00005387   
 000133CB: 8995D3B24000              mov dword ptr \[ebp+0040B2D3\],edx   
\*\*\*\*\*\*\*\*此处调用加载各个函数库\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* ID:CALL000146AD  
 000133D1: E8D7120000                call 000146AD             <---加载函数库  
 000133D6: 83F800                    cmp eax,00000000   
 000133D9: 7520                      jnz 000133FB             <---成功跳走  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*显示 Dial.dll 加载失败，并退出程序  
 000133DB: B8C0B94000                mov eax,0040B9C0   
 000133E0: 03C5                      add eax,ebp   
 000133E2: BBC2B94000                mov ebx,0040B9C2   
 000133E7: 03DD                      add ebx,ebp   
 000133E9: 6A30                      push 00000030   
 000133EB: 50                        push eax   
 000133EC: 53                        push ebx   
 000133ED: 6A00                      push 00000000   
 000133EF: FF9500BC4000              call dword ptr \[ebp+0040BC00\]     <---MessageBoxA  
 000133F5: FF956CBF4000              call dword ptr \[ebp+0040BF6C\]     <---ExitProcess  
                                                  
                                                 
 000133FB: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   <---跳到此处  
\*\*\*\*\*\*\*\*申请内存\*\*\*\*\*\*\*\*\*\*  
 00013401: 83C120                    add ecx,00000020   
 00013404: 51                        push ecx   
 00013405: 6A40                      push 00000040   
 00013407: 6800201000                push 00102000   
 0001340C: 51                        push ecx   
 0001340D: 6A00                      push 00000000   
 0001340F: FF95CBBB4000              call dword ptr \[ebp+0040BBCB\]     <---VirtualAlloc  
 00013415: 59                        pop ecx   
\*\*\*\*\*\*\*\*\*\*\*\*\*申请内存\*\*\*\*\*\*\*\*\*\*  
 00013416: 6A40                      push 00000040   
 00013418: 6800101000                push 00101000   
 0001341D: 51                        push ecx   
 0001341E: 50                        push eax   
 0001341F: FF95CBBB4000              call dword ptr \[ebp+0040BBCB\]     <---VirtualAlloc  
 00013425: 8BD0                      mov edx,eax   
 00013427: 8BF8                      mov edi,eax   
\*\*\*\*\*\*\*\*\*\*\*\*搬家\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013429: BE4D534000                mov esi,0040534D   
 0001342E: 03F5                      add esi,ebp   
 00013430: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   
 00013436: F3A4                      repe movsb es:\[edi\],ds:\[esi\]     <----复制到新位置  
 00013438: B893A74000                mov eax,0040A793   
 0001343D: 2D4D534000                sub eax,0040534D   
 00013442: 03C2                      add eax,edx   
 00013444: 50                        push eax   
 00013445: C3                        ret            <----到新地址执行下面的语句  
  
\*\*\*\*\*\*\*把原来地方的代码清零\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013446: BF4D534000                mov edi,0040534D   
 0001344B: 03FD                      add edi,ebp   
 0001344D: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   
 00013453: 32C0                      xor al,al   
 00013455: F3AA                      repe stosb es:\[edi\],al   <----把原来的位置的内容清零  
\*\*\*\*\*\*\*取新的 EBP 值，以后许多地方以 EBP 为基地址\*\*\*\*\*\*\*\*\*  
 00013457: E800000000                call 0001345C         <---JMP 语句  
 0001345C: 5D                        pop ebp           <---EBP = 41345C  
 0001345D: 81EDA9A74000              sub ebp,0040A7A9       <---EBP = 41345C-40A7A9 = 8CB3  
\*\*\*\*\*\*\*\*\*\*\*\*以下是一段检测跟踪的代码\*\*\*\*\*\*\*\*\*\*\*\*  
 00013463: BEDDA74000                mov esi,0040A7DD   
 00013468: 03F5                      add esi,ebp   
 0001346A: 6A00                      push 00000000   
 0001346C: 6A00                      push 00000000   
 0001346E: 6A00                      push 00000000   
 00013470: 6A00                      push 00000000   
 00013472: 6A00                      push 00000000   
 00013474: 6A00                      push 00000000   
 00013476: 56                        push esi   
 00013477: FF95A3BB4000              call dword ptr \[ebp+0040BBA3\]  <----CreateFileA  
 0001347D: 83F8FF                    cmp eax,FFFFFFFF   
 00013480: 7568                      jnz 000134EA           <----判断是否发现跟踪  
 00013482: 46                        inc esi   
 00013483: 803E00                    cmp byte ptr \[esi\],00   
 00013486: 75FA                      jnz 00013482   
 00013488: 46                        inc esi   
 00013489: 803E00                    cmp byte ptr \[esi\],00   
 0001348C: 75DC                      jnz 0001346A   
 0001348E: EB5A                      jmp 000134EA           <----跳走  
   
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*这里不是代码区\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013490: 5C                        pop esp   
 00013491: 5C                        pop esp   
 00013492: 2E5C                      pop esp   
 00013494: 54                        push esp   
 00013495: 52                        push edx   
 00013496: 57                        push edi   
 00013497: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 0001349B: 5C                        pop esp   
 0001349C: 54                        push esp   
 0001349D: 52                        push edx   
 0001349E: 57                        push edi   
 0001349F: 3230                      xor dh,byte ptr \[eax\]   
 000134A1: 3030                      xor byte ptr \[eax\],dh   
 000134A3: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134A7: 5C                        pop esp   
 000134A8: 54                        push esp   
 000134A9: 52                        push edx   
 000134AA: 57                        push edi   
 000134AB: 44                        inc esp   
 000134AC: 45                        inc ebp   
 000134AD: 42                        inc edx   
 000134AE: 55                        push ebp   
 000134AF: 47                        inc edi   
 000134B0: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134B4: 5C                        pop esp   
 000134B5: 52                        push edx   
 000134B6: 45                        inc ebp   
 000134B7: 47                        inc edi   
 000134B8: 56                        push esi   
 000134B9: 58                        pop eax   
 000134BA: 44                        inc esp   
 000134BB: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134BF: 5C                        pop esp   
 000134C0: 4E                        dec esi   
 000134C1: 54                        push esp   
 000134C2: 49                        dec ecx   
 000134C3: 43                        inc ebx   
 000134C4: 45                        inc ebp   
 000134C5: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134C9: 5C                        pop esp   
 000134CA: 53                        push ebx   
 000134CB: 49                        dec ecx   
 000134CC: 43                        inc ebx   
 000134CD: 45                        inc ebp   
 000134CE: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134D2: 5C                        pop esp   
 000134D3: 56                        push esi   
 000134D4: 4B                        dec ebx   
 000134D5: 45                        inc ebp   
 000134D6: 59                        pop ecx   
 000134D7: 50                        push eax   
 000134D8: 52                        push edx   
 000134D9: 4F                        dec edi   
 000134DA: 44                        inc esp   
 000134DB: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 000134DF: 5C                        pop esp   
 000134E0: 46                        inc esi   
 000134E1: 49                        dec ecx   
 000134E2: 4C                        dec esp   
 000134E3: 45                        inc ebp   
 000134E4: 56                        push esi   
 000134E5: 58                        pop eax   
 000134E6: 44                        inc esp   
 000134E7: 0000                      add byte ptr \[eax\],al   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*非代码区\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*判断检测的结果\*\*\*\*\*\*\*\*\*\*\*\*\*   
 000134E9: 0083F8FF7406              add byte ptr \[ebx+0674FFF8\],al   <---跳转，不跳就死定了  
\*\*\*\*\*\*\*\*\*\*\*退出程序  
 000134EF: FF956CBF4000              call dword ptr \[ebp+0040BF6C\]     <---ExitProcess  
  
\*\*\*\*\*\*\*\*\*\*\*这是一段反跟踪的代码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
 000134F5: 60                        pushad                <---跳到此处  
 000134F6: BE53BD4000                mov esi,0040BD53            
 000134FB: 03F5                      add esi,ebp   
 000134FD: 0F010E                    sidt dword ptr \[esi\]        <---获取 IDTR 内容   
 00013500: 8B7602                    mov esi,dword ptr \[esi+02\]     <---获取 IDT 表的基地址  
 00013503: 668B4608                  mov ax,word ptr \[esi+08\]       <---取 int 1 的低位偏移  
 00013507: 66052302                  add ax,0223             <---计算新值  
 0001350B: 66894608                  mov word ptr \[esi+08\],ax       <---保存 int 1 新偏移值  
 0001350F: 668B460E                  mov ax,word ptr \[esi+0E\]       <---取 int 1 的高位偏移  
 00013513: 668B5E06                  mov bx,word ptr \[esi+06\]       <---取 int 0 的高位偏移  
 00013517: 663BC3                    cmp ax,bx               <---是否相等  
 0001351A: 740E                      jz 0001352A             <---相等则跳转  
 0001351C: 668B4608                  mov ax,word ptr \[esi+08\]       <---取 int 1 的低位偏移  
 00013520: 668B5E18                  mov bx,word ptr \[esi+18\]       <---取 int 3 的低位偏移  
 00013524: FA                        cli   
 00013525: 663BC3                    cmp ax,bx               <---判断是否相等  
 00013528: 7400                      jz 0001352A             <---相等则跳转  
 0001352A: 668B4608                  mov ax,word ptr \[esi+08\]       <---取 int 1 的低位偏移  
 0001352E: 668B5E0E                  mov bx,word ptr \[esi+0E\]       <---取 int 1 的高位偏移  
 00013532: 66898567BE4000            mov word ptr \[ebp+0040BE67\],ax   <---保存  
 00013539: 66899D69BE4000            mov word ptr \[ebp+0040BE69\],bx   <---保存  
 00013540: 668B4618                  mov ax,word ptr \[esi+18\]       <---取 int 3 的低位偏移  
 00013544: 668B5E1E                  mov bx,word ptr \[esi+1E\]       <---取 int 3 的高位偏移  
 00013548: 6689856BBE4000            mov word ptr \[ebp+0040BE6B\],ax   <---保存  
 0001354F: 66899D6DBE4000            mov word ptr \[ebp+0040BE6D\],bx   <---保存  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*保存 EBP 的值  
 00013556: 89AD65BD4000              mov dword ptr \[ebp+0040BD65\],ebp   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*保存 ReadProcessMemory 的首地址  
 0001355C: 8B85C7BB4000              mov eax,dword ptr \[ebp+0040BBC7\]   <---取 ReadProcessMemory 的首地址  
 00013562: 89856FBE4000              mov dword ptr \[ebp+0040BE6F\],eax   <---保存  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*这里是一段反跟踪代码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013568: E8A9140000                call 00014A16             <---这里又是一段 SIDT 代码  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*开始分析注册情况\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
 0001356D: BB63BD4000                mov ebx,0040BD63   
 00013572: 03DD                      add ebx,ebp   
 00013574: 53                        push ebx   
 00013575: 6A64                      push 00000064   
 00013577: 6A63                      push 00000063   
 00013579: 6A00                      push 00000000   
 0001357B: FF9508BC4000              call dword ptr \[ebp+0040BC08\]    <---SetTimer   
 00013581: 61                        popad    
 00013582: 6A00                      push 00000000   
 00013584: FF95C3BB4000              call dword ptr \[ebp+0040BBC3\]     <---GetModuleHandle  
 0001358A: 89855FBD4000              mov dword ptr \[ebp+0040BD5F\],eax   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*判断是否用的是注册版的幻影加密\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013590: 80BDC8B2400001            cmp byte ptr \[ebp+0040B2C8\],01    <---是否注册标志   
 00013597: 7435                      jz 000135CE             <---相等，注册版则跳走  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*用系统时间随机显示未注册的 NAG 窗口\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00013599: BEE1B94000                mov esi,0040B9E1   
 0001359E: 03F5                      add esi,ebp   
 000135A0: 56                        push esi   
 000135A1: FF959FBB4000              call dword ptr \[ebp+0040BB9F\]     <---GetSystemTime  
 000135A7: 668B85EFB94000            mov ax,word ptr \[ebp+0040B9EF\]   
 000135AE: 240F                      and al,0F   
 000135B0: 3C02                      cmp al,02   
 000135B2: 731A                      jnb 000135CE            <---结果随机，跳转  
 000135B4: B8BEB94000                mov eax,0040B9BE           <---到此，将显示 NAG 窗口  
 000135B9: 03C5                      add eax,ebp                        <--- v1.5 This Program  
 000135BB: BB11B24000                mov ebx,0040B211   
 000135C0: 03DD                      add ebx,ebp   
 000135C2: 6A30                      push 00000030   
 000135C4: 50                        push eax   
 000135C5: 53                        push ebx   
 000135C6: 6A00                      push 00000000   
\*\*\*\*\*\*调用 MessageBoxA 显示 NAG 窗口\*\*\*\*\*\*\*  
 000135C8: FF9500BC4000              call dword ptr \[ebp+0040BC00\]     <---MessageBoxA  
\*\*\*\*\*\*\*\*\*\*\*\* 跳到此处\*\*\*\*\*\*\*\*开始处理用户加密的信息  
 000135CE: E862060000                call 00013C35             <---从注册表获取信息  
 000135D3: 663D0100                  cmp ax,0001   
 000135D7: 7416                      jz 000135EF     <---成功就跳走  
\*\*\*\*\*\*\*\*\*没注意此段内容\*\*\*\*\*\*\*\*\*\*\*\*\*   
 000135D9: BE60B14000                mov esi,0040B160   
 000135DE: 03F5                      add esi,ebp   
 000135E0: 668B8589B44000            mov ax,word ptr \[ebp+0040B489\]   
 000135E7: 668906                    mov word ptr \[esi\],ax   
 000135EA: E802070000                call 00013CF1   
\*\*\*\*\*\*\*\*对用户注册信息的处理\*\*\*\*\*\*\*\*\*\*\*\*  
 000135EF: E87C030000                call 00013970             <---有磁盘和卷的调用  
 000135F4: 6689857DB44000            mov word ptr \[ebp+0040B47D\],ax   <--此处是一个标志  
 000135FB: 663D0100                  cmp ax,0001   
 000135FF: 0F84BF000000              jz 000136C4       <--成功，跳走  
 \*\*\*\*\*\*\*\*判断标志，显示用户加的 NAG 提示信息窗口\*\*\*\*\*\*\*\*\*\*\*\*  
 00013605: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000   
 0001360C: 7426                      jz 00013634   
 0001360E: 83BD61B7400001            cmp dword ptr \[ebp+0040B761\],00000001 <--判断标志  
 00013615: 751D                      jnz 00013634       <--不等，就跳走  
 00013617: E80A060000                call 00013C26       <--显示用户加的 NAG 提示信息窗口,1.5beta2 Email: ding-boy  
 0001361C: 6689857FB44000            mov word ptr \[ebp+0040B47F\],ax   
 00013623: BBC9B24000                mov ebx,0040B2C9   
 00013628: 03DD                      add ebx,ebp   
 0001362A: 663D0200                  cmp ax,0002   
 0001362E: 0F8444010000              jz 00013778   
\*\*\*\*\*\*\*\*\*\*\*判断是否是缺省 NAG 提示信息\*\*\*\*\*\*\*\*\*\*\*\*  
 00013634: 6681BDF1B440007C7C        cmp word ptr \[ebp+0040B4F1\],7C7C   
 0001363D: 741A                      jz 00013659    <--相等，就跳走   
 0001363F: B8BEB94000                mov eax,0040B9BE   
 00013644: 03C5                      add eax,ebp   
 00013646: BBF1B44000                mov ebx,0040B4F1   
 0001364B: 03DD                      add ebx,ebp   
 0001364D: 6A30                      push 00000030   
 0001364F: 50                        push eax   
 00013650: 2020                      and byte ptr \[eax\],ah   
 00013652: 2020                      and byte ptr \[eax\],ah   
 00013654: 2038                      and byte ptr \[eax\],bh   
 \*\*\*\*\*\*\*\*\*显示用户加的 NAG 提示信息窗口\*\*\*\*\*\*\*\*\*\*\*  
 00013656: 3530343400                xor eax,00343430   
 0001365B: 0000                      add byte ptr \[eax\],al   
 0001365D: 0000                      add byte ptr \[eax\],al   
 0001365F: 0000                      add byte ptr \[eax\],al   
 00013640: BEB9400003                mov esi,030040B9   
 00013645: C5BBF1B44000              lds edi,pword ptr \[ebx+0040B4F1\]   
 0001364B: 03DD                      add ebx,ebp   
 0001364D: 6A30                      push 00000030   
 0001364F: 50                        push eax   
 00013650: 53                        push ebx   
 00013651: 6A00                      push 00000000           <--欢迎 ,^-^  
 00013653: FF9500BC4000              call dword ptr \[ebp+0040BC00\] <--MessageBoxA  
   
 00013659: E8A0020000                call 000138FE   <--跳到此处  
 0001365E: BB91B84000                mov ebx,0040B891   
 00013663: 03DD                      add ebx,ebp   
 00013665: 663D0000                  cmp ax,0000   
 00013669: 0F8409010000              jz 00013778   <--等于 0，就跳走  
 0001366F: E882010000                call 000137F6   <--获取系统时间  
 00013674: BB1DB64000                mov ebx,0040B61D   
 00013679: 03DD                      add ebx,ebp   
 0001367B: 663D0000                  cmp ax,0000   
 0001367F: 0F84F3000000              jz 00013778   
 00013685: E8C3010000                call 0001384D   <--注册表和磁盘  
 0001368A: BB1DB64000                mov ebx,0040B61D   
 0001368F: 03DD                      add ebx,ebp   
 00013691: 663D0000                  cmp ax,0000   
 00013695: 0F84DD000000              jz 00013778   
 0001369B: 6683BD7DB4400000          cmp word ptr \[ebp+0040B47D\],0000   
 000136A3: 751F                      jnz 000136C4   
 000136A5: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000   
 000136AC: 7416                      jz 000136C4   
 000136AE: 6683BD7FB4400001          cmp word ptr \[ebp+0040B47F\],0001   
 000136B6: 740C                      jz 000136C4   
 000136B8: BBC9B24000                mov ebx,0040B2C9   
 000136BD: 03DD                      add ebx,ebp   
 000136BF: E9B4000000                jmp 00013778   
   
 000136C4: BED7B24000                mov esi,0040B2D7   
 000136C9: 03F5                      add esi,ebp     
   
 000136CB: 833E00                    cmp dword ptr \[esi\],00000000   <---此处循环将还原各 Section  
 000136CE: 746F                      jz 0001373F               <--是否循环结束  
 000136D0: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]   
 000136D6: 031E                      add ebx,dword ptr \[esi\]   
 000136D8: 8B4E04                    mov ecx,dword ptr \[esi+04\]   
 000136DB: 83F900                    cmp ecx,00000000   
 000136DE: 7505                      jnz 000136E5   
 000136E0: 83C608                    add esi,00000008   
 000136E3: EBE6                      jmp 000136CB   
 000136E5: D1E9                      shr ecx,1   
 000136E7: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 000136EE: 66F7D0                    not ax   
   
 000136F1: 663103                    xor word ptr \[ebx\],ax   
 000136F4: 6648                      dec ax   
 000136F6: 66813B3F3F                cmp word ptr \[ebx\],3F3F   
 000136FB: 7518                      jnz 00013715   
 000136FD: 817BFC44425045            cmp dword ptr \[ebx-04\],45504244   
 00013704: 750F                      jnz 00013715   
 00013706: 6683BD7DB4400001          cmp word ptr \[ebp+0040B47D\],0001   
 0001370E: 7505                      jnz 00013715   
 00013710: 66C7032B2B                mov word ptr \[ebx\],2B2B   
 00013715: 66817BFF3F3F              cmp word ptr \[ebx-01\],3F3F   
 0001371B: 7519                      jnz 00013736   
 0001371D: 817BFB44425045            cmp dword ptr \[ebx-05\],45504244   
 00013724: 7510                      jnz 00013736   
 00013726: 6683BD7DB4400001          cmp word ptr \[ebp+0040B47D\],0001   
 0001372E: 7506                      jnz 00013736   
 00013730: 66C743FF2B2B              mov word ptr \[ebx-01\],2B2B   
 00013736: 43                        inc ebx   
 00013737: 43                        inc ebx   
 00013738: E2B7                      loop 000136F1   
   
 0001373A: 83C608                    add esi,00000008   
 0001373D: EB8C                      jmp 000136CB              <---大循环    
   
 0001373F: 90                        nop   
 00013740: 89AD9FBE4000              mov dword ptr \[ebp+0040BE9F\],ebp   
 00013746: E84C030000                call 00013A97   
 0001374B: E82D040000                call 00013B7D   
 00013750: 8B85CFB24000              mov eax,dword ptr \[ebp+0040B2CF\]   
 00013756: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]   
 0001375C: 03C3                      add eax,ebx   
 0001375E: 8985F6B94000              mov dword ptr \[ebp+0040B9F6\],eax   
 00013764: 8BC5                      mov eax,ebp   
 00013766: 5B                        pop ebx   
 00013767: 59                        pop ecx   
 00013768: 5A                        pop edx   
 00013769: 5E                        pop esi   
 0001376A: 5F                        pop edi   
 0001376B: 5D                        pop ebp   
 0001376C: 9D                        popfd    
 0001376D: FFB0F6B94000              push dword ptr \[eax+0040B9F6\]   
 00013773: E90F120000                jmp 00014987         <---跳到后面处理入口处的代码  
   
 00013778: 66813B7C7C                cmp word ptr \[ebx\],7C7C   
 0001377D: 7413                      jz 00013792   
 0001377F: B8C0B94000                mov eax,0040B9C0   
 00013784: 03C5                      add eax,ebp   
 00013786: 6A30                      push 00000030   
 00013788: 50                        push eax   
 00013789: 53                        push ebx   
 0001378A: 6A00                      push 00000000             <---显示 dial.dll 没有找到  
 0001378C: FF9500BC4000              call dword ptr \[ebp+0040BC00\]   <--MessageBoxA  
 00013792: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000   
 00013799: 7446                      jz 000137E1   
 0001379B: E810010000                call 000138B0   
 000137A0: 6689857DB44000            mov word ptr \[ebp+0040B47D\],ax   
 000137A7: 50                        push eax   
 000137A8: 663D0100                  cmp ax,0001   
 000137AC: 7507                      jnz 000137B5   
 000137AE: BBF5B84000                mov ebx,0040B8F5   
 000137B3: EB05                      jmp 000137BA   
 000137B5: BB59B94000                mov ebx,0040B959   
 000137BA: 03DD                      add ebx,ebp   
 000137BC: 66813B7C7C                cmp word ptr \[ebx\],7C7C   
 000137C1: 7413                      jz 000137D6   
 000137C3: B8C0B94000                mov eax,0040B9C0   
 000137C8: 03C5                      add eax,ebp   
 000137CA: 6A30                      push 00000030   
 000137CC: 50                        push eax   
 000137CD: 53                        push ebx   
 000137CE: 6A00                      push 00000000   
 000137D0: FF9500BC4000              call dword ptr \[ebp+0040BC00\]   <--MessageBoxA  
 000137D6: 58                        pop eax   
 000137D7: 663D0100                  cmp ax,0001   
 000137DB: 0F84E3FEFFFF              jz 000136C4   
 000137E1: C6855DBD400000            mov byte ptr \[ebp+0040BD5D\],00   
 000137E8: 8BC5                      mov eax,ebp   
 000137EA: 5B                        pop ebx   
 000137EB: 59                        pop ecx   
 000137EC: 5A                        pop edx   
 000137ED: 5E                        pop esi   
 000137EE: 5F                        pop edi   
 000137EF: 5D                        pop ebp   
 000137F0: 9D                        popfd    
 000137F1: E991110000                jmp 00014987   
  
\-------子程序开始-------   
\*\*\*\*\*\*\*\*获取系统时间，不知道要干什么，不会是比较时间差来判断是否被 SOFTICE 跟踪吧\*\*\*\*\*\*\*\*\*  
 000137F6: BEE1B94000                mov esi,0040B9E1   
 000137FB: 03F5                      add esi,ebp   
 000137FD: 56                        push esi   
 000137FE: FF959FBB4000              call dword ptr \[ebp+0040BB9F\]   <---GetSystemTime  
 00013804: 668B8585B44000            mov ax,word ptr \[ebp+0040B485\]   
 0001380B: 668B9D81B44000            mov bx,word ptr \[ebp+0040B481\]   
 00013812: 663D0000                  cmp ax,0000   
 00013816: 7430                      jz 00013848   
 00013818: 663985E3B94000            cmp word ptr \[ebp+0040B9E3\],ax   
 0001381F: 7209                      jb 0001382A   
 00013821: 66399DE1B94000            cmp word ptr \[ebp+0040B9E1\],bx   
 00013828: 7309                      jnb 00013833   
 0001382A: 66399DE1B94000            cmp word ptr \[ebp+0040B9E1\],bx   
 00013831: 7606                      jbe 00013839   
 00013833: 66B80000                  mov ax,0000   
 00013837: EB0D                      jmp 00013846   
 00013839: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001   
 00013842: 66B80100                  mov ax,0001   
 00013846: EB04                      jmp 0001384C   
 00013848: 66B80100                  mov ax,0001   
 0001384C: C3                        ret   
 --------子程序结束----------  
  
\---------子程序开始-------------   
 0001384D: 60                        pushad    
 0001384E: 6683BD89B4400000          cmp word ptr \[ebp+0040B489\],0000   
 00013856: 750B                      jnz 00013863   
 00013858: 66C785FBAB40000100        mov word ptr \[ebp+0040ABFB\],0001   
 00013861: EB42                      jmp 000138A5   
 00013863: E8CD030000                call 00013C35   <--从注册表中获取信息  
 00013868: BE60B14000                mov esi,0040B160   
 0001386D: 03F5                      add esi,ebp   
 0001386F: 663D0100                  cmp ax,0001   
 00013873: 7405                      jz 0001387A   
 00013875: 66C7060000                mov word ptr \[esi\],0000   
 0001387A: 66833E00                  cmp word ptr \[esi\],0000   
 0001387E: 7417                      jz 00013897   
 00013880: 66FF0E                    dec word ptr \[esi\]   
 00013883: 66C785FBAB40000100        mov word ptr \[ebp+0040ABFB\],0001   
 0001388C: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001   
 00013895: EB09                      jmp 000138A0   
 00013897: 66C785FBAB40000000        mov word ptr \[ebp+0040ABFB\],0000   
 000138A0: E84C040000                call 00013CF1  <--获取磁盘和卷的信息   
 000138A5: 61                        popad    
 000138A6: 668B85FBAB4000            mov ax,word ptr \[ebp+0040ABFB\]   
 000138AD: C3                        ret   
  
 000138AE: 0000                      add byte ptr \[eax\],al   
 000138B0: E8BB000000                call 00013970   
 000138B5: 8B85A8AD4000              mov eax,dword ptr \[ebp+0040ADA8\]   
 000138BB: BBB0AD4000                mov ebx,0040ADB0   
 000138C0: 03DD                      add ebx,ebp   
 000138C2: B983B14000                mov ecx,0040B183   
 000138C7: 03CD                      add ecx,ebp   
 000138C9: 50                        push eax   
 000138CA: 53                        push ebx   
 000138CB: 51                        push ecx   
 000138CC: FF9504BC4000              call dword ptr \[ebp+0040BC04\]   
 000138D2: 83C40C                    add esp,0000000C   
 000138D5: B865B14000                mov eax,0040B165   
 000138DA: 03C5                      add eax,ebp   
 000138DC: BB79B14000                mov ebx,0040B179   
 000138E1: 03DD                      add ebx,ebp   
 000138E3: B983B14000                mov ecx,0040B183   
 000138E8: 03CD                      add ecx,ebp   
 000138EA: 51                        push ecx   
 000138EB: 53                        push ebx   
 000138EC: 50                        push eax   
 000138ED: FF95BCBC4000              call dword ptr \[ebp+0040BCBC\]   
 000138F3: E8F9030000                call 00013CF1   
 000138F8: E873000000                call 00013970   
 000138FD: C3                        ret   
   
 ------子程序------  
 000138FE: 6683BD49B740007C          cmp word ptr \[ebp+0040B749\],007C   
 00013906: 7505                      jnz 0001390D   
 00013908: 66B80100                  mov ax,0001   
 0001390C: C3                        ret   
   
 0001390D: B89FAC4000                mov eax,0040AC9F   
 00013912: 03C5                      add eax,ebp   
 00013914: 50                        push eax   
 00013915: FF95B8BC4000              call dword ptr \[ebp+0040BCB8\]   
 0001391B: BE49B74000                mov esi,0040B749   
 00013920: 03F5                      add esi,ebp   
 00013922: BF9FAC4000                mov edi,0040AC9F   
 00013927: 03FD                      add edi,ebp   
 00013929: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 00013930: 2020                      and byte ptr \[eax\],ah   
 00013932: 2020                      and byte ptr \[eax\],ah   
 00013934: 2038                      and byte ptr \[eax\],bh   
 00013936: 3530343400                xor eax,00343430   
 0001393B: 0000                      add byte ptr \[eax\],al   
 0001393D: 0000                      add byte ptr \[eax\],al   
 0001393F: 0000                      add byte ptr \[eax\],al   
 00013920: 03F5                      add esi,ebp   
 00013922: BF9FAC4000                mov edi,0040AC9F   
 00013927: 03FD                      add edi,ebp   
 00013929: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 00013930: 8A26                      mov ah,byte ptr \[esi\]   
 00013932: 3A27                      cmp ah,byte ptr \[edi\]   
 00013934: 7509                      jnz 0001393F   
 00013936: 80FC00                    cmp ah,00   
 00013939: 7409                      jz 00013944   
 0001393B: 46                        inc esi   
 0001393C: 47                        inc edi   
 0001393D: EBF1                      jmp 00013930   
 0001393F: 66B80000                  mov ax,0000   
 00013943: C3                        ret   
 00013944: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001   
 0001394D: 66B80100                  mov ax,0001   
 00013951: C3                        ret   
 ------子程序------------  
  
\*\*\*\*\*\*非代码区\*\*\*\*\*\*\*\*\*   
 00013952: 0000                      add byte ptr \[eax\],al   
 00013954: 0000                      add byte ptr \[eax\],al   
 00013956: 0000                      add byte ptr \[eax\],al   
 00013958: 0000                      add byte ptr \[eax\],al   
 0001395A: 0000                      add byte ptr \[eax\],al   
 0001395C: 0000                      add byte ptr \[eax\],al   
 0001395E: 0000                      add byte ptr \[eax\],al   
 00013960: 0000                      add byte ptr \[eax\],al   
 00013962: 0000                      add byte ptr \[eax\],al   
 00013964: 0000                      add byte ptr \[eax\],al   
 00013966: 0000                      add byte ptr \[eax\],al   
 00013968: 0000                      add byte ptr \[eax\],al   
 0001396A: 0000                      add byte ptr \[eax\],al   
 0001396C: 0000                      add byte ptr \[eax\],al   
 0001396E: 0000                      add byte ptr \[eax\],al   
\*\*\*\*\*\*非代码区结束\*\*\*\*\*\*\*  
  
\----------子程序开始---------   
 \*\*\*\*\*\*\*\*获取 C 盘的卷信息\*\*\*\*\*\*\*\*\*\*  
 00013970: B810000000                mov eax,00000010   
 00013975: 50                        push eax   
 00013976: B8B8AD4000                mov eax,0040ADB8   
 0001397B: 03C5                      add eax,ebp   
 0001397D: 50                        push eax   
 0001397E: B8C8AD4000                mov eax,0040ADC8   
 00013983: 03C5                      add eax,ebp   
 00013985: 50                        push eax   
 00013986: B8CCAD4000                mov eax,0040ADCC   
 0001398B: 03C5                      add eax,ebp   
 0001398D: 50                        push eax   
 0001398E: B8D0AD4000                mov eax,0040ADD0   
 00013993: 03C5                      add eax,ebp   
 00013995: 50                        push eax   
 00013996: B810000000                mov eax,00000010   
 0001399B: 50                        push eax   
 0001399C: B8D4AD4000                mov eax,0040ADD4   
 000139A1: 03C5                      add eax,ebp   
 000139A3: 50                        push eax   
 000139A4: B8B4AD4000                mov eax,0040ADB4   
 000139A9: 03C5                      add eax,ebp   
 000139AB: 50                        push eax     <--"C:\\"  
 000139AC: FF95BFBB4000              call dword ptr \[ebp+0040BBBF\]   <--GetVolumeInformationA  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*获取 C 盘空余磁盘信息\*\*\*\*\*\*\*\*  
 000139B2: B8A4AD4000                mov eax,0040ADA4   
 000139B7: 03C5                      add eax,ebp   
 000139B9: BBA8AD4000                mov ebx,0040ADA8   
 000139BE: 03DD                      add ebx,ebp   
 000139C0: B9ACAD4000                mov ecx,0040ADAC   
 000139C5: 03CD                      add ecx,ebp   
 000139C7: C701633A5C00              mov dword ptr \[ecx\],005C3A63   
 000139CD: 50                        push eax   
 000139CE: 53                        push ebx   
 000139CF: 6A00                      push 00000000   
 000139D1: 51                        push ecx   
 000139D2: FF95B7BB4000              call dword ptr \[ebp+0040BBB7\]   <--GetDiskFreeSpaceExA  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*从注册表获取信息\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
 000139D8: 8B85A8AD4000              mov eax,dword ptr \[ebp+0040ADA8\]   
 000139DE: F7D0                      not eax   
 000139E0: 3385D0AD4000              xor eax,dword ptr \[ebp+0040ADD0\]   
 000139E6: 8985A8AD4000              mov dword ptr \[ebp+0040ADA8\],eax   
 000139EC: E844020000                call 00013C35     <--从注册表中获取信息  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*好象是还原信息\*\*\*\*\*\*\*\*   
 000139F1: B801000000                mov eax,00000001   
 000139F6: 33DB                      xor ebx,ebx   
 000139F8: BE65B14000                mov esi,0040B165   
 000139FD: 03F5                      add esi,ebp   
 000139FF: 8A1E                      mov bl,byte ptr \[esi\]   
 00013A01: 80FB00                    cmp bl,00   
 00013A04: 7405                      jz 00013A0B   
 00013A06: F7E3                      mul ebx   
 00013A08: 46                        inc esi   
 00013A09: EBF4                      jmp 000139FF   
 00013A0B: 33855DB74000              xor eax,dword ptr \[ebp+0040B75D\]   
 00013A11: 3385A8AD4000              xor eax,dword ptr \[ebp+0040ADA8\]   
 00013A17: 86E0                      xchg al,ah   
 00013A19: 8BD8                      mov ebx,eax   
 \*\*\*\*\*\*\*处理信息\*\*\*\*\*\*\*\*\*  
 00013A1B: BE79B14000                mov esi,0040B179   
 00013A20: 03F5                      add esi,ebp   
 00013A22: E80E000000                call 00013A35   <--处理信息  
 00013A27: 3BD8                      cmp ebx,eax   
 00013A29: 7405                      jz 00013A30   
 00013A2B: 66B80000                  mov ax,0000   
 00013A2F: C3                        ret   
 00013A30: 66B80100                  mov ax,0001   
 00013A34: C3                        ret   
\----------子程序结束-----------  
  
\-----------子程序开始----------   
 00013A35: 53                        push ebx   
 00013A36: 56                        push esi   
 00013A37: 33DB                      xor ebx,ebx   
 00013A39: 8A06                      mov al,byte ptr \[esi\]   
 00013A3B: 3C39                      cmp al,39   
 00013A3D: 7606                      jbe 00013A45   
 00013A3F: 24CF                      and al,CF   
 00013A41: 2C37                      sub al,37   
 00013A43: EB02                      jmp 00013A47   
   
 00013A45: 240F                      and al,0F   
 00013A47: C1E304                    shl ebx,04   
 00013A4A: 0AD8                      or bl,al   
 00013A4C: 46                        inc esi   
 00013A4D: 803E00                    cmp byte ptr \[esi\],00   
 00013A50: 75E7                      jnz 00013A39   
 00013A52: 8BC3                      mov eax,ebx   
 00013A54: 5E                        pop esi   
 00013A55: 5B                        pop ebx   
 00013A56: C3                        ret   
 ---------子程序结束-----------  
   
 00013A57: 0000                      add byte ptr \[eax\],al   
 00013A59: 0000                      add byte ptr \[eax\],al   
 00013A5B: 0000                      add byte ptr \[eax\],al   
 00013A5D: 0000                      add byte ptr \[eax\],al   
 00013A5F: 633A                      arpl word ptr \[edx\],di   
 00013A61: 5C                        pop esp   
 00013A62: 00256C580043              add byte ptr \[4300586C\],ah   
 00013A68: 3A5C0000                  cmp bl,byte ptr \[eax+eax\]   
 00013A6C: 0000                      add byte ptr \[eax\],al   
 00013A6E: 0000                      add byte ptr \[eax\],al   
 00013A70: 0000                      add byte ptr \[eax\],al   
 00013A72: 0000                      add byte ptr \[eax\],al   
 00013A74: 0000                      add byte ptr \[eax\],al   
 00013A76: 0000                      add byte ptr \[eax\],al   
 00013A78: 0000                      add byte ptr \[eax\],al   
 00013A7A: 0000                      add byte ptr \[eax\],al   
 00013A7C: 0000                      add byte ptr \[eax\],al   
 00013A7E: 0000                      add byte ptr \[eax\],al   
 00013A80: 0000                      add byte ptr \[eax\],al   
 00013A82: 0000                      add byte ptr \[eax\],al   
 00013A84: 0000                      add byte ptr \[eax\],al   
 00013A86: 0000                      add byte ptr \[eax\],al   
 00013A88: 0000                      add byte ptr \[eax\],al   
 00013A8A: 0000                      add byte ptr \[eax\],al   
 00013A8C: 0000                      add byte ptr \[eax\],al   
 00013A8E: 0000                      add byte ptr \[eax\],al   
 00013A90: 0000                      add byte ptr \[eax\],al   
 00013A92: 0000                      add byte ptr \[eax\],al   
 00013A94: 0000                      add byte ptr \[eax\],al   
   
 ----子程序----------  
 00013A96: 00608B                    add byte ptr \[eax-75\],ah   
 00013A99: 9D                        popfd    
 00013A9A: 69B4400083FB000F84A600    imul esi,dword ptr \[eax+eax\*2+00FB8300\],00A6840F   
 00013AA5: 0000                      add byte ptr \[eax\],al   
 00013AA7: 039DD3B24000              add ebx,dword ptr \[ebp+0040B2D3\]   
 00013AAD: 8B430C                    mov eax,dword ptr \[ebx+0C\]   
 00013AB0: 83F800                    cmp eax,00000000   
 00013AB3: 0F8494000000              jz 00013B4D   
 00013AB9: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00013ABF: 8BF0                      mov esi,eax   
 00013AC1: 50                        push eax   
 00013AC2: FF9564BF4000              call dword ptr \[ebp+0040BF64\]   <--LoadLibraryA  
 00013AC8: 89857DBA4000              mov dword ptr \[ebp+0040BA7D\],eax   
 00013ACE: 8B33                      mov esi,dword ptr \[ebx\]   
 00013AD0: 8B7B10                    mov edi,dword ptr \[ebx+10\]   
 00013AD3: 83FE00                    cmp esi,00000000   
 00013AD6: 7502                      jnz 00013ADA   
 00013AD8: 8BF7                      mov esi,edi   
 00013ADA: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
 00013AE0: 03BDD3B24000              add edi,dword ptr \[ebp+0040B2D3\]   
 00013AE6: 8B06                      mov eax,dword ptr \[esi\]   
 00013AE8: 83F800                    cmp eax,00000000   
 00013AEB: 7505                      jnz 00013AF2   
 00013AED: 83C314                    add ebx,00000014   
 00013AF0: EBBB                      jmp 00013AAD   
 00013AF2: 807E0380                  cmp byte ptr \[esi+03\],80   
 00013AF6: 7514                      jnz 00013B0C   
 00013AF8: 33C0                      xor eax,eax   
 00013AFA: 668706                    xchg word ptr \[esi\],ax   
 00013AFD: 50                        push eax   
 00013AFE: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00013B04: FF9568BF4000              call dword ptr \[ebp+0040BF68\]   
 00013B0A: EB37                      jmp 00013B43   
   
 00013B0C: 33C0                      xor eax,eax   
 00013B0E: 8706                      xchg dword ptr \[esi\],eax   
 00013B10: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00013B16: 83C002                    add eax,00000002   
 00013B19: 8BC8                      mov ecx,eax   
 00013B1B: E83B000000                call 00013B5B   
 00013B20: 663D0100                  cmp ax,0001   
 00013B24: 7510                      jnz 00013B36   
 00013B26: C6855DBD400001            mov byte ptr \[ebp+0040BD5D\],01   
 00013B2D: B89EBE4000                mov eax,0040BE9E   
 00013B32: 03C5                      add eax,ebp   
 00013B34: EB0D                      jmp 00013B43   
 00013B36: 50                        push eax   
 00013B37: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00013B3D: FF9568BF4000              call dword ptr \[ebp+0040BF68\]   <--GetProcAddress  
 00013B43: 8907                      mov dword ptr \[edi\],eax   
 00013B45: 83C604                    add esi,00000004   
 00013B48: 83C704                    add edi,00000004   
 00013B4B: EB99                      jmp 00013AE6   
   
 00013B4D: 61                        popad    
 00013B4E: C3                        ret   
 ----子程序结束-----------  
   
 00013B4F: 45                        inc ebp   
 00013B50: 7869                      js 00013BBB   
 00013B52: 7450                      jz 00013BA4   
 00013B54: 726F                      jb 00013BC5   
 00013B56: 636573                    arpl word ptr \[ebp+73\],sp   
 00013B59: 7300                      jnb 00013B5B   
   
\------子程序开始----------   
 00013B5B: 53                        push ebx   
 00013B5C: 56                        push esi   
 00013B5D: 57                        push edi   
 00013B5E: 8BF0                      mov esi,eax   
 00013B60: BF9CAE4000                mov edi,0040AE9C   
 00013B65: 03FD                      add edi,ebp   
 00013B67: 8A1E                      mov bl,byte ptr \[esi\]   
 00013B69: 3A1F                      cmp bl,byte ptr \[edi\]   
 00013B6B: 750C                      jnz 00013B79   
 00013B6D: 46                        inc esi   
 00013B6E: 47                        inc edi   
 00013B6F: 80FB00                    cmp bl,00   
 00013B72: 75F3                      jnz 00013B67   
 00013B74: B801000000                mov eax,00000001   
 00013B79: 5F                        pop edi   
 00013B7A: 5E                        pop esi   
 00013B7B: 5B                        pop ebx   
 00013B7C: C3                        ret   
 ------子程序结束-----------  
   
 00013B7D: 60                        pushad    
 00013B7E: 33C0                      xor eax,eax   
 00013B80: 8BB56DB44000              mov esi,dword ptr \[ebp+0040B46D\]   
 00013B86: 83FE00                    cmp esi,00000000   
 00013B89: 7459                      jz 00013BE4   
 00013B8B: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
 00013B91: 668B3E                    mov di,word ptr \[esi\]   
 00013B94: 663B7E02                  cmp di,word ptr \[esi+02\]   
 00013B98: 744A                      jz 00013BE4   
 00013B9A: 8B3E                      mov edi,dword ptr \[esi\]   
 00013B9C: 83FF00                    cmp edi,00000000   
 00013B9F: 7443                      jz 00013BE4   
 00013BA1: 03BDD3B24000              add edi,dword ptr \[ebp+0040B2D3\]   
 00013BA7: 8B4E04                    mov ecx,dword ptr \[esi+04\]   
 00013BAA: 83E908                    sub ecx,00000008   
 00013BAD: 83F900                    cmp ecx,00000000   
 00013BB0: 7432                      jz 00013BE4   
 00013BB2: D1E9                      shr ecx,1   
 00013BB4: 83C608                    add esi,00000008   
 00013BB7: 668B06                    mov ax,word ptr \[esi\]   
 00013BBA: 662500F0                  and ax,F000   
 00013BBE: 663D0030                  cmp ax,3000   
 00013BC2: 7519                      jnz 00013BDD   
 00013BC4: 668B06                    mov ax,word ptr \[esi\]   
 00013BC7: 6625FF0F                  and ax,0FFF   
 00013BCB: 8B9D79B44000              mov ebx,dword ptr \[ebp+0040B479\]   
 00013BD1: 291C07                    sub dword ptr \[edi+eax\],ebx   
 00013BD4: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]   
 00013BDA: 011C07                    add dword ptr \[edi+eax\],ebx   
 00013BDD: 83C602                    add esi,00000002   
 00013BE0: E2D5                      loop 00013BB7   
 00013BE2: EBAD                      jmp 00013B91   
 00013BE4: 61                        popad    
 00013BE5: C3                        ret   
   
   
 00013BE6: 60                        pushad    
 00013BE7: 8BB571B44000              mov esi,dword ptr \[ebp+0040B471\]   
 00013BED: 83FE00                    cmp esi,00000000   
 00013BF0: 7432                      jz 00013C24   
 00013BF2: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
 00013BF8: 8B8D75B44000              mov ecx,dword ptr \[ebp+0040B475\]   
 00013BFE: 83F900                    cmp ecx,00000000   
 00013C01: 7421                      jz 00013C24   
 00013C03: 8B06                      mov eax,dword ptr \[esi\]   
 00013C05: 83F800                    cmp eax,00000000   
 00013C08: 7412                      jz 00013C1C   
 00013C0A: 2B8579B44000              sub eax,dword ptr \[ebp+0040B479\]   
 00013C10: 2020                      and byte ptr \[eax\],ah   
 00013C12: 2020                      and byte ptr \[eax\],ah   
 00013C14: 2038                      and byte ptr \[eax\],bh   
 00013C16: 3530343400                xor eax,00343430   
 00013C1B: 0000                      add byte ptr \[eax\],al   
 00013C1D: 0000                      add byte ptr \[eax\],al   
 00013C1F: 0000                      add byte ptr \[eax\],al   
 00013C00: 0074218B                  add byte ptr \[ecx-75\],dh   
 00013C04: 06                        push es   
 00013C05: 83F800                    cmp eax,00000000   
 00013C08: 7412                      jz 00013C1C   
 00013C0A: 2B8579B44000              sub eax,dword ptr \[ebp+0040B479\]   
 00013C10: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00013C16: C70000000000              mov dword ptr \[eax\],00000000   
 00013C1C: 83C604                    add esi,00000004   
 00013C1F: 83E904                    sub ecx,00000004   
 00013C22: 75DF                      jnz 00013C03   
 00013C24: 61                        popad    
 00013C25: C3                        ret   
   
 ----子程序------  
 \*\*\*\*\*\*\*显示用户加的启动 NAG 提示信息  
 00013C26: B865B74000                mov eax,0040B765   
 00013C2B: 03C5                      add eax,ebp   
 00013C2D: 50                        push eax   
 00013C2E: FF95C0BC4000              call dword ptr \[ebp+0040BCC0\]   <--MessageBoxA  
 00013C34: C3                        ret  
\------子程序-------  
   
 ---被主程序调用------   
 00013C35: 60                        pushad    
 00013C36: 66C7855EB140000000        mov word ptr \[ebp+0040B15E\],0000   <---作标记  
 \*\*\*\*\*\*\*\*\*开始注册表中的键 wrifile\\shell\\open\\command\\config  
 00013C3F: B848B14000                mov eax,0040B148   
 00013C44: 03C5                      add eax,ebp   
 00013C46: BB40B14000                mov ebx,0040B140   
 00013C4B: 03DD                      add ebx,ebp   
 00013C4D: B91EB14000                mov ecx,0040B11E   
 00013C52: 03CD                      add ecx,ebp   
 00013C54: 50                        push eax   
 00013C55: 53                        push ebx   
 00013C56: 683F001F00                push 001F003F   
 00013C5B: 6A00                      push 00000000   
 00013C5D: 6A00                      push 00000000   
 00013C5F: 6A00                      push 00000000   
 00013C61: 6A00                      push 00000000   
 00013C63: 51                        push ecx     <-- "wrifile\\shell\\open\\command\\config"  
 00013C64: 6800000080                push 80000000   
 00013C69: FF9576BC4000              call dword ptr \[ebp+0040BC76\]     <---RegCreateKeyA  
 00013C6F: 663D0000                  cmp ax,0000   
 00013C73: 7566                      jnz 00013CDB   <--无此键，就跳走  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*查找某键\*\*\*\*\*\*\*\*\*\*\*  
 00013C75: BE1AB14000                mov esi,0040B11A   
 00013C7A: 03F5                      add esi,ebp   
 00013C7C: BF16B14000                mov edi,0040B116   
 00013C81: 03FD                      add edi,ebp   
 00013C83: B860B14000                mov eax,0040B160   
 00013C88: 03C5                      add eax,ebp   
 00013C8A: BB50B14000                mov ebx,0040B150   
 00013C8F: 03DD                      add ebx,ebp   
 00013C91: 8B8D40B14000              mov ecx,dword ptr \[ebp+0040B140\]   
 00013C97: 56                        push esi   
 00013C98: 50                        push eax   
 00013C99: 57                        push edi   
 00013C9A: 6A00                      push 00000000   
 00013C9C: 53                        push ebx   <--91,80,88,80,B5,80,A3,80,C8,80  
 00013C9D: 51                        push ecx   
 00013C9E: FF957EBC4000              call dword ptr \[ebp+0040BC7E\]     <---RegQueryValueEx  
 00013CA4: 663D0000                  cmp ax,0000   
 00013CA8: 7531                      jnz 00013CDB   <--无此键，就跳走  
   
 00013CAA: 66C7855EB140000100        mov word ptr \[ebp+0040B15E\],0001  <--作标记   
\*\*\*\*\*\*\*\*开始处理用户注册信息的还原\*\*\*\*\*\*\*\*\*   
 00013CB3: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 00013CBA: 8AE0                      mov ah,al   
 00013CBC: BE60B14000                mov esi,0040B160   
 00013CC1: 03F5                      add esi,ebp   
 00013CC3: B93D000000                mov ecx,0000003D   
 00013CC8: 51                        push ecx   
 00013CC9: 8A06                      mov al,byte ptr \[esi\]   
  
 00013CCB: 3C00                      cmp al,00   
 00013CCD: 7402                      jz 00013CD1   
 00013CCF: 32C4                      xor al,ah   
 00013CD1: B104                      mov cl,04   
 00013CD3: D2C8                      ror al,cl   
 00013CD5: 8806                      mov byte ptr \[esi\],al   
 00013CD7: 46                        inc esi   
 00013CD8: 59                        pop ecx   
 00013CD9: E2ED                      loop 00013CC8   
\*\*\*\*\*\*\*\*\*\*\*\*关闭注册表\*\*\*\*\*\*\*\*\*\*\*   
 00013CDB: 8B8540B14000              mov eax,dword ptr \[ebp+0040B140\]   
 00013CE1: 50                        push eax   
 00013CE2: FF957ABC4000              call dword ptr \[ebp+0040BC7A\]     <---RegCloseKey  
 00013CE8: 61                        popad    
 00013CE9: 668B855EB14000            mov ax,word ptr \[ebp+0040B15E\]   
 00013CF0: C3                        ret   
 -----子程序结束------  
  
\---被主程序调用-------对磁盘的一些调用   
 00013CF1: 60                        pushad    
 00013CF2: 66C7855EB140000000        mov word ptr \[ebp+0040B15E\],0000   
 00013CFB: B848B14000                mov eax,0040B148   
 00013D00: 03C5                      add eax,ebp   
 00013D02: BB40B14000                mov ebx,0040B140   
 00013D07: 03DD                      add ebx,ebp   
 00013D09: B91EB14000                mov ecx,0040B11E   
 00013D0E: 03CD                      add ecx,ebp   
 00013D10: 50                        push eax   
 00013D11: 53                        push ebx   
 00013D12: 683F001F00                push 001F003F   
 00013D17: 6A00                      push 00000000   
 00013D19: 6A00                      push 00000000   
 00013D1B: 6A00                      push 00000000   
 00013D1D: 6A00                      push 00000000   
 00013D1F: 51                        push ecx   
 00013D20: 6800000080                push 80000000   
 00013D25: FF9576BC4000              call dword ptr \[ebp+0040BC76\]     <--GetVolumeInformation  
 00013D2B: 663D0000                  cmp ax,0000   
 00013D2F: 755A                      jnz 00013D8B   
   
 00013D31: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 00013D38: 8AE0                      mov ah,al   
 00013D3A: BE60B14000                mov esi,0040B160   
 00013D3F: 03F5                      add esi,ebp   
 00013D41: B93D000000                mov ecx,0000003D   
   
 00013D46: 51                        push ecx   
 00013D47: 8A06                      mov al,byte ptr \[esi\]   
 00013D49: B104                      mov cl,04   
 00013D4B: D2C0                      rol al,cl   
 00013D4D: 3C00                      cmp al,00   
 00013D4F: 7402                      jz 00013D53   
 00013D51: 32C4                      xor al,ah   
 00013D53: 8806                      mov byte ptr \[esi\],al   
 00013D55: 46                        inc esi   
 00013D56: 59                        pop ecx   
 00013D57: E2ED                      loop 00013D46   
   
 00013D59: B860B14000                mov eax,0040B160   
 00013D5E: 03C5                      add eax,ebp   
 00013D60: BB50B14000                mov ebx,0040B150   
 00013D65: 03DD                      add ebx,ebp   
 00013D67: 8B8D40B14000              mov ecx,dword ptr \[ebp+0040B140\]   
 00013D6D: 6A3D                      push 0000003D   
 00013D6F: 50                        push eax   
 00013D70: 6A03                      push 00000003   
 00013D72: 6A00                      push 00000000   
 00013D74: 53                        push ebx   
 00013D75: 51                        push ecx   
 00013D76: FF9582BC4000              call dword ptr \[ebp+0040BC82\]     <---GetDiskFreeSpaceExA  
 00013D7C: 663D0000                  cmp ax,0000   
 00013D80: 7509                      jnz 00013D8B   
 00013D82: 66C7855EB140000100        mov word ptr \[ebp+0040B15E\],0001   
 00013D8B: 8B8540B14000              mov eax,dword ptr \[ebp+0040B140\]   
 00013D91: 50                        push eax   
 00013D92: FF957ABC4000              call dword ptr \[ebp+0040BC7A\]    <---GetExitCodeProcess   
 00013D98: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
 00013D9F: 8AE0                      mov ah,al   
 00013DA1: BE60B14000                mov esi,0040B160   
 00013DA6: 03F5                      add esi,ebp   
 00013DA8: B93D000000                mov ecx,0000003D   
   
 00013DAD: 51                        push ecx   
 00013DAE: 8A06                      mov al,byte ptr \[esi\]   
 00013DB0: 3C00                      cmp al,00   
 00013DB2: 7402                      jz 00013DB6   
 00013DB4: 32C4                      xor al,ah   
 00013DB6: B104                      mov cl,04   
 00013DB8: D2C8                      ror al,cl   
 00013DBA: 8806                      mov byte ptr \[esi\],al   
 00013DBC: 46                        inc esi   
 00013DBD: 59                        pop ecx   
 00013DBE: E2ED                      loop 00013DAD   
   
 00013DC0: 61                        popad    
 00013DC1: 668B855EB14000            mov ax,word ptr \[ebp+0040B15E\]   
 00013DC8: C3                        ret   
 --------子程序结束----------  
   
 00013DC9: 0300                      add eax,dword ptr \[eax\]   
 00013DCB: 0000                      add byte ptr \[eax\],al   
 00013DCD: 3D00000077                cmp eax,77000000   
 00013DD2: 7269                      jb 00013E3D   
 00013DD4: 66696C655C7368            imul bp,word ptr \[ebp+5C\],6873   
 00013DDB: 656C                      insb es:\[edi\],dx   
 00013DDD: 6C                        insb es:\[edi\],dx   
 00013DDE: 5C                        pop esp   
 00013DDF: 6F                        outsd dx,ds:\[esi\]   
 00013DE0: 7065                      jo 00013E47   
 00013DE2: 6E                        outsb dx,ds:\[esi\]   
 00013DE3: 5C                        pop esp   
 00013DE4: 636F6D                    arpl word ptr \[edi+6D\],bp   
 00013DE7: 6D                        insd es:\[edi\],dx   
 00013DE8: 61                        popad    
 00013DE9: 6E                        outsb dx,ds:\[esi\]   
 00013DEA: 645C                      pop esp   
 00013DEC: 636F6E                    arpl word ptr \[edi+6E\],bp   
 00013DEF: 666967000000              imul sp,word ptr \[edi\],0000   
 00013DF5: 0000                      add byte ptr \[eax\],al   
 00013DF7: 0000                      add byte ptr \[eax\],al   
 00013DF9: 0000                      add byte ptr \[eax\],al   
 00013DFB: 0000                      add byte ptr \[eax\],al   
 00013DFD: 0000                      add byte ptr \[eax\],al   
 00013DFF: 0000                      add byte ptr \[eax\],al   
 00013E01: 0000                      add byte ptr \[eax\],al   
 00013E03: 91                        xchg eax,ecx   
 00013E04: 808880B580A380            or byte ptr \[eax-5C7F4A80\],80   
 00013E0B: C8800000                  enter 0080,00   
 00013E0F: 0000                      add byte ptr \[eax\],al   
 00013E11: 0000                      add byte ptr \[eax\],al   
 00013E13: 0000                      add byte ptr \[eax\],al   
 00013E15: 0000                      add byte ptr \[eax\],al   
 00013E17: 0000                      add byte ptr \[eax\],al   
 00013E19: 0000                      add byte ptr \[eax\],al   
 00013E1B: 0000                      add byte ptr \[eax\],al   
 00013E1D: 0000                      add byte ptr \[eax\],al   
 00013E1F: 0000                      add byte ptr \[eax\],al   
 00013E21: 0000                      add byte ptr \[eax\],al   
 00013E23: 0000                      add byte ptr \[eax\],al   
 00013E25: 0000                      add byte ptr \[eax\],al   
 00013E27: 0000                      add byte ptr \[eax\],al   
 00013E29: 0000                      add byte ptr \[eax\],al   
 00013E2B: 0000                      add byte ptr \[eax\],al   
 00013E2D: 0000                      add byte ptr \[eax\],al   
 00013E2F: 0000                      add byte ptr \[eax\],al   
 00013E31: 0000                      add byte ptr \[eax\],al   
 00013E33: 0000                      add byte ptr \[eax\],al   
 00013E35: 0000                      add byte ptr \[eax\],al   
 00013E37: 0000                      add byte ptr \[eax\],al   
 00013E39: 0000                      add byte ptr \[eax\],al   
 00013E3B: 0000                      add byte ptr \[eax\],al   
 00013E3D: 0000                      add byte ptr \[eax\],al   
 00013E3F: 0000                      add byte ptr \[eax\],al   
 00013E41: 0000                      add byte ptr \[eax\],al   
 00013E43: 0000                      add byte ptr \[eax\],al   
 00013E45: 0000                      add byte ptr \[eax\],al   
 00013E47: 0000                      add byte ptr \[eax\],al   
 00013E49: 0000                      add byte ptr \[eax\],al   
 00013E4B: 0000                      add byte ptr \[eax\],al   
 00013E4D: 0000                      add byte ptr \[eax\],al   
 00013E4F: 0000                      add byte ptr \[eax\],al   
 00013E51: 0000                      add byte ptr \[eax\],al   
 00013E53: 0000                      add byte ptr \[eax\],al   
 00013E55: 0000                      add byte ptr \[eax\],al   
 00013E57: 0000                      add byte ptr \[eax\],al   
 00013E59: 0000                      add byte ptr \[eax\],al   
 00013E5B: 0000                      add byte ptr \[eax\],al   
 00013E5D: 0000                      add byte ptr \[eax\],al   
 00013E5F: 0000                      add byte ptr \[eax\],al   
 00013E61: 0000                      add byte ptr \[eax\],al   
 00013E63: 0000                      add byte ptr \[eax\],al   
 00013E65: 0000                      add byte ptr \[eax\],al   
 00013E67: 0000                      add byte ptr \[eax\],al   
 00013E69: 0000                      add byte ptr \[eax\],al   
 00013E6B: 0000                      add byte ptr \[eax\],al   
 00013E6D: 0000                      add byte ptr \[eax\],al   
 00013E6F: 0000                      add byte ptr \[eax\],al   
 00013E71: 0000                      add byte ptr \[eax\],al   
 00013E73: 0000                      add byte ptr \[eax\],al   
 00013E75: 0000                      add byte ptr \[eax\],al   
 00013E77: 0000                      add byte ptr \[eax\],al   
 00013E79: 0000                      add byte ptr \[eax\],al   
 00013E7B: 0000                      add byte ptr \[eax\],al   
 00013E7D: 0000                      add byte ptr \[eax\],al   
 00013E7F: 0000                      add byte ptr \[eax\],al   
 00013E81: 0000                      add byte ptr \[eax\],al   
 00013E83: 0000                      add byte ptr \[eax\],al   
 00013E85: 0000                      add byte ptr \[eax\],al   
 00013E87: 0000                      add byte ptr \[eax\],al   
 00013E89: 0000                      add byte ptr \[eax\],al   
 00013E8B: 0000                      add byte ptr \[eax\],al   
 00013E8D: 0000                      add byte ptr \[eax\],al   
 00013E8F: 0000                      add byte ptr \[eax\],al   
 00013E91: 0000                      add byte ptr \[eax\],al   
 00013E93: 0000                      add byte ptr \[eax\],al   
 00013E95: 0000                      add byte ptr \[eax\],al   
 00013E97: 0000                      add byte ptr \[eax\],al   
 00013E99: 0000                      add byte ptr \[eax\],al   
 00013E9B: 0000                      add byte ptr \[eax\],al   
 00013E9D: 0000                      add byte ptr \[eax\],al   
 00013E9F: 0000                      add byte ptr \[eax\],al   
 00013EA1: 0000                      add byte ptr \[eax\],al   
 00013EA3: 0000                      add byte ptr \[eax\],al   
 00013EA5: 0000                      add byte ptr \[eax\],al   
 00013EA7: 0000                      add byte ptr \[eax\],al   
 00013EA9: 0000                      add byte ptr \[eax\],al   
 00013EAB: 0000                      add byte ptr \[eax\],al   
 00013EAD: 0003                      add byte ptr \[ebx\],al   
 00013EAF: 0000                      add byte ptr \[eax\],al   
 00013EB1: 0029                      add byte ptr \[ecx\],ch   
 00013EB3: 0000                      add byte ptr \[eax\],al   
 00013EB5: 002A                      add byte ptr \[edx\],ch   
 00013EB7: 00494E                    add byte ptr \[ecx+4E\],cl   
 00013EBA: 5F                        pop edi   
 00013EBB: 43                        inc ebx   
 00013EBC: 4F                        dec edi   
 00013EBD: 44                        inc esp   
 00013EBE: 45                        inc ebp   
 00013EBF: 0000                      add byte ptr \[eax\],al   
 00013EC1: 0000                      add byte ptr \[eax\],al   
 00013EC3: 0020                      add byte ptr \[eax\],ah   
 00013EC5: 2020                      and byte ptr \[eax\],ah   
 00013EC7: 2020                      and byte ptr \[eax\],ah   
 00013EC9: 2020                      and byte ptr \[eax\],ah   
 00013ECB: 2020                      and byte ptr \[eax\],ah   
 00013ECD: 2020                      and byte ptr \[eax\],ah   
 00013ECF: B1BE                      mov cl,BE   
 00013ED1: C8EDBCFE                  enter BCED,FE   
 00013ED5: D3C9                      ror ecx,cl   
 00013ED7: 20BBC3D3B076              and byte ptr \[ebx+76B0D3C3\],bh   
 00013EDD: 312E                      xor dword ptr \[esi\],ebp   
 00013EDF: 3520CEB4D7                xor eax,D7B4CE20   
 00013EE4: A2B2E1B0E6                mov \[E6B0E1B2\],al   
 00013EE9: B1BE                      mov cl,BE   
 00013EEB: BCD3C3DC20                mov esp,20DCC3D3   
 00013EF0: 2020                      and byte ptr \[eax\],ah   
 00013EF2: 2020                      and byte ptr \[eax\],ah   
 00013EF4: 2038                      and byte ptr \[eax\],bh   
 00013EF6: 3530343400                xor eax,00343430   
 00013EFB: 0000                      add byte ptr \[eax\],al   
 00013EFD: 0000                      add byte ptr \[eax\],al   
 00013EFF: 0000                      add byte ptr \[eax\],al   
 00013EE0: 20CE                      and dh,cl   
 00013EE2: B4D7                      mov ah,D7   
 00013EE4: A2B2E1B0E6                mov \[E6B0E1B2\],al   
 00013EE9: B1BE                      mov cl,BE   
 00013EEB: BCD3C3DC20                mov esp,20DCC3D3   
 00013EF0: 2020                      and byte ptr \[eax\],ah   
 00013EF2: 2020                      and byte ptr \[eax\],ah   
 00013EF4: 2020                      and byte ptr \[eax\],ah   
 00013EF6: 2020                      and byte ptr \[eax\],ah   
 00013EF8: 2020                      and byte ptr \[eax\],ah   
 00013EFA: 0D0A0D0A20                or eax,200A0D0A   
 00013EFF: 54                        push esp   
 00013F00: 6869732070                push 70207369   
 00013F05: 726F                      jb 00013F76   
 00013F07: 677261                    jb 00013F6B   
 00013F0A: 6D                        insd es:\[edi\],dx   
 00013F0B: 207072                    and byte ptr \[eax+72\],dh   
 00013F0E: 6F                        outsd dx,ds:\[esi\]   
 00013F0F: 7465                      jz 00013F76   
 00013F11: 63746564                  arpl word ptr \[ebp+64\],si   
 00013F15: 206279                    and byte ptr \[edx+79\],ah   
 00013F18: 20444250                  and byte ptr \[edx+eax\*2+50\],al   
 00013F1C: 45                        inc ebp   
 00013F1D: 207631                    and byte ptr \[esi+31\],dh   
 00013F20: 2E352020556E              xor eax,6E552020   
 00013F26: 7265                      jb 00013F8D   
 00013F28: 6769737465726564          imul esi,dword ptr \[bp+di+74\],64657265   
 00013F30: 207665                    and byte ptr \[esi+65\],dh   
 00013F33: 7273                      jb 00013FA8   
 00013F35: 696F6E0D0A0D0A            imul ebp,dword ptr \[edi+6E\],0A0D0A0D   
 00013F3C: 0D0A0D0A20                or eax,200A0D0A   
 00013F41: 48                        dec eax   
 00013F42: 6F                        outsd dx,ds:\[esi\]   
 00013F43: 6D                        insd es:\[edi\],dx   
 00013F44: 657061                    jo 00013FA8   
 00013F47: 67653A20                  cmp ah,byte ptr gs:\[bx+si\]   
 00013F4B: 687474703A                push 3A707474   
 00013F50: 2F                        das   
 00013F51: 2F                        das   
 00013F52: 667364                    jnb 00013FB9   
 00013F55: 622E                      bound ebp,dword ptr \[esi\]   
 00013F57: 7965                      jns 00013FBE   
 00013F59: 61                        popad    
 00013F5A: 682E6E6574                push 74656E2E   
 00013F5F: 2020                      and byte ptr \[eax\],ah   
 00013F61: 2020                      and byte ptr \[eax\],ah   
 00013F63: 45                        inc ebp   
 00013F64: 2D6D61696C                sub eax,6C69616D   
 00013F69: 3A20                      cmp ah,byte ptr \[eax\]   
 00013F6B: 44                        inc esp   
 00013F6C: 2E42                      inc edx   
 00013F6E: 6F                        outsd dx,ds:\[esi\]   
 00013F6F: 7940                      jns 00013FB1   
 00013F71: 3132                      xor dword ptr \[edx\],esi   
 00013F73: 362E636F6D                arpl word ptr cs:\[edi+6D\],bp   
 00013F78: 200D0A007C7C              and byte ptr \[7C7C000A\],cl   
 00013F7E: 00D0                      add al,dl   
 00013F80: 0000                      add byte ptr \[eax\],al   
 00013F82: CC                        int 3   
 00013F83: 1000                      adc byte ptr \[eax\],al   
 00013F85: 0000                      add byte ptr \[eax\],al   
 00013F87: 0000                      add byte ptr \[eax\],al   
 00013F89: 0000                      add byte ptr \[eax\],al   
 00013F8B: 1000                      adc byte ptr \[eax\],al   
 00013F8D: 0000                      add byte ptr \[eax\],al   
 00013F8F: 40                        inc eax   
 00013F90: 0000                      add byte ptr \[eax\],al   
 00013F92: 005000                    add byte ptr \[eax\],dl   
 00013F95: 0000                      add byte ptr \[eax\],al   
 00013F97: 1000                      adc byte ptr \[eax\],al   
 00013F99: 0000                      add byte ptr \[eax\],al   
 00013F9B: 60                        pushad    
 00013F9C: 0000                      add byte ptr \[eax\],al   
 00013F9E: 0010                      add byte ptr \[eax\],dl   
 00013FA0: 0000                      add byte ptr \[eax\],al   
 00013FA2: 00C0                      add al,al   
 00013FA4: 0000                      add byte ptr \[eax\],al   
 00013FA6: 0010                      add byte ptr \[eax\],dl   
 00013FA8: 0000                      add byte ptr \[eax\],al   
 00013FAA: 0000                      add byte ptr \[eax\],al   
 00013FAC: 0000                      add byte ptr \[eax\],al   
 00013FAE: 0000                      add byte ptr \[eax\],al   
 00013FB0: 0000                      add byte ptr \[eax\],al   
 00013FB2: 0000                      add byte ptr \[eax\],al   
 00013FB4: 0000                      add byte ptr \[eax\],al   
 00013FB6: 0000                      add byte ptr \[eax\],al   
 00013FB8: 0000                      add byte ptr \[eax\],al   
 00013FBA: 0000                      add byte ptr \[eax\],al   
 00013FBC: 0000                      add byte ptr \[eax\],al   
 00013FBE: 0000                      add byte ptr \[eax\],al   
 00013FC0: 0000                      add byte ptr \[eax\],al   
 00013FC2: 0000                      add byte ptr \[eax\],al   
 00013FC4: 0000                      add byte ptr \[eax\],al   
 00013FC6: 0000                      add byte ptr \[eax\],al   
 00013FC8: 0000                      add byte ptr \[eax\],al   
 00013FCA: 0000                      add byte ptr \[eax\],al   
 00013FCC: 0000                      add byte ptr \[eax\],al   
 00013FCE: 0000                      add byte ptr \[eax\],al   
 00013FD0: 0000                      add byte ptr \[eax\],al   
 00013FD2: 0000                      add byte ptr \[eax\],al   
 00013FD4: 0000                      add byte ptr \[eax\],al   
 00013FD6: 0000                      add byte ptr \[eax\],al   
 00013FD8: 0000                      add byte ptr \[eax\],al   
 00013FDA: 0000                      add byte ptr \[eax\],al   
 00013FDC: 0000                      add byte ptr \[eax\],al   
 00013FDE: 0000                      add byte ptr \[eax\],al   
 00013FE0: 0000                      add byte ptr \[eax\],al   
 00013FE2: 0000                      add byte ptr \[eax\],al   
 00013FE4: 0000                      add byte ptr \[eax\],al   
 00013FE6: 0000                      add byte ptr \[eax\],al   
 00013FE8: 0000                      add byte ptr \[eax\],al   
 00013FEA: 0000                      add byte ptr \[eax\],al   
 00013FEC: 0000                      add byte ptr \[eax\],al   
 00013FEE: 0000                      add byte ptr \[eax\],al   
 00013FF0: 0000                      add byte ptr \[eax\],al   
 00013FF2: 0000                      add byte ptr \[eax\],al   
 00013FF4: 0000                      add byte ptr \[eax\],al   
 00013FF6: 0000                      add byte ptr \[eax\],al   
 00013FF8: 0000                      add byte ptr \[eax\],al   
 00013FFA: 0000                      add byte ptr \[eax\],al   
 00013FFC: 0000                      add byte ptr \[eax\],al   
 00013FFE: 0000                      add byte ptr \[eax\],al   
 00014000: 0000                      add byte ptr \[eax\],al   
 00014002: 0000                      add byte ptr \[eax\],al   
 00014004: 0000                      add byte ptr \[eax\],al   
 00014006: 0000                      add byte ptr \[eax\],al   
 00014008: 0000                      add byte ptr \[eax\],al   
 0001400A: 0000                      add byte ptr \[eax\],al   
 0001400C: 0000                      add byte ptr \[eax\],al   
 0001400E: 0000                      add byte ptr \[eax\],al   
 00014010: 0000                      add byte ptr \[eax\],al   
 00014012: 0000                      add byte ptr \[eax\],al   
 00014014: 0000                      add byte ptr \[eax\],al   
 00014016: 0000                      add byte ptr \[eax\],al   
 00014018: 0000                      add byte ptr \[eax\],al   
 0001401A: 0000                      add byte ptr \[eax\],al   
 0001401C: 0000                      add byte ptr \[eax\],al   
 0001401E: 0000                      add byte ptr \[eax\],al   
 00014020: 0000                      add byte ptr \[eax\],al   
 00014022: 0000                      add byte ptr \[eax\],al   
 00014024: 0000                      add byte ptr \[eax\],al   
 00014026: 0000                      add byte ptr \[eax\],al   
 00014028: 0000                      add byte ptr \[eax\],al   
 0001402A: 0000                      add byte ptr \[eax\],al   
 0001402C: 0000                      add byte ptr \[eax\],al   
 0001402E: 0000                      add byte ptr \[eax\],al   
 00014030: 0000                      add byte ptr \[eax\],al   
 00014032: 0000                      add byte ptr \[eax\],al   
 00014034: 0000                      add byte ptr \[eax\],al   
 00014036: 0000                      add byte ptr \[eax\],al   
 00014038: 0000                      add byte ptr \[eax\],al   
 0001403A: 0000                      add byte ptr \[eax\],al   
 0001403C: 0000                      add byte ptr \[eax\],al   
 0001403E: 0000                      add byte ptr \[eax\],al   
 00014040: 0000                      add byte ptr \[eax\],al   
 00014042: 0000                      add byte ptr \[eax\],al   
 00014044: 0000                      add byte ptr \[eax\],al   
 00014046: 0000                      add byte ptr \[eax\],al   
 00014048: 0000                      add byte ptr \[eax\],al   
 0001404A: 0000                      add byte ptr \[eax\],al   
 0001404C: 0000                      add byte ptr \[eax\],al   
 0001404E: 0000                      add byte ptr \[eax\],al   
 00014050: 0000                      add byte ptr \[eax\],al   
 00014052: 0000                      add byte ptr \[eax\],al   
 00014054: 0000                      add byte ptr \[eax\],al   
 00014056: 0000                      add byte ptr \[eax\],al   
 00014058: 0000                      add byte ptr \[eax\],al   
 0001405A: 0000                      add byte ptr \[eax\],al   
 0001405C: 0000                      add byte ptr \[eax\],al   
 0001405E: 0000                      add byte ptr \[eax\],al   
 00014060: 0000                      add byte ptr \[eax\],al   
 00014062: 0000                      add byte ptr \[eax\],al   
 00014064: 0000                      add byte ptr \[eax\],al   
 00014066: 0000                      add byte ptr \[eax\],al   
 00014068: 0000                      add byte ptr \[eax\],al   
 0001406A: 0000                      add byte ptr \[eax\],al   
 0001406C: 0000                      add byte ptr \[eax\],al   
 0001406E: 0000                      add byte ptr \[eax\],al   
 00014070: 0000                      add byte ptr \[eax\],al   
 00014072: 0000                      add byte ptr \[eax\],al   
 00014074: 0000                      add byte ptr \[eax\],al   
 00014076: 0000                      add byte ptr \[eax\],al   
 00014078: 0000                      add byte ptr \[eax\],al   
 0001407A: 0000                      add byte ptr \[eax\],al   
 0001407C: 0000                      add byte ptr \[eax\],al   
 0001407E: 0000                      add byte ptr \[eax\],al   
 00014080: 0000                      add byte ptr \[eax\],al   
 00014082: 0000                      add byte ptr \[eax\],al   
 00014084: 0000                      add byte ptr \[eax\],al   
 00014086: 0000                      add byte ptr \[eax\],al   
 00014088: 0000                      add byte ptr \[eax\],al   
 0001408A: 0000                      add byte ptr \[eax\],al   
 0001408C: 0000                      add byte ptr \[eax\],al   
 0001408E: 0000                      add byte ptr \[eax\],al   
 00014090: 0000                      add byte ptr \[eax\],al   
 00014092: 0000                      add byte ptr \[eax\],al   
 00014094: 0000                      add byte ptr \[eax\],al   
 00014096: 0000                      add byte ptr \[eax\],al   
 00014098: 0000                      add byte ptr \[eax\],al   
 0001409A: 0000                      add byte ptr \[eax\],al   
 0001409C: 0000                      add byte ptr \[eax\],al   
 0001409E: 0000                      add byte ptr \[eax\],al   
 000140A0: 0000                      add byte ptr \[eax\],al   
 000140A2: 0000                      add byte ptr \[eax\],al   
 000140A4: 0000                      add byte ptr \[eax\],al   
 000140A6: 0000                      add byte ptr \[eax\],al   
 000140A8: 0000                      add byte ptr \[eax\],al   
 000140AA: 0000                      add byte ptr \[eax\],al   
 000140AC: 0000                      add byte ptr \[eax\],al   
 000140AE: 0000                      add byte ptr \[eax\],al   
 000140B0: 0000                      add byte ptr \[eax\],al   
 000140B2: 0000                      add byte ptr \[eax\],al   
 000140B4: 0000                      add byte ptr \[eax\],al   
 000140B6: 0000                      add byte ptr \[eax\],al   
 000140B8: 0000                      add byte ptr \[eax\],al   
 000140BA: 0000                      add byte ptr \[eax\],al   
 000140BC: 0000                      add byte ptr \[eax\],al   
 000140BE: 0000                      add byte ptr \[eax\],al   
 000140C0: 0000                      add byte ptr \[eax\],al   
 000140C2: 0000                      add byte ptr \[eax\],al   
 000140C4: 0000                      add byte ptr \[eax\],al   
 000140C6: 0000                      add byte ptr \[eax\],al   
 000140C8: 0000                      add byte ptr \[eax\],al   
 000140CA: 0000                      add byte ptr \[eax\],al   
 000140CC: 0000                      add byte ptr \[eax\],al   
 000140CE: 0000                      add byte ptr \[eax\],al   
 000140D0: 0000                      add byte ptr \[eax\],al   
 000140D2: 0000                      add byte ptr \[eax\],al   
 000140D4: 0000                      add byte ptr \[eax\],al   
 000140D6: 0000                      add byte ptr \[eax\],al   
 000140D8: 0000                      add byte ptr \[eax\],al   
 000140DA: 0000                      add byte ptr \[eax\],al   
 000140DC: 0000                      add byte ptr \[eax\],al   
 000140DE: 0000                      add byte ptr \[eax\],al   
 000140E0: 0000                      add byte ptr \[eax\],al   
 000140E2: 0000                      add byte ptr \[eax\],al   
 000140E4: 0000                      add byte ptr \[eax\],al   
 000140E6: 0000                      add byte ptr \[eax\],al   
 000140E8: 0000                      add byte ptr \[eax\],al   
 000140EA: 0000                      add byte ptr \[eax\],al   
 000140EC: 0000                      add byte ptr \[eax\],al   
 000140EE: 0000                      add byte ptr \[eax\],al   
 000140F0: 0000                      add byte ptr \[eax\],al   
 000140F2: 0000                      add byte ptr \[eax\],al   
 000140F4: 0000                      add byte ptr \[eax\],al   
 000140F6: 0000                      add byte ptr \[eax\],al   
 000140F8: 0000                      add byte ptr \[eax\],al   
 000140FA: 0000                      add byte ptr \[eax\],al   
 000140FC: 0000                      add byte ptr \[eax\],al   
 000140FE: 0000                      add byte ptr \[eax\],al   
 00014100: 0000                      add byte ptr \[eax\],al   
 00014102: 0000                      add byte ptr \[eax\],al   
 00014104: 0000                      add byte ptr \[eax\],al   
 00014106: 0000                      add byte ptr \[eax\],al   
 00014108: 0000                      add byte ptr \[eax\],al   
 0001410A: 0000                      add byte ptr \[eax\],al   
 0001410C: 0000                      add byte ptr \[eax\],al   
 0001410E: 0000                      add byte ptr \[eax\],al   
 00014110: 0000                      add byte ptr \[eax\],al   
 00014112: 0000                      add byte ptr \[eax\],al   
 00014114: 0000                      add byte ptr \[eax\],al   
 00014116: 0000                      add byte ptr \[eax\],al   
 00014118: 0000                      add byte ptr \[eax\],al   
 0001411A: C8800060                  enter 0080,60   
 0001411E: 0000                      add byte ptr \[eax\],al   
 00014120: 00C0                      add al,al   
 00014122: 0000                      add byte ptr \[eax\],al   
 00014124: 0000                      add byte ptr \[eax\],al   
 00014126: 0000                      add byte ptr \[eax\],al   
 00014128: 0000                      add byte ptr \[eax\],al   
 0001412A: 0000                      add byte ptr \[eax\],al   
 0001412C: 0000                      add byte ptr \[eax\],al   
 0001412E: 40                        inc eax   
 0001412F: 0000                      add byte ptr \[eax\],al   
 00014131: 0000                      add byte ptr \[eax\],al   
 00014133: 0000                      add byte ptr \[eax\],al   
 00014135: 0000                      add byte ptr \[eax\],al   
 00014137: 0000                      add byte ptr \[eax\],al   
 00014139: 0000                      add byte ptr \[eax\],al   
 0001413B: 0000                      add byte ptr \[eax\],al   
 0001413D: 0000                      add byte ptr \[eax\],al   
 0001413F: 00443A5C                  add byte ptr \[edx+edi+5C\],al   
 00014143: 54                        push esp   
 00014144: 45                        inc ebp   
 00014145: 4D                        dec ebp   
 00014146: 50                        push eax   
 00014147: 5C                        pop esp   
 00014148: 64627065                  bound esi,dword ptr fs:\[eax+65\]   
 0001414C: 5C                        pop esp   
 0001414D: 54                        push esp   
 0001414E: 45                        inc ebp   
 0001414F: 53                        push ebx   
 00014150: 54                        push esp   
 00014151: 5C                        pop esp   
 00014152: 54                        push esp   
 00014153: 45                        inc ebp   
 00014154: 53                        push ebx   
 00014155: 54                        push esp   
 00014156: 315C4442                  xor dword ptr \[esp+eax\*2+42\],ebx   
 0001415A: 4E                        dec esi   
 0001415B: 6F                        outsd dx,ds:\[esi\]   
 0001415C: 7465                      jz 000141C3   
 0001415E: 2E45                      inc ebp   
 00014160: 58                        pop eax   
 00014161: 45                        inc ebp   
 00014162: 0000                      add byte ptr \[eax\],al   
 00014164: 0000                      add byte ptr \[eax\],al   
 00014166: 0000                      add byte ptr \[eax\],al   
 00014168: 0000                      add byte ptr \[eax\],al   
 0001416A: 0000                      add byte ptr \[eax\],al   
 0001416C: 0000                      add byte ptr \[eax\],al   
 0001416E: 0000                      add byte ptr \[eax\],al   
 00014170: 0000                      add byte ptr \[eax\],al   
 00014172: 0000                      add byte ptr \[eax\],al   
 00014174: 0000                      add byte ptr \[eax\],al   
 00014176: 0000                      add byte ptr \[eax\],al   
 00014178: 0000                      add byte ptr \[eax\],al   
 0001417A: 0000                      add byte ptr \[eax\],al   
 0001417C: 0000                      add byte ptr \[eax\],al   
 0001417E: 0000                      add byte ptr \[eax\],al   
 00014180: 0000                      add byte ptr \[eax\],al   
 00014182: 0000                      add byte ptr \[eax\],al   
 00014184: 0000                      add byte ptr \[eax\],al   
 00014186: 0000                      add byte ptr \[eax\],al   
 00014188: 0000                      add byte ptr \[eax\],al   
 0001418A: 0000                      add byte ptr \[eax\],al   
 0001418C: 0000                      add byte ptr \[eax\],al   
 0001418E: 0000                      add byte ptr \[eax\],al   
 00014190: 0000                      add byte ptr \[eax\],al   
 00014192: 0000                      add byte ptr \[eax\],al   
 00014194: 0000                      add byte ptr \[eax\],al   
 00014196: 0000                      add byte ptr \[eax\],al   
 00014198: 0000                      add byte ptr \[eax\],al   
 0001419A: 0000                      add byte ptr \[eax\],al   
 0001419C: 0000                      add byte ptr \[eax\],al   
 0001419E: 0000                      add byte ptr \[eax\],al   
 000141A0: 0000                      add byte ptr \[eax\],al   
 000141A2: 0000                      add byte ptr \[eax\],al   
 000141A4: 7C7C                      jl 00014222   
 000141A6: 0020                      add byte ptr \[eax\],ah   
 000141A8: BBB6D3AD20                mov ebx,20ADD3B6   
 000141AD: 205E5F                    and byte ptr \[esi+5F\],bl   
 000141B0: 5E                        pop esi   
 000141B1: 0000                      add byte ptr \[eax\],al   
 000141B3: 0000                      add byte ptr \[eax\],al   
 000141B5: 0000                      add byte ptr \[eax\],al   
 000141B7: 0000                      add byte ptr \[eax\],al   
 000141B9: 0000                      add byte ptr \[eax\],al   
 000141BB: 0000                      add byte ptr \[eax\],al   
 000141BD: 0000                      add byte ptr \[eax\],al   
 000141BF: 0000                      add byte ptr \[eax\],al   
 000141C1: 0000                      add byte ptr \[eax\],al   
 000141C3: 0000                      add byte ptr \[eax\],al   
 000141C5: 0000                      add byte ptr \[eax\],al   
 000141C7: 0000                      add byte ptr \[eax\],al   
 000141C9: 0000                      add byte ptr \[eax\],al   
 000141CB: 0000                      add byte ptr \[eax\],al   
 000141CD: 0000                      add byte ptr \[eax\],al   
 000141CF: 0020                      add byte ptr \[eax\],ah   
 000141D1: 2020                      and byte ptr \[eax\],ah   
 000141D3: 2020                      and byte ptr \[eax\],ah   
 000141D5: 383530343400              cmp byte ptr \[00343430\],dh   
 000141DB: 0000                      add byte ptr \[eax\],al   
 000141DD: 0000                      add byte ptr \[eax\],al   
 000141DF: 0000                      add byte ptr \[eax\],al   
 000141C0: 0000                      add byte ptr \[eax\],al   
 000141C2: 0000                      add byte ptr \[eax\],al   
 000141C4: 0000                      add byte ptr \[eax\],al   
 000141C6: 0000                      add byte ptr \[eax\],al   
 000141C8: 0000                      add byte ptr \[eax\],al   
 000141CA: 0000                      add byte ptr \[eax\],al   
 000141CC: 0000                      add byte ptr \[eax\],al   
 000141CE: 0000                      add byte ptr \[eax\],al   
 000141D0: 0000                      add byte ptr \[eax\],al   
 000141D2: 0000                      add byte ptr \[eax\],al   
 000141D4: 0000                      add byte ptr \[eax\],al   
 000141D6: 0000                      add byte ptr \[eax\],al   
 000141D8: 0000                      add byte ptr \[eax\],al   
 000141DA: 0000                      add byte ptr \[eax\],al   
 000141DC: 0000                      add byte ptr \[eax\],al   
 000141DE: 0000                      add byte ptr \[eax\],al   
 000141E0: 0000                      add byte ptr \[eax\],al   
 000141E2: 0000                      add byte ptr \[eax\],al   
 000141E4: 0000                      add byte ptr \[eax\],al   
 000141E6: 0000                      add byte ptr \[eax\],al   
 000141E8: 0000                      add byte ptr \[eax\],al   
 000141EA: 0000                      add byte ptr \[eax\],al   
 000141EC: 0000                      add byte ptr \[eax\],al   
 000141EE: 0000                      add byte ptr \[eax\],al   
 000141F0: 0000                      add byte ptr \[eax\],al   
 000141F2: 0000                      add byte ptr \[eax\],al   
 000141F4: 0000                      add byte ptr \[eax\],al   
 000141F6: 0000                      add byte ptr \[eax\],al   
 000141F8: 0000                      add byte ptr \[eax\],al   
 000141FA: 0000                      add byte ptr \[eax\],al   
 000141FC: 0000                      add byte ptr \[eax\],al   
 000141FE: 0000                      add byte ptr \[eax\],al   
 00014200: 0000                      add byte ptr \[eax\],al   
 00014202: 0000                      add byte ptr \[eax\],al   
 00014204: 0000                      add byte ptr \[eax\],al   
 00014206: 0000                      add byte ptr \[eax\],al   
 00014208: 0000                      add byte ptr \[eax\],al   
 0001420A: 0000                      add byte ptr \[eax\],al   
 0001420C: 0000                      add byte ptr \[eax\],al   
 0001420E: 0000                      add byte ptr \[eax\],al   
 00014210: 0000                      add byte ptr \[eax\],al   
 00014212: 0000                      add byte ptr \[eax\],al   
 00014214: 0000                      add byte ptr \[eax\],al   
 00014216: 0000                      add byte ptr \[eax\],al   
 00014218: 0000                      add byte ptr \[eax\],al   
 0001421A: 0000                      add byte ptr \[eax\],al   
 0001421C: 0000                      add byte ptr \[eax\],al   
 0001421E: 0000                      add byte ptr \[eax\],al   
 00014220: 0000                      add byte ptr \[eax\],al   
 00014222: 0000                      add byte ptr \[eax\],al   
 00014224: 0000                      add byte ptr \[eax\],al   
 00014226: 0000                      add byte ptr \[eax\],al   
 00014228: 0000                      add byte ptr \[eax\],al   
 0001422A: 0000                      add byte ptr \[eax\],al   
 0001422C: 0000                      add byte ptr \[eax\],al   
 0001422E: 0000                      add byte ptr \[eax\],al   
 00014230: 0000                      add byte ptr \[eax\],al   
 00014232: 0000                      add byte ptr \[eax\],al   
 00014234: 0000                      add byte ptr \[eax\],al   
 00014236: 0000                      add byte ptr \[eax\],al   
 00014238: 0000                      add byte ptr \[eax\],al   
 0001423A: 0000                      add byte ptr \[eax\],al   
 0001423C: 0000                      add byte ptr \[eax\],al   
 0001423E: 0000                      add byte ptr \[eax\],al   
 00014240: 0000                      add byte ptr \[eax\],al   
 00014242: 0000                      add byte ptr \[eax\],al   
 00014244: 0000                      add byte ptr \[eax\],al   
 00014246: 0000                      add byte ptr \[eax\],al   
 00014248: 0000                      add byte ptr \[eax\],al   
 0001424A: 0000                      add byte ptr \[eax\],al   
 0001424C: 0000                      add byte ptr \[eax\],al   
 0001424E: 0000                      add byte ptr \[eax\],al   
 00014250: 0000                      add byte ptr \[eax\],al   
 00014252: 0000                      add byte ptr \[eax\],al   
 00014254: 0000                      add byte ptr \[eax\],al   
 00014256: 0000                      add byte ptr \[eax\],al   
 00014258: 0000                      add byte ptr \[eax\],al   
 0001425A: 0000                      add byte ptr \[eax\],al   
 0001425C: 0000                      add byte ptr \[eax\],al   
 0001425E: 0000                      add byte ptr \[eax\],al   
 00014260: 0000                      add byte ptr \[eax\],al   
 00014262: 0000                      add byte ptr \[eax\],al   
 00014264: 0000                      add byte ptr \[eax\],al   
 00014266: 0000                      add byte ptr \[eax\],al   
 00014268: 0000                      add byte ptr \[eax\],al   
 0001426A: 0000                      add byte ptr \[eax\],al   
 0001426C: 0000                      add byte ptr \[eax\],al   
 0001426E: 0000                      add byte ptr \[eax\],al   
 00014270: 0000                      add byte ptr \[eax\],al   
 00014272: 0000                      add byte ptr \[eax\],al   
 00014274: 0000                      add byte ptr \[eax\],al   
 00014276: 0000                      add byte ptr \[eax\],al   
 00014278: 0000                      add byte ptr \[eax\],al   
 0001427A: 0000                      add byte ptr \[eax\],al   
 0001427C: 0000                      add byte ptr \[eax\],al   
 0001427E: 0000                      add byte ptr \[eax\],al   
 00014280: 0000                      add byte ptr \[eax\],al   
 00014282: 0000                      add byte ptr \[eax\],al   
 00014284: 0000                      add byte ptr \[eax\],al   
 00014286: 0000                      add byte ptr \[eax\],al   
 00014288: 0000                      add byte ptr \[eax\],al   
 0001428A: 0000                      add byte ptr \[eax\],al   
 0001428C: 0000                      add byte ptr \[eax\],al   
 0001428E: 0000                      add byte ptr \[eax\],al   
 00014290: 0000                      add byte ptr \[eax\],al   
 00014292: 0000                      add byte ptr \[eax\],al   
 00014294: 0000                      add byte ptr \[eax\],al   
 00014296: 0000                      add byte ptr \[eax\],al   
 00014298: 0000                      add byte ptr \[eax\],al   
 0001429A: 0000                      add byte ptr \[eax\],al   
 0001429C: 0000                      add byte ptr \[eax\],al   
 0001429E: 0000                      add byte ptr \[eax\],al   
 000142A0: 0000                      add byte ptr \[eax\],al   
 000142A2: 0000                      add byte ptr \[eax\],al   
 000142A4: 0000                      add byte ptr \[eax\],al   
 000142A6: 0000                      add byte ptr \[eax\],al   
 000142A8: 0000                      add byte ptr \[eax\],al   
 000142AA: 0000                      add byte ptr \[eax\],al   
 000142AC: 0000                      add byte ptr \[eax\],al   
 000142AE: 0000                      add byte ptr \[eax\],al   
 000142B0: 0000                      add byte ptr \[eax\],al   
 000142B2: 0000                      add byte ptr \[eax\],al   
 000142B4: 0000                      add byte ptr \[eax\],al   
 000142B6: 0000                      add byte ptr \[eax\],al   
 000142B8: 0000                      add byte ptr \[eax\],al   
 000142BA: 0000                      add byte ptr \[eax\],al   
 000142BC: 0000                      add byte ptr \[eax\],al   
 000142BE: 0000                      add byte ptr \[eax\],al   
 000142C0: 0000                      add byte ptr \[eax\],al   
 000142C2: 0000                      add byte ptr \[eax\],al   
 000142C4: 0000                      add byte ptr \[eax\],al   
 000142C6: 0000                      add byte ptr \[eax\],al   
 000142C8: 0000                      add byte ptr \[eax\],al   
 000142CA: 0000                      add byte ptr \[eax\],al   
 000142CC: 0000                      add byte ptr \[eax\],al   
 000142CE: 0000                      add byte ptr \[eax\],al   
 000142D0: 7C7C                      jl 0001434E   
 000142D2: 0020                      add byte ptr \[eax\],ah   
 000142D4: 2020                      and byte ptr \[eax\],ah   
 000142D6: 2020                      and byte ptr \[eax\],ah   
 000142D8: 2020                      and byte ptr \[eax\],ah   
 000142DA: CAD4D3                    retf D3D4   
 000142DD: C3                        ret   
 000142DE: C6DAD2                    mov dl,D2   
 000142E1: D1B9FDA1A320              sar dword ptr \[ecx+20A3A1FD\],1   
 000142E7: 0D0A0D0A00                or eax,000A0D0A   
 000142EC: 0000                      add byte ptr \[eax\],al   
 000142EE: 0000                      add byte ptr \[eax\],al   
 000142F0: 0000                      add byte ptr \[eax\],al   
 000142F2: 0000                      add byte ptr \[eax\],al   
 000142F4: 0000                      add byte ptr \[eax\],al   
 000142F6: 0000                      add byte ptr \[eax\],al   
 000142F8: 0000                      add byte ptr \[eax\],al   
 000142FA: 0000                      add byte ptr \[eax\],al   
 000142FC: 0000                      add byte ptr \[eax\],al   
 000142FE: 0000                      add byte ptr \[eax\],al   
 00014300: 0000                      add byte ptr \[eax\],al   
 00014302: 0000                      add byte ptr \[eax\],al   
 00014304: 0000                      add byte ptr \[eax\],al   
 00014306: 0000                      add byte ptr \[eax\],al   
 00014308: 0000                      add byte ptr \[eax\],al   
 0001430A: 0000                      add byte ptr \[eax\],al   
 0001430C: 0000                      add byte ptr \[eax\],al   
 0001430E: 0000                      add byte ptr \[eax\],al   
 00014310: 0000                      add byte ptr \[eax\],al   
 00014312: 0000                      add byte ptr \[eax\],al   
 00014314: 0000                      add byte ptr \[eax\],al   
 00014316: 0000                      add byte ptr \[eax\],al   
 00014318: 0000                      add byte ptr \[eax\],al   
 0001431A: 0000                      add byte ptr \[eax\],al   
 0001431C: 0000                      add byte ptr \[eax\],al   
 0001431E: 0000                      add byte ptr \[eax\],al   
 00014320: 0000                      add byte ptr \[eax\],al   
 00014322: 0000                      add byte ptr \[eax\],al   
 00014324: 0000                      add byte ptr \[eax\],al   
 00014326: 0000                      add byte ptr \[eax\],al   
 00014328: 0000                      add byte ptr \[eax\],al   
 0001432A: 0000                      add byte ptr \[eax\],al   
 0001432C: 0000                      add byte ptr \[eax\],al   
 0001432E: 0000                      add byte ptr \[eax\],al   
 00014330: 0000                      add byte ptr \[eax\],al   
 00014332: 0000                      add byte ptr \[eax\],al   
 00014334: 0000                      add byte ptr \[eax\],al   
 00014336: 0000                      add byte ptr \[eax\],al   
 00014338: 0000                      add byte ptr \[eax\],al   
 0001433A: 0000                      add byte ptr \[eax\],al   
 0001433C: 0000                      add byte ptr \[eax\],al   
 0001433E: 0000                      add byte ptr \[eax\],al   
 00014340: 0000                      add byte ptr \[eax\],al   
 00014342: 0000                      add byte ptr \[eax\],al   
 00014344: 0000                      add byte ptr \[eax\],al   
 00014346: 0000                      add byte ptr \[eax\],al   
 00014348: 0000                      add byte ptr \[eax\],al   
 0001434A: 0000                      add byte ptr \[eax\],al   
 0001434C: 0000                      add byte ptr \[eax\],al   
 0001434E: 0000                      add byte ptr \[eax\],al   
 00014350: 0000                      add byte ptr \[eax\],al   
 00014352: 0000                      add byte ptr \[eax\],al   
 00014354: 0000                      add byte ptr \[eax\],al   
 00014356: 0000                      add byte ptr \[eax\],al   
 00014358: 0000                      add byte ptr \[eax\],al   
 0001435A: 0000                      add byte ptr \[eax\],al   
 0001435C: 0000                      add byte ptr \[eax\],al   
 0001435E: 0000                      add byte ptr \[eax\],al   
 00014360: 0000                      add byte ptr \[eax\],al   
 00014362: 0000                      add byte ptr \[eax\],al   
 00014364: 0000                      add byte ptr \[eax\],al   
 00014366: 0000                      add byte ptr \[eax\],al   
 00014368: 0000                      add byte ptr \[eax\],al   
 0001436A: 0000                      add byte ptr \[eax\],al   
 0001436C: 0000                      add byte ptr \[eax\],al   
 0001436E: 0000                      add byte ptr \[eax\],al   
 00014370: 0000                      add byte ptr \[eax\],al   
 00014372: 0000                      add byte ptr \[eax\],al   
 00014374: 0000                      add byte ptr \[eax\],al   
 00014376: 0000                      add byte ptr \[eax\],al   
 00014378: 0000                      add byte ptr \[eax\],al   
 0001437A: 0000                      add byte ptr \[eax\],al   
 0001437C: 0000                      add byte ptr \[eax\],al   
 0001437E: 0000                      add byte ptr \[eax\],al   
 00014380: 0000                      add byte ptr \[eax\],al   
 00014382: 0000                      add byte ptr \[eax\],al   
 00014384: 0000                      add byte ptr \[eax\],al   
 00014386: 0000                      add byte ptr \[eax\],al   
 00014388: 0000                      add byte ptr \[eax\],al   
 0001438A: 0000                      add byte ptr \[eax\],al   
 0001438C: 0000                      add byte ptr \[eax\],al   
 0001438E: 0000                      add byte ptr \[eax\],al   
 00014390: 0000                      add byte ptr \[eax\],al   
 00014392: 0000                      add byte ptr \[eax\],al   
 00014394: 0000                      add byte ptr \[eax\],al   
 00014396: 0000                      add byte ptr \[eax\],al   
 00014398: 0000                      add byte ptr \[eax\],al   
 0001439A: 0000                      add byte ptr \[eax\],al   
 0001439C: 0000                      add byte ptr \[eax\],al   
 0001439E: 0000                      add byte ptr \[eax\],al   
 000143A0: 0000                      add byte ptr \[eax\],al   
 000143A2: 0000                      add byte ptr \[eax\],al   
 000143A4: 0000                      add byte ptr \[eax\],al   
 000143A6: 0000                      add byte ptr \[eax\],al   
 000143A8: 0000                      add byte ptr \[eax\],al   
 000143AA: 0000                      add byte ptr \[eax\],al   
 000143AC: 0000                      add byte ptr \[eax\],al   
 000143AE: 0000                      add byte ptr \[eax\],al   
 000143B0: 0000                      add byte ptr \[eax\],al   
 000143B2: 0000                      add byte ptr \[eax\],al   
 000143B4: 0000                      add byte ptr \[eax\],al   
 000143B6: 0000                      add byte ptr \[eax\],al   
 000143B8: 0000                      add byte ptr \[eax\],al   
 000143BA: 0000                      add byte ptr \[eax\],al   
 000143BC: 0000                      add byte ptr \[eax\],al   
 000143BE: 0000                      add byte ptr \[eax\],al   
 000143C0: 0000                      add byte ptr \[eax\],al   
 000143C2: 0000                      add byte ptr \[eax\],al   
 000143C4: 0000                      add byte ptr \[eax\],al   
 000143C6: 0000                      add byte ptr \[eax\],al   
 000143C8: 0000                      add byte ptr \[eax\],al   
 000143CA: 0000                      add byte ptr \[eax\],al   
 000143CC: 0000                      add byte ptr \[eax\],al   
 000143CE: 0000                      add byte ptr \[eax\],al   
 000143D0: 0000                      add byte ptr \[eax\],al   
 000143D2: 0000                      add byte ptr \[eax\],al   
 000143D4: 0000                      add byte ptr \[eax\],al   
 000143D6: 0000                      add byte ptr \[eax\],al   
 000143D8: 0000                      add byte ptr \[eax\],al   
 000143DA: 0000                      add byte ptr \[eax\],al   
 000143DC: 0000                      add byte ptr \[eax\],al   
 000143DE: 0000                      add byte ptr \[eax\],al   
 000143E0: 0000                      add byte ptr \[eax\],al   
 000143E2: 0000                      add byte ptr \[eax\],al   
 000143E4: 0000                      add byte ptr \[eax\],al   
 000143E6: 0000                      add byte ptr \[eax\],al   
 000143E8: 0000                      add byte ptr \[eax\],al   
 000143EA: 0000                      add byte ptr \[eax\],al   
 000143EC: 0000                      add byte ptr \[eax\],al   
 000143EE: 0000                      add byte ptr \[eax\],al   
 000143F0: 0000                      add byte ptr \[eax\],al   
 000143F2: 0000                      add byte ptr \[eax\],al   
 000143F4: 0000                      add byte ptr \[eax\],al   
 000143F6: 0000                      add byte ptr \[eax\],al   
 000143F8: 0000                      add byte ptr \[eax\],al   
 000143FA: 0000                      add byte ptr \[eax\],al   
 000143FC: 7C00                      jl 000143FE   
 000143FE: 0000                      add byte ptr \[eax\],al   
 00014400: 0000                      add byte ptr \[eax\],al   
 00014402: 0000                      add byte ptr \[eax\],al   
 00014404: 0000                      add byte ptr \[eax\],al   
 00014406: 0000                      add byte ptr \[eax\],al   
 00014408: 0000                      add byte ptr \[eax\],al   
 0001440A: 0000                      add byte ptr \[eax\],al   
 0001440C: 0000                      add byte ptr \[eax\],al   
 0001440E: 0000                      add byte ptr \[eax\],al   
 00014410: 0000                      add byte ptr \[eax\],al   
 00014412: 0000                      add byte ptr \[eax\],al   
 00014414: 0000                      add byte ptr \[eax\],al   
 00014416: 0000                      add byte ptr \[eax\],al   
 00014418: 2020                      and byte ptr \[eax\],ah   
 0001441A: 2020                      and byte ptr \[eax\],ah   
 0001441C: 2020                      and byte ptr \[eax\],ah   
 0001441E: 2020                      and byte ptr \[eax\],ah   
 00014420: 2020                      and byte ptr \[eax\],ah   
 00014422: 2020                      and byte ptr \[eax\],ah   
 00014424: 2020                      and byte ptr \[eax\],ah   
 00014426: 2020                      and byte ptr \[eax\],ah   
 00014428: 20BBB6D3ADC4              and byte ptr \[ebx-3B522C4A\],bh   
 0001442E: E3CA                      jecxz 000143FA   
 00014430: B9D3C3BBC3                mov ecx,C3BBC3D3   
 00014435: D3B020312E35              shl dword ptr \[eax+352E3120\],cl   
 0001443B: 206265                    and byte ptr \[edx+65\],ah   
 0001443E: 7461                      jz 000144A1   
 00014440: 32A3AC0D0A0D              xor ah,byte ptr \[ebx+0D0A0DAC\]   
 00014446: 0A20                      or ah,byte ptr \[eax\]   
 00014448: 2020                      and byte ptr \[eax\],ah   
 0001444A: BBC3D3B076                mov ebx,76B0D3C3   
 0001444F: 312E                      xor dword ptr \[esi\],ebp   
 00014451: 35CAC7B9B2                xor eax,B2B9C7CA   
 00014456: CF                        iretd    
 00014457: ED                        in eax,dx   
 00014458: C8EDBCFE                  enter BCED,FE   
 0001445C: A3ACC8E7B9                mov \[B9E7C8AC\],eax   
 00014461: FB                        sti   
 00014462: C4E3                      les esp,(Invalid)bx   
 00014464: CF                        iretd    
 00014465: B2BB                      mov dl,BB   
 00014467: B6CB                      mov dh,CB   
 00014469: FC                        cld   
 0001446A: A3ACCFA3CD                mov \[CDA3CFAC\],eax   
 0001446F: FB                        sti   
 00014470: C4E3                      les esp,(Invalid)bx   
 00014472: D7                        xlat   
 00014473: A2B2E1A1A3                mov \[A3A1E1B2\],al   
 00014478: 200D0A0D0A20              and byte ptr \[200A0D0A\],cl   
 0001447E: 2020                      and byte ptr \[eax\],ah   
 00014480: 2020                      and byte ptr \[eax\],ah   
 00014482: 2020                      and byte ptr \[eax\],ah   
 00014484: 2020                      and byte ptr \[eax\],ah   
 00014486: 2020                      and byte ptr \[eax\],ah   
 00014488: 2020                      and byte ptr \[eax\],ah   
 0001448A: 2020                      and byte ptr \[eax\],ah   
 0001448C: 2020                      and byte ptr \[eax\],ah   
 0001448E: 200D0A0D0A20              and byte ptr \[200A0D0A\],cl   
 00014494: 2020                      and byte ptr \[eax\],ah   
 00014496: 2020                      and byte ptr \[eax\],ah   
 00014498: 2020                      and byte ptr \[eax\],ah   
 0001449A: 2020                      and byte ptr \[eax\],ah   
 0001449C: 2020                      and byte ptr \[eax\],ah   
 0001449E: 2020                      and byte ptr \[eax\],ah   
 000144A0: 2020                      and byte ptr \[eax\],ah   
 000144A2: 2020                      and byte ptr \[eax\],ah   
 000144A4: 2020                      and byte ptr \[eax\],ah   
 000144A6: 45                        inc ebp   
 000144A7: 2D6D61696C                sub eax,6C69616D   
 000144AC: 2020                      and byte ptr \[eax\],ah   
 000144AE: 203A                      and byte ptr \[edx\],bh   
 000144B0: 2020                      and byte ptr \[eax\],ah   
 000144B2: 2020                      and byte ptr \[eax\],ah   
 000144B4: 2038                      and byte ptr \[eax\],bh   
 000144B6: 3530343400                xor eax,00343430   
 000144BB: 0000                      add byte ptr \[eax\],al   
 000144BD: 0000                      add byte ptr \[eax\],al   
 000144BF: 0000                      add byte ptr \[eax\],al   
 000144A0: 2020                      and byte ptr \[eax\],ah   
 000144A2: 2020                      and byte ptr \[eax\],ah   
 000144A4: 2020                      and byte ptr \[eax\],ah   
 000144A6: 45                        inc ebp   
 000144A7: 2D6D61696C                sub eax,6C69616D   
 000144AC: 2020                      and byte ptr \[eax\],ah   
 000144AE: 203A                      and byte ptr \[edx\],bh   
 000144B0: 2064696E                  and byte ptr \[ecx+ebp\*2+6E\],ah   
 000144B4: 672D626F7940              sub eax,40796F62   
 000144BA: 6E                        outsb dx,ds:\[esi\]   
 000144BB: 657465                    jz 00014523   
 000144BE: 61                        popad    
 000144BF: 7365                      jnb 00014526   
 000144C1: 2E636F6D                  arpl word ptr cs:\[edi+6D\],bp   
 000144C5: 200D0A202020              and byte ptr \[2020200A\],cl   
 000144CB: 2020                      and byte ptr \[eax\],ah   
 000144CD: 2020                      and byte ptr \[eax\],ah   
 000144CF: 2020                      and byte ptr \[eax\],ah   
 000144D1: 2020                      and byte ptr \[eax\],ah   
 000144D3: 2020                      and byte ptr \[eax\],ah   
 000144D5: 2020                      and byte ptr \[eax\],ah   
 000144D7: 2020                      and byte ptr \[eax\],ah   
 000144D9: 2020                      and byte ptr \[eax\],ah   
 000144DB: 48                        dec eax   
 000144DC: 6F                        outsd dx,ds:\[esi\]   
 000144DD: 6D                        insd es:\[edi\],dx   
 000144DE: 657061                    jo 00014542   
 000144E1: 6765203A                  and byte ptr gs:\[bp+si\],bh   
 000144E5: 206874                    and byte ptr \[eax+74\],ch   
 000144E8: 7470                      jz 0001455A   
 000144EA: 3A2F                      cmp ch,byte ptr \[edi\]   
 000144EC: 2F                        das   
 000144ED: 667364                    jnb 00014554   
 000144F0: 622E                      bound ebp,dword ptr \[esi\]   
 000144F2: 7965                      jns 00014559   
 000144F4: 61                        popad    
 000144F5: 682E6E6574                push 74656E2E   
 000144FA: 2020                      and byte ptr \[eax\],ah   
 000144FC: 2020                      and byte ptr \[eax\],ah   
 000144FE: 2020                      and byte ptr \[eax\],ah   
 00014500: 2020                      and byte ptr \[eax\],ah   
 00014502: 2020                      and byte ptr \[eax\],ah   
 00014504: 2020                      and byte ptr \[eax\],ah   
 00014506: 200D0A0D0A00              and byte ptr \[000A0D0A\],cl   
 0001450C: 0000                      add byte ptr \[eax\],al   
 0001450E: 0000                      add byte ptr \[eax\],al   
 00014510: 0000                      add byte ptr \[eax\],al   
 00014512: 0000                      add byte ptr \[eax\],al   
 00014514: 0000                      add byte ptr \[eax\],al   
 00014516: 0000                      add byte ptr \[eax\],al   
 00014518: 0000                      add byte ptr \[eax\],al   
 0001451A: 0000                      add byte ptr \[eax\],al   
 0001451C: 0000                      add byte ptr \[eax\],al   
 0001451E: 0000                      add byte ptr \[eax\],al   
 00014520: 0000                      add byte ptr \[eax\],al   
 00014522: 0000                      add byte ptr \[eax\],al   
 00014524: 0000                      add byte ptr \[eax\],al   
 00014526: 0000                      add byte ptr \[eax\],al   
 00014528: 0000                      add byte ptr \[eax\],al   
 0001452A: 0000                      add byte ptr \[eax\],al   
 0001452C: 0000                      add byte ptr \[eax\],al   
 0001452E: 0000                      add byte ptr \[eax\],al   
 00014530: 0000                      add byte ptr \[eax\],al   
 00014532: 0000                      add byte ptr \[eax\],al   
 00014534: 0000                      add byte ptr \[eax\],al   
 00014536: 0000                      add byte ptr \[eax\],al   
 00014538: 0000                      add byte ptr \[eax\],al   
 0001453A: 0000                      add byte ptr \[eax\],al   
 0001453C: 0000                      add byte ptr \[eax\],al   
 0001453E: 0000                      add byte ptr \[eax\],al   
 00014540: 0000                      add byte ptr \[eax\],al   
 00014542: 0000                      add byte ptr \[eax\],al   
 00014544: 49                        dec ecx   
 00014545: 6E                        outsb dx,ds:\[esi\]   
 00014546: 7661                      jbe 000145A9   
 00014548: 6C                        insb es:\[edi\],dx   
 00014549: 6964202050617373          imul esp,dword ptr \[eax+20\],73736150   
 00014551: 57                        push edi   
 00014552: 6F                        outsd dx,ds:\[esi\]   
 00014553: 7264                      jb 000145B9   
 00014555: 0000                      add byte ptr \[eax\],al   
 00014557: 0000                      add byte ptr \[eax\],al   
 00014559: 0000                      add byte ptr \[eax\],al   
 0001455B: 0000                      add byte ptr \[eax\],al   
 0001455D: 0000                      add byte ptr \[eax\],al   
 0001455F: 0000                      add byte ptr \[eax\],al   
 00014561: 0000                      add byte ptr \[eax\],al   
 00014563: 0000                      add byte ptr \[eax\],al   
 00014565: 0000                      add byte ptr \[eax\],al   
 00014567: 0000                      add byte ptr \[eax\],al   
 00014569: 0000                      add byte ptr \[eax\],al   
 0001456B: 0000                      add byte ptr \[eax\],al   
 0001456D: 0000                      add byte ptr \[eax\],al   
 0001456F: 0000                      add byte ptr \[eax\],al   
 00014571: 0000                      add byte ptr \[eax\],al   
 00014573: 0000                      add byte ptr \[eax\],al   
 00014575: 0000                      add byte ptr \[eax\],al   
 00014577: 0000                      add byte ptr \[eax\],al   
 00014579: 0000                      add byte ptr \[eax\],al   
 0001457B: 0000                      add byte ptr \[eax\],al   
 0001457D: 0000                      add byte ptr \[eax\],al   
 0001457F: 0000                      add byte ptr \[eax\],al   
 00014581: 0000                      add byte ptr \[eax\],al   
 00014583: 0000                      add byte ptr \[eax\],al   
 00014585: 0000                      add byte ptr \[eax\],al   
 00014587: 0000                      add byte ptr \[eax\],al   
 00014589: 0000                      add byte ptr \[eax\],al   
 0001458B: 0000                      add byte ptr \[eax\],al   
 0001458D: 0000                      add byte ptr \[eax\],al   
 0001458F: 0000                      add byte ptr \[eax\],al   
 00014591: 0000                      add byte ptr \[eax\],al   
 00014593: 0000                      add byte ptr \[eax\],al   
 00014595: 0000                      add byte ptr \[eax\],al   
 00014597: 0000                      add byte ptr \[eax\],al   
 00014599: 0000                      add byte ptr \[eax\],al   
 0001459B: 0000                      add byte ptr \[eax\],al   
 0001459D: 0000                      add byte ptr \[eax\],al   
 0001459F: 0000                      add byte ptr \[eax\],al   
 000145A1: 0000                      add byte ptr \[eax\],al   
 000145A3: 0000                      add byte ptr \[eax\],al   
 000145A5: 0000                      add byte ptr \[eax\],al   
 000145A7: 005265                    add byte ptr \[edx+65\],dl   
 000145AA: 6769737465722053          imul esi,dword ptr \[bp+di+74\],53207265   
 000145B2: 7563                      jnz 00014617   
 000145B4: 636573                    arpl word ptr \[ebp+73\],sp   
 000145B7: 7366                      jnb 0001461F   
 000145B9: 756C                      jnz 00014627   
 000145BB: 6C                        insb es:\[edi\],dx   
 000145BC: 7900                      jns 000145BE   
 000145BE: 0000                      add byte ptr \[eax\],al   
 000145C0: 0000                      add byte ptr \[eax\],al   
 000145C2: 0000                      add byte ptr \[eax\],al   
 000145C4: 0000                      add byte ptr \[eax\],al   
 000145C6: 0000                      add byte ptr \[eax\],al   
 000145C8: 0000                      add byte ptr \[eax\],al   
 000145CA: 0000                      add byte ptr \[eax\],al   
 000145CC: 0000                      add byte ptr \[eax\],al   
 000145CE: 0000                      add byte ptr \[eax\],al   
 000145D0: 0000                      add byte ptr \[eax\],al   
 000145D2: 0000                      add byte ptr \[eax\],al   
 000145D4: 0000                      add byte ptr \[eax\],al   
 000145D6: 0000                      add byte ptr \[eax\],al   
 000145D8: 0000                      add byte ptr \[eax\],al   
 000145DA: 0000                      add byte ptr \[eax\],al   
 000145DC: 0000                      add byte ptr \[eax\],al   
 000145DE: 0000                      add byte ptr \[eax\],al   
 000145E0: 0000                      add byte ptr \[eax\],al   
 000145E2: 0000                      add byte ptr \[eax\],al   
 000145E4: 0000                      add byte ptr \[eax\],al   
 000145E6: 0000                      add byte ptr \[eax\],al   
 000145E8: 0000                      add byte ptr \[eax\],al   
 000145EA: 0000                      add byte ptr \[eax\],al   
 000145EC: 0000                      add byte ptr \[eax\],al   
 000145EE: 0000                      add byte ptr \[eax\],al   
 000145F0: 0000                      add byte ptr \[eax\],al   
 000145F2: 0000                      add byte ptr \[eax\],al   
 000145F4: 0000                      add byte ptr \[eax\],al   
 000145F6: 0000                      add byte ptr \[eax\],al   
 000145F8: 0000                      add byte ptr \[eax\],al   
 000145FA: 0000                      add byte ptr \[eax\],al   
 000145FC: 0000                      add byte ptr \[eax\],al   
 000145FE: 0000                      add byte ptr \[eax\],al   
 00014600: 0000                      add byte ptr \[eax\],al   
 00014602: 0000                      add byte ptr \[eax\],al   
 00014604: 0000                      add byte ptr \[eax\],al   
 00014606: 0000                      add byte ptr \[eax\],al   
 00014608: 0000                      add byte ptr \[eax\],al   
 0001460A: 0000                      add byte ptr \[eax\],al   
 0001460C: 49                        dec ecx   
 0001460D: 6E                        outsb dx,ds:\[esi\]   
 0001460E: 7661                      jbe 00014671   
 00014610: 6C                        insb es:\[edi\],dx   
 00014611: 6964205365726961          imul esp,dword ptr \[eax+53\],61697265   
 00014619: 6C                        insb es:\[edi\],dx   
 0001461A: 204E4F                    and byte ptr \[esi+4F\],cl   
 0001461D: 0000                      add byte ptr \[eax\],al   
 0001461F: 0000                      add byte ptr \[eax\],al   
 00014621: 0000                      add byte ptr \[eax\],al   
 00014623: 0000                      add byte ptr \[eax\],al   
 00014625: 0000                      add byte ptr \[eax\],al   
 00014627: 0000                      add byte ptr \[eax\],al   
 00014629: 0000                      add byte ptr \[eax\],al   
 0001462B: 0000                      add byte ptr \[eax\],al   
 0001462D: 0000                      add byte ptr \[eax\],al   
 0001462F: 0000                      add byte ptr \[eax\],al   
 00014631: 0000                      add byte ptr \[eax\],al   
 00014633: 0000                      add byte ptr \[eax\],al   
 00014635: 0000                      add byte ptr \[eax\],al   
 00014637: 0000                      add byte ptr \[eax\],al   
 00014639: 0000                      add byte ptr \[eax\],al   
 0001463B: 0000                      add byte ptr \[eax\],al   
 0001463D: 0000                      add byte ptr \[eax\],al   
 0001463F: 0000                      add byte ptr \[eax\],al   
 00014641: 0000                      add byte ptr \[eax\],al   
 00014643: 0000                      add byte ptr \[eax\],al   
 00014645: 0000                      add byte ptr \[eax\],al   
 00014647: 0000                      add byte ptr \[eax\],al   
 00014649: 0000                      add byte ptr \[eax\],al   
 0001464B: 0000                      add byte ptr \[eax\],al   
 0001464D: 0000                      add byte ptr \[eax\],al   
 0001464F: 0000                      add byte ptr \[eax\],al   
 00014651: 0000                      add byte ptr \[eax\],al   
 00014653: 0000                      add byte ptr \[eax\],al   
 00014655: 0000                      add byte ptr \[eax\],al   
 00014657: 0000                      add byte ptr \[eax\],al   
 00014659: 0000                      add byte ptr \[eax\],al   
 0001465B: 0000                      add byte ptr \[eax\],al   
 0001465D: 0000                      add byte ptr \[eax\],al   
 0001465F: 0000                      add byte ptr \[eax\],al   
 00014661: 0000                      add byte ptr \[eax\],al   
 00014663: 0000                      add byte ptr \[eax\],al   
 00014665: 0000                      add byte ptr \[eax\],al   
 00014667: 0000                      add byte ptr \[eax\],al   
 00014669: 0000                      add byte ptr \[eax\],al   
 0001466B: 0000                      add byte ptr \[eax\],al   
 0001466D: 0000                      add byte ptr \[eax\],al   
 0001466F: 004520                    add byte ptr \[ebp+20\],al   
 00014672: 0020                      add byte ptr \[eax\],ah   
 00014674: 0020                      add byte ptr \[eax\],ah   
 00014676: 2020                      and byte ptr \[eax\],ah   
 00014678: 4C                        dec esp   
 00014679: 6F                        outsd dx,ds:\[esi\]   
 0001467A: 61                        popad    
 0001467B: 642020                    and byte ptr fs:\[eax\],ah   
 0001467E: 20446961                  and byte ptr \[ecx+ebp\*2+61\],al   
 00014682: 6C                        insb es:\[edi\],dx   
 00014683: 2E646C                    insb es:\[edi\],dx   
 00014686: 6C                        insb es:\[edi\],dx   
 00014687: 2020                      and byte ptr \[eax\],ah   
 00014689: 6661                      popa    
 0001468B: 6C                        insb es:\[edi\],dx   
 0001468C: 7365                      jnb 000146F3   
 0001468E: 642120                    and dword ptr fs:\[eax\],esp   
 00014691: 2020                      and byte ptr \[eax\],ah   
 00014693: 00D0                      add al,dl   
 00014695: 07                        pop es   
 00014696: 0800                      or byte ptr \[eax\],al   
 00014698: 0400                      add al,00   
 0001469A: 1100                      adc dword ptr \[eax\],eax   
 0001469C: 0800                      or byte ptr \[eax\],al   
 0001469E: 35002300C8                xor eax,C8002300   
 000146A3: 0000                      add byte ptr \[eax\],al   
 000146A5: 346C                      xor al,6C   
 000146A7: 0000                      add byte ptr \[eax\],al   
 000146A9: 0000                      add byte ptr \[eax\],al   
 000146AB: 0000                      add byte ptr \[eax\],al   
   
 \*\*\*\*\*\*\*\*\*\*\*\*\*子程序 ID:CALL000146AD \*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 000146AD: 60                        pushad    
 000146AE: BB85BA4000                mov ebx,0040BA85   
 000146B3: 03DD                      add ebx,ebp   
 000146B5: 807B0800                  cmp byte ptr \[ebx+08\],00   
 000146B9: 746D                      jz 00014728   
 000146BB: 8BC3                      mov eax,ebx   
 000146BD: 83C008                    add eax,00000008   
 000146C0: 50                        push eax   
 000146C1: FF9564BF4000              call dword ptr \[ebp+0040BF64\]     <--LoadLibraryA  
 000146C7: 89857DBA4000              mov dword ptr \[ebp+0040BA7D\],eax   
 000146CD: 83F800                    cmp eax,00000000   
 000146D0: 750C                      jnz 000146DE   
 000146D2: C78581BA400000000000      mov dword ptr \[ebp+0040BA81\],00000000   
 000146DC: EB4A                      jmp 00014728   
   
 000146DE: 8B33                      mov esi,dword ptr \[ebx\]   
 000146E0: 8B7B04                    mov edi,dword ptr \[ebx+04\]   
 000146E3: 03F5                      add esi,ebp   
 000146E5: 03FD                      add edi,ebp   
 000146E7: 803E00                    cmp byte ptr \[esi\],00   
 000146EA: 750E                      jnz 000146FA   
 000146EC: 83C308                    add ebx,00000008   
   
 000146EF: 803B00                    cmp byte ptr \[ebx\],00   
 000146F2: 7403                      jz 000146F7   
 000146F4: 43                        inc ebx   
 000146F5: EBF8                      jmp 000146EF   
   
 000146F7: 43                        inc ebx   
 000146F8: EBBB                      jmp 000146B5   
   
 000146FA: 56                        push esi   
 000146FB: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00014701: FF9568BF4000              call dword ptr \[ebp+0040BF68\]     <--GetProcAddress  
 00014707: 83F800                    cmp eax,00000000   
 0001470A: 750C                      jnz 00014718   
   
 0001470C: C78581BA400000000000      mov dword ptr \[ebp+0040BA81\],00000000   
 00014716: EB10                      jmp 00014728   
   
 00014718: 8907                      mov dword ptr \[edi\],eax   
 0001471A: 83C704                    add edi,00000004   
   
 0001471D: 803E00                    cmp byte ptr \[esi\],00   
 00014720: 7403                      jz 00014725   
 00014722: 46                        inc esi   
 00014723: EBF8                      jmp 0001471D   
   
 00014725: 46                        inc esi   
 00014726: EBBF                      jmp 000146E7   
   
 00014728: 61                        popad    
 00014729: 8B8581BA4000              mov eax,dword ptr \[ebp+0040BA81\]   
 0001472F: C3                        ret   
 \*\*\*\*\*\*\*\*\*\*\*\*结束\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
   
 00014730: 0000                      add byte ptr \[eax\],al   
 00014732: 0000                      add byte ptr \[eax\],al   
 00014734: 0100                      add dword ptr \[eax\],eax   
 00014736: 0000                      add byte ptr \[eax\],al   
 00014738: E3BA                      jecxz 000146F4   
 0001473A: 40                        inc eax   
 0001473B: 009FBB40006B              add byte ptr \[edi+6B0040BB\],bl   
 00014741: 65726E                    jb 000147B2   
 00014744: 656C                      insb es:\[edi\],dx   
 00014746: 3332                      xor esi,dword ptr \[edx\]   
 00014748: 2E646C                    insb es:\[edi\],dx   
 0001474B: 6C                        insb es:\[edi\],dx   
 0001474C: 00D3                      add bl,dl   
 0001474E: BB400000BC                mov ebx,BC000040   
 00014753: 40                        inc eax   
 00014754: 007573                    add byte ptr \[ebp+73\],dh   
 00014757: 657233                    jb 0001478D   
 0001475A: 322E                      xor ch,byte ptr \[esi\]   
 0001475C: 646C                      insb es:\[edi\],dx   
 0001475E: 6C                        insb es:\[edi\],dx   
 0001475F: 0014BC                    add byte ptr \[esp+edi\*4\],dl   
 00014762: 40                        inc eax   
 00014763: 0072BC                    add byte ptr \[edx-44\],dh   
 00014766: 40                        inc eax   
 00014767: 006164                    add byte ptr \[ecx+64\],ah   
 0001476A: 7661                      jbe 000147CD   
 0001476C: 7069                      jo 000147D7   
 0001476E: 3332                      xor esi,dword ptr \[edx\]   
 00014770: 2E646C                    insb es:\[edi\],dx   
 00014773: 6C                        insb es:\[edi\],dx   
 00014774: 008EBC4000B8              add byte ptr \[esi-47FFBF44\],cl   
 0001477A: BC40006469                mov esp,69640040   
 0001477F: 61                        popad    
 00014780: 6C                        insb es:\[edi\],dx   
 00014781: 2E646C                    insb es:\[edi\],dx   
 00014784: 6C                        insb es:\[edi\],dx   
 00014785: 0000                      add byte ptr \[eax\],al   
 00014787: 0000                      add byte ptr \[eax\],al   
 00014789: 0000                      add byte ptr \[eax\],al   
 0001478B: 0000                      add byte ptr \[eax\],al   
 0001478D: 0000                      add byte ptr \[eax\],al   
 0001478F: 0020                      add byte ptr \[eax\],ah   
 00014791: 2020                      and byte ptr \[eax\],ah   
 00014793: 2020                      and byte ptr \[eax\],ah   
 00014795: 383530343400              cmp byte ptr \[00343430\],dh   
 0001479B: 0000                      add byte ptr \[eax\],al   
 0001479D: 0000                      add byte ptr \[eax\],al   
 0001479F: 0000                      add byte ptr \[eax\],al   
 00014780: 6C                        insb es:\[edi\],dx   
 00014781: 2E646C                    insb es:\[edi\],dx   
 00014784: 6C                        insb es:\[edi\],dx   
 00014785: 0000                      add byte ptr \[eax\],al   
 00014787: 0000                      add byte ptr \[eax\],al   
 00014789: 0000                      add byte ptr \[eax\],al   
 0001478B: 0000                      add byte ptr \[eax\],al   
 0001478D: 0000                      add byte ptr \[eax\],al   
 0001478F: 0000                      add byte ptr \[eax\],al   
 00014791: 0000                      add byte ptr \[eax\],al   
 00014793: 0000                      add byte ptr \[eax\],al   
 00014795: 004765                    add byte ptr \[edi+65\],al   
 00014798: 7453                      jz 000147ED   
 0001479A: 7973                      jns 0001480F   
 0001479C: 7465                      jz 00014803   
 0001479E: 6D                        insd es:\[edi\],dx   
 0001479F: 54                        push esp   
 000147A0: 696D6500437265            imul ebp,dword ptr \[ebp+65\],65724300   
 000147A7: 61                        popad    
 000147A8: 7465                      jz 0001480F   
 000147AA: 46                        inc esi   
 000147AB: 696C654100577269          imul ebp,dword ptr \[ebp+41\],69725700   
 000147B3: 7465                      jz 0001481A   
 000147B5: 46                        inc esi   
 000147B6: 696C6500436C6F73          imul ebp,dword ptr \[ebp\],736F6C43   
 000147BE: 6548                      dec eax   
 000147C0: 61                        popad    
 000147C1: 6E                        outsb dx,ds:\[esi\]   
 000147C2: 646C                      insb es:\[edi\],dx   
 000147C4: 65004372                  add byte ptr gs:\[ebx+72\],al   
 000147C8: 6561                      popad    
 000147CA: 7465                      jz 00014831   
 000147CC: 50                        push eax   
 000147CD: 726F                      jb 0001483E   
 000147CF: 636573                    arpl word ptr \[ebp+73\],sp   
 000147D2: 7341                      jnb 00014815   
 000147D4: 004765                    add byte ptr \[edi+65\],al   
 000147D7: 7445                      jz 0001481E   
 000147D9: 7869                      js 00014844   
 000147DB: 7443                      jz 00014820   
 000147DD: 6F                        outsd dx,ds:\[esi\]   
 000147DE: 646550                    push eax   
 000147E1: 726F                      jb 00014852   
 000147E3: 636573                    arpl word ptr \[ebp+73\],sp   
 000147E6: 7300                      jnb 000147E8   
 000147E8: 47                        inc edi   
 000147E9: 657444                    jz 00014830   
 000147EC: 69736B46726565            imul esi,dword ptr \[ebx+6B\],65657246   
 000147F3: 53                        push ebx   
 000147F4: 7061                      jo 00014857   
 000147F6: 636545                    arpl word ptr \[ebp+45\],sp   
 000147F9: 7841                      js 0001483C   
 000147FB: 0044656C                  add byte ptr \[ebp+6C\],al   
 000147FF: 657465                    jz 00014867   
 00014802: 46                        inc esi   
 00014803: 696C654100476574          imul ebp,dword ptr \[ebp+41\],74654700   
 0001480B: 56                        push esi   
 0001480C: 6F                        outsd dx,ds:\[esi\]   
 0001480D: 6C                        insb es:\[edi\],dx   
 0001480E: 756D                      jnz 0001487D   
 00014810: 6549                      dec ecx   
 00014812: 6E                        outsb dx,ds:\[esi\]   
 00014813: 666F                      outsw dx,ds:\[esi\]   
 00014815: 726D                      jb 00014884   
 00014817: 61                        popad    
 00014818: 7469                      jz 00014883   
 0001481A: 6F                        outsd dx,ds:\[esi\]   
 0001481B: 6E                        outsb dx,ds:\[esi\]   
 0001481C: 41                        inc ecx   
 0001481D: 004765                    add byte ptr \[edi+65\],al   
 00014820: 744D                      jz 0001486F   
 00014822: 6F                        outsd dx,ds:\[esi\]   
 00014823: 64756C                    jnz 00014892   
 00014826: 6548                      dec eax   
 00014828: 61                        popad    
 00014829: 6E                        outsb dx,ds:\[esi\]   
 0001482A: 646C                      insb es:\[edi\],dx   
 0001482C: 6541                      inc ecx   
 0001482E: 005265                    add byte ptr \[edx+65\],dl   
 00014831: 61                        popad    
 00014832: 6450                      push eax   
 00014834: 726F                      jb 000148A5   
 00014836: 636573                    arpl word ptr \[ebp+73\],sp   
 00014839: 734D                      jnb 00014888   
 0001483B: 656D                      insd es:\[edi\],dx   
 0001483D: 6F                        outsd dx,ds:\[esi\]   
 0001483E: 7279                      jb 000148B9   
 00014840: 005669                    add byte ptr \[esi+69\],dl   
 00014843: 7274                      jb 000148B9   
 00014845: 7561                      jnz 000148A8   
 00014847: 6C                        insb es:\[edi\],dx   
 00014848: 41                        inc ecx   
 00014849: 6C                        insb es:\[edi\],dx   
 0001484A: 6C                        insb es:\[edi\],dx   
 0001484B: 6F                        outsd dx,ds:\[esi\]   
 0001484C: 6300                      arpl word ptr \[eax\],ax   
 0001484E: 0000                      add byte ptr \[eax\],al   
 00014850: 0000                      add byte ptr \[eax\],al   
 00014852: 0000                      add byte ptr \[eax\],al   
 00014854: 0000                      add byte ptr \[eax\],al   
 00014856: 0000                      add byte ptr \[eax\],al   
 00014858: 0000                      add byte ptr \[eax\],al   
 0001485A: 0000                      add byte ptr \[eax\],al   
 0001485C: 0000                      add byte ptr \[eax\],al   
 0001485E: 0000                      add byte ptr \[eax\],al   
 00014860: 0000                      add byte ptr \[eax\],al   
 00014862: 0000                      add byte ptr \[eax\],al   
 00014864: 0000                      add byte ptr \[eax\],al   
 00014866: 0000                      add byte ptr \[eax\],al   
 00014868: 0000                      add byte ptr \[eax\],al   
 0001486A: 0000                      add byte ptr \[eax\],al   
 0001486C: 0000                      add byte ptr \[eax\],al   
 0001486E: 0000                      add byte ptr \[eax\],al   
 00014870: 0000                      add byte ptr \[eax\],al   
 00014872: 0000                      add byte ptr \[eax\],al   
 00014874: 0000                      add byte ptr \[eax\],al   
 00014876: 0000                      add byte ptr \[eax\],al   
 00014878: 0000                      add byte ptr \[eax\],al   
 0001487A: 0000                      add byte ptr \[eax\],al   
 0001487C: 0000                      add byte ptr \[eax\],al   
 0001487E: 0000                      add byte ptr \[eax\],al   
 00014880: 0000                      add byte ptr \[eax\],al   
 00014882: 0000                      add byte ptr \[eax\],al   
 00014884: 0000                      add byte ptr \[eax\],al   
 00014886: 4D                        dec ebp   
 00014887: 657373                    jnb 000148FD   
 0001488A: 61                        popad    
 0001488B: 676542                    inc edx   
 0001488E: 6F                        outsd dx,ds:\[esi\]   
 0001488F: 7841                      js 000148D2   
 00014891: 007773                    add byte ptr \[edi+73\],dh   
 00014894: 7072                      jo 00014908   
 00014896: 696E7466410053            imul ebp,dword ptr \[esi+74\],53004166   
 0001489D: 657454                    jz 000148F4   
 000148A0: 696D6572004B69            imul ebp,dword ptr \[ebp+65\],694B0072   
 000148A7: 6C                        insb es:\[edi\],dx   
 000148A8: 6C                        insb es:\[edi\],dx   
 000148A9: 54                        push esp   
 000148AA: 696D6572000000            imul ebp,dword ptr \[ebp+65\],00000072   
 000148B1: 0000                      add byte ptr \[eax\],al   
 000148B3: 0000                      add byte ptr \[eax\],al   
 000148B5: 0000                      add byte ptr \[eax\],al   
 000148B7: 0000                      add byte ptr \[eax\],al   
 000148B9: 0000                      add byte ptr \[eax\],al   
 000148BB: 0000                      add byte ptr \[eax\],al   
 000148BD: 0000                      add byte ptr \[eax\],al   
 000148BF: 0000                      add byte ptr \[eax\],al   
 000148C1: 0000                      add byte ptr \[eax\],al   
 000148C3: 0000                      add byte ptr \[eax\],al   
 000148C5: 0000                      add byte ptr \[eax\],al   
 000148C7: 52                        push edx   
 000148C8: 65674F                    dec edi   
 000148CB: 7065                      jo 00014932   
 000148CD: 6E                        outsb dx,ds:\[esi\]   
 000148CE: 4B                        dec ebx   
 000148CF: 657945                    jns 00014917   
 000148D2: 7841                      js 00014915   
 000148D4: 005265                    add byte ptr \[edx+65\],dl   
 000148D7: 6743                      inc ebx   
 000148D9: 7265                      jb 00014940   
 000148DB: 61                        popad    
 000148DC: 7465                      jz 00014943   
 000148DE: 4B                        dec ebx   
 000148DF: 657945                    jns 00014927   
 000148E2: 7841                      js 00014925   
 000148E4: 005265                    add byte ptr \[edx+65\],dl   
 000148E7: 6743                      inc ebx   
 000148E9: 6C                        insb es:\[edi\],dx   
 000148EA: 6F                        outsd dx,ds:\[esi\]   
 000148EB: 7365                      jnb 00014952   
 000148ED: 4B                        dec ebx   
 000148EE: 657900                    jns 000148F1   
 000148F1: 52                        push edx   
 000148F2: 656751                    push ecx   
 000148F5: 7565                      jnz 0001495C   
 000148F7: 7279                      jb 00014972   
 000148F9: 56                        push esi   
 000148FA: 61                        popad    
 000148FB: 6C                        insb es:\[edi\],dx   
 000148FC: 7565                      jnz 00014963   
 000148FE: 45                        inc ebp   
 000148FF: 7841                      js 00014942   
 00014901: 005265                    add byte ptr \[edx+65\],dl   
 00014904: 6753                      push ebx   
 00014906: 657456                    jz 0001495F   
 00014909: 61                        popad    
 0001490A: 6C                        insb es:\[edi\],dx   
 0001490B: 7565                      jnz 00014972   
 0001490D: 45                        inc ebp   
 0001490E: 7841                      js 00014951   
 00014910: 005265                    add byte ptr \[edx+65\],dl   
 00014913: 6744                      inc esp   
 00014915: 656C                      insb es:\[edi\],dx   
 00014917: 657465                    jz 0001497F   
 0001491A: 56                        push esi   
 0001491B: 61                        popad    
 0001491C: 6C                        insb es:\[edi\],dx   
 0001491D: 7565                      jnz 00014984   
 0001491F: 41                        inc ecx   
 00014920: 0000                      add byte ptr \[eax\],al   
 00014922: 0000                      add byte ptr \[eax\],al   
 00014924: 0000                      add byte ptr \[eax\],al   
 00014926: 0000                      add byte ptr \[eax\],al   
 00014928: 0000                      add byte ptr \[eax\],al   
 0001492A: 0000                      add byte ptr \[eax\],al   
 0001492C: 0000                      add byte ptr \[eax\],al   
 0001492E: 0000                      add byte ptr \[eax\],al   
 00014930: 0000                      add byte ptr \[eax\],al   
 00014932: 0000                      add byte ptr \[eax\],al   
 00014934: 0000                      add byte ptr \[eax\],al   
 00014936: 0000                      add byte ptr \[eax\],al   
 00014938: 0000                      add byte ptr \[eax\],al   
 0001493A: 0000                      add byte ptr \[eax\],al   
 0001493C: 0000                      add byte ptr \[eax\],al   
 0001493E: 0000                      add byte ptr \[eax\],al   
 00014940: 004765                    add byte ptr \[edi+65\],al   
 00014943: 7450                      jz 00014995   
 00014945: 61                        popad    
 00014946: 7373                      jnb 000149BB   
 00014948: 776F                      jnbe 000149B9   
 0001494A: 7264                      jb 000149B0   
 0001494C: 004765                    add byte ptr \[edi+65\],al   
 0001494F: 7452                      jz 000149A3   
 00014951: 656769737465720053        imul esi,dword ptr gs:\[bp+di+74\],53007265   
 0001495A: 686F775472                push 7254776F   
 0001495F: 7957                      jns 000149B8   
 00014961: 696E646F770000            imul ebp,dword ptr \[esi+64\],0000776F   
 00014968: 0000                      add byte ptr \[eax\],al   
 0001496A: 0000                      add byte ptr \[eax\],al   
 0001496C: 0000                      add byte ptr \[eax\],al   
 0001496E: 0000                      add byte ptr \[eax\],al   
 00014970: 0000                      add byte ptr \[eax\],al   
 00014972: 0000                      add byte ptr \[eax\],al   
 00014974: 0000                      add byte ptr \[eax\],al   
 00014976: 0000                      add byte ptr \[eax\],al   
 00014978: 0000                      add byte ptr \[eax\],al   
 0001497A: 0000                      add byte ptr \[eax\],al   
 0001497C: 0000                      add byte ptr \[eax\],al   
 0001497E: 0000                      add byte ptr \[eax\],al   
 00014980: 0000                      add byte ptr \[eax\],al   
 00014982: 0000                      add byte ptr \[eax\],al   
 00014984: 0000                      add byte ptr \[eax\],al   
   
 00014986: 00608B                    add byte ptr \[eax-75\],ah   
 00014989: E880BD5DBD                call BD5F070E   
 0001498E: 40                        inc eax   
 0001498F: 0000                      add byte ptr \[eax\],al   
 00014991: 7555                      jnz 000149E8   
 00014993: BE63BD4000                mov esi,0040BD63   
 00014998: 03F5                      add esi,ebp   
 0001499A: C606C3                    mov byte ptr \[esi\],C3   
 0001499D: 8B8573BE4000              mov eax,dword ptr \[ebp+0040BE73\]   
 000149A3: 83F800                    cmp eax,00000000   
 000149A6: 7407                      jz 000149AF        <---不跳就死定了  
    
 000149A8: 898577BE4000              mov dword ptr \[ebp+0040BE77\],eax   
 000149AE: CC                        int 3   
   
 000149AF: BE53BD4000                mov esi,0040BD53   
 000149B4: 03F5                      add esi,ebp   
 000149B6: 0F010E                    sidt dword ptr \[esi\]   
 000149B9: 8B7602                    mov esi,dword ptr \[esi+02\]   
 000149BC: 668B8567BE4000            mov ax,word ptr \[ebp+0040BE67\]   
 000149C3: 668B9D69BE4000            mov bx,word ptr \[ebp+0040BE69\]   
 000149CA: 66894608                  mov word ptr \[esi+08\],ax   
 000149CE: 66895E0E                  mov word ptr \[esi+0E\],bx   
 000149D2: 668B856BBE4000            mov ax,word ptr \[ebp+0040BE6B\]   
 000149D9: 668B9D6DBE4000            mov bx,word ptr \[ebp+0040BE6D\]   
 000149E0: 66894618                  mov word ptr \[esi+18\],ax   
 000149E4: 66895E1E                  mov word ptr \[esi+1E\],bx   
 000149E8: 61                        popad    
 000149E9: 56                        push esi   
 000149EA: 51                        push ecx   
 000149EB: BE4D534000                mov esi,0040534D   
 000149F0: 03F0                      add esi,eax   
 000149F2: B9F7690000                mov ecx,000069F7   
 000149F7: C60600                    mov byte ptr \[esi\],00   
 000149FA: 46                        inc esi   
 000149FB: E2FA                      loop 000149F7           <--灭迹  
 000149FD: 59                        pop ecx   
 000149FE: 5E                        pop esi   
 000149FF: 58                        pop eax   
 00014A00: FFE0                      jmp eax              <--跳到入口处，EAX=OEP  
 00014A02: 00EB                      add bl,ch