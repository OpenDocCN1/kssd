
这一部分主要讲一下Hook的注意事项和部分大厂关于Hook的面经。部分面经之前讲解了，在这里不做赘述。

首先是二次HOOK，就是被别人HOOK了之后自己再次HOOK，这里可以提供4种方法，第一可以**换个位置HOOK**。**第二就是替换原HOOK**，也就是说将别人HOOK的指令修改为自己HOOK的指令。这样应该是比较有效的，但是需要注意的是修改指令数量一定要和对方的一致，或者修改之前将原来的HOOK还原，不然容易产生错误。**第三，在Detour函数中HOOK**，**第四，在Target函数中的原来HOOK的地址后面HOOK**。

第二是X64下HOOK应该注意什么？首先X64和X86本质区别就是地址总线上的差别，一个是2^64次，一次传输64位数据，一个是2^32次，一次传输32位数据。由此造成的差异就是内存地址大小问题，在32位机器上主要是4个字节，64位机器上就变成了8个字节。这样的话对于指针的使用就需要考虑到两个架构上的兼容性和差异性。例如在32下可以使用`ULONG`，但是在64位下使用`ULONG_PTR`。这样就可以有效避免由于编码问题产生的异常(或者统一使用`ULONG_PTR`)。第二就是PE格式上，由于x86和x64PE结构上存在微小差异，所以在进行AddressHook的时候需要注意。**第三可能涉及到跳转的问题**。

由于地址长度导致跳转指令长度变化。想`mov-jmp`就需要利用2+8+2的长度进行跳转，又像`push-ret`的方法，在32位系统下直接`push`就是32位数据，但是64位下只能`push`32位数据，这样的话，只能先`push`低位数据，然后修改高位数据，例如这样：`push 55667788h;mov [esp+4],11223344`。再如使用`jmp[addr]`方法。`FF25`类型`jmp`在X86平台下是一种绝对偏移的跳转，但是在x64下也是一种相对偏移的跳转。计算公式为当前`EIP`+0x6(指令长度)

检测HOOK：

1. HOOK修改的是内存中的数据，本地文件却没有修改。可以将本地文件加载到内存中，然后进行对比

2. 对内存模块进行`CRC`校验

3. 设置回调函数，检测某个`IAT`或者函数的前几个指令是否被修改

4. 对`VirtualProtect`函数和`WriteProcess`函数进行HOOK，检测修改内容的合法性

5. 利用`PsSetCreateProcessNotifyRoutineEx`注册回调函数，监控进程创建，对比特定的进程，如果创建，设置创建标志为假，创建失败

6. 利用`PsSetCreateThreadNotifyRoutine`注册回调函数，监控线程创建，通过进程路径.找到对应进程名.判断是否符合，如果是的话.找到回调函数地址( `pWin32Address = *(UCHAR**)((UCHAR*)Thread + 0x410);`)并改为`C3`

7. 利用`PsSetLoadImageNotifyRoutine`拦截模块，首先需要获取模块基地址(让其载入)，PE寻找基地址，解析到OEP，修改oep为`ret`即可

[https://bbs.pediy.com/thread-224514.htm](https://bbs.pediy.com/thread-224514.htm)

# 0x0 前言

1. 这是《一篇文章带你....》系列的第三篇，也是自己的学习总结，很多不懂的地方找的论坛前辈的资料。代码都是自己手敲，边敲边注释，对一些细节做了微调。前两篇是[一篇文章带你学会Armadillo脱壳](https://bbs.pediy.com/thread-251148.htm)和[一篇文章带你理解PE三表](https://bbs.pediy.com/thread-250924.htm)

2. 几个驱动相关的HOOK，从代码层面上讲理解的不是很深刻。但是手敲了一边除了一些驱动相关的知识，HOOK原理上应该有所领悟。

3. 还是一样，不提供源码下载，驱动相关的HOOK，前辈们都写有源码。不作伸手党。

4. 想来想去，这篇文章已经停滞1个多月了，今天草草结笔。关于HOOK检测部分估计需要鸽了。有空再学习吧。看雪第四版有源码。

5. 这篇文章是新瓶装旧酒，大佬别喷我。下一篇文章预计写注入，代码已经写完了，就剩下总结了，同样的还是新瓶装旧酒。旨在学习。下下一篇我也不知道写什么？加壳？花指令？？管他呢。

6. 唉，我什么时候才能成为像各位一样的大佬啊^\_^

# 0x1 AddressHook

## 0x1.1 IAT_HOOK

`IAT`是程序中存储导入函数地址的数据结构，如果HOOK了导入函数地址。就可以在函数调用的时候，将函数流程HOOK到我们指定的流程。但是我个人觉得这种方式最好要结合DLL注入的方式，如果单纯的使用HOOK，那么就需要将需要执行的操作的shellcode写入目标进程，如果操作复杂，可能需要的shellcode量特别大，**所以我们需要借助DLL注入**，这样就将我们需要执行的代码写入进程内部，在HOOK的Detour函数只需要实现`LoadLibrary`的操作。

`IATHOOK`的基本原理就是通过修改程序`IAT`数据结构，将原始调用API函数地址Target函数地址修改为Detour函数地址。所以`IAT_HOOK`需要实现以下几个步骤:

- 构造Detour函数

- 获取Target函数地址

- 通过PE获取Target函数所在的`IAT`的地址

- 保存原始的`IAT`地址和`IAT`地址所存储的内容

- 修改`IAT`地址中的数据

- 如果需要调用原来API函数，可以直接使用保存的API地址，可以就保证了HOOK的有效性

首先需要构造Detour函数，为了堆栈平衡和一些其他原因，最好Detour函数的函数原型和Target函数原型保持一致。

``` 
typedef int(WINAPI *PFN_MessageBoxA)(HWND hWnd, LPCTSTR lpText, LPCTSTR lpCaption, UINT uType);

int WINAPI My_MessageBoxA(

    HWND hWnd,

    LPCTSTR lpText,

    LPCTSTR lpCaption,

    UINT uType

)
```

这个函数的内容可以任意设置，主要执行两个方面的操作，**一是执行我们想进行的操作，二是控制Target函数的返回值。**为了简单期间，设置`MessageBox`作为HOOK，标志，`MessageBox`函数地址可以使用保存的API函数

``` 
    bReturn = OldMessageBox(NULL, "You Are Hooked", "Warning", MB_OK);

    //2.你可以控制API函数的返回值

    BOOL bReturn = FALSE;

    return bReturn;
```

第二歩是获取Target函数地址，这一步的目的是为了遍历`IAT`的时候比较`IAT`中所存储的Target函数地址。以便找到存放目标函数的`IAT`地址。

``` 
#ifdef _WIN64
    ULONG_PTR TargetFunAddr = (ULONG_PTR)GetProcAddress(hModule, szFuncName);
    PULONG_PTR lpAddr = NULL;
    SIZE_T size = sizeof(PULONG_PTR);
#else
    ULONG32  TargetFunAddr = (ULONG32)GetProcAddress(hModule, szFuncName);
    PULONG32 lpAddr = NULL;
    SIZE_T size = sizeof(PULONG32);
#endif
//MARKDOWN原因，不能以#开头，所以加个||符号，实际代码不存在||符号
```

第三步是获取Target函数的`IAT`地址，首先需要获取导入表的RVA，这里可以使用函数`ImageDirectoryEntryToData`获取

``` 
PIMAGE_IMPORT_DESCRIPTOR pImportDescriptor = (PIMAGE_IMPORT_DESCRIPTOR)    ImageDirectoryEntryToData(hModToHook,

        TRUE,

        IMAGE_DIRECTORY_ENTRY_IMPORT,

        &ulSize);
```

PE装载器已经将PE文件载入内存，可以使用`IAT`获取函数地址，所以通过`FirstThunk`指向的`IAT`遍历Target函数。需要判断`DLLNAME`是否是Target函数所在的模块，也就是说需要确定`IID`, **毕竟一个`IID`对应一个DLL**

``` 
while (pImportDescriptor->FirstThunk)
{
    //存放DllName
    szModName = (char*)((PBYTE)hModToHook + pImportDescriptor->Name);
    printf("[*]Cur Module Name:%s\n", szModName);
    //比较DLLName与目标DLL是否相同  使用stricmp函数是不区分大小写的
    if (stricmp(szModName, szModuleName) != 0)
    {
        pImportDescriptor++;
        continue;
    }
    ...
}
```

`First`指向的是一个`IAT`结构，存储的API函数地址。所以可以利用`PIMAGE_THUNK_DATA`(`IAT`)遍历Target函数。**如果找到了最好保存修改的IAT地址和数据**。

``` 
PIMAGE_THUNK_DATA pThunkData = (PIMAGE_THUNK_DATA)((BYTE*)hModToHook + pImportDescriptor->FirstThunk);

while (pThunkData->u1.Function)
{
    if ((*lpAddr) == TargetFunAddr)
    {
        //保存数据，修改为Detour函数地址
    }
}

//最好保存一下
if (pThunkPointer != NULL)   //保存修改内存的地址
{
    *pThunkPointer = lpAddr;
}

if (pOriginalFuncAddr != NULL)   //保存修改内存的数据，也就是Target函数的地址
{
    *pOriginalFuncAddr = *lpAddr;
}
```

**这里我实现了X64和X86的兼容，但是在实现X86的时候，出现了内存访问异常在`VirtuallProtest`处，但是在debug模式下程序没奔溃，需要那位大佬可以解答一下**

## 0x1.2 EAT_HOOK

使用`EAT_HOOK`需要注意一下两点：第一：`EAT`存储的是函数地址的偏移，**所以在HOOK EAT的时候需要加上基地址**，在写入`EAT`的时候，Detour地址需要减去`BaseAddress`。第二，`EAT`不对隐式链接起作用，**只对显示链接起作用**，也就是说对于那种`GetProcAddress`的那种调用起作用。

`EAT_HOOK`的原理和`IAT_HOOK`类似，都是通过修改函数地址数据从而HOOK。`EAT_HOOK`，也需要进行以下步骤：

1. 获取Target函数在`HookModule`上的`RVA`

2. 获取导出函数数组首地址

3. 遍历查找Target函数`RVA`

4. 切记在修改函数地址之前，需要保存`EAT`地址和原函数地址、

5. 将Detour函数地址写入`EAT`

首先是获取Target函数`RVA`，因为`EAT`存的是函数的`RVA`，所以，我们需要获取Target函数`RVA`

``` 
//1.获取Target函数在HookModule上的RVA
ULONG_PTR TargetFunAddr = NULL;
TargetFunAddr = (ULONG_PTR)GetProcAddress(hModToHook, szFuncName);    
ULONG_PTR TargetFunRVA = NULL;
TargetFunRVA = (ULONG_PTR)(TargetFunAddr - (ULONG_PTR)hModToHook);
```

第二歩是获取导出函数数组首地址，在`EAT->AddressOfFunctions`可以获取导出函数地址数组`AddressOfFunctions`的首地址

``` 
PIMAGE_EXPORT_DIRECTORY pExportDir = NULL;
pExportDir = (PIMAGE_EXPORT_DIRECTORY)ImageDirectoryEntryToData(hModToHook,//BaseAddress
        TRUE,
        IMAGE_DIRECTORY_ENTRY_EXPORT,   //Type
        &ulSize);                       //接收数据的大小

ULONG* FuncAddr = NULL;
FuncAddr = (ULONG*)((BYTE*)hModToHook + pExportDir->AddressOfFunctions);  //导出函数数组首地址
```

第三步是在`EAT`在寻找Target函数的`RVA`

``` 
if (FuncAddr[i] == TargetFunRVA)
{
    //修改内存保护属性
    DWORD OldProtect = NULL;
    if (VirtualProtect(&FuncAddr[i], sizeof(ULONG*), PAGE_EXECUTE_READWRITE, &OldProtect))
    {
        //修改保存数据
    }
}
```

第四步是保存`EAT`地址和Target函数`RVA`

``` 
*pAddrPointer = (PULONG_PTR)&FuncAddr[i];
*pOriginalFuncAddr = FuncAddr[i];
```

**`EAT`保存的是Target函数的`RVA`**，这一定要记住。所以在写入Detour函数的时候，是需要减去`BaseAddress`的。

``` 
//5.将Detour函数地址写入EAT
//因为EAT里面保存的是函数地址RVA值，所以在写入Detour函数地址需要减去BaseAddress
FuncAddr[i] = (ULONG)((ULONG_PTR)DetourFunc-(ULONG_PTR)hModToHook);
```

## 0x1.3 VirtualFunctionHook

C++虚函数存在的意义是为了方便使用多态性。在实现虚函数Hook的时候需要注意如下问题：1.在构建`DetourFun`函数的时候，一定要构造`DetourClass`，因为在调用虚函数的时候使用了`Thiscall`的函数调用约定，如果直接调用`detourfun`函数应该使用的标准调用约定，两者不统一，会出错。2.当使用`Trampolinefun`回调的时候，需要重新实例化一个`TrampolineClass`。

第一步：仍然是构造`DetourClass`类和`TrampolineClass`类

``` 
//因为使用了This的调用方法，所以在Hook的时候同时需要创建DetourClass类，保证函数约定是一致的
class DetourClass
{
public:
    virtual int DetourFun(int a, int b);
};

class TrampolineClass
{
public:
    virtual int TrampolineFun(int a,int b)
    {
        printf("TrampolineFun");
        return a + b;
    }
};

//此处构造DetourFun
int DetourClass::DetourFun(int a, int b)
{
    //此处执行自定义操作
    MessageBox(NULL, "Hooked", "warning", MB_OK);
    //调用TrampolineFun，首先需要将TrampolineClass实例化
    TrampolineClass *pTrampoline = new TrampolineClass;
    int iRet = pTrampoline->TrampolineFun(a, b);
    delete pTrampoline;
    return iRet+10;
}
```
