
# 2、实现细节

首先编写程序枚举消息钩子句柄，需要得到 `GUI TABLE`，它的地址实际上存储于 `User32.dll` 的一个全局变量中，该模块导出的函数 `UserRegisterWowHandlers` 将返回该全局变量的值。所以我们只要调用这个函数就能够得到 `GUI TABLE`。然而 `UserRegisterWowHandlers` 是一个未公开的函数，不确定它的函数原型，需要反汇编猜出它的原型。笔者反汇编后得到的原型如下。

```
typedef PSHAREDINFO (__stdcall *USERREGISTERWOWHANDLERS) (PBYTE, PBYTE);
```

仅知道它两个参数是两个指针，但是不知道它的两个参数的含义，所以我们无法构造出合理的参数。如果随便构造参数传进去又会导致 `user32.dll` 模块发生错误。所以通过调用这个函数接收其返回值的方法就不能用了。再次反汇编该函数的实现可以看出，在不同操作系统下该函数的最后三行代码如下。

**2K 系统:** (5.0.2195.7032)

```
:77E3565D B880D2E477 mov eax, 77E4D280
:77E35662 C20800 ret 0008
```

**XP 系统:** (5.1.2600.2180)

```
:77D535F5 B88000D777 mov eax, 77D70080
:77D535FA 5D pop ebp
:77D535FB C20800 ret 0008
```

**2003 系统:** (5.2.3790.1830)

```
:77E514D9 B8C024E777 mov eax, 77E724C0
:77E514DE C9 leave
:77E514DF C2080000 ret 0008
```

可以看到共同点，该函数的倒数第三行代码就是将保存 `GUI TABLE` 指针的全局变量值赋值给寄存器 `EAX`，只要我们想办法搜索到这个值即可。能够看出无论是哪个版本的函数实现中，都有 `C20800` 代码，含义是 `ret 0008`。我们可以自 `UserRegisterWowHandlers` 函数的入口地址开始一直搜索到 `C20800`，找到它以后再向前搜索 `B8` 指令，搜到以后 `B8` 指令后面的四个字节数据就是我们需要的数据。代码如下。

```
//获得 UserRegisterWowHandlers 函数的入口地址
DWORD UserRegisterWowHandlers = (DWORD) GetProcAddress(LoadLibrary("user32.dll"), "UserRegisterWowHandlers");

PSHAREDINFO pGUITable;  //保存 GUITable 地址的指针

for(DWORD i=UserRegisterWowHandlers; i<UserRegisterWowHandlers+1000; i++)
{
  if((*(USHORT*)i==0x08c2)&&*(BYTE *)(i+2)== 0x00)
  {     //已找到 ret 0008 指令，然后往回搜索 B8
    for (int j=i; j>UserRegisterWowHandlers; j--)
    {   //找到 B8 它后面四个字节保存的数值即为 GUITable 地址
      if (*(BYTE *)j == 0xB8)           
      {
        pGUITable = (PSHAREDINFO)*(DWORD *)(j+1);
        break;
      }
    }break;
  }
}
```

得到 `SHAREDINFO` 结构指针后，它的成员 `pServerInfo` 的成员 `cHandleEntries` 就是句柄的总个数，然后循环遍历每一个句柄，找到属于指定模块的消息钩子句柄。代码如下。

```
int iHandleCount = pGUITable->pServerInfo->cHandleEntries;

HOOK_INFO HookInfo;
DWORD dwModuleBase;

struct TINFO  
{
  DWORD dwProcessID;
  DWORD dwThreadID;
};

char cModuleName[256] = {0};

for (i=0; i<iHandleCount; i++)
{               //判断句柄类型是否为消息钩子句柄
  if (pGUITable->pHandleEntry[i].bType == TYPE_HOOK)
  {
    DWORD dwValue = (DWORD)pGUITable->pHandleEntry[i].pObject;
    //获得消息钩子内核对象数据
    GetKernelMemory(pGUITable->pHandleEntry[i].pObject, (BYTE *)&HookInfo, sizeof(HookInfo));
    
    W32THREAD w32thd;
    
    if( GetKernelMemory(HookInfo.pWin32Thread,(BYTE *)&w32thd , sizeof(w32thd)) )  
    {  //获取钩子函数所在模块的基址
      if (!GetKernelMemory((PVOID)((ULONG)w32thd.ppi+0xA8+4*HookInfo.iMod),
        (BYTE *)&dwModuleBase,   sizeof(dwModuleBase)))
      {
        continue;
      }
      
      TINFO tInfo;
      //获取钩子所属进程 ID 和线程 ID
      if (!GetKernelMemory((PVOID)((ULONG)w32thd.pEThread+0x1ec),
        (BYTE *)&tInfo,   sizeof(tInfo)))
      {
        continue;
      }
      
      HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, tInfo.dwProcessID);
      
      if (hProcess == INVALID_HANDLE_VALUE)
      {
        continue;
      }
      
      //根据模块基址,获取钩子函数所属模块的名称
      if (GetModuleFileNameEx(hProcess, (HMODULE)dwModuleBase, cModuleName, 256))
      {
        OutputDebugString(cModuleName);
        OutputDebugString("\r\n");
      }     
    }
  }
}
```

利用上面的代码就可以找到所属病毒 DLL 的消息钩子句柄，然后调用 `UnhookWindowsHookEx` 函数卸载这个消息钩子就 OK 了。

# 题外话

文章写完了，写点题外话。上述文章内容大部分摘自王倍昌的《计算机病毒揭秘与对抗》一书，该书已于 2011 年 10 月 1 日出版。该书揭秘了 Windows 系统下计算机病毒常用的技术和对抗技术。如：隐藏注入技术、SPI 网络劫持技术、DLL 劫持技术、浏览器绑架技术、服务劫持技术、感染型病毒技术、加壳实现与静态脱壳技术、反病毒虚拟机技术、主动防御技术等。欢迎阅读此书，并提出宝贵意见。关于此书的更多内容请查看 [www.safe163.com](http://www.safe163.com) 网站。

