

# anti wmware

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-120.htm`](https://www.pediy.com/kssd/pediy09/pediy09-120.htm)

> #include <windows.h>  
> #pragma comment(linker, "/entry:main")  
>   
> int main()  
> {  
>     
>   \_\_asm  
>   {  
>     mov    dx, 1  
> \_l1:  
>     verr  dx        //其中的 dx, 可以为其他。只要 dx & 4 == TRUE, 那么就会在 vmware 中发生异常，而外部没有发现这种现象  
>     shl    dx, 1  
>     jmp    \_l1  
>   }  
>   
>   return 0;  
> } 

> bool VMWareTest()  
> {  
>   BYTE PortValue1,PortValue2;  
>   \_\_try  
>   {  
>     \_\_asm  
>     {  
>       pushad   
>       pushfd  
>       xor ebx,ebx  
>       mov ecx,0xa   
>       mov eax, 'VMXh'      ; EAX=magic  
>       mov dx, 'VX'      ; DX=magic  
>       in  eax, dx        ; specially processed io cmd  
>       cmp ebx, 'VMXh'      ; also eax/ecx modified (maybe vmw/os ver?)  
>       jne local\_001  
>       mov gInVMWARE,1  
> local\_001:  
>       popfd  
>       popad  
>     }  
>   }  
>   \_\_except(EXCEPTION\_EXECUTE\_HANDLER)  
>   {  
>     gInVMWARE=false;  
>   }  
>   return gInVMWARE;  
> }  
>   
> bool VirtualPCTest()  
> {  
>   \_\_try  
>   {  
>     \_\_asm  
>     {  
>       pushad        
>       mov  ebx, 0 // Flag  
>       mov  eax, 1 // VPC function number  
>       \_\_emit 0Fh  
>       \_\_emit 3Fh  
>       \_\_emit 07h  
>       \_\_emit 0Bh  
>       test ebx, ebx  
>       jnz local\_001  
>       mov gInVirtualPC,1  
> local\_001:  
>       popad  
>     }  
>   
>   }  
>   \_\_except(EXCEPTION\_EXECUTE\_HANDLER)  
>   {  
>     gInVirtualPC=false;  
>   }  
>   if(gInVirtualPC)  
>     DbgPrint("Syser : Host machine is VirtualPC !\\n");  
>   return gInVirtualPC;  
> } 

> 楼主提的指令是 vmware 虚拟 cpu 不支持的,所以会异常退出.  
> 现在 007 里都包含此指令,其实可以直接硬断 nop 掉,就可以继续在 vmware 里跑了. 

 4dd466d0e1e0cc123afbe504da2462ca