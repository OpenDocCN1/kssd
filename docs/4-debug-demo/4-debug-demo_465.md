

# spider

> 原文：[`www.pediy.com/kssd/pediy06/pediy6224.htm`](https://www.pediy.com/kssd/pediy06/pediy6224.htm)

标题:windows XP 自带游戏"蜘蛛牌--spider"空位不发牌限制的破解   
作者:FTBirthday   
工具:olly   
  
我在玩 windows XP 自带游戏"蜘蛛牌"时,遇到有空位不发牌限制,于是   
想破了它,开始吧.   
  
设断 MessageBoxExW,然后创造一次空位发牌的动作.   
0006EF58   77D4885D  /CALL to MessageBoxExW from USER32.77D48858   
0006EF5C   00060298  |hOwner = 00060298 ('蜘蛛',class='蜘蛛')7D48   
0006EF60   010107C0  |Text = "有空位时不"298 (   
0006EF64   0006EF84  |Title = "蜘?时不   
0006EF68   00000000  |Style = MB\_OK|MB\_APPLMODAL   
0006EF6C   00000000  LanguageID = 0 (LANG\_NEUTRAL)   
0006EF70   010034E9  返回到 spider.010034E9 来自 USER32.MessageBoxW58   
  
77D48860 > 55               PUSH EBP   
77D48861   8BEC             MOV EBP,ESP   
77D48863   6A FF            PUSH -1   
77D48865   FF75 18          PUSH DWORD PTR SS:\[EBP+18\]   
77D48868   FF75 14          PUSH DWORD PTR SS:\[EBP+14\]   
77D4886B   FF75 10          PUSH DWORD PTR SS:\[EBP+10\]   
77D4886E   FF75 0C          PUSH DWORD PTR SS:\[EBP+C\]   
77D48871   FF75 08          PUSH DWORD PTR SS:\[EBP+8\]   
77D48874   E8 FA24FFFF      CALL USER32.MessageBoxTimeoutW   
77D48879   5D               POP EBP   
77D4887A   C2 1400          RETN 14   
  
010034AE  /$ 55             PUSH EBP   
010034AF  |. 8BEC           MOV EBP,ESP   
010034B1  |. 81EC 00080000  SUB ESP,800   
010034B7  |. FF75 10        PUSH DWORD PTR SS:\[EBP+10\]   
010034BA  |. E8 CDFFFFFF    CALL spider.0100348C   
010034BF  |. 50             PUSH EAX                                 ; /String2   
010034C0  |. 8D85 00F8FFFF  LEA EAX,DWORD PTR SS:\[EBP\-800\]           ; |   
010034C6  |. 50             PUSH EAX                                 ; |String1   
010034C7  |. FF15 54110001  CALL DWORD PTR DS:\[<&KERNEL32.lstrcpyW>\] ; lstrcpyW   
010034CD  |. FF75 14        PUSH DWORD PTR SS:\[EBP+14\]   
010034D0  |. 8D85 00F8FFFF  LEA EAX,DWORD PTR SS:\[EBP\-800\]   
010034D6  |. 50             PUSH EAX   
010034D7  |. FF75 0C        PUSH DWORD PTR SS:\[EBP+C\]   
010034DA  |. E8 ADFFFFFF    CALL spider.0100348C   
010034DF  |. 50             PUSH EAX                                 ; |Text   
010034E0  |. FF75 08        PUSH DWORD PTR SS:\[EBP+8\]                ; |hOwner   
010034E3  |. FF15 90110001  CALL DWORD PTR DS:\[<&USER32.MessageBoxW>>; MessageBoxW   
010034E9  |. C9             LEAVE   
010034EA  . C2 1000        RETN 10   
  
0006F788   0100662D  返回到 spider.0100662D 来自 spider.010034AE   
  
01006604  /$ 83EC 24        SUB ESP,24   
01006607  |. 56             PUSH ESI   
01006608  |. 8BF1           MOV ESI,ECX   
0100660A  |. 837E 58 05     CMP DWORD PTR DS:\[ESI+58\],5   
0100660E  |. 0F8D FB010000  JGE spider.0100680F         <-----正常情况下,这个不跳   
01006614  |. 8B4E 08        MOV ECX,DWORD PTR DS:\[ESI+8\]   
01006617  |. E8 1D120000    CALL spider.01007839   
0100661C  |. 85C0           TEST EAX,EAX   
0100661E  |. 74 12          JE SHORT spider.01006632     <-----正常情况下,这个跳,这个才是关键的 call   
01006620  |. 6A 00          PUSH 0                                   ; /Arg4 = 00000000   
01006622  |. 6A 02          PUSH 2                                   ; |Arg3 = 00000002   
01006624  |. 6A 05          PUSH 5                                   ; |Arg2 = 00000005   
01006626  |. FF36           PUSH DWORD PTR DS:\[ESI\]                  ; |Arg1   
01006628  |. E8 81CEFFFF    CALL spider.010034AE                     ; spider.010034AE   
0100662D  |. E9 DD010000    JMP spider.0100680F   
01006632  |> 53             PUSH EBX   
01006633  |. 55             PUSH EBP   
  
修改如下   
01006604  /$ 83EC 24        SUB ESP,24   
01006607  |. 56             PUSH ESI   
01006608  |. 8BF1           MOV ESI,ECX   
0100660A  |. 837E 58 05     CMP DWORD PTR DS:\[ESI+58\],5   
0100660E  |. 0F8D FB010000  JGE spider.0100680F   
01006614  |. 8B4E 08        MOV ECX,DWORD PTR DS:\[ESI+8\]   
01006617  |. E8 1D120000    CALL spider.01007839   
0100661C  |. 85C0           TEST EAX,EAX   
0100661E     EB 12          JMP SHORT spider.01006632  
 4dd466d0e1e0cc123afbe504da2462ca