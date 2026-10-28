
# [分享]获取IP地址的另一种方法

由于工作需要，想实现不调用任何函数获取IP地址，于是把`GetIpAddrTable`函数逆了，发现它是向`\device\tcp`查询ip实现的，进一步逆向进行不下去了（水平有限）。

模拟其实现写了段获取IP的代码，虽然不知道有什么用，还是跟大家分享一下吧：

```
#include <windows.h>

#include <stdio.h>

//By Fypher

//4d7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6F1L8h3^5%4x3e0b7`.

int main(){

	HANDLE hDevice=CreateFile(

		TEXT("\\\\.\\Tcp"),

		GENERIC_READ|GENERIC_WRITE,

		FILE_SHARE_READ|FILE_SHARE_WRITE,

		0,

		OPEN_EXISTING,

		0,

		0

	);

	if(hDevice==INVALID_HANDLE_VALUE){

		return 0;

	}

	BYTE pInBuffer[36]={

		0x01,0x03,0x00,0x00,0x00,0x00,

		0x00,0x00,0x00,0x02,0x00,0x00,

		0x00,0x01,0x00,0x00,0x02,0x01,

		0x00,0x00,0x00,0x00,0x00,0x00,

		0x00,0x00,0x00,0x00,0x00,0x00,

		0x00,0x00,0x00,0x00,0x00,0x00

	};

	BYTE pOutBuffer[296];

	DWORD len;

	if(!DeviceIoControl(hDevice,

						0x00120003,&pInBuffer,

						sizeof(pInBuffer),

						&pOutBuffer,

						sizeof(pOutBuffer),

						&len,

						NULL))

	{

		return 0;

	}

	CloseHandle(hDevice);

	ULONG a=*(PULONG)pOutBuffer & 0x000000FF;

	ULONG b=(*(PULONG)pOutBuffer & 0x0000FF00)>>8;

	ULONG c=(*(PULONG)pOutBuffer & 0x00FF0000)>>16;

	ULONG d=(*(PULONG)pOutBuffer & 0xFF000000)>>24;

	printf("ip:%d.%d.%d.%d\n",a,b,c,d);

	system("pause");

	return 0;

}
```

但是我的问题依然没有解决。尝试过暴力搜索，发现内存中没有一个跟系统版本相关的固定地址存放IP地址。![](https://bbs.kanxue.com/view/img/face/50.gif)

不调用任何函数获取IP地址（`Ring0`/`Ring3`都行），请大牛们给点思路……

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
