
# Talisman Desktop v2.6

> 原文：[`www.pediy.com/kssd/pediy05/pediy50609.htm`](https://www.pediy.com/kssd/pediy05/pediy50609.htm)

破解者：`HMILY[CCG][BCG]`

软件名称：`Talisman Desktop v2.6`

软件下载：`http://www.lighttek.com`

说明：这个软件从一开始到现在，算法都没有改过。以前的版本就有人分析过了，

呵呵，这次新版本，新地址我再分析一下，然后加上注册机源码！！！

软件没有加壳，`Delphi` 写的，反汇编后迅速找到 `usercode`，再向上一点，就到了注册计算的核心了

```
:00484B8C E833F2F7FF          call 00403DC4 ->取注册名位数

:00484B91 85C0                test eax, eax

:00484B93 7E13                jle 00484BA8

:00484B95 BA01000000          mov edx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00484BA6(C)

|

:00484B9A 8B4DF8              mov ecx, dword ptr [ebp-08] ->注册名传入 ecx

:00484B9D 0FB64C11FF          movzx ecx, byte ptr [ecx+edx-01] ->依次取注册名

:00484BA2 03F1                add esi, ecx                ->注册名累加

:00484BA4 42                  inc edx                     ->累加器

:00484BA5 48                  dec eax                     ->位数--

:00484BA6 75F2                jne 00484B9A                ->未取完，继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00484B93(C)

|

:00484BA8 8975EC              mov dword ptr [ebp-14], esi

:00484BAB DB45EC              fild dword ptr [ebp-14]

:00484BAE E8A1DEF7FF          call 00402A54  ->取累加结果

:00484BB3 69C009030000        imul eax, 00000309 ->累加结果*0x309

:00484BB9 8BF0                mov esi, eax ->相乘结果传入 esi 下命令? eax 就是真码

:00484BBB 3B75FC              cmp esi, dword ptr [ebp-04]

:00484BBE 0F85BF000000        jne 00484C83 ->注册码不相等，over！

:00484BC4 B201                mov dl, 01

:00484BC6 A168F34400          mov eax, dword ptr [0044F368]

:00484BCB E8D8A8FCFF          call 0044F4A8

:00484BD0 8BF0                mov esi, eax

:00484BD2 B101                mov cl, 01

* Possible StringData Ref from Code Obj ->"\Software\Lighttek\Talisman"

                        |

:00484BD4 BAE04C4800          mov edx, 00484CE0

:00484BD9 8BC6                mov eax, esi

:00484BDB E8BCAAFCFF          call 0044F69C

:00484BE0 8D55F4              lea edx, dword ptr [ebp-0C]

:00484BE3 8B83E4020000        mov eax, dword ptr [ebx+000002E4]

:00484BE9 E80A67FAFF          call 0042B2F8

:00484BEE 8B4DF4              mov ecx, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"username"

                        |

:00484BF1 BA044D4800          mov edx, 00484D04

:00484BF6 8BC6                mov eax, esi

:00484BF8 E8ABB0FCFF          call 0044FCA8

:00484BFD 8D55F4              lea edx, dword ptr [ebp-0C]

:00484C00 8B83E0020000        mov eax, dword ptr [ebx+000002E0]

:00484C06 E8ED66FAFF          call 0042B2F8

:00484C0B 8B45F4              mov eax, dword ptr [ebp-0C]

:00484C0E E8E536F8FF          call 004082F8

:00484C13 8945EC              mov dword ptr [ebp-14], eax

:00484C16 DB45EC              fild dword ptr [ebp-14]

:00484C19 83C4F8              add esp, FFFFFFF8

:00484C1C DD1C24              fstp qword ptr [esp]

:00484C1F 9B                  wait

* Possible StringData Ref from Code Obj ->"usercode"

                        |

:00484C20 BA184D4800          mov edx, 00484D18

:00484C25 8BC6                mov eax, esi

:00484C27 E8B0B1FCFF          call 0044FDDC

:00484C2C 8BC6                mov eax, esi

:00484C2E E8DDA8FCFF          call 0044F510

:00484C33 8BC6                mov eax, esi

:00484C35 E8BAE2F7FF          call 00402EF4

:00484C3A 8D55F4              lea edx, dword ptr [ebp-0C]

:00484C3D 8B83E4020000        mov eax, dword ptr [ebx+000002E4]

:00484C43 E8B066FAFF          call 0042B2F8

:00484C48 8B4DF4              mov ecx, dword ptr [ebp-0C]

:00484C4B 8D45E8              lea eax, dword ptr [ebp-18]

* Possible StringData Ref from Code Obj ->"Registered for "

                        |

:00484C4E BA2C4D4800          mov edx, 00484D2C

:00484C53 E8B8F1F7FF          call 00403E10

:00484C58 8B55E8              mov edx, dword ptr [ebp-18]

:00484C5B 8B83E8020000        mov eax, dword ptr [ebx+000002E8]

:00484C61 E8C266FAFF          call 0042B328

:00484C66 BA2C010000          mov edx, 0000012C

:00484C6B A168694C00          mov eax, dword ptr [004C6968]

:00484C70 E8B35EFAFF          call 0042AB28

:00484C75 A16C564B00          mov eax, dword ptr [004B566C]

:00484C7A 8B00                mov eax, dword ptr [eax]

:00484C7C 33D2                xor edx, edx

:00484C7E 89500C              mov dword ptr [eax+0C], edx

:00484C81 EB1A                jmp 00484C9D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00484BBE(C)

|

:00484C83 B201                mov dl, 01

:00484C85 8B83FC020000        mov eax, dword ptr [ebx+000002FC]

:00484C8B E88065FAFF          call 0042B210

:00484C90 B201                mov dl, 01

:00484C92 8B8300030000        mov eax, dword ptr [ebx+00000300]

:00484C98 E87365FCFF          call 0044B210

```

============================================================================

## TC v2.0 源码

```c
#include "stdio.h"

main()
{
    int a;
    unsigned char name[60];
    unsigned long b=0, c=0;

    printf("This Keygen by HMILY[CCG][BCG]\n");
    printf("My QQ:5289322  E-main:gyyxll@21cn.com\n");
    printf("Enter your register name: ");
    gets(name);

    for(a=0; name[a]!='\0'; a++)
    {
        b = b + name[a];
    }

    c = b * 0x309;
    printf("Your register code is   : %ld\n", c);
    printf("Good Luck!!!\n");
}
```
