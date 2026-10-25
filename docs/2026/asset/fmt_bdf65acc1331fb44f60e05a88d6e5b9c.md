
```assembly
00401159  |. |E8 AE000000   call    0040120C                     ;  关键过程
0040115E  |. |8BC8          mov     ecx, eax
00401160  |. |81E1 FF0F0000 and     ecx, 0FFF
00401166  |. |66:83F9 00    cmp     cx, 0
0040116A  |. |76 13         jbe     short 0040117F
0040116C  |. |6A 10         push    10                           ; /Style = MB\_OK|MB\_ICONHAND|MB\_APPLMODAL
0040116E  |. |6A 00         push    0                            ; |Title = NULL
00401170  |. |68 84304000   push    00403084                     ; |Text = "Invalid input or imagebase already allocated!"
00401175  |. |FF75 08       push    dword ptr [ebp+8]            ; |hOwner
00401178  |. |E8 69000000   call    <jmp.&user32.MessageBoxA>    ; \MessageBoxA
0040117D  |. |EB 3D         jmp     short 004011BC
0040117F  |> |A3 30334000   mov     dword ptr [403330], eax      ;  输入的基地址被保存在这里
00401184  |. |6A 04         push    4                            ; /Protect = PAGE\_READWRITE    此处的属性值需要注意[没有可执行属性]    这就是断点的原因
00401186  |. |68 00200000   push    2000                         ; |AllocationType = MEM\_RESERVE
0040118B  |. |68 00000100   push    10000                        ; |Size = 10000 (65536.)
00401190  |. |FF35 30334000 push    dword ptr [403330]           ; |Address = 10000000
00401196  |. |E8 69000000   call    <jmp.&kernel32.VirtualAlloc> ; \VirtualAlloc
0040119B  |. |0BC0          or      eax, eax                     ;  利用VirtualAlloc指定DLL初始化的地址
0040119D  |. |75 13         jnz     short 004011B2
```

注意`00401184`处，关键就在这个`VirtualAlloc`函数上，他设置的属性中不包括执行属性。

上面的分析是为了找出`VirtualAlloc`处的`Address`是从何而来。

当`VirtualAlloc`未设置可执行属性时，程序用`LoadLibraryA`载入DLL时就会产生异常，也就被中断在DLL的入口。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
