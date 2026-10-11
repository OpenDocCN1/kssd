
### 代码片段 1

```
.text:260D4144                 jz      short loc_260D414D  

.text:260D4146                 mov     [ebp+var_4], 0     //第一组有重复字符这里放 0  

.text:260D414D  

.text:260D414D loc_260D414D:                    

.text:260D414D                 mov     ecx, [ebp+var_28]  

.text:260D4150                 add     ecx, 4  

.text:260D4153                 call    fun1_260D3940  

.text:260D4158                 test    eax, eax  

.text:260D415A                 jz      short loc_260D4163  

.text:260D415C                 mov     [ebp+var_4], 1     //第二组有重复字符这里放 1  

.text:260D4163  

.text:260D4163 loc_260D4163:                    

.text:260D4163                 mov     ecx, [ebp+var_28]  

.text:260D4166                 add     ecx, 8  

.text:260D4169                 call    fun1_260D3940  

.text:260D416E                 test    eax, eax  

.text:260D4170                 jz      short loc_260D4179  

.text:260D4172                 mov     [ebp+var_4], 2     //第三组有重复字符这里放 2  

.text:260D4179  

.text:260D4179 loc_260D4179:                    

.text:260D4179                 cmp     [ebp+lParam], 0  

.text:260D417D                 jnz     short loc_260D41B5  

.text:260D417F                 mov     ecx, [ebp+var_4]   //我们先假定这个变量为 i  

.text:260D4182                 mov     edx, [ebp+var_28]  //并且假定这个数组是 unsigned int ci[3]  

.text:260D4185                 lea     eax, [edx+ecx*4]   //取有重复字符那组  

.text:260D4188                 mov     [ebp+var_8], eax  

.text:260D418B                 mov     ecx, [ebp+var_8]  

.text:260D418E                 xor     edx, edx  

.text:260D4190                 mov     dl, [ecx+3]  

.text:260D4193                 mov     eax, [ebp+var_8]  

.text:260D4196                 xor     ecx, ecx  

.text:260D4198                 mov     cl, [eax+2]  

.text:260D419B                 mov     eax, [ebp+var_8]  

.text:260D419E                 xor     ebx, ebx  

.text:260D41A0                 mov     bl, [eax+1]  

.text:260D41A3                 mov     esi, [ebp+var_8]  

.text:260D41A6                 xor     eax, eax  

.text:260D41A8                 mov     al, [esi]  

.text:260D41AA                 add     eax, ebx  

.text:260D41AC                 add     eax, ecx  

.text:260D41AE                 add     eax, edx  

.text:260D41B0                 mov     [ebp+lParam], eax  //重复的那组索引累加和要等于 60h 或者 61h  

.text:260D41B3                 jmp     short loc_260D41F2  

.text:260D41B5  

.text:260D41B5 loc_260D41B5:                   

.text:260D41B5                 mov     ecx, [ebp+var_4]  

.text:260D41B8                 mov     edx, [ebp+var_28]  

.text:260D41BB                 lea     eax, [edx+ecx*4]  

.text:260D41BE                 mov     [ebp+var_C], eax  

.text:260D41C1                 mov     ecx, [ebp+var_C]  

.text:260D41C4                 xor     edx, edx  

.text:260D41C6                 mov     dl, [ecx+3]  

.text:260D41C9                 mov     eax, [ebp+var_C]  

.text:260D41CC                 xor     ecx, ecx  

.text:260D41CE                 mov     cl, [eax+2]  

.text:260D41D1                 mov     eax, [ebp+var_C]  

.text:260D41D4                 xor     ebx, ebx  

.text:260D41D6                 mov     bl, [eax+1]  

.text:260D41D9                 mov     esi, [ebp+var_C]  

.text:260D41DC                 xor     eax, eax  

.text:260D41DE                 mov     al, [esi]  

.text:260D41E0                 add     eax, ebx  

.text:260D41E2                 add     eax, ecx  

.text:260D41E4                 add     eax, edx  

.text:260D41E6                 cmp     eax, [ebp+lParam]  

.text:260D41E9                 jz      short loc_260D41F2  

.text:260D41EB                 xor     eax, eax  

.text:260D41ED                 jmp     loc_260D43B5  

.text:260D41F2  

.text:260D41F2 loc_260D41F2:                    

.text:260D41F2                                 

.text:260D41F2                 mov     ecx, [ebp+lParam]  

.text:260D41F5                 mov     [ebp+var_2C], ecx  

.text:260D41F8                 cmp     [ebp+var_2C], 60h  

.text:260D41FC                 jz      short loc_260D420D  //和等于 60H 跳  

.text:260D41FE                 cmp     [ebp+var_2C], 61h  

.text:260D4202                 jz      loc_260D42D7        //和等于 61H 跳  

.text:260D4208                 jmp     loc_260D434C  

.text:260D420D  

.text:260D420D loc_260D420D:                    

.text:260D420D                 mov     eax, [ebp+var_4]  //和 60H,  

.text:260D4210                 add     eax, 1  

.text:260D4213                 cdq  

.text:260D4214                 mov     ecx, 3  

.text:260D4219                 idiv    ecx  

.text:260D421B                 mov     eax, [ebp+var_28]  

.text:260D421E                 mov     eax, [eax+edx*4]  //i = (i + 1) % 3  

.text:260D4221                 xor     edx, edx  

.text:260D4223                 mov     ecx, 3  

.text:260D4228                 div     ecx  

.text:260D422A                 test    edx, edx          //ci[i] % 3 要等于 0  

.text:260D422C                 jz      short loc_260D4235  

.text:260D422E                 xor     eax, eax  

.text:260D4230                 jmp     loc_260D43B5  

.text:260D4235  

.text:260D4235 loc_260D4235:                    

.text:260D4235                 mov     eax, [ebp+var_4]  

.text:260D4238                 add     eax, 2  

.text:260D423B                 cdq  

.text:260D423C                 mov     ecx, 3  

.text:260D4241                 idiv    ecx  

.text:260D4243                 mov     eax, [ebp+var_28]  

.text:260D4246                 mov     eax, [eax+edx*4]  //i = (i + 2) % 3  

.text:260D4249                 xor     edx, edx  

.text:260D424B                 mov     ecx, 3  

.text:260D4250                 div     ecx  

.text:260D4252                 test    edx, edx          //ci[i] % 3 要等于 0  

.text:260D4254                 jz      short loc_260D425D  

.text:260D4256                 xor     eax, eax  

.text:260D4258                 jmp     loc_260D43B5  

.text:260D425D  

.text:260D425D loc_260D425D:                    

.text:260D425D                 mov     eax, [ebp+var_4]  

.text:260D4260                 add     eax, 1  

.text:260D4263                 cdq  

.text:260D4264                 mov     ecx, 3  

.text:260D4269                 idiv    ecx  

.text:260D426B                 mov     eax, [ebp+var_28]  

.text:260D426E                 mov     eax, [eax+edx*4] //i = (i + 1) % 3  

.text:260D4271                 xor     edx, edx  

.text:260D4273                 mov     ecx, 5  

.text:260D4278                 div     ecx  

.text:260D427A                 test    edx, edx         //ci[i] % 5 要等于 0  

.text:260D427C                 jz      short loc_260D4285  

.text:260D427E                 xor     eax, eax  

.text:260D4280                 jmp     loc_260D43B5  

.text:260D4285  

.text:260D4285 loc_260D4285:                    

.text:260D4285                 mov     eax, [ebp+var_4]  

.text:260D4288                 add     eax, 2  

.text:260D428B                 cdq  

.text:260D428C                 mov     ecx, 3  

.text:260D4291                 idiv    ecx  

.text:260D4293                 mov     eax, [ebp+var_28]  

.text:260D4296                 mov     eax, [eax+edx*4] //i = (i + 2) % 3  

.text:260D4299                 xor     edx, edx  

.text:260D429B                 mov     ecx, 5  

.text:260D42A0                 div     ecx  

.text:260D42A2                 test    edx, edx         //ci[i] % 5 要等于 0  

.text:260D42A4                 jz      short loc_260D42AD  

.text:260D42A6                 xor     eax, eax  

.text:260D42A8                 jmp     loc_260D43B5  

.text:260D42AD  
```
