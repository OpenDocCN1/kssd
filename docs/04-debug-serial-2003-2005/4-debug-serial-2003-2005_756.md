
# mIRC6.16 注册算法分析 － 易

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-763.htm`](https://www.pediy.com/kssd/pediy07/pediy7-763.htm)

```
/////////////////////////////////
//  mIRC6.16 注册算法分析
//
//  Author: icytear
//  Date  : 2005-12-14
//
/////////////////////////////////
```

1. 查看没有加壳，哈哈省了不少麻烦 `:)`。
2. 用 `OllyDbg` 加载，`F9` 运行.... 弹出讨厌的 `"Evaluation time left: 28 days"`，点 `"Continue"` 继续试用，选择菜单 `Help-Register..` 打开注册对话框，`"Full Name"` 输入 `"icytear"`，`"Registration Code"` 输入 `"280060443"` 点 `"OK"`，弹出提示 `"注册码无效"`，这里可以试一下常用的 `MessageBoxA(W)` 断点，`HIHI`，没有用这个函数。
3. 那就来个绝的 `bp SendDlgItemMessageA`，点 `"OK"`，程序断在 `SendDlgItemMessageA` 处，`ALT+F9` 返回到 `mIRC` 领空

```
代码:

* * *

004C7BAA    push    mirc.005A07E0           ; 
/lParam = 5A07E0
004C7BAF    push    3E7                     ; 
|wParam = 3E7
004C7BB4    push    0D                      ; 
|Message = WM_GETTEXT
004C7BB6    push    83                      ; 
|ControlID = 83 (131.)
004C7BBB    push    esi                     ; 
|hWnd
004C7BBC    call    edi                     ; 
\SendDlgItemMessageA
004C7BBE    push    mirc.005A0BC8           ; 
/lParam = 5A0BC8          <===ALT+F9 程序返回到这里
004C7BC3    push    3E7                     ; 
|wParam = 3E7
004C7BC8    push    0D                      ; 
|Message = WM_GETTEXT
004C7BCA    push    84                      ; 
|ControlID = 84 (132.)
004C7BCF    push    esi                     ; 
|hWnd
004C7BD0    call    edi                     ; 
\SendDlgItemMessageA
004C7BD2    mov     edx,mirc.005A0BC8       ;  
ASCII "280060443"
004C7BD7    mov     ecx,mirc.005A07E0       ;  
ASCII "icytear"
004C7BDC    call    mirc.004C7600           ;      
                   ==== >关键函数
004C7BE1    test    eax,eax
004C7BE3    je      mirc.004C7CCE           
;==== >关键跳转,跳转注册失败

* * *
```

关闭 `SendDlgItemMessageA` 断点，进入 `F7` 跟进 `004c7600`

```
代码:

* * *

004C7600    sub     esp,208
004C7606    push    esi
004C7607    push    edi
004C7608    mov     esi,edx
004C760A    mov     edi,ecx
004C760C    mov     edx,104
004C7611    lea     ecx,dword ptr ss:[esp+8]
004C7615    call    mirc.0041EF30
004C761A    mov     edx,104
004C761F    lea     ecx,dword ptr ss:[esp+10C]
004C7626    call    mirc.0041EF30
004C762B    push    edi
004C762C    mov     edx,104
004C7631    lea     ecx,dword ptr ss:[esp+C]
004C7635    call    mirc.0041EE40
004C763A    push    esi
004C763B    mov     edx,104
004C7640    lea     ecx,dword ptr ss:[esp+110]
004C7647    call    mirc.0041EE40
004C764C    lea     edx,dword ptr ss:[esp+10C]
004C7653    lea     ecx,dword ptr ss:[esp+8]
004C7657    call    mirc.004C7400            
;====>跟进去
004C765C    test    eax,eax
004C765E    je      short mirc.004C766E     
如果上面注册失败跳转        
004C7660    pop     edi
004C7661    mov     eax,1
004C7666    pop     esi
004C7667    add     esp,208
004C766D    retn
004C766E    mov     edx,104
004C7673    lea     ecx,dword ptr ss:[esp+8]
004C7677    call    mirc.0041EF30
004C767C    mov     edx,104
004C7681    lea     ecx,dword ptr ss:[esp+10C]
004C7688    call    mirc.0041EF30
004C768D    push    edi
004C768E    mov     edx,104
004C7693    lea     ecx,dword ptr ss:[esp+C]
004C7697    call    mirc.0041EE40
004C769C    push    esi
004C769D    mov     edx,104
004C76A2    lea     ecx,dword ptr ss:[esp+110]
004C76A9    call    mirc.0041EE40
004C76AE    mov     edx,2
004C76B3    lea     ecx,dword ptr ss:[esp+8]
004C76B7    call    mirc.004793B0            
;====>这里对用户名进行转换
004C76BC    lea     edx,dword ptr ss:[esp+10C]
004C76C3    lea     ecx,dword ptr ss:[esp+8]
004C76C7    call    mirc.004C7400            
;====>又调用一次
004C76CC    neg     eax
004C76CE    sbb     eax,eax
004C76D0    pop     edi
004C76D1    neg     eax
004C76D3    pop     esi
004C76D4    add     esp,208
004C76DA    retn

* * *
```

; 跟进 `004c7400`

```
代码:

* * *

sub_004c7400:
004C7400    sub     esp,0A4
004C7406    push    ebx
004C7407    push    ebp
004C7408    push    esi
004C7409    mov     esi,0B
004C740E    mov     eax,0A
004C7413    push    edi
004C7414    mov     edi,ecx
004C7416    mov     ecx,0E
004C741B    mov     dword ptr ss:[esp+14],esi
004C741F    mov     ebp,6
004C7424    mov     ebx,10
004C7429    mov     dword ptr ss:[esp+3C],esi
004C742D    mov     dword ptr ss:[esp+50],esi
004C7431    mov     dword ptr ss:[esp+64],esi
004C7435    mov     dword ptr ss:[esp+70],esi
004C7439    mov     esi,8
004C743E    mov     dword ptr ss:[esp+10],edx
004C7442    mov     edx,0C
004C7447    mov     dword ptr ss:[esp+28],ecx
004C744B    mov     dword ptr ss:[esp+38],eax
004C744F    mov     dword ptr ss:[esp+44],ecx
004C7453    mov     dword ptr ss:[esp+48],ecx
004C7457    mov     dword ptr ss:[esp+58],ecx
004C745B    mov     dword ptr ss:[esp+5C],ecx
004C745F    mov     ecx,4
004C7464    mov     dword ptr ss:[esp+74],eax
004C7468    mov     dword ptr ss:[esp+7C],eax
004C746C    mov     dword ptr ss:[esp+80],eax
004C7473    mov     dword ptr ss:[esp+94],eax
004C747A    mov     dword ptr ss:[esp+A4],eax
004C7481    mov     eax,edi
004C7483    mov     dword ptr ss:[esp+78],esi
004C7487    mov     dword ptr ss:[esp+88],esi
004C748E    mov     dword ptr ss:[esp+A0],esi
004C7495    mov     dword ptr ss:[esp+18],ebp
004C7499    mov     dword ptr ss:[esp+1C],11
004C74A1    mov     dword ptr ss:[esp+20],edx
004C74A5    mov     dword ptr ss:[esp+24],edx
004C74A9    mov     dword ptr ss:[esp+2C],5
004C74B1    mov     dword ptr ss:[esp+30],edx
004C74B5    mov     dword ptr ss:[esp+34],ebx
004C74B9    mov     dword ptr ss:[esp+40],ebp
004C74BD    mov     dword ptr ss:[esp+4C],4
004C74C5    mov     dword ptr ss:[esp+54],ebp
004C74C9    mov     dword ptr ss:[esp+60],ecx
004C74CD    mov     dword ptr ss:[esp+68],9
004C74D5    mov     dword ptr ss:[esp+6C],edx
004C74D9    mov     dword ptr ss:[esp+84],ebx
004C74E0    mov     dword ptr ss:[esp+8C],ecx
004C74E7    mov     dword ptr ss:[esp+90],ebp
004C74EE    mov     dword ptr ss:[esp+98],edx
004C74F5    mov     dword ptr ss:[esp+9C],ebx
004C74FC    mov     dword ptr ss:[esp+A8],ecx
004C7503    mov     dword ptr ss:[esp+AC],ebx
004C750A    mov     dword ptr ss:[esp+B0],0

* * *
```

以上代码生成一个表如下

```
DWORD tb[] = {
0x0B, 0x06, 0x11, 0x0C,
0x0C, 0x0E, 0x05, 0x0C,
0x10, 0x0A, 0x0B, 0x06,
0x0E, 0x0E, 0x04, 0x0B,
0x06, 0x0E, 0x0E, 0x04,
0x0B, 0x09, 0x0C, 0x0B,
0x0A, 0x08, 0x0A, 0x0A,
0x10, 0x08, 0x04, 0x06,
0x0A, 0x0C, 0x10, 0x08,
0x0A, 0x04}
```

```
代码:

* * *

004C7515    lea     esi,dword ptr ds:[eax+1]
004C7518    mov     dl,byte ptr ds:[eax]
004C751A    inc     eax
004C751B    test    dl,dl
004C751D    jnz     short mirc.004C7518
004C751F    sub     eax,esi                 ;
计算字符串长度
004C7521    cmp     eax,ecx
004C7523    jnb     short mirc.004C7532     ;
输入的用户名小于 4 个字符直接返回 0,注册失败
004C7525    pop     edi
004C7526    pop     esi
004C7527    pop     ebp
004C7528    xor     eax,eax                 ;
失败,返回 0
004C752A    pop     ebx
004C752B    add     esp,0A4
004C7531    retn
004C7532    mov     ebx,dword ptr ss:[esp+10]
004C7536    push    2D                      ;
字符 "-"
004C7538    push    ebx
004C7539    call    <mirc._strchr>
004C753E    mov     esi,eax
004C7540    add     esp,8
004C7543    test    esi,esi
004C7545    je      short mirc.004C7525
;上面代码在 RegisterCode 串中查找字符 "-", 如果没

* * *
```



有找到跳转到 `004C7525` 返回失败.

重新输入注册码 `"2800-60443"`, 跟到以下代码:

```
004C7547    push    ebx
004C7548    mov     byte ptr ds:[esi],0
004C754B    call    <mirc._aotl>
004C7550    add     esp,4
004C7553    mov     byte ptr ds:[esi],2D
004C7556    inc     esi
004C7557    mov     ebp,eax
004C7559    cmp     byte ptr ds:[esi],0
004C755C    je      short mirc.004C7525
004C755E    push    esi
004C755F    call    <mirc._aotl>
004C7564    mov     ecx,edi
004C7566    add     esp,4
;以第一个"-"字符把注册码分成两部分, 再分别转换为两个整数
004C7569    mov     dword ptr ss:[esp+10],eax
004C756D    lea     edx,dword ptr ds:[ecx+1]
004C7570    mov     al,byte ptr ds:[ecx]
004C7572    inc     ecx
004C7573    test    al,al
004C7575    jnz     short mirc.004C7570
004C7577    sub     ecx,edx                    ; 计算字符串长度(用户名或其变形)
004C7579    mov     esi,3
004C757E    xor     edx,edx
004C7580    xor     ebx,ebx
004C7582    cmp     ecx,esi
004C7584    jle     short mirc.004C75A8        ; 比较用户名的长度是否大于 3
004C7586    jmp     short mirc.004C7590
004C7588    lea     esp,dword ptr ss:[esp]
004C758F    nop
```

下面是注册算法 1, 用用户名第 3 位(0 位基址)以后的字符的值与查表 `tb` 得到的值乘积之和.

```
004C7590    movzx   eax,byte ptr ds:[esi+edi]
004C7594    imul    eax,dword ptr ss:[esp+edx*4+14]
004C7599    add     ebx,eax
004C759B    inc     edx
004C759C    cmp     edx,26
004C759F    jle     short mirc.004C75A3
004C75A1    xor     edx,edx
004C75A3    inc     esi
004C75A4    cmp     esi,ecx
004C75A6    jl      short mirc.004C7590
004C75A8    cmp     ebp,ebx           ;比较 1
004C75AA    jnz     mirc.004C7525     ;输入的注册码的第一部分和计算出的值不等,跳转,注册失败
```

用 C 描述:

```
j=0;
for(i=3; i < strlen(username); i++)
{
    RegCode1 += (username[i] * tb[j]);
    j++;
    if(j > sizeof(tb))
        j = 0;
}
```

```
004C75B0    mov     esi,3
004C75B5    xor     edx,edx
004C75B7    xor     ebx,ebx
004C75B9    cmp     ecx,esi
004C75BB    jle     short mirc.004C75E0
004C75BD    lea     ecx,dword ptr ds:[ecx]
```

以下是注册算法 2, 从用户名第 3 位(0 位基址)开始, 取其值和前一个字符的值相乘,然后再查表 `tb` 得到的相应值相乘, 再取其和.

```
004C75C0    movzx   eax,byte ptr ds:[esi+edi-1]
004C75C5    movzx   ebp,byte ptr ds:[esi+edi]
004C75C9    imul    eax,ebp
004C75CC    imul    eax,dword ptr ss:[esp+edx*4+14]
004C75D1    add     ebx,eax
004C75D3    inc     edx
004C75D4    cmp     edx,26
004C75D7    jle     short mirc.004C75DB
004C75D9    xor     edx,edx
004C75DB    inc     esi
004C75DC    cmp     esi,ecx
004C75DE    jl      short mirc.004C75C0
```

用 C 描述:

```
j = 0;
for(i=3; i < strlen(username); i++)
{
    RegCode2 += (username[i-1] * username[i] * tb[j]);
    j++;
    if(j > sizeof(tb))
        j = 0;
}
```

```
004C75E0    mov     ecx,dword ptr ss:[esp+10]
004C75E4    pop     edi
004C75E5    pop     esi
004C75E6    xor     eax,eax
004C75E8    cmp     ecx,ebx              ;比较 2
004C75EA    pop     ebp
004C75EB    sete    al
004C75EE    pop     ebx
004C75EF    add     esp,0A4
004C75F5    retn
;到这里已经生成一次注册码了, 回到第一次调用 004C7400 的地方
004C7657    call    mirc.004C7400        ;====>跟进去
004C765C    test    eax,eax
004C765E    je      short mirc.004C766E  ; 如果上面注册失败跳转, 下面会对用户名进行转换后继续注册
004C7660    pop     edi
004C7661    mov     eax,1
004C7666    pop     esi
004C7667    add     esp,208
004C766D    retn
```

如果我们在 `004C7657` 校验注册码失败的话回返回 0,然后跳转到下面代码

```
004C766E    mov     edx,104
004C7673    lea     ecx,dword ptr ss:[esp+8]
004C7677    call    mirc.0041EF30
004C767C    mov     edx,104
004C7681    lea     ecx,dword ptr ss:[esp+10C]
004C7688    call    mirc.0041EF30
004C768D    push    edi
004C768E    mov     edx,104
004C7693    lea     ecx,dword ptr ss:[esp+C]
004C7697    call    mirc.0041EE40
004C769C    push    esi
004C769D    mov     edx,104
004C76A2    lea     ecx,dword ptr ss:[esp+110]
004C76A9    call    mirc.0041EE40
004C76AE    mov     edx,2
004C76B3    lea     ecx,dword ptr ss:[esp+8]
004C76B7    call    mirc.004793B0        ;====>这里对用户名进行转换，跟进
004C76BC    lea     edx,dword ptr ss:[esp+10C]
004C76C3    lea     ecx,dword ptr ss:[esp+8]
004C76C7    call    mirc.004C7400        ;====>又调用一次, 这次注册失败的话,就没有机会了
004C76CC    neg     eax
004C76CE    sbb     eax,eax
004C76D0    pop     edi
004C76D1    neg     eax
004C76D3    pop     esi
004C76D4    add     esp,208
004C76DA    retn
```

跟进 `004793B0`:

```
sub_004793b0:
004793B0    sub     esp,18
004793B3    push    ebx
004793B4    push    ebp
004793B5    push    esi
004793B6    mov     esi,ecx
004793B8    mov     eax,esi
004793BA    mov     dword ptr ss:[esp+1C],edx
004793BE    push    edi
004793BF    lea     edx,dword ptr ds:[eax+1]
004793C2    mov     cl,byte ptr ds:[eax]
004793C4    inc     eax
004793C5    test    cl,cl
004793C7    jnz     short mirc.004793C2
004793C9    sub     eax,edx              ; 计算字符串长度(用户名)
004793CB    mov     edi,eax
004793CD    mov     eax,mirc.0059B9F8    ; 0059B9F8 这里存放了一个全局表 g_tb0[]
004793D2    mov     dword ptr ss:[esp+1C],edi
004793D6    lea     edx,dword ptr ds:[eax+1]
004793D9    lea     esp,dword ptr ss:[esp]
004793E0    mov     cl,byte ptr ds:[eax]
004793E2    inc     eax
004793E3    test    cl,cl
004793E5    jnz     short mirc.004793E0
004793E7    sub     eax,edx              ; 计算 g_tb0[]长度
```

表 `g_tb0` 如下:

```
char g_tb0[] = {
    3, 7, 4, 2, 3, 8, 4, 6,
    9, 3, 9, 8, 6, 5, 2, 6,
    5, 2, 5, 4, 8, 9, 2, 6,
    3, 4, 2, 7, 5, 6, 8, 9,
    3, 2, 3, 5, 6, 8, 4, 0
};
```



```asm
mov ebp,eax
mov eax,dword ptr ss:[esp+20]
xor ebx,ebx
cmp eax,ebx
mov dword ptr ss:[esp+24],ebp
mov dword ptr ss:[esp+18],ebx
mov dword ptr ss:[esp+14],ebx
je short mirc.00479457
xor ecx,ecx
cmp edi,ebx
mov dword ptr ss:[esp+18],ebx
mov dword ptr ss:[esp+14],ebx
jle short mirc.00479457
nop
movzx eax,byte ptr ds:[ecx+esi]
add ebx,eax
cmp ebx,ebp
jle short mirc.00479426
lea ebx,dword ptr ds:[ebx]
sub ebx,ebp
cmp ebx,ebp
jg short mirc.00479420
mov edi,dword ptr ss:[esp+14]
add edi,eax
cmp edi,1E
mov dword ptr ss:[esp+14],edi
jl short mirc.00479448
mov eax,88888889
mul edi
shr edx,4
imul edx,edx,-1E
add edi,edx
mov dword ptr ss:[esp+14],edi
mov eax,dword ptr ss:[esp+1C]
inc ecx
cmp ecx,eax
jl short mirc.00479410
mov dword ptr ss:[esp+18],ebx
mov edi,eax
xor eax,eax
xor ecx,ecx
test edi,edi
jle short mirc.00479475
nop
mov dl,byte ptr ds:[eax+59B9F8]          ; 用户名转换
add byte ptr ds:[ecx+esi],dl
inc eax
cmp eax,ebp
jl short mirc.00479470
xor eax,eax
inc ecx
cmp ecx,edi
jl short mirc.00479460
mov eax,esi
xor ecx,ecx
lea edi,dword ptr ds:[eax+1]
lea esp,dword ptr ss:[esp]
mov dl,byte ptr ds:[eax]
inc eax
test dl,dl
jnz short mirc.00479480
sub eax,edi                                ; 计算字符串长度
dec eax
test eax,eax
jle short mirc.004794AA
mov edi,edi
mov dl,byte ptr ds:[ecx+esi]               ; 字符串反序
mov byte ptr ss:[esp+13],dl
mov dl,byte ptr ds:[eax+esi]
mov byte ptr ds:[ecx+esi],dl
mov dl,byte ptr ss:[esp+13]
inc ecx
mov byte ptr ds:[eax+esi],dl
dec eax
cmp ecx,eax
jl short mirc.00479490
mov eax,esi
lea edx,dword ptr ds:[eax+1]
nop
mov cl,byte ptr ds:[eax]
inc eax
test cl,cl
jnz short mirc.004794B0
sub eax,edx                                ; 计算字符串长度
lea ecx,dword ptr ds:[eax-1]
test ecx,ecx
jle short mirc.004794DB
dec ecx
shr ecx,1
lea eax,dword ptr ds:[esi+1]
inc ecx
mov dl,byte ptr ds:[eax-1]                 ; 字符串每两个字符反序
mov bl,byte ptr ds:[eax]
mov byte ptr ds:[eax-1],bl
mov byte ptr ds:[eax],dl
add eax,2
dec ecx
jnz short mirc.004794C7
mov ebx,dword ptr ss:[esp+18]
mov eax,dword ptr ss:[esp+20]
test eax,eax
je mirc.004795E0
cmp eax,1
jnz short mirc.00479500
call mirc.00572EFA
cdq
mov ecx,7
idiv ecx
inc edx
mov dword ptr ss:[esp+18],edx
jmp short mirc.0047950C
mov dword ptr ss:[esp+18],1
mov edx,dword ptr ss:[esp+18]
lea eax,dword ptr ds:[edx+ebx]
cmp eax,ebp
jle short mirc.00479515
sub eax,ebp
mov edi,esi
lea ebp,dword ptr ds:[edi+1]
lea ebx,dword ptr ds:[ebx]
mov cl,byte ptr ds:[edi]
inc edi
test cl,cl
jnz short mirc.00479520
sub edi,ebp                                ; 字符串长度
xor ecx,ecx
test edi,edi
jle short mirc.0047954D
nop
mov dl,byte ptr ds:[eax+59B9F8]            ; 又进行一次转换
add byte ptr ds:[ecx+esi],dl
mov edx,dword ptr ss:[esp+24]
inc eax
cmp eax,edx
jl short mirc.00479544
xor eax,eax
inc ecx
cmp ecx,edi
jl short mirc.00479530
mov edx,dword ptr ss:[esp+18]
mov al,byte ptr ss:[esp+14]
add bl,64
mov byte ptr ds:[edi+esi],bl               ; 加了一个字节
inc edi
add al,6E
mov byte ptr ds:[edi+esi],al               ; 加一个字节
inc edi
lea ebp,dword ptr ds:[edx+edi]
cmp edi,ebp
jge short mirc.00479581
call mirc.00572EFA
cdq
mov ecx,64
idiv ecx
add dl,20
mov byte ptr ds:[edi+esi],dl               ; 加一个字节
inc edi
cmp edi,ebp
jl short mirc.00479565
mov edx,dword ptr ss:[esp+18]
add dl,78
mov byte ptr ds:[edi+esi],dl               ; 加一个字节
mov eax,esi
mov byte ptr ds:[edi+esi+1],0              ; 串结束
xor ecx,ecx
lea edi,dword ptr ds:[eax+1]
mov dl,byte ptr ds:[eax]
inc eax
test dl,dl
jnz short mirc.00479593
sub eax,edi
dec eax
test eax,eax
jle short mirc.004795B3
mov bl,byte ptr ds:[eax+esi]               ; 字符串反序
mov dl,byte ptr ds:[ecx+esi]
mov byte ptr ds:[ecx+esi],bl
inc ecx
mov byte ptr ds:[eax+esi],dl
dec eax
cmp ecx,eax
jl short mirc.004795A1
mov eax,esi
lea edx,dword ptr ds:[eax+1]
mov cl,byte ptr ds:[eax]
inc eax
test cl,cl
jnz short mirc.004795B8
sub eax,edx
lea ecx,dword ptr ds:[eax-1]
test ecx,ecx
jle short mirc.004795E0
dec ecx
shr ecx,1
lea eax,dword ptr ds:[esi+1]
inc ecx
nop
mov dl,byte ptr ds:[eax-1]                 ; 每两个字符反序
mov bl,byte ptr ds:[eax]
mov byte ptr ds:[eax-1],bl
mov byte ptr ds:[eax],dl
add eax,2
dec ecx
jnz short mirc.004795D0
pop edi
pop esi
pop ebp
pop ebx
add esp,18
retn
```

上面函数对用户名进行了转化以后又调用 `004C7400` 注册。



这里会生成另外一个注册码. 如果在比较失败的话, 注册就失败了.

### 4. 注册机

```cpp
代码:

#include <iostream>
#include <windows.h>

using namespace std;

BOOL generator(char *username, DWORD *pRegCode1,
DWORD *pRegCode2)
{
  int i, j, len;
  DWORD tb[] ={
      0x0B, 0x06, 0x11, 0x0C,
      0x0C, 0x0E, 0x05, 0x0C,
      0x10, 0x0A, 0x0B, 0x06,
      0x0E, 0x0E, 0x04, 0x0B,
      0x06, 0x0E, 0x0E, 0x04,
      0x0B, 0x09, 0x0C, 0x0B,
      0x0A, 0x08, 0x0A, 0x0A,
      0x10, 0x08, 0x04, 0x06,
      0x0A, 0x0C, 0x10, 0x08,
      0x0A, 0x04};

  len = strlen(username);
  j=0;
  for(i=3; i < len; i++)
  {
    *pRegCode1 += (username[i] *
tb[j]);
    j++;
    if(j > sizeof(tb))
      j = 0;
  }
  j = 0;
  for(i=3; i < len; i++)
  {
    *pRegCode2 += (username[i-1] *
username[i] * tb[j]);
    j++;
    if(j > sizeof(tb))
      j = 0;
  }
  return TRUE;
}


int main()
{
  char username[256];
  DWORD RegCode1, RegCode2;

  cout << "please user name:";
  cin >> username;
  if(strlen(username)<4){
    cout << "username length need
great 4 ." << endl;
    return 1;
  }
  RegCode1 = 0;
  RegCode2 = 0;
  generator(username, &RegCode1,
&RegCode2);
  cout << RegCode1 << "-" << RegCode2 <<
endl;

  return 0;
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
