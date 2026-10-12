
### 1. License Key 第一段检查

------------------------------------------------------------------------

Index...................111111111111

Index...0123456789ABCDEF0123456789AB

`LK[]` = "XXXXXXX-XXXXXX-XXXXXX-XXXXXX"

`Data[]` = 55429C64: 57 69 6E 33 32 00 00 00 6C 65 00 00 5B 20 2D 2D -- Win32...le..\[ --

`ebp` = `LK[]`

- `LK[0]` = 'X' / 'W' / 'x'
- `LK[1]` >= '6' (If equal to '6', specific condition applies)
- `LK[2]` >= '0'

------------------------------------------------------------------------

```assembly
55389576     8A45 00           mov al,byte ptr ss:[ebp]              ; al = LK[0]
55389579     3C 58             cmp al,58                             ; Is 'X'?
5538957B     74 2E             je short pdflib.553895AB              ; Yes,go on...
5538957D     8A0D 649C4255     mov cl,byte ptr ds:[55429C64]         ; cl = Data[0] = 'W'
55389583     3AC1              cmp al,cl                             ; Is 'W'?
55389585     74 24             je short pdflib.553895AB              ; Yes,go on...
55389587     3C 78             cmp al,78                             ; Is 'x'?
55389589     75 0A             jnz short pdflib.55389595            ; No,error
5538958B     80F9 5A           cmp cl,5A                             ; Is 'Z'?
5538958E     74 05             je short pdflib.55389595              ; Yes,error
55389590     80F9 49           cmp cl,49                             ; Is 'I'?
55389593     75 16             jnz short pdflib.553895AB            ; Yes,go on...

Error:
55389595     6A 00             push 0
55389597     6A 00             push 0
55389599     6A 00             push 0
5538959B     6A 00             push 0
5538959D     68 C0070000       push 7C0
553895A2     53                push ebx
553895A3     E8 58DFF7FF       call pdflib.55307500
553895A8     83C4 18           add esp,18

Go on:
553895AB     0FBE45 01         movsx eax,byte ptr ss:[ebp+1]         ; eax = LK[1]
553895AF     8B4C24 40         mov ecx,dword ptr ss:[esp+40]         ; ecx = 0x00000006
553895B3     83C0 D0           add eax,-30                           ; eax = eax - 0x30
553895B6     3BC1              cmp eax,ecx                           ; Is 0x00000006 ?
553895B8     7C 11             jl short pdflib.553895CB              ; Less than, error
553895BA     75 25             jnz short pdflib.553895E1             ; Greater, go on...
553895BC     0FBE45 02         movsx eax,byte ptr ss:[ebp+2]         ; Equal, eax = LK[2]
553895C0     8B4C24 14         mov ecx,dword ptr ss:[esp+14]         ; ecx = 0x00000000
553895C4     83E8 30           sub eax,30                            ; eax = eax - 0x30
553895C7     3BC1              cmp eax,ecx                           ; Is 0x00000000 ?
553895C9     7D 16             jge short pdflib.553895E1             ; Greater or equal, go on...

Error:
553895CB     6A 00             push 0
553895CD     6A 00             push 0
553895CF     6A 00             push 0
553895D1     6A 00             push 0
553895D3     68 C2070000       push 7C2
553895D8     53                push ebx
553895D9     E8 22DFF7FF       call pdflib.55307500
553895DE     83C4 18           add esp,18
```

### 2. License Key 第二段检查

-------------------------------------

Index...................111111111111

Index...0123456789ABCDEF0123456789AB

`LK[]` = "XXXXXXX-XXXXXX-XXXXXX-XXXXXX"

`ebp` = `LK[]`

- `LK[D] + LK[C] * 10 - 0x210` <= `0x2A` (`'*'`)
- `LK[9] + LK[8] * 10 - 0x210` <= `0x40` (`'@'`)
- `LK[B] + LK[A] * 10 - 0x210` <= `0x63` (`'c'`)

-------------------------------------

```assembly
553895E1     0FBE45 0C         movsx eax,byte ptr ss:[ebp+C]         ; eax = LK[C]
553895E5     0FBE55 0D         movsx edx,byte ptr ss:[ebp+D]         ; edx = LK[D]
553895E9     8D0C80            lea ecx,dword ptr ds:[eax+eax*4]      ; ecx = LK[C] * 5
553895EC     0FBE45 08         movsx eax,byte ptr ss:[ebp+8]         ; eax = LK[8]
553895F0     8DBC4A F0FDFFFF    lea edi,dword ptr ds:[edx+ecx*2-210]    ; edi = LK[D] + (LK[C] * 5) * 2 - 0x210
553895F7     0FBE4D 09         movsx ecx,byte ptr ss:[ebp+9]         ; ecx = LK[9]
553895FB     8D0480            lea eax,dword ptr ds:[eax+eax*4]      ; eax = LK[8] * 5
553895FE     83FF 2A           cmp edi,2A                            ; Is '*'?
55389601     8D9C41 F0FDFFFF   lea ebx,dword ptr ds:[ecx+eax*2-210]  ; ebx = LK[9]+(LK[8]*5)*2 - 0x210
55389608     7F 1C             jg short pdflib.55389626              ; Greater, error
5538960A     83FB 40           cmp ebx,40                            ; Is '@'?
5538960D     7F 17             jg short pdflib.55389626              ; Greater, error
5538960F     0FBE45 0A         movsx eax,byte ptr ss:[ebp+A]        ; eax = LK[A]
55389613     8D1480            lea edx,dword ptr ds:[eax+eax*4]      ; edx = LK[A]*5
55389616     0FBE45 0B         movsx eax,byte ptr ss:[ebp+B]        ; eax = LK[B]
5538961A     8D8C50 F0FDFFFF    lea ecx,dword ptr ds:[eax+edx*2-210]  ; ecx = LK[B]+(LK[A]*5)*2 - 0x210
55389621     83F9 63           cmp ecx,63                            ; Is 'c'?
55389624     7E 1C             jle short pdflib.55389642             ; Less or equal, go on...

Error:
55389626     8B7424 38         mov esi,dword ptr ss:[esp+38]
5538962A     6A 00             push 0
5538962C     6A 00             push 0
5538962E     6A 00             push 0
55389630     6A 00             push 0
55389632     68 C2070000       push 7C2
55389637     56                push esi
55389638     E8 C3DEF7FF       call pdflib.55307500
5538963D     83C4 18           add esp,18
55389640     EB 04             jmp short pdflib.55389646

Stack:
0012F57C  50 00 51 00 52 00 53 00 54 00 55 00 56 00 57 00  P.Q.R.S.T.U.V.W. (UNICODE)
0012F58C  58 00 59 00 5A 00 7B 00 7C 00 7D 00              X.Y.Z.{.|.}.      (UNICODE)

Total 28 bytes

Go on:
55389642     8B7424 38         mov esi,dword ptr ss:[esp+38]          ; esi = 003C3E78
55389646     8D5424 18         lea edx,dword ptr ss:[esp+18]          ; edx = 0012F57C
5538964A     52                push edx                                ; edx pushed to stack

Stack:
0012F560   0012F57C          UNICODE "PQRSTUVWXYZ{|}"                  ; edx = UNICODE = 28 bytes
0012F564   003C8E98          ASCII "W600105-009100-701111-C6EA46"     ; "license key" = 28 bytes
0012F568   003C3B88
0012F56C   00000000
0012F570   003C3B40
0012F574   29C6EA46
0012F578   00000000
0012F57C   00510050
0012F580   00530052
0012F584   00550054
0012F588   00570056
0012F58C   00590058
0012F590   007B005A
5538964B     E8 40EFF7FF       call pdflib.55308590
```

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```assembly
55308590     83EC 28           sub esp,28
55308593     8D4424 00         lea eax,dword ptr ss:[esp]              ; eax = UNICODE
55308597     56                push esi                                ; esi = 003C3E78
55308598     57                push edi                                ; edi = 0
55308599     50                push eax                                ; eax = UNICODE
5530859A     E8 8B010700       call pdflib.5537872A
5530859F     8D4C24 0C         lea ecx,dword ptr ss:[esp+C]
553085A3     51                push ecx
553085A4     E8 21000700       call pdflib.553785CA
553085A9     B9 09000000       mov ecx,9
553085AE     8BF0              mov esi,eax
553085B0     8B4424 3C         mov eax,dword ptr ss:[esp+3C]
553085B4     8D7C24 14         lea edi,dword ptr ss:[esp+14]
553085B8     F3:A5             rep movs dword ptr es:[edi],dword ptr d>
553085BA     8B5424 14         mov edx,dword ptr ss:[esp+14]
553085BE     8B4C24 18         mov ecx,dword ptr ss:[esp+18]
553085C2     8910              mov dword ptr ds:[eax],edx
```
