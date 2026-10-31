
# Windows内核-异常（十）

## 10.1.cpu异常

中断和异常（都是IDT里面的函数）

中断源：

- 硬件产生
- 软件产生

异常源：

- 处理器检测到的程序错误异常
- 软件产生的异常
- 机器检测异常

异常分三类

- 错误：是一种通常能够被修正的异常
- 陷进：当引起陷进的指令发生时，马上产生异常
- 终止：严重错误

## 10.2.内核态异常派发

![](https://bbs.kanxue.com/upload/attach/202208/939298_T7WGAKEY57G2M7A.jpg)

如果这是第一次机会处理该异常，即`FirstChance`参数为`TRUE`，那么，交给内核调试器处理该异常，若内核调试器处理了该异常，则返回，发生异常的指令流继续执行；

若不存在内核调试器或内核调试器没处理该异常，则调用`RtlDispatchException`函数，试图将该异常分发到一个基于调用帧的异常处理器（call frame based handler）。只要能找到一个异常处理器能处理此异常，就返回。若第一次机会该异常未被处理，则再给内核调试器一次机会。若这次仍未被处理，则进入错误检查（bugcheck）。

因此，内核模式的异常必须被处理，一旦未被处理，则系统崩溃。

`_EXCEPTION_RECORD`异常记录

```
typedef struct _EXCEPTION_RECORD { 

	 NTSTATUS ExceptionCode;   // 异常码

	 ULONG ExceptionFlags;    // 异常标志

 	 struct _EXCEPTION_RECORD *ExceptionRecord;   //相关的另一个异常

	 PVOID ExceptionAddress;   // 异常发生地址

	 ULONG NumberParameters;   // 参数数组中的元素个数

 	ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];   // 参数数组

 } EXCEPTION_RECORD;
```

`KiDispatchException`

```
KiDispatchException (

    IN PEXCEPTION_RECORD ExceptionRecord,

    IN PKEXCEPTION_FRAME ExceptionFrame,   //一般为NULL

    IN PKTRAP_FRAME TrapFrame,		  //KTRAP_FRAME结构

    IN KPROCESSOR_MODE PreviousMode,     //0：内核 1：用户

    IN BOOLEAN FirstChance              //TRUE:第一轮   FALSE：第二轮

)
```

内核态派发流程逆向分析

![](https://bbs.kanxue.com/upload/attach/202208/939298_JDSEGGWM2U8FGJW.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_GDQGT6CA2YU75EZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_CGCMTNQRZTPMP4K.jpg)

## 10.3.用户态异常派发

首先判断是否这是第一次机会处理该异常。如果这是第一次机会，若发生异常的进程（即当前进程）的调试端口为空并且有内核调试器，则交给内核调试器。

若调试端口不为空，则发送一个消息至调试端口，然后等待应答。如果调试器处理了异常，则返回，继续执行。否则，把异常信息填充到用户栈中，并且设置好用户模式的`Eip`为`KeUserExceptionDispatcher`函数指针（指向`ntdll.dll`中的`KiUserExceptionDispatcher`函数，在进程管理子系统初始化时设置），准备好该函数所需要的参数，然后`KiDispatchException`函数返回。

因此，当从内核模式的异常处理例程返回以后，`ntdll.dll`的`KiUserException.Dispatcher`函数获得控制，它会试图将该异常分发给一个基于调用帧的异常处理器。如果存在一个基于调用帧的异常处理器处理了该异常，则程序继续执行。如果该异常仍然未被处理，则通过`NtRaiseException`函数再次进入到内核中处理该异常。这一次`KiDispatchException`函数被调用时，`FirstChance`参数为`FALSE`。

如果是第二次机会处理一个异常，并且当前进程有一个调试端口，则发送一个消息至调试端口，然后等待应答。如果进程的调试器处理了该异常，则返回，并继续执行。否则，如果当前进程有一个异常端口，则发送一个消息至异常端口，然后等待应答。如果连接异常端口的环境子系统（Win32 子系统）处理了该异常，则返回，并继续执行。如果该异常仍未被处理，则进程被终止。

如果是用户态，从这里会跳转

![](https://bbs.kanxue.com/upload/attach/202208/939298_T2ZXGYGMH269APJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_8XNNCCSZUM46QSY.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_Y9T5W4YE7YZQBQP.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_ASTZSAP7YEWJZUW.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_NU2J8FC3F5GE2KW.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_A5X4AGDCDJ46ASY.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_4P57R6SKFYVVSDG.jpg)

## 10.4.VEH逆向分析

vector exception handler，一个进程的全局异常捕获，是一个链表

`main.cpp`

```
#include <stdio.h>

#include <Windows.h>

LONG NTAPI VectoredExceptionHandler(EXCEPTION_POINTERS *ExceptionInfo)

{

	printf("ExceptionAddress = %x \n", ExceptionInfo->ExceptionRecord->ExceptionAddress);

	printf("ExceptionCode = %x \n", ExceptionInfo->ExceptionRecord->ExceptionCode);

	printf("ExceptionFlags = %x \n", ExceptionInfo->ExceptionRecord->ExceptionFlags);

	printf("NumberParameters = %x \n", ExceptionInfo->ExceptionRecord->NumberParameters);

	

    //跳过两个字节，也就是跳过这个int i = x / 0; 

	ExceptionInfo->ContextRecord->Eip += 2;   



	MessageBoxA(NULL, NULL, NULL, NULL);



	return EXCEPTION_CONTINUE_EXECUTION;

}

int main()

{

    //第一个参数为TRUE：插在链表头部,为FALSE：插在链表尾部; 第二个参数为异常处理函数

	PVOID mem = AddVectoredExceptionHandler(TRUE, VectoredExceptionHandler);

	int x = 0;

	int i = x / 0;

	printf("测试\r\n");

	system("pause");

	return 0;

}
```

结果

![](https://bbs.kanxue.com/upload/attach/202208/939298_RP3AUWQAA7NDKHW.jpg)

`ntdll.dll`--->`RtlAddVectoredExceptionHandler`逆向分析

![](https://bbs.kanxue.com/upload/attach/202208/939298_HDMB3GBPX52W8YR.jpg)

进入`_RtlAddVectoredExceptionHandler@12`

![](https://bbs.kanxue.com/upload/attach/202208/939298_6ZXRY8UGC4GZHSB.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_3JAYNU2BN69M67A.jpg)

`KiUserExceptionDispatcher`逆向分析

![](https://bbs.kanxue.com/upload/attach/202208/939298_5ZPJFJ64CT6SN8W.jpg)

进入`_RtlDispatchException@8`

![](https://bbs.kanxue.com/upload/attach/202208/939298_T4VHBKRAVTPPVHM.jpg)

进入`_RtlCallVectoredExceptionHandlers@8`

![](https://bbs.kanxue.com/upload/attach/202208/939298_XWJD63E24CK5JAX.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_XWQQRHDFUR6HEY7.jpg)

跳转

![](https://bbs.kanxue.com/upload/attach/202208/939298_74SN8AJ5HRQ622N.jpg)

如果引用计数位0，则跳转到

![](https://bbs.kanxue.com/upload/attach/202208/939298_VCZHMHYCRKM5ACT.jpg)

## 10.5.SEH异常

seh：结构化异常，每一个线程一个，`TEB`和`KPCR`都保存一个异常链表

![](https://bbs.kanxue.com/upload/attach/202208/939298_BTZSGX2PJHSK3ZJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_6PQPPBW6QVZW64A.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/939298_NCQ8AB55SNVMQ7T.jpg)

## 10.6.SEH编译器扩展

### 1.异常捕获

```
// Defined values for the exception filter expression

#define EXCEPTION_EXECUTE_HANDLER      1      //无论什么异常都会捕获

#define EXCEPTION_CONTINUE_SEARCH      0	 //不处理，等下一个去处理

#define EXCEPTION_CONTINUE_EXECUTION (-1)    //发生异常的地方去处理
```

### 2.try except

```
#include <stdio.h>

#include <Windows.h>

void test()

{

	__try

	{

		int x = 0;

		int y = x / 0;

		printf("-------1------\r\n");

	}

	__except (EXCEPTION_CONTINUE_SEARCH)

	{

		printf("------__except---2----\r\n");

	}

}

DWORD FilterException(PEXCEPTION_POINTERS info)

{

	if (info->ExceptionRecord->ExceptionCode == 0xC0000094)

	{

		printf("------除0错误----\r\n");

		info->ContextRecord->Eip += 2;

		return EXCEPTION_CONTINUE_EXECUTION;

	}

	return EXCEPTION_EXECUTE_HANDLER;

}

int main()

{

	__try

	{

		test();

	}

	__except (FilterException(GetExceptionInformation()))

	{

		printf("------__except---3----\r\n");

	}

	system("pause");

	return 0;

}
```

结果

![](https://bbs.kanxue.com/upload/attach/202208/939298_BQHZFX6SHVRDH3E.jpg)

## 10.7.UEH

顶层异常处理器，是整个异常处理的最后一环

```
#include <stdio.h>

#include <Windows.h>

LPTOP_LEVEL_EXCEPTION_FILTER oldFun = NULL;

LONG WINAPI filterFunc(

	_In_ struct _EXCEPTION_POINTERS* ExceptionInfo

)

{

	MessageBoxA(NULL,NULL,NULL,NULL);

	return oldFun(ExceptionInfo);

}

int main()

{

	oldFun = SetUnhandledExceptionFilter(filterFunc);

	int x = 0;

	int y = x / 0;

	return 0;

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-8-27 15:01 被zhang_derek编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
