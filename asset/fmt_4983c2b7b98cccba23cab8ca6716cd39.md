
```
……
00427711   push    96000                                                 ;  映像大小＝96000
00427716   mov     ecx, eax
00427718   push    dword ptr [ebp+8]                                     ;  映像名＝"PCHMPROM"
0042771B   call    00469E4C                                              ;  读取映像数据

……
0046A00B   push    -1
0046A00D   mov     ecx, eax
0046A00F   call    <jmp.&MFC42.#4167_CSINGLELOCK::LOCK>                ;  锁定数据
0046A014   push    dword ptr [ebp+8]                                     ;  /MapName
0046A017   mov     eax, dword ptr [esi+28]                               ;  |
0046A01A   neg     eax                                                   ;  |
0046A01C   push    dword ptr [ebp+C]                                     ;  |MaximumSizeLow
0046A01F   lea     ecx, dword ptr [esi+40]                               ;  |
0046A022   sbb     eax, eax                                              ;  |
0046A024   push    edi                                                   ;  |MaximumSizeHigh
0046A025   and     eax, ecx                                              ;  |
0046A027   push    ebx                                                   ;  |Protection
0046A028   push    eax                                                   ;  |pSecurity
0046A029   push    -1                                                    ;  |hFile = FFFFFFFF
0046A02B   call    dword ptr [<&KERNEL32.CreateFileMappingA>]           ;  \\建立映像
0046A031   mov     edi, dword ptr [<&KERNEL32.GetLastError>]            ;  ntdll.RtlGetLastWin32Error
0046A037   mov     dword ptr [esi+24], eax
0046A03A   call    edi                                                   ;  [GetLastError
0046A03C   cmp     eax, 0B7                                              ;  映像已经存在？
0046A041   jnz     short 0046A047
0046A043   and     dword ptr [esi+14], 0                                 ;  映像已经存在时置标志位
0046A047   mov     eax, dword ptr [esi+24]
0046A04A   xor     ecx, ecx
0046A04C   cmp     eax, ecx
0046A04E   je      0046A0F9
0046A054   push    ecx                                                   ;  /MapSize => 0
0046A055   push    ecx                                                   ;  |OffsetLow => 0
0046A056   push    ecx                                                   ;  |OffsetHigh => 0
0046A057   push    6                                                     ;  |AccessMode = 6
0046A059   push    eax                                                   ;  |hMapObject
0046A05A   call    dword ptr [<&KERNEL32.MapViewOfFile>]                ;  \\读取映像
0046A060   mov     edx, eax
0046A062   test    edx, edx
0046A064   mov     dword ptr [esi+20], edx
0046A067   je      0046A0F9
0046A06D   cmp     dword ptr [esi+14], 0                                 ;  映像是否存在？
0046A071   je      short 0046A098
0046A073   mov     ecx, dword ptr [ebp+C]                                 ;  映像不存在时：
0046A076   mov     edi, edx
0046A078   mov     edx, ecx
0046A07A   xor     eax, eax
0046A07C   shr     ecx, 2
0046A07F   rep     stos dword ptr es:[edi]                                ;  把映像数据全部置为 0
0046A081   mov     ecx, edx
0046A083   push    ebx
0046A084   and     ecx, 3
0046A087   rep     stos byte ptr es:[edi]
0046A089   lea     eax, dword ptr [esi+1C]
0046A08C   mov     ecx, edx
0046A08E   push    eax
0046A08F   mov     dword ptr [eax], ecx
0046A091   mov     eax, dword ptr [esi+20]
0046A094   add     eax, ebx
0046A096   jmp     short 0046A0A0
0046A098   add     edx, 4
0046A09B   push    ebx
0046A09C   push    edx
0046A09D   lea     eax, dword ptr [esi+1C]
0046A0A0   push    eax                                                   ;  |dest
0046A0A1   call    <jmp.&MSVCRT.memcpy>                                  ;  \\读映像的 4－7 字节
```

## 六、转单进程：胜利在望

分析完主程序可以发现，程序从引导程序创建的名为 `PCHMPROM` 的内存映像中读取数据，如果映像不存在程序就拒绝执行。那么转单进程的方法就是把引导程序创建的内存映像作为一个区段添加到主程序中，在主程序读取映像时直接指向这个区段，即可完全转单进程。

具体方法为：

1. 在 `OD` 创建好内存映像后，把内存映像另存为 `mem.bin` 文件，用 `PE` 工具打开主程序 `PocketCHM_Pro.exe`，我这里用的工具是 `Stud_PE` 国庆版，选择“新建区段”，区段名为 `.mem`，原始大小和虚拟大小都设为 `96000`，选择“区段来自文件”并指定刚才另存出来的内存映像文件 `mem.bin`，保存退出。
2. 用 `OD` 打开已添加区段的主程序，按 `CTRL+G`，填入 `46A014` 来到建立内存映像的位置，修改以下代码：

```
0046A014      B8 00505800   mov     eax, 00585000                      ;  .mem 区段的首地址
0046A019      8366 14 00    and     dword ptr [esi+14], 0             ;  置标志位
0046A01D      EB 41         jmp     short 0046A060                    ;  跳往建立完映像后的位置继续执行
```

3. 直接 `F9` 测试一下，程序果然跑起来了！用 `16` 进制工具把修改的数据写到程序中。

## 七、尾声：暴破

程序的注册算法有点复杂，但是做完注册检测后只写了一个注册标志位，暴破起来就相当简单了：

找到注册算法过程中的这个位置并修改：

```
00429274  |.  8378 F8 0C      cmp     dword ptr [eax-8], 0C           ;  判断用户名注册码的长度
00429278      33FF            xor     edi, edi                        ;  //modify
0042927A      47              inc     edi                             ;  //modify
0042927B      EB 06           jmp     short 00429283                  ;  //modify
0042927D  |.  8378 F8 18      cmp     dword ptr [eax-8], 18
00429281  |.  74 0B           je      short 0042928E
00429283  |>  89BB 280D0000   mov     dword ptr [ebx+D28], edi      ;  置标志位，为 1 时已注册
00429289  |.  E9 ED040000     jmp     0042977B
```

用 `16` 进制工具把标为 `modify` 的地方改好后试运行一下，主界面上的 `Unregistered copy. Available trial days:30` 已经没有了，菜单中的注册项也没有了，再测试一下编译 `CHM` 文件和 `Read CHM on Your PDA` 这些有限制的功能，没发现暗桩。至此已大功告成！

## 后记

什么时候再静下心来分析一下算法吧。。。

用资源修改工具打开可以发现这个程序很多地方都有中文，是汉化？还是国产软件？迷惑 ing...

`by lelfei on 2008.4.29 14:30`
