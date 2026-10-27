
# [翻译]关于C++编程的42条建议（三）

## 21. 检查文件终止符（`EOF`）有正确到达终点

让我们继续讨论和文件有关的话题，再来看`EOF`。但这次我们要讨论的是完全不同的bug类型。它通常只出现在本地化软件版本里。

下面的代码片段是选自**Computational Network Toolkit**。错误被**PVS-Studio**诊断为：`V739` `EOF`不应该和`char`类型比较。‘`c`’应该是`int`类型。

```c
string fgetstring(FILE* f)
{ 
  string res;  
  for (;;)  
  {     
    char c = (char) fgetc(f); 
    if (c == EOF)      
      RuntimeError("error reading .... 0: %s", strerror(errno));
    if (c == 0)    
      break;    
    res.push_back(c); 
  }  
  return res; 
}
```

### 解释

让我们来看`EOF`是怎样声明的：

```c
#define EOF (-1)
```

如你所见，`EOF`就是一个值为`-1`的`int`类型。`fgetc()`返回一个`int`类型的值。也就是说，它可以返回`0`到`255`的一个数，或者是`-1`（`EOF`）。

使用扩展字符集（*Extended ASCII Codes*）的用户在程序中某一个字母处理不当的时候可能会出错。

比如说，在`Windows1251`编码表中，俄文的最后一个字母是`0xFF`，而在程序中就会被编译为文件终止符。

### 正确代码

```c
for (;;)
{
    int c = fgetc(f);
    if (c == EOF)
        RuntimeError("error reading .... 0: %s", strerror(errno));
    if (c == 0)
        break;
    res.push_back(static_cast<char>(c));
}
```

### 建议

在这里并没有什么特殊的建议，但既然我们谈到了`EOF`，我想展示一些人没有注意到的比较有趣的错误。

只要记住，如果一个函数的返回值是`int`类型，就不要急于把它转化为`char`类型。停下来检查看有没有错。顺便提一句，我们在第二个技巧“大于`0`并不意味着是`1`”那里讨论`memcmp()`函数的时候用到了相似的代码。（关于*MySQL*漏洞那一块）。

## 22. 不要用`#pragma warning(default:X)`

下面的代码来自**TortoiseGIT**项目。该错误被**PVS-Studio**诊断为：`V665` 也许，在此处，‘`#pragma warning(default: X)`’用得不对。应该用‘`#pragma warning(push/pop)`’。

```c
#pragma warning(disable:4996) 
LONG result = regKey.QueryValue(buf, _T(""), &buf_size); 
#pragma warning(default:4996)
```

### 解释

程序员经常认为，早期让警告实现的`#pragma warning(disable: X)`指令在使用`#pragma warning(default : X)`后就能继续工作。但并非如此。`#pragma warning(default : X)`这条指令是把‘`X`’警告设置为*DEFAULT*状态，这两个不是同一件事。

假设一个文件在编译时用了`-Wall`，那么就会出现`C4061`警告。如果你增加了`#pragma warning(default : 4061)`指令，那么这条警告就不会出现，因为它已经被默认关掉了。

### 正确代码

```c
#pragma warning(push) 
#pragma warning(disable:4996) 
LONG result = regKey.QueryValue(buf, _T(""), &buf_size)
#pragma warning(pop)
```

### 建议

返回警告的之前状态的正确方法是使用`#pragma warning(push[ ,n ])`和`#pragma warning(pop)`这两条指令。可以看*Visual C++11*关于这些指令描述的文档：[Pragma Directives. Warnings](https://bbs.kanxue.com/elink@b8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1j5J5j5K6S2X3y4K6j5$3k6g2)9J5y4e0t1^5N6W2)9K6c8s2k6K6i4K6u0W2z5o6m8Q4x3U0f1J5z5g2)9J5k6h3q4K6M7s2R3`.)

库开发者应该要特别注意`V665`警告。忽视定制警告会在库使用者这边引起巨大的灾难。

关于这个话题的，值得一看的好文章：[So, You Want to Suppress This Warning in Visual C++](https://bbs.kanxue.com/elink@1d8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6A6N6X3p5$3y4q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0r3K9#2)9J5c8U0l9H3y4o6S2Q4x3V1j5`.)

## 23. 自动计算字符串长度

下面的代码来自**OpenSSL**库。错误被**PVS-Studio**诊断为：`V666` 检查‘`strncmp`’的第三个参数。它可能和字符串的长度不一致，字符串长度取决于第二个参数。

```c
if (!strncmp(vstart, "ASCII", 5))  
    arg->format = ASN1_GEN_FORMAT_ASCII; 
else if (!strncmp(vstart, "UTF8", 4))  
    arg->format = ASN1_GEN_FORMAT_UTF8; 
else if (!strncmp(vstart, "HEX", 3))   
    arg->format = ASN1_GEN_FORMAT_HEX; 
else if (!strncmp(vstart, "BITLIST", 3)) 
    arg->format = ASN1_GEN_FORMAT_BITLIST; 
else   
    ....
```

### 解释

很难停止使用魔数。而且没有理由不使用类似`0, 1, -1, 10`这些常数。更难的是，给这些常数命名，而且，它们会使阅读代码变得更复杂。

然而，减少使用魔数的个数是有用的。比如说，避免使用用来定义字符串长度的魔数就很有用。

让我们来看一下上面给出的代码。代码看上去很像是复制粘贴的。程序员复制了下面这一行：

```c
else if (!strncmp(vstart, "HEX", 3))
```

之后，用`"BITLIST"`代替`"HEX"`，但是程序员忘了把`3`改为`7`。结果就是，字符串不是和`"BITLIST"`做比较，而是仅仅比较了`"BIT"`。这个错误似乎不太严重，但终究是个错误。

用复制粘贴来写代码真的很糟糕。更严重的是，字符串长度由魔数来定义。一直以来，我们[遇到](https://bbs.kanxue.com/elink@cb8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6A6N6X3p5$3y4q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0r3k6i4S2S2L8i4m8D9k6i4y4Q4x3V1k6h3y4U0j5$3i4K6u0r3)这样的错误，就是字符串长度和确定的数字不一致的错误，都是因为拼写错误或者程序员的疏忽。所以，这是一个典型的错误，我们需要做点什么来避免它。让我们来看看应该如何避免。

### 正确代码

乍一看，只要把`strncmp()`换成`strcmp()`就好了。那样魔数就消失了。

```c
else if (!strcmp(vstart, "HEX"))
```

不好——我们改变了代码运作的逻辑。`strncmp()`的作用是检查一个字符串是否是以`'HEX'`开始的，而`strcmp()`是检查两个字符串是否相等的。它们做的是不同的检查。

要解决这个问题最简单的方法就是改变常数：

```c
else if (!strncmp(vstart, "BITLIST", 7))   
    arg->format = ASN1_GEN_FORMAT_BITLIST;
```

这个代码是正确的，但是魔数`7`还在那里，这就有点不好了。这也是为什么我要建议另一种方法。

### 建议

如果我们能够正确估计字符串的长度，这样的错误就能够避免。最简单的方法就是用`strlen()`函数。

```c
else if (!strncmp(vstart, "BITLIST", strlen("BITLIST")))
```

在下面例子中，如果你忘记更改字符串，你能更快地发现它们的不匹配。

```c
else if (!strncmp(vstart, "BITLIST", strlen("HEX")))
```

但是这个建议有两个缺点：

1. 无法保证编译器会不会优化`strlen()`调用：用一个常数来代替它。

2. 你要逐字复制字符串。看上去不好看，而且也会出错。

第一个问题可以在编译阶段用专门计算字符串长度的结构来解决。比如说，你可以用这样的宏：

```c
#define StrLiteralLen(arg) ((sizeof(arg) / sizeof(arg[0])) - 1) ....

else if (!strncmp(vstart, "BITLIST", StrLiteralLen("BITLIST")))
```

但这个宏有点危险。在重构的时候会出现下面的代码：

```c
const char *StringA = "BITLIST"; 

if (!strncmp(vstart, StringA, StrLiteralLen(StringA)))
```

在这个例子中，`StrLiteralLen`宏会返回一些没意义的东西。取决于指针的大小（`4`或`8`字节），我们会得到`3`或`7`。但是在`C++`中我们可以避免这种尴尬的局面，通过使用更复杂的技巧：

```c
template <typename T, size_t N> 
char (&ArraySizeHelper(T (&array)[N]))[N];

#define StrLiteralLen(str) (sizeof(ArraySizeHelper(str)) - 1)
```

现在，如果`StrLiteralLen`宏的参数是一个简单的指针，我们可能无法编译上面的代码。

让我们来看第二个问难题（复制字符串）。我不知道要对`C`程序员说什么。你可以为它写一个特别的宏，但是我个人不喜欢这样。我不热衷于宏。所以我知道要建议什么。

在`C++`中，一切都很好。而且，我们可以用更聪明的法子来解决第一个问题。模板函数会帮我们很大的忙。你可以用不同的方法写，但是一般是长这样的：

```c
template<typename T, size_t N> 
int mystrncmp(const T *a, const T (&b)[N])
{   
    return _tcsnccmp(a, b, N - 1); 
}
```

现在，字符串只用一次。在编译阶段就能计算字符串的长度。你不会遇到简单指针也不会错误的计算字符串的长度。完美！

### 总结

在处理字符串的时候避免使用魔数。使用宏或模板函数。这样函数不仅会变得更安全，而且会更漂亮更简短。

作为例子，你可以看[`strcpy_s()`](https://bbs.kanxue.com/elink@638K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8F1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1j5#2k6r3q4W2y4h3b7@1x3#2)9J5k6h3q4K6M7s2R3`.)的声明：

```c
errno_t strcpy_s(  
    char *strDestination, 
    size_t numberOfElements, 
    const char *strSource  
);

template <size_t size> 
errno_t strcpy_s(  
    char (&strDestination)[size],  
    const char *strSource 
); // C++ only
```

第一个是针对于`C`语言的，或者在不是提前知道缓冲区大小的情况的。如果我们要处理缓冲区，创建一个栈，这样我们就可以在`C++`中用第二个了。

原文链接：[48fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3W2F1N6r3g2D9i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3N6r3S2W2i4K6u0V1N6h3I4@1K9h3#2S2N6r3g2Q4x3X3c8I4N6h3g2K6N6r3W2G2L8W2)9J5k6r3!0X3i4K6u0V1M7s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3X3c8J5k6h3k6S2j5%4c8G2M7X3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3g2$3k6i4u0&6N6r3S2A6L8X3M7`.](https://bbs.kanxue.com/elink@91cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3W2F1N6r3g2D9i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3N6r3S2W2i4K6u0V1N6h3I4@1K9h3#2S2N6r3g2Q4x3X3c8I4N6h3g2K6N6r3W2G2L8W2)9J5k6r3!0X3i4K6u0V1M7s2u0G2k6%4u0S2L8h3#2A6L8X3N6Q4x3X3c8J5k6h3k6S2j5%4c8G2M7X3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3g2$3k6i4u0&6N6r3S2A6L8X3M7`.)

本文由 看雪翻译小组 lumou 编辑

第一部分：<http://bbs.pediy.com/thread-215956.htm>

[第二部分：](http://bbs.pediy.com/thread-216074.htm)

原本是打算每十个发一次的，但是当比较多的时候调整结构我又比较烦，所以还是翻译得多少就传多少吧。。。

因为论文看不进，只好又翻译了（捂脸）。。。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
