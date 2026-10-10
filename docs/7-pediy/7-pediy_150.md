

# ARM3.7xCopyMEMII Code injection

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-555.htm`](https://www.pediy.com/kssd/pediy07/pediy7-555.htm)

【附     件】:**附件:ci1.rar** **附件:ci2.rar**  
【目     标】:在 DFCG 里下了一个别人加壳后的 CRACKME  
【工     具】:OllyDbg v1.1(DIY Version)   
【任     务】:不脱壳通过注入代码，破程序。   
【操作平台】:Windows 2003 server   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【简要说明】： 这次挑了个别人加壳后的目标来操作，这样做可以防止我知道代码而去直接定位到相关位置进行修改。这个 CrackME 加上了全部的保护选项,这也算是 ARM 里很好的保护，我个人认为如果没有用个 SDK 话，这种保护是最强的了。(ARM3.7x copy memII+CC+乱序输入表+Stolen code)  
【详细过程】:

  
  因为之前我已经写过了一篇这样的文章，所以我这次不会写的那么详细。如果你想了解更多，可以先看看我的上一篇文章。载入目标通过 copymem2 的脱壳方法，可以定位到程序的关键代码，把代码改为 NOP 就可以破解成功。  
003A2AF6    90              NOP  
003A2AF7    90              NOP  
此时计算从起始位置到 003a2af6 处距离为：B86   
记下 PATCH ①为:003a2af6      
重新有 Copmem2 脱壳的方法可以很快定位到关键代码：  
0041ABFA   .  8945 E0       MOV DWORD PTR SS:\[EBP-20\],EAX  
0041ABFD   >  837D E0 00    CMP DWORD PTR SS:\[EBP-20\],0  
0041AC01   .  75 11         JNZ SHORT 0041AC14  
  
ECX=00019C06  
  
0041ABFA    E8 014C0200     CALL 0043F800  
加密后为:97 96 8F 8D CA  
记下 PATCH ②为:41ABFA  
通过 SetFilePointer 的方法定位到：  
00AA0877    3341 24         XOR EAX,DWORD PTR DS:\[ECX+24\]  
00AA087A    3345 DC         XOR EAX,DWORD PTR SS:\[EBP-24\]  
  
Stack SS:\[0012D6F4\]=A5989495     ;crc 值  
  
记下 CRC 值：A5989495,起始处的距离为：20877  
记下 PATCH ③为:00aa0877  
   
通过搜索定位到可以 patch 3 的地址，因为 patch 3 是动态申请和动态解压出来的代码。  
搜索到以下地址：  
00408334  |.  833D A0524400>CMP DWORD PTR DS:\[4452A0\],0  
0040833B  |.  0F85 54010000 JNZ 00408495  
  
记下 PATCH ④为:00408334  
第一个 patch 点在以下代码处已经解压出来：  
004238C1   .  50            PUSH EAX                                 ; |Buffer => 003A1F70  
004238C2   .  8B4D EC       MOV ECX,DWORD PTR SS:\[EBP-14\]            ; |  
004238C5   .  51            PUSH ECX                                 ; |Address  
  
ECX==00010F3F  
  
004238C1  - E9 7ABF0100     JMP 0043F840  
加密后为:33 3D 46 D8 5C  
  
第 5 个 patch 点已经出来了，因为这里有几次还原数据，所以要加条进行判断。  
记下 PATCH ⑤为:004238c0  
分析完，开始写代码：  
0043F7EE    0000            ADD BYTE PTR DS:\[EAX\],AL  
0043F7F0    0000            ADD BYTE PTR DS:\[EAX\],AL                  ; 这里保存动态申请的地址  
0043F7F2    0000            ADD BYTE PTR DS:\[EAX\],AL  
0043F7F4    0000            ADD BYTE PTR DS:\[EAX\],AL  
0043F7F6    0090 90909090   ADD BYTE PTR DS:\[EAX+90909090\],DL  
0043F7FC    90              NOP  
0043F7FD    90              NOP  
0043F7FE    90              NOP  
0043F7FF    90              NOP  
0043F800    50              PUSH EAX                                  ; Patch 2 处的代码  
0043F801    A3 F0F74300     MOV DWORD PTR DS:\[43F7F0\],EAX             ; 保存地址信息  
0043F806    B8 34834000     MOV EAX,00408334  
0043F80B    53              PUSH EBX  
0043F80C    BB 65F84300     MOV EBX,0043F865  
0043F811    2BD8            SUB EBX,EAX  
0043F813    83EB 05         SUB EBX,5  
0043F816    C600 E8         MOV BYTE PTR DS:\[EAX\],0E8                 ; 修改 patch 4 的代码  
0043F819    8958 01         MOV DWORD PTR DS:\[EAX+1\],EBX  
0043F81C    5B              POP EBX  
0043F81D    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0043F821    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0043F824    8B00            MOV EAX,DWORD PTR DS:\[EAX\]                ; 还原并执行原代码  
0043F826    C700 8945E083   MOV DWORD PTR DS:\[EAX\],83E04589  
0043F82C    C640 04 7D      MOV BYTE PTR DS:\[EAX+4\],7D  
0043F830    58              POP EAX  
0043F831    C3              RETN  
0043F832    90              NOP  
0043F833    90              NOP  
0043F834    90              NOP  
0043F835    90              NOP  
0043F836    90              NOP  
0043F837    90              NOP  
0043F838    90              NOP  
0043F839    90              NOP  
0043F83A    90              NOP  
0043F83B    90              NOP  
0043F83C    90              NOP  
0043F83D    90              NOP  
0043F83E    90              NOP  
0043F83F    90              NOP  
0043F840    817D EC 0010400>CMP DWORD PTR SS:\[EBP-14\],00401000        ; patch 5 处的代码  
0043F847    75 0C           JNZ SHORT 0043F855                        ; 这里判断是否解压出 patch 1 的正确代码  
0043F849    50              PUSH EAX  
0043F84A    05 860B0000     ADD EAX,0B86  
0043F84F    66:C700 9090    MOV WORD PTR DS:\[EAX\],9090                ; 如果代码已经完全解码了，那么 patch 1  
0043F854    58              POP EAX  
0043F855    50              PUSH EAX                                  ; 执行壳代码  
0043F856    8B4D EC         MOV ECX,DWORD PTR SS:\[EBP-14\]  
0043F859    51              PUSH ECX  
0043F85A  - E9 6740FEFF     JMP 004238C6  
0043F85F    90              NOP  
0043F860    90              NOP  
0043F861    90              NOP  
0043F862    90              NOP  
0043F863    90              NOP  
0043F864    90              NOP  
0043F865    50              PUSH EAX                                  ; patch 4 处的代码  
0043F866    A1 F0F74300     MOV EAX,DWORD PTR DS:\[43F7F0\]  
0043F86B    05 77080200     ADD EAX,20877  
0043F870    53              PUSH EBX  
0043F871    BB 9AF84300     MOV EBX,0043F89A                          ; 修改 patch 3  
0043F876    2BD8            SUB EBX,EAX  
0043F878    83EB 05         SUB EBX,5  
0043F87B    C600 E8         MOV BYTE PTR DS:\[EAX\],0E8  
0043F87E    8958 01         MOV DWORD PTR DS:\[EAX+1\],EBX  
0043F881    5B              POP EBX  
0043F882    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0043F886    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0043F889    8B00            MOV EAX,DWORD PTR DS:\[EAX\]                ; 还原并执行原代码  
0043F88B    C700 833DA052   MOV DWORD PTR DS:\[EAX\],52A03D83  
0043F891    C640 04 44      MOV BYTE PTR DS:\[EAX+4\],44  
0043F895    58              POP EAX  
0043F896    C3              RETN  
0043F897    90              NOP  
0043F898    90              NOP  
0043F899    90              NOP  
0043F89A    50              PUSH EAX                                  ; patch 3 处的代码  
0043F89B    C745 DC 959498A>MOV DWORD PTR SS:\[EBP-24\],A5989495        ; 写入原 CRC 值  
0043F8A2    8D4424 04       LEA EAX,DWORD PTR SS:\[ESP+4\]  
0043F8A6    8328 05         SUB DWORD PTR DS:\[EAX\],5  
0043F8A9    8B00            MOV EAX,DWORD PTR DS:\[EAX\]  
0043F8AB    C700 33412433   MOV DWORD PTR DS:\[EAX\],33244133           ; 还原并执行原壳代码  
0043F8B1    C640 04 45      MOV BYTE PTR DS:\[EAX+4\],45  
0043F8B5    58              POP EAX  
0043F8B6    C3              RETN  
写完后保存再运行就成功了。这篇文章写的很简单，很乱，其实这个壳的 code injection 也就是这么简单容易的:-)  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
By loveboom\[DFCG\]\[FCG\]\[US\]  
Email:loveboom#163.com  
Date:5/12/2005 4:25:00 PM  
  

 4dd466d0e1e0cc123afbe504da2462ca