
### 1. 获取 `AdjustTokenPrivileges` 地址并调用

```
  0040756E  |.  894424 14     mov     dword ptr [esp+14], eax  
  00407572  |.  8B4424 0C     mov     eax, dword ptr [esp+C]  
  00407576  |.  894424 18     mov     dword ptr [esp+18], eax  
  0040757A  |.  C74424 10 010>mov     dword ptr [esp+10], 1  
  00407582  |.  83CB 02       or      ebx, 2  
  00407585  |.  895C24 1C     mov     dword ptr [esp+1C], ebx  
  00407589  |.  54            push    esp                      ; /pRetLen  
  0040758A  |.  6A 00         push    0                        ; |pPrevState = NULL  
  0040758C  |.  8B4424 08     mov     eax, dword ptr [esp+8]     ; |  
  00407590  |.  50            push    eax                      ; |PrevStateSize  
  00407591  |.  8D4424 1C     lea     eax, dword ptr [esp+1C]    ; |  
  00407595  |.  50            push    eax                      ; |pNewState  
  00407596  |.  6A 00         push    0                        ; |DisableAllPrivileges = FALSE  
  00407598  |.  8B4424 18     mov     eax, dword ptr [esp+18]    ; |  
  0040759C  |.  50            push    eax                      ; |hToken  
  0040759D  |.  E8 0EF6FFFF   call    <jmp.&ADVAPI32.AdjustTokenPrivil>; \AdjustTokenPrivileges  
  004075A2  |.  83C4 30       add     esp, 30  
  004075A5  |.  5B            pop     ebx  
  004075A6  \.  C3            retn  
```

### 3) 获取进程相关的一些函数地址

引用:

```
  0040705C  /$  53            push    ebx                      ;  (initial cpu selection)  
  0040705D  |.  BB 80364300   mov     ebx, 00433680  
  00407062  |.  833B 00       cmp     dword ptr [ebx], 0  
  00407065  |.  0F85 35010000 jnz     004071A0  
  0040706B  |.  68 B8714000   push    004071B8                 ; /kernel32.dll  
  00407070  |.  E8 9BFCFFFF   call    <jmp.&kernel32.GetModuleHandleA> ; \GetModuleHandleA  
  00407075  |.  8903          mov     dword ptr [ebx], eax  
  00407077  |.  833B 00       cmp     dword ptr [ebx], 0  
  0040707A  |.  0F84 20010000 je      004071A0  
  00407080  |.  68 C8714000   push    004071C8                 ; /createtoolhelp32snapshot  
  00407085  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407087  |.  50            push    eax                      ; |hModule  
  00407088  |.  E8 8BFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040708D  |.  A3 84364300   mov     dword ptr [433684], eax  
  00407092  |.  68 E4714000   push    004071E4                 ; /heap32listfirst  
  00407097  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407099  |.  50            push    eax                      ; |hModule  
  0040709A  |.  E8 79FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040709F  |.  A3 88364300   mov     dword ptr [433688], eax  
  004070A4  |.  68 F4714000   push    004071F4                 ; /heap32listnext  
  004070A9  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070AB  |.  50            push    eax                      ; |hModule  
  004070AC  |.  E8 67FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070B1  |.  A3 8C364300   mov     dword ptr [43368C], eax  
  004070B6  |.  68 04724000   push    00407204                 ; /heap32first  
  004070BB  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070BD  |.  50            push    eax                      ; |hModule  
  004070BE  |.  E8 55FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070C3  |.  A3 90364300   mov     dword ptr [433690], eax  
  004070C8  |.  68 10724000   push    00407210                 ; /heap32next  
  004070CD  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070CF  |.  50            push    eax                      ; |hModule  
  004070D0  |.  E8 43FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070D5  |.  A3 94364300   mov     dword ptr [433694], eax  
  004070DA  |.  68 1C724000   push    0040721C                 ; /toolhelp32readprocessmemory  
  004070DF  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070E1  |.  50            push    eax                      ; |hModule  
  004070E2  |.  E8 31FCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070E7  |.  A3 98364300   mov     dword ptr [433698], eax  
  004070EC  |.  68 38724000   push    00407238                 ; /process32first  
  004070F1  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  004070F3  |.  50            push    eax                      ; |hModule  
  004070F4  |.  E8 1FFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  004070F9  |.  A3 9C364300   mov     dword ptr [43369C], eax  
  004070FE  |.  68 48724000   push    00407248                 ; /process32next  
  00407103  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407105  |.  50            push    eax                      ; |hModule  
  00407106  |.  E8 0DFCFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040710B  |.  A3 A0364300   mov     dword ptr [4336A0], eax  
  00407110  |.  68 58724000   push    00407258                 ; /process32firstw  
  00407115  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407117  |.  50            push    eax                      ; |hModule  
  00407118  |.  E8 FBFBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040711D  |.  A3 A4364300   mov     dword ptr [4336A4], eax  
  00407122  |.  68 68724000   push    00407268                 ; /process32nextw  
  00407127  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407129  |.  50            push    eax                      ; |hModule  
  0040712A  |.  E8 E9FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  0040712F  |.  A3 A8364300   mov     dword ptr [4336A8], eax  
  00407134  |.  68 78724000   push    00407278                 ; /thread32first  
  00407139  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040713B  |.  50            push    eax                      ; |hModule  
  0040713C  |.  E8 D7FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407141  |.  A3 AC364300   mov     dword ptr [4336AC], eax  
  00407146  |.  68 88724000   push    00407288                 ; /thread32next  
  0040714B  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040714D  |.  50            push    eax                      ; |hModule  
  0040714E  |.  E8 C5FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407153  |.  A3 B0364300   mov     dword ptr [4336B0], eax  
  00407158  |.  68 98724000   push    00407298                 ; /module32first  
  0040715D  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  0040715F  |.  50            push    eax                      ; |hModule  
  00407160  |.  E8 B3FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407165  |.  A3 B4364300   mov     dword ptr [4336B4], eax  
  0040716A  |.  68 A8724000   push    004072A8                 ; /module32next  
  0040716F  |.  8B03          mov     eax, dword ptr [ebx]     ; |  
  00407171  |.  50            push    eax                      ; |hModule  
  00407172  |.  E8 A1FBFFFF   call    <jmp.&kernel32.GetProcAddress>   ; \GetProcAddress  
  00407177  |.  A3 B8364300   mov     dword ptr [4336B8], eax  
  0040717C  |.  68 B8724000   push    004072B8                 ; /module32firstw  
```
