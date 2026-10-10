

# zoom1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum792.htm`](https://www.pediy.com/kssd/pediy03/forum792.htm)

谁能搞定此程序==>zoom(内详)，我吃方便面 N 天了!   
地址：http://www.peakstars.com/IMAGES/Zoom10.zip  573KB   
简介：屏幕放大镜，有点创意。   
      Magnify any portion of your screen.   
      Capture or continously grab screen areas.   
      Finally be able to measure screen distances.   
      Register Zoom：(Price: $9.99 plus shipping).   
保护：20 days trial。   
问题：一、程序安装好后再安装会有警告。希望大家在第一次安装时   
          用监测程序看看它在哪里作了手脚。   
      二、把日期来回乱改会有问题，反正我的有问题。ERD 恢复以前的注册信息无效。   
          不知程序在哪里作了手脚？   
          检查日期有几处？   
      三、在 20 天有效期内改程序可达到注册目的，但有效期过后改程序放大镜乱屏！   
      四、最好能找到注册码!   
  
:0044E84D E87A52FBFF              call 00403ACC   
:0044E852 83F806                  cmp eax, 00000006---->注册码为 6 位   
:0044E855 0F85BA020000            jne 0044EB15   
:0044E85B C745F4FFFFFFFF          mov \[ebp\-0C\], FFFFFFFF   
:0044E862 A18C8F4600              mov eax, dword ptr \[00468F8C\]   
:0044E867 8945E4                  mov dword ptr \[ebp\-1C\], eax   
:0044E86A 33C0                    xor eax, eax   
:0044E86C 55                      push ebp   
:0044E86D 680EEB4400              push 0044EB0E   
:0044E872 64FF30                  push dword ptr fs:\[eax\]   
:0044E875 648920                  mov dword ptr fs:\[eax\], esp   
:0044E878 B8288E4600              mov eax, 00468E28   
:0044E87D 8B55F8                  mov edx, dword ptr \[ebp\-08\]   
:0044E880 E81F50FBFF              call 004038A4   
:0044E885 33C0                    xor eax, eax   
:0044E887 55                      push ebp   
:0044E888 68C7EA4400              push 0044EAC7   
:0044E88D 64FF30                  push dword ptr fs:\[eax\]   
:0044E890 648920                  mov dword ptr fs:\[eax\], esp   
:0044E893 8B15408F4600            mov edx, dword ptr \[00468F40\]   
:0044E899 A1288E4600              mov eax, dword ptr \[00468E28\]   
:0044E89E FF15048E4600            call dword ptr \[00468E04\]   
:0044E8A4 8BD8                    mov ebx, eax   
:0044E8A6 891DF88D4600            mov dword ptr \[00468DF8\], ebx   
:0044E8AC A13C8E4600              mov eax, dword ptr \[00468E3C\]   
:0044E8B1 89581C                  mov dword ptr \[eax+1C\], ebx   
:0044E8B4 B8808D4600              mov eax, 00468D80   
:0044E8B9 E876EBFFFF              call 0044D434   
:0044E8BE 833D308E460002          cmp dword ptr \[00468E30\], 00000002---->关键对比   
:0044E8C5 7425                    je 0044E8EC   
:0044E8C7 6A00                    push 00000000   
  
\* Possible StringData Ref from Code Obj ->"Zoom"   
                                  |   
:0044E8C9 A130D34500              mov eax, dword ptr \[0045D330\]   
:0044E8CE E8BD53FBFF              call 00403C90   
:0044E8D3 50                      push eax   
  
\* Possible StringData Ref from Code Obj ->"Incorrect Key"   
                                  |   
:0044E8D4 6854EB4400              push 0044EB54   
:0044E8D9 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:0044E8DC 50                      push eax   
  
\* Reference To: user32.MessageBoxA, Ord:0000h   
                                  |   
:0044E8DD E87672FBFF              Call 00405B58   
:0044E8E2 33C0                    xor eax, eax   
:0044E8E4 8945F4                  mov dword ptr \[ebp\-0C\], eax   
:0044E8E7 E9D1010000              jmp 0044EABD   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044E8C5(C)   
|   
:0044E8EC 85F6                    test esi, esi   
:0044E8EE 740F                    je 0044E8FF   
:0044E8F0 A1F88D4600              mov eax, dword ptr \[00468DF8\]   
:0044E8F5 E896FCFFFF              call 0044E590   
:0044E8FA E9BE010000              jmp 0044EABD   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044E8EE(C)   
|   
:0044E8FF B201                    mov dl, 01   
  
\* Possible StringData Ref from Code Obj ->",罖"   
                                  |   
:0044E901 A19CAA4000              mov eax, dword ptr \[0040AA9C\]   
:0044E906 E83544FBFF              call 00402D40   
:0044E90B 8945EC                  mov dword ptr \[ebp\-14\], eax   
:0044E90E 33C0                    xor eax, eax   
:0044E910 8945E8                  mov dword ptr \[ebp\-18\], eax   
  
\* Possible StringData Ref from Data Obj ->"愅 D"   
                                  |   
:0044E913 C745E038D34500          mov \[ebp\-20\], 0045D338   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0044EA73(C)   
|   
:0044E91A 8B45E0                  mov eax, dword ptr \[ebp\-20\]   
:0044E91D 8B00                    mov eax, dword ptr \[eax\]   
:0044E91F E8A851FBFF              call 00403ACC   
:0044E924 85C0                    test eax, eax   
:0044E926 0F8E3C010000            jle 0044EA68   
  
......   
......   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0044E926(C), :0044EA59(C)   
|   
:0044EA68 FF45E8                  inc \[ebp\-18\]   
:0044EA6B 8345E004                add dword ptr \[ebp\-20\], 00000004   
:0044EA6F 837DE80B                cmp dword ptr \[ebp\-18\], 0000000B   
:0044EA73 0F85A1FEFFFF            jne 0044E91A   
:0044EA79 6A00                    push 00000000   
  
\* Possible StringData Ref from Code Obj ->"Zoom"   
                                  |   
:0044EA7B A130D34500              mov eax, dword ptr \[0045D330\]   
:0044EA80 E80B52FBFF              call 00403C90   
:0044EA85 50                      push eax   
:0044EA86 FF3530D34500            push dword ptr \[0045D330\]   
  
\* Possible StringData Ref from Code Obj ->" successfully registered !"   
                                  |   
:0044EA8C 6858EC4400              push 0044EC58   
:0044EA91 6894EB4400              push 0044EB94   
:0044EA96 8D45AC                  lea eax, dword ptr \[ebp\-54\]   
:0044EA99 BA03000000              mov edx, 00000003   
:0044EA9E E8E950FBFF              call 00403B8C   
:0044EAA3 8B45AC                  mov eax, dword ptr \[ebp\-54\]   
:0044EAA6 E8E551FBFF              call 00403C90   
:0044EAAB 50                      push eax   
:0044EAAC 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:0044EAAF 50                      push eax   
  
\* Reference To: user32.MessageBoxA, Ord:0000h   
                                  |   
:0044EAB0 E8A370FBFF              Call 00405B58   
  
  
                                      -=zest=-

破 zoom1.0！谢谢 blowfish 大侠援助！   
\================================================================   
一、日期限制的问题：   
  
0167:004078D0 E863DBFFFF      CALL    \`KERNEL32!GetLocalTime\`   
0167:004078D5 668B4C240E      MOV      CX,\[ESP+0E\]---->置日期   
0167:004078DA 668B54240A      MOV      DX,\[ESP+0A\]---->置月份   
0167:004078DF 668B442408      MOV      AX,\[ESP+08\]---->置年份   
                      ~~      ~~~~~~~~~~~~~~~~~~~~   
                    改 0A      MOV      AX,\[ESP+0A\]====>改置月份，其它可试试   
0167:004078E4 E86BFFFFFF      CALL    00407854   
0167:004078E9 DD1C24          FSTP    QWORD \[ESP\]   
0167:004078EC 9B              WAIT      
0167:004078ED DD0424          FLD      QWORD \[ESP\]   
0167:004078F0 83C418          ADD      ESP,BYTE +18   
0167:004078F3 C3              RET      
\----------------------------------------------------------------   
0167:00453813 E8201CFBFF      CALL    \`KERNEL32!GetLocalTime\`   
0167:00453818 668B4DDA        MOV      CX,\[EBP\-26\]---->置日期   
0167:0045381C 668B55D6        MOV      DX,\[EBP\-2A\]---->置月份   
0167:00453820 668B45D4        MOV      AX,\[EBP\-2C\]---->置年份   
                    ~~        ~~~~~~~~~~~~~~~~~~~~   
                  改 D6        MOV      AX,\[ESP\-2A\]====>改置月份，其它可试试   
0167:00453824 E82B40FBFF      CALL    00407854   
0167:00453829 DD5DF4          FSTP    QWORD \[EBP\-0C\]   
0167:0045382C 9B              WAIT      
0167:0045382D 8D45E4          LEA      EAX,\[EBP\-1C\]   
0167:00453830 BA48394500      MOV      EDX,00453948   
0167:00453835 E8AE00FBFF      CALL    004038E8   
0167:0045383A 6804010000      PUSH    DWORD 0104   
0167:0045383F 8D85CFFEFFFF    LEA      EAX,\[EBP+FFFFFECF\]   
0167:00453845 50              PUSH    EAX   
\================================================================   
为什么这样改，我也是雾水一头，试出来的。   
这的确是一个学破解的好软件。   
日期的保护都做得那么好，   
注册码的寻找就可想而知了。   
跟踪了一下，计算都颇复杂。   
\================================================================   
二、想显示注册成功，可改：   
  
0167:00454E7A 83B8B000000002  CMP      DWORD \[EAX+B0\],BYTE +02---->即\[00468E30\]见上篇   
0167:00454E81 0F85A8000000    JNZ      NEAR 00454F2F   
                ~~            ~~~~~~~~~~~~~~~~~~~~~~   
              改 84            JZ      NEAR 00454F2F   
0167:00454E87 33D2            XOR      EDX,EDX   
0167:00454E89 8B83F4010000    MOV      EAX,\[EBX+01F4\]   
0167:00454E8F E85C6DFCFF      CALL    0041BBF0   
0167:00454E94 33D2            XOR      EDX,EDX   
0167:00454E96 8B832C020000    MOV      EAX,\[EBX+022C\]   
0167:00454E9C E84F6DFCFF      CALL    0041BBF0   
0167:00454EA1 B201            MOV      DL,01   
0167:00454EA3 8B83F0010000    MOV      EAX,\[EBX+01F0\]   
\================================================================   
  
※我至今无法知道程序安装时间存放在什么地方。   
  哪位知道了请一定贴出来噢！   
  谢谢！   
  
  
  
                                      -=zest=-  2001.4 

在这个 dword 的前后左右就是用来算注册码的一些表格。  4dd466d0e1e0cc123afbe504da2462ca