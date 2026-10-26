
```asm
0064BF3B     |.  8BF9            mov edi,ecx

0064BF3D     |.  8955 FC         mov dword ptr ss:[ebp-4],edx

0064BF40     |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]                        ;  用户名

0064BF43     |.  E8 C08FDBFF     call jxc.00404F08

0064BF48     |.  33C0            xor eax,eax

0064BF4A     |.  55              push ebp

0064BF4B     |.  68 E5C06400     push jxc.0064C0E5

0064BF50     |.  64:FF30         push dword ptr fs:[eax]

0064BF53     |.  64:8920         mov dword ptr fs:[eax],esp

0064BF56     |.  8BC7            mov eax,edi

0064BF58     |.  E8 FB8ADBFF     call jxc.00404A58

0064BF5D     |.  8B45 FC         mov eax,dword ptr ss:[ebp-4]

0064BF60     |.  E8 B38DDBFF     call jxc.00404D18                                   ;  用户名长度

0064BF65     |.  8BF0            mov esi,eax

0064BF67     |.  85F6            test esi,esi

0064BF69     |.  7E 26           jle short jxc.0064BF91

0064BF6B     |.  BB 01000000     mov ebx,1

0064BF70     |>  8D4D EC         /lea ecx,dword ptr ss:[ebp-14]

0064BF73     |.  8B45 FC         |mov eax,dword ptr ss:[ebp-4]                       ;  用户名kyc

0064BF76     |.  0FB64418 FF     |movzx eax,byte ptr ds:[eax+ebx-1]                  ;  eax=name[i]

0064BF7B     |.  33D2            |xor edx,edx                                        ;  edx=0

0064BF7D     |.  E8 E2E1DBFF     |call jxc.0040A164

0064BF82     |.  8B55 EC         |mov edx,dword ptr ss:[ebp-14]

0064BF85     |.  8D45 F8         |lea eax,dword ptr ss:[ebp-8]

0064BF88     |.  E8 938DDBFF     |call jxc.00404D20

0064BF8D     |.  43              |inc ebx

0064BF8E     |.  4E              |dec esi

0064BF8F     |.^ 75 DF           \jnz short jxc.0064BF70                             ;  以上循环是提取用户名的ASCII

0064BF91     |>  8B45 F8         mov eax,dword ptr ss:[ebp-8]                        ;  用户名的ASCII=6B7963

0064BF94     |.  E8 7F8DDBFF     call jxc.00404D18                                   ;  用户名的ASCII长度

0064BF99     |.  8BF0            mov esi,eax

0064BF9B     |.  85F6            test esi,esi

0064BF9D     |.  7E 2C           jle short jxc.0064BFCB

0064BF9F     |.  BB 01000000     mov ebx,1                                           ;  EBX=1

0064BFA4     |>  8B45 F8         /mov eax,dword ptr ss:[ebp-8]                       ;  用户名的ASCII

0064BFA7     |.  E8 6C8DDBFF     |call jxc.00404D18                                  ;  EAX=用户名的ASCII长度

0064BFAC     |.  2BC3            |sub eax,ebx                                        ;  EAX=EAX-EBX

0064BFAE     |.  8B55 F8         |mov edx,dword ptr ss:[ebp-8]

0064BFB1     |.  8A1402          |mov dl,byte ptr ds:[edx+eax]

0064BFB4     |.  8D45 E8         |lea eax,dword ptr ss:[ebp-18]

0064BFB7     |.  E8 848CDBFF     |call jxc.00404C40

0064BFBC     |.  8B55 E8         |mov edx,dword ptr ss:[ebp-18]

0064BFBF     |.  8D45 F4         |lea eax,dword ptr ss:[ebp-C]

0064BFC2     |.  E8 598DDBFF     |call jxc.00404D20

0064BFC7     |.  43              |inc ebx

0064BFC8     |.  4E              |dec esi                                            ;  3697B6

0064BFC9     |.^ 75 D9           \jnz short jxc.0064BFA4                             ;  以上循环是对用户名的ASCII进行倒序排列

0064BFCB     |>  8D45 F8         lea eax,dword ptr ss:[ebp-8]

0064BFCE     |.  50              push eax

0064BFCF     |.  B9 04000000     mov ecx,4

0064BFD4     |.  BA 01000000     mov edx,1

0064BFD9     |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]

0064BFDC     |.  E8 978FDBFF     call jxc.00404F78

0064BFE1     |.  8D45 F4         lea eax,dword ptr ss:[ebp-C]

0064BFE4     |.  50              push eax

0064BFE5     |.  B9 04000000     mov ecx,4

0064BFEA     |.  BA 05000000     mov edx,5

0064BFEF     |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]

0064BFF2     |.  E8 818FDBFF     call jxc.00404F78

0064BFF7     |.  8B45 F8         mov eax,dword ptr ss:[ebp-8]

0064BFFA     |.  E8 198DDBFF     call jxc.00404D18                                   ;  用户名的ASCII进行倒序取前4个字符

0064BFFF     |.  83F8 04         cmp eax,4

0064C002     |.  7D 2F           jge short jxc.0064C033

0064C004     |.  8B45 F8         mov eax,dword ptr ss:[ebp-8]

0064C007     |.  E8 0C8DDBFF     call jxc.00404D18

0064C00C     |.  8BD8            mov ebx,eax

0064C00E     |.  83FB 03         cmp ebx,3

0064C011     |.  7F 20           jg short jxc.0064C033

0064C013     |>  8D4D E4         /lea ecx,dword ptr ss:[ebp-1C]

0064C016     |.  8BC3            |mov eax,ebx

0064C018     |.  C1E0 02         |shl eax,2

0064C01B     |.  33D2            |xor edx,edx

0064C01D     |.  E8 42E1DBFF     |call jxc.0040A164

0064C022     |.  8B55 E4         |mov edx,dword ptr ss:[ebp-1C]

0064C025     |.  8D45 F8         |lea eax,dword ptr ss:[ebp-8]

0064C028     |.  E8 F38CDBFF     |call jxc.00404D20

0064C02D     |.  43              |inc ebx

0064C02E     |.  83FB 04         |cmp ebx,4

0064C031     |.^ 75 E0           \jnz short jxc.0064C013

0064C033     |>  8B45 F4         mov eax,dword ptr ss:[ebp-C]

0064C036     |.  E8 DD8CDBFF     call jxc.00404D18                                   ;  取前4个字符

0064C03B     |.  83F8 04         cmp eax,4

0064C03E     |.  7D 2F           jge short jxc.0064C06F

0064C040     |.  8B45 F4         mov eax,dword ptr ss:[ebp-C]

0064C043     |.  E8 D08CDBFF     call jxc.00404D18

0064C048     |.  8BD8            mov ebx,eax

0064C04A     |.  83FB 03         cmp ebx,3

0064C04D     |.  7F 20           jg short jxc.0064C06F

0064C04F     |>  8D4D E0         /lea ecx,dword ptr ss:[ebp-20]

0064C052     |.  8BC3            |mov eax,ebx

0064C054     |.  C1E0 02         |shl eax,2

0064C057     |.  33D2            |xor edx,edx

0064C059     |.  E8 06E1DBFF     |call jxc.0040A164

0064C05E     |.  8B55 E0         |mov edx,dword ptr ss:[ebp-20]

0064C061     |.  8D45 F4         |lea eax,dword ptr ss:[ebp-C]

0064C064     |.  E8 B78CDBFF     |call jxc.00404D20

0064C069     |.  43              |inc ebx

0064C06A     |.  83FB 04         |cmp ebx,4

0064C06D     |.^ 75 E0           \jnz short jxc.0064C04F

0064C06F     |>  8D45 F0         lea eax,dword ptr ss:[ebp-10]

0064C072     |.  BA FCC06400     mov edx,jxc.0064C0FC                                ;  ASCII "JXCw268d58k"

0064C077     |.  E8 748ADBFF     call jxc.00404AF0

0064C07C     |.  8D45 DC         lea eax,dword ptr ss:[ebp-24]

0064C07F     |.  50              push eax

0064C080     |.  B9 04000000     mov ecx,4

0064C085     |.  BA 01000000     mov edx,1

0064C08A     |.  8B45 F0         mov eax,dword ptr ss:[ebp-10]

0064C08D     |.  E8 E68EDBFF     call jxc.00404F78

0064C092     |.  FF75 DC         push dword ptr ss:[ebp-24]

0064C095     |.  68 10C16400     push jxc.0064C110

0064C09A     |.  FF75 F8         push dword ptr ss:[ebp-8]

0064C09D     |.  8D45 D8         lea eax,dword ptr ss:[ebp-28]

0064C0A0     |.  50              push eax

0064C0A1     |.  B9 05000000     mov ecx,5

0064C0A6     |.  BA 05000000     mov edx,5

0064C0AB     |.  8B45 F0         mov eax,dword ptr ss:[ebp-10]                       ;  固定字符串=JXCw268d58k

0064C0AE     |.  E8 C58EDBFF     call jxc.00404F78

0064C0B3     |.  FF75 D8         push dword ptr ss:[ebp-28]

0064C0B6     |.  68 10C16400     push jxc.0064C110

0064C0BB     |.  FF75 F4         push dword ptr ss:[ebp-C]

0064C0BE     |.  8BC7            mov eax,edi

0064C0C0     |.  BA 06000000     mov edx,6

0064C0C5     |.  E8 0E8DDBFF     call jxc.00404DD8

0064C0CA     |.  33C0            xor eax,eax

0064C0CC     |.  5A              pop edx

0064C0CD     |.  59              pop ecx

0064C0CE     |.  59              pop ecx

0064C0CF     |.  64:8910         mov dword ptr fs:[eax],edx

0064C0D2     |.  68 ECC06400     push jxc.0064C0EC

/////////////////////////////////////////////////////////////////////////////////////////
```

总结：算法比较简单，主要利用用户名的ASCII生成字符串后和字符串常量`"JXCw268d58k"`进行插入，连接形成注册码。

具体实现看注册机源码。

```cpp
// 进销存软件.cpp : 定义控制台应用程序的入口点。
//
#include "stdafx.h"

using namespace std;
```
