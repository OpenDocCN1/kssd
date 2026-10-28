
```c
PVOID CheckFixHookedBaseAddress(PVOID);

PVOID GetNativeFunctionBaseAddress(PCWSTR);

DWORD GetFixOpcodeByLength(PVOID,LPVOID,LPVOID,DWORD);

DWORD SetHookAPIHead(LPVOID,PVOID,LPVOID,LPVOID,BYTE);

DWORD FixOpcodeForBackup(LPVOID,LPVOID,DWORD);
```

### 模块名字：`PVOID CheckFixHookedBaseAddress(PVOID lpBaseAddress)`

**模块功能**：对指定的基址试图寻找更合适的HOOK入口

**返回数值**：返回合适的HOOK入口

**参数说明**：
参数名 | 输入/输出 | 参数说明
`lpBaseAddress` | `IN` | 正常的需要优化的HOOK基址

```c
PVOID CheckFixHookedBaseAddress(PVOID lpBaseAddress)
{
	__asm
	{	
		mov		esi, lpBaseAddress
		mov		edi, esi
		mov		ecx, 2					; 检测前2条指令是否含直接跳转, 是则跟进之
	CheckFixHookedBaseAddress_next:		; 支持更深层次的Inline HOOK
		push	ecx
		push	eax
		push	esp
		push	esi
		call	LDE32					; LDE32 会修改EDX, ECX, EAX,需自己手动保护其值 	
		pop		edx
		pop		ecx
		cmp		edx, 6
		je		CheckFixHookedBaseAddress_jmp_mem
		cmp		edx, 5
		je		CheckFixHookedBaseAddress_jmp_imm
	CheckFixHookedBaseAddress_jmp_continue:
		add		esi, edx
		loop	CheckFixHookedBaseAddress_next
		jmp		CheckFixHookedBaseAddress_quit
	CheckFixHookedBaseAddress_jmp_imm:
		lodsb
		dec		edx
		cmp		al, 0xe9
		jne		CheckFixHookedBaseAddress_jmp_continue
		lodsd
		add		eax, esi				; 需修正跳转地址
		jmp		CheckFixHookedBaseAddress_new_addr
	CheckFixHookedBaseAddress_jmp_mem:
		lodsw
		cmp		ax, 0x25ff
		jne		CheckFixHookedBaseAddress_quit
		lodsd
		mov		eax, dword ptr [eax]
	CheckFixHookedBaseAddress_new_addr:
		push	eax
		pop		lpBaseAddress
	CheckFixHookedBaseAddress_quit:
	}
	return lpBaseAddress;
}
```

```c
PVOID GetNativeFunctionBaseAddress(IN PCWSTR pusNativeKeJmpFunctionName)
{
	UNICODE_STRING usFunctionName;
	RtlInitUnicodeString(&usFunctionName, pusNativeKeJmpFunctionName);
	return CheckFixHookedBaseAddress(MmGetSystemRoutineAddress(&usFunctionName));
}
```

### 模块名字：`DWORD SetHookAPIHead(LPVOID lpOpcode,PVOID lpBuffer,LPVOID lpParam, LPVOID lpHookAPI,BYTE bAPICount)`

**模块功能**：设置被Inline HOOK的API头

**返回数值**：返回实际设置Inline HOOK的API头指令串长度

**参数说明**：
参数名 | 输入/输出 | 参数说明
`lpOpcode` | `IN` | 需要处理的操作码基址(即原API的EPO)
`lpBuffer` | `OUT` | 接受Inline HOOK的API头指令串的本地缓冲区地址
`lpParam` | `IN` | 远程参数缓冲区地址,(没有可以为NULL)
`lpHookAPI` | `IN` | HOOK API代码所存放的远程参数缓冲区地址
`bAPICount` | `IN` | 被Inline HOOK的原API需要传入的实际参数数目

**更新**：
2008-05-13: 自动生成最优化HOOK头, 增加对EAX的保存,使之支持更深层次的Inline HOOK

```c
DWORD SetHookAPIHead(LPVOID lpOpcode,PVOID lpBuffer,LPVOID lpParam,LPVOID lpHookAPI,BYTE bAPICount)
{	LARGE_INTEGER dwSeed;
	#if WIN32_VERSION
	QueryPerformanceCounter(&dwSeed);
	#else
	KeQueryPerformanceCounter(&dwSeed);
	#endif
__asm
	{	
		mov		ecx, dwSeed.HighPart
		mov		eax, dwSeed.LowPart
		add		ecx, eax
		ror		ecx, cl				; 产生随机数
		mov		eax, ecx
	;===============================================================================	
		push	lpHookAPI			; 开始填写可变参数
		push	lpParam
		lea		edx, SHQ_HookParam
		pop		dword ptr [edx]	
		xor		dword ptr [edx], eax
		lea		edx, SHQ_HookProc
		pop		dword ptr [edx]	
		xor		dword ptr [edx], eax ; 用随机密钥加密两个地址参数,加强检测难度
		cld
		lea		edi, SHQ_AddressKey
		stosd	
		lea		edi, SHQ_ClsESP
		mov		al, bAPICount
		shl		al, 2
		stosb					
		lea		edi, SHQ_FuckESP
		stosb						; 填写可变参数完毕
		call	SHQ_SaveData
	;===============================================================================
	;###############################################################################
		mov     edi, edi		; 8B FF	
		push    ebp				; 55	
		mov     ebp, esp		; 8B EC 伪扮入口标志,加强检测难度 ★★★ 	
		pop		ebp				; 保存EBP (伪装时必需)
		xchg	eax, dword ptr [esp] ; 保存返回地址(这个很重要), 同时入栈保存EAX
		push    ecx
		push    edx				; 保护EBX, ECX,可以兼容fastcall api	★★★★
	SHQ_HookParam_Fix:	
		_emit	0x68
	SHQ_HookParam:	
		DD_ADRESS				; push    lpParam	; 传入HOOK参数
		_emit	0x68			; 不使用这些, 其实这样更省空间
	SHQ_AddressKey:				; 栈中解析地址,加强检测难度 ★★★★★
		DD_ADRESS				; push	dword ptr [SHQ_AddressKey]			
		pop		ecx				; 取得解码Key
	SHQ_HookParam_Fix_Sec:	
		_emit	0x31			; 地址解码
		_emit	0x0c
		_emit	0x24			; xor   dword ptr [esp], ecx ★★★★★	
		push	eax				; 修正返回地址
	SHQ_BackAddress:
		_emit	0x83
		_emit	0xEC			; 平衡堆栈  ●●●●●●●●
	SHQ_FuckESP:				; 制造假象申请局部栈空间的同时,能保留压入的参数共HOOK使用
		_emit	0x00			; sub     esp, x
	SHQ_FuckESP_Fix:
		_emit	0x68
	SHQ_HookProc:
		DD_ADRESS				; push	eax, lpHookAPI	; 设置真正的跳转地址
		_emit	0x31			; 地址解码
		_emit	0x0c
		_emit	0x24			; xor   dword ptr [esp], ecx ★★★★★
	SHQ_HookProc_Fix:
		push	ebp				; 模拟原API返回前恢复原EBP (不仅实现了跳转,还恢复了EBP)
		mov		ebp, esp		; 将HOOK代码做成正常调用退出格式,加强检测难度
		_emit	0x0c9			; leave 由于无法编译通过,只好采用机器码(WIN32时却可以)
		_emit	0x0c2			; ret	x  =>	pop	eip
	SHQ_ClsESP:
		DW_DATA					; x
	;###############################################################################
	SHQ_SaveData:				; 保存HOOK 头数据
		pop		ecx
    ;--------------------------------------------------------------------------------
		mov		edi, lpOpcode					; 生产最优化HOOK头,指令实时修正保存
		test	edi, edi
		je		SHQ_SaveData_Exit
		mov		esi, edi
		cld
		xor		edx, edx						; EDX 作为属否含有PROC的标志(默认为0,表无)
		lodsw
		cmp		ax, 0FF8Bh						; 8BFF	    mov     edi, edi
		jne		SHQ_SaveData_Nt					; 据说NT以上MS才添加了这个,方便大家HOOK
		or		dl, 01b							; 设置标志: -> XP
	SHQ_SaveData_Nt:
		lodsb									; 55		push    ebp				
		cmp		al, 55h
		je		SHQ_SaveData_Proc
		lodsw 
		jmp		SHQ_SaveData_Proc_Head
	SHQ_SaveData_Proc:
		lodsw	
		cmp		ax,	0EC8Bh						; 8BEC		mov     ebp, esp
		jne		SHQ_SaveData_Proc_Head
		sub		esi, 3+1						; 有PROC时,需要 POP EBP -> (+1)
		or		dl, 10b							; 设置标志: ->标准的PROC
		and		dl, 01b							; 检测是否为旧的OS的PROC
		je		SHQ_SaveData_Proc_Head
		sub		esi, 2
	SHQ_SaveData_Proc_Head:
		lodsb									; 默认不需要 POP EBP
		sub		esi, edi
		add		esi, ecx						; 得到最优HOOK头基址
		mov		edi, lpBuffer
		lea		ecx, SHQ_HookParam_Fix
		sub		ecx, esi
		rep		movsb
		mov		eax, lpParam
		test	eax, eax						; 如果HookParam=NULL,可以精简指令为
		jne		SHQ_SaveData_ParamExist	
		mov		ax, 6Ah							; 6A 00         push    0
		stosw
		add		esi, 5
		lea		ecx, SHQ_HookParam_Fix_Sec
		sub		ecx, esi
		rep		movsb
		add		esi, 3							; 0 时无需地址解码
		mov		al, 50h							; 50            push    eax
		stosb									; 采用手工完成 push    eax
		jmp		SHQ_SaveData_BackMe
	SHQ_SaveData_ParamExist:
		lea		ecx, SHQ_BackAddress
		sub		ecx, esi
		rep		movsb
	SHQ_SaveData_BackMe:						; 处理返回HOOK调用部分
		test	edx, edx
		jne		SHQ_SaveData_Proc_Back
		add		esi, 4
		lea		ecx, SHQ_HookProc_Fix
		sub		ecx, esi
		rep		movsb
		mov		al, 0c3h
		stosb									; 无需伪装,直接RET跳转调用HOOK
		jmp		SHQ_SaveData_Exit
	SHQ_SaveData_Proc_Back:	
		lea		ecx, SHQ_SaveData				; 余下的指令都需要
		cmp		bAPICount, 0
		jne		SHQ_SaveData_Proc_Normal
		mov		esi, SHQ_FuckESP_Fix			; 不用移动ESP
		sub		ecx, 2							; 参数个数为0,又可省 2 BYTE
		mov		byte ptr [ECX-1], 0c3h
	SHQ_SaveData_Proc_Normal:
		sub		ecx, esi
		rep		movsb
	SHQ_SaveData_Exit:
		sub		edi, lpBuffer
		mov		dwSeed.LowPart, edi
	;--------------------------------------------------------------------------------
	}
	return  dwSeed.LowPart;
}
```

### 模块名字：`DWORD GetFixOpcodeByLength(LPVOID lpBuffer,LPVOID lpOpcode,LPVOID lpRemoteBuffer,DWORD dwNumberOfBytes)`

**模块功能**：备份被Inline HOOK的API头

**返回数值**：返回实际备份指令串长度, 零表示失败

**参数说明**：
参数名 | 输入/输出 | 参数说明
`lpBuffer` | `OUT` | 备份API的本地缓冲区地址
`lpOpcode` | `IN` | 需要处理的操作码基址(即原API的EPO)
`lpRemoteBuffer` | `IN` | 备份API的远程缓冲区地址
`dwNumberOfBytes` | `IN` | 需要备份指令串长度

**更新**：
2008-05-11: 在内核环境下当检测到ret指令,就应该是为该指令串结束标志,不宜继续处理
2008-05-08: 增加对长CALL,JMP跳转的检测功能
2008-05-03: 增加对CALL跳转的修正功能
2008-04-28: 如果空间不够则,不被HOOK
2008-04-27: 由HSQ花12h用OD和VC9调试通过, 增加了对简单跳转指令的修正功能，及优化代码结构

```c
DWORD	GetFixOpcodeByLength(PVOID lpBuffer,LPVOID lpOpcode,LPVOID lpRemoteBuffer,DWORD dwNumberOfBytes)
{	
	__asm
	{	
		push	ebx
		xor		ebx, ebx
		mov		esi, lpOpcode
		test	esi, esi
		je		GetFixOpcodeByLength_exit		
		mov		edi, lpBuffer
		mov		ecx, ebx
		cld
	GetFixOpcodeByLength_next:
		push	ecx
		push	esp
		push	esi
		call	LDE32 	
		pop		ecx
		add		ebx, ecx
		cmp		ecx, 2
		ja		GetFixOpcodeByLength_check
		lodsb
		dec		esi
		cmp		al, 0c2h
		je		GetFixOpcodeByLength_hook_false
		cmp		al, 0c3h
		je		GetFixOpcodeByLength_hook_false
		jmp		GetFixOpcodeByLength_normal
	GetFixOpcodeByLength_hook_false:
		xor		ebx, ebx
		jmp		GetFixOpcodeByLength_exit
	GetFixOpcodeByLength_check:
		cmp		ecx, 5
		jne		GetFixOpcodeByLength_check_jmp
		lodsb
		stosb
		cmp		al, 0XE8
		je		GetFixOpcodeByLength_fix_dword
		jmp		GetFixOpcodeByLength_no_call
	GetFixOpcodeByLength_check_jmp:
		cmp		ecx, 6	
		jne		GetFixOpcodeByLength_normal
		lodsw
		stosw
		xchg	ah, al
		cmp		ax, 0FF25h
		je		GetFixOpcodeByLength_hook_false	
		cmp		ax, 0FF15h
		je		GetFixOpcodeByLength_hook_false
		cmp		ax, 0F8Fh
		ja		GetFixOpcodeByLength_no_jmp
		cmp		ax, 0F80h
		jb		GetFixOpcodeByLength_no_jmp
	GetFixOpcodeByLength_fix_dword:	
		lodsd
		add		eax, lpOpcode 
		sub		eax, lpRemoteBuffer
		stosd
		jmp		GetFixOpcodeByLength_continue
	GetFixOpcodeByLength_no_jmp:
		dec		ecx
	GetFixOpcodeByLength_no_call:
		dec		ecx
	GetFixOpcodeByLength_normal:
		rep		movsb
	GetFixOpcodeByLength_continue:	
		cmp		ebx, dwNumberOfBytes
		jb		GetFixOpcodeByLength_next
		mov		al, 0e9h
		stosb
		mov		eax, esi
		sub		eax, lpRemoteBuffer
		sub		eax, ebx
		sub		eax, 5
		stosd	
	GetFixOpcodeByLength_exit:
		mov		eax, ebx
		pop		ebx
		mov		dwNumberOfBytes, eax
	}
	return dwNumberOfBytes;
}
```

### 模块名字：`DWORD FixOpcodeForBackup(LPVOID lpOpcode,LPVOID lpFixBase,DWORD dwNumberOfBytes)`

**模块功能**：修正备份Inline HOOK的API头

**返回数值**：返回实际修正指令串长度, 零表示失败

**参数说明**：
参数名 | 输入/输出 | 参数说明
`lpOpcode` | `IN` | 需要处理的操作码基址
`lpFixBase` | `IN` | 需修正目标空间的基址
`dwNumberOfBytes` | `IN` | 需修正的指令串长度

```c
DWORD	FixOpcodeForBackup(LPVOID lpOpcode,LPVOID lpFixBase,DWORD dwNumberOfBytes)
{	
	__asm
	{
		push	ebx
		xor		ebx, ebx
		mov		esi, lpOpcode
		test	esi, esi
		je		FixOpcodeForBackup_exit	
		mov		ecx, ebx
		cld
	FixOpcodeForBackup_next:
		push	ecx
		push	esp
		push	esi
		call	LDE32 	
		pop		ecx
		add		ebx, ecx
		cmp		ecx, 5
		jne		FixOpcodeForBackup_check_jmp
		lodsb
		cmp		al, 0XE8
		jne		FixOpcodeForBackup_no_call	
		jmp		FixOpcodeForBackup_fix_dword
	FixOpcodeForBackup_check_jmp:
		cmp		ecx, 6	
		jne		FixOpcodeForBackup_normal
		lodsw
		xchg	ah, al
		cmp		ax, 0F8Fh
		ja		FixOpcodeForBackup_no_jmp
		cmp		ax, 0F80h
		jb		FixOpcodeForBackup_no_jmp
	FixOpcodeForBackup_fix_dword:	
		mov		edi, esi
		lodsd
		add		eax, lpFixBase
		sub		eax, lpOpcode
		stosd
		jmp		FixOpcodeForBackup_continue
	FixOpcodeForBackup_no_jmp:
		dec		ecx
	FixOpcodeForBackup_no_call:
		dec		ecx
	FixOpcodeForBackup_normal:
		add		esi, ecx
	FixOpcodeForBackup_continue:
		cmp		ebx, dwNumberOfBytes
		jb		FixOpcodeForBackup_next
	FixOpcodeForBackup_exit:
		mov		eax, ebx
		pop		ebx
		mov		dwNumberOfBytes, eax
	}
	return dwNumberOfBytes;
}
```
