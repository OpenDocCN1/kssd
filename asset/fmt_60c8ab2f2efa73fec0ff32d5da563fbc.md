
```asm
mov edx,0C0C1h ;rol regxx,xxx

mov ebx,reg

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

mov AddressOfCode2[esi],edi ;保存 Rol regxx,xxx 指令的地址

mov byte ptr [edi],0 ;eax 为 Rol regxx,cl 里的 regxx

inc edi

lbl_Exit3:

popall

ret

G2Code endp

;随机产生第三部分代码

G3Code proc regOfAddr:dword,key:dword

pushall

push 8 ; 1 2 3 4 5 6 7

call Random ;决定采用的加密方式 -->add sub inc dec not rol ror... 等等

cmp eax,1

jz lbl_AddG3

cmp eax,2

jz lbl_SubG3

cmp eax,3

jz lbl_IncG3

cmp eax,4

jz lbl_DecG3

cmp eax,5

jz lbl_NotG3

; cmp eax,6

; jz lbl_Rol

; cmp eax,7

; jz lbl_Ror

lbl_XorG3:

mov EncryptSelected[esi],0 ;保存加密方式

mov edx,3080h ;Xor byte ptr [regxx],xxx

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

mov edx,key

mov byte ptr [edi],dl

inc edi

jmp lbl_Exit4

lbl_AddG3:

mov EncryptSelected[esi],1 ;保存加密方式

mov edx,2880h ;Sub byte ptr [regxx],xxx

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

mov edx,key

mov byte ptr [edi],dl

inc edi

jmp lbl_Exit4

lbl_SubG3:

mov EncryptSelected[esi],2 ;保存加密方式

mov edx,0080h ;Add byte ptr [regxx],xxx

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

mov edx,key

mov byte ptr [edi],dl

inc edi

jmp lbl_Exit4

lbl_IncG3:

mov EncryptSelected[esi],3 ;保存加密方式

mov edx,008FEh ;Dec byte ptr [regxx]

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

jmp lbl_Exit4

lbl_DecG3:

mov EncryptSelected[esi],4 ;保存加密方式

mov edx,000FEh ;Inc byte ptr [regxx]

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

jmp lbl_Exit4

lbl_NotG3:

mov EncryptSelected[esi],5 ;保存加密方式

mov edx,010F6h ;Not byte ptr [regxx]

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

jmp lbl_Exit4

lbl_RolG3:

mov EncryptSelected[esi],6 ;保存加密方式

mov eax,key

xor edx,edx

mov ecx,20h

div ecx

mov cl,dl

mov edx,51h ;push ecx

mov byte ptr [edi],dl

inc edi

push ecx

call GFI

pop ecx

mov edx,0C1C6h ;mov cl,xxx

mov word ptr [edi],dx

inc edi

inc edi

mov byte ptr [edi],cl

inc edi

call GFI

mov edx,008D3h ;ror dword ptr [edi],cl

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

jmp lbl_Exit4

lbl_RorG3:

mov EncryptSelected[esi],7 ;保存加密方式

mov eax,key

xor edx,edx

mov ecx,20h

div ecx

mov cl,dl

mov edx,51h ;push ecx

mov byte ptr [edi],dl

inc edi

push ecx

call GFI

pop ecx

mov edx,0C1C6h ;mov cl,xxx

mov word ptr [edi],dx

inc edi

inc edi

mov byte ptr [edi],cl

inc edi

call GFI

mov edx,000D3h ;rol dword ptr [regxx],cl

mov ebx,regOfAddr

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

lbl_Exit4:

popall

ret

G3Code endp

pGetModuleCode dd 0

;随机产生第四部分代码

G4Code proc regOfLen:dword,regOfAddr:dword,key:dword,EntryPoint:dword

pushall

rep100:

push 2

call Random

cmp eax,0

jz lbl_DecJge

cmp regOfAddr,1

jz rep100

; push reg00 ; 保护首地址寄存器

; mov ecx,reg11 ; 长度寄存器

; Rep:

; Xor dword ptr [reg00],key

; inc reg00

; Loop Rep

; pop reg00

lbl_Loop:

mov edx,50h ;push regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GenerateRandOpCode ;产生垃圾指令

call GFI ;产生花指令

mov edx,0C88Bh ;mov ecx,regxx

mov ebx,regOfLen

add dh,bl

mov word ptr [edi],dx

inc edi

inc edi

call GFI

push edi

pop edx

push edx ;保存 Loop xxx 的循环地址

call GFI

mov byte ptr [edi],51h ;push ecx

inc edi

call GenerateRandOpCode ;产生垃圾指令

call GFI ;产生花指令

push key

push regOfAddr

call G3Code

call GFI ;产生花指令

mov edx,40h ;inc regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GenerateRandOpCode

mov byte ptr [edi],59h ;pop ecx

inc edi

call GFI ;产生花指令

mov edx,0E2h ;Loop xxx

mov byte ptr [edi],dl

; inc edi

inc edi

mov byte ptr [edi],0

inc edi

pop edx

sub edx,edi

mov byte ptr [edi-1],dl

mov edx,58h ;pop regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GFI

; invoke VirtualAlloc,NULL,1000h,MEM_COMMIT,PAGE_EXECUTE_READWRITE

; or eax,eax

; jz lbl_Exit5

; mov pGetModuleCode,eax

call GenerateRandOpCode

call GFI

; push En_ImageBase

; push NewRva

; push edi

; call GenerateGetModule

; mov byte ptr [edi],68h ;Push EntryPoint

; inc edi

; mov edx,EntryPoint

; mov dword ptr [edi],edx

; add edi,4

; call GenerateRandOpCode

; call GFI

; mov byte ptr [edi],0C3h ;Ret

; inc edi

jmp lbl_Exit5

lbl_DecJge:

mov edx,50h ;push regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GenerateRandOpCode ;产生垃圾指令

call GFI ;产生花指令

call GFI ;产生花指令

push edi

pop edx

push edx ;保存 Jge xxx 的循环地址

call GFI ;产生花指令

push key

push regOfAddr

call G3Code ;产生解密代码,如 Xor,sub,not...等等

call GFI ;产生花指令

mov edx,40h ;inc regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GFI ;产生花指令

mov edx,48h ;Dec regxx

add edx,regOfLen

mov byte ptr [edi],dl

inc edi

call GFI

mov edx,07fh ;Jge xxx

mov byte ptr [edi],dl

inc edi

mov byte ptr [edi],0

inc edi

pop edx ;恢复 Jge xxx 的 xxx

sub edx,edi

mov byte ptr [edi-1],dl

call GenerateRandOpCode

call GFI

; push En_ImageBase

; push NewRva

; push edi

; call GenerateGetModule

call GenerateRandOpCode

call GFI

mov edx,58h ;pop regxx

add edx,regOfAddr

mov byte ptr [edi],dl

inc edi

call GenerateRandOpCode

call GFI

; mov byte ptr [edi],0FFh

; inc edi

; mov edx,0E0h

; add edx,regOfAddr

; mov byte ptr [edi],dl ;Jmp regxx

; inc edi

; call GenerateRandOpCode

; call GFI

; mov byte ptr [edi],68h ;Push EntryPoint

; inc edi
```
