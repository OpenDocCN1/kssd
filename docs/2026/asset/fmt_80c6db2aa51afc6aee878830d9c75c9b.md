
```
00403375  |.  50            push eax                                        ;  C:\Program File\Common Files\Common Filesa\System\ke\*\*\*\*\*\*.dla
00403376  |.  FFD7          call edi                                        ;  edi = GetFileAttributesA,
00403378  |.  0C 02         or al,0x2                                       ;  与上 FILE\_ATTRIBUTE\_HIDDEN
0040337A  |.  50            push eax
0040337B  |.  8D85 48FEFFFF lea eax,[local.110]
00403381  |.  50            push eax
00403382  |.  FFD3          call ebx                                        ;  SetFIleAttributes  隐藏文件
```

3. 加载临时文件，获取他的导出函数`LoadDll`，然后调用`LoadDll`安全全局钩子（钩子类型`WH_GETMESSAGE`）

```
0040339F  |.  C645 C5 6C    mov byte ptr ss:[ebp-0x3B],0x6C
004033A3  |.  C645 C6 6C    mov byte ptr ss:[ebp-0x3A],0x6C                 ;  LoadDll
004033A7  |.  FFD6          call esi                                        ;  LoadLibrary （加载临时文件）
004033A9  |.  8D4D C0       lea ecx,[local.16]
004033AC  |.  51            push ecx                                        ;  LoadDll
004033AD  |.  50            push eax                                        ;  hModule
004033AE  |.  E8 6DE5FFFF   call Cracker.00401920
004033B3  |.  59            pop ecx
004033B4  |.  59            pop ecx
004033B5  |.  85C0          test eax,eax
004033B7  |.  74 08         je XCracker.004033C1
004033B9  |.  FFD0          call eax                                        ;  LoadDll 导出函数
```

跟进`call eax`。

发现安装`WH_GETMESSAGE`类型的全局钩子，在回调函数里都没做，说明这个导出函数目的就是让任何线程调用`GetMessage`或`PeekMessage`时加载这个dll,,,, 感觉这个dll里面很邪恶。

```
10002082    FF7424 0C       push dword ptr ss:[esp+0xC]
10002086    FF7424 0C       push dword ptr ss:[esp+0xC]
1000208A    FF7424 0C       push dword ptr ss:[esp+0xC]
1000208E    FF35 00600010   push dword ptr ds:[0x10006000]
10002094    FF15 DC400010   call dword ptr ds:[0x100040DC]                  ; USER32.CallNextHookEx
1000209A    C2 0C00         retn 0xC
1000209D >  6A 00           push 0x0                                        ; 0 全局钩子
1000209F    FF35 00530010   push dword ptr ds:[0x10005300]                  ; kb372004.10000000
100020A5    68 82200010     push kb372004.10002082                          ; Hook\_CallBack
100020AA    6A 03           push 0x3                                        ; WH\_GETMESSAGE
100020AC    FF15 D8400010   call dword ptr ds:[0x100040D8]                  ; USER32.SetWindowsHookExA
100020B2    A3 00600010     mov dword ptr ds:[0x10006000],eax
100020B7    C3              retn
100020B8 >  FF35 00600010   push dword ptr ds:[0x10006000]
100020BE    FF15 D4400010   call dword ptr ds:[0x100040D4]                  ; USER32.UnhookWindowsHookEx
100020C4    C3              retn
```

4. 判断`C:\windows\system32\dsound.dll`文件是都存在，存在就拷贝一份，命名为`C:\windows\system32\dsound.dll.dat`

```
00402411  |.  C645 D8 41    mov byte ptr ss:[ebp-0x28],0x41                 ;  CopyFile
00402415  |.  885D D9       mov byte ptr ss:[ebp-0x27],bl
00402418  |.  E8 70F4FFFF   call Cracker.0040188D                           ;  这个call里调用GetSystemDirectory
0040241D  |.  8D85 08FCFFFF lea eax,[local.254]
00402423  |.  50            push eax
00402424  |.  8D85 0CFDFFFF lea eax,[local.189]
0040242A  |.  50            push eax
0040242B  |.  E8 73F2FFFF   call Cracker.004016A3
00402430  |.  FF75 08       push [arg.1]                                    ;  dsound.dll
00402433  |.  8D85 0CFDFFFF lea eax,[local.189]
00402439  |.  50            push eax
0040243A  |.  E8 5BF3FFFF   call Cracker.0040179A                           ;  strcat
0040243F  |.  8D85 0CFDFFFF lea eax,[local.189]                             ;  C:\windows\system32\dsound.dll
00402445  |.  50            push eax
00402446  |.  8D85 10FEFFFF lea eax,[local.124]                             ;  newbuf
0040244C  |.  50            push eax
0040244D  |.  E8 51F2FFFF   call Cracker.004016A3                           ;  memcpy
00402452  |.  8D45 E4       lea eax,[local.7]
00402455  |.  50            push eax                                        ;  .dat
00402456  |.  8D85 10FEFFFF lea eax,[local.124]
0040245C  |.  50            push eax
0040245D  |.  E8 38F3FFFF   call Cracker.0040179A                           ;  C:\windows\system32\dsound.dll.dat
00402462  |.  83C4 24       add esp,0x24
00402465  |.  8D45 D0       lea eax,[local.12]
00402468  |.  50            push eax                                        ;  CopyFile
00402469  |.  68 10604000   push Cracker.00406010                           ; /FileName = "Kernel32.dll"
0040246E  |.  FF15 00504000 call dword ptr ds:[<&kernel32.LoadLibraryA>]    ; \LoadLibraryA
00402474  |.  50            push eax
00402475  |.  E8 A6F4FFFF   call Cracker.00401920
0040247A  |.  8BF8          mov edi,eax                                     ;  edi =  CopyFile
0040247C  |.  8D85 0CFDFFFF lea eax,[local.189]
00402482  |.  50            push eax                                        ;  C:\windows\system32\dsound.dll
00402483  |.  E8 3CF3FFFF   call Cracker.004017C4                           ;  调用FindFirstFile 查看传入参数文件是否存在
00402488  |.  8BF0          mov esi,eax
0040248A  |.  8D85 10FEFFFF lea eax,[local.124]
00402490  |.  50            push eax                                        ;  看C:\windows\system32\dsound.dll.dat是否存在
004024B2  |> \8D85 10FEFFFF lea eax,[local.124]
004024B8  |.  53            push ebx
004024B9  |.  50            push eax
004024BA  |.  8D85 0CFDFFFF lea eax,[local.189]                             ;  C:\windows\system32\dsound.dll.dat
004024C0  |.  50            push eax                                        ;  C:\windows\system32\dsound.dll
004024C1  |.  FFD7          call edi                                        ;  copyfile
004024C3  |>  8D8D 14FFFFFF lea ecx,[local.59]                              ;  系统目录\system\"下是否存在"dsound.dll"文件，如果存在则备份dsound.dll
```

接下来使用备份文件`dsound.dll.bat`

```
004024DD  |.  C645 E1 36    mov byte ptr ss:[ebp-0x1F],0x36           ;  .text6
004024E1  |.  885D E2       mov byte ptr ss:[ebp-0x1E],bl
004024E4  |.  C645 EC 2E    mov byte ptr ss:[ebp-0x14],0x2E
004024E8  |.  C645 ED 74    mov byte ptr ss:[ebp-0x13],0x74
004024EC  |.  C645 EE 65    mov byte ptr ss:[ebp-0x12],0x65
004024F0  |.  C645 EF 78    mov byte ptr ss:[ebp-0x11],0x78
004024F4  |.  C645 F0 74    mov byte ptr ss:[ebp-0x10],0x74
004024F8  |.  C645 F1 38    mov byte ptr ss:[ebp-0xF],0x38            ;  .text8
004024FC  |.  885D F2       mov byte ptr ss:[ebp-0xE],bl
004024FF  |.  E8 6D130000   call Cracker.00403871                     ;  new 后面拷贝备份文件使用
00402504  |.  8D85 10FEFFFF lea eax,[local.124]                       ;  C:\windows\system32\dsound.dll.dat
0040250A  |.  8D8D 14FFFFFF lea ecx,[local.59]
00402510  |.  50            push eax
00402511  |.  895D FC       mov [local.1],ebx
00402514  |.  E8 A5170000   call Cracker.00403CBE
```

跟进`call 00403CBE`

```
00403D5E  |.  6A 00         push 0x0                                  ; /pOverlapped = NULL
00403D60  |.  51            push ecx                                  ; |pBytesRead
00403D61  |.  FF76 04       push dword ptr ds:[esi+0x4]               ; | FileSize
00403D64  |.  50            push eax                                  ; |Buffer
00403D65  |.  57            push edi                                  ; |hFile
00403D66  |.  FF15 10504000 call dword ptr ds:[<&kernel32.ReadFile>]  ; \ReadFile
00403D6C  |.  57            push edi                                  ;  将备份的dsound.dll.bat文件读入缓冲区
```

接下来使用`memcpy`分段拷贝`dsound.dll.bat`到全面准备好的缓冲区中

```
00403D6D  |.  FF5424 14     call dword ptr ss:[esp+0x14]
00403D71  |.  55            push ebp                                  ; /40  拷贝dsound.dll.bat文件pe头前0x40字节到00393AD0
00403D72  |.  FF36          push dword ptr ds:[esi]                   ; |src
00403D74  |.  FF76 08       push dword ptr ds:[esi+0x8]               ; |dest
00403D77  |.  E8 B00A0000   call <jmp.&MSVCRT.memcpy>                 ; \memcpy
```
