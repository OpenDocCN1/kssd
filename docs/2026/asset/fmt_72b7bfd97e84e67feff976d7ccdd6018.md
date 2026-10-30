
> 根据这个漏洞程序的代码将构建一个常规模糊测试器和内存模糊测试器，模糊测试器的源代码已经上传到github上，代码地址：**`[736K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8V1k6#2P5Y4A6W2M7R3`.`.](https://bbs.kanxue.com/elink@68cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8V1k6#2P5Y4A6W2M7R3`.`.)`**

# 二.包含漏洞的被测程序

以下是被测程序的源代码，可以看到，该程序的`vuln`函数包含了栈溢出漏洞：

```
/`/` `vuln.cpp : 此文件包含` `"main"` `函数。程序执行将在此处开始并结束。` `/``/` `#include <cstdio>` `#include <Windows.h>` `void vuln(char` `*``szInput)` `{` `char szStr[``100``]` `=` `{` `0` `};` `strcpy(szStr, szInput);` `printf(``"Vuln func...\n"``);` `}` `void test(char` `*``szInput)` `{` `vuln(szInput);` `printf(``"test func...\n"``);` `}` `void Decrypt(char` `*``szInput)` `{` `DWORD dwLen` `=` `strlen(szInput);` `for` `(DWORD i` `=` `0``; i < dwLen; i``+``+``)` `{` `szInput[i] ^``=` `190``;` `}` `printf(``"Decrypt func...\n"``);` `}` `void Init(char` `*``szInput, char` `*``szInit)` `{` `if` `(strlen(szInit) < MAXBYTE)` `{` `strcpy(szInput, szInit);` `printf(``"Init func...\n"``);` `}` `else` `{` `printf(``"输入的字符串过长\n"``);` `}` `}` `int` `main(``int` `argc, char` `*``argv[])` `{` `char szInput[MAXBYTE]` `=` `{` `0` `};` `Init(szInput, argv[``0``]);` `Decrypt(szInput);` `test(szInput);` `return` `0``;` `}`
```

# 三.常规模糊测试器

常规的模糊测试器的实现是通过产生不同的输入数据来启动程序完成测试，实现起来相对简单，只需要不断构建满足条件的输入数据然后启动程序就好。为了捕获到程序发生的异常，还需要在模糊测试器和被测程序中建立调试关系，测试代码如下：

```
#include "Fuzzy.h"
#include "auxiliary.h"

Fuzzy::Fuzzy(char *szFilePath)
{
if (strlen(szFilePath) < MAX_PATH)
{
strcpy(this->szFilePath, szFilePath);
}
else
{
printf("文件名过长\n");
}
}

char *Fuzzy::GetFilePath()
{
return this->szFilePath;
}

void Fuzzy::BeginFuzzy()
{
STARTUPINFO si = { 0 };
PROCESS_INFORMATION pi = { 0 };
DEBUG_EVENT DbgEvt = { 0 };
/`/` 保存调试事件的数据结构
bool bContinue = true;
/`/` 是否继续
DWORD dwContinueStatus = DBG_CONTINUE;
/`/` 恢复继续执行用的状态代码
DWORD dwSize = 4;
/`/` 输入数据的大小
char szInput[MAXBYTE] = { 0 };
/`/` 输入数据缓冲区
bool bExit = false;
/`/` 被调试进程结束则设为true

printf("Fzzy Begin....\n");

while (dwSize < MAXBYTE)
{
/`/` 随机输入数据
memset(szInput, 0, MAXBYTE);
for (DWORD i = 0; i < dwSize; i++)
{
szInput[i] = rand() % 255 + 1;
}

memset(&si, 0, sizeof(si));
memset(&pi, 0, sizeof(pi));
si.cb = sizeof(si);
if (!CreateProcess(this->szFilePath,
szInput,
NULL,
NULL,
FALSE,
CREATE_NEW_CONSOLE |
DEBUG_PROCESS |
DEBUG_ONLY_THIS_PROCESS,
NULL,
NULL,
&si,
&pi))
{
ShowError("CreateProcess");
goto exit;
}

bExit = false;
while (bContinue && !bExit)
{
memset(&DbgEvt, 0, sizeof(DbgEvt));

/`/` 等待调试事件发生
bContinue = WaitForDebugEvent(&DbgEvt, INFINITE);
if (!bContinue)
{
ShowError("WaitForDebugEvent");
break;
}

switch (DbgEvt.dwDebugEventCode)
{
case EXCEPTION_DEBUG_EVENT:
{
switch (DbgEvt.u.Exception.ExceptionRecord.ExceptionCode)
{
/`/` 捕获到程序异常
case EXCEPTION_ACCESS_VIOLATION:
{
printf(".....Catch Access Violation.....\n");
printf("dwSize is: %d\n", dwSize);
printf("Address is 0x%X\n", DbgEvt.u.Exception.ExceptionRecord.ExceptionAddress);
bExit = true;
break;
}
default:
{
break;
}
}
break;
}
case EXIT_PROCESS_DEBUG_EVENT:
{
bExit = true;
/`/` 该进程已退出
break;
}
default:
{
break;
}
}

/`/` 恢复被调试进程继续运行
bContinue = ContinueDebugEvent(DbgEvt.dwProcessId,
DbgEvt.dwThreadId,
dwContinueStatus);
if (!bContinue)
{
ShowError("ContinueDebugEvent");
break;
}
}

/`/` 增加数据大小，供下一次进行测试
dwSize += 4;

if (pi.hProcess) CloseHandle(pi.hProcess);
if (pi.hThread) CloseHandle(pi.hThread);
}

exit:
printf("Fuzzy End...\n");
}
```

运行结果如下：

![avatar](https://bbs.kanxue.com/upload/attach/202202/835440_5NQ3FPQRBPMKSWQ.png)

可以看到模糊测试器虽然能发现异常，但是性能开销是很大的。因此每次测试都要开启一个新的进程，被测程序每次都要经过`Init`->`Decrypt`->`test`->`vuln`才可以完成验证。如果这是一个需要大量运算的被测程序，或者是需要经过网络传输的程序，这个开销就会进一步增大。

# 四.内存模糊测试器构建方法

有以下两种构建内存模糊测试器的方法：

- 变异循环插入(Mutation Loop Insertion: MLI)
- 快照恢复变异(Snapshot restoration mutation: SRM)

变异循环插入方法需要首先通过逆向工程，人工定位解析例程的起始和结束地址。一旦定位完成，变异循环插入工具能够向应用程序中插入一个`mutation`例程（变异例程），变异例程负责修改解析例程拿到的数据。接下来，需要在内存中插入两条无条件跳转指令，这两条指令分别是"从解析例程的结尾跳到变异例程的开始处"和"从变异例程的结尾跳到解析例程的开始处"。

根据上面的被测程序的代码，此时的程序执行就会如下图所示：

![avatar](https://bbs.kanxue.com/upload/attach/202202/835440_NYPR44PF952FYKG.png)

此时，当程序运行到`test`函数末尾的时候，就会跳转到`mutate`变异例程执行。变异例程会将输入数据经过变异以后在次跳转到`test`函数起始地址开始执行。这样，围绕解析代码和目标应用创建了一个自给自足的数据变异循环。不需要为每个测试用例都启动一次进程，自然就节省了性能开销。变异例程每次迭代都会向`mutate`例程传入不同的，可能引发错误的数据。

与变异循环插入方法一样，快照恢复变异方法也需要定位到解析代码的开始和结束位置。在标识出这些位置后，快照变异工具会在到达解析代码的开始位置时为目标进程建立快照。在解析函数执行完成后，快照恢复变异工具恢复进程快照，对原先数据产生变异，并使用变异得到的数据重新执行解析代码。

此时程序的执行流程就会如下图所示：

![avatar](https://bbs.kanxue.com/upload/attach/202202/835440_KB4GWKEYNPVMT9Q.png)

> 由于变异循环插入的方法需要把变异例程写入到被测程序，实现起来比较麻烦。而且要不少数据需要硬编码，所以在实现的模糊测试器中就没有实现该方法。只实现了快照恢复变异的插入方法

# 五.快照恢复变异

快照恢复变异的方法需要在程序的解析例程中找到插入点，这样才可以进行相应的保存快照和恢复快照的操作。在`vuln.exe`程序中，通过调试器可以定位，`test`函数的起始地址是`0x004010B0`，结束位置是`0x004010D8`。

![avatar](https://bbs.kanxue.com/upload/attach/202202/835440_GDYPVZESPB429A2.png)

由于模糊测试器和被测程序处于调试关系中，所以可以通过向`test`函数的起始和结束位置插入软件断点的方式来实现对被测程序的HOOK操作。在插入软件断点后，当被测程序运行到相应位置，模糊测试器就可以获得控制权，然后在对程序进行下一步操作。

当被测程序在函数起始处中断的时候，首先要判断是不是第一次运行到这个地址，如果是的话就需要保存进程的快照。

而进程快照由两部分构成：

- 被测进程的线程`CONTEXT`
- 被测进程的内存页状态和内容

当被测程序运行到`test`函数末尾的时候，就需要恢复快照，也就是恢复保存的线程`CONTEXT`和内存页的状态和内容。这两部分的代码分别在`Thread.cpp`和`Pages.cpp`中实现：
