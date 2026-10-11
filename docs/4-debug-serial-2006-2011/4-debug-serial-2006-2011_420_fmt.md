
# IdealDVDCopy 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/62104.html`](https://www.pediy.com/kssd/pediy10/62104.html)

【文章标题】: `IdealDVDCopy` 算法分析

【文章作者】: 小子贼野

【作者主页】: `http://mayday.unpack.cn`

【下载地址】: `http://www.idealdvdcopy.com/`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

***

```
00427AA0    53                push ebx
00427AA1    8B5C24 08         mov ebx,dword ptr ss:[esp+8]
00427AA5    55                push ebp
00427AA6    57                push edi
00427AA7    8BFB              mov edi,ebx
00427AA9    83C9 FF           or ecx,FFFFFFFF
00427AAC    33C0              xor eax,eax
00427AAE    33ED              xor ebp,ebp
00427AB0    F2:AE             repne scas byte ptr es:[edi]
00427AB2    F7D1              not ecx
00427AB4    49                dec ecx
00427AB5    33D2              xor edx,edx
00427AB7    85C9              test ecx,ecx
00427AB9    7E 36             jle short IdealDVD.00427AF1     ; 这里测试有没有填用户名
00427ABB    56                push esi
00427ABC    8BC1              mov eax,ecx                   ; 用户名位数放到 EAX 里
00427ABE    2BC2              sub eax,edx
00427AC0    48                dec eax                       ; 用户名位数减 1
00427AC1    66:85C0           test ax,ax
00427AC4    8BF0              mov esi,eax                   ; 用户名位数减 1 后放到 ESI 中
00427AC6    75 05             jnz short IdealDVD.00427ACD
00427AC8    BE 01000000       mov esi,1
00427ACD    66:0FB63C1A       movzx di,byte ptr ds:[edx+ebx] ; 用户名第一位 Ascii 放 di 里
00427AD2    25 FFFF0000       and eax,0FFFF
00427AD7    66:0FB60418       movzx ax,byte ptr ds:[eax+ebx] ; 用户名最后一位 Ascii 放 ax 里
00427ADC    0FAFC7            imul eax,edi                  ; EAX=用户名第一位 Ascii 乘第二位 Ascii
00427ADF    0FAFC6            imul eax,esi                  ; EAX=EAX*ESi（ESI=用户名位数减 1）
00427AE2    03E8              add ebp,eax                   ; 存到 EBP 里
00427AE4    42                inc edx                       ; 计数器加 1
00427AE5    3BD1              cmp edx,ecx                   ; 比较
00427AE7   7C D3              jl short IdealDVD.00427ABC      ; 循环
00427AE9    5E                pop esi
00427AEA    66:8BC5           mov ax,bp                     ; 取 BP 放到 AX 里，也就是取 EBP 的后四位放到 EAX 的后四位
00427AED    5F                pop edi
00427AEE    5D                pop ebp
00427AEF    5B                pop ebx
00427AF0    C3                retn
```

***

这里取用户名计算的方式是：用户名：`abcdef`，对应的计算方式就是：`a-f`，`b-e`，`c-d`，`d-c`，`e-b`，`f-a`

就相当于把字符串倒转过来，分别为 `a`，`b`，分别取 `a`，`b` 的 `Ascii` 做以上循环计算

***

```
00427BB0    6A FF          push -1                     ; 在这下断
00427BB2    68 E81E5100    push IdealDVD.00511EE8
00427BB7    64:A1 0000000> mov eax,dword ptr fs:[0]
00427BBD    50             push eax
00427BBE    64:8925 00000> mov dword ptr fs:[0],esp
00427BC5    83EC 0C        sub esp,0C
00427BC8    53             push ebx
00427BC9    55             push ebp
00427BCA    56             push esi
00427BCB    57             push edi
00427BCC    8D4C24 14      lea ecx,dword ptr ss:[esp+14]
00427BD0    E8 95EEFFFF    call <jmp.&MFC42.#540>
00427BD5    8B7C24 30      mov edi,dword ptr ss:[esp+30]
00427BD9    8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00427BDD    57             push edi
00427BDE    C74424 28 000> mov dword ptr ss:[esp+28],0
00427BE6    E8 47F0FFFF    call <jmp.&MFC42.#537>
00427BEB    8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00427BEF    C64424 24 01   mov byte ptr ss:[esp+24],1
00427BF4    E8 DDEEFFFF    call <jmp.&MFC42.#6282>
00427BF9    8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00427BFD    E8 CEEFFFF     call <jmp.&MFC42.#6283>
00427C02    8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00427C06    E8 7FEFFFFF    call <jmp.&MFC42.#4204>
00427C0B    6A 30          push 30
00427C0D    6A 4F          push 4F
00427C0F    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C13    E8 C6EFFFFF    call <jmp.&MFC42.#6876>
00427C18    83CE FF        or esi,FFFFFFFF
00427C1B    33C0           xor eax,eax
00427C1D    8BCE           mov ecx,esi
00427C1F    F2:AE          repne scas byte ptr es:[edi]
00427C21    F7D1           not ecx
00427C23    49             dec ecx
00427C24    83F9 14        cmp ecx,14                  ; 比较注册码是不是填够了$14 位，也就是说要 20 位注册码
00427C27    75 3E          jnz short IdealDVD.00427C67  ; 不够就跳了
00427C29    8B4424 2C      mov eax,dword ptr ss:[esp+2C]
00427C2D    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C31    50             push eax
00427C32    E8 FBEFFFFF    call <jmp.&MFC42.#537>
00427C37    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C3B    C64424 24 02   mov byte ptr ss:[esp+24],2
00427C40    E8 91EEFFFF    call <jmp.&MFC42.#6282>
00427C45    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C49    E8 82EEFFFF    call <jmp.&MFC42.#6283>
00427C4E    8B4424 18      mov eax,dword ptr ss:[esp+18]
00427C52    8B48 F8        mov ecx,dword ptr ds:[eax-8]
00427C55    85C9           test ecx,ecx
00427C57    75 25          jnz short IdealDVD.00427C7E
00427C59    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C5D    C64424 24 01   mov byte ptr ss:[esp+24],1
00427C62    E8 DFEDFFFF    call <jmp.&MFC42.#800>
00427C67    8D4C24 10      lea ecx,dword ptr ss:[esp+10]
00427C6B    C64424 24 00   mov byte ptr ss:[esp+24],0
00427C70    E8 D1EDFFFF    call <jmp.&MFC42.#800>
00427C75    897424 24      mov dword ptr ss:[esp+24],esi
00427C79    E9 F2000000    jmp IdealDVD.00427D70
00427C7E    50             push eax
00427C7F    E8 1CFEFFFF    call IdealDVD.00427AA0       ; 这个 Call 是对用户名进行计算的，要分析的
00427C84    8BC8           mov ecx,eax                 ; ECX=EAX
00427C86    BE 14000000    mov esi,14                  ; ESI=$14
00427C8B    81E1 FFFF0000  and ecx,0FFFF               ; ECX And $FFFF,66AF6A92 And $FFFF=6A92，不理解的手算一下就理解了
00427C91    83C4 04        add esp,4
00427C94    8BC1           mov eax,ecx                 ; EAX=ECX=6A92
00427C96    99             cdq
00427C97    F7FE           idiv esi                    ; $6A92 idiv $14,商在 EAX,余数在 EDX
00427C99    51             push ecx                    ; ECX 入栈
00427C9A    8D4C24 18      lea ecx,dword ptr ss:[esp+18]
00427C9E    68 802E5400    push IdealDVD.00542E80      ; ASCII "%04x"
00427CA3    51             push ecx
00427CA4    8BFA           mov edi,edx                 ; 这句是将余数放到 EDI
00427CA6    E8 5BEEFFFF    call <jmp.&MFC42.#2818>
00427CAB    83C4 0C        add esp,0C
00427CAE    8D4C24 14      lea ecx,dword ptr ss:[esp+14]
00427CB2    E8 D3EEFFFF    call <jmp.&MFC42.#4204>
00427CB7    8BCF           mov ecx,edi                 ; 又将余数放到 ECX
00427CB9    BF 14000000    mov edi,14
00427CBE    81E1 FFFF0000  and ecx,0FFFF
00427CC4    8B7424 14      mov esi,dword ptr ss:[esp+14]
00427CC8    8D41 05        lea eax,dword ptr ds:[ecx+5]
00427CCB    8A1E           mov bl,byte ptr ds:[esi]
00427CCD    99             cdq
00427CCE    F7FF           idiv edi
00427CD0    8B7C24 10      mov edi,dword ptr ss:[esp+10]
00427CD4    3A1C3A         cmp bl,byte ptr ds:[edx+edi] ; 比较第 8 位是不是用户名计算后的值的第 1 位
00427CD7    75 73          jnz short IdealDVD.00427D4C  ; 不相等就 Over
```



```
00427CD9    8D41 0B       lea eax,dword ptr ds:[ecx+B]

00427CDC    BD 14000000   mov ebp,14

00427CE1    99            cdq

00427CE2    F7FD          idiv ebp

00427CE4    8A5E 01       mov bl,byte ptr ds:[esi+1]

00427CE7    3A1C3A        cmp bl,byte ptr ds:[edx+edi]    ; 比较第 14 位是不是用户名计算后的值的第 2 位

00427CEA    75 60         jnz short IdealDVD.00427D4C     ; 不相等就 Over

00427CEC    8D41 12       lea eax,dword ptr ds:[ecx+12]

00427CEF    8A5E 02       mov bl,byte ptr ds:[esi+2]

00427CF2    99            cdq

00427CF3    F7FD          idiv ebp

00427CF5    3A1C3A        cmp bl,byte ptr ds:[edx+edi]    ; 比较第 1 位是不是用户名计算后的值的第 3 位

00427CF8    75 52         jnz short IdealDVD.00427D4C     ; 不相等就 Over

00427CFA    8D41 09       lea eax,dword ptr ds:[ecx+9]

00427CFD    8BCD          mov ecx,ebp

00427CFF    99            cdq

00427D00    F7F9          idiv ecx

00427D02    8A5E 03       mov bl,byte ptr ds:[esi+3]

00427D05    3A1C3A        cmp bl,byte ptr ds:[edx+edi]    ; 比较第 12 位是不是用户名计算后的值的第 4 位

00427D08    75 42         jnz short IdealDVD.00427D4C     ; 不相等就 Over

00427D0A    8D4C24 18     lea ecx,dword ptr ss:[esp+18]

00427D0E    C64424 24 01  mov byte ptr ss:[esp+24],1

00427D13    E8 2EEDFFFF   call <jmp.&MFC42.#800>

00427D18    8D4C24 10     lea ecx,dword ptr ss:[esp+10]

00427D1C    C64424 24 00  mov byte ptr ss:[esp+24],0

00427D21    E8 20EDFFFF   call <jmp.&MFC42.#800>

00427D26    8D4C24 14     lea ecx,dword ptr ss:[esp+14]

00427D2A    C74424 24 FFF>mov dword ptr ss:[esp+24],-1

00427D32    E8 0FEDFFFF   call <jmp.&MFC42.#800>

00427D37    5F            pop edi

00427D38    5E            pop esi

00427D39    5D            pop ebp

00427D3A    B0 01         mov al,1

00427D3C    5B            pop ebx

00427D3D    8B4C24 0C     mov ecx,dword ptr ss:[esp+C]

00427D41    64:890D 00000>mov dword ptr fs:[0],ecx

00427D48    83C4 18       add esp,18

00427D4B    C3            retn
```

一组可用的序列号：

- 用户名：`abcdef`
- 注册码：`923456769012A4567890`
