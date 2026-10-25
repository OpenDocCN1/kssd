
- 所以只需要定位到`Il2CppCodeRegistration` `codeRegistration`这个全局变量即可，然后遍历`codeGenModules`找到方法对应的`Image`，然后通过方法的token（`Il2CppMethodDefinition`下有）去`codeGenModules->methodPointers`取出函数指针，即可将方法和函数指针对应上。

- 具体代码如下：

```
uint32_t GetMethodPointer(const Il2CppImageDefinition *image,uint32_t token)
{
for(int i=0;i<CodeRegistration->codeGenModulesCount;i++)
{
const Il2CppCodeGenModule *module=CodeRegistration->codeGenModules[i];
if(!strcmp(module->moduleName,GetStringFromIndex(image->nameIndex)))
{
return module->methodPointers[GetTokenRowId(token)-1];
}
}
printf("invalid!\n");
return 0;
}
```

# 0x4 获得field的偏移地址

- 同样的从il2cpp的api出发，可以发现该函数直接调用了`Class::GetFieldFromName`，直接去查看该函数。

![](https://bbs.kanxue.com/upload/attach/202206/948449_YYUSA9RTKHW92JV.png)

- 找到`Class::GetFieldFromName`，它通过`GetFields`获得所有的`FieldInfo`，然后返回对应的`FieldInfo`。

![](https://bbs.kanxue.com/upload/attach/202205/948449_78CWZ5WAP2JBGHY.png)

- 看看`GetFields`函数，其中调用了`SetupFieldsLocked`。

![](https://bbs.kanxue.com/upload/attach/202205/948449_4TC9NWBBYQFXGER.png)

- 继续找到`SetupFieldsLocked`，该函数递归初始化了类和父类的所有`FieldInfo`，使用`SetupFieldsFromDefinitionLocked`进行初始化。

![](https://bbs.kanxue.com/upload/attach/202205/948449_T7TWNAZYE2UX3J8.png)

- 找到`SetupFieldsFromDefinitionLocked`，在这个函数中就能找到具体的初始化过程了，可以看到其`offset`是通过`MetadataCache::GetFieldOffsetFromIndexLocked`计算而出的。

![](https://bbs.kanxue.com/upload/attach/202205/948449_95EGET8CPJ3BDYW.png)

- 继续找到`MetadataCache::GetFieldOffsetFromIndexLocked`，该函数进一步调用了`GlobalMetadata::GetFieldOffset`。

![](https://bbs.kanxue.com/upload/attach/202205/948449_AY7PV9EBTJA73FY.png)

- 最后一步，`GlobalMetadata::GetFieldOffset`。该函数通过`typeIndex`和`fieldIndexInType`进行查表。`typeIndex`可以通过查当前`type`的`Il2CppTypeDefinition`在`global-metadata.dat`的序号获得，而`fieldIndexInType`在遍历`type`的`field`可以直接获得。

![](https://bbs.kanxue.com/upload/attach/202205/948449_YNKDWUCAKWCWBJS.png)

- 获取`typeIndex`代码如下

```
static TypeDefinitionIndex GetIndexForTypeDefinitionInternal(const Il2CppTypeDefinition* typeDefinition)
{
const Il2CppTypeDefinition* typeDefinitions=(const Il2CppTypeDefinition*)(ptr+((Il2CppGlobalMetadataHeader*)ptr)->typeDefinitionsOffset);
ptrdiff_t index=typeDefinition-typeDefinitions;
return (TypeDefinitionIndex)index;
}
```

- 索引信息都能获取了，现在需要找到的是`s_Il2CppMetadataRegistration->fieldOffsets`，也就是要定位到`s_Il2CppMetadataRegistration`。该变量的类型为`Il2CppMetadataRegistration`

![](https://bbs.kanxue.com/upload/attach/202205/948449_6CJTBG7NAZNMXMH.png)

- 该变量存在于`GameAssemly.dll`，也是由il2cpp aot编译生成注册到il2cpp runtime的，而刚好之前发现`codeRegistration`注册时一并注册了`metadataRegistration`。

![](https://bbs.kanxue.com/upload/attach/202206/948449_KFPRQD957UH5CES.png)

- 于是我们根据上文找到的`codeRegistration`进行交叉引用，成功定位到`metadataRegistration`。

![](https://bbs.kanxue.com/upload/attach/202205/948449_7ZCY6B7XGH8DDFQ.png)

![](https://bbs.kanxue.com/upload/attach/202205/948449_3U6APXX4QDNWZWC.png)

- 所以只需要根据这个变量直接用`typeIndex`和`fieldIndexInType`查表即可。

```
uint32_t GetFieldOffset(TypeDefinitionIndex typeIndex,uint32_t index)
{
return MetadataRegistration->fieldOffsets[typeIndex][index];
}
```

# 0x5 完整代码

- 由于使用了大量结构体，不方便直接贴上来，请下载附件，同时还有对应的测试文件和输入。

- 代码不够完善，无法用于生产，仅用于学习。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)

上传的附件：

- [src.zip](javascript:void(0);)

（6.80MB，175次下载）
