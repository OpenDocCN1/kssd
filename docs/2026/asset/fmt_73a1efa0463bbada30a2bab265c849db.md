
对于`0x000400`这个指令，首先`0x00`代表了`add Eb, Gb`。接着将`0x04`拆分为二进制则是`00 000 100`，此时`Reg/Opcode`为`000`，所以`Eb`就是`al`，`Mod`为`00`，`R/M`是`100`，此时就需要继续将后面的`0x00`作为`SIB`，将`SIB`拆分以后就是`00 000 000`，此时`Base`为`000`，`Scale`为`00`，`Index`为`000`，所以`Eb`的内容就是`[eax + eax]`，所以对这条指令解析的最终结果就就会是`add byte ptr ds:[eax + eax], al`。

而对于`0x000420`指令，由于此时`SIB`为`0x20`，拆成二进制以后是`00 100 000`，所以此时的`Eb`寄存器就是`eax`，那么这条指令的解析结果就是`add byte ptr ds:[eax], al`。

对于`Base`为`101`的情况下，需要指令继续解析就需要看白皮书的Table 2-3下面的NOTES

![](https://bbs.kanxue.com/upload/attach/202111/835440_EA56N9F3PJKDPPY.jpg)

> 当出现[\*]情况的时候，`Eb`的具体形式是要根据`Mod`的值来确定的：
>
> - `Mod`为`00`，则`Eb`为`Scale`与`Index`所代表的寄存器+32位的偏移
> - `Mod`为`01`，则`Eb`为`Scale`与`Index`代表的寄存器加+8位偏移+`[ebp]`
> - `Mod`为`10`，则`Eb`为`Scale`与`Index`所代表的寄存器+32位偏移+`[ebp]`

比如`0x00040578563412`，此时的指令格式是`add Eb, Gb`，`ModR/M`的值为`0x04`，对应二进制是`00 000 100`，`Reg/Opcode`为`000`，则`Gb`为`al`，而`Mod`为`00`，`R/M`为`100`，就需要将后一字节，也就是`0x05`作为`SIB`解析，`0x05`的二进制为`00 000 101`，此时`Scale`与`Index`表示为`eax`，而`Base`为`101`，而此时的`Mod`为`00`，所以`Eb`的形式就是`[eax+disp32]`，就需要继续读取4个字节作为偏移，那么最后的结果就是`add byte ptr ds:[eax + 0x12345678], al`。

而指令如果是`0x00840578563412`，此时的`ModR/M`就是`0x84`，对应二进制是`10 000 100`，`Reg/Opcode`为`000`，`Mod`为`10`，`R/M`为`100`，`SIB`为`0x05`，由上可以知道，此时`Scale`和`Index`表示为`eax`，而`Base`则是`101`，因为此时的`Mod`为`10`，所以`Eb`的形式应该是`[eax + disp32 + ebp]`，所以最终指令的解析结果就将是`add byte ptr ss:[ebp + eax + 0x12345678], al`。

因此可以得出结论，当查Table A-2的时候，操作数带有`Eb`，`Ev`，`Gb`这类的指令时候，就需要对`Mod/RM`继续解析，而`Mod/RM`字段又决定了是否有`SIB`，此时指令的长度是不可以只通过操作码来确定的。

# 五.操作码扩展

此时Table A-2的指令已经基本解析完成，除了像`0x80-0x84`这种带有`Grp`的指令

![](https://bbs.kanxue.com/upload/attach/202111/835440_S7ECHR7PDRWHVUN.jpg)

这类指令的操作数的格式是确定的，比如`0x80`对应的操作数的为`Eb, Ib`。但是并不知道操作码是怎么样的，此时的`ModR/M`字段中的`Reg/Opcode`就被用来识别指令的操作码，而要通过`Reg/Opcode`正确识别操作码，需要查看Intel白皮书1486页的Table A-6。

![](https://bbs.kanxue.com/upload/attach/202111/835440_B9VS2UP4UY8WUMG.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/835440_RJF4YTY3DNPFC6C.jpg)

这张表的作用就是根据`Mod/RM`字段中的`Reg/Opcode`字段来查询操作码，第一列，说明了`Opcode`的值为多少，而右边的八列则是不同的`Reg/Opcode`的值所对应的不同的操作码。

对于`0x80301F`这条指令，`0x80`说明了操作数为`Eb, Ib`，而操作码是未定的，继续解析值为`0x30`的`Mod/RM`，`0x30`对应的二进制为`00 110 000`，此时`Mod`为`00`，`R/M`为`000`，所以`Eb`的值就是`[eax]`，`Reg/Opcode`为`110`，根据Table A-6说明此时的操作码是`xor`，而剩下的`Ib`说明是一字节的立即数，就会将`1A`继续读取进来作为`Ib`，所以指令最后的解析结果是`xor byte ptr ds:[eax], 0x1F`。

# 六.扫尾知识

常见的指令基本都解析完了，在这本表皮书中最主要的的表是Table A-2，这张表是用来查询`Opcode`的，不同的`Opcode`有着不同的指令形式。前面有说过当`Opcode`为`0x0F`的时候，操作码至少有两个，需要查Table A-3才能解析指令。

而在Table A-3中可以知道，当第二个操作码是`0x38`或者`0x3A`的时候，此时的操作码就是3个

![](https://bbs.kanxue.com/upload/attach/202111/835440_AJFNGV8U7BQJKD4.jpg)

而要知道第三个操作码的作用就需要1481页的Table A-4

![](https://bbs.kanxue.com/upload/attach/202111/835440_SZYMSG5QSDSKUVZ.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/835440_6DQQEM5HJK58CZF.jpg)

以及1483页的Table A-5

![](https://bbs.kanxue.com/upload/attach/202111/835440_SVM2S894KQGEBMF.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/835440_C4CUGW2HQSKS83P.jpg)

由于这些指令并不常见，所以需要的时候查询就好了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2022-1-8 12:38

被1900编辑

，原因：

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

上传的附件：

- [Intel白皮书（卷二）.rar](javascript:void(0);)

  （4.94MB，45次下载）
