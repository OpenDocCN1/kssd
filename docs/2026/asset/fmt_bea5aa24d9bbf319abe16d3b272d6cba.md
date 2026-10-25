
```
bp kernelbase!sleepex ".printf \"Application tried to sleep: %u seconds...\",poi(esp+4)/1000;.echo;ed esp+4 0x0;g"
//创建文件行为过滤，一般文件创建依赖于CreateFileA/W API函数。但是该函数不止有创建文件行为，所以需要通过API的第5个参数进行过滤减少打开文件误报
//0n3代表打开文件行为属性，过滤成功后打印第二个参数指向的字符串，不支持%s格式化字符串只能通过%mu进行传递
bp kernelbase!CreateFileW ".if(poi(esp+0x14) != 0n3){.printf \"Creating File: %mu\",poi(esp+0x4);.echo};g"
bp kernelbase!CreateFileA ".if(poi(esp+0x14) != 0n3){.printf \"Creating File: %ma\",poi(esp+0x4);.echo};g"
//过滤写文件行为
bp kernelbase!WriteFile ".printf \"Dumping file contents from 0x%p as ASCII: %ma\",poi(esp+0x8),poi(esp+0x8);.echo;g"
//过滤删除文件行为
bp kernel32!DeleteFileW ".printf \"Deleting File: %mu\",poi(esp+4);.echo;g"
bp kernel32!DeleteFileA ".printf \"Deleting File: %ma\",poi(esp+4);.echo;g"
//过滤移动文件行为
bp kernel32!MoveFileExW ".printf \"File moved.\";.echo;.printf \"From: %mu\",poi(esp+0x4);.echo;.printf \"To: %mu\",poi(esp+0x8);.echo;g"
bp kernel32!MoveFileExA ".printf \"File moved.\";.echo;.printf \"From: %ma\",poi(esp+0x4);.echo;.printf \"To: %ma\",poi(esp+0x8);.echo;g"
//过滤拷贝文件行为
bp kernel32!CopyFileW ".printf \" Copying file: \";.echo;.printf \"\tFrom: %mu\",poi(esp+0x4);.echo;.printf \"\tTo: %mu\",poi(esp+0x8);.echo;g"
bp kernel32!CopyFileA ".printf \" Copying file: \";.echo;.printf \"\tFrom: %ma\",poi(esp+0x4);.echo;.printf \"\tTo: %ma\",poi(esp+0x8);.echo;g"
//过滤创建注册表子键行为
bp kernel32!RegCreateKeyExA ".printf \"Creating RegKey: %ma\",poi(esp+0x8);.echo;g"
bp kernel32!RegCreateKeyExW ".printf \"Creating RegKey: %mu\",poi(esp+0x8);.echo;g"
//打开注册表键值行为
bp kernel32!RegOpenKeyExA ".printf \"Accessed RegKey: %ma\",poi(esp+0x8);.echo;g"
bp kernel32!RegOpenKeyExW ".printf \"Accessed RegKey: %mu\",poi(esp+0x8);.echo;g"
//过滤键值请求行为
bp kernel32!RegQueryValueExA ".printf \"\tAccessed RegValue: %ma\",poi(esp+0x8);.echo;g"
bp kernel32!RegQueryValueExW ".printf \"\tAccessed RegValue: %mu\",poi(esp+0x8);.echo;g"
//过滤设置注册表键值行为
bp kernel32!RegSetValueExA ".printf \"Setting RegKey %ma to value: %ma\",poi(esp+0x8),poi(esp+0x14);.echo;g"
bp kernel32!RegSetValueExW ".printf \"Setting RegKey %mu to value: %mu\",poi(esp+0x8),poi(esp+0x14);.echo;g"
//过滤创建服务行为
bp advapi32!CreateServiceA ".printf \"Creating Service: \";.echo;.printf \"\tService Name: %ma\",poi(esp+0x4);.echo;.printf \"\tDisplay Name: %ma\",poi(esp+0x8);.echo;g"
bp advapi32!CreateServiceW ".printf \"Creating Service: \";.echo;.printf \"\tService Name: %mu\",poi(esp+0x4);.echo;.printf \"\tDisplay Name: %mu\",poi(esp+0x8);.echo;g"
//过滤创建进程行为
bp kernel32!CreateProcessW ".printf \"Creating Process: %mu\",poi(esp+0x8);.echo;g"
bp kernel32!CreateProcessA ".printf \"Creating Process: %ma\",poi(esp+0x8);.echo;g"
//过滤shell行为以及命令行
bp shell32!shellexecuteW ".printf \"Running Command:\";.echo;.printf \"\tOperation: %mu\",poi(esp+0x8);.echo;.printf \"\tTarget: %mu\",poi(esp+0xC);.echo;.printf \"\tParams: %mu\",poi(esp+0x10);.echo;g"
bp shell32!shellexecuteA ".printf \"Running Command:\";.echo;.printf \"\tOperation: %ma\",poi(esp+0x8);.echo;.printf \"\tTarget: %ma\",poi(esp+0xC);.echo;.printf \"\tParams: %ma\",poi(esp+0x10);.echo;g"
bp shell32!shellExecuteExW ".printf \"Running Command:\";.echo;.printf \"\tOperation: %mu\",poi(poi(esp+0x4)+0xC);.echo;.printf \"\tTarget: %mu\",poi(poi(esp+0x4)+0x10);.echo;.printf \"\tParams: %mu\",poi(poi(esp+0x4)+0x14);.echo;g"
bp shell32!shellExecuteExA ".printf \"Running Command:\";.echo;.printf \"\tOperation: %ma\",poi(poi(esp+0x4)+0xC);.echo;.printf \"\tTarget: %ma\",poi(poi(esp+0x4)+0x10);.echo;.printf \"\tParams: %ma\",poi(poi(esp+0x4)+0x14);.echo;g"
//过滤模块加载行为
bp kernel32!LoadLibraryA ".printf \"Loading LibraryA: %ma\",poi(esp+0x4);.echo;g"
bp kernel32!LoadLibraryW ".printf \"Loading LibraryW: %mu\",poi(esp+0x4);.echo;g"
//过滤动态获取API函数行为
bp kernel32!GetProcAddress ".printf \"\t Looking up function: %ma\",poi(esp+0x8);.echo;g"
```

文件写入操作，可以看到写入的关键属性。

![图片描述](upload/attach/202011/784955_FN3K2SAD4VUB75A.png)

将文件dump出来发现为html勒索信内容，该勒索信详细的介绍了受害者是被何种算法加密以及解密链接的下载以及加密公钥等内容。

![图片描述](upload/attach/202011/784955_XQ2DH663NP29QJR.png)

勒索信内容如下。

![图片描述](upload/attach/202011/784955_G2WQ6BXX62JPE3U.png)

动态方式获取API。

![图片描述](upload/attach/202011/784955_ZDVZZ34NU2WHFNR.png)

根据windbg动态跟踪API断点，可以发现病毒调用了微软官方加密库使用RSA+AES加密算法对文件进行加密。

![图片描述](upload/attach/202011/784955_SDJFQT5YGUU27RF.png)

移动方式覆盖原文件，完成加密。

![图片描述](upload/attach/202011/784955_SNFP2WC4NXR27RF.png)

创建注册表子键并设置值为locky。

![图片描述](upload/attach/202011/784955_RBWX67HMV2GT2YK.png)

使用命令行显示勒索信息。

![图片描述](upload/attach/202011/784955_VDMVBVF6GAJXJM7.png)

![图片描述](upload/attach/202011/784955_2BB4UZT5W6UA4WC.png)

还有一些其他行为感兴趣的可以自己跟踪研究，这里不做赘述。

参考资料：<a href="elink@caaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3E0W2P5i4y4A6k6$3S2@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6@1k6h3y4Z5i4K6u0r3L8Y4N6$3M7#2)9J5k6h3g2F1N6s2u0&6i4K6u0W2K9s2c8E0L8q4)9J5c8U0t1H3x3U0m8Q4x3V1j5H3y4#2)9J5c8U0t1%4i4K6u0r3k6r3g2T1N6h3N6Y4K9h3&6Y4i4K6g2X3L8h3q4D9N6$3q4J5k6i4N6A6i4K6u0V1K9r3D9#2N6g2)9J5k6h3S2@1L8h3I4Q4x3@1k6H3j5i4u0@1L8X3g2J5M7X3g2X3i4K6y4p5e0#2y4y4i4K6u0V1g2q4N6Q4x3U0k6K6k6U0t1K6y4U0x3%4x3U0j5&6x3W2)9K6c8o6p5`.">windbg分析locky勒索病毒</a>

最近OD用腻了，想要换换口味于是有了这篇文章。这篇文章只是记录了分析过程中windbg的实用技巧，不会细致完整的将病毒跟踪分析。感兴趣的朋友可以自己抽时间跟踪学习。

locky勒索病毒MD5: 5DA38A8EF49834A13219BEDA15A1303C

首先查壳，发现病毒样本并没有加壳。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_YV4KHWX2VTRC53G.png)

查看资源，发现可疑数据。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_XVQJ3DQE4ZHRHD7.png)

查看各个区段熵值，只有代码段有较大波动可能代码段存在加密，具体需要调试行为确定。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_46TBJTSW76FKXQU.png)

加载病毒到内存，确定模块基址。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_RMUMEM2G6XYU5A7.png)

观察病毒模块详细信息，可以看到病毒具体编译的时间戳为16年。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_X8RVN83N8JGAVQY.png)

获取病毒文件DOS头相对PE头的偏移值，该值记录在`e_lfanew`字段中。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_4D3R9NDRESXP4QH.png)

计算验证是否为NT头的起始位置，可以看到PE字符串标识证明`0x400080`为PE头。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_XKG7PK4BG5WHVVT.png)

解析NT头结构体信息，因为我是使用的win7 x64位查看的32位进程所以需要使用`_IMAGE_NT_HEADERS64`解析结构体。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_GMYHMS7JCFWF2H4.png)

根据结构体树，我们可以清晰的看到拓展头中的入口点RVA为`0x2af8`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_U2A36542792CC98.png)

windbg虽然不能像OD一样在程序载入的时候自动在入口点设置断点，但是却为我们提供了一个名为`$exentry`的伪寄存器，它可以帮我们计算入口点的VA地址。直接利用该伪寄存器在入口点下断并按下F5让寄存器跑起来就可以断在OEP。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_EGCXQTNQV4NNURR.png)

windbg还给我们提供了拓展指令`!dh`用来详细解析PE文件。例如我们可以使用`!dh -s`详细查看各个区段内容。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_HGYAVC32GP2PCDC.png)

进入入口点代码，发现病毒通过计算差值获取全局变量的数据地址。固定差值为`0x9864e321`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_89R5N53UH5VVR5N.png)

正式分析前我们需要考虑到有的病毒样本中存在睡眠方式反沙箱，我们可以设置条件断点，当触发睡眠事件时打印出要睡眠的时间，并且将睡眠时间修改为0。

```
bp kernelbase!sleepex ".printf \"Application tried to sleep: %u seconds...\",poi(esp+4)/1000;.echo;ed esp+4 0x0;g"
```

设置完断点可以让调试器运行起来可以观察到反沙箱睡眠时间。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_UUK9MJVJQ2WC3W7.png)

病毒首先调用`GetModuleHandleW()`函数获取`kernel32.dll`模块在内存中的基址。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_5KRUT7PPKGV6MR9.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_PD8CV53WNFBF88A.png)

查看返回值与`kernel32.dll`中模块基址相同。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_KCH8C348SFMKXK9.png)

病毒会在栈中解密以下数据，然后调用`call`指令跳转到该地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_4KYCJDY46DUPFYM.png)

解密后的地址会申请一块虚拟地址，我这里分配的随机地址为`0x3d0000`。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_DEFPVME9ZHMSP7T.png)

以每次4字节加减异或解密，并将解密后的数据用`pop`指令弹入`esi`寄存器指向的虚拟内存中。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_BT6TFFYQGY5ZYMJ.png)

设置条件断点，让程序解密完成后自动断住。

```
bp 004052b5 ".if(edx==0x688){};.else{gc;}"
```

![图片描述](https://bbs.kanxue.com/upload/attach/202011/784955_NSTTUXKDYZQ2NME.png)

可以看到中断后的解密数据如下。
