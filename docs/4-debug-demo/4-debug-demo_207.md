

# 计算机资料大全 3.0 Overlay 反脱壳去 NAG

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-503.htm`](https://www.pediy.com/kssd/pediy07/pediy7-503.htm)

【脱文标题】 计算机资料大全 3.0 Overlay 反脱壳分析+Ctrl+F8 智取 NAG  
【破文作者】 weiyi75\[Dfcg\]  
【作者邮箱】 weiyi75@sohu.com  
【作者主页】 Dfcg 官方大本营  
【使用工具】 Peid,UpxShell,olldbg,winhex  
【破解平台】 WinNT/2000/XP  
【软件名称】 计算机资料大全 3.0   
【下载页面】 [`downloads.daqing.net/`](http://downloads.daqing.net/)   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
  
【破解分析】  
  
首先 Peid 查壳，为 UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo \[Overlay\],注意到\[Overlay\],然后用 Upxshell 轻松脱壳,正常运行。  
  
幸运的是它是个普通 Upx 壳，我们假设它是一个变形 Upx 壳。  
  
004A1430 >  60              PUSHAD    //Upx 外壳入口  
004A1431    BE 00804600     MOV ESI,《计算机.00468000  
004A1436    8DBE 0090F9FF   LEA EDI,DWORD PTR DS:\[ESI+FFF99000\]  
004A143C    C787 10270800 1>MOV DWORD PTR DS:\[EDI+82710\],F82C9510  
004A1446    57              PUSH EDI  
004A1447    83CD FF         OR EBP,FFFFFFFF  
004A144A    EB 0E           JMP SHORT 《计算机.004A145A  
004A144C    90              NOP  
.........................................................  
  
直接 Ctrl+F 向下找 Popad  
  
004A159A    61              POPAD    
004A159B  ^ E9 64DEFDFF     JMP 《计算机.0047F404  //直接 F4 下来。  
  
0047F404    55              PUSH EBP  //到达 Oep,Loadpe 脱壳。  
0047F405    8BEC            MOV EBP,ESP  
0047F407    83C4 F0         ADD ESP,-10  
0047F40A    53              PUSH EBX  
0047F40B    B8 A4F14700     MOV EAX,《计算机.0047F1A4  
0047F410    E8 CF76F8FF     CALL 《计算机.00406AE4  
0047F415    33C0            XOR EAX,EAX  
0047F417    55              PUSH EBP  
0047F418    68 53F54700     PUSH 《计算机.0047F553  
0047F41D    64:FF30         PUSH DWORD PTR FS:\[EAX\]  
0047F420    64:8920         MOV DWORD PTR FS:\[EAX\],ESP  
0047F423    E8 146CFEFF     CALL 《计算机.0046603C  
0047F428    BA 143E4800     MOV EDX,《计算机.00483E14  
0047F42D    B8 68F54700     MOV EAX,《计算机.0047F568                    ; ASCII "XSEI.NATC"  
0047F432    E8 216EFEFF     CALL 《计算机.00466258  
0047F437    833D 143E4800 0>CMP DWORD PTR DS:\[483E14\],0  
0047F43E    74 04           JE SHORT 《计算机.0047F444  
.................................................................  
  
运行 ImportREC，选择这个进程。把 OEP 改为 7F404，点 IT AutoSearch，点“Get Import”，看到输入表全部有效针，FixDump。      
  
运行前看到加壳程序 13.2M,脱壳程序 668KB,肯定是无法运行的,这个是因为加壳程序尾部的数据没有复制到脱壳程序里面。  
  
用 Winhex 打开加壳程序。  
  
我们看到尾部数据从偏移量 987 开始。在 987 处点右键选块开始。  
  
然后滚动条拉到底，到偏移量 13889591 为止，在 13889591 处点右键选块结束。  
  
Crtl+C 复制选定内容。  
  
然后打开脱壳文件，直接滚动条到尾部偏移量 684031 处，点右键-编辑-剪贴板数据写入,确定两次，原理搞清楚就不犹豫,保存文件，运行正常。  
  
短短的三次试用一会就过期了，先前没有用 RegSnap 给这个程序照相。  
  
破解 NAG 吧。  
  
下断点  
  
Bp MessageBoxA  
  
无法中断 NAG,没办法  
  
OD 载入程序  
  
0047F404 > $  55            PUSH EBP  //入口点。  
0047F405   .  8BEC          MOV EBP,ESP  
0047F407   .  83C4 F0       ADD ESP,-10  
0047F40A   .  53            PUSH EBX  
0047F40B   .  B8 A4F14700   MOV EAX,《计算机.0047F1A4  
0047F410   .  E8 CF76F8FF   CALL 《计算机.00406AE4  
0047F415   .  33C0          XOR EAX,EAX  
0047F417   .  55            PUSH EBP  
.................................................  
  
这里想起风飘雪老师的加密和解密的实战功略里面对付 NAG 的方法。  
  
那就是必杀 Ctrl+F8  
  
简单解释一下，Ctrl+F8 自动步过，遇到 Call 中 Call，然后 Nag 调用运行。  
  
所以遇到 Call 停止后就下断点，不断 Ctrl+F2 和 F9 Ctrl+F8 三键灵活使用。  
  
0045A5D8   .  E8 0396FAFF   CALL 《计算机.00403BE0   //这个 Call 要调用两次。  
  
0047F51A   .  E8 9125FEFF   CALL 《计算机.00461AB0  //第一次 NAG,下断点重启再次中断,F7 跟进，Ctrl+F8 到下一个 Call NAG 出现。  
  
00461AB0   $  55            PUSH EBP      //跟进这里，Ctrl+F8  
00461AB1   .  8BEC          MOV EBP,ESP  
00461AB3   .  51            PUSH ECX  
00461AB4   .  53            PUSH EBX  
00461AB5   .  56            PUSH ESI  
00461AB6   .  57            PUSH EDI  
00461AB7   .  894D FC       MOV DWORD PTR SS:\[EBP\-4\],ECX  
00461ABA   .  8BDA          MOV EBX,EDX  
00461ABC   .  8BF0          MOV ESI,EAX  
00461ABE   .  8BC3          MOV EAX,EBX  
  
00461AE1   .  FF57 2C       CALL DWORD PTR DS:\[EDI+2C\]           //第二次 NAG,下断点重启再次中断,F7 跟进  
  
0045A464   $  55            PUSH EBP    //跟进这里，Ctrl+F8  
0045A465   .  8BEC          MOV EBP,ESP  
0045A467   .  81C4 ECFEFFFF ADD ESP,-114  
0045A46D   .  53            PUSH EBX  
0045A46E   .  33DB          XOR EBX,EBX  
0045A470   .  899D ECFEFFFF MOV DWORD PTR SS:\[EBP\-114\],EBX  
0045A476   .  84D2          TEST DL,DL  
0045A478   .  74 08         JE SHORT 《计算机.0045A482  
0045A47A   .  83C4 F0       ADD ESP,-10  
0045A47D   .  E8 0697FAFF   CALL 《计算机.00403B88  
0045A482   >  8BD9          MOV EBX,ECX  
  
0045A5D8   .  E8 0396FAFF   CALL 《计算机.00403BE0           //第三次 NAG,下断点重启再次中断,F7 跟进  
  
0047BA46   .  E8 2D200000   CALL 《计算机.0047DA78          //第四次 NAG,下断点重启再次中断,F7 跟进  
  
0047DD13   .  E8 F8D1FBFF   CALL 《计算机.0043AF10         //第五次 NAG,向上找到 Magic jmp  
  
整个过程就是吃卷心菜，一层层拨开。  
  
0047DC4B   .  3C 30         CMP AL,30    //千辛万苦找到的 Magic jmp，大家可以看 OD 的跳转线和单步跟踪最后一次 Call 的全部过程确定这是爆破点,必须跳转，刚好跳出死亡 Call。  
0047DC4D   .  0F84 97010000 JE 《计算机.0047DDEA  
0047DC53   .  3C 31         CMP AL,31  
0047DC55   .  75 45         JNZ SHORT 《计算机.0047DC9C  
0047DC57   .  E8 68CAF8FF   CALL 《计算机.0040A6C4  
0047DC5C   .  83C4 F8       ADD ESP,-8                               ; /  
0047DC5F   .  DD1C24        FSTP QWORD PTR SS:\[ESP\]                  ; |Arg3 (8-byte)  
0047DC62   .  9B            WAIT                                     ; |  
0047DC63   .  FF75 FC       PUSH DWORD PTR SS:\[EBP\-4\]                ; |Arg2  
0047DC66   .  FF75 F8       PUSH DWORD PTR SS:\[EBP\-8\]                ; |Arg1  
0047DC69   .  E8 72C8FFFF   CALL 《计算机.0047A4E0                       ; \\《计算机.0047A4E0  
0047DC6E   .  3B05 9C3D4800 CMP EAX,DWORD PTR DS:\[483D9C\]  
0047DC74   .  7E 26         JLE SHORT 《计算机.0047DC9C  
0047DC76   .  6A 01         PUSH 1  
0047DC78   .  8D45 CC       LEA EAX,DWORD PTR SS:\[EBP\-34\]  
0047DC7B   .  8B0D C03D4800 MOV ECX,DWORD PTR DS:\[483DC0\]  
0047DC81   .  8B15 D03D4800 MOV EDX,DWORD PTR DS:\[483DD0\]  
0047DC87   .  E8 086CF8FF   CALL 《计算机.00404894  
0047DC8C   .  8B55 CC       MOV EDX,DWORD PTR SS:\[EBP\-34\]  
0047DC8F   .  B9 50DF4700   MOV ECX,《计算机.0047DF50                    ;  ASCII "eje"  
0047DC94   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]  
0047DC97   .  E8 2877FFFF   CALL 《计算机.004753C4  
0047DC9C   >  A1 C43D4800   MOV EAX,DWORD PTR DS:\[483DC4\]  
0047DCA1   .  8078 14 32    CMP BYTE PTR DS:\[EAX+14\],32  
0047DCA5   .  75 2E         JNZ SHORT 《计算机.0047DCD5  
0047DCA7   .  3B35 A03D4800 CMP ESI,DWORD PTR DS:\[483DA0\]  
0047DCAD   .  7E 26         JLE SHORT 《计算机.0047DCD5  
0047DCAF   .  6A 01         PUSH 1  
0047DCB1   .  8D45 C8       LEA EAX,DWORD PTR SS:\[EBP\-38\]  
0047DCB4   .  8B0D C03D4800 MOV ECX,DWORD PTR DS:\[483DC0\]  
0047DCBA   .  8B15 D03D4800 MOV EDX,DWORD PTR DS:\[483DD0\]  
0047DCC0   .  E8 CF6BF8FF   CALL 《计算机.00404894  
0047DCC5   .  8B55 C8       MOV EDX,DWORD PTR SS:\[EBP\-38\]  
0047DCC8   .  B9 50DF4700   MOV ECX,《计算机.0047DF50                    ;  ASCII "eje"  
0047DCCD   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]  
0047DCD0   .  E8 EF76FFFF   CALL 《计算机.004753C4  
0047DCD5   >  6A 00         PUSH 0  
0047DCD7   .  8D45 C4       LEA EAX,DWORD PTR SS:\[EBP\-3C\]  
0047DCDA   .  8B0D C03D4800 MOV ECX,DWORD PTR DS:\[483DC0\]  
0047DCE0   .  8B15 D03D4800 MOV EDX,DWORD PTR DS:\[483DD0\]  
0047DCE6   .  E8 A96BF8FF   CALL 《计算机.00404894  
0047DCEB   .  8B55 C4       MOV EDX,DWORD PTR SS:\[EBP\-3C\]            ; |  
0047DCEE   .  B9 50DF4700   MOV ECX,《计算机.0047DF50                    ; |ASCII "eje"  
0047DCF3   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]             ; |  
0047DCF6   .  E8 8576FFFF   CALL 《计算机.00475380                       ; \\《计算机.00475380  
0047DCFB   .  84C0          TEST AL,AL  
0047DCFD   .  0F84 E7000000 JE 《计算机.0047DDEA  
0047DD03   .  6A 00         PUSH 0                                   ; /Arg1 = 00000000  
0047DD05   .  66:8B0D 54DF4>MOV CX,WORD PTR DS:\[47DF54\]              ; |  
0047DD0C   .  B2 02         MOV DL,2                                 ; |  
0047DD0E   .  A1 B43D4800   MOV EAX,DWORD PTR DS:\[483DB4\]            ; |  
0047DD13   .  E8 F8D1FBFF   CALL 《计算机.0043AF10                       ; \\《计算机.0043AF10  
.................................................................................  
  
继续努力搞退出 NAG。  
  
经龙哥指点  
  
退出断点下  
  
bp **DestroyWindow**  
  
**DestroyWindow**  用途是  
  
破坏（即清除）指定的窗口以及它的所有子窗口（在 vb 里使用：用处不大。原文：it is unlikely to be of much use.）一般程序退出时调用或关闭窗口时调用。  
  
点不看了按钮,立即中断。  
  
77DF736E >  B8 59110000     MOV EAX,1159  //这里中断。  
77DF7373    8D5424 04       LEA EDX,DWORD PTR SS:\[ESP+4\]  
77DF7377    CD 2E           INT 2E  //如果返回看到这样的语句，必须 Nop 掉，不然 Over  
77DF7379    C2 0400         RETN 4  
  
堆栈友好提示  
  
0012F9DC   00445769  /CALL 到 DestroyWindow 来自 Crack.00445764   //这里清除断点,Ctrl+F9 返回。  
0012F9E0   001104A0  \\hWnd = 001104A0 ('计算机资料大全 v3.0  火力加强版',class='TMessageForm',parent=000404BC)  
0012F9E4   0012FBCC  指针到下一个 SEH 记录  
0012F9E8   00445789  SE 句柄  
0012F9EC   0012F9F4  
  
00445769   .  85C0          TEST EAX,EAX  
0044576B   .  75 05         JNZ SHORT Crack.00445772  
0044576D   .  E8 A283FCFF   CALL Crack.0040DB14  
00445772   >  33C0          XOR EAX,EAX  
00445774   .  5A            POP EDX  
00445775   .  59            POP ECX  
00445776   .  59            POP ECX  
00445777   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
0044577A   .  68 90574400   PUSH Crack.00445790  
0044577F   >  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
00445782   .  66:8160 54 FF>AND WORD PTR DS:\[EAX+54\],0FDFF  
00445788   .  C3            RETN         
  
不断 Ctrl+F9 返回。  
  
0043AFCF  |> \\8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]  
0043AFD2  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
0043AFD4  |.  FF92 E8000000 CALL DWORD PTR DS:\[EDX+E8\]  //F7 进入。  
  
0045E754  /.  55            PUSH EBP   //进这里。  
0045E755  |.  8BEC          MOV EBP,ESP  
0045E757  |.  83C4 E0       ADD ESP,-20  
0045E75A  |.  53            PUSH EBX  
0045E75B  |.  56            PUSH ESI  
0045E75C  |.  33D2          XOR EDX,EDX  
0045E75E  |.  8955 E0       MOV DWORD PTR SS:\[EBP\-20\],EDX  
0045E761  |.  8945 FC       MOV DWORD PTR SS:\[EBP\-4\],EAX  
0045E764  |.  BB 2C3C4800   MOV EBX,Crack.00483C2C  
0045E769  |.  33C0          XOR EAX,EAX  
0045E76B  |.  55            PUSH EBP  
0045E76C  |.  68 E4E94500   PUSH Crack.0045E9E4  
0045E771  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
0045E774  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
0045E777  |.  E8 481CFEFF   CALL Crack.004403C4  
0045E77C  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E77F  |.  8078 57 00    CMP BYTE PTR DS:\[EAX+57\],0  
0045E783  |.  75 24         JNZ SHORT Crack.0045E7A9  
0045E785  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E788  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
0045E78A  |.  FF52 50       CALL DWORD PTR DS:\[EDX+50\]  
0045E78D  |.  84C0          TEST AL,AL  
0045E78F  |.  74 18         JE SHORT Crack.0045E7A9  
0045E791  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E794  |.  F680 EC020000>TEST BYTE PTR DS:\[EAX+2EC\],8  
0045E79B  |.  75 0C         JNZ SHORT Crack.0045E7A9  
0045E79D  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E7A0  |.  80B8 2F020000>CMP BYTE PTR DS:\[EAX+22F\],1  
0045E7A7  |.  75 21         JNZ SHORT Crack.0045E7CA  
  
0045E7CA  |>  E8 698AFAFF   CALL <JMP.&user32.GetCapture>            ; \[GetCapture  
0045E7CF  |.  85C0          TEST EAX,EAX  
0045E7D1  |.  74 11         JE SHORT Crack.0045E7E4  
  
0045E7E4  |>  E8 D78CFAFF   CALL <JMP.&user32.ReleaseCapture>        ; \[ReleaseCapture  
0045E7E9  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E7EC  |.  8088 EC020000>OR BYTE PTR DS:\[EAX+2EC\],8  
0045E7F3  |.  E8 388AFAFF   CALL <JMP.&user32.GetActiveWindow>       ; \[GetActiveWindow  
0045E7F8  |.  8945 E4       MOV DWORD PTR SS:\[EBP\-1C\],EAX  
0045E7FB  |.  A1 9C0D4800   MOV EAX,DWORD PTR DS:\[480D9C\]  
0045E800  |.  8945 F0       MOV DWORD PTR SS:\[EBP\-10\],EAX  
0045E803  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E808  |.  8B48 78       MOV ECX,DWORD PTR DS:\[EAX+78\]  
0045E80B  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E810  |.  8B40 7C       MOV EAX,DWORD PTR DS:\[EAX+7C\]  
0045E813  |.  33D2          XOR EDX,EDX  
0045E815  |.  E8 46CAFBFF   CALL Crack.0041B260  
0045E81A  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E81F  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP\-4\]  
0045E822  |.  8950 78       MOV DWORD PTR DS:\[EAX+78\],EDX  
0045E825  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E82A  |.  66:8B40 44    MOV AX,WORD PTR DS:\[EAX+44\]  
0045E82E  |.  66:8945 EE    MOV WORD PTR SS:\[EBP\-12\],AX  
0045E832  |.  33D2          XOR EDX,EDX  
0045E834  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E839  |.  E8 7E130000   CALL Crack.0045FBBC  
0045E83E  |.  A1 303C4800   MOV EAX,DWORD PTR DS:\[483C30\]  
0045E843  |.  8B40 48       MOV EAX,DWORD PTR DS:\[EAX+48\]  
0045E846  |.  8945 E8       MOV DWORD PTR SS:\[EBP\-18\],EAX  
0045E849  |.  33C0          XOR EAX,EAX  
0045E84B  |.  E8 E4A2FFFF   CALL Crack.00458B34  
0045E850  |.  8945 F4       MOV DWORD PTR SS:\[EBP\-C\],EAX  
0045E853  |.  33D2          XOR EDX,EDX  
0045E855  |.  55            PUSH EBP  
0045E856  |.  68 C4E94500   PUSH Crack.0045E9C4  
0045E85B  |.  64:FF32       PUSH DWORD PTR FS:\[EDX\]  
0045E85E  |.  64:8922       MOV DWORD PTR FS:\[EDX\],ESP  
0045E861  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E864  |.  E8 3BFEFFFF   CALL Crack.0045E6A4  
0045E869  |.  33D2          XOR EDX,EDX  
0045E86B  |.  55            PUSH EBP  
0045E86C  |.  68 23E94500   PUSH Crack.0045E923  
0045E871  |.  64:FF32       PUSH DWORD PTR FS:\[EDX\]  
0045E874  |.  64:8922       MOV DWORD PTR FS:\[EDX\],ESP  
0045E877  |.  6A 00         PUSH 0  
0045E879  |.  6A 00         PUSH 0  
0045E87B  |.  68 00B00000   PUSH 0B000  
0045E880  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E883  |.  E8 D499FEFF   CALL Crack.0044825C  
0045E888  |.  50            PUSH EAX                                 ; |hWnd  
0045E889  |.  E8 728CFAFF   CALL <JMP.&user32.SendMessageA>          ; \\SendMessageA  
0045E88E  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E891  |.  33D2          XOR EDX,EDX  
0045E893  |.  8990 4C020000 MOV DWORD PTR DS:\[EAX+24C\],EDX  
0045E899  |>  8B03          /MOV EAX,DWORD PTR DS:\[EBX\]     --------------------------  
0045E89B  |.  E8 FC300000   |CALL Crack.0046199C  
0045E8A0  |.  8B03          |MOV EAX,DWORD PTR DS:\[EBX\]  
0045E8A2  |.  80B8 9C000000>|CMP BYTE PTR DS:\[EAX+9C\],0  
0045E8A9  |.  74 0F         |JE SHORT Crack.0045E8BA  
  
0045E8BA  |>  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP\-4\]      里面构成一个循环，在循环里面 NAG 出现了，只能人为关闭。  
0045E8BD  |.  83B8 4C020000>|CMP DWORD PTR DS:\[EAX+24C\],0  
0045E8C4  |.  74 08         |JE SHORT Crack.0045E8CE  
  
0045E8CE  |>  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP\-4\]  
0045E8D1  |.  8B80 4C020000 |MOV EAX,DWORD PTR DS:\[EAX+24C\]  
0045E8D7  |.  85C0          |TEST EAX,EAX  
0045E8D9  |.^ 74 BE         \\JE SHORT Crack.0045E899       ----------------------------  
  
//人为关闭同样是到达下面一句，如果爆破的话，NAG 窗口在你眼睛没看见的时候就关闭了，我的 P4 2G 的 Cpu 我根本没看见 NAG,这里就是第二个爆破点。  
  
0045E8DB  |.  8945 F8       MOV DWORD PTR SS:\[EBP\-8\],EAX  
0045E8DE  |.  6A 00         PUSH 0  
0045E8E0  |.  6A 00         PUSH 0  
0045E8E2  |.  68 01B00000   PUSH 0B001  
...............................................// 往下正常 Over.  
  
【破解总结】  
  
0047DC4D   .  0F84 97010000 JE 0047DDEA  
          
              修改为  
  
0047DC4D   .  0F85 97010000 JnE 0047DDEA  
  
  
0045E8D9  |.^ 74 BE         \\JE SHORT Crack.0045E899    
  
               修改为  
                           
0045E8D9  |.^ 75 BE         \\JNE SHORT Crack.0045E899       
  
第二处强力爆破后，启动 Nag 一起搞调了，原来它们是调用的同一个语句，程序帮我们 Kill 掉了(自动关闭)  
  
最后请看胜利截图。  
  
  
\--------------------------------------------------------------------------------    
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!  
本代码的着色效果由 xTiNt 自动完成  
下载 xTiNt [`211.90.75.84/web/kanaun/download/xTiNt.rar`](http://211.90.75.84/web/kanaun/download/xTiNt.rar) 

> 引用:
> 
> * * *
> 
> *最初由 ljy3282393 发布*  
> **二哥：下 bp DestroyWindow 断点后，应该按多少次 Ctrl+F9 ?为什么? **
> 
> * * *

  
  
退出断点下  
  
bp DestroyWindow  
  
DestroyWindow  用途是  
  
破坏（即清除）指定的窗口以及它的所有子窗口（在 vb 里使用：用处不大。原文：it is unlikely to be of much use.）一般程序退出时调用或关闭窗口时调用。  
  
点不看了按钮,立即中断。  
  
77DF736E >  B8 59110000     MOV EAX,1159  //这里中断。  
77DF7373    8D5424 04       LEA EDX,DWORD PTR SS:\[ESP+4\]  
77DF7377    CD 2E           INT 2E  //如果返回看到这样的语句，必须 Nop 掉，不然 Over  
77DF7379    C2 0400         RETN 4  
  
堆栈友好提示  
  
0012F9DC   00445769  /CALL 到 DestroyWindow 来自 Crack.00445764   //这里清除断点,Ctrl+F9 返回。  
0012F9E0   001104A0  \\hWnd = 001104A0 ('计算机资料大全 v3.0  火力加强版',class='TMessageForm',parent=000404BC)  
0012F9E4   0012FBCC  指针到下一个 SEH 记录  
0012F9E8   00445789  SE 句柄  
0012F9EC   0012F9F4  
  
00445769   .  85C0          TEST EAX,EAX  
0044576B   .  75 05         JNZ SHORT Crack.00445772  
0044576D   .  E8 A283FCFF   CALL Crack.0040DB14  
00445772   >  33C0          XOR EAX,EAX  
00445774   .  5A            POP EDX  
00445775   .  59            POP ECX  
00445776   .  59            POP ECX  
00445777   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
0044577A   .  68 90574400   PUSH Crack.00445790  
0044577F   >  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
00445782   .  66:8160 54 FF>AND WORD PTR DS:\[EAX+54\],0FDFF  
00445788   .  C3            RETN         
  
不断 Ctrl+F9 返回。  
  
0043AFCF  |> \\8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
0043AFD2  |.  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
0043AFD4  |.  FF92 E8000000 CALL DWORD PTR DS:\[EDX+E8\]  //F7 进入。  
  
以到这里为主  
  
0043AFCF  
  
这个是调试发现的，你将深入到一些 Api，可以根据需要调试下去。 

 4dd466d0e1e0cc123afbe504da2462ca