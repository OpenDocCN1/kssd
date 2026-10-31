
# [原创]年末知识分享 Windows 操作系统的安全性

记录下学习进度，为了看这块内容，最好学会如何调试`Ring3`发起的那一次系统调用；

我这次调试的是`nt!NtOpenProcess`，这个函数很多线程都会去调用，那么如何固定到特定的`Ring3`进程的堆栈调用是很有技巧的~

测试代码很简单：

```
int _tmain(int argc, _TCHAR* argv[])

{

	HANDLE procSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

	if(procSnap == INVALID_HANDLE_VALUE)

	{



		printf("CreateToolhelp32Snapshot failed, %d ",GetLastError());

		return 0;

	}



	DWORD dwPid = 0;

	PROCESSENTRY32 procEntry = { 0 };

	procEntry.dwSize = sizeof(PROCESSENTRY32);

	BOOL bRet = Process32First(procSnap,&procEntry);

	while(bRet)

	{

		if (!_wcsicmp(procEntry.szExeFile, L"csrss.exe")) 

		{

			dwPid = procEntry.th32ProcessID;

			__asm int 3;

			HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid);

			if (hProcess)

			{

				printf("open csrss.exe success dwPid:%d\r\n", dwPid);

				CloseHandle(hProcess);

			}

		}

		bRet = Process32Next(procSnap, &procEntry);

	}





	CloseHandle(procSnap);

	getchar();



	return 0;

}
```

这个`int 3`是用来让`OpenProcess`这次调用离第一次下 `nt!NtOpenProcess` 断点时机很近；保证下一次很大可能性可以拦住这个`Ring3`的调用堆栈

到断点的时候确认下 当前的`eprocess`是否为你的`Ring3`进程

紧接着 `bd` 这个断点，确保以后其他线程的调用不会被调试器捕获到，否则你会很难受~

再来还有一点，如果有循环语句，你企图绕过它，那么不要下断点哦，否则你又会跑到其他线程里去，也会很难受~，这个你就得自己想想办法啦，实在不行单步

本文给大家分享的是内核对象的安全性校验知识，不涉及多么牛逼的技术，希望对新手有所帮助，因为在学习这块内容的时候资料真少，希望大家不要走弯路~

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Security-Subsystem.pdf](javascript:void(0);)

  （568.64kb，221次下载）
