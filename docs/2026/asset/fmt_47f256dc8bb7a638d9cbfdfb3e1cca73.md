
## 4. 注册机

上面函数对用户名进行了转化以后又调用`004C7400`注册, 这里会生成另外一个注册码。如果在比较失败的话,注册就失败了。

```
004793E9    mov		ebp,eax

004793EB    mov		eax,dword ptr ss:[esp+20]

004793EF    xor		ebx,ebx

004793F1    cmp		eax,ebx

004793F3    mov		dword ptr ss:[esp+24],ebp

004793F7    mov		dword ptr ss:[esp+18],ebx

004793FB    mov		dword ptr ss:[esp+14],ebx

004793FF    je		short mirc.00479457

00479401    xor		ecx,ecx

00479403    cmp		edi,ebx

00479405    mov		dword ptr ss:[esp+18],ebx

00479409    mov		dword ptr ss:[esp+14],ebx

0047940D    jle		short mirc.00479457

0047940F    nop

00479410    /movzx	eax,byte ptr ds:[ecx+esi]

00479414    |add	ebx,eax

00479416    |cmp	ebx,ebp

00479418    |jle	short mirc.00479426

0047941A    |lea	ebx,dword ptr ds:[ebx]

00479420    |/sub	ebx,ebp

00479422    ||cmp	ebx,ebp

00479424    |\jg	short mirc.00479420

00479426    |mov	edi,dword ptr ss:[esp+14]

0047942A    |add	edi,eax

0047942C    |cmp	edi,1E

0047942F    |mov	dword ptr ss:[esp+14],edi

00479433    |jl		short mirc.00479448

00479435    |mov	eax,88888889

0047943A    |mul	edi

0047943C    |shr	edx,4

0047943F    |imul	edx,edx,-1E

00479442    |add	edi,edx

00479444    |mov	dword ptr ss:[esp+14],edi

00479448    |mov	eax,dword ptr ss:[esp+1C]

0047944C    |inc	ecx

0047944D    |cmp	ecx,eax

0047944F    \jl		short mirc.00479410

00479451    mov		dword ptr ss:[esp+18],ebx

00479455    mov		edi,eax

00479457    xor		eax,eax

00479459    xor		ecx,ecx

0047945B    test	edi,edi

0047945D    jle		short mirc.00479475

0047945F    nop

00479460    /mov	dl,byte ptr 

ds:[eax+59B9F8]             ;  用户名转换

00479466    |add	byte ptr ds:[ecx+esi],dl

00479469    |inc	eax

0047946A    |cmp	eax,ebp

0047946C    |jl		short mirc.00479470

0047946E    |xor	eax,eax

00479470    |inc	ecx

00479471    |cmp	ecx,edi

00479473    \jl		short mirc.00479460

00479475    mov		eax,esi

00479477    xor		ecx,ecx

00479479    lea		edi,dword ptr ds:[eax+1]

0047947C    lea		esp,dword ptr ss:[esp]

00479480    /mov	dl,byte ptr ds:[eax]

00479482    |inc	eax

00479483    |test	dl,dl

00479485    \jnz	short mirc.00479480

00479487    sub		eax,edi                   

               ;  计算字符串长度

00479489    dec	    eax

0047948A    test	eax,eax

0047948C    jle		short mirc.004794AA

0047948E    mov		edi,edi

00479490    /mov	dl,byte ptr ds:[ecx+esi]  

              ;  字符串反序

00479493    |mov	byte ptr ss:[esp+13],dl

00479497    |mov	dl,byte ptr ds:[eax+esi]

0047949A    |mov	byte ptr ds:[ecx+esi],dl

0047949D    |mov	dl,byte ptr ss:[esp+13]

004794A1    |inc	ecx

004794A2    |mov	byte ptr ds:[eax+esi],dl

004794A5    |dec	eax

004794A6    |cmp	ecx,eax

004794A8    \jl		short mirc.00479490

004794AA    mov		eax,esi

004794AC    lea		edx,dword ptr ds:[eax+1]

004794AF    nop

004794B0    /mov	cl,byte ptr ds:[eax]

004794B2    |inc	eax

004794B3    |test	cl,cl

004794B5    \jnz	short mirc.004794B0

004794B7    sub		eax,edx                   

               ;  计算字符串长度

004794B9    lea		ecx,dword ptr ds:[eax-1]

004794BC    test	ecx,ecx

004794BE    jle		short mirc.004794DB

004794C0    dec		ecx

004794C1    shr		ecx,1

004794C3    lea		eax,dword ptr ds:[esi+1]

004794C6    inc		ecx

004794C7    /mov	dl,byte ptr ds:[eax-1]    

              ;  字符串每两个字符反序

004794CA    |mov	bl,byte ptr ds:[eax]

004794CC    |mov	byte ptr ds:[eax-1],bl

004794CF    |mov	byte ptr ds:[eax],dl

004794D1    |add	eax,2

004794D4    |dec	ecx

004794D5    \jnz	short mirc.004794C7

004794D7    mov		ebx,dword ptr ss:[esp+18]

004794DB    mov		eax,dword ptr ss:[esp+20]

004794DF    test	eax,eax

004794E1    je		mirc.004795E0

004794E7    cmp		eax,1

004794EA    jnz		short mirc.00479500

004794EC    call	mirc.00572EFA

004794F1    cdq

004794F2    mov		ecx,7

004794F7    idiv	ecx

004794F9    inc		edx

004794FA    mov		dword ptr ss:[esp+18],edx

004794FE    jmp		short mirc.0047950C

00479500    mov		dword ptr ss:[esp+18],1

00479508    mov		edx,dword ptr ss:[esp+18]

0047950C    lea		eax,dword ptr 
ds:[edx+ebx]

0047950F    cmp		eax,ebp

00479511    jle		short mirc.00479515

00479513    sub		eax,ebp

00479515    mov		edi,esi

00479517    lea		ebp,dword ptr ds:[edi+1]

0047951A    lea		ebx,dword ptr ds:[ebx]

00479520    /mov	cl,byte ptr ds:[edi]

00479522    |inc	edi

00479523    |test	cl,cl

00479525    \jnz	short mirc.00479520

00479527    sub		edi,ebp                   

               ;  字符串长度

00479529    xor		ecx,ecx

0047952B    test	edi,edi

0047952D    jle		short mirc.0047954D

0047952F    nop

00479530    /mov	dl,byte ptr 
ds:[eax+59B9F8]             ;  又进行一次转换

00479536    |add	byte ptr ds:[ecx+esi],dl

00479539    |mov	edx,dword ptr ss:[esp+24]

0047953D    |inc	eax

0047953E    |cmp	eax,edx

00479540    |jl		short mirc.00479544

00479542    |xor	eax,eax

00479544    |inc	ecx

00479545    |cmp	ecx,edi

00479547    \jl		short mirc.00479530

00479549    mov		edx,dword ptr ss:[esp+18]

0047954D    mov		al,byte ptr ss:[esp+14]

00479551    add		bl,64

00479554    mov		byte ptr ds:[edi+esi],bl  

               ;  加了一个字节

00479557    inc		edi

00479558    add		al,6E

0047955A    mov		byte ptr ds:[edi+esi],al  

               ;  加一个字节

0047955D    inc		edi

0047955E    lea		ebp,dword ptr 
ds:[edx+edi]

00479561    cmp		edi,ebp

00479563    jge		short mirc.00479581

00479565    /call	mirc.00572EFA

0047956A    |cdq

0047956B    |mov	ecx,64

00479570    |idiv	ecx

00479572    |add	dl,20

00479575    |mov	byte ptr ds:[edi+esi],dl  

              ;  加一个字节

00479578    |inc	edi

00479579    |cmp	edi,ebp

0047957B    \jl		short mirc.00479565

0047957D    mov		edx,dword ptr ss:[esp+18]

00479581    add		dl,78

00479584    mov		byte ptr ds:[edi+esi],dl  

               ;  加一个字节

00479587    mov		eax,esi

00479589    mov		byte ptr ds:[edi+esi+1],0 

               ;  串结束

0047958E    xor		ecx,ecx

00479590    lea		edi,dword ptr ds:[eax+1]

00479593    /mov	dl,byte ptr ds:[eax]

00479595    |inc	eax

00479596    |test	dl,dl

00479598    \jnz	short mirc.00479593

0047959A    sub		eax,edi

0047959C    dec		eax

0047959D    test	eax,eax

0047959F    jle		short mirc.004795B3

004795A1    /mov	bl,byte ptr ds:[eax+esi]  

              ;  字符串反序

004795A4    |mov	dl,byte ptr ds:[ecx+esi]

004795A7    |mov	byte ptr ds:[ecx+esi],bl

004795AA    |inc	ecx

004795AB    |mov	byte ptr ds:[eax+esi],dl

004795AE    |dec	eax

004795AF    |cmp	ecx,eax

004795B1    \jl		short mirc.004795A1

004795B3    mov		eax,esi

004795B5    lea		edx,dword ptr ds:[eax+1]

004795B8    /mov	cl,byte ptr ds:[eax]

004795BA    |inc	eax

004795BB    |test	cl,cl

004795BD    \jnz	short mirc.004795B8

004795BF    sub		eax,edx

004795C1    lea		ecx,dword ptr ds:[eax-1]

004795C4    test	ecx,ecx

004795C6    jle		short mirc.004795E0

004795C8    dec		ecx

004795C9    shr		ecx,1

004795CB    lea		eax,dword ptr ds:[esi+1]

004795CE    inc		ecx

004795CF    nop

004795D0    /mov	dl,byte ptr ds:[eax-1]    

              ;  每两个字符反序

004795D3    |mov	bl,byte ptr ds:[eax]

004795D5    |mov	byte ptr ds:[eax-1],bl

004795D8    |mov	byte ptr ds:[eax],dl

004795DA    |add	eax,2

004795DD    |dec	ecx

004795DE    \jnz	short mirc.004795D0

004795E0    pop		edi

004795E1    pop		esi

004795E2    pop		ebp

004795E3    pop		ebx

004795E4    add		esp,18

004795E7    retn
```
