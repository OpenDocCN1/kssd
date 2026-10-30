
```
#define _CRT_SECURE_NO_WARNINGS

#include <windows.h> 

#include <tchar.h>

#include <iostream>

using namespace std;



extern "C" __declspec(dllexport) int main(int argc, char** argv);

void FreeImage_test(HINSTANCE hinstLib, wchar_t* pathfile);

wchar_t* charToWChar(const char* text);



// 加载卸载函数 

typedef DWORD(__stdcall *FreeImage_GetFileTypeU)(const wchar_t* lpszPathName, int flag);

typedef DWORD(__stdcall *FreeImage_Initialise)(BOOL load_local_plugins_only);

typedef DWORD(__stdcall *FreeImage_DeInitialise)();

typedef DWORD(__stdcall *FreeImage_LoadU)(DWORD format, const wchar_t* lpszPathName, int flag);

typedef DWORD(__stdcall *FreeImage_UnLoad)(DWORD dib);



// 图片格式判断函数

typedef DWORD(__stdcall *FreeImage_HasBackgroundColor)(DWORD dib); FreeImage_HasBackgroundColor HasBackgroundColor;

typedef DWORD(__stdcall *FreeImage_HasPixels)(DWORD dib); FreeImage_HasPixels HasPixels;

typedef DWORD(__stdcall *FreeImage_HasRGBMasks)(DWORD dib); FreeImage_HasRGBMasks HasRGBMasks;

typedef DWORD(__stdcall *FreeImage_IsTransparent)(DWORD dib); FreeImage_IsTransparent IsTransparent;



// 转换函数

typedef DWORD(__stdcall *FreeImage_ConvertTo24Bits)(DWORD dib); FreeImage_ConvertTo24Bits ConvertTo24Bits;

typedef DWORD(__stdcall *FreeImage_ConvertTo32Bits)(DWORD dib); FreeImage_ConvertTo32Bits ConvertTo32Bits;



FreeImage_Initialise Initialise;

FreeImage_GetFileTypeU LoadFileType;

FreeImage_LoadU LoadU; DWORD load;

FreeImage_UnLoad UnLoad;

FreeImage_DeInitialise DeInitialise;



int main(int argc, char** argv)

{

	if (argc < 2) {

		printf("Usage: %s <xml file>\n", argv[0]);

		return 0;

	}

	

	wchar_t* PathName = charToWChar(argv[1]);



	HINSTANCE hinstLib; BOOL fFreeResult, fRunTimeLinkSuccess = FALSE; DWORD Error = NULL;

	hinstLib = LoadLibrary(TEXT("E:\\FreeImage.dll"));



	if (hinstLib != NULL)

	{

		fRunTimeLinkSuccess = TRUE;

		

		Initialise = (FreeImage_Initialise)GetProcAddress(hinstLib, (LPCSTR)163); // 初始化 FreeImage 库

		LoadFileType = (FreeImage_GetFileTypeU)GetProcAddress(hinstLib, (LPCSTR)126);// 获取位图文件类型

		LoadU = (FreeImage_LoadU)GetProcAddress(hinstLib, (LPCSTR)181);	// 加载位图

		UnLoad = (FreeImage_UnLoad)GetProcAddress(hinstLib, (LPCSTR)242);// 卸载位图

		DeInitialise = (FreeImage_DeInitialise)GetProcAddress(hinstLib, (LPCSTR)83);//卸载 FreeImage 库

		

		//图片格式加载函数

		HasBackgroundColor = (FreeImage_HasBackgroundColor)GetProcAddress(hinstLib, (LPCSTR)160);

		HasPixels = (FreeImage_HasPixels)GetProcAddress(hinstLib, (LPCSTR)161);

		HasRGBMasks = (FreeImage_HasRGBMasks)GetProcAddress(hinstLib, (LPCSTR)162);

		IsTransparent = (FreeImage_IsTransparent)GetProcAddress(hinstLib, (LPCSTR)168);

	

		//转换函数

		ConvertTo24Bits = (FreeImage_ConvertTo24Bits)GetProcAddress(hinstLib, (LPCSTR)65);

		ConvertTo32Bits = (FreeImage_ConvertTo32Bits)GetProcAddress(hinstLib, (LPCSTR)66);

		

		(Initialise)(FALSE);

		

		FreeImage_test(hinstLib, PathName);

		

		(DeInitialise)();

		fFreeResult = FreeLibrary(hinstLib);

	}



	if (!fRunTimeLinkSuccess)

		cout << "加载函数失败, Error: " << Error << endl;

	return 0;

}



void FreeImage_test(HINSTANCE hinstLib, wchar_t* pathfile)

{

	// 加载函数 

	DWORD FileType = (LoadFileType)(pathfile, 0);

	load = (LoadU)(FileType, pathfile, 0);

	

	//图片格式加载函数

	HasBackgroundColor(load);

	HasPixels(load);

	HasRGBMasks(load);

	IsTransparent(load);



	//转换函数

	ConvertTo24Bits(load);

	ConvertTo32Bits(load);

	

	(UnLoad)(load);

}



wchar_t* charToWChar(const char* text)

{

	size_t size = strlen(text) + 1;

	wchar_t* wa = new wchar_t[size];

	mbstowcs(wa, text, size);

	return wa;

}
```

![](https://bbs.kanxue.com/upload/attach/201910/856782_XYNEGK5ZSGVP2CF.jpg)

等所有测试结束之后就可以开始整理一下被模糊测试找出的 Bug，调试一下出现 Crash 的原因，以及是否可以被利用。当然这个过程比较繁琐，利用 BugID 可以帮助你进行 Bug 分类，或者编写 Windbg 脚本。筛选之后如图下图所示：

![](https://bbs.kanxue.com/upload/attach/201910/856782_TA9C78T6TCUK928.jpg)

首先第一个 Bug 是读写未初始化的堆，通过查看函数调用发现是 `FreeImage_GetImageType` 出的问题。

![](https://bbs.kanxue.com/upload/attach/201910/856782_WXY69GMFCEQ8EQS.jpg)

通过 `!heap -p -a` 和 `!address` 可以看出非法访问未初始化的堆。

![](https://bbs.kanxue.com/upload/attach/201910/856782_ZGX59VJGDV488NK.jpg)

样本信息如下：

![](https://bbs.kanxue.com/upload/attach/201910/856782_NB5KZ42YZ9SZVB5.jpg)

第二个 Bug 是读写未分配的堆，其实和上面差不多，但是读写的内存地址不一样。如下图所示出问题的函数为 `FreeImage_GetFIFFromFilenameU`。

![](https://bbs.kanxue.com/upload/attach/201910/856782_37286A9RTWPAJHC.jpg)

从内存信息上来看访问了保留的内存地址。

![](https://bbs.kanxue.com/upload/attach/201910/856782_NQ6YM2KBURTE3UD.jpg)

样本信息如下：

![](https://bbs.kanxue.com/upload/attach/201910/856782_6Q7XJGGBD4VSWTA.jpg)

第三个 Bug 是堆溢出，出问题的函数是 `FreeImage_OutputMessageProc`，父函数为 `FreeImage_LoadU`，这里函数调用关系可能没有 IDA 的准确。

![](https://bbs.kanxue.com/upload/attach/201910/856782_4EFCRKJY6PEMJC7.jpg)

`edi = 0x11eadd08`，`esi = 0x11ea6000`，`0x11eadd08 + 0x2f3 - 11eae000 = -5`，从堆空间 +5 的地方开始溢出。

![](https://bbs.kanxue.com/upload/attach/201910/856782_38EURED24FEA9EZ.jpg)

最后一个是堆越界写入，出问题的函数为 `FreeImage_OutputMessageProc`，从 +1 的位置开始写入，写入的了 1 个 DWORD 大小的字节。

![](https://bbs.kanxue.com/upload/attach/201910/856782_FN85YSPTNBXUKFY.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/856782_K4F6GQMXNDHXKDW.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/856782_2MTWMC8HBMCRANT.jpg)

样本如下图所示：

![](https://bbs.kanxue.com/upload/attach/201910/856782_X2233J3S7XNTYZ6.jpg)

对比漏洞库的漏洞看看是否发现了旧的 Bug，通过仔细的分析相关 POC 后发现只有堆溢出被记录在 CVE 中。

![](https://bbs.kanxue.com/upload/attach/201910/856782_NEW9NBHWYCDRB7Q.jpg)

![](https://bbs.kanxue.com/upload/attach/201910/856782_72JC7EUW4DRCV74.jpg)

### WinAFL 使用感受

从以上简单的模糊测试过程可以看出 WinAFL 这款测试工具的局限性还是很大的，我总结出这么几个槽点：

(1) 利用 WinAFL 进行模糊测试首先必须保证源程序能够在命令行接受参数（`GetCommandLineA`）

(2) 如果要编写测试文件，必须保证通过 `LoadLibrary + GetProcAddress` 获取到的函数传入的参数不能有太强的逻辑性，比如需要父程序自定义虚函数表作为参数，以及需要传入一些根本不可能构造出的复杂数据结构

(3) 那为什么不直接用源程序测试呢，很多源程序支持命令行传入测试用例，但是 WinAFL 就是不支持，比如通过逆向了解源程序输入逻辑，包括完整的打开文件，操作文件以及关闭文件的流程，如果用 Winafl 测试这些函数，百分之八十的情况下会崩溃。并且如果找到了 `Main` 函数进行测试，那么速度又会太慢

(4) WinAFL 的覆盖率好像不支持子进程....

> 本次测试只是简单的 Fuzzing 了一下 ABC 看图软件的加载图片功能，但是效果并不是很理想。由于时间比较紧张，错误在所难免，请勿喷

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-10-26 14:00

被护花使者cxy\_编辑

，原因： 修改一些错误，完善文章
