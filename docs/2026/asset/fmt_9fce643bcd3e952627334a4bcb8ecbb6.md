
```
000000014031FB09                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.CurrentDirectory]

000000014031FB0D                 lea     r8, [rsp+118h+Dst]

000000014031FB15                 lea     rcx, [rsp+118h+CurrentDirectory] ; src

000000014031FB1A                 call    PspCopyUnicodeString

000000014031FB1F                 mov     esi, eax

000000014031FB21                 cmp     eax, ebx

000000014031FB23                 jl      loc_1403CA0AC

000000014031FB29                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.DllPath]

000000014031FB2D                 lea     r8, [rsp+118h+Dst]

000000014031FB35                 lea     rcx, [rsp+118h+dllPath]

000000014031FB3A                 call    PspCopyUnicodeString

000000014031FB3F                 mov     esi, eax

000000014031FB41                 cmp     eax, ebx

000000014031FB43                 jl      loc_1403CA0AC

000000014031FB49                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.ImagePathName]

000000014031FB4D                 lea     r8, [rsp+118h+Dst]

000000014031FB55                 lea     rcx, [rsp+118h+ImagePathName]

000000014031FB5D                 call    PspCopyUnicodeString

000000014031FB62                 mov     esi, eax

000000014031FB64                 cmp     eax, ebx

000000014031FB66                 jl      loc_1403CA0AC

000000014031FB6C                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.CommandLine]

000000014031FB70                 lea     r8, [rsp+118h+Dst]

000000014031FB78                 lea     rcx, [rsp+118h+CommandLine]

000000014031FB80                 call    PspCopyUnicodeString

000000014031FB85                 mov     esi, eax

000000014031FB87                 cmp     eax, ebx

000000014031FB89                 jl      loc_1403CA0AC

000000014031FB8F                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.WindowTitle]

000000014031FB96                 lea     r8, [rsp+118h+Dst]

000000014031FB9E                 lea     rcx, [rsp+118h+WindowTitle]

000000014031FBA6                 call    PspCopyUnicodeString

000000014031FBAB                 mov     esi, eax

000000014031FBAD                 cmp     eax, ebx

000000014031FBAF                 jl      loc_1403CA0AC

000000014031FBB5                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.DesktopInfo]

000000014031FBBC                 lea     r8, [rsp+118h+Dst]

000000014031FBC4                 lea     rcx, [rsp+118h+DesktopInfo]

000000014031FBCC                 call    PspCopyUnicodeString

000000014031FBD1                 mov     esi, eax

000000014031FBD3                 cmp     eax, ebx

000000014031FBD5                 jl      loc_1403CA0AC

000000014031FBDB                 lea     rdx, [rdi+_RTL_USER_PROCESS_PARAMETERS.ShellInfo] ; Src

000000014031FBE2                 lea     r8, [rsp+118h+Dst] ; Dst

000000014031FBEA                 lea     rcx, [rsp+118h+ShellInfo] ; Size

000000014031FBF2                 call    PspCopyUnicodeString
```

6、最后设置`Flages2`和`pProcessParameters`并退出

```
000000014031FC91                 or      [r13+CreateProcessContext.Flags2], 4

000000014031FC96                 mov     [r13+CreateProcessContext.PRTL_USER_PROCESS_PARAMETERS], rdi

000000014031FC9D                 xor     eax, eax

000000014031FC9F                 add     rsp, 0E0h

000000014031FCA6                 pop     r15

000000014031FCA8                 pop     r14

000000014031FCAA                 pop     r13
```

### IDB文件：

[ntoskrnl.7z.001.rar](https://bbs.kanxue.com/attach-download-103072.htm)

[ntoskrnl.7z.002.rar](https://bbs.kanxue.com/attach-download-103073.htm)

[ntoskrnl.7z.003.rar](https://bbs.kanxue.com/attach-download-103074.htm)

请将IDB文件的后缀名“.rar”删除后用`7z`解压

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [1.png](javascript:void(0);)

  （23.62kb，11次下载）

- [2.png](javascript:void(0);)

  （23.34kb，5次下载）

- [3.png](javascript:void(0);)

  （299.45kb，39次下载）

- [4.png](javascript:void(0);)

  （46.63kb，22次下载）

- [5.png](javascript:void(0);)

  （12.37kb，4次下载）

- [6.png](javascript:void(0);)

  （42.87kb，12次下载）

- [7.png](javascript:void(0);)

  （180.08kb，18次下载）

- [8.png](javascript:void(0);)

  （327.51kb，15次下载）

- [9.png](javascript:void(0);)

  （13.91kb，6次下载）

- [10.png](javascript:void(0);)

  （30.94kb，4次下载）

- [12.png](javascript:void(0);)

  （39.91kb，5次下载）

- [11.png](javascript:void(0);)

  （34.25kb，6次下载）

- [13.png](javascript:void(0);)

  （25.20kb，8次下载）

- [NtCreateUserProcess.doc](javascript:void(0);)

  （1.09MB，128次下载）

- [NtCreateUserProcess.pdf](javascript:void(0);)

  （1.23MB，178次下载）

- [ntoskrnl.7z.001.rar](javascript:void(0);)

  （4.00MB，126次下载）

- [ntoskrnl.7z.002.rar](javascript:void(0);)

  （4.00MB，112次下载）

- [ntoskrnl.7z.003.rar](javascript:void(0);)

  （31.09kb，111次下载）
