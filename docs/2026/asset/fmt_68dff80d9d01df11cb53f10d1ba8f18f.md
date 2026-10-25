
# 极光多能闹钟 3.2

## 简单分析――极光多能闹钟 3.2

下载页面：`http://www.skycn.com/soft/10136.html`

软件大小：

1980 KB

软件语言：简体中文

软件类别：国产软件 / 共享版 / 开关定时

应用平台：Win9x/NT/2000/XP

加入时间：

2003-03-19 09:00:44

下载次数：13146

推荐等级：***

开 发 商：`http://jgsoft.wx-e.com/`

【软件简介】：支持电脑自带喇叭响铃和电脑音乐，并可自谱喇叭音乐，享受DIY的乐趣。可以按具体日期、每日、每周、每月、农历生日等不同方式提醒，灵活的提醒方式几乎可满足您所有的提醒需要。漂亮的石英钟界面和直观的液晶数字界面随时切换。使用中您会发现这是一款为您定制的程序，面板上几乎所有的东西都可依您所好来改变，石英钟大小可多级缩放，时钟的粗细，颜色都可改变，并支持自选图片。软件提供重要日期提醒、每小时固定响铃和使用方便的临时闹钟，支持正点报时和间隔报时，定时关机，定时执行指定程序，内置极光拖存工具，方便保存网上文摘。同时还附有一个原子钟对时工具和万年历。软件有非常方便的在线升级功能，升级前自动预读最新版本的更新内容，只下载需要更新的文件。未注册用户在两个月的试用期内没有任何功能的限制，也不会弹出窗口来干扰您。

【软件限制】：试用两个月

【作者声明】：初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`TRW2000`娃娃修改版、`Ollydbg1.09`、`PEiD`、`WKT_tElock_Dumper` 1.2、`W32Dasm 9.0`白金版

―――――――――――――――――――――――――――――――――

### 【过    程】：

首先必须说明的是我现在分析的是从《软件王》光盘里找的 V3.2 的旧版本，而“天空”上面的现在是 V4.4 的新版本，可能有些地方是不一样的。现在软件升级的速度都挺快，比我的破解水平提升的快多了。 ^\_^

呵呵，又碰上这个VB的东东，实在是不大乐意跟踪VB程序的算法，但是也不能见VB就“逃”呀，^O^^O^

静下心来我跟踪了好几遍，哎，现在是两眼都冒着VB的“金花”呀，更惨的是只找到了算法的地方而没有能力完整的总结出来，惭愧！这个东东太麻烦了，很多迷惑视线的“冗余代码”、假比较，教我如何不难过呀！

`JgClockXP.exe`

是`tElock 0.98`壳，用`WKT_tElock_Dumper 1.2`脱之。196K->676K。

机器码：`5554908250`

试炼码：`1357246890`

需要10位注册码！

―――――――――――――――――――――――――――――――――

```
* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
|:0048B333(C)
|
:0048B3ED
8B45BC                  mov eax, dword ptr [ebp-44]
                 ====>EAX=1357246890
:0048B3F0
50                      push eax
* Reference To:
MSVBVM60.__vbaLenBstr, Ord:0000h
                     |
:0048B3F1
FF1538104000            Call dword ptr [00401038]
         ====>取试炼码长度         EAX=A
:0048B3F7
BE0A000000              mov esi, 0000000A
         ====>ESI=A
:0048B3FC
3BC6                    cmp eax, esi
             ====>是否是10位注册码？注册码需要10位！
:0048B3FE
0F8460010000            je 0048B564
     ====>不跳则OVER！
:0048B404
B804000280              mov eax, 80020004
:0048B409
898558FFFFFF            mov dword ptr [ebp+FFFFFF58], eax
:0048B40F 89B550FFFFFF            mov dword ptr [ebp+FFFFFF50], esi
:0048B415 898568FFFFFF   mov dword ptr [ebp+FFFFFF68], eax
:0048B41B 89B560FFFFFF         mov dword ptr [ebp+FFFFFF60], esi
*
Possible StringData Ref from Data Obj ->"c:y"
   |
:0048B421 C78548FFFFFFE81B4100    mov dword ptr [ebp+FFFFFF48], 00411BE8
:0048B42B C78540FFFFFF08000000    mov dword ptr [ebp+FFFFFF40], 00000008
:0048B435 8D9540FFFFFF            lea edx, dword ptr [ebp+FFFFFF40]
:0048B43B 8D8D70FFFFFF   lea ecx, dword ptr [ebp+FFFFFF70]
*
Reference To: MSVBVM60.__vbaVarDup, Ord:0000h
 |
:0048B441 FF156C124000            Call dword ptr [0040126C]
*
Possible StringData Ref from Data Obj ->"e/TR,gz^lQOo`"
       |
:0048B447 68F41B4100     push 00411BF4
*
Possible StringData Ref from Data Obj ->""
     |
:0048B44C 684CF64000   push 0040F64C
*
Reference To: MSVBVM60.__vbaStrCat, Ord:0000h
 |
:0048B451 8B3574104000            mov esi, dword ptr [00401074]
:0048B457 FFD6         call esi
:0048B459 8BD0             mov edx, eax
:0048B45B 8D4DAC                 lea ecx, dword ptr [ebp-54]
:0048B45E FFD7   call edi
:0048B460 50         push eax
*
Possible StringData Ref from Data Obj ->""
     |
:0048B461 684CF64000   push 0040F64C
:0048B466 FFD6         call esi
:0048B468 8BD0             mov edx, eax
:0048B46A 8D4DA8                 lea ecx, dword ptr [ebp-58]
:0048B46D FFD7   call edi
:0048B46F 50         push eax
*
Possible StringData Ref from Data Obj ->"z^-Nlg lQ P[USRlQbR"
         |
:0048B470 68181C4100       push 00411C18
:0048B475 FFD6             call esi
:0048B477 894588               mov dword ptr [ebp-78], eax
:0048B47A C7458008000000          mov [ebp-80], 00000008
:0048B481 8D8D50FFFFFF            lea ecx, dword ptr [ebp+FFFFFF50]
:0048B487 51            push ecx
:0048B488 8D9560FFFFFF         lea edx, dword ptr [ebp+FFFFFF60]
:0048B48E 52                      push edx
:0048B48F 8D8570FFFFFF            lea eax, dword ptr [ebp+FFFFFF70]
:0048B495 50         push eax
:0048B496 6A40             push 00000040
:0048B498 8D4D80                 lea ecx, dword ptr [ebp-80]
:0048B49B 51     push ecx
*
Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
                       |
:0048B49C FF15CC104000            Call dword ptr [004010CC]
         ====>提示重启程序验证注册码！到这就OVER了！
:0048B4A2 8BC8                    mov ecx, eax
* Reference To: MSVBVM60.__vbaI2I4, Ord:0000h
                     |
:0048B4A4 FF1550114000            Call dword ptr [00401150]
:0048B4AA 8D55A8                  lea edx, dword ptr [ebp-58]
:0048B4AD 52       push edx
:0048B4AE 8D45AC         lea eax, dword ptr [ebp-54]
:0048B4B1 50                     push eax
:0048B4B2 6A02   push 00000002
*
Reference To: MSVBVM60.__vbaFreeStrList, Ord:0000h
 |
:0048B4B4 FF152C124000            Call dword ptr [0040122C]
:0048B4BA 8D8D50FFFFFF lea ecx, dword ptr [ebp+FFFFFF50]
:0048B4C0 51             push ecx
:0048B4C1 8D9560FFFFFF           lea edx, dword ptr [ebp+FFFFFF60]
:0048B4C7 52                      push edx
:0048B4C8 8D8570FFFFFF            lea eax, dword ptr [ebp+FFFFFF70]
:0048B4CE 50         push eax
:0048B4CF 8D4D80           lea ecx, dword ptr [ebp-80]
:0048B4D2 51                      push ecx
:0048B4D3 6A04   push 00000004
*
Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
 |
:0048B4D5 FF1540104000            Call dword ptr [00401040]
:0048B4DB 83C420     add esp, 00000020
:0048B4DE 8B13           mov edx, dword ptr [ebx]
:0048B4E0 53                      push ebx
:0048B4E1 FF9200070000            call dword ptr [edx+00000700]
:0048B4E7 85C0       test eax, eax
:0048B4E9 7D12             jge 0048B4FD
:0048B4EB 6800070000             push 00000700
:0048B4F0 68C4074100             push 004107C4
:0048B4F5 53                     push ebx
:0048B4F6 50   push eax
*
Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
   |
:0048B4F7 FF15A4104000            Call dword ptr [004010A4]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B4E9(C)
|
:0048B4FD
A1D0FD4900              mov eax, dword ptr [0049FDD0]
:0048B502 85C0     test eax, eax
:0048B504 7510           jne 0048B516
:0048B506 68D0FD4900             push 0049FDD0
:0048B50B 68B4F54000             push 0040F5B4
*
Reference To: MSVBVM60.__vbaNew2, Ord:0000h
                       |
:0048B510 FF1514124000            Call dword ptr [00401214]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B504(C)
|
:0048B516
A1D0FD4900              mov eax, dword ptr [0049FDD0]
:0048B51B 8985E4FEFFFF            mov dword ptr [ebp+FFFFFEE4], eax
:0048B521 8B30         mov esi, dword ptr [eax]
:0048B523 53                   push ebx
:0048B524 8D459C                  lea eax, dword ptr [ebp-64]
:0048B527 50       push eax
*
Reference To: MSVBVM60.__vbaObjSetAddref, Ord:0000h
 |
:0048B528 FF15DC104000            Call dword ptr [004010DC]
:0048B52E 50         push eax
:0048B52F 8BCE             mov ecx, esi
:0048B531 8BB5E4FEFFFF           mov esi, dword ptr [ebp+FFFFFEE4]
:0048B537 56                      push esi
:0048B538 FF5110 call [ecx+10]
:0048B53B DBE2       fclex
:0048B53D 85C0           test eax, eax
:0048B53F 7D0F                 jge 0048B550
:0048B541 6A10                    push 00000010
:0048B543 68A4F54000              push 0040F5A4
:0048B548 56     push esi
:0048B549 50           push eax
*
Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
   |
:0048B54A FF15A4104000            Call dword ptr [004010A4]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048B53F(C)
|
:0048B550
8D4D9C                  lea ecx, dword ptr [ebp-64]
*
Reference To: MSVBVM60.__vbaFreeObj, Ord:0000h
 |
```
