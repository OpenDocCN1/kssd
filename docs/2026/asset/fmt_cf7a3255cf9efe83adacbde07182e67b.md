
```
00417ECF   .  FF75 D8       push    dword ptr [ebp-28]                         ; |nBytesToWrite
00417ED2   .  FF75 E8       push    dword ptr [ebp-18]                         ; |Buffer
00417ED5   .  FF75 EC       push    dword ptr [ebp-14]                         ; |hFile
00417ED8   .  FFD7          call    edi                                        ; \WriteFile
00417EDA   >  837D EC FF    cmp     dword ptr [ebp-14], -1
00417EDE   .  74 1D         je      short 00417EFD
00417EE0   .  FF75 EC       push    dword ptr [ebp-14]                         ; /hObject
00417EE3   .  FF15 68324500 call    dword ptr [<&KERNEL32.CloseHandle>]        ; \CloseHandle
00417EE9   .  395D E4       cmp     dword ptr [ebp-1C], ebx
00417EEC   .  74 0F         je      short 00417EFD
00417EEE   .  8D85 F0DEFFFF lea     eax, dword ptr [ebp-2110]
00417EF4   .  6A 02         push    2                                          ; /FileAttributes = HIDDEN
00417EF6   .  50            push    eax                                        ; |FileName
00417EF7   .  FF15 5C324500 call    dword ptr [<&KERNEL32.SetFileAttributesA>] ; \SetFileAttributesA
00417EFD   >  FF75 E8       push    dword ptr [ebp-18]
```

至此，病毒提示框的问题已解决。

6、注册码的问题比较好解决，修改1个关键跳的地方就行了。用`w32dasm`找到字符串位置。再用`OD`载入。找到：

```
00409E73  |. /75 78         jnz     short 00409EED
```

改为`jz`就可以了。

到此，至此完毕。

![](https://bbs.kanxue.com/upload/attach/201008/192638_c4ajf7w6uriskdo.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`修改后.jpg`](javascript:void(0);)

  （15.67kb，124次下载）
