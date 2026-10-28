
```
;===============================================================================

;use ZwQueryVirtualMemory to enum Modules of a process

;subsystem:console

;OS Platform:tested on Windows XP Professional simplified with Service Pack 3

;轩辕小聪 cafK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6&6K9h3y4G2L8X3M7J5x3o6l9%4

;

;release notes:

;2009.04.11

;完全改写CreateDeviceList和ConvertDeviceStr两个函数，Ring3下使用QueryDosDevice，

;通过对所有的DOS符号连接获取其对应设备名，与得到的模块路径一一对照，克服了上一版本

;只能转换A-Z盘符的问题。

;

;2009.04.06 

;改写ConvertDeviceStr函数，实现A-Z盘符的转换，以支持非本地磁盘盘符的获取。

;遍历时只对具有PAGE_EXECUTE属性的内存查找MemorySectionName。

;如将被注释掉的四处代码还原，可过滤掉nls文件的显示。

;

;2009.04.05 

;第一个版本

;

;===============================================================================

include \masm32\include\masm32rt.inc

include \masm32\include\w2k\native.inc

includelib \masm32\lib\masm32.lib

include \masm32\macros\macros.asm

include \masm32\include\advapi32.inc

includelib \masm32\lib\advapi32.lib



.data

PID dd 0

Base dd 0

hProcess dd 0

DataLength dd 0

lpszPID db 120 dup(0)

lpoutdata db 300h dup(0)

lpFileName db 120h dup(0)

lpDosList dd 0

lpDeviceList dd 0

lpDosBuffer dd 0

lpDeviceBuffer dd 0

.code



CreateDeviceList proto



CreateDeviceList proc uses esi edi ebx edx

	local	num

	local	lpBuffer

	local	BufferLen



	mov	edi, 0

	xor	esi, esi

	.repeat	

		.if	esi!=0

			hfree(esi)	

		.endif

		add	edi, 1000h		

		mov	esi,halloc(edi)

		invoke	QueryDosDevice, NULL, esi, edi

	.until	eax!=0

	mov	lpDosBuffer, esi	

	mov	num, 0

	xor	eax, eax

	.repeat	

		inc	num

		invoke	crt_strlen, esi		

		lea	esi, [esi+eax+1]

	.until eax==0

	dec	num

	mov	eax, num

	lea	eax, [eax*4+4]

	mov	lpDosList, halloc(eax)

	mov	edi, lpDosList

	mov	eax, num

	mov	[edi], eax

	add	edi, 4

	xor	ebx, ebx

	mov	esi, lpDosBuffer

	.repeat	

		inc	ebx

		mov	[edi], esi

		add	edi, 4

		invoke	crt_strlen, esi		

		lea	esi, [esi+eax+1]

	.until ebx==num		

	mov	eax, num

	shl	eax, 8

	mov	ebx, num

	shl	ebx, 2

	add	eax, ebx

	push	eax

	mov	esi, halloc(eax)	

	mov	lpBuffer, esi

	pop	eax

	invoke	RtlZeroMemory, lpBuffer, eax

	mov	eax, num

	lea	eax, [eax*4+4]

	mov	lpDeviceList, halloc(eax)

	mov	edx, lpDeviceList

	mov	eax, num

	mov	[edx], eax

	mov	edi, lpDosList

	add	edi, 4

	xor	ebx, ebx

	mov	BufferLen, 0

	.repeat

		inc	ebx

		mov	ecx, [edi]

		invoke	QueryDosDevice, ecx, esi, MAX_PATH

		.if	eax!=0

			invoke	crt_strlen, esi

			inc	eax

			add	BufferLen, eax	

		.endif		

		add	edi, 4

		lea	esi, [esi+MAX_PATH]	

	.until	ebx==num

	inc	BufferLen

	mov	lpDeviceBuffer, halloc(BufferLen)

	invoke	RtlZeroMemory, lpDeviceBuffer, BufferLen

	mov	edi, lpDeviceBuffer

	mov	edx, lpDeviceList

	add	edx, 4

	mov	esi, lpBuffer

	xor	ebx, ebx

	.repeat

		inc	ebx

		push	edx

		invoke	crt_strlen, esi

		pop	edx

		.if	eax!=0	

			inc	eax

			push	eax

			push	edx

			invoke	crt_strncpy, edi, esi, eax

			pop	edx

			mov	[edx], edi

			pop	eax

			add	edi, eax

		.else

			mov	[edx], eax

		.endif	

		add	edx, 4	

		lea	esi, [esi+MAX_PATH]	

	.until	ebx==num

	hfree(lpBuffer)	

	mov	eax, 1		

	ret	

CreateDeviceList endp



ConvertDeviceStr proto :DWORD, :DWORD



;=====================================================

;把得到的文件名中的盘符翻成平常的C、D……并将字符串改

;成输出所需的格式

;=====================================================

ConvertDeviceStr proc uses esi edi ebx lpSource:DWORD, ImageBase:DWORD



	local	lptmp[MAX_PATH]:byte

	local	totalnum

	local	match

	local	lpmatchDev

	local	DevLen

	local	NameLen

	lea	edi, lptmp

	xor	eax, eax

	mov	match, eax

	mov	lpmatchDev, eax

	mov	NameLen, eax

	mov	DevLen, eax

	mov	ecx, MAX_PATH

	rep	stosb

	mov	esi, lpDeviceList

	mov	eax, [esi]

	mov	totalnum, eax

	add	esi, 4

	xor	ebx, ebx

	.repeat

		inc	ebx

		push	esi

		mov	edi, [esi]

		.if	edi!=0

			invoke	crt_strstr, lpSource, edi

			.if	eax==lpSource 

				.if	match==0

					mov	match, 1

				.endif	

				mov	eax, ebx

				lea	eax, [eax*4]

				mov	esi, lpDosList

				add	esi, eax

				mov	esi, [esi]

				invoke	crt_strlen, esi

				.if	NameLen==0 || NameLen>eax

					mov	NameLen, eax

					invoke	crt_strlen, edi

					mov	DevLen, eax

					mov	lpmatchDev, esi

				.endif	

			.endif	

		.endif	

		pop	esi

		add	esi, 4

	.until	ebx==totalnum	



	.if	match==0

		invoke	crt_strncpy, addr lptmp, lpSource, MAX_PATH

	.else	

		mov	ebx, lpmatchDev

		mov	eax, DevLen

		mov	esi, lpSource

		lea	esi, [esi+eax]

		invoke	crt_sprintf, addr lptmp, CTXT('%s%s'), ebx, esi

	.endif	

	invoke	crt_sprintf, lpSource, CTXT('%08X   %s',0dh,0ah), ImageBase, addr lptmp

	ret

			

ConvertDeviceStr endp



CleanUp proto



CleanUp proc uses esi edi ebx



	hfree(lpDosBuffer)

	hfree(lpDosList)

	hfree(lpDeviceList)

	hfree(lpDeviceBuffer)

	ret

CleanUp endp



DebugPrivilege   PROTO :DWORD



;=====================================================

;提SeDebugPrivilege，copy from network

;=====================================================



DebugPrivilege    proc uses esi edi ebx dwEnbled



	local	hToken

	local	tmpLuid:LUID,tkp:TOKEN_PRIVILEGES

    

	invoke	GetCurrentProcess

	lea	ebx,hToken

	invoke	OpenProcessToken, eax, TOKEN_ADJUST_PRIVILEGES Or TOKEN_QUERY, ebx

	invoke	LookupPrivilegeValue, NULL, CTXT('SeDebugPrivilege'), addr tmpLuid

	mov	tkp.PrivilegeCount,1

	push	tmpLuid.LowPart

	pop	tkp.Privileges[0].Luid.LowPart

	push	tmpLuid.HighPart

	pop	tkp.Privileges[0].Luid.HighPart

	.if	dwEnbled

		mov	tkp.Privileges[0].Attributes, SE_PRIVILEGE_ENABLED

	.else	

		mov	tkp.Privileges[0].Attributes,NULL

	.endif	

	invoke	AdjustTokenPrivileges, hToken, FALSE, addr tkp, sizeof TOKEN_PRIVILEGES, NULL, NULL

	invoke	GetLastError

	.if	eax == ERROR_SUCCESS

		push	TRUE

	.else	

		push	FALSE

	.endif	       

	invoke	CloseHandle,hToken

	pop	eax

	ret

	

DebugPrivilege    endp



Start        proc uses esi edi ebx



	invoke	GetCL, 1, offset lpszPID	

	invoke	crt_sscanf, offset lpszPID, CTXT('%d'), offset PID

	.if	PID==0

		invoke	StdOut, CTXT('Invaild PID.')

		ret	

	.endif	

	invoke	DebugPrivilege, 1

	.if	eax==FALSE

		invoke	StdOut, CTXT('Enable SeDebugPrivilege failed.')

		ret

	.endif	

	invoke	OpenProcess, PROCESS_QUERY_INFORMATION, 0, PID

	.if	eax==0

		invoke	StdOut, CTXT('OpenProcess failed.')

		ret

	.endif	

	mov	hProcess, eax

	invoke	GetModuleHandle, CTXT('ntdll.dll')

	mov	esi, eax

	invoke	GetProcAddress, esi, CTXT('ZwQueryVirtualMemory')

	mov	edi, eax

	invoke	StdOut, CTXT('ModuleBase ImageFileName',0dh,0ah)

	invoke	CreateDeviceList

	.repeat	

		push	offset DataLength

		push	300h

		push	offset lpoutdata

		push	MemoryBasicInformation

		push	Base

		push	hProcess

		call	edi

		mov	esi, offset lpoutdata

		;mov	edx, [esi+18h]

		assume	esi:ptr MEMORY_BASIC_INFORMATION

		;mov	ebx, [esi].AllocationProtect

		;and	ebx, WSLE_PAGE_EXECUTE

		mov	ecx, [esi].Protect

		and	ecx, WSLE_PAGE_EXECUTE

		mov	esi, [esi].AllocationBase

		assume	esi:nothing

		.if	eax==0 && esi==Base && ecx!=0 ;&& edx == SEC_IMAGE && ebx==0		

			push	offset DataLength

			push	300h

			push	offset lpoutdata

			push	MemorySectionName

			push	Base

			push	hProcess

			call	edi	

			.if	eax==0

				mov	esi, offset lpoutdata

				assume	esi:ptr MEMORY_SECTION_NAME

				movzx	eax, [esi].SectionFileName._Length

				.if	eax!=0

					mov	esi, [esi].SectionFileName.Buffer

					assume	esi:nothing

					invoke	crt_sprintf, offset lpFileName, CTXT('%ws'),esi

					invoke	ConvertDeviceStr, offset lpFileName, Base

					invoke	StdOut, offset lpFileName

				.endif					

			.endif

		.endif	

		mov	eax, Base

		add	eax, 1000h

		mov	Base, eax

	.until	Base==80000000h

	invoke	CleanUp

	ret



Start endp



end Start
```

结果演示：以下是一个`svchost.exe`进程的内存模块：

![](https://bbs.kanxue.com/upload/attach/200904/64132_v4ro2aft1r57gkc.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`result.JPG`](javascript:void(0);)

  （116.88kb，912次下载）

- [`Query.rar`](javascript:void(0);)
