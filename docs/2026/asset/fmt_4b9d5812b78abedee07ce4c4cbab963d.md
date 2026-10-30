
```
Address                     Instruction                         Result                      

.text:00413131              add     esp, 0FFFFFFF0h             ESP=0019FF34 CF=1 AF=0                      

.text:loc_413150            xor     eax, eax                    EAX=00000000 CF=0 PF=1 ZF=1                 

.text:loc_413184            mov     check_count, eax                                                                                                             

.text:loc_4131BA            pop     eax             ; Symbolized memory: 0x19ff34   EAX=64636261 8                                                                         

.text:loc_4131EB            mov     ecx, eax        ; Symbolized regs: eax  ECX=64636261                                                                                  

.text:loc_41321F            pop     eax             ; Symbolized memory: 0x19ff38   EAX=68676665 C                                                                         

.text:loc_413254            mov     ebx, eax        ; Symbolized regs: eax  EBX=68676665                                                                                  

.text:loc_413289            pop     eax             ; Symbolized memory: 0x19ff3c   EAX=6C6B6A69 0                                                                      

.text:loc_4132B5            mov     edx, eax        ; Symbolized regs: eax  EDX=6C6B6A69                                                                                 

.text:loc_4132AD            mov     edx, eax        ; Symbolized regs: eax                                                                                                 

.text:loc_4132E2            mov     eax, ecx        ; Symbolized regs: ecx  EAX=64636261                                                                                 

.text:loc_413316            sub     eax, ebx        ; Symbolized regs: eax ebx  EAX=FBFBFBFC CF=1 AF=1 ZF=0                                               

.text:loc_413349            shl     eax, 2          ; Symbolized regs: eax cf of pf sf zf   EAX=EFEFEFF0                                                               

.text:loc_413380            add     eax, ecx        ; Symbolized regs: eax ecx  EAX=54535251 PF=0                                                         

.text:loc_4133B5            add     eax, edx        ; Symbolized regs: eax edx  EAX=C0BEBCBA CF=0 SF=1                                                   

.text:loc_4133E9            sub     eax, 0EAF917E2h ; Symbolized regs: eax  EAX=D5C5A4D8 CF=1 PF=1 OF=0              





.text:loc_413455            add     eax, ecx        ; Symbolized regs: eax ecx  EAX=3A290739 ZF=0                                                   

.text:loc_413489            sub     eax, ebx        ; Symbolized regs: eax ebx  EAX=D1C1A0D4                                                               

.text:loc_4134BF            mov     ebx, eax        ; Symbolized regs: eax  EBX=D1C1A0D4                                                                     

.text:loc_4134F3            shl     eax, 1          ; Symbolized regs: eax cf of pf sf zf   EAX=A38341A8                                                              

.text:loc_413525            add     eax, ebx        ; Symbolized regs: eax ebx  EAX=7544E27C SF=0                                                   

.text:loc_413559            add     eax, ecx        ; Symbolized regs: eax ecx  EAX=D9A844DD CF=0 PF=1                                                    

.text:loc_41358F            mov     ecx, eax        ; Symbolized regs: eax  ECX=D9A844DD                                                                      

.text:loc_4135C3            add     eax, edx        ; Symbolized regs: eax edx  EAX=4613AF46 CF=1 PF=0 AF=1 SF=0                                    

.text:loc_4135F7            sub     eax, 0E8F508C8h ; Symbolized regs: eax  EAX=5D1EA67E PF=1    







.text:loc_413665            mov     eax, ecx                    EAX=D9A844DD                                

.text:loc_41365D            mov     eax, ecx                                                                                                                        

.text:loc_4136A7            sub     eax, edx                    EAX=6D3CDA74 CF=0 AF=0 ZF=0 OF=1                                                                    

.text:loc_4136D8            sub     eax, 0C0A3C68h              EAX=61329E0C AF=1 OF=0     





.text:loc_413747            pop     eax                         EAX=00413131 ESP=0019FF44                                                                         

.text:loc_413777            xor     eax, 8101h                  EAX=0041B030 AF=0 ZF=0                                                                             

.text:loc_4137A9            mov     edi, eax                    EDI=0041B030                                                                                       

.text:loc_4137E2            xor     eax, eax                    EAX=00000000 ZF=1                                                                                                                                         

.text:loc_413830            call    loc_413841                  ESP=0019FF40                                                                                       

.text:loc_41385C            pop     eax                         EAX=00413835 ESP=0019FF44                                                                           

.text:loc_41388E            push    eax                         ESP=0019FF40                                                                                       

.text:loc_4138BA            mov     edi, eax                    EDI=00413835                                                                                      

.text:loc_4138B2            mov     edi, eax 







.text:loc_4138E6            push    4E000969h                   ESP=0019FF3C                                                                                      

.text:loc_41391F            pop     eax                         EAX=4E000969 ESP=0019FF40                                                                          

.text:loc_413950            xor     eax, edx                    EAX=226B6300 ZF=0                                                                                                                     

.text:loc_4139B4+1          xor     eax, 10A3Eh                 EAX=226A693E PF=0                                                                                

.text:loc_413A1C            xor     eax, ebx                    EAX=F3ABC9EA SF=1                                                                      

.text:loc_413A4C+1          xor     eax, 22511E14h              EAX=D1FAD7FE                                                                            

.text:loc_413AB5+1          xor     eax, 61642Dh                EAX=D19BB3D3                                                                                       

.text:loc_413B83            xor     eax, check_count                                                                                                              

D19BB3D3                    Debug event: Exception -1073741819 occurred: The instruction at 0xD19BB3D3 referenced xD19BB3D3. The memory could not be executed    Debug event: Exception -1073741819 occurred: The instruction at referenced memory at 0xD19BB3D3. The memory could not be executed    

.text:loc_413BBB            jmp     eax                         EFL=00010382 TF=1 RF=1
```
照着汇编抄了一份`z3`
