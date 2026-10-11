
# 汇编分析与逆向工程日志

## 1. 内存释放与异常处理
首先，通过 `call 00CD9770` 进行调用，推测可能是计算密钥（key）的逻辑。随后执行内存栈清理及指针比较：

```asm
add     esp, 1C
cmp     edi, -1
jnz     short 0051804A

mov     edx, dword ptr [ebp+C]
mov     eax, dword ptr [edx]
push    eax
call    <FREE>
add     esp, 4

lea     ecx, dword ptr [ebp-4C]
push    0
push    016CE14C              ; ASCII "ArcDHKeyExchange::GenerateSharedKey: error computing key."
call    00420A40

lea     ecx, dword ptr [ebp-4C]
push    015C5980
push    ecx
mov     dword ptr [ebp-4C], 01541880
call    <jmp.&MSVCRT._CxxThrowException>

mov     ecx, dword ptr [ebp-18]
mov     dword ptr [ebp-4], -1
test    ecx, ecx
je      short 0051805D
call    004149C5

mov     ecx, dword ptr [ebp-C]
pop     edi
pop     esi
mov     dword ptr fs:[0], ecx
```

逐一跟踪 `call` 指令（按 `F7`）是必要的体力活，关键在于耐心与毅力。

## 2. 定位 Encode 函数入口
经过深入分析，找到了关键的函数入口点，该函数负责编码（encode）操作：

```asm
push    ebp                  ; encode 函数
mov     ebp, esp

mov     eax, dword ptr [ebp+10]    ; 形参 4 [ebp+10] 待 encode 长度
test    eax, eax
jbe     short 005115DF

lea     edx, dword ptr [ecx+1068]
push    1                  ; 可能是参数 1 代表 encode, 0 代表 decode
push    edx                ; ecx+1068 压栈

lea     edx, dword ptr [ecx+1058]
add     ecx, 0C
push    edx                ; ecx+1058 压栈
push    ecx                ; ecx+0C 压栈

mov     ecx, dword ptr [ebp+8]     ; 形参 2 [ebp+8] 待 encode buff
push    eax                  ; encode 字符串长度

mov     eax, dword ptr [ebp+C]     ; 形参 3 [ebp+C] encode 后结果
push    eax                  ; encode 后结果指针
push    ecx                  ; encode 字符串指针

call    <encode/decode 函数>
add     esp, 1C              ; 未知参数来源于外层 ecx
pop     ebp
retn    0C
```

通过这一操作，未编码前的缓冲区（buff）得以暴露。

## 3. 利用 DEBUG API 拦截日志
为了实现报文捕获，需利用调试 API（DEBUG API），在调用 `encode/decode` 函数前设置断点。当程序执行到该函数时，保存待发送数据及其长度，写入日志，并恢复原始指令指针（`eip`）。

示例代码逻辑如下：

```c
if(DbgEv.u.Exception.ExceptionRecord.ExceptionAddress == (PVOID)encode_base)
{
    dwThreadPid = DbgEv.dwThreadId;
    DebugHandle = OpenThread(THREAD_ALL_ACCESS, FALSE, dwThreadPid);
    
    thContext.ContextFlags = CONTEXT_FULL;
    GetThreadContext(DebugHandle, &thContext);
    
    // 修改指令指针以跳过或处理后续逻辑
    thContext.Eip--;
    
    // 从栈中读取参数
    memaddr = thContext.Ebp + 0x10;     // [ebp+c] result buff
    // ebp+10 是 origin len
    // [ebp+8] 是 origin buff
    
    ReadProcessMemory(DbgHandle, (LPVOID)(memaddr), &addr, 4, NULL);
    len = (int)addr;
    
    memaddr = thContext.Ebp + 0x08;     // [ebp+8] origin len (注意：这里逻辑可能需要根据实际偏移调整，原文如此)
    ReadProcessMemory(DbgHandle, (LPVOID)(memaddr), &addr, 4, NULL);
    
    memset(Buf, 0x00, sizeof(Buf));
    memset(Hex, 0x00, sizeof(Hex));
    
    ReadProcessMemory(DbgHandle, (LPVOID)(addr), Buf, len, NULL);
    BinToHexFormat(Buf, Hex, len);
    
    sprintf(loghead, "\nSrv->Clt: %d ", len);
    logger->Add(loghead);
    logger->Add(Hex);
    
    // 恢复原有指令
    WriteProcessMemory(DbgHandle, (LPVOID)decode_base, decode_temp1, 1, &dwWritten); 
    WriteProcessMemory(DbgHandle, (LPVOID)decode_after, int3, 1, &dwWritten);       
    WriteProcessMemory(DbgHandle, (LPVOID)encode_base, int3, 1, &dwWritten);       
    WriteProcessMemory(DbgHandle, (LPVOID)encode_after, int3, 1, &dwWritten);      
}
```

*注：由于调试步骤限制，`SINGLE_STEP` 模式不稳定，故采用双断点互写 `int3` 的方法。*

## 4. 报文协议分析
经过多次游戏操作及日志分析，确定了客户端（clt）选择怪物（NPC）时的报文格式。前8字节为固定头部，后续字节包含唯一标识符。

观察到的报文示例：

```text
0x64 0xe1 0x09 0x38 0x0c 0x80 0x00 0x00 == 0x00 0xee 0x93 0xe4
0x64 0xe1 0x09 0x38 0x0c 0x80 0x00 0x00 == 0x00 0xac 0x55 0xd2
0x64 0xe1 0x09 0x38 0x0c 0x80 0x00 0x00 == 0x00 0xbb 0xa7 0xe4
0x64 0xe1 0x09 0x38 0x0c 0x80 0x00 0x00 == 0x00 0xe1 0x91 0x78
0x64 0xe1 0x09 0x38 0x0c 0x80 0x00 0x00 == 0x00 0x32 0x40 0xa3
```

**字段解析：**
*   `0x64 0xe1 0x09 0x38`：操作码（opcode），代表选择 NPC。
*   `0x0c 0x80 0x00 0x00`：类型标识，怪物、NPC 或玩家的值各不相同。
*   `0x00 0xee 0x93 0xe4`...：ID，应为游戏世界中的唯一标识号。

*修正说明：之前关于 `clt<-->srv` 40 字节报文的猜测有误，实际为客户端版本号信息上传。*

## 5. 实现远程选怪功能
为了验证协议，需编写一个外置函数来实现自动选怪。基本思路如下：

1.  过滤未加密报文，对于以 `0x64 0xe1 0x09 0x38` 开头的报文，保存其后的全局 ID（global id）备用。
2.  编写注入函数，参数为一个指向 global id 的指针。
3.  在目标进程分配空间，将函数代码写入目标进程内存。
4.  在目标进程分配空间，将 global id 写入目标进程内存。
5.  使用 `CreateRemoteThread` 在目标进程中创建线程，执行选怪报文发送逻辑。

## 6. 结语
成功实现了目标。尽管最终的注入代码尚未完全调试完毕，但核心逆向流程已验证。

给初学者的一点鼓励：只要具备耐心、毅力及基本编程基础，即可学习逆向工程。但需注意，仅满足于学习爆破（blowout/patching）是不够的。
