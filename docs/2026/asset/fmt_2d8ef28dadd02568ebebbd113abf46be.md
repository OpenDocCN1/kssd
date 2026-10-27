
以下函数用于分配单个 PXE 目录，并返回物理地址与 PPE 页大小：

```
ULONG_PTR AllocateSinglePXEDirectory(OUT PULONG_PTR BaseAddress, OUT PULONG SizeOfPPEPages)

{

	ULONG_PTR Result = 0;

	ULONG_PTR PteBase = PTEBase;

	ULONG_PTR OffsetMask = 0x7FFFFFFFF8;

	ULONG_PTR PXEVA = PteBase + (((PteBase + (((PteBase + ((PteBase >> 9) & OffsetMask)) >> 9) & OffsetMask)) >> 9) & OffsetMask) + 0x800; //有效起始内核虚拟地址0xFFFF800000000000对应的PXE子项

	ULONG_PTR PXEVAEnd = PXEVA + 0x800; //0x800 + 0x800 == 0x1000 == PAGE_SIZE

	do

	{

		if (!(*(PULONGLONG)PXEVA & 0xFFFFFFFFF001))

		{

			PHYSICAL_ADDRESS Alloc;

			Alloc.QuadPart = MAXULONG64;

			ULONG TotalSizeOfValidPPEPages = 0x200 * PAGE_SIZE;

			while (TotalSizeOfValidPPEPages >= PAGE_SIZE && !(Result = (ULONG_PTR)MmAllocateContiguousMemory(TotalSizeOfValidPPEPages + PAGE_SIZE, Alloc)))

				TotalSizeOfValidPPEPages >>= 1;

			if (Result)

			{

				if (SizeOfPPEPages) *SizeOfPPEPages = TotalSizeOfValidPPEPages;

				ULONG64 OringinalIRQL = __readcr8();

				__writecr8(DISPATCH_LEVEL);

				if (BaseAddress) *BaseAddress = ((PXEVA & 0xFF8) + 0xFFFF000) << 36;

				ULONG_PTR PTEVA = PteBase + ((Result >> 9) & OffsetMask);

				ULONG_PTR PDEVA = PteBase + ((PTEVA >> 9) & OffsetMask);

				ULONG_PTR PPEVA = PteBase + ((PDEVA >> 9) & OffsetMask);

				ULONGLONG StartValue = *(PULONGLONG)PPEVA;

				if (StartValue & 0x80)

				{

					StartValue &= ~0x80;

					StartValue += (Result & 0x3FFFF000);

				}

				else

				{

					StartValue = *(PULONGLONG)PDEVA;

					if (StartValue & 0x80)

					{

						StartValue &= ~0x80;

						StartValue += (Result & 0x1FF000);

					}

					else StartValue = *(PULONGLONG)PTEVA;

				}

				*(PULONGLONG)PXEVA = StartValue;

				ULONG PPEOffset = 0;

				ULONG PPEOffsetEnd = TotalSizeOfValidPPEPages >> 9;

				ULONG_PTR PPEBase = Result;

				do

				{

					*(PULONGLONG)(PPEBase + PPEOffset) = StartValue + ((PPEOffset + 8) << 9);

					PPEOffset += 8;

				} while (PPEOffset < PPEOffsetEnd);

				RtlZeroMemory((PVOID)(PPEBase + PPEOffset), PAGE_SIZE - PPEOffset);

				__writecr8(OringinalIRQL);

			}

			break;

		}

		PXEVA += 8;

	} while (PXEVA < PXEVAEnd);

	return Result;

}
```

以下函数用于为所有有效的 PPE 页填充 PDE 数组：

```
ULONG_PTR FillPDEArrayForAllValidPPEs(ULONG_PTR PagePointer, ULONG_PTR BaseAddress, ULONG SizeOfPPEPages, ULONG_PTR VirtualAddress)

{

	ULONG_PTR Result = 0;

	ULONG_PTR PteBase = PTEBase;

	ULONG_PTR OffsetMask = 0x7FFFFFFFF8;

	ULONG_PTR PDEVA = PteBase + (((PteBase + (((VirtualAddress & 0xFFFFFF8000000000) >> 9) & OffsetMask)) >> 9) & OffsetMask);

	ULONG_PTR PPEVA = PteBase + ((PDEVA >> 9) & OffsetMask);

	ULONG_PTR PXEVA = PteBase + ((PPEVA >> 9) & OffsetMask);

	if ((*(PUCHAR)PXEVA & 0x81) == 1) //不支持512G超大页面

	{

		if (SizeOfPPEPages >= PAGE_SIZE)

		{

			ULONG_PTR NewPDEVA = PagePointer + PAGE_SIZE;

			ULONG_PTR NewPDEVAEnd = NewPDEVA + SizeOfPPEPages;

			ULONG64 OringinalIRQL = __readcr8();

			__writecr8(DISPATCH_LEVEL);

			do

			{

				UCHAR ByteFlag = *(PUCHAR)PPEVA;

				if (ByteFlag & 1)

				{

					if ((CHAR)ByteFlag < 0) //为了不改变原先填充的PPE项，将1G大页面等分成512个2M大页面

					{

						ULONGLONG PDEStartValue = *(PULONGLONG)PPEVA;

						ULONG PDEOffset = 0;

						do

						{

							*(PULONGLONG)(NewPDEVA + PDEOffset) = PDEStartValue + (PDEOffset << 9);

							PDEOffset += 8;

						} while (PDEOffset < PAGE_SIZE);

					}

					else memcpy((PVOID)NewPDEVA, (PVOID)PDEVA, PAGE_SIZE);

				}

				else RtlZeroMemory((PVOID)NewPDEVA, PAGE_SIZE);				

				PDEVA += PAGE_SIZE;

				PPEVA += 8;

				NewPDEVA += PAGE_SIZE;

			} while (NewPDEVA < NewPDEVAEnd);

			__writecr8(OringinalIRQL);

			__writecr3(__readcr3());

			Result = BaseAddress + (VirtualAddress & 0x7FFFFFFFFF);

		}

	}	

	return Result;

}
```

以下函数将指定虚拟地址对应的页标记为脏页：

```
BOOLEAN MakeDirtyPage(ULONG_PTR VirtualAddress)

{

	BOOLEAN Result = FALSE;

	ULONG_PTR PteBase = PTEBase;

	ULONG_PTR OffsetMask = 0x7FFFFFFFF8;

	ULONG_PTR PTEVA = PteBase + ((VirtualAddress >> 9) & OffsetMask);

	ULONG_PTR PDEVA = PteBase + ((PTEVA >> 9) & OffsetMask);

	ULONG_PTR PPEVA = PteBase + ((PDEVA >> 9) & OffsetMask);

	ULONG_PTR PXEVA = PteBase + ((PPEVA >> 9) & OffsetMask);

	if ((*(PUCHAR)PXEVA & 0x81) == 1) //不支持512G超大页面

	{

		UCHAR ByteFlag = *(PUCHAR)PPEVA;

		if (ByteFlag & 1)

		{

			if ((CHAR)ByteFlag < 0)

			{

				*(PUCHAR)PPEVA |= 0x42; //Dirty1 & Dirty

				Result = TRUE;

			}

			else

			{

				ByteFlag = *(PUCHAR)PDEVA;

				if (ByteFlag & 1)

				{

					if ((CHAR)ByteFlag < 0)

					{

						*(PUCHAR)PDEVA |= 0x42;

						Result = TRUE;

					}

					else

					{

						if (_bittest((PLONG)PTEVA, 0))

						{

							*(PUCHAR)PTEVA |= 0x42;

							Result = TRUE;

						}

					}

				}

			}

		}

	}

	__invlpg((PVOID)VirtualAddress);

	return Result;

}
```

以下函数用于释放单个 PXE 目录：

```
void FreeSinglePXEDirectory(ULONG_PTR PagePointer, ULONG_PTR BaseAddress)

{

	ULONG_PTR PteBase = PTEBase;

	ULONG_PTR OffsetMask = 0x7FFFFFFFF8;

	*(PULONGLONG)(PteBase + (((PteBase + (((PteBase + (((PteBase + ((BaseAddress >> 9) & OffsetMask)) >> 9) & OffsetMask)) >> 9) & OffsetMask)) >> 9) & OffsetMask)) = 0;

	MmFreeContiguousMemory((PVOID)PagePointer);

	__writecr3(__readcr3());

}
```

在`Win10 18362.207 x64`上的测试代码与结果：

```
NTKERNELAPI NTSTATUS ObReferenceObjectByName(IN PUNICODE_STRING ObjectName, IN ULONG Attributes, IN PACCESS_STATE PassedAccessState OPTIONAL, IN ACCESS_MASK DesiredAccess OPTIONAL, IN POBJECT_TYPE ObjectType, IN KPROCESSOR_MODE AccessMode, IN OUT PVOID ParseContext OPTIONAL, OUT PVOID *Object);

extern POBJECT_TYPE *IoDriverObjectType;



NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString)

{

	ULONG_PTR PagePointer = 0;

	ULONG_PTR BaseAddress = 0;

	ULONG SizeOfValidPPEPages = 0;

	if (GetPTEBase())

	{

		UNICODE_STRING UDrvName;

		PDRIVER_OBJECT DrvObj = 0;

		if (NT_SUCCESS(RtlInitUnicodeString(&UDrvName, L"\\Driver\\kbdclass")) && NT_SUCCESS(ObReferenceObjectByName(&UDrvName, OBJ_CASE_INSENSITIVE, 0, 0, *IoDriverObjectType, KernelMode, 0, (PVOID*)&DrvObj)))

		{

			ObDereferenceObject(DrvObj);

			PagePointer = AllocateSinglePXEDirectory(&BaseAddress, &SizeOfValidPPEPages);

			ULONG_PTR MappedKbdClassBase = FillPDEArrayForAllValidPPEs(PagePointer, BaseAddress, SizeOfValidPPEPages, (ULONG_PTR)DrvObj->DriverStart);

			if (MakeDirtyPage(MappedKbdClassBase)) *(PULONG)(MappedKbdClassBase + 4) = 0x78563412;

			FreeSinglePXEDirectory(PagePointer, BaseAddress);

		}

	}

	return STATUS_UNSUCCESSFUL;

}
```

![](https://bbs.kanxue.com/upload/attach/202010/805252_STPTFCV493MMK4E.jpg)

另外附上`Win10`的`_MMPTE_HARDWARE`以供参考：

```
typedef struct _MMPTE_HARDWARE

{

	ULONGLONG Valid : 1;

	ULONGLONG Dirty1 : 1;

	ULONGLONG Owner : 1;

	ULONGLONG WriteThrough : 1;

	ULONGLONG CacheDisable : 1;

	ULONGLONG Accessed : 1;

	ULONGLONG Dirty : 1;

	ULONGLONG LargePage : 1;

	ULONGLONG Global : 1;

	ULONGLONG CopyOnWrite : 1;

	ULONGLONG Unused : 1;

	ULONGLONG Write : 1;

	ULONGLONG PageFrameNumber : 36; //Vista SP0为28，Vista SP1--10通用36

	ULONGLONG ReservedForHardware : 4;

	ULONGLONG ReservedForSoftware : 4;

	ULONGLONG WsleAge : 4;

	ULONGLONG WsleProtection : 3;

	ULONGLONG NoExecute : 1;

} MMPTE_HARDWARE, *PMMPTE_HARDWARE;
```

总结一下，以上实现的代码大概类似弟中弟中弟版`MmBuildMdlForNonPagedPool`和`MmMapLockedPagesSpecifyCache`，不同的是本文的代码更多地相当于一份`512G`的虚拟地址空间物理内存的分布快照，这其中可能存在部分分页内存在快照过程中被放入物理内存而快照后再次被分回硬盘的情况，因此在映射地址空间使用`MmIsAddressValid`远远不如在原地址空间使用靠谱。

`10-7`更新：感谢`@syser`老铁的关于刷新`TLB`的建议，测试发现刷`TLB`的确不需要`KeFlushTb`，只要对要访问或修改的映射页面`invlpg`刷新单个`PTE`对应的`TLB`项即可。以上代码在`Vista SP0`至`Win10 18363`均测试有效。

[冰与火的战歌：`Windows`内核攻防实战高级班！从零到实战，融合`AI`与`Windows`内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 `2020-10-7 23:22` 被 `hhkqqs` 编辑，原因：
