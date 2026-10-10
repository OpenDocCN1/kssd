
```
0040189F  |.  59             POP ECX  
004018A0  |.  59             POP ECX  
004018A1  \.  C3             RETN  

                  .  
                  .  
                  .  

004018E2  /$  FF7424 10     PUSH DWORD PTR SS:[ESP+10]                     ; (02)进去后来到这里。  
004018E6  |.  FF7424 10     PUSH DWORD PTR SS:[ESP+10]  
004018EA  |.  FF7424 10     PUSH DWORD PTR SS:[ESP+10]  
004018EE  |.  FF7424 10     PUSH DWORD PTR SS:[ESP+10]  
004018F2  |.  E8 43000000   CALL <JMP.&MFC42.#1576_?AfxWinMain>          ; (03)按[F4]执行到这里，接着对该主程序的代码段下内存访问断点，然后按[F9]运行。  
004018F7  \.  C2 1000       RETN 10  

                  .  
                  .  
                  .  

0040193A   $- FF25 6C214000 JMP DWORD PTR DS:[<&MFC42.#1576_?AfxWinMain>]; (04)[F9]后来到这里，再按[F9]运行。  
00401940   .  8D4D 8C       LEA ECX,DWORD PTR SS:[EBP-74]  
00401943   .^ E9 A2FCFFFF   JMP <JMP.&MFC42.#800_?CString>  
00401948   .  8D4D 90       LEA ECX,DWORD PTR SS:[EBP-70]  
0040194B   .^ E9 10F8FFFF   JMP Samples.00401160  
00401950   $  B8 10254000   MOV EAX,Samples.00402510                     ; 结构异常处理程序  
00401955   .^ E9 E8FDFFFF   JMP <JMP.&MSVCRT.__CxxFrameHandler>  

                  .  
                  .  
                  .  

00401524   .- FF25 1C204000 JMP DWORD PTR DS:[<&MFC42.#3922_?InitApplication>]; (05)[F9]后来到这里，再按[F9]运行。  
0040152A   .- FF25 20204000 JMP DWORD PTR DS:[<&MFC42.#1089_?AddToRecentFileList>]  
00401530   .- FF25 24204000 JMP DWORD PTR DS:[<&MFC42.#5199_?OpenDocumentFile@CWinApp>]  
00401536   .- FF25 28204000 JMP DWORD PTR DS:[<&MFC42.#2396_?Delete@CWinThread>]  
0040153C   .- FF25 2C204000 JMP DWORD PTR DS:[<&MFC42.#3346_?GetMainWnd@CWinThread>]  

                  .  
                  .  
                  .  

004010B0   .  6A FF         PUSH -1                                    ; (06)[F9]后来到这里，这里就是“InitInstance()”函数的入口处。  
004010B2   .  68 50194000   PUSH Samples.00401950                      ; SE 处理程序安装  
004010B7   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]  
004010BD   .  50            PUSH EAX  
004010BE   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP  
004010C5   .  83EC 68       SUB ESP,68  
004010C8   .  56            PUSH ESI  
004010C9   .  8BF1          MOV ESI,ECX  
004010CB   .  6A 00         PUSH 0  
004010CD   .  E8 3C050000   CALL <JMP.&MFC42.#1134_?AfxEnableControlContainer>  
004010D2   .  83C4 04       ADD ESP,4  
004010D5   .  8D4C24 04     LEA ECX,DWORD PTR SS:[ESP+4]  
004010D9   .  68 20304000   PUSH Samples.00403020                      ; (07)第一处初始化位置：“InitInstance()”函数中的用户代码开头！  
004010DE   .  E8 25050000   CALL <JMP.&MFC42.#537_?CString>  
004010E3   .  8B4424 04     MOV EAX,DWORD PTR SS:[ESP+4]  
004010E7   .  6A 00         PUSH 0  
004010E9   .  6A 00         PUSH 0  
004010EB   .  50            PUSH EAX  
004010EC   .  C78424 800000>MOV DWORD PTR SS:[ESP+80],0  
004010F7   .  E8 06050000   CALL <JMP.&MFC42.#1200_?AfxMessageBox>  
004010FC   .  8BCE          MOV ECX,ESI  
004010FE   .  E8 F9040000   CALL <JMP.&MFC42.#2621_?Enable3dControls@CWinApp>  
00401103   .  6A 00         PUSH 0  
00401105   .  8D4C24 0C     LEA ECX,DWORD PTR SS:[ESP+C]  
00401109   .  E8 B2000000   CALL Samples.004011C0  
0040110E   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]  
00401112   .  C64424 74 01  MOV BYTE PTR SS:[ESP+74],1  
00401117   .  894E 20       MOV DWORD PTR DS:[ESI+20],ECX  
0040111A   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]  
0040111E   .  E8 D3040000   CALL <JMP.&MFC42.#2514_?DoModal@CDialog>  
00401123   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]  
00401127   .  C64424 74 00  MOV BYTE PTR SS:[ESP+74],0  
0040112C   .  E8 BF040000   CALL <JMP.&MFC42.#641_?CDialog>  
00401131   .  8D4C24 04     LEA ECX,DWORD PTR SS:[ESP+4]  
00401135   .  C74424 74 FFF>MOV DWORD PTR SS:[ESP+74],-1  
0040113D   .  E8 A8040000   CALL <JMP.&MFC42.#800_?CString>  
00401142   .  8B4C24 6C     MOV ECX,DWORD PTR SS:[ESP+6C]  
00401146   .  33C0          XOR EAX,EAX  
00401148   .  5E            POP ESI  
00401149   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX  
00401150   .  83C4 74       ADD ESP,74  
00401153   .  C3            RETN  
\-----------------------------------------------------------
\==================================================================
\==================================================================
```

### 第二启动函数“OnInitDialog()”：(作者：Coderui)

定位方法：

\-----------------------------------------------------------

OD 设置：(OD 设置为不忽略任何异常。`[F2]`：下软断点、`[F4]`：执行到当前代码处、`[F7]`：单步步入、`[F8]`单步步过、`[F9]`运行。)

请按照注解顺序观看(00)-(01)-(02)…(99)，不然很容易混乱。

```
00401750 >/$  55             PUSH EBP                                ; (00)程序被 OD 打开后会停在这里，这个是程序的入口点 OEP。然后使用命令“BP SendMessageA”对“SendMessageA”函数下断点，[F9]运行。  
00401751  |.   8BEC           MOV EBP,ESP  
00401753  |.   6A FF          PUSH -1  
00401755  |.   68 00254000    PUSH Samples.00402500  
0040175A  |.   68 D6184000    PUSH <JMP.&MSVCRT.__except_handler3>    ; SE 处理程序安装  
0040175F  |.   64:A1 0000000>MOV EAX,DWORD PTR FS:[0]  
00401765  |.   50            PUSH EAX  
00401766  |.   64:8925 00000>MOV DWORD PTR FS:[0],ESP  
0040176D  |.   83EC 68       SUB ESP,68  
00401770  |.   53            PUSH EBX  
00401771  |.   56            PUSH ESI  
00401772  |.   57            PUSH EDI  
00401773  |.   8965 E8       MOV DWORD PTR SS:[EBP-18],ESP  
00401776  |.   33DB          XOR EBX,EBX  
00401778  |.   895D FC       MOV DWORD PTR SS:[EBP-4],EBX  
0040177B  |.   6A 02         PUSH 2  
0040177D  |.   FF15 98214000 CALL DWORD PTR DS:[<&MSVCRT.__set_app_type>]  
00401783  |.   59            POP ECX  
00401784  |.   830D 8C314000>OR DWORD PTR DS:[40318C],FFFFFFFF  
0040178B  |.   830D 90314000>OR DWORD PTR DS:[403190],FFFFFFFF  
00401792  |.   FF15 94214000 CALL DWORD PTR DS:[<&MSVCRT.__p__fmode>]  
00401798  |.   8B0D 80314000 MOV ECX,DWORD PTR DS:[403180]  
0040179E  |.   8908          MOV DWORD PTR DS:[EAX],ECX  
004017A0  |.   FF15 90214000 CALL DWORD PTR DS:[<&MSVCRT.__p__commode>]  
004017A6  |.   8B0D 7C314000 MOV ECX,DWORD PTR DS:[40317C]  
004017AC  |.   8908          MOV DWORD PTR DS:[EAX],ECX  
004017AE  |.   A1 8C214000   MOV EAX,DWORD PTR DS:[<&MSVCRT._adjust_fdiv>]  
004017B3  |.   8B00          MOV EAX,DWORD PTR DS:[EAX]  
004017B5  |.   A3 88314000   MOV DWORD PTR DS:[403188],EAX  
004017BA  |.   E8 16010000   CALL Samples.004018D5  
004017BF  |.   391D A0304000 CMP DWORD PTR DS:[4030A0],EBX  
004017C5  |.   75 0C         JNZ SHORT Samples.004017D3  
004017C7  |.   68 D2184000   PUSH Samples.004018D2  
004017CC  |.   FF15 88214000 CALL DWORD PTR DS:[<&MSVCRT.__setusermatherr>]  
004017D2  |.   59            POP ECX  
004017D3  |>   E8 E8000000   CALL Samples.004018C0  
004017D8  |.   68 14304000   PUSH Samples.00403014  
004017DD  |.   68 10304000   PUSH Samples.00403010  
004017E2  |.   E8 D3000000   CALL <JMP.&MSVCRT._initterm>  
004017E7  |.   A1 78314000   MOV EAX,DWORD PTR DS:[403178]  
004017EC  |.   8945 94       MOV DWORD PTR SS:[EBP-6C],EAX  
004017EF  |.   8D45 94       LEA EAX,DWORD PTR SS:[EBP-6C]  
004017F2  |.   50            PUSH EAX  
004017F3  |.   FF35 74314000 PUSH DWORD PTR DS:[403174]  
004017F9  |.   8D45 9C       LEA EAX,DWORD PTR SS:[EBP-64]  
004017FC  |.   50            PUSH EAX  
004017FD  |.   8D45 90       LEA EAX,DWORD PTR SS:[EBP-70]  
00401800  |.   50            PUSH EAX  
00401801  |.   8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]  
00401804  |.   50            PUSH EAX  
00401805  |.   FF15 80214000 CALL DWORD PTR DS:[<&MSVCRT.__getmainargs>]  
0040180B  |.   68 0C304000   PUSH Samples.0040300C  
00401810  |.   68 00304000   PUSH Samples.00403000  
00401815  |.   E8 A0000000   CALL <JMP.&MSVCRT._initterm>  
0040181A  |.   83C4 24       ADD ESP,24  
0040181D  |.   A1 7C214000   MOV EAX,DWORD PTR DS:[<&MSVCRT._acmdln>]  
```
