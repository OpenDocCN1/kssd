
# 总结

壳作者做了两层加固保护，第一次加密了代码段中`JNI_OnLoad`相关的代码，原理比较简单，在`init`中将加密的代码解密就OK。第二次是在`JNI_OnLoad`中，加载真正的业务逻辑代码（`true_code`）, 新的`loadsegment`布局到内存后，用新的`symtab`、`symstr`、`hash`表内容覆盖壳的对应数据，这样系统对该`so`再次`dlopen`和`dlsym`就是在`true_code`查找相关函数了。另外两个重要操作是对`true_code`重定位，以及修正新的`symtab`数据，因为`linker`在`dlsym`的时候符号地址要加`load_bias`，所以这里提前对符号表内容加了新`mmap`地址和壳`load_bias`之间的差值。

# 思考

如果我们正向考虑实现这套加固系统，应该怎么做呢？

1. 需要提前编译好一个`libstub.so`，内容是`init_array`的解密函数和`JNI_OnLoad`的加载器代码，声明一些全局标记用来注入

2. 打包`so`的工具，对`libstub.so`和需要加固的`so`处理，把加密信息和`true_code`的地址信息注入到`libstub.so`中

如果要改进，把`JNI_OnLoad`的内容放在`init`中做，以实现更强的通用性，需要注意重定位时不能再用`dlopen`和`dlsym`了，需要借助`maps`表找函数地址，但这种方式会有一些兼容性问题。

这套系统应该可以“免汇编”实现，维护方便，向壳作者致敬！如果分析有错误和纰漏，请大家多多指教！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [libYSDK.decrypt.zip](javascript:void(0);)

  （49.45kb，190次下载）
