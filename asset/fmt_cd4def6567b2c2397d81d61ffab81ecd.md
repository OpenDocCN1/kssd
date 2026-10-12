
## 代码

```
00401B8D    > \FF35 144040>push dword ptr ds:\[404014\]                        ;  write2.00404070
00401B93    .  8D85 C4FEFF>lea eax,dword ptr ss:\[ebp-13C\]
00401B99    .  50           push eax
00401B9A    .  8D85 58FCFF>lea eax,dword ptr ss:\[ebp-3A8\]
00401BA0    .  68 F0434000 push write2.004043F0                                ;  ASCII "%s\\%sboot.class"
00401BA5    .  50           push eax
00401BA6    .  FFD7        call edi
00401BA8    .  83C4 10     add esp,10
00401BAB    .  33C0        xor eax,eax
00401BAD    .  50           push eax                                           ; /hTemplateFile => NULL
00401BAE    .  50           push eax                                           ; |Attributes => 0
00401BAF    .  6A 02       push 2                                             ; |Mode = CREATE_ALWAYS
00401BB1    .  50           push eax                                           ; |pSecurity => NULL
00401BB2    .  6A 03       push 3                                             ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
00401BB4    .  8D85 58FCFF>lea eax,dword ptr ss:\[ebp-3A8\]                   ; |
00401BBA    .  68 000000C0 push C0000000                                       ; |Access = GENERIC_READ|GENERIC_WRITE
00401BBF    .  50           push eax                                           ; |FileName
00401BC0    .  FF15 383040>call dword ptr ds:\[<&KERNEL32.CreateFileA>\]      ; \CreateFileA
                                ；jexepackboot.class 生成，但无内容
00401BC6    .  83F8 FF     cmp eax,-1
00401BC9    .  8945 F0     mov dword ptr ss:\[ebp-10\],eax
00401BCC    .  74 6B       je short write2.00401C39
00401BCE    .  8B45 E8     mov eax,dword ptr ss:\[ebp-18\]
00401BD1    .  8B4D DC     mov ecx,dword ptr ss:\[ebp-24\]
00401BD4    .  6A 01       push 1
00401BD6    .  8B5C01 18   mov ebx,dword ptr ds:\[ecx+eax+18\]
00401BDA    .  53           push ebx
00401BDB    .  FFD6        call esi
00401BDD    .  8945 F4     mov dword ptr ss:\[ebp-C\],eax
00401BE0    .  59           pop ecx
00401BE1    .  33C0        xor eax,eax
00401BE3    .  59           pop ecx
00401BE4    .  85DB        test ebx,ebx
00401BE6    .  7E 28       jle short write2.00401C10
00401BE8    .  8B55 DC     mov edx,dword ptr ss:\[ebp-24\]
00401BEB    .  2B55 F4     sub edx,dword ptr ss:\[ebp-C\]
00401BEE    .  8955 E0     mov dword ptr ss:\[ebp-20\],edx
00401BF1    .  EB 03       jmp short write2.00401BF6
00401BF3    >  8B55 E0     mov edx,dword ptr ss:\[ebp-20\]
00401BF6    >  8B4D F4     mov ecx,dword ptr ss:\[ebp-C\]
00401BF9    .  8B75 E8     mov esi,dword ptr ss:\[ebp-18\]
00401BFC    .  03C8        add ecx,eax
00401BFE    .  03D1        add edx,ecx
00401C00    .  8A5432 1C   mov dl,byte ptr ds:\[edx+esi+1C\]
00401C04    .  32D0        xor dl,al
00401C06    .  80EA 64     sub dl,64
00401C09    .  40           inc eax
00401C0A    .  3BC3        cmp eax,ebx
00401C0C    .  8811        mov byte ptr ds:\[ecx\],dl
00401C0E    .^ 7C E3       jl short write2.00401BF3
00401C10    >  8D45 C8     lea eax,dword ptr ss:\[ebp-38\]
00401C13    .  6A 00       push 0                                             ; /pOverlapped = NULL
00401C15    .  50           push eax                                           ; |pBytesWritten
00401C16    .  53           push ebx                                           ; |nBytesToWrite
00401C17    .  FF75 F4     push dword ptr ss:\[ebp-C\]                        ; |Buffer
00401C1A    .  FF75 F0     push dword ptr ss:\[ebp-10\]                       ; |hFile
00401C1D    .  FF15 3C3040>call dword ptr ds:\[<&KERNEL32.WriteFile>\]        ; \WriteFile
                                ；写入 jexepackboot.class
00401C23    .  85C0        test eax,eax
00401C25    .  75 07       jnz short write2.00401C2E
00401C27    .  C745 F8 112>mov dword ptr ss:\[ebp-8\],2711
00401C2E    >  FF75 F0     push dword ptr ss:\[ebp-10\]                       ; /hObject
00401C31    .  FF15 083040>call dword ptr ds:\[<&KERNEL32.CloseHandle>\]     ; \CloseHandle
00401C37    .  EB 07       jmp short write2.00401C40
00401C39    >  C745 F8 112>mov dword ptr ss:\[ebp-8\],2711
00401C40    >  33DB        xor ebx,ebx
00401C42    .  53           push ebx
00401C43    .  68 EC434000 push write2.004043EC                                ;  ASCII "mf"
00401C48    .  FF75 E4     push dword ptr ss:\[ebp-1C\]
00401C4B    .  E8 AEF7FFFF call write2.004013FE
00401C50    .  53           push ebx
00401C51    .  68 E4434000 push write2.004043E4                                ;  ASCII "minver"
00401C56    .  FF75 E4     push dword ptr ss:\[ebp-1C\]
00401C59    .  8945 F4     mov dword ptr ss:\[ebp-C\],eax
00401C5C    .  E8 9DF7FFFF call write2.004013FE
00401C61    .  68 D4444000 push write2.004044D4
00401C66    .  68 E0434000 push write2.004043E0                                ;  ASCII "jop"
00401C6B    .  FF75 E4     push dword ptr ss:\[ebp-1C\]
00401C6E    .  8945 F0     mov dword ptr ss:\[ebp-10\],eax
00401C71    .  E8 88F7FFFF call write2.004013FE
00401C76    .  FF75 E8     push dword ptr ss:\[ebp-18\]                       ; /block
00401C79    .  A3 CC444000 mov dword ptr ds:\[4044CC\],eax                   ; |
00401C7E    .  FF15 A03040>call dword ptr ds:\[<&MSVCRT.free>\]               ; \free
00401C84    .  83C4 28     add esp,28
00401C87     395D F8     cmp dword ptr ss:\[ebp-8\],ebx
00401C8A    .  895D E8     mov dword ptr ss:\[ebp-18\],ebx
00401C8D    .  0F85 350200>jnz write2.00401EC8
00401C93    .  BE E8424000 mov esi,write2.004042E8                             ;  ASCII "This EXE was produced using an           

                        UNREGISTERED version of JexePack.  Any                


                        distribution of this EXE is prohibited 

                        and a violation of US Copyright law and              

                        international treaty.

An EXE produced 

                        with a registered JexePack does not display          

                        this "..."
00401C98    .  56           push esi                                           ; /s => "This EXE was produced using an            

                        UNREGISTERED version of JexePack.  Any                


                        distribution of this EXE is prohibited and a          

                        violation of US Copyright law and                    

                        international treaty.

An EXE produced with a                    

                        registered JexePack does not display this "..."
00401C99    .  E8 B8040000 call <jmp.&MSVCRT.strlen>                          ; \strlen
00401C9E    .  59           pop ecx
00401C9F    .  33C9        xor ecx,ecx
00401CA1    .  85C0        test eax,eax
00401CA3    .  EB 6A       jmp short write2.00401D0F
00401CA5    >  0FBE91 E842>movsx edx,byte ptr ds:\[ecx+4042E8\]
00401CAC    .  03D1        add edx,ecx
00401CAE    .  69D2 71FEC5>imul edx,edx,1FC5FE71
00401CB4    .  33DA        xor ebx,edx
00401CB6    .  41           inc ecx
00401CB7    .  3BC8        cmp ecx,eax
00401CB9    .^ 7C EA       jl short write2.00401CA5
00401CBB    .  81FB B70D15>cmp ebx,94150DB7
00401CC1    .  75 40       jnz short write2.00401D03
00401CC3    .  FF15 4C3040>call dword ptr ds:\[<&KERNEL32.GetTickCount>\]     ; [GetTickCount
00401CC9    .  8945 E0     mov dword ptr ss:\[ebp-20\],eax
00401CCC    .  33DB        xor ebx,ebx
00401CCE    >  FF15 4C3040>call dword ptr ds:\[<&KERNEL32.GetTickCount>\]     ; [GetTickCount
00401CD4    .  2B45 E0     sub eax,dword ptr ss:\[ebp-20\]                   ；两次得到的 tick 数相减
00401CD7    .  3D E8030000 cmp eax,3E8    
```
