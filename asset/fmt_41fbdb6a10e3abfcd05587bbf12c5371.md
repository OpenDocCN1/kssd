
```
0041E283    B1 5C               mov cl,5C

0041E285    8D4424 3C           lea eax,dword ptr ss:[esp+3C]

0041E289    885C24 10           mov byte ptr ss:[esp+10],bl

0041E28D    885C24 12           mov byte ptr ss:[esp+12],bl

0041E291    885C24 17           mov byte ptr ss:[esp+17],bl

0041E295    885C24 21           mov byte ptr ss:[esp+21],bl

0041E299    885C24 31           mov byte ptr ss:[esp+31],bl

0041E29D    884C24 0C           mov byte ptr ss:[esp+C],cl

0041E2A1    884C24 15           mov byte ptr ss:[esp+15],cl

0041E2A5    884C24 1F           mov byte ptr ss:[esp+1F],cl

0041E2A9    884C24 27           mov byte ptr ss:[esp+27],cl

0041E2AD    884C24 2D           mov byte ptr ss:[esp+2D],cl

0041E2B1    884C24 34           mov byte ptr ss:[esp+34],cl

0041E2B5    33DB                xor ebx,ebx

0041E2B7    50                  push eax

0041E2B8    68 3F000F00         push 0F003F

0041E2BD    8D4C24 0C           lea ecx,dword ptr ss:[esp+C]

0041E2C1    53                  push ebx

0041E2C2    B2 63               mov dl,63

0041E2C4    51                  push ecx

0041E2C5    68 02000080         push 80000002

0041E2CA    C64424 18 53        mov byte ptr ss:[esp+18],53

0041E2CF    C64424 19 4F        mov byte ptr ss:[esp+19],4F

0041E2D4    C64424 1A 46        mov byte ptr ss:[esp+1A],46

0041E2D9    C64424 1B 54        mov byte ptr ss:[esp+1B],54

0041E2DE    C64424 1C 57        mov byte ptr ss:[esp+1C],57

0041E2E3    C64424 1D 41        mov byte ptr ss:[esp+1D],41

0041E2E8    C64424 1E 52        mov byte ptr ss:[esp+1E],52

0041E2ED    C64424 1F 45        mov byte ptr ss:[esp+1F],45

0041E2F2    C64424 21 50        mov byte ptr ss:[esp+21],50

0041E2F7    C64424 23 6C        mov byte ptr ss:[esp+23],6C

0041E2FC    885424 25           mov byte ptr ss:[esp+25],dl

0041E300    C64424 27 65        mov byte ptr ss:[esp+27],65

0041E305    C64424 28 73        mov byte ptr ss:[esp+28],73

0041E30A    C64424 2A 4D        mov byte ptr ss:[esp+2A],4D

0041E30F    885424 2C           mov byte ptr ss:[esp+2C],dl

0041E313    C64424 2D 72        mov byte ptr ss:[esp+2D],72

0041E318    C64424 2F 73        mov byte ptr ss:[esp+2F],73

0041E31D    C64424 31 66        mov byte ptr ss:[esp+31],66

0041E322    C64424 32 74        mov byte ptr ss:[esp+32],74

0041E327    C64424 34 57        mov byte ptr ss:[esp+34],57

0041E32C    C64424 36 6E        mov byte ptr ss:[esp+36],6E

0041E331    C64424 37 64        mov byte ptr ss:[esp+37],64

0041E336    C64424 39 77        mov byte ptr ss:[esp+39],77

0041E33B    C64424 3A 73        mov byte ptr ss:[esp+3A],73

0041E340    C64424 3C 49        mov byte ptr ss:[esp+3C],49

0041E345    C64424 3D 50        mov byte ptr ss:[esp+3D],50

0041E34A    C64424 3E 53        mov byte ptr ss:[esp+3E],53

0041E34F    C64424 3F 65        mov byte ptr ss:[esp+3F],65

0041E354    885424 40           mov byte ptr ss:[esp+40],dl

0041E358    C64424 42 50        mov byte ptr ss:[esp+42],50

0041E35D    C64424 44 6C        mov byte ptr ss:[esp+44],6C

0041E362    885424 46           mov byte ptr ss:[esp+46],dl

0041E366    C64424 47 79        mov byte ptr ss:[esp+47],79

0041E36B    C64424 49 4C        mov byte ptr ss:[esp+49],4C

0041E370    885424 4B           mov byte ptr ss:[esp+4B],dl

0041E374    C64424 4C 61        mov byte ptr ss:[esp+4C],61

0041E379    C64424 4D 6C        mov byte ptr ss:[esp+4D],6C

0041E37E    885C24 4E           mov byte ptr ss:[esp+4E],bl
```

上面的代码形成了注册表路径：`SOFTWARE\Policies\Microsoft\Windows\IPSec\Policy\Local`

```
0041E382    E8 59FDFFFF         call x1un_.0041E0E0
```

这个函数前面已经出现过了，就是加载 `DLL`，这次加载的是 `advapi32`，基址是：`77DA0000`，开始对注册表进行操作

```
{
0041E0E0    83EC 20             sub esp,20

0041E0E3    B0 65               mov al,65

0041E0E5    56                  push esi

0041E0E6    884424 15           mov byte ptr ss:[esp+15],al

0041E0EA    884424 19           mov byte ptr ss:[esp+19],al

0041E0EE    884424 1C           mov byte ptr ss:[esp+1C],al

0041E0F2    B0 64               mov al,64

0041E0F4    884424 05           mov byte ptr ss:[esp+5],al

0041E0F8    884424 0D           mov byte ptr ss:[esp+D],al

0041E0FC    B0 6C               mov al,6C

0041E0FE    B2 70               mov dl,70

0041E100    884424 0E           mov byte ptr ss:[esp+E],al

0041E104    884424 0F           mov byte ptr ss:[esp+F],al

0041E108    8D4424 04           lea eax,dword ptr ss:[esp+4]

0041E10C    B1 61               mov cl,61

0041E10E    50                  push eax

0041E10F    C64424 18 52        mov byte ptr ss:[esp+18],52

0041E114    C64424 1A 67        mov byte ptr ss:[esp+1A],67

0041E119    C64424 1B 4F        mov byte ptr ss:[esp+1B],4F

0041E11E    885424 1C           mov byte ptr ss:[esp+1C],dl

0041E122    C64424 1E 6E        mov byte ptr ss:[esp+1E],6E

0041E127    C64424 1F 4B        mov byte ptr ss:[esp+1F],4B

0041E12C    C64424 21 79        mov byte ptr ss:[esp+21],79

0041E131    C64424 22 45        mov byte ptr ss:[esp+22],45

0041E136    C64424 23 78        mov byte ptr ss:[esp+23],78

0041E13B    C64424 24 41        mov byte ptr ss:[esp+24],41

0041E140    C64424 25 00        mov byte ptr ss:[esp+25],0

0041E145    884C24 08           mov byte ptr ss:[esp+8],cl

0041E149    C64424 0A 76        mov byte ptr ss:[esp+A],76

0041E14E    884C24 0B           mov byte ptr ss:[esp+B],cl

0041E152    885424 0C           mov byte ptr ss:[esp+C],dl

0041E156    C64424 0D 69        mov byte ptr ss:[esp+D],69

0041E15B    C64424 0E 33        mov byte ptr ss:[esp+E],33

0041E160    C64424 0F 32        mov byte ptr ss:[esp+F],32

0041E165    C64424 10 2E        mov byte ptr ss:[esp+10],2E

0041E16A    C64424 14 00        mov byte ptr ss:[esp+14],0

0041E16F    FF15 24104000       call dword ptr ds:[<&KERNEL32.LoadLibraryA>]     ; kernel32.LoadLibraryA

0041E175    8D4C24 14           lea ecx,dword ptr ss:[esp+14]                  ; RegOpenKeyExA

0041E179    8BF0                mov esi,eax                                    ; advapi32 的基址：77da0000

0041E17B    51                  push ecx                                       ; RegOpenKeyExA

0041E17C    56                  push esi                                       ; 基址

0041E17D    FF15 20104000       call dword ptr ds:[<&KERNEL32.GetProcAddress>] ; GetProcAddress

0041E183    85C0                test eax,eax

0041E185    74 2D               je short x1un_.0041E1B4

0041E187    8B5424 38           mov edx,dword ptr ss:[esp+38]

0041E18B    8B4C24 34           mov ecx,dword ptr ss:[esp+34]

0041E18F    57                  push edi

0041E190    52                  push edx

0041E191    8B5424 38           mov edx,dword ptr ss:[esp+38]

0041E195    51                  push ecx

0041E196    8B4C24 38           mov ecx,dword ptr ss:[esp+38]

0041E19A    52                  push edx

0041E19B    8B5424 38           mov edx,dword ptr ss:[esp+38]

0041E19F    51                  push ecx                                       ; SOFTWARE\Policies\Microsoft\Windows\IPSec\Policy\Local

0041E1A0    52                  push edx

0041E1A1    FFD0                call eax                                       ; RegOpenKeyExA

0041E1A3    56                  push esi

0041E1A4    8BF8                mov edi,eax

0041E1A6    FF15 30104000       call dword ptr ds:[<&KERNEL32.FreeLibrary>]     ; kernel32.FreeLibrary

0041E1AC    8BC7                mov eax,edi

0041E1AE    5F                  pop edi

0041E1AF    5E                  pop esi

0041E1B0    83C4 20             add esp,20

0041E1B3    C3                  retn

}
```

```
0041E387    83C4 14             add esp,14

0041E38A    85C0                test eax,eax

0041E38C    74 0A               je short x1un_.0041E398                         跳转未实现

0041E38E    33C0                xor eax,eax

0041E390    5B                  pop ebx

0041E391    81C4 3C020000       add esp,23C

0041E397    C3                  retn

}
```

```
0041E6D0    68 D0070000         push 7D0

0041E6D5    E8 66F7FFFF         call x1un_.0041DE40    这个函数前面分析过      ；同理这个函数加载 Kernel.dll
```
