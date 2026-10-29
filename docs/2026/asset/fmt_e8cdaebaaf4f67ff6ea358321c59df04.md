
> `c67K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6t1K6x3e0p5%4x3R3`.`

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 | `void soinfo::CallFunction(const char* function_name UNUSED, linker_function_t function) {`<br>`if (function == NULL || reinterpret_cast<uintptr_t>(function) == static_cast<uintptr_t>(-1)) {`<br>`return;`<br>`}`<br><br>`TRACE("[ Calling %s @ %p for '%s' ]", function_name, function, name);`<br>`//在这里被调用了，其他没啥好说的`<br>`function();`<br>`TRACE("[ Done calling %s @ %p for '%s' ]", function_name, function, name);`<br><br>`// The function may have called dlopen(3) or dlclose(3), so we need to ensure our data structures`<br>`// are still writable. This happens with our debug malloc (see http://b/7941716).`<br>`set_soinfo_pool_protection(PROT_READ | PROT_WRITE);`<br>`}` |

至此，`Linker`就分析结束了。

## 总结

我们在最后说一个`Unidbg`细节的`bug`，但是现在已经被修复了，就是作为一个扩展吧。我们来看下面一段`Unidbg`加载`So`的代码。

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 | `if (elfFile.file_type == ElfFile.FT_DYN) {`<br>`// not executable`<br>`int init = dynamicStructure.getInit();`<br>`if (init != 0) {`<br>`initFunctionList.add(new LinuxInitFunction(load_base, soName, init));`<br>`// new LinuxInitFunction(load_base, soName, init).call(emulator);`<br>`}`<br><br>`int initArraySize = dynamicStructure.getInitArraySize();`<br>`int count = initArraySize / emulator.getPointerSize();`<br>`if (count > 0) {`<br>`Pointer pointer = UnidbgPointer.pointer(emulator, load_base + dynamicStructure.getInitArrayOffset());`<br>`if (pointer == null) {`<br>`throw new IllegalStateException("DT_INIT_ARRAY is null");`<br>`}`<br>`for (int i = 0; i < count; i++) {`<br>`Pointer func = pointer.getPointer((long) i * emulator.getPointerSize());`<br>`if (func != null) {`<br>`initFunctionList.add(new AbsoluteInitFunction(load_base, soName, ((UnidbgPointer) func).peer));`<br>`}`<br>`}`<br>`}`<br>`}` |

如果我们细心地阅读`Linker`的源码，就会发现`Unidbg`这里处理得是不恰当的。在本文的最后，我们看到了初始化函数的调用，是`DT_INIT`函数先被执行，后面再处理`DT_INIT_ARRAY`，而`Unidbg`这里就是将它们都添加到一个`List`，再一起调用。这样就会产生一个问题，在某些加壳的`So`中，它的`DT_INIT_ARRAY`是在`DT_INIT`函数执行之后，才会有值的（进行修复），所以按照`Unidbg`这个写法就无法执行`INIT_ARRAY`或部分`INIT_ARRAY`无法执行。处理方法也很简单，注释在上面了，只需要让`DT_INIT`先执行就可以了。

那么本篇文章对`Linker`的讲解就到这里了，如果您觉得有用，可以加个`VX`一起学习呀：roy5ue。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
