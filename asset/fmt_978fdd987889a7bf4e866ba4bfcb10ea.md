
# ACDSee3.0

> 原文：[`www.pediy.com/kssd/pediy01/forum096.htm`](https://www.pediy.com/kssd/pediy01/forum096.htm)

初学破解，一点心得，希望与大家交流。

破解 `ACDSee3.0` 如下：

使用 `TRW` 找到 `ACDSee!.ASPACK` 的最后地址为：`5554CC`

相关为：

```
005554B6 89 85 5C 2E 44 00     mov        dword ptr [ebp+442E5Ch],eax
005554BC 61                     popad
005554BD 75 08                  jne        005554C7
005554BF B8 01 00 00 00         mov        eax,1
005554C4 C2 0C 00              ret        0Ch
005554C7 68 DF 32 4A 00        push       4A32DFh
005554CC C3                     ret
```

`mov        dword ptr [ebp+442E5Ch],eax` 将 `dword ptr [5554C8]` 从 0 改到 4A32DF

`ACDSee!.TEXT` 第一条指令为：

```
004A32DF 55                     push       ebp
004A32E0 8B EC                  mov        ebp,esp
004A32E2 6A FF                  push       0FFh
004A32E4 68 A8 69 4D 00        push       4D69A8h
004A32E9 68 80 81 4A 00        push       4A8180h
```

做以下修改：

1. 修改 `5554B6` 为，`RawAddress = 864B6`（文件中静态地址）

```
005554B6 E9 15 A1 01 00         jmp        0056F5D0
```

2. 修改 `5550FD` 为，`RawAddress = A05D0`

```
0056F5D0 89 85 5C 2E 44 00     mov        dword ptr [ebp+442E5Ch],eax
0056F5D6 BB 30 71 41 00        mov        ebx,417130h
0056F5DB C7 03 B8 01 00 00     mov        dword ptr [ebx],1B8h
0056F5E1 BB 34 71 41 00        mov        ebx,417134h
0056F5E6 C7 03 00 C3 90 90     mov        dword ptr [ebx],9090C300h
0056F5EC BB EA 3F 43 00        mov        ebx,433FEAh
0056F5F1 66 C7 03 EB 17        mov        word ptr [ebx],17EBh
0056F5F6 E9 C1 5E FE FF        jmp        005554BC
```

3. 其中 `417130` 使 About 弹出信息为注册

原来是：

```
00417130 33 C0                  xor        eax,eax      ;返回 0 认为没有注册
00417132 C3                     ret
00417133 90                     nop
00417134 90                     nop
00417135 90                     nop
00417136 90                     nop
```

改后为：

```
00417130 B8 01 00 00 00        mov        eax,1        ;返回 1 认为已经注册
00417135 C3                     ret
00417136 90                     nop
```

4. `433FEA` 使软件不过期

原来是：

```
00433FEA 6A 00                  push       0
00433FEC 68 10 40 43 00        push       434010h
00433FF1 50                     push       eax
00433FF2 68 07 04 00 00        push       407h
00433FF7 51                     push       ecx
00433FF8 FF 15 A4 E7 4C 00     call       dword ptr ds:[4CE7A4h]
00433FFE 48                     dec        eax
00433FFF F7 D8                  neg        eax
00434001 1B C0                  sbb        eax,eax
00434003 40                     inc        eax
00434004 C3                     ret
```

改后为：

```
00433FEA EB 17                  jmp        00434003     ;跳过过期对话框
00433FEC 68 10 40 43 00        push       434010h
00433FF1 50                     push       eax
00433FF2 68 07 04 00 00        push       407h
00433FF7 51                     push       ecx
00433FF8 FF 15 A4 E7 4C 00     call       dword ptr ds:[4CE7A4h]
00433FFE 48                     dec        eax
00433FFF F7 D8                  neg        eax
00434001 1B C0                  sbb        eax,eax
00434003 40                     inc        eax
00434004 C3                     ret
```

按照这种思路，基本上破解任何加壳的软件都不需要使用专门的脱壳软件。仅仅在壳对本体程序代码解密完成后，做 `JMP` 或 `CALL` 到自己代码，将相关的代码修改成自己想要的，然后再回到原来代码执行，就能完成任务。在 `ACDSee3.0` 的 `.text` 块的属性是可写的，所以不会产生写错误。

跳过过期画面的处理参考 `baoleigz` 的《也谈 ACDSee V3.0 破解》。
