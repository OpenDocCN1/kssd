
重点关注`text`部分，想要研究文件的实际内容。打开`.text`部分，发现全为0。接下来查看文件是否被映射。PE文件呈现这种格式有两种原因。一是磁盘上的文件，这是PE文件的标准格式并且所有的头部信息将匹配原始偏移量，因此原始偏移量意味着实际文件；二是当一个磁盘上的PE文件被映射到内存中并且被运行时，由原始地址转移为虚拟地址，分割了PE文件并将其注入到内存中，因为此时原始的地址指向内存中的地址，因此此时的原始格式全为0

![image-20211231110727043](https://bbs.kanxue.com/upload/attach/202112/942128_9E7WYXZPQC9V4PR.png)

### (6) 使用PE-bear打开dump出的PE文件即`difxapi_00140000.bin`，首先查看`Section Hdrs`发现两个数据表，一个重定位表，一个包含代码的文本部分

![image-20211231110653381](https://bbs.kanxue.com/upload/attach/202112/942128_JDV62ZCHPDJ3DJU.png)

希望在该部分看到很多二进制代码，意味着为可执行的PE文件。一个简单且直接的方法是修改PE头部的原始地址去匹配虚拟偏移地址，这可能会导致该PE文件变的很大，但是我们仅仅想要在IDA中进行逆向分析，并不会去执行该PE文件，因而对分析的影响很小

![image-20211231110841585](https://bbs.kanxue.com/upload/attach/202112/942128_D9M6ZS77K959BKH.png)

接下来在`Optional Hdr`部分修改PE文件的基地址即PE文件实际加载到内存中的部分为从内存中拉取出来时的地址，即`140000`

![image-20211231110940815](https://bbs.kanxue.com/upload/attach/202112/942128_NSA5HCFW36P8VD7.png)

查看`.text`的内容，发现`.text`区段非全为0，但是打开`Imports`表，发现依然有些问题

![image-20211231111027002](https://bbs.kanxue.com/upload/attach/202112/942128_W8Q2CFU7NA8EME8.png)

先暂时将修改后的文件中`.text`部分另存为`difxapi_00140000_align.bin`

![image-20211231111055015](https://bbs.kanxue.com/upload/attach/202112/942128_PAW4U4K3U42B7Y3.png)

### (7) 从内存中转储的 PE 重建导入地址表，使用自动化工具`PyIATRebuild(185K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6b7f1I4S2j5Y4y4Q4x3V1k6b7P5f1W2m8g2q4u0W2j5Y4g2A6L8r3c8Q4x3U0V1`.

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `环境python2.7.8`  `pip install winappdbg`  `pip install elfesteem`  `安装VCForPython27.msi(python2.7的C语言编译器)`  `【https://github.com/reider-roque/sulley-win-installer/VCForPython27.msi】`  `pip install distorm3` |

![image-20211230102138859](https://bbs.kanxue.com/upload/attach/202112/942128_QXQW7P23TQGP6NB.png)

参数均为10进制

1) `pid`

![image-20211231112446119](https://bbs.kanxue.com/upload/attach/202112/942128_VBQDRTZGA9Y83FS.png)

2) 基地址即默认文件名中的`140000`，十六进制基地址转化成十进制基地址为`1310720`

![image-20211231111752333](https://bbs.kanxue.com/upload/attach/202112/942128_X9GKC4TPSZWKUYH.png)

3) `OEP=Entry Point+基地址`，并转换成10进制

![image-20211231112619352](https://bbs.kanxue.com/upload/attach/202112/942128_JR2PAP3QC9QXZS7.png)

重建地址表，虽然有个警告信息，但是不影响

![image-20211231112253579](https://bbs.kanxue.com/upload/attach/202112/942128_EQ7F2ZPPQGHQEXH.png)

函数对齐了字符串的引用，所有的导入表都可以显现。表明已正确对其了所有的内容

## 四、总结

通过`Proexp`寻找病毒停止状态时调用的`API`，通过该`API`可以发现病毒在内存中脱壳时的状态。我们发现该病毒创建了一个子进程，因此给`createprocessInternalW`下断点，接下来在内存布局中寻找进程中特殊的可执行区域，然后得到一个`PE`文件，`dump`下该`PE`文件，对该`PE`文件进行反映射，之后使用`import`重建表来重建`import`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-12-31 13:11

被kanxue编辑，原因：
