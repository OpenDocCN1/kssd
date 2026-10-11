
# 家庭财务精灵 1.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6075.htm`](https://www.pediy.com/kssd/pediy06/pediy6075.htm)

【软件名称】`家庭财务精灵 1.0`

【下载地址】`http://www.softfairy.com/`

【应用平台】`Win9x`, `WinME`, `WinNT`, `Win2000`, `WinXP`, `Linux/Unix`

【软件大小】`4.23M`

【软件限制】`时间限制`

【破解声明】破解只是感兴趣，无其它目的。失误之处敬请诸位大侠赐教!

【破解工具】`Pescan3.31`, `OllyDbg1.09`, `Wdasm8.93`

【软件简介】我们中国人大多都有记帐的习惯，以记录日常的开支，明确资金的去向，进而做到开源节流。但是在当今这个快节奏的社会里，对于缺少会计专业知识的非专业人士来说，传统的手工记帐方式费时劳神，查询往日的帐目也很困难，更不用说进行财务分析了。对于在时间和金钱上都精打细算的您来说，这款小巧玲珑而功能完备的家庭财务软件正是您最佳的选择。

家庭事件记录功能；收入/支出数据录入及查询功能,图表分析功能,系统运行监控功能(能够监控系统的运行时间),密码保护功能，部分报表打印功能，每月收入支出趋势图分析功能.

家庭财务精灵软件的八大特点：1.该软件适合中国广大家庭使用，使用简单，容易上手。2.包括多种图表分析，一目了然。3.有资金预算功能，便于控制资金的支出，使资金的运用具有计划性。3.具有更多的自定义选项，更适合用户的需求。4.数据备份方便简单，安全性强。5.系统升级简单，升级周期短。6.不需要更多的外挂程序，软件小，下载方便。7.注册简单、方便，备有多种注册方式。8.帮助功能强大，具有再线帮助功能。

========================================================================================

【分析过程】

用 `Pescan` 检查,无壳.反汇编,查找字串,很快就找到关键点,用 `OD` 载入!

任意填入序列号 `12345678` 和注册码 `13572468`。

----------------------------------------

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00643611(U)
|
:0064362F 8B55FC                mov edx, dword ptr [ebp-04]
:00643632 B8CC366400            mov eax, 006436CC
:00643637 E8E817DCFF            call 00404E24
:0064363C 85C0                  test eax, eax
:0064363E 7FD3                  jg 00643613
:00643640 FF3584D36400          push dword ptr [0064D384]
:00643646 FF3580D36400          push dword ptr [0064D380]
:0064364C 8D4DF4                lea ecx, dword ptr [ebp-0C]
:0064364F B870D36400            mov eax, 0064D370
:00643654 8BD3                  mov edx, ebx      //序列号: 0x00bc614e
:00643656 E82D51F8FF            call 005C8788     //关键 call 进入==> 1
:0064365B 8D55F0                lea edx, dword ptr [ebp-10]
:0064365E 8B45FC                mov eax, dword ptr [ebp-04]
:00643661 E87A5DDCFF            call 004093E0
:00643666 8B45F0                mov eax, dword ptr [ebp-10]  //伪码 13572468
:00643669 50                    push eax
:0064366A 8D4DE8                lea ecx, dword ptr [ebp-18]
:0064366D 8D45F4                lea eax, dword ptr [ebp-0C]
:00643670 BA08000000            mov edx, 00000008
:00643675 E87240F8FF            call 005C76EC      //次 call 将激活码转换位 ASCII 码形式
:0064367A 8B45E8                mov eax, dword ptr [ebp-18]
:0064367D 8D55EC                lea edx, dword ptr [ebp-14]
:00643680 E85B5DDCFF            call 004093E0      
:00643685 8B55EC                mov edx, dword ptr [ebp-14]  //激活码
:00643688 58                    pop eax                  //激活码
:00643689 E89E15DCFF            call 00404C2C
:0064368E 0F94C3                sete bl
:00643691 33C0                  xor eax, eax
:00643693 5A                    pop edx
:00643694 59                    pop ecx
:00643695 59                    pop ecx
:00643696 648910                mov dword ptr fs:[eax], edx
:00643699 68BB366400            push 006436BB
------------------------------------call 1 ----------------------------------------------

* Referenced by a CALL at Address:
|:00643656   
|
:005C8788 55                    push ebp
:005C8789 8BEC                  mov ebp, esp
:005C878B 53                    push ebx
:005C878C 56                    push esi
:005C878D 57                    push edi
:005C878E 8BD9                  mov ebx, ecx
:005C8790 8BFA                  mov edi, edx
:005C8792 8BF0                  mov esi, eax
:005C8794 66C703693C            mov word ptr [ebx], 3C69
:005C8799 FF750C                push [ebp+0C]
:005C879C FF7508                push [ebp+08]
:005C879F E8B4FeFFFF            call 005C8658
:005C87A4 66894302              mov word ptr [ebx+02], ax
:005C87A8 897B04                mov dword ptr [ebx+04], edi
:005C87AB 8BD3                  mov edx, ebx   //序列号: 0x00bc614e
:005C87AD 8BC6                  mov eax, esi
:005C87AF B101                  mov cl, 01
:005C87B1 E8AEF7FFFF            call 005C7F64   //算法 call 进入==> 2
:005C87B6 5F                    pop edi
:005C87B7 5E                    pop esi
:005C87B8 5B                    pop ebx
:005C87B9 5D                    pop ebp
:005C87BA C20800                ret 0008
-------------------------------------call 2 ---------------------------------------------

* Referenced by a CALL at Addresses:
|:005C86C3   , :005C870E   , :005C87B1   
|
:005C7F64 53                    push ebx
:005C7F65 56                    push esi
:005C7F66 57                    push edi
:005C7F67 83C4E8                add esp, FFFFFFE8
:005C7F6A 884C2408              mov byte ptr [esp+08], cl 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005C7F07(C)
|
:005C7F6E 89542404              mov dword ptr [esp+04], edx
:005C7F72 890424                mov dword ptr [esp], eax
:005C7F75 8B442404              mov eax, dword ptr [esp+04]
:005C7F79 8B00                  mov eax, dword ptr [eax]
:005C7F7B 8944240C              mov dword ptr [esp+0C], eax
:005C7F7F 8B442404              mov eax, dword ptr [esp+04] 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005C7F1D(C)
|
:005C7F83 8B4004                mov eax, dword ptr [eax+04]
:005C7F86 89442410              mov dword ptr [esp+10], eax
:005C7F8A C744241404000000      mov [esp+14], 00000004  //[esp+14]=00000004
:005C7F92 BE1C946400            mov esi, 0064941C 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005C8030(C)
|
:005C7F97 8B54240C              mov edx, dword ptr [esp+0C] //edx=0x00003c69
:005C7F9B 33C0                  xor eax, eax
:005C7F9D 8A442408              mov al, byte ptr [esp+08]
:005C7FA1 8BD8                  mov ebx, eax
:005C7FA3 03DB                  add ebx, ebx
:005C7FA5 8D1C5B                lea ebx, dword ptr [ebx+2*ebx]
:005C7FA8 8B04DE                mov eax, dword ptr [esi+8*ebx]
:005C7FAB 8B0C24                mov ecx, dword ptr [esp]
:005C7FAE 8B0C81                mov ecx, dword ptr [ecx+4*eax]  //ecx=0x64d370
:005C7FB1 8B44DE04              mov eax, dword ptr [esi+8*ebx+04]
:005C7FB5 8B3C24                mov edi, dword ptr [esp]
:005C7FB8 8B0487                mov eax, dword ptr [edi+4*eax]  //eax=0xd92da051
:005C7FBB 8B5CDE08              mov ebx, dword ptr [esi+8*ebx+08]
:005C7FBF 8B3C24                mov edi, dword ptr [esp]     
:005C7FC2 8B1C9F                mov ebx, dword ptr [edi+4*ebx] //ebx=0xb1a48361
----下面的 eax ebx ecx 每次循环前是变化的,但是都是固定的参数,不随序列号变化-------

第 1 次循环前 eax=0xd92da051  ebx=0xb1a48361   ecx=0x0064d370  edx=0x00003c69
第 2 次循环前 eax=0xb1a48361  ebx=0xd92da051   ecx=0x91057ebe  edx=循环后的 eax
第 3 次循环前 eax=0x91057ebe  ebx=0x9f638ce6   ecx=0xd92da051  edx=循环后的 eax
第 4 次循环前 eax=0x9f638ce6  ebx=0x91057ebe   ecx=0xb1a48361  edx=循环后的 eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005C7F5E(C)
|--------------------------------call 2 算法开始----------------------------------
:005C7FC5 03D3                  add edx, ebx    //edx=edx+ebx
:005C7FC7 03DA                  add ebx, edx    //ebx=ebx+edx
:005C7FC9 8BFA                  mov edi, edx    //edi=edx
:005C7FCB C1EF07                shr edi, 07     //edi=edi 逻辑右移 0x7
:005C7FCE 33D7                  xor edx, edi    //edx=edx xor edi
:005C7FD0 03CA                  add ecx, edx    //ecx=ecx+edx
:005C7FD2 03D1                  add edx, ecx    //edx=edx+ecx
```
