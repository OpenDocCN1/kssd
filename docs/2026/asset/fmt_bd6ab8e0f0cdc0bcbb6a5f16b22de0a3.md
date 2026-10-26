
005FA499   .  85DB          test ebx,ebx

005FA49B   .^ 75 DF         jnz short SSQDYJ.005FA47C         ;  循环 16 (字符串长度) 轮

005FA49D   >  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA4A0   .  E8 4BAAE0FF   call SSQDYJ.00404EF0              ;  strlen( szMac )

005FA4A5   .  83F8 05       cmp eax,5

005FA4A8   .  0F8C D9010000 jl SSQDYJ.005FA687                ;  strlen ( szMac ) 不小于 5

005FA4AE   .  8B45 F8       mov eax,dword ptr ss:[ebp-8]

005FA4B1   .  E8 3AAAE0FF   call SSQDYJ.00404EF0              ;  strlen( szReg )

005FA4B6   .  83F8 08       cmp eax,8

005FA4B9   .  0F8C C8010000 jl SSQDYJ.005FA687                ;  strlen ( szReg ) 不小于 8

005FA4BF   .  E8 F487E0FF   call SSQDYJ.00402CB8              ;  GetSystemTime

005FA4C4   .  33C0          xor eax,eax

005FA4C6   .  55            push ebp

005FA4C7   .  68 79A65F00   push SSQDYJ.005FA679

005FA4CC   .  64:FF30       push dword ptr fs:[eax]

005FA4CF   .  64:8920       mov dword ptr fs:[eax],esp

005FA4D2   .  B8 0A000000   mov eax,0A

005FA4D7   .  E8 788EE0FF   call SSQDYJ.00403354

005FA4DC   .  85C0          test eax,eax

005FA4DE   .  7D 29         jge short SSQDYJ.005FA509

005FA4E0   .  8D45 E4       lea eax,dword ptr ss:[ebp-1C]

005FA4E3   .  8B55 F8       mov edx,dword ptr ss:[ebp-8]

005FA4E6   .  E8 EDAFE0FF   call SSQDYJ.004054D8

005FA4EB   .  8B55 E4       mov edx,dword ptr ss:[ebp-1C]

005FA4EE   .  B9 01000000   mov ecx,1

005FA4F3   .  8BC6          mov eax,esi

005FA4F5   .  E8 5EFEFFFF   call SSQDYJ.005FA358

005FA4FA   .  66:83F8 01    cmp ax,1

005FA4FE   .  1BC0          sbb eax,eax

005FA500   .  40            inc eax

005FA501   .  8845 F7       mov byte ptr ss:[ebp-9],al

005FA504   .  E9 66010000   jmp SSQDYJ.005FA66F

005FA509   >  8D45 F0       lea eax,dword ptr ss:[ebp-10]

005FA50C   .  50            push eax

005FA50D   .  8B45 F8       mov eax,dword ptr ss:[ebp-8]

005FA510   .  E8 DBA9E0FF   call SSQDYJ.00404EF0              ;  strlen( szReg ) ( 此时 szReg 中的字母已经被去除 )

005FA515   .  8BC8          mov ecx,eax

005FA517   .  49            dec ecx

005FA518   .  BA 01000000   mov edx,1

005FA51D   .  8B45 F8       mov eax,dword ptr ss:[ebp-8]

005FA520   .  E8 23ACE0FF   call SSQDYJ.00405148              ;  取 szReg 在最后一位字符"前面"的字符

005FA525   .  8B45 F0       mov eax,dword ptr ss:[ebp-10]

005FA528   .  E8 9FFAE0FF   call SSQDYJ.00409FCC              ;  转换为 64 位整数 \_\_int64

005FA52D   .  52            push edx                          ; /Arg2

005FA52E   .  50            push eax                          ; |Arg1

005FA52F   .  8D55 E0       lea edx,dword ptr ss:[ebp-20]     ; |

005FA532   .  B8 0C000000   mov eax,0C                        ; |转换结果的长度是 12 位

005FA537   .  E8 DCF9E0FF   call SSQDYJ.00409F18              ; \将 \_\_int64 转换成 16 进制字符串

    -------------------------------------------------------------------------------------

    以上代码是去掉注册码 `szReg` 中的字母，并且去掉最后一个数值，留下来的数字继续向下验证

    我输入的假码是 987a654b987c654d，经过处理后，`szReg` = "98765498765"

    98765498765 转为16进制文本得到 "00A6E1236B9AF7" 后面会用到，也是 `F2`(注册码)的部分代码

    -------------------------------------------------------------------------------------

005FA53C   .  8B55 E0       mov edx,dword ptr ss:[ebp-20]

005FA53F   .  8D45 F0       lea eax,dword ptr ss:[ebp-10]

005FA542   .  E8 89A7E0FF   call SSQDYJ.00404CD0              ;  ...

005FA547   .  8D4D E8       lea ecx,dword ptr ss:[ebp-18]

005FA54A   .  33D2          xor edx,edx

005FA54C   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA54F   .  E8 2CE4FFFF   call SSQDYJ.005F8980              ;  机器码通过异或运算运算得到文本 szKey = "6962>:514;4",跟进1

005FA554   .  8D4D EC       lea ecx,dword ptr ss:[ebp-14]

005FA557   .  8B55 E8       mov edx,dword ptr ss:[ebp-18]

005FA55A   .  8B45 F0       mov eax,dword ptr ss:[ebp-10]

005FA55D   .  E8 16E2FFFF   call SSQDYJ.005F8778              ;  需要跟进的关键调用，得到字符串 str1，跟进2

005FA562   .  8D45 D8       lea eax,dword ptr ss:[ebp-28]

005FA565   .  50            push eax

005FA566   .  B9 06000000   mov ecx,6

005FA56B   .  BA 01000000   mov edx,1

005FA570   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA573   .  E8 D0ABE0FF   call SSQDYJ.00405148              ;  szMac 的 1 到 6 位 "613262"

005FA578   .  8B45 D8       mov eax,dword ptr ss:[ebp-28]

005FA57B   .  E8 D8F9E0FF   call SSQDYJ.00409F58              ;  十进制文本转换为数值1

005FA580   .  8BD8          mov ebx,eax

005FA582   .  8D45 D4       lea eax,dword ptr ss:[ebp-2C]

005FA585   .  50            push eax

005FA586   .  B9 06000000   mov ecx,6

005FA58B   .  BA 07000000   mov edx,7

005FA590   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA593   .  E8 B0ABE0FF   call SSQDYJ.00405148              ;  机器码的 7 到 12 位 "10431"

005FA598   .  8B45 D4       mov eax,dword ptr ss:[ebp-2C]

005FA59B   .  E8 B8F9E0FF   call SSQDYJ.00409F58              ;  十进制文本转换为数值2

005FA5A0   .  03D8          add ebx,eax                       ;  数值1 + 数值2 即 613262 + 10431 = 623693

005FA5A2   .  8BC3          mov eax,ebx

005FA5A4   .  8D55 DC       lea edx,dword ptr ss:[ebp-24]

005FA5A7   .  E8 CCF8E0FF   call SSQDYJ.00409E78              ;  相加结果转换回十进制文本 str2 = "623693"

005FA5AC   .  8B55 DC       mov edx,dword ptr ss:[ebp-24]

005FA5AF   .  8D45 FC       lea eax,dword ptr ss:[ebp-4]

005FA5B2   .  E8 19A7E0FF   call SSQDYJ.00404CD0              ;  ...

005FA5B7   .  8D45 FC       lea eax,dword ptr ss:[ebp-4]

005FA5BA   .  50            push eax

005FA5BB   .  B9 06000000   mov ecx,6

005FA5C0   .  BA 01000000   mov edx,1

005FA5C5   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA5C8   .  E8 7BABE0FF   call SSQDYJ.00405148              ;  取 str2 的 1 到 6 位 "623693"

005FA5CD   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

005FA5D0   .  E8 1BA9E0FF   call SSQDYJ.00404EF0              ;  strlen( str2 ) 这里应该等于 6

005FA5D5   .  8BD8          mov ebx,eax

005FA5D7   .  43            inc ebx

005FA5D8   .  83FB 06       cmp ebx,6

005FA5DB   .  7F 16         jg short SSQDYJ.005FA5F3

005FA5DD   >  8D45 FC       lea eax,dword ptr ss:[ebp-4]

005FA5E0   .  8B4D FC       mov ecx,dword ptr ss:[ebp-4]

005FA5E3   .  BA E0A65F00   mov edx,SSQDYJ.005FA6E0

005FA5E8   .  E8 4FA9E0FF   call SSQDYJ.00404F3C              ;  str2 没有 6 位则在前面添加 0

005FA5ED   .  43            inc ebx

005FA5EE   .  83FB 07       cmp ebx,7

005FA5F1   .^ 75 EA         jnz short SSQDYJ.005FA5DD

005FA5F3      8B55 EC       mov edx,dword ptr ss:[ebp-14]     ;  str1( 由于插入了随机位置，所以每次跟踪会得到不同的结果，详见后面 )

005FA5F6   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]      ;  str2 = "623693"

005FA5F9   .  E8 2EACE0FF   call SSQDYJ.0040522C              ;  返回 str1 中 str2 的位置（失败返回 0）

005FA5FE   .  8BD8          mov ebx,eax

005FA600   .  85DB          test ebx,ebx

005FA602   .  7E 6B         jle short SSQDYJ.005FA66F         ;  关键跳

005FA604   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]

    ---------------------------------------------------------------------------------------------------------------------------

    上面代码有两行需要跟进：

    005FA54F   .  E8 2CE4FFFF   call SSQDYJ.005F8980              ;  机器码通过异或运算运算得到文本 `szKey` = "6962>:514;4",跟进1

    005FA55D   .  E8 16E2FFFF   call SSQDYJ.005F8778              ;  需要跟进的关键调用，得到字符串 `str1`，跟进2

    等下说说这两个跟进，看看是如何得到 `str1` 的。

    继续向下看，看到了 `F1`( 机器码 ) 的处理：

    大致过程这样：我的机器码 "61326210431"

    机器码的前 6 位 + 接下来的 6 位 即 613236 + 10431 = 623693，然后再取前 6 位得到 `str2` = '623693'

    ---------------------------------------------------------------------------------------------------------------------------

下面看下两个跟进，看看如何得到 `str1`：

    ---------------------------------------------------------------------------------------------------------------------------

    005FA54F   .  E8 2CE4FFFF   call SSQDYJ.005F8980              ;  机器码通过异或运算运算得到文本 `szKey` = "6962>:514;4",跟进1

    ---------------------------------------------------------------------------------------------------------------------------

```
005F89CC  |.  8D45 F8       lea eax,dword ptr ss:[ebp-8]
```
