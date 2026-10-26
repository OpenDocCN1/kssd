
# [原创]Everedit的试用限制绕过

因为蜜汁自信导致我在短短两天内重装了7次系统，导致Everedit当年的注册码生成次数耗尽，所以想着先把试用限制过了凑合着用到明年。

## 1.分析

从4.0版本开始授权文件就没有保存在本地的选项了，都是保存在注册表里，试用限制也是如此，先跟一下。

其实都不用查壳都知道是vmp，先拉到x64dbg里面对`RegQueryValueExW()`跨模块调用里面的关键函数下断：

![](https://bbs.kanxue.com/upload/attach/202008/615441_3K52SD3AQ4A5MRR.jpg)

跑起来以后断下了：

![](https://bbs.kanxue.com/upload/attach/202008/615441_VHQE3PRMSF3XYUG.jpg)

`RDX`应该是第二个参数: `lpValueName`（在x64下对函数的调用都是优先使用寄存器的，第一个参数是`RCX`），`license`不是我们想要的，继续跑。F9以后直接跑飞了，说明试用限制的call被加密了。直接对`kernelbase.RegQueryValueExW()`这个函数下断，并记录他的`RBX`。跑了一遍以后发现了一个可疑的字符串：

![](https://bbs.kanxue.com/upload/attach/202008/NRYANUAYEPNKTFD.jpg)

跟踪他的来源，发现调用果然被加密了：

![](https://bbs.kanxue.com/upload/attach/202008/615441_RG8Q8PK526H4BJ5.jpg)

但是我们的目的已经达到了，打开注册表编辑器，看看这个项到底有什么玄机：

![](https://bbs.kanxue.com/upload/attach/202008/615441_EFQHRTTKXXJCZMA.jpg)

我觉得这个默认项很像时间戳啊，于是转换之：

`5ECCEFBD -> 1590489021 -> 2020-05-26 18:30:21`

那我们把试用条件改成今天0点试试：

`2020-08-10  00:00:00 -> 1596988800 -> 5F301D80`

发现还是提示过期，尝试将`Value`这个值删除，能正常打开了。

分析部分就到此结束了，下面开始敲代码

## 2.编码

采用劫持导入表hook的方案。

由前面的分析已经可以大致确定几个关键点了：

- `RegQueryValueExW()`
- `L"ZQBKAF8AMQA0ADIANGA4"`

由于我们并没有去找机器码的算法，所以要做一些额外的工作。

大致思路如下：

1. hook `advapi32.RegOpenKeyW()`获取项的名称和`HKey`用于比对。
2. hook `kernelbase.RegQueryValueExW()`欺骗程序，告诉它：值“`Value`”并不存在；生成一个合法的“时间点”，然后改写函数的参数。

`dllmain.cpp`：

```cpp
// dllmain.cpp : 定义 DLL 应用程序的入口点。

#include "framework.h"

#include "DllModule.h"



extern "C" __declspec(dllexport) void Out();

void Out()

{

	OutputDebugStringW(L"Out");

}

namespace

{

	bool is_init = false;

}



BOOL APIENTRY DllMain(HMODULE hModule,

	DWORD  ul_reason_for_call,

	LPVOID lpReserved

)

{

	if (DLL_PROCESS_ATTACH == ul_reason_for_call)

	{

		UNREFERENCED_PARAMETER(lpReserved);

		DisableThreadLibraryCalls(hModule);

		if (is_init)

		{

			return TRUE;

		}

		if (nullptr == g_DMod)

		{

			g_DMod = std::make_unique<DllModule>();

		}

		g_DMod->Attach();

		is_init = true;

	}

	else if (DLL_PROCESS_DETACH == ul_reason_for_call)

	{

		if (nullptr == g_DMod)

		{

			return TRUE;

		}

		g_DMod->Detach();

		if (is_init)

		{

			is_init = false;

		}

	}

	return TRUE;
}
```

创建一个名为“`Out`”的导出函数用于导入表劫持。

`DllModule.h`：

```cpp
#pragma once

#include <cstdint>

#include <memory>



class DllModule

{

public:

	DllModule() = default;

	virtual ~DllModule() = default;

	static void Attach();

	static void Detach();

private:

	

};

using DllModulePtr = std::unique_ptr<DllModule>;

extern DllModulePtr g_DMod;
```

`DllModule.cpp`：

```cpp
#include "DllModule.h"

#include "hooks.h"



DllModulePtr g_DMod;



void DllModule::Attach()

{

	hooks::Install();

}



void DllModule::Detach()

{

	hooks::UnInstall();
}
```

`hooks.cpp`:
