
```
004048ED    74 09            je short lyns\_\_.004048F8  

004048EF    0341 FC          add eax,dword ptr ds:\[ecx\-4\]        ; EAX 加\[ecx-4\]=1 送 EAX（已经计算出的真码长度）  

004048F2    39CF            cmp edi,ecx                             ; ECX 和 EDI 比较  

004048F4    75 02           jnz short lyns\_\_.004048F8               ; 不等就跳  

004048F6    31FF            xor edi,edi  

004048F8    4A              dec edx  

004048F9  ^ 75 EC           jnz short lyns\_\_.004048E7               ; EDX 不为 0 就继续  

004048FB    85FF            test edi,edi  

004048FD    74 17           je short lyns\_\_.00404916  

004048FF    89C2            mov edx,eax  

00404901    8B0424          mov eax,dword ptr ss:\[esp\]  

00404904    8B77 FC         mov esi,dword ptr ds:\[edi\-4\]  

00404907    E8 88020000     call lyns\_\_.00404B94  

0040490C    8B3C24          mov edi,dword ptr ss:\[esp\]  

0040490F    FF37            push dword ptr ds:\[edi\]  

00404911    0337            add esi,dword ptr ds:\[edi\]              ; \[edi\]加 ESI  

00404913    4B              dec ebx                                   ; EBX 减一  

00404914    EB 08           jmp short lyns\_\_.0040491E  

00404916    E8 F1FCFFFF     call lyns\_\_.0040460C  

0040491B    50              push eax  

0040491C    89C6            mov esi,eax  

0040491E    8B449C 18       mov eax,dword ptr ss:\[esp+ebx\*4+18\] ; 计算出的真码保存在 EAX  

00404922    89F2            mov edx,esi  

00404924    85C0            test eax,eax  

00404926    74 0A           je short lyns\_\_.00404932  

00404928    8B48 FC         mov ecx,dword ptr ds:\[eax\-4\]          ; ECX=1  

0040492B    01CE            add esi,ecx                               ; ESI 加 ECX  

0040492D    E8 32E0FFFF     call lyns\_\_.00402964                     ;  

00404932    4B              dec ebx  

00404933  ^ 75 E9           jnz short lyns\_\_.0040491E               ; EBX 不为 0 继续  

00404935    5A              pop edx  

00404936    58              pop eax  

00404937    85FF            test edi,edi  

00404939    75 0C           jnz short lyns\_\_.00404947  

0040493B    85D2            test edx,edx  

0040493D    74 03           je short lyns\_\_.00404942  

0040493F    FF4A F8         dec dword ptr ds:\[edx\-8\]  

00404942    E8 55FCFFFF     call lyns\_\_.0040459C  

00404947    5A              pop edx  

00404948    5F              pop edi  

00404949    5E              pop esi  

0040494A    5B              pop ebx  

0040494B    58              pop eax  

0040494C    8D2494          lea esp,dword ptr ss:\[esp+edx\*4\]  

0040494F    FFE0            jmp eax  

00404951    C3              retn  

 \--------------------------------------------------------------------------------
```

## 破解总结

注册机：

* 中断地址：`4B9D97`
* 中断次数：`1`
* 第一字节：`E8`
* 指令长度：`5`
* 内存方式：寄存器 `EDX`

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
