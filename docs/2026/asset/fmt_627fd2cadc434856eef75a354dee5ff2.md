
如果还存在用户特殊APC，那么会重新将`ApcState.UserApcPendingAll.SpecialUserApcPeding`置1。

若使用while循环调用`KiDeliverApc(1)`处理用户APC（[例如这张图](https://bbs.kanxue.com/#while_ua)），就会重复在用户堆栈中添加相应用户特殊APC的信息（因为这个while内的指令执行了多次），在返回R3，进入`ntdll!KiUserApcDispatcher`后，可配合`ZwContinue`一次性执行多个用户特殊APC。

**例**：

当用户APC链表中存在3个用户特殊APC时：

通过`whiel`循环调用`KiDeliverApc(1)`处理后，返回R3`ntdll!KiUserApcDispatcher`时的堆栈：（`_context`仅展示部分）

![](https://bbs.kanxue.com/upload/attach/202302/914360_PA4U679E2FJY43Y.png)

**当返回R3时**，此时RIP定位到`ntdll!KiUserApcDispatcher`，RSP定位到红色部分，准备处理第一个APC（红色部分）。

**处理完第一个APC后**（红色部分），通过`ZwContinue`再次跳转到`ntdll!KiUserApcDispatcher`，同时RSP也定位到了黄色部分，处理第二个APC（黄色部分）。

**处理完第二个APC后**，又通过`ZwContinue`再次跳转到`ntdll!KiUserApcDispatcher`，同时RSP也定位到了青色部分，处理第三个APC（青色部分）。

**处理完第三个APC后**，已经没有APC要执行了，就通过`ZwContinue`返回到原本RIP的下一行，同时RSP也恢复到灰色部分。

自此，三个用户特殊APC执行完成。线程也就可以干自己的事情了。

**注意这种情况只有用户特殊APC才会有！！！**

## APC执行篇总结<a name="APC执行篇总结"></a>

整理了一下

![](https://bbs.kanxue.com/upload/attach/202302/914360_MSJFZGZ9WCFSVVQ.png)

APC的执行总是从链表表尾开始，所以特殊APC执行的比普通APC要早。

内核特殊APC的`_KAPC.NormalRoutine`为空。

不论是内核APC还是用户APC，选中后总要先执行`_KAPC.KernelRoutine`。

调用一次`KiDeliverApc`，就会把全部内核APC执行。

调用一次`KiDeliverApc`，只能选出一个用户普通APC，然后将相关信息添加到用户堆栈中。

用户APC总是在从R0返回R3的时候执行。

用户普通APC只能在一个一个分开的时间段执行。而用户特殊APC是一次性连续全部执行。

用户普通APC只能等待线程可接警时才能执行，而用户特殊APC不需要，它可以快速执行。

# APC玩法<a name="APC玩法"></a>

## R3注册用户特殊APC<a name="第一个玩法"></a>

`NtQueueApcThreadEx`这个函数竟然是`ntdll`导出的！微软还把它藏起来不让用了是吧？！

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 | `NTSTATUS __fastcall NtQueueApcThreadEx(` `void` `*``ThreadHandle,` `/``/``需要插入的线程句柄` `BOOLEAN flag,` `/``/``0``：用户普通APC` `1``：用户特殊APC 其余值也为普通APC` `__int64 ApcRoutine,` `/``/``需要执行的APC函数指针` `__int64 NormalContext,` `/``/``需要执行的APC函数的第零个参数` `__int64 SystemArgument1,` `/``/``需要执行的APC函数的第一个参数` `__int64 SystemArgument2)` `/``/``需要执行的APC函数的第二个参数` |

**示例代码**：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 | `#include<Windows.h>` `#include<iostream>` `NTSTATUS(*NtQueueApcThreadEx)` `(HANDLE thread,` `/``/``线程句柄` `ULONG64 flag,` `/``/``1``：用户特殊APC，无需使用 TestAlert` `ULONG64 NormalRoutine,` `/``/``需要执行的函数` `ULONG64 NormalContext,` `/``/``第零个参数` `ULONG64 s1,` `/``/``第一个参数` `ULONG64 s2` `/``/``第二个参数` `)` `=` `NULL;` `VOID SpecialUserApcTest(ULONG64 a, ULONG64 b, ULONG64 c) {` `/``/``打印参数测试` `printf("\n%X %X %X \n", a, b, c);` `}` `int main() {` `PULONG64 NtQueueApcThreadEx_s` `=` `(PULONG64)&NtQueueApcThreadEx;` `/``/``获取NtQueueApcThreadEx函数指针` `*NtQueueApcThreadEx_s` `=` `(ULONG64)GetProcAddress(GetModuleHandleA("ntdll"),` `"NtQueueApcThreadEx");` `/``/``插入用户特殊APC` `NtQueueApcThreadEx(GetCurrentThread(),` `1`,`(ULONG64)SpecialUserApcTest,` `0x1111`,`0x2222`,`0x3333`);` `system("pause");` `}` |

有其他进程的线程句柄的话，也能向其他进程插入用户特殊APC。

## R0通过插入内核特殊APC读取进程内存<a name="第二个玩法"></a>

内核特殊APC，是能执行的APC中最早执行的，因此可以用它来做些事情。（不仅限于读内存）

项目地址：[Kernel-Special-APC-ReadProcessMemory](https://bbs.kanxue.com/elink@e57K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6u0j5@1g2&6i4K6u0V1z5e0V1&6i4K6u0r3d9$3g2J5L8X3g2D9i4K6u0V1f1%4m8W2j5$3W2S2L8q4)9J5k6p5q4b7b7#2)9J5k6q4u0W2j5h3c8b7M7X3!0U0k6i4y4K6e0h3g2E0L8%4u0&6)

**因为我电脑上没有网络游戏，所以还没测试过读取受保护进程的内存。但是理论上是可读的。**

**更详细的信息请点击项目地址查看。**

**代码仅作学习与交流使用，请勿用作非法用途！！！！！**

### 优点：

无需获取进程句柄，无需挂靠进程（线程），无需切换CR3，让目标进程自己将内存交出来。

### 缺点：

因为没有创建新的线程在目标进程上下文，所以，读取内存的时间相比挂靠进程（线程）的方式要慢上许多。（但也可以接受）

![](https://bbs.kanxue.com/upload/attach/202302/914360_WBEB99BFSCHTTTH.png)

**读取速度测试（Read speed test）**：

`KernelSpecialAPC` ：

读取 1000000 次长度为 30 字节的内存所需时间为：21985 ms

`ReadProcessMemory`：

读取1000000 次长度为 30 字节的内存所需时间为：890 ms

**当进程全部线程被挂起时，无法读取内存！**

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-2-9 12:46

被icey\_编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
