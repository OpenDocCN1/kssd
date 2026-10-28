
# C++创建各种快照，实现【进程名取ID】【进程ID取全部线程ID】【取模块基址】等基础功能

## 铺垫

自从我们从易语言换到C++，一些易语言中的简单操作却很难实现，比如精易模块用的如此熟络的“`进程_名取 ID（）`”，又或者是为了一个功能专门添加一个模块的“`追风__取模块句柄（）`”，这些都可以在今天的探究里得以解决~

## 准备

1. Vs2019（别的也行）
2. 百度
3. 一颗优秀的大脑
4. 一个对象（估计你没有，我也没有）

准备完毕的话，咱们开工

## 开始

他非要和我试试，我说可以。诶…我一说完他啪就站起来了，很快啊!找到了这几个函数

```
HANDLE CreateToolhelp32Snapshot(
[in] DWORD dwFlags,
[in] DWORD th32ProcessID
);
BOOL Thread32First(
[in] HANDLE hSnapshot,
[in, out] LPTHREADENTRY32 lpte
);
BOOL Thread32Next(
[in] HANDLE hSnapshot,
[out] LPTHREADENTRY32 lpte
);
```

必须要说一下，在创建快照的时候，进程快照和线程快照都不需要`th32ProcessID`的参数，我们直接填写0就可以，但是指定进程的模块快照需要填写该进程的ID

```
#include <Windows.h>
#include <iostream>
#include <tlhelp32.h>
#include <vector>
using namespace std;
std::vector<DWORD> GetAllProcessThread(DWORD ProcessId)//一定要保证 是多字节模式，不然可能会出错
{
std::vector<DWORD>v;
HANDLE ThreadList = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);
THREADENTRY32 ThreadInfo{ sizeof(THREADENTRY32) };
if (Thread32First(ThreadList, &ThreadInfo))
{
do
{
if (ThreadInfo.th32OwnerProcessID == ProcessId)
{
v.push_back(ThreadInfo.th32ThreadID);
}
} while (Thread32Next(ThreadList, &ThreadInfo));
}
CloseHandle(ThreadList);
return v;
}
DWORD GetProcessId(char* ProcessName)//一定要保证 是多字节模式，不然可能会出错
{
DWORD ProcessId = 0;
HANDLE hProcessList = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
PROCESSENTRY32 processInfo{ sizeof(PROCESSENTRY32) };
if (Process32First(hProcessList, &processInfo))
{
do {
if (strcmp(processInfo.szExeFile, ProcessName) == 0)
{
ProcessId = processInfo.th32ProcessID;
break;
}
} while (Process32Next(hProcessList, &processInfo));
}
CloseHandle(hProcessList);
return ProcessId;
}
ULONG64 GetModuleBaseAdress(DWORD ProcessId,char* Module)//一定要保证 是多字节模式，不然可能会出错
{
ULONG64 ModuleBaseAdr = 0;
char p[20] = { 0 };
HANDLE ModuleList = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, ProcessId);
MODULEENTRY32 ModuleInfo{ sizeof(MODULEENTRY32) };
if (Module32First(ModuleList, &ModuleInfo))
{
do
{
if (strcmp(ModuleInfo.szModule, Module)==0)
{
sprintf(p, "%d", ModuleInfo.modBaseAddr);
ModuleBaseAdr = atoi(p);
//感谢指出 转换可以用
//ModuleBaseAdr =(DWORD)ModuleInfo.modBaseAddr;强转
break;
}
} while (Module32Next(ModuleList, &ModuleInfo));
}
CloseHandle(ModuleList);
return ModuleBaseAdr;
}
```

### 拿取模块地址讲讲

其他的都是以此类推的繁琐操作，作为浑元形意太极门掌门人的你们肯定不会有问题啊

~~1.第一步创建快照，获取快照句柄~~

#### 1. 第一步，声明变量

```
ULONG64 ModuleBaseAdr = 0;//模块地址 长整数来存
char p[20] = { 0 };//一会字节转字节集要用
```

#### 2. 第二步创建快照，获取快照句柄

```
HANDLE ModuleList = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, ProcessId);
```

`TH32CS_SNAPMODULE`是指所有的模块，`id`指定进程

#### 3. 第三步，构造模块信息结构体

```
MODULEENTRY32 ModuleInfo{ sizeof(MODULEENTRY32) };
```

也可以先构造，随后设置大小

#### 4. 第四步，判断能否获取，循环取出

```
if (Module32First(ModuleList, &ModuleInfo))
```

`Module32First`会返回一个布尔变量，如果该变量为`false`，说明我们因为一些原因无法获取，直接结束吧，如果为`true`，代码如下:

```
do
{
if (strcmp(ModuleInfo.szModule, Module)==0)
{
sprintf(p, "%d", ModuleInfo.modBaseAddr);
ModuleBaseAdr = atoi(p);
break;
//感谢指出 转换可以用
//ModuleBaseAdr =(DWORD)ModuleInfo.modBaseAddr;强转
}
} while (Module32Next(ModuleList, &ModuleInfo));
```

我们在`Module32First`中将进程快照的第一号进程信息写入了`ModuleInfo`中，所以我们先取信息，然后循环，`strcmp(ModuleInfo.szModule, Module)`可以帮我们比较是否是我们需要的模块名称。

如果是，那就取出当前模块的地址，受限于`ModuleInfo.modBaseAddr`的类型是字节，那我们用一步转换就好了，刚才的`char p`也排上了用处，最后`atoi`转换长整数，跳出循环，逻辑闭环。

如果不是，那就通过`Module32Next(ModuleList, &ModuleInfo)`取顺次的下一个模块信息，继续循环

## 最后

1. 选择多字节方案
2. 百度可以解决很多问题
3. 易语言真的是世界上最好的语言呢
4. 看雪新人·多多指教

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
