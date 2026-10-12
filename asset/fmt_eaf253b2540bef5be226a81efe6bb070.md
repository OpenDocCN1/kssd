
```
  1002ECD5    50                   push eax  

  1002ECD6    8D4D 08             lea ecx,dword ptr ss:[ebp+8]  

  1002ECD9    FF15 C8220C10       call dword ptr ds:[<&MFC71LU.#2895>]     ; 计算连接后字符串  

的长度  

  1002ECDF    50                   push eax  

  1002ECE0    53                   push ebx  

  1002ECE1    8D4D 08             lea ecx,dword ptr ss:[ebp+8]  

  1002ECE4    FF15 CC230C10       call dword ptr ds:[<&MFC71LU.#2460>]     ; MFC71LU.#5149  

  1002ECEA    50                   push eax  

  1002ECEB    53                   push ebx  

  1002ECEC    53                   push ebx  

  1002ECED    FF15 DC7F1110       call dword ptr ds:[10117FDC]             ;  

kernel32.WideCharToMultiByte  

  1002ECF3    8BF8                mov edi,eax  

  1002ECF5    6A FF             push -1  

  1002ECF7    8D4D 08             lea ecx,dword ptr ss:[ebp+8]  

  1002ECFA    889C3D F0FBFFFF   mov byte ptr ss:[ebp+edi-410],bl  

  1002ED01    FF15 C8230C10       call dword ptr ds:[<&MFC71LU.#5398>]     ; 字符串全部变成小  

写  

  1002ED07    33C0                xor eax,eax                               ; 清空 EAX  

  1002ED09    33F6                xor esi,esi                               ; 清空 ESI  

  1002ED0B    3BFB                cmp edi,ebx  

  1002ED0D    7E 13               jle short Converte.1002ED22  

  1002ED0F    0FBE8C05 F0FBFF>  movsx ecx,byte ptr ss:[ebp+eax-410]      ; 字符串逐位送 ECX  

  1002ED17    8D740E 0D           lea esi,dword ptr ds:[esi+ecx+D]  

  1002ED1B    03DE                add ebx,esi                               ; 将字符串各位累加  

，结果放 EBX  

  1002ED1D    40                  inc eax                                     ; EAX 加一  

  1002ED1E    3BC7                cmp eax,edi                                 ; 取完了没  

  1002ED20  ^ 7C ED             jl short Converte.1002ED0F                 ; 没有就继续  

  1002ED22    8D4D 08             lea ecx,dword ptr ss:[ebp+8]             ; 循环结束后  

EBX=000175B2  

  1002ED25    FF15 B8220C10       call dword ptr ds:[<&MFC71LU.#577>]       ; MFC71LU.#577  

  1002ED2B    8B4D F4             mov ecx,dword ptr ss:[ebp-C]  

  1002ED2E    8BC3                mov eax,ebx                               ; 结果送 EAX  

  1002ED30    C1E0 10             shl eax,10                                ; EAX 左移 10 位（16  

进制）  

  1002ED33    5F                  pop edi  

  1002ED34    03C6                add eax,esi                               ; EAX+ESI 送 EAX，  

ESI 为 118A  

  1002ED36    5E                  pop esi  

  1002ED37    64:890D 0000000>  mov dword ptr fs:[0],ecx  

  1002ED3E    8B4D F0             mov ecx,dword ptr ss:[ebp-10]  

  1002ED41    5B                  pop ebx  

  1002ED42    E8 A24B0800         call Converte.100B38E9  

  1002ED47    C9                  leave  

  1002ED48    C3                  retn  //返回  
```

跟进`1003ABCF`处`CALL`来到：

```
  00521463 >  8B5424 04       mov edx,dword ptr ss:[esp+4]  

  00521467    85D2            test edx,edx  

  00521469    7C 0E           jl short MFC71LU.00521479  

  0052146B    8B01            mov eax,dword ptr ds:[ecx]                 ; 字符表  

bcdfghkmnpqrstvwxyz 送 EAX  

  0052146D    3B50 F4         cmp edx,dword ptr ds:[eax-C]  

  00521470    7F 07           jg short MFC71LU.00521479  

  00521472    66:8B0450       mov ax,word ptr ds:[eax+edx*2]             ; [eax+edx*2]送 AX  

，查表结果  

  00521476    C2 0400         retn 4                                     ; 返回  
```

--------------------------------------------------------------------------------

### 【经验总结】

注册码只和邮件地址有关，将`SolidConverterPDFv3Pro`和所填邮件地址连接起来然后将各位的 ASCII 码累加，将累加结果加上`118A`记为`S`，用`S`除以`13(16 进制)`，将商作为下次循环的被除数，余数用于查表`bcdfghkmnpqrstvwxyz`，循环次数大于 4 或商为零时退出循环，最后将查表结果转换成大写字母即为注册码!!!

比如：

- 姓名：`yijun`
- 电子邮箱：`yijun8354@sina.com`
- 机构名称：`www.cqupt.edu.cn`
- 解锁码：`HSDX`

--------------------------------------------------------------------------------

### 【版权声明】

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2006 年 04 月 19 日 16:22:36

`4dd466d0e1e0cc123afbe504da2462ca`
