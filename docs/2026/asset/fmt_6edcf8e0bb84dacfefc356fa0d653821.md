
本例程是实验04-1的改版，`A::test1`，`A::test2`的两个虚函数在类成员中被指定为`__stdcall`调用约定。这就是说，我们可以实现一个全局函数而不是一个C++成员函数，对`class A`中的某个虚函数进行替换。具体交换过程如图4.4所示。COM标准中规定，所有的接口函数的调用约定必须是`__stdcall`，本例程更接近于真实的COM接口。虚函数的替换过程，及实验输出结果与实验04-1相同。

本节实验代码如下：

```cpp
0001 class A
0002 {
0003 public:
0004         A() {n = 99; m = 25;}
0005         virtual void __stdcall test1(int a){printf("A::test1, a = %d, n = %d, m = %d\n", a, n++, m++);}
0006         virtual void __stdcall test2(int a){printf("A::test2, a = %d, n = %d, m = %d\n", a, n++, m++);}
0007
0008         int n;
0009         int m;
0010 };
0011
0012 typedef void (__stdcall* PFN_MEMBER)(void* ThisPtr, int a);
0013
0014 void __stdcall A_test2(void* ThisPtr, int a) // 替换A::test2
0015 {
0016         printf("A::test2, hook successed, a = %d, n = %d, m = %d\n",
0017                 a, ((A*)ThisPtr)->n++, ((A*)ThisPtr)->m++);
0018 }
0019
0020 int main(int argc, char* argv[])
0021 {
0022         A* p1 = new A();
0023
0024         p1->test1(1);
0025         p1->test2(2);
0026
0027         LPDWORD VtablePtr = (LPDWORD)(*((LPDWORD)(LPVOID)p1)); // 取虚表指针
0028
0029         DWORD dwOld = 0;
0030         if(!::VirtualProtect(VtablePtr, sizeof(LONG) * 2, PAGE_READWRITE, &dwOld))
0031                 goto _FREE;
0032
0033         // 修改class A的第二个虚函数，即A::test2
0034         VtablePtr[1] = (DWORD)(LPVOID)A_test2;
0035
0036         if(!::VirtualProtect(VtablePtr, sizeof(LONG) * 2, dwOld, &dwOld))
0037                 goto _FREE;
0038
0039         p1->test1(3);
0040         p1->test2(4);
0041
0042 _FREE:
0043         if (p1)
0044         {
0045                 delete p1;
0046                 p1 = NULL;
0047         }
0048
0049         return 0;
0050 }
```

```text
A::test1, a = 1, n = 99, m = 25
A::test2, a = 2, n = 100, m = 26
A::test1, a = 3, n = 101, m = 27
A::test2, hook successed, a = 4, n = 102, m = 28
```

![](https://bbs.kanxue.com/upload/attach/201412/169576_ce9sbz8sr4w0tef.png)

图4.4 虚函数表中函数地址被修改

Fig.4.4 Modified virtual function address

## 4.6 实验05：进程内Hook API（`MessageBoxW`）

什么是Hook API? 这里所说的API单指Win32 API，由于在Windows环境下，应用程序必须要使用Windows 提供的API才能编写一个合格的Windows程序。在程序中更是频频调用系统API来完成各种功能，如窗口管理，进程线程管理，文件读写，内存管理等等。一部分应用，由于需求较为特殊，例如文件透明加密，需要在其他进程在文件读写时，为用户将文件解密和加密，必须先于目标进程获取文件读写的内容。也就是说，在目标进程调用文件读写相关API时，先要调用文件透明加密系统提供的“替代”函数。自己进行挂钩系统API，需要开发者了解DLL基址重定位，远线程注入，跳转指令机器码，跳转指令的偏移计算等相关知识。虽说Hook API并不是什么新兴技术，但是要自己实现起来，还是要花一定的功夫。

该例程中使用了一个开源并免费的Hook API库-mhook来实现这一过程。该例程仅仅是为了演示如何使用mhook，所以直接挂钩的是进程内的`MessageBoxW`函数。而真实的挂钩Win32 API通常要实现在一个DLL中，并且在适时，将DLL注入到目标进程中。对于mhook库的使用，可以说是傻瓜化的。一般的步骤如下：

1. 声明一个与将要挂钩函数相同的函数指针。本例程中要挂钩的函数是`MessageBox`，实际上系统并没有提供`MessageBox`函数，`MessageBox`实际是一个宏，跟据用户所创建的工程使用的字符集，如果是UNICODE，则`MessageBox`自动切换为`MessageBoxW`函数，如果工程使用的字符集是ANSI，则`MessageBox`，自动切换为`MessageBoxA`函数。因为本例程使用的是UNICODE字符集，所以所以声明函数指针时，参照了`MessageBoxW`。基本上，只要涉及到字符串的函数，Windows都提供了两个版本。

2. 使用声明好的函数指针保存原始的API函数地址。通常我们挂钩的API都是在DLL中实现的，DLL在被进程加载后，有一个基址重定位的过程。使用mhook库时，不用考虑这个。只需要使用`GetModuleHandle`函数和`GetProcAddress`获取地址即可。这两个函数的原型如下：

```text
HMODULE GetModuleHandle(
  LPCTSTR lpModuleName // 模块名称
);

FARPROC WINAPI GetProcAddress(
  __in HMODULE hModule,  // 模块句柄
  __in LPCSTR lpProcName // 函数名称（只能是ANSI字符串）
);
```

要挂钩的API所在的模块名称，可以通过查阅MSDN来获得。而函数的真实名称，有一个简单有效的方法得到。即在Visual C++编辑器中输入函数名称，如`MessageBox`，然后在此函数名上点击鼠标右键，在弹出的菜单是，点选`Go To Definition`，即可跳转到真实的函数声明处，如图4.5所示

![](https://bbs.kanxue.com/upload/attach/201412/169576_di5po84j74gsshf.png)

图4.5 查看函数声明

Fig.4.5 View the function declaration

3. 按照将要挂钩的函数原型，准备一个“替代”函数。

4. 使用`Mhook_SetHook`函数开始挂钩，或使用`Mhook_Unhook`函数停止挂钩。

5. 将写好的挂钩模块，注入到目标进程。本文后续实验中仍然使用`WH_GETMESSAGE`钩子来完成注入过程。

```cpp
0001 #include "mhook/mhook-lib/mhook.h"
0002 #if defined _DEBUG || defined DEBUG
0003         #pragma comment(lib, "Debug/mhook.lib")
0004 #else
0005         #pragma comment(lib, "Release/mhook.lib")
0006 #endif
0007
0008 // 声明MessageBoxW的函数指针
0009 typedef int (WINAPI* PFN_MessageBoxW)(HWND hWnd, LPCWSTR lpText, LPCWSTR lpCaption, UINT uType);
0010
0011 // 保存MessageBoxW原来的地址
0012 PFN_MessageBoxW MessageBoxW_OLD = (PFN_MessageBoxW)GetProcAddress(GetModuleHandle(_T("user32.dll")), "MessageBoxW");
0013
0014 // 用于替换原始的MessageBoxW函数
0015 int __stdcall MessageBoxW_NEW(HWND hWnd, LPCWSTR lpText, LPCWSTR lpCaption, UINT uType)
0016 {
0017         // 简单的修改一下标题栏文字，将控制权返回给原始的MessageBoxW
0018         return MessageBoxW_OLD(hWnd, lpText, L"Hook - We are modified the caption", uType);
0019 }
0020
0021 // 开始拦截MessageBoxW的调用
0022 VOID MessageBoxW_Hook()
0023 {
0024         if (Mhook_SetHook((PVOID*)&MessageBoxW_OLD, MessageBoxW_NEW))
0025         {
0026                 // successfully
0027         }
0028 }
0029
0030 // 关闭拦截MessageBoxW的调用
0031 VOID MessageBoxW_UnHook()
0032 {
0033         Mhook_Unhook((PVOID*)&MessageBoxW_OLD);
0034 }
0035
0036 // 主函数
0037 int APIENTRY _tWinMain(HINSTANCE hInstance,
0038                      HINSTANCE hPrevInstance,
0039                      LPTSTR    lpCmdLine,
0040                      int       nCmdShow)
0041 {
0042         MessageBoxW(NULL, L"Hello World", L"Title", MB_OK);
0043
0044         MessageBoxW_Hook();
0045         MessageBoxW(NULL, L"Hello World", L"Title", MB_OK);
0046         MessageBoxW_UnHook();
0047
0048         MessageBoxW(NULL, L"Hello World", L"Title", MB_OK);
0049
0050         return 0;
0051 }
```

程序运行结果如图4.6，图示4.7，图4.8所示：

![](https://bbs.kanxue.com/upload/attach/201412/169576_5kai1lkrx7x8u5d.png)

图4.6 未挂钩前输出

Fig.4.6 before unhook

![](https://bbs.kanxue.com/upload/attach/201412/169576_mo74vemfpc6kqbd.png)

图4.7 挂钩后输出

Fig.4.7 after hook

![](https://bbs.kanxue.com/upload/attach/201412/169576_v6kuc1pw68n9wmp.png)

图4.8 取消挂钩后输出

Fig.4.8 Cancel hook

## 本章小结

通过前几个实验，我们已经撑握了如何交换两个C++对象的的虚函数表；我们也撑握了通过虚函数表指针查找表中的虚函数，及进行虚函数地址改写，

这些知识，为我们挂钩真正的COM接口奠定了基础。但是在Hook实际的COM接口时，还有一些问题有待解决。

# 第5章 针对COM接口函数的挂钩研究与实现

## 5.1 实验06：准备COM组件样本，并实现一个`IMath`接口（程序名称：`Sample.dll`）

在此例程中，我们创建了一个真正的COM组件，即`Sample`组件。在此组件中添加了一个接口`IMath`，并由`CMath`对象实现它。然后在`IMath`接口中添加了两个接口函数，一个是`Add`用于计算两数之和，一个是`Sub`用于计算两数之差。`IMath`接口除了继承了必须的接口（如`IUnknow`），还继承自`IDispatchImpl`，这是一个自动化支持接口，在后续的实验是，我们将还可以使用网页来调用`Sample`组件。

该组件核心代码如下：

```cpp
// Sample.dll
0001 STDMETHODIMP CMath::Add(int a, int b, int *c)
0002 {
0003         *c = a + b;
0004         return S_OK;
0005 }
0006
0007 STDMETHODIMP CMath::Sub(int a, int b, int *c)
0008 {
0009         *c = a - b;
0010         return S_OK;
0011 }
0012
0013 MIDL_DEFINE_GUID(IID, IID_IMath,0x09783CD5,0xCCAF,0x4D1D,0xA4,0x53,0x01,0x6E,0x68,0xE8,0x52,0x09);
0014 MIDL_DEFINE_GUID(CLSID, CLSID_Math,0x6BD26856,0x7090,0x4F8F,0xA8,0xF8,0xE6,0x05,0x9C,0xEB,0x73,0xAC);
```

## 5.2 实验07-1：测试`Sample.dll`组件（程序名称：`SampleTest.exe`）

在调用实验06中的组件前，首先要将编译好的`Sample.dll`注册到系统当中。注册COM组件可以使用系统提供的实用工具`Regsvr32`。具体的注册方法是，在命令行中输入：`regsvr32 -i "组件的完整路径及名称"`，反注册一个COM组件只需将`-i`改为`-u`即可。在客户程序（调用组件的程序）调用一个COM组件前，最重要的是知道将要使用的接口的GUID，以及实现接口函数的COM对象的CLSID。这两个信息，在实验06中已经给出。

组件注册好以后，启动本例程`SampleTest.exe`，上面一行文本框，分别是，加数1，加数2，和数。下面一行文本框分别是减数1，减数2，差值。通过运行观察，`Sample`组个提供的加减法功能正常，测试通过。

后续实验中，我们将通过挂钩`IMath`接口中的函数，进而从`SampleTest.exe`中获取用户输入的原始值，并将其“偷偷修改”。

本节测试代码如下：

```cpp
// SampleTest.exe
0001 class CSampleTestDlg : public CDialog
0002 {
0003         ...
0004 private:
0005
0006         IMath* m_lpIMath;
0007 };
0008
0009 BOOL CSampleTestDlg::OnInitDialog()
0010 {
0011         ...
0012
0013         AfxMessageBox("Start ?");
0014
0015         ::CoInitialize(NULL);
0016
0017         HRESULT hr;
0018
0019         // 方法一
0020
0021         /*
0022         CLSID  clsid;
0023         CLSIDFromProgID(OLESTR("Sample.Math"),&clsid);
0024         hr = CoCreateInstance(clsid, NULL, CLSCTX_INPROC_SERVER, __uuidof(IMath), (LPVOID*)&m_lpIMath);
0025         */
0026
0027         // 方法二
0028         hr = CoCreateInstance(CLSID_Math, NULL, CLSCTX_INPROC_SERVER, IID_IMath, (LPVOID*)&m_lpIMath);
0029
0030         if (FAILED(hr))
0031         {
0032                 CString str;
0033                 str.Format("Error: 0x%08X", hr);
0034                 AfxMessageBox(str);
```
