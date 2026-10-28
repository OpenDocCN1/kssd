
# [原创]杀鸡焉用牛刀，再探好听音乐网

看了破船兄的文章[【原创】好听音乐网 -- 歌曲下载分析流程及程序的编写](http://bbs.pediy.com/showthread.php?t=92286)，我也来写一个。破船的代码是智慧型的，我来个猛士型的。。![](https://bbs.kanxue.com/view/img/face/28.gif)

打开好听音乐网的首页，随便点一首歌来听，出现了播放界面，同时音乐也出来了，这时查看源文件，容易发现如下关键代码

```
           if("undefined"==typeof(UUAuthCode)|| UUAuthCode==null)  UUAuthCode="";

	else UUAuthCode="?"+UUAuthCode;

        urlht = "/21z/0/daodaidq090508/1/8f4257328b24c8d0_5.wma"+UUAuthCode;

} 

seto();

var url='f0cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6E0j5g2)9J5k6h3S2S2L8%4c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3U0N6Q4x3@1u0Q4y4f1u0Q4x3V1k6o6e0@1I4a6f1W2)9#2c8l9`.`.

document.writeln("<object id=\"mPlayer\" width=365 height=62");

document.writeln(" classid=\"CLSID:6BF52A52-394A-11D3-B153-00C04F79FAA6\" type=application\/x-oleobject standby=\"Loading Windows Media Player ");

document.writeln("");

document.writeln("components...\">");

document.writeln("             <param name=\"URL\" value='"+url+""+urlht+"'>");
```

注意看加红部分，`url + urlht` 后所得字符串就是音乐下载地址，现在已经很明朗了，我们只要打开音乐播放页面，然后查看源文件，在里面找到类似`/21z/0/daodaidq090508/1/8f4257328b24c8d0_5.wma`的字符串，再和``2ecK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6E0j5g2)9J5k6h3S2S2L8%4c8A6L8X3N6Q4x3X3g2U0L8$3@1`.``连接起来就是一个完整的下载地址了，现在是主要问题就是怎么找到这个`/21z/0/daodaidq090508/1/8f4257328b24c8d0_5.wma`，这个就是靠RP了![](https://bbs.kanxue.com/view/img/face/12.gif)，通过观察，要查找到它的位置就要找到能唯一确定它的特征代码，而`/21z/`这个字符串正是它的特征代码，这个字符串是唯一的，而且是通用的。发现了这个，剩下的就是体力活了，Now is Code Time , Go !

```
#include <windows.h>
#include <UrlMon.h>

#pragma comment(lib, "URLmon.lib")

WCHAR wcURL[2048];
WCHAR wcFileName[2048];
char szBuffer[256];
char szFullPath[512];

int WINAPI WinMain(HINSTANCE hInstance,
				   HINSTANCE hPrevInstance,
				   LPSTR lpCmdLine,
				   int nShowCmd)
{
	memset(szFullPath, 0, sizeof(char) * 512);
	memset(szBuffer, 0, sizeof(char) * 256);
	memset(wcFileName, 0, sizeof(WCHAR) * 2048);
	memset(wcURL, 0, sizeof(WCHAR) * 2048);
	
	lstrcpyA(szFullPath, "592K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6E0j5g2)9J5k6h3S2S2L8%4c8A6L8X3N6Q4x3X3g2U0L8$3@1`."); //好听音乐网的音乐服务器网址,域名是固定的
	lstrcpy(wcFileName, TEXT("C:\\Test.txt")); //临时保存的文件，会在文件关闭后自动被删除
	lstrcpy(wcURL, TEXT("141K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2L8%4c8A6L8X3N6Q4x3X3g2U0L8$3#2Q4x3V1k6Z5N6r3#2#2M7$3W2U0i4K6u0r3x3K6b7&6z5o6l9&6K9s2c8Q4x3X3g2Z5N6r3@1`.")); //示例网址

	if (S_OK == URLDownloadToFile(NULL, 
		wcURL, 
		wcFileName,
		0,
		NULL))
	{
		HANDLE hFile = CreateFile(wcFileName, 
			GENERIC_READ, 
			FILE_SHARE_READ,
			NULL, OPEN_EXISTING,
			FILE_FLAG_DELETE_ON_CLOSE,
			NULL);
		
		if (INVALID_HANDLE_VALUE == hFile)
		{
			hFile = NULL;
			return 0;
		}

		LARGE_INTEGER FileSize = {0};

		GetFileSizeEx(hFile, &FileSize);
		
		HANDLE hMemFile = CreateFileMapping(hFile, NULL, PAGE_READONLY, 0, 0, NULL);

		if (INVALID_HANDLE_VALUE == hMemFile)
		{
			hMemFile = NULL;
			return 0;
		}

		LPBYTE lpByte = (LPBYTE)MapViewOfFile(hMemFile, FILE_MAP_READ, 0, 0, 0);

		for (UINT i = 0; i < FileSize.u.LowPart; i++)
		{
			if ((*lpByte == '/') 
				&& (*(lpByte + 1) == '2') 
				&& (*(lpByte + 2) == '1') 
				&& (*(lpByte + 3) == 'z')
				&& (*(lpByte + 4) == '/'))
			{
				for (int j = 0;; j++)
				{
					szBuffer[j] = *lpByte;

					if ((*lpByte == 'a')
						&& (*(lpByte - 1) == 'm')
						&& (*(lpByte - 2) == 'w')
						&& (*(lpByte - 3) == '.'))
					{
						break;
					}

					lpByte += 1;
				}

				break;
			}

			lpByte += 1;
		}
		
		UnmapViewOfFile((LPVOID)lpByte);
		CloseHandle(hMemFile);
		CloseHandle(hFile);
		
		lstrcatA(szFullPath, szBuffer);

		MessageBoxA(NULL, szFullPath, NULL, MB_OK); //测试是否弹出了我们想要的下载地址 ^_^ ~~

	}

	return 0;
}
```

我在VS 2005 + server 2003下编译测试通过~!

有了这个查找引擎，做个GUI来实现批量下载还难吗![](https://bbs.kanxue.com/view/img/face/12.gif)

PS:这个服务器都是提供WMA格式的，价值不大，我的方法也很挫，但是所想即所得![](https://bbs.kanxue.com/view/img/face/41.gif)

附件中是一个编译好的示例，演示用的。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`test.zip`](javascript:void(0);)

  （13.13kb，72次下载）
