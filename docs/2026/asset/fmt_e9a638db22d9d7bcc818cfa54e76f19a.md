
```c
if (size == 8958236) {
    u4 classDefSize = dex_file.pHeader->classDefsSize;
    u4 classDefsOff = dex_file.pHeader->classDefsOff;

    __android_log_print(5, "hookso", "base:%p", (void *) base);
    __android_log_print(5, "hookso", "classDefSize:%ld   classDefsOff:%p", classDefSize, (void *) classDefsOff);

    u4 typeIdsOff = dex_file.pHeader->typeIdsOff;
    u4 stringIdsOff = dex_file.pHeader->stringIdsOff;

    __android_log_print(5, "hookso", "typeIdsOff:%p   stringIdsOff:%p", (void *) typeIdsOff, (void *) stringIdsOff);

    int i;
    for (i = 0; i < classDefSize; i++) {
        long currClassAddr = (long) classDefsOff + i * 32 + (long) base;
        __android_log_print(5, "hookso", "currClass ptr:%p", (void *) currClassAddr);
        int *idx = (int *) (currClassAddr);
        __android_log_print(5, "hookso", "currClassIdx:%i", *idx);

        int tmpIdx = *idx;
        long currTypeIdAddr = ((long) typeIdsOff + 4 * tmpIdx + (long) base);
        int *currTypeIdx = (int *) currTypeIdAddr;
        __android_log_print(5, "hookso", "currTypeIdx:%ld", *currTypeIdx);

        tmpIdx = *currTypeIdx;
        long currStringOffAddr = ((long) stringIdsOff + 4 * tmpIdx + (long) base);
        int *currStringOff = (int *) currStringOffAddr;
        __android_log_print(5, "hookso", "currStringOff:%ld", *currStringOff);

        tmpIdx = *currStringOff;
        long off = (long) base + tmpIdx;
        __android_log_print(5, "hookso", "string off:%p", (void *) off);

        const uint8_t *strPtr = (uint8_t *) off;
        DecodeUnsignedLeb128(&strPtr);
        char *classname = (char *) strPtr;

        __android_log_print(5, "hookso", "classname:%s", classname);
    }
}
```

在这里我们可以一层一层的不断获取不断遍历，就可以拿到我们想要的`classname`

![图片描述](https://bbs.kanxue.com/upload/attach/202011/777497_JHQTP55FZQVMBKG.png)

拿到`classname`后我们就可以用这个去遍历`loadClass`后再dump dex即可。

可以看到这个题目最后想要说的就是自定义`classloader`如何进行函数抽取还原，那么这里我们需要熟悉`Dexfile`的文件结构以及了解`classloader`如何使用

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [2.3测试.7z.001](javascript:void(0);)（6.68MB，27次下载）
- [2.3测试.7z.002](javascript:void(0);)（6.68MB，27次下载）
- [2.3测试.7z.003](javascript:void(0);)（6.68MB，25次下载）
- [2.3测试.7z.004](javascript:void(0);)（6.68MB，25次下载）
- [2.3测试.7z.005](javascript:void(0);)（3.14MB，24次下载）
