
# `Totalcmd_FixCRC.cpp` : 自校验

```cpp
// Totalcmd_FixCRC.cpp : 自校验

#include "stdafx.h"

// 使用说明

void Usage();

// 计算校验和

void CalculateCRC(unsigned char * pData, unsigned int nSize, unsigned int * nCRC);

// 修复数据及其大小

void FixData(char * pData, unsigned int * nSize);



int _tmain(int argc, _TCHAR* argv[])

{

	TCHAR * szFileName = NULL;

	char lpBuffer[0x8001] = {0};

	unsigned int nNumberOfBytesToRead = 0;

	unsigned int nNumberOfBytesRead = 0;

	unsigned int nSize = 0;

	unsigned int nSizeRead = 0;

	unsigned int nCRC = 0xffffffff;

	HANDLE hFile = NULL;



	printf("\nTotalcmd_FixCRC by uuk   2012.03.23\n\n");



	if ( argc != 2 )

	{

		Usage();

		return 0;

	}



	szFileName = argv[1];



	hFile = CreateFile(szFileName, GENERIC_READ | GENERIC_WRITE, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);

	if ( hFile == INVALID_HANDLE_VALUE )

	{

		printf("CreateFile Error!\n");

		return 1;

	}



	nSize = SetFilePointer(hFile, 0, NULL, FILE_END);

	if ( nSize == INVALID_SET_FILE_POINTER)

	{

		printf("SetFilePointer Error!\n");

		CloseHandle(hFile);

		return 2;

	}

	nSize -= 36;

	SetFilePointer(hFile, 0, NULL, FILE_BEGIN);



	do 

	{

		if ( nSize >= 0x8000 )

			nNumberOfBytesToRead = 0x8000;

		else

			nNumberOfBytesToRead = nSize;



		ReadFile(hFile, lpBuffer, nNumberOfBytesToRead, (LPDWORD)&nNumberOfBytesRead, NULL);



		if ( nSizeRead == 0 )

			FixData(lpBuffer, &nSize);



		CalculateCRC((unsigned char *)lpBuffer, nNumberOfBytesRead, &nCRC);



		nSizeRead += nNumberOfBytesRead;

		nSize -= nNumberOfBytesRead;

	} while ( (nNumberOfBytesRead==0x8000) && nSize );



	// 读取文件中存储的校验和

	ReadFile(hFile, lpBuffer, 36, (LPDWORD)&nNumberOfBytesRead, NULL);

	unsigned int nCRC2 = *(unsigned int *)(lpBuffer+0x10);



	nCRC = nCRC ^ 0xF5A3E289;

	nCRC2 = nCRC2 ^ 0x2A67BE65;

	if ( nCRC != nCRC2 )

	{

		unsigned int nCRC3 = nCRC ^ 0x2A67BE65;

		SetFilePointer(hFile, nSizeRead+0x10, NULL, FILE_BEGIN);

		int nWrite = 0;

		WriteFile(hFile, &nCRC3, 4, (LPDWORD)&nWrite, NULL);

		printf("File is Fixed!\n");

	} 

	else

	{	printf("File is not modified!\n");	}



	CloseHandle(hFile);

	return 0;

}



void Usage()

{	printf("Usage: Totalcmd_FixCRC.exe FileName\n");	}



void CalculateCRC(unsigned char * pData, unsigned int nSize, unsigned int * nCRC)

{

	unsigned int nSizeRead;

	int nCRCLow, nCRCHigh;



	nCRCLow = *nCRC & 0xffff;

	nCRCHigh = (*nCRC>>16) & 0xffff;



	if ( pData )

	{

		if ( nSize > 0 )

		{

			do 

			{

				if ( nSize >= 5552 )

					nSizeRead = 5552;

				else

					nSizeRead = nSize;

				nSize -= nSizeRead;

				do 

				{

					nCRCLow += *pData;

					nCRCHigh += nCRCLow;

					pData++;

					nSizeRead--;

				} while ( nSizeRead );

				nCRCLow %= 65521;

				nCRCHigh %= 65521;

			} while ( nSize > 0 );

		}

		*nCRC = ( nCRCHigh << 16 ) | nCRCLow;

	} 

	else

	{	*nCRC = 1;	}

}



void FixData(char * pData, unsigned int * nSize)

{

	int i = 0;

	if ( pData[0x198] != '\0' )

	{

		*nSize = *(unsigned int *)(pData+0x198) - 36;

		for (i=0; i<8; i++)

		{	pData[0x198+i] = '\0';	}

	}

	for (i=0; i<4; i++)

	{	pData[0x158+i] = '\0';	}

}
```

## 注册算法与黑名单

新程序采用的注册算法没有变，只是将网上公开的许可文件加入黑名单。黑名单中是许可文件的许可号（每个占4字节）。`Totalcmd756a`版本的黑名单如下所示：

`004E0ABC`    `Totalcmd`    `mov`    `eax, 006C8A3C`        ;  黑名单1

`004E0AD5`    `Totalcmd`    `mov`    `eax, 006C8BE0`        ;  黑名单2

`004E0AEB`    `Totalcmd`    `mov`    `eax, 006C90F4`        ;  黑名单3

`004E0B07`    `Totalcmd`    `mov`    `eax, 006C9168`        ;  黑名单4

比较有意思的是我们可以通过特殊的方法修改黑名单里面的数据而不改变校验和。推导`CalculateCRC()`函数中校验和的计算公式得：

![](https://bbs.kanxue.com/upload/attach/201203/108558_o7ltu0kaufwcvms.png)     （`Bn`代表第n个字节的值）

![](https://bbs.kanxue.com/upload/attach/201203/108558_dvr7dgzby3gm0c6.png)

由于 `(n - 1) + (n+1) = 2n`，所以根据校验码的计算公式可以将`Bn-1`和`Bn+1`的值各加1，然后将`Bn`值减2，则总结果不变。因此可以通过这个办法将许可证黑名单中的许可号改掉。如许可号#1283在黑名单中，其HEX值为`0x00000503`，在文件中是以`0x03050000`存储的，将其改成`0x04030100`则许可文件可继续使用，程序也不报错。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

## 上传的附件

- [eq1.PNG](javascript:void(0);)

  （0.74kb，767次下载）

- [eq2.PNG](javascript:void(0);)

  （1.81kb，762次下载）
