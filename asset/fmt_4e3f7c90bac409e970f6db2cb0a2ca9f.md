
### Ring0 by IDT

`"Ring0 activated", 0`

`.CODE`

```asm
Start:   
    push edx   
    sidt [esp-2] ;read IDT to stack   
    pop edx ;address of Interrupt table   
    add edx,(Interrupt*8)+4 ;Interrupt table base+Int number+size for ;Int in Interrupt table=Int vector address   
    mov ebx,[edx]   
    mov bx,word ptr [edx-4] ;read old address our interrupt (INT 5h)   
    lea edi,InterruptHandler   
    mov [edx-4],di ;ror edi,16 ;set our new interrupt handler   
    mov [edx+2],di   
    push ds ;save registers   
    push es   
    int Interrupt ;jump to Ring0 (our int 5h handler)   
    pop es ;restore registers   
    pop ds   
    mov [edx-4],bx ;set old int 5h handler ror ebx,16   
    mov [edx+2],bx   
    call MessageBoxA,0, offset msg2, offset msg1,0   
    call ExitProcess, -1   

;-----------------------------------------------------------------------------   
;OUR NEW INT 5h HANDLER (it run in Ring0)   
;-----------------------------------------------------------------------------   

InterruptHandler:   
    mov eax,dr7 ;test for Ring0   
    iretd ;jump back to Ring3   

ends   

end Start
```

`\--------------------------------------------------------------------------------`

还有很多方法可以实现 `Ring0`，我也不知道了。大家请自己学习吧。 `4dd466d0e1e0cc123afbe504da2462ca`
