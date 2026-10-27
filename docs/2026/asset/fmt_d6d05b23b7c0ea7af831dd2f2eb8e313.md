
# 调用`CsrClientCallServer`来给Windows子系统(`Csrss`)发送消息

```nasm
.text:7C81060B mov eax, [ebp+ThreadHandle]
.text:7C810611 mov [ebp+var_ThreadHandle], eax
.text:7C810617 mov eax, [ebp+ClientId.UniqueProcess]
.text:7C81061D mov [ebp+var_UniqueProcess], eax
.text:7C810623 mov eax, [ebp+ClientId.UniqueThread]
.text:7C810629 mov [ebp+var_UniqueThread], eax
.text:7C81062F push 0Ch ; DataLength
.text:7C810631 push 10001h ; ApiNumber
.text:7C810636 push ebx ; CaptureBuffer
.text:7C810637 lea eax, [ebp+ApiMessage]
.text:7C81063D push eax ; ApiMessage
.text:7C81063E call ds:__imp__CsrClientCallServer@16
```

判断是否带有`CREATE_SUSPENDED`标志，如果没有，则会调用`NtResumeThread`来恢复线程的执行

```nasm
.text:7C81066E test byte ptr [ebp+dwCreationFlags], 4 ; 是否带有CREATESUSPENDED
.text:7C810672 jnz short loc_7C810687
.text:7C810674 lea eax, [ebp+SuspendCount]
.text:7C81067A push eax ; SuspendCount
.text:7C81067B push [ebp+ThreadHandle] ; ThreadHandle
.text:7C810681 call ds:__imp__NtResumeThread@8
```

## 2.系统调用`NtCreateThread`

在`NtCreateThread`中会首先判断是否是内核模式传递的调用

```nasm
PAGE:004ADD59 ; NTSTATUS __stdcall NtCreateThread(PHANDLE ThreadHandle, ACCESS_MASK DesiredAccess, POBJECT_ATTRIBUTES ObjectAttributes, HANDLE ProcessHandle, PCLIENT_ID ClientId, PCONTEXT ThreadContext, PINITIAL_TEB InitialTeb, BOOLEAN CreateSuspended)
PAGE:004ADD59 _NtCreateThread@32 proc near ; DATA XREF: .text:0040D8F4↑o
PAGE:004ADD59
PAGE:004ADD59 var_InitialTeb = dword ptr -38h
PAGE:004ADD59 var_34 = dword ptr -34h
PAGE:004ADD59 var_ClientId = dword ptr -24h
PAGE:004ADD59 var_CurETHREAD = dword ptr -20h
PAGE:004ADD59 var_1C = dword ptr -1Ch
PAGE:004ADD59 ms_exc = CPPEH_RECORD ptr -18h
PAGE:004ADD59 ThreadHandle = dword ptr 8
PAGE:004ADD59 DesiredAccess = dword ptr 0Ch
PAGE:004ADD59 ObjectAttributes= dword ptr 10h
PAGE:004ADD59 ProcessHandle = dword ptr 14h
PAGE:004ADD59 ClientId = dword ptr 18h
PAGE:004ADD59 ThreadContext = dword ptr 1Ch
PAGE:004ADD59 InitialTeb = dword ptr 20h
PAGE:004ADD59 CreateSuspended = byte ptr 24h
PAGE:004ADD59 push 28h
PAGE:004ADD5B push offset stru_423B30
PAGE:004ADD60 call __SEH_prolog
PAGE:004ADD65 and [ebp+ms_exc.registration.TryLevel], 0
PAGE:004ADD69 mov eax, large fs:124h
PAGE:004ADD6F mov [ebp+var_CurETHREAD], eax
PAGE:004ADD72 cmp byte ptr [eax+140h], 0 ; 判断PreviousMode是否为0
PAGE:004ADD79 jz loc_4E902E
```

如果是内核模式，则直接参数`InitialTeb`赋值给`ebx`

```nasm
PAGE:004E902E loc_4E902E: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+20↑j
PAGE:004E902E mov ebx, [ebp+InitialTeb]
PAGE:004E9031 jmp loc_4ADDF3
```

如果不是内核模式，就会对输出参数`ThreadHandle`, `ClientId`和输入参数`ThreadContext`以及`InitialTeb`进行可写检查

```nasm
PAGE:004ADD7F mov eax, _MmUserProbeAddress
PAGE:004ADD84 mov ecx, [ebp+ThreadHandle]
PAGE:004ADD87 cmp ecx, eax
PAGE:004ADD89 jnb loc_52E0D8
PAGE:004ADD8F
PAGE:004ADD8F loc_4ADD8F: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+80385↓j
PAGE:004ADD8F mov eax, [ecx]
PAGE:004ADD91 mov [ecx], eax
PAGE:004ADD93 mov ebx, [ebp+ClientId]
PAGE:004ADD96 test ebx, ebx
PAGE:004ADD98 jz short loc_4ADDBD
PAGE:004ADD9A mov [ebp+var_ClientId], ebx
PAGE:004ADD9D mov eax, _MmUserProbeAddress
PAGE:004ADDA2 cmp ebx, eax
PAGE:004ADDA4 jnb loc_52E0E3
PAGE:004ADDAA
PAGE:004ADDAA loc_4ADDAA: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+8038D↓j
PAGE:004ADDAA test bl, 3
PAGE:004ADDAD jnz loc_52E0EB
PAGE:004ADDB3
PAGE:004ADDB3 loc_4ADDB3: ; CODE XREF: PAGE:0052E0F0↓j
PAGE:004ADDB3 mov al, [ebx]
PAGE:004ADDB5 mov [ebx], al
PAGE:004ADDB7 mov al, [ebx+4]
PAGE:004ADDBA mov [ebx+4], al
PAGE:004ADDBD
PAGE:004ADDBD loc_4ADDBD: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+3F↑j
PAGE:004ADDBD cmp [ebp+ThreadContext], 0
PAGE:004ADDC1 jz loc_52E129
PAGE:004ADDC7 test byte ptr [ebp+ThreadContext], 3
PAGE:004ADDCB jnz loc_52E0F5
PAGE:004ADDD1
PAGE:004ADDD1 loc_4ADDD1: ; CODE XREF: PAGE:0052E0FA↓j
PAGE:004ADDD1 mov eax, _MmUserProbeAddress
PAGE:004ADDD6 cmp [ebp+ThreadContext], eax
PAGE:004ADDD9 jnb loc_52E0FF
PAGE:004ADDDF
PAGE:004ADDDF loc_4ADDDF: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+803B1↓j
PAGE:004ADDDF mov ebx, [ebp+InitialTeb] ; 将InitialTeb赋给ebx
PAGE:004ADDE2 test bl, 3
PAGE:004ADDE5 jnz loc_52E10F
PAGE:004ADDEB
PAGE:004ADDEB loc_4ADDEB: ; CODE XREF: PAGE:0052E119↓j
PAGE:004ADDEB cmp ebx, eax
PAGE:004ADDED jnb loc_52E11E
```

把传入的参数`InitialTeb`赋值给局部变量

```nasm
PAGE:004ADDF3 loc_4ADDF3: ; CODE XREF: NtCreateThread(x,x,x,x,x,x,x,x)+3B2D8↓j
PAGE:004ADDF3 ; NtCreateThread(x,x,x,x,x,x,x,x)+803CB↓j
PAGE:004ADDF3 mov eax, [ebx]
PAGE:004ADDF5 mov [ebp+var_InitialTeb], eax
PAGE:004ADDF8 mov ecx, [ebx+4]
PAGE:004ADDFB mov [ebp+var_InitialTebPlus4], ecx
```

调用`PspCreateThread`来完成创建，再次之前`edx`被清0，所以最后两个参数都为`NULL`

```nasm
PAGE:004ADDFE xor edx, edx
....
PAGE:004ADE16 push edx ; StartContext
PAGE:004ADE17 push edx ; StartRoutine
PAGE:004ADE18 push dword ptr [ebp+CreateSuspended] ; CreateSuspended
PAGE:004ADE1B lea eax, [ebp+var_InitialTeb]
PAGE:004ADE1E push eax ; InitialTeb
PAGE:004ADE1F push [ebp+ThreadContext] ; ThraedContext
PAGE:004ADE22 push [ebp+ClientId] ; ClientId
PAGE:004ADE25 push edx ; ProcessPointer
PAGE:004ADE26 push [ebp+ProcessHandle] ; ProcessHandle
PAGE:004ADE29 push [ebp+ObjectAttributes] ; ObjectAttributes
PAGE:004ADE2C push [ebp+DesiredAccess] ; AccessMask
PAGE:004ADE2F push [ebp+ThreadHandle] ; ThreadHandle
PAGE:004ADE32 call _PspCreateThread@44ateThread@44
```

线程是属于进程的，所以参数中必须得有个进程句柄。参数`ClientId`和`ThradHandle`是用来返回`CID`和句柄的。参数`ThreadContext`用来给定一个线程的原始上下文，实际上就是各个寄存器的初值。另一个参数`CreateSuspended`则说明新创的线程是否要被挂起，等到对其调用`NtResumeThread`时才开始运行。

相比于`NtCreateThread`，`PspCreateThread`多了三个参数，这三个参数是为创建内核线程而设的，内核线程都属于系统初始进程`PsInitialSystemProcess`，但是这个进程没有句柄，所以只好使用`EPROCESS`结构指针`ProcessPointer`。同样，`StartRoutine`和`StartContext`也是用来给定内核线程的程序入口和上下文。非内核线程的程序入口是有`ThreadContext`寄存器中的`EIP`来决定的，所以对于来自用户空间的系统调用这三个参数都是`NULL`。

在`PspCreateThraed`中首先会判断是否存在`StartRoutine`，如果存在就会对局部变量`var_PreviousMode`赋值为0
