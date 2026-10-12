
```assembly
004017F7             push    offset dword_410B18  
004017FC             push    eax  
004017FD             call    ebx ; lstrcpy  
004017FF             push    0               ; lpOverlapped  
00401801             lea     eax, [ebp+NumberOfBytesWritten]  
00401804             push    eax             ; lpNumberOfBytesWritten  
00401805             lea     eax, [ebp+Buffer]  
0040180B             push    eax  
0040180C             call    esi ; lstrlen  
0040180E             push    eax             ; nNumberOfBytesToWrite  
0040180F             lea     eax, [ebp+Buffer]  
00401815             push    eax             ; lpBuffer  
00401816             push    [ebp+hObject]   ; hFile  
00401819             call    edi ; WriteFile  
0040181B             push    1  
0040181D             pop     edi  
0040181E  
0040181E loc_40181E:  
0040181E             push    [ebp+hObject]   ; hObject  
00401821             call    ds:CloseHandle  
00401827             lea     eax, [ebp+var_szDelCmdLine]  
0040182D             push    SW_HIDE         ; uCmdShow  
0040182F             push    eax             ; lpCmdLine  
00401830             call    ds:WinExec      ; 执行 del.bat  
00401836  
00401836 loc_401836:  
00401836             mov     eax, edi  
00401838  
00401838 loc_401838:  
00401838             pop     edi  
00401839             pop     esi  
0040183A             pop     ebx  
0040183B             leave  
0040183C             retn  
0040183C Del_InfectingHost endp  
```

## 总结

判处 `auto.exe` 病毒死刑，剥夺政治权利终身，此审为终审判决，不得上诉。

加个脱了壳的尸体：

### 上传的附件

![image](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) `Unpacked_auto.rar`

解压密码：`PEDIY`
