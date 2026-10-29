
# [原创]How to use VBScript to turn on the God Mode?

关于Yuange vbscript safemode 介绍及debug :)

牛人可以主动忽略了。

![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)![](https://bbs.kanxue.com/view/img/face/27.gif)

How to use VBScript to turn on the God Mode?

怎么用VB开启仙人模式。。。

## What is the God Mode?

什么是神模式？

This concept is came from yuange, it means if we turn on this God Mode, we can do anything we what.

这个概念源于yuange，神模式的意思就是如果我们能开启神模式，我们可以做我们想做的任何事，譬如隐身进女澡堂。

## And What the God Mode really is?

那么到底什么是神模式？

We know if we want to execute the script code in the browser to create a object like Shell.Application, the script engine will check the SafeMode. if this is set, it will block the code. That is the God Mode. Yuange also call this is DVE(Data Virtual Execution), This is just execute the script, not the binary code. So the ASLR,DEP,EMET,CFI all of the defense technology will be fail.

我们的目的是在浏览器里通过执行脚本创建一个shell之类的东西。脚本引擎就会对`SafeMode`（安全模式）进行检查。如果浏览器设置了安全模式，我们的脚本就不会被执行。这就是神模式的本质。袁哥（一个很厉害的黑客）也把这个叫做DVE（数据虚拟执行），这仅仅只是执行脚本而不是二进制代码，所以什么ASLR（Address Space Layout Randomization,地址空间随机化），DEP（Data Execution Prevention，数据执行保护），EMET（Enhanced Mitigation Experience Toolkit，增强的缓解体验工具包），CFI（Control-Flow Integrity），所有的这些保护都不是事儿。

## What we are going to do is to clear the `SafeMode` flag.

(Notice: All the following code is vbscript code.)

I will show a demo code first.

我们接下来要做的工作就是去掉`SafeMode`（下面的代码都是VB脚本，我先做个演示）

```
"
<script type="text/javascript" language="VBScript">
set sh=createobject("Shell.Application")
sh.ShellExecute "calc.exe"


</script>
"
```

It can’t run because of the `SafeMode`.

## Let’s see the function `InSafeMode` in the `vbscript.dll`.

上面这段脚本是不能被执行的，因为浏览器开启了`SafeMode`。

我们看看在`vbscript.dll`里面的`InSafeMode`函数。

```
.text:7327624B ; int __thiscall COleScript::InSafeMode(COleScript *__hidden
this)
.text:7327624B ?InSafeMode@COleScript@@QAEHXZ proc near
.text:7327624B ; CODE XREF: VbsGetObject2(VAR *,int,VAR *)+8Ap
.text:7327624B ; sub_73282607+1Bp ...
.text:7327624B mov eax, [ecx+174h]
.text:73276251 and eax, 0Bh
.text:73276254 neg al
.text:73276256 sbb eax, eax
.text:73276258 neg eax
.text:7327625A retn
.text:7327625A ?InSafeMode@COleScript@@QAEHXZ endp
```

In this function it get the `safemode` flag first, then do some operations then return.

We make a break on this function, then rewrite the 0 to `[ecx+174h]`, then the vbs code will executed, the calc pop.

That’s cool.

这个函数里面先得到了`SafeMode`的标志位，然后做了一些操作就返回了。

我们在这个函数开头下个断点，然后向内存`[ecx+174h]`写进0，继续执行，我们的代码就可以执行了，经典的计算器小程序出现了。

这是不是很酷。

## Here I will show some base structures in the VBscript.

下面我再展示VB脚本里面的一些结构：

```
VAR struct

struct tagVARIANT

{

union

{

struct __tagVARIANT

{

VARTYPE vt;

WORD wReserved1;

WORD wReserved2;

WORD wReserved3;

union

{

LONG lVal;

BYTE bVal;

SHORT iVal;

FLOAT fltVal;

DOUBLE dblVal;

VARIANT_BOOL boolVal;

_VARIANT_BOOL bool;

SCODE scode;

CY cyVal;

DATE date;

BSTR bstrVal;

IUnknown __RPC_FAR* punkVal;

IDispatch __RPC_FAR* pdispVal;

SAFEARRAY __RPC_FAR* parray;

BYTE __RPC_FAR* pbVal;

SHORT __RPC_FAR* piVal;

LONG __RPC_FAR* plVal;

FLOAT __RPC_FAR* pfltVal;

DOUBLE __RPC_FAR* pdblVal;

VARIANT_BOOL __RPC_FAR* pboolVal;

_VARIANT_BOOL __RPC_FAR* pbool;

SCODE __RPC_FAR* pscode;

CY __RPC_FAR* pcyVal;

DATE __RPC_FAR* pdate;

BSTR __RPC_FAR* pbstrVal;

IUnknown __RPC_FAR*__RPC_FAR * ppunkVal;

IDispatch __RPC_FAR*__RPC_FAR * ppdispVal;

SAFEARRAY __RPC_FAR*__RPC_FAR * pparray;

VARIANT __RPC_FAR* pvarVal;

PVOID byref;

CHAR cVal;

USHORT uiVal;

ULONG ulVal;

INT intVal;

UINT uintVal;

DECIMAL __RPC_FAR* pdecVal;

CHAR __RPC_FAR* pcVal;

USHORT __RPC_FAR* puiVal;

ULONG __RPC_FAR* pulVal;

INT __RPC_FAR* pintVal;

UINT __RPC_FAR* puintVal;

struct __tagBRECORD

{

PVOID pvRecord;

IRecordInfo __RPC_FAR* pRecInfo;

IRecordInfo __RPC_FAR* pRecInfo;

} __VARIANT_NAME_4;

} __VARIANT_NAME_3;

} __VARIANT_NAME_2;

DECIMAL decVal;

} __VARIANT_NAME_1;

};

VAR type

typedef unsigned short VARTYPE;

enum VARENUM {

VT_EMPTY = 0,

VT_NULL = 1,

VT_I2 = 2,

VT_I4 = 3,

VT_R4 = 4,

VT_R8 = 5,

VT_CY = 6,

VT_DATE = 7,

VT_BSTR = 8,

VT_DISPATCH = 9,

VT_ERROR = 10,

VT_BOOL = 11,

VT_VARIANT = 12,

VT_UNKNOWN = 13,

VT_UI1 = 17,

};

VT_RESERVED = 0x8000

VT_BYREF = 0x4000

VT_ARRAY = 0x2000

VT_FUNC = 0x4c

SafeArray struct

typedef struct tagSAFEARRAY

{

USHORT cDims;

USHORT fFeatures;

ULONG cbElements;

ULONG cLocks;

PVOID pvData;

SAFEARRAYBOUND rgsabound[ 1 ];

} SAFEARRAY;

typedef struct tagSAFEARRAYBOUND

{

ULONG cElements;

LONG lLbound;

} SAFEARRAYBOUND;
```

We know there is only one `VAR` type, so it’s easy for us to do the type confusion.

我们清楚`VAR`类型只有一种，所以进行类型混淆(type confusion)是很安全的.

## Then I will show a demo poc code from promised\_lu.

下面我再展示一点从promised\_lu那里A的POC(Proof of Concept,概念验证):
