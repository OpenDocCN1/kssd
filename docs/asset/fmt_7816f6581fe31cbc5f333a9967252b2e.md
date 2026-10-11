
# API HOOK

> 原文：[`www.pediy.com/kssd/pediy12/140580.html`](https://www.pediy.com/kssd/pediy12/140580.html)

第一次写文章，希望各位大侠，牛人体谅，呵。

`API HOOK` 是个好古老的话题了，但作为一个不懂算入门没有的新手，也只能从这些基本的学起。`HOOK API` 的作用主要是在原程序调用 `API` 是首先进入自己的处理过程，对调用结果进程一定处理后再按自己意愿返回。如比较经典的封包助手，就是 `HOOK` 了系统的 `SOCKET` 的 `recv` 函数，把包显示出来再调用返回给应用处理。

`OK`，下面我要 `HOOK` 的是 `EnumProcesses`，钩 `API` 有几种方式：

1. `API` 实现函数内钩，起作用范围广，对进程内所有位置内包括远程注入线程调用均有效。但钩的位置要比较猥琐才行，一般的会选择头几字节来钩，但说实在的，做一个简单的内存校验就可以发现被钩了。一般是选择中间位置来钩,或钩更下层的 `API`，如 `GetProcAddress` 这个，它会调用 `LdrGetProcedureAddress` 这个 `API`，所以可以选择钩这个，但要注意处理堆栈。当然，原应用做内存检测的话，还是逃不过被检测的命运的。

2. 可以在 `API` 调用位置上下钩，作用范围仅在于当前调用处有效，假如程序在另一地方有调用 `API` 则是钩不到的。严格来说这不算是钩 `API` 了，和一般的内联 `hook` 差不多。但在某些场合还是比较有用的。

3. 可以修改 `IAT` 表，作用范围仅在当前模块，这种方式比较常用，当然要小心原程序调用 `GetProcAddress` 来检查：

```
if [IATBase] != GetProcAddress(APIName) 
{  
    cout<<" 你的钩子被发现了   你的计算机即在 10s 内发生'砰'......................"; 
}
```

下面说下 `HOOK EnumProcesses` 过程，选用 `IAT HOOK`,操作系统为 `win server03`.先查 `MSDN`，实现语言是 `Delphi`。先看函数的原型:

```
BOOL WINAPI EnumProcesses(
  __out         DWORD* pProcessIds,
  __in          DWORD cb,
  __out         DWORD* pBytesReturned
);
```

看解释可以知道，第一个参数是一个进程 `ID` 数组指针，即返回当前系统的所有进程的 `ID`,第二个参数是这个数组的长度，注意它是按字节算的，这个参数一般可以用 `SizeOf(array)`来传入，因为 `SizeOf` 获取的是数组的字节长度。第三个参数是返回的数据长度，也是按字节计算，即获取的进程个数实际上是`(pBytesReturned*)/SizeOf(DWORD);`好了，下面再看看实际调用的汇编代码：

![](img/628cbb246ee3562e8a1681e49b7b63b6.jpg)

压栈顺序是从右至左,然后跟进这个 `CALL`：

![](img/067bd041c74743d3e0931abb0f142e10.jpg)

这个就是它的输入表的位置了，看看它的基址是：`0x00B2C700` .这个地址下面的值就是 `EnumProcesses` 的真正地址了：

![](img/ce2702acad2e4528e0a6773d7a8ba570.jpg)

为什么要有 `IAT` 表呢？因为 `DLL` 载入内存需要重定位的，会把真正的 `API` 地址放入这个基址下面。嗯，接下来的工作就简单了，我们把 `0x00B2C700` 这个地址下一值改成我们自己的处理函数就行了。我们要先写一个函数过程，注意参数要和原 `API` 一致。至于类型，一般来说 `WIN` 下的 `API` 都是指针操作，所以一般用 `DWORD` 就可以了，你自己的函数内使用时要记住它是指针就行。先复制代码:

```
/// <summary>
/// EnumProcesses 的过滤处理过程
/// 可以修改输出的结果
/// </summary>
/// <param name="pProcessIds">输出参数：进程 ID 数组指针</param>
/// <param name="cb">进程 ID 数组的初始化长度，以 byte 为单位</param>
/// <param name="pBytesReturned">输出参数：返回的数据大小，以 byte 为单位</param>
/// <param name="res">原 API 的处理结果，也压入，通过这个过程返回给原应用调用者</param>
/// <returns></returns>
function MYEnumProcessesFilter(pProcessIds: DWORD; cb: DWORD; pBytesReturned: DWORD; res: DWORD): DWORD; stdcall;
begin     
  Result:=res;   
end;

/// <summary>
/// 自己的 EnumProcesses 处理过程
/// </summary>
/// <param name="pProcessIds">输出参数：进程 ID 数组指针</param>
/// <param name="cb">进程 ID 数组的初始化长度，以 byte 为单位</param>
/// <param name="pBytesReturned">输出参数：返回的数据大小，以 byte 为单位</param>
/// <returns></returns>
function MyEnumProcesses(pProcessIds: DWORD; cb: DWORD; pBytesReturned: DWORD): Integer; stdcall;
asm    
    push pBytesReturned  
    push cb  
    push pProcessIds  
    mov eax,EnumProcesseApiAddr  
    call eax  
    push eax  
    push pBytesReturned  
    push cb  
    push pProcessIds  
    call MYEnumProcessesFilter  
end;

/// <summary>
/// HOOK EnumProcesses
/// </summary>
procedure BeginHookEnumProcesses; stdcall;
var
  dwAddr: DWORD;
  oldProtect: DWORD;
begin
  VirtualProtect(Pointer(EnumProcessIATBase), SizeOf(DWORD), PAGE_EXECUTE_READWRITE, oldProtect);
  dwAddr := DWORD(@MyEnumProcesses);
  asm      
     push ecx  
     push eax  
     mov ecx,EnumProcessIATBase//将 EnumProcesses 的输入表基址读入  
     mov eax, dword ptr ds:[ecx]  
     mov EnumProcesseApiAddr,eax//将原 EnumProcesses 的地址保存  
     mov eax,dwAddr   //将新地址读入  
     mov dword ptr ds:[ecx],eax//修改 IAT 的值  
     pop eax  
     pop ecx  
   end;
  VirtualProtect(Pointer(EnumProcessIATBase), SizeOf(DWORD), PAGE_EXECUTE_READ, oldProtect);
end;

/// <summary>
/// 恢复 EnumProcesses 的调用
/// </summary>
procedure EndHookHookEnumProcesses; stdcall;
var
  dwAddr: DWORD;
  oldProtect: DWORD;
begin
  VirtualProtect(Pointer(EnumProcessIATBase), SizeOf(DWORD), PAGE_EXECUTE_READWRITE, oldProtect);
  asm      
     push ecx  
     push eax  
     mov ecx,EnumProcessIATBase//将 EnumProcesses 的输入表基址读入  
     mov eax,EnumProcesseApiAddr   //将原地址读入  
     mov dword ptr ds:[ecx],eax//修改 IAT 的值  恢复为原 API 的调用  
     pop eax  
     pop ecx  
   end;
  VirtualProtect(Pointer(EnumProcessIATBase), SizeOf(DWORD), PAGE_EXECUTE_READ, oldProtect);
end;
```

其实上述代码很简单，先看 `BeginHookEnumProcesses` 这个开始 `HOOK` 的函数调用。先执行 `VirtualProtect`,这个 `API` 是修改某块内存的属性，如可读可写，要注意它的第三个参数，以前是用 `PAGE_READ`、`PAGE_READWRITE`,但现在多核了要用 `PAGE_EXECUTE_READ`,`PAGE_EXECUTE_READWRITE`。即先把把 `IAT` 表的基址所指向的内存块置为可写，`EnumProcessIATBase` 就是上述汇编代码中的基址 `0x00B2C700`(当然这基址每次编译 `EXE` 都会变).下面的代码就是把自己的 `MyEnumProcesses` 地址写入这基址，再恢复内存属性即可。同理 `EndHookHookEnumProcesses` 是把原 `API` 地址重新写入这个 `IAT` 基址下面。再看看那两个处理过程，其实我分两个是为了更分明些，`MyEnumProcesses` 仅是先调用原 `API`，然后把调用结果做参数，再压入 `MYEnumProcessesFilter` 来处理进程的过滤。所以 `MYEnumProcessesFilter` 需要额外一个参数 `res` 把原 `API` 调用的返回值也压入，然后在 `MYEnumProcessesFilter` 把这个返回值返回给用户。

再说下 `HOOK IAT` 时简单的防检测，一些应用会调用 `GetProcAddress` 来获取 `API` 地址，然后把这个地址和输入表基址如 `0x00B2C700` 下的地址做比较，就可以知道是否被 `HOOK` 了。所以应把 `GetProcAddress` 也钩上，在它获取 `EnumProcesses` 的地址时，把自己 的 `MyEnumProcesses` 地址返回给它。不过要注意钩 `GetProcAddress` 的处理效率和 `HOOK` 的方法哦，因为那肯定是一个线程不停的调用的，假如你内部处理慢了，嘿嘿，等着程序挂掉吧。 

感慨：这个简单的钩子，搞了足足两天多。另外还要注意 `WIN7`，特别是 `64` 位，你会发现用 `GetProcAddress` 获取 `EnumProcess` 的地址是不对的，要经过啥啥啥的转换。好麻烦。所以，可以考虑用特征码大法，把地址搜到再钩。新手作品，有大牛看到的话，不要太鄙视了，哈哈哈。。。
