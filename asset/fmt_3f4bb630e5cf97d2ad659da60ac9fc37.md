
### 恢复代码

```
mov r15, [sr15_1]
iretq
```

### .TASK2:

```
inc [ctrl]
pop [srax1]
mov [srbx1], rbx
mov [srcx1], rcx
mov [srdx1], rdx
mov [srbp1], rbp
mov [srsi1], rsi
mov [srdi1], rdi
mov [sr81], r8
mov [sr91], r9
mov [sr10_1], r10
mov [sr11_1], r11
mov [sr12_1], r12
mov [sr13_1], r13
mov [sr14_1], r14
mov [sr15_1], r15
pop [srip_1]
pop rax
pop [sr***_1]
pop [srsp1]
push [srsp]
push [sr***]
push rax
push [srip]
mov rax, [srax]
mov rbx, [srbx]
mov rcx, [srcx]
mov rdx, [srdx]
mov rbp, [srbp]
mov rsi, [srsi]
mov rdi, [srdi]
mov r8, [sr8]
mov r9, [sr9]
mov r10, [sr10]
mov r11, [sr11]
mov r12, [sr12]
mov r13, [sr13]
mov r14, [sr14]
mov r15, [sr15]
iretq
```

### DUMP_RAX:

```
push rax
push rbx
push rcx
push rdx
mov ebx, 0xB8000 + 2 * ( 0 * 25 + 80 - 10 )
mov dh, 0xC
mov dl, '0'
mov [ebx], dx
inc ebx
inc ebx
mov dl, 'x'
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 28
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 24
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 20
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 16
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 12
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 8
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
shr ecx, 4
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
inc ebx
inc ebx
mov ecx, eax
and ecx, 0x0000000F
mov dl, byte [HEX_TABLE + ecx]
mov [ebx], dx
pop rdx
pop rcx
pop rbx
pop rax
ret
```

////////////////////////////////////////////////

欢迎大家有空到我的博客：[`blog.sina.com.cn/tigerdeng`](http://blog.sina.com.cn/tigerdeng) 来逛逛啊。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || drboot1.rar |
