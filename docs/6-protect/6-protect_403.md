

# hook api 反 OD 调试的一种思路

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-209.htm`](https://www.pediy.com/kssd/pediy09/pediy09-209.htm)

> hook api 反 OD 调试的一种思路  
> by 胡华  
>   
> 本程序在 backer 的耐心指导下完成，非常感谢 backer 的帮助！！！  
> 关于 OD 怎样在程序中下断点相信大家都知道，就是用 WriteProcessMemory 函数在要下断点的地方写入 0xCC，也就是 int 3 的中断，  
> 如果我们挂钩 OD 的 WriteProcessMemory 函数不让它下断点是不是就可以让 OD 不能正常跟踪了呢？有了这个思路，下面就是实践了。  
>   
> 首先得解决几个问题：  
> 1.我们的目标是最好在 OD 正常加载本程序后就已经被 hook,怎么解决？  
>   
> 这个问题其实很简单，我们知道全局类的初始化工作在 main 函数之前，OD 正常加载后会停在 WinMain 函数处，如果把 hook api 代  
> 码放在一个全局类的构造函数中就可以了，呵呵，就这么简单。  
>   
>   
> 2.要得到 OD 的输入表中的 WriteProcessMemory 函数的跳转地址  
>   
> 呵呵，用 OD 来调试 OD 不就很容易得到了么？  
> 用 OD 调试 OD 后我们得到 WriteProcessMemory 函数的跳转地址放在 0x0050D450 这个地方。  
>   
> 3.如果前面的两个问题解决，现在剩下的就是程序在运行期间要判断自己的父进程是否是 OD,如果是 OD 那么就可以肯定被 OD 加载，  
> 问题是怎么方便的取得父进程？如果用列取进程列表的方式来获的父进程，这比较麻烦，但是的确是一种思路。我们不采取这种  
> 思路，其实在 ntdll.dll 中有一个函数可以得到父进程，下面用代码来说话吧：  
>   
> NTDLL.DLL 中有一个函数叫 NtQueryInformationProcess，用它可以将指定类型的进程信息拷贝到某个缓冲。其原型如下：  
> NTSYSAPI  
> NTSTATUS  
> NTAPI  
> NtQueryInformationProcess (  
> IN HANDLE ProcessHandle, // 进程句柄  
> IN PROCESSINFOCLASS InformationClass, // 信息类型  
> OUT PVOID ProcessInformation, // 缓冲指针  
> IN ULONG ProcessInformationLength, // 以字节为单位的缓冲大小  
> OUT PULONG ReturnLength OPTIONAL // 写入缓冲的字节数  
> );  
>     第一个参数是希望操作的进程句柄，这个句柄必须以 PROCESS\_QUERY\_INFORMATION 模式存取。为了取得一个句柄，我们必须用  
> OpenProcess 函数：  
> HANDLE hProcess = OpenProcess(PROCESS\_QUERY\_INFORMATION,FALSE,dwProcessID);  
>     第二个参数是请求信息的类型，这个参数可以有许多个值，本文例子中将用 ProcessBasicInformation （值为 0）。  
>     因此，如果第二个参数是 ProcessBasicInformation 的话，则第三个参数必须为一个指针指向结构 PROCESS\_BASIC\_INFORMATION：  
> typedef struct  
> {  
>       DWORD ExitStatus; // 接收进程终止状态  
>       DWORD PebBaseAddress; // 接收进程环境块地址  
>       DWORD AffinityMask; // 接收进程关联掩码  
>       DWORD BasePriority; // 接收进程的优先级类  
>       ULONG UniqueProcessId; // 接收进程 ID  
>       ULONG InheritedFromUniqueProcessId; //接收父进程 ID  
> } PROCESS\_BASIC\_INFORMATION;  
>   
>   这个结构的最后一个参数是 InheritedFromUniqueProcessId，它就是我们所要的东西。  
>   
> 得到了父进程 id 之后，呵呵，现在开始打开进程了  
>         DWORD dwId=GetCurrentProcessId();  
>   DWORD dwParentId;  
>         dwParentId=GetParentProcessID(dwId); //获取父进程 id  
>   hParentProcess=OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwParentId);  
> 又有一个问题怎么判断是否是 OD 呢？  
> 我们可以用读取内存的方式查看某一段代码是不是 OD 的就行了，要检查哪段代码那就随你了，  
> 我这里是读取 OD 的 0x00401000 开始的几个字节。  
> 更简单的方法是读取 0x0050D450 这个地址判断是不是 WriteProcessMemory 这个函数的真实跳转地址，如果是那就基本可以断定是 OD 了，呵呵，不会那么巧其它的有的程序也会是这个地址吧！  
>   
> 判断是 OD 后，现在要做的就是在 OD 中申请一段空间把要执行的代码写在这个地址了  
> BYTE bRet=0xC3; //这是 hook api 后转到这要执行的代码  
> LPVOID lpWriteFunAddress=(void\*)0x0050D450;  
> LPVOID lpNewWriteFunAddress=NULL;  
> lpNewWriteFunAddress=VirtualAllocEx(hParentProcess,NULL,sizeof(bRet),MEM\_COMMIT,PAGE\_READWRITE);  
> //插入函数返回代码  
> if(lpNewWriteFunAddress==NULL)  
> {  
>    MessageBox(NULL,"virtualalloc err","error",MB\_OK);  
>         
> }  
> if(WriteProcessMemory(hParentProcess, lpNewWriteFunAddress, &bRet, sizeof(bRet), NULL)==0)  
> {  
>   MessageBox(NULL,"insert retn err","error",MB\_OK);        
> }  
> 写入后现在就是把 0x0050D450 这个地址的 WriteProcessMemory 函数的真实跳转地址改为我们上面申请的地址了：  
>        //改变 api 地址  
>       
>   DWORD flOldProtect;  
>   VirtualProtect(lpWriteFunAddress,sizeof               (lpWriteFunAddress),PAGE\_READWRITE,&flOldProtect);  
>   if(WriteProcessMemory(hParentProcess, lpWriteFunAddress, &lpNewWriteFunAddress,                                                sizeof(lpNewWriteFunAddress), NULL)==0)  
>   {  
>        MessageBox(NULL,"insert fun addr err","error",MB\_OK);  
>         
>   }  
>   
> 不知道大家注意没，我插入的要执行的代码只有一个字节 BYTE bRet=0xC3，  
> 其实就是 ret，呵呵，知道了吧，只要在 OD 中按 F2 下断点就会执行 WriteProcessMemory 这个函数，而这个函数现在是什么都不做就直接返回了，也不用考虑堆栈平衡，我们要的就是 OD 悄无声息的挂掉！如果想忽悠一下而不让 OD 挂掉，那就用 ret14 了，大家尽情的发挥吧！！！  
>   
> 好了，到这里已经结束了，再次感谢 backer 的指导！！！ 

> od 可以选择停在 system breakpoint 或者 entrypoint of program  
> bp OpenProcess 你这个就挂了  
>   
> btw: 只是修改 writeprocessmemory 可以绕开，参考 themida 

> 其次最简单的办法就是我以前写过的插件  
> OllgDbg 为 Admin 权限  
> 把被调试的程序的权限令牌弄成 Users 就行了。  
> 操作系统会帮你返回 Access Denied.  
> 目前因为窗体没有权限表，所以说要判断还是从窗体判断最好。 

 4dd466d0e1e0cc123afbe504da2462ca