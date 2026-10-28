
```cpp
class StringChunk {
char\* build\_raw() {
int raw\_size = sizeof(sStringChunk);
for (int i = 0; i < StringOffset\_list.size(); i++) {
raw\_size += 4;
}
for (int i = 0; i < StringItem\_list.size(); i++) {
raw\_size += StringItem\_list[i]->size;
}
// 4字节对齐
if (raw\_size % 4 != 0) {
raw\_size += 2;
}
char\* new\_raw = (char\*)malloc(raw\_size);
memset(new\_raw, 0, raw\_size);
char\* cp\_point = new\_raw;
((sStringChunk\*)cp\_point)->scSignature = scSignature;
((sStringChunk\*)cp\_point)->scSize = raw\_size;
((sStringChunk\*)cp\_point)->scStringCount = StringItem\_list.size();
((sStringChunk\*)cp\_point)->scStyleCount = scStyleCount;
((sStringChunk\*)cp\_point)->scUNKNOWN = scUNKNOWN;
((sStringChunk\*)cp\_point)->scStringPoolOffset = scStringPoolOffset + 4;
((sStringChunk\*)cp\_point)->scStylePoolOffset = scStylePoolOffset;
cp\_point += sizeof(sStringChunk);
for (int i = 0; i < StringOffset\_list.size(); i++) {
\*(int\*)cp\_point = StringOffset\_list[i];
cp\_point += 4;
}
for (int i = 0; i < StringItem\_list.size(); i++) {
memcpy(cp\_point, StringItem\_list[i]->raw, StringItem\_list[i]->size);
cp\_point += StringItem\_list[i]->size;
}
return new\_raw;
}
StringChunk\* add\_string(std::wstring wstr) {
StringItem\* mStringItem = new StringItem(wstr);
int new\_StringOffset = StringOffset\_list[scStringCount - 1] + StringItem\_list[scStringCount - 1]->size;
StringOffset\_list.push\_back(new\_StringOffset);
StringItem\_list.push\_back(mStringItem);
char\* new\_raw = build\_raw();
return new StringChunk(new\_raw);
}
};
class AndroidManifestEditor {
void add\_string(std::wstring wstr) {
for (int i = 0; i < chunk\_list.size(); i++) {
if (chunk\_list[i]->type == "StringChunk") {
StringChunk\* newStringChunk = mStringChunk->add\_string(wstr);
mAndroidManifestHead->file\_size += (newStringChunk->scSize - mStringChunk->scSize);
delete mStringChunk;
mStringChunk = newStringChunk;
delete chunk\_list[i];
chunk\_list[i] = new Chunk(newStringChunk->addr);
}
}
}
};
```

## 修改启动项

当 AndroidManifest 中已存在 ApplicationName 时直接修改其字符串偏移即可

```cpp
class AndroidManifestEditor {
void change\_application\_name(std::wstring application\_name) {
int id = mStringChunk->get\_id(application\_name);
if (id == -1) {
add\_string(application\_name);
id = mStringChunk->get\_id(application\_name);
}
for (int i = 0; i < mStartTagChunkList.size(); i++) {
if (mStartTagChunkList[i]->mEndTagChunk->get\_string() == L"application") {
std::vector<AttributeChunk\*> tAttributeChunkList =
mStartTagChunkList[i]->mAttributeChunkList;
for (int i = 0; i < tAttributeChunkList.size(); i++) {
if (tAttributeChunkList[i]->get\_srting().find(L"android:name=") != std::wstring::npos) {
tAttributeChunkList[i]->change\_value(id);
}
}
}
}
}
};
```

## 总结

AndroidManifest 的数据结构总体来讲不算过于复杂，和PE/ELF比算是小巫见大巫了，不过由于搞得不是很精细，后续还需要多做一些测试

## 参考资料

<a href="elink@616K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5x3r3#2G2L8X3E0W2P5g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4e0M7#2">Android逆向之旅—解析编译之后的AndroidManifest文件格式</a>

[AndroidManifest二进制文件格式分析](https://bbs.kanxue.com/thread-194206.htm)

<a href="elink@227K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5$3z5o6l9H3y4e0M7@1i4K6y4r3N6i4c8E0i4K6g2X3K9h3c8Q4x3@1b7H3">Android逆向笔记 —— AndroidManifest.xml 文件格式解析</a>

# AndroidManifest解析笔记

## 前言

自笔者开始学习Android逆向的第一天起就一直被一个问题所困扰，那就是 AndroidManifest 的编码总是会有问题，相信大家在使用 apktool 的时候如果直接 `java -jar apktool.jar d HelloWorld.apk` 那么在回编译的时候大概率会出现报错，这是因为 apktool 的资源的解析是有问题的，添加参数 -r 可以屏蔽资源的解析，但这样一来 AndroidManifest 就无法解析，而在逆向的过程中很常见的需求就是修改 AndroidManifest，例如修改启动类又或者添加debug标志，所以很容易想到的方法就是 apktool.jar 屏蔽资源解析，然后使用针对 AndroidManifest 修改的工具，这其中有许多大佬开源了自己的 AndroidManifest 修改工具，但在笔者的使用过程中总是会有这样那样的问题，正好最近有个需求是修改 AndroidManifest，所以在这里记录下 AndroidManifest.xml 二进制文件格式分析。

## 格式总览

首先按照惯例先贴上这张神图，可以说总结的非常全面，但此图也有自己的问题，那就是难以让读者在心中对整个 AndroidManifest 格式有一个大概的雏形。

![image-20240118115059789](https://bbs.kanxue.com/upload/attach/202401/950902_YAJ6SDC6MNAM9TH.jpg)

在这张图中我们需要明白整个 AndroidManifest 格式分为六大区域，七种 Chunk 类型，依次分别为 AndroidManifestHead、StringChunk、ResourceChunk、StartNamespaceChunk、StartTagChunk&EndTagChunk、EndNameSpaceChunk，其中多数情况 ResourceChunk为空，不太需要处理。

![image-20240118120250532](https://bbs.kanxue.com/upload/attach/202401/950902_AQQZXXNE6CDAS8A.jpg)

## AndroidManifestHead

AndroidManifestHead 占据 AndroidManifest 的起始8个字节，magic 标志着这个文件是 AndroidManifest 格式，file\_size 记录 AndroidManifest.xml 文件的大小，这种文件头可以说是异常简单了。

|  |  |
| --- | --- |
| 1  2  3  4 | `struct sAndroidManifestHead {`  `int` `magic;` `/``/` `标志着这个文件是 AndroidManifest 格式`  `int` `file_size;` `/``/` `记录 AndroidManifest.xml  文件的大小`  `};` |

## Chunk

笔者在分析完 AndroidManifest 后总结出 StringChunk、ResourceChunk、StartNamespaceChunk、StartTagChunk、EndTagChunk、EndNameSpaceChunk 这六种块的前 8 个字节是一样的

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50 | `class` `Chunk {`  `public``:`  `struct` `sChunk {`  `int` `cSignature;` `// 块类型标识`  `// cSignature = 0x001c0001  stringChunk`  `// cSignature = 0x00080180  resourceChunk`  `// cSignature = 0x00100100  startNameSpaceChunk`  `// cSignature = 0x00100102  startTagChunk`  `// cSignature = 0x00100103  endTagChunk`  `// cSignature = 0x00100101  EndNameSpaceChunk`  `int` `cSize;` `// 块的大小`  `};`    `char``* addr;`  `int` `cSignature;`  `int` `cSize;`  `std::string type;`    `Chunk(``char``* chunk_addr) {`  `addr = chunk_addr;`  `sChunk* mChunk = (sChunk*)chunk_addr;`    `if` `(mChunk->cSignature == 0x001c0001) {`  `type =` `"StringChunk"``;`  `}`  `else` `if` `(mChunk->cSignature == 0x00080180) {`  `type =` `"ResourceChunk"``;`  `}`  `else` `if` `(mChunk->cSignature == 0x00100100) {`  `type =` `"StartNameSpaceChunk"``;`  `}`  `else` `if` `(mChunk->cSignature == 0x00100102) {`  `type =` `"StartTagChunk"``;`  `}`  `else` `if` `(mChunk->cSignature == 0x00100103) {`  `type =` `"EndTagChunk"``;`  `}`  `else` `if` `(mChunk->cSignature == 0x00100101) {`  `type =` `"EndNameSpaceChunk"``;`  `}`  `else` `{`  `type =` `"Unknown"``;`  `}`  `cSize = mChunk->cSize;`  `};`    `Chunk* NextChunk() {`  `return` `new` `Chunk((addr + cSize));`  `}`  `};` |

由此我们可以先对 AndroidManifest 进行一个大致的遍历

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `int` `main() {`  `const` `char``* AndroidManifestPath =` `"C:\\Users\\lxz\\source\\repos\\AndroidManifestEditor\\AndroidManifest.xml"``;`  `char``* mAndroidManifestData = read_file(AndroidManifestPath);`  `sAndroidManifestHead* mAndroidManifestHead = (sAndroidManifestHead*)mAndroidManifestData;`    `std::vector<Chunk*> chunkList;`    `Chunk* chunk =` `new` `Chunk(((``char``*)mAndroidManifestHead +` `sizeof``(sAndroidManifestHead)));`  `while` `(chunk->type !=` `"Unknown"``) {`  `chunkList.push_back(chunk);`  `chunk = chunk->NextChunk();`  `}`    `for` `(Chunk* chunk : chunkList) {`  `printf``(``"%s\n\n"``, chunk->type.c_str());`  `delete` `chunk;`  `}`  `chunkList.clear();`  `}` |

输出的结果大致为这样，可以完美对应上方的结构图

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `stringChunk`    `resourceChunk`    `startNameSpaceChunk`    `startTagChunk`    `startTagChunk`    `endTagChunk`    `startTagChunk`    `......`    `endTagChunk`    `endTagChunk`    `EndNameSpaceChunk` |

## StringChunk

StringChunk 的格式就略显复杂了，首先我们先看下 StringChunk 的头部，这部分共计 7 \* 4 = 28 个字节的大小

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `struct sStringChunk {`  `int` `scSignature;` `/``/` `块类型标识`  `int` `scSize;` `/``/` `块大小`  `int` `scStringCount;` `/``/` `字符串数量`  `int` `scStyleCount;` `/``/` `未知`  `int` `scUNKNOWN;` `/``/` `未知`  `int` `scStringPoolOffset;` `/``/` `字符串集合的偏移`  `int` `scStylePoolOffset;` `/``/` `未知`  `};` |

StringChunk 的头部后面跟着的就是 StringOffsets，这部分区域是的大小为 scStringCount \*4，其中每4个字节记录了对应 StringItem 在文件中的的偏移，这里画了一个图可以直观理解一下。
