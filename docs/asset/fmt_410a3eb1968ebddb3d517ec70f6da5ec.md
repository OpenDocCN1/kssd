
```
005047E2  .|>|SBB DWORD PTR SS:[ESP+4],EDX  

005047E6  .|>|POP EAX  

005047E7  .|>|POP EDX  

005047E8  .|>|MOV DWORD PTR SS:[EBP-10],EAX        ;  保存新的结果，等会做迭加  

005047EB  .|>|MOV DWORD PTR SS:[EBP-C],EDX         ;  同上  

005047EE  .|>|DEC EBX  

005047EF  |>> MOV EAX,DWORD PTR SS:[EBP-20]  

005047F2  .|>|CALL DataColl.00404FFC  

005047F7  .|>|CMP EBX,EAX  

005047F9  .|>|JG SHORT DataColl.005047FF  

005047FB  .|>|TEST EBX,EBX  

005047FD  .|>\JG SHORT DataColl.005047B8  

005047FF  |>>MOV EBX,DWORD PTR DS:[ESI+60]  

00504802  .|.>TEST EBX,EBX  

00504804  .|.>JG SHORT DataColl.00504817          ;  比较检验码长度，大于 0 跳，正常应该会跳的  

00504806  .|.>PUSH DWORD PTR SS:[EBP-C]            ; /Arg2  

00504809  .|.>PUSH DWORD PTR SS:[EBP-10]           ; |Arg1  

0050480C  .|.>MOV EDX,EDI                          ; |  

0050480E  .|.>XOR EAX,EAX                          ; |  

00504810  .|.>CALL DataColl.00409B44              ; \DataColl.00409B44  

00504815  .|.>JMP SHORT DataColl.0050483D  

00504817  |>>PUSH DWORD PTR SS:[EBP-C]            ; /刚才的结果１  

0050481A  .|.>PUSH DWORD PTR SS:[EBP-10]           ; |结果２  

0050481D  .|.>MOV EDX,EDI                          ; |  

0050481F  .|.>MOV EAX,EBX                          ; |  

00504821  .|.>CALL DataColl.00409B44              ; \串接  

00504826  .|.>MOV EAX,DWORD PTR DS:[EDI]           ;  DS:[EDI]就是注册码了  
```

总的来说，算法还是可以的，思路也很清淅，可惜用了明码作比较，败笔！ 

`4dd466d0e1e0cc123afbe504da2462ca`
