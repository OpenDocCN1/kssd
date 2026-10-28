
```
100018F8  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

100018FB  |.  E8 F0F9FFFF   call    <GetAPIAddressByHash_2>

10001900  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001903  |.  8981 BC000000 mov     dword ptr [ecx+BC], eax

10001909  |.  68 01144601   push    1461401

1000190E  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001911  |.  E8 DAF9FFFF   call    <GetAPIAddressByHash_2>

10001916  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001919  |.  8982 04010000 mov     dword ptr [edx+104], eax

1000191F  |.  68 2EBBE374   push    74E3BB2E

10001924  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001927  |.  E8 24FAFFFF   call    <GetAPIAddressByHash>

1000192C  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

1000192F  |.  8981 C0000000 mov     dword ptr [ecx+C0], eax

10001935  |.  68 10CA4937   push    3749CA10

1000193A  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

1000193D  |.  E8 0EFAFFFF   call    <GetAPIAddressByHash>

10001942  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001945  |.  8982 C4000000 mov     dword ptr [edx+C4], eax

1000194B  |.  68 B73CF449   push    49F43CB7

10001950  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001953  |.  E8 F8F9FFFF   call    <GetAPIAddressByHash>

10001958  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

1000195B  |.  8981 C8000000 mov     dword ptr [ecx+C8], eax

10001961  |.  68 EE4A7F2C   push    2C7F4AEE

10001966  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001969  |.  E8 E2F9FFFF   call    <GetAPIAddressByHash>

1000196E  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001971  |.  8982 CC000000 mov     dword ptr [edx+CC], eax

10001977  |.  68 4DD72863   push    6328D74D

1000197C  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

1000197F  |.  E8 CCF9FFFF   call    <GetAPIAddressByHash>

10001984  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001987  |.  8981 D0000000 mov     dword ptr [ecx+D0], eax

1000198D  |.  68 9FD49516   push    1695D49F

10001992  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001995  |.  E8 B6F9FFFF   call    <GetAPIAddressByHash>

1000199A  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

1000199D  |.  8982 D8000000 mov     dword ptr [edx+D8], eax

100019A3  |.  68 9AC2BE2A   push    2ABEC29A

100019A8  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

100019AB  |.  E8 A0F9FFFF   call    <GetAPIAddressByHash>

100019B0  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

100019B3  |.  8981 DC000000 mov     dword ptr [ecx+DC], eax

100019B9  |.  C645 AC 00    mov     byte ptr [ebp-54], 0

100019BD  |.  C645 AD 3F    mov     byte ptr [ebp-53], 3F

100019C1  |.  C645 AE 24    mov     byte ptr [ebp-52], 24

100019C5  |.  C645 AF 22    mov     byte ptr [ebp-51], 22

100019C9  |.  C645 B0 23    mov     byte ptr [ebp-50], 23

100019CD  |.  C645 B1 37    mov     byte ptr [ebp-4F], 37

100019D1  |.  C645 B2 3A    mov     byte ptr [ebp-4E], 3A

100019D5  |.  C645 B3 17    mov     byte ptr [ebp-4D], 17

100019D9  |.  C645 B4 3A    mov     byte ptr [ebp-4C], 3A

100019DD  |.  C645 B5 3A    mov     byte ptr [ebp-4B], 3A

100019E1  |.  C645 B6 39    mov     byte ptr [ebp-4A], 39

100019E5  |.  C645 B7 35    mov     byte ptr [ebp-49], 35

100019E9  |.  C645 B8 13    mov     byte ptr [ebp-48], 13

100019ED  |.  C645 B9 2E    mov     byte ptr [ebp-47], 2E

100019F1  |.  C645 BA 56    mov     byte ptr [ebp-46], 56

100019F5  |.  6A 56         push    56

100019F7  |.  6A 0F         push    0F

100019F9  |.  8D55 AC       lea     edx, dword ptr [ebp-54]

100019FC  |.  52            push    edx

100019FD  |.  8D45 AC       lea     eax, dword ptr [ebp-54]

10001A00  |.  50            push    eax

10001A01  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A04  |.  E8 47F6FFFF   call    <DecodeString>

10001A09  |.  8D4D AC       lea     ecx, dword ptr [ebp-54]

10001A0C  |.  51            push    ecx

10001A0D  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A10  |.  E8 DBF7FFFF   call    <GetApiAddressByName>

10001A15  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001A18  |.  8982 D4000000 mov     dword ptr [edx+D4], eax

10001A1E  |.  68 C021115F   push    5F1121C0

10001A23  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A26  |.  E8 25F9FFFF   call    <GetAPIAddressByHash>

10001A2B  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A2E  |.  8981 E0000000 mov     dword ptr [ecx+E0], eax

10001A34  |.  68 62B2F477   push    77F4B262

10001A39  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A3C  |.  E8 0FF9FFFF   call    <GetAPIAddressByHash>

10001A41  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001A44  |.  8982 E4000000 mov     dword ptr [edx+E4], eax

10001A4A  |.  68 7BB7AA51   push    51AAB77B

10001A4F  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A52  |.  E8 F9F8FFFF   call    <GetAPIAddressByHash>

10001A57  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A5A  |.  8981 E8000000 mov     dword ptr [ecx+E8], eax

10001A60  |.  68 962F901C   push    1C902F96

10001A65  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A68  |.  E8 E3F8FFFF   call    <GetAPIAddressByHash>

10001A6D  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001A70  |.  8982 EC000000 mov     dword ptr [edx+EC], eax

10001A76  |.  68 D3774938   push    384977D3

10001A7B  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A7E  |.  E8 9DF8FFFF   call    10001320

10001A83  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A86  |.  8981 F0000000 mov     dword ptr [ecx+F0], eax

10001A8C  |.  68 E558D366   push    66D358E5

10001A91  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001A94  |.  E8 87F8FFFF   call    10001320

10001A99  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001A9C  |.  8982 F4000000 mov     dword ptr [edx+F4], eax

10001AA2  |.  68 A2341367   push    671334A2

10001AA7  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001AAA  |.  E8 71F8FFFF   call    10001320

10001AAF  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001AB2  |.  8981 F8000000 mov     dword ptr [ecx+F8], eax

10001AB8  |.  68 CDF81A09   push    91AF8CD

10001ABD  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001AC0  |.  E8 FBF7FFFF   call    <GetAPIAddressByHash_2>

10001AC5  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001AC8  |.  8982 FC000000 mov     dword ptr [edx+FC], eax

10001ACE  |.  68 15157A25   push    257A1515

10001AD3  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001AD6  |.  E8 E5F7FFFF   call    <GetAPIAddressByHash_2>

10001ADB  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001ADE  |.  8981 00010000 mov     dword ptr [ecx+100], eax

10001AE4  |.  68 A168A759   push    59A768A1

10001AE9  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001AEC  |.  E8 CFF7FFFF   call    <GetAPIAddressByHash_2>

10001AF1  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001AF4  |.  8982 08010000 mov     dword ptr [edx+108], eax

10001AFA  |.  68 7BBE2E07   push    72EBE7B

10001AFF  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001B02  |.  E8 B9F7FFFF   call    <GetAPIAddressByHash_2>

10001B07  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001B0A  |.  8981 0C010000 mov     dword ptr [ecx+10C], eax

10001B10  |.  68 40954A19   push    194A9540

10001B15  |.  8B4D 9C       mov     ecx, dword ptr [ebp-64]

10001B18  |.  E8 A3F7FFFF   call    <GetAPIAddressByHash_2>

10001B1D  |.  8B55 9C       mov     edx, dword ptr [ebp-64]

10001B20  |.  8982 10010000 mov     dword ptr [edx+110], eax

10001B26  |.  8B45 9C       mov     eax, dword ptr [ebp-64]

10001B29  |.  8BE5          mov     esp, ebp

10001B2B  |.  5D            pop     ebp

10001B2C  \.  C3            retn
```

再看`GetAPIAddressByHash`这个`call`。这个`call`是根据函数名的`Hash`值得到其`API`地址并返回的。如下：

```
10001350 >/$  55            push    ebp

10001351  |.  8BEC          mov     ebp, esp

10001353  |.  83EC 0C       sub     esp, 0C
```
