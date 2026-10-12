
# Flashdown

> 原文：[`www.pediy.com/kssd/pediy06/pediy6155.htm`](https://www.pediy.com/kssd/pediy06/pediy6155.htm)

- **软件大小**： 799 KB
- **软件语言**： 英文
- **软件类别**： 国外软件 / 共享版 / 下载工具
- **应用平台**： `Win9x/NT/2000/XP`
- **加入时间**： 2003-07-05 09:48:51
- **下载次数**： 5008
- **推荐等级**： ★★★
- **软件下载**： [`count.skycn.com/softdown.php?id=9310&url=http://gd-http.skycn.net:8080/down/fdt.zip`](http://count.skycn.com/softdown.php?id=9310&url=http://gd-http.skycn.net:8080/down/fdt.zip)

## 软件介绍

`Flashdown` 让你将 `Internet` 网页中的 `Flash` 动画文件下载到电脑中，`Flash` 精简的文件大小与音效支持功能成为许多人的最爱，因此是目前常见的网络动画文件格式。不过当你想要将这些网页上的动画放到自己的电脑内慢慢欣赏时，就会发现这些文件并无法让你下载。不过这个问题遇上了 `Flashdown` 就能迎刃而解。

**破解工具**：`TRW1.22`, `OllyDbg1.09`, `PWdasm`

**作者声明**：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!

软件未加壳，`VC` 编写，用 `bpx getwindowtexta` 下断，`OD` 载入分析!

### 【简要过程】

- **用户名**：`ShenGe[BCG]`
- **试验码**：`12345678908765`

.............(略)

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0043687A(C)`

|

```asm
:0043688A 52                                  push edx
:0043688B 68D8525300                          push 005352D8
:00436890 E86B1A0300                          call 00468300
:00436895 83C408                              add esp, 00000008
```

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:00436854(C)`

|

```asm
:00436898 8B542414                            mov edx, dword ptr [esp+14]
                                                ; edx="12345678908765", 试验码
:0043689C 837AF40E                            cmp dword ptr [edx-0C], 0000000E
                                                ; 判断试验码长度是否为 14 位
:004368A0 0F8522010000                        jne 004369C8
:004368A6 8B442410                            mov eax, dword ptr [esp+10]
                                                ; eax="ShenGe[BCG]", 用户名
:004368AA 3958F4                              cmp dword ptr [eax-0C], ebx
                                                ; 判断有无输入用户名
:004368AD 0F8E15010000                        jle 004369C8
:004368B3 6A04                                push 00000004
                                                ; 入栈参数 1
:004368B5 8D4C2420                            lea ecx, dword ptr [esp+20]
:004368B9 51                                 push ecx
:004368BA 8D4C241C                            lea ecx, dword ptr [esp+1C]
                                                ; [ecx]="12345678908765"
:004368BE E80D6DFEFF                          call 0041D5D0
                                                ; 取试验码的前 4 位为"1234"
:004368C3 B302                                mov bl, 02
:004368C5 8D4C2410                            lea ecx, dword ptr [esp+10]
                                                ; ecx="ShenGe[BCG]", 用户名
:004368C9 885C242C                            mov byte ptr [esp+2C], bl
:004368CD E8FEC5FCFF                          call 00402ED0
                                                ; 此 Call 将字母大写转小写
:004368D2 8B442410                            mov eax, dword ptr [esp+10]
                                                ; eax="shenge[bcg]"
:004368D6 51                                 push ecx
:004368D7 83C0F0                              add eax, FFFFFFF0
:004368DA 8964241C                            mov dword ptr [esp+1C], esp
:004368DE 8BFC                                mov edi, esp
:004368E0 50                                 push eax
:004368E1 E8FAB4FCFF                          call 00401DE0
:004368E6 83C010                              add eax, 00000010
:004368E9 8907                                mov dword ptr [edi], eax
:004368EB 8B6C2424                            mov ebp, dword ptr [esp+24]
:004368EF 83C5F0                              add ebp, FFFFFFF0
:004368F2 89642428                            mov dword ptr [esp+28], esp
:004368F6 8BFC                                mov edi, esp
:004368F8 55                                 push ebp
:004368F9 C644243803                          mov [esp+38], 03
:004368FE E8DDB4FCFF                          call 00401DE0
:00436903 83C010                              add eax, 00000010
:00436906 83C404                              add esp, 00000004
:00436909 8907                                mov dword ptr [edi], eax
                                                ; [edi]="1234", 试验码前 4 位
:0043690B 885C2434                            mov byte ptr [esp+34], bl
                                                ; [esp+34]=bl=02
:0043690F E87CFBFFFF                          call 00436490
                                                ; 追算法的话跟进此 Call!①
:00436914 83C408                              add esp, 00000008
:00436917 83F801                              cmp eax, 00000001
:0043691A 8BCE                                mov ecx, esi
:0043691C 7521                                jne 0043693F
:0043691E C7467803000000                      mov [esi+78], 00000003
:00436925 8B15E07C5700                        mov edx, dword ptr [00577CE0]
:0043692B C782B001000020000000                mov dword ptr [ebx+000001B0], 00000020
:00436935 8B06                                mov eax, dword ptr [esi]
:00436937 FF904C010000                        call dword ptr [eax+0000014C]
:0043693D EB6B                                jmp 004369AA
```

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0043691C(C)`

|

```asm
:0043693F 8B7E70                              mov edi, dword ptr [esi+70]
:00436942 47                                  inc edi
:00436943 6874040000                          push 00000474
:00436948 897E70                              mov dword ptr [esi+70], edi
:0043694B E877A90C00                          call 005012C7
:00436950 8BF0                                mov esi, eax
:00436952 E8C3B50C00                          call 00501F1A
:00436957 8B10                                mov edx, dword ptr [eax]
:00436959 8BC8                                mov ecx, eax
:0043695B FF520C                              call [edx+0C]
:0043695E 83C010                              add eax, 00000010
:00436961 89442418                            mov dword ptr [esp+18], eax
```

* `Possible Reference to String Resource ID=00151: "Incomplete or incorrect information."`

|

```asm
:00436965 6897000000                          push 00000097
:0043696A 8D4C241C                            lea ecx, dword ptr [esp+1C]
:0043696E C644243004                          mov [esp+30], 04
:00436973 E8F8C5FCFF                          call 00402F70
:00436978 8B7C2418                            mov edi, dword ptr [esp+18]
:0043697C 57                                  push edi
:0043697D 8BCE                                mov ecx, esi
:0043697F E867AA0C00                          call 005013EB
:00436984 6AFF                                push FFFFFFFF
```

★★★★★★★★★★★★★★★★★★

①跟进这个关键的 `Call`，来到如下代码：

```asm
:00436490 6AFF                                push FFFFFFFF
:00436492 683B5A5200                          push 00525A3B
:00436497 64A100000000                        mov eax, dword ptr fs:[00000000]
:0043649D 50                                  push eax
:0043649E 64892500000000                      mov dword ptr fs:[00000000], esp
:004364A5 81EC40010000                        sub esp, 00000140
:004364AB 53                                  push ebx
:004364AC 55                                  push ebp
:004364AD 56                                  push esi
:004364AE 57                                  push edi
:004364AF BD01000000                          mov ebp, 00000001
:004364B4 33C9                                 xor ecx, ecx
:004364B6 89AC2458010000                      mov dword ptr [esp+00000158], ebp
:004364BD 894C2418                            mov dword ptr [esp+18], ecx
:004364C1 C744242417000000                    mov [esp+24], 00000017
:004364C9 C744241C05000000                    mov [esp+1C], 00000005
:004364D1 896C2420                            mov dword ptr [esp+20], ebp
:004364D5 33C0                                 xor eax, eax
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:004364E4(C)`

```asm
:004364D7 8AD0                                 mov dl, al
:004364D9 80C241                               add dl, 41
:004364DC 8854042C                             mov byte ptr [esp+eax+2C], dl
:004364E0 40                                   inc eax
:004364E1 83F81A                               cmp eax, 0000001A
:004364E4 7CF1                                 jl 004364D7
:004364E6 C644244630                           mov [esp+46], 30
:004364EB C644244731                           mov [esp+47], 31
:004364F0 C644244832                           mov [esp+48], 32
:004364F5 C644244933                           mov [esp+49], 33
:004364FA C644244A34                           mov [esp+4A], 34
:004364FF C644244B35                           mov [esp+4B], 35
:00436504 C644244C36                           mov [esp+4C], 36
:00436509 C644244D37                           mov [esp+4D], 37
```
