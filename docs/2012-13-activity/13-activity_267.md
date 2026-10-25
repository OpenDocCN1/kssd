
# 第一阶段第三题答案

> 原文：[`www.pediy.com/kssd/pediy10/74378.html`](https://www.pediy.com/kssd/pediy10/74378.html)

\=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=- TestFloat 分析记录 -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=  

写一个程序和 `TestFloat` 对照着调试，确定最终出错的函数位置。为了方便，我这里直接调试 `TestFloat` 本身。

用 `WinDbg` 载入 `TestFloat` 程序，设置 `crt` 源码的位置：

```
.sympath+ d:\\Program Files\\Microsoft Visual Studio 8\\VC\\crt\\src
```

`g @$exentry` 执行到 `TestFloat` 入口点位置，在 `Crash` 按钮函数上下断点：

```
bp TestFloat!CTestFloatDlg::OnBnClickedBtnCrash
```

继续执行，点击 `Crash` 按钮，中断到 `WinDbg` 后，单步进入 `snwprintf` 函数内。继续单步慢慢执行，发现在 `_woutput_l` 函数上程序崩溃，所以重新执行 `TestFloat`，单步进入 `_woutput_l` 函数内。继续单步慢慢执行，发现在 `_cfltcvt_l` 函数上程序崩溃。

```
00413de9 8d459c          lea     eax,[ebp-64h]
00413dec 50              push    eax
00413ded ff7594          push    dword ptr [ebp-6Ch]
00413df0 0fbec2          movsx   eax,dl
00413df3 ff75e8          push    dword ptr [ebp-18h]
00413df6 895dd8          mov     dword ptr [ebp-28h],ebx
00413df9 50              push    eax
00413dfa ff75e0          push    dword ptr [ebp-20h]
00413fd 8d4588          lea     eax,[ebp-78h]
00413e00 56              push    esi
00413e01 50              push    eax
00413e02 ff35c8a04200    push    dword ptr [TestFloat!_cfltcvt_tab+0x18 (0042a0c8)]
00413e08 e83fefffff      call    TestFloat!_decode_pointer (00412d4c)
00413e0d 59              pop     ecx
00413e0e ffd0            call    eax
```

反汇编 `_cfltcvt_l` 发现该函数对应成两个函数调用，先通过 `_decode_pointer` 函数解码出第二个函数的地址，然后调用第二个函数。执行到第一个函数调用后，查看返回值。

```
0:000> ln @eax
f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c(46)
(0041ae1f)   TestFloat!_fptrap   |  (0041ae28)   TestFloat!_isdigit_l
Exact matches:
    TestFloat!_fptrap (void)
```

再往下执行 `call eax` 就会出错，正常情况下这里解码出来的函数应该是 `_cfltcvt_l`，而不是 `_fptrap`。所以跟进 `_decode_pointer` 函数看看为啥解码出来的函数不对。

跟进去后发现 `_decode_pointer` 函数一切正常，估计就是 `_cfltcvt_tab` 中的值本来就不对。所以下一步跟踪一下 `_cfltcvt_tab` 的初始化过程。

查看 `cmiscdat.c` 源文件，估计 `_cfltcvt_tab` 是一个指针数组，里面存放着支持浮点运算的一些函数，在初始化的时候加密起来存放，然后在使用的时候把函数指针解密出来调用。

重新加载 `TestFloat`，`g @$exentry` 执行到入口点，显示一下 `_cfltcvt_tab` 的信息：

```
0:000> dps TestFloat!_cfltcvt_tab l10
0042a0b0  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0b4  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0b8  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0bc  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0c0  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0c4  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0c8  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0cc  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0d0  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0d4  0041ae1f TestFloat!_fptrap [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0fp.c @ 46]
0042a0d8  00000000
```

可以看到初始指针都指向 `_fptrap` 函数。下内存访问断点：

```
0:000> ba w4 TestFloat!_cfltcvt_tab
0:000> g
```

中断后查看堆栈：

```
0:000> kn
  #  ChildEBP  RetAddr  
00 0012fef8 0040f640 TestFloat!_initp_misc_cfltcvt_tab+0x1a [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\cmiscdat.c @ 58]
01 0012fefc 0040ebd8 TestFloat!_cinit+0x28 [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0dat.c @ 283]
02 0012ff88 76074911 TestFloat!_tmainCRTStartup+0x149 [f:\\sp\\vctools\\crt_bld\\self_x86\\crt\\src\\crt0.c @ 310]
03 0012ff94 7728e4b6 kernel32!BaseThreadInitThunk+0xe
04 0012ffd4 7728e489 ntdll!__RtlUserThreadStart+0x23
05 0012ffec 00000000 ntdll!_RtlUserThreadStart+0x1b
```

重新加载 `TestFloat`，在 `TestFloat!_cinit` 函数上下断点，执行中断后单步跟踪，很快就能看到如下代码：

```
        if (_FPinit != NULL &&
            _IsNonwritableInCurrentImage((PBYTE)&_FPinit))
        {
            (*_FPinit)(initFloatingPrecision);
        }
        _initp_misc_cfltcvt_tab();
```

`_initp_misc_cfltcvt_tab` 函数就是上面用来给 `_cfltcvt_tab` 中的指针做加密的，在这个函数之前有个 `if` 判断，从而决定是否初始化 `_cfltcvt_tab` 指针表。其中 `_IsNonwritableInCurrentImag` 函数判断 `PE` 文件的只读节是否具有可写属性，如果具有可写属性，则跳过对 `_cfltcvt_tab` 的初始化过程，`_cfltcvt_tab` 中保存的仍然是最初的 `_fptrap` 函数指针。往后程序中用到浮点操作时，解密后得到的 `_fptrap` 函数就会弹出浮点错误的提示。

`TestFloat` 程序编译时，我添加了 `/section:.rdata,rw` 链接选项，所以最终编译出来的 `TestFloat` 会出错，源代码上没有任何问题，并不是没有链接浮点库的原因。

这个 `bug` 是以前调试我们内部游戏加壳时发现的，是个实际的例子。直接从弹出的错误框来看，和网上能搜索到的答案不一样，觉得有点意思，所以弄出来给大家分析。谢谢大家的积极参与！

\=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=- end -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
