
```
while (seg)

{

    nextSeg = seg->next;

    // If seg.length == 0, it is possible that (seg.left + seg.length == prev.left + prev.length),

    // resulting in 2 segments sharing the same "left".

    if (seg->length > 0)

    {

        if (isIntArray)

        {

            ((SparseArraySegment<int32>*)seg)->ReverseSegment(recycler);

        }

        else if (isFloatArray)

        {

            ((SparseArraySegment<double>*)seg)->ReverseSegment(recycler);

        }

        else

        {

            ((SparseArraySegment<Var>*)seg)->ReverseSegment(recycler);

        }



        seg->left = ((uint32)length) - (seg->left + seg->length);

        seg->next = prevSeg;

        // Make sure size doesn't overlap with next segment.

        // An easy fix is to just truncate the size...

        seg->EnsureSizeInBound();



        // If the last segment is a leaf, then we may be losing our last scanned pointer to its previous

        // segment. Hold onto it with pinPrevSeg until we reallocate below.

        pinPrevSeg = prevSeg;

        prevSeg = seg;

    }

    seg = nextSeg;

}
```

其中`ReverseSegment`函数负责真正的置换操作，根据数组元素的类型执行对应的模版函数。

```
template<typename T>

void SparseArraySegment<T>::ReverseSegment(Recycler *recycler)

{

    if (length <= 1)

    {

        return;

    }



    T temp;

    uint32 lower = 0;

    uint32 upper = length - 1;

    while (lower < upper)

    {

        temp = elements[lower];

        elements[lower] = elements[upper];

        elements[upper] = temp;

        ++lower;

        --upper;

    }

}
```

```
seg->left = ((uint32)length) - (seg->left + seg->length);
```

其中`length`是由上层函数传递的，`seg->length`是从对象中获取的，调试发现这两个值有所不同：`length`为1000，而`seg->length`为1002，这里相减产生负值导致crash发生。

## 关于 getter

根据POC猜测这里对象中的`length`改变是由于对`a`数组设置的getter：

```
var o = {};

Object.defineProperty(o, '1', {

    get: function() {

      a.length = 1002;

      j.fill.call(a, 7.7);

      return 2;

    }

  });

a.__proto__ = o;
```

根据调试发现`ForEachOwnMissingArrayIndexOfObject`中调用`ES5Array::GetItem`触发了`Getter`，导致array的segment发生变化：

```
if (descriptor->Getter)

{

    RecyclableObject* func = RecyclableObject::FromVar(descriptor->Getter);

    *value = Js::JavascriptOperators::CallGetter(func, originalInstance, requestContext);

}
```

那么getter中的`j.fill.call(a, 7.7)`存在的意义是什么呢？如果只是把`a.length`设为1002，虽然array object的`length`会变成1002，但是array对应的segment的`length`和`size`并不会变成1002，因为这块内存并没有实际使用，因此就不会更改segment的长度，从而无法触发漏洞。

至此我们明确了漏洞产生的原因在于segment的长度可以在函数执行过程中被改变（比如通过getter）。

## 小结

```
seg->left = ((uint32)length) - (seg->left + seg->length);
```

这里的`seg->left`是直接使用上层传递的参数`length`进行运算的，因此`length`是固定的，但是`seg->length`是可以通过user callback控制的，从而使得用户可以控制`seg`值。

此外还有一个版本的POC是通过`Proxy`劫持`getPrototypeOf`实现的user callback篡改segment长度，基本原理一致不再详细分析。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
