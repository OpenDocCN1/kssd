
# 逆一个驱动中带 Exception 的函数

> 原文：[`www.pediy.com/kssd/pediy11/122227.html`](https://www.pediy.com/kssd/pediy11/122227.html)

金山推出了隐私保护器，拿来看看：

下载地址：`http://bbs.duba.net/thread-22314119-1-1.html`

`1013`版本。压缩包中只有一个`EXE`文件。不需要安装，直接运行。开始监控后，在目录：

`C:\\Documents and Settings\\MyUserName\\Application Data\\kprivacy.exe\\`

下发现一个文件

`ksafefilemon.sys`文件长度`129128`

这就是它的驱动了。要实现文件监控，一般还是要驱动的。

用`IDA`打开，分析一下这个驱动。

第一个函数`00012100`明显可以改名为`AllocMem`

第二个函数`00012120`明显可以改名为`FreeMem`

第三个函数`00012140`中有这么一句：

```
.text:0001214A                push    offset _except_handler3
```

显然用到了`Exception`。这种带`Exception`代码的逆向，一般是记不住的。为了逆它，写一个测试函数：

```
void puts(const char* p);

void test()
{
  puts("first");
  __try
  {
    puts("seccond");
  }
  __except(0)
  {
    puts("third");
  }
}
```

把以上测试代码用`DDK 7600.16385.0 fre x86 WXP`编译，用`IDA`打开生成的`OBJ`文件，`test`函数中只出现

```
  call __SEH_prolog4
```

不行。不是我们所要的。

把以上测试代码用`DDK 7600.16385.0 chk x86 WXP`编译，用`IDA`打开生成的`OBJ`文件，`test`函数中出现了

```
  push offset __except_handler4
```

也不行。

为什么我们出现了`__except_handler4`而没有出现`_except_handler3`呢？网上搜索一下，原来，新版的`DDK`使用了`MSVC8`作为编译器，就改为`__except_handler4`了。看来要找一个旧版的`DDK`试一试。

网上搜索，

`WIN DDK 3790.1830 ISO 下载`

`http://download.microsoft.com/download/9/0/f/90f019ac-8243-48d3-91cf-81fc4093ecfd/1830_usa_ddk.iso`

把测试代码用`DDK 3790.1830 fre x86 WXP`编译，用`IDA`打开生成的`OBJ`文件，`test`函数中出现了

```
  call __SEH_prolog
```

也不行。

把测试代码用`DDK 3790.1830 chk x86 WXP`编译，用`IDA`打开生成的`OBJ`文件，`test`函数中出现了

```
  push    offset __except_handler3
```

终于找到了！

虽然这个`__except_handler3`比我们要找的`_except_handler3`多了一个下划线，不去深究了，我们认为它就是一样的。

还有，难道金山的这个驱动文件，是用`Checked Build`编译的？而不是一般的`Free Build`?

对比我们写的`test`函数和`sub_12140`，就容易知道它是怎么使用`__try` `__except`的了。

最后，逆向出来的函数是这样的：

```
#include <excpt.h>

#define OUT

typedef unsigned short wchar_t;

int __stdcall sub_12140(const char* a0, OUT wchar_t* a4, unsigned short a8_len)
{
  int i = 0;  //v1c
  __try
  {
    for (i = 0; i < a8_len; i++)
    {
      a4[i] = a0[i];
    }
    return 0;
  }
  __except(EXCEPTION_EXECUTE_HANDLER) //EXCEPTION_EXECUTE_HANDLER = 1
  {
    return 0xE00B0002;
  }
}
```

原始的汇编是这样的：

```
sub_12140       proc near                 ; CODE XREF: sub_1AFC0+D9p
var_20            = dword ptr -20h
var_1C            = dword ptr -1Ch
var_18            = dword ptr -18h
var_10            = dword ptr -10h
var_4             = dword ptr -4
arg_0             = dword ptr  8
arg_4             = dword ptr  0Ch
arg_8             = word ptr  10h
                  push    ebp
                  mov     ebp, esp
                  push    -1
                  push    offset dword_27C98
                  push    offset _except_handler3
                  mov     eax, large fs:0
                  push    eax
                  mov     large fs:0, esp
                  add     esp, 0FFFFFFF0h
                  push    ebx
                  push    esi
                  push    edi
                  mov     [ebp+var_18], esp
                  mov     [ebp+var_20], 0E00B0001h
                  mov     [ebp+var_1C], 0
                  mov     [ebp+var_4], 0
                  mov     [ebp+var_1C], 0
                  jmp     short loc_1218D
; ---------------------------------------------------------------------------
loc_12184:                   ; CODE XREF: sub_12140+6Aj
                  mov     eax, [ebp+var_1C]
                  add     eax, 1
                  mov     [ebp+var_1C], eax
loc_1218D:                   ; CODE XREF: sub_12140+42j
                  movzx   ecx, [ebp+arg_8]
                  cmp     [ebp+var_1C], ecx
                  jge     short loc_121AC
                  mov     edx, [ebp+arg_0]
                  add     edx, [ebp+var_1C]
                  movsx   ax, byte ptr [edx]
                  mov     ecx, [ebp+var_1C]
                  mov     edx, [ebp+arg_4]
                  mov     [edx+ecx*2], ax
                  jmp     short loc_12184
; ---------------------------------------------------------------------------
loc_121AC:                   ; CODE XREF: sub_12140+54j
                  mov     [ebp+var_4], -1
                  jmp     short loc_121D5
; ---------------------------------------------------------------------------
loc_121B5:                   ; DATA XREF: .text:00027C9Co
                  mov     eax, 1
                  retn
; ---------------------------------------------------------------------------
loc_121BB:                   ; DATA XREF: .text:00027CA0o
                  mov     esp, [ebp+var_18]
                  mov     [ebp+var_20], 0E00B0002h
                  mov     [ebp+var_4], -1
                  jmp     short loc_121DC
; ---------------------------------------------------------------------------
                  mov     [ebp+var_4], -1
loc_121D5:                   ; CODE XREF: sub_12140+73j
                  mov     [ebp+var_20], 0
loc_121DC:                   ; CODE XREF: sub_12140+8Cj
                  mov     eax, [ebp+var_20]
                  mov     ecx, [ebp+var_10]
                  mov     large fs:0, ecx
                  pop     edi
                  pop     esi
                  pop     ebx
                  mov     esp, ebp
                  pop     ebp
                  ret     0Ch
sub_12140       endp

dword_27C98     dd -1                    ; DATA XREF: sub_12140+5o
                  dd offset loc_121B5
                  dd offset loc_121BB
```
