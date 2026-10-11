
### 代码片段 1

```
0042C540   |.  8BD8            mov ebx,eax
0042C542   |.  83C4 08         add esp,8
0042C545   |.  85DB            test ebx,ebx
0042C547   |.  75 40           jnz short unpacked.0042C589
0042C549   |.  83C9 FF         or ecx,FFFFFFFF
0042C54C   |.  8BFE            mov edi,esi
0042C54E   |.  F2:AE           repne scas byte ptr es:[edi]
0042C550   |.  F7D1            not ecx
0042C552   |.  83C1 07         add ecx,7
0042C555   |.  51              push ecx
0042C556   |.  E8 C1500600     call <jmp.&MSVCRT.operator new>
0042C55B   |.  56              push esi                          ; /<%s> = "C:\Program Files\Rhino2\key"
0042C55C   |.  8BF8            mov edi,eax                       ; |
0042C55E   |.  68 88694B00     push unpacked.004B6988              ; |format = "%s.dat"
0042C563   |.  57              push edi                          ; |s
0042C564   |.  FF15 64A24900   call dword ptr ds:[<&MSVCRT.sprintf>]   ; \sprintf

0042C56A   |.  68 90694B00     push unpacked.004B6990                ; /mode = "rb"
0042C56F   |.  57              push edi                            ; |path = "C:\Program Files\Rhino2\key.dat"
0042C570   |.  FF15 20A24900   call dword ptr ds:[<&MSVCRT.fopen>]     ; \fopen

0042C576   |.  57              push edi
0042C577   |.  8BD8            mov ebx,eax
0042C579   |.  E8 72500600     call unpacked.004915F0
0042C57E   |.  83C4 1C         add esp,1C
0042C581   |.  85DB            test ebx,ebx
0042C583   |.  0F84 7C010000   je unpacked.0042C705
0042C589   |>  6A 02           push 2                            ; /whence = SEEK_END
0042C58B   |.  6A 00           push 0                            ; |offset = 0
0042C58D   |.  53              push ebx                          ; |stream = msvcrt.77C2FCE0
0042C58E   |.  FF15 0CA24900   call dword ptr ds:[<&MSVCRT.fseek>]    ; \fseek

0042C594   |.  53              push ebx                          ; /stream = msvcrt.77C2FCE0
0042C595   |.  FF15 10A24900   call dword ptr ds:[<&MSVCRT.ftell>]    ; \ftell
0042C59B   |.  83C4 10         add esp,10
0042C59E   |.  3D 8C010000     cmp eax,18C                       ;size(key.dat)=18C
0042C5A3   |.  0F85 B2000000   jnz unpacked.0042C65B
0042C5A9   |.  53              push ebx                          ; /stream = msvcrt.77C2FCE0
0042C5AA   |.  FF15 18A24900   call dword ptr ds:[<&MSVCRT.rewind>]    ; \rewind

0042C5B0   |.  53              push ebx                          ; /stream
0042C5B1   |.  68 8C010000     push 18C                          ; |n = 18C (396.)
0042C5B6   |.  8D4424 1C       lea eax,dword ptr ss:[esp+1C]     ; |
0042C5BA   |.  6A 01           push 1                            ; |size = 1
0042C5BC   |.  50              push eax                          ; |ptr = 0012F4A4
0042C5BD   |.  FF15 14A24900   call dword ptr ds:[<&MSVCRT.fread>]    ; \fread

//18C byte= 63 Dword


0042C5C3   |.  83C4 14         add esp,14
0042C5C6   |.  33FF            xor edi,edi
0042C5C8   |.  B8 78563412     mov eax,12345678
0042C5CD   |.  33C9            xor ecx,ecx
0042C5CF   |.  90              nop
0042C5D0   |>  8B548C 10       /mov edx,dword ptr ss:[esp+ecx*4+10]  ;esp+10=0012F4A4
0042C5D4   |.  8BF0            |mov esi,eax
0042C5D6   |.  0FAFF0          |imul esi,eax
0042C5D9   |.  03F8            |add edi,eax  ====\\
0042C5DB   |.  2BF2            |sub esi,edx   ===/exchanged
0042C5DD   |.  33FA            |xor edi,edx
0042C5DF   |.  41              |inc ecx
0042C5E0   |.  83F9 63         |cmp ecx,63                       ;Is the END?
0042C5E3   |.  8BC6            |mov eax,esi                      ;can be moved before 'inc ecx'
0042C5E5   |.^ 7C E9           \jl short unpacked.0042C5D0
0042C5E7   |.  33C9            xor ecx,ecx
0042C5E9   |.  3BF9            cmp edi,ecx                       ;edi=0?
0042C5EB   |.  75 37           jnz short unpacked.0042C624        ;Don't jump
```

`mov eax,12345678`

```
mov esi,12345678
xor edi,edi
for(i=0;i<63;i++)
{
    edx=base[i]
    //esi=eax
    //esi=esi*eax-edx
    esi=esi*esi-edx
    edi=(edi+esi) xor edx
    //eax=esi
}
```
after the loop,`edi` expects to be zero!
if the last Dword equal with the (`edi+esi`) ,then `edi` will be ZERO!
so we could do that easily!

### 代码片段 2

```
0042C5ED   |.  8B4424 10       mov eax,dword ptr ss:[esp+10]       ;the first Dword
0042C5F1   |.  B9 01000000     mov ecx,1
0042C5F6   |>  8B7C8C 10       /mov edi,dword ptr ss:[esp+ecx*4+10]
0042C5FA   |.  8D3401          |lea esi,dword ptr ds:[ecx+eax]     ;esi=ecx+eax
0042C5FD   |.  0FAFF0          |imul esi,eax                       ;esi=esi*eax
0042C600   |.  33F8            |xor edi,eax
0042C602   |.  897C8C 10       |mov dword ptr ss:[esp+ecx*4+10],edi    ;modify Dword
0042C606   |.  41              |inc ecx                            ;move to the next Dword
0042C607   |.  83F9 63         |cmp ecx,63
0042C60A   |.  8BD7            |mov edx,edi
0042C60C   |.  8D8416 78563412 |lea eax,dword ptr ds:[esi+edx+12345678] ;eax=esi+edx+12345678
0042C613   |.^ 7C E1           \jl short unpacked.0042C5F6
```

`eax=base[0];`
(`edi=0,implied`)

```
for(i=1;i<63;i++)
{
    edi=base[i];
    edi=edi xor eax;
    base[i]=edi;
    //esi=eax+i;
    //esi=esi*eax;
    //esi=(eax+i)*eax;
    //eax=esi+edi+12345678
    eax=(eax+i)*eax+edi+12345678
}
```

### 代码片段 3

```
0042C615   |.  B9 63000000     mov ecx,63
0042C61A   |.  8D7424 10       lea esi,dword ptr ss:[esp+10]        ;0012FAA4,Buffer pointer
0042C61E   |.  8BFD            mov edi,ebp                          ;0202C030,Buffer2 pointer
0042C620   |.  F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
0042C622   |.  33C9            xor ecx,ecx
0042C624   |>  B8 B0C84900     mov eax,unpacked.0049C8B0
```

`[Data]`

```
0009c8b0h: 1A 00 00 00 02 45 F8 01 3E 72 C5 02 A6 2F F5 02 ; .....E?>r???
0009c8c0h: 2C 56 A9 01 B0 85 18 02 B6 9C AB 01 94 05 1C 04 ; ,V?皡..稖??..
0009c8d0h: 2A 09 BC 01 A7 2F F5 02 7A 85 A9 01 F4 91 0C 04 ; *.???z 叐.魬..
0009c8e0h: 5C 2E 2B 04 00 00 00 00                      ; \.+......
```

`[End Data]`

### 代码片段 4

```
0042C629   |.  8DA424 00000000 lea esp,dword ptr ss:[esp]
0042C630   |> /8B55 04         /mov edx,dword ptr ss:[ebp+4]     ;0202C030,the 2nd Dword
0042C633   |.|3B10             |cmp edx,dword ptr ds:[eax]
0042C635   |.|75 03           |jnz short unpacked.0042C63A
0042C637   |.|894D 04         |mov dword ptr ss:[ebp+4],ecx     ;set zero
0042C63A   |> |8B10           |mov edx,dword ptr ds:[eax]
0042C63C   |.|83C0 04         |add eax,4
0042C63F   |.|85D2           |test edx,edx
0042C641   |.^\75 ED          \jnz short unpacked.0042C630
```
if the first Dword=one of the 13 Dword,then,set the 2nd Dword to 0!

```
0042C643   |.  B8 E8C84900     mov eax,unpacked.0049C8E8
```

`[DATA]`

```
0009c8e8h: 2E 37 24 03 E4 54 C4 02 00 00 00 00            ; .7$.銽?.....
```

`[END DATA]`
