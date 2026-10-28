
# [原创]浅谈一下驱动开发中的硬编码

驱动开发中硬编码是比较烦人的,不仅让别人看起来不怎么专业,自己看着心里也疙疙瘩瘩的不舒服

最常见的就是对于`EPROCESS`结构中`ImageFileName`的定位,不同系统下这个域的偏移是不同的,

常见做法就是得到`EPROCESS`之后根据系统的不同来加上不同的偏移,

或者也有在`PsInitialSystemProcess`指向的`system`进程的`EPROCESS`中搜索`system`字串进行定位的.

那么`Windows`真的没有为我们提供定位到这个域的函数吗?

我们可以看一下内核文件的导出表,因为进程线程相关函数都是`Ps`开头,我们看一下这里有没有线索

![](https://bbs.kanxue.com/upload/attach/201111/440388_2walf3bsr1f6qbx.jpg)

函数名称来看应该是与进程名相关的,再来到`IDA`中看一下

![](https://bbs.kanxue.com/upload/attach/201111/440388_nwqlae5oqvdfy7t.jpg)

<`xp`下的实现>

![](https://bbs.kanxue.com/upload/attach/201111/440388_gho44m1yyt280me.jpg)

<`win7`下的实现>

熟悉`EPROCESS`结构的童鞋应该都马上想到这个就是在`EPROCESS`地址加上了一个偏移量来得到了`ImageFileName`

这个函数虽然被导出了,不过`WDK`中并没有介绍它,我们使用的时候定义一下原型,配合`MmGetSystemRoutineAddress`就可以了

```
typedef PUCHAR	(__stdcall *sky_PsGetProcessImageFileName)(PEPROCESS pEprocess);

PUCHAR GetSpecialProcessImageFileName(PEPROCESS pEprocess)
{
	UNICODE_STRING						destString;
	sky_PsGetProcessImageFileName		CallPsGetProcessImageFileName = NULL;

	if ( MmIsAddressValidEx((PVOID)pEprocess) == VCS_INVALID ) return NULL;
	if ( IsValidObject(pEprocess,*PsProcessType) == FALSE ) return NULL;

	RtlInitUnicodeString(&destString,(PWCHAR)L"PsGetProcessImageFileName");
	CallPsGetProcessImageFileName = (sky_PsGetProcessImageFileName)MmGetSystemRoutineAddress(&destString);
	if (CallPsGetProcessImageFileName == NULL) return NULL;

	return CallPsGetProcessImageFileName(pEprocess);
}
```

类似的函数还有不少:

```
PsGetProcessDebugPort 取得指定EPROCESS的调试端口信息
typedef PVOID	(__stdcall *sky_PsGetProcessDebugPort)(PEPROCESS pEprocess);

PsGetProcessId 取得指定EPROCESS的进程Id
typedef ULONG	(__stdcall *sky_PsGetProcessId)(PEPROCESS pEprocess);

PsGetProcessInheritedFromUniqueProcessId 取得指定EPROCESS的父进程Id
typedef ULONG	(__stdcall *sky_PsGetProcessInheritedFromUniqueProcessId)(PEPROCESS pEprocess);

PsGetProcessJob 取得指定EPROCESS的Job信息
typedef PVOID	(__stdcall *sky_PsGetProcessJob)(PEPROCESS pEprocess);

PsGetProcessPeb 取得指定EPROCESS的Peb地址
typedef PPEB	(__stdcall *sky_PsGetProcessPeb)(PEPROCESS pEprocess);

PsGetProcessWin32Process 取得指定EPROCESS的w32process结构地址 <GUI>
typedef PVOID	(__stdcall *sky_PsGetProcessWin32Process)(PEPROCESS pEprocess);

PsGetThreadTeb 取得指定ETHREAD的Teb地址
typedef PVOID	(__stdcall *sky_PsGetThreadTeb)(PETHREAD pEthread);

PsGetThreadId 取得指定ETHREAD的线程Id
typedef ULONG	(__stdcall *sky_PsGetThreadId)(PETHREAD pEthread);

PsGetThreadProcessId 取得指定ETHREAD的进程Id
typedef ULONG	(__stdcall *sky_PsGetThreadProcessId)(PETHREAD pEthread);
```

这里我只是把自己可能用到的都提取出来了,更多的函数大家可以在`IDA`中去看一下~

这些函数的使用方法与`PsGetProcessImageFileName`是一样的,自己可以封装一下

********************************************************************************

再简单说一下关于`SSDT`的服务号吧,这些服务号不同的系统版本也不尽相同

而且有很多未导出的函数,所以定位它们在`SSDT`表中的位置时有点烦

但是所有的`SSDT`都是在`ntdll`中被导出的,我们可以通过在驱动中映射`ntdll`并解析`PE`输出表的方法来找到所有`SSDT`的服务号

因为`Ntdll`中的所有`SSDT`函数形式都类似这样

![](https://bbs.kanxue.com/upload/attach/201111/440388_y8e5o95p4v3du81.jpg)

在输出表中得到函数地址之后越过首条`mov`指令的操作码,后面的双字就是服务号了

下面对驱动中映射文件进行了简单封装:

```
ULONG MapSpecialModule(PWCHAR pModulePath, PHANDLE ReturnModuleSection)
{
	UNICODE_STRING			destString;
	OBJECT_ATTRIBUTES		objAttr;
	NTSTATUS				status = STATUS_SUCCESS;
	HANDLE					hFile = NULL;
	IO_STATUS_BLOCK			ioStatusBlock;
	HANDLE					Section = NULL;
	PVOID					pvSectionBase = NULL;
	ULONG					ulSectionSize = 0;
	ULONG					Result = 0;

	if ( pModulePath == NULL ) return 0;
	if ( ReturnModuleSection == NULL ) return 0;

	RtlInitUnicodeString(&destString,pModulePath);
	InitializeObjectAttributes(&objAttr,&destString,OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE,NULL,NULL);

	status = ZwCreateFile(&hFile,FILE_READ_DATA | SYNCHRONIZE,&objAttr,&ioStatusBlock,NULL,
		FILE_ATTRIBUTE_NORMAL,FILE_SHARE_READ,FILE_OPEN,	//存在则打开 不存在则返回错误码
		FILE_SYNCHRONOUS_IO_NONALERT,NULL,0);
	if ( !NT_SUCCESS(status) ) return 0;
	if ( !NT_SUCCESS(ioStatusBlock.Status) ) return 0;

#define SEC_COMMIT        0x8000000
#define SEC_IMAGE		  0x1000000
	InitializeObjectAttributes(&objAttr,NULL,OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE,NULL,NULL);

	status = ZwCreateSection(&Section,SECTION_QUERY,&objAttr,NULL,PAGE_READONLY,SEC_COMMIT,hFile);
	if (!NT_SUCCESS(status)) goto __exit1;

	status = ZwMapViewOfSection(Section,
								ZwCurrentProcess(),
								&pvSectionBase,		//接受映射的基址初始化为NULL
								0,
								PAGE_SIZE,			//这里指定起始映射多少字节
								0,					//section起始至这里的偏移
								&ulSectionSize,		//初始化为0
								ViewUnmap,
								MEM_TOP_DOWN,
								PAGE_READONLY);
	if ( !NT_SUCCESS(status) )
	{
		ZwClose(Section);
		Section = NULL;
		goto __exit1;
	}

	*ReturnModuleSection = Section;
	Result = (ULONG)pvSectionBase;

__exit1:
	if ( hFile != NULL ) ZwClose(hFile);
	hFile = NULL;
	return Result;
}

BOOLEAN UnMapSpecialModule(ULONG SectionBase, HANDLE hModuleSection)
{
	if ( hModuleSection == NULL ) return FALSE;
	if ( SectionBase == 0 ) return FALSE;

	ZwUnmapViewOfSection(hModuleSection,(PVOID)SectionBase);
	ZwClose(hModuleSection);
	hModuleSection = 0;
	return TRUE;
}

HANDLE globe_NtdllModuleSection = NULL;
ULONG MapNtdllModule()
{
	PWCHAR	pNtdllPath = NULL;
	ULONG	Result = 0;

	if ( GetNtdllModulePath(&pNtdllPath) == FALSE ) return 0;
	Result = MapSpecialModule(pNtdllPath,&globe_NtdllModuleSection);

	if ( pNtdllPath != NULL ) ExFreePool(pNtdllPath);
	pNtdllPath = NULL;
	return Result;
}

BOOLEAN UnMapNtdllModule(ULONG SectionBase)
{
	return UnMapSpecialModule(SectionBase,globe_NtdllModuleSection);
}
```

映射完模块之后我们在其输出表中搜索指定函数名称就行了,这里需要自己实现一个`GetProcAddress`

因为`MmGetSystemRoutineAddress`只能得到内核模块以及`hal`模块中的函数信息

对于`GetProcAddress`的实现,网上已有很多,我就不重发了,大家搜索一下吧~~

`QQ`: `2575439022`欢迎技术交流

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [expt.jpg](javascript:void(0);)

  （89.74kb，578次下载）

- [xp.jpg](javascript:void(0);)

  （40.04kb，579次下载）

- [7.jpg](javascript:void(0);)

  （40.99kb，578次下载）

- [ntdll.jpg](javascript:void(0);)

  （56.77kb，585次下载）
