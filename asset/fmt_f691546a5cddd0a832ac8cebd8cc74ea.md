
```
    mov edx,EntryPoint

    mov dword ptr [edi],edx

    add edi,4

    call GenerateRandOpCode

    call GFI

    mov byte ptr [edi],0C3h         ;Ret

    inc edi

lbl_Exit5:

    popall

    ret

G4Code endp
```

;整个程序的核心(变形引擎)

;   假设解密代码如下：(可分为 4 部分)

;   mov reg00,SizeOfCode             1

;   mov reg11,EntryPointOfCode       2

;Rep:

;   xor dowrd ptr[reg00+reg11],key   3

;   dec reg00                        4

;   jnz Rep

;为了增强变形引擎的强度可以把上面的代码分解(或叫变换)

;如下：

;    第一部分代码可以有下面 5 种选择

;   mov reg00,SizeOfCode   --->  mov reg00,xxx    add reg00,xxx      0 (其中 `reg00` 为随机选取的寄存器)

;                               push xxx         pop reg00          1

;                               mov reg00,xxx    xor reg00,xxx      2

;                               mov reg00,xxx    not reg00          3

;                               mov reg00,xxx    rol reg00,xx       4

;   

;   第二部分同样有 5 种选择

;   mov reg11,EntryPointOfCode 也可作同样处理

;   第三部分有 8 种选择

;       0

;同理  xor dword ptr [reg00+reg11],key 也可变换如下：

;              1     2    3   4   5    6   7

;          -->add   sub  inc  dec not  rol ror... 等等

;

;   第四部分有 2 种选择

;          0                 1

;      dec reg00   --->   Loop xxx

;      jnz xxx

;param `Buf`   存放变形以后代码的缓冲区

;param `StartAddress` 欲加密代码的首地址

;param `dwLength`     欲加密代码的长度

;

```
EncryptSelected dd 0    ;保存被选择的加密方式

G2Selected dd 0         ;保存第二部分产生的方式

AddressOfCode1 dd 0     ;相应的地址

AddressOfCode2 dd 0

key dd 0

PolyEngine proc Buf:dword,StartAddress:dword,dwLength:dword,VA:dword,EntryPoint:dword

    call PolyStart

PolyStart:

    pop esi

    sub esi,offset PolyStart       ;重定位

    call Randomize                 ;初始化随机种子

    push 8

    call RandomReg

    mov dword ptr Reg_Length[esi],eax ;决定代码长度使用的寄存器

Rep0:      

    push 8

    call RandomReg

    cmp eax,Reg_Length[esi]

    jz Rep0

    mov dword ptr Reg_StartAddress[esi],eax ;决定代码起始地址使用的寄存器  

    mov edi,Buf

    push esi

    call GenerateRandOpCode    ;产生 0-9 条随机的垃圾指令

    pop esi

    call GFI                   ;产生花指令

    push esi

    push Reg_Length[esi]

    push dwLength              

    call G1Code

    pop esi           ;产生第一部分代码

    call GenerateRandOpCode    ;产生 0-9 条随机的垃圾指令

    call GFI                   ;产生花指令

    push esi

    push Reg_StartAddress[esi]

    call G2Code

    pop esi

    push esi

    push 0FFh

    call Random            ;取得一个随机的 Key

    pop esi

    mov key[esi],eax

    push esi

    push EntryPoint        ;原程序入口点      

    push key[esi]

    push Reg_StartAddress[esi]

    push Reg_Length[esi]

    call G4Code

    pop esi

    mov eax,G2Selected[esi]

    push edi           ;把 Edi 保护起来

    push ecx
```

;   mov reg00,SizeOfCode   --->  mov reg00,xxx    add reg00,xxx      0 (其中 `reg00` 为随机选取的寄存器)

;                               push xxx         pop reg00          1

;                               mov reg00,xxx    xor reg00,xxx      2

;                               mov reg00,xxx    not reg00          3

;                               mov reg00,xxx    rol reg00,xx       4

;   

```
    .if VA != 0

        mov edi,VA

    .endif

    .if eax == 0         

        push 0FFFFFFFFh

        call Random

        sub edi,eax

        mov ebx,AddressOfCode1[esi]

        mov [ebx],edi

        mov ebx,AddressOfCode2[esi]

        mov [ebx],eax

    .elseif eax ==1

        mov ebx,AddressOfCode1[esi]

        mov [ebx],edi

    .elseif eax == 2

        push 0FFFFFFFFh

        call Random

        xor edi,eax

        mov ebx,AddressOfCode1[esi]

        mov [ebx],edi

        mov ebx,AddressOfCode2[esi]

        mov [ebx],eax

    .elseif eax == 3

        not edi

        mov ebx,AddressOfCode1[esi]

        mov [ebx],edi

    .else

        push 020h

        call Random

        mov cl,al

        ror edi,cl

        mov ebx,AddressOfCode1[esi]

        mov [ebx],edi

        mov ebx,AddressOfCode2[esi]

        mov byte ptr [ebx],cl

    .endif

    pop ecx

    pop edi
```

;同理  xor dword ptr [reg00+reg11],key 也可变换如下：

;              1     2    3   4   5    6   7

;          -->add   sub  inc  dec not  rol ror... 等等

```
    mov eax,EncryptSelected[esi]

    mov ecx,dwLength           ;需加密代码的长度

    .if VA == 0

        pushad

        mov esi,StartAddress

        cld

        rep movsb

        popad

    .else

        mov edi,StartAddress       ;需加密代码的起始地址

    .endif

    mov edx,key[esi]

    .if eax == 0         ;xor

Rep1:

        xor byte ptr [edi],dl

        inc edi

        loop Rep1

    .elseif eax == 1     ;add

Rep2:

        add byte ptr [edi],dl

        inc edi

        loop Rep2         

    .elseif eax == 2     ;sub

Rep3:

        sub byte ptr [edi],dl

        inc edi

        loop Rep3         

    .elseif eax == 3     ;inc 

Rep4:

        inc byte ptr [edi]

        inc edi

        loop Rep4         

    .elseif eax == 4      ;dec

Rep5:

        dec byte ptr [edi]

        inc edi

        loop Rep5         

    .elseif eax == 5      ;not

Rep6:

        mov bl,byte ptr [edi]

        not bl

        mov byte ptr [edi],bl

        inc edi

        loop Rep6         

  ;  .elseif eax == 6      ;rol

Rep7:

  ;      rol dword ptr [edi],edx

   ;     inc edi

    ;    loop Rep7         

 ;   .else                 ;ror

Rep20:

     ;   ror dword ptr [edi],edx

      ;  inc edi

       ; loop Rep20         

    .endif

    mov eax,edi

    ;popad

    ret

PolyEngine endp

RealSize equ $ - offset lbl_PolyStart

SizeOfCode equ  ((($ - offset lbl_PolyStart)*10)/1000h+1)*1000h

;end
```

