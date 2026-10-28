
# [原创]Csgo(反恐精英:全球攻势)的游戏之旅----Dll注入和隐藏(Hacker) 上

说来话长，一直喜欢玩`csgo`游戏的我，前几天在同学的带领下，开始接触了外挂，`csgo`的`wg`封为两类，一波是`Dll`类型的，比如
`OneTap.su.dll`、`Osiris-1.dll`、`Kiddos-1.dl`等等、另一波是`Exe`类型的，比如大地球演员版、`V3`了 等等系列，后来，尝试过了注入器将`Dll`注入到`csgo`程序中，开始了"`wg`"游戏(纯粹是为了学习,不要模仿，不提供任何`DLL` `wg`版本文件，`wg`文件全部来自于外网)，劝大家不要开`g`，在我尝试了解`wg`的过程中，`csgo`账号被封停了。

最近`V`社，要对`CSGO`的引擎进行全面升级了，之前一直用的是起源`1` ，据说要升级成起源`2`，但不知道什么时候才能升级完成，还是据说起源`2`会有新的`VAC`机制，也导致全球的`wg`价格直接跌了很多，之前`wg`邀请码`2000$`的价格，现在也就几十了，所以趁着现在还能用的`wg`，就分析了一下。

但是不过`V`社这种，把院子大门给你打开，但在院子里装门禁的老态不知道能不能改变。（其实我是在说，不做游戏保护,但是封号的行为）

刚开始的时候，用的是外网下载的注入器，原本以为`csgo`会有防止注入的保护，就反汇编`IDA`了一下`ProjectInfinityInjector-1.exe`(一个注入器)，然而并没有发现他有什么牛`X`之处。

这是外网下载的`DLL`注入器，用`PEID`查看一下信息，发现是用`C#`写的。

后来我用到了`IDA`反汇编工具，查看了一下，虽然对`C#`支持不好，但是还是可以简单的看出来是通过什么来方法来注入的。

原本以为这个注入器可能会有过`Csgo`游戏保护的方法，后来发现我错了，原来`Valve`(`V`社)并没有做过防注入保护，直接就可以对原游戏程序进程注入，说到这，就不得不提我国鹅厂了，还是我国注重游戏的外挂保护啊，我觉得完全可以把游戏保护交给鹅肠来做(狗头保命，纯属玩笑)。

开上图的注入方法，其实就是`CreateRemoteThread`的方法注入的`DLL`，也很简单。但是完全没有想过自己开发一个注入器，因为不难，所以没做，但是这个注入器当你注入的时候，会给你谈广告，这个我是真的忍不了

所以我就自己写了一个注入器，还进行升级了一下，隐藏掉`dll`，这样可以躲过`5e`（国内最大的游戏匹配社区）外挂机制的检测了，用`MFC`写的，很简单，流程大概就是这样

> （一）注入外挂`DLL`到`csgo`游戏
> （二）注入`HideModule.DLL`到`csgo`游戏，通过断开`PEB`的三根链表，实现外挂`DLL`的隐藏
> （三）远程卸载`HideModule.DLL`，实现无痕迹。

## （一）注入外挂DLL到csgo游戏

### 第一步：获取进程ID(通过进程名字)

这里我是用`ToolHelp32`的方法来进行查找的，方法有很多，不过这种比较简单。

``` 
BOOL CInjectToCsgo::GetProcessIdByProcessImageName(HANDLE* ProcessID,	const TCHAR* ProcessImageName

	)

{

	BOOL IsOk = FALSE;

	HANDLE SnapshotHandle = INVALID_HANDLE_VALUE;

	PROCESSENTRY32 ProcessEntry32;

	ProcessEntry32.dwSize = sizeof(PROCESSENTRY32);	int LastError = 0;

	

	SnapshotHandle = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);	//TH32CS_SNAPPROCESS:Includes all processes in the system in the snapshot.To enumerate the processes,

	if (SnapshotHandle == INVALID_HANDLE_VALUE)

	{

		LastError = GetLastError();		return FALSE;

	}	if (!Process32First(SnapshotHandle, &ProcessEntry32))

	{



		LastError = GetLastError();		goto Exit;

	}	do

	{		if (_tcsicmp(ProcessEntry32.szExeFile, ProcessImageName) == 0)

		{

			*ProcessID = (HANDLE)ProcessEntry32.th32ProcessID;

			IsOk = TRUE;			goto Exit;

		}

	} while (Process32NextW(SnapshotHandle, &ProcessEntry32));



Exit:	if (SnapshotHandle != INVALID_HANDLE_VALUE)

	{

		CloseHandle(SnapshotHandle);

	}

	SnapshotHandle = INVALID_HANDLE_VALUE;

	SetLastError(LastError);	return IsOk;



}
```

### 第二步：打开进程获取句柄(通过进程ID、提权)

通过包装`OpenProcess`函数，提权然后打开进程，返回句柄

``` 
HANDLE CInjectToCsgo::OpenProcess(DWORD DesiredAccess, BOOL IsInheritHandle, HANDLE ProcessID)

{	if (m_EnableDebugPrivilege)

	{

		EnableSeDebugPrivilege(_T("SeDebugPrivilege"), TRUE);

	}

	HANDLE ProcessHandle = ::OpenProcess(DesiredAccess, IsInheritHandle, (DWORD)ProcessID);



	DWORD LastError = GetLastError();	if (m_EnableDebugPrivilege)

	{

		EnableSeDebugPrivilege(_T("SeDebugPrivilege"), FALSE);

	}

	SetLastError(LastError);	return ProcessHandle;

	

}



BOOL CInjectToCsgo::EnableSeDebugPrivilege(const TCHAR * PriviledgeName, BOOL IsEnable)

{

	BOOL IsOk = FALSE;	int  LastError = 0;	//获取当前进程句柄(伪句柄)

	HANDLE  ProcessHandle = GetCurrentProcess();

	HANDLE  TokenHandle = INVALID_HANDLE_VALUE;

	TOKEN_PRIVILEGES TokenPrivileges = { 0 };	//通过当前进程句柄获得当前进程中令牌句柄

	if (!OpenProcessToken(ProcessHandle, TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &TokenHandle))		/*BOOL OpenProcessToken(     ** 得到进程的令牌句柄

		 __in HANDLE ProcessHandle, //要修改访问权限的进程句柄

		 __in DWORD DesiredAccess, //指定你要进行的操作类型

		 __out PHANDLE TokenHandle //返回的访问令牌指针

)*/

	{

		LastError = GetLastError();		goto Exit;

	}

	LUID	 Luid;   //Locally Unique Identifier

	if (!LookupPrivilegeValue(NULL, PriviledgeName, &Luid))		// 通过权限名称查找uID

	//函数查看系统权限的特权值，返回信息到一个LUID结构体里

	/*BOOL LookupPrivilegeValue(

		LPCTSTR lpSystemName,    表示所要查看的系统，本地系统直接用NULL

		LPCTSTR lpName,          指向一个以零结尾的字符串，指定特权的名称

		PLUID lpLuid);           接收所返回的制定特权名称的信息*/

	{

		LastError = GetLastError();		goto Exit;

	}

	TokenPrivileges.PrivilegeCount = 1;		// 要提升的权限个数

	TokenPrivileges.Privileges[0].Attributes = IsEnable == TRUE ? SE_PRIVILEGE_ENABLED : 0;

	TokenPrivileges.Privileges[0].Luid = Luid;	if (!AdjustTokenPrivileges(TokenHandle, FALSE, &TokenPrivileges,		sizeof(TOKEN_PRIVILEGES), NULL, NULL))  // 启用或禁用特权一个有TOKEN_ADJUST_PRIVILEGES访问的访问令牌.

		/*BOOL AdjustTokenPrivileges(

			HANDLE TokenHandle, //包含特权的句柄  必须有 TOKEN_ADJUST_PRIVILEGES访问令牌

			BOOL DisableAllPrivileges,//禁用所有权限标志

			PTOKEN_PRIVILEGES NewState,//新特权信息的指针(结构体)

			DWORD BufferLength, //缓冲数据大小,以字节为单位的PreviousState的缓存区(sizeof)

			PTOKEN_PRIVILEGES PreviousState,//接收被改变特权当前状态的Buffer

			PDWORD ReturnLength //接收PreviousState缓存区要求的大小

		);*/

	{

		LastError = GetLastError();		goto Exit;

	}





	IsOk = TRUE;



Exit:	if (TokenHandle != INVALID_HANDLE_VALUE)

	{

		CloseHandle(TokenHandle);

		TokenHandle = INVALID_HANDLE_VALUE;

	}



	SetLastError(LastError);	return IsOk;

}
```

### 第三步：在csgo进程中申请内存，并将DLL路径写入目标进程地址空间

这里写入`DL`路径到目标进程是为了，`CreateRemoteThread`的时候，`LoadLibrary`的参数问题。

``` 
VirtualAddress = VirtualAllocEx(csgoHandle, NULL, BufferLength, MEM_COMMIT, PAGE_READWRITE);	if (VirtualAddress == NULL)

	{		goto Exit;

	}	if (ProcessMemoryWriteSafe(csgoHandle, VirtualAddress, FilePath.GetString(), BufferLength, &ReturnLength) == FALSE)

	{		

		goto Exit;

	}
```

``` 
BOOL CInjectToCsgo::ProcessMemoryWriteSafe(HANDLE ProcessHandle, LPVOID VirtualAddress, LPCVOID BufferData, SIZE_T BufferLength, SIZE_T * ReturnLength)

{

	SIZE_T  v1 = 0;

	SIZE_T* v2 = 0;	int    LastError = 0;

	DWORD  OldProtect = 0;

	BOOL   IsOk = FALSE;	if ((ProcessHandle == 0) || (VirtualAddress == 0) || (BufferData == 0) || (BufferLength == 0))

	{



		LastError = ERROR_INVALID_PARAMETER;		goto Exit;

	}	if (!ReturnLength)

	{

		v2 = &v1;

	}	else

	{

		v2 = ReturnLength;

	}	if (!WriteProcessMemory(ProcessHandle, VirtualAddress, BufferData, BufferLength, v2))		/*BOOL WriteProcessMemory(  ****函数能写入某一进程的内存区域

			HANDLE hProcess,       由OpenProcess返回的进程句柄。如参数传数据为 INVALID_HANDLE_VALUE 【即-1】目标进程为自身进程

			LPVOID lpBaseAddress,  要写的内存首地址 再写入之前，此函数将先检查目标地址是否可用，并能容纳待写入的数据

			LPVOID lpBuffer,       指向要写的数据的指针。

			DWORD nSize,           要写入的字节数。

			LPDWORD lpNumberOfBytesWritten  实际数据的长度

		);*/

	{		if (VirtualProtectEx(ProcessHandle, VirtualAddress, BufferLength, PAGE_EXECUTE_READWRITE, &OldProtect))

		{			if (WriteProcessMemory(ProcessHandle, VirtualAddress, BufferData, BufferLength, v2))

			{

				IsOk = TRUE;

			}			else

			{

				LastError = GetLastError();

			}

			VirtualProtectEx(ProcessHandle, VirtualAddress, BufferLength, OldProtect, &OldProtect);

		}		else

		{

			LastError = GetLastError();

		}

	}	else

	{

		IsOk = TRUE;

	}

Exit:



	SetLastError(LastError);	return IsOk;

}
```

### 第四步：远程注入线程

用`CreateRemoteThread`，将线程起始地址设为`LoadLibrary`的地址，至于为什么加载本地的`LoadLibrary`地址，在所有进程中(包括在不同电脑上)都能使用，我以后可能写一篇文章（如果记得起来！）

``` 
	HANDLE ThreadHandle = CreateRemoteThread

	(

		csgoHandle,  //线程所属进程的进程句柄

		NULL,           // 结构指定新线程的安全描述符，并确定子进程是否可以继承返回的句柄 NULL，则线程获取默认安全描述符，并且不能继承句柄

		0,              //线程栈初始大小,以字节为单位,如果该值设为0,那么使用系统默认大小.

		(LPTHREAD_START_ROUTINE)FunctionAddress, //在远程进程的地址空间中, 该线程的线程函数的起始地址.

		VirtualAddress, // 

		0,              //控制线程创建的标志

		NULL);          //线程的创建标志  NULL则不返回线程标识符.



	if (ThreadHandle == NULL)

	{

		

		VirtualFreeEx(csgoHandle, VirtualAddress, BufferLength, MEM_RELEASE);		goto Exit;

	}	//等待远程线程结束

	WaitForSingleObject(ThreadHandle, INFINITE);
```

## 测试结果:
