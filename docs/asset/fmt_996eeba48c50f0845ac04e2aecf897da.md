
# 汇编代码分析

以下是对应汇编指令的详细列表及注释：

```
0040509E |. 8BC1 mov eax, ecx

004050A0 |. 8BF7 mov esi, edi

004050A2 |. BF 54B74000 mov edi, 0040B754

004050A7 |. C1E9 02 shr ecx, 2

004050AA |. F3:A5 rep movs dword ptr es:[edi], dword p>

004050AC |. 8BC8 mov ecx, eax

004050AE |. 33C0 xor eax, eax

004050B0 |. 83E1 03 and ecx, 3

004050B3 |. F3:A4 rep movs byte ptr es:[edi], byte ptr>
```

; 删除 `code_buf2` 中第 9 个字母

```
004050B5 |. BF 49B74000 mov edi, 0040B749 ; ASCII "AE7201C"

004050BA |. 83C9 FF or ecx, FFFFFFFF

004050BD |. F2:AE repne scas byte ptr es:[edi]

004050BF |. F7D1 not ecx

004050C1 |. 2BF9 sub edi, ecx

004050C3 |. 8BD1 mov edx, ecx

004050C5 |. 8BF7 mov esi, edi

004050C7 |. BF 5CB74000 mov edi, 0040B75C

004050CC |. C1E9 02 shr ecx, 2

004050CF |. F3:A5 rep movs dword ptr es:[edi], dword p>

004050D1 |. 8BCA mov ecx, edx

004050D3 |. 83E1 03 and ecx, 3

004050D6 |. 32DB xor bl, bl

004050D8 |. F3:A4 rep movs byte ptr es:[edi], byte ptr>;

004050DA |. BE 41B74000 mov esi, 0040B741 ; code_buf1

004050DF |. BF 54B74000 mov edi, 0040B754 ; code_buf2
```

; 要求注册码去掉第 9 个字母后的字符为大写字母或数字，然后两个一起当一个 16 进制字节来处理，
; 处理结果为 8 个字节，依次保存在 `code_buf1` 中

```
004050E4 |> 57 /push edi ; for(int i=0; i<8; i++) {esi[i] = strToex[edi] ^ (50h + i);}

004050E5 |. E8 E6010000 |call <StrtoHex>

004050EA |. 8ACB |mov cl, bl

004050EC |. 83C4 04 |add esp, 4

004050EF |. 80C1 50 |add cl, 50

004050F2 |. 83C7 02 |add edi, 2

004050F5 |. 32C1 |xor al, cl

004050F7 |. FEC3 |inc bl

004050F9 |. 8846 FF |mov byte ptr [esi-1], al

004050FC |. C606 00 |mov byte ptr [esi], 0

004050FF |. 46 |inc esi

00405100 |. 80FB 08 |cmp bl, 8

00405103 |.^ 72 DF \\jb short 004050E4

00405105 |. 68 54B74000 push 0040B754 ; code_buf2

0040510A |. 68 40B74000 push offset <序列号> ; b

0040510F |. E8 EC010000 call 00405300 ; 另一个处理序列号的函数，我们后面再分析
```

; 把用户名保存复制到一个全局缓冲区中，我们称之为 `name_buf2`，如果用户名长度 < 8，则两个用户名接起来
; 取前 8 个字母

```
00405114 |. BF C0C34000 mov edi, offset <用户名> ; a

00405119 |. 83C9 FF or ecx, FFFFFFFF

0040511C |. 33C0 xor eax, eax

0040511E |. 83C4 08 add esp, 8

00405121 |. F2:AE repne scas byte ptr es:[edi]

00405123 |. F7D1 not ecx

00405125 |. 2BF9 sub edi, ecx

00405127 |. 33ED xor ebp, ebp

00405129 |. 8BD1 mov edx, ecx

0040512B |. 8BF7 mov esi, edi

0040512D |. BF 5EB74000 mov edi, 0040B75E ; ASCII "qiufenghan"

00405132 |. C1E9 02 shr ecx, 2

00405135 |. F3:A5 rep movs dword ptr es:[edi], dword p>

00405137 |. 8BCA mov ecx, edx

00405139 |. 83E1 03 and ecx, 3

0040513C |. F3:A4 rep movs byte ptr es:[edi], byte ptr>

0040513E |. BF C0C34000 mov edi, offset <用户名> ; a

00405143 |. 83C9 FF or ecx, FFFFFFFF

00405146 |. F2:AE repne scas byte ptr es:[edi]

00405148 |. F7D1 not ecx

0040514A |. 49 dec ecx

0040514B |. 80F9 08 cmp cl, 8

0040514E |. 884C24 10 mov byte ptr [esp+10], cl

00405152 |. 73 30 jnb short 00405184 ; 用户名长度是否等于 8

00405154 |. 8B5424 10 mov edx, dword ptr [esp+10]

00405158 |. BF C0C34000 mov edi, offset <用户名> ; a

0040515D |. 81E2 FF000000 and edx, 0FF

00405163 |. 83C9 FF or ecx, FFFFFFFF

00405166 |. 81C2 5EB74000 add edx, 0040B75E ; ASCII "qiufenghan"

0040516C |. F2:AE repne scas byte ptr es:[edi]

0040516E |. F7D1 not ecx

00405170 |. 2BF9 sub edi, ecx

00405172 |. 8BC1 mov eax, ecx

00405174 |. 8BF7 mov esi, edi

00405176 |. 8BFA mov edi, edx

00405178 |. C1E9 02 shr ecx, 2

0040517B |. F3:A5 rep movs dword ptr es:[edi], dword p>

0040517D |. 8BC8 mov ecx, eax

0040517F |. 83E1 03 and ecx, 3

00405182 |. F3:A4 rep movs byte ptr es:[edi], byte ptr>; 如果用户名长度小于 8，则连接用户名。

00405184 |> C605 66B74000> mov byte ptr [40B766], 0 ; 取用户名的前 8 个字母
```

; 逻辑分析：
; for(i=0; i<8; i++)
; {
; if(`code_buf2[i]` >= `0x20`)
; `ebx` += `code_buf2[i]` ^ `name_buf2[i]`; //route1
; else
; `ebx` += `code_buf2[i]` | `name_buf2[i]`; //route2
; }
;
; 后面我们知道，要 `ebx` 等于 0 才算注册成功。`ebx` 的值为 8 个字节的和，不可能溢出
; 所以只能每个加数为 0 才可以
; 因为用户名中不可能有为 0 的字节，所以只要进入路径 2 中的值不可能为 0，
; 所以循环只能完全执行路径一，因为只有两个相同的数异或为 0，所以用户名中只能有小写
; 字母（字节值大于 20h 的字母）。其实就是说此时 `code_buf2` 的值就是用户名

```
0040518B |. B9 54B74000 mov ecx, 0040B754 ; code_buf2

00405190 |. BE 08000000 mov esi, 8

00405195 |> 8A01 /mov al, byte ptr [ecx]

00405197 |. 3C 20 |cmp al, 20

00405199 |. 73 0E |jnb short 004051A9

0040519B |. 33D2 |xor edx, edx

0040519D |. 25 FF000000 |and eax, 0FF

004051A2 |. 8A51 0A |mov dl, byte ptr [ecx+A]

004051A5 |. 0BD0 |or edx, eax

004051A7 |. EB 0C |jmp short 004051B5

004051A9 |> 33D2 |xor edx, edx

004051AB |. 25 FF000000 |and eax, 0FF

004051B0 |. 8A51 0A |mov dl, byte ptr [ecx+A]

004051B3 |. 33D0 |xor edx, eax

004051B5 |> 03EA |add ebp, edx

004051B7 |. 41 |inc ecx

004051B8 |. 4E |dec esi

004051B9 |.^ 75 DA \\jnz short 00405195
```

; 如果 `ebp` 为 0，则认为注册成功

```
004051BB |. 33C0 xor eax, eax

004051BD |. 5F pop edi

004051BE |. 85ED test ebp, ebp

004051C0 |. 5E pop esi

004051C1 |. 5D pop ebp

004051C2 0F94C0 sete al

004051C5 |. 5B pop ebx

004051C6 |. 59 pop ecx

004051C7 \\. C3 retn
```

## 8、先在我们看看函数 `00405300` 做什么事

```
00405300 /$ 53 push ebx

00405301 |. 55 push ebp

00405302 |. 8B6C24 10 mov ebp, dword ptr [esp+10] ; arg2

00405306 |. 56 push esi

00405307 |. 57 push edi

00405308 |. 8B7C24 14 mov edi, dword ptr [esp+14] ; arg1

0040530C |. 33C9 xor ecx, ecx

0040530E |. 2BFD sub edi, ebp

00405310 |. 897C24 18 mov dword ptr [esp+18], edi ; arg2

00405314 |. EB 04 jmp short 0040531A

00405316 |> 8B7C24 18 /mov edi, dword ptr [esp+18]

0040531A |> 8D3429 lea esi, dword ptr [ecx+ebp]

0040531D |. 33D2 |xor edx, edx

0040531F |. B8 01000000 |mov eax, 1
```
