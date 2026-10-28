
```cpp
#include "stdafx.h"

#include<string>

#include<windows.h>

#include<iostream>

#include <WinInet.h>

#pragma comment(lib,"wininet.lib")

using namespace std;

char URL_STRING[] = "acfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3W2Q4x3X3g2H3N6r3I4G2k6$3W2F1x3W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3j5$3N6A6i4K6u0V1j5X3W2F1i4K6u0r3P5r3I4G2k6$3W2F1i4K6y4r3j5i4m8H3K9h3c8Q4x3@1b7$3x3K6j5H3x3e0b7J5x3o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0K6i4K6g2X3N6i4u0D9i4K6y4p5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7i4p5J5x3o6p5J5i4K6u0r3L8r3!0Y4K9h3&6e0N6h3y4U0k6i4y4K6i4K6u0W2K9s2c8E0i4K6t1$3j5h3#2H3i4K6y4n7M7%4c8&6L8r3g2Q4x3@1b7J5x3q4)9J5y4X3q4E0M7q4)9K6b7X3u0G2M7X3c8W2M7W2)9#2k6Y4u0S2k6r3W2#2M7#2)9K6c8o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0@1j5i4u0Y4k6i4c8Q4x3@1c8K6k6h3I4X3i4K6t1$3j5h3#2H3i4K6y4n7L8h3q4K6K9@1!0H3j5h3y4A6N6s2W2Q4x3@1b7@1x3l9`.`.";



int _tmain(int argc, _TCHAR* argv[])

{



	// 初始化URL

	URL_COMPONENTSA crackedURL = { 0 };

	char szHostName[128];

	char szUrlPath[256];

	crackedURL.dwStructSize = sizeof(URL_COMPONENTSA);

	crackedURL.lpszHostName = szHostName;

	crackedURL.dwHostNameLength = ARRAYSIZE(szHostName);

	crackedURL.lpszUrlPath = szUrlPath;

	crackedURL.dwUrlPathLength = ARRAYSIZE(szUrlPath);

	InternetCrackUrlA(URL_STRING, (DWORD)strlen(URL_STRING), 0, &crackedURL);



	// 初始化会话

	HINTERNET hInternet = InternetOpenA("Microsoft Internet Explorer", INTERNET_OPEN_TYPE_DIRECT, NULL, NULL, 0);

	HINTERNET hHttpSession = InternetConnectA(hInternet, crackedURL.lpszHostName, INTERNET_DEFAULT_HTTPS_PORT, NULL, NULL, INTERNET_SERVICE_HTTP, 0, 0);

	HINTERNET hHttpRequest = HttpOpenRequestA(hHttpSession, "GET", crackedURL.lpszUrlPath, NULL, "", NULL, INTERNET_FLAG_SECURE, 0);



	// 发送HTTP请求

	HttpSendRequest(hHttpRequest, NULL, 0, NULL, 0);



	// 查询HTTP请求状态

	DWORD dwRetCode = 0;

	DWORD dwSizeOfRq = sizeof(DWORD);

	BOOL bRet = FALSE;

	bRet = HttpQueryInfo(hHttpRequest, HTTP_QUERY_STATUS_CODE | HTTP_QUERY_FLAG_NUMBER, &dwRetCode, &dwSizeOfRq, NULL);



	// 读取整个Headers

	char lpHeaderBuffer[1024] = {0};

	dwSizeOfRq = 1024;

	bRet = HttpQueryInfo(hHttpRequest, HTTP_QUERY_RAW_HEADERS, lpHeaderBuffer, &dwSizeOfRq, NULL);



	// 从Cookie中提取pt_local_token的值

	char* pt_local_token = lpHeaderBuffer + dwSizeOfRq;

	while (pt_local_token != lpHeaderBuffer)

	{

		if (strstr(pt_local_token, "pt_local_token="))

		{

			// 退出之前，修正偏移

			pt_local_token += sizeof("pt_local_token");

			char* pEndBuffer = strstr(pt_local_token, ";");

			*pEndBuffer = 0;

			break;

		}

		pt_local_token--;

	}



	// 关闭句柄,只需要释放下面两个,注意关闭时按相反的顺序

	InternetCloseHandle(hHttpRequest);

	InternetCloseHandle(hHttpSession);





	/* 第二次建立会话 */



	// 初始化URL参数

	char lpszUrlPath[MAX_PATH] = "/pt_get_uins?callback=ptui_getuins_CB&pt_local_tk=";

	strcat(lpszUrlPath, pt_local_token); // url末尾追加pt_local_token



	// 初始化会话

	hHttpSession = InternetConnectA(hInternet, "localhost.ptlogin2.qq.com", 4301, NULL, NULL, INTERNET_SERVICE_HTTP, 0, 0);

	hHttpRequest = HttpOpenRequestA(hHttpSession, "GET", lpszUrlPath, NULL, "", NULL, INTERNET_FLAG_SECURE, 0);



	// 发送HTTP请求,添加头信息

	char* lpHeaders = "Referer:675K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3W2Q4x3X3g2H3N6r3I4G2k6$3W2F1x3W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3j5$3N6A6i4K6u0V1j5X3W2F1i4K6u0r3P5r3I4G2k6$3W2F1i4K6y4r3j5i4m8H3K9h3c8Q4x3@1b7$3x3K6j5H3x3e0b7J5x3o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0K6i4K6g2X3N6i4u0D9i4K6y4p5K9s2c8@1M7q4)9J5y4e0y4m8i4K6t1#2x3V1k6Q4x3U0f1J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6t1#2x3V1k6I4M7e0t1H3x3e0u0Q4x3U0f1J5c8X3I4G2k6$3W2F1f1%4g2U0j5$3g2K6M7#2)9J5k6h3S2@1L8b7`.`.";

	HttpSendRequestA(hHttpRequest, lpHeaders, strlen(lpHeaders), NULL, 0);



	// 查询HTTP请求状态

	dwRetCode = 0;

	dwSizeOfRq = sizeof(DWORD);

	bRet = HttpQueryInfo(hHttpRequest, HTTP_QUERY_STATUS_CODE | HTTP_QUERY_FLAG_NUMBER, &dwRetCode, &dwSizeOfRq, NULL);



	// 获取返回数据的大小

	DWORD dwNumberOfBytesAvailable = 0;

	bRet = InternetQueryDataAvailable(hHttpRequest, &dwNumberOfBytesAvailable, NULL, NULL);



	// 读取网页内容

	char* lpBuffer = new char[dwNumberOfBytesAvailable]();

	bRet = InternetReadFile(hHttpRequest, lpBuffer, dwNumberOfBytesAvailable, &dwNumberOfBytesAvailable);



	// 从内容中提取已登陆QQ账号,是个js数组,这里只提取第一个

	char* uin = lpBuffer + dwNumberOfBytesAvailable;

	while (uin != lpBuffer)

	{

		if (strstr(uin, "\"account\":\""))

		{

			// 退出之前，修正偏移

			uin += sizeof("\"account\":");

			char* pEndBuffer = strstr(uin, "\"");

			*pEndBuffer = 0;

			break;

		}

		uin--;

	}



	cout << "[+] uin:" << uin << endl;



	// 释放资源,注意关闭句柄时按相反的顺序

	InternetCloseHandle(hHttpRequest);

	InternetCloseHandle(hHttpSession);





	/* 第三次会话 */



	// 初始化URL参数

	ZeroMemory(lpszUrlPath,MAX_PATH);

	strcat(lpszUrlPath, "/pt_get_st?clientuin=");

	strcat(lpszUrlPath, uin);

	strcat(lpszUrlPath, "&callback=ptui_getst_CB&pt_local_tk=");

	strcat(lpszUrlPath, pt_local_token);





	// 发送HTTPS请求

	hHttpSession = InternetConnectA(hInternet, "localhost.ptlogin2.qq.com", 4301, NULL, NULL, INTERNET_SERVICE_HTTP, 0, 0);

	hHttpRequest = HttpOpenRequestA(hHttpSession, "GET", lpszUrlPath, NULL, "", NULL, INTERNET_FLAG_SECURE, 0);



	// 添加头信息

	lpHeaders = "Referer:c0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3W2Q4x3X3g2H3N6r3I4G2k6$3W2F1x3W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3j5$3N6A6i4K6u0V1j5X3W2F1i4K6u0r3P5r3I4G2k6$3W2F1i4K6y4r3j5i4m8H3K9h3c8Q4x3@1b7$3x3K6j5H3x3e0b7J5x3o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0K6i4K6g2X3N6i4u0D9i4K6y4p5K9s2c8@1M7q4)9J5y4e0y4m8i4K6t1#2x3V1k6Q4x3U0f1J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6t1#2x3V1k6I4M7e0t1H3x3e0u0Q4x3U0f1J5c8X3I4G2k6$3W2F1f1%4g2U0j5$3g2K6M7#2)9J5k6h3S2@1L8b7`.`.";

	HttpSendRequestA(hHttpRequest, lpHeaders, strlen(lpHeaders), NULL, 0);



	// 查询HTTP请求状态

	dwRetCode = 0;

	dwSizeOfRq = sizeof(DWORD);

	bRet = HttpQueryInfoA(hHttpRequest, HTTP_QUERY_STATUS_CODE | HTTP_QUERY_FLAG_NUMBER, &dwRetCode, &dwSizeOfRq, NULL);



	// 读取整个Headers

	ZeroMemory(lpHeaderBuffer, 1024);

	dwSizeOfRq = 1024;

	bRet = HttpQueryInfoA(hHttpRequest, HTTP_QUERY_RAW_HEADERS, lpHeaderBuffer, &dwSizeOfRq, NULL);



	// 从Cookie中提取ClientKey的值

	char* clientkey = lpHeaderBuffer + dwSizeOfRq;

	while (clientkey != lpHeaderBuffer)

	{

		if (strstr(clientkey, "clientkey="))

		{

			// 退出之前，修正偏移

			clientkey += sizeof("clientkey");

			char* pEndBuffer = strstr(clientkey, ";");

			*pEndBuffer = 0;

			break;

		}

		clientkey--;

	}



	cout << "[+] client key:" << clientkey << endl;



	InternetCloseHandle(hHttpRequest);

	InternetCloseHandle(hHttpSession);

	InternetCloseHandle(hInternet);

	delete[] lpBuffer;





	return 0;

}
```

冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。 [https://www.kanxue.com/book-section_list-227.htm](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-1-2 11:35

被Adventure编辑，原因： 更新使用方法
