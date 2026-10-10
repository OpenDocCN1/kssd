

# DRx 解码阻止调试 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6653.htm`](https://www.pediy.com/kssd/pediy06/pediy6653.htm)

这些代码是从我逆向出 hying 外壳原码得来,有些描述不准确,凑合看吧  
  
;//////////////////////////////////  
;//drx 解码  
  
        Call    Push\_drx\_handler  
  
Drx\_handler    Proc Near  
  
Pframe        = Dword    Ptr  4  
Pcontext        = Dword    Ptr  0ch  
  
        Mov    Eax, \[esp+pframe\]  
        Mov    Eax, \[eax\]                ; Eax =  Pframe->exceptioncode  
        Mov    Ecx, \[esp+pcontext\]    ; Ecx -> Context  
  
        Inc    Dword Ptr \[ecx+0b8h\] ; Context.regeip++  
  
        Cmp    Eax, Exception\_breakpoint  
        Jnz    Drx\_not\_breakpoint  
  
        Mov    Eax, \[ecx+0b4h\]    ; Eax <- Context.regebp  
        Lea    Eax, \[drx\_0-@symbiont\_start\]\[eax\]  
        Mov    \[ecx+4\], Eax    ; Dr0  
  
        Mov    Eax, \[ecx+0b4h\]    ; Eax <- Context.regebp  
        Lea    Eax, \[drx\_1-@symbiont\_start\]\[eax\]  
        Mov    \[ecx+8\], Eax    ; Dr1  
  
        Mov    Eax, \[ecx+0b4h\]  
        Lea    Eax, \[drx\_2-@symbiont\_start\]\[eax\]  
        Mov    \[ecx+0ch\], Eax    ; Dr2  
  
        Mov    Eax, \[ecx+0b4h\]  
        Lea    Eax, \[drx\_3-@symbiont\_start\]\[eax\]  
        Mov    \[ecx+10h\], Eax    ; Dr3  
  
        Xor    Eax, Eax  
        And    Dword Ptr \[ecx+14h\], 0ffff0ff0h    ; 11111111111111110000111111110000b  
        Mov    Dword Ptr \[ecx+18h\], 155h ; 101010101b  
        Jmp    Drx\_ret  
  
Drx\_not\_breakpoint:  
        Cmp    Eax, Exception\_single\_step  
        Jnz    Drx\_not\_single  
        Call    Drx\_get\_counet  
  
        Dd        0    ; 记录单步断点出现次数  
  
Drx\_get\_counet:  
        Pop    Eax  
        Inc    Dword Ptr \[eax\]  
        Mov    Eax, \[eax\]  
        Cmp    Eax, 1        ; Dr0  
        Jnz    Loc\_401341  
        Not    Dword Ptr \[ecx+0b0h\] ; Context.regeax  
        Jmp    Drx\_processed  
  
Loc\_401341:  
        Cmp    Eax, 2        ; Dr1  
        Jnz    Loc\_401357  
        Mov    Eax, \[ecx+0b0h\]    ; Eax <- Context.regeax  
        Rol    Eax, 13h  
        Mov    \[ecx+0b0h\], Eax    ; Eax -> Context.regeax  
        Jmp    Drx\_processed  
  
Loc\_401357:  
        Cmp    Eax, 3        ; Dr2  
        Jnz    Loc\_401385  
  
        Add    Dword Ptr \[ecx+0b0h\], 78787878 ; Context.regeax  += 78787878h  
  
        Mov    Eax, \[ecx+0b0h\]    ; Eax <- Context.regeax  
        Mov    Ebx, \[ecx+0a4h\]    ; Ebx <- Context.regebx  
  
        Xchg    Ax, Bx  
        Add    Ax, Bx  
  
        Mov    \[ecx+0b0h\], Eax    ; Eax <- Context.regeax  
        Mov    \[ecx+0a4h\], Ebx    ; Ebx <- Context.regebx  
        Jmp    Drx\_processed  
  
Loc\_401385:  
        Mov    Eax, \[ecx+0a0h\]    ; Dr3  
                    ; Eax <- Context.regesi  
  
        Xor    Byte Ptr \[eax\],    55h  
  
Drx\_processed:  
  
        Xor    Eax, Eax  
        Jmp    Drx\_ret  
  
  
Drx\_not\_single:  
        Cmp    Eax, Exception\_int\_divide\_by\_zero  
        Jnz    Drx\_cannot\_process  
  
        Inc    Dword Ptr \[ecx+0b8h\] ; Context.regeip++  (2 Byte  Code)  
        Xor    Eax, Eax  
        And    \[ecx+4\], Eax    ; Context.idr0 <- 0  
        And    \[ecx+8\], Eax    ; Context.idr1 <- 0  
        And    \[ecx+0ch\], Eax    ; Context.idr2 <- 0  
        And    \[ecx+10h\], Eax    ; Context.idr3 <- 0  
        And    Dword Ptr \[ecx+14h\], 0ffff0ff0h    ; Context.idr6 <- 0ffff0ff0h  
                                                            ; 清 Bt,bs,bd,b3,b2,b1,b0 位  
        And    Dword Ptr \[ecx+18h\], 0dc00h ; 仅保留 Gd,ge 位  
        Jmp    Drx\_ret  
  
  
Drx\_cannot\_process:  
        Xor    Eax, Eax  
        Inc    Eax  
  
Drx\_ret:  
        Ret  
  
Drx\_handler    Endp  
  
  
Push\_drx\_handler:  
  
        Xor    Eax, Eax  
        Push    Dword Ptr    Fs:0  
        Mov    Fs:0, Esp  
  
        Int    3        ; 设置 Drx  
        Nop            ; Int 3  后紧跟 Nop 通常是为了兼容某些系统(9X 与 NT，发生 INT3 异常时,context.EIP 不一样。 )   
  
        Mov    Ecx, Ebp  
        Sub    Ecx, Esi  
        Xor    Ebx, Ebx  
        Xor    Eax, Eax  
  
Drx\_crc\_13db:  
        Lodsb  
        Add    Ebx, Eax  
        Loop    Drx\_crc\_13db  
        Mov    Eax, Ebx  
  
Drx\_0:  
        Clc            ; Not Eax  
        Nop  
        Lea    Esi, \[drx\_1-@symbiont\_start\]\[ebp\]  
        Mov    Ecx, @sumbiont\_end-drx\_1  
  
Drx\_decode\_13ef:  
        Mul    Ecx  
        Ror    Eax, Cl  
        Xor    \[esi\], Al  
        Inc    Esi  
        Inc    Eax  
        Aam  
        Loop    Drx\_decode\_13ef  
  
        Mov    Ecx, Drx\_handler-@symbiont\_start ; Esi -> Loader Data  
        Mov    Esi, Ebp                                    ; Mem Ptr Start  
        Xor    Eax, Eax  
  
Drx\_crc\_1404:  
        Xor    Al, \[esi\]  
        Ror    Eax, 8  
        Inc    Esi  
        Loop    Drx\_crc\_1404  
  
Drx\_1:  
  
        Cld    ; Rol    Eax, 13h  
        Nop  
        Mov    Ecx, @sumbiont\_end-drx\_2  
        Lea    Esi, \[drx\_2-@symbiont\_start\]\[ebp\]  
  
Drx\_decode\_1419:  
        Lea    Eax, \[ecx+eax\*4+key\_1\] ; 43h  
        Xor    \[esi\], Al  
        Aam  
        Inc    Esi  
        Loop    Drx\_decode\_1419  
  
        Mov    Ecx, Drx\_2-push\_drx\_handler  
        Shr    Ecx, 2        ; Ecx /= 4 (sizeof Dword)  
        Lea    Esi, \[push\_drx\_handler-@symbiont\_start\]\[ebp\]  
        Xor    Ebx, Ebx  
  
Drx\_crc\_1434:  
        Lodsd  
        Xor    Ebx, Eax  
        Loop    Drx\_crc\_1434  
  
Drx\_2:  
        Stc            ; Add    Eax, 78787878h  
                        ; Xchg  Ax, Bx  
                        ; Add    Ax, Bx  
        Nop  
        Mov    Ecx, @sumbiont\_end-drx\_3  
        Shr    Ecx, 2  
        Lea    Esi, \[drx\_3-@symbiont\_start\]\[ebp\]  
  
Drx\_decode\_1449:  
        Xor    Edx, Edx  
        Mul    Ebx  
        Add    Edx, 78787878  
        Xor    \[esi\], Edx  
        Mov    Eax, Ebx  
        Mov    Ebx, Edx  
        Add    Esi, 4  
        Loop    Drx\_decode\_1449  
  
        Lea    Esi, \[crypted\_code\_start-@symbiont\_start\]\[ebp\]  
        Mov    Ecx, @sumbiont\_end-crypted\_code\_start  
  
Drx\_decode\_1469:  
        Not    Byte Ptr \[esi\]  
  
Drx\_3:  
        Nop            ; Xor    Byte Ptr \[esi\], 55h  
        Nop  
        Inc    Esi  
        Loop    Drx\_decode\_1469  
  
Crypted\_code\_start:  
  
        Mov    Eax, 100h  
        Xor    Edx, Edx  
        Xor    Ebx, Ebx  
        Div    Ebx        ; :d  
        Nop  
  
        Pop    Dword Ptr    Fs:0  
        Pop    Eax

 4dd466d0e1e0cc123afbe504da2462ca