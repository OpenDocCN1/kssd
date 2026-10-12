
# VB DLL 中的函数

> 原文：[`www.pediy.com/kssd/pediy04/kanxue054.htm`](https://www.pediy.com/kssd/pediy04/kanxue054.htm)

VB DLL（主要指 `msvbvm50.dll`、`msvbvm60.dll`）中导出的函数没有正式文档描述，既不知道传递给函数的参数是什么，也不知道函数返回什么值，这对从汇编层次上理解 VB 程序的算法和流程造成了一定的困难。

VB 程序使用的关键数据结构是 `VARIANT`（MSDN 中有描述），这是用作 OLE Automation 的一个通用结构，实际上是一个联合，其中可以存放各种标准的数据类型如 `LONG`、`BYTE`、`FLOAT`、`BSTR` 等。该结构中有一个标志用来指明在某一时刻该结构中具体存放的是哪种数据类型。不同的数据类型之间可以根据需要进行转换。

该结构的定义如下（`#include <oaidl.h>`）。其中 `vt` 就是用来表明实际存放的数据类型的标志。从该结构的偏移 `0x08` 处开始就是实际的数据。一般传给 VB 函数的参数都是某个 `VARIANT` 结构的指针，假定该指针的值放在寄存器 `eax` 中，则用 `"dd eax+8"`（对于值类型）或 `"dd *(eax+8)"`（对于指针类型）就可以看见传给该函数的实际参数值。知道这一点之后再去分析 VB DLL 中的函数相对来说就比较简单一些。

```c
/* VARIANT STRUCTURE
 *
 * VARTYPE vt;         //WORD
 * WORD wReserved1;
 * WORD wReserved2;
 * WORD wReserved3;
 * union {
 *   LONG              VT_I4
 *   BYTE              VT_UI1
 *   SHORT             VT_I2
 *   FLOAT             VT_R4
 *   DOUBLE            VT_R8
 *   VARIANT_BOOL      VT_BOOL
 *   SCODE             VT_ERROR
 *   CY                VT_CY
 *   DATE              VT_DATE
 *   BSTR              VT_BSTR
 *   IUnknown *        VT_UNKNOWN
 *   IDispatch *       VT_DISPATCH
 *   SAFEARRAY *       VT_ARRAY
 *   BYTE *            VT_BYREF|VT_UI1
 *   SHORT *           VT_BYREF|VT_I2
 *   LONG *            VT_BYREF|VT_I4
 *   FLOAT *           VT_BYREF|VT_R4
 *   DOUBLE *          VT_BYREF|VT_R8
 *   VARIANT_BOOL *    VT_BYREF|VT_BOOL
 *   SCODE *           VT_BYREF|VT_ERROR
 *   CY *              VT_BYREF|VT_CY
 *   DATE *            VT_BYREF|VT_DATE
 *   BSTR *            VT_BYREF|VT_BSTR
 *   IUnknown **       VT_BYREF|VT_UNKNOWN
 *   IDispatch **      VT_BYREF|VT_DISPATCH
 *   SAFEARRAY **      VT_BYREF|VT_ARRAY
 *   VARIANT *         VT_BYREF|VT_VARIANT
 *   PVOID             VT_BYREF (Generic ByRef)
 *   CHAR              VT_I1
 *   USHORT            VT_UI2
 *   ULONG             VT_UI4
 *   INT               VT_INT
 *   UINT              VT_UINT
 *   DECIMAL *         VT_BYREF|VT_DECIMAL
 *   CHAR *            VT_BYREF|VT_I1
 *   USHORT *          VT_BYREF|VT_UI2
 *   ULONG *           VT_BYREF|VT_UI4
 *   INT *             VT_BYREF|VT_INT
 *   UINT *            VT_BYREF|VT_UINT
 * }
*/
```

标准数据类型 `VARTYPE` 的定义（`#include <wtypes.h>`）：

```c
typedef unsigned short VARTYPE;

/*
 * VARENUM usage key,
 *
 * [V] - may appear in a VARIANT
 * [T] - may appear in a TYPEDESC
 * [P] - may appear in an OLE property set
 * [S] - may appear in a Safe Array
 *
 *
 * VT_EMPTY              [V] [P]   nothing
 * VT_NULL               [V] [P]   SQL style Null
 * VT_I2                 [V][T][P][S] 2 byte signed int
 * VT_I4                 [V][T][P][S] 4 byte signed int
 * VT_R4                 [V][T][P][S] 4 byte real
 * VT_R8                 [V][T][P][S] 8 byte real
 * VT_CY                 [V][T][P][S] currency
 * VT_DATE               [V][T][P][S] date
 * VT_BSTR               [V][T][P][S] OLE Automation string
 * VT_DISPATCH           [V][T][P][S] IDispatch *
 * VT_ERROR              [V][T][P][S] SCODE
 * VT_BOOL               [V][T][P][S] True=-1, False=0
 * VT_VARIANT            [V][T][P][S] VARIANT *
 * VT_UNKNOWN            [V][T] [S] IUnknown *
 * VT_DECIMAL            [V][T] [S] 16 byte fixed point
 * VT_RECORD             [V] [P][S] user defined type
 * VT_I1                 [V][T][P][s] signed char
 * VT_UI1                [V][T][P][S] unsigned char
 * VT_UI2                [V][T][P][S] unsigned short
 * VT_UI4                [V][T][P][S] unsigned short
 * VT_I8                 [T][P]    signed 64-bit int
 * VT_UI8                [T][P]    unsigned 64-bit int
 * VT_INT                [V][T][P][S] signed machine int
 * VT_UINT               [V][T] [S] unsigned machine int
 * VT_VOID               [T]       C style void
 * VT_HRESULT            [T]       Standard return type
 * VT_PTR                [T]       pointer type
 * VT_SAFEARRAY          [T]       (use VT_ARRAY in VARIANT)
 * VT_CARRAY             [T]       C style array
 * VT_USERDEFINED        [T]       user defined type
 * VT_LPSTR              [T][P]    null terminated string
 * VT_LPWSTR             [T][P]    wide null terminated string
 * VT_FILETIME           [P]       FILETIME
 * VT_BLOB               [P]       Length prefixed bytes
 * VT_STREAM             [P]       Name of the stream follows
 * VT_STORAGE            [P]       Name of the storage follows
 * VT_STREAMED_OBJECT    [P]       Stream contains an object
 * VT_STORED_OBJECT      [P]       Storage contains an object
 * VT_BLOB_OBJECT        [P]       Blob contains an object
 * VT_CF                 [P]       Clipboard format
 * VT_CLSID              [P]       A Class ID
 * VT_VECTOR             [P]       simple counted array
 * VT_ARRAY              [V]       SAFEARRAY*
 * VT_BYREF              [V]       void* for local use
 * VT_BSTR_BLOB          Reserved for system use
*/

enum VARENUM
    {    VT_EMPTY    = 0,
    VT_NULL    = 1,
    VT_I2    = 2,
    VT_I4    = 3,
    VT_R4    = 4,
    VT_R8    = 5,
    VT_CY    = 6,
    VT_DATE    = 7,
    VT_BSTR    = 8,
    VT_DISPATCH    = 9,
    VT_ERROR    = 10,
    VT_BOOL    = 11,
    VT_VARIANT    = 12,
    VT_UNKNOWN    = 13,
    VT_DECIMAL    = 14,
    VT_I1    = 16,
    VT_UI1    = 17,
    VT_UI2    = 18,
    VT_UI4    = 19,
    VT_I8    = 20,
    VT_UI8    = 21,
    VT_INT    = 22,
    VT_UINT    = 23,
    VT_VOID    = 24,
    VT_HRESULT    = 25,
    VT_PTR    = 26,
    VT_SAFEARRAY    = 27,
    VT_CARRAY    = 28,
    VT_USERDEFINED    = 29,
    VT_LPSTR    = 30,
    VT_LPWSTR    = 31,
    VT_RECORD    = 36,
    VT_FILETIME    = 64,
    VT_BLOB    = 65,
    VT_STREAM    = 66,
    VT_STORAGE    = 67,
    VT_STREAMED_OBJECT    = 68,
    VT_STORED_OBJECT    = 69,
    VT_BLOB_OBJECT    = 70,
    VT_CF    = 71,
    VT_CLSID    = 72,
    VT_BSTR_BLOB    = 0xfff,
    VT_VECTOR    = 0x1000,
    VT_ARRAY    = 0x2000,
    VT_BYREF    = 0x4000,
    VT_RESERVED    = 0x8000,
    VT_ILLEGAL    = 0xffff,
    VT_ILLEGALMASKED    = 0xfff,
    VT_TYPEMASK    = 0xfff
    };
```

VB DLL 还调用了 `oleauto32.dll` 中的部分函数。`oleauto32.dll` 是个通用的 proxy/stub DLL，其每个函数的原型在 `<oleauto.h>` 中定义，并在 MSDN 中有详细描述。这也有助于理解 VB DLL 中的函数的作用。

举例：

```asm
LEA EAX, [EBP-58]
PUSH EAX
CALL [MSVBVM60!__vbaI4Var]
```

执行 `call` 之前敲 `dd eax+8`，得到的值为 `3`；
执行完 `call` 之后，`eax = 3`

从而可知 `__vbaI4Var` 的作用是将一个 `VARIANT` 转换为 `I4`（即一个长整数）。
