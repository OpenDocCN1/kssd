
# [翻译] Firefox 中一个 Cross-mmap 溢出的利用（CVE-2016-9066）

*2017-3-10*

这篇文章将会探索一下 [CVE-2016-9066](https://bbs.kanxue.com/elink@d0cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0r3j5h3c8$3K9i4y4G2M7X3W2W2M7#2)9J5c8X3#2X3M7$3p5J5x3o6p5$3i4K6u0V1z5o6W2Q4x3V1k6Q4x3U0y4o6g2V1g2Q4x3X3b7J5x3o6p5$3i4K6u0V1z5e0l9$3y4R3`.`.)，这是 Firefox 中一个简单却有趣（从实际操作的角度来看）的漏洞，可以利用该漏洞来获取代码执行权限。

一段负责加载脚本标签的代码中的一个整数溢出导致了对 `mmap` 结束块的越界写操作。一种利用方法是在缓冲区后放置一个 JavaScript 堆以便便溢出到它的元数据中来创建一个假的未使用堆单元。然后可以将一个 `ArrayBuffer` 实例放置在另一个 `ArrayBuffer` 的内联数据中。`ArrayBuffer` 对象内部可以被任意修改，就产生了任意读/写原语。至此，实现代码执行就变的非常简单了。完整的 exploit 可以在 [此处](https://bbs.kanxue.com/elink@d2cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3g2D9L8#2)9J5c8X3k6G2P5s2m8%4L8R3`.`.) 找到，并针对 macOS 10.11.6 平台上的 Firefox 48.0.1 进行了测试。bugzilla 报告可在 [此处](https://bbs.kanxue.com/elink@96fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6*7K9h3I4D9j5g2)9J5k6h3#2G2P5X3W2D9L8r3q4Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9r3!0%4i4K6g2X3j5Y4g2Y4i4K6u0W2j5$3N6A6i4K6y4r3K9h3c8Q4x3@1b7I4x3U0V1&6y4U0R3$3)找到。

## 漏洞

[以下代码](https://bbs.kanxue.com/elink@3e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8%4A6A6L8r3I4S2i4K6u0r3k6$3g2U0K9$3!0Q4x3X3c8V1k6i4k6Q4x3V1k6T1L8r3!0T1i4K6u0r3y4o6m8S2k6e0f1J5j5e0u0U0x3K6b7&6k6U0V1%4z5r3p5@1y4U0u0S2x3K6S2X3y4K6M7H3k6e0c8W2x3K6g2V1y4o6W2X3y4U0f1$3x3#2)9J5c8X3c8G2L8g2)9J5c8X3u0S2M7$3g2Q4x3V1k6F1M7#2y4U0M7X3W2H3N6p5I4G2j5h3c8W2M7W2)9J5k6h3y4H3M7q4)9J5x3@1H3J5y4K6p5$3) 用来加载外部脚本标签：

```
result

nsScriptLoadHandler::TryDecodeRawData(const uint8_t* aData,

                                      uint32_t aDataLength,

                                      bool aEndOfStream)

{

  int32_t srcLen = aDataLength;

  const char* src = reinterpret_cast<const char *>(aData);

  int32_t dstLen;

  nsresult rv =

    mDecoder->GetMaxLength(src, srcLen, &dstLen);



  NS_ENSURE_SUCCESS(rv, rv);



  uint32_t haveRead = mBuffer.length();

  uint32_t capacity = haveRead + dstLen;

  if (!mBuffer.reserve(capacity)) {

    return NS_ERROR_OUT_OF_MEMORY;

  }



  rv = mDecoder->Convert(src,

                         &srcLen,

                         mBuffer.begin() + haveRead,

                         &dstLen);



  NS_ENSURE_SUCCESS(rv, rv);



  haveRead += dstLen;

  MOZ_ASSERT(haveRead <= capacity, "mDecoder produced more data than expected");

  MOZ_ALWAYS_TRUE(mBuffer.resizeUninitialized(haveRead));



  return NS_OK;

}
```

当服务器中的新数据到达时这段代码将被 `OnIncrementalData` 调用。这是一个简单的整数溢出 Bug，当服务器发送超过 4GB 数据的时候即可发生。数据超过 4GB 的情况下，`capacity` 将会环回（wrap around），接下来的对 `mBuffer.reserve` 函数的调用并不会修改缓冲区。接下来 `mDecode->Convert` 函数将会把数据写入到 8GB 缓冲区的尾部（数据在浏览器中以 `char16_t` 的形式存储），这部分内存将会通过 `mmap` 块备份（对于一个非常大的块这是通用做法）。

修补也是相当简单：

```
int32_t haveRead = mBuffer.length();

-  uint32_t capacity = haveRead + dstLen;

-  if (!mBuffer.reserve(capacity)) {

+

+  CheckedInt<uint32_t> capacity = haveRead;

+  capacity += dstLen;

+

+  if (!capacity.isValid() || !mBuffer.reserve(capacity.value())) {

     return NS_ERROR_OUT_OF_MEMORY;

   }
```

这个漏洞第一眼看上不并没有什么搞头。它有一个必要条件，需要发送和申请多大几个GB的数据。正如我们即将看到的，该漏洞在我的 2015 MacBook Pro 上相当可靠的被利用，完成整个利用代码只需打开页面用时不到 1 分钟。我们接下来先来探索一下此漏洞为什么会在 macOS 上被利用并弹出一个计算器，然后我们来改进一下利用代码让它变的更可靠一些，并且占用更低的带宽（剧透：我们将会使用 HTTP 压缩）

## 操作

当超过 `mmap` 区的溢出发生时，我们首先关注的是有没有可能在溢出的内存之后可靠地申请一块空间。与一些堆分配器相反，`mmap`（可以看作是内核提供的内存分配器）是非常具有确定性的：如果没有合适的内存块，调用 `mmap` 两次将会导致两次连续的内存映射。你可以用下边的代码来尝试一下。注意，实验的结果的异同取决于代码是运行在 Linux 系统还是 macOS 系统上。`mmap` 内存区相较于 macOS 系统上由低向高增长，在 Linux 系统上，是由高向低增长。在本篇文章接下来的部分，我们将会专注于 macOS。Linux 或者 Windows 上应该也可能存在相似的利用代码。

```
#include <sys/mman.h>

#include <stdio.h>



const size_t MAP_SIZE = 0x100000;       // 1 MB



int main()

{

    char* chunk1 = mmap(NULL, MAP_SIZE, PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_PRIVATE, -1, 0);

    char* chunk2 = mmap(NULL, MAP_SIZE, PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_PRIVATE, -1, 0);



    printf("chunk1: %p - %p\n", chunk1, chunk1 + MAP_SIZE);

    printf("chunk2: %p - %p\n", chunk2, chunk2 + MAP_SIZE);



    return 0;

}
```

上边的程序向我们展示了通过简单的映射所有的内存页直到所有已经存在的分页被填满，然后通过 `mmap` 再申请一块内存块。为了验证这个过程，我们接下来将会这样做：

1. 加载一个包含一段脚本（将会触发溢出的 `payload.js`）的 HTML 文档并且异步执行一些 JavaScript 代码（`code.js`，实现第 3 步和第 5 步）。
2. 当浏览器请求 `payload.js` 时，使服务器返回一个 `Content-Length` 为 `0x100000001` 但是只发送 `0xffffffff` 字节的数据。
3. 接下来，让 JavaScript 代码申请多个巨大（1GB）的 `ArrayBuffers`（在缓冲区实际写入前，内存不一定被使用）。
4. 让服务器发送 `payload.js` 剩下的 2 个字节。
5. 检查所有 `ArrayBuffer` 对象的前几个字节，其中的某一个应该会包含服务器发送的数据。

为了实现这个过程，我们需要一些浏览器中的 JavaScript 代码和服务器之间的同步原语。为此，我在 python 的 `asyncio` 库智商写了一个小小的 webserver，它包含一个方便的 `Event` 对象，用来和协同程序同步。创建两个全局事件可使客户端代码完成当前任务等待 webserver 进行下一步的操作时通知服务器。`/sync` 的处理例程如下所示：

```
async def sync(request, response):

    script_ready_event.set()

    await server_done_event.wait()

    server_done_event.clear()



    response.send_header(200, {

        'Content-Type': 'text/plain; charset=utf-8',

        'Content-Length': '2'

    })



    response.write(b'OK')

    await response.drain()
```

客户端中我是用了同步 `XMLHttpRequests` 来阻塞脚本的执行，直到服务器完成相关工作：

```
function synchronize() {

    var xhr = new XMLHttpRequest();

    xhr.open('GET', location.origin + '/sync', false);

    // Server will block until the event has been fired

    xhr.send();

}
```

这样，我们就可以实现上边的场景并且将会看到实际上有一个 `ArrayBuffer` 对象的开始处包含了我们的 payload 字节。不过还有一个小小的限制条件：我们只能通过有效的 UTF-16 来进行溢出，因为这是 Firefox 内部使用的。我们必须记住这一点。现在剩下的就是找到一些更有趣的事情，用内存分配来取代对 `ArrayBuffer` 的溢出。

## 寻找目标对象

因为 `malloc`（同样的 C++ 中的 `new` 操作）将在某些时候使用 `mmap` 请求更多的内存，所以像这些操作分配的内存可能是我们代码所感兴趣的。我走了一条不同的路线。最初我想检测一下是否有可能溢出到 JavaScript 对象中，比如说使数组的或者其他类似对象的长度腐败。为此，我开始围绕着 JavaScript 分配器深入发掘，来看 `JSObject` 被存储在哪里。Spidermonkey（Firefox 中的 JavaScript 引擎）把 `JSObjet` 存储在两个独立的区域中：

1. 永久堆（The tenured heap）。长生命周期的对象，同时少数被选择的对象类型在此处分配。这是一个相当经典的堆，跟踪自由内存然后在未来重复使用。
2. 托管区。这是一块包含短暂生命周期对象的内存区域。大多数的 `JSObject` 最初都被分配在此处，然后如果在下一个垃圾回收周期（包括更新所有指向他们的指针，因此要求牢记收集器知道其对象的所有指针）中依然存在则被移动到永久堆中。托管区不需要释放列表或者相似的结构：在一个垃圾回收循环之后，托管区只是在所有的存活对象被移出之后简单的声明一下自己是可用的。

对 Spidermonkey 内部更深入的讨论可以参见 phrack 上的 [这篇文章](https://bbs.kanxue.com/elink@65fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3y4G2L8g2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5&6i4K6u0r3x3e0c8Q4x3X3g2Z5N6r3#2D9i4K6t1K6j5i4u0@1K9h3y4D9k6b7`.`.)。

永久堆中存储对象的容器叫做 `Arenas`：

```
/*

 * Arenas are the allocation units of the tenured heap in the GC. An arena

 * is 4kiB in size and 4kiB-aligned. It starts with several header fields

 * followed by some bytes of padding. The remainder of the arena is filled

 * with GC things of a particular AllocKind. The padding ensures that the

 * GC thing array ends exactly at the end of the arena:

 *

 * <----------------------------------------------> = ArenaSize bytes

 * +---------------+---------+----+----+-----+----+

 * | header fields | padding | T0 | T1 | ... | Tn |

 * +---------------+---------+----+----+-----+----+

 * <-------------------------> = first thing offset

 */

class Arena

{

    static JS_FRIEND_DATA(const uint32_t) ThingSizes[];

    static JS_FRIEND_DATA(const uint32_t) FirstThingOffsets[];

    static JS_FRIEND_DATA(const uint32_t) ThingsPerArena[];



    /*

     * The first span of free things in the arena. Most of these spans are

     * stored as offsets in free regions of the data array, and most operations

     * on FreeSpans take an Arena pointer for safety. However, the FreeSpans

     * used for allocation are stored here, at the start of an Arena, and use

     * their own address to grab the next span within the same Arena.

     */

    FreeSpan firstFreeSpan;



    // ...
```

