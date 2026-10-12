
# 枚举全局钩子

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-327.htm`](https://www.pediy.com/kssd/pediy07/pediy7-327.htm)

作者: 一块三毛钱
邮箱: `zhongts@163.com`
日期: 2005.6.19

首先强调一点，本文给出的代码只在 `WinXP+SP2` 下测试通过，使用其他系统的朋友最好看懂代码，然后调试运行。如果不巧系统蓝屏可不要找我哦。:)

鼠标钩子、键盘钩子等大家一定是耳熟能详，在 `Windows` 环境下编程的朋友们肯定都和他们打过交道，比如说截获密码什么的都可以用到他们。本文将介绍一种办法可以枚举所有的全局钩子，找到到底是什么动态库创建了钩子（大部分全局钩子都需要通过动态库来实现）。

下图是程序运行后显示的所有的全局钩子，包括钩子函数在那个动态库中，也就是创建钩子的动态库。图中显示的钩子句柄是指得在当前进程中的句柄，不是系统范围的。比如说 `MouseHook` 添加了一个全局的鼠标钩子，那么在 `explorer` 中这个钩子的句柄和在 `word` 中的句柄是不同的（这一点我是通过 `Icesword` 观察到的），但都是 `MouseHook` 添加的钩子都在 `mousehook.dll` 动态库中。如果想获得 `Icesword` 一样的结果可以针对每一个进程枚举全局钩子（没试过）。另外，如果鼠标钩子和键盘钩子不能够显示动态库，可以先动一动鼠标，摁几下键盘再刷新应该就可以显示。

![](img/235f902cff2b2d46dbe917711f31c55f.jpg)

之所以会写这篇文章是因为在编程板块中有一个主题说判断全局钩子的调用者，所以才想着能不能枚举出所有的全局钩子。但那个主题好像说的是钩子的调用者，又好像全局钩子指得是 `Hook` API 类型的钩子，没搞懂。在那个主题下 `zworm` 大侠好像说可以通过一个钩子链的东东来枚举钩子，不过大侠没有细说，我自己也没有在符号文件中找到有关钩子链的符号（我只会这一种办法找系统内部变量，比如活动进程链就可以通过这种办法找到位置），所以准备从 `CallNextHookEx` 函数慢慢往下追希望能够找到系统是怎么找到下一个钩子函数的。本文介绍的方法就是这样来的（另外有一个很重要的一点会在文章的最后提到）。

```
invoke  PsGetCurrentThread
mov     ebx, [eax+130h]
mov     _gptiCurrent, ebx
invoke  DbgPrint, $CTA0("ETHREAD:%08X, Win32Thread:%08X\n"), eax, _gptiCurrent
assume  esi : nothing
mov     ebx, -1
.while ebx!=12
        invoke  _PhkFirstValid, _gptiCurrent, ebx
        assume  eax : ptr HOOK
        .while eax
                push    eax     ;_PhkNextValid 的参数
                ;根据 win32k.sys 中的 _xxxHkCallHook 得到下面的代码
                mov     edx, [eax].ihmod
                cmp     edx, -1
                jz      @F
                mov     esi, _gptiCurrent       ;esi -> ThreadInfo
                mov     esi, [esi+2Ch]          ;esi -> ProcessInfo
                mov     edx, [esi+edx*4+0A8h]   ;edx = [esi].ahmodLibLoaded[ihmod]
                @@:
                ;-------------------------------------
                m2m     [edi].Handle, [eax].hmodule
                m2m     [edi].FuncOffset, [eax].offPfn
                mov     [edi].FuncBaseAddr, edx
                m2m     [edi].iHook, [eax].iHook
                ;-------------------------------------
                invoke  DbgPrint, $CTA0("Handle:%08X    FuncOffset:%08X    FuncBaseAddr:%08X    iHook:%d    ihmod:%d\n"), \
                            [eax].hmodule, [eax].offPfn, edx, [eax].iHook, [eax].ihmod
                call    _PhkNextValid
                add     dwBytesReturned, sizeof HOOK_INFO
                add     edi, sizeof HOOK_INFO
        .endw
        inc     ebx
.endw
```

上面给出的就是核心代码，先找到线程的 `Win32Thread` 结构，通过这个结构就可以调用 `win32k.sys` 的内部函数 `_PhkFirstValid` 找到第一个钩子函数。然后再通过 `_PhkNextValid` 函数枚举所有的钩子函数。因为每个系统中这两个函数的地址不同，所以我把反汇编后的代码直接提取出来使用。这样可以避免因为系统不同的原因而出错。不过我只在 `WinXP+SP2` 下运行过，没有在其他的系统下测试，如果大家不能正确运行只好自己修改源代码啦。

`_PhkFirstValid` 函数接收两个参数，第一个是 `Win32Thread` 结构的地址，第二个是要枚举的钩子的类型，比如鼠标钩子。函数返回 `HOOK` 结构的地址。

```
HOOK struct
        hmodule         dd  ?
        _Z_             dd  4 dup (?)
        phkNext         dd  ?           ;14h
        iHook           dd  ?           ;18h
        offPfn          dd  ?           ;1Ch
        flags           dd  ?           ;20h
        ihmod           dd  ?           ;24h
        ptiHooked       dd  ?
        rpdesk          dd  ?
HOOK ends
```

`hmodule` 是钩子的句柄，

`_Z_` 这几个不知道是干什么的，

`phkNext` 指向下一个 `HOOK` 结构，

`iHook` 钩子类型，

`offPfn` 钩子函数的地址，一般都是相对钩子模块的基地址的偏移值，

`flags` 一些钩子的属性，比如说钩子是不是已经被销毁，

`ihmod` 不太清楚，好像是模块索引。

`_PhkNextValid` 函数通过当前钩子函数的 `HOOK` 结构找到下一个钩子函数的 `HOOK` 结构。

找到了钩子函数的 `HOOK` 结构后，再通过 `ihmod` 成员找到钩子模块的基地址。这一部分可以参考 `_xxxHkCallHook` 函数的代码。然后把这些信息返回给 `ring3` 下的程序显示出来。

本文给出的代码和结构很大一部分来之于大家手上都可能有的“那份”代码。:)

附件: `EnumHook.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
