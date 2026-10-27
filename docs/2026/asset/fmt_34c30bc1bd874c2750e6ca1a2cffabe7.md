
### 汇编跟踪

```
7C5C1F83    55              PUSH EBP
7C5C1F84    8D6B 10         LEA EBP,DWORD PTR DS:[EBX+10]
7C5C1F87    FF548F 04       CALL DWORD PTR DS:[EDI+ECX*4+4]    ; F7 进入 7C598940
7C598940    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]
7C598943    8B08            MOV ECX,DWORD PTR DS:[EAX]
7C598945    8B09            MOV ECX,DWORD PTR DS:[ECX]
7C598947    894D E4         MOV DWORD PTR SS:[EBP-1C],ECX
7C59894A    50              PUSH EAX
7C59894B    E8 8C330000     CALL kernel32.UnhandledExceptionFilter     ; 我们来到了 README 所说的第二个函数, F7 进去看看.
7C598950    C3              RETN
```

```
7C59BCDC >  55              PUSH EBP                                   ; kernel32.UnhandledExceptionFilter
7C59BCDD    8BEC            MOV EBP,ESP
7C59BCDF    6A FF           PUSH -1
7C59BCE1    68 C02E577C     PUSH kernel32.7C572EC0
7C59BCE6    68 441F5C7C     PUSH kernel32.7C5C1F44
7C59BCEB    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
7C59BCF1    50              PUSH EAX
7C59BCF2    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
7C59BCF9    51              PUSH ECX
7C59BCFA    51              PUSH ECX
7C59BCFB    81EC D8020000   SUB ESP,2D8
7C59BD01    53              PUSH EBX
7C59BD02    56              PUSH ESI
7C59BD03    57              PUSH EDI
7C59BD04    8965 E8         MOV DWORD PTR SS:[EBP-18],ESP
7C59BD07    8B75 08         MOV ESI,DWORD PTR SS:[EBP+8]
7C59BD0A    8B06            MOV EAX,DWORD PTR DS:[ESI]
7C59BD0C    8138 050000C0   CMP DWORD PTR DS:[EAX],C0000005
7C59BD12    75 1B           JNZ SHORT kernel32.7C59BD2F
7C59BD14    33DB            XOR EBX,EBX
7C59BD16    3958 14         CMP DWORD PTR DS:[EAX+14],EBX
7C59BD19    74 16           JE SHORT kernel32.7C59BD31
7C59BD1B    FF70 18         PUSH DWORD PTR DS:[EAX+18]
7C59BD1E    E8 E8FEFFFF     CALL kernel32.7C59BC0B
7C59BD23    83F8 FF         CMP EAX,-1
7C59BD26    75 09           JNZ SHORT kernel32.7C59BD31
7C59BD28    0BC0            OR EAX,EAX
7C59BD2A    E9 8E020000     JMP kernel32.7C59BFBD
7C59BD2F    33DB            XOR EBX,EBX                                        ; EBX = 0 ;
7C59BD31    895D C8         MOV DWORD PTR SS:[EBP-38],EBX                      ; ProecssInfo = 0;
7C59BD34    53              PUSH EBX                                           ; NULL
7C59BD35    6A 04           PUSH 4                                             ; sizeof(ProcessInfo)
7C59BD37    8D45 C8         LEA EAX,DWORD PTR SS:[EBP-38]
7C59BD3A    50              PUSH EAX                                           ; &ProcessInfo
7C59BD3B    6A 07           PUSH 7                                             ; ProcessDebugPort
7C59BD3D    E8 7BBBFFFF     CALL kernel32.GetCurrentProcess                    ; 返回 hProcess
7C59BD42    50              PUSH EAX
7C59BD43    FF15 B810577C   CALL DWORD PTR DS:[<&ntdll.NtQueryInformationProce>; ntdll.ZwQueryInformationProcess
7C59BD49    3BC3            CMP EAX,EBX                                        ; EAX = 0 表示成功, EAX = -1 表示不成功
7C59BD4B    7C 09           JL SHORT kernel32.7C59BD56
7C59BD4D    395D C8         CMP DWORD PTR SS:[EBP-38],EBX                      ; [EBP-38]= -1 有调试器, 0 没有
7C59BD50    0F85 49020000   JNZ kernel32.7C59BF9F                              ; 所以我们只要把 [EBP-38] 清零
                                                                               ; 就可以瞒过系统, 使其以为目前未在调试状态
                                                                               ; 这就是插件的作用吧.
```

上面 `ZwQueryInformationProcess` 某个进程是否正被 ring3 调试器所调试。

Blowfish 班主写过一篇文章, 可以参考.

```c
enum PROCESS_INFO_CLASS      { ProcessDebugPort = 7  };
typedef struct _PROCESS_DEBUG_PORT_INFO
{
    HANDLE DebugPort;
}    PROCESS_DEBUG_PORT_INFO;
PROCESS_DEBUG_PORT_INFO ProcessInfo;
ZwQueryInformationProcess(GetCurrentProcess(), ProcessDebugPort, &ProcessInfo, sizeof(ProcessInfo), NULL)
```

我们把 `[EBP-38]` 清零, 到 `401148` (`Final_Handler`) 下断, `F9`, 看看发生了什么, 我们在 OD 里来到了 `Final_Handler`.

没有插件, 我们也可以对付最终异常处理例程了.

### (五) 我的结论

#### 一. 程序没有被调试

0. 线程建立时, 系统安装一个系统线程异常处理例程
1. 异常发生时, `Win2K/XP` 控制权转移给 `NTDLL.KiUserExceptionDispatcher`
2. 如果你安装了线程相关的异常处理例程, 系统就把异常发送给你的处理例程, 交由其处理.
3. 每个线程相关的异常处理例程可以处理或者不处理这个异常, 如果他不处理并且安装了多个线程相关的异常处理例程, 可交由链起来的其他例程处理.
4. 如果这些例程均选择不处理异常, 系统线程异常处理例程将起作用, 调用 `ZwQueryInformationProcess` 判断是否被调试, 没有调试并且你调用 `SetUnhandledExceptionFilter` 安装了最后异常处理例程的话, 系统转向对它的调用.
5. 如果你没有安装最后异常处理例程或者他没有处理这个异常, 系统会调用默认的系统处理程序, 通常显示一个对话框, 你可以选择关闭或者最后将其附加到调试器上的调试按钮. 如果没有调试器能被附加于其上或者调试器也处理不了, 系统就调用 `ExitProcess` 终结程序.
6. 不过在终结之前, 系统仍然对发生异常的线程异常处理句柄来一次展开, 这是线程异常处理例程最后清理的机会. 一般只用来释放资源, 不要试图修复什么. 注意只对线程异常有展开.

#### 二. 程序被调试

0. 线程建立时, 系统安装一个系统线程异常处理例程
1. 异常发生时, 系统挂起程序并向调试器发送 `EXCEPTION_DEBUG_EVENT` 消息.
2. 调试器未能处理异常(比如我们在 OD 里按 `Shift+F7`), 控制权转移给 `NTDLL.KiUserExceptionDispatcher` 如果你安装了线程相关的异常处理例程, 系统就把异常发送给你的程序 seh 处理例程, 交由其处理.
3. 每个线程相关的异常处理例程可以处理或者不处理这个异常, 如果他不处理并且安装了多个线程相关的异常处理例程, 可交由链起来的其他例程处理.
4. 如果这些例程均选择不处理异常, 系统线程异常处理例程将起作用, 调用 `ZwQueryInformationProcess` 判断是否被调试,
5. 由于被调试, 操作系统仍会再次挂起程序通知 debugger.

其实这里, 我也有很多似懂非懂的地方, 各位大侠请多指点.
