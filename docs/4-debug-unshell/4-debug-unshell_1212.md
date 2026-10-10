

# 幻影 v1.5b3 脱壳分析笔记

> 原文：[`www.pediy.com/kssd/pediy03/crack31.htm`](https://www.pediy.com/kssd/pediy03/crack31.htm)

幻影 v1.5b3 脱壳分析笔记 --- ljttt 开门一帖  
  
【声明】  
我写文章以交流为主，希望大家在转载时能保持文章的完整性。  
  
\[作者\]:    ljttt  
\[日期\]:    2000-09-15  
  
【前言】  
首先，我要这儿多谢 wind\[CCG\]。  
  
其中为了补充了一些说明，为了大家好理解。用 \[注：\]说明。我想根据其中的内容去写个脱壳机也不难。  
  
这里有当初我分析幻影的两篇类似的分析笔记，当初为了分析方便，做了两份，各有一些说明（<--所指的内容）。互有补充。大家看看  
  
\[注：由于这是以前分析的笔记，只是为了自己分析方便，没有想到要做为文章，所以现在补充一些内容，让大家好理解\]  
  
\[注：开始幻影用了大量的 XOR 还原方法，共有 223 处，而且，还在其中用到对 IDT 表的修改，所以动态分析不太可能，静态分析也要用编个程序批处理才能得到这些内容\]  
  
\[注：这是我现在找的一段，和以前分析的地址不对应，但我只为了说明问题，这样的代码段大同小异，其中的说明也是我补充的\]  
 0000003A: FA                        cli   
 0000003B: 56                        push esi   
 0000003C: BE53BD4000                mov esi,0040BD53   
 00000041: 03F5                      add esi,ebp   
 00000043: 0F010E                    sidt dword ptr \[esi\]        <--取 IDTR 的内容  
 00000046: 8B7602                    mov esi,dword ptr \[esi+02\]      <--获取 IDT 表的基地址  
 00000049: 66FF4E08                  dec word ptr \[esi+08\]        <--int 1 的低位偏移减一，大家应该知道这是干嘛的。  
 0000004D: FF8DCFB24000              dec dword ptr \[ebp+0040B2CF\]    <--注意，这里修改了以后要 XOR 还原的一处内容，所以算个小干扰  
 00000053: 5E                        pop esi   
 00000054: B927000000                mov ecx,00000027          <--还原开始  
 00000059: 3006                      xor byte ptr \[esi\],al        <--al 为 XOR 用的字节  
 0000005B: 46                        inc esi   
 0000005C: 49                        dec ecx   
 0000005D: 75FA                      jnz 00000059            <--还原结束  
 0000005F: FEC8                      dec al                <--变换，下一次用作 XOR 还原  
 00000061: FA                        cli   
  
\[注：在经过以上的 XOR 还原后，来到这里，将 XOR 还原出所有幻影的代码了，以前我给大家说明的一些内容都可以在这里找到\]  
  
 00412382: E800000000                call 00412387   
 00412387: 5D                        pop ebp   
 00412388: 8BD5                      mov edx,ebp   
 0041238A: 81EDD4A64000              sub ebp,0040A6D4    <--EBP=412388-40A6D4=7CB4  
 00412390: FA                        cli   
 00412391: 58                        pop eax   
 00412392: 80E401                    and ah,01   
 00412395: 32C0                      xor al,al   
 00412397: BE09A74000                mov esi,0040A709   
 0041239C: 03F5                      add esi,ebp      <--ESI=7CB4+40A709=4123BD  
 0041239E: B946160000                mov ecx,00001646   
 004123A3: 03F1                      add esi,ecx   
 004123A5: 4E                        dec esi   
 004123A6: 8A06                      mov al,byte ptr \[esi\]   
 004123A8: 324601                    xor al,byte ptr \[esi+01\]   
 004123AB: 8806                      mov byte ptr \[esi\],al   
 004123AD: 4E                        dec esi   
 004123AE: E2F6                      loop 004123A6      <--代码还原  
 004123B0: FA                        cli   
 004123B1: 6681BD09A740009090        cmp word ptr \[ebp+0040A709\],9090  <--还原成功，则继续  
 004123BA: 75FE                      jnz 004123BA   
 004123BC: 90                        nop   
 004123BD: 90                        nop   
 004123BE: 90                        nop   
  
\[注：这是最后 XOR 还原得到的代码了,内容太多，我懒得加注释了，自己接和我以前的有关幻影的文章看吧\]  
 004123BF: 2B95CBB24000              sub edx,dword ptr \[ebp+0040B2CB\]   
 004123C5: 81EA87530000              sub edx,00005387   
 004123CB: 8995D3B24000              mov dword ptr \[ebp+0040B2D3\],edx   
 004123D1: E8D7120000                call 004136AD      <--加载 DLL  
 004123D6: 83F800                    cmp eax,00000000   
 004123D9: 7520                      jnz 004123FB   
 004123DB: B8C0B94000                mov eax,0040B9C0   
 004123E0: 03C5                      add eax,ebp   
 004123E2: BBC2B94000                mov ebx,0040B9C2   
 004123E7: 03DD                      add ebx,ebp   
 004123E9: 6A30                      push 00000030   
 004123EB: 50                        push eax   
 004123EC: 53                        push ebx   
 004123ED: 6A00                      push 00000000   
 004123EF: FF9500BC4000              call dword ptr \[ebp+0040BC00\]  <--失败，显示 dial.dll 未找到  
 004123F5: FF956CBF4000              call dword ptr \[ebp+0040BF6C\]  <--退出  
  
 004123FB: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   
 00412401: 83C120                    add ecx,00000020   
 00412404: 51                        push ecx   
 00412405: 6A40                      push 00000040   
 00412407: 6800201000                push 00102000   
 0041240C: 51                        push ecx   
 0041240D: 6A00                      push 00000000   
 0041240F: FF95CBBB4000              call dword ptr \[ebp+0040BBCB\]   
 00412415: 59                        pop ecx   
 00412416: 6A40                      push 00000040   
 00412418: 6800101000                push 00101000   
 0041241D: 51                        push ecx   
 0041241E: 50                        push eax   
 0041241F: FF95CBBB4000              call dword ptr \[ebp+0040BBCB\]  <--申请内存  
 00412425: 8BD0                      mov edx,eax   
 00412427: 8BF8                      mov edi,eax   
 00412429: BE4D534000                mov esi,0040534D   
 0041242E: 03F5                      add esi,ebp   
 00412430: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   
 00412436: F3A4                      repe movsb es:\[edi\],ds:\[esi\]  <--搬家  
 00412438: B893A74000                mov eax,0040A793   
 0041243D: 2D4D534000                sub eax,0040534D   
 00412442: 03C2                      add eax,edx   
 00412444: 50                        push eax   
 00412445: C3                        ret              <--到新地址继续  
\-----------------------以上一段分析完毕，加载 DLL，申请内存并复制代码继续执行------------------------  
  
 00412446: BF4D534000                mov edi,0040534D   
 0041244B: 03FD                      add edi,ebp   
 0041244D: 8B8DF2B94000              mov ecx,dword ptr \[ebp+0040B9F2\]   
 00412453: 32C0                      xor al,al   
 00412455: F3AA                      repe stosb es:\[edi\],al    <--灭迹  
   
 00412457: E800000000                call 0041245C   
 0041245C: 5D                        pop ebp   
 0041245D: 81EDA9A74000              sub ebp,0040A7A9  <--EBP=41245C-40A7A9=7CB3  
 00412463: BEDDA74000                mov esi,0040A7DD   
 00412468: 03F5                      add esi,ebp   
 0041246A: 6A00                      push 00000000   
 0041246C: 6A00                      push 00000000   
 0041246E: 6A00                      push 00000000   
 00412470: 6A00                      push 00000000   
 00412472: 6A00                      push 00000000   
 00412474: 6A00                      push 00000000   
 00412476: 56                        push esi   
 00412477: FF95A3BB4000              call dword ptr \[ebp+0040BBA3\] <--检查跟踪  
 0041247D: 83F8FF                    cmp eax,FFFFFFFF   
 00412480: 7568                      jnz 004124EA   
 00412482: 46                        inc esi   
 00412483: 803E00                    cmp byte ptr \[esi\],00   
 00412486: 75FA                      jnz 00412482   
 00412488: 46                        inc esi   
 00412489: 803E00                    cmp byte ptr \[esi\],00   
 0041248C: 75DC                      jnz 0041246A   
 0041248E: EB5A                      jmp 004124EA   
\---------------------以上一段代码分析完毕，检查跟踪--------------------------  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*非代码区\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412490: 5C                        pop esp   
 00412491: 5C                        pop esp   
 00412492: 2E5C                      pop esp   
 00412494: 54                        push esp   
 00412495: 52                        push edx   
 00412496: 57                        push edi   
 00412497: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 0041249B: 5C                        pop esp   
 0041249C: 54                        push esp   
 0041249D: 52                        push edx   
 0041249E: 57                        push edi   
 0041249F: 3230                      xor dh,byte ptr \[eax\]   
 004124A1: 3030                      xor byte ptr \[eax\],dh   
 004124A3: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124A7: 5C                        pop esp   
 004124A8: 54                        push esp   
 004124A9: 52                        push edx   
 004124AA: 57                        push edi   
 004124AB: 44                        inc esp   
 004124AC: 45                        inc ebp   
 004124AD: 42                        inc edx   
 004124AE: 55                        push ebp   
 004124AF: 47                        inc edi   
 004124B0: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124B4: 5C                        pop esp   
 004124B5: 52                        push edx   
 004124B6: 45                        inc ebp   
 004124B7: 47                        inc edi   
 004124B8: 56                        push esi   
 004124B9: 58                        pop eax   
 004124BA: 44                        inc esp   
 004124BB: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124BF: 5C                        pop esp   
 004124C0: 4E                        dec esi   
 004124C1: 54                        push esp   
 004124C2: 49                        dec ecx   
 004124C3: 43                        inc ebx   
 004124C4: 45                        inc ebp   
 004124C5: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124C9: 5C                        pop esp   
 004124CA: 53                        push ebx   
 004124CB: 49                        dec ecx   
 004124CC: 43                        inc ebx   
 004124CD: 45                        inc ebp   
 004124CE: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124D2: 5C                        pop esp   
 004124D3: 56                        push esi   
 004124D4: 4B                        dec ebx   
 004124D5: 45                        inc ebp   
 004124D6: 59                        pop ecx   
 004124D7: 50                        push eax   
 004124D8: 52                        push edx   
 004124D9: 4F                        dec edi   
 004124DA: 44                        inc esp   
 004124DB: 005C5C2E                  add byte ptr \[esp+ebx\*2+2E\],bl   
 004124DF: 5C                        pop esp   
 004124E0: 46                        inc esi   
 004124E1: 49                        dec ecx   
 004124E2: 4C                        dec esp   
 004124E3: 45                        inc ebp   
 004124E4: 56                        push esi   
 004124E5: 58                        pop eax   
 004124E6: 44                        inc esp   
 004124E7: 000000                    add byte ptr \[eax\],al   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*非代码区结束\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
 004124EA: 83F8FF           cmp eax,FFFFFFFF  
 004124ED: 7406             jz 004124F5  
 004124EF: FF956CBF4000              call dword ptr \[ebp+0040BF6C\]  <--有跟踪则退出  
  
 004124F5: 60                        pushad              <--反跟踪  
 004124F6: BE53BD4000                mov esi,0040BD53   
 004124FB: 03F5                      add esi,ebp   
 004124FD: 0F010E                    sidt dword ptr \[esi\]   
 00412500: 8B7602                    mov esi,dword ptr \[esi+02\]   
 00412503: 668B4608                  mov ax,word ptr \[esi+08\]   
 00412507: 66052302                  add ax,0223   
 0041250B: 66894608                  mov word ptr \[esi+08\],ax   
 0041250F: 668B460E                  mov ax,word ptr \[esi+0E\]   
 00412513: 668B5E06                  mov bx,word ptr \[esi+06\]   
 00412517: 663BC3                    cmp ax,bx   
 0041251A: 740E                      jz 0041252A   
 0041251C: 668B4608                  mov ax,word ptr \[esi+08\]   
 00412520: 668B5E18                  mov bx,word ptr \[esi+18\]   
 00412524: FA                        cli   
 00412525: 663BC3                    cmp ax,bx   
 00412528: 7400                      jz 0041252A   
 0041252A: 668B4608                  mov ax,word ptr \[esi+08\]   
 0041252E: 668B5E0E                  mov bx,word ptr \[esi+0E\]   
 00412532: 66898567BE4000            mov word ptr \[ebp+0040BE67\],ax   
 00412539: 66899D69BE4000            mov word ptr \[ebp+0040BE69\],bx   
 00412540: 668B4618                  mov ax,word ptr \[esi+18\]   
 00412544: 668B5E1E                  mov bx,word ptr \[esi+1E\]   
 00412548: 6689856BBE4000            mov word ptr \[ebp+0040BE6B\],ax   
 0041254F: 66899D6DBE4000            mov word ptr \[ebp+0040BE6D\],bx    <--对 IDT 的处理，反跟踪  
  
 00412556: 89AD65BD4000              mov dword ptr \[ebp+0040BD65\],ebp   
 0041255C: 8B85C7BB4000              mov eax,dword ptr \[ebp+0040BBC7\]   
 00412562: 89856FBE4000              mov dword ptr \[ebp+0040BE6F\],eax   
  
 00412568: E8A9140000                call 00413A16          <--内有反跟踪的代码，进入中断处理程序  
 0041256D: BB63BD4000                mov ebx,0040BD63   
 00412572: 03DD                      add ebx,ebp   
 00412574: 53                        push ebx   
 00412575: 6A64                      push 00000064   
 00412577: 6A63                      push 00000063   
 00412579: 6A00                      push 00000000   
 0041257B: FF9508BC4000              call dword ptr \[ebp+0040BC08\]    <--SetTimer  
 00412581: 61                        popad    
 00412582: 6A00                      push 00000000   
 00412584: FF95C3BB4000              call dword ptr \[ebp+0040BBC3\]    <--GetModuleHandle  
 0041258A: 89855FBD4000              mov dword ptr \[ebp+0040BD5F\],eax  <--413A12  
 00412590: 80BDC8B2400001            cmp byte ptr \[ebp+0040B2C8\],01    <--412F7B  
  
\[ebp+0040B2C8\]是一个标志，可能是用来标识是否被注册版幻影加密的  
  
 00412597: 7435                      jz 004125CE   
 00412599: BEE1B94000                mov esi,0040B9E1   
 0041259E: 03F5                      add esi,ebp            <--413694  
 004125A0: 56                        push esi   
 004125A1: FF959FBB4000              call dword ptr \[ebp+0040BB9F\]    <--GetSystemTime  
 004125A7: 668B85EFB94000            mov ax,word ptr \[ebp+0040B9EF\]    <--取毫秒时间  
 004125AE: 240F                      and al,0F   
 004125B0: 3C02                      cmp al,02   
 004125B2: 731A                      jnb 004125CE            <--结果随机，用来随机显示未注册版加密。  
   
 004125B4: B8BEB94000                mov eax,0040B9BE   
 004125B9: 03C5                      add eax,ebp            <--  
 004125BB: BB11B24000                mov ebx,0040B211   
 004125C0: 03DD                      add ebx,ebp            <--412EC4   
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
       本软件由 幻影 v1.5 未注册版本加密             
  
 This program protected by DBPE v1.5  Unregistered version  
  
  
  
 Homepage: http://fsdb.yeah.net    E-mail: D.Boy@126.com   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 004125C2: 6A30                      push 00000030   
 004125C4: 50                        push eax   
 004125C5: 53                        push ebx   
 004125C6: 6A00                      push 00000000   
 004125C8: FF9500BC4000              call dword ptr \[ebp+0040BC00\]    <--MessageBoxA  
\---------------------以上一段代码分析完毕，处理反跟踪和是否被注册版幻影加密，否则将随机显示 NAG 窗-------------  
  
 004125CE: E862060000                call 00412C35    <--查询注册键，是否有注册信息  
 004125D3: 663D0100                  cmp ax,0001    <--是否找到  
 004125D7: 7416                      jz 004125EF    <--找到，跳转，没有则表示第一次使用，在注册表中留下足迹  
 004125D9: BE60B14000                mov esi,0040B160   
 004125DE: 03F5                      add esi,ebp    <--ESI 指向还原后的注册信息首地址  
 004125E0: 668B8589B44000            mov ax,word ptr \[ebp+0040B489\] <--41313C 0x60  
 004125E7: 668906                    mov word ptr \[esi\],ax      <--保存  
 004125EA: E802070000                call 00412CF1          <--创建注册键  
  
 004125EF: E87C030000                call 00412970          <--获取卷信息并且跟据注册表信息来检测软件是否已经注册  
 004125F4: 6689857DB44000            mov word ptr \[ebp+0040B47D\],ax <--40B47D 卷信息标志  
  
\[ebp+0040B47D\]从此开始保存了软件是否已经注册的信息。（此注册是指软件注册，而非标识幻影注册版加密）  
  
 004125FB: 663D0100                  cmp ax,0001   
 004125FF: 0F84BF000000              jz 004126C4          <--已注册，就跳转到还原 Section 处  
\--------------------以上一段代码分析完毕，判断是否软件已经注册------------  
  
 00412605: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000 <--413410  
  
\[ebp+0040B75D\]是一个标志，代码中有多处判断。但没有对其赋值的代码，只从 EXE 中得到。作用目前还不太清楚  
  
 0041260C: 7426                      jz 00412634      <--为 0，则跳走。到显示 StartMessage 处。  
 0041260E: 83BD61B7400001            cmp dword ptr \[ebp+0040B761\],00000001 <--413414  
  
\[ebp+0040B761\]是一个标志，作用不清楚。目前没有发现赋值代码。可能是用来表示是否有 Register Cub 吧  
  
 00412615: 751D                      jnz 00412634          <--不为 1，就跳转到显示 StartMessage 处。  
  
 00412617: E80A060000                call 00412C26          <--显示软件加的 NAG 信息,Register Cub  
  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
                  欢迎你使用幻影 1.5 beta2，  
  
   幻影 v1.5 是共享软件，如果你喜欢它，希望你注册。   
  
                    
  
                   E-mail   : ding-boy@netease.com   
                   Homepage : http://fsdb.yeah.net               
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
 0041261C: 6689857FB44000            mov word ptr \[ebp+0040B47F\],ax      <-413132,显示过 NAG 了  
  
\[ebp+0040B47F\]看来从此作为一个标志了。作用不太清楚。ax 返回的是按的是 Try 还是 Register 按钮。  
  
 00412623: BBC9B24000                mov ebx,0040B2C9   
 00412628: 03DD                      add ebx,ebp          <--412F7C,要显示的内容，如果为"||"则表示不显示。  
 0041262A: 663D0200                  cmp ax,0002          <--2，好象表示选择了 Register 按钮，则跳转到进入注册处  
 0041262E: 0F8444010000              jz 00412778   
  
 00412634: 6681BDF1B440007C7C        cmp word ptr \[ebp+0040B4F1\],7C7C <--4131A4，判断是否有 StartMessage 显示  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
     欢迎  ^\_^  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 0041263D: 741A                      jz 00412659        <--没有则跳走，进入密码输入判断处  
 0041263F: B8BEB94000                mov eax,0040B9BE   
 00412644: 03C5                      add eax,ebp   
 00412646: BBF1B44000                mov ebx,0040B4F1   
 0041264B: 03DD                      add ebx,ebp      <--4131A4  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
     欢迎  ^\_^  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 0041264D: 6A30                      push 00000030   
 0041264F: 50                        push eax   
 00412650: 53                        push ebx   
 00412651: 6A00                      push 00000000   
 00412653: FF9500BC4000              call dword ptr \[ebp+0040BC00\] <---显示 NAG 窗口,Start Message  
\------------------以上一段代码分析未完，有几处标志作用不太清楚。-------------------------------  
  
 00412659: E8A0020000                call 004128FE      <---输入密码并检测,Password 很好得到，还原后的代码中有。  
 0041265E: BB91B84000                mov ebx,0040B891    
 00412663: 03DD                      add ebx,ebp      <---413544  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 Invalid  PassWord  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412665: 663D0000                  cmp ax,0000   
 00412669: 0F8409010000              jz 00412778      <--密码错，跳走到注册处。  
 0041266F: E882010000                call 004127F6      <--检查是否过期  
 00412674: BB1DB64000                mov ebx,0040B61D   
 00412679: 03DD                      add ebx,ebp      <--4132D0  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
       试用期已过。   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 0041267B: 663D0000                  cmp ax,0000   
 0041267F: 0F84F3000000              jz 00412778      <--过期，则跳走到注册处  
 00412685: E8C3010000                call 0041284D      <--在注册表中查找注册信息，没有则创建注册信息。  
 0041268A: BB1DB64000                mov ebx,0040B61D    <--4132D0  
 0041268F: 03DD                      add ebx,ebp   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
       试用期已过。   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412691: 663D0000                  cmp ax,0000   
 00412695: 0F84DD000000              jz 00412778      <--未创建成功，则跳转到注册处  
  
 0041269B: 6683BD7DB4400000          cmp word ptr \[ebp+0040B47D\],0000    <--40B47D 标志,是否已经成为正版用户  
 004126A3: 751F                      jnz 004126C4        <--标志有效则跳转到还原 Section 处  
  
 004126A5: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000  <--413410  
 004126AC: 7416                      jz 004126C4   
 004126AE: 6683BD7FB4400001          cmp word ptr \[ebp+0040B47F\],0001    <-413132,1 表示按的是 Try 按钮  
 004126B6: 740C                      jz 004126C4              <-是，则到还原 Section 处  
 004126B8: BBC9B24000                mov ebx,0040B2C9   
 004126BD: 03DD                      add ebx,ebp              <--412F7C,显示信息如果为"||"则不显示  
 004126BF: E9B4000000                jmp 00412778      <--到注册处  
 -----------------------------以上代码分析未完，还是几个标志作用不太清楚------------------  
  
 ---------------------还原各 Section--------------  
 004126C4: BED7B24000                mov esi,0040B2D7   
 004126C9: 03F5                      add esi,ebp              <--412F8A  
   
 004126CB: 833E00                    cmp dword ptr \[esi\],00000000   
 004126CE: 746F                      jz 0041273F   
 004126D0: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]    <--412F86  
 004126D6: 031E                      add ebx,dword ptr \[esi\]   
 004126D8: 8B4E04                    mov ecx,dword ptr \[esi+04\]   
 004126DB: 83F900                    cmp ecx,00000000   
 004126DE: 7505                      jnz 004126E5   
 004126E0: 83C608                    add esi,00000008   
 004126E3: EBE6                      jmp 004126CB  
   
 004126E5: D1E9                      shr ecx,1   
 004126E7: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]      <--41311A, 82EE  
 004126EE: 66F7D0                    not ax   
   
 004126F1: 663103                    xor word ptr \[ebx\],ax   
 004126F4: 6648                      dec ax   
 004126F6: 66813B3F3F                cmp word ptr \[ebx\],3F3F   
 004126FB: 7518                      jnz 00412715   
 004126FD: 817BFC44425045            cmp dword ptr \[ebx-04\],45504244   
 00412704: 750F                      jnz 00412715   
 00412706: 6683BD7DB4400001          cmp word ptr \[ebp+0040B47D\],0001 <--40B47D 卷信息标志  
 0041270E: 7505                      jnz 00412715   
 00412710: 66C7032B2B                mov word ptr \[ebx\],2B2B   
   
 00412715: 66817BFF3F3F              cmp word ptr \[ebx-01\],3F3F   
 0041271B: 7519                      jnz 00412736   
 0041271D: 817BFB44425045            cmp dword ptr \[ebx-05\],45504244   
 00412724: 7510                      jnz 00412736   
 00412726: 6683BD7DB4400001          cmp word ptr \[ebp+0040B47D\],0001 <--40B47D 卷信息标志  
 0041272E: 7506                      jnz 00412736   
 00412730: 66C743FF2B2B              mov word ptr \[ebx-01\],2B2B   
   
 00412736: 43                        inc ebx   
 00412737: 43                        inc ebx   
 00412738: E2B7                      loop 004126F1   
   
 0041273A: 83C608                    add esi,00000008   
 0041273D: EB8C                      jmp 004126CB   
\-------------以上一段为还原各 Section，-------------------------------  
  
 0041273F: 90                        nop   
 00412740: 89AD9FBE4000              mov dword ptr \[ebp+0040BE9F\],ebp   
 00412746: E84C030000                call 00412A97            <--作用不清  
 0041274B: E82D040000                call 00412B7D            <--作用不清  
 00412750: 8B85CFB24000              mov eax,dword ptr \[ebp+0040B2CF\]  <--412F82  
 00412756: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]  <--412F86  
 0041275C: 03C3                      add eax,ebx   
 0041275E: 8985F6B94000              mov dword ptr \[ebp+0040B9F6\],eax   
 00412764: 8BC5                      mov eax,ebp   
 00412766: 5B                        pop ebx   
 00412767: 59                        pop ecx   
 00412768: 5A                        pop edx   
 00412769: 5E                        pop esi   
 0041276A: 5F                        pop edi   
 0041276B: 5D                        pop ebp   
 0041276C: 9D                        popfd    
 0041276D: FFB0F6B94000              push dword ptr \[eax+0040B9F6\]   
 00412773: E90F120000                jmp 00413987        <--跳转到最后的代码处  
  
\-------未注册时跳到此处----------------  
 00412778: 66813B7C7C                cmp word ptr \[ebx\],7C7C   
 0041277D: 7413                      jz 00412792        <--是否有信息显示，没有则跳走  
  
 0041277F: B8C0B94000                mov eax,0040B9C0   
 00412784: 03C5                      add eax,ebp          <--413673 一个空格  
 00412786: 6A30                      push 00000030   
 00412788: 50                        push eax   
 00412789: 53                        push ebx   
 0041278A: 6A00                      push 00000000   
 0041278C: FF9500BC4000              call dword ptr \[ebp+0040BC00\]  <--MessageBoxA  
  
 00412792: 83BD5DB7400000            cmp dword ptr \[ebp+0040B75D\],00000000 <--413410  
 00412799: 7446                      jz 004127E1   
 0041279B: E810010000                call 004128B0        <--输入 Serial,Your Name 并进行判断结果  
 004127A0: 6689857DB44000            mov word ptr \[ebp+0040B47D\],ax <--40B47D 卷信息标志  
 004127A7: 50                        push eax   
 004127A8: 663D0100                  cmp ax,0001   
 004127AC: 7507                      jnz 004127B5      <--不等，注册失败，跳转  
 004127AE: BBF5B84000                mov ebx,0040B8F5        <--4135A8,注册成功了  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 Register Successfully  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 004127B3: EB05                      jmp 004127BA   
  
 004127B5: BB59B94000                mov ebx,0040B959   
 004127BA: 03DD                      add ebx,ebp          <--41360C  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 Invalid Serial NO  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 004127BC: 66813B7C7C                cmp word ptr \[ebx\],7C7C    <--是否为没有内容显示的标志  
 004127C1: 7413                      jz 004127D6          <--是，则跳转  
 004127C3: B8C0B94000                mov eax,0040B9C0   
 004127C8: 03C5                      add eax,ebp          <--413673 一个空格  
 004127CA: 6A30                      push 00000030   
 004127CC: 50                        push eax   
 004127CD: 53                        push ebx   
 004127CE: 6A00                      push 00000000   
 004127D0: FF9500BC4000              call dword ptr \[ebp+0040BC00\]  <--MessageBoxA  
  
 004127D6: 58                        pop eax   
 004127D7: 663D0100                  cmp ax,0001   
 004127DB: 0F84E3FEFFFF              jz 004126C4      <--注册成功，就回去继续工作  
  
 004127E1: C6855DBD400000            mov byte ptr \[ebp+0040BD5D\],00   
 004127E8: 8BC5                      mov eax,ebp   
 004127EA: 5B                        pop ebx   
 004127EB: 59                        pop ecx   
 004127EC: 5A                        pop edx   
 004127ED: 5E                        pop esi   
 004127EE: 5F                        pop edi   
 004127EF: 5D                        pop ebp   
 004127F0: 9D                        popfd    
 004127F1: E991110000                jmp 00413987      <--从这里跳走，怕是凶多吉少  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*主程序结束\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
 -----------------------------------检查是否过期------------------------  
 004127F6: BEE1B94000                mov esi,0040B9E1   
 004127FB: 03F5                      add esi,ebp          <--413694  
 004127FD: 56                        push esi   
 004127FE: FF959FBB4000              call dword ptr \[ebp+0040BB9F\]  <--GetSystemTime  
 00412804: 668B8585B44000            mov ax,word ptr \[ebp+0040B485\] <--413138  
 0041280B: 668B9D81B44000            mov bx,word ptr \[ebp+0040B481\] <--413134  
 00412812: 663D0000                  cmp ax,0000   
 00412816: 7430                      jz 00412848   
 00412818: 663985E3B94000            cmp word ptr \[ebp+0040B9E3\],ax   
 0041281F: 7209                      jb 0041282A   
 00412821: 66399DE1B94000            cmp word ptr \[ebp+0040B9E1\],bx   
 00412828: 7309                      jnb 00412833   
 0041282A: 66399DE1B94000            cmp word ptr \[ebp+0040B9E1\],bx   
 00412831: 7606                      jbe 00412839   
 00412833: 66B80000                  mov ax,0000   
 00412837: EB0D                      jmp 00412846   
 00412839: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001 <--40B47D 卷信息标志  
 00412842: 66B80100                  mov ax,0001   
 00412846: EB04                      jmp 0041284C   
 00412848: 66B80100                  mov ax,0001   
 0041284C: C3                        ret   
\--------------------------------------------------------------  
  
\------------------------对注册表查找创建注册信息----------------------------------------------  
 0041284D: 60                        pushad    
 0041284E: 6683BD89B4400000          cmp word ptr \[ebp+0040B489\],0000  <--41313C 0x60  
 00412856: 750B                      jnz 00412863   
 00412858: 66C785FBAB40000100        mov word ptr \[ebp+0040ABFB\],0001  <--4128AE 过期标志  
 00412861: EB42                      jmp 004128A5   
  
 00412863: E8CD030000                call 00412C35            <--找注册键  
 00412868: BE60B14000                mov esi,0040B160          <--ESI 指向还原后的注册信息首地址  
 0041286D: 03F5                      add esi,ebp   
 0041286F: 663D0100                  cmp ax,0001   
 00412873: 7405                      jz 0041287A            <--找到则跳转  
  
 00412875: 66C7060000                mov word ptr \[esi\],0000   
 0041287A: 66833E00                  cmp word ptr \[esi\],0000   
 0041287E: 7417                      jz 00412897            <--没找到，或者错误则跳转  
  
 00412880: 66FF0E                    dec word ptr \[esi\]   
 00412883: 66C785FBAB40000100        mov word ptr \[ebp+0040ABFB\],0001  <--4128AE 过期标志  
 0041288C: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001  <--40B47D 卷信息标志  
 00412895: EB09                      jmp 004128A0   
  
 00412897: 66C785FBAB40000000        mov word ptr \[ebp+0040ABFB\],0000  <--4128AE 过期标志  
 004128A0: E84C040000                call 00412CF1            <--创建注册信息  
 004128A5: 61                        popad    
 004128A6: 668B85FBAB4000            mov ax,word ptr \[ebp+0040ABFB\]    <--4128AE 过期标志  
 004128AD: C3                        ret   
\-----------------------------------------------------------------  
  
\----------------------------非代码区------------------------------  
 004128AE: 0000                      add byte ptr \[eax\],al   
\-------------------------------------------------------------  
  
 ---------------------------输入 Serial 并进行判断------------------------------------  
 004128B0: E8BB000000                call 00412970          <--根据卷信息来检测是否原版用户  
 004128B5: 8B85A8AD4000              mov eax,dword ptr \[ebp+0040ADA8\] lpTotalNumberOfBytes,  
 004128BB: BBB0AD4000                mov ebx,0040ADB0   
 004128C0: 03DD                      add ebx,ebp      <--412A63 "%lX"  
 004128C2: B983B14000                mov ecx,0040B183   
 004128C7: 03CD                      add ecx,ebp   
 004128C9: 50                        push eax   
 004128CA: 53                        push ebx   
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 %lX  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 004128CB: 51                        push ecx   
 004128CC: FF9504BC4000              call dword ptr \[ebp+0040BC04\]  <--wsprintfA  
 004128D2: 83C40C                    add esp,0000000C   
 004128D5: B865B14000                mov eax,0040B165      <--存放 Your Name  
 004128DA: 03C5                      add eax,ebp        
 004128DC: BB79B14000                mov ebx,0040B179      <--存放 Serial No  
 004128E1: 03DD                      add ebx,ebp   
 004128E3: B983B14000                mov ecx,0040B183      <--转换后的序列号  
 004128E8: 03CD                      add ecx,ebp   
 004128EA: 51                        push ecx   
 004128EB: 53                        push ebx   
 004128EC: 50                        push eax   
 004128ED: FF95BCBC4000              call dword ptr \[ebp+0040BCBC\]  <--GetRegister  
 004128F3: E8F9030000                call 00412CF1          <--保存注册信息到注册表  
 004128F8: E873000000                call 00412970          <--根据卷信息来检测是否原版用户  
 004128FD: C3                        ret   
\-------------------------------------------------------------------------  
  
 -----------------------------------------检测密码 Password--------------------------  
 004128FE: 6683BD49B740007C          cmp word ptr \[ebp+0040B749\],007C   
 00412906: 7505                      jnz 0041290D   
 00412908: 66B80100                  mov ax,0001   
 0041290C: C3                        ret   
  
 0041290D: B89FAC4000                mov eax,0040AC9F   
 00412912: 03C5                      add eax,ebp          <--412952  
 00412914: 50                        push eax   
 00412915: FF95B8BC4000              call dword ptr \[ebp+0040BCB8\]  <--GetPassword  
 0041291B: BE49B74000                mov esi,0040B749   
 00412920: 03F5                      add esi,ebp          <--4133FC，保存的密码  
 00412922: BF9FAC4000                mov edi,0040AC9F        <--412952  
 00412927: 03FD                      add edi,ebp   
 00412929: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]   
  
 00412930: 8A26                      mov ah,byte ptr \[esi\]   
 00412932: 3A27                      cmp ah,byte ptr \[edi\]   
 00412934: 7509                      jnz 0041293F   
  
 00412936: 80FC00                    cmp ah,00   
 00412939: 7409                      jz 00412944   
 0041293B: 46                        inc esi   
 0041293C: 47                        inc edi   
 0041293D: EBF1                      jmp 00412930   
  
 0041293F: 66B80000                  mov ax,0000   
 00412943: C3                        ret   
 00412944: 66C7857DB440000100        mov word ptr \[ebp+0040B47D\],0001 <--40B47D 卷信息标志  
 0041294D: 66B80100                  mov ax,0001              
 00412951: C3                        ret   
\----------------------------------------------------------  
  
\----------------------------非代码区-----------------------------------  
 00412952: 0000                      add byte ptr \[eax\],al   
 00412954: 0000                      add byte ptr \[eax\],al   
 00412956: 0000                      add byte ptr \[eax\],al   
 00412958: 0000                      add byte ptr \[eax\],al   
 0041295A: 0000                      add byte ptr \[eax\],al   
 0041295C: 0000                      add byte ptr \[eax\],al   
 0041295E: 0000                      add byte ptr \[eax\],al   
 00412960: 0000                      add byte ptr \[eax\],al   
 00412962: 0000                      add byte ptr \[eax\],al   
 00412964: 0000                      add byte ptr \[eax\],al   
 00412966: 0000                      add byte ptr \[eax\],al   
 00412968: 0000                      add byte ptr \[eax\],al   
 0041296A: 0000                      add byte ptr \[eax\],al   
 0041296C: 0000                      add byte ptr \[eax\],al   
 0041296E: 0000                      add byte ptr \[eax\],al   
\---------------------------------------------------------------  
  
  
 --------------------------根据卷信息保存到注册表----------------------  
 00412970: B810000000                mov eax,00000010   
 00412975: 50                        push eax            <--nFileSystemNameSize  
 00412976: B8B8AD4000                mov eax,0040ADB8   
 0041297B: 03C5                      add eax,ebp   
 0041297D: 50                        push eax            <--lpFileSystemNameBuffer,  
 0041297E: B8C8AD4000                mov eax,0040ADC8   
 00412983: 03C5                      add eax,ebp   
 00412985: 50                        push eax            <--lpFileSystemFlags,  
 00412986: B8CCAD4000                mov eax,0040ADCC   
 0041298B: 03C5                      add eax,ebp   
 0041298D: 50                        push eax            <--lpMaximumComponentLength,  
 0041298E: B8D0AD4000                mov eax,0040ADD0   
 00412993: 03C5                      add eax,ebp   
 00412995: 50                        push eax            <--lpVolumeSerialNumber,  
 00412996: B810000000                mov eax,00000010   
 0041299B: 50                        push eax            <--nVolumeNameSize,  
 0041299C: B8D4AD4000                mov eax,0040ADD4   
 004129A1: 03C5                      add eax,ebp   
 004129A3: 50                        push eax            <--lpVolumeNameBuffer,  
 004129A4: B8B4AD4000                mov eax,0040ADB4   
 004129A9: 03C5                      add eax,ebp          <--412A67  
 004129AB: 50                        push eax            <--lpRootPathName, "C:\\"  
 004129AC: FF95BFBB4000              call dword ptr \[ebp+0040BBBF\]  <--GetVolumeInformationA  
 004129B2: B8A4AD4000                mov eax,0040ADA4   
 004129B7: 03C5                      add eax,ebp   
 004129B9: BBA8AD4000                mov ebx,0040ADA8   
 004129BE: 03DD                      add ebx,ebp          lpTotalNumberOfBytes,  
 004129C0: B9ACAD4000                mov ecx,0040ADAC   
 004129C5: 03CD                      add ecx,ebp   
 004129C7: C701633A5C00              mov dword ptr \[ecx\],005C3A63   
 004129CD: 50                        push eax            lpTotalNumberOfFreeBytes  
 004129CE: 53                        push ebx            lpTotalNumberOfBytes,  
 004129CF: 6A00                      push 00000000          lpFreeBytesAvailableToCaller,  
 004129D1: 51                        push ecx            lpDirectoryName,"C:\\"  
 004129D2: FF95B7BB4000              call dword ptr \[ebp+0040BBB7\] <--GetDiskFreeSpaceExA  
 004129D8: 8B85A8AD4000              mov eax,dword ptr \[ebp+0040ADA8\] lpTotalNumberOfBytes,  
 004129DE: F7D0                      not eax   
 004129E0: 3385D0AD4000              xor eax,dword ptr \[ebp+0040ADD0\] lpVolumeSerialNumber,  
 004129E6: 8985A8AD4000              mov dword ptr \[ebp+0040ADA8\],eax lpTotalNumberOfBytes,  
 004129EC: E844020000                call 00412C35          <--查找注册表，提取数据  
 004129F1: B801000000                mov eax,00000001   
 004129F6: 33DB                      xor ebx,ebx   
 004129F8: BE65B14000                mov esi,0040B165   
 004129FD: 03F5                      add esi,ebp          <--412E18,存放 Your Name  
  
 004129FF: 8A1E                      mov bl,byte ptr \[esi\]   
 00412A01: 80FB00                    cmp bl,00   
 00412A04: 7405                      jz 00412A0B   
 00412A06: F7E3                      mul ebx   
 00412A08: 46                        inc esi   
 00412A09: EBF4                      jmp 004129FF   
  
 00412A0B: 33855DB74000              xor eax,dword ptr \[ebp+0040DB75\] <--413410,Register Key  
 00412A11: 3385A8AD4000              xor eax,dword ptr \[ebp+0040ADA8\] lpTotalNumberOfBytes,  
 00412A17: 86E0                      xchg al,ah   
 00412A19: 8BD8                      mov ebx,eax   
 00412A1B: BE79B14000                mov esi,0040B179        <--412E2C,存放 Serial No  
 00412A20: 03F5                      add esi,ebp   
 00412A22: E80E000000                call 00412A35        <--数据转换  
 00412A27: 3BD8                      cmp ebx,eax        <--比较是否正版  
 00412A29: 7405                      jz 00412A30   
 00412A2B: 66B80000                  mov ax,0000   
 00412A2F: C3                        ret   
 00412A30: 66B80100                  mov ax,0001   
 00412A34: C3                        ret   
\--------------------------------------------------------------------------  
  
\-------------------------字符转换成数值---------------------------------  
 00412A35: 53                        push ebx   
 00412A36: 56                        push esi   
 00412A37: 33DB                      xor ebx,ebx   
 00412A39: 8A06                      mov al,byte ptr \[esi\]   
 00412A3B: 3C39                      cmp al,39   
 00412A3D: 7606                      jbe 00412A45   
 00412A3F: 24CF                      and al,CF   
 00412A41: 2C37                      sub al,37   
 00412A43: EB02                      jmp 00412A47   
 00412A45: 240F                      and al,0F   
 00412A47: C1E304                    shl ebx,04   
 00412A4A: 0AD8                      or bl,al   
 00412A4C: 46                        inc esi   
 00412A4D: 803E00                    cmp byte ptr \[esi\],00   
 00412A50: 75E7                      jnz 00412A39   
 00412A52: 8BC3                      mov eax,ebx   
 00412A54: 5E                        pop esi   
 00412A55: 5B                        pop ebx   
 00412A56: C3                        ret   
\----------------------------------------------------------------------  
  
 ---------------------------非代码区---------------------------------  
 00412A57: 0000                      add byte ptr \[eax\],al   
 00412A59: 0000                      add byte ptr \[eax\],al   
 00412A5B: 0000                      add byte ptr \[eax\],al   
 00412A5D: 0000                      add byte ptr \[eax\],al   
 00412A5F: 633A                      arpl word ptr \[edx\],di   
 00412A61: 5C                        pop esp   
 00412A62: 00256C580043              add byte ptr \[4300586C\],ah   
 00412A68: 3A5C0000                  cmp bl,byte ptr \[eax+eax\]   
 00412A6C: 0000                      add byte ptr \[eax\],al   
 00412A6E: 0000                      add byte ptr \[eax\],al   
 00412A70: 0000                      add byte ptr \[eax\],al   
 00412A72: 0000                      add byte ptr \[eax\],al   
 00412A74: 0000                      add byte ptr \[eax\],al   
 00412A76: 0000                      add byte ptr \[eax\],al   
 00412A78: 0000                      add byte ptr \[eax\],al   
 00412A7A: 0000                      add byte ptr \[eax\],al   
 00412A7C: 0000                      add byte ptr \[eax\],al   
 00412A7E: 0000                      add byte ptr \[eax\],al   
 00412A80: 0000                      add byte ptr \[eax\],al   
 00412A82: 0000                      add byte ptr \[eax\],al   
 00412A84: 0000                      add byte ptr \[eax\],al   
 00412A86: 0000                      add byte ptr \[eax\],al   
 00412A88: 0000                      add byte ptr \[eax\],al   
 00412A8A: 0000                      add byte ptr \[eax\],al   
 00412A8C: 0000                      add byte ptr \[eax\],al   
 00412A8E: 0000                      add byte ptr \[eax\],al   
 00412A90: 0000                      add byte ptr \[eax\],al   
 00412A92: 0000                      add byte ptr \[eax\],al   
 00412A94: 0000                      add byte ptr \[eax\],al   
 00412A96: 00                     
\------------------------------------------------------------------  
  
\--------------------------------------------------------  
 00412A97: 60                        pushad    
 00412A98: 8B9D69B44000              mov ebx,dword ptr \[ebp+0040B469\]  <--41311C  
 00412A9E: 83FB00                    cmp ebx,00000000   
 00412AA1: 0F84A6000000              jz 00412B4D   
 00412AA7: 039DD3B24000              add ebx,dword ptr \[ebp+0040B2D3\]   
 00412AAD: 8B430C                    mov eax,dword ptr \[ebx+0C\]   
 00412AB0: 83F800                    cmp eax,00000000   
 00412AB3: 0F8494000000              jz 00412B4D   
 00412AB9: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00412ABF: 8BF0                      mov esi,eax   
 00412AC1: 50                        push eax   
 00412AC2: FF9564BF4000              call dword ptr \[ebp+0040BF64\]   
 00412AC8: 89857DBA4000              mov dword ptr \[ebp+0040BA7D\],eax   
 00412ACE: 8B33                      mov esi,dword ptr \[ebx\]   
 00412AD0: 8B7B10                    mov edi,dword ptr \[ebx+10\]   
 00412AD3: 83FE00                    cmp esi,00000000   
 00412AD6: 7502                      jnz 00412ADA   
 00412AD8: 8BF7                      mov esi,edi   
 00412ADA: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
 00412AE0: 03BDD3B24000              add edi,dword ptr \[ebp+0040B2D3\]   
  
 00412AE6: 8B06                      mov eax,dword ptr \[esi\]   
 00412AE8: 83F800                    cmp eax,00000000   
 00412AEB: 7505                      jnz 00412AF2   
 00412AED: 83C314                    add ebx,00000014   
 00412AF0: EBBB                      jmp 00412AAD   
 00412AF2: 807E0380                  cmp byte ptr \[esi+03\],80   
 00412AF6: 7514                      jnz 00412B0C   
 00412AF8: 33C0                      xor eax,eax   
 00412AFA: 668706                    xchg word ptr \[esi\],ax   
 00412AFD: 50                        push eax   
 00412AFE: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00412B04: FF9568BF4000              call dword ptr \[ebp+0040BF68\]   
 00412B0A: EB37                      jmp 00412B43   
 00412B0C: 33C0                      xor eax,eax   
 00412B0E: 8706                      xchg dword ptr \[esi\],eax   
 00412B10: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00412B16: 83C002                    add eax,00000002   
 00412B19: 8BC8                      mov ecx,eax   
 00412B1B: E83B000000                call 00412B5B            <---判断是否为 ExitProcess 字符串  
 00412B20: 663D0100                  cmp ax,0001   
 00412B24: 7510                      jnz 00412B36            <--不同，则跳转  
 00412B26: C6855DBD400001            mov byte ptr \[ebp+0040BD5D\],01   
 00412B2D: B89EBE4000                mov eax,0040BE9E   
 00412B32: 03C5                      add eax,ebp   
 00412B34: EB0D                      jmp 00412B43   
  
 00412B36: 50                        push eax   
 00412B37: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00412B3D: FF9568BF4000              call dword ptr \[ebp+0040BF68\]   
  
 00412B43: 8907                      mov dword ptr \[edi\],eax   
 00412B45: 83C604                    add esi,00000004   
 00412B48: 83C704                    add edi,00000004   
 00412B4B: EB99                      jmp 00412AE6   
  
 00412B4D: 61                        popad    
 00412B4E: C3                        ret   
\-----------------------------------------------------  
  
\--------------------非代码区------------  
 00412B4F: 45                        inc ebp   
 00412B50: 7869                      js 00412BBB   
 00412B52: 7450                      jz 00412BA4   
 00412B54: 726F                      jb 00412BC5   
 00412B56: 636573                    arpl word ptr \[ebp+73\],sp   
 00412B59: 7300                      jnb 00412B5B  
 -----------------------------------  
  
\----------------判断是否为 ExitProcess 字符串--------------------  
 00412B5B: 53                        push ebx   
 00412B5C: 56                        push esi   
 00412B5D: 57                        push edi   
 00412B5E: 8BF0                      mov esi,eax   
 00412B60: BF9CAE4000                mov edi,0040AE9C   
 00412B65: 03FD                      add edi,ebp        <---412B4F  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 ExitProcess  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412B67: 8A1E                      mov bl,byte ptr \[esi\]   
 00412B69: 3A1F                      cmp bl,byte ptr \[edi\]   
 00412B6B: 750C                      jnz 00412B79   
 00412B6D: 46                        inc esi   
 00412B6E: 47                        inc edi   
 00412B6F: 80FB00                    cmp bl,00   
 00412B72: 75F3                      jnz 00412B67   
 00412B74: B801000000                mov eax,00000001   
 00412B79: 5F                        pop edi   
 00412B7A: 5E                        pop esi   
 00412B7B: 5B                        pop ebx   
 00412B7C: C3                        ret   
 ---------------------------------------------------  
  
\--------------------------------------------------  
 00412B7D: 60                        pushad    
 00412B7E: 33C0                      xor eax,eax   
 00412B80: 8BB56DB44000              mov esi,dword ptr \[ebp+0040B46D\]   
 00412B86: 83FE00                    cmp esi,00000000   
 00412B89: 7459                      jz 00412BE4   
 00412B8B: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
  
 00412B91: 668B3E                    mov di,word ptr \[esi\]   
 00412B94: 663B7E02                  cmp di,word ptr \[esi+02\]   
 00412B98: 744A                      jz 00412BE4   
 00412B9A: 8B3E                      mov edi,dword ptr \[esi\]   
 00412B9C: 83FF00                    cmp edi,00000000   
 00412B9F: 7443                      jz 00412BE4   
 00412BA1: 03BDD3B24000              add edi,dword ptr \[ebp+0040B2D3\]   
 00412BA7: 8B4E04                    mov ecx,dword ptr \[esi+04\]   
 00412BAA: 83E908                    sub ecx,00000008   
 00412BAD: 83F900                    cmp ecx,00000000   
 00412BB0: 7432                      jz 00412BE4   
 00412BB2: D1E9                      shr ecx,1   
 00412BB4: 83C608                    add esi,00000008   
  
 00412BB7: 668B06                    mov ax,word ptr \[esi\]   
 00412BBA: 662500F0                  and ax,F000   
 00412BBE: 663D0030                  cmp ax,3000   
 00412BC2: 7519                      jnz 00412BDD   
 00412BC4: 668B06                    mov ax,word ptr \[esi\]   
 00412BC7: 6625FF0F                  and ax,0FFF   
 00412BCB: 8B9D79B44000              mov ebx,dword ptr \[ebp+0040B479\]   
 00412BD1: 291C07                    sub dword ptr \[edi+eax\],ebx   
 00412BD4: 8B9DD3B24000              mov ebx,dword ptr \[ebp+0040B2D3\]   
 00412BDA: 011C07                    add dword ptr \[edi+eax\],ebx   
 00412BDD: 83C602                    add esi,00000002   
 00412BE0: E2D5                      loop 00412BB7   
 00412BE2: EBAD                      jmp 00412B91   
  
 00412BE4: 61                        popad    
 00412BE5: C3                        ret   
\----------------------------------------------------  
  
  
 00412BE6: 60                        pushad    
 00412BE7: 8BB571B44000              mov esi,dword ptr \[ebp+0040B471\]   
 00412BED: 83FE00                    cmp esi,00000000   
 00412BF0: 7432                      jz 00412C24   
 00412BF2: 03B5D3B24000              add esi,dword ptr \[ebp+0040B2D3\]   
 00412BF8: 8B8D75B44000              mov ecx,dword ptr \[ebp+0040B475\]   
 00412BFE: 83F900                    cmp ecx,00000000   
 00412C01: 7421                      jz 00412C24   
 00412C03: 8B06                      mov eax,dword ptr \[esi\]   
 00412C05: 83F800                    cmp eax,00000000   
 00412C08: 7412                      jz 00412C1C   
 00412C0A: 2B8579B44000              sub eax,dword ptr \[ebp+0040B479\]   
 00412C10: 2020                      and byte ptr \[eax\],ah   
 00412C12: 2020                      and byte ptr \[eax\],ah   
 00412C14: 2032                      and byte ptr \[edx\],dh   
 00412C16: 37                        aaa   
 00412C17: 3133                      xor dword ptr \[ebx\],esi   
 00412C19: 3800                      cmp byte ptr \[eax\],al   
 00412C1B: 0000                      add byte ptr \[eax\],al   
 00412C1D: 0000                      add byte ptr \[eax\],al   
 00412C1F: 0000                      add byte ptr \[eax\],al   
 00412C00: 0074218B                  add byte ptr \[ecx-75\],dh   
 00412C04: 06                        push es   
 00412C05: 83F800                    cmp eax,00000000   
 00412C08: 7412                      jz 00412C1C   
 00412C0A: 2B8579B44000              sub eax,dword ptr \[ebp+0040B479\]   
 00412C10: 0385D3B24000              add eax,dword ptr \[ebp+0040B2D3\]   
 00412C16: C70000000000              mov dword ptr \[eax\],00000000   
 00412C1C: 83C604                    add esi,00000004   
 00412C1F: 83E904                    sub ecx,00000004   
 00412C22: 75DF                      jnz 00412C03   
 00412C24: 61                        popad    
 00412C25: C3                        ret   
  
  
 ---------------------显示 NAG 窗口------------------------  
 00412C26: B865B74000                mov eax,0040B765   
 00412C2B: 03C5                      add eax,ebp          <--413418  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
                  欢迎你使用幻影 1.5 beta2，  
  
   幻影 v1.5 是共享软件，如果你喜欢它，希望你注册。   
  
                    
  
                   E-mail   : ding-boy@netease.com   
                   Homepage : http://fsdb.yeah.net               
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412C2D: 50                        push eax   
 00412C2E: FF95C0BC4000              call dword ptr \[ebp+0040BCC0\]  <--ShowTryWindow  
 00412C34: C3                        ret   
\---------------------------  
  
 ----------------查寻注册键，找到则进行还原并且返回 1，否则返回 0-----------  
 00412C35: 60                        pushad    
 00412C36: 66C7855EB140000000        mov word ptr \[ebp+0040B15E\],0000  <--412E11  
 00412C3F: B848B14000                mov eax,0040B148   
 00412C44: 03C5                      add eax,ebp            <--412DFB  
 00412C46: BB40B14000                mov ebx,0040B140   
 00412C4B: 03DD                      add ebx,ebp            <--412DF3  
 00412C4D: B91EB14000                mov ecx,0040B11E   
 00412C52: 03CD                      add ecx,ebp            <--412DD1  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 wrifile\\shell\\open\\command\\config  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412C54: 50                        push eax   
 00412C55: 53                        push ebx   
 00412C56: 683F001F00                push 001F003F   
 00412C5B: 6A00                      push 00000000   
 00412C5D: 6A00                      push 00000000   
 00412C5F: 6A00                      push 00000000   
 00412C61: 6A00                      push 00000000   
 00412C63: 51                        push ecx   
 00412C64: 6800000080                push 80000000   
 00412C69: FF9576BC4000              call dword ptr \[ebp+0040BC76\]    <--RegCreateKeyA  
 00412C6F: 663D0000                  cmp ax,0000   
 00412C73: 7566                      jnz 00412CDB   
  
 00412C75: BE1AB14000                mov esi,0040B11A            
 00412C7A: 03F5                      add esi,ebp            <--412DCD  
 00412C7C: BF16B14000                mov edi,0040B116   
 00412C81: 03FD                      add edi,ebp            <--  
 00412C83: B860B14000                mov eax,0040B160   
 00412C88: 03C5                      add eax,ebp            <--412E13  
 00412C8A: BB50B14000                mov ebx,0040B150   
 00412C8F: 03DD                      add ebx,ebp            <--412E03  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  91 80 88 80 B8 80 99 80 EE 82  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412C91: 8B8D40B14000              mov ecx,dword ptr \[ebp+0040B140\]   
 00412C97: 56                        push esi              <--412DCD "="  
 00412C98: 50                        push eax              <--412E13   
 00412C99: 57                        push edi   
 00412C9A: 6A00                      push 00000000   
 00412C9C: 53                        push ebx   
 00412C9D: 51                        push ecx   
 00412C9E: FF957EBC4000              call dword ptr \[ebp+0040BC7E\]    <--RegQueryValueEx  
 00412CA4: 663D0000                  cmp ax,0000   
 00412CA8: 7531                      jnz 00412CDB   
 00412CAA: 66C7855EB140000100        mov word ptr \[ebp+0040B15E\],0001  <--412E11  
 00412CB3: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]    <--41311A  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  EE 82  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412CBA: 8AE0                      mov ah,al   
 00412CBC: BE60B14000                mov esi,0040B160   
 00412CC1: 03F5                      add esi,ebp            <--412DE13  
 \*\*\*\*\*\*\*\*\*\*\*\*-------------------关键的还原-------------\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412CC3: B93D000000                mov ecx,0000003D   
 00412CC8: 51                        push ecx   
 00412CC9: 8A06                      mov al,byte ptr \[esi\]   
 00412CCB: 3C00                      cmp al,00   
 00412CCD: 7402                      jz 00412CD1   
 00412CCF: 32C4                      xor al,ah   
 00412CD1: B104                      mov cl,04   
 00412CD3: D2C8                      ror al,cl   
 00412CD5: 8806                      mov byte ptr \[esi\],al   
 00412CD7: 46                        inc esi   
 00412CD8: 59                        pop ecx   
 00412CD9: E2ED                      loop 00412CC8   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*-----------------------------------\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412CDB: 8B8540B14000              mov eax,dword ptr \[ebp+0040B140\]   
 00412CE1: 50                        push eax   
 00412CE2: FF957ABC4000              call dword ptr \[ebp+0040BC7A\]    <--RegCloseKeyA  
 00412CE8: 61                        popad    
 00412CE9: 668B855EB14000            mov ax,word ptr \[ebp+0040B15E\]   
 00412CF0: C3                        ret   
\----------------------------------------------------------------  
  
\-------------------------保存注册信息到注册表--------------------------------  
 00412CF1: 60                        pushad    
 00412CF2: 66C7855EB140000000        mov word ptr \[ebp+0040B15E\],0000   
 00412CFB: B848B14000                mov eax,0040B148   
 00412D00: 03C5                      add eax,ebp   
 00412D02: BB40B14000                mov ebx,0040B140   
 00412D07: 03DD                      add ebx,ebp   
 00412D09: B91EB14000                mov ecx,0040B11E   
 00412D0E: 03CD                      add ecx,ebp            <--412DD1  
 00412D10: 50                        push eax   
 00412D11: 53                        push ebx   
 00412D12: 683F001F00                push 001F003F   
 00412D17: 6A00                      push 00000000   
 00412D19: 6A00                      push 00000000   
 00412D1B: 6A00                      push 00000000   
 00412D1D: 6A00                      push 00000000   
 00412D1F: 51                        push ecx              <--412DD1  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 wrifile\\shell\\open\\command\\config  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412D20: 6800000080                push 80000000   
 00412D25: FF9576BC4000              call dword ptr \[ebp+0040BC76\]    <--RegCreateKeyA  
 00412D2B: 663D0000                  cmp ax,0000   
 00412D2F: 755A                      jnz 00412D8B   
 00412D31: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]    <--41311A  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  EE 82  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412D38: 8AE0                      mov ah,al   
 00412D3A: BE60B14000                mov esi,0040B160   
 00412D3F: 03F5                      add esi,ebp   
 00412D41: B93D000000                mov ecx,0000003D   
 00412D46: 51                        push ecx   
 00412D47: 8A06                      mov al,byte ptr \[esi\]   
 00412D49: B104                      mov cl,04   
 00412D4B: D2C0                      rol al,cl   
 00412D4D: 3C00                      cmp al,00   
 00412D4F: 7402                      jz 00412D53   
 00412D51: 32C4                      xor al,ah   
 00412D53: 8806                      mov byte ptr \[esi\],al   
 00412D55: 46                        inc esi   
 00412D56: 59                        pop ecx   
 00412D57: E2ED                      loop 00412D46   
  
 00412D59: B860B14000                mov eax,0040B160            
 00412D5E: 03C5                      add eax,ebp            <--412DE13  
 00412D60: BB50B14000                mov ebx,0040B150   
 00412D65: 03DD                      add ebx,ebp            <--412DE03  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  91 80 88 80 B8 80 99 80 EE 82  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412D67: 8B8D40B14000              mov ecx,dword ptr \[ebp+0040B140\]   
 00412D6D: 6A3D                      push 0000003D   
 00412D6F: 50                        push eax   
 00412D70: 6A03                      push 00000003   
 00412D72: 6A00                      push 00000000   
 00412D74: 53                        push ebx   
 00412D75: 51                        push ecx   
 00412D76: FF9582BC4000              call dword ptr \[ebp+0040BC82\]    <--RegSetValueExA  
 00412D7C: 663D0000                  cmp ax,0000   
 00412D80: 7509                      jnz 00412D8B   
 00412D82: 66C7855EB140000100        mov word ptr \[ebp+0040B15E\],0001   
  
 00412D8B: 8B8540B14000              mov eax,dword ptr \[ebp+0040B140\]   
 00412D91: 50                        push eax   
 00412D92: FF957ABC4000              call dword ptr \[ebp+0040BC7A\]    <--RegCloseKeyA  
 00412D98: 668B8567B44000            mov ax,word ptr \[ebp+0040B467\]    <--41311A  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  EE 82  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 00412D9F: 8AE0                      mov ah,al   
 00412DA1: BE60B14000                mov esi,0040B160   
 00412DA6: 03F5                      add esi,ebp            <--412DE13  
 00412DA8: B93D000000                mov ecx,0000003D   
 00412DAD: 51                        push ecx   
 00412DAE: 8A06                      mov al,byte ptr \[esi\]   
 00412DB0: 3C00                      cmp al,00   
 00412DB2: 7402                      jz 00412DB6   
 00412DB4: 32C4                      xor al,ah   
 00412DB6: B104                      mov cl,04   
 00412DB8: D2C8                      ror al,cl   
 00412DBA: 8806                      mov byte ptr \[esi\],al   
 00412DBC: 46                        inc esi   
 00412DBD: 59                        pop ecx   
 00412DBE: E2ED                      loop 00412DAD   
 00412DC0: 61                        popad    
 00412DC1: 668B855EB14000            mov ax,word ptr \[ebp+0040B15E\]   
 00412DC8: C3                        ret   
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
\-------------------------------非代码区----------------------------  
 00412DC9: 0300                      add eax,dword ptr \[eax\]   
 00412DCB: 0000                      add byte ptr \[eax\],al   
 00412DCD: 3D00000077                cmp eax,77000000   
 00412DD2: 7269                      jb 00412E3D   
 00412DD4: 66696C655C7368            imul bp,word ptr \[ebp+5C\],6873   
 00412DDB: 656C                      insb es:\[edi\],dx   
 00412DDD: 6C                        insb es:\[edi\],dx   
 00412DDE: 5C                        pop esp   
 00412DDF: 6F                        outsd dx,ds:\[esi\]   
 00412DE0: 7065                      jo 00412E47   
 00412DE2: 6E                        outsb dx,ds:\[esi\]   
 00412DE3: 5C                        pop esp   
 00412DE4: 636F6D                    arpl word ptr \[edi+6D\],bp   
 00412DE7: 6D                        insd es:\[edi\],dx   
 00412DE8: 61                        popad    
 00412DE9: 6E                        outsb dx,ds:\[esi\]   
 00412DEA: 645C                      pop esp   
 00412DEC: 636F6E                    arpl word ptr \[edi+6E\],bp   
 00412DEF: 666967000000              imul sp,word ptr \[edi\],0000   
 00412DF5: 0000                      add byte ptr \[eax\],al   
 00412DF7: 0000                      add byte ptr \[eax\],al   
 00412DF9: 0000                      add byte ptr \[eax\],al   
 00412DFB: 0000                      add byte ptr \[eax\],al   
 00412DFD: 0000                      add byte ptr \[eax\],al   
 00412DFF: 0000                      add byte ptr \[eax\],al   
 00412E01: 0000                      add byte ptr \[eax\],al   
 00412E03: 91                        xchg eax,ecx   
 00412E04: 808880B8809980            or byte ptr \[eax-667F4780\],80   
 00412E0B: EE                        out dx,al   
 00412E0C: 820000                    add byte ptr \[eax\],00   
 00412E0F: 0000                      add byte ptr \[eax\],al   
 00412E11: 0000                      add byte ptr \[eax\],al   
 00412E13: 0000                      add byte ptr \[eax\],al   
 00412E15: 0000                      add byte ptr \[eax\],al   
 00412E17: 0000                      add byte ptr \[eax\],al   
 00412E19: 0000                      add byte ptr \[eax\],al   
 00412E1B: 0000                      add byte ptr \[eax\],al   
 00412E1D: 0000                      add byte ptr \[eax\],al   
 00412E1F: 0000                      add byte ptr \[eax\],al   
 00412E21: 0000                      add byte ptr \[eax\],al   
 00412E23: 0000                      add byte ptr \[eax\],al   
 00412E25: 0000                      add byte ptr \[eax\],al   
 00412E27: 0000                      add byte ptr \[eax\],al   
 00412E29: 0000                      add byte ptr \[eax\],al   
 00412E2B: 0000                      add byte ptr \[eax\],al   
 00412E2D: 0000                      add byte ptr \[eax\],al   
 00412E2F: 0000                      add byte ptr \[eax\],al   
 00412E31: 0000                      add byte ptr \[eax\],al   
 00412E33: 0000                      add byte ptr \[eax\],al   
 00412E35: 0000                      add byte ptr \[eax\],al   
 00412E37: 0000                      add byte ptr \[eax\],al   
 00412E39: 0000                      add byte ptr \[eax\],al   
 00412E3B: 0000                      add byte ptr \[eax\],al   
 00412E3D: 0000                      add byte ptr \[eax\],al   
 00412E3F: 0000                      add byte ptr \[eax\],al   
 00412E41: 0000                      add byte ptr \[eax\],al   
 00412E43: 0000                      add byte ptr \[eax\],al   
 00412E45: 0000                      add byte ptr \[eax\],al   
 00412E47: 0000                      add byte ptr \[eax\],al   
 00412E49: 0000                      add byte ptr \[eax\],al   
 00412E4B: 0000                      add byte ptr \[eax\],al   
 00412E4D: 0000                      add byte ptr \[eax\],al   
 00412E4F: 0000                      add byte ptr \[eax\],al   
 00412E51: 0000                      add byte ptr \[eax\],al   
 00412E53: 0000                      add byte ptr \[eax\],al   
 00412E55: 0000                      add byte ptr \[eax\],al   
 00412E57: 0000                      add byte ptr \[eax\],al   
 00412E59: 0000                      add byte ptr \[eax\],al   
 00412E5B: 0000                      add byte ptr \[eax\],al   
 00412E5D: 0000                      add byte ptr \[eax\],al   
 00412E5F: 0000                      add byte ptr \[eax\],al   
 00412E61: 0000                      add byte ptr \[eax\],al   
 00412E63: 0000                      add byte ptr \[eax\],al   
 00412E65: 0000                      add byte ptr \[eax\],al   
 00412E67: 0000                      add byte ptr \[eax\],al   
 00412E69: 0000                      add byte ptr \[eax\],al   
 00412E6B: 0000                      add byte ptr \[eax\],al   
 00412E6D: 0000                      add byte ptr \[eax\],al   
 00412E6F: 0000                      add byte ptr \[eax\],al   
 00412E71: 0000                      add byte ptr \[eax\],al   
 00412E73: 0000                      add byte ptr \[eax\],al   
 00412E75: 0000                      add byte ptr \[eax\],al   
 00412E77: 0000                      add byte ptr \[eax\],al   
 00412E79: 0000                      add byte ptr \[eax\],al   
 00412E7B: 0000                      add byte ptr \[eax\],al   
 00412E7D: 0000                      add byte ptr \[eax\],al   
 00412E7F: 0000                      add byte ptr \[eax\],al   
 00412E81: 0000                      add byte ptr \[eax\],al   
 00412E83: 0000                      add byte ptr \[eax\],al   
 00412E85: 0000                      add byte ptr \[eax\],al   
 00412E87: 0000                      add byte ptr \[eax\],al   
 00412E89: 0000                      add byte ptr \[eax\],al   
 00412E8B: 0000                      add byte ptr \[eax\],al   
 00412E8D: 0000                      add byte ptr \[eax\],al   
 00412E8F: 0000                      add byte ptr \[eax\],al   
 00412E91: 0000                      add byte ptr \[eax\],al   
 00412E93: 0000                      add byte ptr \[eax\],al   
 00412E95: 0000                      add byte ptr \[eax\],al   
 00412E97: 0000                      add byte ptr \[eax\],al   
 00412E99: 0000                      add byte ptr \[eax\],al   
 00412E9B: 0000                      add byte ptr \[eax\],al   
 00412E9D: 0000                      add byte ptr \[eax\],al   
 00412E9F: 0000                      add byte ptr \[eax\],al   
 00412EA1: 0000                      add byte ptr \[eax\],al   
 00412EA3: 0000                      add byte ptr \[eax\],al   
 00412EA5: 0000                      add byte ptr \[eax\],al   
 00412EA7: 0000                      add byte ptr \[eax\],al   
 00412EA9: 0000                      add byte ptr \[eax\],al   
 00412EAB: 0000                      add byte ptr \[eax\],al   
 00412EAD: 0003                      add byte ptr \[ebx\],al   
 00412EAF: 0000                      add byte ptr \[eax\],al   
 00412EB1: 0029                      add byte ptr \[ecx\],ch   
 00412EB3: 0000                      add byte ptr \[eax\],al   
 00412EB5: 002A                      add byte ptr \[edx\],ch   
 00412EB7: 00494E                    add byte ptr \[ecx+4E\],cl   
 00412EBA: 5F                        pop edi   
 00412EBB: 43                        inc ebx   
 00412EBC: 4F                        dec edi   
 00412EBD: 44                        inc esp   
 00412EBE: 45                        inc ebp   
 00412EBF: 0000                      add byte ptr \[eax\],al   
 00412EC1: 0000                      add byte ptr \[eax\],al   
 00412EC3: 0020                      add byte ptr \[eax\],ah   
 00412EC5: 2020                      and byte ptr \[eax\],ah   
 00412EC7: 2020                      and byte ptr \[eax\],ah   
 00412EC9: 2020                      and byte ptr \[eax\],ah   
 00412ECB: 2020                      and byte ptr \[eax\],ah   
 00412ECD: 2020                      and byte ptr \[eax\],ah   
 00412ECF: B1BE                      mov cl,BE   
 00412ED1: C8EDBCFE                  enter BCED,FE   
 00412ED5: D3C9                      ror ecx,cl   
 00412ED7: 20BBC3D3B076              and byte ptr \[ebx+76B0D3C3\],bh   
 00412EDD: 312E                      xor dword ptr \[esi\],ebp   
 00412EDF: 3520CEB4D7                xor eax,D7B4CE20   
 00412EE4: A2B2E1B0E6                mov \[E6B0E1B2\],al   
 00412EE9: B1BE                      mov cl,BE   
 00412EEB: BCD3C3DC20                mov esp,20DCC3D3   
 00412EF0: 2020                      and byte ptr \[eax\],ah   
 00412EF2: 2020                      and byte ptr \[eax\],ah   
 00412EF4: 2032                      and byte ptr \[edx\],dh   
 00412EF6: 37                        aaa   
 00412EF7: 3133                      xor dword ptr \[ebx\],esi   
 00412EF9: 3800                      cmp byte ptr \[eax\],al   
 00412EFB: 0000                      add byte ptr \[eax\],al   
 00412EFD: 0000                      add byte ptr \[eax\],al   
 00412EFF: 0000                      add byte ptr \[eax\],al   
 00412EE0: 20CE                      and dh,cl   
 00412EE2: B4D7                      mov ah,D7   
 00412EE4: A2B2E1B0E6                mov \[E6B0E1B2\],al   
 00412EE9: B1BE                      mov cl,BE   
 00412EEB: BCD3C3DC20                mov esp,20DCC3D3   
 00412EF0: 2020                      and byte ptr \[eax\],ah   
 00412EF2: 2020                      and byte ptr \[eax\],ah   
 00412EF4: 2020                      and byte ptr \[eax\],ah   
 00412EF6: 2020                      and byte ptr \[eax\],ah   
 00412EF8: 2020                      and byte ptr \[eax\],ah   
 00412EFA: 0D0A0D0A20                or eax,200A0D0A   
 00412EFF: 54                        push esp   
 00412F00: 6869732070                push 70207369   
 00412F05: 726F                      jb 00412F76   
 00412F07: 677261                    jb 00412F6B   
 00412F0A: 6D                        insd es:\[edi\],dx   
 00412F0B: 207072                    and byte ptr \[eax+72\],dh   
 00412F0E: 6F                        outsd dx,ds:\[esi\]   
 00412F0F: 7465                      jz 00412F76   
 00412F11: 63746564                  arpl word ptr \[ebp+64\],si   
 00412F15: 206279                    and byte ptr \[edx+79\],ah   
 00412F18: 20444250                  and byte ptr \[edx+eax\*2+50\],al   
 00412F1C: 45                        inc ebp   
 00412F1D: 207631                    and byte ptr \[esi+31\],dh   
 00412F20: 2E352020556E              xor eax,6E552020   
 00412F26: 7265                      jb 00412F8D   
 00412F28: 6769737465726564          imul esi,dword ptr \[bp+di+74\],64657265   
 00412F30: 207665                    and byte ptr \[esi+65\],dh   
 00412F33: 7273                      jb 00412FA8   
 00412F35: 696F6E0D0A0D0A            imul ebp,dword ptr \[edi+6E\],0A0D0A0D   
 00412F3C: 0D0A0D0A20                or eax,200A0D0A   
 00412F41: 48                        dec eax   
 00412F42: 6F                        outsd dx,ds:\[esi\]   
 00412F43: 6D                        insd es:\[edi\],dx   
 00412F44: 657061                    jo 00412FA8   
 00412F47: 67653A20                  cmp ah,byte ptr gs:\[bx+si\]   
 00412F4B: 687474703A                push 3A707474   
 00412F50: 2F                        das   
 00412F51: 2F                        das   
 00412F52: 667364                    jnb 00412FB9   
 00412F55: 622E                      bound ebp,dword ptr \[esi\]   
 00412F57: 7965                      jns 00412FBE   
 00412F59: 61                        popad    
 00412F5A: 682E6E6574                push 74656E2E   
 00412F5F: 2020                      and byte ptr \[eax\],ah   
 00412F61: 2020                      and byte ptr \[eax\],ah   
 00412F63: 45                        inc ebp   
 00412F64: 2D6D61696C                sub eax,6C69616D   
 00412F69: 3A20                      cmp ah,byte ptr \[eax\]   
 00412F6B: 44                        inc esp   
 00412F6C: 2E42                      inc edx   
 00412F6E: 6F                        outsd dx,ds:\[esi\]   
 00412F6F: 7940                      jns 00412FB1   
 00412F71: 3132                      xor dword ptr \[edx\],esi   
 00412F73: 362E636F6D                arpl word ptr cs:\[edi+6D\],bp   
 00412F78: 200D0A007C7C              and byte ptr \[7C7C000A\],cl   
 00412F7E: 00D0                      add al,dl   
 00412F80: 0000                      add byte ptr \[eax\],al   
 00412F82: CC                        int 3   
 00412F83: 1000                      adc byte ptr \[eax\],al   
 00412F85: 0000                      add byte ptr \[eax\],al   
 00412F87: 0000                      add byte ptr \[eax\],al   
 00412F89: 0000                      add byte ptr \[eax\],al   
 00412F8B: 1000                      adc byte ptr \[eax\],al   
 00412F8D: 0000                      add byte ptr \[eax\],al   
 00412F8F: 40                        inc eax   
 00412F90: 0000                      add byte ptr \[eax\],al   
 00412F92: 005000                    add byte ptr \[eax\],dl   
 00412F95: 0000                      add byte ptr \[eax\],al   
 00412F97: 1000                      adc byte ptr \[eax\],al   
 00412F99: 0000                      add byte ptr \[eax\],al   
 00412F9B: 60                        pushad    
 00412F9C: 0000                      add byte ptr \[eax\],al   
 00412F9E: 0010                      add byte ptr \[eax\],dl   
 00412FA0: 0000                      add byte ptr \[eax\],al   
 00412FA2: 00C0                      add al,al   
 00412FA4: 0000                      add byte ptr \[eax\],al   
 00412FA6: 0010                      add byte ptr \[eax\],dl   
 00412FA8: 0000                      add byte ptr \[eax\],al   
 00412FAA: 0000                      add byte ptr \[eax\],al   
 00412FAC: 0000                      add byte ptr \[eax\],al   
 00412FAE: 0000                      add byte ptr \[eax\],al   
 00412FB0: 0000                      add byte ptr \[eax\],al   
 00412FB2: 0000                      add byte ptr \[eax\],al   
 00412FB4: 0000                      add byte ptr \[eax\],al   
 00412FB6: 0000                      add byte ptr \[eax\],al   
 00412FB8: 0000                      add byte ptr \[eax\],al   
 00412FBA: 0000                      add byte ptr \[eax\],al   
 00412FBC: 0000                      add byte ptr \[eax\],al   
 00412FBE: 0000                      add byte ptr \[eax\],al   
 00412FC0: 0000                      add byte ptr \[eax\],al   
 00412FC2: 0000                      add byte ptr \[eax\],al   
 00412FC4: 0000                      add byte ptr \[eax\],al   
 00412FC6: 0000                      add byte ptr \[eax\],al   
 00412FC8: 0000                      add byte ptr \[eax\],al   
 00412FCA: 0000                      add byte ptr \[eax\],al   
 00412FCC: 0000                      add byte ptr \[eax\],al   
 00412FCE: 0000                      add byte ptr \[eax\],al   
 00412FD0: 0000                      add byte ptr \[eax\],al   
 00412FD2: 0000                      add byte ptr \[eax\],al   
 00412FD4: 0000                      add byte ptr \[eax\],al   
 00412FD6: 0000                      add byte ptr \[eax\],al   
 00412FD8: 0000                      add byte ptr \[eax\],al   
 00412FDA: 0000                      add byte ptr \[eax\],al   
 00412FDC: 0000                      add byte ptr \[eax\],al   
 00412FDE: 0000                      add byte ptr \[eax\],al   
 00412FE0: 0000                      add byte ptr \[eax\],al   
 00412FE2: 0000                      add byte ptr \[eax\],al   
 00412FE4: 0000                      add byte ptr \[eax\],al   
 00412FE6: 0000                      add byte ptr \[eax\],al   
 00412FE8: 0000                      add byte ptr \[eax\],al   
 00412FEA: 0000                      add byte ptr \[eax\],al   
 00412FEC: 0000                      add byte ptr \[eax\],al   
 00412FEE: 0000                      add byte ptr \[eax\],al   
 00412FF0: 0000                      add byte ptr \[eax\],al   
 00412FF2: 0000                      add byte ptr \[eax\],al   
 00412FF4: 0000                      add byte ptr \[eax\],al   
 00412FF6: 0000                      add byte ptr \[eax\],al   
 00412FF8: 0000                      add byte ptr \[eax\],al   
 00412FFA: 0000                      add byte ptr \[eax\],al   
 00412FFC: 0000                      add byte ptr \[eax\],al   
 00412FFE: 0000                      add byte ptr \[eax\],al   
 00413000: 0000                      add byte ptr \[eax\],al   
 00413002: 0000                      add byte ptr \[eax\],al   
 00413004: 0000                      add byte ptr \[eax\],al   
 00413006: 0000                      add byte ptr \[eax\],al   
 00413008: 0000                      add byte ptr \[eax\],al   
 0041300A: 0000                      add byte ptr \[eax\],al   
 0041300C: 0000                      add byte ptr \[eax\],al   
 0041300E: 0000                      add byte ptr \[eax\],al   
 00413010: 0000                      add byte ptr \[eax\],al   
 00413012: 0000                      add byte ptr \[eax\],al   
 00413014: 0000                      add byte ptr \[eax\],al   
 00413016: 0000                      add byte ptr \[eax\],al   
 00413018: 0000                      add byte ptr \[eax\],al   
 0041301A: 0000                      add byte ptr \[eax\],al   
 0041301C: 0000                      add byte ptr \[eax\],al   
 0041301E: 0000                      add byte ptr \[eax\],al   
 00413020: 0000                      add byte ptr \[eax\],al   
 00413022: 0000                      add byte ptr \[eax\],al   
 00413024: 0000                      add byte ptr \[eax\],al   
 00413026: 0000                      add byte ptr \[eax\],al   
 00413028: 0000                      add byte ptr \[eax\],al   
 0041302A: 0000                      add byte ptr \[eax\],al   
 0041302C: 0000                      add byte ptr \[eax\],al   
 0041302E: 0000                      add byte ptr \[eax\],al   
 00413030: 0000                      add byte ptr \[eax\],al   
 00413032: 0000                      add byte ptr \[eax\],al   
 00413034: 0000                      add byte ptr \[eax\],al   
 00413036: 0000                      add byte ptr \[eax\],al   
 00413038: 0000                      add byte ptr \[eax\],al   
 0041303A: 0000                      add byte ptr \[eax\],al   
 0041303C: 0000                      add byte ptr \[eax\],al   
 0041303E: 0000                      add byte ptr \[eax\],al   
 00413040: 0000                      add byte ptr \[eax\],al   
 00413042: 0000                      add byte ptr \[eax\],al   
 00413044: 0000                      add byte ptr \[eax\],al   
 00413046: 0000                      add byte ptr \[eax\],al   
 00413048: 0000                      add byte ptr \[eax\],al   
 0041304A: 0000                      add byte ptr \[eax\],al   
 0041304C: 0000                      add byte ptr \[eax\],al   
 0041304E: 0000                      add byte ptr \[eax\],al   
 00413050: 0000                      add byte ptr \[eax\],al   
 00413052: 0000                      add byte ptr \[eax\],al   
 00413054: 0000                      add byte ptr \[eax\],al   
 00413056: 0000                      add byte ptr \[eax\],al   
 00413058: 0000                      add byte ptr \[eax\],al   
 0041305A: 0000                      add byte ptr \[eax\],al   
 0041305C: 0000                      add byte ptr \[eax\],al   
 0041305E: 0000                      add byte ptr \[eax\],al   
 00413060: 0000                      add byte ptr \[eax\],al   
 00413062: 0000                      add byte ptr \[eax\],al   
 00413064: 0000                      add byte ptr \[eax\],al   
 00413066: 0000                      add byte ptr \[eax\],al   
 00413068: 0000                      add byte ptr \[eax\],al   
 0041306A: 0000                      add byte ptr \[eax\],al   
 0041306C: 0000                      add byte ptr \[eax\],al   
 0041306E: 0000                      add byte ptr \[eax\],al   
 00413070: 0000                      add byte ptr \[eax\],al   
 00413072: 0000                      add byte ptr \[eax\],al   
 00413074: 0000                      add byte ptr \[eax\],al   
 00413076: 0000                      add byte ptr \[eax\],al   
 00413078: 0000                      add byte ptr \[eax\],al   
 0041307A: 0000                      add byte ptr \[eax\],al   
 0041307C: 0000                      add byte ptr \[eax\],al   
 0041307E: 0000                      add byte ptr \[eax\],al   
 00413080: 0000                      add byte ptr \[eax\],al   
 00413082: 0000                      add byte ptr \[eax\],al   
 00413084: 0000                      add byte ptr \[eax\],al   
 00413086: 0000                      add byte ptr \[eax\],al   
 00413088: 0000                      add byte ptr \[eax\],al   
 0041308A: 0000                      add byte ptr \[eax\],al   
 0041308C: 0000                      add byte ptr \[eax\],al   
 0041308E: 0000                      add byte ptr \[eax\],al   
 00413090: 0000                      add byte ptr \[eax\],al   
 00413092: 0000                      add byte ptr \[eax\],al   
 00413094: 0000                      add byte ptr \[eax\],al   
 00413096: 0000                      add byte ptr \[eax\],al   
 00413098: 0000                      add byte ptr \[eax\],al   
 0041309A: 0000                      add byte ptr \[eax\],al   
 0041309C: 0000                      add byte ptr \[eax\],al   
 0041309E: 0000                      add byte ptr \[eax\],al   
 004130A0: 0000                      add byte ptr \[eax\],al   
 004130A2: 0000                      add byte ptr \[eax\],al   
 004130A4: 0000                      add byte ptr \[eax\],al   
 004130A6: 0000                      add byte ptr \[eax\],al   
 004130A8: 0000                      add byte ptr \[eax\],al   
 004130AA: 0000                      add byte ptr \[eax\],al   
 004130AC: 0000                      add byte ptr \[eax\],al   
 004130AE: 0000                      add byte ptr \[eax\],al   
 004130B0: 0000                      add byte ptr \[eax\],al  
004130B2: 0000                      add byte ptr \[eax\],al   
 004130B4: 0000                      add byte ptr \[eax\],al   
 004130B6: 0000                      add byte ptr \[eax\],al   
 004130B8: 0000                      add byte ptr \[eax\],al   
 004130BA: 0000                      add byte ptr \[eax\],al   
 004130BC: 0000                      add byte ptr \[eax\],al   
 004130BE: 0000                      add byte ptr \[eax\],al   
 004130C0: 0000                      add byte ptr \[eax\],al   
 004130C2: 0000                      add byte ptr \[eax\],al   
 004130C4: 0000                      add byte ptr \[eax\],al   
 004130C6: 0000                      add byte ptr \[eax\],al   
 004130C8: 0000                      add byte ptr \[eax\],al   
 004130CA: 0000                      add byte ptr \[eax\],al   
 004130CC: 0000                      add byte ptr \[eax\],al   
 004130CE: 0000                      add byte ptr \[eax\],al   
 004130D0: 0000                      add byte ptr \[eax\],al   
 004130D2: 0000                      add byte ptr \[eax\],al   
 004130D4: 0000                      add byte ptr \[eax\],al   
 004130D6: 0000                      add byte ptr \[eax\],al   
 004130D8: 0000                      add byte ptr \[eax\],al   
 004130DA: 0000                      add byte ptr \[eax\],al   
 004130DC: 0000                      add byte ptr \[eax\],al   
 004130DE: 0000                      add byte ptr \[eax\],al   
 004130E0: 0000                      add byte ptr \[eax\],al   
 004130E2: 0000                      add byte ptr \[eax\],al   
 004130E4: 0000                      add byte ptr \[eax\],al   
 004130E6: 0000                      add byte ptr \[eax\],al   
 004130E8: 0000                      add byte ptr \[eax\],al   
 004130EA: 0000                      add byte ptr \[eax\],al   
 004130EC: 0000                      add byte ptr \[eax\],al   
 004130EE: 0000                      add byte ptr \[eax\],al   
 004130F0: 0000                      add byte ptr \[eax\],al   
 004130F2: 0000                      add byte ptr \[eax\],al   
 004130F4: 0000                      add byte ptr \[eax\],al   
 004130F6: 0000                      add byte ptr \[eax\],al   
 004130F8: 0000                      add byte ptr \[eax\],al   
 004130FA: 0000                      add byte ptr \[eax\],al   
 004130FC: 0000                      add byte ptr \[eax\],al   
 004130FE: 0000                      add byte ptr \[eax\],al   
 00413100: 0000                      add byte ptr \[eax\],al   
 00413102: 0000                      add byte ptr \[eax\],al   
 00413104: 0000                      add byte ptr \[eax\],al   
 00413106: 0000                      add byte ptr \[eax\],al   
 00413108: 0000                      add byte ptr \[eax\],al   
 0041310A: 0000                      add byte ptr \[eax\],al   
 0041310C: 0000                      add byte ptr \[eax\],al   
 0041310E: 0000                      add byte ptr \[eax\],al   
 00413110: 0000                      add byte ptr \[eax\],al   
 00413112: 0000                      add byte ptr \[eax\],al   
 00413114: 0000                      add byte ptr \[eax\],al   
 00413116: 0000                      add byte ptr \[eax\],al   
 00413118: 0000                      add byte ptr \[eax\],al   
 0041311A: EE                        out dx,al   
 0041311B: 820060                    add byte ptr \[eax\],60   
 0041311E: 0000                      add byte ptr \[eax\],al   
 00413120: 00C0                      add al,al   
 00413122: 0000                      add byte ptr \[eax\],al   
 00413124: 0000                      add byte ptr \[eax\],al   
 00413126: 0000                      add byte ptr \[eax\],al   
 00413128: 0000                      add byte ptr \[eax\],al   
 0041312A: 0000                      add byte ptr \[eax\],al   
 0041312C: 0000                      add byte ptr \[eax\],al   
 0041312E: 40                        inc eax   
 0041312F: 0000                      add byte ptr \[eax\],al   
 00413131: 0000                      add byte ptr \[eax\],al   
 00413133: 00D0                      add al,dl   
 00413135: 07                        pop es   
 00413136: 0000                      add byte ptr \[eax\],al   
 00413138: 0A00                      or al,byte ptr \[eax\]   
 0041313A: 0000                      add byte ptr \[eax\],al   
 0041313C: 640000                    add byte ptr fs:\[eax\],al   
 0041313F: 00443A5C                  add byte ptr \[edx+edi+5C\],al   
 00413143: 54                        push esp   
 00413144: 45                        inc ebp   
 00413145: 4D                        dec ebp   
 00413146: 50                        push eax   
 00413147: 5C                        pop esp   
 00413148: 64627065                  bound esi,dword ptr fs:\[eax+65\]   
 0041314C: 5C                        pop esp   
 0041314D: 54                        push esp   
 0041314E: 45                        inc ebp   
 0041314F: 53                        push ebx   
 00413150: 54                        push esp   
 00413151: 5C                        pop esp   
 00413152: 54                        push esp   
 00413153: 45                        inc ebp   
 00413154: 53                        push ebx   
 00413155: 54                        push esp   
 00413156: 37                        aaa   
 00413157: 5C                        pop esp   
 00413158: 44                        inc esp   
 00413159: 42                        inc edx   
 0041315A: 4E                        dec esi   
 0041315B: 6F                        outsd dx,ds:\[esi\]   
 0041315C: 7465                      jz 004131C3   
 0041315E: 2E45                      inc ebp   
 00413160: 58                        pop eax   
 00413161: 45                        inc ebp   
 00413162: 0000                      add byte ptr \[eax\],al   
 00413164: 0000                      add byte ptr \[eax\],al   
 00413166: 0000                      add byte ptr \[eax\],al   
 00413168: 0000                      add byte ptr \[eax\],al   
 0041316A: 0000                      add byte ptr \[eax\],al   
 0041316C: 0000                      add byte ptr \[eax\],al   
 0041316E: 0000                      add byte ptr \[eax\],al   
 00413170: 0000                      add byte ptr \[eax\],al   
 00413172: 0000                      add byte ptr \[eax\],al   
 00413174: 0000                      add byte ptr \[eax\],al   
 00413176: 0000                      add byte ptr \[eax\],al   
 00413178: 0000                      add byte ptr \[eax\],al   
 0041317A: 0000                      add byte ptr \[eax\],al   
 0041317C: 0000                      add byte ptr \[eax\],al   
 0041317E: 0000                      add byte ptr \[eax\],al   
 00413180: 0000                      add byte ptr \[eax\],al   
 00413182: 0000                      add byte ptr \[eax\],al   
 00413184: 0000                      add byte ptr \[eax\],al   
 00413186: 0000                      add byte ptr \[eax\],al   
 00413188: 0000                      add byte ptr \[eax\],al   
 0041318A: 0000                      add byte ptr \[eax\],al   
 0041318C: 0000                      add byte ptr \[eax\],al   
 0041318E: 0000                      add byte ptr \[eax\],al   
 00413190: 0000                      add byte ptr \[eax\],al   
 00413192: 0000                      add byte ptr \[eax\],al   
 00413194: 0000                      add byte ptr \[eax\],al   
 00413196: 0000                      add byte ptr \[eax\],al   
 00413198: 0000                      add byte ptr \[eax\],al   
 0041319A: 0000                      add byte ptr \[eax\],al   
 0041319C: 0000                      add byte ptr \[eax\],al   
 0041319E: 0000                      add byte ptr \[eax\],al   
 004131A0: 0000                      add byte ptr \[eax\],al   
 004131A2: 0000                      add byte ptr \[eax\],al   
 004131A4: 2020                      and byte ptr \[eax\],ah   
 004131A6: 2020                      and byte ptr \[eax\],ah   
 004131A8: BBB6D3AD20                mov ebx,20ADD3B6   
 004131AD: 205E5F                    and byte ptr \[esi+5F\],bl   
 004131B0: 5E                        pop esi   
 004131B1: 0000                      add byte ptr \[eax\],al   
 004131B3: 0000                      add byte ptr \[eax\],al   
 004131B5: 0000                      add byte ptr \[eax\],al   
 004131B7: 0000                      add byte ptr \[eax\],al   
 004131B9: 0000                      add byte ptr \[eax\],al   
 004131BB: 0000                      add byte ptr \[eax\],al   
 004131BD: 0000                      add byte ptr \[eax\],al   
 004131BF: 0000                      add byte ptr \[eax\],al   
 004131C1: 0000                      add byte ptr \[eax\],al   
 004131C3: 0000                      add byte ptr \[eax\],al   
 004131C5: 0000                      add byte ptr \[eax\],al   
 004131C7: 0000                      add byte ptr \[eax\],al   
 004131C9: 0000                      add byte ptr \[eax\],al   
 004131CB: 0000                      add byte ptr \[eax\],al   
 004131CD: 0000                      add byte ptr \[eax\],al   
 004131CF: 0020                      add byte ptr \[eax\],ah   
 004131D1: 2020                      and byte ptr \[eax\],ah   
 004131D3: 2020                      and byte ptr \[eax\],ah   
 004131D5: 3237                      xor dh,byte ptr \[edi\]   
 004131D7: 3133                      xor dword ptr \[ebx\],esi   
 004131D9: 3800                      cmp byte ptr \[eax\],al   
 004131DB: 0000                      add byte ptr \[eax\],al   
 004131DD: 0000                      add byte ptr \[eax\],al   
 004131DF: 0000                      add byte ptr \[eax\],al   
 004131C0: 0000                      add byte ptr \[eax\],al   
 004131C2: 0000                      add byte ptr \[eax\],al   
 004131C4: 0000                      add byte ptr \[eax\],al   
 004131C6: 0000                      add byte ptr \[eax\],al   
 004131C8: 0000                      add byte ptr \[eax\],al   
 004131CA: 0000                      add byte ptr \[eax\],al   
 004131CC: 0000                      add byte ptr \[eax\],al   
 004131CE: 0000                      add byte ptr \[eax\],al   
 004131D0: 0000                      add byte ptr \[eax\],al   
 004131D2: 0000                      add byte ptr \[eax\],al   
 004131D4: 0000                      add byte ptr \[eax\],al   
 004131D6: 0000                      add byte ptr \[eax\],al   
 004131D8: 0000                      add byte ptr \[eax\],al   
 004131DA: 0000                      add byte ptr \[eax\],al   
 004131DC: 0000                      add byte ptr \[eax\],al   
 004131DE: 0000                      add byte ptr \[eax\],al   
 004131E0: 0000                      add byte ptr \[eax\],al   
 004131E2: 0000                      add byte ptr \[eax\],al   
 004131E4: 0000                      add byte ptr \[eax\],al   
 004131E6: 0000                      add byte ptr \[eax\],al   
 004131E8: 0000                      add byte ptr \[eax\],al   
 004131EA: 0000                      add byte ptr \[eax\],al   
 004131EC: 0000                      add byte ptr \[eax\],al   
 004131EE: 0000                      add byte ptr \[eax\],al   
 004131F0: 0000                      add byte ptr \[eax\],al   
 004131F2: 0000                      add byte ptr \[eax\],al   
 004131F4: 0000                      add byte ptr \[eax\],al   
 004131F6: 0000                      add byte ptr \[eax\],al   
 004131F8: 0000                      add byte ptr \[eax\],al   
 004131FA: 0000                      add byte ptr \[eax\],al   
 004131FC: 0000                      add byte ptr \[eax\],al   
 004131FE: 0000                      add byte ptr \[eax\],al   
 00413200: 0000                      add byte ptr \[eax\],al   
 00413202: 0000                      add byte ptr \[eax\],al   
 00413204: 0000                      add byte ptr \[eax\],al   
 00413206: 0000                      add byte ptr \[eax\],al   
 00413208: 0000                      add byte ptr \[eax\],al   
 0041320A: 0000                      add byte ptr \[eax\],al   
 0041320C: 0000                      add byte ptr \[eax\],al   
 0041320E: 0000                      add byte ptr \[eax\],al   
 00413210: 0000                      add byte ptr \[eax\],al   
 00413212: 0000                      add byte ptr \[eax\],al   
 00413214: 0000                      add byte ptr \[eax\],al   
 00413216: 0000                      add byte ptr \[eax\],al   
 00413218: 0000                      add byte ptr \[eax\],al   
 0041321A: 0000                      add byte ptr \[eax\],al   
 0041321C: 0000                      add byte ptr \[eax\],al   
 0041321E: 0000                      add byte ptr \[eax\],al   
 00413220: 0000                      add byte ptr \[eax\],al   
 00413222: 0000                      add byte ptr \[eax\],al   
 00413224: 0000                      add byte ptr \[eax\],al   
 00413226: 0000                      add byte ptr \[eax\],al   
 00413228: 0000                      add byte ptr \[eax\],al   
 0041322A: 0000                      add byte ptr \[eax\],al   
 0041322C: 0000                      add byte ptr \[eax\],al   
 0041322E: 0000                      add byte ptr \[eax\],al   
 00413230: 0000                      add byte ptr \[eax\],al   
 00413232: 0000                      add byte ptr \[eax\],al   
 00413234: 0000                      add byte ptr \[eax\],al   
 00413236: 0000                      add byte ptr \[eax\],al   
 00413238: 0000                      add byte ptr \[eax\],al   
 0041323A: 0000                      add byte ptr \[eax\],al   
 0041323C: 0000                      add byte ptr \[eax\],al   
 0041323E: 0000                      add byte ptr \[eax\],al   
 00413240: 0000                      add byte ptr \[eax\],al   
 00413242: 0000                      add byte ptr \[eax\],al   
 00413244: 0000                      add byte ptr \[eax\],al   
 00413246: 0000                      add byte ptr \[eax\],al   
 00413248: 0000                      add byte ptr \[eax\],al   
 0041324A: 0000                      add byte ptr \[eax\],al   
 0041324C: 0000                      add byte ptr \[eax\],al   
 0041324E: 0000                      add byte ptr \[eax\],al   
 00413250: 0000                      add byte ptr \[eax\],al   
 00413252: 0000                      add byte ptr \[eax\],al   
 00413254: 0000                      add byte ptr \[eax\],al   
 00413256: 0000                      add byte ptr \[eax\],al   
 00413258: 0000                      add byte ptr \[eax\],al   
 0041325A: 0000                      add byte ptr \[eax\],al   
 0041325C: 0000                      add byte ptr \[eax\],al   
 0041325E: 0000                      add byte ptr \[eax\],al   
 00413260: 0000                      add byte ptr \[eax\],al   
 00413262: 0000                      add byte ptr \[eax\],al   
 00413264: 0000                      add byte ptr \[eax\],al   
 00413266: 0000                      add byte ptr \[eax\],al   
 00413268: 0000                      add byte ptr \[eax\],al   
 0041326A: 0000                      add byte ptr \[eax\],al   
 0041326C: 0000                      add byte ptr \[eax\],al   
 0041326E: 0000                      add byte ptr \[eax\],al   
 00413270: 0000                      add byte ptr \[eax\],al   
 00413272: 0000                      add byte ptr \[eax\],al   
 00413274: 0000                      add byte ptr \[eax\],al   
 00413276: 0000                      add byte ptr \[eax\],al   
 00413278: 0000                      add byte ptr \[eax\],al   
 0041327A: 0000                      add byte ptr \[eax\],al   
 0041327C: 0000                      add byte ptr \[eax\],al   
 0041327E: 0000                      add byte ptr \[eax\],al   
 00413280: 0000                      add byte ptr \[eax\],al   
 00413282: 0000                      add byte ptr \[eax\],al   
 00413284: 0000                      add byte ptr \[eax\],al   
 00413286: 0000                      add byte ptr \[eax\],al   
 00413288: 0000                      add byte ptr \[eax\],al   
 0041328A: 0000                      add byte ptr \[eax\],al   
 0041328C: 0000                      add byte ptr \[eax\],al   
 0041328E: 0000                      add byte ptr \[eax\],al   
 00413290: 0000                      add byte ptr \[eax\],al   
 00413292: 0000                      add byte ptr \[eax\],al   
 00413294: 0000                      add byte ptr \[eax\],al   
 00413296: 0000                      add byte ptr \[eax\],al   
 00413298: 0000                      add byte ptr \[eax\],al   
 0041329A: 0000                      add byte ptr \[eax\],al   
 0041329C: 0000                      add byte ptr \[eax\],al   
 0041329E: 0000                      add byte ptr \[eax\],al   
 004132A0: 0000                      add byte ptr \[eax\],al   
 004132A2: 0000                      add byte ptr \[eax\],al   
 004132A4: 0000                      add byte ptr \[eax\],al   
 004132A6: 0000                      add byte ptr \[eax\],al   
 004132A8: 0000                      add byte ptr \[eax\],al   
 004132AA: 0000                      add byte ptr \[eax\],al   
 004132AC: 0000                      add byte ptr \[eax\],al   
 004132AE: 0000                      add byte ptr \[eax\],al   
 004132B0: 0000                      add byte ptr \[eax\],al   
 004132B2: 0000                      add byte ptr \[eax\],al   
 004132B4: 0000                      add byte ptr \[eax\],al   
 004132B6: 0000                      add byte ptr \[eax\],al   
 004132B8: 0000                      add byte ptr \[eax\],al   
 004132BA: 0000                      add byte ptr \[eax\],al   
 004132BC: 0000                      add byte ptr \[eax\],al   
 004132BE: 0000                      add byte ptr \[eax\],al   
 004132C0: 0000                      add byte ptr \[eax\],al   
 004132C2: 0000                      add byte ptr \[eax\],al   
 004132C4: 0000                      add byte ptr \[eax\],al   
 004132C6: 0000                      add byte ptr \[eax\],al   
 004132C8: 0000                      add byte ptr \[eax\],al   
 004132CA: 0000                      add byte ptr \[eax\],al   
 004132CC: 0000                      add byte ptr \[eax\],al   
 004132CE: 0000                      add byte ptr \[eax\],al   
 004132D0: 2020                      and byte ptr \[eax\],ah   
 004132D2: 2020                      and byte ptr \[eax\],ah   
 004132D4: 2020                      and byte ptr \[eax\],ah   
 004132D6: 2020                      and byte ptr \[eax\],ah   
 004132D8: 2020                      and byte ptr \[eax\],ah   
 004132DA: CAD4D3                    retf D3D4   
 004132DD: C3                        ret   
 004132DE: C6DAD2                    mov dl,D2   
 004132E1: D1B9FDA1A320              sar dword ptr \[ecx+20A3A1FD\],1   
 004132E7: 0D0A0D0A00                or eax,000A0D0A   
 004132EC: 0000                      add byte ptr \[eax\],al   
 004132EE: 0000                      add byte ptr \[eax\],al   
 004132F0: 0000                      add byte ptr \[eax\],al   
 004132F2: 0000                      add byte ptr \[eax\],al   
 004132F4: 0000                      add byte ptr \[eax\],al   
 004132F6: 0000                      add byte ptr \[eax\],al   
 004132F8: 0000                      add byte ptr \[eax\],al   
 004132FA: 0000                      add byte ptr \[eax\],al   
 004132FC: 0000                      add byte ptr \[eax\],al   
 004132FE: 0000                      add byte ptr \[eax\],al   
 00413300: 0000                      add byte ptr \[eax\],al   
 00413302: 0000                      add byte ptr \[eax\],al   
 00413304: 0000                      add byte ptr \[eax\],al   
 00413306: 0000                      add byte ptr \[eax\],al   
 00413308: 0000                      add byte ptr \[eax\],al   
 0041330A: 0000                      add byte ptr \[eax\],al   
 0041330C: 0000                      add byte ptr \[eax\],al   
 0041330E: 0000                      add byte ptr \[eax\],al   
 00413310: 0000                      add byte ptr \[eax\],al   
 00413312: 0000                      add byte ptr \[eax\],al   
 00413314: 0000                      add byte ptr \[eax\],al   
 00413316: 0000                      add byte ptr \[eax\],al   
 00413318: 0000                      add byte ptr \[eax\],al   
 0041331A: 0000                      add byte ptr \[eax\],al   
 0041331C: 0000                      add byte ptr \[eax\],al   
 0041331E: 0000                      add byte ptr \[eax\],al   
 00413320: 0000                      add byte ptr \[eax\],al   
 00413322: 0000                      add byte ptr \[eax\],al   
 00413324: 0000                      add byte ptr \[eax\],al   
 00413326: 0000                      add byte ptr \[eax\],al   
 00413328: 0000                      add byte ptr \[eax\],al   
 0041332A: 0000                      add byte ptr \[eax\],al   
 0041332C: 0000                      add byte ptr \[eax\],al   
 0041332E: 0000                      add byte ptr \[eax\],al   
 00413330: 0000                      add byte ptr \[eax\],al   
 00413332: 0000                      add byte ptr \[eax\],al   
 00413334: 0000                      add byte ptr \[eax\],al   
 00413336: 0000                      add byte ptr \[eax\],al   
 00413338: 0000                      add byte ptr \[eax\],al   
 0041333A: 0000                      add byte ptr \[eax\],al   
 0041333C: 0000                      add byte ptr \[eax\],al   
 0041333E: 0000                      add byte ptr \[eax\],al   
 00413340: 0000                      add byte ptr \[eax\],al   
 00413342: 0000                      add byte ptr \[eax\],al   
 00413344: 0000                      add byte ptr \[eax\],al   
 00413346: 0000                      add byte ptr \[eax\],al   
 00413348: 0000                      add byte ptr \[eax\],al   
 0041334A: 0000                      add byte ptr \[eax\],al   
 0041334C: 0000                      add byte ptr \[eax\],al   
 0041334E: 0000                      add byte ptr \[eax\],al   
 00413350: 0000                      add byte ptr \[eax\],al   
 00413352: 0000                      add byte ptr \[eax\],al   
 00413354: 0000                      add byte ptr \[eax\],al   
 00413356: 0000                      add byte ptr \[eax\],al   
 00413358: 0000                      add byte ptr \[eax\],al   
 0041335A: 0000                      add byte ptr \[eax\],al   
 0041335C: 0000                      add byte ptr \[eax\],al   
 0041335E: 0000                      add byte ptr \[eax\],al   
 00413360: 0000                      add byte ptr \[eax\],al   
 00413362: 0000                      add byte ptr \[eax\],al   
 00413364: 0000                      add byte ptr \[eax\],al   
 00413366: 0000                      add byte ptr \[eax\],al   
 00413368: 0000                      add byte ptr \[eax\],al   
 0041336A: 0000                      add byte ptr \[eax\],al   
 0041336C: 0000                      add byte ptr \[eax\],al   
 0041336E: 0000                      add byte ptr \[eax\],al   
 00413370: 0000                      add byte ptr \[eax\],al   
 00413372: 0000                      add byte ptr \[eax\],al   
 00413374: 0000                      add byte ptr \[eax\],al   
 00413376: 0000                      add byte ptr \[eax\],al   
 00413378: 0000                      add byte ptr \[eax\],al   
 0041337A: 0000                      add byte ptr \[eax\],al   
 0041337C: 0000                      add byte ptr \[eax\],al   
 0041337E: 0000                      add byte ptr \[eax\],al   
 00413380: 0000                      add byte ptr \[eax\],al   
 00413382: 0000                      add byte ptr \[eax\],al   
 00413384: 0000                      add byte ptr \[eax\],al   
 00413386: 0000                      add byte ptr \[eax\],al   
 00413388: 0000                      add byte ptr \[eax\],al   
 0041338A: 0000                      add byte ptr \[eax\],al   
 0041338C: 0000                      add byte ptr \[eax\],al   
 0041338E: 0000                      add byte ptr \[eax\],al   
 00413390: 0000                      add byte ptr \[eax\],al   
 00413392: 0000                      add byte ptr \[eax\],al   
 00413394: 0000                      add byte ptr \[eax\],al   
 00413396: 0000                      add byte ptr \[eax\],al   
 00413398: 0000                      add byte ptr \[eax\],al   
 0041339A: 0000                      add byte ptr \[eax\],al   
 0041339C: 0000                      add byte ptr \[eax\],al   
 0041339E: 0000                      add byte ptr \[eax\],al   
 004133A0: 0000                      add byte ptr \[eax\],al   
 004133A2: 0000                      add byte ptr \[eax\],al   
 004133A4: 0000                      add byte ptr \[eax\],al   
 004133A6: 0000                      add byte ptr \[eax\],al   
 004133A8: 0000                      add byte ptr \[eax\],al   
 004133AA: 0000                      add byte ptr \[eax\],al   
 004133AC: 0000                      add byte ptr \[eax\],al   
 004133AE: 0000                      add byte ptr \[eax\],al   
 004133B0: 0000                      add byte ptr \[eax\],al   
 004133B2: 0000                      add byte ptr \[eax\],al   
 004133B4: 0000                      add byte ptr \[eax\],al   
 004133B6: 0000                      add byte ptr \[eax\],al   
 004133B8: 0000                      add byte ptr \[eax\],al   
 004133BA: 0000                      add byte ptr \[eax\],al   
 004133BC: 0000                      add byte ptr \[eax\],al   
 004133BE: 0000                      add byte ptr \[eax\],al   
 004133C0: 0000                      add byte ptr \[eax\],al   
 004133C2: 0000                      add byte ptr \[eax\],al   
 004133C4: 0000                      add byte ptr \[eax\],al   
 004133C6: 0000                      add byte ptr \[eax\],al   
 004133C8: 0000                      add byte ptr \[eax\],al   
 004133CA: 0000                      add byte ptr \[eax\],al   
 004133CC: 0000                      add byte ptr \[eax\],al   
 004133CE: 0000                      add byte ptr \[eax\],al   
 004133D0: 0000                      add byte ptr \[eax\],al   
 004133D2: 0000                      add byte ptr \[eax\],al   
 004133D4: 0000                      add byte ptr \[eax\],al   
 004133D6: 0000                      add byte ptr \[eax\],al   
 004133D8: 0000                      add byte ptr \[eax\],al   
 004133DA: 0000                      add byte ptr \[eax\],al   
 004133DC: 0000                      add byte ptr \[eax\],al   
 004133DE: 0000                      add byte ptr \[eax\],al   
 004133E0: 0000                      add byte ptr \[eax\],al   
 004133E2: 0000                      add byte ptr \[eax\],al   
 004133E4: 0000                      add byte ptr \[eax\],al   
 004133E6: 0000                      add byte ptr \[eax\],al   
 004133E8: 0000                      add byte ptr \[eax\],al   
 004133EA: 0000                      add byte ptr \[eax\],al   
 004133EC: 0000                      add byte ptr \[eax\],al   
 004133EE: 0000                      add byte ptr \[eax\],al   
 004133F0: 0000                      add byte ptr \[eax\],al   
 004133F2: 0000                      add byte ptr \[eax\],al   
 004133F4: 0000                      add byte ptr \[eax\],al   
 004133F6: 0000                      add byte ptr \[eax\],al   
 004133F8: 0000                      add byte ptr \[eax\],al   
 004133FA: 0000                      add byte ptr \[eax\],al   
 004133FC: 6C                        insb es:\[edi\],dx   
 004133FD: 6A74                      push 00000074   
 004133FF: 6C                        insb es:\[edi\],dx   
 00413400: 6A74                      push 00000074   
 00413402: 0000                      add byte ptr \[eax\],al   
 00413404: 0000                      add byte ptr \[eax\],al   
 00413406: 0000                      add byte ptr \[eax\],al   
 00413408: 0000                      add byte ptr \[eax\],al   
 0041340A: 0000                      add byte ptr \[eax\],al   
 0041340C: 0000                      add byte ptr \[eax\],al   
 0041340E: 0000                      add byte ptr \[eax\],al   
 00413410: D20400                    rol byte ptr \[eax+eax\],cl   
 00413413: 0001                      add byte ptr \[ecx\],al   
 00413415: 0000                      add byte ptr \[eax\],al   
 00413417: 0020                      add byte ptr \[eax\],ah   
 00413419: 2020                      and byte ptr \[eax\],ah   
 0041341B: 2020                      and byte ptr \[eax\],ah   
 0041341D: 2020                      and byte ptr \[eax\],ah   
 0041341F: 2020                      and byte ptr \[eax\],ah   
 00413421: 2020                      and byte ptr \[eax\],ah   
 00413423: 2020                      and byte ptr \[eax\],ah   
 00413425: 2020                      and byte ptr \[eax\],ah   
 00413427: 2020                      and byte ptr \[eax\],ah   
 00413429: BBB6D3ADC4                mov ebx,C4ADD3B6   
 0041342E: E3CA                      jecxz 004133FA   
 00413430: B9D3C3BBC3                mov ecx,C3BBC3D3   
 00413435: D3B020312E35              shl dword ptr \[eax+352E3120\],cl   
 0041343B: 206265                    and byte ptr \[edx+65\],ah   
 0041343E: 7461                      jz 004134A1   
 00413440: 32A3AC0D0A0D              xor ah,byte ptr \[ebx+0D0A0DAC\]   
 00413446: 0A20                      or ah,byte ptr \[eax\]   
 00413448: 2020                      and byte ptr \[eax\],ah   
 0041344A: BBC3D3B076                mov ebx,76B0D3C3   
 0041344F: 312E                      xor dword ptr \[esi\],ebp   
 00413451: 35CAC7B9B2                xor eax,B2B9C7CA   
 00413456: CF                        iretd    
 00413457: ED                        in eax,dx   
 00413458: C8EDBCFE                  enter BCED,FE   
 0041345C: A3ACC8E7B9                mov \[B9E7C8AC\],eax   
 00413461: FB                        sti   
 00413462: C4E3                      les esp,(Invalid)bx   
 00413464: CF                        iretd    
 00413465: B2BB                      mov dl,BB   
 00413467: B6CB                      mov dh,CB   
 00413469: FC                        cld   
 0041346A: A3ACCFA3CD                mov \[CDA3CFAC\],eax   
 0041346F: FB                        sti   
 00413470: C4E3                      les esp,(Invalid)bx   
 00413472: D7                        xlat   
 00413473: A2B2E1A1A3                mov \[A3A1E1B2\],al   
 00413478: 200D0A0D0A20              and byte ptr \[200A0D0A\],cl   
 0041347E: 2020                      and byte ptr \[eax\],ah   
 00413480: 2020                      and byte ptr \[eax\],ah   
 00413482: 2020                      and byte ptr \[eax\],ah   
 00413484: 2020                      and byte ptr \[eax\],ah   
 00413486: 2020                      and byte ptr \[eax\],ah   
 00413488: 2020                      and byte ptr \[eax\],ah   
 0041348A: 2020                      and byte ptr \[eax\],ah   
 0041348C: 2020                      and byte ptr \[eax\],ah   
 0041348E: 200D0A0D0A20              and byte ptr \[200A0D0A\],cl   
 00413494: 2020                      and byte ptr \[eax\],ah   
 00413496: 2020                      and byte ptr \[eax\],ah   
 00413498: 2020                      and byte ptr \[eax\],ah   
 0041349A: 2020                      and byte ptr \[eax\],ah   
 0041349C: 2020                      and byte ptr \[eax\],ah   
 0041349E: 2020                      and byte ptr \[eax\],ah   
 004134A0: 2020                      and byte ptr \[eax\],ah   
 004134A2: 2020                      and byte ptr \[eax\],ah   
 004134A4: 2020                      and byte ptr \[eax\],ah   
 004134A6: 45                        inc ebp   
 004134A7: 2D6D61696C                sub eax,6C69616D   
 004134AC: 2020                      and byte ptr \[eax\],ah   
 004134AE: 203A                      and byte ptr \[edx\],bh   
 004134B0: 2020                      and byte ptr \[eax\],ah   
 004134B2: 2020                      and byte ptr \[eax\],ah   
 004134B4: 2032                      and byte ptr \[edx\],dh   
 004134B6: 37                        aaa   
 004134B7: 3133                      xor dword ptr \[ebx\],esi   
 004134B9: 3800                      cmp byte ptr \[eax\],al   
 004134BB: 0000                      add byte ptr \[eax\],al   
 004134BD: 0000                      add byte ptr \[eax\],al   
 004134BF: 0000                      add byte ptr \[eax\],al   
 004134A0: 2020                      and byte ptr \[eax\],ah   
 004134A2: 2020                      and byte ptr \[eax\],ah   
 004134A4: 2020                      and byte ptr \[eax\],ah   
 004134A6: 45                        inc ebp   
 004134A7: 2D6D61696C                sub eax,6C69616D   
 004134AC: 2020                      and byte ptr \[eax\],ah   
 004134AE: 203A                      and byte ptr \[edx\],bh   
 004134B0: 2064696E                  and byte ptr \[ecx+ebp\*2+6E\],ah   
 004134B4: 672D626F7940              sub eax,40796F62   
 004134BA: 6E                        outsb dx,ds:\[esi\]   
 004134BB: 657465                    jz 00413523   
 004134BE: 61                        popad    
 004134BF: 7365                      jnb 00413526   
 004134C1: 2E636F6D                  arpl word ptr cs:\[edi+6D\],bp   
 004134C5: 200D0A202020              and byte ptr \[2020200A\],cl   
 004134CB: 2020                      and byte ptr \[eax\],ah   
 004134CD: 2020                      and byte ptr \[eax\],ah   
 004134CF: 2020                      and byte ptr \[eax\],ah   
 004134D1: 2020                      and byte ptr \[eax\],ah   
 004134D3: 2020                      and byte ptr \[eax\],ah   
 004134D5: 2020                      and byte ptr \[eax\],ah   
 004134D7: 2020                      and byte ptr \[eax\],ah   
 004134D9: 2020                      and byte ptr \[eax\],ah   
 004134DB: 48                        dec eax   
 004134DC: 6F                        outsd dx,ds:\[esi\]   
 004134DD: 6D                        insd es:\[edi\],dx   
 004134DE: 657061                    jo 00413542   
 004134E1: 6765203A                  and byte ptr gs:\[bp+si\],bh   
 004134E5: 206874                    and byte ptr \[eax+74\],ch   
 004134E8: 7470                      jz 0041355A   
 004134EA: 3A2F                      cmp ch,byte ptr \[edi\]   
 004134EC: 2F                        das   
 004134ED: 667364                    jnb 00413554   
 004134F0: 622E                      bound ebp,dword ptr \[esi\]   
 004134F2: 7965                      jns 00413559   
 004134F4: 61                        popad    
 004134F5: 682E6E6574                push 74656E2E   
 004134FA: 2020                      and byte ptr \[eax\],ah   
 004134FC: 2020                      and byte ptr \[eax\],ah   
 004134FE: 2020                      and byte ptr \[eax\],ah   
 00413500: 2020                      and byte ptr \[eax\],ah   
 00413502: 2020                      and byte ptr \[eax\],ah   
 00413504: 2020                      and byte ptr \[eax\],ah   
 00413506: 200D0A0D0A00              and byte ptr \[000A0D0A\],cl   
 0041350C: 0000                      add byte ptr \[eax\],al   
 0041350E: 0000                      add byte ptr \[eax\],al   
 00413510: 0000                      add byte ptr \[eax\],al   
 00413512: 0000                      add byte ptr \[eax\],al   
 00413514: 0000                      add byte ptr \[eax\],al   
 00413516: 0000                      add byte ptr \[eax\],al   
 00413518: 0000                      add byte ptr \[eax\],al   
 0041351A: 0000                      add byte ptr \[eax\],al   
 0041351C: 0000                      add byte ptr \[eax\],al   
 0041351E: 0000                      add byte ptr \[eax\],al   
 00413520: 0000                      add byte ptr \[eax\],al   
 00413522: 0000                      add byte ptr \[eax\],al   
 00413524: 0000                      add byte ptr \[eax\],al   
 00413526: 0000                      add byte ptr \[eax\],al   
 00413528: 0000                      add byte ptr \[eax\],al   
 0041352A: 0000                      add byte ptr \[eax\],al   
 0041352C: 0000                      add byte ptr \[eax\],al   
 0041352E: 0000                      add byte ptr \[eax\],al   
 00413530: 0000                      add byte ptr \[eax\],al   
 00413532: 0000                      add byte ptr \[eax\],al   
 00413534: 0000                      add byte ptr \[eax\],al   
 00413536: 0000                      add byte ptr \[eax\],al   
 00413538: 0000                      add byte ptr \[eax\],al   
 0041353A: 0000                      add byte ptr \[eax\],al   
 0041353C: 0000                      add byte ptr \[eax\],al   
 0041353E: 0000                      add byte ptr \[eax\],al   
 00413540: 0000                      add byte ptr \[eax\],al   
 00413542: 0000                      add byte ptr \[eax\],al   
 00413544: 49                        dec ecx   
 00413545: 6E                        outsb dx,ds:\[esi\]   
 00413546: 7661                      jbe 004135A9   
 00413548: 6C                        insb es:\[edi\],dx   
 00413549: 6964202050617373          imul esp,dword ptr \[eax+20\],73736150   
 00413551: 57                        push edi   
 00413552: 6F                        outsd dx,ds:\[esi\]   
 00413553: 7264                      jb 004135B9   
 00413555: 0000                      add byte ptr \[eax\],al   
 00413557: 0000                      add byte ptr \[eax\],al   
 00413559: 0000                      add byte ptr \[eax\],al   
 0041355B: 0000                      add byte ptr \[eax\],al   
 0041355D: 0000                      add byte ptr \[eax\],al   
 0041355F: 0000                      add byte ptr \[eax\],al   
 00413561: 0000                      add byte ptr \[eax\],al   
 00413563: 0000                      add byte ptr \[eax\],al   
 00413565: 0000                      add byte ptr \[eax\],al   
 00413567: 0000                      add byte ptr \[eax\],al   
 00413569: 0000                      add byte ptr \[eax\],al   
 0041356B: 0000                      add byte ptr \[eax\],al   
 0041356D: 0000                      add byte ptr \[eax\],al   
 0041356F: 0000                      add byte ptr \[eax\],al   
 00413571: 0000                      add byte ptr \[eax\],al   
 00413573: 0000                      add byte ptr \[eax\],al   
 00413575: 0000                      add byte ptr \[eax\],al   
 00413577: 0000                      add byte ptr \[eax\],al   
 00413579: 0000                      add byte ptr \[eax\],al   
 0041357B: 0000                      add byte ptr \[eax\],al   
 0041357D: 0000                      add byte ptr \[eax\],al   
 0041357F: 0000                      add byte ptr \[eax\],al   
 00413581: 0000                      add byte ptr \[eax\],al   
 00413583: 0000                      add byte ptr \[eax\],al   
 00413585: 0000                      add byte ptr \[eax\],al   
 00413587: 0000                      add byte ptr \[eax\],al   
 00413589: 0000                      add byte ptr \[eax\],al   
 0041358B: 0000                      add byte ptr \[eax\],al   
 0041358D: 0000                      add byte ptr \[eax\],al   
 0041358F: 0000                      add byte ptr \[eax\],al   
 00413591: 0000                      add byte ptr \[eax\],al   
 00413593: 0000                      add byte ptr \[eax\],al   
 00413595: 0000                      add byte ptr \[eax\],al   
 00413597: 0000                      add byte ptr \[eax\],al   
 00413599: 0000                      add byte ptr \[eax\],al   
 0041359B: 0000                      add byte ptr \[eax\],al   
 0041359D: 0000                      add byte ptr \[eax\],al   
 0041359F: 0000                      add byte ptr \[eax\],al   
 004135A1: 0000                      add byte ptr \[eax\],al   
 004135A3: 0000                      add byte ptr \[eax\],al   
 004135A5: 0000                      add byte ptr \[eax\],al   
 004135A7: 005265                    add byte ptr \[edx+65\],dl   
 004135AA: 6769737465722053          imul esi,dword ptr \[bp+di+74\],53207265   
 004135B2: 7563                      jnz 00413617   
 004135B4: 636573                    arpl word ptr \[ebp+73\],sp   
 004135B7: 7366                      jnb 0041361F   
 004135B9: 756C                      jnz 00413627   
 004135BB: 6C                        insb es:\[edi\],dx   
 004135BC: 7900                      jns 004135BE   
 004135BE: 0000                      add byte ptr \[eax\],al   
 004135C0: 0000                      add byte ptr \[eax\],al   
 004135C2: 0000                      add byte ptr \[eax\],al   
 004135C4: 0000                      add byte ptr \[eax\],al   
 004135C6: 0000                      add byte ptr \[eax\],al   
 004135C8: 0000                      add byte ptr \[eax\],al   
 004135CA: 0000                      add byte ptr \[eax\],al   
 004135CC: 0000                      add byte ptr \[eax\],al   
 004135CE: 0000                      add byte ptr \[eax\],al   
 004135D0: 0000                      add byte ptr \[eax\],al   
 004135D2: 0000                      add byte ptr \[eax\],al   
 004135D4: 0000                      add byte ptr \[eax\],al   
 004135D6: 0000                      add byte ptr \[eax\],al   
 004135D8: 0000                      add byte ptr \[eax\],al   
 004135DA: 0000                      add byte ptr \[eax\],al   
 004135DC: 0000                      add byte ptr \[eax\],al   
 004135DE: 0000                      add byte ptr \[eax\],al   
 004135E0: 0000                      add byte ptr \[eax\],al   
 004135E2: 0000                      add byte ptr \[eax\],al   
 004135E4: 0000                      add byte ptr \[eax\],al   
 004135E6: 0000                      add byte ptr \[eax\],al   
 004135E8: 0000                      add byte ptr \[eax\],al   
 004135EA: 0000                      add byte ptr \[eax\],al   
 004135EC: 0000                      add byte ptr \[eax\],al   
 004135EE: 0000                      add byte ptr \[eax\],al   
 004135F0: 0000                      add byte ptr \[eax\],al   
 004135F2: 0000                      add byte ptr \[eax\],al   
 004135F4: 0000                      add byte ptr \[eax\],al   
 004135F6: 0000                      add byte ptr \[eax\],al   
 004135F8: 0000                      add byte ptr \[eax\],al   
 004135FA: 0000                      add byte ptr \[eax\],al   
 004135FC: 0000                      add byte ptr \[eax\],al   
 004135FE: 0000                      add byte ptr \[eax\],al   
 00413600: 0000                      add byte ptr \[eax\],al   
 00413602: 0000                      add byte ptr \[eax\],al   
 00413604: 0000                      add byte ptr \[eax\],al   
 00413606: 0000                      add byte ptr \[eax\],al   
 00413608: 0000                      add byte ptr \[eax\],al   
 0041360A: 0000                      add byte ptr \[eax\],al   
 0041360C: 49                        dec ecx   
 0041360D: 6E                        outsb dx,ds:\[esi\]   
 0041360E: 7661                      jbe 00413671   
 00413610: 6C                        insb es:\[edi\],dx   
 00413611: 6964205365726961          imul esp,dword ptr \[eax+53\],61697265   
 00413619: 6C                        insb es:\[edi\],dx   
 0041361A: 204E4F                    and byte ptr \[esi+4F\],cl   
 0041361D: 0000                      add byte ptr \[eax\],al   
 0041361F: 0000                      add byte ptr \[eax\],al   
 00413621: 0000                      add byte ptr \[eax\],al   
 00413623: 0000                      add byte ptr \[eax\],al   
 00413625: 0000                      add byte ptr \[eax\],al   
 00413627: 0000                      add byte ptr \[eax\],al   
 00413629: 0000                      add byte ptr \[eax\],al   
 0041362B: 0000                      add byte ptr \[eax\],al   
 0041362D: 0000                      add byte ptr \[eax\],al   
 0041362F: 0000                      add byte ptr \[eax\],al   
 00413631: 0000                      add byte ptr \[eax\],al   
 00413633: 0000                      add byte ptr \[eax\],al   
 00413635: 0000                      add byte ptr \[eax\],al   
 00413637: 0000                      add byte ptr \[eax\],al   
 00413639: 0000                      add byte ptr \[eax\],al   
 0041363B: 0000                      add byte ptr \[eax\],al   
 0041363D: 0000                      add byte ptr \[eax\],al   
 0041363F: 0000                      add byte ptr \[eax\],al   
 00413641: 0000                      add byte ptr \[eax\],al   
 00413643: 0000                      add byte ptr \[eax\],al   
 00413645: 0000                      add byte ptr \[eax\],al   
 00413647: 0000                      add byte ptr \[eax\],al   
 00413649: 0000                      add byte ptr \[eax\],al   
 0041364B: 0000                      add byte ptr \[eax\],al   
 0041364D: 0000                      add byte ptr \[eax\],al   
 0041364F: 0000                      add byte ptr \[eax\],al   
 00413651: 0000                      add byte ptr \[eax\],al   
 00413653: 0000                      add byte ptr \[eax\],al   
 00413655: 0000                      add byte ptr \[eax\],al   
 00413657: 0000                      add byte ptr \[eax\],al   
 00413659: 0000                      add byte ptr \[eax\],al   
 0041365B: 0000                      add byte ptr \[eax\],al   
 0041365D: 0000                      add byte ptr \[eax\],al   
 0041365F: 0000                      add byte ptr \[eax\],al   
 00413661: 0000                      add byte ptr \[eax\],al   
 00413663: 0000                      add byte ptr \[eax\],al   
 00413665: 0000                      add byte ptr \[eax\],al   
 00413667: 0000                      add byte ptr \[eax\],al   
 00413669: 0000                      add byte ptr \[eax\],al   
 0041366B: 0000                      add byte ptr \[eax\],al   
 0041366D: 0000                      add byte ptr \[eax\],al   
 0041366F: 004520                    add byte ptr \[ebp+20\],al   
 00413672: 0020                      add byte ptr \[eax\],ah   
 00413674: 0020                      add byte ptr \[eax\],ah   
 00413676: 2020                      and byte ptr \[eax\],ah   
 00413678: 4C                        dec esp   
 00413679: 6F                        outsd dx,ds:\[esi\]   
 0041367A: 61                        popad    
 0041367B: 642020                    and byte ptr fs:\[eax\],ah   
 0041367E: 20446961                  and byte ptr \[ecx+ebp\*2+61\],al   
 00413682: 6C                        insb es:\[edi\],dx   
 00413683: 2E646C                    insb es:\[edi\],dx   
 00413686: 6C                        insb es:\[edi\],dx   
 00413687: 2020                      and byte ptr \[eax\],ah   
 00413689: 6661                      popa    
 0041368B: 6C                        insb es:\[edi\],dx   
 0041368C: 7365                      jnb 004136F3   
 0041368E: 642120                    and dword ptr fs:\[eax\],esp   
 00413691: 2020                      and byte ptr \[eax\],ah   
 00413693: 00D0                      add al,dl   
 00413695: 07                        pop es   
 00413696: 0800                      or byte ptr \[eax\],al   
 00413698: 0400                      add al,00   
 0041369A: 1100                      adc dword ptr \[eax\],eax   
 0041369C: 0800                      or byte ptr \[eax\],al   
 0041369E: 3800                      cmp byte ptr \[eax\],al   
 004136A0: 1900                      sbb dword ptr \[eax\],eax   
 004136A2: EE                        out dx,al   
 004136A3: 0200                      add al,byte ptr \[eax\]   
 004136A5: 346C                      xor al,6C   
 004136A7: 0000                      add byte ptr \[eax\],al   
 004136A9: 0000                      add byte ptr \[eax\],al   
 004136AB: 0000                      add byte ptr \[eax\],al   
 004136AD: 60                        pushad    
 004136AE: BB85BA4000                mov ebx,0040BA85   
 004136B3: 03DD                      add ebx,ebp   
 004136B5: 807B0800                  cmp byte ptr \[ebx+08\],00   
 004136B9: 746D                      jz 00413728   
 004136BB: 8BC3                      mov eax,ebx   
 004136BD: 83C008                    add eax,00000008   
 004136C0: 50                        push eax   
 004136C1: FF9564BF4000              call dword ptr \[ebp+0040BF64\]   
 004136C7: 89857DBA4000              mov dword ptr \[ebp+0040BA7D\],eax   
 004136CD: 83F800                    cmp eax,00000000   
 004136D0: 750C                      jnz 004136DE   
 004136D2: C78581BA400000000000      mov dword ptr \[ebp+0040BA81\],00000000   
 004136DC: EB4A                      jmp 00413728   
 004136DE: 8B33                      mov esi,dword ptr \[ebx\]   
 004136E0: 8B7B04                    mov edi,dword ptr \[ebx+04\]   
 004136E3: 03F5                      add esi,ebp   
 004136E5: 03FD                      add edi,ebp   
 004136E7: 803E00                    cmp byte ptr \[esi\],00   
 004136EA: 750E                      jnz 004136FA   
 004136EC: 83C308                    add ebx,00000008   
 004136EF: 803B00                    cmp byte ptr \[ebx\],00   
 004136F2: 7403                      jz 004136F7   
 004136F4: 43                        inc ebx   
 004136F5: EBF8                      jmp 004136EF   
 004136F7: 43                        inc ebx   
 004136F8: EBBB                      jmp 004136B5   
 004136FA: 56                        push esi   
 004136FB: FFB57DBA4000              push dword ptr \[ebp+0040BA7D\]   
 00413701: FF9568BF4000              call dword ptr \[ebp+0040BF68\]   
 00413707: 83F800                    cmp eax,00000000   
 0041370A: 750C                      jnz 00413718   
 0041370C: C78581BA400000000000      mov dword ptr \[ebp+0040BA81\],00000000   
 00413716: EB10                      jmp 00413728   
 00413718: 8907                      mov dword ptr \[edi\],eax   
 0041371A: 83C704                    add edi,00000004   
 0041371D: 803E00                    cmp byte ptr \[esi\],00   
 00413720: 7403                      jz 00413725   
 00413722: 46                        inc esi   
 00413723: EBF8                      jmp 0041371D   
 00413725: 46                        inc esi   
 00413726: EBBF                      jmp 004136E7   
 00413728: 61                        popad    
 00413729: 8B8581BA4000              mov eax,dword ptr \[ebp+0040BA81\]   
 0041372F: C3                        ret   
 00413730: 0000                      add byte ptr \[eax\],al   
 00413732: 0000                      add byte ptr \[eax\],al   
 00413734: 0100                      add dword ptr \[eax\],eax   
 00413736: 0000                      add byte ptr \[eax\],al   
 00413738: E3BA                      jecxz 004136F4   
 0041373A: 40                        inc eax   
 0041373B: 009FBB40006B              add byte ptr \[edi+6B0040BB\],bl   
 00413741: 65726E                    jb 004137B2   
 00413744: 656C                      insb es:\[edi\],dx   
 00413746: 3332                      xor esi,dword ptr \[edx\]   
 00413748: 2E646C                    insb es:\[edi\],dx   
 0041374B: 6C                        insb es:\[edi\],dx   
 0041374C: 00D3                      add bl,dl   
 0041374E: BB400000BC                mov ebx,BC000040   
 00413753: 40                        inc eax   
 00413754: 007573                    add byte ptr \[ebp+73\],dh   
 00413757: 657233                    jb 0041378D   
 0041375A: 322E                      xor ch,byte ptr \[esi\]   
 0041375C: 646C                      insb es:\[edi\],dx   
 0041375E: 6C                        insb es:\[edi\],dx   
 0041375F: 0014BC                    add byte ptr \[esp+edi\*4\],dl   
 00413762: 40                        inc eax   
 00413763: 0072BC                    add byte ptr \[edx-44\],dh   
 00413766: 40                        inc eax   
 00413767: 006164                    add byte ptr \[ecx+64\],ah   
 0041376A: 7661                      jbe 004137CD   
 0041376C: 7069                      jo 004137D7   
 0041376E: 3332                      xor esi,dword ptr \[edx\]   
 00413770: 2E646C                    insb es:\[edi\],dx   
 00413773: 6C                        insb es:\[edi\],dx   
 00413774: 008EBC4000B8              add byte ptr \[esi-47FFBF44\],cl   
 0041377A: BC40006469                mov esp,69640040   
 0041377F: 61                        popad    
 00413780: 6C                        insb es:\[edi\],dx   
 00413781: 2E646C                    insb es:\[edi\],dx   
 00413784: 6C                        insb es:\[edi\],dx   
 00413785: 0000                      add byte ptr \[eax\],al   
 00413787: 0000                      add byte ptr \[eax\],al   
 00413789: 0000                      add byte ptr \[eax\],al   
 0041378B: 0000                      add byte ptr \[eax\],al   
 0041378D: 0000                      add byte ptr \[eax\],al   
 0041378F: 0020                      add byte ptr \[eax\],ah   
 00413791: 2020                      and byte ptr \[eax\],ah   
 00413793: 2020                      and byte ptr \[eax\],ah   
 00413795: 3237                      xor dh,byte ptr \[edi\]   
 00413797: 3133                      xor dword ptr \[ebx\],esi   
 00413799: 3800                      cmp byte ptr \[eax\],al   
 0041379B: 0000                      add byte ptr \[eax\],al   
 0041379D: 0000                      add byte ptr \[eax\],al   
 0041379F: 0000                      add byte ptr \[eax\],al   
 00413780: 6C                        insb es:\[edi\],dx   
 00413781: 2E646C                    insb es:\[edi\],dx   
 00413784: 6C                        insb es:\[edi\],dx   
 00413785: 0000                      add byte ptr \[eax\],al   
 00413787: 0000                      add byte ptr \[eax\],al   
 00413789: 0000                      add byte ptr \[eax\],al   
 0041378B: 0000                      add byte ptr \[eax\],al   
 0041378D: 0000                      add byte ptr \[eax\],al   
 0041378F: 0000                      add byte ptr \[eax\],al   
 00413791: 0000                      add byte ptr \[eax\],al   
 00413793: 0000                      add byte ptr \[eax\],al   
 00413795: 004765                    add byte ptr \[edi+65\],al   
 00413798: 7453                      jz 004137ED   
 0041379A: 7973                      jns 0041380F   
 0041379C: 7465                      jz 00413803   
 0041379E: 6D                        insd es:\[edi\],dx   
 0041379F: 54                        push esp   
 004137A0: 696D6500437265            imul ebp,dword ptr \[ebp+65\],65724300   
 004137A7: 61                        popad    
 004137A8: 7465                      jz 0041380F   
 004137AA: 46                        inc esi   
 004137AB: 696C654100577269          imul ebp,dword ptr \[ebp+41\],69725700   
 004137B3: 7465                      jz 0041381A   
 004137B5: 46                        inc esi   
 004137B6: 696C6500436C6F73          imul ebp,dword ptr \[ebp\],736F6C43   
 004137BE: 6548                      dec eax   
 004137C0: 61                        popad    
 004137C1: 6E                        outsb dx,ds:\[esi\]   
 004137C2: 646C                      insb es:\[edi\],dx   
 004137C4: 65004372                  add byte ptr gs:\[ebx+72\],al   
 004137C8: 6561                      popad    
 004137CA: 7465                      jz 00413831   
 004137CC: 50                        push eax   
 004137CD: 726F                      jb 0041383E   
 004137CF: 636573                    arpl word ptr \[ebp+73\],sp   
 004137D2: 7341                      jnb 00413815   
 004137D4: 004765                    add byte ptr \[edi+65\],al   
 004137D7: 7445                      jz 0041381E   
 004137D9: 7869                      js 00413844   
 004137DB: 7443                      jz 00413820   
 004137DD: 6F                        outsd dx,ds:\[esi\]   
 004137DE: 646550                    push eax   
 004137E1: 726F                      jb 00413852   
 004137E3: 636573                    arpl word ptr \[ebp+73\],sp   
 004137E6: 7300                      jnb 004137E8   
 004137E8: 47                        inc edi   
 004137E9: 657444                    jz 00413830   
 004137EC: 69736B46726565            imul esi,dword ptr \[ebx+6B\],65657246   
 004137F3: 53                        push ebx   
 004137F4: 7061                      jo 00413857   
 004137F6: 636545                    arpl word ptr \[ebp+45\],sp   
 004137F9: 7841                      js 0041383C   
 004137FB: 0044656C                  add byte ptr \[ebp+6C\],al   
 004137FF: 657465                    jz 00413867   
 00413802: 46                        inc esi   
 00413803: 696C654100476574          imul ebp,dword ptr \[ebp+41\],74654700   
 0041380B: 56                        push esi   
 0041380C: 6F                        outsd dx,ds:\[esi\]   
 0041380D: 6C                        insb es:\[edi\],dx   
 0041380E: 756D                      jnz 0041387D   
 00413810: 6549                      dec ecx   
 00413812: 6E                        outsb dx,ds:\[esi\]   
 00413813: 666F                      outsw dx,ds:\[esi\]   
 00413815: 726D                      jb 00413884   
 00413817: 61                        popad    
 00413818: 7469                      jz 00413883   
 0041381A: 6F                        outsd dx,ds:\[esi\]   
 0041381B: 6E                        outsb dx,ds:\[esi\]   
 0041381C: 41                        inc ecx   
 0041381D: 004765                    add byte ptr \[edi+65\],al   
 00413820: 744D                      jz 0041386F   
 00413822: 6F                        outsd dx,ds:\[esi\]   
 00413823: 64756C                    jnz 00413892   
 00413826: 6548                      dec eax   
 00413828: 61                        popad    
 00413829: 6E                        outsb dx,ds:\[esi\]   
 0041382A: 646C                      insb es:\[edi\],dx   
 0041382C: 6541                      inc ecx   
 0041382E: 005265                    add byte ptr \[edx+65\],dl   
 00413831: 61                        popad    
 00413832: 6450                      push eax   
 00413834: 726F                      jb 004138A5   
 00413836: 636573                    arpl word ptr \[ebp+73\],sp   
 00413839: 734D                      jnb 00413888   
 0041383B: 656D                      insd es:\[edi\],dx   
 0041383D: 6F                        outsd dx,ds:\[esi\]   
 0041383E: 7279                      jb 004138B9   
 00413840: 005669                    add byte ptr \[esi+69\],dl   
 00413843: 7274                      jb 004138B9   
 00413845: 7561                      jnz 004138A8   
 00413847: 6C                        insb es:\[edi\],dx   
 00413848: 41                        inc ecx   
 00413849: 6C                        insb es:\[edi\],dx   
 0041384A: 6C                        insb es:\[edi\],dx   
 0041384B: 6F                        outsd dx,ds:\[esi\]   
 0041384C: 6300                      arpl word ptr \[eax\],ax   
 0041384E: 0000                      add byte ptr \[eax\],al   
 00413850: 0000                      add byte ptr \[eax\],al   
 00413852: 0000                      add byte ptr \[eax\],al   
 00413854: 0000                      add byte ptr \[eax\],al   
 00413856: 0000                      add byte ptr \[eax\],al   
 00413858: 0000                      add byte ptr \[eax\],al   
 0041385A: 0000                      add byte ptr \[eax\],al   
 0041385C: 0000                      add byte ptr \[eax\],al   
 0041385E: 0000                      add byte ptr \[eax\],al   
 00413860: 0000                      add byte ptr \[eax\],al   
 00413862: 0000                      add byte ptr \[eax\],al   
 00413864: 0000                      add byte ptr \[eax\],al   
 00413866: 0000                      add byte ptr \[eax\],al   
 00413868: 0000                      add byte ptr \[eax\],al   
 0041386A: 0000                      add byte ptr \[eax\],al   
 0041386C: 0000                      add byte ptr \[eax\],al   
 0041386E: 0000                      add byte ptr \[eax\],al   
 00413870: 0000                      add byte ptr \[eax\],al   
 00413872: 0000                      add byte ptr \[eax\],al   
 00413874: 0000                      add byte ptr \[eax\],al   
 00413876: 0000                      add byte ptr \[eax\],al   
 00413878: 0000                      add byte ptr \[eax\],al   
 0041387A: 0000                      add byte ptr \[eax\],al   
 0041387C: 0000                      add byte ptr \[eax\],al   
 0041387E: 0000                      add byte ptr \[eax\],al   
 00413880: 0000                      add byte ptr \[eax\],al   
 00413882: 0000                      add byte ptr \[eax\],al   
 00413884: 0000                      add byte ptr \[eax\],al   
 00413886: 4D                        dec ebp   
 00413887: 657373                    jnb 004138FD   
 0041388A: 61                        popad    
 0041388B: 676542                    inc edx   
 0041388E: 6F                        outsd dx,ds:\[esi\]   
 0041388F: 7841                      js 004138D2   
 00413891: 007773                    add byte ptr \[edi+73\],dh   
 00413894: 7072                      jo 00413908   
 00413896: 696E7466410053            imul ebp,dword ptr \[esi+74\],53004166   
 0041389D: 657454                    jz 004138F4   
 004138A0: 696D6572004B69            imul ebp,dword ptr \[ebp+65\],694B0072   
 004138A7: 6C                        insb es:\[edi\],dx   
 004138A8: 6C                        insb es:\[edi\],dx   
 004138A9: 54                        push esp   
 004138AA: 696D6572000000            imul ebp,dword ptr \[ebp+65\],00000072   
 004138B1: 0000                      add byte ptr \[eax\],al   
 004138B3: 0000                      add byte ptr \[eax\],al   
 004138B5: 0000                      add byte ptr \[eax\],al   
 004138B7: 0000                      add byte ptr \[eax\],al   
 004138B9: 0000                      add byte ptr \[eax\],al   
 004138BB: 0000                      add byte ptr \[eax\],al   
 004138BD: 0000                      add byte ptr \[eax\],al   
 004138BF: 0000                      add byte ptr \[eax\],al   
 004138C1: 0000                      add byte ptr \[eax\],al   
 004138C3: 0000                      add byte ptr \[eax\],al   
 004138C5: 0000                      add byte ptr \[eax\],al   
 004138C7: 52                        push edx   
 004138C8: 65674F                    dec edi   
 004138CB: 7065                      jo 00413932   
 004138CD: 6E                        outsb dx,ds:\[esi\]   
 004138CE: 4B                        dec ebx   
 004138CF: 657945                    jns 00413917   
 004138D2: 7841                      js 00413915   
 004138D4: 005265                    add byte ptr \[edx+65\],dl   
 004138D7: 6743                      inc ebx   
 004138D9: 7265                      jb 00413940   
 004138DB: 61                        popad    
 004138DC: 7465                      jz 00413943   
 004138DE: 4B                        dec ebx   
 004138DF: 657945                    jns 00413927   
 004138E2: 7841                      js 00413925   
 004138E4: 005265                    add byte ptr \[edx+65\],dl   
 004138E7: 6743                      inc ebx   
 004138E9: 6C                        insb es:\[edi\],dx   
 004138EA: 6F                        outsd dx,ds:\[esi\]   
 004138EB: 7365                      jnb 00413952   
 004138ED: 4B                        dec ebx   
 004138EE: 657900                    jns 004138F1   
 004138F1: 52                        push edx   
 004138F2: 656751                    push ecx   
 004138F5: 7565                      jnz 0041395C   
 004138F7: 7279                      jb 00413972   
 004138F9: 56                        push esi   
 004138FA: 61                        popad    
 004138FB: 6C                        insb es:\[edi\],dx   
 004138FC: 7565                      jnz 00413963   
 004138FE: 45                        inc ebp   
 004138FF: 7841                      js 00413942   
 00413901: 005265                    add byte ptr \[edx+65\],dl   
 00413904: 6753                      push ebx   
 00413906: 657456                    jz 0041395F   
 00413909: 61                        popad    
 0041390A: 6C                        insb es:\[edi\],dx   
 0041390B: 7565                      jnz 00413972   
 0041390D: 45                        inc ebp   
 0041390E: 7841                      js 00413951   
 00413910: 005265                    add byte ptr \[edx+65\],dl   
 00413913: 6744                      inc esp   
 00413915: 656C                      insb es:\[edi\],dx   
 00413917: 657465                    jz 0041397F   
 0041391A: 56                        push esi   
 0041391B: 61                        popad    
 0041391C: 6C                        insb es:\[edi\],dx   
 0041391D: 7565                      jnz 00413984   
 0041391F: 41                        inc ecx   
 00413920: 0000                      add byte ptr \[eax\],al   
 00413922: 0000                      add byte ptr \[eax\],al   
 00413924: 0000                      add byte ptr \[eax\],al   
 00413926: 0000                      add byte ptr \[eax\],al   
 00413928: 0000                      add byte ptr \[eax\],al   
 0041392A: 0000                      add byte ptr \[eax\],al   
 0041392C: 0000                      add byte ptr \[eax\],al   
 0041392E: 0000                      add byte ptr \[eax\],al   
 00413930: 0000                      add byte ptr \[eax\],al   
 00413932: 0000                      add byte ptr \[eax\],al   
 00413934: 0000                      add byte ptr \[eax\],al   
 00413936: 0000                      add byte ptr \[eax\],al   
 00413938: 0000                      add byte ptr \[eax\],al   
 0041393A: 0000                      add byte ptr \[eax\],al   
 0041393C: 0000                      add byte ptr \[eax\],al   
 0041393E: 0000                      add byte ptr \[eax\],al   
 00413940: 004765                    add byte ptr \[edi+65\],al   
 00413943: 7450                      jz 00413995   
 00413945: 61                        popad    
 00413946: 7373                      jnb 004139BB   
 00413948: 776F                      jnbe 004139B9   
 0041394A: 7264                      jb 004139B0   
 0041394C: 004765                    add byte ptr \[edi+65\],al   
 0041394F: 7452                      jz 004139A3   
 00413951: 656769737465720053        imul esi,dword ptr gs:\[bp+di+74\],53007265   
 0041395A: 686F775472                push 7254776F   
 0041395F: 7957                      jns 004139B8   
 00413961: 696E646F770000            imul ebp,dword ptr \[esi+64\],0000776F   
 00413968: 0000                      add byte ptr \[eax\],al   
 0041396A: 0000                      add byte ptr \[eax\],al   
 0041396C: 0000                      add byte ptr \[eax\],al   
 0041396E: 0000                      add byte ptr \[eax\],al   
 00413970: 0000                      add byte ptr \[eax\],al   
 00413972: 0000                      add byte ptr \[eax\],al   
 00413974: 0000                      add byte ptr \[eax\],al   
 00413976: 0000                      add byte ptr \[eax\],al   
 00413978: 0000                      add byte ptr \[eax\],al   
 0041397A: 0000                      add byte ptr \[eax\],al   
 0041397C: 0000                      add byte ptr \[eax\],al   
 0041397E: 0000                      add byte ptr \[eax\],al   
 00413980: 0000                      add byte ptr \[eax\],al   
 00413982: 0000                      add byte ptr \[eax\],al   
 00413984: 000000                    ??   
\-------------------------------------------------------  
  
 ------------------最后结束的代码-------------------------  
 00413987: 60                        pushad    
 00413988: 8BE8                      mov ebp,eax   
 0041398A: 80BD5DBD400000            cmp byte ptr \[ebp+0040BD5D\],00   
 00413991: 7555                      jnz 004139E8          <--是否注册失败，不是跳走  
 00413993: BE63BD4000                mov esi,0040BD63   
 00413998: 03F5                      add esi,ebp            <--413A16  
 0041399A: C606C3                    mov byte ptr \[esi\],C3   
 0041399D: 8B8573BE4000              mov eax,dword ptr \[ebp+0040BE73\]   
 004139A3: 83F800                    cmp eax,00000000   
 004139A6: 7407                      jz 004139AF   
 004139A8: 898577BE4000              mov dword ptr \[ebp+0040BE77\],eax   
 004139AE: CC                        int 3   
 004139AF: BE53BD4000                mov esi,0040BD53   
 004139B4: 03F5                      add esi,ebp   
 004139B6: 0F010E                    sidt dword ptr \[esi\]   
 004139B9: 8B7602                    mov esi,dword ptr \[esi+02\]   
 004139BC: 668B8567BE4000            mov ax,word ptr \[ebp+0040BE67\]   
 004139C3: 668B9D69BE4000            mov bx,word ptr \[ebp+0040BE69\]   
 004139CA: 66894608                  mov word ptr \[esi+08\],ax   
 004139CE: 66895E0E                  mov word ptr \[esi+0E\],bx   
 004139D2: 668B856BBE4000            mov ax,word ptr \[ebp+0040BE6B\]   
 004139D9: 668B9D6DBE4000            mov bx,word ptr \[ebp+0040BE6D\]   
 004139E0: 66894618                  mov word ptr \[esi+18\],ax   
 004139E4: 66895E1E                  mov word ptr \[esi+1E\],bx   
 004139E8: 61                        popad    
 004139E9: 56                        push esi   
 004139EA: 51                        push ecx   
 004139EB: BE4D534000                mov esi,0040534D   
 004139F0: 03F0                      add esi,eax   
 004139F2: B9F7690000                mov ecx,000069F7   
  
 004139F7: C60600                    mov byte ptr \[esi\],00   
 004139FA: 46                        inc esi   
 004139FB: E2FA                      loop 004139F7   
  
 004139FD: 59                        pop ecx   
 004139FE: 5E                        pop esi   
 004139FF: 58                        pop eax   
 00413A00: FFE0                      jmp eax   
\------------------------------------------------  
希望大家多到此地捧捧场。