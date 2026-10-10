

# SsfeDisc 2.x 高速脫殼

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-620.htm`](https://www.pediy.com/kssd/pediy07/pediy7-620.htm)

目的:單純研究,討論用....好久以前的筆記了..=.='  
  
目標: ->SD.EXE  
  
(1) 使用工具,查看,發現為  
  
 SsfeDisc 2.x  
  
(2)開始 S-ICE ,設斷點 BPX GetVersion  
  
(3)返回主程序後,找到 OEP 入口  
  
482302-PUSH   EBP            <------OEP  
482303-MOV    EBP,ESP  
482305-PUSH   FF  
482307-PUSH   00466257  
...  
...  
482302A-CALL \[006F4320\] --->GetVersion  
  
(4)MAP32 DPLAYERX.DLL ,SAFEDISC==>解碼 API   
     
    12BF4CB-61 75 0F -->改 JMP PATCH   
    12BF4F1-EB 09 17 -->改 JMP PATCH   
  
(5)對應代碼 0 =7D ,1 =23 (查看 MEMORY) ; U 120A270 (起點)                             
  
(6)找空白區,寫入 PATCH ,還原 Import table  
    注意,代碼 0,代碼 1 各要 RUN 1 次,才能 OK   
  
PATCH:  
LOOP:   PUSH 9413FFBE  
        PUSH EBX   
        PUSH DWORD PTR 0       ;代碼=0  
        CALL \[0013AF86\]        ;呼叫 解碼 API  
        ADD ECX,EDX            ;ECX=搜尋值  
        MOV EAX,\[EBP+10\]       ;真正 API 位址  
        MOV EDX,6F4000         ;IAT START   
N0:     CMP DWORD PTR\[EDX\],ECX  
        JZ   N1  
        INC EDX  
        CMP EDX,6F6000          ;IAT END  
        JZ   N2  
        JMP  N0   
N1:     MOV DWORD PTR\[EDX\],EAX  
N2:     INC EBX  
        CMP EBX,7D               ;對應代碼==>7D  
        JNZ LOOP  
END:    JMP END        
  
(7) RUN LordPE =>DUMP FULL ==>SD.EXE 存檔 SD1.EXE  
  
(8) RUN ImportREC  輸入 OEP 修正 IAT ,存檔  SD1.EXE  
  
(9) RUN SD1.EXE TEST...OK~!!!! 收工 

> 引用:
> 
> * * *
> 
> *最初由 无聊的菜鸟 发布*  
> **不过我这里并没有 Dplayerx.dll，而且没看懂。没找到 CC 修复。不知道你的 CALL \[0013AF86\]究竟是到什么地方去了。 **
> 
> * * *

  
  
不好意識,這個應該是 SafeDisc 2.x 版,的方法....  
  好像 2.5 版之前,都有  Dplayerx.dll,這個檔  
  之後的版本 ,這個檔,就合併在 .exe 了...  
  
CALL \[0013AF86\] <--到 call safedisc 解碼 api 程序  
  
 SafeDisc 3.x ,的解法,和 2.x ,不太一樣  
  
  必須要修正和重建,所有條用 api 函式的主程序 and new import table 

 4dd466d0e1e0cc123afbe504da2462ca