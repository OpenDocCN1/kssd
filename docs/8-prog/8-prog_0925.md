
# 每天三篇 PaPer 系列之一,多线程同步问题,学习笔记~~~

> 原文：[`www.pediy.com/kssd/pediy11/124021.html`](https://www.pediy.com/kssd/pediy11/124021.html)

最近定下目标,坚持每天最少三篇 paper~

- -

于是有就有了下文~~~

是关于对 多线程同步的一些理解，包括 事件，临界,信号灯，互斥体~

come on~

首先

代码:

```
hInstance dd ?
hWinMain dd ?
hWinCount dd ?

dwCounter1 dd ?
dwCounter2 dd ?

dwThreads dd ? ;thread sum

hEvent dd ?

F_STOP equ 0001h
dwOption dd ?
```

对每个变量做以下 解释（从上到下顺序）

1. 模块 handle
2. 窗口 handle
3. 按钮 handle
4. 和 5 用于 累加的 var
5. 线程数变量
6. 事件 handle
7. 停止的 flag
8. flag 的 var~~~

go~

代码:

```
004011D4 syn_fix.<模块入口点> 6A 00 push 0
004011D6 E8 63000000 call <jmp.&kernel32.GetModuleHandleA>
004011DB A3 00304000 mov dword ptr ds:[403000],eax
004011E0 6A 00 push 0
004011E2 68 7F104000 push syn_fix.0040107F ; 消息处理过程
004011E7 6A 00 push 0
004011E9 68 E8030000 push 3E8
004011EE 50 push eax
004011EF E8 08000000 call <jmp.&user32.DialogBoxParamA>
004011F4 6A 00 push 0
004011F6 E8 3D000000 call <jmp.&kernel32.ExitProcess>

____ 分割线

0040107F /. 55 push ebp
00401080 |. 8BEC mov ebp,esp
00401082 |. 83C4 FC add esp,-4
00401085 |. 53 push ebx
00401086 |. 57 push edi
00401087 |. 56 push esi
00401088 |. 8B45 0C mov eax,dword ptr ss:[ebp+C]
0040108B |. 3D 13010000 cmp eax,113 ; Switch (cases 10..113)
00401090 |. 75 4D jnz short syn_fix.004010DF
00401092 |. 6A FF push -1 ; /Timeout = INFINITE; Case 113 (WM_TIMER) of switch 0040108B
00401094 |. FF35 18304000 push dword ptr ds:[403018] ; |hObject = NULL
0040109A |. E8 AB010000 call <jmp.&kernel32.WaitForSingleObject>; \WaitForSingleObject
0040109F |. 6A 00 push 0 ; /IsSigned = FALSE
....
...
..
```

首先 我们先看 `dlg` 初始化消息的处理~

源代码~

`.elseif eax == WM_INITDIALOG`

```
push hWnd
pop hWinMain
invoke GetDlgItem,hWnd,IDC_BTN1
mov hWinCount,eax; 给我控制室的联系方式~~
invoke CreateEvent,NULL,FALSE,TRUE,NULL;叫黑丝御姐去协调，那 20 多个工人的同步问题
;要是一个工人干完了，直接联系下一个工人,同时通知第一个工人，到了就直接工作，谁也不用报告~
mov hEvent,eax
```

1. 保存 窗口句柄
2. 获得按钮控件句柄
3. 创建一直自动的事件，且为置位，这样第一个线程，在遇到 `WaitForSingleObject` 直接就可以开始,不会陷入等待，造成 无法执行下去的 bug~~~

代码:

```
00401186 |> \3D 10010000 cmp eax,110
0040118B |. 75 2F jnz short syn_fix.004011BC
0040118D |. FF75 08 push dword ptr ss:[ebp+8] ; Case 110 (WM_INITDIALOG) of switch 0040108B
00401190 |. 8F05 04304000 pop dword ptr ds:[403004] ; 得到窗口句柄存放到数据段
00401196 |. 68 EB030000 push 3EB ; /ControlID = 3EB (1003.)
0040119B |. FF75 08 push dword ptr ss:[ebp+8] ; |hWnd
0040119E |. E8 65000000 call <jmp.&user32.GetDlgItem> ; \GetDlgItem
004011A3 |. A3 08304000 mov dword ptr ds:[403008],eax ; 获得 子控件句柄
004011A8 |. 6A 00 push 0 ; /EventName = NULL
004011AA |. 6A 01 push 1 ; |InitiallySignaled = TRUE
004011AC |. 6A 00 push 0 ; |ManualReset = FALSE
004011AE |. 6A 00 push 0 ; |pSecurity = NULL
004011B0 |. E8 77000000 call <jmp.&kernel32.CreateEventA> ; \CreateEventA
004011B5 |. A3 18304000 mov dword ptr ds:[403018],eax ; 控件的句柄
004011BA |. EB 0C jmp short syn_fix.004011C8
```

接下来看~`WM_COMMAND` 消息的处理

代码:

```
004010E6 |. 8B45 10 mov eax,dword ptr ss:[ebp+10] ; Case 111 (WM_COMMAND) of switch 0040108B
004010E9 |. 66:3D EB03 cmp ax,3EB ; 比较是否是相应的按钮控件
004010ED |. 0F85 D5000000 jnz syn_fix.004011C8
004010F3 |. 833D 14304000 00 cmp dword ptr ds:[403014],0
; 这里是通过判断 dwThread 来检测，是要开始计算，还是要停止，如果是已经开始计算了，要停止下来，
;那么这个变量就是非零，反之亦然,如果是非零 就继续向下执行，非零就从 0040110F 开始执行了
004010FA |. 74 13 je short syn_fix.0040110F
004010FC |. 830D 1C304000 01 or dword ptr ds:[40301C],1
00401103 |. 6A 01 push 1 ; /TimerID = 1
00401105 |. FF75 08 push dword ptr ss:[ebp+8] ; |hWnd
00401108 |. E8 01010000 call <jmp.&user32.KillTimer> ; \KillTimer
0040110D |. EB 4D jmp short syn_fix.0040115C
0040110F |> C705 0C304000 0000>mov dword ptr ds:[40300C],0 ; 计数变量清 0
00401119 |. C705 10304000 0000>mov dword ptr ds:[403010],0
00401123 |. 33DB xor ebx,ebx
00401125 |. EB 1F jmp short syn_fix.00401146
00401127 |> 8D45 FC /lea eax,dword ptr ss:[ebp-4] ; 线程 ID 变量的指针
0040112A |. 50 |push eax ; /pThreadId
0040112B |. 6A 00 |push 0 ; |CreationFlags = 0
0040112D |. 6A 00 |push 0 ; |pThreadParm = NULL
0040112F |. 68 00104000 |push syn_fix.00401000 ; |ThreadFunction = syn_fix.00401000
00401134 |. 6A 00 |push 0 ; |StackSize = 0
00401136 |. 6A 00 |push 0 ; |pSecurity = NULL
00401138 |. E8 F5000000 |call <jmp.&kernel32.CreateThread> ; \CreateThread
0040113D |. FF75 FC |push dword ptr ss:[ebp-4] ; /hObject
00401140 |. E8 E1000000 |call <jmp.&kernel32.CloseHandle> ; \CloseHandle
```

这里就开始执行线程了。我们到 线程函数里面 看看把~~~
这个是重点~

第一条线程创建后 `SetWindowText` 后~

代码:



# 同步问题与临界区分析

## 1. 定时器与线程管理

如果创建完 20 条线程以后，我们就创建一个定时器，用以将数值写到控件里面。

```
00401149                 |.  6A 00                push 0                    ; /如果创建完 20 条线程以后~我们就创建一个定时器，用以将数值写到控件里面
0040114B                 |.  68 F4010000          push 1F4                  ; |Timeout = 500. ms
00401150                 |.  6A 01                push 1                    ; |TimerID = 1
00401152                 |.  FF75 08              push dword ptr ss:[ebp+8] ; |hWnd
00401155                 |.  E8 BE000000          call <jmp.&user32.SetTimer> ; \\SetTimer
```

会创建一个定时器，`500MS` 执行一次 `WM_TIMER` 消息。

```
0040108B                 |.  3D 13010000          cmp eax,113               ;  Switch (cases 10..113)
00401090                 |.  75 4D                jnz short syn_fix.004010DF
00401092                 |.  6A FF                push -1                   ; /Timeout = INFINITE; Case 113 (WM_TIMER) of switch
00401094                 |.  FF35 18304000        push dword ptr ds:[403018] ; |hObject = 00000064 (window)
0040109A                 |.  E8 A9010000          call <jmp.&kernel32.WaitForSingleObject> ; \\WaitForSingleObject
```

这里同样是用一组 `WaitForSingleObject` 来等待线程完成，才进行下一个线程。

```
0040109F                 |.  6A 00                push 0                    ; /IsSigned = FALSE
004010A1                 |.  FF35 0C304000        push dword ptr ds:[40300C] ; |Value = A (10.)
004010A7                 |.  68 E9030000          push 3E9                  ; |ControlID = 3E9 (1001.)
004010AC                 |.  FF35 04304000        push dword ptr ds:[403004] ; |hWnd = 00700340 ('同步问题解决',class='#32770')
004010B2                 |.  E8 5B010000          call <jmp.&user32.SetDlgItemInt> ; \\SetDlgItemInt
004010B7                 |.  6A 00                push 0                    ; /IsSigned = FALSE
004010B9                 |.  FF35 10304000        push dword ptr ds:[403010] ; |Value = A (10.)
004010BF                 |.  68 EA030000          push 3EA                  ; |ControlID = 3EA (1002.)
004010C4                 |.  FF35 04304000        push dword ptr ds:[403004] ; |hWnd = 00700340 ('同步问题解决',class='#32770')
004010CA                 |.  E8 43010000          call <jmp.&user32.SetDlgItemInt> ; \\SetDlgItemInt
004010CF                 |.  FF35 18304000        push dword ptr ds:[403018] ; /hEvent = 00000064 (window)
004010D5                 |.  E8 68010000          call <jmp.&kernel32.SetEvent> ; \\SetEvent
```

同样告诉 Windows 自己搞完了，让别的线程进~~~

```
004010DA                 |.  E9 E7000000          jmp syn_fix.004011C6
```

## 2. 事件对象与线程调度

代码逻辑如下：

```
00401025                 |> /6A FF                /push -1                  ; /Timeout = INFINITE
00401027                 |. |FF35 18304000        |push dword ptr ds:[403018] ; |hObject = 00000064 (window)
0040102D                 |. |E8 16020000          |call <jmp.&kernel32.WaitForSingleObject> ; \\WaitForSingleObject
;  这里 WaitForSingleObject 来判断下一个线程是否能运行，当然因为我们创建事件的时候是置位的，所以第一条线程顺利运行.....其他的事件就会等待在这里
00401032                 |. |FF05 0C304000        |inc dword ptr ds:[40300C]
00401043                 |.  FF35 18304000        |push dword ptr ds:[403018] ; /hEvent = 00000064 (window)
00401049                 |.  E8 F4010000          |call <jmp.&kernel32.SetEvent> ; \\SetEvent
;等到线程要运行的执行运行完，我们就使用 SetEvent 置位，相当于告诉 Windows, 我做完了，让别的线程进来 XXOO 把~~~~
```

OK 事件搞完了~~~

## 3. 临界区（CRITICAL_SECTION）

我们看一下临界区~~~

临界区实现的是互斥技术。在任何时刻，只有一个线程能进入临界区。只有在该线程离开临界区后，其它线程才能进入。临界区对象不能被移动或者拷贝，进程也不能够修改该对象，而必须把它看作逻辑上不透明的（小黑箱）。使用由 Win32 API 提供的临界区函数来管理临界区对象。

要使用临界区，首先需要定义一个临界区对象，此时，进程负责分配相应内存：

```
CRITICAL_SECTION cs;
```

然后，临界区对象必须由程序中某个线程初始化：

```
InitializeCriticalSection(&cs);
```

当初始化完该临界区对象后，线程可以调用下面的函数进入临界区：

```
EnterCriticalSection(&cs);
```

进入临界区后，该线程被认为拥有该临界区对象。因此，其它线程调用 `EnterCriticalSection` 想进入该临界区时，会被挂起。等到拥有临界区对象的线程离开临界区后，它才有机会获得该临界区对象。一旦一个线程拥有临界区，那么它可以再调用 `EnterCriticalSection` 或者 `TryEnterCriticalSection` 而不会阻塞它自己的执行。这防止了一个线程因等待它自己所拥有的临界区而出现死锁。

（`TryEnterCriticalSection` 与 `EnterCriticalSection` 的区别：`EnterCriticalSection` 无论是否获得临界区，都立即返回（注：此处原文描述可能有误，通常 `Enter` 是阻塞的，`TryEnter` 是非阻塞的；但根据原文语境，应理解为：`TryEnterCriticalSection` 是立即返回且不阻塞的，而 `EnterCriticalSection` 在未获得时阻塞。原文表述为：“前者无论是否获得临界区，都立即返回；而后者在获得临界区前一直处理阻塞状态。” 这里的“前者”指代不明，根据上下文逻辑，通常 `TryEnter` 是非阻塞的。为了保持原文语义，我将保留原文对 `TryEnterCriticalSection` 的描述：“`TryEnterCriticalSection` 可以通过函数的返回值判断是否获得临界区，如果没有，可以做其它事情，而不是处于等待状态。”）

离开临界区：

```
LeaveCriticalSection(&cs);
```

当临界区不再需要时，可将其删除，从而释放相应系统资源：

```
DeleteCriticalSection(&cs);
```

使用临界区有一个限制，它只能用于同一进程内的线程间的互斥。而在某些情况下，需要协调两个不同进程对同一资源的共享（如共享内存）。此时，已不能再使用临界区做到，需要借助于互斥对象（`mutex object`）来实现（注：`mutex` 是一个合成词 `[mutual exclusion]`）。

理论我是从网上直接粘贴的 - - 大家直接看吧~~~

就是定义了一个 `CRITICAL_SECTION` 结构，然后初始化，然后 `EnterCriticalSection` 一下，这样就保证了只有一条线程可以操作指令，剩下的执行到这里就被挂起了。嘿嘿

还是看 OD~~

## 4. 线程状态观察

代码：

```
线程
标识       入口       数据块       最后错误          状态        优先权     用户时间     系统时间
0000009C   7C810856   7FF99000     ERROR_SUCCESS (000  激活         32 + 0      1.1875 s     0.0468 s
000000F8   7C810856   7FFD3000     ERROR_SUCCESS (000  激活         32 + 0      0.0937 s     0.4531 s
00000344   7C810856   7FFD5000     ERROR_SUCCESS (000  激活         32 + 0      0.0937 s     0.6718 s
0000055C   7C810856   7FF9B000     ERROR_SUCCESS (000  激活         32 + 0      1.4218 s     0.0781 s
00000560   7C810856   7FFDC000     ERROR_SUCCESS (000  激活         32 + 0      0.0312 s     0.5000 s
00000654   7C810856   7FFDA000     ERROR_SUCCESS (000  激活         32 + 0      0.0312 s     0.4218 s
0000071C   7C810856   7FFDB000     ERROR_SUCCESS (000  激活         32 + 0      0.7968 s     0.0937 s
00000748   7C810856   7FFD4000     ERROR_SUCCESS (000  激活         32 + 0      1.0625 s     0.0625 s
00000908(  004011C1   7FFDF000     ERROR_ACCESS_DENIE  激活         32 + 0      0.0468 s     0.1093 s
000009A8   7C810856   7FF9A000     ERROR_SUCCESS (000  激活         32 + 0      0.1093 s     0.9218 s
00000A24   7C810856   7FF9D000     ERROR_SUCCESS (000  激活         32 + 0      0.9062 s     0.1093 s
00000AF0   7C810856   7FFD9000     ERROR_SUCCESS (000  激活         32 + 0      1.1562 s     0.0468 s
00000B00   7C810856   7FFD7000     ERROR_SUCCESS (000  激活         32 + 0      1.0937 s     0.0312 s
00000BA4   7C810856   7FF9C000     ERROR_SUCCESS (000  激活         32 + 0      0.1093 s     0.4531 s
00000CE8   7C810856   7FFD8000     ERROR_SUCCESS (000  激活         32 + 0      0.1406 s     0.7031 s
00000D10   7C810856   7FF9E000     ERROR_SUCCESS (000  激活         32 + 0      0.1406 s     0.6093 s
00000D2C   7C810856   7FFDD000     ERROR_SUCCESS (000  激活         32 + 0      1.5468 s     0.0781 s
00000E20   7C810856   7FF98000     ERROR_SUCCESS (000  激活         32 + 0      0.0937 s     0.4531 s
00000E44   7C810856   7FFDE000     ERROR_SUCCESS (000  激活         32 + 0      0.0625 s     0.5000 s
00000F1C   7C810856   7FF97000     ERROR_SUCCESS (000  激活         32 + 0      1.1250 s     0.0625 s
00000F88   7C810856   7FF9F000     ERROR_SUCCESS (000  激活         32 + 0      1.2187 s     0.0781 s
```

这里可以看到所有线程还都是激活的，当我们运行 `EnterCriticalSection` 后我们看：

线程

标识       入口       数据块       最后错误          状态        优先权     用户时间     系统时间



## 线程状态记录

```
0000009C   7C810856   7FF99000     ERROR_SUCCESS (000  暂停         32 + 0        1.1875 s       0.0468 s  

000000F8   7C810856   7FFD3000     ERROR_SUCCESS (000  暂停         32 + 0        0.0937 s       0.4531 s  

00000344   7C810856   7FFD5000     ERROR_SUCCESS (000  暂停         32 + 0        0.0937 s       0.6718 s  

0000055C   7C810856   7FF9B000     ERROR_SUCCESS (000  暂停         32 + 0        1.4218 s       0.0781 s  

00000560   7C810856   7FFDC000     ERROR_SUCCESS (000  暂停         32 + 0        0.0312 s       0.5000 s  

00000654   7C810856   7FFDA000     ERROR_SUCCESS (000  暂停         32 + 0        0.0312 s       0.4218 s  

0000071C   7C810856   7FFDB000     ERROR_SUCCESS (000  暂停         32 + 0        0.7968 s       0.0937 s  

00000748   7C810856   7FFD4000     ERROR_SUCCESS (000  暂停         32 + 0        1.0625 s       0.0625 s  

00000908(  004011C1   7FFDF000     ERROR_ACCESS_DENIE 暂停         32 + 0        0.0468 s       0.1093 s  

000009A8   7C810856   7FF9A000     ERROR_SUCCESS (000  暂停         32 + 0        0.1093 s       0.9375 s  

00000A24   7C810856   7FF9D000     ERROR_SUCCESS (000  暂停         32 + 0        0.9062 s       0.1093 s  

00000AF0   7C810856   7FFD9000     ERROR_SUCCESS (000  暂停         32 + 0        1.1562 s       0.0468 s  

00000B00   7C810856   7FFD7000     ERROR_SUCCESS (000  暂停         32 + 0        1.0937 s       0.0312 s  

00000BA4   7C810856   7FF9C000     ERROR_SUCCESS (000  暂停         32 + 0        0.1093 s       0.4531 s  

00000CE8   7C810856   7FFD8000     ERROR_SUCCESS (000  暂停         32 + 0        0.1406 s       0.7031 s  

00000D10   7C810856   7FF9E000     ERROR_SUCCESS (000  暂停         32 + 0        0.1406 s       0.6093 s  

00000D2C   7C810856   7FFDD000     ERROR_SUCCESS (000  暂停         32 + 0        1.5468 s       0.0781 s  

00000E20   7C810856   7FF98000     ERROR_SUCCESS (000  暂停         32 + 0        0.0937 s       0.4531 s  

00000E44   7C810856   7FFDE000     ERROR_SUCCESS (000  激活         32 + 0        0.0625 s       0.5000 s  

00000F1C   7C810856   7FF97000     ERROR_SUCCESS (000  暂停         32 + 0        1.1250 s       0.0625 s  

00000F88   7C810856   7FF9F000     ERROR_SUCCESS (000  暂停         32 + 0        1.2187 s       0.0781 s  
```

只有一条线程运行的

执行`LeaveCriticalSection`后~~

### 线程

标识       入口       数据块       最后错误                  状态        优先权     用户时间      系统时间

```
000002A0   7C810856   7FFDE000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

000002E8   7C810856   7FFD6000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

000005A8   7C810856   7FFD9000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

000007E4   7C810856   7FFD4000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000854(  004011C1   7FFDF000     ERROR_SUCCESS (000  激活         32 + 0        0.0625 s       0.1875 s  

0000086C   7C810856   7FFDA000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000880   7C810856   7FF97000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000890   7C810856   7FFD3000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000904   7C810856   7FF9E000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

0000095C   7C810856   7FFD5000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000AC8   7C810856   7FF9B000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000B54   7C810856   7FFD8000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000B64   7C810856   7FF9D000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000B84   7C810856   7FF99000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000D44   7C810856   7FF9C000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000DAC   7C810856   7FFDC000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000E88   7C810856   7FF9F000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000EF0   7C810856   7FFD7000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000F6C   7C810856   7FF98000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  

00000F84   7C810856   7FFDD000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0156 s  

00000F8C   7C810856   7FF9A000     ERROR_SUCCESS (000  激活         32 + 0        0.0000 s       0.0000 s  
```

当再次有任何一条线程执行到`Enter`的时候~~~，`enter`就会把剩下的线程，都挂起。，保证了同步~~~

## 互斥体

`Windows` API 中提供了一个互斥体，功能上要比临界区强大。

`Mutex` 是互斥体的意思，当一个线程持有一个`Mutex`时，其它线程申请持有同一个`Mutex`会被阻塞，

因此可以通过`Mutex`来保证对某一资源的互斥访问（即同一时间最多只有一个线程访问）。

调用`CreateMutex`可以创建一个`Mutex`对象，

```
HANDLE CreateMutex(  
  LPSECURITY_ATTRIBUTES lpMutexAttributes,  
  BOOL bInitialOwner,  
  LPCTSTR lpName  
); 
```

其中参数`lpMutexAttributes`用来设定`Mutex`对象的安全描述符和是否允许子进程继承句柄。

`bInitialOwner`表明是否将`Mutex`的持有者设置为调用线程。

`lpName`参数设置`Mutex`的名字，该名字区分大小写并不能包含""，最大长度为`MAX_PATH`，可设置为`NULL`表明该`Mutex`为匿名对象。

如果调用成功，则返回`Mutex`的句柄，否则返回`NULL`，如果`lpName`不为`NULL`且调用前同名的`Mutex`已被创建，则返回同名`Mutex`的句柄，此时调用`GetLastError`将返回`ERROR_ALREADY_EXISTS`，参数`bInitialOwner`将被忽略。

理论大家自己去`msdn`、`google`、`baidu` ...

- -

我们再使用`Mutex`的时候 要使用 先获取互斥量，

获取互斥量 我们使用`WaitForSingleObject`

当互斥量被等待成功，他将自动复位

其他线程等待

，线程使用完后，调用

`ReleaseMutex, hMutex`

释放后，互斥量自动置位，此时 其他线程可以进入~~~

- - 就是 代码写的不一样

`CreateMutex`

`WaitForSingleObject`

`ReleaseMutex`

- -

很简单~~

## 信号灯

信号灯有点不一样~~

`Semaphore` 是旗语的意思，在`Windows`中，`Semaphore`对象用来控制对资源的并发访问数。`Semaphore`对象具有一个计数值，当值大于0时，`Semaphore`被置信号，当计数值等于0时，`Semaphore`被清除信号。每次针对`Semaphore`的`wait functions`返回时，计数值被减1，调用`ReleaseSemaphore`可以将计数值增加`lReleaseCount`参数值指定的值。

`CreateSemaphore`函数用于创建一个`Semaphore`

```
HANDLE CreateSemaphore(  
LPSECURITY_ATTRIBUTES lpSemaphoreAttributes,  
LONG lInitialCount, 这个就是指示创建后，信号灯对象的计数值，这里我们只需要一个就设置为 1  
LONG lMaximumCount, ;这个是最多，多少个线程能获取这个 计数值  
LPCTSTR lpName  
); 
```

`lpSemaphoreAttributes`为安全属性，

`lInitialCount`为`Semaphore`的初始值，

`lMaximumCount`为最大值，

`lpName`为`Semaphore`对象的名字，`NULL`表示创建匿名`Semaphore`

此外还可以调用`OpenSemaphore`来打开已经创建的非匿名`Semaphore`

```
HANDLE OpenSemaphore(  
DWORD dwDesiredAccess,  
BOOL bInheritHandle,  
LPCTSTR lpName  
); 
```

调用`ReleaseSemaphore`增加`Semaphore`计算值

```
BOOL ReleaseSemaphore(  
HANDLE hSemaphore,  
LONG lReleaseCount,  
LPLONG lpPreviousCount  
); 
```

`lpReleaseCount`参数表示要增加的数值，

`lpPreviousCount`参数用于返回之前的计算值，如果不需要可以设置为`NULL`

当然还是用`WaitForSingleObject`等待到信号灯后，会把信号灯计数-1，这里我们 直接

创建的时候，初始化值，和最大值 都设置为 1 ，那么 一个线程进入后，计数值就会-1 == 0，那么 状态变成 复位，其他的线程就等待这里了

执行到最后 调用

`ReleaseSemaphore`函数

`ReleaseSemaphore`函数会把计数值 +1

那么 对象状态又变成置位，那么，其他线程就可进入执行了~~~

- - 如此反复。。

写完了~~~..

笔记只是,一点愚见，

错误的地方，还请各位多多批评和指出

欢迎指导

然后我说完了~~~
