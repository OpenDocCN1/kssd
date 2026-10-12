
# (呵呵。。我温习功课)通过 IDA 找到 VC++ PE 的 AFX_MSGMAP_ENTRY 数组

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-91.htm`](https://www.pediy.com/kssd/pediy08/pediy8-91.htm)

使用工具：
IDA4.04、ExeScope、LoadPE、VC++6.0

## 一、准备知识：消息映射声明的解释

成员变量有两个：
1. 第一个是 `_messageEntries`
2. 第二个是 `messageMap`

### 第一个成员变量的声明

`AFX_MSGMAP_ENTRY _messageEntries[]`

这是一个 `AFX_MSGMAP_ENTRY` 类型的数组变量，是一个静态成员变量，用来容纳类的消息映射条目。一个消息映射条目可以用 `AFX_MSGMAP_ENTRY` 结构来描述。

`AFX_MSGMAP_ENTRY` 结构的定义如下：

```cpp
struct AFX_MSGMAP_ENTRY
{
    //Windows 消息 ID
    UINT nMessage;
    //控制消息的通知码
    UINT nCode;
    //Windows Control 的 ID
    UINT nID;
    //如果是一定范围的消息被映射，则 nLastID 指定其范围
    UINT nLastID;
    UINT nSig; //消息的动作标识
    //响应消息时应执行的函数(routine to call (or special value))
    AFX_PMSG pfn;
};
```

从上述结构可以看出，每条映射有两部分的内容：第一部分是关于消息 ID 的，包括前四个域；第二部分是关于消息对应的执行函数，包括后两个域。

在上述结构的六个域中，`pfn` 是一个指向 `CCmdTarger` 成员函数的指针。函数指针的类型定义如下：

```cpp
typedef void (AFX_MSG_CALL CCmdTarget::*AFX_PMSG)(void);
```

当使用一条或者多条消息映射条目初始化消息映射数组时，各种不同类型的消息函数都被转换成这样的类型：不接收参数，也不返回参数的类型。因为所有可以有消息映射的类都是从 `CCmdTarge` 派生的，所以可以实现这样的转换。

`nSig` 是一个标识变量，用来标识不同原型的消息处理函数，每一个不同原型的消息处理函数对应一个不同的 `nSig`。在消息分发时，MFC 内部根据 `nSig` 把消息派发给对应的成员函数处理，实际上，就是根据 `nSig` 的值把 `pfn` 还原成相应类型的消息处理函数并执行它。

### 第二个成员变量的声明

`AFX_MSGMAP messageMap;`

这是一个 `AFX_MSGMAP` 类型的静态成员变量，从其类型名称和变量名称可以猜出，它是一个包含了消息映射信息的变量。的确，它把消息映射的信息（消息映射数组）和相关函数打包在一起，也就是说，得到了一个消息处理类的该变量，就得到了它全部的消息映射数据和功能。`AFX_MSGMAP` 结构的定义如下：

```cpp
struct AFX_MSGMAP
{
    //得到基类的消息映射入口地址的数据或者函数
#ifdef _AFXDLL
    //pfnGetBaseMap 指向_GetBaseMessageMap 函数
    const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)();
#else
    //pBaseMap 保存基类消息映射入口_messageEntries 的地址
    const AFX_MSGMAP* pBaseMap;
#endif

    //lpEntries 保存消息映射入口_messageEntries 的地址
    const AFX_MSGMAP_ENTRY* lpEntries;
};
```

从上面的定义可以看出，通过 `messageMap` 可以得到类的消息映射数组 `_messageEntries` 和函数 `_GetBaseMessageMap` 的地址（不使用 MFC DLL 时，是基类消息映射数组的地址）。

成员函数 `_GetBaseMessageMap()` 用来得到基类消息映射的函数。

成员函数 `GetMessageMap()` 用来得到自身消息映射的函数。

## 二、通过 IDA 找到 VC++ PE 的 AFX_MSGMAP_ENTRY 数组

综上所述，我们可以通过 IDA 反汇编 VC++ PE，搜索：`GetRuntimeClass`，会发现几个如下的数据结构：

```text
004DF698                 -------------------------------------------------------
004DF698                 下面是消息映射
004DF698 D0 FB 47 00     off_4DF698 dd offset sub_47FBD0 ; DATA XREF: sub_47FBF0+Co
004DF69C A0 F6 4D 00     dd offset Message_0
004DF6A0 11 BD 00 00     Message_0 dd 0BD11h        ; DATA XREF: .rdata:004DF69Co
004DF6A4 02 00 00 00     nCode_0   dd 2
004DF6A8 00 00 00 00     nID_0     dd 0    控件 ID
004DF6AC 00 00 00 00     nLastID_0 dd 0
004DF6B0 0C 00 00 00     nSig_0    dd 0Ch
004DF6B4 10 FC 47 00     pfn_0     dd 47FC10h  响应函数
以下每 6 个 dword 代表一个
004DF6B8 04 02 00 00     dd 204h                     ; WM_RBUTTONDOWN
004DF6BC 00 00 00 00     dd 0
004DF6C0 00 00 00 00     dd 0
004DF6C4 00 00 00 00     dd 0
004DF6C8 31 00 00 00     dd 31h
004DF6CC 60 FC 47 00     dd 47FC60h   ；WM_RBUTTONDOWN 响应函数
004DF6D0 11 01 00 00     dd 111h
004DF6D4 00 00 00 00     dd 0
004DF6D8 20 80 00 00     dd 8020h
004DF6DC 20 80 00 00     dd 8020h
004DF6E0 0C 00 00 00     dd 0Ch
004DF6E4 70 11 48 00     dd 481170h
004DF6E8 11 01 00 00     dd 111h
004DF6EC 00 00 00 00     dd 0
004DF6F0 21 80 00 00     dd 8021h
004DF6F4 21 80 00 00     dd 8021h
004DF6F8 0C 00 00 00     dd 0Ch
004DF6FC B0 11 48 00     dd 4811B0h
004DF700 11 01 00 00     dd 111h
004DF704 00 00 00 00     dd 0
004DF708 22 80 00 00     dd 8022h
004DF70C 22 80 00 00     dd 8022h
004DF710 0C 00 00 00     dd 0Ch
004DF714 D0 12 48 00     dd 4812D0h
004DF718 11 01 00 00     dd 111h
004DF71C 00 00 00 00     dd 0
004DF720 23 80 00 00     dd 8023h
004DF724 23 80 00 00     dd 8023h
004DF728 0C 00 00 00     dd 0Ch
004DF72C 50 13 48 00     dd 481350h
004DF730 11 01 00 00     dd 111h
004DF734 00 00 00 00     dd 0
004DF738 25 80 00 00     dd 8025h
004DF73C 25 80 00 00     dd 8025h
004DF740 0C 00 00 00     dd 0Ch
004DF744 90 13 48 00     dd 481390h
004DF748 11 01 00 00     dd 111h
004DF74C 00 00 00 00     dd 0
004DF750 26 80 00 00     dd 8026h
004DF754 26 80 00 00     dd 8026h
004DF758 0C 00 00 00     dd 0Ch
004DF75C C0 14 48 00     dd 4814C0h
004DF760 01 02 00 00     dd 201h
004DF764 00 00 00 00     dd 0
004DF768 00 00 00 00     dd 0
004DF76C 00 00 00 00     dd 0
004DF770 31 00 00 00     dd 31h
004DF774 80 FF 47 00     dd 47FF80h
004DF778 00 01 00 00     dd 100h
004DF77C 00 00 00 00     dd 0
004DF780 00 00 00 00     dd 0
004DF784 00 00 00 00     dd 0
004DF788 10 00 00 00     dd 10h
004DF78C 90 15 48 00     dd 481590h
004DF790 00 00 00 00     dd 0   ；6 个全 00 代表结束
004DF794 00 00 00 00     dd 0
004DF798 00 00 00 00     dd 0
004DF79C 00 00 00 00     dd 0
004DF7A0 00 00 00 00     dd 0
004DF7A4 00 00 00 00     dd 0
004DF7A8 BA 8A 4C 00     off_4DF7A8 dd offset j_?GetRuntimeClass@CStatic@@UBEPAUCRuntimeClass@@XZ
                           ; DATA XREF: sub_47FA50+33o
                           ; sub_47FB50+24o
                           ; CStatic::GetRuntimeClass(void)
```

因为各个类可能都有消息循环，所以，可能会找到很多个，但只要仔细查找控件 ID，就很容易找到这个控件的响应代码。不管这个控件是菜单还是按钮，都可以找到。这样就可以对这段响应代码修改或替换了。

同理，知道了本原理，也可以很容易的给某个控件加上一个子控件(如按钮等)。

Spring.W
2006.2
4dd466d0e1e0cc123afbe504da2462ca
