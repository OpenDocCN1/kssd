
# [原创]菜菜简单分析鬼影2

工具：`ida`、`od`、`msdn`、`ie`、计算器

名称：鬼影2

内容：`pe`结构，`api`，和常见的系统知识……

感谢：论坛，`x-man`（一个经常为大家上传病毒样本的好人）以及群主……

上传：样本，分析笔记（不是很详细），`idb`（这个还相对详细），释放源文件以及脱壳后的文件

---

菜菜简单析毒：（我附上所有的东西……俺分析的肯定是不足的，花了2天的时间，但如果想把这个弄好，对于我还得很多天，还得努力。所以错误肯定是有的，但我学习为主，毕竟这方面，我仅仅是爱好……）

## 概要

主体：释放一个文件

释放文件：真正作用的部分看后面

本次分析只是清明节时，无聊干得，还是有很多地方没弄懂的……抱着学习的态度来的，笔记就发上来了，目的只为理解程序的处理思路。

释放文件部分写的不那么详细了，一方面太多了，另一方面能力有限……（本来考虑发不发上来的，算了，发上来给像我一样的菜菜们……呵呵……所有详细东西看`idb`）

附：具体的东西都会通过附件上传……给想看的人，比如像我一样的菜菜……（原样本都在后面上传了……给想练的人……）

---

内容有点多，贴出来后，看的会很不舒服……所以只是贴出了程序的流程部分代码，还有具体的如果贴出来就太长了，想看的看上传的两份`idb`……呵呵……

## 部分分析如下

```asm
81 EC BC 00 00 00 sub     esp, 0BCh
.text:00401CC6 56                push    esi
.text:00401CC7 57                push    edi
.text:00401CC8 33 F6             xor     esi, esi
.text:00401CCA B9 26 00 00 00    mov     ecx, 26h
.text:00401CCF 33 C0             xor     eax, eax
.text:00401CD1 8D 7C 24 10       lea     edi, [esp+0C4h+VersionInformation.dwMajorVersion]
.text:00401CD5 89 74 24 0C       mov     [esp+0C4h+VersionInformation.dwOSVersionInfoSize], esi
.text:00401CD9 89 74 24 08       mov     [esp+0C4h+ThreadId], esi
.text:00401CDD F3 AB             rep stosd
.text:00401CDF FF 15 F4 60 40 00 call    ds:GetInputState
.text:00401CE5 56                push    esi                   ; lParam
.text:00401CE6 56                push    esi                   ; wParam
.text:00401CE7 56                push    esi                   ; Msg
.text:00401CE8 FF 15 38 60 40 00 call    ds:GetCurrentThreadId
.text:00401CEE 50                push    eax                   ; idThread
.text:00401CEF FF 15 F0 60 40 00 call    ds:PostThreadMessageA
.text:00401CF5 56                push    esi                   ; wMsgFilterMax
.text:00401CF6 56                push    esi                   ; wMsgFilterMin
.text:00401CF7 8D 84 24 B0 00 00+lea     eax, [esp+0CCh+Msg]
.text:00401CFE 56                push    esi                   ; hWnd
.text:00401CFF 50                push    eax                   ; lpMsg
.text:00401D00 FF 15 FC 60 40 00 call    ds:GetMessageA
.text:00401D06 E8 45 FD FF FF    call    sub_401A50                        ; 关键call
```

---

上面的部分很简单，有些调用没什么用感觉，如果谁知道，告诉菜菜……

---

```asm
00401A8E 6A 00             push    0                     ; lpsa
.text:00401A90 68 00 00 00 10    push    10000000h             ; dwDesiredAccess
.text:00401A95 6A 01             push    1                     ; dwFlags
.text:00401A97 66 AB             stosw
.text:00401A99 6A 00             push    0                     ; pDevmode
.text:00401A9B 8D 4C 24 24       lea     ecx, [esp+23Ch+szDesktop]
.text:00401A9F 6A 00             push    0                     ; lpszDevice
.text:00401AA1 51                push    ecx                   ; lpszDesktop
.text:00401AA2 89 54 24 34       mov     [esp+244h+var_210], edx
.text:00401AA6 AA                stosb
.text:00401AA7 FF 15 00 61 40 00 call    ds:CreateDesktopA     ; 创建第二桌面，至于为什么下面可以看到
.text:00401AAD 8B F0             mov     esi, eax
.text:00401AAF 89 74 24 10       mov     [esp+22Ch+hDesktop], esi
.text:00401AB3 FF 15 38 60 40 00 call    ds:GetCurrentThreadId
.text:00401AB9 50                push    eax                   ; dwThreadId
.text:00401ABA FF 15 F8 60 40 00 call    ds:GetThreadDesktop
.text:00401AC0 56                push    esi                   ; hDesktop
.text:00401AC1 FF 15 EC 60 40 00 call    ds:SetThreadDesktop   ; 上面的操作就是把当前的线程附加到自己创建的桌面上运行
.text:00401AC7 8B 35 0C 61 40 00 mov     esi, ds:DeleteUrlCacheEntry
```

---

检重点的说：看这个call

---

```asm
E8 6B FD FF FF    call    sub_401910                           ; 关键跟进，看看什么行为
.text:00401BA5 83 C4 08          add     esp, 8
.text:00401BA8 8B F0             mov     esi, eax
.text:00401BAA 8D 84 24 28 01 00+lea     eax, [esp+22Ch+FileName]
.text:00401BB1 50                push    eax                   ; lpFileName
.text:00401BB2 6A 00             push    0                     ; nDefault
.text:00401BB4 68 74 EB 40 00    push    offset aPopspacingdela ; "PopSpacingDelayTime"
.text:00401BB9 68 94 EB 40 00    push    offset AppName        ; "Pop"
.text:00401BBE FF D5             call    ebp ; GetPrivateProfileIntA
.text:00401BC0 8D 04 80          lea     eax, [eax+eax*4]
.text:00401BC3 8D 04 80          lea     eax, [eax+eax*4]
.text:00401BC6 8D 0C 80          lea     ecx, [eax+eax*4]
.text:00401BC9 C1 E1 03          shl     ecx, 3
.text:00401BCC 51                push    ecx                   ; dwMilliseconds
.text:00401BCD FF 15 34 60 40 00 call    ds:Sleep
.text:00401BD3 6A 00             push    0                     ; uExitCode
.text:00401BD5 56                push    esi                   ; hProcess
.text:00401BD6 FF 15 28 60 40 00 call    ds:TerminateProcess
.text:00401BDC
.text:00401BDC                   loc_401BDC:                   ; CODE XREF: sub_401A50+144_j
.text:00401BDC 83 FB 64          cmp     ebx, 64h
.text:00401BDF 0F 8C 4E FF FF FF jl      loc_401B33            ; 这里就体现了，一系列访问。。
.text:00401BDF                                                 ; 这些地方都比较明显，一看就基本知道
.text:00401BDF                                                 ; 但他用虚拟桌面进行访问，还是可以学习到东东的
```

---

跟进call

---

```asm
.text:00401A00 51                push    ecx
.text:00401A01 68 38 EB 40 00    push    offset aSS            ; "\"%s\" \"%s\""
.text:00401A06 52                push    edx                   ; LPSTR
.text:00401A07 FF 15 E8 60 40 00 call    ds:wsprintfA          ; 格式化要访问的东西和地址，od跑起来就看得到
.text:00401A0D 83 C4 10          add     esp, 10h
.text:00401A10 8D 44 24 10       lea     eax, [esp+364h+ProcessInformation]
.text:00401A14 8D 4C 24 20       lea     ecx, [esp+364h+StartupInfo]
.text:00401A18 50                push    eax                   ; lpProcessInformation
.text:00401A19 51                push    ecx                   ; lpStartupInfo
.text:00401A1A 53                push    ebx                   ; lpCurrentDirectory
.text:00401A1B 53                push    ebx                   ; lpEnvironment
.text:00401A1C 68 00 00 00 04    push    4000000h              ; dwCreationFlags
.text:00401A21 53                push    ebx                   ; bInheritHandles
.text:00401A22 53                push    ebx                   ; lpThreadAttributes
.text:00401A23 8D 94 24 80 01 00+lea     edx, [esp+380h+CommandLine]
.text:00401A2A 53                push    ebx                   ; lpProcessAttributes
.text:00401A2B 52                push    edx                   ; lpCommandLine
.text:00401A2C 53                push    ebx                   ; lpApplicationName
.text:00401A2D FF 15 14 60 40 00 call    ds:CreateProcessA     ; 上面一系列的操作就是，在自己创建的桌面上
.text:00401A2D                                                 ; 创建进程，然后访问偶们列表中的网点
```

---

看图，当执行进程时：下面的那个网页会不断地下载、变化。

![](https://bbs.kanxue.com/upload/attach/201104/356233_qlfsyp4cqg8gwlw.jpg)

---

这里下载的东西只是放着，后面我们就看到怎么用。

---

很简单的东西，`ida`上有详细解释，只看重点。

---

```asm
8D 54 24 08       lea     edx, [esp+0C4h+ThreadId]
.text:00401D50 52                push    edx                   ; lpThreadId
.text:00401D51 56                push    esi                   ; dwCreationFlags
.text:00401D52 56                push    esi                   ; lpParameter
.text:00401D53 68 70 14 40 00    push    offset StartAddress   ; lpStartAddress
.text:00401D58 56                push    esi                   ; dwStackSize
```

