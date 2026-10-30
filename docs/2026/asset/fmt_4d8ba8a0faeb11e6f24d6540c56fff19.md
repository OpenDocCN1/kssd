
```c
case PARSER_PARSE_HEX_NUM_MSB:
    this->flag |= 0x8000;
    this->hex_num_byte = 0;
    state = PARSER_PARSE_HEX_NUM_LSB;
case PARSER_PARSE_HEX_NUM_LSB:
    // …
    byte = *(uint8_t*)data.pos;
    data.pos++;
    val = 0;
    if (byte – ‘0’ <= 9)
    {
        val = byte – 0x30;
    }
    else if (byte – ‘a’ <= 5)
    {
        val = byte – 0x57;
    }
    else if (byte – ‘A’ <= 5)
    {
        val = byte – 0x37;
    }
    this->hex_num_byte |= val << ((this->flag >> 0xF) << 2);
    this->flag = ((~this->flag ^ this->flag) & 0x7FFF) ^ ~this->flag;
    if (this->flag & 0x8000)
    {
        // …
        state = PARSER_BEGIN;
    }
    else
    {
        break;
    }
    break;
```

## 状态重置

看下`PARSER_PARSE_HEX_NUM_MSB`、`PARSER_PARSE_HEX_NUM_LSB`和`PARSER_PARSE_HEX_DATA`这三种状态，很容易发现其中的错误。即使它们使用不同的变量来存储解码后的十六进制值，但它们还是使用了相同的位来确定哪个半字节现在被解码了——高位（最高有效位或MSB）或低位（低位有效位或LSB）。并且，`PARSER_PARSE_HEX_NUM_MSB`状态总是将此位重置为MSB。

因此，通过改变`PARSER_PARSE_HEX_NUM_MSB`的状态，就可以使`PARSER_PARSE_HEX_DATA`状态中的上下文里的一些字节消失。

为了达到我们的目的，需要在`\objdata`控制字之后的数据中加入`\'XX`。在这种情况下，当解析器在`PARSER_PARSE_HEX_DATA`状态中遇到`\`时，它就会返回到状态`PARSER_BEGIN`，之后将进入状态`PARSER_PROCESS_CMD`。`\'`控制符号的处理程序不会改变最终的目的地，但会将下一个状态更改为`PARSER_PARSE_HEX_NUM_MSB`。`PARSER_PARSE_HEX_NUM_MSB`和`PARSER_PARSE_HEX_NUM_LSB`控制权转移回`PARSER_BEGIN`后，最终转换为`PARSER_PARSE_HEX_DATA`，因为最终的目的地仍然是`objdata`。在那之后，下一个字节将被解码为高半字节。

值得注意的是，`PARSER_PARSE_HEX_NUM_LSB`不检查提供的值是否为有效的十六进制数；因此，`\'`后面可以是任意的两个字节。

比如说，下面的这个例子：

![图片描述](upload/attach/201902/3_UXWK892BA3C2DJ4.png)

最终结果会删除`"f\'cc"`。

当控制首次转到`PARSER_PARSE_HEX_DATA`状态时，处理`\ objdata`控制字后，MSB位已经设置。我们来详细地看下这个过程的处理细节：

![图片描述](upload/attach/201902/3_WNDEXKHGUPP7R5B.png)

在逆向了关键字处理函数后，我找到了所有控制字及其相应结构的列表：

![图片描述](upload/attach/201902/3_Y9SD8QRYDZ3P3EF.png)

有了这些信息，我们可以找到并查看`objdata`构造函数：

![图片描述](upload/attach/201902/3_42JMRSQYGEY8UMT.png)

可以看到它设置了MSB位，分配一个新的缓冲区并用新的指针替换旧的指针。因此，在两个`\ objdata`控制字之间解码的数据从来没有用到过。

![图片描述](upload/attach/201902/3_RVNS93RAB2G3JSP.png)

最终结果会删除掉`"d0cf11e0a1b11ae1"`。

## 最终目的地

我们知道如果`\`或`\ objdata`被放入数据中，会改变输出结果。但如果是其他控制字和控制符号呢？这里有超过1500个，但几乎没有！

由于某些控制字表示目的地，因此不能使用它们——它们自己更改`objdata`目标，并解码需要`objdata`目标的对象。其他的一些控制字并不影响`objdata`的目的地。

在不丢失先前解码的数据的情况下返回到`objdata`目的地的唯一方法是使用特殊符号——`{`和`}`，这些符号表示组的开始和结束。

当解析器遇到`PARSER_BEGIN`状态的组末尾时，将在组开始之前设置的目标将被恢复。因此，通过在`\objdata`之后放置`{\aftncn FF}`，`FF`不会作为解码数据，因为`FF`现在应用于目标`aftncn`并将根据此目标进行处理。但是，通过使用`{\aftnnalc FF}`，`FF`将进入解码数据，因为目标仍然等于`objdata`。还值得注意的是`{\ objdata FF}`仍然不能使用，因为缓冲区不会被恢复。所有目标控制字的准确列表都是用简单的fuzzer创建的。

## 固定大小的缓冲区

在查看RTF解析器的代码时想到另一种混淆技术（与“MSB”bug无关），但也可用于从十六进制流中移除字节。该技术与临时缓冲区大小以及在`PARSER_PARSE_CONTROL_WORD`和`PARSER_PARSE_CONTROL_WORD_NUM_PARAMETER`状态下控制字和数值参数如何解析有关。在下面的截图中，你可以看到实际样例：

![图片描述](upload/attach/201902/3_DVZB8S2FHNA4Y3J.png)

在此示例中，将使用以下公式计算将那部分被作为数字参数删除的数据的大小：

```text
0xFF（临时缓冲区的大小） - 0xB（“oldlinewrap”的大小） - 2（空终止符字符）= 0xF2。
```

## 不必要的数据

尽管上述技术与通常的RTF解析相关，但某些特定关键字的处理隐藏了一些进一步的混淆措施。

根据规范所称，如果发现了`\\*`，但在查找表中却找不到之后的控制字或控制符号，它将被视为一个未知的目标组，并且直到右括号之前的所有数据都应该被丢弃。而MS Office中的查找表包含规范中不存在的控制字，使人们担忧它将来会改变，从而影响不同版本的MS Office上的同一文档的解析过程。当负责处理关键字的函数遇到这种情况或某个特定控制字（如`\comment`、`\generator`、`\nonshppict`等）时，它会将状态`PARSER_SKIP_DATA`和遇到的`{`设置为1。

```c
enum
{
    // …
    PARSER_SKIP_DATA = 0xF,
    // …
    PARSER_SKIP_DATA_CHECK_B = 0x13,
    PARSER_SKIP_DATA_CHECK_I = 0x14,
    PARSER_SKIP_DATA_CHECK_N = 0x15,
    PARSER_SKIP_DATA_GET_BIN_VAL = 0x16,
    PARSER_SKIP_DATA_INNER_DATA = 0x17,
    // …
};
```

## magic的种类

在分析`PARSER_SKIP_DATA *`状态的过程中，我发现不仅与规范相反，而且还与解析器代码的其余部分相反。

在查找`\bin`控制字时，此状态将跳过数据，更改所遇到`{}`大括号的数量，直到该数字等于零。隐藏的问题在于数字参数处理的方式。

首先，数字参数的最大允许长度增加到`0xFF`——它的计算没有考虑控制字的长度。

第二个问题是数字参数不再是数字了！解析器不仅允许使用十进制字符，而且还允许传递拉丁字符。然后将此参数传递给自定义`strtol`，从而可以指定应该跳过的数据的长度，而不用将`{`和`}`视为十六进制数字。

目前来看，还尚未在野外遇到使用这两种情况的混淆。

## 结论

逆向工程是建立一个解析器的最有效方式，然而对于RTF，却有可能无法达到预期的行为。

精确的解析依赖于小的实现细节和算法错误，而不是一个可能令人困惑的规范或陈述不真实的规范。

<a href="elink@ee2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3W2K6j5i4m8H3k6h3q4J5K9h3&6Y4i4K6u0V1j5Y4W2@1k6i4y4Q4x3V1j5^5y4o6l9I4y4#2)9J5c8R3`.`.">原文链接：<mark class="encrypted">a62K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3W2K6j5i4m8H3k6h3q4J5K9h3&6Y4i4K6u0V1j5Y4W2@1k6i4y4Q4x3V1j5^5y4o6l9I4y4#2)9J5c8R3`.`.</mark></a>

本文由看雪翻译小组 fyb波 编译

# 消失的字节：对 MS Office RTF 解析器的逆向工程

微软的Office是2017年的众多攻击目标之一，除了发现的大量漏洞和发布的POC之外，恶意软件作者认为有必要防止由防病毒软件检测到“one-day”和“old-day”漏洞。很明显，使用[RTF](https://bbs.kanxue.com/elink@b41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6d9K9h3y4Z5i4K6g2X3g2r3g2^5N6q4)9#2k6V1k6G2M7X3#2S2N6l9`.`.)解析功能和特性已不足以有效逃避检测。随着对MS Office利用的兴起，我们遇到了大量的RTF被用作攻击容器的情况，这些样本“利用”Microsoft Word RTF解析器的实现来混淆所有其他第三方的RTF解析器，包括那些在AV软件中的解析器。

为了还原MS Office中的解析，我们需要对其进行逆向工程。

我决定首先看看较早版本的MS Office 2010，可以先研究一下早期的实现方法。然后，再与新版本中的相关实现方法进行比较。

一个RTF解析器包含一个具有37个状态的状态机，其中的22个状态是唯一的：

![](https://bbs.kanxue.com/upload/attach/202002/726271_DT3NAFYJEMA4N2J.jpg)

我们先看下最重要的状态以及那些对解析`\objdata`（包含对象数据的目标控制字）有影响的那些状态。Microsoft OLE链接，Microsoft OLE嵌入对象和Macintosh版本管理器订阅服务器对象在RTF中用对象表示。以下是所有的情况：

```c
enum
{
    PARSER_BEGIN = 0,
    PARSER_CHECK_CONTROL_WORD = 2,
    PARSER_PARSE_CONTROL_WORD = 3,
    PARSER_PARSE_CONTROL_WORD_NUM_PARAMETER = 4,
    PARSER_PARSE_HEX_DATA = 5,
    PARSER_PARSE_HEX_NUM_MSB = 7,
    PARSER_PARSE_HEX_NUM_LSB = 8,
    PARSER_PROCESS_CMD = 0xE,
    PARSER_END_GROUP = 0x10,
    // …
};
```

由于没有Microsoft Office的调试符号，所以无法恢复其原始的状态名称。不过，我已根据其基本功能选择了合适的名称。

在打开的RTF文件上执行的第一个状态是`PARSER_BEGIN`。在大多数情况下，它也是在处理完一个控制字之后执行的。此状态的主要目标是根据遇到的char、目标以及存储在`this`结构中并由控制字处理器设置的其他值来确定下一个状态。默认情况下，下一个状态是`PARSER_CHECK_CONTROL_WORD`。
