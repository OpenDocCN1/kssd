
## 1. `StackWalk64` 原型

（[详情可参考MSDN](<https://bbs.kanxue.com/elink@0f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3c8T1k6$3S2W2L8s2m8Q4x3V1k6F1k6W2)9J5k6r3c8T1k6$3S2W2L8s2m8Q4x3X3c8K6N6r3q4U0K9%4N6S2L8r3D9$3y4l9..)>)）：

```c
BOOL IMAGEAPI StackWalk64(
    DWORD                            MachineType,
    HANDLE                           hProcess,
    HANDLE                           hThread,
    LPSTACKFRAME64                   StackFrame,
    PCONTEXT                         ContextRecord,
    PREAD_PROCESS_MEMORY_ROUTINE64   ReadMemoryRoutine,
    PFUNCTION_TABLE_ACCESS_ROUTINE64 FunctionTableAccessRoutine,
    PGET_MODULE_BASE_ROUTINE64       GetModuleBaseRoutine,
    PTRANSLATE_ADDRESS_ROUTINE64     TranslateAddress
);
```

该函数有两个重要参数，`StackFrame` 和 `ContextRecord`，其中 `ContextRecord` 代表当前待分析的环境，重要的参数为 `ContextRecord.eip` 和 `ContextRecord.ebp`。

## 2. `StackWalk64` 的使用

```cpp
#define GET_CURRENT_THREAD_CONTEXT(c, contextFlags) \
do \
{ \
    c.ContextFlags = contextFlags; \
    __asm    call x \
    __asm x: pop eax \
    __asm    mov c.Eip, eax \
    __asm    mov c.Ebp, ebp \
    __asm    mov c.Esp, esp \
} while (0)

CONTEXT context;
memset(&context, 0, sizeof(context));
context.ContextFlags = CONTEXT_CONTROL;
GET_CURRENT_THREAD_CONTEXT(context, CONTEXT_CONTROL);

for (size_t i = 0; i < 1024; ++i) {
    auto ret = StackWalk64(imageType,
        GetCurrentProcess(),
        GetCurrentThread(),
        &frame64,
        &context,
        NULL,
        SymFunctionTableAccess64,
        SymGetModuleBase64,
        NULL);

    std::cout << "Round: " << i << "\n";
#if defined(_M_IX86)
    std::cout << "context ebp: " << std::hex << context.Ebp << "\n";
    std::cout << "context eip: " << std::hex << context.Eip << "\n";
#elif defined(_M_AMD64)
    std::cout << "context rbp: " << std::hex << context.Rbp << "\n";
    std::cout << "context rip: " << std::hex << context.Rip << "\n";
#endif
    std::cout << "Frame ebp: " << std::hex << frame64.AddrFrame.Offset << "\n";
    std::cout << "Frame eip: " << std::hex << frame64.AddrReturn.Offset << "\n\n\n";

    if (ret == FALSE || frame64.AddrReturn.Offset == 0) {
        break;
    }
}
```

可知，一般 `StackWalk64` 都是循环使用的，直到 `frame64.AddrReturn.Offset` 为0，代表回溯完毕。

每循环一次，`context` 的值会有对应的更新。

## 3. 观察 `StackWalk64` 返回的 `CONTEXT`（上下文）

### 当前的栈（由 windbg 打印）

```text
...
0b 00daefa4 00a4e4c5 dbghelp!StackWalk64+0x89
0c 00daf640 00a69ef3 cpp_test_ano+0xfe4c5     //函数A
0d 00daf714 00aab363 cpp_test_ano+0x119ef3    //函数B
0e 00daf734 00aab1b7 cpp_test_ano+0x15b363    //函数C
0f 00daf790 00aab04d cpp_test_ano+0x15b1b7    //函数D
10 00daf798 00aab3e8 cpp_test_ano+0x15b04d
11 00daf7a0 76336359 cpp_test_ano+0x15b3e8
...
```

### 程序输出结果

```text
// 为执行StackWalk64的初始值如下
// context ebp: daf640
// context eip: a4e34d
// Frame ebp: daf640
// Frame eip: a4e34d

Round: 0
context ebp: daf640
context eip: a4e34d
Frame ebp: daf640
Frame eip: a69ef3

Round: 1
context ebp: daf714
context eip: a69ef3
Frame ebp: daf714
Frame eip: aab363

Round: 2
context ebp: daf734
context eip: aab363
Frame ebp: daf734
Frame eip: aab1b7

Round: 3
context ebp: daf790
context eip: aab1b7
Frame ebp: daf790
Frame eip: aab04d

Round: 4
context ebp: daf798
context eip: aab04d
Frame ebp: daf798
Frame eip: aab3e8
...
```

### 结论

根据以下数据，有以下结论：

#### 第一轮结束后（Round0）

- `frame64.ebp` 和 `context.ebp` 并没有更新，还是 `0xdaf640`
- `context.eip` 并没有更新，还是 `0xa4e34d`
- `frame64.eip` 更新了，为 `0xa69ef3`，该值是第一层栈的地址，即调用函数A的地址，该地址位于函数B中

> 注：`StackWalk64` 回溯栈时，是根据 `context.ebp` 和 `context.eip` 来分析的。为了分析第二层栈，`context.ebp` 和 `context.eip` 应该对应函数B才对，这样下次调用 `StackWalk64` 才能获取函数C是在哪调用函数B的，即 `frame64.eip`。目前 `context.ebp` 和 `context.eip` 对应的是函数A。

#### 第二轮结束后（Round1）

- `frame64.ebp` 和 `context.ebp` 都更新了，是 `0xdaf714`
- `context.eip` 更新了，是 `0xa4e34d`
- `frame64.eip` 更新了，是 `0xaab363`，该值是第二层栈的地址，即调用函数B的地址，该地址位于函数C中

> 注：`context` 的 `ebp` 和 `eip` 对应的是函数B，并不是函数C。与下次要获取的函数D差两层

观察之后的层数，其结果与Round1一样。**`context` 的 `ebp` 和 `eip` 对应的函数与下一次要回溯的函数差两层**。

## 总结

- 本来由来：在栈回溯时，发现release版本的栈回溯会跳过一层函数，这层函数是 `jmp XXXXXXXX`。为了明确何时会跳过函数，所以做了一系列分析，于是有了这篇文章。

- winXP和win10在栈回溯的差异不大，大致流程基本一样（函数的封装有所变化，比如win10的 `DbsStackUnwinder::DoDbhUnwind` 函数在winXP是 `DbsStackUnwinder::DbhUnwind`）。因此win7、win8各位也可以类推，找到其栈回溯的大致流程。

- 在栈回溯时，有符号与无符号的执行流程是有差别的。有符号的情况下，`UnwindInternalContextUsingDiaFrame`（win10下）会读取符号，解析符号，直接返回。如果要分析有符号的情况，各位重点查看这个函数即可（winXP下是 `DbsX86StackUnwinder::ApplyUnwindInfo`）。

- 关于 `ComputeScoreForReturnAddress` 的算法，这里简化了参数验证等无关代码，流程上也做了简化。同时这里省略了一些不重要的细节，比如检验是否是热更新代码；检验 `call [addr A]` 的情况下，`call` 的地址和 `addr A` 是否属于同一个模块。类似这些都属于常规检测，在栈回溯时情况基本相同，可暂时忽略。

- 64位栈回溯在无符号的情况下，会根据Exception Directory数据节的函数信息进行回溯。在分析winXP时，发现 `dbghelp.dll` 会缓存一份Exception Directory数据节进行分析，所以对分析模块的Exception Directory数据节下硬件断点，可能端不下来，找不到 `dbghelp.dll` 回溯的代码。虽然64位的分析流程和32位完全不同，但主流程是一样的，都会调用 `*Unwind` 函数，然后做一些基础检测，分析 `call`、`jmp` 这些基础操作。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-8-12 10:25 被 coneco 编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
