
```text
0FAF45E4                imul eax, dword ptr [ebp-1C]
                       ====> EAX=97 \* 3B=22CD
:004E43AF
0FAF45E0                imul eax, dword ptr [ebp-20]
                       ====> EAX=22CD \* 07=F39B
:004E43B3
0FAFC6                  imul eax, esi
                       ====> EAX=F39B \* 17=15E2ED（H）=1434349（D）
:004E43B6
E8F94EF2FF              call 004092B4
                       ====> 将15E2ED（H）转化为10进制值1434349
:004E43BB
8B45C0                  mov eax, dword ptr [ebp-40]
                       ====> EAX=1434349
:004E43BE
8D4DEC                  lea ecx, dword ptr [ebp-14]
:004E43C1 BA06000000     mov edx, 00000006
:004E43C6 E88DFDFFFF     call 004E4158
                       ====> 整理CALL。上面所得＜6位则前面加0；若＞6位则只取后6位
                       ====> 1434339－->434349
:004E43CB
807DD301                cmp byte ptr [ebp-2D], 01
:004E43CF 751E           jne 004E43EF
:004E43D1 807DD201       cmp byte ptr [ebp-2E], 01
:004E43D5 7518           jne 004E43EF
:004E43D7 FF75F0         push [ebp-10]
:004E43DA
FF75EC                  push [ebp-14]
:004E43DD
FF75F4                  push [ebp-0C]
:004E43E0
8D45E8                  lea eax, dword ptr [ebp-18]
:004E43E3 BA03000000     mov edx, 00000003
:004E43E8 E8F706F2FF     call 00404AE4
:004E43ED EB5E           jmp 004E444D

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E43CF(C), :004E43D5(C)

:004E43EF 807DD301        cmp byte ptr [ebp-2D], 01
:004E43F3 751E            jne 004E4413
:004E43F5 807DD200        cmp byte ptr [ebp-2E], 00
:004E43F9 7518            jne 004E4413
:004E43FB FF75EC          push [ebp-14]
                       ====> [ebp-14]=434349  壹
:004E43FE FF75F0          push [ebp-10]
                       ====> [ebp-10]=016219  贰
:004E4401
FF75F4                  push [ebp-0C]
                       ====> [ebp-0C]=029972  叁
:004E4404
8D45E8                  lea eax, dword ptr [ebp-18]
:004E4407 BA03000000     mov edx, 00000003
:004E440C E8D306F2FF     call 00404AE4
                       ====> 将上面所得的壹 贰 叁 连接起来
:004E4411
EB3A                    jmp 004E444D

…… …… 省 略 ……
……

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E43ED(U), :004E4411(U), :004E4435(U)

:004E444D 8B45F8          mov eax, dword ptr [ebp-08]
:004E4450 8B55E8          mov edx, dword ptr [ebp-18]
                       ====> EDX=434349016219029972
```

***

### 进入整理CALL

```text
\* Referenced by a CALL at Addresses:
|:004E4381, :004E43A0, :004E43C6

:004E4158 55                push ebp
:004E4159 8BEC              mov ebp, esp
:004E415B 51                push ecx
:004E415C 53                push ebx
:004E415D 56                push esi
:004E415E 8BF1              mov esi, ecx
:004E4160 8BDA              mov ebx, edx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E40FB(C)

:004E4162 8945FC            mov dword ptr [ebp-04], eax
:004E4165 8B45FC            mov eax, dword ptr [ebp-04]
:004E4168 E89F0AF2FF        call 00404C0C
:004E416D 33C0              xor eax, eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E4108(C)

:004E416F 55                push ebp

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E4102(C)

:004E4170 68EB414E00        push 004E41EB
:004E4175 64FF30            push dword ptr fs:[eax]
:004E4178 648920            mov dword ptr fs:[eax], esp
:004E417B 8B45FC            mov eax, dword ptr [ebp-04]
:004E417E E8A108F2FF        call 00404A24
                       ====> 求位数
:004E4183 3BD8              cmp ebx, eax
                       ====> 与6比较
:004E4185 7C29              jl 004E41B0
:004E4187 8B45FC            mov eax, dword ptr [ebp-04]
:004E418A E89508F2FF        call 00404A24
:004E418F 50                push eax
:004E4190 8BC3              mov eax, ebx
:004E4192 5A                pop edx
:004E4193 2BC2              sub eax, edx
:004E4195 8BD8              mov ebx, eax
:004E4197 85DB              test ebx, ebx
:004E4199 7E30              jle 004E41CB

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E41AC(C)

:004E419B 8D45FC            lea eax, dword ptr [ebp-04]
:004E419E 8B4DFC            mov ecx, dword ptr [ebp-04]
:004E41A1 BA00424E00        mov edx, 004E4200
:004E41A6 E8C508F2FF        call 00404A70
:004E41AB 4B                dec ebx
:004E41AC 75ED              jne 004E419B
:004E41AE EB1B              jmp 004E41CB

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E4185(C)

:004E41B0 8D45FC            lea eax, dword ptr [ebp-04]
:004E41B3 50                push eax
:004E41B4 8B45FC            mov eax, dword ptr [ebp-04]
:004E41B7 E86808F2FF        call 00404A24
:004E41BC 8BD0              mov edx, eax
:004E41BE 2BD3              sub edx, ebx
:004E41C0 42                inc edx
:004E41C1 8BCB              mov ecx, ebx
:004E41C3 8B45FC            mov eax, dword ptr [ebp-04]
:004E41C6 E8B10AF2FF        call 00404C7C

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E4199(C), :004E41AE(U)

:004E41CB 8BC6              mov eax, esi
:004E41CD 8B55FC            mov edx, dword ptr [ebp-04]
                      1、 ====> EDX=022972
                      2、 ====> EDX=016219
                      3、 ====> EDX=434349
:004E41D0 E8EB05F2FF        call 004047C0
:004E41D5 33C0              xor eax, eax
:004E41D7 5A                pop edx
:004E41D8 59                pop ecx
:004E41D9 59                pop ecx
:004E41DA 648910            mov dword ptr fs:[eax], edx
:004E41DD 68F2414E00        push 004E41F2

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E41F0(U)

:004E41E2 8D45FC            lea eax, dword ptr [ebp-04]
:004E41E5 E88205F2FF        call 0040476C
:004E41EA C3                ret
```

***

### 进入比较CALL：`4D6271`

```text
call 00404B68

:00404B68 53                push ebx
:00404B69 56                push esi
:00404B6A 57                push edi
:00404B6B 89C6              mov esi, eax
:00404B6D 89D7              mov edi, edx
:00404B6F 39D0              cmp eax, edx
                       ====> EAX=434349016219029972
                       ====> EDX=13572468
```

***

### KeyMake之{48th}内存注册机

- 中断地址：`4D625E`
- 中断次数：`1`
- 第一字节：`50`
- 指令长度：`1`
- 内存方式：`EAX`

***

### 注册信息保存

`\CRAP\Data`文件夹下的`Alumni.mdb`中：`UserInfo`

***

### 整 理

- 用户名：`fly`
- 注册码：`434349016219029972`

***

Cracked By 巢水工作坊――fly【OCN】

2003-03-18 19:41:48
