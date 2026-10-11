
### 【返回到 EP 处】

代码:

```
00629459    0F89 2EFDFFFF         jns     0062918D                    
00629548                      push    00628134                        ; Morph instruction
006297C2    C3                retn    

//  检测 EP 处是否有 INT3 断点...
                
00628305    8D05 59946200     lea     eax, dword ptr [<[ModuleEntryPoint>]]
006282F7                      push    006295BC                        ; Morph instruction
00629128                      push    ebp                           ; Morph instruction
0062912B    8BEC              mov     ebp, esp                
00629130                      push    ecx                           ; Morph instruction
00629133    8945 FC           mov     dword ptr [ebp-4], eax     
00628117    8B45 FC           mov     eax, dword ptr [ebp-4]    
       
0062811A    8A00              mov     al, byte ptr [eax]              ; EP 处的第一个字节.. 
0062811C    2C 99             sub     al, 99           
          
00629358    8B12              mov     edx, dword ptr [edx]            ; [edx] = 0x00629459
006287F7    F62A              imul    byte ptr [edx]           
006287F9    3C A4             cmp     al, 0A4                ; 等于就出错。。        
006287FB    0F85 09030000     jnz     00628B0A                    
006287FB    0F85 09030000     jnz     00628B0A      

          
00628B0A    59                pop     ecx                            
00628B10                      pop    ebp                          ; Morph instruction
00628B11    C3                retn                                

// 这里两次 PUSH 都是作为 返回地址
// 
// push 4C1460
// push 628A7B
//
//
00628516                      mov     eax, 4C1460                 ; eax = 4C1460 ==> 4C1460 就是 VM-OEP 处了..
00628DB2                      push    eax                         ; 先入栈.. 等下就返回到 VMOEP...

// push 628A7B

00628DB5    57                push    edi                        
00628DB6                      mov     edi, 628A7B                 ; edi = 628A7B
00628B63    873C24            xchg    dword ptr [esp], edi       
 
00628A84                      push    ebp                         ; Morph instruction
00628A87    8BEC              mov     ebp, esp                
00628A89    51                push    ecx     
                      
00628832    56                push    esi                         
006292F1                      mov     esi, -4                     ; esi = -4
00629309    03F5              add     esi, ebp                    [ebp-4]

// eax = 004C1460 
0062882B    8906              mov     dword ptr [esi], eax        ; mov dword ptr [ebp-4], eax
00628D49    5E                pop     esi    
         

00628D4A    8B45 FC           mov     eax, dword ptr [ebp-4]       
00628D4D    8A00              mov     al, byte ptr [eax]      ; [EAX] = [004C1460] =  0xE9     
00628D4F    2C 99             sub     al, 99                   
00628D51                      mov     edx, -4                     ; edx = -4
00628DE2    03D5              add     edx, ebp                 
0062931F    8B12              mov     edx, dword ptr [edx]         
00629321    F62A              imul    byte ptr [edx]           
00629323    3C A4             cmp     al, 0A4              ; 检测 VmOep 处是否有 CC 断点..         
00629325    0F85 56040000     jnz     00629781                  ; 这句下面有个 SEH..



00628441                      pop    ecx                          ; Morph instruction
00628442    5D                pop     ebp                        
00628F0F    C3                retn             
                
00628A7B    C3                retn                            ; 从这里就是跳到 Vm-Oep 了...

```
