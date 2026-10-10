

# 极光多能闹钟 3.2　　

> 原文：[`www.pediy.com/kssd/pediy05/pediy50619.htm`](https://www.pediy.com/kssd/pediy05/pediy50619.htm)

简单分析——极光多能闹钟 3.2  
  
  
  
下载页面：  [`www.skycn.com/soft/10136.html`](http://www.skycn.com/soft/10136.html)  
软件大小：  1980 KB  
软件语言：  简体中文  
软件类别：  国产软件 / 共享版 / 开关定时  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2003-03-19 09:00:44  
下载次数：  13146  
推荐等级：  \*\*\*  
开 发 商：  [`jgsoft.wx-e.com/`](http://jgsoft.wx-e.com/)

  
【软件简介】：支持电脑自带喇叭响铃和电脑音乐，并可自谱喇叭音乐，享受 DIY 的乐趣。可以按具体日期、每日、每周、每月、农历生日等不同方式提醒，灵活的提醒方式几乎可满足您所有的提醒需要。漂亮的石英钟界面和直观的液晶数字界面随时切换。使用中您会发现这是一款为您定制的程序，面板上几乎所有的东西都可依您所好来改变，石英钟大小可多级缩放，时钟的粗细，颜色都可改变，并支持自选图片。软件提供重要日期提醒、每小时固定响铃和使用方便的临时闹钟，支持正点报时和间隔报时，定时关机，定时执行指定程序，内置极光拖存工具，方便保存网上文摘。同时还附有一个原子钟对时工具和万年历。软件有非常方便的在线升级功能，升级前自动预读最新版本的更新内容，只下载需要更新的文件。未注册用户在两个月的试用期内没有任何功能的限制，也不会弹出窗口来干扰您。

【软件限制】：试用两个月

【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：TRW2000 娃娃修改版、Ollydbg1.09、PEiD、WKT\_tElock\_Dumper 1.2、W32Dasm 9.0 白金版

—————————————————————————————————   
【过    程】：

  
首先必须说明的是我现在分析的是从《软件王》光盘里找的 V3.2 的旧版本， 而“天空”上面的现在是 V4.4 的新版本，可能有些地方是不一样的。现在软件升级的速度都挺快，比我的破解水平提升的快多了。 ^\_^

呵呵，又碰上这个 VB 的东东，实在是不大乐意跟踪 VB 程序的算法，但是也不能见 VB 就“逃”呀，^O^^O^ 静下心来我跟踪了好几遍，哎，现在是两眼都冒着 VB 的“金花”呀，更惨的是只找到了算法的地方而没有能力完整的总结出来，惭愧！这个东东太麻烦了，很多迷惑视线的“冗余代码”、假比较，教我如何不难过呀！

JgClockXP.exe 是 tElock 0.98 壳，用 WKT\_tElock\_Dumper 1.2 脱之。196K->676K。

机器码：5554908250  
试炼码：1357246890    需要 10 位注册码！  
—————————————————————————————————   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B333(C)  
|  
:0048B3ED 8B45BC                  mov eax, dword ptr \[ebp\-44\]  
                                 ====>EAX\=1357246890

:0048B3F0 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaLenBstr, Ord:0000h  
                                 |  
:0048B3F1 FF1538104000            Call dword ptr \[00401038\]  
                                 ====>取试炼码长度         EAX\=A

:0048B3F7 BE0A000000              mov esi, 0000000A  
                                 ====>ESI\=A

:0048B3FC 3BC6                    cmp eax, esi  
                                 ====>是否是 10 位注册码？注册码需要 10 位！

:0048B3FE 0F8460010000            je 0048B564  
                                 ====>不跳则 OVER！

:0048B404 B804000280              mov eax, 80020004  
:0048B409 898558FFFFFF            mov dword ptr \[ebp+FFFFFF58\], eax  
:0048B40F 89B550FFFFFF            mov dword ptr \[ebp+FFFFFF50\], esi  
:0048B415 898568FFFFFF            mov dword ptr \[ebp+FFFFFF68\], eax  
:0048B41B 89B560FFFFFF            mov dword ptr \[ebp+FFFFFF60\], esi

\* Possible StringData Ref from Data Obj ->"衏:y"  
                                 |  
:0048B421 C78548FFFFFFE81B4100    mov dword ptr \[ebp+FFFFFF48\], 00411BE8  
:0048B42B C78540FFFFFF08000000    mov dword ptr \[ebp+FFFFFF40\], 00000008  
:0048B435 8D9540FFFFFF            lea edx, dword ptr \[ebp+FFFFFF40\]  
:0048B43B 8D8D70FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF70\]

\* Reference To: MSVBVM60.\_\_vbaVarDup, Ord:0000h  
                                 |  
:0048B441 FF156C124000            Call dword ptr \[0040126C\]

\* Possible StringData Ref from Data Obj ->"鲖蛻癳/T≧,gz 廭寶翄鑜孮酧 o\`"  
                                 |  
:0048B447 68F41B4100              push 00411BF4

\* Possible StringData Ref from Data Obj ->"  
"  
                                 |  
:0048B44C 684CF64000              push 0040F64C

\* Reference To: MSVBVM60.\_\_vbaStrCat, Ord:0000h  
                                 |  
:0048B451 8B3574104000            mov esi, dword ptr \[00401074\]  
:0048B457 FFD6                    call esi  
:0048B459 8BD0                    mov edx, eax  
:0048B45B 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]  
:0048B45E FFD7                    call edi  
:0048B460 50                      push eax

\* Possible StringData Ref from Data Obj ->"  
"  
                                 |  
:0048B461 684CF64000              push 0040F64C  
:0048B466 FFD6                    call esi  
:0048B468 8BD0                    mov edx, eax  
:0048B46A 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B46D FFD7                    call edi  
:0048B46F 50                      push eax

\* Possible StringData Ref from Data Obj ->"鍌  
z 廭-Ng 鑜孮 P\[軆 USR 鑜孮 b 烺"  
                                 |  
:0048B470 68181C4100              push 00411C18  
:0048B475 FFD6                    call esi  
:0048B477 894588                  mov dword ptr \[ebp\-78\], eax  
:0048B47A C7458008000000          mov \[ebp\-80\], 00000008  
:0048B481 8D8D50FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF50\]  
:0048B487 51                      push ecx  
:0048B488 8D9560FFFFFF            lea edx, dword ptr \[ebp+FFFFFF60\]  
:0048B48E 52                      push edx  
:0048B48F 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]  
:0048B495 50                      push eax  
:0048B496 6A40                    push 00000040  
:0048B498 8D4D80                  lea ecx, dword ptr \[ebp\-80\]  
:0048B49B 51                      push ecx

\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h  
                                 |  
:0048B49C FF15CC104000            Call dword ptr \[004010CC\]  
                                 ====>提示重启程序验证注册码！到这就 OVER 了！

:0048B4A2 8BC8                    mov ecx, eax

\* Reference To: MSVBVM60.\_\_vbaI2I4, Ord:0000h  
                                 |  
:0048B4A4 FF1550114000            Call dword ptr \[00401150\]  
:0048B4AA 8D55A8                  lea edx, dword ptr \[ebp\-58\]  
:0048B4AD 52                      push edx  
:0048B4AE 8D45AC                  lea eax, dword ptr \[ebp\-54\]  
:0048B4B1 50                      push eax  
:0048B4B2 6A02                    push 00000002

\* Reference To: MSVBVM60.\_\_vbaFreeStrList, Ord:0000h  
                                 |  
:0048B4B4 FF152C124000            Call dword ptr \[0040122C\]  
:0048B4BA 8D8D50FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF50\]  
:0048B4C0 51                      push ecx  
:0048B4C1 8D9560FFFFFF            lea edx, dword ptr \[ebp+FFFFFF60\]  
:0048B4C7 52                      push edx  
:0048B4C8 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]  
:0048B4CE 50                      push eax  
:0048B4CF 8D4D80                  lea ecx, dword ptr \[ebp\-80\]  
:0048B4D2 51                      push ecx  
:0048B4D3 6A04                    push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0048B4D5 FF1540104000            Call dword ptr \[00401040\]  
:0048B4DB 83C420                  add esp, 00000020  
:0048B4DE 8B13                    mov edx, dword ptr \[ebx\]  
:0048B4E0 53                      push ebx  
:0048B4E1 FF9200070000            call dword ptr \[edx+00000700\]  
:0048B4E7 85C0                    test eax, eax  
:0048B4E9 7D12                    jge 0048B4FD  
:0048B4EB 6800070000              push 00000700  
:0048B4F0 68C4074100              push 004107C4  
:0048B4F5 53                      push ebx  
:0048B4F6 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B4F7 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B4E9(C)  
|  
:0048B4FD A1D0FD4900              mov eax, dword ptr \[0049FDD0\]  
:0048B502 85C0                    test eax, eax  
:0048B504 7510                    jne 0048B516  
:0048B506 68D0FD4900              push 0049FDD0  
:0048B50B 68B4F54000              push 0040F5B4

\* Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h  
                                 |  
:0048B510 FF1514124000            Call dword ptr \[00401214\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B504(C)  
|  
:0048B516 A1D0FD4900              mov eax, dword ptr \[0049FDD0\]  
:0048B51B 8985E4FEFFFF            mov dword ptr \[ebp+FFFFFEE4\], eax  
:0048B521 8B30                    mov esi, dword ptr \[eax\]  
:0048B523 53                      push ebx  
:0048B524 8D459C                  lea eax, dword ptr \[ebp\-64\]  
:0048B527 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaObjSetAddref, Ord:0000h  
                                 |  
:0048B528 FF15DC104000            Call dword ptr \[004010DC\]  
:0048B52E 50                      push eax  
:0048B52F 8BCE                    mov ecx, esi  
:0048B531 8BB5E4FEFFFF            mov esi, dword ptr \[ebp+FFFFFEE4\]  
:0048B537 56                      push esi  
:0048B538 FF5110                  call \[ecx+10\]  
:0048B53B DBE2                    fclex  
:0048B53D 85C0                    test eax, eax  
:0048B53F 7D0F                    jge 0048B550  
:0048B541 6A10                    push 00000010  
:0048B543 68A4F54000              push 0040F5A4  
:0048B548 56                      push esi  
:0048B549 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B54A FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B53F(C)  
|  
:0048B550 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]

\* Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h  
                                 |  
:0048B553 FF15EC124000            Call dword ptr \[004012EC\]

\* Reference To: MSVBVM60.\_\_vbaEnd, Ord:0000h  
                                 |  
:0048B559 FF1544104000            Call dword ptr \[00401044\]  
:0048B55F BE0A000000              mov esi, 0000000A

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B3FE(C)  
|  
:0048B564 8B55C0                  mov edx, dword ptr \[ebp\-40\]  
:0048B567 52                      push edx

\* Possible StringData Ref from Data Obj ->"SSNHHQXTDCOM"  
                                 |  
:0048B568 68441C4100              push 00411C44

\* Reference To: MSVBVM60.\_\_vbaStrCmp, Ord:0000h  
                                 |  
:0048B56D FF1534114000            Call dword ptr \[00401134\]  
:0048B573 85C0                    test eax, eax  
:0048B575 0F8591000000            jne 0048B60C  
                                 ====>跳过去

:0048B57B B804000280              mov eax, 80020004  
:0048B580 898558FFFFFF            mov dword ptr \[ebp+FFFFFF58\], eax  
:0048B586 89B550FFFFFF            mov dword ptr \[ebp+FFFFFF50\], esi  
:0048B58C 898568FFFFFF            mov dword ptr \[ebp+FFFFFF68\], eax  
:0048B592 89B560FFFFFF            mov dword ptr \[ebp+FFFFFF60\], esi  
:0048B598 898578FFFFFF            mov dword ptr \[ebp+FFFFFF78\], eax  
:0048B59E 89B570FFFFFF            mov dword ptr \[ebp+FFFFFF70\], esi

\* Possible StringData Ref from Data Obj ->"鑜孮 b 烺�"  
                                 |  
:0048B5A4 C78548FFFFFF601C4100    mov dword ptr \[ebp+FFFFFF48\], 00411C60  
:0048B5AE C78540FFFFFF08000000    mov dword ptr \[ebp+FFFFFF40\], 00000008  
:0048B5B8 8D9540FFFFFF            lea edx, dword ptr \[ebp+FFFFFF40\]  
:0048B5BE 8D4D80                  lea ecx, dword ptr \[ebp\-80\]

\* Reference To: MSVBVM60.\_\_vbaVarDup, Ord:0000h  
                                 |  
:0048B5C1 FF156C124000            Call dword ptr \[0040126C\]  
:0048B5C7 8D8550FFFFFF            lea eax, dword ptr \[ebp+FFFFFF50\]  
:0048B5CD 50                      push eax  
:0048B5CE 8D8D60FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF60\]  
:0048B5D4 51                      push ecx  
:0048B5D5 8D9570FFFFFF            lea edx, dword ptr \[ebp+FFFFFF70\]  
:0048B5DB 52                      push edx  
:0048B5DC 6A00                    push 00000000  
:0048B5DE 8D4580                  lea eax, dword ptr \[ebp\-80\]  
:0048B5E1 50                      push eax

\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h  
                                 |  
:0048B5E2 FF15CC104000            Call dword ptr \[004010CC\]  
:0048B5E8 8D8D50FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF50\]  
:0048B5EE 51                      push ecx  
:0048B5EF 8D9560FFFFFF            lea edx, dword ptr \[ebp+FFFFFF60\]  
:0048B5F5 52                      push edx  
:0048B5F6 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]  
:0048B5FC 50                      push eax  
:0048B5FD 8D4D80                  lea ecx, dword ptr \[ebp\-80\]  
:0048B600 51                      push ecx  
:0048B601 6A04                    push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0048B603 FF1540104000            Call dword ptr \[00401040\]  
:0048B609 83C414                  add esp, 00000014

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B575(C)  
|  
:0048B60C 8D7338                  lea esi, dword ptr \[ebx+38\]

\* Possible StringData Ref from Data Obj ->""  
                                 |  
:0048B60F 68085C4000              push 00405C08

\* Reference To: MSVBVM60.\_\_vbaNew, Ord:0000h  
                                 |  
:0048B614 FF1584114000            Call dword ptr \[00401184\]  
:0048B61A 50                      push eax  
:0048B61B 8D559C                  lea edx, dword ptr \[ebp\-64\]  
:0048B61E 52                      push edx

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B61F FF15D0104000            Call dword ptr \[004010D0\]  
:0048B625 50                      push eax  
:0048B626 56                      push esi

\* Reference To: MSVBVM60.\_\_vbaObjSetAddref, Ord:0000h  
                                 |  
:0048B627 FF15DC104000            Call dword ptr \[004010DC\]  
:0048B62D 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]

\* Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h  
                                 |  
:0048B630 FF15EC124000            Call dword ptr \[004012EC\]  
:0048B636 A1CCF24900              mov eax, dword ptr \[0049F2CC\]  
:0048B63B 85C0                    test eax, eax  
:0048B63D 7515                    jne 0048B654  
:0048B63F 68CCF24900              push 0049F2CC

\* Possible StringData Ref from Data Obj ->""  
                                 |  
:0048B644 68046B4000              push 00406B04

\* Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h  
                                 |  
:0048B649 FF1514124000            Call dword ptr \[00401214\]  
:0048B64F A1CCF24900              mov eax, dword ptr \[0049F2CC\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B63D(C)  
|  
:0048B654 8B08                    mov ecx, dword ptr \[eax\]  
:0048B656 50                      push eax  
:0048B657 FF910C030000            call dword ptr \[ecx+0000030C\]  
:0048B65D 50                      push eax  
:0048B65E 8D559C                  lea edx, dword ptr \[ebp\-64\]  
:0048B661 52                      push edx

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B662 FF15D0104000            Call dword ptr \[004010D0\]  
:0048B668 8BF0                    mov esi, eax  
:0048B66A 8B06                    mov eax, dword ptr \[esi\]  
:0048B66C 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]  
:0048B66F 51                      push ecx  
:0048B670 56                      push esi  
:0048B671 FF5050                  call \[eax+50\]  
:0048B674 DBE2                    fclex  
:0048B676 85C0                    test eax, eax  
:0048B678 7D0F                    jge 0048B689  
:0048B67A 6A50                    push 00000050

\* Possible StringData Ref from Data Obj ->"貼?檉??"  
                                 |  
:0048B67C 6878F54000              push 0040F578  
:0048B681 56                      push esi  
:0048B682 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B683 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B678(C)  
|  
:0048B689 8B55AC                  mov edx, dword ptr \[ebp\-54\]  
                                 ====>EDX\=girlboy       呵呵，“女孩男孩”可是重要参数呀！

:0048B68C C745AC00000000          mov \[ebp\-54\], 00000000  
:0048B693 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B696 FFD7                    call edi  
:0048B698 8B4338                  mov eax, dword ptr \[ebx+38\]  
:0048B69B 8B10                    mov edx, dword ptr \[eax\]  
:0048B69D 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B6A0 51                      push ecx  
:0048B6A1 50                      push eax  
:0048B6A2 FF521C                  call \[edx+1C\]  
:0048B6A5 DBE2                    fclex  
:0048B6A7 85C0                    test eax, eax  
:0048B6A9 7D12                    jge 0048B6BD  
:0048B6AB 6A1C                    push 0000001C

\* Possible StringData Ref from Data Obj ->"趠皕 LH?g0 瓬 KRKeyString"  
                                 |  
:0048B6AD 687C1C4100              push 00411C7C  
:0048B6B2 8B5338                  mov edx, dword ptr \[ebx+38\]  
:0048B6B5 52                      push edx  
:0048B6B6 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B6B7 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B6A9(C)  
|  
:0048B6BD 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]

\* Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h  
                                 |  
:0048B6C0 FF15E8124000            Call dword ptr \[004012E8\]  
:0048B6C6 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]

\* Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h  
                                 |  
:0048B6C9 FF15EC124000            Call dword ptr \[004012EC\]  
:0048B6CF 8B03                    mov eax, dword ptr \[ebx\]  
:0048B6D1 53                      push ebx  
:0048B6D2 FF9014030000            call dword ptr \[eax+00000314\]  
:0048B6D8 50                      push eax  
:0048B6D9 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]  
:0048B6DC 51                      push ecx

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B6DD FF15D0104000            Call dword ptr \[004010D0\]  
:0048B6E3 8BF0                    mov esi, eax  
:0048B6E5 8B16                    mov edx, dword ptr \[esi\]  
:0048B6E7 8D45A4                  lea eax, dword ptr \[ebp\-5C\]  
:0048B6EA 50                      push eax  
:0048B6EB 56                      push esi  
:0048B6EC FF92A0000000            call dword ptr \[edx+000000A0\]  
:0048B6F2 DBE2                    fclex  
:0048B6F4 85C0                    test eax, eax  
:0048B6F6 7D12                    jge 0048B70A  
:0048B6F8 68A0000000              push 000000A0

\* Possible StringData Ref from Data Obj ->"酦?檉??"  
                                 |  
:0048B6FD 6898044100              push 00410498  
:0048B702 56                      push esi  
:0048B703 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B704 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B6F6(C)  
|

\* Possible StringData Ref from Data Obj ->"JJgClockV32SnCode"  
                                 |  
:0048B70A BA381D4100              mov edx, 00411D38  
:0048B70F 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]

\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h  
                                 |  
:0048B712 FF1524124000            Call dword ptr \[00401224\]  
:0048B718 8B4338                  mov eax, dword ptr \[ebx+38\]  
:0048B71B 8B08                    mov ecx, dword ptr \[eax\]  
:0048B71D 8D55A8                  lea edx, dword ptr \[ebp\-58\]  
:0048B720 52                      push edx  
:0048B721 8D55AC                  lea edx, dword ptr \[ebp\-54\]  
:0048B724 52                      push edx  
:0048B725 50                      push eax  
:0048B726 FF5120                  call \[ecx+20\]  
:0048B729 DBE2                    fclex  
:0048B72B 85C0                    test eax, eax  
:0048B72D 7D12                    jge 0048B741  
:0048B72F 6A20                    push 00000020

\* Possible StringData Ref from Data Obj ->"趠皕 LH?g0 瓬 KRKeyString"  
                                 |  
:0048B731 687C1C4100              push 00411C7C  
:0048B736 8B4B38                  mov ecx, dword ptr \[ebx+38\]  
:0048B739 51                      push ecx  
:0048B73A 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B73B FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B72D(C)  
|  
:0048B741 8B55A4                  mov edx, dword ptr \[ebp\-5C\]  
:0048B744 52                      push edx  
:0048B745 8B45A8                  mov eax, dword ptr \[ebp\-58\]  
:0048B748 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaStrCmp, Ord:0000h  
                                 |  
:0048B749 FF1534114000            Call dword ptr \[00401134\]  
:0048B74F 8BF0                    mov esi, eax  
:0048B751 F7DE                    neg esi  
:0048B753 1BF6                    sbb esi, esi  
:0048B755 46                      inc esi  
:0048B756 F7DE                    neg esi  
:0048B758 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B75B 51                      push ecx  
:0048B75C 8D55A4                  lea edx, dword ptr \[ebp\-5C\]  
:0048B75F 52                      push edx  
:0048B760 8D45AC                  lea eax, dword ptr \[ebp\-54\]  
:0048B763 50                      push eax  
:0048B764 6A03                    push 00000003

\* Reference To: MSVBVM60.\_\_vbaFreeStrList, Ord:0000h  
                                 |  
:0048B766 FF152C124000            Call dword ptr \[0040122C\]  
:0048B76C 83C410                  add esp, 00000010  
:0048B76F 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]

\* Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h  
                                 |  
:0048B772 FF15EC124000            Call dword ptr \[004012EC\]  
:0048B778 6685F6                  test si, si  
:0048B77B 7440                    je 0048B7BD  
                                 ====>跳下去！

:0048B77D A1B8F24900              mov eax, dword ptr \[0049F2B8\]  
:0048B782 85C0                    test eax, eax  
:0048B784 7510                    jne 0048B796  
:0048B786 68B8F24900              push 0049F2B8

\* Possible StringData Ref from Data Obj ->""  
                                 |  
:0048B78B 680CC14000              push 0040C10C

\* Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h  
                                 |  
:0048B790 FF1514124000            Call dword ptr \[00401214\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B784(C)  
|  
:0048B796 8B35B8F24900            mov esi, dword ptr \[0049F2B8\]  
:0048B79C 8B0E                    mov ecx, dword ptr \[esi\]  
:0048B79E 56                      push esi  
:0048B79F FF9104070000            call dword ptr \[ecx+00000704\]  
:0048B7A5 DBE2                    fclex  
:0048B7A7 85C0                    test eax, eax  
:0048B7A9 7D12                    jge 0048B7BD  
:0048B7AB 6804070000              push 00000704  
:0048B7B0 68E4E74000              push 0040E7E4  
:0048B7B5 56                      push esi  
:0048B7B6 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B7B7 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0048B77B(C), :0048B7A9(C)  
|  
:0048B7BD 8B13                    mov edx, dword ptr \[ebx\]  
:0048B7BF 53                      push ebx  
:0048B7C0 FF9234030000            call dword ptr \[edx+00000334\]  
:0048B7C6 50                      push eax  
:0048B7C7 8D459C                  lea eax, dword ptr \[ebp\-64\]  
:0048B7CA 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B7CB FF15D0104000            Call dword ptr \[004010D0\]  
:0048B7D1 8BF0                    mov esi, eax  
:0048B7D3 8B0E                    mov ecx, dword ptr \[esi\]  
:0048B7D5 8D55AC                  lea edx, dword ptr \[ebp\-54\]  
:0048B7D8 52                      push edx  
:0048B7D9 56                      push esi  
:0048B7DA FF5150                  call \[ecx+50\]  
:0048B7DD DBE2                    fclex  
:0048B7DF 85C0                    test eax, eax  
:0048B7E1 7D0F                    jge 0048B7F2  
:0048B7E3 6A50                    push 00000050

\* Possible StringData Ref from Data Obj ->"貼?檉??"  
                                 |  
:0048B7E5 6878F54000              push 0040F578  
:0048B7EA 56                      push esi  
:0048B7EB 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B7EC FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B7E1(C)  
|  
:0048B7F2 8B45AC                  mov eax, dword ptr \[ebp\-54\]  
                                 ====>EAX\=121

:0048B7F5 50                      push eax

\* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h  
                                 |  
:0048B7F6 FF15F0124000            Call dword ptr \[004012F0\]  
:0048B7FC DD9D00FFFFFF            fstp qword ptr \[ebp+FFFFFF00\]  
                                 ====>ST\=121.00000000000000000

:0048B802 8B0B                    mov ecx, dword ptr \[ebx\]  
:0048B804 53                      push ebx  
:0048B805 FF9130030000            call dword ptr \[ecx+00000330\]  
:0048B80B 50                      push eax  
:0048B80C 8D5598                  lea edx, dword ptr \[ebp\-68\]  
:0048B80F 52                      push edx

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B810 FF15D0104000            Call dword ptr \[004010D0\]  
:0048B816 8BF0                    mov esi, eax  
:0048B818 8B06                    mov eax, dword ptr \[esi\]  
:0048B81A 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B81D 51                      push ecx  
:0048B81E 56                      push esi  
:0048B81F FF5050                  call \[eax+50\]  
:0048B822 DBE2                    fclex  
:0048B824 85C0                    test eax, eax  
:0048B826 7D0F                    jge 0048B837  
:0048B828 6A50                    push 00000050

\* Possible StringData Ref from Data Obj ->"貼?檉??"  
                                 |  
:0048B82A 6878F54000              push 0040F578  
:0048B82F 56                      push esi  
:0048B830 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B831 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B826(C)  
|  
:0048B837 8B55A8                  mov edx, dword ptr \[ebp\-58\]  
                                 ====>EDX\=83

:0048B83A 52                      push edx

\* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h  
                                 |  
:0048B83B FF15F0124000            Call dword ptr \[004012F0\]  
:0048B841 DD9DF8FEFFFF            fstp qword ptr \[ebp+FFFFFEF8\]  
                                 ====>ST\=83.000000000000000000

:0048B847 8B03                    mov eax, dword ptr \[ebx\]  
:0048B849 53                      push ebx  
:0048B84A FF902C030000            call dword ptr \[eax+0000032C\]  
:0048B850 50                      push eax  
:0048B851 8D4D94                  lea ecx, dword ptr \[ebp\-6C\]  
:0048B854 51                      push ecx

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B855 FF15D0104000            Call dword ptr \[004010D0\]  
:0048B85B 8BF0                    mov esi, eax  
:0048B85D 8B16                    mov edx, dword ptr \[esi\]  
:0048B85F 8D45A4                  lea eax, dword ptr \[ebp\-5C\]  
:0048B862 50                      push eax  
:0048B863 56                      push esi  
:0048B864 FF5250                  call \[edx+50\]  
:0048B867 DBE2                    fclex  
:0048B869 85C0                    test eax, eax  
:0048B86B 7D0F                    jge 0048B87C  
:0048B86D 6A50                    push 00000050

\* Possible StringData Ref from Data Obj ->"貼?檉??"  
                                 |  
:0048B86F 6878F54000              push 0040F578  
:0048B874 56                      push esi  
:0048B875 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B876 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B86B(C)  
|  
:0048B87C 8B4DA4                  mov ecx, dword ptr \[ebp\-5C\]  
                                 ====>ECX\=97

:0048B87F 51                      push ecx

\* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h  
                                 |  
:0048B880 FF15F0124000            Call dword ptr \[004012F0\]  
:0048B886 DD9DF0FEFFFF            fstp qword ptr \[ebp+FFFFFEF0\]  
                                 ====>ST\=97.000000000000000000

:0048B88C 8B13                    mov edx, dword ptr \[ebx\]  
:0048B88E 53                      push ebx  
:0048B88F FF9228030000            call dword ptr \[edx+00000328\]  
:0048B895 50                      push eax  
:0048B896 8D4590                  lea eax, dword ptr \[ebp\-70\]  
:0048B899 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048B89A FF15D0104000            Call dword ptr \[004010D0\]  
:0048B8A0 8BF0                    mov esi, eax  
:0048B8A2 8B0E                    mov ecx, dword ptr \[esi\]  
:0048B8A4 8D55A0                  lea edx, dword ptr \[ebp\-60\]  
:0048B8A7 52                      push edx  
:0048B8A8 56                      push esi  
:0048B8A9 FF5150                  call \[ecx+50\]  
:0048B8AC DBE2                    fclex  
:0048B8AE 85C0                    test eax, eax  
:0048B8B0 7D0F                    jge 0048B8C1  
:0048B8B2 6A50                    push 00000050

\* Possible StringData Ref from Data Obj ->"貼?檉??"  
                                 |  
:0048B8B4 6878F54000              push 0040F578  
:0048B8B9 56                      push esi  
:0048B8BA 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B8BB FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B8B0(C)  
|  
:0048B8C1 8B45A0                  mov eax, dword ptr \[ebp\-60\]  
                                 ====>EAX\=1004365879         程序自给？！

:0048B8C4 50                      push eax

  
###############晕——下面这些浮点运算难道又是“冗余代码”？！###################

\* Reference To: MSVBVM60.rtcR8ValFromBstr, Ord:0245h  
                                 |  
:0048B8C5 FF15F0124000            Call dword ptr \[004012F0\]  
:0048B8CB DD9DE8FEFFFF            fstp qword ptr \[ebp+FFFFFEE8\]  
                                 ====>ST\=1004365879.0000000000

:0048B8D1 8B4B3C                  mov ecx, dword ptr \[ebx+3C\]  
                                 ====>ECX\=5554908250

:0048B8D4 51                      push ecx

\* Reference To: MSVBVM60.\_\_vbaR8Str, Ord:0000h  
                                 |  
:0048B8D5 FF1510124000            Call dword ptr \[00401210\]  
                                 ====>取数

:0048B8DB DCA500FFFFFF            fsub qword ptr \[ebp+FFFFFF00\]  
         ====>ST\=5554908250.0000000000 - 121.0000000000000=5554908129.0000000000

:0048B8E1 DC8DF8FEFFFF            fmul qword ptr \[ebp+FFFFFEF8\]  
         ====>ST\=5554908129.0000000000 \* 83.00000000000000=4.6105737470700000000e+11

:0048B8E7 833D00F0490000          cmp dword ptr \[0049F000\], 00000000  
:0048B8EE 7508                    jne 0048B8F8  
:0048B8F0 DCB5F0FEFFFF            fdiv qword ptr \[ebp+FFFFFEF0\]  
         ====>ST\=4.6105737470700000000e+11 / 97.00000000000000=4753168811.4123711340

  
:0048B8F6 EB11                    jmp 0048B909

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B8EE(C)  
|  
:0048B8F8 FFB5F4FEFFFF            push dword ptr \[ebp+FFFFFEF4\]  
:0048B8FE FFB5F0FEFFFF            push dword ptr \[ebp+FFFFFEF0\]

\* Reference To: MSVBVM60.\_adj\_fdiv\_m64, Ord:0000h  
                                 |  
:0048B904 E8AB6FF7FF              Call 004028B4

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B8F6(U)  
|  
:0048B909 DC85E8FEFFFF            fadd qword ptr \[ebp+FFFFFEE8\]  
         ====>ST\=4753168811.4123711340 + 1004365879.000000=5757534690.4123711340

:0048B90F DFE0                    fstsw ax  
:0048B911 A80D                    test al, 0D  
:0048B913 0F857C0B0000            jne 0048C495

\* Reference To: MSVBVM60.\_\_vbaFPInt, Ord:0000h  
                                 |  
:0048B919 FF15D4124000            Call dword ptr \[004012D4\]  
:0048B91F DD5DD0                  fstp qword ptr \[ebp\-30\]  
                                 ====>ST\=5757534690.0000000000

:0048B922 8D55A0                  lea edx, dword ptr \[ebp\-60\]  
:0048B925 52                      push edx  
:0048B926 8D45A4                  lea eax, dword ptr \[ebp\-5C\]  
:0048B929 50                      push eax  
:0048B92A 8D4DA8                  lea ecx, dword ptr \[ebp\-58\]  
:0048B92D 51                      push ecx  
:0048B92E 8D55AC                  lea edx, dword ptr \[ebp\-54\]  
:0048B931 52                      push edx  
:0048B932 6A04                    push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeStrList, Ord:0000h  
                                 |  
:0048B934 FF152C124000            Call dword ptr \[0040122C\]  
:0048B93A 8D4590                  lea eax, dword ptr \[ebp\-70\]  
:0048B93D 50                      push eax  
:0048B93E 8D4D94                  lea ecx, dword ptr \[ebp\-6C\]  
:0048B941 51                      push ecx  
:0048B942 8D5598                  lea edx, dword ptr \[ebp\-68\]  
:0048B945 52                      push edx  
:0048B946 8D459C                  lea eax, dword ptr \[ebp\-64\]  
:0048B949 50                      push eax  
:0048B94A 6A04                    push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeObjList, Ord:0000h  
                                 |  
:0048B94C FF1550104000            Call dword ptr \[00401050\]  
:0048B952 83C428                  add esp, 00000028  
:0048B955 A1B8F24900              mov eax, dword ptr \[0049F2B8\]  
:0048B95A 85C0                    test eax, eax  
:0048B95C 7510                    jne 0048B96E  
:0048B95E 68B8F24900              push 0049F2B8

\* Possible StringData Ref from Data Obj ->""  
                                 |  
:0048B963 680CC14000              push 0040C10C

\* Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h  
                                 |  
:0048B968 FF1514124000            Call dword ptr \[00401214\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B95C(C)  
|  
:0048B96E 8B35B8F24900            mov esi, dword ptr \[0049F2B8\]  
:0048B974 8B0E                    mov ecx, dword ptr \[esi\]  
:0048B976 56                      push esi  
:0048B977 FF9104070000            call dword ptr \[ecx+00000704\]  
:0048B97D DBE2                    fclex  
:0048B97F 85C0                    test eax, eax  
:0048B981 7D12                    jge 0048B995  
:0048B983 6804070000              push 00000704  
:0048B988 68E4E74000              push 0040E7E4  
:0048B98D 56                      push esi  
:0048B98E 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048B98F FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B981(C)  
|  
:0048B995 8B55C0                  mov edx, dword ptr \[ebp\-40\]  
:0048B998 52                      push edx

\* Possible StringData Ref from Data Obj ->"SSNHHQXTDNET"  
                                 |  
:0048B999 68601D4100              push 00411D60

\* Reference To: MSVBVM60.\_\_vbaStrCmp, Ord:0000h  
                                 |  
:0048B99E 8B3534114000            mov esi, dword ptr \[00401134\]  
:0048B9A4 FFD6                    call esi  
:0048B9A6 85C0                    test eax, eax  
:0048B9A8 7431                    je 0048B9DB  
:0048B9AA 8B55C0                  mov edx, dword ptr \[ebp\-40\]  
:0048B9AD 52                      push edx

\* Possible StringData Ref from Data Obj ->"SSNHHQXTDORG"  
                                 |  
:0048B9AE 687C1D4100              push 00411D7C  
:0048B9B3 FFD6                    call esi  
:0048B9B5 85C0                    test eax, eax  
:0048B9B7 7422                    je 0048B9DB  
:0048B9B9 8B55C0                  mov edx, dword ptr \[ebp\-40\]  
:0048B9BC 52                      push edx

\* Possible StringData Ref from Data Obj ->"SSNHHQXTDCN"  
                                 |  
:0048B9BD 68981D4100              push 00411D98  
:0048B9C2 FFD6                    call esi  
:0048B9C4 85C0                    test eax, eax  
:0048B9C6 7413                    je 0048B9DB  
:0048B9C8 8B55C0                  mov edx, dword ptr \[ebp\-40\]  
:0048B9CB 52                      push edx

\* Possible StringData Ref from Data Obj ->"SSNHHQXTDLZK"  
                                 |  
:0048B9CC 68B41D4100              push 00411DB4  
:0048B9D1 FFD6                    call esi  
:0048B9D3 85C0                    test eax, eax  
:0048B9D5 0F859B000000            jne 0048BA76

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0048B9A8(C), :0048B9B7(C), :0048B9C6(C)  
|  
:0048B9DB B904000280              mov ecx, 80020004  
:0048B9E0 898D58FFFFFF            mov dword ptr \[ebp+FFFFFF58\], ecx  
:0048B9E6 B80A000000              mov eax, 0000000A  
:0048B9EB 898550FFFFFF            mov dword ptr \[ebp+FFFFFF50\], eax  
:0048B9F1 898D68FFFFFF            mov dword ptr \[ebp+FFFFFF68\], ecx  
:0048B9F7 898560FFFFFF            mov dword ptr \[ebp+FFFFFF60\], eax  
:0048B9FD 898D78FFFFFF            mov dword ptr \[ebp+FFFFFF78\], ecx  
:0048BA03 898570FFFFFF            mov dword ptr \[ebp+FFFFFF70\], eax

\* Possible StringData Ref from Data Obj ->"鑜孮 b 烺�"  
                                 |  
:0048BA09 C78548FFFFFF601C4100    mov dword ptr \[ebp+FFFFFF48\], 00411C60  
:0048BA13 C78540FFFFFF08000000    mov dword ptr \[ebp+FFFFFF40\], 00000008  
:0048BA1D 8D9540FFFFFF            lea edx, dword ptr \[ebp+FFFFFF40\]  
:0048BA23 8D4D80                  lea ecx, dword ptr \[ebp\-80\]

\* Reference To: MSVBVM60.\_\_vbaVarDup, Ord:0000h  
                                 |  
:0048BA26 FF156C124000            Call dword ptr \[0040126C\]  
:0048BA2C 8D8550FFFFFF            lea eax, dword ptr \[ebp+FFFFFF50\]  
:0048BA32 50                      push eax  
:0048BA33 8D8D60FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF60\]  
:0048BA39 51                      push ecx  
:0048BA3A 8D9570FFFFFF            lea edx, dword ptr \[ebp+FFFFFF70\]  
:0048BA40 52                      push edx  
:0048BA41 6A00                    push 00000000  
:0048BA43 8D4580                  lea eax, dword ptr \[ebp\-80\]  
:0048BA46 50                      push eax

\* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h  
                                 |  
:0048BA47 FF15CC104000            Call dword ptr \[004010CC\]  
:0048BA4D 8D8D50FFFFFF            lea ecx, dword ptr \[ebp+FFFFFF50\]  
:0048BA53 51                      push ecx  
:0048BA54 8D9560FFFFFF            lea edx, dword ptr \[ebp+FFFFFF60\]  
:0048BA5A 52                      push edx  
:0048BA5B 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]  
:0048BA61 50                      push eax  
:0048BA62 8D4D80                  lea ecx, dword ptr \[ebp\-80\]  
:0048BA65 51                      push ecx  
:0048BA66 6A04                    push 00000004

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0048BA68 FF1540104000            Call dword ptr \[00401040\]  
:0048BA6E 83C414                  add esp, 00000014  
:0048BA71 E916080000              jmp 0048C28C

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048B9D5(C)  
|  
:0048BA76 8B13                    mov edx, dword ptr \[ebx\]  
:0048BA78 53                      push ebx  
:0048BA79 FF9214030000            call dword ptr \[edx+00000314\]  
:0048BA7F 50                      push eax  
:0048BA80 8D459C                  lea eax, dword ptr \[ebp\-64\]  
:0048BA83 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaObjSet, Ord:0000h  
                                 |  
:0048BA84 FF15D0104000            Call dword ptr \[004010D0\]  
:0048BA8A 8BF0                    mov esi, eax  
:0048BA8C 8B0E                    mov ecx, dword ptr \[esi\]  
:0048BA8E 8D55A8                  lea edx, dword ptr \[ebp\-58\]  
:0048BA91 52                      push edx  
:0048BA92 56                      push esi  
:0048BA93 FF91A0000000            call dword ptr \[ecx+000000A0\]  
:0048BA99 DBE2                    fclex  
:0048BA9B 85C0                    test eax, eax  
:0048BA9D 7D12                    jge 0048BAB1  
:0048BA9F 68A0000000              push 000000A0

\* Possible StringData Ref from Data Obj ->"酦?檉??"  
                                 |  
:0048BAA4 6898044100              push 00410498  
:0048BAA9 56                      push esi  
:0048BAAA 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048BAAB FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048BA9D(C)  
|  
:0048BAB1 8B4338                  mov eax, dword ptr \[ebx+38\]  
:0048BAB4 8B10                    mov edx, dword ptr \[eax\]  
:0048BAB6 8D4DAC                  lea ecx, dword ptr \[ebp\-54\]  
:0048BAB9 51                      push ecx  
:0048BABA 8D4B3C                  lea ecx, dword ptr \[ebx+3C\]  
:0048BABD 51                      push ecx  
:0048BABE 50                      push eax  
:0048BABF FF5220                  call \[edx+20\]  
                                 ====>算法 CALL！进入！

:0048BAC2 DBE2                    fclex  
:0048BAC4 85C0                    test eax, eax  
:0048BAC6 7D12                    jge 0048BADA  
:0048BAC8 6A20                    push 00000020

\* Possible StringData Ref from Data Obj ->"趠皕 LH?g0 瓬 KRKeyString"  
                                 |  
:0048BACA 687C1C4100              push 00411C7C  
:0048BACF 8B5338                  mov edx, dword ptr \[ebx+38\]  
:0048BAD2 52                      push edx  
:0048BAD3 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaHresultCheckObj, Ord:0000h  
                                 |  
:0048BAD4 FF15A4104000            Call dword ptr \[004010A4\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0048BAC6(C)  
|  
:0048BADA 8B45A8                  mov eax, dword ptr \[ebp\-58\]  
                                 ====>EAX\=1357246890         试炼码！

:0048BADD 50                      push eax  
:0048BADE 8B4DAC                  mov ecx, dword ptr \[ebp\-54\]  
                                 ====>ECX\=bdm1P yZd2         注册码！

:0048BAE1 51                      push ecx

\* Reference To: MSVBVM60.\_\_vbaStrCmp, Ord:0000h  
                                 |  
:0048BAE2 FF1534114000            Call dword ptr \[00401134\]  
                                 ====>比较真假码！

:0048BAE8 8BF0                    mov esi, eax  
                                 ====>ESI\=EAX\=FFFFFFFF   这是试炼码比较后返回的结果

:0048BAEA F7DE                    neg esi  
                                 ====>ESI\=00000001

:0048BAEC 1BF6                    sbb esi, esi  
                                 ====>ESI\=FFFFFFFF       选择此处作为完美爆破点！

:0048BAEE 46                      inc esi  
                                 ====>ESI\=00000000

:0048BAEF F7DE                    neg esi  
                                 ====>ESI\=00000000

:0048BAF1 8D55AC                  lea edx, dword ptr \[ebp\-54\]  
:0048BAF4 52                      push edx  
:0048BAF5 8D45A8                  lea eax, dword ptr \[ebp\-58\]  
:0048BAF8 50                      push eax  
:0048BAF9 6A02                    push 00000002

\* Reference To: MSVBVM60.\_\_vbaFreeStrList, Ord:0000h  
                                 |  
:0048BAFB FF152C124000            Call dword ptr \[0040122C\]  
:0048BB01 83C40C                  add esp, 0000000C  
:0048BB04 8D4D9C                  lea ecx, dword ptr \[ebp\-64\]

\* Reference To: MSVBVM60.\_\_vbaFreeObj, Ord:0000h  
                                 |  
:0048BB07 FF15EC124000            Call dword ptr \[004012EC\]  
:0048BB0D 6685F6                  test si, si  
                                 ====>根据 SI 的值判断成功或是失败！

:0048BB10 0F8497060000            je 0048C1AD  
                                 ====>跳则 OVER

:0048BB16 A1CCF24900              mov eax, dword ptr \[0049F2CC\]  
:0048BB1B 85C0                    test eax, eax  
:0048BB1D 7515                    jne 0048BB34  
:0048BB1F 68CCF24900              push 0049F2CC

\* Possible StringData Ref from Data Obj ->""  
                                 |  
:0048BB24 68046B4000              push 00406B04

\* Reference To: MSVBVM60.\_\_vbaNew2, Ord:0000h  
                                 |  
:0048BB29 FF1514124000            Call dword ptr \[00401214\]  
:0048BB2F A1CCF24900              mov eax, dword ptr \[0049F2CC\]

  
—————————————————————————————————   
进入算法 CALL：0048BABF  call \[edx+20\]后来到这里！

  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C34F(U)  
|  
:0049C0E0 663B8560FFFFFF          cmp ax, word ptr \[ebp+FFFFFF60\]  
:0049C0E7 0F8F67020000            jg 0049C354  
:0049C0ED 8D55E0                  lea edx, dword ptr \[ebp\-20\]

\* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h  
                                 |  
:0049C0F0 8B1D18114000            mov ebx, dword ptr \[00401118\]  
:0049C0F6 895598                  mov dword ptr \[ebp\-68\], edx  
:0049C0F9 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]  
:0049C0FC 0FBFD0                  movsx edx, ax  
:0049C0FF 51                      push ecx  
:0049C100 8D4590                  lea eax, dword ptr \[ebp\-70\]  
:0049C103 52                      push edx  
:0049C104 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]  
:0049C107 50                      push eax  
:0049C108 51                      push ecx  
:0049C109 C745C801000000          mov \[ebp\-38\], 00000001  
:0049C110 C745C002000000          mov \[ebp\-40\], 00000002  
:0049C117 C7459008400000          mov \[ebp\-70\], 00004008  
:0049C11E FFD3                    call ebx  
                                 ====>依次取机器码作为大循环的参数！

:0049C120 8D55B0                  lea edx, dword ptr \[ebp\-50\]  
:0049C123 52                      push edx

\* Reference To: MSVBVM60.\_\_vbaStrVarMove, Ord:0000h  
                                 |  
:0049C124 FF1534104000            Call dword ptr \[00401034\]

\* Reference To: MSVBVM60.\_\_vbaStrMove, Ord:0000h  
                                 |  
:0049C12A 8B3DA4124000            mov edi, dword ptr \[004012A4\]  
:0049C130 8BD0                    mov edx, eax  
:0049C132 8D4DE8                  lea ecx, dword ptr \[ebp\-18\]  
:0049C135 FFD7                    call edi  
:0049C137 8D45B0                  lea eax, dword ptr \[ebp\-50\]  
:0049C13A 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]  
:0049C13D 50                      push eax  
:0049C13E 51                      push ecx  
:0049C13F 6A02                    push 00000002

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0049C141 FF1540104000            Call dword ptr \[00401040\]  
:0049C147 8B563C                  mov edx, dword ptr \[esi+3C\]

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
\[esi+3C\]内存中的值是一张表：

0015C2C4  38 00 78 00 33 00 70 00 35 00 42 00 65 00 61 00  8.x.3.p.5.B.e.a.  
0015C2D4  62 00 63 00 64 00 66 00 67 00 68 00 69 00 6A 00  b.c.d.f.g.h.i.j.  
0015C2E4  6B 00 6C 00 6D 00 6E 00 6F 00 71 00 72 00 73 00  k.l.m.n.o.q.r.s.  
0015C2F4  74 00 75 00 76 00 77 00 79 00 7A 00 41 00 43 00  t.u.v.w.y.z.A.C.  
0015C304  44 00 45 00 46 00 47 00 48 00 49 00 4A 00 4B 00  D.E.F.G.H.I.J.K.  
0015C314  4C 00 4D 00 4E 00 4F 00 50 00 51 00 52 00 53 00  L.M.N.O.P.Q.R.S.  
0015C324  54 00 55 00 56 00 57 00 58 00 59 00 5A 00 20 00  T.U.V.W.X.Y.Z. .  
0015C334  31 00 32 00 34 00 36 00 37 00 39 00 30 00 2D 00  1.2.4.6.7.9.0.-.  
0015C344  2E 00 23 00 2F 00 5C 00 21 00 40 00 24 00 3C 00  ..#./.\\.!.@.$.<.  
0015C354  3E 00 26 00 2A 00 28 00 29 00 5B 00 5D 00 7B 00  >.&.\*.(.).\[.\].{.  
0015C364  7D 00 27 00 3B 00 3A 00 2C 00 3F 00 3D 00 2B 00  }.'.;.:.,.?.=.+.  
0015C374  7E 00 60 00 5E 00 7C 00 25 00 5F 00 0D 00 0A 00  ~.\`.^.|.%.\_.....  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

:0049C14A 8B45E8                  mov eax, dword ptr \[ebp\-18\]  
:0049C14D 83C40C                  add esp, 0000000C  
:0049C150 6A01                    push 00000001  
:0049C152 52                      push edx  
:0049C153 50                      push eax  
:0049C154 6A00                    push 00000000

\* Reference To: MSVBVM60.\_\_vbaInStr, Ord:0000h  
                                 |  
:0049C156 FF150C124000            Call dword ptr \[0040120C\]  
:0049C15C 8BC8                    mov ecx, eax

\* Reference To: MSVBVM60.\_\_vbaI2I4, Ord:0000h  
                                 |  
:0049C15E FF1550114000            Call dword ptr \[00401150\]  
:0049C164 6689463A                mov word ptr \[esi+3A\], ax  
:0049C168 8D4E48                  lea ecx, dword ptr \[esi+48\]  
:0049C16B 0FBF45D8                movsx eax, word ptr \[ebp\-28\]  
:0049C16F 8D55C0                  lea edx, dword ptr \[ebp\-40\]  
:0049C172 894D98                  mov dword ptr \[ebp\-68\], ecx  
:0049C175 52                      push edx  
:0049C176 8D4D90                  lea ecx, dword ptr \[ebp\-70\]  
:0049C179 50                      push eax  
:0049C17A 8D55B0                  lea edx, dword ptr \[ebp\-50\]  
:0049C17D 51                      push ecx  
:0049C17E 52                      push edx  
:0049C17F C745C801000000          mov \[ebp\-38\], 00000001  
:0049C186 C745C002000000          mov \[ebp\-40\], 00000002  
:0049C18D C7459008400000          mov \[ebp\-70\], 00004008  
:0049C194 FFD3                    call ebx  
                                 ====>依次取 girlboy 字符的 HEX 值作为大循环的参数！

:0049C196 8D45B0                  lea eax, dword ptr \[ebp\-50\]  
:0049C199 8D5E4C                  lea ebx, dword ptr \[esi+4C\]  
:0049C19C 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaStrVarMove, Ord:0000h  
                                 |  
:0049C19D FF1534104000            Call dword ptr \[00401034\]  
:0049C1A3 8BD0                    mov edx, eax  
:0049C1A5 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]  
:0049C1A8 FFD7                    call edi  
:0049C1AA 8BD0                    mov edx, eax  
                                 ====>EDX 的内存处是依次取的 girlboy 字符的 HEX 值

:0049C1AC 8BCB                    mov ecx, ebx  
                                 ====>EDX 的内存处是依次取的机器码字符的 HEX 值

\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h  
                                 |  
:0049C1AE FF1524124000            Call dword ptr \[00401224\]  
:0049C1B4 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]

\* Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h  
                                 |  
:0049C1B7 FF15E8124000            Call dword ptr \[004012E8\]  
:0049C1BD 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]  
:0049C1C0 8D55C0                  lea edx, dword ptr \[ebp\-40\]  
:0049C1C3 51                      push ecx  
:0049C1C4 52                      push edx  
:0049C1C5 6A02                    push 00000002

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0049C1C7 FF1540104000            Call dword ptr \[00401040\]  
:0049C1CD 668B4638                mov ax, word ptr \[esi+38\]  
:0049C1D1 83C40C                  add esp, 0000000C  
:0049C1D4 66898558FFFFFF          mov word ptr \[ebp+FFFFFF58\], ax  
:0049C1DB B801000000              mov eax, 00000001  
:0049C1E0 8945E4                  mov dword ptr \[ebp\-1C\], eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C28B(U)  
|  
:0049C1E3 663B8558FFFFFF          cmp ax, word ptr \[ebp+FFFFFF58\]  
:0049C1EA 0F8F2D010000            jg 0049C31D  
:0049C1F0 0FBFF8                  movsx edi, ax  
:0049C1F3 83FF62                  cmp edi, 00000062  
:0049C1F6 C745C801000000          mov \[ebp\-38\], 00000001  
:0049C1FD C745C002000000          mov \[ebp\-40\], 00000002  
:0049C204 7206                    jb 0049C20C

\* Reference To: MSVBVM60.\_\_vbaGenerateBoundsError, Ord:0000h  
                                 |  
:0049C206 FF152C114000            Call dword ptr \[0040112C\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C204(C)  
|  
:0049C20C 8B4E68                  mov ecx, dword ptr \[esi+68\]  
:0049C20F 8D45C0                  lea eax, dword ptr \[ebp\-40\]  
:0049C212 50                      push eax  
:0049C213 8D45B0                  lea eax, dword ptr \[ebp\-50\]  
:0049C216 8D14B9                  lea edx, dword ptr \[ecx+4\*edi\]  
:0049C219 C7459008400000          mov \[ebp\-70\], 00004008  
:0049C220 0FBF4E3A                movsx ecx, word ptr \[esi+3A\]  
:0049C224 895598                  mov dword ptr \[ebp\-68\], edx  
:0049C227 8D5590                  lea edx, dword ptr \[ebp\-70\]  
:0049C22A 51                      push ecx  
:0049C22B 52                      push edx  
:0049C22C 50                      push eax

\* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h  
                                 |  
:0049C22D FF1518114000            Call dword ptr \[00401118\]  
                                 ====>依据运算的不同参数从表中取字符！

:0049C233 8B0B                    mov ecx, dword ptr \[ebx\]  
:0049C235 8D55B0                  lea edx, dword ptr \[ebp\-50\]  
:0049C238 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]  
:0049C23E 52                      push edx  
:0049C23F 50                      push eax  
:0049C240 898D78FFFFFF            mov dword ptr \[ebp+FFFFFF78\], ecx  
:0049C246 C78570FFFFFF08800000    mov dword ptr \[ebp+FFFFFF70\], 00008008

\* Reference To: MSVBVM60.\_\_vbaVarTstEq, Ord:0000h  
                                 |  
:0049C250 FF153C114000            Call dword ptr \[0040113C\]  
:0049C256 8D4DB0                  lea ecx, dword ptr \[ebp\-50\]  
:0049C259 8D55C0                  lea edx, dword ptr \[ebp\-40\]  
:0049C25C 51                      push ecx  
:0049C25D 52                      push edx  
:0049C25E 6A02                    push 00000002  
:0049C260 898568FFFFFF            mov dword ptr \[ebp+FFFFFF68\], eax

\* Reference To: MSVBVM60.\_\_vbaFreeVarList, Ord:0000h  
                                 |  
:0049C266 FF1540104000            Call dword ptr \[00401040\]  
:0049C26C 83C40C                  add esp, 0000000C  
:0049C26F 6683BD68FFFFFF00        cmp word ptr \[ebp+FFFFFF68\], 0000  
:0049C277 7517                    jne 0049C290  
:0049C279 B801000000              mov eax, 00000001  
:0049C27E 660345E4                add ax, word ptr \[ebp\-1C\]  
:0049C282 0F8043010000            jo 0049C3CB  
:0049C288 8945E4                  mov dword ptr \[ebp\-1C\], eax  
:0049C28B E953FFFFFF              jmp 0049C1E3  
                                 ====>继续“疯狂”小循环！

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C277(C)  
|  
:0049C290 83FF62                  cmp edi, 00000062  
:0049C293 7206                    jb 0049C29B

\* Reference To: MSVBVM60.\_\_vbaGenerateBoundsError, Ord:0000h  
                                 |  
:0049C295 FF152C114000            Call dword ptr \[0040112C\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C293(C)  
|  
:0049C29B 8B4668                  mov eax, dword ptr \[esi+68\]  
:0049C29E 8D5590                  lea edx, dword ptr \[ebp\-70\]  
:0049C2A1 6A01                    push 00000001  
:0049C2A3 52                      push edx  
:0049C2A4 8D0CB8                  lea ecx, dword ptr \[eax+4\*edi\]  
:0049C2A7 8D45C0                  lea eax, dword ptr \[ebp\-40\]  
:0049C2AA 50                      push eax  
:0049C2AB 894D98                  mov dword ptr \[ebp\-68\], ecx  
:0049C2AE C7459008400000          mov \[ebp\-70\], 00004008

\* Reference To: MSVBVM60.rtcLeftCharVar, Ord:0269h  
                                 |  
:0049C2B5 FF1590124000            Call dword ptr \[00401290\]  
:0049C2BB 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]  
:0049C2BE 8D7E54                  lea edi, dword ptr \[esi+54\]  
:0049C2C1 51                      push ecx

\* Reference To: MSVBVM60.\_\_vbaStrVarMove, Ord:0000h  
                                 |  
:0049C2C2 FF1534104000            Call dword ptr \[00401034\]  
:0049C2C8 8BD0                    mov edx, eax  
:0049C2CA 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]

\* Reference To: MSVBVM60.\_\_vbaStrMove, Ord:0000h  
                                 |  
:0049C2CD FF15A4124000            Call dword ptr \[004012A4\]  
:0049C2D3 8BD0                    mov edx, eax  
                                 ====>依次循环得出结果：b、d、m、1、P、 、y、X、d、2

:0049C2D5 8BCF                    mov ecx, edi

\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h  
                                 |  
:0049C2D7 FF1524124000            Call dword ptr \[00401224\]  
:0049C2DD 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]

\* Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h  
                                 |  
:0049C2E0 FF15E8124000            Call dword ptr \[004012E8\]  
:0049C2E6 8D4DC0                  lea ecx, dword ptr \[ebp\-40\]

\* Reference To: MSVBVM60.\_\_vbaFreeVar, Ord:0000h  
                                 |  
:0049C2E9 FF152C104000            Call dword ptr \[0040102C\]  
:0049C2EF 8B5650                  mov edx, dword ptr \[esi+50\]  
:0049C2F2 8B07                    mov eax, dword ptr \[edi\]  
:0049C2F4 8D5E50                  lea ebx, dword ptr \[esi+50\]  
:0049C2F7 52                      push edx  
:0049C2F8 50                      push eax

\* Reference To: MSVBVM60.\_\_vbaStrCat, Ord:0000h  
                                 |  
:0049C2F9 FF1574104000            Call dword ptr \[00401074\]  
                                 ====>把依次循环所得字符连接起来！

:0049C2FF 8BD0                    mov edx, eax

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
\[EAX\]内存中的值是上面 10 次循环运算得出的字符：

0017996C  62 00 64 00 6D 00 31 00 50 00 20 00 79 00 58 00  b.d.m.1.P. .y.X.  
0017997C  64 00 32 00                                      d.2.  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

  
:0049C301 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]

\* Reference To: MSVBVM60.\_\_vbaStrMove, Ord:0000h  
                                 |  
:0049C304 FF15A4124000            Call dword ptr \[004012A4\]  
:0049C30A 8BD0                    mov edx, eax  
:0049C30C 8BCB                    mov ecx, ebx

\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h  
                                 |  
:0049C30E FF1524124000            Call dword ptr \[00401224\]  
:0049C314 8D4DD0                  lea ecx, dword ptr \[ebp\-30\]

\* Reference To: MSVBVM60.\_\_vbaFreeStr, Ord:0000h  
                                 |  
:0049C317 FF15E8124000            Call dword ptr \[004012E8\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C1EA(C)  
|  
:0049C31D 668B45D8                mov ax, word ptr \[ebp\-28\]  
:0049C321 66050100                add ax, 0001  
:0049C325 0F80A0000000            jo 0049C3CB  
:0049C32B 663B4634                cmp ax, word ptr \[esi+34\]  
:0049C32F 8945D8                  mov dword ptr \[ebp\-28\], eax  
:0049C332 7E07                    jle 0049C33B  
:0049C334 C745D801000000          mov \[ebp\-28\], 00000001

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C332(C)

\* Reference To: MSVBVM60.\_\_vbaStrCopy, Ord:0000h  
                                 |  
:0049C33B 8B3D24124000            mov edi, dword ptr \[00401224\]  
:0049C341 B801000000              mov eax, 00000001  
:0049C346 660345D4                add ax, word ptr \[ebp\-2C\]  
:0049C34A 707F                    jo 0049C3CB  
:0049C34C 8945D4                  mov dword ptr \[ebp\-2C\], eax  
:0049C34F E98CFDFFFF              jmp 0049C0E0  
                                 ====>大循环，依次得出注册码！

  
—————————————————————————————————   
进入 0049C22D  Call dword ptr \[00401118\]

  
733B45C1   /74 19           JE SHORT MSVBVM60.733B45DC  
733B45C3   |50              PUSH EAX  
733B45C4   |8B45 0C         MOV EAX,DWORD PTR SS:\[EBP+C\]  
733B45C7   |8D4400 FF       LEA EAX,DWORD PTR DS:\[EAX+EAX\-1\]  
733B45CB   |50              PUSH EAX  
733B45CC   |FF75 08         PUSH DWORD PTR SS:\[EBP+8\]  
733B45CF   |E8 1E000000     CALL MSVBVM60.rtcMidBstr  
733B45D4   |C9              LEAVE  
733B45D5   |C2 0C00         RETN 0C

733B45D8    33C9            XOR ECX,ECX  
733B45DA  ^ EB E2           JMP SHORT MSVBVM60.733B45BE  
733B45DC    50              PUSH EAX  
733B45DD    66:C745 F0 0300 MOV WORD PTR SS:\[EBP\-10\],3  
733B45E3    E8 7A010000     CALL MSVBVM60.733B4762  
733B45E8    D1E0            SHL EAX,1  
733B45EA    8945 F8         MOV DWORD PTR SS:\[EBP\-8\],EAX  
733B45ED    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP\-10\]  
733B45F0  ^ EB D1           JMP SHORT MSVBVM60.733B45C3  
733B45F2 >  8B4424 08       MOV EAX,DWORD PTR SS:\[ESP+8\]  
733B45F6    53              PUSH EBX  
733B45F7    56              PUSH ESI  
733B45F8    57              PUSH EDI  
733B45F9    8D78 FF         LEA EDI,DWORD PTR DS:\[EAX\-1\]  
733B45FC    85FF            TEST EDI,EDI  
733B45FE    0F8C 10330200   JL MSVBVM60.733D7914  
733B4604    81FF FFFFFF7F   CMP EDI,7FFFFFFF  
733B460A    0F8F 04330200   JG MSVBVM60.733D7914  
733B4610    8B4424 10       MOV EAX,DWORD PTR SS:\[ESP+10\]  
733B4614    8BDF            MOV EBX,EDI  
733B4616    85C0            TEST EAX,EAX  
733B4618    0F84 FD320200   JE MSVBVM60.733D791B  
733B461E    8B70 FC         MOV ESI,DWORD PTR DS:\[EAX\-4\]  
733B4621    3BFE            CMP EDI,ESI  
733B4623    0F87 F9320200   JA MSVBVM60.733D7922  
733B4629    8B4C24 18       MOV ECX,DWORD PTR SS:\[ESP+18\]  
733B462D    66:8339 0A      CMP WORD PTR DS:\[ECX\],0A  
733B4631    75 35           JNZ SHORT MSVBVM60.733B4668  
733B4633    8179 08 0400028>CMP DWORD PTR DS:\[ECX+8\],80020004  
733B463A    75 2C           JNZ SHORT MSVBVM60.733B4668  
733B463C    83C8 FF         OR EAX,FFFFFFFF  
733B463F    66:85C0         TEST AX,AX  
733B4642    74 28           JE SHORT MSVBVM60.733B466C  
733B4644    2BF3            SUB ESI,EBX  
733B4646    8BC6            MOV EAX,ESI  
733B4648    50              PUSH EAX  
733B4649    8B4424 14       MOV EAX,DWORD PTR SS:\[ESP+14\]  
733B464D    03D8            ADD EBX,EAX  
733B464F    53              PUSH EBX  
733B4650    FF15 EC193973   CALL DWORD PTR DS:\[<&OLEAUT32.#150>\]; OLEAUT32.SysAllocStringByteLen  
733B4656    8BF0            MOV ESI,EAX  
733B4658    85F6            TEST ESI,ESI  
733B465A    0F84 D0320200   JE MSVBVM60.733D7930  
733B4660    8BC6            MOV EAX,ESI  
733B4662    5F              POP EDI  
733B4663    5E              POP ESI  
733B4664    5B              POP EBX  
733B4665    C2 0C00         RETN 0C

733B4648    50              PUSH EAX  
733B4649    8B4424 14       MOV EAX,DWORD PTR SS:\[ESP+14\]  
733B464D    03D8            ADD EBX,EAX  
733B464F    53              PUSH EBX  
733B4650    FF15 EC193973   CALL DWORD PTR DS:\[<&OLEAUT32.#150>\]; OLEAUT32.SysAllocStringByteLen  
733B4656    8BF0            MOV ESI,EAX  
733B4658    85F6            TEST ESI,ESI  
733B465A    0F84 D0320200   JE MSVBVM60.733D7930  
733B4660    8BC6            MOV EAX,ESI  
733B4662    5F              POP EDI  
733B4663    5E              POP ESI  
733B4664    5B              POP EBX  
733B4665    C2 0C00         RETN 0C

  
—————————————————————————————————   
【完 美  爆 破】：

  
1、0048B3FE 0F8460010000            je 0048B564  
    改为： E96101000090            jmp 0048B564     补一个 NOP

2、0048BAEC 1BF6                    sbb esi, esi  
    改为： 33F6                    xor esi, esi    
   
—————————————————————————————————   
【注册信息保存】：

  
C:\\WINDOWS\\SYSTEM 下的 MSEMPXDB.DLL 文件，XP 的在\\WINDOWS\\system32 下。  
呵呵，真够狡猾的呀！我找了半个小时！

—————————————————————————————————   
【整        理】：

  
机器码：5554908250  
注册码：bdm1P yXd2

—————————————————————————————————  
                                 
        ,     \_/   
       /| \_.-~/            \\\_     ,        青春都一饷  
      ( /~   /              \\~-.\_ |\\  
      \`\\\\  \_/                \\   ~\\ )          忍把浮名   
  \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
 /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
\`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
 /~~//'    /' \`~\\         ) /--..\_, )\_  \`~  
 "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                        "     "   "~'  ""  
   

                   Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]

                          2003-04-19  23:48  4dd466d0e1e0cc123afbe504da2462ca