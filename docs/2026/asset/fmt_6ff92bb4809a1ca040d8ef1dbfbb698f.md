
# [原创]借腹怀胎的注入代码个人理解

线程注入代码算是很老的技术，但是涉及的东西还是挺多的，所以不失其重要性。以前走了很多弯路，一句话，工具黑客坑害求学之人。而后很冲动的转了专业，投奔了“妓院”。正儿八经的学编程已经有一年多半了，来看雪就更有年头了（还真忘了怎么弄到的注册号）。坛子多是是ring0的东西，也在偶尔写写驱动代码。貌似走题了，赶紧转正。头一次发原创帖，貌似没啥新东西，就是总结一下，权当给和我一样的小菜看。不喜勿批，嘿嘿！

代码注入的重要思想是自定义函数的代码在远程进程空间执行。要做到这点，首先要在远程空间注入自己的代码和参数。所以，我们首先要明确我们分开注入代码和参数，当然参数也可以在代码里，下边会给予提示。我们已最简单的在远程线程执行`MessageBox`为例

## 1, 参数的注入

我们很容易的定义一个结构体

```c
#pragma pack(1)

typedef struct _RemoteParam
{
  GetModuleHandle_Fun reGetModuleHandle;
  LoadLibrary_Fun reLoadLibrary;
  GetProcAddress_Fun reGetProcAddress;
  char szUser32Dll[11];
  char szMessageBox[12];
  char szText[20];
  char szCaption[20];
}RemoteParam,*PRemoteParam;
#pragma pack()
```

对于`GetModuleHandle`，`GetProcAddress`和`LoadLibrary`这三个函数，可以在目标进程的`kernel32.dll`的导出表内搜索，本例从简，直接写入（由于`kernel32.dll`在不同的进程中`GetModuleHandle`得到的`hMod`都是一样的，所以，它的`GetModuleHandle`，`GetProcAddress`和`LoadLibrary`的实现部分在不同进程的地址也是一样的)。为了告诉编译器这三个地址是三个如它们定义一样的函数，所以我们姑且定义一下，

```c
typedef FARPROC (__stdcall * GetProcAddress_Fun)(
                                         HMODULE hModule,
                                         LPCSTR lpProcName
                                         );
typedef HINSTANCE (__stdcall *LoadLibrary_Fun)(
                                        LPCTSTR lpLibFileName
                                        );
typedef HMODULE (__stdcall *GetModuleHandle_Fun)(
                                              LPCTSTR lpModuleName
                                              );
```

如果你不想自己定义，那么完全可以，在你自定义函数体中的嵌入汇编，用压栈`call`调用的方式也可以。在此不再赘述。函数的定义这里有一点特别要注意的，就是参数的调用约定为题 `__stdcall`不能丢弃，否则破坏了堆栈，生成的`debug`程序运行会出错，而`release`虽然不报错但关掉注入程序，母体程序也会关闭。如果你`od`玩转的很好的，我想你很容易处理一下，在出现出错的程序弹出框后完全可以不关闭它，用`od`附加，定位到错误代码，自己看看植入代码出错问题。有一点要说的是`debug`版本有检测堆栈，汇编代码是`cmp esi,esp`,这个经常是拦路虎，在`inline hook`中经常因为它`debug`版本程序不能正常运行，嘿嘿。

```asm
00401000   .  56            push esi
00401001   .  8B7424 08     mov esi,dword ptr ss:[esp+0x8]  //通过esp寻参
00401005   .  8D46 0C       lea eax,dword ptr ds:[esi+0xC]  //esi指向的就是我们注入的
00401008   .  50            push eax                    //参数的开始地址
00401009   .  FF16          call dword ptr ds:[esi]     //GetModuleHandle
0040100B   .  8D4E 17       lea ecx,dword ptr ds:[esi+0x17]
0040100E   .  51            push ecx
0040100F   .  50            push eax
00401010   .  FF56 08       call dword ptr ds:[esi+0x8]  //GetProcAddress
00401013   .  6A 00         push 0x0
00401015   .  6A 00         push 0x0
00401017   .  6A 00         push 0x0
00401019   .  6A 00         push 0x0
0040101B   .  FFD0          call eax
0040101D   .  83C4 1C       add esp,0x1C
00401020   .  5E            pop esi
00401021   .  C3            retn
```

如果你不加`__stdcall`往往是上边贴出那样（`release`版本）的，`add esp,0x1c`明显是错误的，我们调用的是`api`，除了`wsprintf`等少数函数外，通常`api`都是在函数内部实现堆栈平衡的。

上边已经讲解了参数所能遇到的问题。开头我提过，我们完全不将函数参数和代码分开写入，对于这点，在汇编内很容易实现，我们可以在函数内分配空间，分配空间的伪指令就是 `db`，`dd` 我们可以 `dd`出`GetProcAddress`等三个函数，可以`db`出`szString`等变量名。在`ide`中，以`vc`为例，我们可以通过 `__emit`伪指令实现`db`功能，加个标签就是函数变量的地址，例如`bingle`： `__emit 0`

`__emit 0`

`__emit 0`

`__emit 0`这样就在函数内部定义了一个存储`GetProcAddress`等函数地址的变量，

当然，你想这样的话，你得对`vc`中汇编操作比较熟练。

## 2，代码部分的注入

此部分比较关键。

`CreateRemoteThread`函数允许我们把我们的参数部分传入，当然我看到过另一种定位参数部分的方法。待会给出。

首先也是在远程空间分配空间，`VirtualAllocEx`出代码部分大小的空间。这里涉及到函数长度的问题，貌似我在坛子里问过，大牛说反汇编引擎最精确，可以帮我实现这点，可本人还没研究到那块，所以，我们也不估量了直接分配一页`4*1024`大小的代码空间，足够用。当然节省空间粗略计算的方法我也给出吧。代码的自定位和`staic`内存紧密排布。

```c
Static void InjectCode(RemoteParam *param)
{
}
Static dword __declspec(naked) GetBeginAddr()
{
    __asm
    {
        call lbl_Next
lbl_Next:
        pop eax
        sub eax, 5
        ret
    }
}
```

函数的长度就是`GetBeginAddr`的`return`值减去`InjectCode`。裸函数内部不能定义变量，否则可以直接把`InjectCode`定义为裸函数，那么计算函数的长度就容易多了，哈哈！

另一种定位参数的方法是将代码部分和参数部分放在一页里边，这完全可以通过`Virtualalloc`这个函数实现。然后通过自定位的方法在定位参数。给出代码。

```c
static LRESULT CALLBACK NewProc(
  HWND hwnd,
  UINT uMsg,
  WPARAM wParam,
  LPARAM lParam )
{
        INJDATA* pData;
        _asm {
                call        dummy
dummy:
                pop                ecx
                sub                ecx, OFFSET
                mov                pData, ecx
        }
        pData--;
}
```

内存排布：`Data`数据，`NewProc`函数。`pData--`直接定位到参数部分，是不是很神奇呀，哈哈。

接下来是注入的函数体部分的注意问题了。我们不可以在自己实现的要注入的函数内部定义全局变量和保留字符串等操作。原因就是全局变量和字符串是在你程序的进程空间而不是在母体（宿主体内）中的。

测试的注入代码（注入的是`notepad`和酷我等，所以没有`loadlibrary`入`user32.dll`）：

```c
void bingle(RemoteParam * remoteParam)
{
        HMODULE hMod = remoteParam->reGetModuleHandle(remoteParam->szUser32Dll);
    MessageBox_Fun myMessageBox =     (MessageBox_Fun)remoteParam->reGetProcAddress(hMod,remoteParam->szMessageBox);
    myMessageBox(NULL,remoteParam->szText,remoteParam->szCaption,NULL);
        }
```

对应的汇编代码：

```asm
004010A0   .  56            push esi
004010A1   .  8B7424 08     mov esi,dword ptr ss:[esp+0x8]
//[esp] = 原esi,[esp+4] = eip,[esp+8] = remoteParam
004010A5   .  8D46 0C       lea eax,dword ptr ds:[esi+0xC]
//eax = esi+0xc = RemoteParam的第四个变量
004010A8   .  50            push eax
004010A9   .  FF16          call dword ptr ds:[esi]
//调用GetModuleHandle
004010AB   .  8D4E 17       lea ecx,dword ptr ds:[esi+0x17]
004010AE   .  51            push ecx
004010AF   .  50            push eax
004010B0   .  FF56 08       call dword ptr ds:[esi+0x8]
//调用GetProcAddress
004010B3   .  8D56 37       lea edx,dword ptr ds:[esi+0x37]
004010B6   .  6A 00         push 0x0
004010B8   .  83C6 23       add esi,0x23
004010BB   .  52            push edx
004010BC   .  56            push esi
004010BD   .  6A 00         push 0x0
004010BF   .  FFD0          call eax
//调用MessageBox
004010C1   .  5E            pop esi
004010C2   .  C3            retn
```

这里要说的是想要玩转注入代码，这部分汇编代码的分析跟外重要。纠错就是从此处下手。上边给出简略分析，很容易知道，完全是对应`RemoteParam`结构体的。

还有要注意的问题是`api`函数的宽字符和窄字符的问题，比如`GetProcAddress(hMod,"GetProcAddress");`可见`GetProcAddress`没有对应的宽窄函数（貌似这么称呼很不专业呀，呵呵，理解万岁），我的处理方法是。在`od`的`cmdline`里边测试一下，比如你`dd LoadLibrary`会显示“未知标示符”，可见它是`LoadLibraryA` 和 `LoadLibraryW`两种的。这是我处理的方法，其它函数一样。坛友有好的方法，可以告诉我一下，谢谢啦。

## 3，参数也注入了，代码也注入了

剩下的就是`CreateRemoteThread`，执行一下子啦。呵呵，这个不消说，观者应该没问题的。还有就是`QueueUserApc`也可以执行注入代码，只要你考虑了远程空间是够有你要用的参数问题，还有堆栈问题，一切都简单了，哈哈！

测试结果，`od`打开枚举进程那块的酷狗活动窗口是`test(MessageBox`的标题),`MessageBox`的任务栏图标`icon`也换成它爹`kugou`的图标了，呵呵！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
