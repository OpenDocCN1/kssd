
# [原创]读写硬盘的一种新模式（无视大部分还原哦）

ATAPI系列读写硬盘被大量穿透还原的恶意代码应用从而导致ATAPI读写磁盘已经不靠谱了![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)，所以需要一种新方法![](https://bbs.kanxue.com/view/img/face/28.gif)~

用IDA打开HAL我们发现一组新的导出call~不过有操作系统限制，而且使用起来有操作系统限制（不过随着win7,win8的普及应该没啥问题了）~![](https://bbs.kanxue.com/view/img/face/12.gif)

不过也是很犀利的~详情见代码![](https://bbs.kanxue.com/view/img/face/16.gif)：

```
typedef struct _X86BIOS_REGISTERS	// invented names
{
	ULONG Eax;
	ULONG Ecx;
	ULONG Edx;
	ULONG Ebx;
	ULONG Ebp;
	ULONG Esi;
	ULONG Edi;
	USHORT SegDs;
	USHORT SegEs;
} X86BIOS_REGISTERS, *PX86BIOS_REGISTERS;

NTHALAPI BOOLEAN x86BiosCall (ULONG, PX86BIOS_REGISTERS);

NTHALAPI NTSTATUS x86BiosAllocateBuffer (ULONG *, USHORT *, USHORT *);
NTHALAPI NTSTATUS x86BiosFreeBuffer (USHORT, USHORT);

NTHALAPI NTSTATUS x86BiosReadMemory (USHORT, USHORT, PVOID, ULONG);
NTHALAPI NTSTATUS x86BiosWriteMemory (USHORT, USHORT, PVOID, ULONG);

#pragma pack(1)
typedef struct _X86DISK_PACKET_
{
	USHORT Size;
	USHORT Sectors;
	USHORT Addr;
	USHORT Segment;
	ULONG64 StartSectorNumber;
	ULONG64 L_BufferAddr;
}X86DISK_PACKET,*PX86DISK_PACKET;
#pragma pack()

BOOL ReadDiskByInt13Ext(ULONG64 sector,USHORT numbers,PVOID *OutBuffer,UINT *nSize)
{
	ULONG cb = 0;
	BOOL bRet=FALSE;
	USHORT bufSeg=0,bufAddr=0;
	USHORT dpSeg=0,dpAddr=0;
	BOOL bBuf=FALSE;
	BOOL bDp =FALSE;
	NTSTATUS ns;
	X86DISK_PACKET dp;
	X86BIOS_REGISTERS regs;
	__try
	{
		if (!OutBuffer||!nSize)
			__leave;
		cb = numbers*512;//默认512一个sec这样看起来还行，其实应该好好计算的
		ns = x86BiosAllocateBuffer(&cb,&bufSeg,&bufAddr);
		if (!NT_SUCCESS(ns))
			__leave;
		bBuf =TRUE;
		cb = sizeof(X86DISK_PACKET);
		ns = x86BiosAllocateBuffer(&cb,&dpSeg,&dpAddr);
		if (!NT_SUCCESS(ns))
			__leave;
		bDp =TRUE;
		dp.Size = 0x10;
		dp.Sectors = numbers;
		dp.StartSectorNumber = sector;
		dp.Segment = bufSeg;
		dp.Addr = bufAddr;
		dp.L_BufferAddr=0;
		ns = x86BiosWriteMemory(dpSeg,dpAddr,&dp,sizeof(X86DISK_PACKET));
		if(!NT_SUCCESS(ns))
			__leave;
		RtlZeroMemory(&regs,sizeof(X86BIOS_REGISTERS));
		regs.Eax = 0x4200;//AH=0x42
		regs.Edx = 0x0080;//DL=0x80 //第一个磁盘
		regs.SegDs = dpSeg;
		regs.Esi=dpAddr;
		if (x86BiosCall(0x13,&regs))
		{
			UINT iSize = numbers*512;
			PVOID Buffer = ExAllocatePool(NonPagedPool,iSize);
			if (Buffer)
			{
				ns = x86BiosReadMemory(bufSeg,bufAddr,Buffer,iSize);
				if (NT_SUCCESS(ns))
				{
					*OutBuffer = Buffer;
					*nSize=iSize;
					bRet =TRUE;
				}
				else
				{
					ExFreePool(Buffer);
				}
			}
		}
	}
	__except(EXCEPTION_EXECUTE_HANDLER)
	{
		DbgPrint("Exception From Read");
	}
	if (bBuf)
	{
		x86BiosFreeBuffer(bufSeg,bufAddr);
	}
	if (bDp)
	{
		x86BiosFreeBuffer(dpSeg,dpAddr);
	}
	return bRet;
}

BOOL WriteDiskInt13Ext(ULONG64 Sector,PVOID InBuffer,INT nInBuffSize)
{
	ULONG cb = 0;
	BOOL bRet=FALSE;
	USHORT bufSeg=0,bufAddr=0;
	USHORT dpSeg=0,dpAddr=0;
	BOOL bBuf=FALSE;
	BOOL bDp =FALSE;
	NTSTATUS ns;
	X86DISK_PACKET dp;
	X86BIOS_REGISTERS regs;
	USHORT numbers =0;
	__try
	{
		if (!InBuffer||!nInBuffSize)
			__leave;
		numbers = nInBuffSize/512;
		cb = nInBuffSize;//默认512一个sec这样看起来还行，其实应该好好计算的
		ns = x86BiosAllocateBuffer(&cb,&bufSeg,&bufAddr);
		if (!NT_SUCCESS(ns))
			__leave;
		bBuf =TRUE;
		cb = sizeof(X86DISK_PACKET);
		ns = x86BiosAllocateBuffer(&cb,&dpSeg,&dpAddr);
		if (!NT_SUCCESS(ns))
			__leave;
		bDp =TRUE;
		dp.Size = 0x10;
		dp.Sectors = numbers;
		dp.StartSectorNumber = Sector;
		dp.Segment = bufSeg;
		dp.Addr = bufAddr;
		dp.L_BufferAddr=0;
		ns = x86BiosWriteMemory(bufSeg,bufAddr,InBuffer,nInBuffSize);
		ns = x86BiosWriteMemory(dpSeg,dpAddr,&dp,sizeof(X86DISK_PACKET));
		if(!NT_SUCCESS(ns))
			__leave;
		RtlZeroMemory(&regs,sizeof(X86BIOS_REGISTERS));
		regs.Eax = 0x4300;//AH=0x42
		regs.Edx = 0x0080;//DL=0x80 //第一个磁盘
		regs.SegDs = dpSeg;
		regs.Esi=dpAddr;
		if (x86BiosCall(0x13,&regs))
		{
			bRet =TRUE;
		}
	}
	__except(EXCEPTION_EXECUTE_HANDLER)
	{
		DbgPrint("Exception From Read");
	}
	if (bBuf)
	{
		x86BiosFreeBuffer(bufSeg,bufAddr);
	}
	if (bDp)
	{
		x86BiosFreeBuffer(dpSeg,dpAddr);
	}
	return bRet;
}
```

注意部分系统下要注册一个`KeRegisterBugCheckCallBack`

然后通过自己调用`BugCheck`在自己的`CallBack`里才能随心所欲的使用这个代码——囧~

最后，这个代码只是个POC真正想要可用于各位的伟大事业必须继续努力修改和做很多处理，亲~（伸手党必然被砍手）![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

求捐助，求赞助，求投资，有意者欢迎联系QQ:86879759

欲购买AntiGameProtect或UltraGameProtect或BypassGameProtect系列代码与产品也欢迎联系。

AntiGameProtect系列用于穿透驱动保护调试程序

UltraGameProtect系列用于保护程序不被各类调试器（包括滴水调试器，GoodDbg等）成功调试

BypassGameProtect系列用于穿透驱动保护进行各种操作（如读写内存，注入dll，模拟键盘鼠标，防内存检测，DLL隐藏，反CRC效验等）

qq技术扯淡群1：171797360

qq技术扯淡群2：1748876

yy技术扯淡频道：80252844

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
