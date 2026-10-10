

# 老 Y 的变形引擎

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-34.htm`](https://www.pediy.com/kssd/pediy08/pediy8-34.htm)

看到看雪老大置顶的本月话题,把以前写的一个东西贴出来,希望能对大家有帮助  
  
可以用于病毒,也可以用于壳  
  
;.386  
;.model flat, stdcall  ;32 bit memory model  
;option casemap :none  ;case sensitive  
  
.code  
lbl\_PolyStart:   
  
  
;异常指令表  
SEHOpCode:   
   SEHInt1        db 0CDh,01h       ;int 1  
   SEHDiv0        db 33h,0C9h,0F7h,0F1h   ;xor ecx,ecx div ecx  
  
;单字节指令可变动表(2)  
  
OneByteOpCode db 40h,48h    ;inc regxx,dec regxx    
  
;单字节指令固定表(4)  
  
OneByte db 0F8h,0FCh,0F5h,0F9h  ;clc,cld,cmc,stc  
  
;双字节指令可变动表(9)  
  
TwoByteOpCode dw 08BC0h       ;mov regxx,regxx   
              dw 033C0h        ;xor regxx,regxx  
              dw 003C0h        ;add regxx,regxx  
              dw 02BC0h        ;sub regxx,regxx  
              dw 021C0h        ;and regxx,regxx  
              dw 085C0h        ;test regxx,regxx  
              dw 03BC0h        ;cmp regxx,regxx  
              dw 009C0h        ;or regxx,regxx  
              dw 011C0h       ;adc regxx  
                
                            
                
;双字节固定表(2)  
  
TwoByte dw 0D40Ah,0D50Ah       ;aam,aad  
  
  
;六字节指令变动表(8)  
  
SixByteOpCode dw 0C7C0h,081F0h,081E0h,081C8h       ;mov regxx,xxx   xor regxx,xxx  and regxx,xxx   or regxx,xxx  
              dw 0F7C0h,081F8h,081C0h,081E8h      ;test regxx,xxx   cmp regxx,xxx  add regxx,xxx   sub regxx,xxx  
                 
                                                  
;花指令表(9)  
  
RandOpCode db 70h,7Ah,72h,76h,7Eh,78h,7Ch,0EBh,0E8h   
              ;jo,jno   jp,jnp  jb,jnb  ja,jna  jng,jg  js,jns jl,jnl,jmp,call   
                                                  
;随机表(5)                 
RandomTable dd offset OneByteOpCode,offset OneByte  
            dd offset TwoByteOpCode,offset TwoByte  
            dd offset SixByteOpCode      
  
Reg\_StartAddress dd 0  
Reg\_Length dd 0            
  
OP\_MOV equ 0B8h  
OP\_XOR equ 313400h  
Randx dd 0  
  
  
  
  
pushall macro  
    push eax  
    push ecx  
    push edx  
    push ebx  
    push esi  
      
endm  
  
popall macro  
    pop esi  
    pop ebx  
    pop edx  
    pop ecx  
    pop eax  
endm  
  
  
  
  
;计算 CRC32,Esi 为首地址,Edi 为长度  
;Eax 返回 CRC32 值  
CRC32 proc uses esi edi Buf:dword,nLength:dword  
  
        push  ecx      ;procedure for calculating CRC32s  
      push  edx      ;at run-time  
      push  ebx  
      mov esi,Buf  
      mov edi,nLength         
        xor  ecx,ecx     
        dec  ecx          
        mov  edx,ecx     
NextByteCRC:             
        xor  eax,eax     
        xor  ebx,ebx     
        lodsb            
        xor  al,cl       
      mov  cl,ch  
  mov  ch,dl  
  mov  dl,dh  
  mov  dh,8  
NextBitCRC:  
  shr  bx,1  
  rcr  ax,1  
  jnc  NoCRC  
  xor  ax,08320h  
  xor  bx,0EDB8h  
NoCRC:  dec  dh  
  jnz  NextBitCRC  
  xor  ecx,eax  
  xor  edx,ebx  
    dec  edi  
  jne  NextByteCRC  
  not  edx  
  not  ecx  
  pop  ebx  
  mov  eax,edx  
  rol  eax,16  
  mov  ax,cx  
  pop  edx  
  pop  ecx  
  
  ret  
  
CRC32 endp  
  
;Eax 返回 0---dwNumber-1 之间的随机数  
  
RandSeed dd 0  
Randomize proc  
    pushad  
    call PolyStart  
PolyStart:  
    pop esi  
    sub esi,offset PolyStart       ;重定位  
    db 0fh,31h  
    add eax,edx  
    mov dword ptr RandSeed\[esi\],eax  
    popad  
    ret  
  
Randomize endp  
Random proc uses edx ecx dwNumber:dword  
    call PolyStart  
PolyStart:  
    pop esi  
    sub esi,offset PolyStart       ;重定位  
       mov eax,RandSeed\[esi\]  
         
       mov ecx,41C64E6Dh  
       mul ecx  
       add eax,3039h  
       mov RandSeed\[esi\],eax  
       xor edx,edx  
       div dwNumber  
       xchg eax,edx  
       ret   
         
Random endp  
  
RandomReg proc dwNumber:dword  
    call PolyStart  
PolyStart:  
    pop esi  
    sub esi,offset PolyStart       ;重定位  
Rep8:         
         
       push dwNumber  
       call Random  
       cmp eax,4    ;保护 Esp 寄存器  
       jz Rep8  
       cmp eax,5    ;保护 ebp 寄存器 (ebp 在程序为存取变量)  
       jz Rep8  
       cmp eax,Reg\_StartAddress\[esi\]  
       jz Rep8  
       cmp eax,Reg\_Length\[esi\]  
       jz Rep8  
         
       ret   
         
RandomReg endp  
  
  
;1 字节可变垃圾指令处理  
;Ecx 为要产生的垃圾指令的条数  
  
Sub\_OneByteOpCode proc  
      
    pushall  
    push 2  
    call Random  ;随机选取指令 inc,dec...  
    lea ebx,OneByteOpCode\[esi+eax\] ;取得 1 字节可变垃圾指令的相应地址  
    movzx edx,byte ptr \[ebx\]       ;取得 1 字节可变垃圾指令  
    push edx  
Rep9:  
    push 8  
    call RandomReg  ;随机选取寄存器 eax,ecx,edx...  
    cmp eax,Reg\_Length\[esi\]  
    jz Rep9  
    cmp eax,Reg\_StartAddress\[esi\]  
    jz Rep9      
    pop edx                     ;保护代码起始地址和代码长度已经选取的寄存器  
    add dl,al  
    mov byte ptr \[edi\],dl  
    inc edi  
    popall  
    ret  
  
Sub\_OneByteOpCode endp  
  
;1 字节固定垃圾指令处理  
  
Sub\_OneByte proc  
    pushall  
    push 4  
    call Random  ;随机选取指令 aaa,aas,clc...  
    lea ebx,OneByte\[esi+eax\] ;取得 1 字节固定垃圾指令的相应地址  
    movzx edx,byte ptr \[ebx\]       ;取得 1 字节固定垃圾指令  
    mov byte ptr \[edi\],dl  
    inc edi  
    popall  
    ret  
  
Sub\_OneByte endp  
  
  
;2 字节可变垃圾指令处理  
  
Sub\_TwoByteOpCode proc  
    pushall  
    push 9  
    call Random  
    lea ebx,TwoByteOpCode\[esi+eax\*2\]  
    movzx edx,word ptr \[ebx\]  
    push edx                         ;uses push  
Rep10:  
    push 8  
    call RandomReg  
    cmp eax,Reg\_Length\[esi\]  
    jz Rep10  
    cmp eax,Reg\_StartAddress\[esi\]  
    jz Rep10                         ;保护代码起始地址和代码长度已经选取的寄存器  
      
    mov ebx,eax  
      
Rep11:  
    push 8  
    call RandomReg  
    cmp eax,Reg\_Length\[esi\]  
    jz Rep11  
    cmp eax,Reg\_StartAddress\[esi\]  
    jz Rep11                        ;保护代码起始地址和代码长度已经选取的寄存器  
    mov cl,8                        ;uses ecx  
    mul cl  
    pop edx  
    add dl,al  
    add dl,bl  
    mov byte ptr \[edi\],dh  
    inc edi  
    mov byte ptr \[edi\],dl  
    inc edi  
    popall  
    ret  
  
Sub\_TwoByteOpCode endp  
  
;2 字节固定垃圾指令处理  
  
Sub\_TwoByte proc  
    pushall  
    push 2  
    call Random  
    lea ebx,TwoByte\[esi+eax\*2\]  
    mov dx,word ptr \[ebx\]  
    mov byte ptr\[edi\],dh  
    inc edi  
    mov byte ptr \[edi\],dl  
    inc edi  
    popall  
    ret  
  
Sub\_TwoByte endp  
  
;6 字节垃圾指令处理  
  
Sub\_SixByteOpCode proc  
    pushall  
    push 8  
    call Random  
    lea ebx,SixByteOpCode\[esi+eax\*2\]  
    mov dx,word ptr \[ebx\]  
    push edx  
  
Rep12:  
    push 8  
    call RandomReg  
    cmp eax,Reg\_Length\[esi\]  
    jz Rep12  
    cmp eax,Reg\_StartAddress\[esi\]  
    jz Rep12                         ;保护代码起始地址和代码长度已经选取的寄存器  
      
    pop edx  
    add dl,al  
    mov byte ptr \[edi\],dh  
      
    inc edi  
    mov byte ptr \[edi\],dl  
    inc edi  
    push 600000h  
    call Random  
    mov dword ptr \[edi\],eax  
    add edi,4  
    popall  
      
    ret  
  
Sub\_SixByteOpCode endp  
  
  
  
  
;产生异常块  
GenSEHBlock proc  
  
  
  
  
    ret  
      
GenSEHBlock endp   
  
  
;产生随机的垃圾指令  
  
GenerateRandOpCode proc  
    pushall  
    push 10  
    call Random     ;在每条有效指令之间随机的产生 0-9 条垃圾指令  
    inc eax  
    mov ecx,eax     ;uses ecx  
      
Rep13:  
    push ecx  
    push 5  
    call Random    ;总共有 5 种类型的垃圾指令，随机取一种  
    mov ebx,eax  
  
    mov edx,RandomTable\[esi+ebx\*4\]  
    lea edx,\[edx+esi\]  ;得到垃圾指令表的地址  
      
    cmp ebx,0  
    jz lbl\_OneByteOpCode  ;转 1 字节可变垃圾指令处理  
      
    cmp ebx,1  
    jz lbl\_OneByte        ;转 1 字节固定垃圾指令处理  
      
    cmp ebx,2  
    jz lbl\_TwoByteOpCode  ;转 2 字节可变垃圾指令处理  
      
    cmp ebx,3  
    jz lbl\_TwoByte         ;转 2 字节固定垃圾指令处理  
      
    jmp lbl\_SixByteOpCode  ;转 6 字节垃圾指令处理  
      
       
lbl\_OneByteOpCode:  
    call Sub\_OneByteOpCode  
    jmp lbl\_Next1  
      
lbl\_OneByte:  
    call Sub\_OneByte  
    jmp lbl\_Next1  
  
lbl\_TwoByteOpCode:  
    call Sub\_TwoByteOpCode  
    jmp lbl\_Next1  
  
lbl\_TwoByte:  
    call Sub\_TwoByte  
    jmp lbl\_Next1  
  
lbl\_SixByteOpCode:  
  
    call Sub\_SixByteOpCode  
  
lbl\_Next1:  
    pop ecx  
    dec ecx  
    jnz Rep13    ;继续产生垃圾指令  
   popall  
    ret  
  
GenerateRandOpCode endp  
  
  
;填充 dwLength2 长的随机数据到 edi->的缓冲区  
FillRandomCode proc dwLength2:dword  
    pushall  
    mov ecx,dwLength2        ;uses ecx  
      
Rep14:  
    push 0FFh  
    call Random  
    mov byte ptr \[edi\],al  
    inc edi  
    loop Rep14  
    popall  
    ret  
  
FillRandomCode endp  
  
  
;产生随机的花指令  
  
GFI proc  
    pushall  
      
    push 9  
    call Random  
    cmp eax,7  
    jz lbl\_Jmp  
    cmp eax,8  
    jz lbl\_Call  
    lea ebx,RandOpCode\[esi+eax\]  
    movzx edx,byte ptr \[ebx\]  
    mov byte ptr \[edi\],dl      
    inc edi  
    push 5  
    call Random  
    add eax,3        ;3-7 之间  
    mov byte ptr \[edi\],al  
    inc edi  
    inc dl  
    mov byte ptr \[edi\],dl  
    inc edi  
    dec eax  
    dec eax  
    mov byte ptr \[edi\],al  
    inc edi  
    push eax  
    call FillRandomCode  
    jmp lbl\_Exit1  
      
      
lbl\_Jmp:  
    lea ebx,RandOpCode\[esi+eax\]  
    movzx edx,byte ptr \[ebx\]  
    mov byte ptr \[edi\],dl    ;E9h->\[edi\]  
    inc edi  
    push 4  
    call Random  
    inc eax  
    inc eax  
    mov byte ptr \[edi\],al     
    inc edi  
    mov byte ptr \[edi\],0E8h  
    inc edi  
    dec eax        
    push eax  
    call FillRandomCode  
    jmp lbl\_Exit1  
  
lbl\_Call:  
    lea ebx,RandOpCode\[esi+eax\]  
    movzx edx,byte ptr \[ebx\]  
    mov byte ptr \[edi\],dl    ;E8h->\[edi\]  
    inc edi  
    push 4  
    call Random  
    inc eax  
    inc eax  
    mov dword ptr \[edi\],eax    
    add  edi,4  
    mov byte ptr \[edi\],0E9h  
    dec eax  
    push eax  
    call FillRandomCode  
    inc edi  
    push 8  
    call RandomReg  
    mov edx,58h          ;pop regxx  
    add edx,eax  
    mov byte ptr \[edi\],dl  
    inc edi  
      
lbl\_Exit1:  
    popall  
    ret  
  
GFI endp  
  
  
  
;随机产生第一部分代码  
G1Code proc dwLength1:dword,reg:dword  
    pushall  
    mov ebx,dwLength1  
    push 5  
    call Random  
    cmp eax,1  
    jz lbl\_PushPopG1  
    cmp eax,2  
    jz lbl\_MovXorG1  
    cmp eax,3  
    jz lbl\_MovNotG1  
    cmp eax,4  
    jz lbl\_MovRolG1  
     
lbl\_MovAddG1:  
    push 0FFFFFFFFh  
    call Random  
    sub ebx,eax  
      
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],ebx   ;ebx 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0C081h    ;add regxx,xxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],eax  ;eax 为 add regxx,xxx 里的 xxx  
    add edi,4  
          
    jmp lbl\_Exit2  
      
lbl\_PushPopG1:  
  
    mov byte ptr \[edi\],68h ;push xxx  
    inc edi  
    mov dword ptr \[edi\],ebx ;ebx 为 push xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,58h               ;pop regxx  
    add edx,reg  
    mov byte ptr \[edi\],dl       
    inc edi  
         
      
    jmp lbl\_Exit2  
      
lbl\_MovXorG1:  
  
    push 0FFFFFFFFh  
    call Random  
    xor ebx,eax  
      
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],ebx   ;ebx 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0F081h    ;xor regxx,xxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],eax  ;eax 为 add regxx,xxx 里的 xxx  
    add edi,4  
  
      
    jmp lbl\_Exit2  
  
lbl\_MovNotG1:  
  
    not ebx  
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],ebx   ;ebx 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0D0F7h    ;not regxx  
    mov ecx,reg  
    add dh,cl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
  
    jmp lbl\_Exit2  
  
lbl\_MovRolG1:  
      
    push 020h  
    call Random  
    mov cl,al  
    push eax  
    ror ebx,cl  
      
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov eax,reg  
    add dh,al  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov dword ptr \[edi\],ebx  ;ebx 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0C0C1h    ;rol regxx,xxx  
    mov eax,reg  
    add dh,al  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    pop eax  
    mov byte ptr \[edi\],al  ;eax 为 add regxx,xxx 里的 xxx  
    inc edi  
        
      
lbl\_Exit2:  
    popall  
    ret  
  
G1Code endp  
  
  
;随机产生第二部分代码  
G2Code proc reg:dword  
    pushall  
    push 5  
    call Random  
    cmp eax,1  
    jz lbl\_PushPopG2  
    cmp eax,2  
    jz lbl\_MovXorG2  
    cmp eax,3  
    jz lbl\_MovNotG2  
    cmp eax,4  
    jz lbl\_MovRolG2  
     
lbl\_MovAddG2:  
    mov G2Selected\[esi\],0         ;保存第二部分的产生方式  
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode1\[esi\],edi ;保存 Mov regxx,xxx 指令的地址  
    mov dword ptr \[edi\],0   ;0 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0C081h    ;add regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode2\[esi\],edi ;保存 add regxx,xxx 指令的地址  
  
    mov dword ptr \[edi\],0  ;eax 为 add regxx,xxx 里的 xxx  
    add edi,4  
          
    jmp lbl\_Exit3  
      
lbl\_PushPopG2:  
    mov G2Selected\[esi\],1         ;保存第二部分的产生方式  
  
  
    mov byte ptr \[edi\],68h ;push xxx  
    inc edi  
    mov AddressOfCode1\[esi\],edi ;保存 Push xxx 指令的地址  
      
      
    mov dword ptr \[edi\],0 ;为 push xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,58h               ;pop regxx  
    add edx,reg  
    mov byte ptr \[edi\],dl       
    inc edi  
         
      
    jmp lbl\_Exit3  
      
lbl\_MovXorG2:  
    mov G2Selected\[esi\],2        ;保存第二部分的产生方式  
  
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode1\[esi\],edi ;保存 Mov regxx,xxx 指令的地址  
      
    mov dword ptr \[edi\],0   ;0 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0F081h    ;xor regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode2\[esi\],edi ;保存 xor regxx,xxx 指令的地址  
  
    mov dword ptr \[edi\],0  ;eax 为 xor regxx,xxx 里的 xxx  
    add edi,4  
  
      
    jmp lbl\_Exit3  
  
lbl\_MovNotG2:  
    mov G2Selected\[esi\],3        ;保存第二部分的产生方式  
  
    not ebx  
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode1\[esi\],edi  
    mov dword ptr \[edi\],0   ;0 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0D0F7h    ;not regxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
  
    jmp lbl\_Exit3  
  
lbl\_MovRolG2:  
    mov G2Selected\[esi\],4        ;保存第二部分的产生方式  
  
      
    mov edx,0C0C7h   ;mov regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode1\[esi\],edi ;保存 Mov regxx,xxx 指令的地址  
      
    mov dword ptr \[edi\],0   ;0 为 mov regxx,xxx 里的 xxx  
    add edi,4  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0C0C1h    ;rol regxx,xxx  
    mov ebx,reg  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov AddressOfCode2\[esi\],edi ;保存 Rol regxx,xxx 指令的地址  
  
    mov byte ptr \[edi\],0  ;eax 为 Rol regxx,cl 里的 regxx  
    inc edi  
        
      
lbl\_Exit3:  
    popall  
    ret  
  
G2Code endp  
  
  
;随机产生第三部分代码  
G3Code proc  regOfAddr:dword,key:dword  
    pushall  
    push 8                    ;                       1    2    3    4   5    6   7  
    call Random               ;决定采用的加密方式 -->add   sub  inc  dec not  rol ror... 等等  
    cmp eax,1  
    jz lbl\_AddG3  
    cmp eax,2  
    jz lbl\_SubG3  
    cmp eax,3  
    jz lbl\_IncG3  
    cmp eax,4  
    jz lbl\_DecG3  
    cmp eax,5  
    jz lbl\_NotG3  
  ;  cmp eax,6  
   ; jz lbl\_Rol  
   ; cmp eax,7  
   ; jz lbl\_Ror  
      
lbl\_XorG3:  
    mov EncryptSelected\[esi\],0  ;保存加密方式  
    mov edx,3080h          ;Xor byte ptr \[regxx\],xxx  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov edx,key  
    mov byte ptr \[edi\],dl  
    inc edi  
      
    jmp lbl\_Exit4  
      
lbl\_AddG3:  
  
    mov EncryptSelected\[esi\],1   ;保存加密方式  
    mov edx,2880h                ;Sub byte ptr \[regxx\],xxx  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov edx,key  
    mov byte ptr \[edi\],dl  
    inc edi  
  
  
    jmp lbl\_Exit4  
      
lbl\_SubG3:  
    mov EncryptSelected\[esi\],2  ;保存加密方式  
  
    mov edx,0080h                ;Add byte ptr \[regxx\],xxx  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov edx,key  
    mov byte ptr \[edi\],dl  
    inc edi  
  
    jmp lbl\_Exit4  
  
lbl\_IncG3:  
  
    mov EncryptSelected\[esi\],3   ;保存加密方式  
    mov edx,008FEh                ;Dec byte ptr \[regxx\]  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
      
  
    jmp lbl\_Exit4  
  
lbl\_DecG3:  
  
    mov EncryptSelected\[esi\],4  ;保存加密方式  
    mov edx,000FEh                ;Inc byte ptr \[regxx\]  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
      
    jmp lbl\_Exit4  
  
lbl\_NotG3:  
  
    mov EncryptSelected\[esi\],5   ;保存加密方式  
    mov edx,010F6h                ;Not byte  ptr \[regxx\]  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
      
    jmp lbl\_Exit4  
  
lbl\_RolG3:  
  
    mov EncryptSelected\[esi\],6  ;保存加密方式  
    mov eax,key  
    xor edx,edx  
    mov ecx,20h  
    div ecx  
    mov cl,dl  
    mov edx,51h             ;push ecx  
    mov byte ptr \[edi\],dl  
    inc edi  
    push ecx  
    call GFI  
    pop ecx  
    mov edx,0C1C6h         ;mov cl,xxx  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov byte ptr \[edi\],cl  
    inc edi  
    call GFI  
    mov edx,008D3h         ;ror dword ptr \[edi\],cl  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
  
    jmp lbl\_Exit4  
  
lbl\_RorG3:  
  
    mov EncryptSelected\[esi\],7   ;保存加密方式  
  
    mov eax,key  
    xor edx,edx  
    mov ecx,20h  
    div ecx  
    mov cl,dl  
    mov edx,51h             ;push ecx  
    mov byte ptr \[edi\],dl  
    inc edi  
    push ecx  
    call GFI  
    pop ecx  
    mov edx,0C1C6h         ;mov cl,xxx  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    mov byte ptr \[edi\],cl  
    inc edi  
    call GFI  
    mov edx,000D3h                ;rol dword ptr\[regxx\],cl  
    mov ebx,regOfAddr  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
  
  
      
lbl\_Exit4:  
      
    popall  
    ret  
  
G3Code endp  
  
pGetModuleCode dd 0  
;随机产生第四部分代码  
G4Code proc regOfLen:dword,regOfAddr:dword,key:dword,EntryPoint:dword  
    pushall  
      
rep100:  
    push 2  
    call Random  
    cmp eax,0  
    jz lbl\_DecJge  
    cmp regOfAddr,1  
    jz rep100  
                                ;push reg00      保护首地址寄存器  
                                ;mov ecx,reg11   长度寄存器  
                                  
                            ;Rep:  
                                ;Xor dword ptr \[reg00\],key   
                                ;inc reg00  
                                ;Loop Rep   
                                ;pop reg00   
lbl\_Loop:  
      
    mov edx,50h  ;push regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
    mov edx,0C88Bh          ;mov ecx,regxx      
    mov ebx,regOfLen  
    add dh,bl  
    mov word ptr \[edi\],dx  
    inc edi  
    inc edi  
    call GFI  
    push edi  
    pop edx  
    push edx                 ;保存 Loop xxx 的循环地址  
    call GFI  
    mov byte ptr \[edi\],51h    ;push ecx  
    inc edi  
    call GenerateRandOpCode   ;产生垃圾指令  
      
    call GFI                   ;产生花指令  
     
    push key                
    push regOfAddr  
    call G3Code  
      
    call GFI                    ;产生花指令  
    mov edx,40h                 ;inc regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
      
    call GenerateRandOpCode  
    mov byte ptr \[edi\],59h     ;pop ecx  
    inc edi  
    call GFI                    ;产生花指令  
    mov edx,0E2h               ;Loop xxx  
    mov byte ptr \[edi\],dl  
    ;inc edi  
    inc edi  
    mov byte ptr \[edi\],0  
    inc edi  
    pop edx  
    sub edx,edi  
    mov byte ptr \[edi-1\],dl  
    mov edx,58h                 ;pop regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
    call GFI  
;  invoke VirtualAlloc,NULL,1000h,MEM\_COMMIT,PAGE\_EXECUTE\_READWRITE  
;  or eax,eax  
;  jz lbl\_Exit5  
;  mov pGetModuleCode,eax  
    call GenerateRandOpCode  
    call GFI  
     
;    push En\_ImageBase  
;    push NewRva  
;    push edi  
;    call GenerateGetModule     
      
;    mov byte ptr \[edi\],68h          ;Push EntryPoint  
;    inc edi  
;    mov edx,EntryPoint  
;    mov dword ptr \[edi\],edx  
;    add edi,4  
;    call GenerateRandOpCode  
;    call GFI  
;    mov byte ptr \[edi\],0C3h         ;Ret  
;    inc edi  
    jmp lbl\_Exit5  
      
  
lbl\_DecJge:  
  
    mov edx,50h  ;push regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
    call GenerateRandOpCode   ;产生垃圾指令  
    call GFI                  ;产生花指令  
      
    call GFI   ;产生花指令  
      
    push edi  
    pop edx  
    push edx                 ;保存 Jge xxx 的循环地址  
      
    call GFI                   ;产生花指令  
     
    push key                
    push regOfAddr  
    call G3Code                ;产生解密代码,如 Xor,sub,not...等等  
      
    call GFI                    ;产生花指令  
    mov edx,40h                 ;inc regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
      
    call GFI                    ;产生花指令  
      
    mov edx,48h                 ;Dec regxx  
    add edx,regOfLen  
    mov byte ptr \[edi\],dl  
    inc edi  
    call GFI  
      
    mov edx,07fh               ;Jge xxx  
    mov byte ptr \[edi\],dl  
    inc edi  
    mov byte ptr \[edi\],0  
    inc edi  
    pop edx                  ;恢复 Jge xxx 的 xxx  
    sub edx,edi  
    mov byte ptr \[edi-1\],dl  
    call GenerateRandOpCode  
    call GFI  
     
 ;   push En\_ImageBase  
 ;   push NewRva  
 ;   push edi  
 ;   call GenerateGetModule     
     
     
     
    call GenerateRandOpCode  
    call GFI  
    mov edx,58h                 ;pop regxx  
    add edx,regOfAddr  
    mov byte ptr \[edi\],dl  
    inc edi  
    call GenerateRandOpCode  
    call GFI  
   ; mov byte ptr \[edi\],0FFh  
   ; inc edi  
   ; mov edx,0E0h  
   ; add edx,regOfAddr  
   ; mov byte ptr \[edi\],dl        ;Jmp regxx  
   ; inc edi  
   ; call GenerateRandOpCode  
   ; call GFI  
      
;    mov byte ptr \[edi\],68h          ;Push EntryPoint  
;    inc edi  
;    mov edx,EntryPoint  
;    mov dword ptr \[edi\],edx  
;    add edi,4  
;    call GenerateRandOpCode  
;    call GFI  
;    mov byte ptr \[edi\],0C3h         ;Ret  
;    inc edi  
  
  
      
  
lbl\_Exit5:  
    popall  
    ret  
  
G4Code endp  
  
  
  
;整个程序的核心(变形引擎)  
;   假设解密代码如下：(可分为 4 部分)  
;   mov reg00,SizeOfCode            1  
;   mov reg11,EntryPointOfCode      2  
;Rep:  
;   xor dowrd ptr\[reg00+reg11\],key  3  
;   dec reg00                       4  
;   jnz Rep  
;为了增强变形引擎的强度可以把上面的代码分解(或叫变换)  
;如下：  
;    第一部分代码可以有下面 5 种选择  
;   mov reg00,SizeOfCode   --->  mov reg00,xxx    add reg00,xxx      0 (其中 reg00 为随机选取的寄存器)  
;                                push xxx         pop reg00          1  
;                                mov reg00,xxx    xor reg00,xxx      2  
;                                mov reg00,xxx    not reg00          3  
;                                mov reg00,xxx    rol reg00,xx       4  
;    
  
;   第二部分同样有 5 种选择  
;   mov reg11,EntryPointOfCode 也可作同样处理  
  
  
;   第三部分有 8 种选择  
;       0  
;同理  xor dword ptr \[reg00+reg11\],key 也可变换如下：  
  
;               1     2    3   4   5    6   7  
;          -->add   sub  inc  dec not  rol ror... 等等  
;  
  
;   第四部分有 2 种选择  
;          0                 1  
;      dec reg00   --->   Loop xxx  
;      jnz xxx  
  
  
       
;param Buf   存放变形以后代码的缓冲区  
;param StartAddress 欲加密代码的首地址  
;param dwLength     欲加密代码的长度  
;  
  
EncryptSelected dd 0    ;保存被选择的加密方式  
G2Selected dd 0         ;保存第二部分产生的方式  
AddressOfCode1 dd 0     ;相应的地址  
AddressOfCode2 dd 0  
key dd 0  
  
PolyEngine proc Buf:dword,StartAddress:dword,dwLength:dword,VA:dword,EntryPoint:dword  
    call PolyStart  
PolyStart:  
    pop esi  
    sub esi,offset PolyStart       ;重定位  
    call Randomize                 ;初始化随机种子  
    push 8  
    call RandomReg  
    mov dword ptr Reg\_Length\[esi\],eax ;决定代码长度使用的寄存器  
Rep0:      
    push 8  
    call RandomReg  
    cmp eax,Reg\_Length\[esi\]  
    jz Rep0  
    mov dword ptr Reg\_StartAddress\[esi\],eax ;决定代码起始地址使用的寄存器   
    mov edi,Buf  
    push esi  
    call GenerateRandOpCode    ;产生 0-9 条随机的垃圾指令  
    pop esi  
    call GFI                   ;产生花指令  
    push esi  
    push Reg\_Length\[esi\]  
    push dwLength                
      
    call G1Code  
    pop esi               ;产生第一部分代码  
    call GenerateRandOpCode    ;产生 0-9 条随机的垃圾指令  
    call GFI                   ;产生花指令  
    push esi  
    push Reg\_StartAddress\[esi\]  
    call G2Code  
    pop esi  
    push esi  
    push 0FFh  
    call Random                ;取得一个随机的 Key  
    pop esi  
    mov key\[esi\],eax  
    push esi  
    push EntryPoint            ;原程序入口点      
    push key\[esi\]  
    push Reg\_StartAddress\[esi\]  
    push Reg\_Length\[esi\]  
  
    call G4Code  
    pop esi  
    mov eax,G2Selected\[esi\]  
    push edi               ;把 Edi 保护起来  
    push ecx  
;   mov reg00,SizeOfCode   --->  mov reg00,xxx    add reg00,xxx      0 (其中 reg00 为随机选取的寄存器)  
;                                push xxx         pop reg00          1  
;                                mov reg00,xxx    xor reg00,xxx      2  
;                                mov reg00,xxx    not reg00          3  
;                                mov reg00,xxx    rol reg00,xx       4  
;    
    .if VA != 0  
        mov edi,VA  
    .endif  
      
    .if eax == 0           
        push 0FFFFFFFFh  
        call Random  
        sub edi,eax  
        mov ebx,AddressOfCode1\[esi\]  
        mov \[ebx\],edi  
        mov ebx,AddressOfCode2\[esi\]  
        mov \[ebx\],eax  
          
    .elseif eax ==1  
        mov ebx,AddressOfCode1\[esi\]  
        mov \[ebx\],edi  
    .elseif eax == 2  
        push 0FFFFFFFFh  
        call Random  
        xor edi,eax  
        mov ebx,AddressOfCode1\[esi\]  
        mov \[ebx\],edi  
        mov ebx,AddressOfCode2\[esi\]  
        mov \[ebx\],eax  
                
                 
    .elseif eax == 3  
        not edi  
        mov ebx,AddressOfCode1\[esi\]  
        mov \[ebx\],edi  
          
    .else  
        push 020h  
        call Random  
        mov cl,al  
        ror edi,cl  
        mov ebx,AddressOfCode1\[esi\]  
        mov \[ebx\],edi  
        mov ebx,AddressOfCode2\[esi\]  
        mov byte ptr \[ebx\],cl  
    .endif  
    pop ecx  
    pop edi  
      
      
;同理  xor dword ptr \[reg00+reg11\],key 也可变换如下：  
  
;               1     2    3   4   5    6   7  
;          -->add   sub  inc  dec not  rol ror... 等等  
  
    mov eax,EncryptSelected\[esi\]  
                                
    mov ecx,dwLength           ;需加密代码的长度  
    .if VA == 0  
        pushad  
        mov esi,StartAddress  
        cld  
        rep movsb  
          
        popad  
    .else  
        mov edi,StartAddress       ;需加密代码的起始地址  
          
    .endif  
    mov edx,key\[esi\]  
      
    .if eax == 0         ;xor  
Rep1:  
        xor byte ptr \[edi\],dl  
        inc edi  
        loop Rep1  
    .elseif eax == 1     ;add  
Rep2:  
        add byte ptr \[edi\],dl  
        inc edi  
        loop Rep2          
         
    .elseif eax == 2     ;sub  
Rep3:  
        sub byte ptr \[edi\],dl  
        inc edi  
        loop Rep3          
         
      
    .elseif eax == 3     ;inc   
Rep4:  
        inc byte ptr \[edi\]  
        inc edi  
        loop Rep4          
         
      
    .elseif eax == 4      ;dec  
Rep5:  
        dec byte ptr \[edi\]  
        inc edi  
        loop Rep5          
         
      
    .elseif eax == 5      ;not  
Rep6:  
        mov bl,byte ptr \[edi\]  
        not bl  
        mov byte ptr \[edi\],bl  
        inc edi  
        loop Rep6          
         
      
  ;  .elseif eax == 6      ;rol  
Rep7:  
  ;      rol dword ptr \[edi\],edx  
   ;     inc edi  
    ;    loop Rep7          
         
  
 ;   .else                 ;ror  
      
Rep20:  
     ;   ror dword ptr \[edi\],edx  
      ;  inc edi  
       ; loop Rep20          
      
    .endif  
  
    mov eax,edi  
    ;popad  
      
    ret  
  
PolyEngine endp  
RealSize equ $ - offset lbl\_PolyStart  
  
SizeOfCode equ  ((($ - offset lbl\_PolyStart)\*10)/1000h+1)\*1000h  
;end 

 4dd466d0e1e0cc123afbe504da2462ca