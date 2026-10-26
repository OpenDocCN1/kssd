
# [原创] UltraISO 破解手记

UltraISO 后期的每个版本基本上都差不多，官网上下载的据传是什么加了白名单、红名单的版本，但即使是相同版本的 MD5 值都不同，官网每天都更新。所以我们平时见得最多的就是注册机、注册码，什么李明啊、王健啊等等。现在最新版的是 V9.3.6.2750，好象以前的注册码不能用了，除非你下到的是没名单的裸体版，手上没有，没办法，只好自己动手了。

## 下载最新版本

首先到官网下载最新版本：

[c5aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2\*7j5Y4y4&6M7%4c8W2L8i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8o6q4Q4x3X3g2H3K9s2m8Q4x3@1k6X3K9h3I4W2i4K6y4p5N6h3W2K6L8K6W2Q4y4h3k6U0L8W2)9J5k6h3g2^5k6b7`.`.](https://bbs.kanxue.com/elink@77fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2*7j5Y4y4&6M7%4c8W2L8i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8o6q4Q4x3X3g2H3K9s2m8Q4x3@1k6X3K9h3I4W2i4K6y4p5N6h3W2K6L8K6W2Q4y4h3k6U0L8W2)9J5k6h3g2^5k6b7`.`.)

注意你下到的程序极可能和我的不一样，下面破文地址是不相同的，但大体思路都一样的。只要不太笨，照样画葫芦就行了。闲话少话，转入正文：

## 分析程序

用 PEID 检测了下，ASPACK 的壳，很简单，用 `AspackDie.exe` 一下就解了出来，7.39M。然后 OD 载入，查找所有参考文本字符串，这里可以多个选择，什么 `username`、`registration`、`uikey.ini`、`ultraiso.ini` 等，原因？你以前用个注册版的 ultraiso 都会知道，`username` 是注册表的用户名，`registration` 是注册码，另外两个 INI 是 `KEYFILE`。

`ultraiso` 是既可以注册表注册，也可以 `KEYFILE` 注册的，启动时是先检查有没有 `keyfile`，没有的话从注册表里去找注册码，再没有就变试用了。我们的目的很简单，能拦下来就行，反正它们的代码都在一起。

我用的是 `username` 来断，查找所有的 `username`，F2 都标上断点，F9 运行 OD，在这里断下，注意代码可能不一样：

```asm
00471104  /$  55            push    ebp
00471105  |.  8BEC          mov     ebp, esp
00471107  |.  81C4 C4FEFFFF add     esp, -13C
0047110D  |.  68 E5ED6500   push    0065EDE5                         ;  ASCII "rt"
00471112  |.  FF75 08       push    dword ptr [ebp+8]
00471115  |.  E8 066E1B00   call    00627F20
0047111A  |.  83C4 08       add     esp, 8
0047111D  |.  8945 E4       mov     dword ptr [ebp-1C], eax
00471120  |.  8B45 E4       mov     eax, dword ptr [ebp-1C]
00471123  |.  85C0          test    eax, eax
00471125  |.  75 07         jnz     short 0047112E
00471127  |.  33C0          xor     eax, eax
00471129  |.  E9 D9020000   jmp     00471407
0047112E  |>  8B55 0C       mov     edx, dword ptr [ebp+C]
00471131  |.  C602 00       mov     byte ptr [edx], 0
00471134  |.  8B4D 10       mov     ecx, dword ptr [ebp+10]
00471137  |.  C601 00       mov     byte ptr [ecx], 0
0047113A  |.  E9 85020000   jmp     004713C4
0047113F  |>  8A85 C4FEFFFF /mov     al, byte ptr [ebp-13C]
00471145  |.  8845 FF       |mov     byte ptr [ebp-1], al
00471148  |.  8A55 FF       |mov     dl, byte ptr [ebp-1]
0047114B  |.  80FA 23       |cmp     dl, 23
0047114E  |.  0F84 70020000 |je      004713C4
00471154  |.  8A4D FF       |mov     cl, byte ptr [ebp-1]
00471157  |.  80F9 3B       |cmp     cl, 3B
0047115A  |.  0F84 64020000 |je      004713C4
00471160  |.  8A45 FF       |mov     al, byte ptr [ebp-1]
00471163  |.  3C 0D         |cmp     al, 0D
00471165  |.  0F84 59020000 |je      004713C4
0047116B  |.  8A55 FF       |mov     dl, byte ptr [ebp-1]
0047116E  |.  80FA 0A       |cmp     dl, 0A
00471171  |.  0F84 4D020000 |je      004713C4
00471177  |.  33C9          |xor     ecx, ecx
00471179  |.  894D F0       |mov     dword ptr [ebp-10], ecx
0047117C  |.  EB 03         |jmp     short 00471181
0047117E  |>  FF45 F0       |/inc     dword ptr [ebp-10]
00471181  |>  8B45 F0       | mov     eax, dword ptr [ebp-10]
00471184  |.  0FBE9405 C4FE>||movsx   edx, byte ptr [ebp+eax-13C]
0047118C  |.  83FA 20       ||cmp     edx, 20
0047118F  |.^ 74 ED         ||je      short 0047117E
00471191  |.  8B4D F0       |mov     ecx, dword ptr [ebp-10]
00471194  |.  0FBE840D C4FE>||movsx   eax, byte ptr [ebp+ecx-13C]
0047119C  |.  83F8 08       ||cmp     eax, 8
0047119F  |.^ 74 DD         |\je      short 0047117E
004711A1  |.  33D2          |xor     edx, edx
004711A3  |.  8955 EC       |mov     dword ptr [ebp-14], edx
004711A6  |.  8B4D F0       |mov     ecx, dword ptr [ebp-10]
004711A9  |.  0FBE840D C4FE>|movsx   eax, byte ptr [ebp+ecx-13C]
004711B1  |.  83F8 27       |cmp     eax, 27
004711B4  |.  74 14         |je      short 004711CA
004711B6  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]
004711B9  |.  0FBE8C15 C4FE>|movsx   ecx, byte ptr [ebp+edx-13C]
004711C1  |.  83F9 22       |cmp     ecx, 22
004711C4  |.  0F85 88000000 |jnz     00471252
004711CA  |>  8B45 F0       |mov     eax, dword ptr [ebp-10]
004711CD  |.  8A9405 C4FEFF>|mov     dl, byte ptr [ebp+eax-13C]
004711D4  |.  8855 F7       |mov     byte ptr [ebp-9], dl
004711D7  |.  FF45 F0       |inc     dword ptr [ebp-10]
004711DA  |.  EB 17         |jmp     short 004711F3
004711DC  |>  8B4D F0       |/mov     ecx, dword ptr [ebp-10]
004711DF  |.  8D840D C4FEFF>||lea     eax, dword ptr [ebp+ecx-13C]
004711E6  |.  8B55 EC       ||mov     edx, dword ptr [ebp-14]
004711E9  |.  8A0C10        ||mov     cl, byte ptr [eax+edx]
004711EC  |.  84C9          ||test    cl, cl
004711EE  |.  74 1A         ||je      short 0047120A
004711F0  |.  FF45 EC       ||inc     dword ptr [ebp-14]
004711F3  |>  8B45 F0       | mov     eax, dword ptr [ebp-10]
004711F6  |.  8D9405 C4FEFF>||lea     edx, dword ptr [ebp+eax-13C]
004711FD  |.  8B4D EC       ||mov     ecx, dword ptr [ebp-14]
00471200  |.  8A040A        ||mov     al, byte ptr [edx+ecx]
00471203  |.  8A55 F7       ||mov     dl, byte ptr [ebp-9]
00471206  |.  3AC2          ||cmp     al, dl
00471208  |.^ 75 D2         |\jnz     short 004711DC
0047120A  |>  8B4D F0       |mov     ecx, dword ptr [ebp-10]
0047120D  |.  8D840D C4FEFF>|lea     eax, dword ptr [ebp+ecx-13C]
00471214  |.  8B55 EC       |mov     edx, dword ptr [ebp-14]
00471217  |.  8A0C10        |mov     cl, byte ptr [eax+edx]
0047121A  |.  8A45 F7       |mov     al, byte ptr [ebp-9]
0047121D  |.  3AC8          |cmp     cl, al
0047121F  |.  0F85 9F010000 |jnz     004713C4
00471225  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]
00471228  |.  8D8C15 C4FEFF>|lea     ecx, dword ptr [ebp+edx-13C]
0047122F  |.  8B45 EC       |mov     eax, dword ptr [ebp-14]
00471232  |.  C60401 00     |mov     byte ptr [ecx+eax], 0
00471236  |.  FF45 EC       |inc     dword ptr [ebp-14]
00471239  |.  EB 17         |jmp     short 00471252
0047123B  |>  8B55 F0       |/mov     edx, dword ptr [ebp-10]
0047123E  |.  8D8C15 C4FEFF>||lea     ecx, dword ptr [ebp+edx-13C]
00471245  |.  8B45 EC       ||mov     eax, dword ptr [ebp-14]
00471248  |.  8A1401        ||mov     dl, byte ptr [ecx+eax]
0047124B  |.  84D2          ||test    dl, dl
0047124D  |.  74 19         ||je      short 00471268
0047124F  |.  FF45 EC       ||inc     dword ptr [ebp-14]
00471252  |>  8B4D F0       | mov     ecx, dword ptr [ebp-10]
00471255  |.  8D840D C4FEFF>||lea     eax, dword ptr [ebp+ecx-13C]
0047125C  |.  8B55 EC       ||mov     edx, dword ptr [ebp-14]
0047125F  |.  0FBE0C10      ||movsx   ecx, byte ptr [eax+edx]
00471263  |.  83F9 3D       ||cmp     ecx, 3D
00471266  |.^ 75 D3         |\jnz     short 0047123B
00471268  |>  8B45 F0       |mov     eax, dword ptr [ebp-10]
0047126B  |.  8D9405 C4FEFF>|lea     edx, dword ptr [ebp+eax-13C]
00471272  |.  8B4D EC       |mov     ecx, dword ptr [ebp-14]
00471275  |.  0FBE040A      |movsx   eax, byte ptr [edx+ecx]
00471279  |.  83F8 3D       |cmp     eax, 3D
0047127C  |.  0F85 42010000 |jnz     004713C4
00471282  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]
00471285  |.  8D8C15 C4FEFF>|lea     ecx, dword ptr [ebp+edx-13C]
0047128C  |.  8B45 EC       |mov     eax, dword ptr [ebp-14]
0047128F  |.  C60401 00     |mov     byte ptr [ecx+eax], 0
00471293  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]
00471296  |.  8B4D EC       |mov     ecx, dword ptr [ebp-14]
00471299  |.  03D1          |add     edx, ecx
0047129B  |.  8D85 C5FEFFFF |lea     eax, dword ptr [ebp-13B]
004712A1  |.  03D0          |add     edx, eax
004712A3  |.  8955 F8       |mov     dword ptr [ebp-8], edx
004712A6  |.  8B55 F8       |mov     edx, dword ptr [ebp-8]
004712A9  |.  0FBE0A        |movsx   ecx, byte ptr [edx]
004712AC  |.  83F9 27       |cmp     ecx, 27
004712AF  |.  74 0F         |je      short 004712C0
004712B1  |.  8B45 F8       |mov     eax, dword ptr [ebp-8]
004712B4  |.  0FBE10        |movsx   edx, byte ptr [eax]
004712B7  |.  83FA 22       |cmp     edx, 22
```
