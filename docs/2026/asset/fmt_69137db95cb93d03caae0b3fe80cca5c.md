
# [原创]淫戏植物大战僵尸篇之给游戏添加检测（一）

## 一. 简介

这款游戏就不介绍了，早被玩坏了，放一张检测的效果图:

![](https://bbs.kanxue.com/upload/attach/201712/750125_63jwgqi7i70slax.jpg)

### 这是免费植物的功能，原理相信大家都懂，结果弹出Crc检测到内存错误的的消息框。

## 二.设计思路及原理分析

- 劫持注入检测DLL
- 封装消息提示CALL
- 初始化检测
- 应用Crc检测及实现简单的数据校验

> （一） .为什么要采用劫击注入？

> （二） .封装什么提示函数？

> （三） .检测为什么要初始化？

> （四） .Crc校验的原理和数据校验的原理是什么？

### 为什么要采用劫持注入？

1. 劫持注入更为简单方便，不用控制注入的时机。
2. 劫持注入可以更根底的应用各种检测，不用担心没有权限。
3. 方便采集和备份原始数据

### 封装什么提示函数？

1. 有防必有攻，弹出消息框可以更为显著的对抗检测 Ps：不然直接退出岂不很猥琐 （/手动滑稽）
2. 寻找到游戏提示的消息框封装成函数方便调用 （强迫症患者的需要,不然直接`MessageBox()` 也行）
3. Patch掉鼠标焦点移动到游戏窗口外出现的暂停窗口

### 检测为什么要初始化？

1. 采集和备份原数据
2. 判断各类数据是否初始化以便于开启检测

### Crc校验的原理和数据校验的原理是什么？

1. Crc检测也称 “[循环冗余校验](https://bbs.kanxue.com/elink@761K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8V1y4d9b7#2)9J5c8U0p5@1y4e0x3K6y4e0W2Q4x3@1k6X3M7W2)9K6c8r3q4D9j5h3c8V1K9h3^5`.)“ ,这里我们采用的是`Crc32`
2. 采集主模块镜像`Crc`校验值对于原数据,不想等则非法内存
3. 数据校验的对象为阳光值 ,判断值是否为非法数据和对比短时间差阳光的变化值

## 三.收集数据和实现

### 编写劫持注入代码

（1）.查找适合的劫持对象

![](https://bbs.pediy.com/upload/attach/201712/750125_bo7rdf6lcezxahc.jpg)

劫持Dll为:`Version.dll` 路径为红色的为程序运行再加载进来DLL的，网上这个dll的劫持源码也很多

(2). 加载系统原DLL：

```
    HMODULE m_hModule = NULL;	// 原始模块句柄

    // 加载原始模块
    inline BOOL WINAPI Load()
    {
        WCHAR szPath[MAX_PATH]={};

        GetSystemDirectory(szPath,MAX_PATH);
        //StringCchCat( szPath,MAX_PATH,L"\\version.dll");
		wcscat_s(szPath, TEXT("\\version.dll"));
        m_hModule = ::LoadLibraryW(szPath);
        if (m_hModule == NULL)
        {
			WCHAR tzTemp[MAX_PATH];
			wsprintf(tzTemp, TEXT("无法加载 %s，程序无法正常运行。"), szPath);
			MessageBox(NULL, tzTemp, TEXT("version.dll"), MB_ICONSTOP);
        }

        return (m_hModule != NULL);	
    }

	// 卸载原始模块
    inline VOID UnLoad()
    {
        if ( m_hModule  )
        {
            ::FreeLibrary( m_hModule );
            m_hModule = NULL;
        }
    }

    // 获取原始函数地址
	FARPROC WINAPI GetFunAddress(PCSTR pszProcName)
    {
        FARPROC fpAddress;
        CHAR szProcName[16];
        TCHAR tzTemp[MAX_PATH];

        fpAddress = GetProcAddress(m_hModule, pszProcName);
        if (fpAddress == NULL)
        {
            if (HIWORD(pszProcName) == 0)
            {
                wsprintfA(szProcName, "%d", pszProcName);
                pszProcName = szProcName;
            }

            wsprintf(tzTemp, TEXT("无法找到函数 %s，程序无法正常运行。"), pszProcName);
			MessageBox(NULL, tzTemp, TEXT("version.dll"), MB_ICONSTOP);
            ExitProcess(-2);
        }

        return fpAddress;
    }
```

(3).获取系统原DLL导出函数地址:

```
FARPROC WINAPI GetAllExportFun()
{
	g_pfnGetFileVersionInfoA = GetFunAddress("GetFileVersionInfoA");
	g_pfnGetFileVersionInfoByHandle = GetFunAddress("GetFileVersionInfoByHandle");
	g_pfnGetFileVersionInfoExA = GetFunAddress("GetFileVersionInfoExA");
	g_pfnGetFileVersionInfoExW = GetFunAddress("GetFileVersionInfoExW");
	g_pfnGetFileVersionInfoSizeA = GetFunAddress("GetFileVersionInfoSizeA");
	g_pfnGetFileVersionInfoSizeExA = GetFunAddress("GetFileVersionInfoSizeExA");
	g_pfnGetFileVersionInfoSizeExW = GetFunAddress("GetFileVersionInfoSizeExW");
	g_pfnGetFileVersionInfoSizeW = GetFunAddress("GetFileVersionInfoSizeW");
	g_pfnGetFileVersionInfoW = GetFunAddress("GetFileVersionInfoW");
	g_pfnVerFindFileA = GetFunAddress("VerFindFileA");
	g_pfnVerFindFileW = GetFunAddress("VerFindFileW");
	g_pfnVerInstallFileA = GetFunAddress("VerInstallFileA");
	g_pfnVerInstallFileW = GetFunAddress("VerInstallFileW");
	g_pfnVerLanguageNameA = GetFunAddress("VerLanguageNameA");
	g_pfnVerLanguageNameW = GetFunAddress("VerLanguageNameW");
	g_pfnVerQueryValueA = GetFunAddress("VerQueryValueA");
	g_pfnVerQueryValueW = GetFunAddress("VerQueryValueW");
    return NULL;
}
```

(3).转发导出函数:

```
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// 宏定义
#define EXTERNC extern "C"
#define NAKED __declspec(naked)
#define EXPORT __declspec(dllexport)

#define ALCPP EXPORT NAKED
#define ALSTD EXTERNC EXPORT NAKED void __stdcall
#define ALCFAST EXTERNC EXPORT NAKED void __fastcall
#define ALCDECL EXTERNC EXPORT NAKED void __cdecl
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

namespace AheadLib
{
	ALCDECL GetFileVersionInfoA(void)
	{
		_asm jmp g_pfnGetFileVersionInfoA;
	}

	ALCDECL GetFileVersionInfoByHandle(void)
	{
		_asm jmp g_pfnGetFileVersionInfoByHandle;
	}

	ALCDECL GetFileVersionInfoExA(void)
	{
		_asm jmp g_pfnGetFileVersionInfoExA;
	}

	ALCDECL GetFileVersionInfoExW(void)
	{
		_asm jmp g_pfnGetFileVersionInfoExW;
	}

	ALCDECL GetFileVersionInfoSizeA(void)
	{
		_asm jmp g_pfnGetFileVersionInfoSizeA;
	}

	ALCDECL GetFileVersionInfoSizeExA(void)
	{
		_asm jmp g_pfnGetFileVersionInfoSizeExA;
	}

	ALCDECL GetFileVersionInfoSizeExW(void)
	{
		_asm jmp g_pfnGetFileVersionInfoSizeExW;
	}

	ALCDECL GetFileVersionInfoSizeW(void)
	{
		_asm jmp g_pfnGetFileVersionInfoSizeW;
	}

	ALCDECL GetFileVersionInfoW(void)
	{
		_asm jmp g_pfnGetFileVersionInfoW;
	}

	ALCDECL VerFindFileA(void)
	{
		_asm jmp g_pfnVerFindFileA;
	}

	ALCDECL VerFindFileW(void)
	{
		_asm jmp g_pfnVerFindFileW;
	}

	ALCDECL VerInstallFileA(void)
	{
		_asm jmp g_pfnVerInstallFileA;
	}

	ALCDECL VerInstallFileW(void)
	{
		_asm jmp g_pfnVerInstallFileW;
	}

	ALCDECL VerLanguageNameA(void)
	{
		_asm jmp g_pfnVerLanguageNameA;
	}

	ALCDECL VerLanguageNameW(void)
	{
		_asm jmp g_pfnVerLanguageNameW;
	}

	ALCDECL VerQueryValueA(void)
	{
		_asm jmp g_pfnVerQueryValueA;
	}

	ALCDECL VerQueryValueW(void)
	{
		_asm jmp g_pfnVerQueryValueW;
	}
}
```

(4).加载检测模块:

```
VOID LoadHookDll()
{
    WCHAR szPath[MAX_PATH] = {};
    GetModuleFileNameW( NULL, szPath, MAX_PATH );
    ::PathRemoveFileSpec( szPath );
	//OutputDebugString(szPath);
    StringCchCat( szPath, MAX_PATH, L"\\Pvz_Check.dll" );

    if ( ::PathFileExists( szPath ) )
    {
        HMODULE hModule = ::LoadLibraryW( szPath );
#ifdef _DEBUG
        WCHAR szBuffer[MAX_PATH] = {};
        StringCchPrintf( szBuffer, MAX_PATH, L"LoadLibraryW %X %s", hModule, szPath );
        OutputDebugString ( szBuffer );
#endif
    }
    else
    {
#ifdef _DEBUG
        WCHAR szBuffer[MAX_PATH] = {};
        StringCchPrintf( szBuffer, MAX_PATH, L"Can't Find %s",szPath );
        OutputDebugString ( szBuffer );
#endif
    }
}
```

劫持注入大概原理就是这样，这里推荐一款工具:`AheadLib` 因为懒我也用的工具生成的劫持代码

> https://bbs.pediy.com/thread-196111.htm

### 寻找及封装PvzMessageBox() 消息提示函数

(1).寻找信息提示CALL

![](https://bbs.kanxue.com/upload/attach/201712/750125_l213nr6yu8iesfg.jpg)

可以看到鼠标焦点离开游戏窗口时弹出这个信息提示，我们OD搜索字符串: `Click to resume game`

![](https://bbs.kanxue.com/upload/attach/201712/750125_qobym7eb7ohabhn.jpg)

找到字符串后双击进入，可以看到压入字符串常量的CALL都同一个，我们猜想应该是格式化字符串的。

![](https://bbs.kanxue.com/upload/attach/201712/750125_75n35bb46wc2qzm.jpg)

上面是格式化函数,那下面 call eax 就一定是提示Call了

![](https://bbs.kanxue.com/upload/attach/201712/750125_6tzct6484b6i43y.jpg)

可以看到ecx 为游戏基址的值，下面开始重载这个函数
