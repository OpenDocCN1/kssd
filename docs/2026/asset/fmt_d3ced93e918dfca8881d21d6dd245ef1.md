
```
CODE:006B81E9                 mov     eax, off\_699A78

CODE:006B81EE                 call    sub\_473E8C

CODE:006B81F3                 mov     edx, ds:off\_6C5A5C

CODE:006B81F9                 mov     [edx], eax

CODE:006B81FB                 mov     eax, ds:off\_6C5A5C

CODE:006B8200                 mov     eax, [eax]

CODE:006B8202                 mov     edx, [eax]

CODE:006B8204                 call    dword ptr [edx+0ECh]

CODE:006B820A                 mov     eax, ds:off\_6C5A5C

CODE:006B820F                 mov     eax, [eax]

CODE:006B8211                 call    sub\_403D50

CODE:006B8216                 mov     eax, ebx

CODE:006B8218                 call    sub\_478054

CODE:006B821D                 pop     ebx

CODE:006B821E                 retn

===========================================call    sub\_6A004C=========================================

CODE:006A004C var\_70          = dword ptr -70h

CODE:006A004C var\_6C          = dword ptr -6Ch

CODE:006A004C var\_68          = dword ptr -68h

CODE:006A004C var\_64          = dword ptr -64h

CODE:006A004C var\_60          = dword ptr -60h

CODE:006A004C var\_5C          = dword ptr -5Ch

CODE:006A004C var\_58          = dword ptr -58h

CODE:006A004C var\_54          = dword ptr -54h

CODE:006A004C var\_50          = dword ptr -50h

CODE:006A004C var\_4C          = dword ptr -4Ch

CODE:006A004C var\_48          = dword ptr -48h

CODE:006A004C var\_44          = dword ptr -44h

CODE:006A004C var\_40          = dword ptr -40h

CODE:006A004C var\_3C          = dword ptr -3Ch

CODE:006A004C var\_38          = dword ptr -38h

CODE:006A004C var\_34          = dword ptr -34h

CODE:006A004C var\_24          = dword ptr -24h

CODE:006A004C var\_20          = dword ptr -20h

CODE:006A004C var\_1C          = dword ptr -1Ch

CODE:006A004C var\_18          = dword ptr -18h

CODE:006A004C var\_10          = dword ptr -10h

CODE:006A004C var\_C           = dword ptr -0Ch

CODE:006A004C var\_8           = dword ptr -8

CODE:006A004C var\_4           = dword ptr -4

CODE:006A004C

CODE:006A004C                 push    ebp

CODE:006A004D                 mov     ebp, esp

CODE:006A004F                 mov     ecx, 0Eh

CODE:006A0054

CODE:006A0054 loc\_6A0054:                             ; CODE XREF: sub\_6A004C+Dj

CODE:006A0054                 push    0

CODE:006A0056                 push    0

CODE:006A0058                 dec     ecx

CODE:006A0059                 jnz     short loc\_6A0054

CODE:006A005B                 push    ebx

CODE:006A005C                 push    esi

CODE:006A005D                 push    edi

CODE:006A005E                 mov     esi, eax

CODE:006A0060                 xor     eax, eax

CODE:006A0062                 push    ebp

CODE:006A0063                 push    offset loc\_6A0407

CODE:006A0068

CODE:006A0068 loc\_6A0068:                             ; DATA XREF: CODE:off\_47CF60o

CODE:006A0068                                         ; CODE:off\_49BD82o ...

CODE:006A0068                 push    dword ptr fs:[eax]

CODE:006A006B                 mov     fs:[eax], esp

CODE:006A006E                 mov     bl, 1

CODE:006A0070                 mov     eax, ds:off\_6C5A60

CODE:006A0075                 cmp     dword ptr [eax], 0FFFFFFFFh

CODE:006A0078                 jnz     short loc\_6A00B0

CODE:006A007A                 mov     eax, ds:off\_6C5A60

CODE:006A007F                 cmp     dword ptr [eax+4], 0FFFFFFFFh

CODE:006A0083                 jnz     short loc\_6A00B0

CODE:006A0085                 mov     edi, [esi+520h]

CODE:006A008B                 mov     eax, [esi+48h]

CODE:006A008E                 sub     eax, [edi+2DCh]

CODE:006A0094                 sub     eax, 32h

CODE:006A0097                 lea     ecx, [ebp+var\_18]

CODE:006A009A                 mov     edx, 64h

CODE:006A009F                 call    sub\_41B49C

CODE:006A00A4                 lea     edx, [ebp+var\_18]

CODE:006A00A7                 mov     eax, edi

CODE:006A00A9                 call    sub\_55D0E0

CODE:006A00AE                 jmp     short loc\_6A00C1

CODE:006A00B0 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

CODE:006A00B0

CODE:006A00B0 loc\_6A00B0:                             ; CODE XREF: sub\_6A004C+2Cj

CODE:006A00B0                                         ; sub\_6A004C+37j

CODE:006A00B0                 mov     edx, ds:off\_6C5A60

CODE:006A00B6                 mov     eax, [esi+520h]

CODE:006A00BC                 call    sub\_55D0E0

CODE:006A00C1

CODE:006A00C1 loc\_6A00C1:                             ; CODE XREF: sub\_6A004C+62j

CODE:006A00C1                 mov     eax, ds:off\_6C624C

CODE:006A00C6                 mov     eax, [eax]

CODE:006A00C8                 call    sub\_4792F0

CODE:006A00CD                 cmp     eax, 320h

CODE:006A00D2                 jle     short loc\_6A00DE

CODE:006A00D4                 mov     eax, ds:off\_6C61C8

CODE:006A00D9                 cmp     byte ptr [eax], 0

CODE:006A00DC                 jz      short loc\_6A00F8

CODE:006A00DE

CODE:006A00DE loc\_6A00DE:                             ; CODE XREF: sub\_6A004C+86j

CODE:006A00DE                 mov     dl, 1

CODE:006A00E0                 mov     eax, [esi+520h]

CODE:006A00E6                 call    sub\_55CEC4

CODE:006A00EB                 mov     dl, 1

CODE:006A00ED                 mov     eax, [esi+520h]

CODE:006A00F3                 call    sub\_559708

CODE:006A00F8

CODE:006A00F8 loc\_6A00F8:                             ; CODE XREF: sub\_6A004C+90j

CODE:006A00F8                 xor     edx, edx

CODE:006A00FA                 mov     eax, [esi+780h]

CODE:006A0100                 call    sub\_450F90

CODE:006A0105                 mov     edx, ds:off\_6C5958

CODE:006A010B                 mov     dl, [edx]

CODE:006A010D                 mov     eax, [esi+520h]

CODE:006A0113                 call    sub\_48AC90

CODE:006A0118                 mov     edx, ds:off\_6C5958

CODE:006A011E                 mov     dl, [edx]

CODE:006A0120                 mov     eax, [esi+664h]

CODE:006A0126                 call    sub\_46E810

CODE:006A012B                 mov     eax, esi

CODE:006A012D                 call    sub\_6AF848

CODE:006A0132                 lea     edx, [ebp+var\_4]

CODE:006A0135                 lea     eax, [ebp+var\_10]

CODE:006A0138                 call    sub\_4A1578

CODE:006A013D                 cmp     [ebp+var\_10], 0 ; 注册名为空?

CODE:006A0141                 jz      loc\_6A03C0

CODE:006A0147                 cmp     [ebp+var\_4], 0  ; 注册码为空?

CODE:006A014B                 jz      loc\_6A03C0

CODE:006A0151                 mov     eax, [ebp+var\_4] ; 指向注册码

CODE:006A0154                 call    sub\_404E9C      ; 返回注册码长度

CODE:006A0159                 cmp     eax, 17h        ; 输入的注册码必须为23位

CODE:006A015C                 jnz     loc\_6A03C0

CODE:006A0162                 mov     dl, 1

CODE:006A0164                 mov     eax, off\_41A798

CODE:006A0169                 call    sub\_403D20

CODE:006A016E                 mov     [ebp+var\_8], eax

CODE:006A0171                 push    ebp

CODE:006A0172                 call    sub\_69FF88

CODE:006A0177                 pop     ecx

CODE:006A0178                 mov     eax, [ebp+var\_8]

CODE:006A017B                 mov     edx, [eax]

CODE:006A017D                 call    dword ptr [edx+14h]

CODE:006A0180                 cmp     eax, 4

CODE:006A0183                 jnz     loc\_6A03B8

CODE:006A0189                 lea     ecx, [ebp+var\_1C]

CODE:006A018C                 xor     edx, edx        ; 要取得的注册码的段索引

CODE:006A018E                 mov     eax, [ebp+var\_8]

CODE:006A0191                 mov     edi, [eax]

CODE:006A0193                 call    dword ptr [edi+0Ch]

CODE:006A0196                 mov     eax, [ebp+var\_1C] ; eax指向注册码前5位

CODE:006A0199                 or      edx, 0FFFFFFFFh

CODE:006A019C                 call    sub\_409E40      ; 把eax指向的转化为16进制表示
```
