
# 00437EC8

```
00437EC8 |.  FF85 68FFFFFF             inc dword ptr ss:[ebp-98]
00437ECE |.  8B8D 48FFFFFF             mov ecx,dword ptr ss:[ebp-B8]
00437ED4 |.  8B81 F0020000             mov eax,dword ptr ds:[ecx+2F0]
00437EDA |.  E8 A9840500             call <屏录专家.@TControl@GetText$qqrv>  ;取机器码
00437EDF |.  8D45 EC                 lea eax,dword ptr ss:[ebp-14]
                                       ;**eax->机器码
00437EE2 |.  E8 F1DDFCFF             call <屏录专家.@System@AnsiString@c_str$xqqrv>
00437EE7 |.  50                       push eax
00437EE8 |.  8D95 00FFFFFF             lea edx,dword ptr ss:[ebp-100]
00437EEE |.  52                       push edx
00437EEF |.  E8 08400A00             call <屏录专家._strcpy>           ;copy 机器码
00437EF4 |.  83C4 08                 add esp,8
00437EF7 |.  FF8D 68FFFFFF           dec dword ptr ss:[ebp-98]
00437EFD |.  8D45 EC                 lea eax,dword ptr ss:[ebp-14]
00437F00 |.  BA 02000000             mov edx,2
00437F05 |.  E8 C6F50A00             call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00437F0A |.  33C9                   xor ecx,ecx
00437F0C |.  898D 3CFFFFFF             mov dword ptr ss:[ebp-C4],ecx
```

;**********************************************************************************************

```
00437F12 |>  8B85 3CFFFFFF             /mov eax,dword ptr ss:[ebp-C4]
00437F18 |.  8A9405 E8FEFFFF         |mov dl,byte ptr ss:[ebp+eax-118] ；[ebp+eax-118]=注册名
00437F1F |.  8B8D 3CFFFFFF             |mov ecx,dword ptr ss:[ebp-C4]
00437F25 |.  32940D 00FFFFFF         |xor dl,byte ptr ss:[ebp+ecx-100] ；[ebp+ecx-100h]=机器码
00437F2C |.  8B85 3CFFFFFF             |mov eax,dword ptr ss:[ebp-C4]
00437F32 |.  889405 18FFFFFF         |mov byte ptr ss:[ebp+eax-E8],dl
00437F39 |.  8B95 3CFFFFFF             |mov edx,dword ptr ss:[ebp-C4]
00437F3F |.  0FBE8C15 18FFFFFF       |movsx ecx,byte ptr ss:[ebp+edx-E8]
00437F47 |.  898D CCFEFFFF             |mov dword ptr ss:[ebp-134],ecx
00437F4D |.  DB85 CCFEFFFF             |fild dword ptr ss:[ebp-134]
00437F53 |.  83C4 F8                 |add esp,-8
00437F56 |.  DD1C24                   |fstp qword ptr ss:[esp]
00437F59 |.  E8 02840A00             |call <屏录专家._fabs>           ；st=|[ebp-134h]|
00437F5E |.  83C4 08                 |add esp,8
00437F61 |.  DB85 3CFFFFFF             |fild dword ptr ss:[ebp-C4]    ；st=[ebp-0C4h]
00437F67 |.  DEC9                   |fmulp st(1),st        ; st=|[ebp-134h]|[ebp-0C4h]
00437F69 |.  DB85 40FFFFFF             |fild dword ptr ss:[ebp-C0]    ；st=[ebp-0C0h]
00437F6F |.  DEC1                   |faddp st(1),st          ; st=|[ebp-134h]|[ebp-0C4h]+[ebp-0C0h]
00437F71 |.  E8 12840A00             |call <屏录专家.@_ftol$qv>   ; eax=st
00437F76 |.  8985 40FFFFFF             |mov dword ptr ss:[ebp-C0],eax
00437F7C |.  FF85 3CFFFFFF             |inc dword ptr ss:[ebp-C4]
00437F82 |.  83BD 3CFFFFFF 14          |cmp dword ptr ss:[ebp-C4],14  ;20 位
00437F89 |.^ 7C 87                   \jl short 屏录专家.00437F12
```

#### 假设这段程序输出为 `temp_H`

```c
static char n[14]="ABABABAB" //注册名
static char m[14]="92388863" //机器码

int count;
int temp_H;

temp_H=0;

for count=0 to 14h do
{
temp_H=|n[count] xor m[count]|*count+temp_H
}
```

;**********************************************************************************************

```
00437F8B |.  8185 40FFFFFF 39300000  add dword ptr ss:[ebp-C0],3039
00437F95 |.  FFB5 40FFFFFF            push dword ptr ss:[ebp-C0]
00437F9B |.  68 57B85000              push 屏录专家.0050B857
00437FA0 |.  8D95 18FFFFFF            lea edx,dword ptr ss:[ebp-E8]
00437FA6 |.  52                       push edx
00437FA7 |.  E8 F8690A00              call <屏录专家._sprintf>  ；把 temp_H 转为十进制 temp_D
```

(十六进)`temp_H` -> (十进制)`temp_D`

;**********************************************************************************************

```
00437FAC |.  83C4 0C                  add esp,0C
00437FAF |.  66:C785 5CFFFFFF 3800    mov word ptr ss:[ebp-A4],38
00437FB8 |.  8D45 E8                  lea eax,dword ptr ss:[ebp-18]
00437FBB |.  E8 5099FCFF              call <屏录专家.unknown_libname_37>
00437FC0 |.  8BD0                     mov edx,eax
00437FC2 |.  FF85 68FFFFFF            inc dword ptr ss:[ebp-98]
00437FC8 |.  8B8D 48FFFFFF            mov ecx,dword ptr ss:[ebp-B8]
00437FCE |.  8B81 E4020000            mov eax,dword ptr ds:[ecx+2E4]
00437FD4 |.  E8 AF830500              call <屏录专家.@TControl@GetText$qqrv> ;取注册码
00437FD9 |.  8D45 E8                  lea eax,dword ptr ss:[ebp-18]
                                       ;**EAX-> 注册码
00437FDC |.  E8 F7DCFCFF              call <屏录专家.@System@AnsiString@c_str$xqqrv>
00437FE1 |.  50                       push eax
00437FE2 |.  8D95 D0FEFFFF            lea edx,dword ptr ss:[ebp-130]
00437FE8 |.  52                       push edx
00437FE9 |.  E8 0E3F0A00              call <屏录专家._strcpy>       ;COPY  取注册码
00437FEE |.  83C4 08                  add esp,8
00437FF1 |.  FF8D 68FFFFFF            dec dword ptr ss:[ebp-98]
00437FF7 |.  8D45 E8                  lea eax,dword ptr ss:[ebp-18]
00437FFA |.  BA 02000000              mov edx,2
00437FFF |.  E8 CCF40A00              call <屏录专家.@System@AnsiString@$bdtr$qqrv>
00438004 |.  33C9                    xor ecx,ecx
00438006 |.  898D 3CFFFFFF            mov dword ptr ss:[ebp-C4],ecx
```

```
0043800C |>  8B85 3CFFFFFF            /mov eax,dword ptr ss:[ebp-C4]
00438012 |.  0FBE9405 18FFFFFF       |movsx edx,byte ptr ss:[ebp+eax-E8];[ebp+eax-0E8h]=temp_D
0043801A |.  8B8D 3CFFFFFF            |mov ecx,dword ptr ss:[ebp-C4]
00438020 |.  0FBE840D D0FEFFFF       |movsx eax,byte ptr ss:[ebp+ecx-130];[ebp+ecx-130]=注册码
00438028 |.  83C0 EC                  |add eax,-14
0043802B |.  3BD0                     |cmp edx,eax
0043802D |.  75 5D                    |jnz short 屏录专家.0043808C
                                      |
0043802F |.  83BD 3CFFFFFF 03         |cmp dword ptr ss:[ebp-C4],3；是不是第 5 位
00438036 |.  75 45                    |jnz short 屏录专家.0043807D；不是第 5 位
                                      |
00438038 |.  8B95 40FFFFFF            |mov edx,dword ptr ss:[ebp-C0]；[ebp-C0]=temp_H
0043803E |.  81C2 444D0000            |add edx,4D44
00438044 |.  8995 CCFEFFFF            |mov dword ptr ss:[ebp-134],edx
0043804A |.  DB85 CCFEFFFF            |fild dword ptr ss:[ebp-134]
00438050 |.  DC0D A4894300            |fmul qword ptr ds:[4389A4]  ;3.14
00438056 |.  DB2D AC894300            |fld tbyte ptr ds:[4389AC]  ;1.59489633173843711e-1
0043805C |.  DEC9                    |fmulp st(1),st
0043805E |.  E8 25830A00             |call <屏录专家.@_ftol$qv>    ;eax=st
00438063 |.  8985 40FFFFFF            |mov dword ptr ss:[ebp-C0],eax
00438069 |.  8B85 40FFFFFF            |mov eax,dword ptr ss:[ebp-C0]
0043806F |.  B9 A0860100              |mov ecx,186A0
```
