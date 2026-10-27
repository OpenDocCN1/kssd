
```c
tISH.VirtualAddress = SizeOfImage;           // 改地址应该为增加的一个空闲区   应该是先加区段，然后再改入口点 此地址作为最后的偏移量

tISH.PointerToRawData = st.st_size;              // 行号表中行号的数目

if(g_bHaveSetVSizeaAddress)
{
    tISH.Misc.VirtualSize = atoi(argv[2]);           // 4KB
    tISH.SizeOfRawData = atoi(argv[2]);
}

tISH.PointerToRelocations = 0;          // 在OBJ文件中使用， 重定位的偏移
tISH.PointerToLinenumbers = 0;          // 行号表的偏移(供调试用)
tISH.NumberOfRelocations = 0;           // 在OBJ文件中使用，重定位项目数
tISH.NumberOfLinenumbers = 0;
tISH.Characteristics = 0xE0000020;      // 块属性， 表示包含执行代码、可读写并可
```

加区段工具时，在设置`VirtualAddress`时，该地址实际上是该区段在内存中的偏移量，所以这个地址只要是内存映像大小即可。而文件偏移地址`PointerToRawData`则为文件的大小。

这也是为什么这两个值不一样的原因：尽管内存映像的各区块大概和磁盘文件是一一对应的，但实际上映射的地址还是有偏移的。所以有时当`VirtualAddress`和`PointerToRawData`相同时则可以执行，有时相同时反而出错，这是由于装载地址不一致导致的，也就是一致的地址那个地址值原来可能有数据了，若再装载就等于要覆盖那个地址的数据。既然在磁盘上区块是添加在文件尾，那在内存中当然也就是内存映像尾恩。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
