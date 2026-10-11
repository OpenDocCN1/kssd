
# 汇编代码分析

```
:004AF3F2 8A5202                  mov dl, byte ptr [edx+02]
                                  ==== > 取试炼码的第 3 位！DL=5

:004AF3F5 E8B649F5FF              call 00403DB0

:004AF3FA 8B45D0                  mov eax, dword ptr [ebp-30]

:004AF3FD E89A9CF5FF              call 0040909C

:004AF402 8BD8                    mov ebx, eax
                                  ==== > 5 入 EBX

:004AF404 8D45CC                  lea eax, dword ptr [ebp-34]

:004AF407 8B16                    mov edx, dword ptr [esi]
                                  ==== > EDX=13572468901234567

:004AF409 8A520E                  mov dl, byte ptr [edx+0E]
                                  ==== > 取试炼码的第 15 位！DL=5

:004AF40C E89F49F5FF              call 00403DB0

:004AF411 8B45CC                  mov eax, dword ptr [ebp-34]

:004AF414 E8839CF5FF              call 0040909C

:004AF419 0FAFD8                  imul ebx, eax
                                  ==== > EBX=5*5=19（H）
                                  即：上面所取试炼码的第 3 位与试炼码的第 15 位相乘！

:004AF41C 8BC3                    mov eax, ebx

:004AF41E 8D55F4                  lea edx, dword ptr [ebp-0C]

:004AF421 E8D69BF5FF              call 00408FFC

:004AF426 8B45F4                  mov eax, dword ptr [ebp-0C]
                                  ==== > EAX=25 即：19（H）的十进制值

:004AF429 E85A4AF5FF              call 00403E88

:004AF42E 8BD8                    mov ebx, eax

:004AF430 83FB01                  cmp ebx, 00000001

:004AF433 7F13                    jg 004AF448

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF446(C)
|

:004AF435 8D45F4                  lea eax, dword ptr [ebp-0C]

:004AF438 BAD0F44A00              mov edx, 004AF4D0

:004AF43D E84E4AF5FF              call 00403E90

:004AF442 43                      inc ebx

:004AF443 83FB02                  cmp ebx, 00000002

:004AF446 75ED                    jne 004AF435

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF433(C)
|

:004AF448 8B06                    mov eax, dword ptr [esi]
                                  ==== > EAX=13572468901234567

:004AF44A 8A4004                  mov al, byte ptr [eax+04]
                                  ==== > 取试炼码第 5 位 2 入 AL

:004AF44D 8B55F4                  mov edx, dword ptr [ebp-0C]
                                  ==== > EDX=25

:004AF450 3A02                    cmp al, byte ptr [edx]
                                  ==== > 第 5 位 2 与 25 的第一位 2 比较！

:004AF452 750E                    jne 004AF462
                                  ==== > 跳则 OVER！

:004AF454 8B06                    mov eax, dword ptr [esi]
                                  ==== > EAX=13572468901234567

:004AF456 8A400A                  mov al, byte ptr [eax+0A]
                                  ==== > 取试炼码第 11 位 1 入 AL

:004AF459 8B55F4                  mov edx, dword ptr [ebp-0C]

:004AF45C 3A4201                  cmp al, byte ptr [edx+01]
                                  ==== > 第 11 位 1 与 25 的第二位 5 比较！

:004AF45F 7501                    jne 004AF462
                                  ==== > 跳则 OVER！

:004AF461 47                      inc edi
                                  ==== > EDI 增 1=2

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004AF452(C), :004AF45F(C)
|

:004AF462 33C0                    xor eax, eax

:004AF464 5A                      pop edx

:004AF465 59                      pop ecx

:004AF466 59                      pop ecx

:004AF467 648910                  mov dword ptr fs:[eax], edx

:004AF46A EB11                    jmp 004AF47D

:004AF46C E9F73EF5FF              jmp 00403368

:004AF471 E84E42F5FF              call 004036C4

:004AF476 EB0E                    jmp 004AF486

:004AF478 E84742F5FF              call 004036C4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF46A(U)
|

:004AF47D 83FF02                  cmp edi, 00000002
                                  ==== > 比较 EDI 是否=2？

:004AF480 7C04                    jl 004AF486
                                  ==== > 跳则 OVER！

:004AF482 C645FF01                mov [ebp-01], 01
                                  ==== > 1 入 [ebp-01]  注册标志位置 1！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004AF476(U), :004AF480(C)
|

:004AF486 33C0                    xor eax, eax

:004AF488 5A                      pop edx

:004AF489 59                      pop ecx

:004AF48A 59                      pop ecx

:004AF48B 648910                  mov dword ptr fs:[eax], edx

:004AF48E 68BDF44A00              push 004AF4BD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AF4BB(U)
|

:004AF493 8D45CC                  lea eax, dword ptr [ebp-34]

:004AF496 BA02000000              mov edx, 00000002

:004AF49B E88C47F5FF              call 00403C2C

:004AF4A0 8D45E0                  lea eax, dword ptr [ebp-20]

:004AF4A3 E86047F5FF              call 00403C08

:004AF4A8 8D45F0                  lea eax, dword ptr [ebp-10]

:004AF4AB BA03000000              mov edx, 00000003

:004AF4B0 E87747F5FF              call 00403C2C

:004AF4B5 C3                      ret

:004AF4B6 E96141F5FF              jmp 0040361C

:004AF4BB EBD6                    jmp 004AF493

:004AF4BD 8A45FF                  mov al, byte ptr [ebp-01]
                                  ==== > [ebp-01] 入 AL  注册标志位！

:004AF4C0 5F                      pop edi

:004AF4C1 5E                      pop esi

:004AF4C2 5B                      pop ebx

:004AF4C3 8BE5                    mov esp, ebp

:004AF4C5 5D                      pop ebp

:004AF4C6 C3                      ret
```

## 算法总结

1. 注册码至少 `17` 位！多于 `17` 位则只取前 `17` 位运算。试炼码：`13572468901234567`
2. 试炼码的第 `10` 位 `0`、第 `7` 位 `6`，连接起来=`06`，取其 `16` 进制值=`06`
3. 取试炼码的第 `14` 位 `4`、第 `8` 位 `8`、第 `17` 位 `7`、第 `4` 位 `7`，连接起来=`4877`，取其 `16` 进制值=`130D`
4. 上面两次所得结果相乘：`06*130D=724E`   转换成 `10` 进制值=`29262`
5. `29262` 少于 `5` 位，因此其后再接上 `F` =`29262F`  （若上面所得多于 `5` 位则不接）
6. 试炼码第 `13` 位 `3` 与 `29262F` 第 `6` 位 `F` 比较！所以：注册码第 `13` 位=`F`
7. 试炼码第 `1` 位 `1` 与 `29262F` 第 `4` 位 `6` 比较！所以：注册码第 `1` 位=`6`
8. 试炼码第 `12` 位 `2` 与 `29262F` 第 `3` 位 `2` 比较！所以：注册码第 `12` 位=`2`
9. 试炼码第 `16` 位 `6` 与 `29262F` 第 `2` 位 `9` 比较！所以：注册码第 `16` 位=`9`
10. 试炼码第 `2` 位 `3` 与 `29262F` 第 `5` 位 `2` 比较！所以：注册码第 `2` 位=`2`
11. 试炼码第 `9` 位 `9` 与 `29262F` 第 `1` 位 `2` 比较！所以：注册码第 `9` 位=`2`
12. 取试炼码的第 `3` 位 `5` 与第 `15` 位 `5` 相乘=`19`（`H`）=`25`（`D`）
13. 试炼码第 `5` 位 `2` 与 `25` 的第一位 `2` 比较！所以：注册码第 `5` 位=`2`
14. 试炼码第 `11` 位 `1` 与 `25` 的第二位 `5` 比较！所以：注册码第 `11` 位=`5`

综上所述：我所输入的试炼码所对应的注册码=`625724682052F4597`

呵呵，注册码与用户名无关！

## 注册信息保存

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\[www.sangao.com\MagicMailMaker\]]
"UserName"="fly"
"KeyCode"="625724682052F4597"
```

## 整理

- 用户名：`fly` （随意）
- 试炼码：`625724682052F4597`  或者：`085726481055F45978`    很多的……

```
                 Cracked By 巢水工作坊——fly【OCN】
                              23:36 03-3-3  4dd466d0e1e0cc123afbe504da2462ca
```
