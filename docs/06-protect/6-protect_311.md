
# 手工隐藏 api 函数调用简单对付静态分析

> 原文：[`www.pediy.com/kssd/pediy10/94297.html`](https://www.pediy.com/kssd/pediy10/94297.html)

隐藏 api 函数调用对于对付静态分析，不用说是非常有意义的，许多加壳软件在输入表上大做文章，其中有一个重要的作用就是让破解者在得到的反汇编代码中看不到正常的 api 函数调用。我做一个最简单的测试，在不借助加壳软件和虚拟机保护技术的情况下，用手写代码的方式来实现隐藏 api 函数调用。

先看看我要保护的代码：

代码:

```
#include "windows.h"
int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  MessageBox(0,"Reverse Me","Test",0);
  return 0;
}
```

用 `IDA` 反汇编后，`MessageBoxA` 函数的调用暴露无遗：

引用:
```
.text:0040102A 6A 00               push    0                              ; uType
.text:0040102C 68 2C 00 42 00     push    offset Caption                 ; "Test"
.text:00401031 68 1C 00 42 00     push    offset Text                    ; "Reverse Me"
.text:00401036 6A 00             push    0                              ; hWnd
.text:00401038 FF 15 94 52 42 00 call    ds:__imp__MessageBoxA@16       ; MessageBoxA(x,x,x,x)
```

我们的任务就是让反汇编的代码看不到 `ds:__imp__MessageBoxA@16` ； `MessageBoxA(x,x,x,x)` 这样的提示。

## 1.最基本、最简单的方法

代码:

```
#include "windows.h"
typedef int (WINAPI *MYFUNC)(HWND hWnd,LPCTSTR lpText,LPCTSTR lpCaption,UINT uType);
int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  char MsgBoxA[]={0x5c,0x74,0x62,0x62,0x70,0x76,0x74,0x53,0x7e,0x69,0x50,0x00};  //字符串"MessageBoxA"的加密形式。
  char lpText[]={0x43,0x74,0x67,0x74,0x63,0x62,0x74,0x31,0x5C,0x74,0x00};  //字符串"Reverse Me"的加密形式。
  char lpCaption[]={0x45,0x74,0x62,0x65,0x00};  //字符串"Test"的加密形式。
  for(int i=0;i<strlen(MsgBoxA);i++)  MsgBoxA[i]^=0x11;  //解密字符串"MessageBoxA"
  for(i=0;i<strlen(lpText);i++)    lpText[i]^=0x11;  //解密字符串"Reverse Me"
  for(i=0;i<strlen(lpCaption);i++)  lpCaption[i]^=0x11;  //解密字符串"Test"
  HMODULE hMod=LoadLibrary("user32.dll");
  if(hMod)
  {
    MYFUNC func=(MYFUNC)GetProcAddress(hMod,MsgBoxA); //获取 MessageBoxA 的函数地址。
    func(0,lpText,lpCaption,0);  //调用 MessageBoxA 函数。
    FreeLibrary(hMod);
  }
  return 0;
}
```

现在 `MessageBoxA` 函数调用处 `func(0,lpText,lpCaption,0)`，用 `IDA` 反汇编的结果为：

引用:
```
.text:00401172 6A 00               push    0
.text:00401174 8D 55 E0           lea     edx, [ebp+var_20]
.text:00401177 52                 push    edx
.text:00401178 8D 45 E8           lea     eax, [ebp+var_18]
.text:0040117B 50                 push    eax
.text:0040117C 6A 00             push    0
.text:0040117E FF 55 D4           call    [ebp+var_2C]
```

## 2.不用 GetProcAddress

现在已经看不到赤裸裸的 `MessageBoxA` 了，而且 `IDA` 连个提示也不给了，但是调用的 `GetProcAddress` 函数同样是一个隐患，我们把它也隐藏起来：

代码:

```
。。。。。。
  MYFUNC func=NULL;
  char * pFuncName;
  HMODULE hMod=LoadLibrary("user32.dll");
  if(hMod)
  {
    PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)hMod;
    PIMAGE_NT_HEADERS pNtHeader  = (PIMAGE_NT_HEADERS)((ULONG)hMod+pDosHeader->e_lfanew);
    PIMAGE_OPTIONAL_HEADER32 pOptionalHeader = &(pNtHeader->OptionalHeader);
    PIMAGE_DATA_DIRECTORY pExportData = (PIMAGE_DATA_DIRECTORY)(&(pOptionalHeader->DataDirectory[0]));
    PIMAGE_EXPORT_DIRECTORY pExportTable = (PIMAGE_EXPORT_DIRECTORY)((ULONG)hMod+pExportData->VirtualAddress);
    ULONG * AddrFunctions=(ULONG *)((ULONG)hMod+pExportTable->AddressOfFunctions);
    ULONG * AddrNames=(ULONG *)((ULONG)hMod+pExportTable->AddressOfNames);
    for(i=0;i<pExportTable->NumberOfFunctions;i++)
    {
      pFuncName=(char *)((LONG)hMod+AddrNames[i]);
      if(strcmp(pFuncName,MsgBoxA)==0)
      {
        func=(MYFUNC)((LONG)hMod+AddrFunctions[i]);
        break;
      }
    }
    if(func)
      func(0,lpText,lpCaption,0);
    FreeLibrary(hMod);
  }
。。。。。。
```

## 3.SMC

现在连 `GetProcAddress` 也没有了，隐蔽性更好了，我们还想做得更隐蔽一点,用个简单的 `SMC`：

代码:

```
#pragma comment(linker, "/SECTION:.text,ERW")
void Decrypt(char * start,char * end)
{
  for(char * i=start;i<end;i++)
  {
    (*i)^=0x11;
  }
}
。。。。。。
  Decrypt((char * )401000,(char * )400000); //两个需要更正的地址参数。
  __asm inc eax  __asm dec eax  //16 进制对应 40 48 ，用来标记加密代码段起始地址
  HMODULE hMod=LoadLibrary("user32.dll");
。。。。。。
  __asm inc eax __asm dec eax //16 进制对应 40 48，用来标记加密代码段末地址
  return 0;
。。。。。。
```

编译后，用 `OD` 装载，查找二进制字串 `40 48`，定位我们要加密的代码：

引用:
```
004011B6  |.  68 801A0600   push    61A80     //加密末地址，需更正
004011BB  |.  68 681E0600   push    61E68  //加密起始地址，需更正
004011C0  |.  E8 40FeFFFF   call    00401005   //调用 Decrypt 函数
004011C5  |.  83C4 08       add     esp, 8
004011C8  |.  40            inc     eax
004011C9  |.  48            dec     eax
。。。。。
004012B6  |>  \40           inc     eax
004012B7  |.  48            dec     eax
```

我们在汇编状态下把 `Decrypt` 函数调用的参数更正为我们所加密的代码段的起始地址和末地址

引用:
```
004011B6      68 B8124000   push    004012B8  //加密末地址
004011BB      68 C8114000   push    004011C8  //加密起始地址
```

由于这是个简单的异或运算，加密和解密算法相同。我们就来个投机的办法，将程序运行到这一句 `004011C5 add esp, 8` 。此时 `Decrypt` 函数帮我们完成了加密，呵呵。然后复制到可执行文件->所有修改，接着保存文件即可完成 `SMC` 加密。

到这里已经完成了一个简单的 api 函数隐藏，但是这些方法，主要用来对付静态分析，如果动态跟踪的话就容易原形毕露了.这是我在学习过程中的一点心得,希望看雪的朋友们能多多指教.

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 完整代码.rar

 |
