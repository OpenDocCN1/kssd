
# 让我们一起学 MetaData API（2）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-711.htm`](https://www.pediy.com/kssd/pediy08/pediy8-711.htm)

让我们一起学 MetaData API（2）

上次简单地介绍了在 `win32` 下使用 MetaData API 的方法，本章在深入应用它的各种功能前，先介绍在 `C#` 下运用的方法。因为 `C#` 用于 .net 编程实在是太方便了！本章的示例代码来自开源软件 `DILE`（dotnet il editor），下载地址 `http://dile.sourceforge.net`。

## 1、 在 C# 中引入 COM

`C#` 中使用 COM 要用到 .net 与 COM 的 Interop，所有的内容全部都是公开的，所以直接看代码。一般，需要引用什么接口，就建立一个相应的文件，比如要引用 `IMetaDataDispenserEx` 接口，我们就在工程中新建一个 `IMetaDataDispenserEx.cs`，代码如下（代码引用自我写的 `injectReflector`）：

```
namespace injectReflector.com
{
    [ComImport, GuidAttribute("31BCFCE2-DAFB-11D2-9F81-00C04F79A0A3"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
    public interface IMetaDataDispenserEx
    {
        uint DefineScope(ref Guid rclsid, uint dwCreateFlags, ref Guid riid, [MarshalAs(UnmanagedType.Interface)]out object ppIUnk);
        uint OpenScope([MarshalAs(UnmanagedType.LPWStr)]string szScope, uint dwOpenFlags, ref Guid riid, [MarshalAs(UnmanagedType.Interface)] out object ppIUnk);
        uint OpenScopeOnMemory(IntPtr pData, uint cbData, uint dwOpenFlags, ref Guid riid, [MarshalAs(UnmanagedType.Interface)]out object ppIUnk);
        uint SetOption(ref Guid optionid, [MarshalAs(UnmanagedType.Struct)]object value);
        uint GetOption(ref Guid optionid, [MarshalAs(UnmanagedType.Struct)]out object pvalue);
        uint OpenScopeOnITypeInfo([MarshalAs(UnmanagedType.Interface)]ITypeInfo pITI, uint dwOpenFlags, ref Guid riid, [MarshalAs(UnmanagedType.Interface)]out object ppIUnk);
        uint GetCORSystemDirectory([MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 0)]char[] szBuffer, uint cchBuffer, out uint pchBuffer);
        uint FindAssembly([MarshalAs(UnmanagedType.LPWStr)]string szAppBase, [MarshalAs(UnmanagedType.LPWStr)]string szPrivateBin, [MarshalAs(UnmanagedType.LPWStr)]string szGlobalBin, [MarshalAs(UnmanagedType.LPWStr)]string szAssemblyName, [MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 4)]char[] szName, uint cchName, out uint pcName);
        uint FindAssemblyModule([MarshalAs(UnmanagedType.LPWStr)]string szAppBase, [MarshalAs(UnmanagedType.LPWStr)]string szPrivateBin, [MarshalAs(UnmanagedType.LPWStr)]string szGlobalBin, [MarshalAs(UnmanagedType.LPWStr)]string szAssemblyName, [MarshalAs(UnmanagedType.LPWStr)]string szModuleName, [MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 5)]char[] szName, uint cchName, out uint pcName);
    }
}
```

关键的几处，一是 `ComImport` 属性中的定义，包括 `GUID` 值和接口类型（`IUnknown`），这样我们的接口就定义 OK 了。随后就是在接口中定义 COM 中的所有函数，由于是 `Interface`，所以这里的定义中不包含任何代码。

## 2、 C# 中使用本地数据类型

接口函数的定义中用到了许多 `MarshalAs`，这是 `C#` 与本地数据类型的转换。《COM 本质论》中给出了一张表，列举了所有的需要转换和 `C#` 自动转换的数据类型。看下代码中用到的：

无需转换：`uint`、`Guid`

需要转换：

`object` -> `[MarshalAs(UnmanagedType.Interface)]`
`object` -> `[MarshalAs(UnmanagedType.Struct)]`
`string` -> `[MarshalAs(UnmanagedType.LPWStr)]`
`char[]` -> `[MarshalAs(UnmanagedType.LPArray, SizeParamIndex = 5 或 4)]`

其中 `string` 的转换也许是最常用的，特别是在调用本地 dll 时。指针倒反而简单了，`IntPtr` 直接使用。

## 3、 使用 Interface

定义完接口后，在代码中如何使用呢？首先要定义这些接口的变量：

```
public static IMetaDataDispenserEx dispenser;
public static IMetaDataImport import;
```

随后就可以直接使用接口的方法。看下示例，同样来自 `injectReflector` 中：

```
public static string GetMethodDefName(uint token)
{
    string szRet;
    uint bclass=0;
    char[] szName=new char[MAX_NAME_LENGTH];
    uint ccount;
    uint pAttr;
    IntPtr pBlob;
    uint bcount;
    uint pRVA;
    uint flags;
    import.GetMethodProps(token, out bclass, szName, (uint)MAX_NAME_LENGTH, out ccount, out pAttr, out pBlob, out bcount, out pRVA, out flags);
    if(ccount==0)
    {
        szRet = token.ToString();
    }
    else
    {
        szRet = new string(szName, 0, Convert.ToInt32(ccount) - 1);
    }
    if(bclass!=0)
    {
        szRet = GetTypeDefRefName(bclass) + "." + szRet;
    }
    return szRet;
}
```

这段代码是根据一个 token 取得该 token 所对应的方法的名称。值得注意的仍然是类型的转换，看下 `IMetaDataImport::GetMethodProps` 的原型定义：

```
HRESULT GetMethodProps (
    mdMethodDef         mb,
    mdTypeDef           *pClass,
    LPWSTR              szMethod,
    ULONG               cchMethod,
    ULONG               *pchMethod,
    DWORD               *pdwAttr,
    PCCOR_SIGNATURE     *ppvSigBlob,
    ULONG               *pcbSigBlob,
    ULONG               *pulCodeRVA,
    DWORD               *pdwImplFlags
);
```

对照代码可以看到 `uint` 代替了大多数定义，包括 `mdMethodDef`、`mdTypeDef`、`ulong`、`dword`，还有两个转换是 `LPWSTR` 就直接传递 `char[]`，指针 `PCCOR_SIGNATURE` 直接用 `IntPtr` 代替。所有的 MetaData 接口用到的类型基本上就这么多了。

最后，需要使用哪个接口的某个方法，就直接用 `C#` 的语法进行调用：

```
import.GetMethodProps
import.GetFieldProps
import.GetTypeSpecFromToken
```

## 4、 小结

完整的代码示例请参照开始给出的 `DILE`。本文没有什么难度，只是介绍下 `C#` 中使用 MetaData COM 接口的方法，但是 MetaData API 的威力确是无穷的。以后有时间再介绍吧。
