

# 用 dll 解 MoleBox 打包的文件的一些思路

> 原文：[`www.pediy.com/kssd/pediy11/104648.html`](https://www.pediy.com/kssd/pediy11/104648.html)

上次上网的时候，意外的学会了 dll 注入的方式，有想到 PE 结构一直掌握的不怎么好。于是结合了一下，通过写一个工具来进行实践。  
  这个工具就是 MoleBox 的解包工具。  
    
  因为个人经验有限，所以我只选择了 C 语言的程序来进行尝试。  
  关键在于这 msvcrt.dll 这个文件，这个玩意几乎所有的 C/C++语言的程序都会加载，然后它的导入表中又有一些我们需要的函数，于是就......  
    
  首先我们看看 MoleBox 打包的原理，我们自己写个程序，调用 CreateFile， ReadFile 等函数，再用 MoleBox 加上壳看看。  
  OD 跟一下，很容易发现 MoleBox 在 IAT 里把 CreateFile ReadFile LoadLibrary 等等系列函数全部劫持重写掉了。也就是当你打开文件时，它会先检查是不是包中的，然后特殊处理= =  
    
  很好，这样的话我的思路就是这样的。  
  1.首先让目标程序正常的运行  
  2.向目标进程注入我们自己用来解包的 DLL  
  3.DLL 的注入后想办法解包  
    
  注入的方法文章非常多，这里就不细究了。大致思路就是用 CreateRemoteThread 使得其执行 LoadLibraryA 函数。  
  主要介绍下 DLL 注入后怎么办：  
  首先我们要把其打包的所有文件全部找出来，我利用了以下一些 API：  
  FindFirstFile  
  FindNextFile  
  FindClose  
  要把其中的内容转存出来，我使用的 API 为:  
  CreateFileA  
  ReadFile  
  WriteFile  
  最后还有一个辅助的 API  
  CloseHandle  
    
  使用这些函数，应该很容易构造一个将一个文件夹下的文件转存到另一个文件夹下。  
  而其实我们只要把其中一些 API 换成被 MoleBox 劫持走的函数，岂不是就可以实现将包中的文件转存出来的功能。  
    
  那么，在考虑最后一个问题，怎么得到 MoleBox 劫持走的 API。  
    
  注意我最开始提到的 msvcrt.dll， 对了，它是个很方便的工具。  
  我们可以通过 GetModuleFileName 来得到 msvcrt.dll 的文件地址。 然后快乐的打开它  
  然后就可以开始分析这个 PE 文件了。  
  这个是一个标准的 DLL， 分析 PE 文件的文章也多的不可胜数了。 我们要做的事就是找出其导入表中我们需要的函数  
  然后再用 GetModuleInformation 来得到它在内存中的基址。这样就可以在内存中找到那几个需要的 API 对应的函数的地址的值了。  
    
  最后附上部分参考的代码  
 

代码:

```
 
  //DLL 主过程
  void MainProc() {
    CreateOutput() &&    //这个是我用来重定向 stdout 的， 以输出 LOG
    GetModuleBase() &&   //这个就是取 dll 装载的基址
    GetIATInfo() &&      //然后读取 IAT 信息
    SearchFiles();       //最后搜索文件并转存
  }
  
  //参考函数 ： GetModuleBase
  bool GetModuleBase() {
    MODULEINFO modui;
    ZeroMemory(&modui, sizeof(modui));
    if (!GetModuleInformation(GetCurrentProcess(),
      GetModuleHandle("msvcrt.dll"),
      &modui,
      sizeof(modui))) {
        printf("Can't get Module Information!\\n");
        return false;
    }
    
    dllBase = modui.lpBaseOfDll;
    return true;
  }

```

  
代码:

```
  //然后是最开始处的部分定义， 看得懂的就随便看看吧。
  //需要的 API 类型
  typedef HANDLE (WINAPI \*TCreateFileAFunc) (LPCSTR,DWORD,DWORD,LPSECURITY\_ATTRIBUTES,DWORD,DWORD,HANDLE);
  typedef HANDLE (WINAPI \*TFindFirstFileAFunc) (LPCSTR,LPWIN32\_FIND\_DATA);
  typedef BOOL (WINAPI \*TFindNextFileAFunc) (HANDLE,LPWIN32\_FIND\_DATA);
  typedef BOOL (WINAPI \*TReadFileFunc)(HANDLE,LPVOID,DWORD,LPDWORD,LPOVERLAPPED);
  typedef BOOL (WINAPI \*TCloseHandleFunc)(HANDLE); 
  typedef BOOL (WINAPI \*TFindCloseFunc) (HANDLE); 
  
  //对应的 API
  TCreateFileAFunc CreateFileAFunc;
  TFindFirstFileAFunc FindFirstFileAFunc;
  TFindNextFileAFunc FindNextFileAFunc;
  TReadFileFunc ReadFileFunc;
  TCloseHandleFunc CloseHandleFunc;
  TFindCloseFunc FindCloseFunc;

```

  
  
\--------------------------------------------------------------------------------  
废话：  
  这个程序写出来的话，应该不限于 MoleBox 打包的程序，采用类似的方法打包的壳应该都可以直接弄掉。 写的过程中学到了  
  很多东西，发现果然还是要实践才能够有所收获。  
  
  另提示的应该算是比较多了，于是不解释细节了。  
  再者：如果没有调用 msvcrt.dll 怎么办？这个只是思路一，找 IAT 的方法可多了，总能找到突破口的哦。