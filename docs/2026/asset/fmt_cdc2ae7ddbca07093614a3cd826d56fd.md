
```
0120BF5C    C785 AD0D9E09 00000000mov     dword ptr [0110DDD0], 0         
0120BF66    C685 4D229E09 45      mov     byte ptr [0110F270], 45         
0120BF6D    68 6969728E           push    8E726969                        
0120BF72    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120BF78    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120BF7E    FFD0                  call    eax                       ; 查找API函数ExitProcess = 7C81CB12   (这个是真实的模块地址，非镜像地址)   

0120BF80    8985 912E9E09         mov     dword ptr [0110FEB4], eax   ; eax = 7C81CB12    --> ExitProcess
0120BF86    C685 4D229E09 43      mov     byte ptr [0110F270], 43         
0120BF8D    68 5E6B679C           push    9C676B5E                        
0120BF92    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120BF98    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120BF9E    FFD0                  call    eax                          ; 查找API函数CreateThread

0120BFA0    8985 EDEDAD09         mov     dword ptr [0120BE10], eax   ; eax = 7C8106D7 --> CreateThread
0120BFA6    C685 4D229E09 54      mov     byte ptr [0110F270], 54         
0120BFAD    68 296862EE           push    EE626829                        
0120BFB2    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120BFB8    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120BFBE    FFD0                  call    eax                             

0120BFC0    8985 F1EDAD09         mov     dword ptr [0120BE14], eax   ; eax = 7C81CB3B  --> TerminateThread
0120BFC6    C685 4D229E09 45      mov     byte ptr [0110F270], 45         
0120BFCD    68 C23860DA           push    DA6038C2                        
0120BFD2    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120BFD8    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120BFDE    FFD0                  call    eax                             

0120BFE0    8985 F5EDAD09         mov     dword ptr [0120BE18], eax   ; eax = 7C80C0F8 --> ExitThread
0120BFE6    83BD 391A9E09 00      cmp     dword ptr [0110EA5C], 0         
0120BFED    0F85 45000000         jnz     0120C038                        

0120C038    C685 4D229E09 52      mov     byte ptr [0110F270], 52         
0120C03F    68 0DE44ABC           push    BC4AE40D                        
0120C044    FFB5 E51F9E09         push    dword ptr [0110F008]            
0120C04A    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C050    FFD0                  call    eax                             

0120C052    8985 F9EDAD09         mov     dword ptr [0120BE1C], eax   ; eax = ntdll.RtlEnterCriticalSection
0120C058    C685 4D229E09 52      mov     byte ptr [0110F270], 52         
0120C05F    68 484A4CA6           push    A64C4A48                        
0120C064    FFB5 E51F9E09         push    dword ptr [0110F008]            
0120C06A    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C070    FFD0                  call    eax                             

0120C072    8985 FDEDAD09         mov     dword ptr [0120BE20], eax   ; eax = 7C9210E0 ntdll.RtlLeaveCriticalSection
0120C078    C685 4D229E09 52      mov     byte ptr [0110F270], 52         
0120C07F    68 EAB442FD           push    FD42B4EA                        
0120C084    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120C08A    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C090    FFD0                  call    eax                             

0120C092    8985 E5199E09         mov     dword ptr [0110EA08], eax   ; eax = 7C801812 ReadFile
0120C098    C685 4D229E09 77      mov     byte ptr [0110F270], 77         
0120C09F    68 A3A897F3           push    F397A8A3                        
0120C0A4    FFB5 19119E09         push    dword ptr [0110E13C]            
0120C0AA    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C0B0    FFD0                  call    eax                             

0120C0B2    8985 89139E09         mov     dword ptr [0110E3AC], eax   ; eax = 77D1A8AD wsprintfA
0120C0B8    C685 4D229E09 52      mov     byte ptr [0110F270], 52         
0120C0BF    68 1731DD8A           push    8ADD3117                        
0120C0C4    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120C0CA    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C0D0    FFD0                  call    eax                             

0120C0D2    8985 BD089E09         mov     dword ptr [0110D8E0], eax   ; eax = 7C812AA9 RaiseException
0120C0D8    C685 4D229E09 56      mov     byte ptr [0110F270], 56         
0120C0DF    68 396D1FD4           push    D41F6D39                        
0120C0E4    FFB5 55209E09         push    dword ptr [0110F078]            
0120C0EA    8D85 711DAA09         lea     eax, dword ptr [011CED94]       
0120C0F0    FFD0                  call    eax                           ; 这里是查找镜像API函数VirtualFree地址

0120C0F2    8985 B91B9E09         mov     dword ptr [0110EBDC], eax   ; eax = 00938F84 VirtualFree
0120C0F8    C685 4D229E09 47      mov     byte ptr [0110F270], 47         
0120C0FF    68 7A10DC81           push    81DC107A                        
0120C104    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120C10A    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C110    FFD0                  call    eax                             

0120C112    8985 11059E09         mov     dword ptr [0110D534], eax   ; eax =7C80B56F GetModuleFileNameA
0120C118    C685 4D229E09 47      mov     byte ptr [0110F270], 47         
0120C11F    68 E2B50F8A           push    8A0FB5E2                        
0120C124    FFB5 812E9E09         push    dword ptr [0110FEA4]            
0120C12A    8D85 52369E09         lea     eax, dword ptr [01110675]       
0120C130    FFD0                  call    eax                             

0120C132    8985 1DEEAD09         mov     dword ptr [0120BE40], eax   ; EAX = 7C80AE40  GetProcAddress
0120C138    83BD 3D1D9E09 00      cmp     dword ptr [0110ED60], 0         
0120C13F    0F84 40040000         je      0120C585                        
```

2. 申请两块中间过渡的内存，用来存储API复制代码和经过乱序变形后的API函数代码
