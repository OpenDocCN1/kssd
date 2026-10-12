
```
  0040741C  |.  68 F4010000      |push    1F4                           ; /Timeout = 500. ms  
  00407421  |.  E8 3ADDFFFF      |call    <jmp.&kernel32.Sleep>         ; \\Sleep  
  00407426  |.  8D95 E8FDFFFF    |lea     edx, dword ptr [ebp-218]      ;  Sleep 一下，免得被发现的太快了  
  0040742C  |.  8B45 EC          |mov     eax, dword ptr [ebp-14]  
  0040742F  |.  E8 9CEBFFFF      |call    00405FD0  
  00407434  |.  8B85 E8FDFFFF    |mov     eax, dword ptr [ebp-218]  
  0040743A  |.  E8 BDC9FFFF      |call    00403DFC  
  0040743F  |.  50              |push    eax  
  00407440  |.  6A 01           |push    1  
  00407442  |.  8D95 E4FDFFFF    |lea     edx, dword ptr [ebp-21C]  
  00407448  |.  8B45 F0          |mov     eax, dword ptr [ebp-10]  
  0040744B  |.  E8 80EBFFFF      |call    00405FD0  
  00407450  |.  8B85 E4FDFFFF    |mov     eax, dword ptr [ebp-21C]  
  00407456  |.  E8 A1C9FFFF      |call    00403DFC  
  0040745B  |.  8BC8            |mov     ecx, eax  
  0040745D  |.  BA 14754000     |mov     edx, 00407514                 ;  software\\lydl  
  00407462  |.  B8 02000080     |mov     eax, 80000002  
  00407467  |.  E8 ECECFFFF     |call    00406158  
  0040746C  |>  8BC6            mov     eax, esi  
  0040746E  |.  E8 41B7FFFF     |call    00402BB4  
  00407473  |.  E8 88B1FFFF     |call    00402600  
  00407478  |.  84C0            |test    al, al  
  0040747A  |.^ 0F84 F4FDFFFF   \\je      00407274                    
  00407480  |.  8BC6            mov     eax, esi  
  00407482  |.  E8 D5B6FFFF     call    00402B5C  
  00407487  |.  E8 74B1FFFF     call    00402600  
  0040748C  |.  33C0            xor     eax, eax  
  0040748E  |.  5A             |pop     edx  
  0040748F  |.  59             |pop     ecx  
  00407490  |.  59             |pop     ecx  
  00407491  |.  64:8910        |mov     dword ptr fs:[eax], edx  
  00407494  |.  68 BE744000    |push    004074BE  
  00407499  |>  8D85 E4FDFFFF  |lea     eax, dword ptr [ebp-21C]  
  0040749F  |.  BA 0E000000    |mov     edx, 0E  
  004074A4  |.  E8 F3C5FFFF    |call    00403A9C  
  004074A9  |.  8D45 E8         |lea     eax, dword ptr [ebp-18]  
  004074AC  |.  BA 06000000    |mov     edx, 6  
  004074B1  |.  E8 E6C5FFFF    |call    00403A9C  
  004074B6  \\.  C3            |retn  
```

`utf8.nls` 是一个字符集文件

`F7` 跟进 `004073CE` 的 `call 00405684`

```
  00405684  /$  55              push    ebp  
  00405685  |.  8BEC            mov     ebp, esp  
  00405687  |.  81C4 F4FEFFFF   add     esp, -10C  
  0040568D  |.  53              push    ebx  
  0040568E  |.  56              push    esi  
  0040568F  |.  33C9            xor     ecx, ecx  
  00405691  |.  898D F4FEFFFF   mov     dword ptr [ebp-10C], ecx  
  00405697  |.  898D F8FEFFFF   mov     dword ptr [ebp-108], ecx  
  0040569D  |.  8BF2            mov     esi, edx  
  0040569F  |.  8BD8            mov     ebx, eax  
  004056A1  |.  33C0            xor     eax, eax  
  004056A3  |.  55              push    ebp  
  004056A4  |.  68 85574000    push    00405785  
  004056A9  |.  64:FF30        push    dword ptr fs:[eax]  
  004056AC  |.  64:8920        mov     dword ptr fs:[eax], esp  
  004056AF  |.  8D85 FCFEFFFF  lea     eax, dword ptr [ebp-104]  
  004056B5  |.  BA 04010000    mov     edx, 104  
  004056BA  |.  E8 41F4FFFF    call    00404B00  
  004056BF  |.  83EB 01        sub     ebx, 1                      ;  Switch (cases 0..2)  
  004056C2  |.  72 0A          jb      short 004056CE  
  004056C4  |.  74 43          je      short 00405709  
  004056C6  |.  4B             dec     ebx  
  004056C7  |.  74 7B          je      short 00405744  
  004056C9  |.  E9 99000000    jmp     00405767  
  004056CE  |>  68 04010000    push    104                         ; /BufSize = 104 (260.); Case 0 of switch 004056BF  
  004056D3  |.  8D85 FCFEFFFF  lea     eax, dword ptr [ebp-104]     ; |  
  004056D9  |.  50            push    eax                         ; |Buffer  
  004056DA  |.  E8 99F3FFFF   call    <jmp.&kernel32.GetWindowsDirectoryA> ; \\GetWindowsDirectoryA  
  004056DF  |.  8D85 F8FEFFFF lea     eax, dword ptr [ebp-108]     ;  通过 GetWindowsDirectory 获得 windows 所在目录  
  004056E5  |.  8D95 FCFEFFFF lea     edx, dword ptr [ebp-104]  
  004056EB  |.  B9 04010000   mov     ecx, 104  
  004056F0  |.  E8 EFE4FFFF   call    00403BE4  
  004056F5  |.  8B95 F8FEFFFF mov     edx, dword ptr [ebp-108]  
  004056FB  |.  8BC6          mov     eax, esi  
  004056FD  |.  B9 9C574000   mov     ecx, 0040579C                 ;  \\  
  00405702  |.  E8 41E5FFFF   call    00403C48  
  00405707  |.  EB 5E         jmp     short 00405767  
  00405709  |>  68 04010000   push    104                         ; /BufSize = 104 (260.); Case 1 of switch 004056BF  
  0040570E  |.  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]     ; |  
  00405714  |.  50            push    eax                         ; |Buffer  
  00405715  |.  E8 3EF3FFFF   call    <jmp.&kernel32.GetSystemDirectoryA> ; \\GetSystemDirectoryA  
  0040571A  |.  8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]     ;  通过 GetSystemDirectory 获得 system 所在目录  
  00405720  |.  8D95 FCFEFFFF lea     edx, dword ptr [ebp-104]  
  00405726  |.  B9 04010000   mov     ecx, 104  
  0040572B  |.  E8 B4E4FFFF   call    00403BE4  
  00405730  |.  8B95 F4FEFFFF mov     edx, dword ptr [ebp-10C]  
  00405736  |.  8BC6          mov     eax, esi  
  00405738  |.  B9 9C574000   mov     ecx, 0040579C                 ;  \\  
  0040573D  |.  E8 06E5FFFF   call    00403C48  
  00405742  |.  EB 23         jmp     short 00405767  
  00405744  |>  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]     ;  Case 2 of switch 004056BF  
  0040574A  |.  50            push    eax                         ; /Buffer  
  0040574B  |.  68 04010000   push    104                         ; |BufSize = 104 (260.)  
  00405750  |.  E8 0BF3FFFF   call    <jmp.&kernel32.GetTempPathA> ; \\GetTempPathA  
  00405755  |.  8BC6          mov     eax, esi                      ;  通过 GetTempPath 获得系统的 TEMP 文件夹的路径，然后会在该文件夹下面创建一个文件夹，所下载的一堆东西就在那个文件夹里面  
  00405757  |.  8D95 FCFEFFFF lea     edx, dword ptr [ebp-104]  
  0040575D  |.  B9 04010000   mov     ecx, 104  
  00405762  |.  E8 7DE4FFFF   call    00403BE4  
  00405767  |>  33C0          xor     eax, eax                      ;  Default case of switch 004056BF  
  00405769  |.  5A            pop     edx  
  0040576A  |.  59            pop     ecx  
  0040576B  |.  59            pop     ecx  
  0040576C  |.  64:8910       mov     dword ptr fs:[eax], edx  
  0040576F  |.  68 8C574000   push    0040578C  
  00405774  |>  8D85 F4FEFFFF lea     eax, dword ptr [ebp-10C]  
  0040577A  |.  BA 02000000   mov     edx, 2  
  0040577F  |.  E8 18E3FFFF   call    00403A9C  
  00405784  \\.  C3            retn  
```

跟进 `00407915` 的 `call 004058AC`

```
  004058DA  |.  64:FF30        push    dword ptr fs:[eax]  
  004058DD  |.  64:8920        mov     dword ptr fs:[eax], esp  
  004058E0  |.  8D45 FC         lea     eax, dword ptr [ebp-4]  
```
