
```
0045B646    FF15 2C024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaObjSet>]           ; MSVBVM50.\_\_vbaObjSet

0045B64C    8B45 C8              mov eax,dword ptr ss:[ebp-38]

0045B64F    8D55 A4              lea edx,dword ptr ss:[ebp-5C]

0045B652    8945 BC              mov dword ptr ss:[ebp-44],eax

0045B655    52                   push edx

0045B656    8D45 B4              lea eax,dword ptr ss:[ebp-4C]

0045B659    56                   push esi

0045B65A    8D4D 94              lea ecx,dword ptr ss:[ebp-6C]

0045B65D    50                   push eax

0045B65E    51                   push ecx

0045B65F    C745 AC 01000000     mov dword ptr ss:[ebp-54],1

0045B666    C745 A4 02000000     mov dword ptr ss:[ebp-5C],2

0045B66D    C745 C8 00000000     mov dword ptr ss:[ebp-38],0

0045B674    C745 B4 09000000     mov dword ptr ss:[ebp-4C],9

0045B67B    FF15 68024600        call dword ptr ds:[<&MSVBVM50.#632>]                  ; MSVBVM50.rtcMidCharVar

0045B681    8D55 94              lea edx,dword ptr ss:[ebp-6C]

0045B684    8D45 E0              lea eax,dword ptr ss:[ebp-20]

0045B687    52                   push edx

0045B688    50                   push eax

0045B689    FF15 CC024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaStrVarVal>]        ; MSVBVM50.\_\_vbaStrVarVal

0045B68F    50                   push eax

0045B690    FF15 F8014600        call dword ptr ds:[<&MSVBVM50.#516>]                  ; MSVBVM50.rtcAnsiValueBstr

0045B696    0FBFC8               movsx ecx,ax

0045B699    03CF                 add ecx,edi                                           ; // 用户名ASCII码累加和到ecx

0045B69B    0F80 850A0000        jo supernet.0045C126

0045B6A1    8BF9                 mov edi,ecx

0045B6A3    8D4D E0              lea ecx,dword ptr ss:[ebp-20]

0045B6A6    FF15 5C034600        call dword ptr ds:[<&MSVBVM50.\_\_vbaFreeStr>]          ; MSVBVM50.\_\_vbaFreeStr

0045B6AC    8D4D C8              lea ecx,dword ptr ss:[ebp-38]

0045B6AF    FF15 58034600        call dword ptr ds:[<&MSVBVM50.\_\_vbaFreeObj>]          ; MSVBVM50.\_\_vbaFreeObj

0045B6B5    8D55 94              lea edx,dword ptr ss:[ebp-6C]

0045B6B8    8D45 A4              lea eax,dword ptr ss:[ebp-5C]

0045B6BB    52                   push edx

0045B6BC    8D4D B4              lea ecx,dword ptr ss:[ebp-4C]

0045B6BF    50                   push eax

0045B6C0    51                   push ecx

0045B6C1    6A 03                push 3

0045B6C3    FF15 E8014600        call dword ptr ds:[<&MSVBVM50.\_\_vbaFreeVarList>]      ; MSVBVM50.\_\_vbaFreeVarList

0045B6C9    B8 01000000          mov eax,1

0045B6CE    83C4 10              add esp,10

0045B6D1    03C6                 add eax,esi

0045B6D3    0F80 4D0A0000        jo supernet.0045C126

0045B6D9    8BF0                 mov esi,eax

0045B6DB  ^ E9 4EFFFFFF          jmp supernet.0045B62E

0045B6E0    A1 10E04500          mov eax,dword ptr ds:[45E010]

0045B6E5    85C0                 test eax,eax

0045B6E7    75 19                jnz short supernet.0045B702

0045B6E9    8B1D E0024600        mov ebx,dword ptr ds:[<&MSVBVM50.\_\_vbaNew2>]          ; MSVBVM50.\_\_vbaNew2

0045B6EF    68 10E04500          push supernet.0045E010

0045B6F4    68 1C634000          push supernet.0040631C

0045B6F9    FFD3                 call ebx

0045B6FB    A1 10E04500          mov eax,dword ptr ds:[45E010]

0045B700    EB 06                jmp short supernet.0045B708

0045B702    8B1D E0024600        mov ebx,dword ptr ds:[<&MSVBVM50.\_\_vbaNew2>]          ; MSVBVM50.\_\_vbaNew2

0045B708    85C0                 test eax,eax

0045B70A    8985 2CFFFFFF        mov dword ptr ss:[ebp-D4],eax

0045B710    75 11                jnz short supernet.0045B723

0045B712    68 10E04500          push supernet.0045E010

0045B717    68 1C634000          push supernet.0040631C

0045B71C    FFD3                 call ebx

0045B71E    A1 10E04500          mov eax,dword ptr ds:[45E010]

0045B723    8B10                 mov edx,dword ptr ds:[eax]

0045B725    50                   push eax

0045B726    FF92 00030000        call dword ptr ds:[edx+300]

0045B72C    50                   push eax

0045B72D    8D45 C8              lea eax,dword ptr ss:[ebp-38]

0045B730    50                   push eax

0045B731    FF15 2C024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaObjSet>]           ; MSVBVM50.\_\_vbaObjSet

0045B737    8BF0                 mov esi,eax

0045B739    8D55 E0              lea edx,dword ptr ss:[ebp-20]

0045B73C    52                   push edx

0045B73D    56                   push esi

0045B73E    8B0E                 mov ecx,dword ptr ds:[esi]

0045B740    FF91 A0000000        call dword ptr ds:[ecx+A0]

0045B746    85C0                 test eax,eax

0045B748    7D 12                jge short supernet.0045B75C

0045B74A    68 A0000000          push 0A0

0045B74F    68 10894000          push supernet.00408910

0045B754    56                   push esi

0045B755    50                   push eax

0045B756    FF15 08024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaHresultCheckObj>]  ; MSVBVM50.\_\_vbaHresultCheckObj

0045B75C    A1 10E04500          mov eax,dword ptr ds:[45E010]

0045B761    85C0                 test eax,eax

0045B763    75 11                jnz short supernet.0045B776

0045B765    68 10E04500          push supernet.0045E010

0045B76A    68 1C634000          push supernet.0040631C

0045B76F    FFD3                 call ebx

0045B771    A1 10E04500          mov eax,dword ptr ds:[45E010]

0045B776    8B08                 mov ecx,dword ptr ds:[eax]

0045B778    50                   push eax

0045B779    FF91 00030000        call dword ptr ds:[ecx+300]

0045B77F    8D55 C4              lea edx,dword ptr ss:[ebp-3C]

0045B782    50                   push eax

0045B783    52                   push edx

省略N行 代码

0045B784    FF15 2C024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaObjSet>]           ; MSVBVM50.\_\_vbaObjSet

0045B78A    8BF0                 mov esi,eax

0045B78C    8D4D DC              lea ecx,dword ptr ss:[ebp-24]

0045B78F    51                   push ecx

0045B790    56                   push esi

0045B791    8B06                 mov eax,dword ptr ds:[esi]

0045B793    FF90 A0000000        call dword ptr ds:[eax+A0]

0045B799    85C0                 test eax,eax

0045B79B    7D 12                jge short supernet.0045B7AF

0045B79D    68 A0000000          push 0A0

0045B7A2    68 10894000          push supernet.00408910

0045B7A7    56                   push esi

0045B7A8    50                   push eax

0045B7A9    FF15 08024600        call dword ptr ds:[<&MSVBVM50.\_\_vbaHresultCheckObj>]  ; MSVBVM50.\_\_vbaHresultCheckObj

0045B7AF    8B95 2CFFFFFF        mov edx,dword ptr ss:[ebp-D4]

0045B7B5    8B45 DC              mov eax,dword ptr ss:[ebp-24]

0045B7B8    50                   push eax

0045B7B9    8B1A                 mov ebx,dword ptr ds:[edx]

0045B7BB    FF15 60034600        call dword ptr ds:[<&MSVBVM50.#581>]                  ; MSVBVM50.rtcR8ValFromBstr

0045B7C1    FF15 24034600        call dword ptr ds:[<&MSVBVM50.\_\_vbaFpI4>]             ; MSVBVM50.\_\_vbaFpI4

0045B7C7    99                   cdq

0045B7C8    B9 E8030000          mov ecx,3E8                                           ; // A%=3E8 记为B

0045B7CD    F7F9                 idiv ecx

0045B7CF    8BF2                 mov esi,edx

0045B7D1    8B55 E0              mov edx,dword ptr ss:[ebp-20]

0045B7D4    52                   push edx

0045B7D5    FF15 60034600        call dword ptr ds:[<&MSVBVM50.#581>]                  ; MSVBVM50.rtcR8ValFromBstr

0045B7DB    FF15 24034600        call dword ptr ds:[<&MSVBVM50.\_\_vbaFpI4>]             ; MSVBVM50.\_\_vbaFpI4

0045B7E1    99                   cdq
```
