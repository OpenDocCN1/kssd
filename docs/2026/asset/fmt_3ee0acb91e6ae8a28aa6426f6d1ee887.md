
```cpp
class AttributeChunk {
public:
struct sAttributeChunk {
int acNamespaceUri; // 命名空间 Uri 索引
int acName; // 属性名称(key)的索引
int acValueStr; // 属性内容(value)的索引（当前类型为字符串时使用该属性）
int acType; // 属性内容(value)的类型（String、Int、Bool ...）
int acData; // 属性内容(value)的索引
};
int acNamespaceUri;
int acName;
int acValueStr;
int acType;
int acData;
char* addr;
StringChunk*& mStringChunk;
StartNamespaceChunk*& mStartNamespaceChunk;
AttributeChunk(char* chunk_addr, StringChunk** vStringChunk, StartNamespaceChunk** vStartNamespaceChunk):mStringChunk(*vStringChunk), mStartNamespaceChunk(*vStartNamespaceChunk) {
addr = chunk_addr;
acNamespaceUri = ((sAttributeChunk*)addr)->acNamespaceUri;
acName = ((sAttributeChunk*)addr)->acName;
acValueStr = ((sAttributeChunk*)addr)->acValueStr;
acType = ((sAttributeChunk*)addr)->acType;
acData = ((sAttributeChunk*)addr)->acData;
}
std::wstring get_srting() {
std::wstring resData = L"";
// 当 acNamespaceUri 为 -1 时，命名空间为空
if (acNamespaceUri != -1 && acNamespaceUri == mStartNamespaceChunk->sncUri) {
resData = mStartNamespaceChunk->get_string() + L":";
}
// 属性名称
resData += mStringChunk->get_string(acName);
resData += L"=";
// 属性值的数据类型
if ((acType >> 24) == 0x10) {
// 整数
resData += L"\"";
resData += std::to_wstring(acData);
resData += L"\"";
}
else if ((acType >> 24) == 0x3) {
// 字符串
resData += L"\"";
resData += mStringChunk->get_string(acValueStr);
resData += L"\"";
}
else if ((acType >> 24) == 0x12) {
// 布尔值
if (acData == -1) {
resData += L"\"true\"";
}
else if (acData == 0) {
resData += L"\"false\"";
}
}
else if ((acType >> 24) == 0x1) {
// 资源ID
std::wstringstream wss;
wss << std::hex << acData;
std::wstring hexString = wss.str();
resData += L"\"@";
resData += std::wstring(wss.str());
resData += L"\"";
}
else {
// 未识别格式
std::wstringstream wss;
wss << std::hex << acData;
std::wstring hexString = wss.str();
resData += L"\"[Error]";
resData += std::wstring(wss.str());
resData += L"\"";
}
return resData;
}
void change_value(int value) {
((sAttributeChunk*)addr)->acValueStr = value;
acValueStr = value;
}
};
class StartTagChunk {
public:
struct sStartTagChunk {
int stcSignature; // 块类型标识
int stcSize; // 块大小
int stcLineNumber; // 该属性在明文中的行数
int stcUNKNOWN; // 未知
int stcNamespaceUri; // 字符串 Uri 索引
int stcName; // Tag 名称字符串索引
int stcFlags; // 未知
int stcAttributeCount; // AttributeChunk 的数量
int stcClassAttribute; // 未知
};
char* addr;
int stcSignature;
int stcSize;
int stcLineNumber;
int stcUNKNOWN;
int stcNamespaceUri;
int stcName;
int stcFlags;
int stcAttributeCount;
int stcClassAttribute;
EndTagChunk* mEndTagChunk;
StringChunk*& mStringChunk;
StartNamespaceChunk*& mStartNamespaceChunk;
std::vector<AttributeChunk*> mAttributeChunkList;
StartTagChunk(char* chunk_addr, StringChunk** vStringChunk, StartNamespaceChunk** vStartNamespaceChunk)
: mStringChunk(*vStringChunk), mStartNamespaceChunk(*vStartNamespaceChunk){
addr = chunk_addr;
stcSignature = ((sStartTagChunk*)addr)->stcSignature;
stcSize = ((sStartTagChunk*)addr)->stcSize;
stcLineNumber = ((sStartTagChunk*)addr)->stcLineNumber;
stcUNKNOWN = ((sStartTagChunk*)addr)->stcUNKNOWN;
stcName = ((sStartTagChunk*)addr)->stcName;
stcFlags = ((sStartTagChunk*)addr)->stcFlags;
stcAttributeCount = ((sStartTagChunk*)addr)->stcAttributeCount;
stcClassAttribute = ((sStartTagChunk*)addr)->stcClassAttribute;
for (int i = 0; i < stcAttributeCount; i++) {
AttributeChunk* mAttributeChunk =
new AttributeChunk(addr + sizeof(sStartTagChunk) + 20 * i, &mStringChunk, &mStartNamespaceChunk);
mAttributeChunkList.push_back(mAttributeChunk);
}
};
std::wstring get_string() {
std::wstring resData = L"";
for (int i = 0; i < mAttributeChunkList.size(); i++) {
resData += mAttributeChunkList[i]->get_srting();
resData += L"\n";
}
return resData;
}
};
```

## `EndTagChunk`

`EndTagChunk` 的结构就没 `StartTagChunk` 那么复杂了，可以说简单的很

```cpp
class EndTagChunk {
public:
struct sEndTagChunk {
int etcSignature; // 块类型标识
int etcSize; // 块大小
int etcLineNumber; // 该属性在明文中的行数
int etcUNKNOWN; // 未知
int etcNamespaceUri; // 字符串 Uri 索引
int etcName; // Tag 名称字符串索引
};
char* addr;
int etcSignature;
int etcSize;
int etcLineNumber;
int etcUNKNOWN;
int etcNamespaceUri;
int etcName;
StringChunk*& mStringChunk;
EndTagChunk(char* chunk_addr, StringChunk** vStringChunk):mStringChunk(*vStringChunk) {
addr = chunk_addr;
etcSignature = ((sEndTagChunk*)addr)->etcSignature;
etcSize = ((sEndTagChunk*)addr)->etcSize;
etcLineNumber = ((sEndTagChunk*)addr)->etcLineNumber;
etcUNKNOWN = ((sEndTagChunk*)addr)->etcUNKNOWN;
etcName = ((sEndTagChunk*)addr)->etcName;
};
std::wstring get_string() {
return mStringChunk->get_string(etcName);
}
};
```

## 解析小结

自此我们在为借助其他第三方模块的完成了绝大部分 `Chunk` 的解析，`AndroidManifest` 也变得不再神秘，那么就简单的 show 一下吧

```cpp
class AndroidManifestEditor {
public:
struct sAndroidManifestHead {
int magic;
int file_size;
};
char* mAndroidManifestData;
sAndroidManifestHead* mAndroidManifestHead;
StringChunk* mStringChunk;
StartNamespaceChunk* mStartNamespaceChunk;
EndNameSpaceChunk* mEndNameSpaceChunk;
std::vector<Chunk*> chunk_list;
std::vector<StartTagChunk*> mStartTagChunkList;
std::vector<EndTagChunk*> mEndTagChunkList;
std::vector<StartTagChunk*> tStartTagChunkList; // 临时列表，用来处理 StartTagChunk 和 EndTagChunk 的对应关系
AndroidManifestEditor(const char* AndroidManifestPath) {
mAndroidManifestData = read_file(AndroidManifestPath);
mAndroidManifestHead = (sAndroidManifestHead*)mAndroidManifestData;
Chunk* chunk = new Chunk(((char*)mAndroidManifestHead + sizeof(sAndroidManifestHead)));
while (chunk->type != "Unknown") {
if (chunk->type == "StartNameSpaceChunk") {
mStartNamespaceChunk = new StartNamespaceChunk(chunk->addr, &mStringChunk);
}
else if (chunk->type == "StringChunk") {
mStringChunk = new StringChunk(chunk->addr);
}
else if (chunk->type == "StartTagChunk") {
StartTagChunk* mStartTagChunk = new StartTagChunk(chunk->addr, &mStringChunk, &mStartNamespaceChunk);
mStartTagChunkList.push_back(mStartTagChunk);
tStartTagChunkList.push_back(mStartTagChunk);
}
else if (chunk->type == "EndTagChunk") {
EndTagChunk* mEndTagChunk = new EndTagChunk(chunk->addr, &mStringChunk);
mEndTagChunkList.push_back(mEndTagChunk);
tStartTagChunkList.back()->mEndTagChunk = mEndTagChunk;
tStartTagChunkList.pop_back();
}
else if (chunk->type == "EndNameSpaceChunk") {
mEndNameSpaceChunk = new EndNameSpaceChunk(chunk->addr, &mStringChunk);
}
chunk_list.push_back(chunk);
chunk = chunk->NextChunk();
}
}
void show() {
printf("AndroidManifest 魔数\t\t\t\t0x%x\n", mAndroidManifestHead->magic);
printf("AndroidManifest 文件大小\t\t\t0x%x\n", mAndroidManifestHead->file_size);
int start_tag_id = 0;
int end_tag_id = 0;
for (int i = 0; i < chunk_list.size(); i++) {
Chunk* chunk = chunk_list[i];
printf("[%s] \n", chunk->type.c_str());
if (chunk->type == "StringChunk") {
printf("\n");
}
else if (chunk->type == "ResourceChunk") {
printf("\n");
}
else if (chunk->type == "StartNameSpaceChunk") {
std::wstring wstr = mStartNamespaceChunk->get_string();
printf("%S\n\n", wstr.c_str());
}
else if (chunk->type == "StartTagChunk") {
StartTagChunk* mStartTagChunk = mStartTagChunkList[start_tag_id];
printf("%S\n", mStartTagChunk->mEndTagChunk->get_string().c_str());
printf("%S\n\n", mStartTagChunk->get_string().c_str());
start_tag_id++;
}
else if (chunk->type == "EndTagChunk") {
EndTagChunk* mEndTagChunk = mEndTagChunkList[end_tag_id];
std::wstring wstr = mEndTagChunk->get_string();
end_tag_id++;
printf("%S\n\n", wstr.c_str());
}
else if (chunk->type == "EndNameSpaceChunk") {
std::wstring wstr = mEndNameSpaceChunk->get_string();
printf("%S\n\n", wstr.c_str());
break;
}
}
}
};
```

## 添加字符串

现在 `AndroidManifest` 已经可以任由我们搓扁揉圆了，重新组装 `StringChunk` 一下，就可以向其中添加任意的字符串
