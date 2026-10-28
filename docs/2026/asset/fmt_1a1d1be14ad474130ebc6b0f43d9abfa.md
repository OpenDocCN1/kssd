
`int` `HookIAT(` `char` `* szModuleName,` `char` `* szFuncName,` `int` `NewFuncAddress)`  `{`  `if` `(szModuleName == NULL || szFuncName == NULL || NewFuncAddress == NULL)`  `{`  `return` `-1;`  `}`    `int` `ImageBase = (``int``)GetModuleHandle(NULL);`    `PIMAGE_DOS_HEADER pDosHeader;`  `PIMAGE_NT_HEADERS pNtHeader;`    `//此处暂时不考虑64位的情况，所以直接定义32位的选项头`  `PIMAGE_OPTIONAL_HEADER32 pOptionalHeader;`    `pDosHeader = (PIMAGE_DOS_HEADER)ImageBase;`  `if` `(pDosHeader->e_magic != 0x5A4D) {`  `return` `-1;`  `}`    `//获取NT头`  `pNtHeader = (PIMAGE_NT_HEADERS)(pDosHeader->e_lfanew + ImageBase);`  `if` `(pNtHeader->Signature != 0x00004550) {`  `return` `-1;`  `}`    `//获取选项头`  `pOptionalHeader = &pNtHeader->OptionalHeader;`    `IMAGE_DATA_DIRECTORY DataDirectoryImport = pOptionalHeader->DataDirectory[1];`    `//获取导入表地址`  `PIMAGE_IMPORT_DESCRIPTOR pImportDirectory = (PIMAGE_IMPORT_DESCRIPTOR)(DataDirectoryImport.VirtualAddress + ImageBase);`    `//获取需要hook的函数所在的模块对应的模块句柄`  `HMODULE` `hModule = GetModuleHandle(szModuleName);`  `if` `(hModule == NULL)`  `{`  `return` `-1;`  `}`    `//获取原函数的位置`  `int` `pFunc = (``int``)GetProcAddress(hModule, szFuncName);`  `if` `(pFunc == NULL)`  `{`  `return` `-1;`  `}`    `while` `(pImportDirectory->OriginalFirstThunk != 0)`  `{`  `char``* szDllName = (``char``*)(pImportDirectory->Name + ImageBase);`    `if` `(!``strcmp``(szDllName, szModuleName))`  `{`  `DWORD``* pAddress = (``DWORD``*)(pImportDirectory->FirstThunk + ImageBase);`  `while` `(*pAddress != 0)`  `{`  `if` `(*pAddress == pFunc)`  `{`  `DWORD` `OldProtect = 0;`    `//修改内存属性，使其可写`  `VirtualProtect(pAddress, 0x1000, PAGE_EXECUTE_READWRITE, &OldProtect);`  `*pAddress = (``DWORD``)NewFuncAddress;`    `//改回内存属性，防止兼容性问题`  `VirtualProtect(pAddress, 0x1000, OldProtect, &OldProtect);`  `break``;`  `}`  `pAddress++;`  `}`  `break``;`  `}`  `pImportDirectory++;`  `}`  `return` `0;`  `}`

#### 修改窗口过程函数

接着，我需要在hook的函数当中修改窗口过程函数，我选择hook的函数是GetMessage，因为这个函数调用的时机一定是在窗口创建之后，而且调用非常频繁。如果有其它合适的函数也可以hook。

下面是我修改窗口过程函数的具体流程，首先通过软件标题获取主窗口的句柄，由于该软件支持多种语言，所以有多种标题的可能，此处全都给它查找一下，考虑到标题重名的可能性，所以判断一下获取到的窗口句柄是否属于该进程。

然后通过IsWindowUnicode函数判断该窗口的字符集，然后调用对应版本的SetWindowLong函数修改窗口的过程函数为自己的函数，**这一步骤非常重要，如果调用的函数字符集不对，修改会失败，而且GetLastError也不会报错**，是一个非常难以排查的错误，张老师之前也反复强调过这个问题。

修改完之后再调用原先的GetMessageA函数。

由于窗口过程函数修改一次就好，所以使用一个IsModify标志来判断是否已经被修改过，防止反复修改。

**代码实现：**

```
//记录过程函数是否已经被修改
BOOL IsModify = FALSE;

BOOL WINAPI MyGetMessageA(LPMSG lpMsg,HWND hWnd,UINT wMsgFilterMin,UINT wMsgFilterMax)
{
if (!IsModify)
{
char* szWindowNames[3] = {
"全能字符串批量替换机7.0 无限制版(替换/查找/抽取/改名)" , //Chinese(S)
"つ匡ゅンい猔種称" , //Chinese(T)
"Super Replacer(Replace/Find/extract/rename)"              //English
};

HWND hWnd = NULL;
bool IsMyWindow = false;

//查找窗口
for (int i = 0; hWnd == NULL && i < 3; i++)
{
hWnd = ::FindWindow(NULL, szWindowNames[i]);
}

if (hWnd != NULL)
{
DWORD ProcessId = 0;

//判断窗口是否属于本进程
if (GetWindowThreadProcessId(hWnd, &ProcessId) != 0)
{
DWORD MyProcessId = GetCurrentProcessId();
if (MyProcessId == ProcessId)
{
IsMyWindow = true;
}
}
}

if (IsMyWindow)
{
g_Wnd = hWnd;

//判断窗口字符集
BOOL nRet = IsWindowUnicode(hWnd);

//修改窗口过程函数
if (nRet)
{
g_OldWindowProc = (PFN)SetWindowLongW(hWnd, GWL_WNDPROC, (int)WindowProc);
}
else
{
g_OldWindowProc = (PFN)SetWindowLongA(hWnd, GWL_WNDPROC, (int)WindowProc);
}

IsModify = TRUE;
}
}

//调用原先被hook的函数
return GetMessageA(lpMsg, hWnd, wMsgFilterMin, wMsgFilterMax);
}
```

接着就是在自己的窗口过程函数中实现具体的功能了。

#### 窗口过程函数功能实现

主要功能就是判断来的消息是否是WM\_NOTIFY消息，如果是，就判断它传过来的NMHDR结构体中的code是不是对应LVN\_COLUMNCLICK通知号，接着判断控件ID是否是我想要的，然后判断点击的列数是不是我想要的列。如果全都满足要求，那就对列表控件的各个项按个数进行排序。其它情况全都交给原先的窗口过程函数处理。

这里有一个问题，就是如何获取列表控件的ID号，这个ID号存储在原程序的资源节。

我们可以使用vs将原程序以资源文件的形式打开：

![](https://bbs.kanxue.com/upload/attach/202402/980660_KBWZ2CM9GGTX9S9.webp)

vs会自动解析它的资源节，我们就可以从中获取它的ID号为 1012：

![](https://bbs.kanxue.com/upload/attach/202402/980660_VQ7EUJ5ZE64XCSM.webp)

此时还有一个小细节，就是列表控件的排序函数SortItem，使用时需要先用SetItemData来为其设置上对应项的标号，否则传给比较函数的值会出现问题，如果想避免这个问题，可以考虑调用它的Ex版本，也就是SortItemsEx函数。

**代码实现：**

```
#define ID_LISTCTRL 1012

//旧的窗口过程函数
PFN g_OldWindowProc = NULL;

//记录过程函数是否已经被修改
BOOL IsModify = FALSE;

HWND g_Wnd;

int g_nClickedColumn;
CListCtrl* g_pListCtrl;
bool IsReverseOrder = false;
int CALLBACK CompareFunc(LPARAM lParam1, LPARAM lParam2, LPARAM lParamSort)
{
CListCtrl* pListCtrl = g_pListCtrl;
CString strItem1 = pListCtrl->GetItemText(lParam1, g_nClickedColumn);
CString strItem2 = pListCtrl->GetItemText(lParam2, g_nClickedColumn);

int nItem1 = StrToInt(strItem1);
int nItem2 = StrToInt(strItem2);
int ret = nItem2 - nItem1;

if (!IsReverseOrder)
{
ret = 0 - ret;
}

return ret;
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
if (uMsg == WM_NOTIFY)
{
NMHDR* pNMHDR = (NMHDR*)lParam;
if (pNMHDR->code == LVN_COLUMNCLICK)
{
NMLISTVIEW* pNMLV = (NMLISTVIEW*)pNMHDR;
if (pNMLV->hdr.idFrom == ID_LISTCTRL) // 检查控件ID，确保是你要处理的列表控件
{
HWND hListCtrl = ::GetDlgItem(g_Wnd, ID_LISTCTRL);
CListCtrl* pListCtrl = reinterpret_cast<CListCtrl*>(CListCtrl::FromHandle(hListCtrl));
g_pListCtrl = pListCtrl;

g_nClickedColumn = pNMLV->iSubItem;
if (g_nClickedColumn == 2)
{
//将每一项进行排序
pListCtrl->SortItemsEx(CompareFunc, (LPARAM)pListCtrl);

//再次点击的话，就将顺序颠倒
IsReverseOrder = !IsReverseOrder;
}
return TRUE;
}
}
}

return g_OldWindowProc(hwnd, uMsg, wParam, lParam);
}
```

## 总结

到这个，我们就完成了对数据批量处理软件功能的添加，由于没有这个软件的源码，所以只能通过注入代码，对它打补丁的形式为其添加功能，这种方法也适用于其它一些没有源代码，并且原作者也没有提供插件扩展功能，但是需要对软件功能做调整的情况。但是其中涉及到的知识点以及细节很多，有一步出错就可能导致功能添加失败或者影响到原程序的运行，所以修改的时候需要非常谨慎，如果需要修改软件，就要多做备份，最好每一步都对上一步完成的软件存一个备份，防止出现意外情况。这就是这篇帖子的全部内容了，非常感谢各位的阅读。

科锐47期学员

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2024-3-5 15:28

被kanxue编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [chrep.zip](javascript:void(0);)

  （1.45MB，87次下载）
