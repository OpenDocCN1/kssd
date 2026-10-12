
```
10009550  .|.  50              push eax
10009551  .|.  68 1CC40210     push 1002C41C
10009556  .|.  68 1D680010     push 1000681D
1000955B  .|.  56              push esi
1000955C  .|.  53              push ebx
1000955D  .|.  68 A0F40210     push 1002F4A0
10009562  .|.  E8 DABF0000     call <搜索特征码>
10009567  .|.  83C4 0C         add esp, 0C
1000956A  .|.  50              push eax
1000956B  .|.  56              push esi
1000956C  .|.  53              push ebx
1000956D  .|.  68 8CF40210     push 1002F48C
10009572  .|.  E8 CABF0000     call <搜索特征码>
10009577  .|.  83C4 0C         add esp, 0C
1000957A  .|.  50              push eax
1000957B  .|.  E8 7F7DFFFF     call <计算 hook 点>
10009580  .|.  6A 14          push 14
10009582  .|.  53              push ebx
10009583  .|.  BE 48140310     mov esi, 10031448
10009588  .|.  56              push esi
10009589  .|.  891D 4C170310   mov dword ptr [1003174C], ebx
1000958F  .|.  891D 44170310   mov dword ptr [10031744], ebx
10009595  .|.  891D 24170310   mov dword ptr [10031724], ebx
1000959B  .|.  E8 90870000     call <清理一块内存空间出来>
100095A0  .|.  68 78F40210     push 1002F478
100095A5  .|.  56              push esi
100095A6  .|.  E8 C5A40000     call 10013A70
100095AB  .|.  83C4 28         add esp, 28
100095AE  .|.  53              push ebx                          ; /pThreadId
100095AF  .|.  53              push ebx                          ; |CreationFlags
100095B0  .|.  53              push ebx                          ; |pThreadParm
100095B1  .|.  68 BE910010     push 100091BE                     ; |ThreadFunction = 4bf3db98.100091BE
100095B6  .|.  53              push ebx                          ; |StackSize
100095B7  .|.  53              push ebx                          ; |pSecurity
100095B8  .|.  FF15 94200210   call dword ptr [10022094]         ; \CreateThread 线程 6
100095BE  .|.  50              push eax                          ; /hObject
100095BF  .|.  FF15 90200210   call dword ptr [10022090]         ; \CloseHandle
100095C5  |>  8B4D FC         mov ecx, [local.1]
100095C8  .|.  5F              pop edi
100095C9  .|.  5E              pop esi
100095CA  .|.  33CD            xor ecx, ebp
100095CC  .|.  5B              pop ebx
100095CD  .|.  E8 78A20000     call 1001384A
100095D2  .|.  C9              leave
100095D3  .\\.  C3              retn
```

### 线程 6  判断传奇

```
100091BE          /.  55            push ebp
100091BF          |.  8BEC          mov ebp, esp
100091C1          |.  83EC 10       sub esp, 10
100091C4          |.  56            push esi
100091C5          |.  68 4C3A0210   push 10023A4C             ;  mir.exeyx=ftp&ss=%s
100091CA          |.  BE 50C50210   mov esi, 1002C550         ;  d:\tools\ollyice\loaddll.exe
100091CF          |.  56            push esi
100091D0          |.  E8 6BA50000   call <比较 cmp>
100091D5          |.  59            pop ecx
100091D6          |.  59            pop ecx
100091D7          |.  85C0          test eax, eax
100091D9          |.  74 05         je short 100091E0
100091DB          |.  E8 1BFFFFFF   call 100090FB
100091E0          |>  68 D03A0210   push 10023AD0             ;  wow.exews2_32.dll
100091E5          |.  56            push esi
100091E6          |.  E8 55A50000   call <比较 cmp>
100091EB          |.  59            pop ecx
100091EC          |.  59            pop ecx
100091ED          |.  85C0          test eax, eax
100091EF          |.  74 05         je short 100091F6
100091F1          |.  E8 05FFFFFF   call 100090FB
100091F6          |>  68 403A0210   push 10023A40             ;  mir1.dat
100091FB          |.  56            push esi
100091FC          |.  E8 3FA50000   call <比较 cmp>
10009201          |.  59            pop ecx
10009202          |.  59            pop ecx
10009203          |.  85C0          test eax, eax
10009205          |.  75 37         jnz short 1000923E
10009207          |.  68 343A0210   push 10023A34             ;  mir2.dat
```

### Hook 函数：

```
100012FF <4bf3db9>/$   55            push ebp
10001300              |.  8BEC        mov ebp, esp
10001302              |.  51          push ecx
10001303              |.  51          push ecx
10001304              |.  57          push edi
10001305              |.  8B7D 08     mov edi, [arg.1]
10001308              |.  85FF        test edi, edi
1000130A              |.  75 07       jnz short 10001313
1000130C              |.  32C0        xor al, al
1000130E              |.  E9 E2000000 jmp 100013F5
......
10001338              |.  51          push ecx                        ; /pOldProtect
10001339              |.  6A 40       push 40                         ; |NewProtect = PAGE_EXECUTE_READWRITE
1000133B              |.  6A 0B       push 0B                         ; |Size = B (11.)
1000133D              |.  897E 12     mov dword ptr [esi+12], edi     ; |
10001340              |.  8B3D 30200210 mov edi, dword ptr [10022030] ; |kernel32.GetCurrentProcess
10001346              |.  50          push eax                        ; |Address
10001347              |.  8946 23     mov dword ptr [esi+23], eax     ; |
1000134A              |.  C646 0C 00  mov byte ptr [esi+C], 0         ; |
1000134E              |.  FFD7        call edi                        ; |[GetCurrentProcess
10001350              |.  8B1D 2C200210 mov ebx, dword ptr [1002202C] ; |kernel32.VirtualProtectEx
10001356              |.  50          push eax                        ; |hProcess
10001357              |.  FFD3        call ebx                        ; \VirtualProtectEx  去保护
10001359              |.  6A 0B       push 0B
1000135B              |.  FF75 0C     push [arg.2]
1000135E              |.  56          push esi
1000135F              |.  E8 4C0A0100 call 10011DB0
10001364              |.  6A 0B       push 0B
10001366              |.  68 90000000 push 90
1000136B              |.  FF75 0C     push [arg.2]
1000136E              |.  E8 BD090100 call <清理一块内存空间出来>
10001373              |.  0FB645 18   movzx eax, byte ptr [ebp+18]
10001377              |.  50          push eax
10001378              |.  FF75 08     push [arg.1]
1000137B              |.  8945 F8     mov [local.2], eax
1000137E              |.  FF75 0C     push [arg.2]
10001381              |.  E8 2A0A0100 call 10011DB0
10001386              |.  8B4D 0C     mov ecx, [arg.2]
10001389              |.  8B55 08     mov edx, [arg.1]
1000138C              |.  8B45 F8     mov eax, [local.2]
1000138F              |.  2BD1        sub edx, ecx      两个地址相减
10001391              |.  83C4 24     add esp, 24
10001394              |.  83EA 05     sub edx, 5        ;  计算 hook 跳转的值
10001397              |.  C60408 E9   mov byte ptr [eax+ecx], 0E9
1000139B              |.  895408 01   mov dword ptr [eax+ecx+1], edx
1000139F              |.  8D45 FC     lea eax, [local.1]
100013A2              |.  50          push eax
100013A3              |.  FF75 FC     push [local.1]
100013A6              |.  6A 0B       push 0B
100013A8              |.  51          push ecx
100013A9              |.  FFD7        call edi
```

Hook 了游戏后，从游戏模块 `jmp` 过来
