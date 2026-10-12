
## 跟进算法 call（1）

```
0040910A          8B0D D44A4A00  mov ecx,dword ptr ds:[4A4AD4]      ; ScTc.004A5AE0

00409110          8B01           mov eax,dword ptr ds:[ecx]

00409112          E8 516B0600    call ScTc.0046FC68

00409117          8D45 E4        lea eax,dword ptr ss:[ebp-1C]

0040911A          E8 1D8EFFFF    call ScTc.00401F3C

0040911F          50             push eax

00409120          68 C2DF4900    push ScTc.0049DFC2

00409125          8B45 B8        mov eax,dword ptr ss:[ebp-48]

00409128          E8 9B810700    call ScTc.004812C8

0040912D          50             push eax

0040912E          E8 FB250900    call <jmp.&USER32.MessageBoxA>    ;            注册成功提示处
```

```
00401E40          55             push ebp

00401E41          8BEC           mov ebp,esp

00401E43          83C4 D0        add esp,-30

00401E46          53             push ebx

00401E47          8955 FC        mov dword ptr ss:[ebp-4],edx     ; edx=试练码

00401E4A          8945 D4        mov dword ptr ss:[ebp-2C],eax

00401E4D          B8 10C54900    mov eax,ScTc.0049C510

00401E52          E8 DD8B0900    call ScTc.0049AA34

00401E57          C745 F4 01000> mov dword ptr ss:[ebp-C],1       ; 1=ebp-c

00401E5E          8D55 FC        lea edx,dword ptr ss:[ebp-4]     ; edx=试练码

00401E61          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401E64          E8 D38C0900    call ScTc.0049AB3C

00401E69          FF45 F4        inc dword ptr ss:[ebp-C]         ; 1+1

00401E6C          66:C745 E8 08> mov word ptr ss:[ebp-18],8       ; 8=ebp-18

00401E72          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401E75          E8 9E000000    call ScTc.00401F18

00401E7A          83F8 0C        cmp eax,0C

00401E7D          75 76          jnz short ScTc.00401EF5

00401E7F          33D2           xor edx,edx                     ; edx 清零

00401E81          8955 D0        mov dword ptr ss:[ebp-30],edx    ; 0=ebp-30

00401E84          8D45 FC        lea eax,dword ptr ss:[ebp-4]     ; ebp-4=OC

00401E87          E8 B0000000    call ScTc.00401F3C

00401E8C          8B55 D0        mov edx,dword ptr ss:[ebp-30]    ; eax=试练码

00401E8F          8A0C10         mov cl,byte ptr ds:[eax+edx]     ; cl=E4

00401E92          51             push ecx                        ; 12F032

00401E93          E8 3CFFFFFF    call ScTc.00401DD4             ; 算法分析 call（2）

00401E98          59             pop ecx

00401E99          8BD8           mov ebx,eax                     ; 由 call（2）得出，eax=36

00401E9B          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401E9E          E8 99000000    call ScTc.00401F3C

00401EA3          8B55 D0        mov edx,dword ptr ss:[ebp-30]    ; eax=试练码

00401EA6          3A5C10 06      cmp bl,byte ptr ds:[eax+edx+6]   ; 试练码第 7 位与“6”比较，相等则注册成功

00401EAA          74 20          je short ScTc.00401ECC

00401EAC          33C0           xor eax,eax

00401EAE          50             push eax

00401EAF          FF4D F4        dec dword ptr ss:[ebp-C]

00401EB2          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401EB5          BA 02000000    mov edx,2

00401EBA          E8 058D0900    call ScTc.0049ABC4

00401EBF          58             pop eax

00401EC0          8B55 D8        mov edx,dword ptr ss:[ebp-28]

00401EC3          64:8915 00000> mov dword ptr fs:[0],edx

00401ECA          EB 47          jmp short ScTc.00401F13

00401ECC          B0 01          mov al,1                       ; al 置 1

00401ECE          50             push eax

00401ECF          FF4D F4        dec dword ptr ss:[ebp-C]

00401ED2          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401ED5          BA 02000000    mov edx,2

00401EDA          E8 E58C0900    call ScTc.0049ABC4

00401EDF          58             pop eax

00401EE0          8B55 D8        mov edx,dword ptr ss:[ebp-28]    ; 试练码送 edx

00401EE3          64:8915 00000> mov dword ptr fs:[0],edx

00401EEA          EB 27          jmp short ScTc.00401F13

00401EEC          FF45 D0        inc dword ptr ss:[ebp-30]

00401EEF          837D D0 06     cmp dword ptr ss:[ebp-30],6

00401EF3       ^  7C 8F          jl short ScTc.00401E84

00401EF5          33C0           xor eax,eax

00401EF7          50             push eax

00401EF8          FF4D F4        dec dword ptr ss:[ebp-C]

00401EFB          8D45 FC        lea eax,dword ptr ss:[ebp-4]

00401EFE          BA 02000000    mov edx,2

00401F03          E8 BC8C0900    call ScTc.0049ABC4

00401F08          58             pop eax

00401F09          8B55 D8        mov edx,dword ptr ss:[ebp-28]

00401F0C          64:8915 00000> mov dword ptr fs:[0],edx

00401F13          5B             pop ebx

00401F14          8BE5           mov esp,ebp

00401F16          5D             pop ebp

00401F17          C3             retn
```

## 跟进算法 call（2）

```
00401DD4          55             push ebp

00401DD5          8BEC           mov ebp,esp

00401DD7          0FBE45 08      movsx eax,byte ptr ss:[ebp+8]    ; ebp+8=1，eax=试练码

00401DDB          83C0 D0        add eax,-30                     ; 31 与-30 进行 add 运算

00401DDE          83F8 09        cmp eax,9                        eax=1，与 9 比较

00401DE1          77 57          ja short ScTc.00401E3A           错误判断处，即注册码第一位只能为数字

00401DE3          FF2485 EA1D40> jmp dword ptr ds:[eax*4+401DEA] 正常跳转，我们的试练码第一位为 1，则跳转到 00401E16 处

00401DEA          121E           adc bl,byte ptr ds:[esi]

00401DEC          40             inc eax

00401DED          0016           add byte ptr ds:[esi],dl

00401DEF          1E             push ds

00401DF0          40             inc eax

00401DF1          001A           add byte ptr ds:[edx],bl

00401DF3          1E             push ds

00401DF4          40             inc eax

00401DF5          001E           add byte ptr ds:[esi],bl

00401DF7          1E             push ds

00401DF8          40             inc eax

00401DF9          0022           add byte ptr ds:[edx],ah

00401DFB          1E             push ds

00401DFC          40             inc eax

00401DFD          0026           add byte ptr ds:[esi],ah

00401DFF          1E             push ds

00401E00          40             inc eax

00401E01          002A           add byte ptr ds:[edx],ch

00401E03          1E             push ds

00401E04          40             inc eax

00401E05          002E           add byte ptr ds:[esi],ch

00401E07          1E             push ds

00401E08          40             inc eax

00401E09          0032           add byte ptr ds:[edx],dh

00401E0B          1E             push ds

00401E0C          40             inc eax

00401E0D          0036           add byte ptr ds:[esi],dh

00401E0F          1E             push ds

00401E10          40             inc eax

00401E12          B0 38          mov al,38                        试练码第一位若为 0 的跳转处

00401E14          5D             pop ebp

00401E15          C3             retn

00401E16          B0 36          mov al,36                        ；al=1       试练码第一位若为 1 的跳转处，我们的试练码运行后跳转到该处

00401E18          5D             pop ebp                        ；eax=36

00401E19          C3             retn

00401E1A          B0 34          mov al,34                        试练码第一位若为 2 的跳转处

00401E1C          5D             pop ebp

00401E1D          C3             retn

00401E1E          B0 30          mov al,30                        试练码第一位若为 3 的跳转处

00401E20          5D             pop ebp

00401E21          C3             retn
```
