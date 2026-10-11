
## 运行分析与报错处理

执行 `00681B5E    8B0B             mov ecx,dword ptr ds:[ebx]` 时，程序报错：

> "Access violation at address `1000100A`. Read of address `1000100A`."

使用 `LordPE` 检查内存空间，发现 `ListView` 的 `Item` 总数为 `26`。

### 内存映射信息

```
Path                                                                                                                        
-------------------------------------------------------------------------------
d:\bmjapanese\bmjapanese.exe                                                           00400000       00001000        
c:\winnt\system32\ntdll.dll                                                            77F80000       0007B000        
c:\winnt\system32\kernel32.dll                                                         7C4E0000       000B9000        
c:\winnt\system32\user32.dll                                                            77E10000       00065000        
c:\winnt\system32\gdi32.dll                                                             77F40000       0003C000        
c:\winnt\system32\imm32.dll                                                             75E60000       0001A000        
c:\winnt\system32\advapi32.dll                                                         7C2D0000       00062000        
c:\winnt\system32\rpcrt4.dll                                                            77D30000       00071000        
d:\bmjapanese\data\data.dat                                                              10000000       00037000        
c:\winnt\system32\comctl32.dll                                                         71710000       00084000        
c:\winnt\system32\comdlg32.dll                                                         76B30000       0003E000        
c:\winnt\system32\shlwapi.dll                                                           70BD0000       00065000        
c:\winnt\system32\msvcrt.dll                                                            78000000       00045000        
c:\winnt\system32\shell32.dll                                                            782F0000       00248000        
c:\winnt\system32\hhctrl.ocx                                                             5D300000       00080000        
c:\winnt\system32\ole32.dll                                                             77A50000       000F7000        
c:\winnt\system32\oleaut32.dll                                                          779B0000       0009B000        
c:\winnt\system32\msacm32.dll                                                           77410000       00013000        
c:\winnt\system32\winmm.dll                                                             77570000       00030000        
c:\winnt\system32\version.dll                                                           77820000       00007000        
c:\winnt\system32\lz32.dll                                                              759B0000       00006000        
c:\winnt\system32\winspool.drv                                                         77800000       0001E000        
c:\winnt\system32\mpr.dll                                                               76620000       00011000        
c:\winnt\system32\wsock32.dll                                                           75050000       00008000        
c:\winnt\system32\ws2_32.dll                                                            75030000       00014000        
c:\winnt\system32\ws2help.dll                                                           75020000       00008000        
```

错误地址位于 `data.dat` 空间。使用 `PEID` 检查发现这是一个 `ASPack` 壳的 `DLL` 文件，其输出表仅包含两个函数：`HDSerialNumRead` 和 `WEP`。

由于还有一个 `DLL` 未被加载（`LOAD`），因此出现错误。

### 调试过程

重新载入原文件，在 `LoadLibraryA` 处设置断点。经过几次后，断点停留在：

```
01240873    53               push ebx
01240874    FF95 94E24100    call dword ptr ss:[ebp+41E294]          ; LoadLibraryA .dat\data\data.dat
0124087A    0BC0             or eax,eax
0124087C    75 34            jnz short BmJapane.012408B2
```

随后执行到：

```
01240941    53               push ebx
01240942    FFB5 17FC4000    push dword ptr ss:[ebp+40FC17]
01240948    FF95 8CE24100    call dword ptr ss:[ebp+41E28C]          ; kernel32.GetProcAddress
0124094E    3B9D 1FFC4000    cmp ebx,dword ptr ss:[ebp+40FC1F]
```

此处获取了 `HDSerialNumRead` 的地址。

使用 `Alt+M` 打开内存镜像。在地址 `10001000` 处按下 `F2` 设置断点，取消之前的 `LoadLibraryA` 断点，然后按 `F9` 继续运行，程序停留在此处：

```
10001003    CC               int3
10001004    CC               int3
10001005    E9 66010000      jmp data.10001170
1000100A >  E9 310B0000      jmp data.10001B40       -|jmp
1000100F >  E9 3C0C0000      jmp data.10001C50
10001014    E9 C7030000      jmp data.100013E0
10001019    E9 52000000      jmp data.10001070
1000101E    E9 0D020000      jmp data.10001230
10001023    E9 78080000      jmp data.100018A0
10001028    E9 93020000      jmp data.100012C0
1000102D    E9 BE040000      jmp data.100014F0
10001032    E9 89070000      jmp data.100017C0
10001037    CC               int3
10001038    CC               int3
----------------------------------------------------|
10001B40    55               push ebp
10001B41    8BEC             mov ebp,esp
10001B43    81EC D4070000    sub esp,7D4
10001B49    53               push ebx
10001B4A    56               push esi
```

按下 `CTRL+F9` 返回主程序：

```
0059C567    8D45 C4          lea eax,dword ptr ss:[ebp-3C]
0059C56A    E8 518AE6FF      call BmJapane.00404FC0
0059C56F    E8 D0F1FFFF      call BmJapane.0059B744                   ; jmp to data.HDSerialNumRead
0059C574    8BD0             mov edx,eax
0059C576    8D45 C4          lea eax,dword ptr ss:[ebp-3C]
```

原本的 `CALL 59B744` 实际上是调用 `data.HDSerialNumRead` 函数的地址，直接跳转查看：

```
0059B744  - FF25 2CCC6800    jmp dword ptr ds:[68CC2C]               ; data.HDSerialNumRead
```

只需将 `68CC20` 填充为正确的地址即可。

使用 `OD` 载入 dump 出的程序，在 `LoadLibraryA` 处设置断点。程序中断后按 `Alt+F9` 返回用户代码：

```
0047E97B   /0F85 EE000000    jnz oll.0047EA6F
0047E981   |68 ACEA4700      push oll.0047EAAC                       ; ASCII "imm32.dll"
0047E986   |E8 D594F8FF      call <jmp.&kernel32.LoadLibraryA>
0047E98B   |A3 88326800      mov dword ptr ds:[683288],eax           ; IMM32.75E60000
0047E990   |833D 88326800 0>cmp dword ptr ds:[683288],0
0047E997   |0F84 D2000000    je oll.0047EA6F
```

向下查找代码，找到出口处：

```
0047EA8D    5B               pop ebx                                  ; USER32.77E10000
0047EA8E    59               pop ecx
0047EA8F    5D               pop ebp
0047EA90    C3               retn
0047EA91    0000             add byte ptr ds:[eax],al
```

### 注入自定义代码

寻找一个合适的位置插入自己的代码。区块 `ExeS` 已不再使用，正好用来编写代码。

*   注意：`LoadLibraryA` 的 `CALL` 是 `"call 00407E60"`；`GetProcAddress` 的 `CALL` 是 `"call 00407D50"`。情况不同者需相应修改。

```
0047EA8D  - E9 6EB5DC00      jmp oll.0124A000
0047EA92    C3               retn

0124A000    68 30A02401      push oll.0124A030                       ; ASCII ".\data\Data.dat"
0124A005    E8 56DE1BFF      call <jmp.&kernel32.LoadLibraryA>
0124A00A    68 40A02401      push oll.0124A040                       ; ASCII "HDSerialNumRead"
0124A00F    50               push eax                                 ; EAX 是 LoadLibraryA 后的 DLL 句柄
0124A010    E8 3BDD1BFF      call <jmp.&kernel32.GetProcAddress>
0124A015    A3 2CCC6800      mov dword ptr ds:[68CC2C],eax
0124A01A    5B               pop ebx
0124A01B    59               pop ecx
0124A01C    5D               pop ebp
0124A01D    90               nop
0124A01E  - E9 6F4A23FF      jmp oll.0047EA92
0124A023    90               nop
```

### 处理 ANTI 代码

移除程序的 ANTI 保护代码：

```
0067D1E9  ^\EB CE           jmp short dumped.0067D1B9
0067D1EB    E8 80C4E4FF      call dumped.004C9670
0067D1F0    68 FFFF0000      push 0FFFF
0067D1F5    6A 0D           push 0D
0067D1F7    E8 7CB2D8FF      call <jmp.&user32.ExitWindowsEx>        ; 关机了~
0067D1FC    E9 CD060000      jmp dumped.0067D8CE
```

为了防止程序误用危险的 API，需将其指向自定义代码，直接在 `IAT` 中修改。

原代码：`0067D1F7    E8 7CB2D8FF      call <jmp.&user32.ExitWindowsEx>` 本来是 `call 00408478`

去 `408478` 查看：

```
00408478  - FF25 50C96800    jmp dword ptr ds:[<&user32.ExitWindowsEx>; user32.ExitWindowsEx
```

自定义代码：

```
0124A000    83C4 0C          add esp,0C
0124A003    FF7424 F4        push dword ptr ss:[esp-C]              ; 还原调用 CALL 以后 RETN 的地址
0124A007    C3               retn
```

修改 `IAT` 表：

```
00408478  - E9 831BE400      jmp exit_fix.0124A000
0040847D    90               nop
```

运行程序，一切正常。

## 脱壳完成

**后记**：`EXEStealth` 的 `IAT` 还原部分比较有趣：

```
00FDE535    8B07             mov eax,dword ptr ds:[edi]              ; EDI 是加密函数名结构(可以这样说)的基地址
00FDE537    09C0             or eax,eax
00FDE539    74 3C            je short BmJapane.00FDE577              ; IAT 表是否填充完成,完成的话跳
```
