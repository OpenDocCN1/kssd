
```asm
00401F95  |.  888C24 780100>mov     byte ptr [esp+178], cl  

00401F9C  |.  C68424 790100>mov     byte ptr [esp+179], 6  

00401FA4  |.  C68424 7A0100>mov     byte ptr [esp+17A], 4  

00401FAC  |.  C68424 7B0100>mov     byte ptr [esp+17B], 5  

00401FB4  |.  888C24 7C0100>mov     byte ptr [esp+17C], cl  

00401FBB  |.  889C24 7D0100>mov     byte ptr [esp+17D], bl  

00401FC2  |.  888424 7E0100>mov     byte ptr [esp+17E], al  

00401FC9  |.  C68424 7F0100>mov     byte ptr [esp+17F], 6  

00401FD1  |.  889424 800100>mov     byte ptr [esp+180], dl  

00401FD8  |.  888424 810100>mov     byte ptr [esp+181], al  

00401FDF  |.  889C24 820100>mov     byte ptr [esp+182], bl  

00401FE6  |.  888C24 830100>mov     byte ptr [esp+183], cl  

00401FED  |.  889424 840100>mov     byte ptr [esp+184], dl  

00401FF4  |.  888424 850100>mov     byte ptr [esp+185], al  

00401FFB  |.  C68424 860100>mov     byte ptr [esp+186], 1  

00402003  |.  889424 870100>mov     byte ptr [esp+187], dl  

0040200A  |.  33C0          xor     eax, eax  

0040200C  |.  85ED          test    ebp, ebp  

0040200E  |.  889424 880100>mov     byte ptr [esp+188], dl  

00402015  |.  C68424 890100>mov     byte ptr [esp+189], 1  

0040201D  |.  888C24 8A0100>mov     byte ptr [esp+18A], cl  

00402024  |.  889C24 8B0100>mov     byte ptr [esp+18B], bl  

0040202B  |.  898424 8C0100>mov     dword ptr [esp+18C], eax  

00402032  |.  898424 900100>mov     dword ptr [esp+190], eax  
```

### 解密数据

```
0012FDB4  09 01 08 09 00 02 07 09 02 08 04 05 02 07 09 06   .....
0012FDC4  08 09 07 02 08 09 01 08 09 01 02 07 09 01 08 09   .....
0012FDD4  02 02 05 09 02 06 04 05 02 07 09 06 08 09 07 02   ...
0012FDE4  08 09 01 08 08 01 02 07 00 00 00 00 00 00 00 00   .........
```

### 解密过程

```asm
00402039  |.  76 65         jbe     short 004020A0  
0040203B  |.  8D45 FF       lea     eax, dword ptr [ebp-1]           ; 获取 文件长度
0040203E  |.  C1E8 06       shr     eax, 6                           ; 64 字节块个数
00402041  |.  83C0 01       add     eax, 1  
00402044  |.  8BDF          mov     ebx, edi  
00402046  |.  894424 18     mov     dword ptr [esp+18], eax  
0040204A  |.  8D9B 00000000 lea     ebx, dword ptr [ebx]             ;  解密 RSWBSW.dat\FGHIJKLMNOP
00402050  |>  8D8424 540100>lea     eax, dword ptr [esp+154]  
00402057  |.  B9 10000000   |mov     ecx, 10  
0040205C  |.  8BF3          |mov     esi, ebx  
0040205E  |.  8DBC24 940100>|lea     edi, dword ptr [esp+194]  
00402065  |.  50            |push    eax  
00402066  |.  F3:A5         |rep     movs dword ptr es:[edi], dword >  
00402068  |.  E8 93EFFFFF   |call    00401000  
0040206D  |.  8D8C24 980100>|lea     ecx, dword ptr [esp+198]  
00402074  |.  6A 01         |push    1  
00402076  |.  51            |push    ecx  
00402077  |.  8BD1          |mov     edx, ecx  
00402079  |.  52            |push    edx  
0040207A  |.  E8 51F3FFFF   |call    004013D0  
0040207F  |.  8BFB          |mov     edi, ebx  
00402081  |.  83C4 10       |add     esp, 10  
00402084  |.  B9 10000000   |mov     ecx, 10  
00402089  |.  8DB424 940100>|lea     esi, dword ptr [esp+194]  
00402090  |.  83C3 40       |add     ebx, 40  
00402093  |.  836C24 18 01  |sub     dword ptr [esp+18], 1  
00402098  |.  F3:A5         |rep     movs dword ptr es:[edi], dword >;  RSWBSW.dat N 字节解密完成
0040209A  |.^ 75 B4         \|jnz     short 00402050  
0040209C  |.  8B7C24 10     mov     edi, dword ptr [esp+10]         ;  解密完成!
004020A0  |>  85FF          test    edi, edi  
004020A2  |.  0F84 93000000 je      0040213B  
```

### 注
解密完成后,可以将解密后的数据 保存为 `exe` 文件

### 创建进程 `RSWBSW.exe\K0123456789`,并挂起

```asm
00401A30  /$  81EC 70010000 sub     esp, 170  
00401A36  |.  A1 4CC04000   mov     eax, dword ptr [40C04C]  
00401A3B  |.  33C4          xor     eax, esp  
00401A3D  |.  898424 6C0100>mov     dword ptr [esp+16C], eax  
00401A44  |.  8B8424 7C0100>mov     eax, dword ptr [esp+17C]  
00401A4B  |.  53            push    ebx  
00401A4C  |.  8B9C24 7C0100>mov     ebx, dword ptr [esp+17C]  
00401A53  |.  55            push    ebp  
00401A54  |.  8BAC24 7C0100>mov     ebp, dword ptr [esp+17C]  
00401A5B  |.  56            push    esi  
00401A5C  |.  57            push    edi  
00401A5D  |.  6A 40         push    40  
00401A5F  |.  8D4C24 3C     lea     ecx, dword ptr [esp+3C]  
00401A63  |.  6A 00         push    0  
00401A65  |.  51            push    ecx  
00401A66  |.  894424 1C     mov     dword ptr [esp+1C], eax  
00401A6A  |.  C74424 40 000>mov     dword ptr [esp+40], 0  
00401A72  |.  E8 590A0000   call    004024D0  
00401A77  |.  83C4 0C       add     esp, 0C  
00401A7A  |.  68 04010000   push    104                          ; /BufSize = 104 (260.)  
00401A7F  |.  8D5424 7C     lea     edx, dword ptr [esp+7C]      ; |  
00401A83  |.  52            push    edx                           ; |PathBuffer  
00401A84  |.  6A 00         push    0                             ; |hModule = NULL  
00401A86  |.  FF15 10904000 call    dword ptr [<&KERNEL32.GetModuleFileNameA>] ; \GetModuleFileNameA  
00401A8C  |.  8D4424 78     lea     eax, dword ptr [esp+78]  
00401A90  |.  6A 5C         push    5C  
00401A92  |.  50            push    eax  
00401A93  |.  E8 F40B0000   call    0040268C  
00401A98  |.  8DBC24 800000>lea     edi, dword ptr [esp+80]  
00401A9F  |.  83C4 08       add     esp, 8  
00401AA2  |.  C640 01 00    mov     byte ptr [eax+1], 0  
00401AA6  |.  83C7 FF       add     edi, -1  
00401AA9  |.  8DA424 000000>lea     esp, dword ptr [esp]  
00401AB0  |>  8A47 01       /mov     al, byte ptr [edi+1]  
00401AB3  |.  83C7 01       |add     edi, 1  
00401AB6  |.  84C0          |test    al, al  
00401AB8  |.^ 75 F6         \|jnz     short 00401AB0  
00401ABA  |.  B9 05000000   mov     ecx, 5  
00401ABF  |.  BE B0944000   mov     esi, 004094B0                ;  ASCII "RSWBSW.exe\K0123456789"  
00401AC4  |.  F3:A5         rep     movs dword ptr es:[edi], dword ptr [esi]  
00401AC6  |.  66:A5         movs    word ptr es:[edi], word ptr [esi]  
00401AC8  |.  8D4C24 78     lea     ecx, dword ptr [esp+78]  
00401ACC  |.  6A 5C         push    5C  
00401ACE  |.  51            push    ecx  
00401ACF  |.  A4            movs    byte ptr es:[edi], byte ptr [esi]  
00401AD0  |.  E8 B70B0000   call    0040268C  
00401AD5  |.  83C4 08       add     esp, 8  
00401AD8  |.  55            push    ebp                        ; /pProcessInfo  
00401AD9  |.  8D5424 38     lea     edx, dword ptr [esp+38]     ; |  
00401ADD  |.  52            push    edx                          ; |pStartupInfo  
00401ADE  |.  6A 00         push    0                            ; |CurrentDir = NULL  
00401AE0  |.  6A 00         push    0                            ; |pEnvironment = NULL  
00401AE2  |.  6A 04         push    4                            ; |CreationFlags = CREATE_SUSPENDED  
00401AE4  |.  6A 00         push    0                            ; |InheritHandles = FALSE  
00401AE6  |.  6A 00         push    0                            ; |pThreadSecurity = NULL  
00401AE8  |.  C600 00       mov     byte ptr [eax], 0              ; |  
00401AEB  |.  6A 00         push    0                            ; |pProcessSecurity = NULL  
00401AED  |.  8D8424 980000>lea     eax, dword ptr [esp+98]       ; |  
00401AF4  |.  50            push    eax                        ; |CommandLine  
00401AF5  |.  6A 00         push    0                            ; |ModuleFileName = NULL  
```
