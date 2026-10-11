
## 汇编代码片段

```
00474B78  |.  33C0             XOR EAX,EAX  
00474B7A  |.  55              PUSH EBP  
00474B7B  |.  68 EA4B4700     PUSH QQTelesc.00474BEA  
00474B80  |.  64:FF30         PUSH DWORD PTR FS:[EAX]  
00474B83  |.  64:8920         MOV DWORD PTR FS:[EAX],ESP  
00474B86  |.  BB 01000000     MOV EBX,1  
00474B8B  |.  BE 02000000     MOV ESI,2  
00474B90  |.  33FF            XOR EDI,EDI  
00474B92  |.  33C0            XOR EAX,EAX  
00474B94  |.  8945 F8         MOV DWORD PTR SS:[EBP-8],EAX  
00474B97  |.  EB 0D           JMP SHORT QQTelesc.00474BA6  
00474B99  |>  8B45 FC         /MOV EAX,DWORD PTR SS:[EBP-4]                ;  机器码  
00474B9C  |.  0FB64418 FF     |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]           ;  取机器码第 n 位 奇数位  
00474BA1  |.  03F8            |ADD EDI,EAX                                 ;  edi=edi+eax(取的那位)  
00474BA3  |.  83C3 02         |ADD EBX,2                                   ;  ebx 加 2  
00474BA6  |>  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ;  机器码  
00474BA9  |.  E8 72FCF8FF     |CALL QQTelesc.00404820  
00474BAE  |.  3BD8            |CMP EBX,EAX  
00474BB0  |.^ 7E E7           \JLE SHORT QQTelesc.00474B99  
00474BB2  |.  EB 0E           JMP SHORT QQTelesc.00474BC2  
00474BB4  |>  8B45 FC         /MOV EAX,DWORD PTR SS:[EBP-4]                ;  机器码  
00474BB7  |.  0FB64430 FF     |MOVZX EAX,BYTE PTR DS:[EAX+ESI-1]           ;  取机器码第 n 位 偶数位  
00474BBC  |.  0145 F8         |ADD DWORD PTR SS:[EBP-8],EAX               ;  ebp-8 = ebp-8 +eax(取的那位)  
00474BBF  |.  83C6 02         |ADD ESI,2                                   ;  esi 加 2  
00474BC2  |>  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ;  机器码  
00474BC5  |.  E8 56FCF8FF     |CALL QQTelesc.00404820  
00474BCA  |.  3BF0            |CMP ESI,EAX  
00474BCC  |.^ 7E E6           \JLE SHORT QQTelesc.00474BB4  
00474BCE  |.  8BDF            MOV EBX,EDI  
00474BD0  |.  0FAF5D F8       IMUL EBX,DWORD PTR SS:[EBP-8]               ;  偶数和与奇数和相乘 结果就是注册码  
00474BD4  |.  33C0            XOR EAX,EAX  
00474BD6  |.  5A              POP EDX  
00474BD7  |.  59              POP ECX  
00474BD8  |.  59              POP ECX  
00474BD9  |.  64:8910         MOV DWORD PTR FS:[EAX],EDX  
00474BDC  |.  68 F14B4700     PUSH QQTelesc.00474BF1  
00474BE1  |>  8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]  
00474BE4  |.  E8 7FF9F8FF     CALL QQTelesc.00404568  
00474BE9  \\.  C3              RETN  
```

## 注册机生成代码

```
a="" '你的机器码
For i=1 To Len(a) Step 2
  b=b+Asc(Mid(a,i,1))
  c=c+Asc(Mid(a,i+1,1))
Next
d=b*c
d 就是注册码啦。。。。
```

下载链接：`注册机.rar`

菜鸟第一次写文章 大家见笑了啊。。。。
