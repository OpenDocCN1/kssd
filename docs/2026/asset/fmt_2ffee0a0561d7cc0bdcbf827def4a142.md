
```
7120537C    8B45 B4             mov eax,dword ptr ss:[ebp-4C]

//堆栈 [ebp-4C]=[0013FF38]=004066EE

//特殊函数ThunkValue+基址

7120537F    50                  push eax

71205380    6A 00               push 0

71205382    A1 E0E02171         mov eax,dword ptr ds:[7121E0E0]

71205387    8B00                mov eax,dword ptr ds:[eax]

71205389    8B55 E8             mov edx,dword ptr ss:[ebp-18]

7120538C    8B0490              mov eax,dword ptr ds:[eax+edx*4]

7120538F    8BCF                mov ecx,edi

71205391    8B55 FC             mov edx,dword ptr ss:[ebp-4]

71205394    E8 13D8FFFF         call 71202BAC

//jmp 71232AE4	★7

71205399    EB 27               jmp short 712053C2

7120539B    25 FFFF0000         and eax,0FFFF

712053A0    50                  push eax

712053A1    6A 01               push 1

712053A3    A1 E0E02171         mov eax,dword ptr ds:[7121E0E0]

712053A8    8B00                mov eax,dword ptr ds:[eax]

712053AA    8B55 E8             mov edx,dword ptr ss:[ebp-18]

712053AD    8B0490              mov eax,dword ptr ds:[eax+edx*4]

712053B0    8BCF                mov ecx,edi

712053B2    8B55 FC             mov edx,dword ptr ss:[ebp-4]

712053B5    E8 F2D7FFFF         call 71202BAC

712053BA    3BFB                cmp edi,ebx

712053BC    74 04               je short 712053C2

712053BE    33C0                xor eax,eax

712053C0    8903                mov dword ptr ds:[ebx],eax

712053C2    83C3 04             add ebx,4

712053C5    83C7 04             add edi,4

712053C8    8B03                mov eax,dword ptr ds:[ebx]

712053CA    85C0                test eax,eax

712053CC    0F87 0AFFFFFF       ja 712052DC

//循环处理每个DLL的函数

712053D2    EB 05               jmp short 712053D9

712053D4    E8 937CF2FF         call 7112D06C

712053D9    A1 18DE2171         mov eax,dword ptr ds:[7121DE18]

712053DE    8038 00             cmp byte ptr ds:[eax],0

//[71200685]处保存的是检验标志位

712053E1    75 16               jnz short 712053F9

//NOP掉	★8

712053E3    83C6 14             add esi,14

712053E6    FF45 E0             inc dword ptr ss:[ebp-20]

712053E9    837E 0C 00          cmp dword ptr ds:[esi+C],0

712053ED    76 0A               jbe short 712053F9

712053EF    837E 10 00          cmp dword ptr ds:[esi+10],0

712053F3    0F87 80FEFFFF       ja 71205279

//循环处理输入表[code]

★7. Patch Code:

[code]71232AE4     E8 C300FDFF         call 71202BAC

//71205394处代码挪这里执行

//EAX是写入IAT的加密地址

71232AE9     60                  pushad

71232AEA     9C                  pushfd

71232AEB     8925 DE2A2371       mov dword ptr ds:[71232ADE],esp

//保存ESP

71232AF1     832D DE2A2371 04    sub dword ptr ds:[71232ADE],4

71232AF8     FFD0                call eax  ;0118001D

//模拟程序调用加密函数

71232AFA     8BF0                mov esi,eax

71232AFC     8B4E FC             mov ecx,dword ptr ds:[esi-4]

71232AFF     3E:8B7D B8          mov edi,dword ptr ds:[ebp-48]

71232B03     66:C707 0000        mov word ptr ds:[edi],0

71232B08     83C7 02             add edi,2

71232B0B     F3:A4               rep movs byte ptr es:[edi],byte ptr ds:[esi]

71232B0D     C607 00             mov byte ptr ds:[edi],0

71232B10     9D                  popfd

71232B11     61                  popad

71232B12     E9 8228FDFF         jmp 71205399
```

从OllyDbg中二进制代码复制：

```
E8 C3 00 FD FF 60 9C 89 25 DE 2A 23 71 83 2D DE 2A 23 71 04 FF D0 8B F0 8B 4E FC 3E 8B 7D B8 66

C7 07 00 00 83 C7 02 F3 A4 C6 07 00 9D 61 E9 82 28 FD FF
```

___

进入`71232AF8` `call eax`:

```
0118001D     E8 76470770         call 711F4798

01180022     FF25 28001801       jmp dword ptr ds:[1180028]
```

```
711F4798     74 04               je short 711F479E

711F479A     75 02               jnz short 711F479E

711F479E     9C                  pushfd

711F479F     60                  pushad

711F47A0     74 03               je short 711F47A5

711F47A2     75 01               jnz short 711F47A5

711F47A5     9C                  pushfd

711F47A6     58                  pop eax

711F47A7     A3 94182271         mov dword ptr ds:[71221894],eax

711F47AC     74 04               je short 711F47B2

711F47AE     75 02               jnz short 711F47B2

711F47B2     E8 71000000         call 711F4828

711F47B7     31C0                xor eax,eax

711F47B9     A0 B0182271         mov al,byte ptr ds:[712218B0]

711F47BE     83F8 00             cmp eax,0

711F47C1     75 5F               jnz short 711F4822

711F47C3     8B4424 24           mov eax,dword ptr ss:[esp+24]

711F47C7     89C3                mov ebx,eax

711F47C9     83C0 02             add eax,2

711F47CC     8B00                mov eax,dword ptr ds:[eax]

711F47CE     8B00                mov eax,dword ptr ds:[eax]

711F47D0     31D8                xor eax,ebx

711F47D2     83F8 00             cmp eax,0

711F47D5     75 24               jnz short 711F47FB

711F47D7     8B4424 24           mov eax,dword ptr ss:[esp+24]

711F47DB     83C0 02             add eax,2

711F47DE     8B00                mov eax,dword ptr ds:[eax]

711F47E0     31C9                xor ecx,ecx

711F47E2     8A48 10             mov cl,byte ptr ds:[eax+10]

711F47E5     51                  push ecx

711F47E6     8B48 0C             mov ecx,dword ptr ds:[eax+C]

711F47E9     8B50 08             mov edx,dword ptr ds:[eax+8]

711F47EC     8B40 04             mov eax,dword ptr ds:[eax+4]

711F47EF     E8 FCE2FFFF         call 711F2AF0

//解密特殊函数，进入

711F47F4     31D8                xor eax,ebx

711F47F6     8943 06             mov dword ptr ds:[ebx+6],eax

711F47F9     31D8                xor eax,ebx

711F47FB     89C3                mov ebx,eax

711F47FD     894424 24           mov dword ptr ss:[esp+24],eax

711F4801     8B03                mov eax,dword ptr ds:[ebx]

711F4803     3C CC               cmp al,0CC

//检测普通IN3断点

711F4805     74 14               je short 711F481B

711F4807     80FC CC             cmp ah,0CC

711F480A     74 0F               je short 711F481B

711F480C     C1E8 10             shr eax,10

711F480F     3C CC               cmp al,0CC

711F4811     74 08               je short 711F481B

711F4813     80FC CC             cmp ah,0CC

711F4816     74 03               je short 711F481B

711F4818     EB 08               jmp short 711F4822

711F481A     E9 C605B018         jmp 89CF4DE5

711F481F     2271 01             and dh,byte ptr ds:[ecx+1]

711F4822     61                  popad

711F4823     9D                  popfd

711F4824     C3                  retn

711F4825     C3                  retn
```

___

进入`711F47EF` `call 711F2AF0`:
