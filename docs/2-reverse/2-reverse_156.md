
# 让 OllyDbg 1.10 自动适应并创建 UDD 和 插件 路径

> 原文：[`www.pediy.com/kssd/pediy12/138598.html`](https://www.pediy.com/kssd/pediy12/138598.html)

拿原版的 `od` 开刀，让它无论如何都只使用当前目录下的 `udd` 和 `plugins` 目录。

这样在移动了 `od` 的位置以后，不需要手动设置目录。

如果当前目录下不存在 `udd` 和 `plugins` 目录，则自动创建。

找到原版 `od` 读取数据的地方：

代码:

```
udd

004374E4   .  68 A4534D00                     push    004D53A4                                  ; /IniFileName = "E:\\Software packges\\Debugger\\OllyDbg\\ollydbg.ini"
004374E9   .  68 04010000                     push    104                                       ; |BufSize = 104 (260.)
004374EE   .  68 6C394D00                     push    004D396C                                  ; |ReturnBuffer = OLLYDBG.004D396C
004374F3   .  68 68384D00                     push    004D3868                                  ; |Default = "E:\\Software packges\\Debugger\\OllyDbg"
004374F8   .  8D8E 171B0000                   lea     ecx, dword ptr \[esi+1B17\]                 ; |
004374FE   .  51                              push    ecx                                       ; |Key
004374FF   .  8D86 981A0000                   lea     eax, dword ptr \[esi+1A98\]                 ; |
00437505   .  50                              push    eax                                       ; |Section
00437506   .  E8 7F7B0700                     call    <jmp.&KERNEL32.GetPrivateProfileStringA>  ; \\GetPrivateProfileStringA

plugin

0043750B   .  68 A4534D00                     push    004D53A4                                  ; /IniFileName = "E:\\Software packges\\Debugger\\OllyDbg\\ollydbg.ini"
00437510   .  68 04010000                     push    104                                       ; |BufSize = 104 (260.)
00437515   .  68 703A4D00                     push    004D3A70                                  ; |ReturnBuffer = OLLYDBG.004D3A70
0043751A   .  68 68384D00                     push    004D3868                                  ; |Default = "E:\\Software packges\\Debugger\\OllyDbg"
0043751F   .  8D96 201B0000                   lea     edx, dword ptr \[esi+1B20\]                 ; |
00437525   .  52                              push    edx                                       ; |Key
00437526   .  8D8E 981A0000                   lea     ecx, dword ptr \[esi+1A98\]                 ; |
0043752C   .  51                              push    ecx                                       ; |Section
0043752D   .  E8 587B0700                     call    <jmp.&KERNEL32.GetPrivateProfileStringA>  ; \\GetPrivateProfileStringA
```

把上面的代码修改为：

代码:

```
004374E4     68 68384D00    push OLLYDBG.004D3868
004374E9     68 43F64A00    push OLLYDBG.004AF643                    ;  ASCII "%s\\UDD"
004374EE     68 6C394D00    push OLLYDBG.004D396C
004374F3     E8 0E810700    call <jmp.&USER32.wsprintfA>
004374F8     68 68384D00    push OLLYDBG.004D3868
004374FD     68 4AF64A00    push OLLYDBG.004AF64A                    ;  ASCII "%s\\Plugins"
00437502     68 703A4D00    push OLLYDBG.004D3A70
00437507     E8 FA800700    call <jmp.&USER32.wsprintfA>
0043750C     6A 00          push 0
0043750E     68 6C394D00    push OLLYDBG.004D396C
00437513     E8 AC7A0700    call <jmp.&KERNEL32.CreateDirectoryA>
00437518     6A 00          push 0
0043751A     68 703A4D00    push OLLYDBG.004D3A70
0043751F     E8 A07A0700    call <jmp.&KERNEL32.CreateDirectoryA>
```

保存，完工！

附件是修改后的原版 OD。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `OLLYDBG.rar` |
