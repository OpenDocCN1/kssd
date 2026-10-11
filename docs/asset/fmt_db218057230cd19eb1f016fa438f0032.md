
# Ring3 Inline 钩子库

> 原文：[`www.pediy.com/kssd/pediy11/125585.html`](https://www.pediy.com/kssd/pediy11/125585.html)

虽然已经有 `Detour` 等一批优秀的钩子实现库了，本着学习的原则还是弄了一下，参考了海风月影大神的挂钩库，汇编就学了个皮毛，大侠飘过，提供给需要的人参考参考……

这个库的优点：原始函数地址会作为第一个参数传递给 `Detour` 函数，`Detour` 函数的后面参数和原始函数一致，所以很方便实现调用前过滤和调用后过滤，反正就是使用很简单，也不需要自己记录原始函数地址了。为了方便移植到内核里使用，专门增加了 `fastcall` 的处理……先上一个例子，原始代码见附件了,看了例子就知道使用有多么简单了……

说一下那个 `Stub` 为啥不需要 `pushad` 之类的现场保护指令

`__asm`

```
{
    pop eax
    push OLD_CODE
    push eax///如果使用其它的寄存器可能会失败，一般是在 eax 里面返回数据，所以这个可以随便用
    _emit 0xE9 这个 E9 指令就是 JMP 指令，JMP 过去的那个函数自己破坏的现场编译器自己会很好的进行保存，那个函数给上层函数暴露的相当于一个标准的 API 函数，也就是说现场不会因为中途调用了这个函数而被破坏。而原始函数暴露给这个中继函数的也是一个标准的函数，和 API 无异，所以 Detour 直接调用不会有什么问题。当然这个库只能用于 Release 版本，道理很明显，在 Debug 里面，给出函数名最后编译得到的不是函数的开头地址，而是一个中继地址
                 多个 nop
OLD_CODE:
      _emit 0xCC
      nop
}
```

代码:

```
#include"windows.h"
#include"hooklib.h"
#include"stdio.h"
#include"ADE32.h"
////测试时要禁用自动内联函数，不然这里的测试函数太段了，可能会被优化掉……
typedef int (WINAPI *FUN)(char*);

int WINAPI My_LoadLibraryA(FUN f,char* Path)
{
  printf("监控到要加载模块！%s\n",Path);
  return f(Path);
}
typedef int (WINAPI * Message)(ULONG,wchar_t*,wchar_t *,ULONG);

int WINAPI My_MessageBoxW(Message f,ULONG hwnd,wchar_t* Mes,wchar_t * Cap,ULONG Type)
{
  wchar_t mes[]=L"哈哈!函数被 HOOK!";
  return f(hwnd,mes,L"HOOK",Type);
}

int __fastcall Test1(int a)
{
  printf("Test1 原始函数输出：%d\n",a);
  return 1;
}


typedef  DWORD (__fastcall *Detour_Tun)(int a);

/*
小于两个参数的话也要这样做
*/
int __stdcall Function1(Detour_Tun Fun,int a,int Edx)
{
  printf("Detour Fastcall ^-^\n");
  return Fun(a);
}

void main()
{
  HOOK_INFO Info={0};
  MessageBoxA(NULL,"This is a Test String!","TEST",MB_OK);
  InstallInlineByName("Kernel32.dll","LoadLibraryA",My_LoadLibraryA,&Info);
  InstallInlineByName("user32.dll","MessageBoxW",My_MessageBoxW,NULL);
  MessageBoxW(NULL,L"This is a Test String!",L"TEST",MB_OK);
  LoadLibraryA("Test.dll");
  UnInstallInline(&Info);
  Test1(6);
  InstallInlineForFastCall((unsigned char *)Test1,(unsigned char *)Function1,NULL,1);
  Test1(6);
  system("pause");
}
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | | `HooklibEx.rar` |
|---|---|---|
