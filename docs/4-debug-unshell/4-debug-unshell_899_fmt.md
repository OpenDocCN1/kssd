
# 手脱 PESHiELD 0.25 并完美优化

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-550.htm`](https://www.pediy.com/kssd/pediy09/pediy09-550.htm)
>
> 一个很老的保护壳，对比看看才几年时间壳的技术已经日新月异了。。。
>
> 本文适合脱壳爱好者初级向中级进军时参考一下，见笑了～
>
> 这里的测试样本是经典的 Win98 记事本，用 `PEiD` 看下：`PESHiELD 0.25 -> ANAKiN`
>
> 由于一般的加密壳都会用系统的异常处理更改程序的运行流程，我们取消 `OD` 的忽略异常选项，可以很快到达程序的关键点。当然，也可以单步跟踪，遇到回跳就 `F4` 带过，很快就能到达异常中断。
>
> 先 `F9` 直接运行，遇到异常中断时用 `SHIFT+F9` 步过，发现经过 2 次“非法使用寄存器”异常后，`OD` 会崩溃，看来是反调试检测到 `OD` 了。
>
> 重新载入程序，在第二次异常中断时停下来，观察堆栈：
>
> ```
> 0013FF9C   0013FFE0   指向下一个 SEH 记录的指针
> 0013FFA0   0040D4AC   SE 处理程序
> ```
>
> 程序中断后会转到 `0040D4AC` 处继续执行，我们在这一句下断，用 `SHIFT+F9` 运行后会中断在这一句：
>
> ```
> 0040D4AC  mov     eax, dword ptr [esp+C]
> 0040D4B0  add     dword ptr [eax+B8], 4            ; 注意这一句，该处保存程序继续执行的入口点
> 0040D4B7  push    ebx
> 0040D4B8  xor     ebx, ebx
> 0040D4BA  mov     dword ptr [eax+4], ebx
> 0040D4BD  mov     dword ptr [eax+8], ebx
> 0040D4C0  mov     dword ptr [eax+18], 155
> 0040D4C7  mov     dword ptr [eax+C], ebx
> 0040D4CA  mov     dword ptr [eax+10], ebx
> 0040D4CD  pop     ebx
> 0040D4CE  xor     eax, eax
> 0040D4D0  retn                               ; 这里程序会返回到系统领空
> ```
>
> 这一段代码也是更改程序流程的一种方式，程序执行到 `0040D4D0` 处的 `retn` 后会迷失在系统领空，而在前面 `0040D4B0` 处会把程序流程的入口保存到 `[eax+B8]` 中。我们直接在 `[eax+B8] = 0040D4DB` 处下断，`F9` 后中断下来。去掉花指令后就剩下下面的了：
>
> ```
> 0040D4DB  pop     dword ptr fs:[ebx]
> 0040D4DE  mov     esp, eax
> 0040D4E0  mov     eax, dword ptr [ebp+12A7]        ; EAX＝00400000,Base Image
> 0040D4EB  add     eax, dword ptr [eax+3C]          ; [eax+3C]=PE Header
> 0040D4F1  mov     ecx, eax                         ; 保存 PE Header
> 0040D4F7  mov     esi, dword ptr [eax+8]           ; TimeDateStamp
> 0040D4FD  xor     edi, edi
> 0040D502  dec     edi
> 0040D506  mov     edx, dword ptr [eax+28]           ; OEP
> 0040D50C  xor     ebx, ebx
> 0040D512  mov     eax, dword ptr fs:[30]
> 0040D51B  test    eax, eax
> 0040D520  js      short 0040D53A
> 0040D525  mov     eax, dword ptr [eax+C]
> 0040D52B  mov     eax, dword ptr [eax+C]
> 0040D531  mov     dword ptr [eax+20], ebx
> 0040D561  mov     ebx, dword ptr [ebp+124F]
> 0040D56A  add     ebx, ecx                       ; NOTEPAD.00400080
> 0040D56F  movzx   eax, byte ptr [ebx]
> 0040D575  mov     ah, al
> 0040D57A  add     dword ptr [ebp+1243], eax
> 0040D580  lea     eax, dword ptr [ebp+10F3]        ; EAX="VirtualAlloc"
> 0040D589  lea     ebx, dword ptr [ebp+127B]
> 0040D594  call    0040E1B6                       ; GetProcAddress VirtualAlloc
> 0040D599  mov     eax, dword ptr [ebp+12A7]
> 0040D5A3  add     eax, dword ptr [ebp+1273]        ; eax=00406000
> 0040D5AD  mov     dword ptr [ebp+1227], eax
> 0040D5B3  mov     ebx, dword ptr [ebp+12AB]
> 0040D5BC  lea     esi, dword ptr [ebp+1100]        ; esi="NuxHmzvcgd\ywkauz",加密字符
> 0040D5C2  call    0040DF44
> 0040D5CA  call    0040DF44
> 0040D5D2  call    0040DF44                       ; 字符串解密
> 0040D5DC  lea     eax, dword ptr [ebp+1100]        ; eax="GetCurrentProcess"
> 0040D5E5  lea     ebx, dword ptr [ebp+125B]
> 0040D5EB  call    0040E1B6                       ; GetProcAddress GetCurrentProcess
> 0040D5F3  lea     eax, dword ptr [ebp+1112]        ; eax="WriteProcessMemory"
> 0040D5FE  lea     ebx, dword ptr [ebp+125F]
> 0040D604  call    0040E1B6                       ; GetProcAddress WriteProcessMemory
> 0040D60D  lea     eax, dword ptr [ebp+1125]        ; eax="CreateFileA"
> 0040D617  lea     ebx, dword ptr [ebp+126B]
> 0040D61D  call    0040E1B6                       ; GetProcAddress CreateFileA
> 0040D622  push    0
> 0040D627  push    80
> 0040D62F  push    3
> 0040D634  push    0
> 0040D639  push    3
> 0040D63E  push    800000
> 0040D646  lea     eax, dword ptr [ebp+113A]        ; eax="\\.\TRW.VXD"
> 0040D650  push    eax                       ; NOTEPAD.0040E13A
> 0040D654  mov     eax, dword ptr [ebp+126B]        ; kernel32.CreateFileA
> 0040D65D  call    0040DDBB                       ; call CreateFileA
> 0040D665  cmp     eax, -1                       ; 检测 TRW
> 0040D66B  jnz     0040D75B
> 0040D671  lea     eax, dword ptr [ebp+115F]        ; eax="CreateThread"
> 0040D67A  lea     ebx, dword ptr [ebp+116C]
> 0040D684  call    0040E1B6                       ; GetProcAddress CreateThread
> 0040D68C  lea     eax, dword ptr [ebp+1174]        ; eax="ExitThread"
> 0040D695  lea     ebx, dword ptr [ebp+117F]
> 0040D6A0  call    0040E1B6                       ; GetProcAddress ExitThread
> 0040D6A8  mov     dword ptr [ebp+81F], ebp      ; NOTEPAD.<模块入口点>
> 0040D6B1  lea     edx, dword ptr [ebp+1187]
> 0040D6BA  lea     ecx, dword ptr [ebp+81E]
> 0040D6C3  push    edx                       ; NOTEPAD.0040E187
> 0040D6C7  push    0
> 0040D6CC  add     edx, 4
> 0040D6D2  push    edx                       ; NOTEPAD.0040E18B
> 0040D6D6  push    ecx                       ; NOTEPAD.0040D81E
> 0040D6DA  push    0
> 0040D6DF  push    0
> 0040D6E4  mov     eax, dword ptr [ebp+116C]        ; kernel32.CreateThread
> 0040D6ED  call    0040DDBB                       ; call CreateThread
> 0040D6F5  push    dword ptr [ebp+117F]             ; kernel32.ExitThread
> 0040D6FB  inc     dword ptr [ebp+1183]
> 0040D701  retn                               ; 返回系统领空，不知道怎么返回
> ```
>
> 执行到这里就需要注意下了，程序用 `CreateThread` 建立了一个新线程并跳到新线程执行，而当前进程就迷失在系统领空了。而创建的新线程的入口点即为 `0040D6D6` 这一句保存的 `0040D81E`。直接在 `0040D81E` 处下断，`F9` 中断下来：
>
> ```
> 0040D81E  mov     ebp, offset <模块入口点>
> 0040D823  lea     esi, dword ptr [ebp+10C5]
> 0040D829  sub     esp, 20
> 0040D82C  mov     edi, esp
> 0040D82E  mov     ecx, 8
> 0040D833  rep     movs dword ptr es:[edi], dword ptr [esi]
> 0040D835  mov     eax, dword ptr [ebp+1183]
> 0040D83B  or      eax, eax
> 0040D83D  je      short 0040D835
> 0040D83F  push    0
> 0040D844  push    80
> 0040D84C  push    3
> 0040D851  push    0
> 0040D856  push    3
> 0040D85B  push    800000
> 0040D863  lea     eax, dword ptr [ebp+1131]        ; 地址=0040E131, (ASCII "\\.\SICE")
> 0040D86D  push    eax                       ; NOTEPAD.0040E131
> 0040D871  mov     eax, dword ptr [ebp+126B]        ; kernel32.CreateFileA
> 0040D87A  call    0040DDBB                       ; call CreateFileA
> 0040D882  cmp     eax, -1                       ; 检测 SICE
> 0040D888  jnz     0040D772
> 0040D891  cmp     byte ptr [ebp+121B], 1
> 0040D89B  je      short 0040D907
> 0040D8A2  mov     ecx, 0C8
> 0040D8AA  push    ecx
> 0040D8B0  loopd   short 0040D8A7
> 0040D8B6  mov     eax, 64
> 0040D8BB  call    0040DC88
> 0040D8C0  add     eax, 2710
> 0040D8C5  neg     eax
> 0040D8C7  push    eax
> 0040D8C8  mov     ecx, 0E
> 0040D8CD  push    ecx
> 0040D8CE  loopd   short 0040D8CD
> 0040D8D0  push    5A4D
> 0040D8D5  mov     ebx, esp
> 0040D8D7  lea     eax, dword ptr [ebp+12AB]
> 0040D8DD  push    eax
> 0040D8DE  push    360
> 0040D8E3  push    ebx
> 0040D8E4  push    dword ptr [ebp+12A7]
> 0040D8EA  mov     eax, dword ptr [ebp+125B]        ; kernel32.GetCurrentProcess
> 0040D8F0  call    0040DDBB
> 0040D8F5  push    eax
> 0040D8F6  mov     eax, dword ptr [ebp+125F]        ; kernel32.WriteProcessMemory
> 0040D8FC  call    0040DDBB
> 0040D901  add     esp, 360
> 0040D907  lea     esi, dword ptr [ebp+15C8]        ; "NON-COMMERCIAL!!"
> 0040D90D  mov     edx, 14356241
> 0040D912  lods    dword ptr [esi]
> 0040D913  xor     edx, eax
> 0040D915  rol     edx, 1
> 0040D917  lods    dword ptr [esi]
> 0040D918  add     edx, eax
> ```



```
0040D91A  ror     edx, 1
0040D91C  lods    dword ptr [esi]
0040D91D  add     edx, eax
0040D91F  lods    dword ptr [esi]
0040D920  xor     edx, eax
0040D922  cmp     edx, 8446AB4
0040D928  jnz     <模块入口点>
0040D92E  cmp     byte ptr [ebp+1219], 1
0040D935  jnz     short 0040D953
0040D953  mov     esi, dword ptr [ebp+1253]
0040D959  add     esi, ebp
0040D95B  mov     ebx, dword ptr [ebp+1243]
0040D961  lods    dword ptr [esi]                 ; －－开始还原 401000 处的代码－－
0040D965  or      eax, eax
0040D96A  je      short 0040D9BE                 ; 是否继续循环？
0040D96F  mov     edi, eax
0040D974  xor     edi, ebx
0040D979  rol     ebx, 1
0040D97B  add     ebx, edi
0040D980  add     edi, dword ptr [ebp+12A7]       ; NOTEPAD.00400000
0040D986  lods    dword ptr [esi]
0040D98A  mov     ecx, eax
0040D98C  xor     ecx, ebx
0040D991  lods    dword ptr [esi]
0040D992  add     eax, ebx
0040D998  rol     ebx, 1
0040D99A  xor     dword ptr [edi], eax
0040D9A0  xor     dword ptr [edi], ecx           ; 还原代码
0040D9A2  rol     eax, 3
0040D9AA  add     eax, ecx
0040D9AC  add     edi, 4
0040D9B2  dec     ecx
0040D9B7  jnz     short 0040D99A                 ; 循环长度为 0FF8
0040D9BC  jmp     short 0040D961                 ; 回跳，继续循环
0040D9BE  cmp     byte ptr [ebp+1219], 1         ; －－代码还原完毕－－
0040D9C5  jnz     short 0040D9D5
0040D9C7  mov     ebx, dword ptr [ebp+1263]
0040D9CD  mov     eax, dword ptr [ebp+1277]
0040D9D3  mov     dword ptr [ebx], eax
0040D9D5  mov     eax, dword ptr [ebp+1227]       ; ss:[0040E227]=00406000 (NOTEPAD.00406000)
0040D9DB  mov     dword ptr [ebp+121F], eax
0040D9E1  cmp     byte ptr [ebp+121E], 1
0040D9E8  jnz     short 0040DA54
0040DA54  mov     edx, dword ptr [ebp+1227]       ; NOTEPAD.00406000
0040DA5A  sub     edx, dword ptr [ebp+121F]
0040DA60  add     edx, dword ptr [ebp+12A7]
0040DA66  mov     dword ptr [ebp+122B], edx
0040DA6C  mov     esi, dword ptr [ebp+1267]
0040DA72  or      esi, esi
0040DA74  je      short 0040DAA9
0040DA76  add     esi, dword ptr [ebp+12A7]       ; NOTEPAD.00400000
0040DA7C  add     esi, 10
0040DA7F  mov     edx, dword ptr [ebp+1227]
0040DA85  sub     edx, dword ptr [ebp+12A3]
0040DA8B  or      edx, edx
0040DA8D  je      short 0040DA94
0040DA8F  call    0040DEE8
0040DA94  mov     edx, dword ptr [ebp+12A7]
0040DA9A  sub     edx, dword ptr [ebp+12AF]
0040DAA0  or      edx, edx
0040DAA2  je      short 0040DAA9
0040DAA4  call    0040DEE8
0040DAA9  cmp     byte ptr [ebp+121A], 0
0040DAB0  je      short 0040DAD6
0040DAB2  push    4
0040DAB4  push    1000
0040DAB9  push    dword ptr [ebp+128B]
0040DABF  push    0
0040DAC1  call    0040D702
0040DAC6  or      eax, eax
0040DAC8  je      0040D806
0040DACE  mov     esi, eax
0040DAD0  mov     dword ptr [ebp+127F], eax
0040DAD6  mov     ecx, dword ptr [ebp+12A7]
0040DADC  mov     edx, dword ptr [ebp+1227]
0040DAE2  mov     eax, dword ptr [edx+C]         ; －－开始修复 IAT－－
0040DAE5  or      eax, eax
0040DAE7  je      0040DBE0                       ; 是否还有 DLL？
0040DAED  mov     dword ptr [edx+C], ecx
0040DAF0  add     eax, dword ptr [ebp+122B]
0040DAF6  push    edx
0040DAF7  push    ecx
0040DAF8  push    eax
0040DAF9  push    eax
0040DAFA  mov     byte ptr [ebp+121D], 0
0040DB01  mov     ebx, dword ptr [eax]
0040DB03  and     ebx, 0DFDFDF
0040DB09  cmp     ebx, 43464D                   ; "MFC"
0040DB0F  jnz     short 0040DB29
0040DB11  mov     ebx, dword ptr [eax+5]
0040DB14  and     ebx, DFDFDFFF
0040DB1A  cmp     ebx, 4C4C442E
0040DB20  jnz     short 0040DB29
0040DB22  mov     byte ptr [ebp+121D], 1
0040DB29  mov     ebx, eax                       ; "SHELL32.dll"
0040DB2B  call    0040D70D                       ; GetModuleHandleA
0040DB30  pop     ebx
0040DB31  pop     ecx
0040DB32  pop     edx
0040DB33  or      eax, eax
0040DB35  jnz     short 0040DB49
0040DB37  push    edx
0040DB38  push    ecx
0040DB39  push    ebx
0040DB3A  call    0040D718                       ; LoadLibraryA
0040DB3F  or      eax, eax
0040DB41  je      0040D789                       ; Load 失败？
0040DB47  pop     ecx
0040DB48  pop     edx
0040DB49  call    0040DC3D
0040DB4E  mov     dword ptr [ebp+BAE], eax
0040DB54  mov     esi, dword ptr [edx]
0040DB56  mov     dword ptr [edx], ecx
0040DB58  mov     edi, dword ptr [edx+10]
0040DB5B  mov     dword ptr [edx+10], ecx
0040DB5E  or      esi, esi
0040DB60  jnz     short 0040DB64
0040DB62  mov     esi, edi
0040DB64  add     esi, dword ptr [ebp+122B]
0040DB6A  add     edi, dword ptr [ebp+122B]
0040DB70  mov     eax, dword ptr [esi]
0040DB72  or      eax, eax
0040DB74  je      short 0040DBD8                 ; 当前 DLL 是否结束？
0040DB7A  movzx   eax, ax
0040DB7D  jmp     short 0040DBA5
0040DB7F  add     eax, dword ptr [ebp+122B]
0040DB85  mov     word ptr [eax], 0
0040DB8A  inc     eax
0040DB8B  inc     eax
0040DB8C  push    ebx
0040DB8D  push    esi
0040DB8E  push    eax
0040DB8F  mov     ebx, dword ptr [ebp+1287]
0040DB95  mov     esi, eax
0040DB97  call    0040DF44                       ; 字符串解密
0040DB9C  mov     dword ptr [ebp+1287], ebx
0040DBA2  pop     eax
0040DBA3  pop     esi
0040DBA4  pop     ebx
0040DBA5  push    eax
0040DBA6  push    edx
0040DBA7  push    esi
0040DBA8  push    edi
0040DBA9  push    ecx
0040DBAA  push    ebx
0040DBAB  push    eax
0040DBAC  push    eax
0040DBAD  push    offset SHELL32.#599
0040DBB2  call    0040D723                       ; GetProcAddress
0040DBB7  pop     ebx
0040DBB8  or      eax, eax
0040DBBA  je      0040D7D2                       ; GetProcAddress 失败？
0040DBC0  call    0040DC3D
0040DBC5  pop     ebx
0040DBC6  pop     ecx
0040DBC7  pop     edi
0040DBC8  call    0040DC54
0040DBCD  pop     esi
0040DBCE  pop     edx
0040DBCF  pop     ebx
0040DBD0  add     esi, 4
0040DBD3  add     edi, 4
0040DBD6  jmp     short 0040DB70                 ; 循环，恢复 IAT
0040DBD8  add     edx, 14
0040DBDB  jmp     0040DAE2                       ; 继续循环，获取下一个 DLL
0040DBE3  mov     eax, dword ptr [ebp+1247]      ; －－IAT 修复完毕－－
0040DBED  xor     eax, dword ptr [ebp+129F]
0040DBF7  add     eax, dword ptr [ebp+12A7]      ; 真正的 OEP 出现了！
0040DC00  mov     dword ptr [esp+1C], eax        ; NOTEPAD.004010CC
0040DC09  lea     edi, dword ptr [ebp+C3D]
0040DC14  mov     ecx, 707
0040DC1C  xor     al, al
0040DC22  rep     stos byte ptr es:[edi]         ; 清空当前段代码
0040DC27  mov     edi, ebp                       ; NOTEPAD.<模块入口点>
0040DC2D  mov     ecx, 0C35
0040DC35  rep     stos byte ptr es:[edi]         ; 清空当前段代码
0040DC37  popad
0040DC3B  jmp     eax                            ; NOTEPAD.004010CC，光明之巅！
```

然后在`004010CC`处`DUMP`，脱壳后的程序可以直接运行，但菜单中的部分功能会导致程序崩溃。用`ImportREC`修复`IAT`，发现有大量无用指针指向`00400000`，直接`CUT`，试运行之，修复成功！

总结：这个壳用了二个非法指令和一个新建线程来更改程序流程，代码中插入了大量的花指令，不过去除花指令后程序代码还是比较易懂的，没有那些双进程保护、动态区段、`Stolen Bytes`之类的变态保护方式，适合我等小菜练手。

后记：

尝试了一下完美优化文件，很有意思！

1. 在`OD`里跟到`OEP`后停下，`DUMP`为`dump.exe`。注意`DUMP`时不用选中`Rebuild Import`重建输入表，因为后面会用`ImportREC`重建的。



### 2. 用 `FixRes` 导出资源文件

最好在导出前用 `FixRes` 修复一下资源。导出时还要注意，由于用 `ImportREC` 修复 `IAT` 时会添加一个新的输入表段，因此导出资源文件的 `RVA` 要再加上一个偏移，在这里要把原 `RVA＝7000` 再加上 `1000`，即 `NewRVA＝8000`。`FileAlignment` 填当前文件的对齐大小，可以用 `LordPE` 查出：`FileAlignment＝1000`。导出资源的文件名为 `rsrc.bin`。

（提示：这里的 `RVA` 要加上的 `1000` 是由于程序的块对齐为 `1000`，而且输入表的大小`<1000`。如果另外一个程序的 `FileAlignment=200`，`IAT SIZE=34C`，那么 `NewRVA` 就要加上 `200×2＝400`。）

### 3. 删除多余的区段

在 `OD` 里按 `ALT+M` 观察文件结构：`407000-40C000` 为资源段，`40C000-40D000` 为重定位表，`40D000-410000` 为 `SFX` 和输入表。资源段我们在后面会重建，重定位表用处不大（其实是我不知道有什么用处，在 `LordPE` 里查看修复完的文件的重定位表显示为 `error`），而后面的 `SFX` 段为壳添加的解压代码，都可以直接删除。

在 `LordPE` 里删除区段只是删除了区段的头信息，物理数据并没有删除。查看到 `ROffset=7000`，用 `WinHex` 打开 `dump.exe`，把 `7000` 之后的所有数据直接删除。

### 4. 修复 `IAT`

在 `ImportREC` 中填 `OEP` 为 `10CC`，用自动搜索，再把无效指针 `CUT` 掉即可。

也可以用手工搜索得到更精确的位置和大小：在 `OD` 中随意选中一个 `API` 调用，比如 `004010D3` 处的 `call dword ptr [4063E4] ; GetCommandLineA`，输入 `dd 4063E4` 可以看到程序用到的所有系统 `API`。

向上翻找到第一个 `API` 的位置：`004062E4  77DA7883  ADVAPI32.RegQueryValueExA`。

向下翻找到最后一个 `API` 的位置：`00406520  76322533  comdlg32.GetFileTitleA`。

因此在 `ImportREC` 里填 `IAT` 的 `RVA` 为 `004062E4－400000－4＝62E0`，`IAT` 的大小为 `00406520－004062E4＋8＝248`，点“获取输入表”可以发现已经没有无效指针了。修复 `IAT` 后的文件为“`dump_.exe`”。

### 5. 添加资源段

在 `LordPE` 里载入 `dump_.exe`，打开区段表，载入前面保存出来的 `rsrc.bin` 即可。注意看一下新添加的区段的 `Vffsett` 和 `VSize` 是否分别为 `8000` 和 `5000`。

### 6. 修复文件目录结构

导入表已经由 `ImportREC` 修复了，不用管了；资源表我们已经把它往后移了，需要修改 `RVA` 为 `8000`；重定位表已经被删除了，需要把 `RVA` 和大小都改为 `0`。

至此已经优化完成，在浏览器中已经识别出图标，测试运行完全通过！

### 总结

文件优化需要多学习 `PE` 文件的结构，在调整区块时涉及很多 `PE` 结构的知识。在优化文件时我遇到二个问题：

1. 为什么一定要导出资源并把资源段放到文件的最后面？
   我的理解是有些资源修改工具（比如 `eXeCope`）只认标准资源，像这种脱壳后修复的资源还是不被接受。
2. 为什么要把重定位表设为空，并删除重定位表段？
   重定位表的作用是如果文件不是载入到默认的基址，而是载入到虚拟内存的另一个地址，链接器所登记的那个地址就是错误的，这时就需要用重定位表来调整。一般来说 `EXE` 文件不需要用到重定位表，而 `DLL` 会经常重定位。在这个程序里重定位表本身就是错误的，因此可以直接删除。

但是当删除重定位表段后，程序的重定位表指向一个非法地址，这样会导致文件在载入到 `OD` 时有一个“非法 `EXE` 文件”的提示，这个问题刚开始困扰了我好长时间，最后终于发现是这个基本用不到的重定位表地址导致的！把重定位表地址和大小都改为 `0` 后就是一个完全合法的 `PE` 文件了。

by `lelfei` on 2007.10.1～

```
4dd466d0e1e0cc123afbe504da2462ca
```
