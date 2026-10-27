
# [原创]PE文件解析 系列文章(二）

## 前言

- PE文件解析系列文章的第二篇
- 介绍区块头表和区块
- 解析出区段表，完成RVA转FOA的功能
- 解析出数据目录表各种表的位置和大小
- 源码放在附件

有错误或不清楚的地方还请您指正。

## 1.区块头表

- PE文件头与原始数据之间存在一个区块表，区块表包含了每个块在映像(内存)中的信息，分别指向不同的区块实体。

![图片](https://bbs.pediy.com/upload/tmp/825190_6GZQ5M6DNDNDAWJ.png)

- PE文件中所有节的属性都被定义在节表中，节表由一系列的`IMAGE_SECTION_HEADER`结构排列而成，每个结构用来描述一个节，结构的排列顺序和它们描述的节在文件中的排列顺序是一致的。全部有效结构的最后以一个空的`IMAGE_SECTION_HEADER`结构作为结束，所以节表中总的`IMAGE_SECTION_HEADER`结构数量等于节的数量加一。节表总是被存放在紧接在PE文件头的地方。另外，节表中`IMAGE_SECTION_HEADER`结构的总数总是由PE文件头`IMAGE_NT_HEADERS`结构中的`FileHeader.NumberOfSections`字段来指定的。

- `IMAGE_SECTION_HEADER`结构体包含了对应的区块的具体信息：位置、长度、属性。

```c
typedef struct _IMAGE_SECTION_HEADER 
{
       BYTE Name[IMAGE_SIZEOF_SHORT_NAME]; // 节表名称,如“.text”
        //IMAGE_SIZEOF_SHORT_NAME=8
        union
         {
                DWORD PhysicalAddress;      // 物理地址
                DWORD VirtualSize;          // 真实长度，这两个值是一个联合结构，可以使用其中的任何一个，一般是取后一个
        } Misc;
        DWORD VirtualAddress;               // 节区的 RVA 地址        
        DWORD SizeOfRawData;                // 在文件中对齐后的尺寸     
        DWORD PointerToRawData;             // 在文件中的偏移量        
        DWORD PointerToRelocations;         // 在OBJ文件中使用，重定位的偏移  
        DWORD PointerToLinenumbers;         // 行号表的偏移（供调试使用地)
        WORD NumberOfRelocations;           // 在OBJ文件中使用，重定位项数目
        WORD NumberOfLinenumbers;           // 行号表中行号的数目
        DWORD Characteristics;              // 节属性如可读，可写，可执行等
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

`Name`：这是一个由8位的ASCII码名，用来定义区块的名称。

`VirtualAddress`：区块的RVA。

`SizeOfRawData`：区块在磁盘文件中的占用大小200h。

`PointerToRawData`：文件中的偏移量。

`NumberOfRelocations`：在exe文件中无意义，在OBJ文件中是本块在重定位表中重定位数目。

用loadPE打开：

![图片](https://bbs.kanxue.com/upload/attach/202002/825190_DQDK568V6FXP7GZ.jpg)

代码实现区段头表的解析：

```cpp
    //通过NT头找到区段头首地址
    PIMAGE_SECTION_HEADER pSec = IMAGE_FIRST_SECTION(m_pNTHeader);

    for (int i = 0;i< m_pNTHeader->FileHeader.NumberOfSections;i++)
    {
        CHAR pName[9] = {};
        memcpy_s(pName,9,pSec[i].Name,8);
        m_strName = pName;

        m_strVO.Format(L"%p",pSec[i].VirtualAddress);
        m_strVS.Format(L"%p", pSec[i].Misc.VirtualSize);
        m_strRO.Format(L"%p", pSec[i].PointerToRawData);
        m_strRS.Format(L"%p", pSec[i].SizeOfRawData);
        m_strSig.Format(L"%p", pSec[i].Characteristics);

        m_SectionInfoList.AddItem(6, m_strName, m_strVO, m_strVS, m_strRO, m_strRS, m_strSig);
    }
```

## 2.区块

- PE文件至少要有两个区块：代码块、数据块。
- 常见区块的介绍：

`.text`：默认的代码区块，内容都是指令代码。

`.data`：默认的读写数据块，全局变量、静态变量一般放在这里。

`.rdata`：默认的只读数据块，一般很少用到。

`.idata`：包含外来的DLL数据及数据信息，也就是输入表，之后会讲到，通常情况下把他合并到`.rdata`中。

`.edata`：当创建一个用于输出数据的可执行文件时（输出表），数据会放在这里，通常情况下会被合并到`.text`或`.tdata`中。

`.rsrc`：资源块，包含一切图标菜单等。

（还有一些可参考《加密与解密》，不在这里列举了）。

- 区块的对齐：

区块的对齐有两种，一是磁盘当中的区块对齐，二是内存当中的区块对齐。磁盘当中的对齐值是200h，所以每个区块都应该是200h的倍数。

内存当中的对齐值为1000h，也就是4KB。

## 3.文件偏移与虚拟地址的转换

- 由于磁盘与内存当中的对齐值不一样，不免会带来地址的相互转换问题。要转换的RVA一定落在一个区段内，首先判断它落在哪个区段。然后减去这个区段的RVA再加上这个区段的文件偏移量，就可以得到要转换的FOA值。`Offect(转) = RVA(转) -RVA(区段）+Offect(区段)`。而这些关于区段的信息都保存在区段头表中。

具体找个例子实验一下：

用loadPE打开一个exe。随机选取一个RVA值如1100h。首先找出它所在的区段。发现所在的区段为`.text`区段。`.text`区段的RVA值为1000h，大小为110D1h，可以判断1100h落在了`.text`区段内。所以用1100h-1000h+600h = 700h，即为FOA。

![图片](https://bbs.kanxue.com/upload/attach/202002/825190_NFGTVKJNGJ2263Z.png)

验证一下：

![图片](https://bbs.kanxue.com/upload/attach/202002/825190_JZ5JW8ZAY7BXC4J.png)

写代码实现一下：

```cpp
    //循环查找
    for (int i=0; i < pNt->FileHeader.NumberOfSections;i++)
    {
        if (dwRva >= pSec[i].VirtualAddress && 
            dwRva <= pSec[i].VirtualAddress + pSec[i].SizeOfRawData)  //判断在哪个区段
        {
            return (dwRva - pSec[i].VirtualAddress + pSec[i].PointerToRawData);  //用公式进行计算
        }
    }
```

## 4.完整效果

- 实现RVA到VA和FOA的转换：

- 打印数据目录信息：

![图片](https://bbs.kanxue.com/upload/attach/201810/825190_49254P9DZASXME5.png)

- 打印区段相关信息：

![图片](https://bbs.kanxue.com/upload/attach/201810/825190_NGR26SRSTHFDMHR.png)

- 完整代码放到附件

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)

最后于 2018-10-17 08:08 被Jabez编辑，原因：上传的附件：

- [PE解析工具.rar](javascript:void(0);) （142.82kb，205次下载）
