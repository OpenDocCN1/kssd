
# 恶意代码分析

## 动态加载机制

使用 `LoadLibraryA` 加载 `kernel32.dll`，然后在 `kernel32.dll` 中查找 `CreateToolhelp32Snapshot`、`Process32First`、`Process32Next`、`CloseHandle` 这四个函数的地址。

得到这四个函数的地址之后：

- 分别调用 `CreateToolhelp32Snapshot`、`Process32First`、`Process32Next` 进行进程查找，看有没有 `my.exe` 的进程。
- 如果没有，就调用 `CloseHandle` 关闭句柄。

这样程序刚开始的那个 `CALL 00402A66` 到这里就分析完成了。

## 内存分配与关键调用分析

接着前面的分析：

```asm
0040370F  |.  BE 04010000    mov     esi, 104
00403714  |.  56             push    esi                    ; /size => 104 (260.)
00403715  |.  FF15 78504000  call    dword ptr [<&msvcrt.malloc>] ; \malloc
0040371B  |.  8BF8           mov     edi, eax
0040371D  |.  56             push    esi                    ; /n
0040371E  |.  6A 00          push    0                      ; |c = 00
00403720  |.  57             push    edi                    ; |s
00403721  |.  E8 EC0F0000    call    <jmp.&msvcrt.memset>           ; \memset
00403726  |.  6A 00          push    0
00403728  |.  57             push    edi
00403729  |.  6A 06          push    6
0040372B  |.  E8 E4F3FFFF    call    00402B14      --------->关键 CALL
00403730  |.  57             push    edi                    ; /block
00403731  |.  FF15 74504000  call    dword ptr [<&msvcrt.free>]    ; \free
```

这段代码先通过 `malloc` 分配一段内存，然后用 `memset` 将这段内存清零，接着调用关键函数 `CALL 00402B14` 进行一些操作，最后调用 `free` 释放前面分配的内存。

### 关键函数 CALL 00402B14 分析

`CALL 00402B14` 是主程序的关键，代码较长。以下分析主要流程，分支操作仅简要介绍：

```asm
00402B14  /$  55             push    ebp
00402B15  |.  8BEC           mov     ebp, esp
00402B17  |.  81EC 980A0000 sub     esp, 0A98
00402B1D  |.  53             push    ebx
00402B1E  |.  80A5 7CFAFFFF>and     byte ptr [ebp-584], 0
00402B25  |.  56             push    esi
00402B26  |.  57             push    edi
00402B27  |.  6A 40          push    40
00402B29  |.  33C0           xor     eax, eax
00402B2B  |.  5A             pop     edx
00402B2C  |.  8DBD 7DFAFFFF lea     edi, dword ptr [ebp-583]       ;  edi = "88"
00402B32  |.  8BCA           mov     ecx, edx                       ;  ecx = edx = 40
00402B34  |.  80A5 6CF6FFFF>and     byte ptr [ebp-994], 0
00402B3B  |.  F3:AB         rep     stos dword ptr es:[edi]            ;  将指定地址处数据清零
00402B3D  |.  66:AB         stos    word ptr es:[edi]
00402B3F  |.  AA             stos    byte ptr es:[edi]
00402B40  |.  8BCA           mov     ecx, edx
00402B42  |.  33C0           xor     eax, eax
00402B44  |.  8DBD 6DF6FFFF lea     edi, dword ptr [ebp-993]
00402B4A  |.  80A5 4CFEFFFF>and     byte ptr [ebp-1B4], 0
00402B51  |.  F3:AB         rep     stos dword ptr es:[edi]
00402B53  |.  66:AB         stos    word ptr es:[edi]
00402B55  |.  AA             stos    byte ptr es:[edi]
00402B56  |.  8BCA           mov     ecx, edx
00402B58  |.  33C0           xor     eax, eax
......
00402BD0  |.  8DBD 79F9FFFF lea     edi, dword ptr [ebp-687]
00402BD6  |.  6A 10          push    10
00402BD8  |.  F3:AB         rep     stos dword ptr es:[edi]
00402BDA  |.  66:AB         stos    word ptr es:[edi]
00402BDC  |.  AA             stos    byte ptr es:[edi]
```

上面的代码将 `ebp` 相对应的一些位置的数据清零，可能用于存放新的数据。

```asm
00402BDE  |.  BE 70604000    mov     esi, 00406070
00402BE3  |.  8DBD 88FDFFFF lea     edi, dword ptr [ebp-278]
00402BE9  |.  C645 A8 43     mov     byte ptr [ebp-58], 43
00402BED  |.  F3:A5         rep     movs dword ptr es:[edi], dword ptr [esi]
```

这里将 `00406070` 处的数据拷贝到 `0012FC80` 处。

`00406070` 处存储的是关于梦幻西游的安装信息（基于截图分析）。

继续单步执行，当执行完 `00402BED` 后，会将注册表中的关于梦幻西游的安装信息拷贝到 `0012FC80` 的堆栈中。

## 字符串混淆与资源查找

再单步执行，来到如下代码处：

```asm
00402BEF  |.  C645 A9 6F    mov     byte ptr [ebp-57], 6F
00402BF3  |.  C645 AA 70    mov     byte ptr [ebp-56], 70
00402BF7  |.  C645 AB 79    mov     byte ptr [ebp-55], 79
00402BFB  |.  C645 AC 46    mov     byte ptr [ebp-54], 46
00402BFF  |.  C645 AD 69    mov     byte ptr [ebp-53], 69
00402C03  |.  C645 AE 6C    mov     byte ptr [ebp-52], 6C
00402C07  |.  C645 AF 65    mov     byte ptr [ebp-51], 65
00402C0B  |.  C645 B0 41    mov     byte ptr [ebp-50], 41          ;  CopyFileA
00402C0F  |.  A4             movs    byte ptr es:[edi], byte ptr [esi]
00402C10  |.  C645 8C 49    mov     byte ptr [ebp-74], 49
00402C14  |.  C645 8D 6E    mov     byte ptr [ebp-73], 6E
00402C18  |.  C645 8E 73    mov     byte ptr [ebp-72], 73
00402C1C  |.  8B7D 0C        mov     edi, dword ptr [ebp+C]
00402C1F  |.  8065 9B 00    and     byte ptr [ebp-65], 0
00402C23  |.  8065 BB 00    and     byte ptr [ebp-45], 0
00402C27  |.  8065 C1 00    and     byte ptr [ebp-3F], 0
00402C2B  |.  8D45 BC       lea     eax, dword ptr [ebp-44]
00402C2E  |.  6A 76          push    76
00402C30  |.  50             push    eax
00402C31  |.  57             push    edi
00402C32  |.  C645 8F 74    mov     byte ptr [ebp-71], 74
00402C36  |.  C645 90 61    mov     byte ptr [ebp-70], 61
00402C3A  |.  C645 91 6C    mov     byte ptr [ebp-6F], 6C
00402C3E  |.  C645 92 6C    mov     byte ptr [ebp-6E], 6C
00402C42  |.  C645 93 4C    mov     byte ptr [ebp-6D], 4C
00402C46  |.  C645 94 6F    mov     byte ptr [ebp-6C], 6F
00402C4A  |.  C645 95 63    mov     byte ptr [ebp-6B], 63
00402C4E  |.  C645 96 61    mov     byte ptr [ebp-6A], 61
00402C52  |.  C645 97 74    mov     byte ptr [ebp-69], 74
00402C56  |.  C645 98 69    mov     byte ptr [ebp-68], 69
00402C5A  |.  C645 99 6F    mov     byte ptr [ebp-67], 6F
00402C5E  |.  C645 9A 6E    mov     byte ptr [ebp-66], 6E          ;  InstallLocation
00402C62  |.  C645 B4 5C    mov     byte ptr [ebp-4C], 5C
00402C66  |.  C645 B5 6D    mov     byte ptr [ebp-4B], 6D
00402C6A  |.  C645 B6 79    mov     byte ptr [ebp-4A], 79
00402C6E  |.  C645 B7 2E    mov     byte ptr [ebp-49], 2E
00402C72  |.  C645 B8 65    mov     byte ptr [ebp-48], 65
00402C76  |.  C645 B9 78    mov     byte ptr [ebp-47], 78
00402C7A  |.  C645 BA 65    mov     byte ptr [ebp-46], 65
00402C7E  |.  C645 BC 6D    mov     byte ptr [ebp-44], 6D
00402C82  |.  C645 BD 73    mov     byte ptr [ebp-43], 73
00402C86  |.  C645 BE 63    mov     byte ptr [ebp-42], 63
00402C8A  |.  C645 BF 72    mov     byte ptr [ebp-41], 72
00402C8E  |.  C645 C0 6F    mov     byte ptr [ebp-40], 6F          ;  \my.exe.mscro
00402C92  |.  E8 C7F8FFFF   call    0040255E
```

上面的代码是将函数名（如 `CopyFileA`、`InstallLocation` 等）的字母逐个压入堆栈以组合成完整函数名，这通常是反病毒逃避技术（免杀）。

在进行完这种操作之后，调用了一个函数 `CALL 0040255E`。

## CALL 0040255E 功能分析

`CALL 0040255E` 进行了以下工作：

1. 首先用 `LoadLibraryA` 加载 `kernel32.dll` 动态链接库。
2. 得到如下字符串：
   `GetTempPathA`, `FindResourceA`, `SizeOfResource`, `LoadResource`, `LockResource`, `FreeResource`
3. 调用函数获取上述函数的地址，还得到 `SetHandleCount` 的函数地址。
4. 调用 `GetTempPath` 获得临时目录。
5. 调用 `srand`、`time`、`rand` 等函数得到一个随机的文件名。
6. 调用 `GetModuleHandleA`。
7. 调用 `FindResourceA` 和 `LoadResource` 来查找程序中的资源。
8. 调用 `SetHandleCount`。
9. 最后调用 `SizeOfResource` 函数计算资源的大小。
