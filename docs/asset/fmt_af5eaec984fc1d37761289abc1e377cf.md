
`0040120D`  |.  `8BEC`          `mov`     `ebp, esp`                                 ;  \[调试到此处是的堆栈显示如下\]  

`0012FBBC`  /0012FBC8  塞.  

`0012FBC0`  |`0040115E`  ^@.  `RETURN` to `LoadDll.0040115E` from `LoadDll.0040120C`  

`0012FBC4`  |`00403334`  43@.  `ASCII` "10000000"  

`0012FBC8`  \]0012FBF4  酐.  

这说明`[EBP+8]`是存放输入的基地址的地方  

\===============================================================================================================  

从 DLL 关键过程出来后,

```
00401152  |. /EB 68         jmp     short 004011BC
00401154  |> |68 34334000   push    00403334                     ;  ASCII "10000000"
00401159  |. |E8 AE000000   call    0040120C                     ;  关键过程
0040115E  |. |8BC8          mov     ecx, eax
00401160  |. |81E1 FF0F0000 and     ecx, 0FFF
00401166  |. |66:83F9 00    cmp     cx, 0
0040116A  |. |76 13         jbe     short 0040117F
0040116C  |. |6A 10         push    10                           ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
0040116E  |. |6A 00         push    0                            ; |Title = NULL
00401170  |. |68 84304000   push    00403084                     ; |Text = "Invalid input or imagebase already allocated!"
00401175  |. |FF75 08       push    dword ptr [ebp+8]            ; |hOwner
00401178  |. |E8 69000000   call    <j mp.&user32.MessageBoxA>    ; \\MessageBoxA
0040117D  |. |EB 3D         jmp     short 004011BC
0040117F  |> |A3 30334000   mov     dword ptr [403330], eax      ;  输入的基地址被保存在这里
00401184  |. |6A 04         push    4                            ; /Protect = PAGE_READWRITE    此处的属性值需要注意[没有可执行属性]    这就是断点的原因
00401186  |. |68 00200000   push    2000                         ; |AllocationType = MEM_RESERVE
0040118B  |. |68 00000100   push    10000                        ; |Size = 10000 (65536.)
00401190  |. |FF35 30334000 push    dword ptr [403330]           ; |Address = 10000000
00401196  |. |E8 69000000   call    <j mp.&kernel32.VirtualAlloc> ; \\VirtualAlloc
0040119B  |. |0BC0          or      eax, eax                     ;  利用 VirtualAlloc 指定 DLL 初始化的地址
0040119D  |. |75 13         jnz     short 004011B2
```

注意 `00401184` 处,关键就在这个 `VirtualAlloc` 函数上,他设置的属性中不包括执行属性  

上面的分析是为了找出 `VirtualAlloc` 处的 `Address` 是从何而来  

当 `VirtualAlloc` 未设置可执行属性时,程序用 `LoadLibraryA` 载入 DLL 时就会产生异常,也就被中断在 DLL 的入口. 

如果大家想要打造自己的 `Loaddll.exe`，只需要将原版 `Loaddll.exe` 修改一下就可以了！  

原版 `Loaddll.exe`  

\[未修改第一处\]

```
004100AE  |> \\56            push    esi                              ; /FileName
004100AF  |.  E8 FC090000   call    <j mp.&KERNEL32.LoadLibraryA>     ; \\LoadLibraryA
```

\[未修改第二处\]

```
00410298 >    90            nop
00410299      90            nop
0041029A      90            nop
0041029B      90            nop
0041029C      90            nop
0041029D      90            nop
0041029E      90            nop
0041029F      90            nop
004102A0      90            nop
004102A1      90            nop
```

修改后的 `Loaddll.exe`  

\========================================================  

\[修改第一处\]

```
004100AE  |> \\90            nop
004100AF  |.  E9 E4010000   jmp     Patcharea
004100B4  |>  0BC0          or      eax, eax
004100B6  |.  75 0F         jnz     short 004100C7
```

\[修改第二处\]

```
00410298 >|$  6A 04         push    4                                ; /Protect = PAGE_READWRITE
0041029A  |.  68 00200000   push    2000                             ; |AllocationType = MEM_RESERVE
0041029F  |.  68 00000100   push    10000                            ; |Size = 10000 (65536.)
004102A4  |.  68 00000010   push    10000000                         ; |Address = 10000000 [此处地址自行指定]
004102A9  |.  E8 43983F7C   call    kernel32.VirtualAlloc            ; \\VirtualAlloc
004102AE  |.  56            push    esi                              ; /FileName
004102AF  |.  E8 FC070000   call    <j mp.&KERNEL32.LoadLibraryA>     ; \\LoadLibraryA
004102B4  \\.^ E9 FBFDFFFF   jmp     004100B4
```
