
```
0038A7E2    55                     PUSH EBP                     ;push ebp  

0038A7E6    54                     PUSH ESP  

0038A7E7    5D                     POP EBP                      ;mov ebp, esp  

0038A7EC    54                     PUSH ESP  

0038A7ED    830424 F0              ADD DWORD PTR SS:[ESP],-10  

0038A7F1    5C                     POP ESP                      ;add esp, -10  

0038A7F4    68 54175700            PUSH 571754  

0038A7F9    58                     POP EAX                      ;mov eax, 571754  

0038A7FF    E8 02000000            CALL 0038A806                ;把返回地址 38A804 压栈  

0038A804    EB 11                  JMP SHORT 0038A817  

0038A806    50                     PUSH EAX                     ;为下面的 xchg 作准备  

0038A807    E8 04000000            CALL 0038A810               

0038A80C    C471 40                LES ESI,FWORD PTR DS:[ECX+40]    ; 

0038A810    58                     POP EAX                      ;第一 proc ptr [38A80C]  

0038A811    8B00                   MOV EAX,DWORD PTR DS:[EAX]             

0038A813    870424                 XCHG DWORD PTR SS:[ESP],EAX  

0038A816    C3                     RETN                       ;call 4071C4  

;=============  

0038A81C    E8 04000000            CALL 0038A825  

0038A821    4C                     DEC ESP  

0038A822    73 57                  JNB SHORT 0038A87B  

0038A825    58                     POP EAX                      ;[38A821]  

0038A826    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, 57734C  

0038A828    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, [eax]  

0038A82C    FF30                   PUSH DWORD PTR DS:[EAX]  

0038A82E    58                     POP EAX                      ;mov eax, [eax]  

0038A832    E8 02000000            CALL 0038A839                ;把返回地址 38A837 压栈  

0038A837    EB 11                  JMP SHORT 0038A84A  

0038A839    50                     PUSH EAX  

0038A83A    E8 04000000            CALL 0038A843  

0038A83F    60                     PUSHAD  

0038A840    B7 46                  MOV BH,46  

0038A843    58                     POP EAX                      ;第二 proc ptr [0038A83F]  

0038A844    8B00                   MOV EAX,DWORD PTR DS:[EAX]  

0038A846    870424                 XCHG DWORD PTR SS:[ESP],EAX  

0038A849    C3                     RETN                       ;call 46B760  

;=============  

0038A84C    FF35 A86D5700          PUSH DWORD PTR DS:[576DA8]                     

0038A852    59                     POP ECX                      ;mov ecx, [576DA8]  

0038A858    E8 04000000            CALL 0038A861  

0038A85D    4C                     DEC ESP  

0038A85E    73 57                  JNB SHORT 0038A8B7  

0038A861    58                     POP EAX                      ;  

0038A862    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, 57734C  

0038A864    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, [eax]  

0038A869    FF30                   PUSH DWORD PTR DS:[EAX]  

0038A86B    58                     POP EAX                      ;mov eax, [eax]  

0038A86E    FF35 E87A5300          PUSH DWORD PTR DS:[537AE8]           

0038A874    5A                     POP EDX                      ;mov edx, [537AE8]  

0038A879    E8 02000000            CALL 0038A880                ;把返回地址 38A87E 压栈  

0038A87E    EB 11                  JMP SHORT 0038A891  

0038A880    50                     PUSH EAX  

0038A881    E8 04000000            CALL 0038A88A  

0038A886   ^ 78 B7                  JS SHORT 0038A83F              ;第三 proc ptr  

0038A888    46                     INC ESI  

0038A88A    58                     POP EAX  

0038A88B    8B00                   MOV EAX,DWORD PTR DS:[EAX]             

0038A88D    870424                 XCHG DWORD PTR SS:[ESP],EAX  

0038A890    C3                     RETN                       ;call 46B778  

;=============  

0038A895    FF35 DC6B5700          PUSH DWORD PTR DS:[576BDC]         

0038A89B    59                     POP ECX                      ;mov ecx, [576BDC]  

0038A8A1    E8 04000000            CALL 0038A8AA  

0038A8A6    4C                     DEC ESP  

0038A8A7    73 57                  JNB SHORT 0038A900  

0038A8AA    58                     POP EAX  

0038A8AB    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, 57734C  

0038A8AD    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, [eax]  

0038A8B3    FF30                   PUSH DWORD PTR DS:[EAX]  

0038A8B5    58                     POP EAX                      ;mov eax, [eax]  

0038A8BB    FF35 74405200          PUSH DWORD PTR DS:[524074]           

0038A8C1    5A                     POP EDX                      ;mov edx, [524074]  

0038A8C5    E8 02000000            CALL 0038A8CC                ;把返回地址 38A8CA 压栈  

0038A8CA    EB 11                  JMP SHORT 0038A8DD  

0038A8CD    E8 04000000            CALL 0038A8D6  

0038A8D2   ^ 78 B7                  JS SHORT 0038A88B              ;第四 proc ptr  

0038A8D4    46                     INC ESI  

0038A8D6    58                     POP EAX                       

0038A8D7    8B00                   MOV EAX,DWORD PTR DS:[EAX]  

0038A8D9    870424                 XCHG DWORD PTR SS:[ESP],EAX  

0038A8DC    C3                     RETN                       ;call 46B778  

;=============  

0038A8E0    FF35 30765700          PUSH DWORD PTR DS:[577630]           

0038A8E6    59                     POP ECX                      ;mov ecx, [577630]  

0038A8EC    E8 04000000            CALL 0038A8F5  

0038A8F1    4C                     DEC ESP  

0038A8F2    73 57                  JNB SHORT 0038A94B  

0038A8F5    58                     POP EAX  

0038A8F6    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, 57734C  

0038A8F8    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, [eax]  

0038A8FD    FF30                   PUSH DWORD PTR DS:[EAX]  

0038A8FF    58                     POP EAX                      ;mov eax, [eax]  

0038A903    FF35 A0755300          PUSH DWORD PTR DS:[5375A0]           

0038A909    5A                     POP EDX                      ;mov edx, [5375A0]  

0038A90D    E8 02000000            CALL 0038A914                ;把返回地址 38A912 压栈  

0038A912    EB 11                  JMP SHORT 0038A925                 

0038A914    50                     PUSH EAX  

0038A915    E8 04000000            CALL 0038A91E  

0038A91A   ^ 78 B7                  JS SHORT 0038A8D3              ;第五 proc ptr  

0038A91C    46                     INC ESI  

0038A91E    58                     POP EAX                      

0038A91F    8B00                   MOV EAX,DWORD PTR DS:[EAX]             

0038A921    870424                 XCHG DWORD PTR SS:[ESP],EAX           

0038A924    C3                     RETN                       ;call 46B778  

;=============  

0038A928    E8 04000000            CALL 0038A931  

0038A92D    4C                     DEC ESP  

0038A92E    73 57                  JNB SHORT 0038A987  

0038A931    58                     POP EAX                      

0038A932    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, 57734C  

0038A934    8B00                   MOV EAX,DWORD PTR DS:[EAX]        ;mov eax, [eax]  

0038A939    FF30                   PUSH DWORD PTR DS:[EAX]  

0038A93B    58                     POP EAX                      ;mov eax, [eax]  

0038A93E    E8 02000000            CALL 0038A945                ;把返回地址 38A943 压栈
```
