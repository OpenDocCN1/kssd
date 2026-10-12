
:0044D53E 03D8                    `add ebx, eax`

:0044D540 83EB33                  `sub ebx, 00000033`

:0044D543 8BC3                    `mov eax, ebx`

:0044D545 03C0                    `add eax, eax`

:0044D547 8D0480                  `lea eax, dword ptr [eax+4*eax]`

:0044D54A 8BD8                    `mov ebx, eax`

:0044D54C 8B45FC                  `mov eax, dword ptr [ebp-04]`

:0044D54F 8A5005                  `mov dl, byte ptr [eax+05]`

:0044D552 8BCF                    `mov ecx, edi`

:0044D554 8B45F8                  `mov eax, dword ptr [ebp-08]`

:0044D557 E890F9FFFF              `call 0044CEEC`

:0044D55C 03D8                    `add ebx, eax`

:0044D55E 83EB33                  `sub ebx, 00000033`

:0044D561 8BFE                    `mov edi, esi`==============>>>>

//=================================== 0   0   0   0   1   5

//求索引和；表示将检查的长度，分为：  十万 万位 千位 百位 十位 个位

//                    idx1 idx2 idx3 idx4 idx5 idx6

//                    2    2    2    2    f    h

//===========================

:0044D563 03FF                    `add edi, edi`

:0044D565 8D3CBF                  `lea edi, dword ptr [edi+4*edi]`

:0044D568 8D45F0                  `lea eax, dword ptr [ebp-10]`

:0044D56B 8BD3                    `mov edx, ebx`======>必须等于序列好算出来的数字的长度

:0044D56D E8426BFBFF              `call 004040B4`

:0044D572 85DB                    `test ebx, ebx`======>必须等于序列好算出来的数字的长度

:0044D574 0F8EA2000000            `jle 0044D61C`

:0044D57A 895DEC                  `mov dword ptr [ebp-14], ebx`

:0044D57D BE01000000              `mov esi, 00000001`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044D616(C)

|

//===========================================================

`for index:=1 to lenCode do`
`begin`
处理输入的注册码
`end;`

//===================================================================

:0044D582 8D0476                  `lea eax, dword ptr [esi+2*esi]`

:0044D585 8B55FC                  `mov edx, dword ptr [ebp-04]`

:0044D588 8A540203                `mov dl, byte ptr [edx+eax+03]`

:0044D58C 8BCF                    `mov ecx, edi`

:0044D58E 8B45F8                  `mov eax, dword ptr [ebp-08]`

`JDqpBkLvn8Cm0HYIr1KQMSagURlieFAEb3jd7N4zwZoOGyxu6T2fstchVP9X5WwNC3OHDlm1efyqzcbM4GrZjP9TQ7gk2ov8F0IitXSJaAEBYxKunVL5Uhs6pRWd`

:0044D591 E856F9FFFF              `call 0044CEEC`

:0044D596 8BD8                    `mov ebx, eax`

:0044D598 2BDF                    `sub ebx, edi`

:0044D59A 8BC6                    `mov eax, esi`

:0044D59C 48                      `dec eax`

:0044D59D B90A000000              `mov ecx, 0000000A`

:0044D5A2 99                      `cdq`

:0044D5A3 F7F9                    `idiv ecx`

:0044D5A5 42                      `inc edx`

:0044D5A6 2BDA                    `sub ebx, edx`

:0044D5A8 8BC3                    `mov eax, ebx`

:0044D5AA 03C0                    `add eax, eax`

:0044D5AC 8D0480                  `lea eax, dword ptr [eax+4*eax]`

:0044D5AF 8BD8                    `mov ebx, eax`

//=======================================(fpos(MagicStr,edi,strCode[])-((index  mod 10)+1))*10

//==========fpos 函数实际如下

`function fpos(const str:string;startIndex:integer;tag:char):integer;`
`var`
`index,len:integer;`
`mmeax:integer;`
`begin`
`len:=length(str);`
`mmeax:=-1;`
`if startIndex>len then`
`begin`
`//index:=startIndex-1;`
`mmeax:=startIndex-1;`
`repeat`
`inc(mmeax);`
`until ((str[mmeax]=tag) or (mmeax=len));`
`end;`
`fpos:=mmeax;`
`end;`

//===========================================================================

:0044D5B1 8D0476                  `lea eax, dword ptr [esi+2*esi]`

:0044D5B4 8B55FC                  `mov edx, dword ptr [ebp-04]`

:0044D5B7 8A540204                `mov dl, byte ptr [edx+eax+04]`

:0044D5BB 8BCF                    `mov ecx, edi`

:0044D5BD 8B45F8                  `mov eax, dword ptr [ebp-08]`

:0044D5C0 E827F9FFFF              `call 0044CEEC`

:0044D5C5 03D8                    `add ebx, eax`

:0044D5C7 2BDF                    `sub ebx, edi`

:0044D5C9 8BC6                    `mov eax, esi`

:0044D5CB 48                      `dec eax`

:0044D5CC B90A000000              `mov ecx, 0000000A`

:0044D5D1 99                      `cdq`

:0044D5D2 F7F9                    `idiv ecx`

:0044D5D4 42                      `inc edx`

//====================================(第几个注册码  mod 10+1)

:0044D5D5 2BDA                    `sub ebx, edx`

:0044D5D7 8BC3                    `mov eax, ebx`

:0044D5D9 03C0                    `add eax, eax`

:0044D5DB 8D0480                  `lea eax, dword ptr [eax+4*eax]`

:0044D5DE 8BD8                    `mov ebx, eax`

:0044D5E0 8D0476                  `lea eax, dword ptr [esi+2*esi]`

:0044D5E3 8B55FC                  `mov edx, dword ptr [ebp-04]`

:0044D5E6 8A540205                `mov dl, byte ptr [edx+eax+05]`

:0044D5EA 8BCF                    `mov ecx, edi`

:0044D5EC 8B45F8                  `mov eax, dword ptr [ebp-08]`

:0044D5EF E8F8F8FFFF              `call 0044CEEC`

:0044D5F4 03D8                    `add ebx, eax`

:0044D5F6 2BDF                    `sub ebx, edi`===>索引=1-124，-1

\=================================================> 1-62

\=================================================>62-124

//由连续的 3 个注册码计算出： 百位 十位 个位

//            idx1 idx2 idx3

//0x30-0x39=48-57

\================================================>

:0044D5F8 8BC6                    `mov eax, esi`

:0044D5FA 48                      `dec eax`

:0044D5FB B90A000000              `mov ecx, 0000000A`

:0044D600 99                      `cdq`

:0044D601 F7F9                    `idiv ecx`

:0044D603 42                      `inc edx`=((index-1) mod 10)+1;

//===========================================================

//                    index=[1-lenCode]

//===========================================================

//========mpos(MagicStr,edi,strCode[])-((index  mod 10)+1)

//

//====================================(第几个注册码 mod 10+1)

//===========================================================

:0044D604 2BDA                    `sub ebx, edx`

:0044D606 8D45F0                  `lea eax, dword ptr [ebp-10]`

:0044D609 E8A269FBFF              `call 00403FB0`

:0044D60E 885C30FF                `mov byte ptr [eax+esi-01], bl`

//===============================================

//==================>这个 bl 一定是在 0x30-0x39 之内，应该可以简化还原

//===============================================

:0044D612 46                      `inc esi`

:0044D613 FF4DEC                  `dec [ebp-14]`=========>循环注册码程度

:0044D616 0F8566FFFFFF            `jne 0044D582`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044D574(C)

|

:0044D61C 8B45F4                  `mov eax, dword ptr [ebp-0C]`

:0044D61F 8B55F0                  `mov edx, dword ptr [ebp-10]`

:0044D622 E8D565FBFF              `call 00403BFC`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044D4AB(U)

|

:0044D627 33C0                    `xor eax, eax`

:0044D629 5A                      `pop edx`

:0044D62A 59                      `pop ecx`

:0044D62B 59                      `pop ecx`

:0044D62C 648910                  `mov dword ptr fs:[eax], edx`

:0044D62F 6851D64400              `push 0044D651`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0044D64F(U)

|

:0044D634 8D45F0                  `lea eax, dword ptr [ebp-10]`

:0044D637 E82865FBFF              `call 00403B64`

:0044D63C 8D45F8                  `lea eax, dword ptr [ebp-08]`
