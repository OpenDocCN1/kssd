

```
[EAX+3C],EBX  〈== 修改PE头，[EAX+3C] = 40003C
0187:00A8EF60 PUSH DWORD [EBP-04]
0187:00A8EF63 PUSH    BYTE +40
……
……
……
0187:00A8EF89 AND      EAX,BYTE +03
0187:00A8EF8C LEA      ECX,[EBP-08]
0187:00A8EF8F INC EAX
0187:00A8EF90 ADD      [EDI+06],AX    〈==修改PE头，[EDI+06] = 4000FE
0187:00A8EF94 CALL    00A88597
0187:00A8EF99 MOV EBX,EAX
```

共修改`3C`和`FE`两处，参考有关PE头的资料，可知偏移`3C`的`DWORD`是PE表头的偏移（原值是`F8`）。偏移`FE`的`WORD`是sections的个数（原值是8）。Armadillo故意将PE头改错，难怪没法dump出。

### （三）dump出加壳的程序

好了，原理分析清楚了。要dump出加壳程序就很容易了。

事先用`peditor`查看，得知`size of image`是`104000`；

首先找到OEP是`41EF80`，另外修改`004C428F`，将`ADD EDX,BYTE +01`改为`ADD EDX,BYTE +00`。

在`41EF80`下断点。中断后，将PE头改回，`40003C`处是`000000F8`，`4000FE`处是`0008`，注意高位在前！！！

在程序中找一个没用到的地址。不要选代码段，因为解码后会覆盖掉我们写的代码。我选`.Data1`段的最后256个byte即`4CF000`处，将`eip`改为`4CF000`，输入以下代码：

```
push esi
push ecx
push eax
mov esi,401000
mov ecx,103000
//size of image 减 1000h
rep lodsb
pop eax
pop ecx
pop esi
int3
```

注意我没有使用hying的代码，因为要申请内存和查找api地址，而且内存不够的话也麻烦，其实只要扫描一遍内存就足以触发调试器了，修改`004C428F`的原因就是防止外壳把解码过的页面从新加密。

`F5`执行，在`int3`处停下（注意打开 `i3here on`），将`eip`改回`41EF80`，可看到已经解码了，挂起进程，回到window桌面，这时用什么工具都可以dump出了！（任我鱼肉了！！）。

另外，还有一个地方要注意，最好在OEP处挂起进程再dump，因为这时程序的初始化变量还没有被修改。

大家可对比一下，当程序运行后，用`procdump` dump出的文件和在OEP处dump出的有何不同。（会非法操作！！因为初始化变量`[438110]`已经不同了。）

### （四）修复IAT

没什么方便的方法!:(

我的方法是用`peditor`查看，得知IAT在`426000`处，下断点 `bpm 426000 w` 在这停下：

```
0187:00A8E4BB 8A06            MOV  AL,[ESI]
0187:00A8E4BD 3AC3            CMP  AL,BL
0187:00A8E4BF 7468            JZ   00A8E529
0187:00A8E4C1 3CFF            CMP  AL,FF
0187:00A8E4C3 7537            JNZ  00A8E4FC
0187:00A8E4C5 668B7E01        MOV  DI,[ESI+01]
0187:00A8E4C9 46              INC  ESI
0187:00A8E4CA 0FB7C7          MOVZX EAX,DI
0187:00A8E4CD 50              PUSH EAX
0187:00A8E4CE 46              INC  ESI
0187:00A8E4CF FF75F4          PUSH DWORD [EBP-0C]
0187:00A8E4D2 46              INC  ESI
0187:00A8E4D3 E84075FFFF      CALL 00A85A18
```

然后把`[ESI]`前后的内容保存下来，以后手工修复IAT时参考。

### （五）小结

无论用什么方法脱壳，原理和hying的是一样的。

但为什么用`procdump`不会触发调试器呢？我的解释是这样：无论`procdump`或`WinHex`在读取其他进程的内容时，是通过`WriteProcessMemory`来完成的，而`WriteProcessMemory`最终调用VMM完成读取，VMM是在ring0上，所以不会触发ring3上的调试器。我能想到的方法是：在ring3下、在自身进程空间内扫描整个内存，以触发调试器。

### 附录：

附上部分`DEBUG_EVENT`结构的声明：（注释是我加的）

```
typedef struct _DEBUG_EVENT
{
    DWORD dwDebugEventCode;  //偏移0x0
    DWORD dwProcessId;       //偏移0x4
    DWORD dwThreadId;        //偏移0x8
    union {
        EXCEPTION_DEBUG_INFO Exception;             //偏移0xC
        CREATE_THREAD_DEBUG_INFO CreateThread;
        CREATE_PROCESS_DEBUG_INFO CreateProcessInfo;
        EXIT_THREAD_DEBUG_INFO ExitThread;
        EXIT_PROCESS_DEBUG_INFO ExitProcess;
        LOAD_DLL_DEBUG_INFO LoadDll;
        UNLOAD_DLL_DEBUG_INFO UnloadDll;
        OUTPUT_DEBUG_STRING_INFO DebugString;
        RIP_INFO RipInfo;
    } u;
} DEBUG_EVENT, *LPDEBUG_EVENT;

typedef struct _EXCEPTION_DEBUG_INFO
{ //即异常代码是0x00000001时的结构
    EXCEPTION_RECORD ExceptionRecord;  //偏移0xC;0x80000001为STATUS_GUARD_PAGE_VIOLATION(NT)
    //0xC0000005为STATUS_ACCESS_VIOLATION(WIN (win9x)
    DWORD dwFirstChance;
} EXCEPTION_DEBUG_INFO, *LPEXCEPTION_DEBUG_INFO;

typedef struct _EXCEPTION_RECORD {
    DWORD ExceptionCode;            //偏移0xc
    DWORD ExceptionFlags;           //偏移0x10
    struct _EXCEPTION_RECORD *ExceptionRecord;  //偏移0x14
    PVOID ExceptionAddress;         //偏移0x18
    DWORD NumberParameters;
    ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD, *PEXCEPTION_RECORD;
```

