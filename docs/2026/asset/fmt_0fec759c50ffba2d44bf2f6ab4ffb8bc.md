
```text
.text:7C94AA07 loc_7C94AA07:                           ; CODE XREF: RtlDispatchException(x,x)+239BF↓j
.text:7C94AA07                 mov     eax, edi
.text:7C94AA09                 xor     ecx, ecx        ; ecx清0
.text:7C94AA0B                 sub     eax, ecx
.text:7C94AA0D                 jnz     loc_7C94A2F5
.text:7C94AA13                 test    byte ptr [esi+_EXCEPTION_RECORD.ExceptionFlags], EXCEPTION_NONCONTINUABLE
.text:7C94AA17                 jnz     loc_7C96E351
.text:7C94AA1D                 mov     [ebp+var_res], 1
```

如果`ExceptionFlags`不满足条件，会调用`RtlRaiseException`分发异常

```text
.text:7C96E351 loc_7C96E351:                           ; CODE XREF: RtlDispatchException(x,x)+C7↑j
.text:7C96E351                 lea     eax, [ebp+var_ExecptRec]
.text:7C96E354                 push    eax
.text:7C96E355                 mov     [ebp+var_ExecptRec.ExceptionCode], EXCEPTION_NONCONTINUABLE_EXCEPTION
.text:7C96E35C                 mov     [ebp+var_ExecptRec.ExceptionFlags], 1
.text:7C96E363                 mov     [ebp+var_ExecptRec.ExceptionRecord], esi
.text:7C96E366                 mov     [ebp+var_ExecptRec.NumberParameters], ecx
.text:7C96E369                 call    _RtlRaiseException@4
```

如果返回值为1，且`ExceptionFlags`符合条件，就会跳转到上面的`loc_7C94AA21`处退出函数

```text
.text:7C94A2F5 loc_7C94A2F5:                           ; CODE XREF: RtlDispatchException(x,x)+BD↓j
.text:7C94A2F5                 dec     eax
.text:7C94A2F6                 jnz     loc_7C96E314
.text:7C94A2FC
.text:7C94A2FC loc_7C94A2FC:                           ; CODE XREF: .text:7C96E36E↓j
.text:7C94A2FC                 test    byte ptr [esi+_EXCEPTION_RECORD.ExceptionFlags], 8
.text:7C94A300                 jnz     loc_7C94AA21
```

否则就取出下一异常处理器，判断`Next`是否为-1，不为-1就会跳转到`loc_7C94A994`继续上面操作，为-1则退出函数

```text
.text:7C94A306 loc_7C94A306:                           ; CODE XREF: .text:7C96E334↓j
                                                       ; RtlDispatchException(x,x)+239F3↓j ...
.text:7C94A306                 mov     ebx, [ebx+_EXCEPTION_REGISTRATION_RECORD.Next]
.text:7C94A308                 cmp     ebx, 0FFFFFFFFh
.text:7C94A30B                 jnz     loc_7C94A994
.text:7C94A311                 jmp     loc_7C94AA21
```

如果返回值不为0, 1, 2就会调用`RtlRaiseException`来分发异常

```text
.text:7C96E314 loc_7C96E314:                           ; CODE XREF: RtlDispatchException(x,x)-65A↑j
.text:7C96E314                 dec     eax
.text:7C96E315                 jz      short loc_7C96E339
.text:7C96E317                 lea     eax, [ebp+var_ExecptRec]
.text:7C96E31A                 push    eax
.text:7C96E31B                 mov     [ebp+var_ExecptRec.ExceptionCode], EXCEPTION_INVALID_DISPOSITION
.text:7C96E322                 mov     [ebp+var_ExecptRec.ExceptionFlags], 1
.text:7C96E329                 mov     [ebp+var_ExecptRec.ExceptionRecord], esi
.text:7C96E32C                 mov     [ebp+var_ExecptRec.NumberParameters], ecx
.text:7C96E32F                 call    _RtlRaiseException@4
```

如果为2，对`pNestedRegistration`继续判断，如果地址小于`pExecptRec`，来决定是否要为`pExecptRec`赋值，随后跳转到上面的`loc_7C94A306`来取下一个异常处理器后继续执行上述循环过程

```text
.text:7C96E339 loc_7C96E339:                           ; CODE XREF: RtlDispatchException(x,x)+239C5↑j
.text:7C96E339                 mov     eax, [ebp+var_pNestedRegistration]
.text:7C96E33C                 or      [esi+_EXCEPTION_RECORD.ExceptionFlags], 10h
.text:7C96E340                 cmp     eax, [ebp+pExecptRec]
.text:7C96E343                 jbe     loc_7C94A306    ; 低于等于则跳转
.text:7C96E349                 mov     [ebp+pExecptRec], eax
.text:7C96E34C                 jmp     loc_7C94A306
```

下图是对上述过程的展现:

![avatar](https://bbs.kanxue.com/upload/attach/202112/835440_3R2D9PXP5QDFBQQ.png)

对于异常处理函数`RtlpExecuteHanlderForException`，首先将函数地址`SehHandler`赋给`edx`，随后跳转到`ExecuteHandler`

```text
.text:7C923247 ; __stdcall RtlpExecuteHandlerForException(x, x, x, x, x)
.text:7C923247 _RtlpExecuteHandlerForException@20 proc near
.text:7C923247                                         ; CODE XREF: RtlDispatchException(x,x)+9A↓p
.text:7C923247                 mov     edx, offset SehHandler
.text:7C92324C                 jmp     short ExecuteHandler@20 ; ExecuteHandler(x,x,x,x,x)
.text:7C92324C _RtlpExecuteHandlerForException@20 endp
```

而`ExecuteHandler`则是通过`ExecuteHandler2`实现的

```text
.text:7C923256 ; int __stdcall ExecuteHandler(PVOID pExceptionRecord, PVOID pExcepttionRegistration, PCONTEXT pContext, int *pDispatcherContext, PVOID pfnHandler)
.text:7C923256 ExecuteHandler@20 proc near             ; CODE XREF: RtlpExecuteHandlerForException(x,x,x,x,x)+5↑j
.text:7C923256
.text:7C923256 pExceptionRecord = dword ptr 4
.text:7C923256 pExcepttionRegistration = dword ptr 8
.text:7C923256 pContext = dword ptr 0Ch
.text:7C923256 pDispatcherContext = dword ptr 10h
.text:7C923256 pfnHandler = dword ptr 14h
.text:7C923256
.text:7C923256                 push    ebx
.text:7C923257                 push    esi
.text:7C923258                 push    edi
.text:7C923259                 xor     eax, eax
.text:7C92325B                 xor     ebx, ebx
.text:7C92325D                 xor     esi, esi
.text:7C92325F                 xor     edi, edi
.text:7C923261                 push    [esp+pfnHandler] ; pfnHandler
.text:7C923265                 push    [esp+pDispatcherContext] ; pDispatcherContext
.text:7C923269                 push    [esp+pContext] ; pContext
.text:7C92326D                 push    [esp+pExcepttionRegistration] ; pExcepttionRegistration
.text:7C923271                 push    [esp+pExceptionRecord] ; pExceptionRecord
.text:7C923275                 call    ExecuteHandler2@20 ; ExecuteHandler2(x,x,x,x,x)
.text:7C92327A                 pop     edi
.text:7C92327B                 pop     esi
.text:7C92327C                 pop     ebx
.text:7C92327D                 retn    14h
.text:7C92327D ExecuteHandler@20 endp
```

`ExecuteHandler2`则是会将`SehHandler`函数登记到`SEH`异常链条中，随后在调用传入的函数，此时的`edx`在前面被赋值为`SehHandler`函数地址
