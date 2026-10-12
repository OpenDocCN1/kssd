
# masm32 开发 com 组件介绍 [一][二][三]

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-675.htm`](https://www.pediy.com/kssd/pediy09/pediy09-675.htm)
>
> 声明：本贴参考网站：`http://ourworld.compuserve.com/`

## [一] 基础知识篇

组件对象模型（`Com`）在 `windows` 操作系统中应用越来越广泛。`com` 因为大量的技术细节显得很复杂，但是正是这种复杂才使 `com` 组件的调用显得十分简单。`com` 和使用程序采用 `server/client` 架构。下面我们将在后续的两篇中介绍 `com` 组件的编写与调用。

`com` 编程时当前程序开发的热点，各种编程语言都为组件编写提供了很好的支持，但是汇编语言例外，汇编语言开发组件没有优势。但是透过汇编开发的了解，可以使我们了解 `com` 组件的工作原理。好了，闲话少说，开始介绍：）

所有的 `inc` 头文件都要满足如下特点：

1. `masm32` 松散的类型定义约定将继续使用。就是说参数可以被定义为他们的基本类型，代表性的如：`DWORD`
2. 里面不能创建任何的代码，仅仅包含定义信息，头文件里面需要包含代码，则必须定义为宏。
3. 结构体应该参照他们的 `C` 原形来定义。
4. `GUID` 结构定义在 `windows.inc` 文件中，`GUID` 的值应该通过 `textequ` 宏来定义，这样不会直接产生任何代码。
5. 接口定义分为两步：
    1. 一个通用的宏产生一个通用的接口结构。
    2. 使用接口名字来修饰结构自身的方法名字。
    
    这种方式可以有效地避免 `namespace` 冲突，并且方便接口定义结构继承。
6. `COM` 接口函数调用使用 `coinvoke` 宏。

### GUIDS

```
EXAMPLE: 

sIID_IUnknown  TEXTEQU  <{000000000H, 00000H, 00000H, \
                           {0C0H, 000H, 000H, 000H, 000H, 000H, 000H, 046H}} 

可以被用来定义 
IID_IUnknown   GUID    sIID_IUnknown 
```

### 接口

由于 `MASM32` 原形约定松散的类型检查，主要检查编译时函数的参数的个数。因此可以非常简单的定义接口函数，如下所示，用下表值来表示函数参数的个数。

```
comethod1Proto   typedef proto :DWORD 
comethod2Proto   typedef proto :DWORD, :DWORD 
comethod3Proto   typedef proto :DWORD, :DWORD, :DWORD 
comethod4Proto   typedef proto :DWORD, :DWORD, :DWORD, :DWORD 
comethod4Proto   typedef proto :DWORD, :DWORD, :DWORD, :DWORD, :DWORD 
```

函数指针如下：

```
comethod1        typedef ptr comethod1Proto 
comethod2        typedef ptr comethod2Proto 
comethod3        typedef ptr comethod3Proto 
comethod4        typedef ptr comethod4Proto 
comethod5        typedef ptr comethod4Proto
```

#### IUnknown 接口

`IUnknown` 接口是基本接口，其他所有接口都是派生于它。函数原形定义如上，其定义如下：

```
_vtIUnknown MACRO CastName:REQ 
    ; IUnknown methods 
    &CastName&_QueryInterface comethod3  ? 
    &CastName&_AddRef         comethod1  ? 
    &CastName&_Release        comethod1  ? 
ENDM 

IUnknown                  STRUCT 
    _vtIUnknown IUnknown 
IUnknown                  ENDS
```

其展开如下:

```
IUnknown                  STRUCT 
    IUnknown_QueryInterface comethod3    ? 
    IUnknown_AddRef         comethod1    ? 
    IUnknown_Release        comethod1    ? 
IUnknown                  ENDS
```

#### IClassFactory 接口

`IClassFactory` 派生于 `IUnknown`.它的结构开始是 `IUnknown` 的方法, 后面添加了 2 个自己的方法.

```
_vtIClassFactory MACRO CastName:REQ 
    ; IUnknown methods   
    _vtIUnknown CastName 
    ; IClassFactory methods 
    &CastName&_CreateInstance comethod4 ? 
    &CastName&_LockServer     comethod2 ? 
ENDM 

IClassFactory               STRUCT 
    _vtIClassFactory IClassFactory 
IClassFactory               ENDS
```

展开如下:

```
IClassFactory               STRUCT 
    IClassFactory_QueryInterface comethod3 ? 
    IClassFactory_AddRef         comethod1 ? 
    IClassFactory_Release        comethod1 ? 
    IClassFactory_CreateInstance comethod4 ? 
    IClassFactory_LockServer     comethod2 ? 
IClassFactory               ENDS
```

### Coinvoke 宏

```
;---------------------------------------------------------------------
; coinvoke MACRO 
; pInterface    pointer to a specific interface instance
; Interface     the Interface's struct typedef
; Function      which function or method of the interface to perform
; args          all required arguments 
;                   (type, kind and count determined by the function)
;
coinvoke MACRO pInterface:REQ, Interface:REQ, Function:REQ, args:VARARG
    LOCAL istatement, arg
    FOR arg, <args>     ;; run thru args to see if edx is lurking in there
        IFIDNI <&arg>, <edx>
            .ERR <edx is not allowed as a coinvoke parameter>
        ENDIF
    ENDM
    istatement CATSTR <invoke (Interface PTR[edx]).&Interface>,<\_>,<&Function, pInterface>
    IFNB <args>     ;; add the list of parameter arguments if any
        istatement CATSTR istatement, <, >, <&args>  
    ENDIF 
    mov edx, pInterface
    mov edx, [edx]
    istatement
ENDM
;---------------------------------------------------------------------
```

例如：`QueryInterface` 方法调用如下：

```
coinvoke ppv ,IUnknown, QueryInterface, ADDR IID_SomeOtherInterface, 
ADDR ppnew
```

### HRESULTS

任何一个 `com` 接口函数的返回值类型都是一个 `hResult`， 4 个字节长。返回值在 `eax` 寄存器中。可以用这个值来判断函数调用是否成功。

```
.IF !SIGN? 
; function passed 
.ELSE 
; function failed 
.ENDIF
```

接下来，我们定义了宏来简化它:

```
.IF SUCCEEDED   ; TRUE if SIGN bit not set
.IF FAILED      ; TRUE is SIGN bit set
```

### 结论

以上这些是你用汇编开发 `com` 需要用到的，这些适用于 `activex` 的开发。

## [二] 用汇编语言编写 com 组件

组件对象模型 `com` 是以 `win32 dll` 或 `exe` 形式发布的执行代码组成的。`Com` 是由一些对象和对象的接口组成，在 `com` 里，接口提供对象操作的机制。 而接口是由一个或者多个相关的方法、属性、事件组成的。在这里我们开发一个简单的但是功能齐全的一个进程内 `com` 组件（即以 `dll` 形式存在）。

这里假设你已经了解了 `com` 对象模型的基础知识，了解什么是虚表，什么是虚函数表指针。如果你不熟悉这些，建议看看《`com` 本质论》这本书。

我们先来分析下进程内 `com` 服务的组成。由于它是一个 `dll` 形式发布的，其中包括 5 个重要的函数。其中后面的四个是要作为 `dll` 导出函数来导出的。

- `DllMain`: 这是动态链接库德第一个入口函数，它在库被加载的时候被调用，通过这个函数，可以对客户端程序进行检查。
- `DllRegisterServer`: 通过这个函数能够实现组建的自我注册，注册信息作为资源保存在动态链接库中，这个函数能够读取资源，把信息写进注册表，使用 `regsvr32.exe` 注册组建时，实际上是调用了组件输出的这个函数。
- `DllUnregisterServer`: 当一个组件不再使用时，这个组件应该能够提供自我卸载，`regsvr32.exe` 能调用这个函数，实现这一步。
- `DllCanUnloadNow`: `com` 服务中的全局变量用于保存它的状态，客户端可以周期性的调用这个函数，检查组件服务器是否在使用，然后把它卸载。
- `DllGetClassObject`: 这是完成组件输出的函数，这个输出需要 3 个参数，创建组件的 `GUID`，要创建组件接口的 `GUID` 以及创建后指向对象的指针。如果组件对象或者接口不被支持，执行将失败。

到现在为止，我们应该注意到一件事情，就是如果不是因为间接访问，`com` 将什么也不是。实际上，`DllGetClassObject` 函数返回的对象不是我们要寻找的对象，它是类厂对象，一个类厂对象了解如何实例化其他任何的类。第一层的间接访问允许组件创建的细节被指定，如果它仅仅是简单而又直接的返回一个我们要寻找的对象指针，那么说明对象已经存在，那样，我们将不能设置和控制关于构造对象的任何参数。

`DllGetClassObject` 返回一个 `IClassFactory` 接口，这个接口是从 `IUnknown` 派生的，另外他还有自己的两个重要的成员函数。

```
HRESULT CreateInstance(
IUnknown * pUnkOuter,   //Pointer to outer object when part of an 
// aggregate REFIID riid, 
REFIID riid,           //Reference to the interface identifier
oid** ppvObject);      //Address of output variable that receives 
// the interface pointer requested in riid

HRESULT LockServer(BOOL fLock); 
//Increments or decrements the lock count
```

`LockServer` 用来控制类厂对象的引用计数，系统检查改计数以确定是否要卸载组件，即：控制类厂的生存期。

`CreateInstance` 是最重要的，类厂组件的唯一功能是创建其它组件。一个类厂组件可以对应多种普通 `COM` 组件，但每个类厂组件的实例只能创建一种 `COM` 组件。它接收一个接口 `GUID`，返回该接口的指针。它并不接受组件的 `CLSID`，所以一个类厂实例只能够创建一种 `COM` 组件，即传给 `CoGetClassObject` 的 `CLSID` 对应的组件。

客户、`COM` 库、组件 `dll`、类厂、组件之间的交互过程：

1. 客户首先调用 `COM` 库的 `CoCreateInstance` 函数来创建 `COM` 组件。
2. `CoCreateInstance` 首先调用 `COM` 库的 `CoGetClassObject` 获取类厂。
3. 该函数具体是通过调用了组件 `DLL` 输出的 `DllGetClassObject` 来创建类厂。
4. `DllGetClassObject` 通过 `new` 函数产生一个 `Cfactory` 的对象，并通过 `QueryInterface` 获取其接口指针（一般是 `IclassFactory` 指针）。
5. 返回到 `COM` 库的 `CoCreateInstance` 调用刚才获得的接口指针（`IclassFactory`，类厂）的 `CreateInstance` 函数。
6. 该函数 `new` 指定的组件类，通过 `QueryInterface` 获得指定的接口
7. `CoCreateInstanse` 释放掉 `IclassFactory` 指针（通过 `Release`），然后向客户程序返回获得的指针。
8. 可以在客户中使用获得的接口了。

在第 6 步中，根据不同的 `CLSID` 创建不同的组件，可以实现一个类厂供该 `DLL` 中多个组件共用。但只是类共用，不是实例共用。一旦在创建类厂时通过 `CoGetClassObject` 指定了 `CLSID`，则只能创建该 `COM` 组件的实例。

在这里我们将深入 `c++` 对象模型，来看下一些内部的实现细节。通常编译器来处理这些。`com` 的设计者充分利用了这些，因此，我们需要了解它。

当我们用汇编写一个常规的程序时，我依靠编译器为我们创建代码段和数据段，内存中的一块区域是我们执行的代码，另一块区域保存了我们需要的数据。

`C++` 运行时动态内存分配，给每一个类实例，每一个小的代码段它自己的数据段。换句话讲，一个类的实例就是这个数据段，每一个类实例的数据描述都是保存在一个动态的数据区域。

或许你听说过 `c++` 传递对象成员函数参数时，有一个隐藏的参数，即 `this` 指针。当一个人为对象写一个低层的代码时（在 `c++` 中编译器会作这个工作，你不需要考虑），你首先遇到的问题是”我在给哪个对象写代码？“



`this` 指针是一个简单的指针，它指向这个动态数据内存区域的这个类对象实例。当一个类对象函数被调用时，`this` 指针就会被悄悄地传递过去。当这个对象的私有数据被访问时，类的代码区域就会使用 `this` 指针，来找到它的对象实例的数据。

对于一个 `com` 接口指针跟 `this` 指针很类似。使用中，`com` 是一个接口规范，让你看不到它的代码实现。

```
; declare the ClassFactory object structure

ClassFactoryObject STRUCT
    lpVtbl    DWORD 0 ; function table pointer
    nRefCount DWORD 0 ; object and reference count
ClassFactoryObject ENDS

; declare the MyCom object structure

MyComObject STRUCT
    lpVtbl    DWORD 0 ; function table pointer
    nRefCount DWORD 0 ; reference count
    nValue    DWORD 0 ; interface private data
MyComObject ENDS
```

第一个 `lpVtbl` 是一个虚表指针，它指向一个虚函数表，我用它来控制每个接口的私有数据。就像这里的 `nRefCount` 和 `nValue`。

这些结构所在的动态内存是通过 `CoTaskMemAlloc` 和 `CoTaskMemFree` 这两个 API 函数来分配和释放的。这两个函数是由 `ole32.dll` 导出的。`Ole32.dll` 还导出了很多的函数，例如比对 `GUIDs` 值和把转换 `GUIDs` 为字符串，或者把字符串转换为 `GUIDs`。

为了举例说明 `com` 接口的工作原理，我们创建一个简单接口 `IMyCom`（注：所有的 `com` 接口都有一个“I”前缀。同其他接口一样，他派生于 `IUnknown` 接口，也就是说他的前三个函数是 `QueryInterface`, `AddRef`, 和 `Release`。下面我们添加几个接口函数。下面看到的是 c 风格的函数原形：

```
HRESULT SetValue(long *pVal);
HRESULT GetValue(long newVal);
HRESULT RaiseValue(long newVal);
```

其中，`SetValue` 和 `GetValue` 用于读，设置我们接口的数据成员。`RaiseValue` 用于增加这个数据的值。

这个结构在内存中的形式如下：

客户端仅仅拥有一个分布式结构的指针（`ppv`）这个名字来源于它的 c++形式的定义（"pointer to pointer to (void).")，当创建类实例的时候，这个对象数据块是动态分配和初始化的，虚函数表 `vtable` 和 `server functions` 是静态的，他们在编译时定义好。

有一点需要注意的是，虚函数表拥有的是函数指针，而并非是函数本身。因此，我们可以修改虚函数表中指向的例程，就可以简单的"override“一个派生函数。

在例子中，`IClassFactory` 和 `IMyCom` 都是派生于 `IUnknown` 接口，都继承了 `QueryInterface`，但是他们支持不同的接口，它们需要指向不同的例程，返回不同的结果。

因此，它们有各自的 `QueryInterface` 例程(`QueryInterfaceCF` 和 `QueryInterfaceMC`)被不同的虚函数表指向。

同样的，`AddRef` 和 `Release` 也要被不同的支持他们的接口来定制。

### 类型库

每一个 `com` 接口都是从系统注册表中得到信息，这些接口的定义都是由一个被称为接口定义语言（IDL）来描述的，在 windows 平台下，使用 `MIDL` 进行编译。我们可以利用 vc 开发环境，通过向导来创建一个原始的接口定义文件。

我建一个 `ATL` 工程，命名为 `MyComApp`，然后选择“insert a new ATL object“，然后选择“Simple Object”，命名为：`MyCom`。这样就创建了一个空的 `IMyCom` 接口，然后通过右键菜单，我们添加属性 `SetValue` 和 `GetValue`，并增加一个 `RaiseValue` 方法。然后我们保存退出工程，拷贝 `MyComApp.idl` 文件到我的汇编工程目录。

下面就是这个 idl 文件的内容：

```
// MyCom.idl : IDL source for MyCom.dll

//
// This file will be processed by the MIDL tool to
// produce the type library (MyCom.tlb) and marshalling code

import "oaidl.idl";
import "ocidl.idl";

[
    object,
    uuid(F8CE5E41-1135-11d4-A324-0040F6D487D9),
    helpstring("IMyCom Interface"),
    pointer_default(unique)
]
interface IMyCom : IUnknown
{
    [propget, helpstring("property Value")]
    HRESULT Value([out, retval] long *pVal);
    [propput, helpstring("property Value")]
    HRESULT Value([in] long newVal);
    [helpstring("method Raise")]
    HRESULT Raise(long Value);
};

[
    uuid(F8CE5E42-1135-11d4-A324-0040F6D487D9),
    version(1.0),
    helpstring("MyComApp 1.0 Type Library")
]
library MyComLib
{
    importlib("stdole32.tlb");
    importlib("stdole2.tlb");
    [
        uuid(F8CE5E43-1135-11d4-A324-0040F6D487D9),
        helpstring("MyCom Class")
    ]
    coclass MyCom
    {
        [default] interface IMyCom;
    };
};
```

这个文件可以被用来作为原型进一步进行接口定义。注意这里面有三个 `GUIDs`，一个是为接口，一个是为 `coclass`，一个是为类型库。对于新的应用，它们的值一定不同。

透过这个定义的文件结构，我们很容易了解他的内容。

```
[propget, helpstring("property Value")] HRESULT Value([out, retval] long *pVal);
[propput, helpstring("property Value")] HRESULT Value([in] long newVal);
[helpstring("method Raise")] HRESULT Raise(long Value);
```

下面是这些接口在 `masm32` 中的定义:

```
GetValue PROTO :DWORD, :DWORD
SetValue PROTO :DWORD, :DWORD
RaiseValue PROTO :DWORD, :DWORD
```

他们有很大的不同，但是原因很简单。类型库中的接口是作为通用的，可以直接被客户端象 `VB` 来使用。

为了创建类型库，可以使用 `MIDL` 命令行来编译 idl 文件：

```
MIDL MyCom.idl
```

编译产生的几个文件，除了 `MyCom.tlb` 外，其他的都可以忽略，接下来我们需要把类型库添加到 dll 资源文件中。例如：

```
1 typelib MyCom.tlb
```

让他作为资源文件中的第一个元素是很重要的，后续我们将会使用 `LoadTypeLib` API 函数来使用这个库，同时这个函数也是希望在第一位置发现这个库。

### 注册组件

`DllRegisterServer` 和 `DllUnregisterServer` 为我们注册组件和注销组件用.内容如下：

```
HKEY_CLASSES_ROOT\CMyCom
    (Default) "CMyCom simple client"
HKEY_CLASSES_ROOT\CMyCom\CLSID
    (Default) "{A21A8C43-1266-11D4-A324-0040F6D487D9}"

HKEY_CLASSES_ROOT\CLSID\{A21A8C43-1266-11D4-A324-0040F6D487D9}
    (Default) "CMyCom simple client"
HKEY_CLASSES_ROOT\CLSID\{A21A8C43-1266-11D4-A324-0040F6D487D9}\CMyCom
    (Default) "CMyCom"
HKEY_CLASSES_ROOT\CLSID\{A21A8C43-1266-11D4-A324-0040F6D487D9}\InprocServer32
    (Default) "C:\MASM32\MYCOM\MYCOM.DLL"
    ThreadingModel "Single"

HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}
    (Default) (value not set)
HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}\1.0
    (Default) "MyCom 1.0 Type Library"
HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}\1.0\0
    (Default) (value not set)
HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}\1.0\0\win32
    (Default) " C:\masm32\COM\MyCom \MYCOM.DLL"
HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}\1.0\FLAGS
    (Default) "O"
HKEY_CLASSES_ROOT\TypeLib\{A21A8C42-1266-11D4-A324-0040F6D487D9}\1.0\HELPDIR
    (Default) "C:\masm32\COM\MyCom"
```

有一个键值是变化的，它是服务 dll 自身的路径和文件名，在我的系统上，我把它放置在 `"C:\MASM32\COM\MYCOM\MYCOM.DLL"`，当我注册组件的时候，这个可以被检测到，`DllRegisterServer` 通过调用 `GetModuleFileName` 可以发现 dll 自身的存储位置。

这里有大量的信息是关于这个 `com` 服务的，但是我们仅仅需要传递`{A21A8C43-1266-11D4-A324-0040F6D487D9}`这个 ID 和一个有效的接口 ID 给 `CoCreateInstance` 函数来实例化我们的 `com` 服务。这个函数将会跟踪注册表设置，利用 `CLSID` 来发现创建组件需要的东西，一旦它创建了组件，它将加载类型库，以获取更多需要的信息。

对我们来说非常幸运，最后的 5 个注册入口项通过 `RegisterTypeLib` 函数可以完成。在 `DllRegisterServer` 中，我们通过一些列的注册表函数来设置前面 5 项键值。然后调用 `RegisterTypeLib`。 `DllUnregisterServer` 函数删除 `DllRegisterServer` 中的注册表项，然后调用 `UnRegisterTypeLib`。注意不要完全删除 `HKEY_CLASSES_ROOT\CLSID\`

### 实现 Unknown

```
AddRef_MC proc this_:DWORD
    mov eax, this_
    inc (MyComObject ptr [eax]).nRefCount
    mov eax, (MyComObject ptr [eax]).nRefCount
    ret         ; note we return the object count
AddRef_MC endp

Release_MC proc this_:DWORD
    mov eax, this_
    dec (MyComObject ptr [eax]).nRefCount
    mov eax, (MyComObject ptr [eax]).nRefCount
    .IF (eax == 0)
        ; the reference count has dropped to zero
        ; no one holds reference to the object
        ; so let's delete it
        invoke CoTaskMemFree, this_
        dec MyCFObject.nRefCount
    xor eax, eax    ; clear eax (count = 0)
    .ENDIF
    ret         ; note we return the object count
Release_MC endp
```

`MyCom` 自己的成员实现：

```
GetValue proc this_:DWORD, pval:DWORD
    mov eax, this_
    mov eax, (MyComObject ptr [eax]).nValue
    mov edx, pval
    mov [edx], eax
    xor eax, eax        ; return S_OK
    ret
GetValue endp

SetValue proc this_:DWORD, val:DWORD
    mov eax, this_
    mov edx, val
    mov (MyComObject ptr [eax]).nValue, edx
    xor eax, eax        ; return S_OK
    ret
SetValue endp

RaiseValue PROC this_:DWORD, val:DWORD
    mov eax, this_
    mov edx, val
    add (MyComObject ptr [eax]).nValue, edx
    xor eax, eax        ; return S_OK
    ret
RaiseValue ENDP
```

`MyCom.dll`，这个 `com` 服务工程需要以下 5 个文件来编译：

*   `MyCom.asm` 汇编源程序
*   `MyCom.idl` IDL 文件，用于编译产生 `MyCom.tlb`
*   `MyCom.tlb` 类型库，需要一个 `rc` 资源文件
*   `rsrc.rc` 资源文件，从中可以获得类型库信息
*   `MyCom.DEF` 标准的 dll 输出文件



编译后，代码不会做任何事情，直到我们注册它，我们可以使用命令行：`regsvr32 MyCom.dll` 注册。

### [三] 用汇编语言访问 com 对象

大量的细节使得 Com 看上去很复杂，但是使用起来却很简单。最难的部分就是理解里面的数据结构，尽管 COM 是语言无关的，但是他借用了很多 `c++`的术语来描述自己。

为了能使用某个对象的 com 接口函数，你必须首先要从类厂中创建这个对象，并且让他来返回接口指针。这个过程被 `CoCreateInstance` 这个 API 函数完成。当你使用完接口时，要调用 `Release` 方法。一个 COM 对象可以看作是一个服务，调用 com 的应用程序就是他的客户端。

在调用 com 接口函数之前，你需要了解接口是什么，一个 com 接口就是一个函数指针表，我们还是从 `IUnknown` 接口开始，如果你创建了一个组件导出了 `IUnknown` 接口，那么你就有了一个全功能的 com 对象。`IUnknown` 有三个基本的接口方法，既然所有的接口都是从它派生出来，那么我们一定要记住，一个接口实际上就是一个函数指针成员组成的结构体。

例如：

```
IUnknown STRUCT DWORD 
; IUnknown methods 
IUnknown_QueryInterface   QueryInterface_Pointer ? 
IUnknown_AddRef     AddRef_Pointer ? 
IUnknown_Release    Release_Pointer ? 
IUnknown ENDS
```

它只有 12 个字节长，它具有 3 个 `DWORD` 指针来指向实际的实现函数。对于虚函数表，你一定听说过，这些指针定义如下，因此我们可以让 `masm` 在编译我们的调用时进行一些类型检查。

```
QueryInterface_Pointer    typedef ptr QueryInterface_Proto
AddRef_Pointer     typedef ptr AddRef_Proto 
Release_Pointer     typedef ptr Release_Proto
```

最后我们定义我们的函数如下：

```
QueryInterface_Proto    typedef PROTO :DWORD, :DWORD, :DWORD 
AddRef_Pointer    typedef PROTO :DWORD 
Release_Pointer     typedef PROTO :DWORD 
```

为了保持 `masm32` 松散的类型检查一致，函数参数都定义为 `dword`

定义接口是一个相当大的编辑就是，`masm` 不支持前向引用。因此，我们不得不颠倒一下定义的顺序。先定义函数头，再定义函数指针，最后定义接口。实际上在使用接口时，你需要一个指向它的指针。

`CoCreateInstance` 函数能用来直接返回一个接口指针。它实际上指向了拥有接口的对象。这个结构看上去如图所示：

这个结构里有大量的间接访问，使用宏可以简化它。

当客户端调用 COM 库创建 com 组件时，它传进了一个地址用于存放对象指针。这个就是我们所说的 `ppv`。 从 `c++`的角度来讲，叫做指向指针的指针，`void` 类型代表无类型。它保存了另一个指针 `pv` 的地址。`pv` 指向了虚函数表。

例如：我们使用 `CoCreateInstance` 函数成功的返回了一个接口指针 `ppv`,我们想看下它是否支持其他的接口，我们可以调用 `QueryInterface` 方法。用 `c++`的方法描述 `QueryInterface` 如下：

```
(HRESULT) SomeObject::QueryInterface (this:pObject, IID:pGUID, ppv2:pInterface)
```

用汇编写法如下： 

```
; get pointer to the object 
mov eax, ppv 
; and use it to find the interface structure
mov edx, [eax] 

; push the function parameters onto the stack
push OFFSET ppv2
push OFFSET IID_ISomeOtherInterface
push dword ppv 

; and then call that method 
call dword ptr [edx + 0]
```

使用 `invoke` 调用简化如下：

```
; get pointer to the object 
mov eax, ppv 
; and use it to find the interface structure
mov edx, [eax] 
; and then call that method 
invoke (IUnknown PTR [edx]).IUnknown_QueryInterface, ppv,
    ADDR IID_SomeOtherInterface, ADDR ppv_new
```

注意 `IUnknown PTR [edx]`这个类型转换，是让编译器知道使用哪个结构来得到 `QueryInterface` 函数在虚表中的正确偏移。其中有一个模糊的地方，注意我修改了函数名字为"`IUnknown_QueryInterface`"，这个名字修饰时必要的。当你有一个大的 com 工程，有许多相似的接口，你就会遇到麻烦。不同的接口对应不同的方法表示，是非常有效的。

`coinvoke` 宏，这个宏定义在 `oaidl.inc` 文件中。使用它，可以进一步简化 com 调用。

```
;---------------------------------------------------------------------
; coinvoke MACRO 
;
;
; pInterface    pointer to a specific interface instance
; Interface     the Interface's struct typedef
; Function      which function or method of the interface to perform
; args          all required arguments 
;                   (type, kind and count determined by the function)
;
coinvoke MACRO pInterface:REQ, Interface:REQ, Function:REQ, args:VARARG
    LOCAL istatement, arg
    FOR arg, <args>     ;; run thru args to see if edx is lurking in there
        IFIDNI <&arg>, <edx>
            .ERR <edx is not allowed as a coinvoke parameter>
        ENDFIN
    ENDM
    istatement CATSTR <invoke (Interface PTR[edx]).&Interface>,<_>,<&Function, pInterface>
    IFNB <args>     ;; add the list of parameter arguments if any
        istatement CATSTR istatement, <, >, <&args> 
    ENDFIN  
    mov edx, pInterface
    mov edx, [edx]
    istatement
ENDM
;---------------------------------------------------------------------
```

因此，前面的 `QueryInterface` 方法调用就可以简化成： 

```
coinvoke ppv ,IUnknown, QueryInterface, ADDR IID_SomeOtherInterface, 
ADDR ppnew
```
注意这里名字修饰是隐藏在宏中处理的。

代码后面附上。。。
