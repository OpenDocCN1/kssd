
# SEH 简单笔记

> 原文：[`www.pediy.com/kssd/pediy12/144533.html`](https://www.pediy.com/kssd/pediy12/144533.html)

回头看了下，大致做了下笔记，正好出了一个新手版块，就发这里了，看的时候尚需抱怀疑态度

## 结构化异常处理 SEH

```text
TIB 线程信息块                   TEB 线程环境块                  PEB 进程环境块

+00h[   ExceptionList    ]←      +00h[         NtTib          ]{TIB}  │ +00h[ InheritedAddressSpace ]
+04h[     StackBase      ]  │    +1Ch[   EnvironmentPointer   ]       │ +01h[ReadImageFileExecOpions]
+08h[     StackLimit     ]  │    +20h[        ClienId         ]       │ +02h[     BeingDebugged     ]
+0ch[    SubSystemTib    ]  │          [           ...          ]     │     [           ...         ]
+10h[    u.FiberData     ]  │          [           ...          ]     │     [           ...         ]
    [    u.Version      ]  │    +30h[ProcessEnvironmentBlock**]────  +68h[     NtGlobalFlag      ]
+14h[ArbitraryUserPointer]  │          [           ...          ]      +70h[CriticalSectionTimeOut ]
+18h[        Self        ]─     +fb5h[      BooleanSpare      ]      +78h[  HeapSegmentReserve   ]

EXCEPTION_REGISTRATION →EXCEPTION_REGISTRATION →...         →EXCEPTION_REGISTRATION

FS:[0]→[  prev  ]────       [  prev  ]────       ──────       [FFFFFFFh]

        [handler1]                    [handler2]                    [handler3]

                        ─→   EXCEPTION_RECORD
                        │    [    ExceptionCode     ] //switch
                        │    [    ExceptionFlags    ]
                        │    [  EXCEPTION_RECORD*   ]
                        │    [   ExceptionAddress   ]
                        │    [   NumberParameters   ]

EXCEPTION_POINTERS       │    [ExceptionInformation[]] //[0]为 0 则读,为 1 则写   [1]触发地址
[pEXCEPTION_RECORD]──
[    pCONTEXT     ]────→       CONTEXT
                            +00h[   ContextFlags    ]CONTEXT_i386
                            +04h[       Dr0        ]①          CONTEXT_i486
                            +08h[       Dr1        ]│          CONTEXT_DEBUG_REGISTERS①
                            +0Ch[       Dr2        ]│          CONTEXT_FLOATING_POINT②
                            +10h[       Dr3        ]│          CONTEXT_SEGMENTS③
                            +14h[       Dr6        ]│          CONTEXT_INTEGER④
                            +18h[       Dr7        ]①          CONTEXT_CONTROL⑤
                        +1C~+88h[    FloatSave      ]②          CONTEXT_EXTENDED_REGISTERS⑥
                            +8Ch[     SegGs        ]③          CONTEXT_FULL③~⑤
                            +90h[     SegFs        ]│          CONTEXT_ALL①~⑥
                            +94h[     SegEs        ]│
                            +98h[     SegDs        ]③
                            +9Ch[      Edi         ]④
                            +A0h[      Esi         ]│
                            +A4h[      Ebx         ]│
                            +A8h[      Edx         ]│
                            +Ach[      Ecx         ]│
                            +B0h[      Eax         ]④
                            +B4h[      Ebp         ]⑤
                            +B8h[      Eip         ]│
                            +BCh[     SegCs        ]│
                            +C0h[     EFlags       ]│
                            +C4h[      Esp         ]│
                            +C8h[     SegSs        ]⑤
                            +CCh[ExtendedRegisters[]]⑥

1 个线程对应 1 个 TEB 结构;FS:[0]指向当前线程的 TEB 结构(即 TIB.ExceptionList)

程序构造 ERR(常用原 FS[0]的 prev)并使 FS:[0]指向;FS:[4]即 handler(常异常前现于栈)尽早设断

异常后步入系统(F7 或 Shift+F7),栈顶为 EXCEPTION_POINTERS,故 D [esp+4]+0b8 为 CONTEXT.Eip

㈠若被调试则系统填写 EXCEPTION_DEBUG_INFO(其 dwFirstChance=1)并通知调试器

㈡若未处理则经过线程 ERR 或 ERR 链;回调 ERR.handler 内常修改 CONTEXT.Eip 且清零 CONTEXT.DrX;返回后将进系统再至新 eip

㈢若被调试则系统填写 EXCEPTION_DEBUG_INFO(其 dwFirstChance=0)并通知调试器

㈣若未处理且程序 SetUnhandledExceptionFilter 且未被调试则转至进程 filter

㈤若未处理则系统弹框让选直接终止或附加调试器

㈥直接终止或附加失败或未处理都将再过线程 ERR 链并终止
```

## 调试器

```text
                     DEBUG_EVENT
               [ dwDebugEventCode  ]-
               [    dwProcessId    ] ↓
               [    dwThreadId     ]

EXCEPTION_DEBUG_INFO [u.Exception        ]//执行首指令前(EXCEPTION_DEBUG_EVENT): switch(.ExceptionRecord.ExceptionCode)
CREATE_THREAD_DEBUG_INFO [u.CreateThread     ]//创建支线程或被捆绑(CREATE_THREAD_DEBUG_EVENT)时用此
CREATE_PROCESS_DEBUG_INFO [u.CreateProcessInfo]//进程刚被创建或捆绑(CREATE_PROCESS_DEBUG_EVENT)时用此
   EXIT_THREAD_DEBUG_INFO [u.ExitThread       ]//支线程退出(EXIT_THREAD_DEBUG_EVENT)时用此
  EXIT_PROCESS_DEBUG_INFO [u.ExitProcess      ]//进程退出(EXIT_PROCESS_DEBUG_EVENT)时用此
     LOAD_DLL_DEBUG_INFO [u.LoadDll          ]//加载动态链接库(LOAD_DLL_DEBUG_EVENT)时用此
    UNLOAD_DLL_DEBUG_INFO [u.UnloadDll        ]//卸载动态链接库(UNLOAD_DLL_DEBUG_EVENT)时用此
 OUTPUT_DEBUG_STRING_INFO [u.DebugString      ]//向调试器发消息字符串(OUTPUT_DEBUG_STRING_EVENT)时用此
             RIP_INFO [u.RipInfo          ]//系统调试出错(RIP_EVENT)时用此

EXCEPTION_DEBUG_INFO       CREATE_PROCESS_DEBUG_INFO            LOAD_DLL_DEBUG_INFO      UNLOAD_DLL_DEBUG_INFO
 [ExceptionRecord]          [        hFile        ]o         [        hFile        ]o      [lpBaseOfDll]
 [ dwFirstChance ]          [      hProcess       ]         [     lpBaseOfDll     ]
                          [       hThread       ]         [dwDebugInfoFileOffset]
                          [    lpBaseOfImage    ]         [   nDebugInfoSize    ]
                          [dwDebugInfoFileOffset]         [     lpImageName     ]x
                          [   nDebugInfoSize    ]         [      fUnicode      ]
                          [  lpThreadLocalBase  ]
                          [   lpStartAddress    ]
                          [     lpImageName     ]x
                          [      fUnicode      ]

CREATE_THREAD_DEBUG_INFO     EXIT_THREAD_DEBUG_INFO     OUTPUT_DEBUG_STRING_INFO    EXIT_PROCESS_DEBUG_INFO
  [      hThread     ]              [dwExitCode]               [lpDebugStringData ]           [dwExitCode]
  [lpThreadLocalBase]                              [      fUnicode     ]
  [ lpStartAddress  ]                              [nDebugStringLength]
```

补充一下，`ERR` 就是 `EXCEPTION_REGISTRATION_RECORD`，上面那个链的结点

还有就是调试器 `ExceptionRecord` 和系统压的那个结构里的可以不区分，因为寄存器硬件是同一套

既然是笔记，大可记事本伺候
