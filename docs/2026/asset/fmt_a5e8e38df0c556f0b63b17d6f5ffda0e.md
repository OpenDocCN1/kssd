
``` 
0002 typedef HRESULT (__stdcall* PFN_SUB)(IUnknown* This, int a, int b, int* c);
0003
0004 #if defined WIN64
0005         typedef DWORD64* MEMADDR_PTR;
0006         typedef DWORD64 MEMADDR_VAL;
0007 #else
0008         typedef LPDWORD MEMADDR_PTR;
0009         typedef DWORD MEMADDR_VAL;
0010 #endif
0011
0012 class IMathVtable
0013 {
0014 public:
0015
0016         static HRESULT HookMethod();
0017         static HRESULT UnHookMethod();
0018
0019         static HRESULT __stdcall Add(IUnknown* This, int a, int b, int* c);
0020         static HRESULT __stdcall Sub(IUnknown* This, int a, int b, int* c);
0021
0022         static MEMADDR_PTR m_Add_OLD;
0023         static MEMADDR_PTR m_Sub_OLD;
0024
0025         static MEMADDR_PTR m_lpVtable;
0026         static HMODULE m_hModule;
0027 };
0028
0029 MEMADDR_PTR IMathVtable::m_Add_OLD   = NULL;
0030 MEMADDR_PTR IMathVtable::m_Sub_OLD   = NULL;
0031 MEMADDR_PTR IMathVtable::m_lpVtable  = NULL;
0032 HMODULE IMathVtable::m_hModule       = NULL;
0033
0034 HRESULT IMathVtable::HookMethod()
0035 {
0036         m_hModule = ::LoadLibrary("Sample.dll");
0037
0038         // 改写虚表前九项(DWORD_PTR)内存的属性
0039         DWORD dwVtableMaxLen = 9;
0040
0041         // 确保COM环境补初始化
0042         ::CoInitialize(NULL);
0043
0044         // 自己创建一个接口
0045
0046         IMath* lpIMath = NULL;
0047         HRESULT hr = CoCreateInstance(CLSID_Math, NULL,
0048                 CLSCTX_INPROC_SERVER, IID_IMath, (LPVOID*)&lpIMath);
0049
0050         if (FAILED(hr))
0051                 return hr;
0052
0053         MEMADDR_VAL VtableAddr = *((MEMADDR_PTR)lpIMath);
0054         m_lpVtable = (MEMADDR_PTR)VtableAddr;
0055
0056         // 将内存属性改为可读写
0057         DWORD dwOld = 0;
0058     if(!::VirtualProtect(m_lpVtable, sizeof(MEMADDR_VAL) * dwVtableMaxLen,
0059                 PAGE_EXECUTE_READWRITE, &dwOld))
0060         return E_FAIL;
0061
0062         // Hook Add method, Add函数位于虚表的第七项
0063         m_Add_OLD = (MEMADDR_PTR)m_lpVtable[7];
0064         m_lpVtable[7] = (MEMADDR_VAL)(MEMADDR_PTR)IMathVtable::Add;
0065
0066         // Hook Sub method, Sub函数位于虚表的第八项
0067         m_Sub_OLD = (MEMADDR_PTR)m_lpVtable[8];
0068         m_lpVtable[8] = (MEMADDR_VAL)(MEMADDR_PTR)IMathVtable::Sub;
0069
0070         // 内存属性再改回来
0071     if(!::VirtualProtect(m_lpVtable, sizeof(MEMADDR_VAL) * dwVtableMaxLen,
0072                 dwOld, &dwOld))
0073         return E_FAIL;
0074
0075         lpIMath->Release();
0076         return S_OK;
0077 }
0078
0079 HRESULT IMathVtable::UnHookMethod()
0080 {
0081         ...
0082         return S_OK;
0083 }
0084
0085 HRESULT __stdcall IMathVtable::Add(IUnknown* This, int a, int b, int* c)
0086 {
0087         ...
0088         // 调用原始的函数
0089         PFN_ADD pFN_Add = (PFN_ADD)m_Add_OLD;
0090         return pFN_Add(This, a, b, c);
0091 }
0092
0093 HRESULT __stdcall IMathVtable::Sub(IUnknown* This, int a, int b, int* c)
0094 {
0095         ...
0096         // 调用原始的函数
0097         PFN_SUB pFN_Sub = (PFN_SUB)m_Sub_OLD;
0098         return pFN_Sub(This, a, b, c);
0099 }
```

### 5.6 实验10：对实验程序`SampleProxy.dll`，`SampleVtable.dll`的综合测试（程序名称：`SampleHook.exe`）

#### 测试一：

步骤1：先启动`SampleHook.exe`，然后点击`Button1`。此时，程序内部通过对象代理法对`IMath::Add`，`IMath::Sub`进行挂钩。

步骤2：启动`SampleTest.exe`，输入数字进行加减法。这时会弹出一个输入值被修改的提示，点击确定后查看计算结果。即是被修改过的值的计算结果。

步骤3：关闭`SampleHook.exe`程序，再次使用`SampleTest.exe`和`SampleTest.html`进行数值计算，此时计算结果正是用户输入的原始值的计算结果。

测试结果，如图5.1所示：

![](https://bbs.kanxue.com/upload/attach/201412/169576_g6n6d77ynweqpk4.png)

图5.1 综合测试一

Fig.5.1 Test 1

#### 测试二：

步骤1：启动`SampleTest.exe`，或者启动IE浏览器，并打开`SampleTest.html`，输入数字进行加减法，计算结果正确。

步骤2：启动`SampleHook.exe`，点击`Button3`，此时，程序内部通过虚表补丁法对`IMath::Add`，`IMath::Sub`进行挂钩。

步骤3：再次使用`SampleTest.exe`和`SampleTest.html`进行数值计算，这时会弹出一个输入值被修改的提示，点击确定后查看计算结果。即是被修改过的值的计算结果。

步骤4：关闭`SampleHook.exe`程序，再次使用`SampleTest.exe`和`SampleTest.html`进行数值计算，此时计算结果正是用户输入的原始值的计算结果。

测试结果，如图5.2所示：

![](https://bbs.kanxue.com/upload/attach/201412/169576_66oy6vq9iimqk0l.png)

图5.2 综合测试二

Fig.5.2 Test 2

该实验，分别对`SampleProxy.dll`程序中的对象代理法及`SampleVtable.dll`程序中的虚表补丁法进行了测试。测试结果表明，当`SampleTest.exe`和`SampleTest.html`（IE）调用`IMath::Add`，`IMath::Sub`这两个函数时，均已被挂钩，因为挂钩程序修改了参数中的值，使得计算结果也被改变了。这也证明了，在前面所得出的实验结论都是正确的。

测试关键代码如下：

```
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
0035         }
0036
0037         return TRUE;  // return TRUE  unless you set the focus to a control
0038 }
0039
0040 void CSampleTestDlg::OnButton1()
0041 {
0042         if (NULL == m_lpIMath)
0043         {
0044                 AfxMessageBox("NULL");
0045                 return;
0046         }
0047
0048         int a = GetDlgItemInt(IDC_EDIT1);
0049         int b = GetDlgItemInt(IDC_EDIT2);
0050
0051         int c;
0052         m_lpIMath->Add(a, b, &c);
0053
0054         CString str;
0055         str.Format("%d", c);
0056         SetDlgItemInt(IDC_EDIT3, c);
0057 }
0058
0059 void CSampleTestDlg::OnButton2()
0060 {
0061         IMath* lpIMath = NULL;
0062         HRESULT hr = CoCreateInstance(CLSID_Math, NULL, CLSCTX_INPROC_SERVER, IID_IMath, (LPVOID*)&lpIMath);
0063
0064         if (FAILED(hr))
0065         {
0066                 CString str;
0067                 str.Format("Error: 0x%08X", hr);
0068                 AfxMessageBox(str);
0069                 return;
0070         }
0071
0072         int a = GetDlgItemInt(IDC_EDIT4);
0073         int b = GetDlgItemInt(IDC_EDIT5);
0074
0075         int c;
0076         lpIMath->Sub(a, b, &c);
0077
0078         CString str;
0079         str.Format("%d", c);
0080         SetDlgItemInt(IDC_EDIT6, c);
0081
0082         lpIMath->Release();
0083 }
```

### 本章小结

本章中所做的实验，经过编译，生成如下程序。

- `Sample.dll` - `IMath`接口
- `SampleTest.exe` -- `IMath`测试程序。
- `SampleTest.html` -- `IMath`测试程序，必须使用IE浏览器打开。
- `SampleProxy.dll` -- 运用对象代理法挂接`IMath`接口中的`Add`，`Sub`函数
- `SampleVtable.dll` -- 运用虚表补丁法挂接`IMath`接口中的`Add`，`Sub`函数
- `SampleHook.exe` -- 用于调用`SampleProxy.dll`，`SampleVtable.dll`中提供的函数完成进程注入及挂钩。

通过挂钩一个真正的COM组件（`Sample.dll`），充分印证了前面章节中理论的正确性。并在实际的实践过程中，对这些理论有了更清晰的进一步认识和理解。但是，这仅仅是初始阶段，要想真正将挂钩COM接口的技术应用在各种场合，还会存在各种各样的困难。同时，出于对系统以及目标进程和稳定性，和效率的考虑，编程过程中，还有许多细节问题还需仔细处理。特别是目标COM组件的线程模型及同步问题。这些问题，将在实际的应用中遇到，并且需要开发者依据自身的技术功底，以及对众多问题的综合认识，才能够得以完成一个稳定且安全的挂钩程序。这些问题，已然超出了本文范围，还需开发者有的放矢针对具体问题做出最合适的处理。

## 第6章 实例：面向IE的网页弹窗拦截器

### 6.1 实现背景

毫无疑问，网络在今天已经渗透到每个人的工作，生活的方方面面。在使用网络的过程中，最重要的就是通过浏览器浏览互联网上万千变化的我们感兴趣的信息。在浏览网页的过程中，比较让人头疼的第一号问题就是病毒，木马。但随着网民的安全意识逐步提高，加之国内安全厂商已免费开放杀毒软件，所以，对于当前的环境，较前些年要好得很多。所以在生活中，其次让人头疼的问题，就是许许多多不道德的网站站长，为了追求利益，在网站上加了许许多多的弹窗广告，甚至是循环弹出，直到耗尽系统资源为止。有的甚至是色情的页面。鉴于此，有必要开发一款针对网页的弹窗拦截器。这个实例仅是针对IE浏览器。

### 6.2 网页弹窗原理1：`IHTMLWindow2Vtable::open`

知其然，才知其所以然。要想开发一个网页拦截器，必须得先知道，网页是如何弹出一个窗口的。对于IE浏览器来说，最为官方的弹出方法是调用IE浏览器内置的`window`对象的`open`方法。典型的调用方法如下：

```
<input type = "button" value = "button" onClick = "window.open('76aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4A6Z5j5h3&6Y4L8s2g2V1N6h3!0Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3U0N6Q4x3U0V1`.">
```
