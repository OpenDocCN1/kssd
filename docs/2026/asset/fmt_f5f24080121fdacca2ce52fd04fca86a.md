
```c
typedef struct _KTIMER_TABLE_ENTRY_2K3 {

	LIST_ENTRY Entry;

	ULARGE_INTEGER Time;

}KTIMER_TABLE_ENTRY_2K3, *PKTIMER_TABLE_ENTRY_2K3;



#pragma PAGECODE

static ULONG GetDpcTimerInformation_2K3(PVOID* pvBuf)

{

	ULONG						NumberOfTimerTable;

	ULONG						i;

	ULONG						ulCount = 0;

	MyDpcTimer					MyDpc;

	PKDPCTIMER					pTimer = NULL;

	PKTIMER_TABLE_ENTRY_2K3		pTimerTableEntry = NULL;

	PLIST_ENTRY					pNextList = NULL;



	NumberOfTimerTable = 0x200;																	//_KTIMER_TABLE_ENTRY数量

	pTimerTableEntry = (PKTIMER_TABLE_ENTRY_2K3)GetDpcTimerListHeadForXp_2K3();

	if (pTimerTableEntry == NULL) return 0;





	*pvBuf = CallExAllocatePoolWithTag(PagedPool,MAX_DPCTIMER_COUNT*sizeof(MyDpcTimer),64152);		//分配足够大的缓冲

	if (*pvBuf == NULL) return 0;

	RtlZeroMemory(*pvBuf,MAX_DPCTIMER_COUNT*sizeof(MyDpcTimer));







	for ( i = 0; i < NumberOfTimerTable; i++, pTimerTableEntry++ )								//保存在ntos中的一个结构数组

	{

		if (MmIsAddressValidEx((PVOID)pTimerTableEntry) == VCS_INVALID) goto __exit1;



		if (pTimerTableEntry->Time.HighPart == 0xFFFFFFFF) continue;							//为空的数组高位双字为FFFFFFFF





		if ( MmIsAddressValidEx((PVOID)pTimerTableEntry->Entry.Blink) == VCS_INVALID ) continue;	//如果listentry域地址无效,continue

		if ( MmIsAddressValidEx((PVOID)pTimerTableEntry->Entry.Flink) == VCS_INVALID ) continue;





		for ( pNextList	 = (PLIST_ENTRY)pTimerTableEntry->Entry.Blink;

			  pNextList	!= (PLIST_ENTRY)&pTimerTableEntry->Entry;

			  pNextList	 = pNextList->Blink)

		{

			//取得timer对象

			pTimer = CONTAINING_RECORD(pNextList,KDPCTIMER,TimerListEntry);



			if (!MmIsAddressValid((PVOID)pTimer) || 

				!MmIsAddressValid((PVOID)pTimer->Dpc) || 

				!MmIsAddressValid((PVOID)pTimer->Dpc->DeferredRoutine))

			{

				if ( !MmIsAddressValid(pNextList->Blink) ) break;

				continue;

			}



			while (TRUE)

			{

				RtlZeroMemory(&MyDpc,sizeof(MyDpcTimer));

				if ( !MmIsAddressValid((PVOID)pTimer) ) break;

				MyDpc.TimerAddress = (ULONG)pTimer;



				if ( !MmIsAddressValid((PVOID)pTimer->Dpc) ) break;

				MyDpc.DpcAddress = (ULONG)pTimer->Dpc;



				if ( !MmIsAddressValid((PVOID)pTimer->Dpc->DeferredRoutine) ) break;

				MyDpc.DpcRoutineAddress = (ULONG)pTimer->Dpc->DeferredRoutine;



				if ( !MmIsAddressValid((PVOID)&pTimer->Period) ) break;

				MyDpc.Period = pTimer->Period;



				RtlMoveMemory( (PVOID)((ULONG)*pvBuf+ulCount*sizeof(MyDpcTimer)),&MyDpc,sizeof(MyDpcTimer) );

				ulCount++;

				if (ulCount >= MAX_DPCTIMER_COUNT) break;	



				break;

			}



			if ( !MmIsAddressValid(pNextList->Blink) ) break;



		}//end while

	}// end for



	return ulCount * sizeof(MyDpcTimer);



__exit1:

	if (*pvBuf != NULL) ExFreePool(*pvBuf);

	*pvBuf = NULL;

	return 0;

}
```

最后说一下 win7 下面的遍历,在 win7 下`KiTimerTableListHead`被从符号表中抹去了,所以也就没办法通过搜索的方式寻找了

在 win7 下定时器链表被放到了`KPRCB`结构中,在 win7 7600 和 win7 7601 中`KPRCB+0x1960`的位置:

![](https://bbs.kanxue.com/upload/attach/201203/440388_x8jnop2zbsze7z5.jpg)

相关结构:

![](https://bbs.kanxue.com/upload/attach/201203/440388_241cc0wu0nl7t3y.jpg)

win7下`_KTIMER_TABLE_ENTRY`数量又被调整为256,找到`_KTIMER_TABLE.TimerEntries`后,

遍历`TimerEntries.Entry`就行了

所以大体流程就是:取`KPRCB`地址,+0x1960偏移得到`_KTIMER_TABLE`,+0x40偏移得到`_KTIMER_TABLE.TimerEntries`

最后遍历`_KTIMER_TABLE.TimerEntries.Entry`

需要注意的是`KPRCB`是每个处理器都拥有一个的,所以需要遍历每一个处理器

`KiProcessorBlock`是一个未导出的全局变量,指向一个数组,这个数组保存了每一个`CPU`的`KPRCB`结构地址

而`KiProcessorBlock`在`KdDebuggerData64`这个结构中出现:

![](https://bbs.kanxue.com/upload/attach/201203/440388_ofyg0qjfhpdidq1.jpg)

`KdDebuggerData64`可以从`KdVersionBlock`后面得到,`KdVersionBlock`又是个啥啊?

[d9aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2T1L8$3A6A6j5h3&6Y4i4K6u0r3j5X3I4G2k6#2)9J5c8X3W2@1k6h3#2Q4x3V1k6T1k6X3p5%4j5e0k6V1z5h3j5I4j5K6V1I4x3$3g2W2x3K6R3H3x3e0u0X3x3U0S2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@ad6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2T1L8$3A6A6j5h3&6Y4i4K6u0r3j5X3I4G2k6#2)9J5c8X3W2@1k6h3#2Q4x3V1k6T1k6X3p5%4j5e0k6V1z5h3j5I4j5K6V1I4x3$3g2W2x3K6R3H3x3e0u0X3x3U0S2Q4x3X3g2Z5N6r3#2D9)

combojiang大牛在这里有介绍,其中关于`KdDebuggerData64`结构的定义可以在wrk中找到 <搜索`_KDDEBUGGER_DATA64`>

遍历过程:

```c
#define MAX_PROCESSOR_COUNT 32
#pragma PAGECODE
static ULONG GetDpcTimerInformation_WIN7(PVOID* pvBuf)

{

	ULONG						TimerTableOffsetInKprcb;

	ULONG						NumberOfTimerTable;

	ULONG						NumberOfProcessor;

	ULONG						i,j;

	ULONG						ulTemp;

	ULONG						ulCount = 0;

	PULONG						pKiProcessorBlock = NULL;

	PKTIMER_TABLE_ENTRY_WIN7	pTimerTableEntryWin7 = NULL;

	PLIST_ENTRY					pNextList = NULL;

	PKDPCTIMER					pTimer = NULL;

	MyDpcTimer					MyDpc;



	TimerTableOffsetInKprcb = 0x1960+0x40;														//首个_KTIMER_TABLE_ENTRY在PRCB中的偏移

	NumberOfTimerTable = 0x100;																	//_KTIMER_TABLE_ENTRY数量



	NumberOfProcessor = (ULONG)KeNumberProcessors;												//当前机器处理器数量

	if (NumberOfProcessor > MAX_PROCESSOR_COUNT) return 0;





	pKiProcessorBlock = (PULONG)GetKiProcessorBlock();											//取得KiProcessorBlock,包含了NumberOfProcessor个KPRCB

	if (pKiProcessorBlock == NULL) return 0;



	

	*pvBuf = CallExAllocatePoolWithTag(PagedPool,MAX_DPCTIMER_COUNT*sizeof(MyDpcTimer),652);		//分配足够大的缓冲

	if (*pvBuf == NULL) return 0;

	RtlZeroMemory(*pvBuf,MAX_DPCTIMER_COUNT*sizeof(MyDpcTimer));





	for ( i = 0; i < NumberOfProcessor; i++, pKiProcessorBlock++ )								//DPC timer在每个cpu中都有一个队列,所以枚举每一个KPRCB

	{

		if ( MmIsAddressValidEx((PVOID)pKiProcessorBlock) == VCS_INVALID ) goto __exit1;		//检测一下当前的KPRCB地址是否可访问





		ulTemp = *pKiProcessorBlock + TimerTableOffsetInKprcb;									//取得当前CPU的KPRCB中首个KTIMER_TABLE_ENTRY地址

		if ( MmIsAddressValidEx((PVOID)ulTemp) == VCS_INVALID )	goto __exit1;





		pTimerTableEntryWin7 = (PKTIMER_TABLE_ENTRY_WIN7)ulTemp;								//此时ulTemp是 timer table entry地址



		for ( j = 0; j < NumberOfTimerTable; j++, pTimerTableEntryWin7++ )						//准备遍历timer table表

		{

			if ( MmIsAddressValidEx((PVOID)pTimerTableEntryWin7) == VCS_INVALID ) goto __exit1;

			if ( pTimerTableEntryWin7->Time.HighPart == 0xFFFFFFFF ) continue;					//为空的数组高位双字为FFFFFFFF





			if ( MmIsAddressValidEx((PVOID)pTimerTableEntryWin7->Entry.Blink) == VCS_INVALID ) continue;

			if ( MmIsAddressValidEx((PVOID)pTimerTableEntryWin7->Entry.Flink) == VCS_INVALID ) continue;





			for ( pNextList	 = (PLIST_ENTRY)pTimerTableEntryWin7->Entry.Blink;

				  pNextList	!= (PLIST_ENTRY)&pTimerTableEntryWin7->Entry;

				  pNextList	 = pNextList->Blink)

			{



				pTimer = CONTAINING_RECORD(pNextList,KDPCTIMER,TimerListEntry);					//取得timer对象



				if (!MmIsAddressValid((PVOID)pTimer)||

					!MmIsAddressValid((PVOID)pTimer->Dpc)||

					!MmIsAddressValid((PVOID)pTimer->Dpc->DeferredRoutine))

				{

					if (!MmIsAddressValid((PVOID)pNextList->Blink)) break;

					continue;

				}





				while (TRUE)

				{

					RtlZeroMemory(&MyDpc,sizeof(MyDpcTimer));

					if ( !MmIsAddressValid((PVOID)pTimer) ) break;

					MyDpc.TimerAddress = (ULONG)pTimer;



					if ( !MmIsAddressValid((PVOID)pTimer->Dpc) ) break;

					MyDpc.DpcAddress = (ULONG)pTimer->Dpc;



					if ( !MmIsAddressValid((PVOID)pTimer->Dpc->DeferredRoutine) ) break;

					MyDpc.DpcRoutineAddress = (ULONG)pTimer->Dpc->DeferredRoutine;



					if ( !MmIsAddressValid((PVOID)&pTimer->Period) ) break;

					MyDpc.Period = pTimer->Period;



					RtlMoveMemory( (PVOID)((ULONG)*pvBuf+ulCount*sizeof(MyDpcTimer)),&MyDpc,sizeof(MyDpcTimer) );

					ulCount++;

					if (ulCount >= MAX_DPCTIMER_COUNT) break;	



					break;

				}



				if (!MmIsAddressValid((PVOID)pNextList->Blink)) break;



			}// end while

		}// end for

	}// end for



	return ulCount * sizeof(MyDpcTimer);



__exit1:

	if (*pvBuf != NULL) ExFreePool(*pvBuf);

	*pvBuf = NULL;

	return 0;

}
```

大体过程与 2k3是一样的,可以修改一下以兼容两个系统

最后看一下 win7 下的枚举效果:

![](https://bbs.kanxue.com/upload/attach/201203/440388_5almlycf3qlsm4q.jpg)

xt:

![](https://bbs.kanxue.com/upload/attach/201203/440388_j461fomfh2sw7a2.jpg)
