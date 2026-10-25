
```
10001CB8    8D4C24 48           lea ecx,dword ptr ss:[esp+0x48]

10001CBC    E8 CF100000         call aveData.10002D90                      ; 类输入函数 ,这里初化q

10001CC1    68 01000100         push 0x10001

10001CC6    8D4C24 24           lea ecx,dword ptr ss:[esp+0x24]

10001CCA    E8 F1110000         call aveData.10002EC0                      ; 类构造函数,这里初始化 e=0x10001

10001CCF    8D5424 20           lea edx,dword ptr ss:[esp+0x20]

10001CD3    8D4C24 30           lea ecx,dword ptr ss:[esp+0x30]

10001CD7    52                  push edx

10001CD8    C68424 D4020000 06  mov byte ptr ss:[esp+0x2D4],0x6

10001CE0    E8 3B120000         call aveData.10002F20                      ; Lgint copy

10001CE5    8D4C24 20           lea ecx,dword ptr ss:[esp+0x20]

10001CE9    889C24 D0020000     mov byte ptr ss:[esp+0x2D0],bl

10001CF0    E8 7B120000         call aveData.10002F70                      ; 类析造函数

10001CF5    8D8424 88000000     lea eax,dword ptr ss:[esp+0x88]

10001CFC    6A 08               push 0x8

10001CFE    50                  push eax

10001CFF    8D4C24 30           lea ecx,dword ptr ss:[esp+0x30]

10001D03    E8 88100000         call aveData.10002D90                      ; 类输入函数 ,这里初始化 n

10001D08    B9 08000000         mov ecx,0x8

10001D0D    33C0                xor eax,eax                                ; ---------

10001D0F    8DBC24 A8000000     lea edi,dword ptr ss:[esp+0xA8]

10001D16    33D2                xor edx,edx

10001D18    F3:AB               rep stos dword ptr es:[edi]

10001D1A    8DBC24 C8000000     lea edi,dword ptr ss:[esp+0xC8]

10001D21    83C9 FF             or ecx,0xFFFFFFFF

10001D24    F2:AE               repne scas byte ptr es:[edi]

10001D26    F7D1                not ecx

10001D28    49                  dec ecx

10001D29    74 24               je XaveData.10001D4F

10001D2B    8A8C14 C8000000     mov cl,byte ptr ss:[esp+edx+0xC8]

10001D32    8DBC24 C8000000     lea edi,dword ptr ss:[esp+0xC8]

10001D39    888C14 A8000000     mov byte ptr ss:[esp+edx+0xA8],cl

10001D40    83C9 FF             or ecx,0xFFFFFFFF

10001D43    33C0                xor eax,eax                                ; 这里将输入的 szUserName

10001D45    42                  inc edx                                    ; 格式化以作为大整数输入

10001D46    F2:AE               repne scas byte ptr es:[edi]

10001D48    F7D1                not ecx                                    ; 具体是将每4个字节通过移位使其逆序,

10001D4A    49                  dec ecx                                    ; 各个4字节的顺序不变

10001D4B    3BD1                cmp edx,ecx

10001D4D  ^ 72 DC               jb XaveData.10001D2B

10001D4F    B9 08000000         mov ecx,0x8

10001D54    33C0                xor eax,eax

10001D56    8D7C24 48           lea edi,dword ptr ss:[esp+0x48]

10001D5A    F3:AB               rep stos dword ptr es:[edi]

10001D5C    33D2                xor edx,edx

10001D5E    33C9                xor ecx,ecx

10001D60    8A9404 A8000000     mov dl,byte ptr ss:[esp+eax+0xA8]

10001D67    8A8C04 A9000000     mov cl,byte ptr ss:[esp+eax+0xA9]

10001D6E    C1E2 08             shl edx,0x8

10001D71    03D1                add edx,ecx

10001D73    33C9                xor ecx,ecx

10001D75    8A8C04 AA000000     mov cl,byte ptr ss:[esp+eax+0xAA]

10001D7C    83C0 04             add eax,0x4

10001D7F    C1E2 08             shl edx,0x8

10001D82    03D1                add edx,ecx

10001D84    33C9                xor ecx,ecx

10001D86    8A8C04 A7000000     mov cl,byte ptr ss:[esp+eax+0xA7]

10001D8D    C1E2 08             shl edx,0x8

10001D90    03D1                add edx,ecx

10001D92    83F8 20             cmp eax,0x20

10001D95    895404 44           mov dword ptr ss:[esp+eax+0x44],edx

10001D99  ^ 7C C1               jl XaveData.10001D5C                       ; --------

10001D9B    8D5424 48           lea edx,dword ptr ss:[esp+0x48]

10001D9F    6A 08               push 0x8

10001DA1    52                  push edx

10001DA2    8D4C24 18           lea ecx,dword ptr ss:[esp+0x18]

10001DA6    E8 E50F0000         call aveData.10002D90                      ; 初始化username为大整数

10001DAB    8D4424 10           lea eax,dword ptr ss:[esp+0x10]

10001DAF    8D4C24 20           lea ecx,dword ptr ss:[esp+0x20]

10001DB3    50                  push eax

10001DB4    51                  push ecx

10001DB5    8D4C24 30           lea ecx,dword ptr ss:[esp+0x30]

10001DB9    E8 C2030000         call aveData.10002180                      ; F7跟进,这里是主要的加密在地方

10001DBE    50                  push eax                                   ; eax为计算结果对象指针

10001DBF    8D4C24 1C           lea ecx,dword ptr ss:[esp+0x1C]

10001DC3    C68424 D4020000 07  mov byte ptr ss:[esp+0x2D4],0x7

10001DCB    E8 50110000         call aveData.10002F20                      ; LgInt Copy()

10001DD0    8D4C24 20           lea ecx,dword ptr ss:[esp+0x20]

10001DD4    889C24 D0020000     mov byte ptr ss:[esp+0x2D0],bl

10001DDB    E8 90110000         call aveData.10002F70

10001DE0    8D5424 48           lea edx,dword ptr ss:[esp+0x48]

10001DE4    6A 08               push 0x8

10001DE6    52                  push edx

10001DE7    8D4C24 20           lea ecx,dword ptr ss:[esp+0x20]

10001DEB    E8 E00F0000         call aveData.10002DD0                      ; 类输入函数,将大整数复制到参数缓冲区a

10001DF0    8B5424 58           mov edx,dword ptr ss:[esp+0x58]            ; edx=a[4]

10001DF4    8B4C24 54           mov ecx,dword ptr ss:[esp+0x54]            ; ecx=a[3]

10001DF8    8B7424 50           mov esi,dword ptr ss:[esp+0x50]            ; esi=a[2]

10001DFC    8B7C24 4C           mov edi,dword ptr ss:[esp+0x4C]            ; edi=a[1]

10001E00    8B6C24 64           mov ebp,dword ptr ss:[esp+0x64]            ; ebp=a[7]

10001E04    8B5C24 48           mov ebx,dword ptr ss:[esp+0x48]            ; ebx=a[0]

10001E08    8D040A              lea eax,dword ptr ds:[edx+ecx]

10001E0B    0FAFCE              imul ecx,esi

10001E0E    03C6                add eax,esi

10001E10    03C7                add eax,edi

10001E12    33E8                xor ebp,eax                                ; ebp=a[7] xor (a[4]+a[3]+a[2]+a[1])

10001E14    8B4424 5C           mov eax,dword ptr ss:[esp+0x5C]

10001E18    03D8                add ebx,eax

10001E1A    8B4424 60           mov eax,dword ptr ss:[esp+0x60]

10001E1E    33C3                xor eax,ebx                                ; eax=(a[5]+a[0]) xor a[6]

10001E20    896C24 64           mov dword ptr ss:[esp+0x64],ebp

10001E24    894424 60           mov dword ptr ss:[esp+0x60],eax

10001E28    0FAFC5              imul eax,ebp

10001E2B    8B6C24 48           mov ebp,dword ptr ss:[esp+0x48]

10001E2F    2BC1                sub eax,ecx

10001E31    8B4C24 5C           mov ecx,dword ptr ss:[esp+0x5C]

10001E35    2BC1                sub eax,ecx

10001E37    8D8C24 C8010000     lea ecx,dword ptr ss:[esp+0x1C8]

10001E3E    03C5                add eax,ebp

10001E40    03C2                add eax,edx

10001E42    03C7                add eax,edi                                ; eax=eax\*ebp-a[3]\*a[2]-a[5]+a[4]+a[1]+a[0]

10001E44    50                  push eax

10001E45    68 8CE40010         push aveData.1000E48C                      ; ASCII "%08lX"

10001E4A    51                  push ecx

10001E4B    E8 58240000         call aveData.100042A8                      ; wsprintf( szSerial ,"%08lX" ,eax)

10001E50    8B8424 E8020000     mov eax,dword ptr ss:[esp+0x2E8]

10001E57    8D9424 D4010000     lea edx,dword ptr ss:[esp+0x1D4]

10001E5E    6A 08               push 0x8

10001E60    52                  push edx

10001E61    50                  push eax

10001E62    E8 09240000         call aveData.10004270                      ; memcmp(szPasswod  ,szSerial ,8)

10001E67    83C4 18             add esp,0x18

10001E6A    85C0                test eax,eax

10001E6C    5F                  pop edi

10001E6D    5E                  pop esi

10001E6E    5D                  pop ebp

10001E6F    5B                  pop ebx
```
