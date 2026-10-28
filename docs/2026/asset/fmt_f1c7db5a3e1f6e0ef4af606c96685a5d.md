
# [原创]貌似很多主动防御忽略了WDM式驱动的加载

这些天搞过主动防御。发现有些不拦截WDM式的驱动加载。

只要能让我加载驱动, 还有什么不能XX。

发个WDM式加载的代码。。。然后去XX你的主动防御吧。

遗憾的是我这里只安装了TSZ,有兴趣的帮忙测试一下其他的主动防御拦截否。

如果不愿意看以下代码了，找个EzInstall去随便安装一个驱动，或者去在带有主动防御软件机器上重新安装一下你的某个硬件设备的驱动,看看拦截否。

```
VOID FindComma(LPSTR szData)
{
	WORD wLen = (WORD)strlen(szData);
	WORD wIdx;
	WORD wLoop; 
	CHAR szTmp[128] = {0};
	
	for (wIdx = 0, wLoop = 0; wLoop < wLen; wLoop++)
	{
		if (szData[wLoop] == ',')
			szData[wLoop] = '.';
		else if (szData[wLoop] == ' ')
			continue;
		szTmp[wIdx++] = szData[wLoop];
	}
	memcpy(szData, szTmp, wIdx*sizeof(char));
	szData[wIdx] = 0;
}

VOID StrLTrim(LPSTR szData)
{
	LPSTR ptr = szData;
	while (isspace(*ptr))
		//ptr = _tcsinc(ptr);
		ptr++;
	if (strcmp(ptr, szData))
	{
		WORD wLen = (WORD)(strlen(szData) - (ptr - szData));
		memmove(szData, ptr, (wLen+1)*sizeof(char));
	}
}

VOID StrRTrim(LPSTR szData)
{
	LPSTR ptr  = szData;
	LPSTR pTmp = NULL;
	while (*ptr != 0)
	{
		if (isspace(*ptr))
		{
			if (!pTmp)
				pTmp = ptr;
		}
		else
			pTmp = NULL;
		//ptr = _tcsinc(ptr);
		ptr++;
	}
	if (pTmp)
	{
		*pTmp = 0;
		memmove(szData, szData, strlen(szData) - strlen(pTmp));
	}
}

VOID StrRight(LPSTR szData, WORD wCount)
{
	WORD wLen = (WORD)strlen(szData) - wCount;
	if (wCount > 0x7FFF)//负数
		wCount = 0;
	if (wCount >= (WORD)strlen(szData))
		return;
	memmove(szData, szData + wLen, wCount * sizeof(char));
	szData[wCount] = 0;
}

VOID ConvertGUIDToString(const GUID guid, LPSTR pData)
{
	CHAR szData[30] = {0};
	CHAR szTmp[3]   = {0};
	WORD wLoop;
	sprintf(pData, "%04X-%02X-%02X-", guid.Data1, guid.Data2, guid.Data3);
	for (wLoop = 0; wLoop < 8; wLoop++)
	{
		if (wLoop == 2)
			strcat(szData, "-");
		sprintf(szTmp, "%02X", guid.Data4[wLoop]);
		strcat(szData, szTmp);
	}
	memcpy(pData + strlen(pData), szData, strlen(szData));
}

BOOL AnsiToUnicode(LPCSTR Source, const WORD sLen, LPWSTR Destination, const WORD wLen)
{
	return MultiByteToWideChar(CP_ACP, MB_PRECOMPOSED, Source, sLen, Destination, wLen);
}

BOOL UnicodeToAnsi(LPCWSTR Source, const WORD wLen, LPSTR Destination, const WORD sLen)
{
	return WideCharToMultiByte(CP_ACP, WC_COMPOSITECHECK, Source, wLen, Destination, sLen, 0L, 0L);
}
```

// 主代码
