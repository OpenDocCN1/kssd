
# [原创]一个用CRC程序保护程序的具体例子

这是一个用CRC程序保护的具体例子。前几天在网上找具体的例子，想直接放在自己的程序里，竟然没有找到完整的实例，只有算法和原理。我整理了一下找到的内容，在控制台下写了个完整的程序。

只要大家用改动程序的一个字节，程序也能检测出来，比如把`int3` `nop`掉也能检测出来。

程序链接成可执行文件后要用16进制工具把原来程序的CRC值写在PE `Optional_Section`的`Win32Version`处。

下面是源代码，附件是CRC值已经写入的可执行文件，大家可试验一下。

```c
#include "stdio.h"
#include "windows.h"

DWORD CRC32(BYTE* ptr,DWORD Size)
{
  DWORD crcTable[256],crcTmp1;
  //动态生成CRC-32表
  for (int i=0; i<256; i++)
   {
    crcTmp1 = i;
    for (int j=8; j>0; j--)
     {
      if (crcTmp1&1) crcTmp1 = (crcTmp1 >> 1) ^ 0xEDB88320L;
       else crcTmp1 >>= 1;
    }
     crcTable[i] = crcTmp1;
   }
  //计算CRC32值
  DWORD crcTmp2= 0xFFFFFFFF;
  while(Size--)
  {
    crcTmp2 = ((crcTmp2>>8) & 0x00FFFFFF) ^ crcTable[ (crcTmp2^( *ptr)) & 0xFF ];
    ptr++;
  }
  return (crcTmp2^0xFFFFFFFF);
}

bool isModified = false;
void main()
{
/*        DWORD res = CRC32((BYTE*)p, 6);
        printf("%x    ", res);*/
        DWORD dwNumberOfBytesReadWritten;
        HMODULE hmod;
        HGLOBAL hMemory;
        LPVOID pMemory;
        DWORD dwFileSize;
        CHAR szFileName[20];

        hmod = ::GetModuleHandle(0);
        ::GetModuleFileName(hmod, szFileName, MAX_PATH);
        HANDLE hFile = ::CreateFile(szFileName, GENERIC_READ, FILE_SHARE_READ, NULL,
                OPEN_EXISTING, FILE_ATTRIBUTE_ARCHIVE, 0);
        if(hFile != INVALID_HANDLE_VALUE)
                dwFileSize = ::GetFileSize(hFile, NULL);
        hMemory = ::GlobalAlloc(GMEM_MOVEABLE|GMEM_ZEROINIT,  dwFileSize);
        pMemory = ::GlobalLock(hMemory);
        ReadFile(hFile, pMemory, dwFileSize, &dwNumberOfBytesReadWritten, NULL);
        ::CloseHandle(hFile);

        DWORD dwPEHead, dwWin32Version;
        LPVOID pWin32Version;
        LPVOID pMemoryPEHead = pMemory;
        BYTE* pp;
        pp = static_cast<BYTE*>(pMemory) + 0x3c;
        pMemoryPEHead = (LPVOID)pp;
        dwPEHead = *((DWORD*)pMemoryPEHead);
        pWin32Version = pMemory;
        dwWin32Version = dwPEHead + 0x4c;
        BYTE* Version;
        Version = static_cast<BYTE*>(pWin32Version) + dwWin32Version;
        pWin32Version = (BYTE*)Version;
        DWORD OriginalCRC32 = *((DWORD*)pWin32Version);
        *((DWORD*)pWin32Version) = 0;
        DWORD NowCRC32 = CRC32((BYTE*)pMemory, dwFileSize);
        if(OriginalCRC32 != NowCRC32)
                ::MessageBox(0, "error", "e", MB_OK);
        else
                ::MessageBox(0, "chenggong", "c", MB_OK);
        getchar();
}
