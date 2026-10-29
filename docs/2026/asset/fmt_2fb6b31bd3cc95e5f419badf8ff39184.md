
接着以上步骤，在Frida中使用`getExportByName`，来获取函数指针并Hook。第一个参数和第二个参数都是兼容OC的类型，可以转为Frida的`ObjC.Object`类型进行访问。第三个参数为Swift的`Foundation.Data`类型，怎么获取其数据呢？

先来研究`Foundation.Data`，Swift的Foundation库已经[开源](https://bbs.kanxue.com/elink@396K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8Y4y4%4K9h3k6@1i4K6u0V1j5$3!0J5k6h3I4A6j5Y4y4Q4x3X3c8X3L8%4g2F1k6r3q4@1K9h3!0F1)，可以一探究竟。找到`Data.swift`，将其简化一下，只保留相关定义：

```
public struct Data : ReferenceConvertible, Equatable, Hashable, RandomAccessCollection, MutableCollection, RangeReplaceableCollection, MutableDataProtocol, ContiguousBytes {

    // A small inline buffer of bytes suitable for stack-allocation of small data.

    struct InlineData {// max=15 bytes

        var bytes: Buffer // 15 bytes

        var length: UInt8 // 1 bytes

    }

    // A buffer of bytes too large to fit in an InlineData, but still small enough to fit a storage pointer + range in two words.

    struct InlineSlice {  // max=HalfInt.max

        var slice: Range<HalfInt>

        var storage: __DataStorage

    }

    final class RangeReference {

    }

    // A buffer of bytes whose range is too large to fit in a signle word. Used alongside a RangeReference to make it fit into _Representation's two-word size.

    struct LargeSlice {

        var slice: RangeReference

        var storage: __DataStorage

    }



    // The actual storage for Data's various representations.

    internal enum _Representation {

        case empty

        case inline(InlineData)

        case slice(InlineSlice)

        case large(LargeSlice)

    }

    internal var _representation: _Representation

}

// Underlying storage representation for medium and large data.

internal final class __DataStorage {

    //

}

```

基本结构已知，根据的`Data`大小不同有不同的表示方式，具体参考上面代码的注释。在arm64环境下，我们的测试请求响应数据大小只有几百bytes，符合`InlineSlice`。结构如下图所示：

![](https://bbs.kanxue.com/upload/attach/202007/736021_9Q9Y8DF6N8KTF4P.jpg)

[Swift ABI调用约定](https://bbs.kanxue.com/elink@564K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8Y4y4%4K9h3k6@1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6m8b7V1W2Q4x3V1k6o6j5h3I4D9K9h3&6Y4b7$3!0F1N6X3g2F1N6r3W2G2L8W2)9J5k6i4u0K6N6q4)9J5x3$3W2V1x3e0x3`.)里有介绍，swift非常注重第一类型，并且能够在寄存器中直接传递类型值。

在此处的`Data`类型就用到了这种方法。`Data`类型的数据被分为两个寄存器来传递：`args[2]`传递range信息，`args[3]`传递`__DataStorage`指针(`args[0]`指向`NSURLSession`，`args[1]`指向`NSURLSessionDataTask`，函数原型只有三个参数，但实际上调用时会有4个参数)。

```
args[2] = 0x1e000000000 => 0x0,0x1e0 => range from 0 to 0x1e0

args[3] = DataStorage's address

```

而`__DataStorage`是swift class，可以根据[class memory-layout](https://bbs.kanxue.com/elink@cd6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1j5h3&6F1k6i4u0v1K9h3&6Q4x3V1k6e0N6$3W2X3N6q4)9J5k6p5#2W2L8h3!0J5P5f1I4S2P5h3!0#2N6q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6e0N6$3W2X3N6p5y4G2M7X3g2Q4x3V1k6e0N6s2u0A6L8X3N6Q4x3X3g2K6N6$3W2X3N6l9`.`.)，解析出对应的地址（参考上面的结构关系图，代码实现在`SDSwiftDataStorage.ts`中）。

注：事实上，我们在正向开发过程中，用Xcode打断点调试，也能看到swift `Data`的大概结构，如下图：

![](https://bbs.kanxue.com/upload/attach/202007/736021_MNHTHFZWAA6WXFX.png)

# 0x04. Kill SSL Pinning

先看看Alamofire怎么使用SSL Pinning:

```
let manager = ServerTrustManager(evaluators: ["httpbin.org": PinnedCertificatesTrustEvaluator()])

self.session = Session(serverTrustManager: manager)

```

跟踪代码，在`func attemptServerTrustAuthentication(with challenge: URLAuthenticationChallenge) -> ChallengeEvaluation` 里会判断是否需要检查证书，设置了Pinning后会返回`useCredential`，如果没有设置任何检查项，则返回`(.performDefaultHandling, nil, nil)`，我们在此只需修改该函数永远返回这个Tuple值。

Tuple的memory layout是线性布局的(如果未内存对齐，会为了内存对齐填充0)，用Frida Hook后，指针指向的第一个值就是要修改的值。

代码在`HookAFServerTrust.ts`中，如下：

```
// TypeScript

let func_attemptServerTrust = Module.getExportByName(null, '$s9Alamofire15SessionDelegateC32attemptServerTrustAuthentication4withSo36NSURLSessionAuthChallengeDispositionV11disposition_So15NSURLCredentialCSg10credentialAA7AFErrorOSg5errortSo019NSURLAuthenticationK0C_tF'); 

log(`[HookAFServerTrust] hook func_attemptServerTrust ${func_attemptServerTrust}`);

Interceptor.attach(func_attemptServerTrust, {

    onLeave(retval:InvocationReturnValue) {

        // force set retval to 0x1 to enable .performDefaultHandling

        let val = retval.toInt32();

        if (val != 0x1) {

            log(`[HookAFServerTrust] attemptServerTrustAuthentication retval ${retval}, reset to 0x1`);

            let fakeret = new NativePointer(0x1)

            retval.replace(fakeret)

        }

    }

});

```

# 0x05. Swift String

如果我们想获取字符串`Swift.String`的值，该怎么做呢？

查看`String`的[内存布局](https://bbs.kanxue.com/elink@b67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1j5h3&6F1k6i4u0v1K9h3&6Q4x3V1k6e0N6$3W2X3N6q4)9J5k6p5#2W2L8h3!0J5P5f1I4S2P5h3!0#2N6q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6e0N6$3W2X3N6p5y4G2M7X3g2Q4x3V1k6e0N6s2u0A6L8X3N6Q4x3X3g2K6N6$3W2X3N6l9`.`.)。我们可知，`String`是Struct，值类型，分为 Small(<=15bytes) 和 Large两种表示方式。我们知道`Foundation.URL`有个函数可以根据`String`初始化。符号如下：

`s10Foundation3URLV6stringACSgSSh_tcfC ---> Foundation.URL.init(string: __shared Swift.String) -> Foundation.URL?`。hook这个函数，笔者发现，对于Small String，Swift会直接使用两个参数来传递该值。对于Large String，是使用两个参数分别指向了两个属性（符合ABI里介绍的调用约定，Swift值类型在传递时特定的情况下会直接用寄存器传递）。具体代码在文件`SDSwiftString.ts`中。

# 0x06. One More Step

能否直接调用Swift的运行时函数呢？

答案是可以的，但是需要传入合适的参数。

举个例子：

使用IDA分析我们的测试程序，可以发现`$s10Foundation4DataV19_bridgeToObjectiveCSo6NSDataCyF` 函数原型为

`Foundation.Data._bridgeToObjectiveC() -> __C.NSData`

它可以将`Data`类型转为`NSData`类型，而`NSData`类型我们是可以直接在Frida下使用的OC类型。参考以上，可以将`Data`用两个参数传递给这个函数即可，分别表示range和`DataStorage`。示例代码在`SwiftRuntime.ts`中：

```
// 获取函数符号地址

const func_data2nsdata_ptr = Module.getExportByName(null, '$s10Foundation4DataV19_bridgeToObjectiveCSo6NSDataCyF');

//生成函数指针用于调用

funcptr_data_bridgeToObjectiveC = new NativeFunction(func_data2nsdata_ptr,'pointer', ['pointer', 'pointer']);



//调用

let ret:NativePointer = funcptr_data_bridgeToObjectiveC(rangePtr, dataStoragePtr) as NativePointer;

let ocret = new ObjC.Object(ret); // is __NSSwiftData: NSData

// 接下来可以直接调用OC的函数了，比如： ocret.length() ， ocret.bytes()

```

# 0x07. 总结

Swift为了与OC兼容，保留了部分动态性，同时，为了更高效的运行，充分利用了寄存器，能直接传递就不间接传递，以实现的复杂性来得到运行的高效性。当然对于逆向研究来说，这里面还有很多可以挖掘的地方，笔者在此仅尝试了一小部分且仍有瑕疵，欢迎大家留言交流学习。如果觉得有用，欢迎star项目 [FridaHookSwiftAlamofire](https://bbs.kanxue.com/elink@373K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6h3W2D9i4K6u0V1N6%4g2Q4x3V1k6r3M7X3W2V1j5f1S2G2L8$3E0e0N6$3W2X3N6p5q4D9j5h3#2G2k6X3W2J5k6b7`.`.)

#### 参考资料

- [Swift ABI](https://bbs.kanxue.com/elink@eddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8Y4y4%4K9h3k6@1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6m8b7V1W2Q4x3V1k6o6j5h3I4D9K9h3&6Y4b7$3!0F1N6X3g2F1N6r3W2G2L8W2)9J5k6i4u0K6N6l9`.`.)

- [swift-corelibs-foundation](https://bbs.kanxue.com/elink@845K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8Y4y4%4K9h3k6@1i4K6u0V1j5$3!0J5k6h3I4A6j5Y4y4Q4x3X3c8X3L8%4g2F1k6r3q4@1K9h3!0F1)

- [Swift-MemoryLayout](https://bbs.kanxue.com/elink@859K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1j5h3&6F1k6i4u0v1K9h3&6Q4x3V1k6e0N6$3W2X3N6q4)9J5k6p5#2W2L8h3!0J5P5f1I4S2P5h3!0#2N6l9`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
