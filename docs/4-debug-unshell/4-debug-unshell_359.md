

# mmxbricksht

> 原文：[`www.pediy.com/kssd/pediy01/forum093.htm`](https://www.pediy.com/kssd/pediy01/forum093.htm)

这是一个游戏程序，即防 ICE 又防 TRW，而且还加了壳，我真是拿它没办法!   
软件下载：http://go.163.com/~szycool/crackme/mmxbricksht.zip 

普通的 Asprotect 保护，你用 TRW2000 前先：faults off。Import 表就在 idata 里：464000.你也可按我以前的方法定位，得到结果一至。今天我要休息，到此为止。 

 前面的代码每次不一样，你只能以机器码和后面的指令来参考。

 :bpx loadlibrarya  
  
0137:005693D8 EB13              JMP    005693ED   
0137:005693DA E9A907FFFF        JMP    00559B88   
0137:005693DF 66B80E00          MOV    AX,000E   
0137:005693E3 E814F2FFFF        CALL    005685FC   
0137:005693E8 E80709FFFF        CALL    00559CF4   
0137:005693ED 8B4508            MOV    EAX,\[EBP+08\]   
0137:005693F0 8D4818            LEA    ECX,\[EAX+18\]   
0137:005693F3 8B4508            MOV    EAX,\[EBP+08\]   
0137:005693F6 8B10              MOV    EDX,\[EAX\]   
0137:005693F8 8B4508            MOV    EAX,\[EBP+08\]   
0137:005693FB 8B401C            MOV    EAX,\[EAX+1C\]   
0137:005693FE E881F7FFFF        CALL    00568B84—————进去   
0137:00569403 5F                POP    EDI   
0137:00569404 5E                POP    ESI   
0137:00569405 5B                POP    EBX   
0137:00569406 59                POP    ECX   
0137:00569407 59                POP    ECX   
0137:00569408 5D                POP    EBP   
  
来到：   
0137:005652CC 61                POPAD   
0137:005652CD EB01              JMP    005652D0   
0137:005652CF E850EB02E9        CALL    E9593E24   
0137:005652D4 17                POP    SS   
0137:005652D5 E802000000        CALL    005652DC———进去   
0137:005652DA E91758C35B        JMP    5C19AAF6   
0137:005652DF 59                POP    ECX   
0137:005652E0 59                POP    ECX   
0137:005652E1 5D                POP    EBP   
  
来到：   
0137:0045FA1C 55                PUSH    EBP—————入口点   
0137:0045FA1D 8BEC              MOV    EBP,ESP   
0137:0045FA1F 83C4F4            ADD    ESP,FFFFFFF4   
0137:0045FA22 53                PUSH    EBX   
0137:0045FA23 B87CF84500        MOV    EAX,0045F87C   
0137:0045FA28 E83765FAFF        CALL    00405F64   
0137:0045FA2D 8B1D7C0F4600      MOV    EBX,\[00460F7C\]   
0137:0045FA33 8B03              MOV    EAX,\[EBX\]   
0137:0045FA35 E8EA58FEFF        CALL    00445324   
0137:0045FA3A 8B03              MOV    EAX,\[EBX\]   
0137:0045FA3C BA00FB4500        MOV    EDX,0045FB00   
0137:0045FA41 E80255FEFF        CALL    00444F48   
0137:0045FA46 8B0DAC0E4600      MOV    ECX,\[00460EAC\] 

程序入口地址：0045fa1c   
需要重建 import 表 RAV:00064000    
  
  
先用 bpx loadlibrarya do "dd\*(esp+4)"   
两三下后，数据区显示为 KERNEL32.DLL   
此时 dump 取 Import 表。   
w 64000 l 3000   
  
来到入口点 0045fa1c dump 取整个程序。   
然后用 Procdump 打开这个程序，看到 idata 的 RAW office=60400 RAW size=221c   
用 Hex Workshop 打开刚 dump 的 Import 表，选取大小 221c.替换到主程序的 60400 处，大小 221c,并修正 Import 表的值为 64000   
这样程序完全可运行。  4dd466d0e1e0cc123afbe504da2462ca