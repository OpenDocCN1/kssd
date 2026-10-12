
# 仿真科学计算器 1.8+

> 原文：[`www.pediy.com/kssd/pediy06/pediy6801.htm`](https://www.pediy.com/kssd/pediy06/pediy6801.htm)

先用 `FrogICE` 隐藏一下,进去后可以来到下面:

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AA12A(C)
|
:004AA0F4 8D8558FFFFFF            lea eax, dword ptr [ebp+FFFFFF58]
:004AA0FA 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA100 8A5432FF                mov dl, byte ptr [edx+esi-01] <--EDX 是软件给的序列号,依次取出来
:004AA104 E8ABA5F5FF              call 004046B4
:004AA109 8B8558FFFFFF            mov eax, dword ptr [ebp+FFFFFF58]
:004AA10F E8D4E9F5FF              call 00408AE8  <--把取出来的结果转成数值放在 EAX
:004AA114 8B048520CA4A00          mov eax, dword ptr [4*eax+004ACA20]<--从 4ACA20 处按 EAX 来取数
:004AA11B 99                      cdq
:004AA11C 3345D8                  xor eax, dword ptr [ebp-28] <--[ebp-28]开始是假码的值,和取出来的数异或
:004AA11F 3355DC                  xor edx, dword ptr [ebp-24]
:004AA122 8945D8                  mov dword ptr [ebp-28], eax <--把结果再放在[ebp-28]继续计算
:004AA125 8955DC                  mov dword ptr [ebp-24], edx
:004AA128 46                      inc esi      <--循环变量+1
:004AA129 4B                      dec ebx
:004AA12A 75C8                    jne 004AA0F4 <--循环计算(记为计算 1)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AA0ED(C)
|
:004AA12C 33F6                    xor esi, esi
:004AA12E B920CA4A00              mov ecx, 004ACA20

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AA15F(C)
|
:004AA133 8B01                    mov eax, dword ptr [ecx] <--从 4ACA20 处取数
:004AA135 99                      cdq
:004AA136 3345D8                  xor eax, dword ptr [ebp-28] <--和上面的计算结果异或
:004AA139 3355DC                  xor edx, dword ptr [ebp-24]
:004AA13C 8945D8                  mov dword ptr [ebp-28], eax <--结果仍放在[EBP-28]
:004AA13F 8955DC                  mov dword ptr [ebp-24], edx
:004AA142 8B8100020000            mov eax, dword ptr [ecx+00000200] <--从 4ACC20 处取数
:004AA148 99                      cdq
:004AA149 3345D8                  xor eax, dword ptr [ebp-28] <--再异或
:004AA14C 3355DC                  xor edx, dword ptr [ebp-24]
:004AA14F 8945D8                  mov dword ptr [ebp-28], eax <--仍放在[EBP-28]
:004AA152 8955DC                  mov dword ptr [ebp-24], edx
:004AA155 46                      inc esi        <--循环变量+1
:004AA156 83C104                  add ecx, 00000004 <--ECX+4,取数指针后移 4 个字节
:004AA159 81FE80000000            cmp esi, 00000080 <--取 0x80 次
:004AA15F 75D2                    jne 004AA133      <--循环(记为计算 2)
:004AA161 DF6DD8                  fild qword ptr [ebp-28]
:004AA164 83C4F4                  add esp, FFFFFFFF4
:004AA167 DB3C24                  fstp tbyte ptr [esp]
:004AA16A 9B                      wait
:004AA16B 8D8554FFFFFF            lea eax, dword ptr [ebp+FFFFFF54]
:004AA171 E8E6FEF5FF              call 0040A05C
:004AA176 8B9554FFFFFF            mov edx, dword ptr [ebp+FFFFFF54]
:004AA17C B8C4ED4A00              mov eax, 004AEDC4
:004AA181 E8A2A3F5FF              call 00404528
:004AA186 BB08000000              mov ebx, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004AA1FB(C)
|
:004AA18B 8D45CC                  lea eax, dword ptr [ebp-34]
:004AA18E 50                      push eax
:004AA18F B901000000              mov ecx, 00000001
:004AA194 8BD3                    mov edx, ebx
:004AA196 A1C4ED4A00              mov eax, dword ptr [004AEDC4]
:004AA19B E844A8F5FF              call 004049E4
:004AA1A0 8D45A8                  lea eax, dword ptr [ebp-58]
:004AA1A3 50                      push eax
:004AA1A4 8D7B02                  lea edi, dword ptr [ebx+02]
:004AA1A7 8BD7                    mov edx, edi
:004AA1A9 B901000000              mov ecx, 00000001
:004AA1AE A1C4ED4A00              mov eax, dword ptr [004AEDC4]
:004AA1B3 E82CA8F5FF              call 004049E4
:004AA1B8 B8C4ED4A00              mov eax, 004AEDC4
:004AA1BD B901000000              mov ecx, 00000001
:004AA1C2 8BD3                    mov edx, ebx
:004AA1C4 E85BA8F5FF              call 00404A24
:004AA1C9 BAC4ED4A00              mov edx, 004AEDC4
:004AA1CE 8BCB                    mov ecx, ebx
:004AA1D0 8B45A8                  mov eax, dword ptr [ebp-58]
:004AA1D3 E894A8F5FF              call 00404A6C
:004AA1D8 8BD7                    mov edx, edi
:004AA1DA B8C4ED4A00              mov eax, 004AEDC4
:004AA1DF B901000000              mov ecx, 00000001
:004AA1E4 E83BA8F5FF              call 00404A24
:004AA1E9 8BCF                    mov ecx, edi
:004AA1EB BAC4ED4A00              mov edx, 004AEDC4
:004AA1F0 8B45CC                  mov eax, dword ptr [ebp-34]
:004AA1F3 E874A8F5FF              call 00404A6C
:004AA1F8 4B                      dec ebx
:004AA1F9 85DB                    test ebx, ebx
:004AA1FB 758E                    jne 004AA18B <--又是一个循环计算,这里变态得很,每循环一次把计算结果的十位字符中的后两位向前移动一位,循环 8 次的结果是使最后两位插入到最前面,这就是最终结果了.
:004AA1FD 8D45B0                  lea eax, dword ptr [ebp-50]
:004AA200 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA206 E861A3F5FF              call 0040456C
:004AA20B 8B45FC                  mov eax, dword ptr [ebp-04]
:004AA20E E89D99FFFF              call 004A3BB0
```

内存中的一些数值: (没有贴全)

```
:004ACA20 70 50 30 10 96 30 07 77   pP0..0.w
:004ACA28 2C 61 0E EE BA 51 09 99   ,a...Q..
:004ACA30 19 C4 6D 07 8F F4 6B 70   ..m...kp
:004ACA38 35 A5 63 E9 A3 95 65 9E   5.c...e.
:004ACA40 32 88 DC 0E A4 E8 DC 79   2......y
:004ACA48 1E E9 D5 E0 88 D9 D2 97   ........
:004ACC20 20 83 B8 ED B6 B3 BF 9A    .......
:004ACC28 0C E2 B6 03 9A D2 B1 74   .......t
:004ACC30 39 47 D6 EA AF 77 D2 9D   9G...w..
:004ACC38 15 26 DB 04 83 16 DC 73   .&.....s
:004ACC40 12 0B 63 E3 84 3B 64 94   ..c..;d.
:004ACC48 3E 6A 6D 0D A8 5A 6A 7A   >jm..Zjz
:004ACC50 0B CF 0E E4 9D FF 09 93   ........
:004ACC58 27 AE 00 0A B1 9E 07 7D   '......}
:004ACC60 44 93 0F F0 D2 A3 08 87   D.......
```

下面是比较过程,大概作者觉得这样写比较隐蔽,我怎么觉得更明显了 :),反正一看到这里我就知道程序想干什么了...

```
:004AA30A 8B45B0                  mov eax, dword ptr [ebp-50]
:004AA30D 8A4002                  mov al, byte ptr [eax+02]
:004AA310 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA316 3A4202                  cmp al, byte ptr [edx+02]
:004AA319 0F85E5030000            jne 004AA704
:004AA31F 8B45B0                  mov eax, dword ptr [ebp-50]
:004AA322 8A00                    mov al, byte ptr [eax]
:004AA324 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA32A 3A02                    cmp al, byte ptr [edx]
:004AA32C 0F85D2030000            jne 004AA704
:004AA332 8B45B0                  mov eax, dword ptr [ebp-50]
:004AA335 8A4004                  mov al, byte ptr [eax+04]
:004AA338 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA33E 3A4204                  cmp al, byte ptr [edx+04]
:004AA341 0F85BD030000            jne 004AA704
:004AA347 8B45B0                  mov eax, dword ptr [ebp-50]
:004AA34A 8A4006                  mov al, byte ptr [eax+06]
:004AA34D 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
:004AA353 3A4206                  cmp al, byte ptr [edx+06]
:004AA356 0F85A8030000            jne 004AA704
:004AA35C 8B45B0                  mov eax, dword ptr [ebp-50]
:004AA35F 8A4003                  mov al, byte ptr [eax+03]
:004AA362 8B15C4ED4A00            mov edx, dword ptr [004AEDC4]
```
