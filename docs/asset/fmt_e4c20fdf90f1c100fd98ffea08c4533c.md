
```asm
/*711E6923*/  sub     ecx, 3000
/*711E6929*/  add     eax, ecx
/*711E692B*/  add     eax, [ebp-20]
/*711E692E*/  dec     eax
/*711E692F*/  sub     edx, eax
/*711E6931*/  sub     edx, 4
/*711E6934*/  mov     [eax], edx
/*711E6936*/  jmp     711E6A9D
/*711E693B*/  call    711DBBD0 // 同样申请内存
/*711E6940*/  mov     word ptr [eax], 25FF // 这里有所不同，应为 call [xxxx] 和 jmp [xxxx] 都在这里处理了，所以我们要先判断。
```

修改为

```asm
cmp     byte ptr [ebx+1], 0E9
je      short 711E6948
mov     word ptr [ebx], 15FF // 是 call [xxxx]
jmp     short 711E694D
mov     word ptr [ebx], 25FF  // 是 jmp [xxxx]
mov     [ebx+2], esi
jmp     711E6A9D
```

```asm
/*711E6945*/  mov     [eax+2], esi
/*711E6948*/  mov     edx, eax
/*711E694A*/  mov     ebx, [ebp-10]
/*711E694D*/  add     ebx, 6
/*711E6950*/  mov     eax, [ebp-18]
/*711E6953*/  add     eax, eax
/*711E6955*/  add     ebx, eax
/*711E6957*/  mov     eax, [ebp-10]
/*711E695A*/  mov     eax, [eax]
/*711E695C*/  movzx   ecx, word ptr [ebx]
/*711E695F*/  sub     ecx, 3000
/*711E6965*/  add     eax, ecx
/*711E6967*/  add     eax, [ebp-20]
/*711E696A*/  sub     edx, eax
/*711E696C*/  sub     edx, 4
/*711E696F*/  mov     [eax], edx
/*711E6971*/  jmp     711E6A9D
/*711E6976*/  mov     edx, [711FFD94]
/*711E697C*/  mov     edx, [edx]
```

下面还有一些类型的加密，包括 `MOV`，但是我们不需要了解其详细，直接命中要害。

```asm
/*711E69BA*/  cmp     ax, 500
/*711E69BE*/  je      short 711E6A0C
/*711E69C0*/  cmp     ax, 1500
/*711E69C4*/  je      short 711E6A0C
/*711E69C6*/  cmp     ax, 2500
/*711E69CA*/  je      short 711E6A0C
/*711E69CC*/  cmp     ax, 3500
/*711E69D0*/  je      short 711E6A0C
/*711E69D2*/  cmp     ax, 0D00
/*711E69D6*/  je      short 711E6A0C
/*711E69D8*/  cmp     ax, 1D00
/*711E69DC*/  je      short 711E6A0C
/*711E69DE*/  cmp     ax, 2D00
/*711E69E2*/  je      short 711E6A0C
/*711E69E4*/  cmp     ax, 3D00
/*711E69E8*/  je      short 711E6A0C
/*711E69EA*/  mov     ax, [ebx]
/*711E69ED*/  and     ax, 0FF00
/*711E69F1*/  cmp     ax, 8000
/*711E69F5*/  jb      711E6A9D
/*711E69FB*/  mov     ax, [ebx]
/*711E69FE*/  and     ax, 0FF00
/*711E6A02*/  cmp     ax, 0C000
/*711E6A06*/  jnb     711E6A9D
/*711E6A0C*/  mov     ax, [ebx]
/*711E6A0F*/  and     ax, 0FF
/*711E6A13*/  cmp     ax, 89
/*711E6A17*/  je      short 711E6A26
/*711E6A19*/  mov     ax, [ebx]
/*711E6A1C*/  and     ax, 0FF
/*711E6A20*/  cmp     ax, 8B
/*711E6A24*/  jnz     short 711E6A9D
/*711E6A26*/  call    711DBBD0
/*711E6A2B*/  mov     dx, [ebx]
/*711E6A2E*/  and     dx, 0FF
/*711E6A33*/  cmp     dx, 89
/*711E6A38*/  jnz     short 711E6A4C
/*711E6A3A*/  mov     dx, [ebx]
/*711E6A3D*/  and     dx, 0FF00
/*711E6A42*/  add     dx, 8B
/*711E6A47*/  mov     [eax], dx
/*711E6A4A*/  jmp     short 711E6A5C
/*711E6A4C*/  mov     dx, [ebx]
/*711E6A4F*/  and     dx, 0FF00
/*711E6A54*/  add     dx, 89
/*711E6A59*/  mov     [eax], dx // 这里写入加密代码。我们在这里动手
```

修改为 `mov [ebx], dx`

```asm
/*711E6A5C*/  mov     [eax+2], esi
```

修改为 `mov [ebx+2], esi`

于是乎。基本信息处理完毕。现在往下找 `OEP`。

```asm
/*711E6A5F*/  mov     word ptr [eax+6], 0E990
/*711E6A65*/  mov     edx, ebx
/*711E6A67*/  sub     edx, eax
/*711E6A69*/  sub     edx, 6
/*711E6A6C*/  mov     [eax+8], edx
/*711E6A6F*/  mov     edx, eax
/*711E6A71*/  mov     word ptr [ebx], 0E990
/*711E6A76*/  mov     ebx, [ebp-10]
```

到达

```asm
/*711E4DEE*/  cmp     eax, 0 // 判断 OEP 是否为 0, 是就出错。
/*711E4DF1*/  je      short 711E4E45
/*711E4DF3*/  mov     bl, [7121272B] // 是否是 S 加密方式。
/*711E4DF9*/  call    V2200611.05007EC0
/*711E4DFE*/  xor     eax, FFFFFFFF // 如果是，解密 OEP
/*711E4E03*/  mov     [esp+34], eax
/*711E4E07*/  add     esp, 10
/*711E4E0A*/  pop     dword ptr fs:[0]
/*711E4E11*/  pop     eax
/*711E4E12*/  and     eax, 100
/*711E4E17*/  cmp     eax, 100
/*711E4E1C*/  je      short 711E4E45
/*711E4E1E*/  call    711DB074 // 飞向光明之颠。（盗版一下 fly 老大的。）
/*711E4E23*/  jmp     short 711E4E45
/*711E4E25*/  mov     [esp+34], eax // 如果不是，那么用 4 个 int3 通知 explorer 里面的处理过程，OEP 到了。OEP 加密方式，仍然为 xor oep, ffffffff
/*711E4E29*/  add     esp, 10
/*711E4E2C*/  pop     dword ptr fs:[0]
/*711E4E33*/  pop     eax
/*711E4E34*/  and     eax, 100
/*711E4E39*/  cmp     eax, 100
/*711E4E3E*/  je      short 711E4E45
/*711E4E40*/  call    711DB150
/*711E4E45*/  call    711DD288
/*711E4E4A*/  push    10
/*711E4E4C*/  mov     eax, [711FFBB4]
```

于是乎，强制结束程序，然后再运行。`dump`，修复 `IAT`，优化，收工。有 `SDK` 的，记得再在 `IAT` 添加一个 `v22006114.epe` 如果有。没有就用原版的 `v22006115.epe` 附上主脱后的主程序和修改后的 `dll`。

附件太大。。。我分。

`4dd466d0e1e0cc123afbe504da2462ca`
