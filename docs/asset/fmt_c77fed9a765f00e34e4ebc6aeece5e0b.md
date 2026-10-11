
```
    0012FF6C   00000024  |hFile = 00000024  
    0012FF70   00421E8D  |Buffer = Try.00421E8D  
    0012FF74   0000318C  |nBytesToWrite = 318C (12684.)  
    0012FF78   00421DC1  |pBytesWritten = Try.00421DC1  
    0012FF7C   00000000  \pOverlapped = NULL  
```

```
    00421D3D    FFB5 8A764000   push    dword ptr ss:[ebp+40768A]  
    00421D43    FF95 50724000   call    dword ptr ss:[ebp+407250]        ; kernel32.CloseHandle  
```

```
    0012FF78   00421D49  /CALL to CloseHandle from Try.00421D43  
    0012FF7C   00000024  \hObject = 00000024  
```

```
    00421D76    61              popad  
    00421D7C    B8 01000000     mov     eax, TRUE                     ; 收工  
    00421D86    C3              ret                                     ; 回家  
    00421D8C    90              nop  
    00421D91    61              popad  
    00421D7C    B8 01000000     mov     eax, FALSE  
    00421D86    C3              ret  
```

```
00420807    83F8 00         cmp     eax, FALSE                      ; 失败?  
0042080F   /0F84 22480000  je      Try.00425037  
```

```
    00425037    50              push    eax  
    00425038    FFB5 2A6C4000   push    dword ptr ss:[ebp+406C2A]  
    0042503E    FF95 30724000   call    dword ptr ss:[ebp+407230]        ; kernel32.ReleaseMutex,释放互斥体  
    00425044    FFB5 2A6C4000   push    dword ptr ss:[ebp+406C2A]  
    0042504A    FF95 50724000   call    dword ptr ss:[ebp+407250]        ; kernel32.CloseHandle,关闭互斥体句柄  
    00425050    58              pop     eax  
    00425068    83F8 00         cmp     eax, 0  
    0042506B    0F85 4C010000  jnz     Try.004251BD  
    ; ..............我没走这条线,不跟了  
```

```
0042085E    8D85 E25C4000   lea     eax, dword ptr ss:[ebp+405CE2]  
00420874    E8 7F060000     call    Try.00420EF8  
```

```
    -----------------------------  
    以后不跟那么多线了,受不了  
```

```
    00420F2A    68 3F000F00     push    0F003F  
    00420F2F    6A 00           push    0  
    00420F31    6A 00           push    0  
    00420F33    FF95 CD724000   call    dword ptr ss:[ebp+4072CD]    ; advapi32.OpenSCManagerA  
    00420F50    83F8 00         cmp     eax, 0  
    00420F58   /0F84 8F030000  je      Try.004212ED  
    00420F75    8985 3A6C4000   mov     dword ptr ss:[ebp+406C3A], eax  
    00420F85    B8 3E6C4000     mov     eax, Try.00406C3E  
    00420FA1    03C5            add     eax, ebp  
    00420FBA    68 FF010F00     push    0F01FF  
    00420FBF    50              push    eax  
    00420FC0    FFB5 3A6C4000   push    dword ptr ss:[ebp+406C3A]  
    00420FC6    FF95 D5724000   call    dword ptr ss:[ebp+4072D5]        ; advapi32.OpenServiceA  
    00420FD1    83F8 00         cmp     eax, 0  
    00420FD4    0F85 97000000  jnz     Try.00421071                     ; Jump  
    0042109E    83F8 00         cmp     eax, 0  
    004210A6   /0F84 41020000  je      Try.004212ED  
    004210B1    8985 586C4000   mov     dword ptr ss:[ebp+406C58], eax  
    004210C1    6A 00           push    0  
    004210C3    6A 00           push    0  
    004210C5    FFB5 586C4000   push    dword ptr ss:[ebp+406C58]  
    004210CB    FF95 C5724000   call    dword ptr ss:[ebp+4072C5]        ; advapi32.StartServiceA  
    ; 启动服务,不好玩^^,我这里服务开启,所以产生了 ERROR_SERVICE_ALREADY_RUNNING (00000420)  
    004210E4    FF95 40724000   call    dword ptr ss:[ebp+407240]        ; ntdll.RtlGetLastWin32Error  
    00421101    3D E5030000     cmp     eax, 3E5  
    00421106   /0F84 DD000000  je      Try.004211E9  
    00421123    3D 20040000     cmp     eax, 420  
    00421128   /0F84 B6000000  je      Try.004211E4                       ; GoGoGo,否则 Game Over 就不好玩了  
    0042120A    B8 446C4000     mov     eax, Try.00406C44  
    00421226    03C5            add     eax, ebp  
    ; Oh God save me!!!  
```

```
    ---------------------------------------------------------------------------  
    00421377  5C 5C 2E 5C 44 62 70 65 44 65 76 69 63 65 30 00  \\.\DbpeDevice0.  
    ---------------------------------------------------------------------------  
```

```
    0042123F    6A 00           push    0  
    00421241    6A 00           push    0  
    00421243    6A 03           push    3  
    00421245    6A 00           push    0  
    00421247    6A 01           push    1  
    00421249    68 000000C0     push    C0000000  
    0042124E    50              push    eax  
    0042124F    FF95 48724000   call    dword ptr ss:[ebp+407248]        ; kernel32.CreateFileA  
```

```
    0012FF7C   0012FFE0  
    0012FF80   00421255  /CALL to CreateFileA from Try.0042124F  
    0012FF84   00421377  |FileName = "\\\\.\\DbpeDevice0"  
    0012FF88   C0000000  |Access = GENERIC_READ|GENERIC_WRITE  
    0012FF8C   00000001  |ShareMode = FILE_SHARE_READ  
    0012FF90   00000000  |pSecurity = NULL  
    0012FF94   00000003  |Mode = OPEN_EXISTING  
```
