
# 打造对抗 OpenProcess 检测的 OD

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-674.htm`](https://www.pediy.com/kssd/pediy07/pediy7-674.htm)

有些壳，如 `ExeCrypter 2.2.x`，用 `OpenProcess("CSRSS.EXE")` 的方法来检测 `OD`。

其原理如下：

普通进程是没有 `SeDebugPrivilege` 的，这时 `OpenProcess(系统进程)` 不能成功。

但用 `OD` 调试时，`OpenProcess(系统进程)` 成功，壳利用这一点来检测 `OD`。

我们先来看两个例子。

## 第一个例子

`test.exe`

```asm
=====================================================================================================
include masm32.inc

.const
szNewline         dd  0a0dh
szTitle           db  'Process can be opened',0

.data
align 4
hSnap             dd 0
pe32              PROCESSENTRY32 <0>
szTemp            db  1024 dup (0)

.code
start:
  pushad
  invoke CreateToolhelp32Snapshot, TH32CS_SNAPPROCESS, 0
  mov  hSnap, eax
  lea  esi, szTemp
  lea  ebx, pe32
assume ebx : ptr PROCESSENTRY32
  mov [ebx].dwSize, sizeof pe32
  invoke Process32First, hSnap, ebx
  .while eax
     mov eax, [ebx].th32ProcessID
     invoke OpenProcess, PROCESS_ALL_ACCESS, 0, eax
     .if eax
         invoke CloseHandle, eax
         lea eax, [ebx].szExeFile
         invoke lstrcat, esi, eax
         invoke lstrcat, esi, offset szNewline
     .endif
     invoke Process32Next, hSnap, ebx
  .endw
assume ebx : nothing
  invoke CloseHandle, hSnap
  invoke MessageBox, 0, esi, offset szTitle, MB_OK
  popad
  invoke ExitProcess, 0
end start
=====================================================================================================
```

`test.exe` 是用来快照系统当前的进程，显示可以 `OpenProcess` 的进程。

用和不用 `OD` 来运行的结果不一样。

## 第二个例子

`OD1.exe`

```asm
==================================================================================================================
include masm32.inc

.const
szNewline         dd  0a0dh
szOk              db  'OK',0
szErr             db  'Error',0
szTitle           db  'Adjust Debug Privilege',0
szDebugName       db  'SeDebugPrivilege',0
szTest            db  'test.exe',0

.data
align 4
hToken            dd 0
tkp               TOKEN_PRIVILEGES  <>
pInfo             PROCESS_INFORMATION <0>
sInfo             STARTUPINFO <0>
szTemp            db  1024 dup (0)

.code
; 改变进程的 Debug 权限
AdjustPrivilege proc uses esi, hProcess:dword, bEnable:dword
 local retVal : dword
  mov retVal, 0
  lea esi, tkp
  mov dword ptr [esi], 1

  .if bEnable
     mov dword ptr [esi+0Ch], SE_PRIVILEGE_ENABLED
  .else
     mov dword ptr [esi+0Ch], 0
  .endif

  mov eax, esi
  add eax, 4
  invoke LookupPrivilegeValue, 0, offset szDebugName, eax
  .if eax
     invoke OpenProcessToken, hProcess, TOKEN_ADJUST_PRIVILEGES or TOKEN_QUERY, offset hToken
     .if eax
         invoke AdjustTokenPrivileges, hToken, 0, esi, 0, 0, 0
         mov retVal, eax
         invoke CloseHandle, hToken
     .endif
  .endif
  mov eax, retVal
  ret
AdjustPrivilege endp

start:
  pushad
  lea esi, sInfo
  mov dword ptr [esi], sizeof STARTUPINFO
  invoke GetStartupInfo, esi
  invoke CreateProcess, offset szTest, 0, 0, 0, 0, 0, 0, 0, esi, offset pInfo
  invoke GetCurrentProcess
  invoke AdjustPrivilege, eax, 1
  .if eax
     invoke MessageBox, 0, offset szOk, offset szTitle, MB_OK
  .else
     invoke MessageBox, 0, offset szErr,offset szTitle, MB_OK
  .endif

  invoke GetStartupInfo, esi
  invoke CreateProcess, offset szTest, 0, 0, 0, 0, 0, 0, 0, esi, offset pInfo
  invoke GetCurrentProcess
  invoke AdjustPrivilege, eax, 0
  .if eax
     invoke MessageBox, 0, offset szOk, offset szTitle, MB_OK
  .else
     invoke MessageBox, 0, offset szErr,offset szTitle, MB_OK
  .endif

  invoke GetStartupInfo, esi
  invoke CreateProcess, offset szTest, 0, 0, 0, 0, 0, 0, 0, esi, offset pInfo
  popad
  invoke ExitProcess, 0
end start
=======================================================================================================
```

这个例子里示范了怎样改变本身进程的 `SeDebugPrivilege`，并三次创建子进程：

1. 普通情况下 `CreateProcess(test.exe)`
2. `Enable SeDebugPrivilege` 的情况下 `CreateProcess(test.exe)`
3. `Disable SeDebugPrivilege` 的情况下 `CreateProcess(test.exe)`

注意比较结果，我们可以明白 `OD` 是怎么做的。

（提升权限大概是为了调试系统进程吧？）

用 `OD` 调试一下自己，我们很容易找到下面的地方。

其中 `00435421` 中的 `2` 就是 `SE_PRIVILEGE_ENABLED`，把它改成 `0`，我们这次的改造就完成了。

用改造好的 `OD` 调试了一下 `ASP`、`ARM`、`ExeCrypter` 等，都没有问题。

其实我们还可以写一个工具，不修改 `OD`，仅改变 `OD` 的子进程的权限。

```asm
0043537C  /$  53              PUSH EBX
0043537D  |.  56              PUSH ESI
0043537E  |.  57              PUSH EDI
0043537F  |.  83C4 E4         ADD ESP,-1C
00435382  |.  68 C66E4B00     PUSH EXPLORER.004B6EC6                 ; /ProcNameOrOrdinal = "OpenProcessToken"
00435387  |.  A1 085A4D00     MOV EAX,DWORD PTR DS:[4D5A08]          ; |
0043538C  |.  50              PUSH EAX                                 ; |hModule => 7C2D0000 (ADVAPI32)
0043538D  |.  E8 FE9C0700     CALL <JMP.&KERNEL32.GetProcAddress>     ; \GetProcAddress
00435392  |.  8BD8            MOV EBX,EAX
00435394  |.  68 D76E4B00     PUSH EXPLORER.004B6ED7                 ; /ProcNameOrOrdinal = "LookupPrivilegeValueA"
00435399  |.  A1 085A4D00     MOV EAX,DWORD PTR DS:[4D5A08]          ; |
0043539E  |.  50              PUSH EAX                                 ; |hModule => 7C2D0000 (ADVAPI32)
0043539F  |.  E8 EC9C0700     CALL <JMP.&KERNEL32.GetProcAddress>     ; \GetProcAddress
004353A4  |.  8BF0            MOV ESI,EAX
004353A6  |.  68 ED6E4B00     PUSH EXPLORER.004B6EED                 ; /ProcNameOrOrdinal = "AdjustTokenPrivileges"
004353AB  |.  A1 085A4D00     MOV EAX,DWORD PTR DS:[4D5A08]          ; |
004353B0  |.  50              PUSH EAX                                 ; |hModule => 7C2D0000 (ADVAPI32)
004353B1  |.  E8 DA9C0700     CALL <JMP.&KERNEL32.GetProcAddress>     ; \GetProcAddress
004353B6  |.  8BF8            MOV EDI,EAX
004353B8  |.  85DB            TEST EBX,EBX
004353BA  |.  74 08           JE SHORT EXPLORER.004353C4
004353BC  |.  85F6            TEST ESI,ESI
004353BE  |.  74 04           JE SHORT EXPLORER.004353C4
004353C0  |.  85FF            TEST EDI,EDI
004353C2  |.  75 05           JNZ SHORT EXPLORER.004353C9
004353C4  |>  83C8 FF         OR EAX,FFFFFFFF
004353C7  |.  EB 7F           JMP SHORT EXPLORER.00435448
004353C9  |>  6A 08           PUSH 8                                   ; /Arg3 = 00000008
004353CB  |.  6A 00           PUSH 0                                   ; |Arg2 = 00000000
004353CD  |.  8D5424 08       LEA EDX,DWORD PTR SS:[ESP+8]             ; |
004353D1  |.  52              PUSH EDX                                 ; |Arg1
004353D2  |.  E8 C9E10600     CALL EXPLORER.004A35A0                  ; \EXPLORER.004A35A0
004353D7  |.  83C4 0C         ADD ESP,0C
004353DA  |.  E8 459C0700     CALL <JMP.&KERNEL32.GetCurrentProcess>   ; [GetCurrentProcess
004353DF  |.  8D4C24 08       LEA ECX,DWORD PTR SS:[ESP+8]
004353E3  |.  51              PUSH ECX
004353E4  |.  6A 28           PUSH 28
004353E6  |.  50              PUSH EAX
004353E7  |.  FFD3            CALL EBX
004353E9  |.  85C0            TEST EAX,EAX
004353EB  |.  75 05           JNZ SHORT EXPLORER.004353F2
004353ED  |.  83C8 FF         OR EAX,FFFFFFFF
004353F0  |.  EB 56           JMP SHORT EXPLORER.00435448
004353F2  |>  54              PUSH ESP
004353F3  |.  68 036F4B00     PUSH EXPLORER.004B6F03                  ;  ASCII "SeDebugPrivilege"
004353F8  |.  6A 00           PUSH 0
004353FA  |.  FFD6            CALL ESI
004353FC  |.  85C0            TEST EAX,EAX
004353FE  |.  75 05           JNZ SHORT EXPLORER.00435405
00435400  |.  83C8 FF         OR EAX,FFFFFFFF
00435403  |.  EB 43           JMP SHORT EXPLORER.00435448
```



```asm
00435405  |> C74424 0C 010>MOV DWORD PTR SS:[ESP+C],1  

0043540D  |. 6A 08            PUSH 8                                     ; /Arg3 = 00000008  

0043540F  |. 8D5424 04       LEA EDX,DWORD PTR SS:[ESP+4]             ; |  

00435413  |. 52             PUSH EDX                                   ; |Arg2  

00435414  |. 8D4C24 18      LEA ECX,DWORD PTR SS:[ESP+18]            ; |  

00435418  |. 51             PUSH ECX                                   ; |Arg1  

00435419  |. E8 12E10600   CALL EXPLORER.004A3530                     ; \EXPLORER.004A3530  

0043541E  |. 83C4 0C        ADD ESP,0C  

00435421  |. C74424 18 020>MOV DWORD PTR SS:[ESP+18],2               ; 这个 2 就是 SE\_PRIVILEGE\_ENABLED, 改成 0 ********************************  

00435429  |. 6A 00          PUSH 0  

0043542B  |. 6A 00          PUSH 0  

0043542D  |. 6A 00          PUSH 0  

0043542F  |. 8D4424 18      LEA EAX,DWORD PTR SS:[ESP+18]  

00435433  |. 50             PUSH EAX  

00435434  |. 6A 00          PUSH 0  

00435436  |. 8B5424 1C      MOV EDX,DWORD PTR SS:[ESP+1C]  

0043543A  |. 52             PUSH EDX  

0043543B  |. FFD7           CALL EDI  

0043543D  |. 85C0           TEST EAX,EAX  

0043543F  |. 75 05          JNZ SHORT EXPLORER.00435446  

00435441  |. 83C8 FF        OR EAX,FFFFFFFF  

00435444  |. EB 02          JMP SHORT EXPLORER.00435448  

00435446  |> 33C0           XOR EAX,EAX  

00435448  |> 83C4 1C        ADD ESP,1C  

0043544B  |. 5F             POP EDI  

0043544C  |. 5E             POP ESI  

0043544D  |. 5B             POP EBX  

0043544E  \. C3             RETN  
```

加上这个，似乎也还是不够![](img/8f62d873668cbf759316096ebfcf749d.jpg)  

`ExeCryptor v2.2.6` 主程序是直接用 `GetTokenPrivilege` 检查当前进程的权限。  

`simonzh` 兄跟了这个主程序吗？我前段时间跟了一部分（我指那些线程）。  

主程序的线程函数只有 5 个，我跟了 3 个。地址是:  

`51A515` - 检测内存代码完整性,`anti-patch`  
`54329D` - 检测 `ExeCryptor.exe` 文件完整性  
`54AFCF` - 这个复杂,有检查 `SeDebugPrivilege`,`FindWindow` `GetTickCount`,`EnumWindows`  
有一点不明白，在以前的版本也见过，代码 `LoadLibrary("uxtheme.dll")` 然后立即 `FreeLibrary` 了，不明白是什么意思。  
这个线程最后注册了 1 个窗口类后退出。好象还是这个线程，再次启动时估计会创建窗口，我还没有跟，应该与 `anti` 有关。  

还有 2 个 `thread` 没有跟,是 `570F6D` 和 `563EA3`。  

这个东东我放下有一段时间了，盼望 `simonzh` 出手![](img/83bdd63c29485f7768335d7ba60e67a5.jpg)

`od` 用 `SE\_PRIVILEGE\_ENABLED` 用启用 `SE\_DEBUG\_NAME`,最好不要禁用。  

为什么要启用 `SE\_DEBUG\_NAME`？  

调试某些进程需要此权限。某些调试ＡＰＩ需要此权限。  

有没有更好的办法呢？有！  

`win32` 里面其实没有父进程的概念，只有从哪个进程继承数据的概念。  

同样这个继承了 `SE\_DEBUG\_NAME`  

所以，只要改变，让子进程从 `explorer` 继承就行了。  

我有打算写这么一个插件，  

只是近期没有时间,可能要过些日子才行。  

到时候反父进程方法和楼主这篇文章就算废了，因为没此必要了，呵呵。  

等不及想自己写的，如果有什么问题可以问我。 

> 引用:
>
> * * *
>
> *最初由 `goldenegg` 发布*  
>
> `od` 用 `SE\_PRIVILEGE\_ENABLED` 用启用 `SE\_DEBUG\_NAME`,最好不要禁用。  
> 为什么要启用 `SE\_DEBUG\_NAME`？  
> 调试某些进程需要此权限。某些调试ＡＰＩ需要此权限。  
>
> 有没有更好的办法呢？有！  
>
> ........ 

权限我是这样搞的:`Hook` 掉 `OD` 自己的 `WaitForDebugEvent`,在收到被调试进程的第 1 个 `int3` 后降低其权限。  

修改被调试进程的父进程,我用过这个办法:`Hook` `ntoskrnl!NtCreateProcessEx`,创建被调试进程后,修改 `EPROCESS.InheritedFromUniqueProcessId` 为 `explorer` 的 `pid`,但这样做 `xikuge` 的那个 `reverseme` 不能运行,不知为什么。  

`<加密技术内幕>`里的 `antiloader` 倒是可以通过。  

最后放弃了修改父进程的打算，换个方式:`Hook` `ntoskrnl!NtOpenProcess`,如果要打开的是 `OD`,就让它去打开 `explorer`

`4dd466d0e1e0cc123afbe504da2462ca`
