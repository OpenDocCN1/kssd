
```
void buildReloadtionTable(DWORD baseAddress, char* pRelateTable) {
    // 1. 位移+基值，变化后拿到offset
    vector<MyReloadtion> reloadtionTable;
    MyReloadtion reloadtion;

    int index = 0;
    int size = 0;
    // 记录上一个计算出来的重定位表项的虚拟内存地址
    DWORD lastVirtualAddress = NULL;
    // 为第一个区块内存地址-4
    DWORD pReload = baseAddress;
    while (true)
    {
        // 1.拿取位移表一个字节
        WORD relate = (BYTE)*pRelateTable;
        pRelateTable++;
        if (relate == 0)
        {
            reloadtion.sizeOfBlock = calSizeofBlock(size);
            reloadtionTable.push_back(reloadtion);
            break;
        }
        // 2. 判断拿到的字节是否大于EF
        if (relate > 0xEF)
        {
            relate = *pRelateTable;
            BYTE hightRelate = *(pRelateTable + 1);
            BYTE lowRelate = relate;
            relate = ((hightRelate << 8) | lowRelate);
            pRelateTable += 2;
        }
        pReload += relate;

        // 3. 位运算获取offset
        DWORD virtualAddress = pReload & 0xF000;
        WORD typeOffset = pReload & 0xFFF | 0x3000;

        if (lastVirtualAddress == NULL)
        {
            reloadtion = MyReloadtion();
            reloadtion.virtualAddress = virtualAddress;
        }
        else if(lastVirtualAddress != virtualAddress)
        {
            reloadtion.sizeOfBlock = calSizeofBlock(size);
            reloadtionTable.push_back(reloadtion);

            size = 0;
            reloadtion = MyReloadtion();
            reloadtion.virtualAddress = virtualAddress;
        }
        reloadtion.typeOffset.push_back(typeOffset);
        size++;
        lastVirtualAddress = virtualAddress;
    }
    // 保存到文件
    saveToFile(reloadtionTable);
}
```

1. 将新的重定位表加到脱壳的程序里:

    整体思路：

    4.1 为程序添加一个新的区块（使用工具如`PEStudy`）

    4.2 将重定位表复制过去(`HEX WORKSHOP`)

    4.3 修改PE文件的重定位表指向(`LordPE`)

![](https://bbs.kanxue.com/upload/attach/202011/899080_9K3ACS8B7WQ6KGE.png)

![](https://bbs.kanxue.com/upload/attach/202011/899080_JBM7RB73PYGQQSE.png)

![](https://bbs.kanxue.com/upload/attach/202011/899080_8WFW64RMFWUBRTW.png)

# 遗憾收场

那一天显得特别宁静，我如往常一样坐在那里。“终于做出来了...”我轻轻的叹了口气，神态略显轻松，滑动着鼠标的滚轮。看着眼前快速滑动的代码行，心中百感交集。想想当初零行的代码到现在上百行，对于刚入门WIN32以及C++来说确实是不容易。

再用了几次写的代码修复程序后，心中却开始感到了不满。“为什么我每次都得自己找位移表，为什么我每次都得用工具新建区块，为什么每次都要我修改重定位表的`rva`，为什么我的代码不能一次实现呢？”

带着这个遗憾，我写下了这篇文章，不为别的，只为和大家分享。

# 那段汇编的循环代码

```
31 C0 8A 07 47 09 C0 74 22 3C EF 77 11 01 C3 8B 03 86 C4 C1 C0 10 86 C4 01 F0 89 03 EB E2 24 0F C1 E0 10 66 8B 07 83 C7 02 EB E2
```

大家如果想探索文章提到的`upx`还原重定位表的部分，可以直接在调试器中用上面数据搜索直接定位到相关代码。

# 关于代码使用

将exe放到加了`upx`壳的程序的目录下，将提取出来的位移表命名为`rva.bin`，加壳程序命名为`target.exe`，成功运行后，会生成一个叫`result.bin`的文件，这个就是重定位表。

本程序并没有做多版本的window测试，也没有做`upx`多版本测试，目前只在自家的win10下测试并通过，使用的`upx`版本为3.96w。

![](https://bbs.kanxue.com/upload/attach/202011/899080_MBM7KD93Z4J46ZS.png)

# 结语

这篇文章废话较多，作为技术文来说或许是不及格的。其实一直都想用一种叙事的手法来写，这可能是新的尝试呢，所以希望各位学友不嫌烦，能较为有趣的看完。

关于刚刚提到的遗憾，笔者也会着手尝试，让一切自动起来。

# 附件

[0f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3q4J5k6g2)9J5k6i4N6W2K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4N6f1N6%4x3#2f1V1b7K6](https://bbs.kanxue.com/elink@6b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3q4J5k6g2)9J5k6i4N6W2K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4N6f1N6%4x3#2f1V1b7K6)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
