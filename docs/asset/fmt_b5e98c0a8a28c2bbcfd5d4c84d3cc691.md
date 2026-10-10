
# Microsoft Office Web Components. CVE-2009-1136 分析

> 原文：[`www.pediy.com/kssd/pediy10/94756.html`](https://www.pediy.com/kssd/pediy10/94756.html)

Microsoft Office Web Components. CVE-2009-1136 分析

粗略一看，这个漏洞似乎跟 `OWC10.DLL` 跟没有什么关系，细跟 `Evaluate`、`msDataSourceObject` 貌似很难发现有什么不妥的代码，但它的确就是 `Evaluate` 的某个 BUG 导致漏洞被利用,(似乎跟 `msDataSourceObject` 没啥关系)。

`Evaluate` 的声明如下：

```
HRESULT _Evaluate(
                        [in] VARIANT Expression,
                        [out, retval] VARIANT* Result);
```

它的主要功能是计算一个表达式的值，并将计算的结果送到 `*Result`，它要求 `VARIANT` 型的 `Expression.VARTYPE` 必须为 `VT_BSTR`,如果不为 `VT_BSTR` 则会把输入原样送到输出变量 `*Result` 中

造成漏洞的主要代码是 `Evaluate` 中的

```
.text:38AC76D2 66 83 7D 0C 08          cmp     [ebp+arg_4], 8
.text:38AC76D7 74 11                   jz      short loc_38AC76EA
.text:38AC76D9 8D 75 0C                lea     esi, [ebp+arg_4]
.text:38AC76DC 8B 7D 1C                mov     edi, [ebp+pvarg]
.text:38AC76DF A5                      movsd
.text:38AC76E0 A5                      movsd
.text:38AC76E1 A5                      movsd
.text:38AC76E2 A5                      movsd
.text:38AC76E3 33 C0                   xor     eax, eax
.text:38AC76E5 E9 0C 03 00 00          jmp     loc_38AC79F6
```

意思差不多就是如下:

```
if (Expression.VARTYPE !=  VT_BSTR)
{
*Result = Expression;
goto exit;
}
else
{
//执行表达式的计算
...........................
//将运算结果送入*Result
*Result = xxxxxxxxxx;
}
```

表面上看逻辑似乎没有什么问题。

再看看漏洞利用脚本是怎么利用该函数的：

```
e.push(1);
e.push(2);
e.push(0);
e.push(window);
for(i=0;i<e.length;i++)
{
    for(j=0;j<10;j++)
    {
        try
        {
            obj.Evaluate(e[i]);
        }
        catch(e)
        {
        }
    }
}
window.status=e[3] +'';
```

`Evaluate` 有一次会传入 `window` 对象，循环 10 次，上面说过如果传入的参数不为 `VT_BSTR` 的话，`Evaluate` 会将输入原样送到输出，`window` 对象的类型为 `IDISPATCH` 类型，所会会执行 `*Result = Expression;` 操作，如若把 `window` 对象也换成数字或字符串的话，漏洞不会被利用，问题已经很明显了，COM 规范里有说过："当把一个非空接口指针写到局部变量中时、当被调用方返回一个非空接口指针作为函数的实际结果时，都要增加接口的引用计数"

但是，为什么不增加引用计数会造成如此严重漏洞？

用工具查找到 `Evaluate` 地址后，观察传入 `window` 对象时的栈信息

```
0012DFC8   770F73D0  RETURN to OLEAUT32.770F73D0
0012DFCC   02890110
0012DFD0   00390009
0012DFD4   0012E43C
0012DFD8   012014B0  <-----window 对象
0012DFDC   0012E41C
0012DFE0   0012E034  <-------用来指向返回值
      HRESULT _Evaluate([in] VARIANT Expression,
                       [out, retval] VARIANT* Result);
```

对应栈中数据，一个 `THIS` 指针（1 个 `DWORD`），一个 `VARIANT` 结构(4 个 `DWORD`)，一个 `VARIANT` 指针(一个 `DWORD`)

`Window` 对应的 `VARIANT` 结构信息中

`VARTYPE =9 (VT_DISPATCH)`

`pdispVal = 012014B0`

这个 `pdispVal` 指向的是一个 `TEAROFF_THUNK` 结构（很像一个对象，有虚函数表），在 `WIN2K` 源码中可以找到

```
struct TEAROFF_THUNK
{
    void *      papfnVtblThis;     // Thunk's vtable
    ULONG       ulRef;             // Reference count for this thunk.
    DWORD       unused;            // BUGBUG delete when can be compiled and tested on ALPHA
    void *      pvObject1;         // Delegate other methods to this object using...
    void *      apfnVtblObject1;   // ...this array of pointers to member functions.
    void *      pvObject2;         // Delegate methods to this object using...
    void *      apfnVtblObject2;   // ...this array of pointers to member functions...
    DWORD       dwMask;            // ...the index of the method is set in the mask.
}
```

第一次对 `window` 对象 `evalute` 时 `pdispVal` 指向的数据为

```
012014B0  70 C2 F1 7C 05 00 00 00 CC EF C8 7C 30 0F 20 01  p 埋|...田 0  
012014C0  28 49 C9 7C 50 8F 20 01 B4 F2 C8 7C 07 00 00 00  (IP?打...
```

其中偏移 `0X04` 处的数据为 `rlRef` 也就是引用计数，这时为 5（该值为 JS 脚本中对 `window` 对象循环 `evaluate` 的最小值）

每执行完一次 `evaluate` 之后，`oleaut32.dll` 会以 `Evaluate` 函数的返回值 `VARIANT* Result` 作为参数调用 `VariantClear`，该函数会检测参数类型，如果为 `VT_DISPATCH` 的话会调用它的 `Release` 函数，`Release` 函数对应于 `0X7CF1C270`(papfnVtblThis)的 `0x08` 偏移位置的数据，带着符号看该地址的话，函数名为 `PlainRelease`,该函数的代码也可以从 `WIN2K` 中找到。

说说 `PlainRelease` 的功能（只说和漏洞相关的重点），它将传入对象的引用计数减 1，也就是上面的 `TEAROFF_THUNK` 的 `ulRef` 减 1，判断是否为 0，如果为 0，则调用 `InterlockedExchangePointer` 将该 `TREAOFF_THUNK` 指针存入静态变量 `s_pvCache1` 中，同时会判断原 `s_pvCache1` 的值，如果不为 0，则调用 `InterlockedExchangePointer` 将原 `s_pvCache1` 的值存入静态变量 `s_pvCache2` 中，也会判断原 `s_pvCache2` 的值，如果为不为 0 则调用 `MemFree` 来释放该原 `s_pvCache2` 指向的 `0X20` 大小的 `TEAROFF_THUNK`，这里的 `s_pvCache1` 和 `s_pvCache2` 为什么要用在这里？（猜测是缓存用的，避免不停的分配这 `0X20` 大小的数据，`MSHTML` 和 `JSCRIPT` 里面有很多这样的 `TEAROFF_THUNK` 结构）

当执行五次 `obj.Evaluate(window)` 后 `window` 对象的引用计数被减为 0，则会执行上面说的操作:"将 `window` 对象对应的 `TEAROFF_THUN` 结构指针存入 `s_pvCache1` 中",就这样，好端端的 `window` 对象，引用计数被无辜的减到了 0，虽然引用计数被减为 0，但是它对应的 `TEAROFF_THUNK` 结构还是存在的，没有被释放，当下次要用 `TEAROFF_THUNK` 结构时，它会被"废物"重利用，何时会被利用？

在漏洞利用脚本中有这么一条“`window.status=e[3] +'';`”，它会触发先前被缓存到 `s_pvCache1` 的 `TEAROFF_THUNK` 被再利用，功能存在于 `CreateTearOffThunk`，该函数在 `WIN2K` 源码中也可以找到。

说说 `CreateTearOffThunk`（只说和漏洞相前的重点），它会检查 `s_pvCache1` 或 `s_pvCache2`

是否为 0，如果都为 0 的话会 `MemAlloc` 一个 `TEAROFF_THUNK` 结构，随后会根据参数来填充 `TEAROFF_THUNK` 结构。

当 `window.status=e[3] +'';` 执行时，首先会取查询 `window` 接口，会调用 `CWindow__PrivateQueryInterface` 于是就进入 `CreateTearOffThunk` 函数，这时先前被缓存的 `TEAROFF_THUNK` 指针 `0X012014B0` 被会再利用，`TEAROFF_THUNK` 的 `apfnVtblObject1`(偏移 `0X10`)会被置为 `0X7CC8EFA4`(XP SP2 IE6 MSHTML VERSION:6.0.2900.2180)

```
<mshtml. CWindow__PrivateQueryInterface>
7CCD7A1E       8BFF                mov     edi, edi
  8BFF                mov     edi, edi
7CCD7A20            55                   push    ebp
7CCD7A21            8BEC                 mov     ebp, esp
7CCD7A23            53                   push    ebx
7CCD7A24            8B5D 10              mov     ebx, dword ptr [ebp+10]
7CCD7A27            56                   push    esi
7CCD7A28            8B75 0C              mov     esi, dword ptr [ebp+C]
7CCD7A2B            33D2                 xor     edx, edx
7CCD7A2D            8913                 mov     dword ptr [ebx], edx
7CCD7A2F            8B06                 mov     eax, dword ptr [esi]
7CCD7A31            B9 26442C33          mov     ecx, 332C4426
7CCD7A36            3BC1                 cmp     eax, ecx
7CCD7A38            57                   push    edi
7CCD7A39            0F87 21700000        ja      7CCDEA60
7CCD7A3F            0F84 3BD00600        je      7CD44A80
7CCD7A45            B9 B1F65030          mov     ecx, 3050F6B1
7CCD7A4A            3BC1                 cmp     eax, ecx
7CCD7A4C            0F86 FC2C0200        jbe     7CCFA74E
7CCD7A52            2D CFF65030          sub     eax, 3050F6CF
7CCD7A57            0F84 E5120A00        je      7CD78D42
7CCD7A5D            83E8 0D              sub     eax, 0D
7CCD7A60            0F84 ED910200        je      7CD00C53
7CCD7A66            2D 09010000          sub     eax, 109
7CCD7A6B            0F85 BE120A00        jnz     7CD78D2F
7CCD7A71            BF 5C42CB7C          mov     edi, 7CCB425C
7CCD7A76            6A 04                push    4
7CCD7A78            59                   pop     ecx
7CCD7A79            33C0                 xor     eax, eax
7CCD7A7B            F3:A7                repe    cmps dword ptr es:[edi], dword pt>
7CCD7A7D            0F85 4E700000        jnz     7CCDEAD1
7CCD7A83            52                   push    edx
7CCD7A84            53                   push    ebx
7CCD7A85            52                   push    edx
7CCD7A86            68 A4EFC87C          push    7CC8EFA4  <-----这个值指向函数表
7CCD7A8B            FF75 08              push    dword ptr [ebp+8]
7CCD7A8E            E8 EAC4FFFF          call    <CreateTearOffThunk>
```

接着再对 `window.status` 赋值一下，这样会触发一个 `GetIDsOfNames` 的操作和 `invoke` 的操作，在 `IDISPATCH` 可以提供接口函数给脚本来调用，脚本不知道函数的址址，而直接给出函数名来调用，最终会转入 COM 系统中由 `IDISPATCH` 的 `GetIDsOfNames` 获取指定函数的 ID,然后根据获取的 ID 来调用 `Invoke`，最终完成脚本所要执行的操作。

当要获取 `status` 的 ID 时会执行到 `JSCRIPT.DLL` 的

```
75BD2E67     FF75 18              push    dword ptr [ebp+18]
75BD2E6A     8B45 0C              mov     eax, dword ptr [ebp+C]
75BD2E6D     FF75 14              push    dword ptr [ebp+14]
```
