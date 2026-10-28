
# [旧帖][原创]Run cmd.exe as Local System 0.00雪花

由于已经申请看雪的号很久了，发现自己还是临时会员，最近又在学习内核编程，遇到了一些问题想跟高手请教一下，却不能在其他版块发帖，于是想写这篇文章来申请一下邀请码。

前段时间看了一篇文章《Running CMD.EXE as Local System》，引自msdn的博客c25K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2L8i4y4V1L8W2)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6S2k6r3W2G2L8s2c8W2j5h3&6Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9H3y4q4)9J5c8U0p5I4i4K6u0r3x3U0N6Q4x3V1j5J5y4K6p5H3y4U0y4Q4x3X3g2S2M7%4m8^5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1^5i4K6R3%4i4@1q4m8i4@1f1#2i4@1t1%4i4@1t1I4i4@1f1#2i4@1q4q4i4K6W2q4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1@1i4@1u0m8i4K6R3$3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1#2i4K6W2o6i4@1p5^5P5s2m8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0g2Q4z5e0m8Q4b7f1k6Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0k6Q4z5f1y4Q4z5p5c8Q4c8e0g2Q4z5p5q4Q4b7e0q4Q4c8e0g2Q4b7e0c8Q4b7U0q4Q4c8e0S2Q4b7U0c8Q4b7e0g2Q4c8e0g2Q4z5p5c8Q4b7U0c8Q4c8e0S2Q4b7V1k6Q4z5e0S2Q4c8e0S2Q4z5o6y4Q4b7V1c8Q4c8e0c8Q4b7V1c8Q4b7V1k6U0L8h3c8Q4x3X3g2W2P5r3g2Q4c8e0c8Q4b7V1u0Q4b7e0g2K6P5i4y4@1k6h3#2Q4c8e0S2Q4b7V1q4Q4b7f1u0Q4c8e0c8Q4b7V1u0Q4b7V1c8Q4c8e0S2Q4b7V1k6Q4z5e0m8Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0c8Q4b7V1c8Q4z5o6k6Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0c8Q4b7U0W2Q4z5p5u0Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5e0y4Q4z5p5c8Q4c8e0c8Q4b7V1c8Q4z5f1y4Q4c8e0N6Q4b7U0y4Q4b7V1u0Q4c8e0N6Q4b7V1u0Q4z5f1k6Q4c8e0c8Q4b7U0S2Q4z5p5q4Q4c8e0c8Q4b7U0S2Q4z5p5c8Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0c8Q4b7V1q4Q4z5p5g2Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0W2Q4z5o6m8Q4z5f1q4Q4c8e0S2Q4b7V1k6Q4z5o6N6Q4c8e0g2Q4b7f1c8Q4b7e0k6Q4c8e0c8Q4b7U0W2Q4b7e0m8Q4c8e0y4Q4z5o6m8Q4z5p5q4Q4c8e0k6Q4b7U0N6Q4b7U0q4Q4c8e0g2Q4z5o6g2Q4b7e0g2Q4c8e0S2Q4b7e0N6Q4b7e0y4Q4c8e0k6Q4z5f1g2Q4z5e0m8%4K9h3&6V1L8%4N6K6i4@1f1$3i4K6V1K6i4K6S2p5i4@1f1@1i4@1u0p5i4K6W2o6i4@1f1%4i4@1t1K6i4@1u0n7i4@1f1%4i4@1u0n7i4K6W2r3i4@1f1K6i4K6R3H3i4K6S2n7i4@1f1@1i4@1t1^5i4@1q4p5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4@1q4o6i4@1q4o6i4@1f1#2i4K6W2n7i4K6W2n7i4@1f1%4i4@1q4n7i4@1p5H3i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6W2o6i4K6S2p5i4@1f1#2i4K6S2m8i4@1p5I4i4@1f1%4i4@1q4n7i4@1p5H3i4@1f1^5i4K6S2m8i4K6R3J5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4K6R3%4i4@1q4m8i4@1f1#2i4@1t1%4i4@1t1I4i4@1f1#2i4K6R3$3i4K6S2p5i4@1f1#2i4K6R3$3i4K6V1&6i4@1f1@1i4@1u0m8i4K6R3$3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1#2i4@1u0m8i4K6S2r3i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1#2i4K6W2o6i4@1p5^5P5s2m8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0g2Q4z5e0m8Q4b7f1k6Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0k6Q4z5f1y4Q4z5p5c8Q4c8e0g2Q4z5p5q4Q4b7e0q4Q4c8e0k6Q4z5o6S2Q4z5e0m8Q4c8e0g2Q4z5p5q4Q4z5f1k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4b7U0W2Q4b7U0k6Q4c8e0S2Q4z5o6y4Q4b7V1c8Q4c8e0c8Q4b7V1c8Q4b7V1k6U0L8h3c8Q4x3X3g2W2P5r3g2Q4c8e0c8Q4b7V1u0Q4b7e0g2K6P5i4y4@1k6h3#2Q4c8e0S2Q4b7V1q4Q4b7f1u0Q4c8e0c8Q4b7V1u0Q4b7V1c8Q4c8e0S2Q4b7V1k6Q4z5e0m8Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5f1y4Q4b7e0S2%4K9h3^5%4i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1^5i4K6R3K6i4@1u0p5i4@1f1^5i4@1u0r3i4K6V1H3i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1$3i4K6R3^5i4K6V1H3i4@1f1#2i4K6S2m8i4K6W2r3i4@1f1K6i4K6R3H3i4K6R3J5

## 下面是程序的思路：

### 1.首先是编写服务程序：

#### （1）服务主函数
任何一个应用程序都需要一个入口函数，一个服务程序必须具有服务主函数，服务主函数是服务启动时执行的入口，也是服务的主线程执行起点。服务主函数一般称作`ServiceMain`函数。但是服务主函数的名称与线程函数`ThreadProc`一样，其函数名并没有特殊要求，只是其参数接口和调用类型必须与要求一致。服务主函数的参数不是通过在命令行启动时设定的，而是通过`SCM`（服务控制管理器）的相关`API` `StartService`进行传递的。

#### （2）`StartServiceCtrlDispatcher`
`SCM`要对服务进行管理，就必须知道服务程序的服务主函数。服务程序通过调用`StartServiceCtrlDispatcher`函数设置服务主函数，同时通知`SCM`。`StartServiceCtrlDispatcher`函数原型如下：

`BOOL StartServiceCtrlDispatcher(const LPSERVICE_TABLE_ENTRY lpServicTable);`

结构`SERVICE_TABLE_ENTRY`的原型如下：

`typedef struct _SERVICE_TABLE_ENTRY{ LPTSTR lpServiceName; //服务名称 LPSERVICE_MAIN_FUNCTION lpServiceProc; //指向ServiceMain的函数指针 }SERVICE_TABLE_ENTRY,*LPSERVICE_TABLE_ENTRY;`

只有将函数的指针赋给`lpServiceProc`，再调用`StartServiceCtrlDispatcher`，这个函数就成为了服务主函数。

#### （3）控制处理函数：

1）控制处理函数`Handler`，控制处理函数用于处理`SCM`向服务传递的服务控制请求。控制处理函数原型如下：`VOID WINAPI Handler（DWORD fdwControl）`；与`ServiceMain`函数一致，其函数名没有特殊要求

2）注册控制管理函数：`RegisterServiceCtrlHandler`函数用于向`SCM`设置一个服务的控制处理函数

`SERVICE_STATUS_HANDLE RegisterServiceCtrlHandler( LPCTSTR lpServiceName; //服务名称 LPHANDLER_FUNCTION lpHandlerProc //为Handler函数指针）；`

#### （4）服务程序流程

先填充`SERVICE_STATUS`结构，然后注册服务控制请求处理历程，代码如下：

``` 
/*************************************
* VOID WINAPI SplSrvServiceStart (DWORD argc, LPTSTR *argv)
* 功能	服务启动函数
*
* 参数	未使用
**************************************/

VOID WINAPI SplSrvServiceStart (DWORD argc, LPTSTR *argv) 
{ 
	DWORD status; 
	DWORD specificError; 
	HANDLE hThread;
	// 填充SERVICE_STATUS 结构
	SplSrvServiceStatus.dwServiceType        = SERVICE_WIN32|SERVICE_INTERACTIVE_PROCESS;
	SplSrvServiceStatus.dwCurrentState       
		= SERVICE_START_PENDING;		// 服务在运行
	SplSrvServiceStatus.dwControlsAccepted   
		= SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_PAUSE_CONTINUE; 
	SplSrvServiceStatus.dwWin32ExitCode      = 0; 
	SplSrvServiceStatus.dwServiceSpecificExitCode = 0; 
	SplSrvServiceStatus.dwCheckPoint         = 0; 
	SplSrvServiceStatus.dwWaitHint           = 0; 
	// 注册服务控制请求处理例程
	SplSrvServiceStatusHandle = RegisterServiceCtrlHandler( 
		"Sample_Srv",	// 服务名，在创建服务时使用了
		// SERVICE_WIN32_OWN_PROCESS，因此本参数被忽略。
		SplSrvServiceCtrlHandler);	// 控制请求处理例程，函数名

	if (SplSrvServiceStatusHandle == (SERVICE_STATUS_HANDLE)0) 
	{ 
		SvcDebugOut(" [SPLSRV_SERVICE] RegisterServiceCtrlHandler "
			"failed %d\n", GetLastError()); 
		return; 
	}  
	// 初始化工作，本示例未使用，函数为空
	//status = SplSrvServiceInitialization(argc,argv, &specificError);  
	// 初始化出错，用户自行修改
	/*if (status != NO_ERROR) 
	{ 
		SplSrvServiceStatus.dwCurrentState       = SERVICE_STOPPED; 
		SplSrvServiceStatus.dwCheckPoint         = 0; 
		SplSrvServiceStatus.dwWaitHint           = 0; 
		SplSrvServiceStatus.dwWin32ExitCode      = status; 
		SplSrvServiceStatus.dwServiceSpecificExitCode = specificError; 

		SetServiceStatus (SplSrvServiceStatusHandle, &SplSrvServiceStatus); 
		return; 
	} */
	// 初始化完成，设置运行状态
	SplSrvServiceStatus.dwCurrentState       = SERVICE_RUNNING; 
	SplSrvServiceStatus.dwCheckPoint         = 0; 
	SplSrvServiceStatus.dwWaitHint           = 0; 

	if (!SetServiceStatus (SplSrvServiceStatusHandle, &SplSrvServiceStatus)) 
	{ 
		status = GetLastError(); 
		SvcDebugOut(" [SPLSRV_SERVICE] SetServiceStatus error %ld\n",status); 
	} 
	// 用户自行修改，用于完成服务的工作
	hThread=CreateThread(NULL,0,CmdService,NULL,0,NULL);
	SvcDebugOut(" [SPLSRV_SERVICE] Returning the Main Thread \n",0); 

	return; 
} 
```

线程函数如下：

```
DWORD WINAPI CmdService(LPVOID lpParam)
{
        ShellExecute(0,0,TEXT("cmd.exe"),0,0,SW_SHOW);
        return(0);
}
```

下面是入口主函数：

```
int WINAPI WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{ 
	// 设置SERVICE_TABLE_ENTRY 数据结构，以NULL 结构结束，
	// 作为StartServiceCtrlDispatcher 函数的参数。
	SERVICE_TABLE_ENTRY   DispatchTable[] = 
	{ 
		{ "Sample_Srv", (LPSERVICE_MAIN_FUNCTION) SplSrvServiceStart }, 
		{ NULL, NULL } 
	}; 
	if (!StartServiceCtrlDispatcher( DispatchTable)) 
	{ 
		SvcDebugOut(" [SPLSRV_SERVICE] StartServiceCtrlDispatcher (%d)\n", 
			GetLastError()); 
	} 
} 
```

加为红色的类型`SERVICE_INTERACTIVE_PROCESS`：允许该服务在控制台上显示窗口，并且接受用户的输入，刚开始我没加这个类型，运行时一直没看到`cmd`窗口，后来看书才明白是这样

### 2.接下来是编写对服务的控制和管理：

通过`OpenSCManager`获得服务控制管理器句柄，然后用`CreateService`创建服务，然后是启动服务，主要过程代码如下：
