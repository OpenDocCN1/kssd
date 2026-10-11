
# 小试削笔刀：ReadBOOK V1.46 注册

> 原文：[`www.pediy.com/kssd/pediy11/119070.html`](https://www.pediy.com/kssd/pediy11/119070.html)

**目标**: `ReadBOOK` V1.46

**工具**: `PEiD0.94`、`OD`、`w32dasm`

**任务**: 小软件练手。

**平台**: `WinXp.sp3`

**作者**: `fairynull` [`pediy`]

**申明**：新人只是出于练习，没有别的目的，请大家支持原作者！软件有个特点，自校验，还回自修复，只要你愿意和话。

因为我不会看算法，不会算注册码，想看算法的，可以越过。

1. 先看 `peid v0.94` 检测软件有无加密，检查结果为：`PEtite 2.x [Level 1/9]-> Ian Luck`。
2. 再用 `pdid` 的 `genOEP` 插件检测其入口点为:`426f67`。
3. 再用 `pdiy` 的 `gen-unpack` 插件强制脱克，填入上面查找的入口点选重建输入表。
4. 运行软件后，能正常运行。并用 `OD` 载入无加密提示。
    运行软件是，软件会提示自身已被病毒修改，点击是：软件会自动修复。点击否：进入软件。
5. 现在，我们要想办法把这个提示弄掉。用 `OD` 载入后，找不到提示字符。我换用 `w32dasm` 载入调试的程序，此时，可以明确的看到此提示所在的位置。用 `OD` 再载入软件。`^G` 到下面位置。如下：

```
00417DD2  /74 2D  je      short 00417E01   A 处：jmp  short 00417dea                        ;   
00417DD4   |6A 14        push    14
00417DD6   |68 D8794600  push    004679D8
00417DDB . |68 BC774600  push    004677BC                             ; |Text = "    ReadBook",B2,"炀醯",BD,"",D7,"陨碛?,B8,"谋洌",AC,"如果",B2,"",BB,"是文件损",BB,"档木褪侨?,B2,"",A1,"",B6,"?,B8,"腥玖耍",A1,LF,"    ?,BB,"",B0,"闼道",B4,"?,AC,"?,B2,"盘很少损",BB,"担",AC,"如果您的 ReadBook 是",B0,"",B2,"",D7,"",B0..提示被病毒修改，是否修复,修复的话，程序回退出，之后，程序会回到未脱壳之前的状态。
00417DE0 . |53            push    ebx                                   ; |hOwner
00417DE1   |FF15 3C364500 call    dword ptr [<&USER32.MessageBoxA>]     ;  USER32.MessageBoxA     此处就是提示消息框，不过，并不能直接 nop 掉，直接 Nop 掉的话，程序会运行出错，不解！ 试了 N 遍终于找出一个方法，虽然不是最优的，至少能解决掉提示的问题。方法如下：把流程改了一下。从 A 处开始：
00417DE7 . |83F8 06       cmp     eax, 6   
00417DEA   |0F85 0D010000 jnz     00417EFD  B 处： mov     eax, 7  我这样改的原因是：当我正常运行程序时，点击否的时候，eax 变为 7,并程序直接能正常运行，我直接跳到
00417DF0   |E8 BDEEFFFF   call    00416CB2  C 处把这个改为 jmp     00417EFD 修改完毕后，再保存，运行程序就没有那个病毒提示框了。
00417DF4    90            nop
00417DF5 . |53            push    ebx                                   ; /ExitCode
00417DF6 . |FF15 FC344500 call    dword ptr [<&USER32.PostQuitMessage>] ; \\PostQuitMessage
00417DFC . |E9 FC000000   jmp     00417EFD
00417E01 > \\6A 10        push    10                                  ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
00417E03 .  68 9C774600   push    0046779C                             ; |Title = "这里是 ReadBOOK 的",B2,"",A1,"",B6,"炯?,B2,"夤",A6,"能"
00417E08 .  68 B0754600   push    004675B0                             ; |Text = "    ReadBook",B2,"炀醯",BD,"",D7,"陨碛?,B8,"谋洌",AC,"如果",B2,"",BB,"是文件损",BB,"档木褪侨?,B2,"",A1,"",B6,"?,B8,"腥玖耍",A1,LF,"    ?,BB,"",B0,"闼道",B4,"?,AC,"?,B2,"盘很少损",BB,"担",AC,"如果您的 ReadBook 是",B0,"",B2,"",D7,"",B0..提示被病毒严重修改，无法修复
00417E0D .  53            push    ebx                                   ; |hOwner
00417E0E .  FF15 3C364500 call    dword ptr [<&USER32.MessageBoxA>]     ; \\MessageBoxA
00417E14 .  E9 E4000000   jmp     00417EFD
00417E19 >  8D85 F0DEFFFF lea     eax, dword ptr [ebp-2110]
00417E1F .  50            push    eax
00417E20 .  E8 BBB20000  call    004230E0
00417E25 .  85C0          test    eax, eax
00417E27 .  59            pop     ecx
00417E28 .  0F85 CF000000 jnz     00417EFD
00417E2E .  8B3D 60324500 mov     edi, dword ptr [<&KERNEL32.CreateFileA>]   ;  kernel32.CreateFileA
00417E34 .  53            push    ebx                                   ; /hTemplateFile
00417E35 .  53            push    ebx                                   ; |Attributes
00417E36 .  6A 03         push    3                                       ; |Mode = OPEN_EXISTING
00417E38 .  53            push    ebx                                   ; |pSecurity
00417E39 .  53            push    ebx                                   ; |ShareMode
00417E3A .  8D85 F0DEFFFF lea     eax, dword ptr [ebp-2110]                   ; |
00417E40 .  68 000000C0   push    C0000000                             ; |Access = GENERIC_READ|GENERIC_WRITE
00417E45 .  50            push    eax                                   ; |FileName
00417E46 .  FFD7          call    edi                                   ; \\CreateFileA
00417E48 .  83F8 FF       cmp     eax, -1
00417E4B .  8945 EC       mov     dword ptr [ebp-14], eax
00417E4E .  895D E4       mov     dword ptr [ebp-1C], ebx
00417E51 .  53            push    ebx                                   ; /pOverlapped
00417E52 .  74 1D         je      short 00417E71                         ; |
00417E54 .  8D45 DC       lea     eax, dword ptr [ebp-24]                 ; |
00417E57 .  50            push    eax                                   ; |pBytesRead
00417E58 .  8D45 D4       lea     eax, dword ptr [ebp-2C]                 ; |
00417E5B .  6A 04         push    4                                       ; |BytesToRead = 4
00417E5D .  50            push    eax                                   ; |Buffer
00417E5E .  FF75 EC       push    dword ptr [ebp-14]                     ; |hFile
00417E61 .  FF15 10324500 call    dword ptr [<&KERNEL32.ReadFile>]       ; \\ReadFile
00417E67 .  8B45 D4       mov     eax, dword ptr [ebp-2C]
00417E6A .  3B45 F0       cmp     eax, dword ptr [ebp-10]
00417E6D .  74 6B         je      short 00417EDA
00417E6F .  EB 16         jmp     short 00417E87
00417E71 >  53            push    ebx
00417E72 .  6A 02         push    2
00417E74 .  53            push    ebx
00417E75 .  53            push    ebx
00417E76 .  8D85 F0DEFFFF lea     eax, dword ptr [ebp-2110]
00417E7C .  68 00000040   push    40000000
00417E81 .  50            push    eax
00417E82 .  FFD7          call    edi
00417E84 .  8945 EC       mov     dword ptr [ebp-14], eax
00417E87 >  53            push    ebx                                   ; /Origin
00417E88 .  53            push    ebx                                   ; |pOffsetHi
00417E89 .  53            push    ebx                                   ; |OffsetLo
00417E8A .  C745 E4 01000>mov     dword ptr [ebp-1C], 1                  ; |
00417E91 .  FF75 EC       push    dword ptr [ebp-14]                     ; |hFile
00417E94 .  FF15 84334500 call    dword ptr [<&KERNEL32.SetFilePointer>] ; \\SetFilePointer
00417E9A .  FF75 EC       push    dword ptr [ebp-14]                     ; /hFile
00417E9D .  FF15 64334500 call    dword ptr [<&KERNEL32.SetEndOfFile>]   ; \\SetEndOfFile
00417EA3 .  8D45 D4       lea     eax, dword ptr [ebp-2C]
00417EA6 .  8B3D 64324500 mov     edi, dword ptr [<&KERNEL32.WriteFile>] ;  kernel32.WriteFile
00417EAC .  53            push    ebx                                   ; /pOverlapped
00417EAD .  50            push    eax                                   ; |pBytesWritten
00417EAE .  8D45 F0       lea     eax, dword ptr [ebp-10]                 ; |
00417EB1 .  6A 04         push    4                                       ; |nBytesToWrite = 4
00417EB3 .  50            push    eax                                   ; |Buffer
00417EB4 .  FF75 EC       push    dword ptr [ebp-14]                     ; |hFile
00417EB7 .  FFD7          call    edi                                   ; \\WriteFile
00417EB9 .  8D45 D4       lea     eax, dword ptr [ebp-2C]
00417EBC .  53            push    ebx                                   ; /pOverlapped
00417EBD .  50            push    eax                                   ; |pBytesWritten
00417EBE .  6A 04         push    4                                       ; |nBytesToWrite = 4
00417EC0 .  68 AC754600   push    004675AC                             ; |Buffer = ReadBook.004675AC
```



00417EC5   .  FF75 EC       push    dword ptr \[ebp-14\]                         ; |hFile  

00417EC8   .  FFD7          call    edi                                        ; \\WriteFile  

00417ECA   .  8D45 D4       lea     eax, dword ptr \[ebp-2C\]  

00417ECD   .  53            push    ebx                                        ; /pOverlapped  

00417ECE   .  50            push    eax                                        ; |pBytesWritten  

00417ECF   .  FF75 D8       push    dword ptr \[ebp-28\]                         ; |nBytesToWrite  

00417ED2   .  FF75 E8       push    dword ptr \[ebp-18\]                         ; |Buffer  

00417ED5   .  FF75 EC       push    dword ptr \[ebp-14\]                         ; |hFile  

00417ED8   .  FFD7          call    edi                                        ; \\WriteFile  

00417EDA   >  837D EC FF    cmp     dword ptr \[ebp-14\], -1  

00417EDE   .  74 1D         je      short 00417EFD  

00417EE0   .  FF75 EC       push    dword ptr \[ebp-14\]                         ; /hObject  

00417EE3   .  FF15 68324500 call    dword ptr \[<&KERNEL32.CloseHandle>\]        ; \\CloseHandle  

00417EE9   .  395D E4       cmp     dword ptr \[ebp-1C\], ebx  

00417EEC   .  74 0F         je      short 00417EFD  

00417EEE   .  8D85 F0DEFFFF lea     eax, dword ptr \[ebp-2110\]  

00417EF4   .  6A 02         push    2                                          ; /FileAttributes = HIDDEN  

00417EF6   .  50            push    eax                                        ; |FileName  

00417EF7   .  FF15 5C324500 call    dword ptr \[<&KERNEL32.SetFileAttributesA>\] ; \\SetFileAttributesA  

00417EFD   >  FF75 E8       push    dword ptr \[ebp-18\]  

```
00417EC5   .  FF75 EC       push    dword ptr [ebp-14]                        ; |hFile  
00417EC8   .  FFD7         call    edi                                      ; \WriteFile  
00417ECA   .  8D45 D4      lea     eax, dword ptr [ebp-2C]  
00417ECD   .  53           push    ebx                                      ; /pOverlapped  
00417ECE   .  50           push    eax                                      ; |pBytesWritten  
00417ECF   .  FF75 D8      push    dword ptr [ebp-28]                       ; |nBytesToWrite  
00417ED2   .  FF75 E8      push    dword ptr [ebp-18]                      ; |Buffer  
00417ED5   .  FF75 EC      push    dword ptr [ebp-14]                      ; |hFile  
00417ED8   .  FFD7         call    edi                                      ; \WriteFile  
00417EDA   >  837D EC FF   cmp     dword ptr [ebp-14], -1  
00417EDE   .  74 1D        je      short 00417EFD  
00417EE0   .  FF75 EC      push    dword ptr [ebp-14]                      ; /hObject  
00417EE3   .  FF15 68324500 call   dword ptr [<&KERNEL32.CloseHandle>]     ; \CloseHandle  
00417EE9   .  395D E4      cmp     dword ptr [ebp-1C], ebx  
00417EEC   .  74 0F        je      short 00417EFD  
00417EEE   .  8D85 F0DEFFFF lea    eax, dword ptr [ebp-2110]  
00417EF4   .  6A 02        push    2                                        ; /FileAttributes = HIDDEN  
00417EF6   .  50          push    eax                                      ; |FileName  
00417EF7   .  FF15 5C324500 call   dword ptr [<&KERNEL32.SetFileAttributesA>] ; \SetFileAttributesA  
00417EFD   >  FF75 E8      push    dword ptr [ebp-18]  
```

至此，病毒提示框的问题已解决。

## 注册码的问题

注册码的问题比较好解决，修改 1 个关键跳的地方就行了。用 `w32dasm` 找到字符串位置。再用 `OD` 载入。找到

```
00409E73  |.  /75 78         jnz     short 00409EED  
```

改为 `jz` 就可以了。

到此，至此完毕。

![](img/c4dbab1e53d1b0a7168e3616e63b781e.jpg)
