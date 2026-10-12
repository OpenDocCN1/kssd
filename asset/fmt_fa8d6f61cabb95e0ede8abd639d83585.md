
```
0040D308  ^ 75 FB           jnz     short 0040D305  
0040D30A    AD               lods    dword ptr [esi]  
0040D30B    85C0             test    eax, eax  
0040D30D  ^ 74 E6            je      short 0040D2F5  
0040D30F    83EE 04          sub     esi, 4  
0040D312    AD               lods    dword ptr [esi]                              ; 取 LoadLibraryA  
0040D313    A9 00000080      test    eax, 80000000  
0040D318    75 0B            jnz     short 0040D325  
0040D31A    83EE 04          sub     esi, 4  
0040D31D    56               push    esi  
0040D31E    53               push    ebx  
0040D31F    FF55 F8          call    dword ptr [ebp-8]                             ; GetProcAddress  
0040D322    AB               stos    dword ptr es:[edi]  
0040D323  ^ EB E0            jmp     short 0040D305  
0040D325    25 FFFFFF7F      and     eax, 7FFFFFFF  
0040D32A    50               push    eax  
0040D32B    53               push    ebx  
0040D32C    FF55 F8          call    dword ptr [ebp-8]  
0040D32F    AB               stos    dword ptr es:[edi]  
0040D330  ^ EB D8            jmp     short 0040D30A  
0040D332    5D               pop     ebp  
0040D333    5F               pop     edi  
0040D334    C3               retn  
```

这段代码大家很熟悉，重建输入表，这里是将重建的输入表存放在了 `00405000` 处了~~

![](img/d1bc00950cc57626ba4775f78c78e580.jpg)

在 `0040D334` 这里，程序返回到了如下地址处(系统模块 `kernel32.dll` 中)：

```
7C809B74 >  8BFF              mov     edi, edi  
7C809B76    55               push    ebp  
7C809B77    8BEC             mov     ebp, esp                              ; 00400000  
7C809B79    FF75 10          push    dword ptr [ebp+10]                    ; 8000  
7C809B7C    FF75 0C          push    dword ptr [ebp+C]  
7C809B7F    FF75 08          push    dword ptr [ebp+8]                     ; 3A0000  
7C809B82    6A FF            push    -1  
7C809B84    E8 09000000      call    VirtualFreeEx  
7C809B89    5D               pop     ebp  
7C809B8A    C2 0C00          retn    0C  
```

这里的 `8000` = `1000` + `1000` + `1000` + `5000`(这些内存是在 `380000` 的代码中执行完成的)，释放分配在 `3A0000` 中的 `8000` 的内存空间

上面的 `retn` 返回后，单步就可以来到 `OEP` 了，嘿嘿~~~因为前面的工作都已经做完了，前面的工作，主要有一处就是在执行 `380000` 的代码时不清楚，还请高人指点~~~

```
00404794    55               push    ebp        ----->OEP  
00404795    8BEC             mov     ebp, esp  
00404797    6A FF            push    -1  
00404799    68 E8504000      push    004050E8  
0040479E    68 00474000      push    00404700                       ; jmp 到 MSVCRT._except_handler3  
```

同时有两种比较快的方式达到 `OEP`，上面的分析对于刚入门的同学可能不是很懂，以后在慢慢学习，吸收吧~~~

第一种：就是凭直觉，发现单步执行程序到

```
0040D06F    60               pushad  
0040D070    83EC 38          sub     esp, 38  
```

这里时，可以看到上面有一条语句：

```
0040D06D    FFE0             jmp     eax      ----->直接在这里 F2 下断，然后 F9,来到这里，F7 进入即可来到 OEP!  
```

第二种方法：就是上一篇文章有人留言说可以用 `ESP` 定律法达到 `OEP`，这里我试了试，可以~~单步来到如下地址

```
0040D06F    60               pushad  
0040D070    83EC 38          sub     esp, 38    ---->单步到这里时，用 ESP 定律即可，不会 ESP 定律的不要问我了!  
```

同样可以快速达到 `OEP`!

这样我们就对这个样本的外壳分析完成了，下面就是脱壳了，脱壳我就不讲了，直接用 `OD` 插件或用 `LordPE`+`ImportREC`,随你便!

脱壳中.........

脱壳完成了，嘿嘿~~非常顺利!脱壳后的文件命名为 `Cracker.exe`

休息了半个小时~~~

继续吧！人生不是在不停的忙碌，忙碌，再忙碌~~

## 第二个阶段：主程序 `Cracker.exe` 的分析

首先查壳：`PEID` 查得是 `Visual C++6.0`(其实我也很喜欢用 `VC6.0`,老是用不习惯 `VS2008`)

`OD` 载入，停留在这里~~

```
00404794 >/$  55             push    ebp  
00404795  |.  8BEC            mov     ebp, esp  
00404797  |.  6A FF           push    -1  
00404799  |.  68 E8504000     push    004050E8  
0040479E  |.  68 00474000     push    <jmp.&msvcrt._except_handler3>   ;  SE 处理程序安装  
004047A3  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]  
004047A9  |.  50             push    eax  
004047AA  |.  64:8925 00000>mov     dword ptr fs:[0], esp  
```

不用解释什么了吧，很明显 `VC6.0` 的程序~~~

这里不用单步跟踪，可以直接将鼠标向下翻，来到如下地址处：

```
004048BB  |.  53             push    ebx                     ; /pModule  
004048BC  |.  FF15 04504000  call    dword ptr [<&kernel32.GetModuleH>; \GetModuleHandleA  
004048C2  |.  50             push    eax  
004048C3  |.  E8 3BEEFFFF    call    00403703      -------->关键 CALL  
004048C8  |.  8945 98        mov     dword ptr [ebp-68], eax  
004048CB  |.  50             push    eax                     ; /status  
004048CC  |.  FF15 B8504000  call    dword ptr [<&msvcrt.exit>]      ; \exit  
```

前面是一个 `GetModuleHandleA`，后面是一个 `exit`，中间只有一个 `CALL`，那很明显这个 `CALL` 就是程序的主功能了吧，所以的操作都应该在这个程序中完成!

在 `004048C3` 处，先按 `F2` 下断，然后等程序运行到这里，按 `F7` 进入，来到如下地址处：

```
00403703  /$  55             push    ebp  
00403704  |.  8BEC            mov     ebp, esp  
00403706  |.  56             push    esi  
00403707  |.  57             push    edi  
00403708  |.  55             push    ebp  
00403709  |.  5D             pop     ebp  
0040370A  |.  E8 57F3FFFF    call    00402A66  ---------->F7 进入  
0040370F  |.  BE 04010000    mov     esi, 104  
\-----------------------------------------------------------------  
call 00402A66  
```

```
00402A66  /$  55             push    ebp  
00402A67  |.  8BEC            mov     ebp, esp  
00402A69  |.  83EC 1C         sub     esp, 1C  
00402A6C  |.  8065 FE 00      and     byte ptr [ebp-2], 0  
00402A70  |.  8065 F4 00      and     byte ptr [ebp-C], 0  
00402A74  |.  8D45 E4         lea     eax, dword ptr [ebp-1C]  
00402A77  |.  6A 01           push    1  
00402A79  |.  50             push    eax  
00402A7A  |.  C645 F8 6D      mov     byte ptr [ebp-8], 6D  
00402A7E  |.  C645 F9 79      mov     byte ptr [ebp-7], 79  
00402A82  |.  C645 FA 2E      mov     byte ptr [ebp-6], 2E  
00402A86  |.  C645 FB 65      mov     byte ptr [ebp-5], 65  
00402A8A  |.  C645 FC 78      mov     byte ptr [ebp-4], 78  
00402A8E  |.  C645 FD 65      mov     byte ptr [ebp-3], 65          ;  my.exe  
00402A92  |.  C645 E4 53      mov     byte ptr [ebp-1C], 53  
00402A96  |.  C645 E5 65      mov     byte ptr [ebp-1B], 65  
00402A9A  |.  C645 E6 44      mov     byte ptr [ebp-1A], 44  
00402A9E  |.  C645 E7 65      mov     byte ptr [ebp-19], 65  
00402AA2  |.  C645 E8 62      mov     byte ptr [ebp-18], 62  
00402AA6  |.  C645 E9 75      mov     byte ptr [ebp-17], 75  
00402AAA  |.  C645 EA 67      mov     byte ptr [ebp-16], 67  
00402AAE  |.  C645 EB 50      mov     byte ptr [ebp-15], 50  
00402AB2  |.  C645 EC 72      mov     byte ptr [ebp-14], 72  
00402AB6  |.  C645 ED 69      mov     byte ptr [ebp-13], 69  
00402ABA  |.  C645 EE 76      mov     byte ptr [ebp-12], 76  
00402ABE  |.  C645 EF 69      mov     byte ptr [ebp-11], 69  
00402AC2  |.  C645 F0 6C      mov     byte ptr [ebp-10], 6C  
00402AC6  |.  C645 F1 65      mov     byte ptr [ebp-F], 65  
00402ACA  |.  C645 F2 67      mov     byte ptr [ebp-E], 67  
00402ACE  |.  C645 F3 65      mov     byte ptr [ebp-D], 65          ;  SeDebugPrivilege  
00402AD2  |.  E8 58E9FFFF     call    0040142F           ------------>CALL 1  
00402AD7  |.  8D45 F8         lea     eax, dword ptr [ebp-8]  
00402ADA  |.  50             push    eax  
00402ADB  |.  E8 A5E7FFFF     call    00401285           ------------>CALL 2  
00402AE0  |.  83C4 0C         add     esp, 0C  
00402AE3  |.  68 F4010000     push    1F4                   ; /Timeout = 500. ms  
00402AE8  |.  FF15 0C504000   call    dword ptr [<&kernel32.Sleep>]    ; \Sleep------->调用上面这两个 CALL 之后，睡眠 0.5S  
00402AEE  |.  C9             leave  
00402AEF  \.  C3             retn  
\~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
```

由于代码较长，我就不在这里详解了，大家在下面可以跟踪进去看看，很简单~~我在说下我跟踪之后的详细结果吧!

`call 0040142F` 这里的 `CALL` 主要功能如下：

用 `LoadLibraryA` 加载 `Advapi32.dll`,然后在 `Advapi32.dll` 中，查找 `OpenProcessToken`,`LookupPrivilegeValueA`,`AdjustTokenPrivileges` 这三个函数的函数地址

得到这三个函数地址之后

再用 `LoadLibraryA` 加载 `kernel32.dll`，然后在 `kernel32.dll` 中，查找 `CloseHandle`,`GetCurrentProcess` 的函数地址，并调用 `GetCurrentProcess` 函数，得到当前程序的进程~~

再得到当前进程之后，用前面得到的三个函数地址，调用上面三个函数 `OpenProcessToken`,`LookupPrivilegeValueA`,`AdjustTokenPrivileges` 来进行权限提升，最后调用 `CloseHandle` 关闭句柄进程句柄

`call 00401285` 这里的 `CALL` 主要功能如下：
