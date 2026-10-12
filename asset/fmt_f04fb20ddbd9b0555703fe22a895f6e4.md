
# WoW's 一点小诡计简单分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-515.htm`](https://www.pediy.com/kssd/pediy09/pediy09-515.htm)

> WoW's 一点小诡计简单分析

> 不说大家都知道什么游戏。不过我还是以 US 服务器为例来讨论问题。

> 前两天我做了一个软件，照常规方式 `OpenProcess` à `Read/WriteProcessMemory`。然后很不幸我的程序报错，给我说 `System Error : 5 On OpenProcess`。晕，我很确信我不可能代码有问题，因为这个模板我已经用了多次了。当时我还很郁闷。明明 `OD` 可以调试啊，不像是 `ANTI-DEBUG` 之类的啊，而且任务管理器也可以结束这个进程。郁闷了半天。而用 `Cheat Engine` 打开也提示 `OpenProcess failed`！这下晕了。我首先想到可能是全局钩子。迅速打开 `OD` 加载程序查找模块，我一个一个的眼睛弄得圆圆的也没有找到半个 `DLL` 的痕迹，一下子懵了，因为在 `Vista` 中默认是没有权限加载驱动的。这是怎么的呢？但是还是拿出 `IceSword` 这个尚方宝剑来弄了半天检查了一下 `SSTD`。结果很令人失望，啥都没有。这下我就奇怪了，在相关可能 `HOOK` 之类的 `API` 下断点，也没见程序断下…

> 突然想起那天打开 `IDA` 的时候不小心看到了几个导入函数。会不会是这个呢？

> ![](img/39cdfb64db6e99037063d6460382eeaa.jpg)

> 于是我在所有的这几个的函数参考下断点，然后顺利来到

> ![](img/1e9ea4e68ec1a0627312a2fc03692102.jpg)

> 经过测试发现只要经过了这个过程，就无法用程序 `OpenProcess` 打开。这种进程反外挂的方法还有点特别。至少本人觉得是。所以把它写成文章给大家分享：

> 这段代码很标准，应该是 `VC7` 或者 `VC7.1` 生成的。

> 我把它简单逆向：

> ```c
> BOOL Lock_CurrentProcess()
> {
>     HANDLE hProcess = ::GetCurrentProcess();
>     SID_IDENTIFIER_AUTHORITY sia = SECURITY_WORLD_SID_AUTHORITY;
>     PSID pSid;
>     BOOL bSus = FALSE;
>     bSus = ::AllocateAndInitializeSid(&sia,1,NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL,&pSid);
>     if(!bSus) goto Cleanup;
>     HANDLE hToken;
>     bSus = ::OpenProcessToken(hProcess,TOKEN_QUERY,&hToken);
>     if(!bSus) goto Cleanup;
>     DWORD dwReturnLength;
>     ::GetTokenInformation(hToken,TokenUser,NULL,NULL,&dwReturnLength);
>     if(dwReturnLength > 0x400) goto Cleanup;
>     LPVOID TokenInformation;
>     TokenInformation = ::LocalAlloc(LPTR,0x400); //这里就引用 SDK 的函数不引用 CRT 的了
>     DWORD dw;
>     bSus = ::GetTokenInformation(hToken,TokenUser,TokenInformation,0x400,&dw);
>     if(!bSus) goto Cleanup;
>     PTOKEN_USER pTokenUser = (PTOKEN_USER)TokenInformation;
>     BYTE Buf[0x200];
>     PACL pAcl = (PACL)&Buf;
>     bSus = ::InitializeAcl(pAcl,1024,ACL_REVISION);
>     if(!bSus) goto Cleanup;
>     bSus = ::AddAccessDeniedAce(pAcl,ACL_REVISION,0x000000FA,pSid);
>     if(!bSus) goto Cleanup;
>     bSus = ::AddAccessAllowedAce(pAcl,ACL_REVISION,0x00100701,pTokenUser->User.Sid);
>     if(!bSus) goto Cleanup;
>     if(::SetSecurityInfo(hProcess,SE_KERNEL_OBJECT,DACL_SECURITY_INFORMATION | PROTECTED_DACL_SECURITY_INFORMATION,NULL,NULL,pAcl,NULL) == 0)
>         bSus = TRUE;
> Cleanup:
>     if(hProcess != NULL)
>         ::CloseHandle(hProcess);
>     if(pSid != NULL)
>         ::FreeSid(pSid);
>     return bSus;
> }
> ```

> 这段代码就可以锁住其他进程打开本进程，当然也就防止了注入，和读写内存。

> 来段测试代码吧：

> ```c
> void TestFunc()
> {
>     BOOL bSus;
>     HANDLE hProcess;
>     hProcess = ::OpenProcess(PROCESS_ALL_ACCESS,FALSE,GetCurrentProcessId());
>     printf("Before Call Lock_AccessProcess OpenProcess Returns : 0x%08X \r\n",hProcess);
>     CloseHandle(hProcess);
>     bSus = ::Lock_CurrentProcess();
>     hProcess = ::OpenProcess(PROCESS_ALL_ACCESS,FALSE,GetCurrentProcessId());
>     printf("After Call Lock_AccessProcess OpenProcess Returns : 0x%08X ,System Error Code :%d",hProcess,GetLastError());
>     CloseHandle(hProcess);
>     system("PAUSE");
> }
> ```

> 执行后控制台输出：

> ```
> Before Call Lock_AccessProcess OpenProcess Returns : 0x00000034
> After Call Lock_AccessProcess OpenProcess Returns : 0x00000000 ,System Error Code :5 请按任意键继续. . .
> ```

> 我们大家都知道 `Code : 5`

> | `ERROR_ACCESS_DENIED`  
> `5` | Access is denied. |
> | :--- | :--- |

> 这一点就连 `WINDOWS` 的任务管理器也无法躲过：

> ![](img/53e4c1e63c291ee01e46cd251bcb6b40.jpg)

> 看到没有哈 用户名根本出不来，甚至你可以更绝点 `Denied ALL ACCESS` (`0xFFFFFFFF`) 就连结束都不可能了哈。

> 既然代码给出了还是论一下代码，因为我这个是逆向，就完全按照代码框架来写的其实。

> `::AllocateAndInitializeSid` 可以换成 `::InitializeSid`。因为我们并不需要初始化子 `Sid`。

> 另外。

> ```c
>     bSus = ::AddAccessDeniedAce(pAcl,ACL_REVISION,0x000000FA,pSid);
>     if(!bSus) goto Cleanup;
> 
>     bSus = ::AddAccessAllowedAce(pAcl,ACL_REVISION,0x00100701,pTokenUser->User.Sid);
> ```

> 实际上只需要下面的一句，或者干脆把它去掉，因为如果不添加 `Ace` 默认就是没有权限。既然这样上面的那句话 `AllocateAndInitializeSid` 也可以省掉，也似乎有些多余。这里可能有别的什么原因，不明，有知道的可以指点一下。
