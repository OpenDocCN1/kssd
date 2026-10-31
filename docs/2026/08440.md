
# [讨论]星际2的防调试手段。

星际2在启动的时候，会创建2个exe进程。

这两个互相监视，关闭某一个，另外一个都会关掉。并且2个都无法用`od`附加。

我对比了进程里的那两个exe，虽然都来自一个`sc2.exe`，但是明显不是一个文件。

首先我先hook了`WriteProcessMemory`：

```
BOOL __stdcall Dbg::WriteProcessMemory_Hook(HANDLE hProcess, LPVOID lpBaseAddress, LPCVOID lpBuffer, SIZE_T nSize, SIZE_T *lpNumberOfBytesWritten)
{
	char szBuffer[1024] = "";
	
	sprintf_s(szBuffer, sizeof(szBuffer), "Process: %x\nBaseAddr: %x\nBuffer: %x\nSize: %x", hProcess, lpBaseAddress, lpBuffer, nSize);
	MessageBox(0, szBuffer, 0, 0);
	
	IMAGE_DOS_HEADER* pDos = (IMAGE_DOS_HEADER*)lpBuffer;
	
	if(pDos->e_magic == 0x5A4D)
	{
		DWORD dwWritten;
		HANDLE hFile = CreateFileA("C:\\CHNHack.exe", GENERIC_WRITE, FILE_SHARE_WRITE, NULL, CREATE_NEW, NULL, 0);
		WriteFile(hFile, lpBuffer, 0x400 * 8, &dwWritten, NULL);
		CloseHandle(hFile);
	}
	
	return WriteProcessMemory(hProcess, lpBaseAddress, lpBuffer, nSize, lpNumberOfBytesWritten);
}
```

将动态写出来的文件dmp出来。

然后hook `CreateProcessW`：

```
BOOL __stdcall Dbg::CreateProcessW_Hook(LPCWSTR lpApplicationName, LPWSTR lpCommandLine, LPSECURITY_ATTRIBUTES lpProcessAttributes, LPSECURITY_ATTRIBUTES lpThreadAttributes, BOOL bInheritHandles, DWORD dwCreationFlags, LPVOID lpEnvironment, LPCWSTR lpCurrentDirectory, LPSTARTUPINFOW lpStartupInfo, LPPROCESS_INFORMATION lpProcessInformation)
{
	USES_CONVERSION;
	std::string hack(W2A(lpCommandLine)); 
	replace_all(hack, "SC2.exe", "CHNHack.exe"); //我写的一个字符串替换方法
	LPWSTR tmp(A2W(hack.c_str()));
	CreateProcessW(lpApplicationName, tmp, lpProcessAttributes, lpThreadAttributes, bInheritHandles, dwCreationFlags, lpEnvironment, lpCurrentDirectory, lpStartupInfo, lpProcessInformation);
	MessageBoxW(0, L"断下来1", 0, 0);
	MessageBoxW(0, L"断下来2", 0, 0);
	return true;
}
```

他`CreateProcessW`的方法是`DEBUG_PROCESS|CREATE_SUSPENDED|DETACHED_PROCESS`。这个dmp出来的数据其实是一个很普通的控制台程序。

后面，我把系统的`notepad.exe`覆盖过去，改名`CHNHack.exe`，发现星际2还是无法调试，这个记事本也无法附加。

我就纳闷了，这个另外启动的线程是干嘛的啊？？？？？

另外，

```
MessageBoxW(0, L"断下来1", 0, 0);
MessageBoxW(0, L"断下来2", 0, 0);
```

在这里，弹出第一个`MessageBoxW`的时候，游戏进程`SC2.exe`是可以附加的，这个时候再`bp MessageBoxW`可以跟到代码区。

可是这样跟进去了，od把调试给占用了，可能他本身要调试占死主程序失效。那么联网的时候又不行了。

我说的比较乱，不知道有没有在搞星际2的人能看懂。

直接`CreateProcessW`里，我试过了`ExitThread(0);` 这样是可以调试主程序的。来达到模式单人模式已经不是很困难了。关键是只要一联网，程序就关掉了，估计他有判断他的附属线程是不是打开的。

这个`od`不能附加的问题，哪个大大能够搞定嘛？一起交流交流。

单机游戏破解，非外挂，应该没违规吧？![](https://bbs.kanxue.com/view/img/face/27.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
