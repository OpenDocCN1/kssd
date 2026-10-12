
# 获取 IP 地址的另一种方法

> 原文：[`www.pediy.com/kssd/pediy10/95920.html`](https://www.pediy.com/kssd/pediy10/95920.html)

由于工作需要，想实现不调用任何函数获取 IP 地址，于是把`GetIpAddrTable`函数逆了，发现它是向`\device\tcp`查询`ip`实现的，进一步逆向进行不下去了（水平有限）。

模拟其实现写了段获取 IP 的代码，虽然不知道有什么用，还是跟大家分享一下吧：

代码:

```
#include <windows.h>
#include <stdio.h>
//By Fypher
//http://hi.baidu.com/nmn714

int main(){
  HANDLE hDevice=CreateFile(
    TEXT("\\\\?\\Tcp"),
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

但是我的问题依然没有解决。尝试过暴力搜索，发现内存中没有一个跟系统版本相关的固定地址存放 IP 地址。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

不调用任何函数获取 IP 地址（`Ring0`/`Ring3` 都行），请大牛们给点思路……
