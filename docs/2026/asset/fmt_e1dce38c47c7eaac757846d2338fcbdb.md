
```
PAGE:005709B3 ; NTSTATUS __stdcall PsSetLoadImageNotifyRoutine(PLOAD_IMAGE_NOTIFY_ROUTINE NotifyRoutine)

PAGE:005709B3                 public PsSetLoadImageNotifyRoutine

PAGE:005709B3 PsSetLoadImageNotifyRoutine proc near   ; CODE XREF: sub_580241+215↓p

PAGE:005709B3

PAGE:005709B3 NotifyRoutine   = dword ptr  8

PAGE:005709B3

PAGE:005709B3                 mov     edi, edi

PAGE:005709B5                 push    ebp

PAGE:005709B6                 mov     ebp, esp

PAGE:005709B8                 push    ebx

PAGE:005709B9                 push    esi

PAGE:005709BA                 push    edi

PAGE:005709BB                 xor     edi, edi        ; edi清0

PAGE:005709BD                 push    edi             ; 将0入栈

PAGE:005709BE                 push    [ebp+NotifyRoutine] ; 将函数地址入栈

PAGE:005709C1                 call    AllocateAssign  ; 分配12字节的内存，低4位和高4位清0，中间4位存放函数地址

PAGE:005709C6                 mov     ebx, eax        ; 申请到的内存地址赋给ebx

PAGE:005709C8                 cmp     ebx, edi        ; 判断是否为0，也就是是否分配失败

PAGE:005709CA                 jz      short loc_5709F1 ; 如果内存分配失败则退出

PAGE:005709CC                 mov     esi, offset LoadImageFuncArray ; 将一个数组地址赋给esi

PAGE:005709D1

PAGE:005709D1 loc_5709D1:                             ; CODE XREF: PsSetLoadImageNotifyRoutine+36↓j

PAGE:005709D1                 push    0               ; 将0入栈

PAGE:005709D3                 mov     ecx, ebx        ; 将分配的地址赋给ecx

PAGE:005709D5                 mov     eax, esi        ; 将数组地址赋给eax

PAGE:005709D7                 call    SetArray

PAGE:005709DC                 test    al, al

PAGE:005709DE                 jnz     short loc_5709FD ; 判断返回值是否为0，不为0则执行成功，跳转

PAGE:005709E0                 add     edi, 4          ; edi加4

PAGE:005709E3                 add     esi, 4          ; esi,也就是数组地址+4,所以这是在取数组的下一个元素

PAGE:005709E6                 cmp     edi, 20h        ; 判断edi是否小于0x20，小于则跳转过去执行函数

PAGE:005709E9                 jb      short loc_5709D1 ; 根据前面的清0操作知道这里是为了保证循环调用8次，所以可以判断这个数组中最多保存7个地址

PAGE:005709EB                 push    ebx             ; Buffer

PAGE:005709EC                 call    FreeAllocate    ; 否则函数执行失败，释放申请到的内存

PAGE:005709F1

PAGE:005709F1 loc_5709F1:                             ; CODE XREF: PsSetLoadImageNotifyRoutine+17↑j

PAGE:005709F1                 mov     eax, STATUS_INSUFFICIENT_RESOURCES ; 赋值失败的返回值

PAGE:005709F6

PAGE:005709F6 loc_5709F6:                             ; CODE XREF: PsSetLoadImageNotifyRoutine+6B↓j

PAGE:005709F6                 pop     edi

PAGE:005709F7                 pop     esi

PAGE:005709F8                 pop     ebx

PAGE:005709F9                 pop     ebp

PAGE:005709FA                 retn    4

PAGE:005709FD ; ---------------------------------------------------------------------------

PAGE:005709FD

PAGE:005709FD loc_5709FD:                             ; CODE XREF: PsSetLoadImageNotifyRoutine+2B↑j

PAGE:005709FD                 xor     ecx, ecx

PAGE:005709FF                 mov     eax, offset unk_542BA0

PAGE:00570A04                 inc     ecx

PAGE:00570A05                 lock xadd [eax], ecx

PAGE:00570A09                 mov     eax, dword_542B78

PAGE:00570A0E                 test    al, 1

PAGE:00570A10                 jnz     short loc_570A1C

PAGE:00570A12                 mov     eax, offset dword_542B78

PAGE:00570A17                 lock bts dword ptr [eax], 0

PAGE:00570A1C

PAGE:00570A1C loc_570A1C:                             ; CODE XREF: PsSetLoadImageNotifyRoutine+5D↑j

PAGE:00570A1C                 xor     eax, eax

PAGE:00570A1E                 jmp     short loc_5709F6

PAGE:00570A1E PsSetLoadImageNotifyRoutine endp
```

基本上和这篇一样[通过对`PsSetCreateProcessNotifyRoutineEx`的逆向分析得出的结果来实现反进程监控](https://bbs.pediy.com/thread-269831.htm)，那这篇就不再详细说，唯一的区别就是这里的数组的大小是8。

## 2.反模块监控

接下来就根据下图选择`0x7425BE`作为硬编码来查找模块数组

![](https://bbs.kanxue.com/upload/attach/202110/835440_S4NYGSYUEMSJJ66.jpg)

得出如下的反模块监控代码

```c
#include <ntifs.h>

#include <ntimage.h>



VOID DriverUnload(IN PDRIVER_OBJECT driverObject);

PULONG GetImageArray();



NTSTATUS DriverEntry(IN PDRIVER_OBJECT driverObject, IN PUNICODE_STRING registryPath)

{

	NTSTATUS status = STATUS_SUCCESS;

	PULONG pImageArray = NULL;

	PULONG pFuncAddr = NULL;

	ULONG i = 0;



	pImageArray = GetImageArray();

	if (pImageArray)

	{

		for (i = 0; i < 8; i++)

		{

			if (pImageArray[i] & ~0x7)

			{

				pFuncAddr = (PULONG)(pImageArray[i] & ~0x7 + 4);

				status = PsRemoveLoadImageNotifyRoutine((PLOAD_IMAGE_NOTIFY_ROUTINE)*pFuncAddr);

				if (NT_SUCCESS(status))

				{

					DbgPrint("模块监控删除成功 模块地址:0x%X\r\n", *pFuncAddr);

				}

			}

		}

	}

	

exit:

	driverObject->DriverUnload = DriverUnload;

	return STATUS_SUCCESS;

}



VOID DriverUnload(IN PDRIVER_OBJECT driverObject)

{

	DbgPrint("驱动卸载完成\r\n");

}



PULONG GetImageArray()

{

	PULONG pImageArray = NULL;

	//要获取的函数地址的函数名

	UNICODE_STRING uStrFuncName = RTL_CONSTANT_STRING(L"PsSetLoadImageNotifyRoutine");

	PUCHAR pPsSetCreateThreadNotifyRoutine = NULL;



	//获取函数地址

	pPsSetCreateThreadNotifyRoutine = (PUCHAR)MmGetSystemRoutineAddress(&uStrFuncName);

	while (*pPsSetCreateThreadNotifyRoutine != 0xC2)

	{

		if (*pPsSetCreateThreadNotifyRoutine == 0x74 && 

			*(pPsSetCreateThreadNotifyRoutine + 1) == 0x25 && 

			*(pPsSetCreateThreadNotifyRoutine + 2) == 0xBE)

		{

			pImageArray = (PULONG)*(PULONG)(pPsSetCreateThreadNotifyRoutine + 3);

			break;

		}

		pPsSetCreateThreadNotifyRoutine++;

	}



	return pImageArray;

}
```

## 3.运行截图

首先在驱动卸载前，可以看到所有模块的加载都会监控到

![](https://bbs.kanxue.com/upload/attach/202110/835440_2DN5R39DQFY9UY5.jpg)

接下来运行反监控代码可以看到模块的加载就不再被监控到

![](https://bbs.kanxue.com/upload/attach/202110/835440_HD6W3UUMSGCBVEY.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-11-30 10:00 被1900编辑，原因：

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
