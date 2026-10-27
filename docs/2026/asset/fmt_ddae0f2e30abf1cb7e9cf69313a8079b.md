
接下来会调用函数从创建的进程的内存中读出数据

```
.text:7C81FE2E                 push    ebx             ; NumberOfBytesRead

.text:7C81FE2F                 push    4               ; NumberOfBytesToRead

.text:7C81FE31                 lea     eax, [ebp+var_798]

.text:7C81FE37                 push    eax             ; Buffer

.text:7C81FE38                 mov     eax, [ebp+var_848]

.text:7C81FE3E                 add     eax, 10h

.text:7C81FE41                 push    eax             ; BaseAddress

.text:7C81FE42                 push    [ebp+ProcessHandle] ; ProcessHandle

.text:7C81FE48                 call    ds:__imp__NtReadVirtualMemory@20 ; NtReadVirtualMemory(x,x,x,x,x)

.text:7C81FE4E                 mov     [ebp+Status], eax

.text:7C81FE54                 cmp     eax, ebx

.text:7C81FE56                 jl      loc_7C819DEC
```

接下来会分别对输入输出以及错误句柄进行判断，如果不是控制台句柄就会通过`StuffStdHandle`进行复制

```
.text:7C81FE5C                 mov     eax, large fs:18h

.text:7C81FE62                 mov     [ebp+var_teb0], eax

.text:7C81FE68                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C81FE6B                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C81FE6E                 mov     eax, [eax+RTL_USER_PROCESS_PARAMETERS.StandardInput] ; 对输入句柄判断

.text:7C81FE71                 mov     esi, 10000003h

.text:7C81FE76                 and     eax, esi

.text:7C81FE78                 cmp     eax, edi

.text:7C81FE7A                 jnz     loc_7C82DF7C

            。。。

.text:7C82DF7C loc_7C82DF7C: 

.text:7C82DF7C                 mov     eax, large fs:18h

.text:7C82DF82                 mov     [ebp+var_teb5], eax

.text:7C82DF88                 mov     ecx, [ebp+var_798]

.text:7C82DF8E                 add     ecx, 18h

.text:7C82DF91                 push    ecx             ; BaseAddress

.text:7C82DF92                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C82DF95                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C82DF98                 push    [eax+RTL_USER_PROCESS_PARAMETERS.StandardInput] ; 对输入句柄复制

.text:7C82DF9B                 push    [ebp+ProcessHandle] ; TargetProcessHandle

.text:7C82DFA1                 call    _StuffStdHandle@12 ; StuffStdHandle(x,x,x)

.text:7C82DFA6                 jmp     loc_7C81FE80
```

```
.text:7C81FE80 loc_7C81FE80: 

.text:7C81FE80                 mov     eax, large fs:18h

.text:7C81FE86                 mov     [ebp+var_teb1], eax

.text:7C81FE8C                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C81FE8F                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C81FE92                 mov     eax, [eax+RTL_USER_PROCESS_PARAMETERS.StandardOutput] ; 输出句柄的判断

.text:7C81FE95                 and     eax, esi

.text:7C81FE97                 cmp     eax, edi

.text:7C81FE99                 jnz     loc_7C82DF29

            。。。

.text:7C82DF29 loc_7C82DF29:  

.text:7C82DF29                 mov     eax, large fs:18h

.text:7C82DF2F                 mov     [ebp+var_teb4], eax

.text:7C82DF35                 mov     ecx, [ebp+var_798]

.text:7C82DF3B                 add     ecx, 1Ch

.text:7C82DF3E                 push    ecx             ; BaseAddress

.text:7C82DF3F                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C82DF42                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C82DF45                 push    [eax+RTL_USER_PROCESS_PARAMETERS.StandardOutput] ; 对输出句柄复制

.text:7C82DF48                 push    [ebp+ProcessHandle] ; TargetProcessHandle

.text:7C82DF4E                 call    _StuffStdHandle@12 ; StuffStdHandle(x,x,x)

.text:7C82DF53                 jmp     loc_7C81FE9F
```

```
.text:7C81FE9F loc_7C81FE9F:  

.text:7C81FE9F                 mov     eax, large fs:18h

.text:7C81FEA5                 mov     [ebp+var_teb2], eax

.text:7C81FEAB                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C81FEAE                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C81FEB1                 mov     eax, [eax+RTL_USER_PROCESS_PARAMETERS.StandardError] ; 对错误句柄进行判断

.text:7C81FEB4                 and     eax, esi

.text:7C81FEB6                 cmp     eax, edi

.text:7C81FEB8                 jz      loc_7C819DEC

.text:7C81FEBE                 mov     eax, large fs:18h

.text:7C81FEC4                 mov     [ebp+var_teb3], eax

.text:7C81FECA                 mov     ecx, [ebp+var_798]

.text:7C81FED0                 add     ecx, 20h

.text:7C81FED3                 push    ecx             ; BaseAddress

.text:7C81FED4                 mov     eax, [eax+_TEB.ProcessEnvironmentBlock]

.text:7C81FED7                 mov     eax, [eax+_PEB.ProcessParameters]

.text:7C81FEDA                 push    [eax+RTL_USER_PROCESS_PARAMETERS.StandardError] ; 对错误句柄进行复制

.text:7C81FEDD                 push    [ebp+ProcessHandle] ; TargetProcessHandle

.text:7C81FEE3                 call    _StuffStdHandle@12 ; StuffStdHandle(x,x,x)

.text:7C81FEE8                 jmp     loc_7C819DEC
```

现在进程执行体已经创建完成，但是想要执行指令，还需要线程才可以完成。因此，要继续创建再进程中运行的线程。

## 4. 创建线程和栈及上下文

至此，Windows执行体进程对象已经创建完成，然而其中依然不具备线程，因此还无法开始工作。接下来的工作就需要创建线程，而创建线程之前需要初始化创建线程需要的资源。

首先需要调用`BaseCreateStack`来创建线程栈

```
.text:7C819DEC loc_7C819DEC:                           ; CODE XREF: CreateProcessInternalW(x,x,x,x,x,x,x,x,x,x,x,x)-4B0↑j

.text:7C819DEC                                         ; CreateProcessInternalW(x,x,x,x,x,x,x,x,x,x,x,x)+665C↓j ...

.text:7C819DEC                 mov     eax, 40000h     ; 是否小于0x4000

.text:7C819DF1                 cmp     [ebp+ImageInformation.MaximumStackSize], eax

.text:7C819DF7                 jb      short loc_7C819DFF

.text:7C819DF9                 mov     eax, [ebp+ImageInformation.MaximumStackSize]

.text:7C819DFF

.text:7C819DFF loc_7C819DFF:                           ; CODE XREF: CreateProcessInternalW(x,x,x,x,x,x,x,x,x,x,x,x)+647↑j

.text:7C819DFF                 lea     ecx, [ebp+InitialTeb]

.text:7C819E05                 push    ecx             ; int

.text:7C819E06                 push    eax             ; RegionSize

.text:7C819E07                 push    [ebp+ImageInformation.CommittedStackSize] ; UINT_PTR

.text:7C819E0D                 push    [ebp+ProcessHandle] ; ProcessHandle

.text:7C819E13                 call    _BaseCreateStack@16 ; BaseCreateStack(x,x,x,x)

.text:7C819E18                 mov     [ebp+var_CreateStackStatus], eax

.text:7C819E1E                 cmp     eax, ebx

.text:7C819E20                 jl      loc_7C82DF58
```

接着需要调用`BaseInitializeContext`来初始化线程的`CONTEXT`

```
.text:7C819E26                 push    ebx

.text:7C819E27                 push    [ebp+InitialTeb.StackBase]

.text:7C819E2D                 push    [ebp+SectionInformation]

.text:7C819E33                 push    [ebp+var_PEB]

.text:7C819E39                 lea     eax, [ebp+ThreadContext]

.text:7C819E3F                 push    eax

.text:7C819E40                 call    _BaseInitializeContext@20 ; BaseInitializeContext(x,x,x,x,x)
```

最后通过系统调用`NtCreateThread`来创建内核线程对象，但是此时是以挂起的方式创建的线程
