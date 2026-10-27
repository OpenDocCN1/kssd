
```cpp
if(FUN1())FUN0();if(FUN2()) FUN3();if(FUN3()) FUN1();if(FUN1()) FUN0();}

#define _FLOWER_FUN_1 {if(FUN3())FUN1();if(FUN1()) FUN2();if(FUN2()) FUN0();if(FUN0()) FUN1(); \
                          if(FUN2())FUN1();if(FUN0()) FUN3();if(FUN1()) FUN2();if(FUN3()) FUN1();}
```

If语句得到的值不会去影响下面的判断，只要最后return 0就行。

### 2. 花指令2

`asInvoker`：

父进程是什么权限级别，那么此应用程序作为子进程运行时就是什么权限级别。

默认情况下用户启动应用程序都是使用 Windows 资源管理器（`explorer.exe`）运行的；在开启了 UAC 的情况下，资源管理器是以标准用户权限运行的。于是对于用户点击打开的应用程序，默认就是以标准用户权限运行的。如果已经以管理员权限启动了一个程序，那么这个程序启动的子进程也会是管理员权限。典型的情况是一个应用程序安装包安装的时候使用管理员权限运行，于是这个安装程序在安装完成后启动的这个应用程序进程实例就是管理员权限的。有时候这种设定会出现问题，你可以阅读 在 Windows 系统上降低 UAC 权限运行程序（从管理员权限降权到普通用户权限）。

`requireAdministrator`

此程序需要以管理员权限运行。在资源管理器中可以看到这样的程序图标的右下角会有一个盾牌图标。

![](https://bbs.kanxue.com/upload/attach/202002/856851_KREAZBYNBTWZZCY.jpg)

用户在资源管理器中双击启动此程序，或者在程序中使用 `Process.Start` 启动此程序，会弹出 UAC 提示框。点击“是”会提权，点击“否”则操作取消。

`highestAvailable`

此程序将以当前用户能获取的最高权限来运行。这个概念可能会跟前面说的 `requireAdministrator` 弄混淆。

如果你指定为 `highestAvailable`：当你在管理员账户下运行此程序，就会要求权限提升。资源管理器上会出现盾牌图标，双击或使用 `Process.Start` 启动此程序会弹出 UAC 提示框。在用户同意后，你的程序将获得完全访问令牌（Full Access Token）。当你在标准账户下运行此程序，此账户的最高权限就是标准账户。受限访问令牌（Limited Access Token）就是当前账户下的最高令牌了，于是 `highestAvailable` 已经达到了要求。资源管理器上不会出现盾牌图标，双击或使用 `Process.Start` 启动此程序也不会出现 UAC 提示框，此程序将以受限权限执行。对进程控制块的操作，有管理员权限的程序就能控制。

![](https://bbs.kanxue.com/upload/attach/202002/856851_CJZ5MG7VY2A4Z89.jpg)

## 5. 用户名一机一码实现

首先获得CPU制造商，调用函数如下：

```cpp
string GetManID()//获取制造商信息
{
    char ID[25];//存储制造商信息
    memset(ID, 0, sizeof(ID));//先清空数组 ID
    ExeCPUID(0);//初始化
    memcpy(ID + 0, &debx, 4);//制造商信息前四个字符复制到数组
    memcpy(ID + 4, &dedx, 4);//中间四个
    memcpy(ID + 8, &decx, 4);//最后四个
    //如果返回 char*, 会出现乱码;故返回 string 值
    return string(ID);
}
```

然后里面的`ExeCPUID(0)`，是另一个函数，`CPUID`指令是intel IA32架构下获得CPU信息的汇编指令，可以得到CPU类型，型号，制造商信息，商标信息，序列号，缓存等一系列CPU相关的东西。前面原理部分有介绍。

```cpp
void ExeCPUID(DWORD veax)//初始化CPU
{
    __asm
    {
        mov eax, veax
        cpuid
        mov deax, eax
        mov debx, ebx
        mov decx, ecx
        mov dedx, edx
    }
}
```

所以用户名的唯一性就是得到的CPU生产商再和进程`pid`进行循环异或：（至于为什么还是选择进程`pid`参与计算，是因为在动态调试的时候，用`GetCurrentProcessId()`函数返回的可能是调试器的进程id，后面调试会讲。）

```cpp
processId = GetCurrentProcessId();   //进程pid

string username(string CPU, DWORD pid)  //异或拼接
{
    string cur_str = to_string(long long(pid));
    int j = 0;
    for (int i=0; i < CPU.length(); i++)
    {
        CPU[i] = CPU[i] ^cur_str[j];
        j++;
        if (j == cur_str.length() - 1)
            j = 0;
    }
    return CPU;
}
```

## 6. 序列号加密算法

序列号的加密思路是，函数接收刚刚生成的唯一序列号和CPU制造商，然后解密关键字符串“20171120051”，这个字符串预先跟`Y`进行了异或，使用的时候需要先解密。接着根据学号这个字符串转换成的数组，根据具体数值取出CPU制造商对应的字符，如数字2，就是取出`GenuineIntel`第二位字符`n`。然后取出字符之后，再根据数值把字符插入到“用户名”的对应位置，得到序列号。算法如下:

```cpp
string Fun1(string userkey, string cpu)  //处理之后的用户名，和CPU型号  处理序列号函数
{
    string name = "kihnhhkiilh";  //20171120051
    char a[50];
    char CPU[50];
    strcpy_s(CPU, cpu.c_str());
    for (int i = 0; i < name.length(); i++)
    {
        char k = name[i] ^ 'Y';
        a[i] = CPU[hex2int(k)];
    }
    for (int i = 0; i < name.length(); i++)
    {
        char k = name[i] ^ 'Y';
        string s(1, a[i]);
        userkey.insert(hex2int(k), s);
    }
    return userkey;
}
```

## 7. 隐藏`GetProcAddress`函数

原理在之前的部分已经介绍了，这里给出实现的代码和一些注释：

```cpp
DWORD MyGetProcAddress(
    HMODULE hModule,    // DLL的句柄，就是模板映射的基地址
    LPCSTR lpProcName   // 需要查找的函数名字或者序号
)
{
    int i = 0;
    char* pRet = NULL;
    PIMAGE_DOS_HEADER pImageDosHeader = NULL;
    PIMAGE_NT_HEADERS pImageNtHeader = NULL;
    PIMAGE_EXPORT_DIRECTORY pImageExportDirectory = NULL;
    pImageDosHeader = (PIMAGE_DOS_HEADER)hModule;
    pImageNtHeader = (PIMAGE_NT_HEADERS)((DWORD)hModule + pImageDosHeader->e_lfanew);
    pImageExportDirectory = (PIMAGE_EXPORT_DIRECTORY)((DWORD)hModule + pImageNtHeader->OptionalHeader.DataDirectory
           [IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
    DWORD dwExportRVA = pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;
    DWORD dwExportSize = pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].Size;
    DWORD* pAddressOfFunction = (DWORD*)(pImageExportDirectory->AddressOfFunctions + (DWORD)hModule);
    DWORD* pAddressOfNames = (DWORD*)(pImageExportDirectory->AddressOfNames + (DWORD)hModule);
    DWORD dwNumberOfNames = (DWORD)(pImageExportDirectory->NumberOfNames);
    DWORD dwBase = (DWORD)(pImageExportDirectory->Base);
    WORD* pAddressOfNameOrdinals = (WORD*)(pImageExportDirectory->AddressOfNameOrdinals + (DWORD)hModule);
    //这个是查一下是按照什么方式（函数名称or函数序号）来查函数地址的
    DWORD dwName = (DWORD)lpProcName;
    if ((dwName & 0xFFFF0000) == 0)
    {
        goto xuhao;
    }
    for (i = 0; i < (int)dwNumberOfNames; i++)
    {
        char* strFunction = (char*)(pAddressOfNames[i] + (DWORD)hModule);
        if (strcmp(strFunction, (char*)lpProcName) == 0)
        {
            pRet = (char*)(pAddressOfFunction[pAddressOfNameOrdinals[i]] + (DWORD)hModule);
            goto _exit11;
        }
    }
    //这个是通过以序号的方式来查函数地址的
xuhao:
    if (dwName < dwBase || dwName > dwBase + pImageExportDirectory->NumberOfFunctions - 1)
    {
        return 0;
    }
    pRet = (char*)(pAddressOfFunction[dwName - dwBase] + (DWORD)hModule);
_exit11:
    //判断得到的地址有没有越界
    if ((DWORD)pRet<dwExportRVA + (DWORD)hModule || (DWORD)pRet > dwExportRVA + (DWORD)hModule + dwExportSize)
    {
        return (DWORD)pRet;
    }
    char pTempDll[100] = { 0 };
    char pTempFuction[100] = { 0 };
    strcpy(pTempDll, pRet);
    char* p = strchr(pTempDll, '.');
    if (!p)
    {
        return (DWORD)pRet;
    }
    *p = 0;
    strcpy(pTempFuction, p + 1);
    strcat(pTempDll, ".dll");
    LPWSTR k[100] = { 0 };
    TCHAR aaa[31];
    MultiByteToWideChar(0, 0, pTempDll, 31, aaa, 62);
    HMODULE h = LoadLibrary(aaa);
    if (h == NULL)
    {
        return (DWORD)pRet;
    }
    return MyGetProcAddress(h, pTempFuction);
}
```

根据句柄得到映射地址，然后判断是字符串函数名字还是序号导出（这一块代码需要借助，没写过）。再在导出表中得到函数的映射地址。

## 8. `VMProtectSDK` 保护

首先下载最新的`vmp`，可以从官网进行下载，下载之后根目录下有一个`example`文件夹：

![](https://bbs.kanxue.com/upload/attach/202002/856851_A5FCYSNTCK8MQH5.jpg)

然后点击里面的`VC++`，里面有一个头文件和一个资源文件：

![](https://bbs.kanxue.com/upload/attach/202002/856851_AQBGKQ8VBB8NBFV.jpg)

把这两个文件在项目里面包含它。然后就可以调用了。官方的头文件里面有很多内容：

![](https://bbs.kanxue.com/upload/attach/202002/856851_8SBW4R396XDSQAE.jpg)

要注意虚拟机的调用的顺序，特别是涉及到堆栈操作的话。

大作业中对整个反调试函数进行虚拟操作：

![](https://bbs.kanxue.com/upload/attach/202002/856851_ZFY3CW8CCNQXG79.jpg)

也可以选择其他的方式，比如代码变异等。默认就是代码虚拟。

## 9. 反调试实现（代码和注释，原理在第一部分）

在写大作业之前，我就把论坛里面常见的反调试技术都实现了一遍。把整个反调试弄成一个头文件里面，并用一个主函数包含起来。这样就可以像花指令那样想在哪放就在哪放。实现的原理见前面第一章，这里只给出实现代码，原理见第一部分。

用`win API`函数实现的反调试：

```cpp
BOOL Fan1()
{
    return IsDebuggerPresent();
}

BOOL Fan2()
{
    BOOL ret;
    CheckRemoteDebuggerPresent(GetCurrentProcess(), &ret);
    return ret;
}
```

检查进程控制块PEB结构函数：

```cpp
bool PebIsDebugged()  //检查PEB  BeingDebugged标志位
{
    char result = 0;
    __asm
    {
        // 进程的PEB地址放在fs这个寄存器位置上
        mov eax, fs: [30h]
        // 查询BeingDebugged标志位
        mov al, BYTE PTR[eax + 2]
        mov result, al
    }
    return result != 0;
}
```

检查NtGlobal标志位：

```cpp
bool PebNtGlobalFlags()  //检查NtGlobal标志位  NtGlobal标志位
{
    int result = 0;
    __asm
    {
        // 进程的PEB
        mov eax, fs: [30h]
        // 控制堆操作函数的工作方式的标志位
        mov eax, [eax + 68h]
    }
}
```
