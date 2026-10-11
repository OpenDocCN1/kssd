
# XP/Win7 上无硬编码枚举了各种 NotifyRoutine 和 CmpCallback

> 原文：[`www.pediy.com/kssd/pediy11/112516.html`](https://www.pediy.com/kssd/pediy11/112516.html)

目前大多数做法似乎是从 `PsSetCreateProcessNotifyRoutine` 等导出的地址开始，用特征码搜索保存 `Notify` 的数组来枚举，缺点是不够通用。

我的思路是，先注册一个自己的 `NotifyRoutine`，然后从 `CmUnRegisterCallback` 等导出函数地址开始查找自己登录的函数地址，从而找到保存 `Notify` 的数组。这样做也有缺点，在 `XP` 最多登录 8 个 `processnotify`，如果超过了，我这个方法就没法用了。

`PspLoadImageNotifyRoutine`、`PspCreateThreadNotifyRoutine`、`PspCreateProcessNotifyRoutine` 这三个的找法是一样的，`xp` 和 `win7` 也没什么区别，都是一个数组。

值得一提的是 `CmpCallback` 的找法，`xp` 和 `win7` 有很大的区别，`xp` 里面是个数组，到了 `win7` 变成了队列了。

我的逆向水平很弱，着实花了一段时间，才看出这个队列的结构。

唯一的硬编码是数组的最大个数，比如 `CreateProcessNotify`。

代码：

```
XP:

805d1c59 43             inc     ebx
805d1c5a 83c704         add     edi,4
805d1c5d 83fb08         cmp     ebx,8
805d1c60 72cc           jb      nt!PsSetCreateProcessNotifyRoutine+0x14 (805d1c2e)



win7:

8339a8c6 43             inc     ebx
8339a8c7 83fb40         cmp     ebx,40h
8339a8ca 72af           jb      nt!PspSetCreateProcessNotifyRoutine+0x29 (8339a87b)
```

一个是 8 个，一个是 64 个。

对照了一下，和 `xuetr` 的结果一样：

![](img/a6f2368f23d15f278e53ba239f2f76f8.jpg)
![](img/5f3324b499b6917566771c0ce190772c.jpg)

引用的文字都保存到论坛一份

引用：从 `CmUnRegisterCallback` 进去在 `XP` 有一个 `_CmpCallBackVector` 数组，数目为 100，数组元素为

```
typedef struct _EX_FAST_REF
{
    union
    {
        PVOID Object;
        ULONG_PTR RefCnt:3;
        ULONG_PTR Value;
    };
} EX_FAST_REF, *PEX_FAST_REF;
```

其中的 `Object` 指向的结构为

```
typedef struct _EX_CALLBACK_ROUTINE_BLOCK
{
    EX_RUNDOWN_REF RundownProtect;
    PEX_CALLBACK_FUNCTION Function;
    PVOID Context;//
} EX_CALLBACK_ROUTINE_BLOCK, *PEX_CALLBACK_ROUTINE_BLOCK;
```

而 `Context` 结构指向

```
typedef struct _CM_ARRY_XP
{
LARGE_INTEGER Cookie;//
LIST_ENTRY       ListEntry1;//8
ULONG UnKnown1;//16
ULONG UnKnown2;//20
ULONG UnKnown3;//24
ULONG UnKnown4;//28
ULONG UnKnown5;//32
LIST_ENTRY ListEntry2;//36
ULONG   UnKnown;//44
ULONG   Context;//48
ULONG   UnKnown6;
}CM_CONTEXT,*PCM_CONTEXT;
```

取得 `Cookie` 就可以实现卸载了……在 `XP` 这个 `Cookie` 居然就是注册时的系统时间……

上面那种方式很明显寻址效率超低，所以 `Windows7` 下就做了改进了，没有使用数组而是一个链表：

`_CallbackListHead`，这个链表连接在下面结构的第一项里（就是 `ListEntryHead`）：

```
typedef struct _CM_NOTIFY_ENTRY
{
LIST_ENTRY ListEntryHead;///初始化时指向自己
ULONG   UnKnown1;//+8
ULONG   UnKnown2;//12
LARGE_INTEGER Cookie;//16
ULONG   Context;//+24
ULONG   Function;//+28
ULONG   Altitudes;//32
PVOID     BufferPointer;//36
LIST_ENTRY ListEntry1;//40 初始化时指向自己,没搞清楚到底指哪去
}CM_NOTIFY_ENTRY,*PCM_NOTIFY_ENTRY;
```

要卸载只需要遍历这个双向链表，取得 `Cookie` 就可以了……`Windows7` 下那个 `Cookie` 值好像是去一个全局变量里取，没搞懂是在哪里设置的
