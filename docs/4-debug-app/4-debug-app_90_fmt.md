
# OICQ 的密码比较

> 原文：[`www.pediy.com/kssd/pediy03/forum333.htm`](https://www.pediy.com/kssd/pediy03/forum333.htm)

学 `CRACK` 已经一个多月了,想想当初成功破解第一个软件时,激动的我连续两个小时心跳加速,以后的三天晚上没睡好觉,多可笑啊!现在回头再看看,可真是一段美好的回忆!

好了,不说废话了!下面我浅谈一下 `OICQ` 的密码比较过程.

前提是你必须在这台机子上,曾经成功登陆上服务器.而且保留 `matrix.cnt` 文件.

我没技术和时间去反推出密码.如那位 `CRACKER` 成功的话,别忘了告诉我一声,`OK`?

`EMAIL:CL517@YEAH.NET` (可别炸我呀!!!)

希望能交个朋友.

`OICQ:9062494`

如有错误请各位指出.

\* `Possible StringData Ref from Data Obj ->"matrix.cnt"` <===读入这个文件!!!

```
:0044192B 68C8125100               push 005112C8
:00441930 51                       push ecx
:00441931 50                       push eax
:00441932 C645FC05                 mov [ebp-04], 05
:00441936 E87FEF0500               call 004A08BA
:0044193B 8D4DB0                   lea ecx, dword ptr [ebp-50]
:0044193E E892A60200               call 0046BFD5
:00441943 3BC3                     cmp eax, ebx
:00441945 741F                     je 00441966
:00441947 51                       push ecx
:00441948 8D450C                   lea eax, dword ptr [ebp+0C]
:0044194B 8BCC                     mov ecx, esp
:0044194D 8965F0                   mov dword ptr [ebp-10], esp
:00441950 50                       push eax
:00441951 E844EA0500               call 004A039A
:00441956 8D4DB0                   lea ecx, dword ptr [ebp-50]
:00441959 E87EA70200               call 0046C0DC  ====>关键 CALL
:0044195E 3BC3                     cmp eax, ebx  **比较(这里只是一个标志,看不到你输入的 PASSWORD)
:00441960 7404                     je 00441966  **一跳就完(很奇怪吧!)
:00441962 8BF7                     mov esi, edi
:00441964 EB02                     jmp 00441968
```

`matrix.cnt` 存着上次登陆成功的密码(当然是加密后的!)

这是我的:

```
 5F 65 00 00 89 3B C2 4A 99 D9 31 B4 29 3C 39 6E 22 BB 2B DD
```

```
 \    /                \                                               /
 ---  -------------------------------------------------------------
这个一会儿         这是上次登陆成功密码的加密形式
 会用到
```

追进关键 `CALL` 中:

```
:0046C0FB 50                       push eax
:0046C0FC E85AA3FeFF               call 0045645B  ***CALL(对你输入的密码进行加密处理,
:0046C101 83C40C                   add esp, 0000000C        然后存在 7AEF20-7AEF2A 中)
:0046C104 6A01                     push 00000001
:0046C106 5F                       pop edi
:0046C107 397E04                   cmp dword ptr [esi+04], edi
:0046C10A 7618                     jbe 0046C124
:0046C10C 8D45E4                   lea eax, dword ptr [ebp-1C]-----------  对刚才加密过的密码再
:0046C10F 6A10                     push 00000010                                   |  进行循环加密处理
:0046C111 50                       push eax                                     |  这里的循环次数,是根据
:0046C112 8D45E4                   lea eax, dword ptr [ebp-1C]                 |  matrix.cnt 的内容
:0046C115 50                       push eax                                     |  656F 那就是 25951 次,
:0046C116 E840A3FeFF               call 0045645B  ***CALL                       |  好恐怖的数字呀!!
:0046C11B 83C40C                   add esp, 0000000C                          |
:0046C11E 47                       inc edi                                      |
:0046C11F 3B7E04                   cmp edi, dword ptr [esi+04]                |
:0046C122 72E8                     jb 0046C10C  ------------------------          |
:0046C124 83C608                   add esi, 00000008
:0046C127 6A10                     push 00000010
:0046C129 8D45E4                   lea eax, dword ptr [ebp-1C]
:0046C12C 56                       push esi
:0046C12D 50                       push eax
:0046C12E E87DB70100               call 004878B0  <==比较加密后的注册码
:0046C133 8BF0                     mov esi, eax
:0046C135 83C40C                   add esp, 0000000C
:0046C138 F7DE                     neg esi
:0046C13A 1BF6                     sbb esi, esi
:0046C13C 834DFCFF                 or dword ptr [ebp-04], FFFFFFFF
:0046C140 8D4D08                   lea ecx, dword ptr [ebp+08]
:0046C143 46                       inc esi
:0046C144 E8DC440300               call 004A0625
:0046C149 8B4DF4                   mov ecx, dword ptr [ebp-0C]
:0046C14C 8BC6                     mov eax, esi
:0046C14E 5F                       pop edi
:0046C14F 5E                       pop esi
:0046C150 64890D00000000           mov dword ptr fs:[00000000], ecx
:0046C157 C9                       leave
:0046C158 C20400                   ret 0004
```

进入关键 `CALL 0045645B`

```
:0045645B 55                       push ebp
:0045645C 8BEC                     mov ebp, esp
:0045645E 83EC5C                   sub esp, 0000005C
:00456461 8D45A4                   lea eax, dword ptr [ebp-5C]
:00456464 50                       push eax
:00456465 E847F3FFFF               call 004557B1
:0045646A FF7510                   push [ebp+10]
:0045646D 8D45A4                   lea eax, dword ptr [ebp-5C]
:00456470 FF750C                   push [ebp+0C]
:00456473 50                       push eax
:00456474 E873F3FFFF               call 004557EC
:00456479 8D45A4                   lea eax, dword ptr [ebp-5C]
:0045647C 50                       push eax
:0045647D FF7508                   push [ebp+08]
:00456480 E8BBFEFFFF               call 00456340    ****这里进行加密处理
:00456485 83C418                   add esp, 00000018
:00456488 C9                       leave
:00456489 C3                       ret
:00456340 53                       push ebx
:00456341 56                       push esi
:00456342 8B742410                 mov esi, dword ptr [esp+10]
:00456346 57                       push edi

* Possible StringData Ref from Data Obj ->"€"
                                                          |
:00456347 BAAC195100               mov edx, 005119AC
:0045634C 8B4E58                   mov ecx, dword ptr [esi+58]
:0045634F 8D5E18                   lea ebx, dword ptr [esi+18]
:00456352 8BC1                     mov eax, ecx
:00456354 83E103                   and ecx, 00000003
:00456357 C1F802                   sar eax, 02
:0045635A 83E900                   sub ecx, 00000000
:0045635D 8B3C83                   mov edi, dword ptr [ebx+4*eax]
:00456360 740B                     je 0045636D
:00456362 49                       dec ecx
:00456363 7414                     je 00456379
:00456365 49                       dec ecx
:00456366 7418                     je 00456380
:00456368 49                       dec ecx
:00456369 741E                     je 00456389
:0045636B EB24                     jmp 00456391

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00456360(C)
|
:0045636D 0FB63DAC195100           movzx edi, byte ptr [005119AC]

* Possible StringData Ref from Data Obj ->""
                                                          |
:00456374 BAAD195100               mov edx, 005119AD

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00456363(C)
|
:00456379 33C9                     xor ecx, ecx
:0045637B 8A2A                     mov ch, byte ptr [edx]
:0045637D 0BF9                     or edi, ecx
:0045637F 42                       inc edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00456366(C)
|
:00456380 0FB60A                   movzx ecx, byte ptr [edx]
:00456383 C1E110                   shl ecx, 10
:00456386 0BF9                     or edi, ecx
:00456388 42                       inc edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00456369(C)
|
:00456389 0FB60A                   movzx ecx, byte ptr [edx]
:0045638C C1E118                   shl ecx, 18
:0045638F 0BF9                     or edi, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0045636B(U)
|
```



# 汇编代码段分析

## 主循环与初始化

```asm
:00456391 893C83             mov dword ptr [ebx+4*eax], edi   

:00456394 40                 inc eax   

:00456395 837E5838           cmp dword ptr [esi+58], 00000038   

:00456399 7C1E               jl 004563B9   

:0045639B 6A10               push 00000010   

:0045639D 59                 pop ecx   

:0045639E 3BC1               cmp eax, ecx   

:004563A0 7D09               jge 004563AB   

:004563A2 8D3C83             lea edi, dword ptr [ebx+4*eax]   

:004563A5 2BC8               sub ecx, eax   

:004563A7 33C0               xor eax, eax   

:004563A9 F3                 repz   

:004563AA AB                 stosd   
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`004563A0` (C)

```asm
:004563AB 6A40               push 00000040   

:004563AD 53                 push ebx   

:004563AE 56                 push esi   

:004563AF E8B5F6FFFF         call 00455A69   

:004563B4 83C40C             add esp, 0000000C   

:004563B7 33C0               xor eax, eax   
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`00456399` (C)

```asm
:004563B9 6A0E               push 0000000E   

:004563BB 59                 pop ecx   

:004563BC 3BC1               cmp eax, ecx   

:004563BE 7D09               jge 004563C9   

:004563C0 8D3C83             lea edi, dword ptr [ebx+4*eax]   

:004563C3 2BC8               sub ecx, eax   

:004563C5 33C0               xor eax, eax   

:004563C7 F3                 repz   

:004563C8 AB                 stosd   
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`004563BE` (C)

```asm
:004563C9 8B4610             mov eax, dword ptr [esi+10]   

:004563CC 6A40               push 00000040   

:004563CE 894338             mov dword ptr [ebx+38], eax   

:004563D1 8B4614             mov eax, dword ptr [esi+14]   

:004563D4 53                 push ebx   

:004563D5 56                 push esi   

:004563D6 89433C             mov dword ptr [ebx+3C], eax   

:004563D9 E88BF6FFFF         call 00455A69               这里   

:004563DE 8B44241C           mov eax, dword ptr [esp+1C]   

:004563E2 8B0E               mov ecx, dword ptr [esi]   

:004563E4 83C40C             add esp, 0000000C   

:004563E7 8BD1               mov edx, ecx                 ======>从这里到 0045645A 处   

:004563E9 8808               mov byte ptr [eax], cl       (开始向 7AEF20-7AEF2A 存入加密后的密码)   

:004563EB 40                 inc eax   

:004563EC C1EA08             shr edx, 08   

:004563EF 8810               mov byte ptr [eax], dl   

:004563F1 8BD1               mov edx, ecx   

:004563F3 40                 inc eax   

:004563F4 5F                 pop edi   

:004563F5 C1EA10             shr edx, 10   

:004563F8 8810               mov byte ptr [eax], dl   

:004563FA 40                 inc eax   

:004563FB C1E918             shr ecx, 18   

:004563FE 8808               mov byte ptr [eax], cl   

:00456400 8B4E04             mov ecx, dword ptr [esi+04]   

:00456403 40                 inc eax   

:00456404 8BD1               mov edx, ecx   

:00456406 C1EA08             shr edx, 08   

:00456409 8808               mov byte ptr [eax], cl   

:0045640B 40                 inc eax   

:0045640C 8810               mov byte ptr [eax], dl   

:0045640E 8BD1               mov edx, ecx   

:00456410 40                 inc eax   

:00456411 C1EA10             shr edx, 10   

:00456414 8810               mov byte ptr [eax], dl   

:00456416 40                 inc eax   

:00456417 C1E918             shr ecx, 18   

:0045641A 8808               mov byte ptr [eax], cl   

:0045641C 8B4E08             mov ecx, dword ptr [esi+08]   

:0045641F 40                 inc eax   

:00456420 8BD1               mov edx, ecx   

:00456422 C1EA08             shr edx, 08   

:00456425 8808               mov byte ptr [eax], cl   

:00456427 40                 inc eax   

:00456428 8810               mov byte ptr [eax], dl   

:0045642A 8BD1               mov edx, ecx   

:0045642C 40                 inc eax   

:0045642D C1EA10             shr edx, 10   

:00456430 8810               mov byte ptr [eax], dl   

:00456432 40                 inc eax   

:00456433 C1E918             shr ecx, 18   

:00456436 8808               mov byte ptr [eax], cl   

:00456438 8B4E0C             mov ecx, dword ptr [esi+0C]   

:0045643B 40                 inc eax   

:0045643C 8BD1               mov edx, ecx   

:0045643E C1EA08             shr edx, 08   

:00456441 8808               mov byte ptr [eax], cl   

:00456443 40                 inc eax   

:00456444 8810               mov byte ptr [eax], dl   

:00456446 8BD1               mov edx, ecx   

:00456448 40                 inc eax   

:00456449 C1EA10             shr edx, 10   

:0045644C C1E918             shr ecx, 18   

:0045644F 8810               mov byte ptr [eax], dl   

:00456451 884801             mov byte ptr [eax+01], cl   

:00456454 83665800           and dword ptr [esi+58], 00000000   

:00456458 5E                 pop esi   

:00456459 5B                 pop ebx   

:0045645A C3                 ret  
```

进入 `call 00455A69`

瞧这一大堆代码!

## 子程序入口 (00455A69)

```asm
:00455A69 55                 push ebp   

:00455A6A 8BEC               mov ebp, esp   

:00455A6C 51                 push ecx   

:00455A6D 51                 push ecx   

:00455A6E 8B4D08             mov ecx, dword ptr [ebp+08]   

:00455A71 56                 push esi   

:00455A72 57                 push edi   

:00455A73 8B01               mov eax, dword ptr [ecx]      ; [ECX]=01234567      

:00455A75 8B7904             mov edi, dword ptr [ecx+04]  ;[ECX+4]=89ABCDEF     

:00455A78 8B7108             mov esi, dword ptr [ecx+08]  ;[ECX+8]=FEDCBA98     

:00455A7B 8B510C             mov edx, dword ptr [ecx+0C]  ;[ECX+C]=76543210     

:00455A7E 894508             mov dword ptr [ebp+08], eax   

:00455A81 8B4510             mov eax, dword ptr [ebp+10]   

:00455A84 83E840             sub eax, 0000040   

:00455A87 897DFC             mov dword ptr [ebp-04], edi   

:00455A8A 0F88AC080000       js 0045633C   

:00455A90 83C040             add eax, 00000040   

:00455A93 53                 push ebx   

:00455A94 C1E806             shr eax, 06   

:00455A97 8945F8             mov dword ptr [ebp-08], eax   

:00455A9A 8B450C             mov eax, dword ptr [ebp+0C]  把要进行处理的密码地址放入 EAX   
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`00456335` (C)

```asm
:00455A9D 8BDA               mov ebx, edx      *****开始喽!(好长呀!!)                       

:00455A9F 33DE               xor ebx, esi   

:00455AA1 23DF               and ebx, edi   

:00455AA3 8B7D08             mov edi, dword ptr [ebp+08]   

:00455AA6 33DA               xor ebx, edx   

:00455AA8 0318               add ebx, dword ptr [eax]     

:00455AAA 8DBC1F78A46AD7     lea edi, dword ptr [edi+ebx-28955B88]   

:00455AB1 8BDF               mov ebx, edi   

:00455AB3 C1EB19             shr ebx, 19   

:00455AB6 C1E707             shl edi, 07   

:00455AB9 0BDF               or ebx, edi   

:00455ABB 8BFE               mov edi, esi   

:00455ABD 035DFC             add ebx, dword ptr [ebp-04]   

:00455AC0 337DFC             xor edi, dword ptr [ebp-04]   

:00455AC3 23FB               and edi, ebx   
```



```
:00455AC5 895D08                 mov dword ptr [ebp+08], ebx

:00455AC8 33FE                   xor edi, esi

:00455ACA 037804                 add edi, dword ptr [eax+04]

:00455ACD 8D943A56B7C7E8         lea edx, dword ptr [edx+edi-173848AA]

:00455AD4 8BFA                   mov edi, edx

:00455AD6 C1EF14                 shr edi, 14

:00455AD9 C1E20C                 shl edx, 0C

:00455ADC 0BFA                   or edi, edx

:00455ADE 8B55FC                 mov edx, dword ptr [ebp-04]

:00455AE1 03FB                   add edi, ebx

:00455AE3 33D3                   xor edx, ebx

:00455AE5 23D7                   and edx, edi

:00455AE7 3355FC                 xor edx, dword ptr [ebp-04]

:00455AEA 035008                 add edx, dword ptr [eax+08]

:00455AED 8DB416DB702024         lea esi, dword ptr [esi+edx+242070DB]

:00455AF4 8BD6                   mov edx, esi

:00455AF6 C1EA0F                 shr edx, 0F

:00455AF9 C1E611                 shl esi, 11

:00455AFC 0BD6                   or edx, esi

:00455AFE 8BF7                   mov esi, edi

:00455B00 33F3                   xor esi, ebx

:00455B02 03D7                   add edx, edi

:00455B04 23F2                   and esi, edx

:00455B06 33F3                   xor esi, ebx

:00455B08 8B5DFC                 mov ebx, dword ptr [ebp-04]

:00455B0B 03700C                 add esi, dword ptr [eax+0C]

:00455B0E 8DB433EECEBDC1         lea esi, dword ptr [ebx+esi-3E423112]

:00455B15 8BDE                   mov ebx, esi

:00455B17 C1E316                 shl ebx, 16

:00455B1A C1EE0A                 shr esi, 0A

:00455B1D 0BDE                   or ebx, esi

:00455B1F 8BF7                   mov esi, edi

:00455B21 03DA                   add ebx, edx

:00455B23 33F2                   xor esi, edx

:00455B25 23F3                   and esi, ebx

:00455B27 895DFC                 mov dword ptr [ebp-04], ebx

:00455B2A 8B5D08                 mov ebx, dword ptr [ebp+08]

:00455B2D 33F7                   xor esi, edi

:00455B2F 037010                 add esi, dword ptr [eax+10]

:00455B32 8DB433AF0F7CF5         lea esi, dword ptr [ebx+esi-0A83F051]

:00455B39 8BDE                   mov ebx, esi

:00455B3B C1EB19                 shr ebx, 19

:00455B3E C1E607                 shl esi, 07

:00455B41 0BDE                   or ebx, esi

:00455B43 8BF2                   mov esi, edx

:00455B45 035DFC                 add ebx, dword ptr [ebp-04]

:00455B48 3375FC                 xor esi, dword ptr [ebp-04]

:00455B4B 23F3                   and esi, ebx

:00455B4D 895D08                 mov dword ptr [ebp+08], ebx

:00455B50 33F2                   xor esi, edx

:00455B52 037014                 add esi, dword ptr [eax+14]

:00455B55 8DBC372AC68747         lea edi, dword ptr [edi+esi+4787C62A]

:00455B5C 8BF7                   mov esi, edi

:00455B5E C1EE14                 shr esi, 14

:00455B61 C1E70C                 shl edi, 0C

:00455B64 0BF7                   or esi, edi

:00455B66 8B7DFC                 mov edi, dword ptr [ebp-04]

:00455B69 03F3                   add esi, ebx

:00455B6B 33FB                   xor edi, ebx

:00455B6D 23FE                   and edi, esi

:00455B6F 337DFC                 xor edi, dword ptr [ebp-04]

:00455B72 037818                 add edi, dword ptr [eax+18]

:00455B75 8D943A134630A8         lea edx, dword ptr [edx+edi-57CFB9ED]

:00455B7C 8BFA                   mov edi, edx

:00455B7E C1EF0F                 shr edi, 0F

:00455B81 C1E211                 shl edx, 11

:00455B84 0BFA                   or edi, edx

:00455B86 8BD6                   mov edx, esi

:00455B88 03FE                   add edi, esi

:00455B8A 33D3                   xor edx, ebx

:00455B8C 23D7                   and edx, edi

:00455B8E 33D3                   xor edx, ebx

:00455B90 8B5DFC                 mov ebx, dword ptr [ebp-04]

:00455B93 03501C                 add edx, dword ptr [eax+1C]

:00455B96 8D9413019546FD         lea edx, dword ptr [ebx+edx-02B96AFF]

:00455B9D 8BDA                   mov ebx, edx

:00455B9F C1E316                 shl ebx, 16

:00455BA2 C1EA0A                 shr edx, 0A

:00455BA5 0BDA                   or ebx, edx

:00455BA7 8BD6                   mov edx, esi

:00455BA9 03DF                   add ebx, edi

:00455BAB 33D7                   xor edx, edi

:00455BAD 23D3                   and edx, ebx

:00455BAF 895DFC                 mov dword ptr [ebp-04], ebx

:00455BB2 8B5D08                 mov ebx, dword ptr [ebp+08]

:00455BB5 33D6                   xor edx, esi

:00455BB7 035020                 add edx, dword ptr [eax+20]

:00455BBA 8D9413D8988069         lea edx, dword ptr [ebx+edx+698098D8]

:00455BC1 8BDA                   mov ebx, edx

:00455BC3 C1EB19                 shr ebx, 19

:00455BC6 C1E207                 shl edx, 07

:00455BC9 0BDA                   or ebx, edx

:00455BCB 8BD7                   mov edx, edi

:00455BCD 035DFC                 add ebx, dword ptr [ebp-04]

:00455BD0 3355FC                 xor edx, dword ptr [ebp-04]

:00455BD3 23D3                   and edx, ebx

:00455BD5 895D08                 mov dword ptr [ebp+08], ebx

:00455BD8 33D7                   xor edx, edi

:00455BDA 035024                 add edx, dword ptr [eax+24]

:00455BDD 8DB416AFF7448B         lea esi, dword ptr [esi+edx-74BB0851]

:00455BE4 8BD6                   mov edx, esi

:00455BE6 C1EA14                 shr edx, 14

:00455BE9 C1E60C                 shl esi, 0C

:00455BEC 0BD6                   or edx, esi

:00455BEE 8B75FC                 mov esi, dword ptr [ebp-04]

:00455BF1 03D3                   add edx, ebx

:00455BF3 33F3                   xor esi, ebx

:00455BF5 23F2                   and esi, edx

:00455BF7 3375FC                 xor esi, dword ptr [ebp-04]

:00455BFA 037028                 add esi, dword ptr [eax+28]

:00455BFD 8DBC37B15BFFFF         lea edi, dword ptr [edi+esi-0000A44F]

:00455C04 8BF7                   mov esi, edi

:00455C06 C1EE0F                 shr esi, 0F

:00455C09 C1E711                 shl edi, 11

:00455C0C 0BF7                   or esi, edi

:00455C0E 8BFA                   mov edi, edx

:00455C10 03F2                   add esi, edx

:00455C12 33FB                   xor edi, ebx

:00455C14 23FE                   and edi, esi

:00455C16 33FB                   xor edi, ebx

:00455C18 8B5DFC                 mov ebx, dword ptr [ebp-04]

:00455C1B 03782C                 add edi, dword ptr [eax+2C]

:00455C1E 8D9C3BBED75C89         lea ebx, dword ptr [ebx+edi-76A32842]

:00455C25 8BFB                   mov edi, ebx

:00455C27 C1E716                 shl edi, 16

:00455C2A C1EB0A                 shr ebx, 0A

:00455C2D 0BFB                   or edi, ebx

:00455C2F 8BDA                   mov ebx, edx

:00455C31 03FE                   add edi, esi

:00455C33 33DE                   xor ebx, esi

:00455C35 23DF                   and ebx, edi

:00455C37 897DFC                 mov dword ptr [ebp-04], edi

:00455C3A 8B7D08                 mov edi, dword ptr [ebp+08]

:00455C3D 33DA                   xor ebx, edx

:00455C3F 035830                 add ebx, dword ptr [eax+30]

:00455C42 8DBC1F2211906B         lea edi, dword ptr [edi+ebx+6B901122]

:00455C49 8BDF                   mov ebx, edi

:00455C4B C1EB19                 shr ebx, 19

:00455C4E C1E707                 shl edi, 07
```



```
:00455C51 0BDF                or ebx, edi
:00455C53 8BFE                mov edi, esi
:00455C55 035DFC              add ebx, dword ptr [ebp-04]
:00455C58 337DFC              xor edi, dword ptr [ebp-04]
:00455C5B 23FB                and edi, ebx
:00455C5D 895D08              mov dword ptr [ebp+08], ebx
:00455C60 33FE                xor edi, esi
:00455C62 037834              add edi, dword ptr [eax+34]
:00455C65 8D943A937198FD      lea edx, dword ptr [edx+edi-02678E6D]
:00455C6C 8BFA                mov edi, edx
:00455C6E C1EF14              shr edi, 14
:00455C71 C1E20C              shl edx, 0C
:00455C74 0BFA                or edi, edx
:00455C76 8B55FC              mov edx, dword ptr [ebp-04]
:00455C79 03FB                add edi, ebx
:00455C7B 33D3                xor edx, ebx
:00455C7D 23D7                and edx, edi
:00455C7F 897D10              mov dword ptr [ebp+10], edi
:00455C82 3355FC              xor edx, dword ptr [ebp-04]
:00455C85 035038              add edx, dword ptr [eax+38]
:00455C88 8DB4168E4379A6      lea esi, dword ptr [esi+edx-5986BC72]
:00455C8F 8BD6                mov edx, esi
:00455C91 C1EA0F              shr edx, 0F
:00455C94 C1E611              shl esi, 11
:00455C97 0BD6                or edx, esi
:00455C99 8BF7                mov esi, edi
:00455C9B 03D7                add edx, edi
:00455C9D 33F3                xor esi, ebx
:00455C9F 23F2                and esi, edx
:00455CA1 33F3                xor esi, ebx
:00455CA3 8B5DFC              mov ebx, dword ptr [ebp-04]
:00455CA6 03703C              add esi, dword ptr [eax+3C]
:00455CA9 8D9C332108B449      lea ebx, dword ptr [ebx+esi+49B40821]
:00455CB0 8BF3                mov esi, ebx
:00455CB2 C1E616              shl esi, 16
:00455CB5 C1EB0A              shr ebx, 0A
:00455CB8 0BF3                or esi, ebx
:00455CBA 8BDA                mov ebx, edx
:00455CBC 03F2                add esi, edx
:00455CBE 33DE                xor ebx, esi
:00455CC0 23DF                and ebx, edi
:00455CC2 8B7D08              mov edi, dword ptr [ebp+08]
:00455CC5 33DA                xor ebx, edx
:00455CC7 035804              add ebx, dword ptr [eax+04]
:00455CCA 8D9C1F62251EF6      lea ebx, dword ptr [edi+ebx-09E1DA9E]
:00455CD1 8BFB                mov edi, ebx
:00455CD3 C1EF1B              shr edi, 1B
:00455CD6 C1E305              shl ebx, 05
:00455CD9 0BFB                or edi, ebx
:00455CDB 8BDE                mov ebx, esi
:00455CDD 03FE                add edi, esi
:00455CDF 33DF                xor ebx, edi
:00455CE1 897D08              mov dword ptr [ebp+08], edi
:00455CE4 8B7D10              mov edi, dword ptr [ebp+10]
:00455CE7 23DA                and ebx, edx
:00455CE9 33DE                xor ebx, esi
:00455CEB 035818              add ebx, dword ptr [eax+18]
:00455CEE 8D9C1F40B340C0      lea ebx, dword ptr [edi+ebx-3FBF4CC0]
:00455CF5 8BFB                mov edi, ebx
:00455CF7 C1EF17              shr edi, 17
:00455CFA C1E309              shl ebx, 09
:00455CFD 0BFB                or edi, ebx
:00455CFF 037D08              add edi, dword ptr [ebp+08]
:00455D02 897D10              mov dword ptr [ebp+10], edi
:00455D05 337D08              xor edi, dword ptr [ebp+08]
:00455D08 23FE                and edi, esi
:00455D0A 337D08              xor edi, dword ptr [ebp+08]
:00455D0D 03782C              add edi, dword ptr [eax+2C]
:00455D10 8D943A515A5E26      lea edx, dword ptr [edx+edi+265E5A51]
:00455D17 8BFA                mov edi, edx
:00455D19 C1EF12              shr edi, 12
:00455D1C C1E20E              shl edx, 0E
:00455D1F 0BFA                or edi, edx
:00455D21 8B5510              mov edx, dword ptr [ebp+10]
:00455D24 037D10              add edi, dword ptr [ebp+10]
:00455D27 33D7                xor edx, edi
:00455D29 235508              and edx, dword ptr [ebp+08]
:00455D2C 335510              xor edx, dword ptr [ebp+10]
:00455D2F 0310                add edx, dword ptr [eax]
:00455D31 8DB416AAC7B6E9      lea esi, dword ptr [esi+edx-16493856]
:00455D38 8BD6                mov edx, esi
:00455D3A C1E214              shl edx, 14
:00455D3D C1EE0C              shr esi, 0C
:00455D40 0BD6                or edx, esi
:00455D42 8BF7                mov esi, edi
:00455D44 03D7                add edx, edi
:00455D46 33F2                xor esi, edx
:00455D48 237510              and esi, dword ptr [ebp+10]
:00455D4B 33F7                xor esi, edi
:00455D4D 037014              add esi, dword ptr [eax+14]
:00455D50 8B5D08              mov ebx, dword ptr [ebp+08]
:00455D53 8D9C335D102FD6      lea ebx, dword ptr [ebx+esi-29D0EFA3]
:00455D5A 8BF3                mov esi, ebx
:00455D5C C1EE1B              shr esi, 1B
:00455D5F C1E305              shl ebx, 05
:00455D62 0BF3                or esi, ebx
:00455D64 8BDA                mov ebx, edx
:00455D66 03F2                add esi, edx
:00455D68 33DE                xor ebx, esi
:00455D6A 897508              mov dword ptr [ebp+08], esi
:00455D6D 8B7510              mov esi, dword ptr [ebp+10]
:00455D70 23DF                and ebx, edi
:00455D72 33DA                xor ebx, edx
:00455D74 035828              add ebx, dword ptr [eax+28]
:00455D77 8D9C1E53144402      lea ebx, dword ptr [esi+ebx+02441453]
:00455D7E 8BF3                mov esi, ebx
:00455D80 C1EE17              shr esi, 17
:00455D83 C1E309              shl ebx, 09
:00455D86 0BF3                or esi, ebx
:00455D88 8B5D08              mov ebx, dword ptr [ebp+08]
:00455D8B 037508              add esi, dword ptr [ebp+08]
:00455D8E 897510              mov dword ptr [ebp+10], esi
:00455D91 337508              xor esi, dword ptr [ebp+08]
:00455D94 23F2                and esi, edx
:00455D96 337508              xor esi, dword ptr [ebp+08]
:00455D99 03703C              add esi, dword ptr [eax+3C]
:00455D9C 8DBC3781E6A1D8      lea edi, dword ptr [edi+esi-275E197F]
:00455DA3 8BF7                mov esi, edi
:00455DA5 C1EE12              shr esi, 12
:00455DA8 C1E70E              shl edi, 0E
:00455DAB 0BF7                or esi, edi
:00455DAD 8B7D10              mov edi, dword ptr [ebp+10]
:00455DB0 037510              add edi, dword ptr [ebp+10]
4dd466d0e1e0cc123afbe504da2462ca
```
