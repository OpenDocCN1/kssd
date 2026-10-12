
### 0044BE49 Block

```asm
0044BE49  .|   8945 94        mov     [ebp-6C], eax                ; |
0044BE4C  .|   8D4D DC        lea     ecx, [ebp-24]                ; |
0044BE4F  .|   51             push    ecx                           ; |Arg1
0044BE50  .|   8B4D F0        mov     ecx, [ebp-10]                ; |
0044BE53  .|   E8 68000000    call    0044BEC0                     ; \\算法 call（3），跟进
0044BE58  .|   8945 90        mov     [ebp-70], eax
0044BE5B  .|   8B55 90        mov     edx, [ebp-70]
0044BE5E  .|   8955 8C        mov     [ebp-74], edx
0044BE61  .|   C645 FC 04     mov     byte ptr [ebp-4], 4
0044BE65  .|   8B45 8C        mov     eax, [ebp-74]
0044BE68  .|   50             push    eax
0044BE69  .|   8D4D EC        lea     ecx, [ebp-14]                ; |
0044BE6C  .|   E8 7EF70A00    call    004FB5EF                     ;  真码 1 与真码 2 比较
0044BE71  .|   C645 FC 02     mov     byte ptr [ebp-4], 2
0044BE75  .|   8D4D DC        lea     ecx, [ebp-24]
0044BE78  .|   E8 39F60A00    call    004FB4B6
0044BE7D  .|   8D45 EC        lea     eax, [ebp-14]
0044BE80  .|   50             push    eax
0044BE81  .|   8B4D 08        mov     ecx, [ebp+8]
0044BE84  .|   E8 A2F30A00    call    004FB22B
0044BE89  .|   8B4D D8        mov     ecx, [ebp-28]
0044BE8C  .|   83C9 01        or      ecx, 1
0044BE8F  .|   894D D8        mov     [ebp-28], ecx
0044BE92  .|   C645 FC 01     mov     byte ptr [ebp-4], 1
0044BE96  .|   8D4D EC        lea     ecx, [ebp-14]
0044BE99  .|   E8 18F60A00    call    004FB4B6
0044BE9E  .|   C645 FC 00     mov     byte ptr [ebp-4], 0
0044BEA2  .|   8D4D 0C        lea     ecx, [ebp+C]
0044BEA5  .|   E8 0CF60A00    call    004FB4B6
0044BEAA  .|   8B45 08        mov     eax, [ebp+8]
0044BEAD  |>   8B4D F4        mov     ecx, [ebp-C]
0044BEB0  .|   64:890D 00000> mov     fs:[0], ecx
0044BEB7  .|   5F             pop     edi
0044BEB8  .|   5E             pop     esi
0044BEB9  .|   5B             pop     ebx
0044BEBA  .|   8BE5           mov     esp, ebp
0044BEBC  .|   5D             pop     ebp
0044BEBD  \.   C2 0800        retn    8
```

### 算法 call（2）：

```asm
0044C0E0  /$   55             push    ebp
0044C0E1  .|   8BEC           mov     ebp, esp
0044C0E3  .|   6A FF          push    -1
0044C0E5  .|   68 B2795200    push    005279B2                     ;  SE 处理程序安装
0044C0EA  .|   64:A1 0000000> mov     eax, fs:[0]
0044C0F0  .|   50             push    eax
0044C0F1  .|   64:8925 00000> mov     fs:[0], esp
0044C0F8  .|   83EC 5C        sub     esp, 5C
0044C0FB  .|   53             push    ebx
0044C0FC  .|   56             push    esi
0044C0FD  .|   57             push    edi
0044C0FE  .|   894D F0        mov     [ebp-10], ecx
0044C101  .|   C745 D8 00000> mov     dword ptr [ebp-28], 0
0044C108  .|   C745 FC 01000> mov     dword ptr [ebp-4], 1
0044C10F  .|   8D4D E0        lea     ecx, [ebp-20]
0044C112  .|   E8 1975FCFF    call    00413630
0044C117  .|   C645 FC 02     mov     byte ptr [ebp-4], 2
0044C11B  .|   8D4D DC        lea     ecx, [ebp-24]
0044C11E  .|   E8 0D75FCFF    call    00413630
0044C123  .|   C645 FC 03     mov     byte ptr [ebp-4], 3
0044C127  .|   6A 48          push    48                             ;  H
0044C129  .|   8D4D E0        lea     ecx, [ebp-20]
0044C12C  .|   E8 368F0A00    call    004F5067
0044C131  .|   6A 45          push    45                             ;  E
0044C133  .|   8D4D E0        lea     ecx, [ebp-20]
0044C136  .|   E8 7EF70A00    call    004FB8B9
0044C13B  .|   6A 4E          push    4E                             ;  N
0044C13D  .|   8D4D E0        lea     ecx, [ebp-20]
0044C140  .|   E8 74F70A00    call    004FB8B9
0044C145  .|   6A 52          push    52                             ;  R
0044C147  .|   8D4D E0        lea     ecx, [ebp-20]
0044C14A  .|   E8 6AF70A00    call    004FB8B9
0044C14F  .|   6A 59          push    59                             ;  Y
0044C151  .|   8D4D E0        lea     ecx, [ebp-20]
0044C154  .|   E8 60F70A00    call    004FB8B9
0044C159  .|   6A 53          push    53                             ;  S
0044C15B  .|   8D4D E0        lea     ecx, [ebp-20]
0044C15E  .|   E8 56F70A00    call    004FB8B9
0044C163  .|   6A 43          push    43                             ;  C
0044C165  .|   8D4D E0        lea     ecx, [ebp-20]
0044C168  .|   E8 4CF70A00    call    004FB8B9
0044C16D  .|   6A 4F          push    4F                             ;  O
0044C16F  .|   8D4D E0        lea     ecx, [ebp-20]
0044C172  .|   E8 42F70A00    call    004FB8B9
0044C177  .|   6A 44          push    44                             ;  D
0044C179  .|   8D4D E0        lea     ecx, [ebp-20]
0044C17C  .|   E8 38F70A00    call    004FB8B9
0044C181  .|   6A 45          push    45                             ;  E
0044C183  .|   8D4D E0        lea     ecx, [ebp-20]
0044C186  .|   E8 2EF70A00    call    004FB8B9
0044C18B  .|   6A 50          push    50                             ;  P
0044C18D  .|   8D4D DC        lea     ecx, [ebp-24]
0044C190  .|   E8 D28E0A00    call    004F5067
0044C195  .|   6A 48          push    48                             ;  H
0044C197  .|   8D4D DC        lea     ecx, [ebp-24]
0044C19A  .|   E8 1AF70A00    call    004FB8B9
0044C19F  .|   6A 50          push    50                             ;  P
0044C1A1  .|   8D4D DC        lea     ecx, [ebp-24]
0044C1A4  .|   E8 10F70A00    call    004FB8B9
0044C1A9  .|   6A 31          push    31                             ;  1
0044C1AB  .|   8D4D DC        lea     ecx, [ebp-24]
0044C1AE  .|   E8 06F70A00    call    004FB8B9
0044C1B3  .|   C745 EC 00000> mov     dword ptr [ebp-14], 0
0044C1BA  .|   C745 E8 00000> mov     dword ptr [ebp-18], 0
0044C1C1  .|   C745 E4 00000> mov     dword ptr [ebp-1C], 0
0044C1C8  .|   EB 09          jmp     short 0044C1D3
0044C1CA  |>   8B45 EC        /mov    eax, [ebp-14]
0044C1CD  .|   83C0 01        |add    eax, 1
0044C1D0  .|   8945 EC        |mov    [ebp-14], eax
0044C1D3  |>   8D4D 0C         lea     ecx, [ebp+C]
0044C1D6  .|   E8 0577FCFF    |call    004138E0                    ;  取用户名
0044C1DB  .|   3945 EC        |cmp    [ebp-14], eax                ;  用户名各字符是否取完
0044C1DE  .|   7D 7D          |jge    short 0044C25D               ;  完则跳
0044C1E0  .|   8B45 E4        |mov    eax, [ebp-1C]
0044C1E3  .|   50             |push   eax                           ; /Arg1
0044C1E4  .|   8D4D E0        |lea    ecx, [ebp-20]                ; |
0044C1E7  .|   E8 3477FCFF    |call   00413920                    ; \\取固定字串（HENRYSCODE），记为 A
0044C1EC  .|   0FBEF0         |movsx  esi, al                      ;  固定字串（HENRYSCODE）逐位移至 ESI
0044C1EF  .|   8B4D EC        |mov    ecx, [ebp-14]
0044C1F2  .|   51             |push   ecx                           ; /Arg1
0044C1F3  .|   8D4D 0C        |lea    ecx, [ebp+C]                 ; |
0044C1F6  .|   E8 2577FCFF    |call   00413920                    ; \\取用户名
0044C1FB  .|   0FBED0         |movsx  edx, al                      ;  用户名逐位移至 EDX
0044C1FE  .|   03F2           |add    esi, edx                      ;  A 与用户名 ASCII 值逐位相加，结果保留于 ESI
0044C200  .|   8B45 E8        |mov    eax, [ebp-18]
0044C203  .|   50             |push   eax                           ; /Arg1
0044C204  .|   8D4D DC        |lea    ecx, [ebp-24]                ; |
0044C207  .|   E8 1477FCFF    |call   00413920                    ; \\取固定字串（PHP1），记为 B
0044C20C  .|   0FBEC8         |movsx  ecx, al                      ;  固定字串（PHP1）逐位移至 ECX
```
