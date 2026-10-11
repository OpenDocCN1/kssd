
# SetUnhandledExceptionFilter 的讨论

> 原文：[`www.pediy.com/kssd/pediy06/pediy6921.htm`](https://www.pediy.com/kssd/pediy06/pediy6921.htm)

现在有好多壳用 `SetUnhandledExceptionFilter` 安装了最后异常处理例程来愚弄 `Ollydbg`，

一开始确实难倒了我等菜鸟，幸好后来有位俄罗斯高人写了个插件，解决了这个问题，但一直想知道原因。

最近抽空把 `Hume` 大侠的 `SEH` 文章反反复复看了好几遍，又看了插件的 `README`，总算有点明白了。

把他写出来，请各位大侠看看，多多指点。

实例下载：点击此处下载

## (一) 发生异常时系统的处理顺序 (by `Jeremy Gordon`, `Hume`)

1. 系统首先判断异常是否应发送给目标程序的异常处理例程，如果决定应该发送，并且目标程序正在被调试，则系统挂起程序并向调试器发送 `EXCEPTION_DEBUG_EVENT` 消息。
2. 如果你的程序没有被调试或者调试器未能处理异常，系统就会继续查找你是否安装了线程相关的异常处理例程，如果你安装了线程相关的异常处理例程，系统就把异常发送给你的程序 `seh` 处理例程，交由其处理。
3. 每个线程相关的异常处理例程可以处理或者不处理这个异常，如果他不处理并且安装了多个线程相关的异常处理例程，可交由链起来的其他例程处理。
4. 如果这些例程均选择不处理异常，如果程序处于被调试状态，操作系统仍会再次挂起程序通知 `debugger`。
5. 如果程序未处于被调试状态或者 `debugger` 没有能够处理，并且你调用 `SetUnhandledExceptionFilter` 安装了最后异常处理例程的话，系统转向对它的调用。
6. 如果你没有安装最后异常处理例程或者他没有处理这个异常，系统会调用默认的系统处理程序，通常显示一个对话框，你可以选择关闭或者最后将其附加到调试器上的调试按钮。如果没有调试器能被附加于其上或者调试器也处理不了，系统就调用 `ExitProcess` 终结程序。
7. 不过在终结之前，系统仍然对发生异常的线程异常处理句柄来一次展开，这是线程异常处理例程最后清理的机会。

### 插件的 README

```
THEORY

When exception happens, Win2K/XP gives control to NTDLL!KiUserExceptionFilter,
which in turn calls KERNEL32!UnhandledExceptionFilter. This function checks if
faulty software is being debugged. If not, UnhandledExceptionFilter calls
softwares' exception handler.

SOLUTION:

Patch KERNEL32!UnhandledExceptionFilter so that softwares' exception handler
is always called. Used signatures work for both Win2K SP4/WinXP SP1, and
hopefully for all other versions.
```

## (二) 参考 `Hume` 的例子写的第一个 MASM 程序，没有最终异常处理例程

```asm
;//============================  SEH1.EXE  ============================ 

.386 

.model flat, stdcall 

option casemap :none  ; case sensitive 

include c:\masm32\include\windows.inc 

include c:\masm32\include\kernel32.inc 

include c:\masm32\include\user32.inc 

includelib c:\masm32\lib\kernel32.lib 

includelib c:\masm32\lib\user32.lib 

;//============================ 

.data 

szCap1     db "Thread SEH 1",0 

szMsg1     db "We are in thread SEH handler 1.  ",0 

szCap2     db "Thread SEH 2",0 

szMsg2     db "We are in thread SEH handler 2.  ",0 

szCap     db "SEH example",0 

szMsg     db "It would never get here.",0 

.code 

start: 

;//========prog begin==================== 

    ASSUME FS:NOTHING 

    push    offset pThread_Handler1 

    push    fs:[0]       

    mov     fs:[0],esp                    ;//建立 SEH 的基本 ERR 结构 

    push    offset pThread_Handler2 

    push    fs:[0]       

    mov     fs:[0],esp                    ;//建立 SEH 的基本 ERR 结构                      

    xor     ecx,ecx 

    mov     eax,200     

    cdq 

    div     ecx 

                              ;//以下永远不会被执行 

    invoke    MessageBox,NULL,addr szMsg,addr szCap,MB_OK+MB_ICONEXCLAMATION 

    invoke    ExitProcess,NULL 

;//============================ 

pThread_Handler1: 

    invoke    MessageBox,NULL,addr szMsg1,addr szCap1,MB_OK+MB_ICONINFORMATION 

    mov     eax,1                     ;//ExceptionContinueSearch,不处理,由其他例程或系统处理 

    ;mov     eax,0                    ;//ExceptionContinueExecution,表示已经修复,可从异常发生处继续执行 

    ret                               ;//这里如果返回 0,你会陷入死循环,不断跳出对话框.... 

pThread_Handler2: 

    invoke    MessageBox,NULL,addr szMsg2,addr szCap2,MB_OK+MB_ICONINFORMATION 

    mov     eax,1                     ;//ExceptionContinueSearch,不处理,由其他例程或系统处理 

    ;mov     eax,0                    ;//ExceptionContinueExecution,表示已经修复,可从异常发生处继续执行 

    ret                               ;//这里如果返回 0,你会陷入死循环,不断跳出对话框.... 

;//=============================Prog Ends============== 

end start 
```

`SEH1.EXE` 有两个线程异常处理例程，但都不处理除零异常。

运行 `SEH1.EXE`，你将会看到 5 次对话框，其中第三次是系统默认对话框，最后两次是线程异常展开。

## (三) 参考 `Hume` 的例子写的第二个 MASM 程序，有最终异常处理例程

```asm
;//===============================   SEH2.EXE  ========================= 

.386 

.model flat, stdcall 

option casemap :none  ; case sensitive 

include c:\masm32\include\windows.inc 

include c:\masm32\include\kernel32.inc 

include c:\masm32\include\user32.inc 

includelib c:\masm32\lib\kernel32.lib 

includelib c:\masm32\lib\user32.lib 

;//============================ 

.data 

szCap1     db "Thread SEH 1",0 

szMsg1     db "We are in thread SEH handler 1.  ",0 

szCap2     db "Thread SEH 2",0 

szMsg2     db "We are in thread SEH handler 2.  ",0 

szCap0     db "Final SEH",0 

szMsg0     db "We are in final SEH handler.  ",0 

szCap     db "SEH example",0 

szMsg     db "It would never get here.",0 

.code 

start: 

;//========prog begin==================== 

    ASSUME FS:NOTHING 

    push    offset pThread_Handler1 

    push    fs:[0]       

    mov     fs:[0],esp                    ;//建立 SEH 的基本 ERR 结构 

    ;lea    eax,Final_Handler 

    ;invoke SetUnhandledExceptionFilter,eax        ;//调用 SetUnhandledExceptionFilter 来安装 final SEH 

                              ;//原型很简单 SetUnhandledExceptionFilter proto 

                              ;//pTopLevelExceptionFilter:DWORD 

    push    offset pThread_Handler2 

    push    fs:[0]       

    mov     fs:[0],esp                    ;//建立 SEH 的基本 ERR 结构, 只是说明两种异常先后加载的顺序对系统没影响                      

    xor     ecx,ecx 

    mov     eax,200     

    cdq 

    div     ecx 

                              ;//以下永远不会被执行 

    invoke    MessageBox,NULL,addr szMsg,addr szCap,MB_OK+MB_ICONEXCLAMATION 

    invoke    ExitProcess,NULL 

;//============================ 

pThread_Handler1: 

    invoke    MessageBox,NULL,addr szMsg1,addr szCap1,MB_OK+MB_ICONINFORMATION 

    mov     eax,1                     ;//ExceptionContinueSearch,不处理,由其他例程或系统处理 

    ;mov     eax,0                    ;//ExceptionContinueExecution,表示已经修复,可从异常发生处继续执行 

    ret                               ;//这里如果返回 0,你会陷入死循环,不断跳出对话框.... 

pThread_Handler2: 

    invoke    MessageBox,NULL,addr szMsg2,addr szCap2,MB_OK+MB_ICONINFORMATION 

    mov     eax,1                     ;//ExceptionContinueSearch,不处理,由其他例程或系统处理 

    ;mov     eax,0                    ;//ExceptionContinueExecution,表示已经修复,可从异常发生处继续执行 

    ret                               ;//这里如果返回 0,你会陷入死循环,不断跳出对话框.... 

Final_Handler: 

    invoke    MessageBox,NULL,addr szMsg0,addr szCap0,MB_OK+MB_ICONEXCLAMATION 

    mov     eax,EXCEPTION_EXECUTE_HANDLER        ;//==1 这时不出现非法操作的讨厌对话框, 已经处理了, 可以结束了. 

    ;mov     eax,EXCEPTION_CONTINUE_SEARCH       ;//==0 出现,这时是调用系统默认的异常处理过程,程序被终结了 

    ;mov     eax,EXCEPTION_CONTINUE_EXECUTION    ;//==-1 表示已经修复,可从异常发生处继续执行, 因为我们并没有修复 ecx 

    ret                             ;//     所以不断产生异常,不断调用这个例程, 出现对话框,陷入死循环, 

;//=============================Prog Ends============== 

end start 
```

`SEH2.EXE` 有两个线程异常处理例程，一个最终异常处理例程，都不处理除零异常。

运行 `SEH2.EXE`，还是会看到 5 次对话框，但是其中第三次换成我们自己的了，最后两次还是线程异常展开。

还要注意最终异常处理例程不会展开。

## (四) 下面我们开始用 `OD` 调试 `SEH2.EXE`，注意先不要安装插件，不要忽略异常。

加载后，`OD` 停在 `004010C0`。

```asm
004010C0 >/$  68 16114000   PUSH SEH2.00401116                 ;  SE handler installation
004010C5  |.  64:FF35 00000>PUSH DWORD PTR FS:[0]
004010CC  |.  64:8925 00000>MOV DWORD PTR FS:[0],ESP
004010D3  |.  8D05 48114000 LEA EAX,DWORD PTR DS:[401148]
004010D9  |.  50            PUSH EAX                            ; /pTopLevelFilter => SEH2.00401148
004010DA  |.  E8 89000000   CALL <JMP.&kernel32.SetUnhandledExceptio>; \SetUnhandledExceptionFilter
```



### 004010DF |. 68 2F114000 PUSH SEH2.0040112F
```
004010DF  |.  68 2F114000   PUSH SEH2.0040112F          ;  SE handler installation
```

### 0040112F |. 64:FF35 00000> PUSH DWORD PTR FS:\[0\]
```
0040112F  |.  64:FF35 00000> PUSH DWORD PTR FS:\[0\]
```

### 004011EB |. 64:8925 00000> MOV DWORD PTR FS:\[0\],ESP
```
004011EB  |.  64:8925 00000> MOV DWORD PTR FS:\[0\],ESP
```

### 004010F2 |. 33C9 XOR ECX,ECX
```
004010F2  |.  33C9          XOR ECX,ECX
```

### 004010F4 |. B8 C8000000 MOV EAX,0C8
```
004010F4  |.  B8 C8000000   MOV EAX,0C8
```

### 004010F9 |. 99 CDQ
```
004010F9  |.  99            CDQ
```

### 004010FA |. F7F1 DIV ECX
```
004010FA  |.  F7F1          DIV ECX                     ; // 除零异常
```

### 004010FC |. 6A 30 PUSH 30
```
004010FC  |.  6A 30         PUSH 30                     ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
```

### 004010FE |. 68 9B104000 PUSH SEH2.0040109B
```
004010FE  |.  68 9B104000   PUSH SEH2.0040109B          ; |Title = "SEH example"
```

### 00401103 |. 68 A7104000 PUSH SEH2.004010A7
```
00401103  |.  68 A7104000   PUSH SEH2.004010A7          ; |Text = "It would never get here."
```

### 00401108 |. 6A 00 PUSH 0
```
00401108  |.  6A 00         PUSH 0                      ; |hOwner = NULL
```

### 0040110A |. E8 5F000000 CALL <JMP.&user32.MessageBoxA>
```
0040110A  |.  E8 5F000000   CALL <JMP.&user32.MessageBoxA> ; \MessageBoxA
```

### 0040110F |. 6A 00 PUSH 0
```
0040110F  |.  6A 00         PUSH 0                      ; /ExitCode = 0
```

### 00401111 \. E8 4C000000 CALL <JMP.&kernel32.ExitProcess>
```
00401111  \.  E8 4C000000   CALL <JMP.&kernel32.ExitProcess> ; \ExitProcess
```

### 00401116 /$ 6A 40 PUSH 40
```
00401116  /$  6A 40         PUSH 40                     ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL; Structured exception handler
```

### 00401118 |. 68 14104000 PUSH SEH2.00401014
```
00401118  |.  68 14104000   PUSH SEH2.00401014          ; |Title = "Thread SEH 1"
```

### 0040111D |. 68 21104000 PUSH SEH2.00401021
```
0040111D  |.  68 21104000   PUSH SEH2.00401021          ; |Text = "We are in thread SEH handler 1."
```

### 00401122 |. 6A 00 PUSH 0
```
00401122  |.  6A 00         PUSH 0                      ; |hOwner = NULL
```

### 00401124 |. E8 45000000 CALL <JMP.&user32.MessageBoxA>
```
00401124  |.  E8 45000000   CALL <JMP.&user32.MessageBoxA> ; \MessageBoxA
```

### 00401129 |. B8 01000000 MOV EAX,1
```
00401129  |.  B8 01000000   MOV EAX,1
```

### 0040112E \. C3 RETN
```
0040112E  \.  C3            RETN
```

### 0040112F /$ 6A 40 PUSH 40
```
0040112F  /$  6A 40         PUSH 40                     ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL; Structured exception handler
```

### 00401131 |. 68 43104000 PUSH SEH2.00401043
```
00401131  |.  68 43104000   PUSH SEH2.00401043          ; |Title = "Thread SEH 2"
```

### 00401136 |. 68 50104000 PUSH SEH2.00401050
```
00401136  |.  68 50104000   PUSH SEH2.00401050          ; |Text = "We are in thread SEH handler 2."
```

### 0040113B |. 6A 00 PUSH 0
```
0040113B  |.  6A 00         PUSH 0                      ; |hOwner = NULL
```

### 0040113D |. E8 2C000000 CALL <JMP.&user32.MessageBoxA>
```
0040113D  |.  E8 2C000000   CALL <JMP.&user32.MessageBoxA> ; \MessageBoxA
```

### 00401142 |. B8 01000000 MOV EAX,1
```
00401142  |.  B8 01000000   MOV EAX,1
```

### 00401147 \. C3 RETN
```
00401147  \.  C3            RETN
```

### 00401148 . 6A 30 PUSH 30
```
00401148   .  6A 30         PUSH 30                     ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
```

### 0040114A . 68 72104000 PUSH SEH2.00401072
```
0040114A   .  68 72104000   PUSH SEH2.00401072          ; |Title = "Final SEH"
```

### 0040114F . 68 7C104000 PUSH SEH2.0040107C
```
0040114F   .  68 7C104000   PUSH SEH2.0040107C          ; |Text = "We are in final SEH handler."
```

### 00401154 . 6A 00 PUSH 0
```
00401154   .  6A 00         PUSH 0                      ; |hOwner = NULL
```

### 00401156 . E8 13000000 CALL <JMP.&user32.MessageBoxA>
```
00401156   .  E8 13000000   CALL <JMP.&user32.MessageBoxA> ; \MessageBoxA
```

### 0040115B . B8 01000000 MOV EAX,1
```
0040115B   .  B8 01000000   MOV EAX,1
```

### 00401160 . C3 RETN
```
00401160   .  C3            RETN
```

不要运行，先看看`OD`的`SEH chain`窗口，如下，系统已经为我们设好了一个异常处理例程。

#### SEH chain of main thread

| Address | SE handler |
| :--- | :--- |
| `0012FFE0` | `kernel32.7C5C1F44` |

```
; 记住这里, 称其为系统线程异常处理例程
; 关键的地方.
```

`F9`运行，`OD`停在`004010FA`，除零异常。再看一下`SEH chain`窗口。

#### SEH chain of main thread

| Address | SE handler |
| :--- | :--- |
| `0012FFB4` | `SEH2.0040112F` |
| `0012FFBC` | `SEH2.00401116` |
| `0012FFE0` | `kernel32.7C5C1F44` |

我们按照`OD`的提示`SHIFT+F7`走，注意不是`SHIFT+F9`。

```
77F9FF60  >   8B4C24 04     MOV ECX,DWORD PTR SS:\[ESP+4\]   ; ntdll.KiUserExceptionDispatcher
77F9FF64    8B1C24          MOV EBX,DWORD PTR SS:\[ESP\]
77F9FF67    51              PUSH ECX
77F9FF68    53              PUSH EBX
77F9FF69    E8 6F6CFFFF     CALL ntdll.77F96BDD              ; F7
```

我们发现系统把控制权交给了`ntdll.KiUserExceptionDispatcher`，

而不是`README`里说的`KiUserExceptionFilter`，这也许是说法上的差异吧。

我们一路`F7`走

```
77F96BDD    55              PUSH EBP
77F96BDE    8BEC            MOV EBP,ESP
77F96BE0    83EC 60         SUB ESP,60
77F96BE3    53              PUSH EBX
77F96BE4    56              PUSH ESI
77F96BE5    8D45 F4         LEA EAX,DWORD PTR SS:\[EBP-C\]
77F96BE8    57              PUSH EDI
77F96BE9    50              PUSH EAX
77F96BEA    8D45 F8         LEA EAX,DWORD PTR SS:\[EBP-8\]
77F96BED    50              PUSH EAX
77F96BEE    E8 C8FFFFFF     CALL ntdll.77F96BBB              ; F8
77F96BF3    E8 DEFFFFFF     CALL ntdll.77F96BD6              ; F8
77F96BF8    8365 FC 00      AND DWORD PTR SS:\[EBP-4\],0
77F96BFC    8BD8            MOV EBX,EAX
77F96BFE    83FB FF         CMP EBX,-1
77F96C01    74 71           JE SHORT ntdll.77F96C74
77F96C03    8B75 08         MOV ESI,DWORD PTR SS:\[EBP+8\]
77F96C06    3B5D F8         CMP EBX,DWORD PTR SS:\[EBP-8\]    ; // 循环处理线程异常链
77F96C09    8D43 08         LEA EAX,DWORD PTR DS:\[EBX+8\]
77F96C0C    0F82 40AF0100   JB ntdll.77FB1B52
77F96C12    3B45 F4         CMP EAX,DWORD PTR SS:\[EBP-C\]
77F96C15    0F87 37AF0100   JA ntdll.77FB1B52
77F96C1B    F6C3 03         TEST BL,3
77F96C1E    0F85 2EAF0100   JNZ ntdll.77FB1B52
77F96C24    F605 5604FD77 8> TEST BYTE PTR DS:\[77FD0456\],80
77F96C2B    0F85 AAAE0100   JNZ ntdll.77FB1ADB
77F96C31    FF73 04         PUSH DWORD PTR DS:\[EBX+4\]
77F96C34    8D45 F0         LEA EAX,DWORD PTR SS:\[EBP-10\]
77F96C37    50              PUSH EAX
77F96C38    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]
77F96C3B    53              PUSH EBX
77F96C3C    56              PUSH ESI
77F96C3D    E8 3AFFFFFF     CALL ntdll.77F96B7C             ; F7, 见下面
77F96C42    F605 5604FD77 8> TEST BYTE PTR DS:\[77FD0456\],80
77F96C49    8BF8            MOV EDI,EAX
77F96C4B    0F85 A0AE0100   JNZ ntdll.77FB1AF1
77F96C51    395D FC         CMP DWORD PTR SS:\[EBP-4\],EBX
77F96C54    0F84 A5AE0100   JE ntdll.77FB1AFF
77F96C5A    8BC7            MOV EAX,EDI
77F96C5C    33C9            XOR ECX,ECX
77F96C5E    2BC1            SUB EAX,ECX
77F96C60    0F84 8A030000   JE ntdll.77F96FF0
77F96C66    48              DEC EAX
77F96C67    0F85 9FAE0100   JNZ ntdll.77FB1B0C
77F96C6D    8B1B            MOV EBX,DWORD PTR DS:\[EBX\]
77F96C6F    83FB FF         CMP EBX,-1                    ; // 0FFFFFFFFh
77F96C72  ^ 75 92           JNZ SHORT ntdll.77F96C06       ; // 是不是最后一个线程异常处理例程
77F96B7C    BA AE6DF977     MOV EDX,ntdll.77F96DAE
77F96B81    55              PUSH EBP
77F96B82    8BEC            MOV EBP,ESP
77F96B84    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]
77F96B87    52              PUSH EDX
77F96B88    64:FF35 0000000> PUSH DWORD PTR FS:\[0\]
77F96B8F    64:8925 0000000> MOV DWORD PTR FS:\[0\],ESP
77F96B96    FF75 14         PUSH DWORD PTR SS:\[EBP+14\]    ; pDispatch
77F96B99    FF75 10         PUSH DWORD PTR SS:\[EBP+10\]    ; pContext
77F96B9C    FF75 0C         PUSH DWORD PTR SS:\[EBP+C\]     ; pException_Registration_Record
77F96B9F    FF75 08         PUSH DWORD PTR SS:\[EBP+8\]     ; pException_Record
77F96BA2    8B4D 18         MOV ECX,DWORD PTR SS:\[EBP+18\]
77F96BA5    FFD1            CALL ECX                      ; 线程异常处理例程, F7 跟进,
77F96BA7    64:8B25 0000000> MOV ESP,DWORD PTR FS:\[0\]
77F96BAE    64:8F05 0000000> POP DWORD PTR FS:\[0\]
77F96BB5    8BE5            MOV ESP,EBP
77F96BB7    5D              POP EBP
77F96BB8    C2 1400         RETN 14                     ; ret to 77F96C42
```

我们将在`77F96BA5`进入线程处理例程，一共三次。

前两次是我们自己的代码，都不处理除零异常，返回`EAX=1`。

最后一次，就是前面记下的系统预设的线程处理例程`7C5C1F44`，`F7`进去

```
7C5C1F44    55              PUSH EBP
7C5C1F45    8BEC            MOV EBP,ESP
7C5C1F47    83EC 08         SUB ESP,8
7C5C1F4A    53              PUSH EBX
7C5C1F4B    56              PUSH ESI
7C5C1F4C    57              PUSH EDI
7C5C1F4D    55              PUSH EBP
7C5C1F4E    FC              CLD
```



```
7C5C1F4F    8B5D 0C         MOV EBX,DWORD PTR SS:[EBP+C]        ; pException_Registration_Record
7C5C1F52    8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]        ; pException_Record
7C5C1F55    F740 04 0600000>TEST DWORD PTR DS:[EAX+4],6         ; 异常标志, =6 表示正展开中.
7C5C1F5C    75 77          JNZ SHORT kernel32.7C5C1FD5
7C5C1F5E    8945 F8        MOV DWORD PTR SS:[EBP-8],EAX
7C5C1F61    8B45 10        MOV EAX,DWORD PTR SS:[EBP+10]       ; pContext
7C5C1F64    8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
7C5C1F67    8D45 F8        LEA EAX,DWORD PTR SS:[EBP-8]
7C5C1F6A    8943 FC        MOV DWORD PTR DS:[EBX-4],EAX
7C5C1F6D    8B73 0C        MOV ESI,DWORD PTR DS:[EBX+C]
7C5C1F70    8B7B 08        MOV EDI,DWORD PTR DS:[EBX+8]
7C5C1F73    83FE FF        CMP ESI,-1
7C5C1F76    74 56          JE SHORT kernel32.7C5C1FCE
7C5C1F78    8D0C76         LEA ECX,DWORD PTR DS:[ESI+ESI*2]
7C5C1F7B    837C8F 04 00   CMP DWORD PTR DS:[EDI+ECX*4+4],0
7C5C1F80    74 3A          JE SHORT kernel32.7C5C1FBC
7C5C1F82    56             PUSH ESI
7C5C1F83    55             PUSH EBP
7C5C1F84    8D6B 10        LEA EBP,DWORD PTR DS:[EBX+10]
7C5C1F87    FF548F 04      CALL DWORD PTR DS:[EDI+ECX*4+4]    ; F7 进入 7C598940
7C598940    8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
7C598943    8B08           MOV ECX,DWORD PTR DS:[EAX]
7C598945    8B09           MOV ECX,DWORD PTR DS:[ECX]
7C598947    894D E4        MOV DWORD PTR SS:[EBP-1C],ECX
7C59894A    50             PUSH EAX
7C59894B    E8 8C330000    CALL kernel32.UnhandledExceptionFilter     ; 我们来到了 README 所说的第二个函数, F7 进去看看.
7C598950    C3             RETN
7C59BCDC >   55             PUSH EBP                                 ; kernel32.UnhandledExceptionFilter
7C59BCDD    8BEC           MOV EBP,ESP
7C59BCDF    6A FF          PUSH -1
7C59BCE1    68 C02E577C    PUSH kernel32.7C572EC0
7C59BCE6    68 441F5C7C    PUSH kernel32.7C5C1F44
7C59BCEB    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
7C59BCF1    50             PUSH EAX
7C59BCF2    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
7C59BCF9    51             PUSH ECX
7C59BCFA    51             PUSH ECX
7C59BCFB    81EC D8020000  SUB ESP,2D8
7C59BD01    53             PUSH EBX
7C59BD02    56             PUSH ESI
7C59BD03    57             PUSH EDI
7C59BD04    8965 E8        MOV DWORD PTR SS:[EBP-18],ESP
7C59BD07    8B75 08        MOV ESI,DWORD PTR SS:[EBP+8]
7C59BD0A    8B06           MOV EAX,DWORD PTR DS:[ESI]
7C59BD0C    8138 050000C0  CMP DWORD PTR DS:[EAX],C0000005
7C59BD12    75 1B          JNZ SHORT kernel32.7C59BD2F
7C59BD14    33DB           XOR EBX,EBX
7C59BD16    3958 14        CMP DWORD PTR DS:[EAX+14],EBX
7C59BD19    74 16          JE SHORT kernel32.7C59BD31
7C59BD1B    FF70 18        PUSH DWORD PTR DS:[EAX+18]
7C59BD1E    E8 E8FEFFFF    CALL kernel32.7C59BC0B
7C59BD23    83F8 FF        CMP EAX,-1
7C59BD26    75 09          JNZ SHORT kernel32.7C59BD31
7C59BD28    0BC0           OR EAX,EAX
7C59BD2A    E9 8E020000    JMP kernel32.7C59BFBD
7C59BD2F    33DB           XOR EBX,EBX                                ; EBX = 0 ;
7C59BD31    895D C8        MOV DWORD PTR SS:[EBP-38],EBX              ; ProecssInfo = 0;
7C59BD34    53             PUSH EBX                                   ; NULL
7C59BD35    6A 04          PUSH 4                                     ; sizeof(ProcessInfo)
7C59BD37    8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]
7C59BD3A    50             PUSH EAX                                   ; &ProcessInfo
7C59BD3B    6A 07          PUSH 7                                     ; ProcessDebugPort
7C59BD3D    E8 7BBBFFFF    CALL kernel32.GetCurrentProcess            ; 返回 hProcess
7C59BD42    50             PUSH EAX
7C59BD43    FF15 B810577C  CALL DWORD PTR DS:[<&ntdll.NtQueryInformationProce>; ntdll.ZwQueryInformationProcess
7C59BD49    3BC3           CMP EAX,EBX                                ; EAX = 0 表示成功, EAX = -1 表示不成功
7C59BD4B    7C 09          JL SHORT kernel32.7C59BD56
7C59BD4D    395D C8        CMP DWORD PTR SS:[EBP-38],EBX             ; [EBP-38]= -1 有调试器, 0 没有
7C59BD50    0F85 49020000  JNZ kernel32.7C59BF9F                     ; 所以我们只要把 [EBP-38] 清零
                                                                          ; 就可以瞒过系统, 使其以为目前未在调试状态
                                                                          ; 这就是插件的作用吧.
```

上面 `ZwQueryInformationProcess` 某个进程是否正被 `ring3` 调试器所调试。

`Blowfish` 班主写过一篇文章，可以参考。

```
enum PROCESS_INFO_CLASS     { ProcessDebugPort = 7  };

typedef struct _PROCESS_DEBUG_PORT_INFO

{

    HANDLE DebugPort;

}    PROCESS_DEBUG_PORT_INFO;

PROCESS_DEBUG_PORT_INFO ProcessInfo;

ZwQueryInformationProcess(GetCurrentProcess( ), ProcessDebugPort, &ProcessInfo, sizeof(ProcessInfo), NULL)
```

我们把 `[EBP-38]` 清零，到 `401148(Final_Handler)` 下断，`F9`，看看发生了什么，我们在 `OD` 里来到了 `Final_Handler`。

没有插件，我们也可以对付最终异常处理例程了。

### (五) 我的结论

#### 一. 程序没有被调试

0. 线程建立时，系统安装一个系统线程异常处理例程
1. 异常发生时，`Win2K/XP` 控制权转移给 `NTDLL.KiUserExceptionDispatcher`
2. 如果你安装了线程相关的异常处理例程，系统就把异常发送给你的处理例程，交由其处理。
3. 每个线程相关的异常处理例程可以处理或者不处理这个异常，如果他不处理并且安装了多个线程相关的异常处理例程，可交由链起来的其他例程处理。
4. 如果这些例程均选择不处理异常，系统线程异常处理例程将起作用，调用 `ZwQueryInformationProcess` 判断是否被调试，没有调试并且你调用 `SetUnhandledExceptionFilter` 安装了最后异常处理例程的话，系统转向对它的调用。
5. 如果你没有安装最后异常处理例程或者他没有处理这个异常，系统会调用默认的系统处理程序，通常显示一个对话框，你可以选择关闭或者最后将其附加到调试器上的调试按钮。如果没有调试器能被附加于其上或者调试器也处理不了，系统就调用 `ExitProcess` 终结程序。
6. 不过在终结之前，系统仍然对发生异常的线程异常处理句柄来一次展开，这是线程异常处理例程最后清理的机会。一般只用来释放资源，不要试图修复什么。注意只对线程异常有展开。

#### 二. 程序被调试

0. 线程建立时，系统安装一个系统线程异常处理例程
1. 异常发生时，系统挂起程序并向调试器发送 `EXCEPTION_DEBUG_EVENT` 消息。
2. 调试器未能处理异常(比如我们在 `OD` 里按 `Shift+F7`)，控制权转移给 `NTDLL.KiUserExceptionDispatcher`。如果你安装了线程相关的异常处理例程，系统就把异常发送给你的程序 `seh` 处理例程，交由其处理。
3. 每个线程相关的异常处理例程可以处理或者不处理这个异常，如果他不处理并且安装了多个线程相关的异常处理例程，可交由链起来的其他例程处理。
4. 如果这些例程均选择不处理异常，系统线程异常处理例程将起作用，调用 `ZwQueryInformationProcess` 判断是否被调试。
5. 由于被调试，操作系统仍会再次挂起程序通知 `debugger`。

其实这里，我也有很多似懂非懂的地方，各位大侠请多指点。

`4dd466d0e1e0cc123afbe504da2462ca`
