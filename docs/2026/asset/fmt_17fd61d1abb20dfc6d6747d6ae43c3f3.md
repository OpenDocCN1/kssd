
```c
void MyMapDll(byte* DllData, PVOID* BaseAddress, PSIZE_T ViewSize) {
    //获取基地址
    __int64 DllMemoryBase = (__int64)&DllData[0];
    PIMAGE_DOS_HEADER DOSHeader = (PIMAGE_DOS_HEADER)DllMemoryBase;
    //读出文件DOS头
    if (DOSHeader->e_magic == IMAGE_DOS_SIGNATURE) {
        //检测MZ头
        PIMAGE_NT_HEADERS NTHeader = (PIMAGE_NT_HEADERS)(DllMemoryBase + DOSHeader->e_lfanew);
        if (NTHeader->Signature == IMAGE_NT_SIGNATURE) {
            //检测PE头
            PVOID l_location, l_lpBaseAddress;
            PIMAGE_SECTION_HEADER Sectionheaders;
            __int64 SectionOffset;
            SectionOffset = DOSHeader->e_lfanew + sizeof(IMAGE_NT_HEADERS);//IMAGE_SECTION_HEADER在内存中的偏移
            *BaseAddress = VirtualAlloc(NULL, NTHeader->OptionalHeader.SizeOfImage, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
            //分配一块内存供给Dll使用
            *ViewSize = NTHeader->OptionalHeader.SizeOfImage;
            //下面开始根据SECTION信息将Dll数据逐步映射到内存
            int l_count = NTHeader->FileHeader.NumberOfSections;
            //拷贝DOS头
            RtlMoveMemory(*BaseAddress, &DllData[0], SectionOffset + sizeof(IMAGE_SECTION_HEADER) * l_count);
            //设置头部的保护
            VirtualProtect(&DllData[0], 0x1000, PAGE_READONLY, 0);
            for (int i = 0; i < l_count; i++) {
                Sectionheaders = (PIMAGE_SECTION_HEADER)(DllMemoryBase + SectionOffset);
                if (Sectionheaders->PointerToRawData != 0) {
                    //如果Sectionheaders->PointerToRawData不为0，表示该段是有数据的，需要拷贝。
                    l_location = (PVOID)(DllMemoryBase + Sectionheaders->PointerToRawData);
                    l_lpBaseAddress = (PVOID)((__int64)*BaseAddress + Sectionheaders->VirtualAddress);
                    //将段数据拷贝到分配的内存中
                    RtlMoveMemory(l_lpBaseAddress, l_location, Sectionheaders->SizeOfRawData);
                    //设置保护
                    VirtualProtect(l_lpBaseAddress, Sectionheaders->Misc.VirtualSize, GetProtect(Sectionheaders->Characteristics), 0);
                }
                SectionOffset += sizeof(IMAGE_SECTION_HEADER); //读取下一个SECTION
            }
        }
    }
}
```

- 其实说白了就是将Dll的SECTION全部加载到内存就可以了
- 拷贝完真实的Dll之后，只需将“映射”到的地址和大小数据给回`NtMapViewOfSection`即可。
- 什么IAT修复、TLS、异常之类东西，就全部交回给系统原生的Dll加载程序处理就好了。

# 一些闲话：

- 为什么使用这样的方法加载Dll呢？
- 因为这样可以提高内存Dll的稳定性（比较我们只处理了加载数据部分，其他部分全是系统处理的）
- 本方法实现的`MemoryDll`可以基本兼容系统的相关Library命令（比如`GetProcAddress`）
- 同时因为没有调用`NtMapViewOfSection`，可以绕过监听到Dll加载的方法（注意Dll文件的PE头、PEB中的双链并没有抹掉）
- 不过话说回来，C++中这样搞的意义确实不是很大，可能对我来说主要的价值是学习价值吧。
- 提示：如果你在其他论坛看到类似代码，那也可能我发的。

## 效果：

- 测试前记得将源码中的`CDll_x64.dll`和`CDll_x86.dll`编译出来（也可以换其他Dll测试，经过测试VMP加壳程序不行，其他都还好）
- x64测试：![](https://bbs.kanxue.com/upload/attach/202210/962506_XJFYAFPRMC5R77P.jpg)
- x86测试：![](https://bbs.kanxue.com/upload/attach/202210/962506_Y437MS5A8WUCCEU.jpg)

# 源码：

- 已经上传Github：[MemoryDll-DllRedirect](https://bbs.kanxue.com/elink@f97K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8i4K6u0V1e0X3!0J5L8h3q4D9i4K6u0V1g2i4y4W2M7W2)9J5c8V1#2W2L8h3!0J5P5f1c8D9L8q4)9J5k6p5c8D9L8q4u0W2k6r3W2J5k6h3y4@1)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)
