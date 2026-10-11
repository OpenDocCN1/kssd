
```
00EF092C    FF55 44               call dword ptr ss:[ebp+44]  

00EF09BF    FF55 3C               call dword ptr ss:[ebp+3C]  ; kernel32.GetProcAddress  

0012FFB0   77E40000  |hModule = 77E40000 (kernel32)  

0012FFB4   00EF08A3  \ProcNameOrOrdinal = "Sleep"  

00EF0C64    8D85 F6040000         lea eax,dword ptr ss:[ebp+4F6] ;00ef1125  

00EF0CA4    64:FF30               push dword ptr fs:[eax]  

00EF16F1    FFD0                  call eax      ;Sleep，在这设 00EEF35F 段点，会进入线程 2  

00EF80BC    FF55 3C               call dword ptr ss:[ebp+3C]  

00EFB56E    FF10                  call dword ptr ds:[eax]   ;Sleep  

00EFB57E    B9 CB0E0000           mov ecx,0ECB          ;eax=efb659,ecx=0ecb  

00EFB58E    8330 33               xor dword ptr ds:[eax],33  

00EFB591    83C0 01               add eax,1  

00EFB594  ^ E2 F8                 loopd short 00EFB58E  

//解压下面一段代码  

00EFB68E    64:FF35 30000000      push dword ptr fs:[30]  

00EFB6F3    8B40 0C               mov eax,dword ptr ds:[eax+C]  

00EFB72B    8B40 0C               mov eax,dword ptr ds:[eax+C]  

00EFB7A7    8B40 18               mov eax,dword ptr ds:[eax+18]   ; BaseAddress=0  

00EFB7AA   /74 2A                 je short 00EFB7D6  

//下面是线程 1 在循环检测一些软件：OLLYDBG、ODbyDYK、LordPE、ImportREC、PE Tools、WINHEX、EnjoyC32asm  

00EFBD72    6A 00                 push 0  

00EFBD74    E8 08000000           call 00EFBD81  

00EFBD79  4F 4C 4C 59 44 42 47 00  OLLYDBG.  

00EFBD81    E8 00000000           call 00EFBD86  

00EFBD86    58                    pop eax  

00EFBD87    2D 293C0000           sub eax,3C29  

00EFBD8C    8B00                  mov eax,dword ptr ds:[eax]   ; User32.FindWindowA  

00EFC2F4    FFD0                  call eax                ;FindWindowA  

00EFC2F6    85C0                  test eax,eax  

00EFC2F8  ^ 0F85 9BF1FFFF         jnz 00EFB499  

00EFC2FE    6A 00                 push 0  

00EFC300    E8 08000000           call 00EFC30D  

00EFC305  4F 44 62 79 44 59 4B 00  ODbyDYK.  

00EFC318    FF10                  call dword ptr ds:[eax]    ; User32.FindWindowA  

00EFC31A    85C0                  test eax,eax  

00EFC31C  ^ 0F85 77F1FFFF         jnz 00EFB499  

00EFC322    E8 1A000000           call 00EFC341  

00EFC327  5B 20 4C 6F 72 64 50 45  [ LordPE  

00EFC32F  20 44 65 6C 75 78 65 20   Deluxe  

00EFC337  5D 20 62 79 20 79 6F 64  ] by yod  

00EFC33F  61 00                   a.  

00EFC34E    FF10                  call dword ptr ds:[eax]      ; User32.FindWindowA  

00EFC350    85C0                  test eax,eax  

00EFC352  ^ 0F85 41F1FFFF         jnz 00EFB499  

00EFC358    E8 37000000           call 00EFC394  

00EFC35D  A1 EE 20 20 49 6D 70 6F  ☆  Impo  

00EFC365  72 74 52 45 43 20 20 A1  rtREC    

00EFC3A1    FF10                  call dword ptr ds:[eax]       ; User32.FindWindowA  

00EFC3A3    85C0                  test eax,eax  

00EFC3A5  ^ 0F85 EEF0FFFF         jnz 00EFB499  

00EFC3AB    E8 41000000           call 00EFC3F1  

00EFC3B0  50 45 20 54 6F 6F 6C 73  PE Tools  

00EFC3B8  20 76 31 2E 35 20 58 6D   v1.5 Xm  

00EFC3FE    FF10                  call dword ptr ds:[eax]        ; User32.FindWindowA  

00EFC400    85C0                  test eax,eax  

00EFC402  ^ 0F85 91F0FFFF         jnz 00EFB499  

00EFC408    E8 07000000           call 00EFC414  

00EFC40D  57 49 4E 48 45 58 00     WINHEX.  

00EFC414    6A 00                 push 0  

00EFC421    FF10                  call dword ptr ds:[eax]         ; User32.FindWindowA  

00EFC423    85C0                  test eax,eax  

00EFC425  ^ 0F85 6EF0FFFF         jnz 00EFB499  

00EFC42B    E8 12000000           call 00EFC442  

00EFC430  20 2D 20 5B 45 6E 6A 6F   - [Enjo  

00EFC438  79 20 43 33 32 61 73 6D  y C32asm  

00EFC442    6A 00                 push 0  

00EFC44F    FF10                  call dword ptr ds:[eax]       ; User32.FindWindowA  

00EFC451    85C0                  test eax,eax  

00EFC453  ^ 0F85 40F0FFFF         jnz 00EFB499  

00EFC459    E8 10000000           call 00EFC46E  

00EFC45E  66 6C 79 30 44 42 47 20  fly0DBG  

00EFC466  2D 20 5B 43 50 55 5D 00  - [CPU].  

00EFC47B    FF10                  call dword ptr ds:[eax]       ; User32.FindWindowA  

00EFC47D    85C0                  test eax,eax  

00EFC47F  ^ 0F85 14F0FFFF         jnz 00EFB499  

00EFC485    E8 35000000           call 00EFC4BF  

00EFC48A  A1 EE 20 20 49 6D 70 6F  ☆  Impo  

00EFC492  72 74 52 45 43 20 20 A1  rtREC   

00EFC4CC    FF10                  call dword ptr ds:[eax]         ; User32.FindWindowA  

00EFC4CE    85C0                  test eax,eax  

00EFC4D0  ^ 0F85 C3EFFFFF         jnz 00EFB499  

00EFC4D6    E8 32000000           call 00EFC50D  

00EFC4DB  50 45 20 54 6F 6F 6C 73  PE Tools  

00EFC4E3  20 76 31 2E 35 20 62 79   v1.5 by  

00EFC51A    FF10                  call dword ptr ds:[eax]         ; User32.FindWindowA  

00EFC51C    85C0                  test eax,eax  

00EFC51E  ^ 0F85 75EFFFFF         jnz 00EFB499  

00EFC524    B9 CB0E0000           mov ecx,0ECB         ;eax=efb659,ecx=0ecb  

00EFC534    8330 33               xor dword ptr ds:[eax],33  

00EFC537    83C0 01               add eax,1  

00EFC53A  ^ E2 F8                 loopd short 00EFC534  

00EFC53C    C3                    retn                    ;00EFB546  

00EFB546    64:FF35 30000000      push dword ptr fs:[30]  

00EFB54D    58                    pop eax  

00EFB54E    8B40 0C               mov eax,dword ptr ds:[eax+C]  

00EFB551    8B40 0C               mov eax,dword ptr ds:[eax+C]  

00EFB554    8378 20 03            cmp dword ptr ds:[eax+20],3  

00EFB558  ^\0F85 3BFFFFFF         jnz 00EFB499  

00EFB56E    FF10                  call dword ptr ds:[eax]          ; kernel32.Sleep  
```

线程 2：入口 `00EEF35F`

```
00EEF35F    E8 00000000           call 00EEF364  

00EEF40B    8B50 04               mov edx,dword ptr ds:[eax+4]   ;ds:[00EEF18D]=00EE6402  

00EFCA48    66:9C                 pushfw  

00EFCA7C    B9 855A0000           mov ecx,5A85  

00EFCAF7    8D85 F6040000         lea eax,dword ptr ss:[ebp+4F6]  ;地址=00EFCFB8  

00EFCB37    64:FF30               push dword ptr fs:[eax]  

00EFCBD1    CC                    int3  

00EFCBD2    90                    nop  

00EFCBEC    F7F3                  div ebx  

00EFCBEE    74 2A                 je short 00EFCC1A  

00EFD3C8   /0F83 74010000         jnb 00EFD542  

00EFD542    61                    popad  

00EFD543    E8 00000000           call 00EFD548  

00EFD5A2    8130 CCC30000         xor dword ptr ds:[eax],0C3CC  ;eax=efed8d,ecx=5a85  

00EFD5A8    40                    inc eax  

00EFD5A9  ^ E2 F7                 loopd short 00EFD5A2  

00EFD5AB    50                    push eax  

//解压下面一段代码  

00EFEDE9    6A 08                 push 8  

00EFF363    6A 00                 push 0  

00EFF39A    FF7424 08             push dword ptr ss:[esp+8]  

00EFF52C    6A 00                 push 0  

00EFF563    6A 00                 push 0  

00EFF59A    E8 0D000000           call 00EFF5AC  

00EFF59F  43 72 65 61 74 65 54 68  CreateTh  

00EFF5A7  72 65 61 64 00         read.  

00EFF8EF    FF55 44               call dword ptr ss:[ebp+44]  

00EFF954    FF55 3C               call dword ptr ss:[ebp+3C]   ；CreateThread  

00EFFE25    85C0                  test eax,eax  

00F00157    FFD0                  call eax      ；CreateThread  

0106FF7C   00000000  |pSecurity = NULL  

0106FF80   00000000  |StackSize = 0  

0106FF84   00EFEEB3  |ThreadFunction = 00EFEEB3     // 线程 3：入口  
```
