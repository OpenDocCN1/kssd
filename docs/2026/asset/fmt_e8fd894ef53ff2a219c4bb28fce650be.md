
```
for (i = (FOAEntry - RSRC); i > 0; i--) { //向前遍历整个资源节
    if (prsrc[i] == 0x0 || prsrc[i] == 0x47) //优化速度
        if ((prsrc[i] == 0x0 && prsrc[i-1] == 0x0 && prsrc[i-2] == 0x0 && prsrc[i-3] == 0x0 && prsrc[i-4] == 0x0 && prsrc[i-5] == 0x0) || //连续6个0
            (prsrc[i] == 0x47 && prsrc[i-1] == 0x4E && prsrc[i-2] == 0x49 && prsrc[i-3] == 0x44 && prsrc[i-4] == 0x44 && prsrc[i-5] == 0x41 && prsrc[i-6] == 0x50)) { //PADDINGX
            printf("\n匹配到原始资源末尾:%x", i); //相对于资源段开始的位置偏移,即是资源段实际大小
            tail = i;
            break;
        }
}
printf("\n向上物理大小矫正:%x", ((tail/0x200)+1)*0x200);
tail = ((tail/0x200)+1) * 0x200;
DWORD virussize = RSRCSIZE - tail; //大约是7000H
printf("\n虚拟大小矫正:%x", ((virussize/0x1000)+1)*0x1000);
pSectionHeader--;
pSectionHeader->SizeOfRawData = tail; //这个必须是200h的倍数
pSectionHeader->Misc.VirtualSize = tail; //union结构体不能用->
//改资源段的物理大小
DWORD VIRUSSIZE = ((virussize / 0x1000) + 1) * 0x1000;
pOptionalHeader->SizeOfImage = pOptionalHeader->SizeOfImage - VIRUSSIZE; //这个必须是1000h的倍数

handle = CreateFile(L"C:\\cleaner.exe.v", GENERIC_READ | GENERIC_WRITE, 0x3, 0, CREATE_NEW, FILE_ATTRIBUTE_NORMAL, 0);
WriteFile(handle, pFile, dwFileSize - virussize, &haveread, 0); //去除掉
```

删除`.rsrc`资源段并修复信息。原始资源段如果不足`0x200`字节对齐的话，都会在区段末尾有`PADDINGX`的字符，搜索到这种字符直接剪切。或者搜索到固定的6个零字节，那么也认为是原始资源段的结尾（这是不严密的，但是此变种感染附加的代码是纯汇编而写，不会存在函数间隙，的确没有发现过多的零字节，这样才能作为资源段结尾特征，其他感染性病毒要看情况而定）。

最后根据对齐方式校正文件大小（一般都是文件大小`0x200`对齐，镜像内存大小`0x1000`对齐），修复`SizeOfImage`大小，整个修复过程基本完成。

可见感染型病毒的修复过程是具有一定挑战性的，因为很难保证自己的一些判断依据（如入口点寻找的特征码、附加病毒数据的定位）是严格正确又严密的。如果这些判断标准比较严格，那么很有可能会减小用户文件的大小甚至造成原文件损坏，相反标准太低可能会造成部分被感染文件未被修复，从而造成反弹重复感染（客户现场见太多了）。彻底查杀并且完美查杀一种感染型病毒需要对病毒有非常深入的了解，才能设计出完美独特的修复方案，做到百无一失。

最后附上源代码，供大家学习交流。共享看雪论坛，共建网络安全家园。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-9-22 11:14 被绝望的皮卡丘编辑，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`VirutRemoveTool.cpp`](javascript:void(0);)（8.86kb，30次下载）
