
## 汇编代码分析

```
.text:09253496                 mov     [ebp+var_20], 0  
.text:0925349D                 mov     [ebp+var_24], 3  
.text:092534A4                 mov     eax, [ebp+arg_0]  
.text:092534A7                 mov     edx, [eax+6Ch]  
.text:092534AA                 mov     ax, [edx+0F54h]  
.text:092534B1                 and     eax, 0FFFF8000h  
.text:092534B6                 test    ax, ax  
.text:092534B9                 jz      short loc_92534E6  
.text:092534BB                 cmp     l_n36_buff, 0          ; =>l_n36_buff 第一次  
.text:092534C2                 jz      short loc_92534E6  
.text:092534C4                 mov     eax, [ebp+arg_8]  
.text:092534C7                 push    eax  
.text:092534C8                 mov     eax, [ebp+arg_4]  
.text:092534CB                 push    eax  
.text:092534CC                 mov     eax, [ebp+arg_0]  
.text:092534CF                 push    eax  
.text:092534D0                 mov     ebx, l_n36_buff        ; =>l_n36_buff 第二次  
.text:092534D6                 mov     [ebp+var_28], ebx  
.text:092534D9                 mov     edi, [ebp+var_28]  
.text:092534DC                 call    edi                    ; =>这个 CALL 很著名了  
.text:092534DE                 add     esp, 0Ch  
.text:092534E1                 jmp     loc_9253628  
.text:092534E6 ; ---------------------------------------------------------------------------  
```

## 关于 `l_n36_buf` 函数

依据 `CrackZ` 的文章，列个 `lc_new_job` 里的来说明吧。

```
.text:0925D4F0 ; *************** S U B R O U T I N E ****************  
.text:0925D4F0 ; Attributes: bp-based frame  
.text:0925D4F0 public lc_new_job  
.text:0925D4F0 lc_new_job      proc near              ; CODE XREF: sub_92241D0+60p  
.text:0925D4F0 var_1C          = dword ptr -1Ch  
.text:0925D4F0 var_14          = dword ptr -14h  
.text:0925D4F0 var_10          = dword ptr -10h  
.text:0925D4F0 var_C           = dword ptr -0Ch  
.text:0925D4F0 arg_0           = dword ptr  8  
.text:0925D4F0 arg_4           = dword ptr  0Ch  
.text:0925D4F0 arg_8           = dword ptr  10h  
.text:0925D4F0 arg_C           = dword ptr  14h  
.text:0925D4F0                 push    ebp  
.text:0925D4F1                 mov     ebp, esp  
.text:0925D4F3                 sub     esp, 14h  
.text:0925D4F6                 push    esi  
.text:0925D4F7                 push    ebx  
.text:0925D4F8                 mov     eax, l_n36_buf         ; =>这个是什么？  
.text:0925D4FD                 mov     [ebp+arg_4], eax  
.text:0925D500                 lea     eax, [ebp+var_14]  
.text:0925D503                 push    eax  
.text:0925D504                 push    0  
.text:0925D506                 push    0  
.text:0925D508                 push    0  
.text:0925D50A                 mov     eax, [ebp+arg_8]  
.text:0925D50D                 push    eax  
.text:0925D50E                 lea     eax, [ebp+var_C]  
.text:0925D511                 push    eax  
.text:0925D512                 mov     ebx, [ebp+arg_4]  
.text:0925D515                 call    ebx                  ; =>这个呢？  
.text:0925D517                 add     esp, 18h  
.text:0925D51A                 push    0  
.text:0925D51C                 push    0  
.text:0925D51E                 push    0  
.text:0925D520                 push    0  
.text:0925D522                 push    0  
.text:0925D524                 push    0  
.text:0925D526                 mov     ebx, [ebp+arg_4]  
.text:0925D529                 call    ebx  
.text:0925D52B                 add     esp, 18h  
.text:0925D52E                 mov     eax, [ebp+arg_C]  
.text:0925D531                 push    eax  
.text:0925D532                 mov     eax, [ebp+arg_8]  
.text:0925D535                 push    eax  
.text:0925D536                 lea     eax, [ebp+var_C]  
.text:0925D539                 push    eax  
.text:0925D53A                 mov     eax, [ebp+arg_0]  
.text:0925D53D                 push    eax  
.text:0925D53E                 call    lc_init  
.text:0925D543                 add     esp, 10h  
.text:0925D546                 mov     eax, eax  
.text:0925D548                 mov     [ebp+var_10], eax  
.text:0925D54B                 cmp     [ebp+var_10], 0  
.text:0925D54F                 jnz     loc_925D5F1  
.text:0925D555                 mov     eax, [ebp+arg_C]  
.text:0925D558                 mov     edx, [eax]  
.text:0925D55A                 mov     eax, [edx+6Ch]  
.text:0925D55D                 mov     edx, [ebp+arg_C]  
.text:0925D560                 mov     ecx, [edx]  
.text:0925D562                 mov     edx, [ecx+6Ch]  
.text:0925D565                 mov     esi, [edx+0F54h]  
.text:0925D56B                 or      esi, 8000h  
.text:0925D571                 mov     [eax+0F54h], esi  
.text:0925D577                 mov     eax, [ebp+arg_C]  
.text:0925D57A                 mov     edx, [eax]  
.text:0925D57C                 mov     eax, [ebp+arg_4]  
.text:0925D57F                 mov     [edx+618h], eax  
.text:0925D585                 mov     eax, [ebp+arg_C]  
.text:0925D588                 mov     edx, [eax]  
.text:0925D58A                 cmp     word ptr [edx+440h], 2655h  
.text:0925D593                 jnz     short loc_925D5B1  
.text:0925D595                 push    0  
.text:0925D597                 push    0  
.text:0925D599                 push    0  
.text:0925D59B                 push    2  
.text:0925D59D                 push    0  
.text:0925D59F                 push    0  
.text:0925D5A1                 mov     ebx, [ebp+arg_4]  
.text:0925D5A4                 call    ebx  
.text:0925D5A6                 add     esp, 18h  
.text:0925D5A9                 mov     eax, eax  
.text:0925D5AB                 test    eax, eax  
.text:0925D5AD                 jnz     short loc_925D5B1  
.text:0925D5AF                 jmp     short loc_925D5D3  
.text:0925D5B1 ; ---------------------------------------------------------------------------  
.text:0925D5B1 loc_925D5B1:                     ; CODE XREF: lc_new_job+A3j  
.text:0925D5B1                                ; lc_new_job+BDj  
.text:0925D5B1                 mov     eax, [ebp+arg_C]  
.text:0925D5B4                 mov     edx, [eax]  
.text:0925D5B6                 mov     eax, [edx+6Ch]  
.text:0925D5B9                 mov     edx, [ebp+arg_C]  
.text:0925D5BC                 mov     ecx, [edx]  
.text:0925D5BE                 mov     edx, [ecx+6Ch]  
.text:0925D5C1                 mov     esi, [edx+0F54h]  
.text:0925D5C7                 or      esi, 40000h  
.text:0925D5CD                 mov     [eax+0F54h], esi  
.text:0925D5D3 loc_925D5D3:                     ; CODE XREF: lc_new_job+BFj  
.text:0925D5D3                 mov     eax, [ebp+arg_C]  
.text:0925D5D6                 mov     edx, [eax]  
.text:0925D5D8                 mov     eax, [ebp+var_14]  
.text:0925D5DB                 cmp     eax, [edx+5D0h]  
.text:0925D5E1                 jle     short loc_925D5F1  
.text:0925D5E3                 mov     eax, [ebp+arg_C]  
.text:0925D5E6                 mov     edx, [eax]  
.text:0925D5E8                 mov     eax, [ebp+var_14]  
.text:0925D5EB                 mov     [edx+5D0h], eax  
.text:0925D5F1 loc_925D5F1:  
```
