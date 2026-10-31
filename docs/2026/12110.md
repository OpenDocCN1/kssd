
# [分享]64位驱动取当前进程的全路径

```c
#if DBG

#define dprintf DbgPrint

#else

#define dprintf

#endif

PCWSTR GetCurrentProcessPathName(void) 
{
	UNICODE_STRING imagName={0};
	RTL_OSVERSIONINFOW os={0};
	ULONG_PTR uProcessPtr;
	ULONG_PTR uImagNameAddr=NULL ;
	ULONG uPebOffset=0;
	ULONG uParamOffset=0x10;
	ULONG uImagNameOffset=0x3c;

	RtlGetVersion (&os);
	dprintf ("MajorVer=%d\tMinorVer=%d\tBuildNumber=%d\r\n",
		os.dwMajorVersion,os.dwMinorVersion,os.dwBuildNumber);
	if(KeGetCurrentIrql() != PASSIVE_LEVEL)
		return NULL;

	if(os.dwMajorVersion==5)
	{
		switch(os.dwMinorVersion)
		{
		case 1://xp sp3
			uPebOffset=0x1b0;
			break;
		case 2://2003 sp2
			uPebOffset=0x1a0;
			break;
		}
	}
	else if(os.dwMajorVersion==6)
	{
		switch(os.dwMinorVersion)
		{
		case 0://2008
			break;
		case 1:
#ifdef _AMD64_
			//win7x64 sp1, 2008R2x64 sp1
			uPebOffset=0x338;
			uParamOffset =0x20;
			uImagNameOffset=0x68;
#else
			//win7x32 sp1
			uPebOffset=0x1a8;
#endif
		}
	}	
	if(uPebOffset==0)
	{
		dprintf ("this function do not supported current os.\r\n");
		return NULL ;
	}
	uProcessPtr =(ULONG_PTR)PsGetCurrentProcess();

	uImagNameAddr=(uProcessPtr+uPebOffset);	
    if((uImagNameAddr =*(ULONG_PTR*)uImagNameAddr)==0)return NULL ;

	uImagNameAddr+=uParamOffset;
	if((uImagNameAddr =*(ULONG_PTR*)uImagNameAddr)==0)return NULL;

	uImagNameAddr+=uImagNameOffset;	
	if((uImagNameAddr =*(ULONG_PTR*)uImagNameAddr)==0)return NULL ;		
	dprintf ("Process full path name: %ws\r\n",(PCWSTR)uImagNameAddr );	
	return (PCWSTR)uImagNameAddr;

}

//64位只在win2008 R2 sp1 下测试
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
