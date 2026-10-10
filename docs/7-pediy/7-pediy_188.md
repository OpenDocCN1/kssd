

# 在代码段最后添加了几行，为什么有的机器上能用，有的机器上就不行？

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-297.htm`](https://www.pediy.com/kssd/pediy09/pediy09-297.htm)

> 一个 DLL，他里面原来的代码是这样  
>   
> 1000AAFA    mov     dword ptr \[1000F438\], ebp        ; 我要把\[1000F438\]里放入 1  
> 1000AB00    mov     dword ptr \[1000F234\], ebp        ; 我要把\[1000F234\]里放入 270Fh  
> 1000AB06    call    dword ptr \[<&USER32.SetTimer>\]   ; \\  
> 我是这样做的，改成这样  
> 1000AAFA    jmp     1000CDD0 ; 在代码段最后加上些代码，然后跳到那  
> 1000AAFF    nop  
> 1000AB00    dword ptr \[1000F234\], ebp        ; |  
> 1000AB06    call    dword ptr \[<&USER32.SetTimer>\]   ; \\  
>   
> 代码段最后找个空白地方加上  
> 1000CDD0     mov     dword ptr \[1000F438\], 1  
> 1000CDDA     mov     dword ptr \[1000F234\], 270F  
> 1000CDE4     jmp     1000AB06  
>   
> 改好之后，在我的机器上能用，在其他人的机器上却运行不了(同样是 xp sp2)，  exe 刚 Load 这个 dll 就自动关闭了，这可能是什么原因？谢谢。 

> 重定位 

> 1000AAFA   NOP  
> 1000AAFB   MOV EAX,1000CDD0  
> 1000AB00   JMP EAX  
>   
> 1000CDD0   SUB EAX,0CDD0  
> 1000CDD5   MOV DWORD PTR \[EAX+0F438\],1  
> 1000CDDF   MOV DWORD PTR \[EAX+0F234\],270F  
> 1000CDE9   JMP 1000AB06 

> ccfer 的方法很巧妙，学习。  
>    
> 不修改重定位表，我也来个传统的方法：  
> 
> 代码:
> 
> * * *
> 
> `1000AAFA  nop   1000AAFB  push    1000CDD0 //利用了原重定位表   1000AB00  retn` 
> 
> * * *
> 
>   
>  代码:
> 
> * * *
> 
> `    1000CDD0  push    edx   1000CDD1  call    1000CDD6                          1000CDD6  pop     edx   1000CDD7  sub     edx, 1000CDD6   1000CDDD  mov     dword ptr [edx+1000F438], 1   1000CDE7  mov     dword ptr [edx+1000F234], 270F   1000CDF1  pop     edx   1000CDF2  jmp     1000AB06                         ` 
> 
> * * *

 4dd466d0e1e0cc123afbe504da2462ca