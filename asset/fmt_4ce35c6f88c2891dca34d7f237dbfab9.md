
并取全局变量 `ObpStackSequence` 作为序数(用后将其增 1)，调用 `ObpPushRefDerefInfo()`，(此时若 `MmCanThreadFault()` 返回 FALSE，则调用 `ObpDeferPushRefDerefInfo` 在一个 WorkItem 中完成该操作，关键过程相同)

该函数还原成代码如下,具体过程已经写在注释中了：

```
BOOLEAN __stdcall 
ObpPushRefDerefInfo(
    PVOID ObjectHeader,//对象头
    BOOLEAN bRefrenceOrDefrence,//当前操作是增加引用还是减少引用
    WORD Count,//当前操作增加或减少的计数
    ULONG CurrentSequence,//一个序数
    POBJECT_REF_TRACE Stacks,//当前调用栈地址信息
    LONG Tag//当前增加或减少引用时所使用的 Tag 参数
    )
{
    WORD Index=0;
    WORD NextPos;
    OBJECT_REF_INFO RefInfo={0};
    POBJECT_REF_STACK_INFO pObjRefStackInfo;
    POBJECT_REF_STACK_INFO RefStackInfo,PreRefStackInfo;
    //判断 ObpTraceFlag 及获取 ObpStackTraceLock 这个锁，过程略过略过
    if ( NT_SUCCESS(ObpGetObjectRefInfo(ObjectHeader, &RefInfo))) //查找 ObpObjectTable 获取该 Object 对应的 RefInfo，此 RefInfo->ObjectHeader 即查找的目标
    {
        CurRefInfo = RefInfo;
        if ( RefInfo )
        {
            Index = ObpGetTraceIndex(Stacks);//该函数通过计算调用栈地址的 Hash 值，将其存入 ObpStackTable 表中，并返回在表中的索引
            if ( Index >= 16381 )  //判断 Index 是否超过了允许的最大值，若超过则认为溢出
            {
                DbgPrintEx(0, 1, "ObpPushRefDerefInfo - ObpStackTable overflow\n");
            }
            else   //若没有超过最大值，正常处理
            {
                NextPos = RefInfo->NextPos;//取当前可用的位置指针
                while ( NexPos }//若有效
                {
                    RefStackInfo=RefInfo.StackInfo[NextPos];//当前要保存栈信息的位置
                    PreRefStackInfo=RefInfo.StackInfo[NextPos-1];//最后一次保存栈信息的位置
                    if ( CurrentSequence >= PreRefStackInfo->Sequence )//上一序数未超过当前值，则认为正常，跳出循环
                        break;
                    //超出的情况处置
                    RefStackInfo->Sequence=PreRefStackInfo->Sequence;
                    RefStackInfo->Index=PreRefStackInfo->Index;
                    RefStackInfo->NumTraces=PreRefStackInfo->NumTraces;
                    RefStackInfo->Tag=PreRefStackInfo->Tag;
                    NextPos -= 1;//上移一个位置
                }
                pObjRefStackInfo=RefInfo.StackInfo[NextPos];//取当前可用的位置
                pObjRefStackInfo->Index = Index | (WORD)(-(bRefrenceOrDefrence != 0) & 0x8000);//保存 Index，并根据是增加引用还是减少引用设置标志位
                pObjRefStackInfo->NumTraces = Count;//保存此次的引用计数
                pObjRefStackInfo->Sequence = NextSequence;
                pObjRefStackInfo->Tag = Tag;
                RefInfo->NextPos+=1;   //NextPos 加 1,指向下一个可用位置
            }
        }
    }
    //释放锁及其它，略
}
```

上述函数执行结束以后，本次对象引用时的调用上下文、引用时的 Tag、引用记数、增加还是减少等信息就被记录到了 `ObpStackTable` 中.

其中 `ObpGetTraceIndex()` 计算调用地址 Hash 的算法如下:

```
ULONG GetStackInfoHash(PVOID Addrs)
{
    ULONG Index=0;
    ULONG Value=0;
    ULONG Hash;
    PUSHORT Key = (PUSHORT)Addrs;
    for (Index = 0; Index < 0x40 / sizeof(*Key); Index += 2) {

        Value += Key[Index] ^ Key[Index + 1];
    }
    Hash = ((ULONG)Value) % OBTRACE_STACKS;
    return Hash;
}
```

### 三、如何查看跟踪记录的结果？

根据 MS 的说明，Windbg 提供了命令 `!obtrace` 来查看某一对象的增加引用(Refrence)/减少引用(Derefrence)跟踪信息。

具体用法: `!obtrace [对象地址]`

然后 Windbg 将显示出所有对该对象的增加引用(Refrence)/减少引用(Derefrence)操作的记录。在这个记录中，可以清楚看到每次调用的调用栈，便于查找出问题的位置，同时还有 Tag、计数等信息。可参考 [`msdn.microsoft.com/en-us/libr...(v=VS.85).aspx`](http://msdn.microsoft.com/en-us/library/ff564594\(v=VS.85\).aspx)

下面我们来手工实现一下这个过程，同时也是从反方向加深对上述存储过程及相关结构的理解。

命令: `!obtrace 0x862551b8`

#### (1)在 `ObpObjectTable` 表中查找该对象

`ObpObjectTable` 的地址为 `0x84daf548`

Hash 算法是：

```
#define GetObjHash(Object) ((((ULONG)Object>>4)&0xFFFFF)% 401)
```

`Object=0x862551b8`

计算 Hash, `GetObjHash(Object)=0x4E,`

`Offset=Hash*4=0x138;`

所以位于 `ObpObjectTable+0x138` 处

```
kd>dd ObpObjectTable+130

84daf678  00000000 00000000 8626a000 00000000

84daf688  00000000 00000000 00000000 00000000
```

可以看到，偏移 `0x138` 处的值为 `8626a000`，这是一个 `_OBJECT_REF_INFO` 结构的指针。

```
kd> dt _OBJECT_REF_INFO  8626a000 

nt!_OBJECT_REF_INFO 

   +0x000 ObjectHeader     : 0x862551a0 _OBJECT_HEADER 

   +0x004 NextRef          : (null) //用于链接 Hash 值相同的 Object 

   +0x008 ImageFileName    : [16]  "svchost.exe" 

   +0x018 NextPos          : 0x243//本结构中 StackInfo 数组的索引，表明下一个可用的位置，达到 MaxStacks 时会自动申请更大的空间 

   +0x01a MaxStacks        : 0x3fc//当前最大可用 StackIndex 

   +0x01c StackInfo        : [0] _OBJECT_REF_STACK_INFO//_OBJECT_REF_STACK_INFO 数组，偏移为 0x1C 
```

接下来查看本进程的第一次引用跟踪记录：

```
kd> dt _OBJECT_REF_STACK_INFO 8626a000+1C//第一个 StackInfo

nt!_OBJECT_REF_STACK_INFO 

   +0x000 Sequence         : 0x230//序列号,即 ObpPushRefDerefInfo 函数中的 Sequence 参数，只是一个全局的序数 

   +0x004 Index            : 0x8000//在 ObpStackTable 中的索引，需要去掉 0x8000 标志位 

   +0x006 NumTraces        : 1//此次引用的计数 

   +0x008 Tag              : 0x746c6644//该次引用的 Tag 
```

这里得到一些有用的信息，`Sequence` 为 `0x230`，而 `Index` 为 0(需要去掉 `0x8000` 标志位，有这个标志表明本次操作是增加计数的，否则是减少计数的，这个可以在 `ObpPushRefDerefInfo()` 的代码中得到印证)，`Tag` 为 `0x746c6644`，即 Dflt，这是内核中相关函数使用的默认 Tag。这里面的 `Index` 是个很关键的值，它就是实际存储的调用栈信息在 `ObpStackTable` 中的索引。

`ObpStackTable` 是下面这样一个结构(逆向得来):

```
typedef struct _OBJECT_TRACE_STACK_TABLE

{

    WORD CurUsedStacksCnt;//当前已经使用的数目，从 0 开始递增,直至 1024 时对表项进行扩充

    WORD TotalStacksCnt;//当前可用栈的总数目

    POBJECT_REF_TRACE StackBuckets[16];//总共 16 个表项,每个表的大小为 0x10000，共包括 0x10000/0x40=0x400=1024 个项目

    WORD IndexTable[16382];//最大的 Index 为 16381，最后一项为 0,其余初始化为 0xFFFF，该值与_OBJECT_REF_STACK_INFO 结构中的 Index 值对应

}OBJECT_TRACE_STACK_TABLE,*POBJECT_TRACE_STACK_TABLE;//TotalSize=0x8040
```

实际观察如下：

```
kd> dd 84e35000  //ObpStackTable
84e35000  0400009f 84e3e000 00000000 00000000
84e35010  00000000 00000000 00000000 00000000
84e35020  00000000 00000000 00000000 00000000
84e35030  00000000 00000000 00000000 00000000
84e35040  00000000 ffffffff ffffffff ffffffff
84e35050  ffffffff ffffffff ffffffff ffffffff
84e35060  ffffffff ffffffff ffffffff ffffffff
84e35070  ffffffff ffffffff ffffffff ffffffff
```

可以看到，总共有 16 张表来存储调用栈地址，每个调用栈结构的大小为回溯深度 `16*sizeof(PVOID)=0x40`，而每张表的大小为 `0x10000`，所以每张表可放置 `0x400` 项内容。当超出 `0x400` 时，会申请第二张表，整体结构和三层句柄表有些相似，也是可扩充的。因为我们刚才得到的索引是 0，所以显然是位于第一张表里，并且是第一项;如果索引是 `0x403`，那么将位于第二张表里，位于第四项。

```
kd> dd 84e3e000+0*0x40
84e3e000  83c25eff 83c70c43 83c582fa 83a4542a 
84e3e010  00000000 00000000 00000000 00000000
84e3e020  00000000 00000000 00000000 00000000
84e3e030  00000000 00000000 00000000 00000000
```

可以看到，调用栈的返回地址已经在这里躺着了，共有 4 个有效的地址，看一看第一个地址:

```
kd> u 83c25eff

nt!ObCreateObject+0x1c4:

83c25eff 8b4528           mov     eax,dword ptr [ebp+28h]
```

这和我们用 `!obtrace` 看到的结果是一样的：

```
     230    +1     Dflt       nt!ObCreateObject+1c4 
                                nt!PspAllocateProcess+e0 
                                nt!NtCreateUserProcess+51a 
                                nt!KiFastCallEntry+12a
```

这样，就对整个实现过程有了一个清晰的认识。

关于 Win7 的对象引用跟踪机制，分析到此结束，部分内容参考自 Wrk,其它为逆向所得。

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || ObTrace.rar |
