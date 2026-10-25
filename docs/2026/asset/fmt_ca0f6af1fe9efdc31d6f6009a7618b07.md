
|set\_local 2|局部变量`var1`=`堆栈顶部的值：(param1 + param2)/4/2*4`|

|get\_local 0|将`param1`放入堆栈，准备函数调用|

|get\_local 1|将`param2`放入堆栈，准备函数调用|

|get\_local 2|将`var1`放入堆栈，准备函数调用|

|i32.load|弹出栈顶的值，作为指向全局内存的指针，然后获取它指向的数据并将该数据压入堆栈|

|call 0|使用刚刚设置的参数,调用函数0(名为“partition”)|

|set\_local 2|局部变量`var1`=函数调用的返回值|

再一次，用高级伪代码表达：

```
var1 = (param1 + param2) / 4 / 2 * 4;
var1 = call partition(*var1,param2,param1);
```

如果这很难掌握，请尝试在Chrome中运行示例，单步执行并在调试器中查看（即DevTools）堆栈和局部变量值的变化，以及全局内存的构造。一篇关于Wasm分析的早期博文介绍了做法。当我们位于'call 0'指令时，在执行之前，局部变量和堆栈将如下所示：

![图片](upload/attach/202002/793242_N8XDUX3M8AKC5DS.jpg)

继续查看Wasm代码：

|指令|描述|

|:---|:--|

|get\_local 0|将`param1`压入堆栈|

|get\_local 2|将`var1`压入堆栈|

|i32.const 4|将4压入堆栈|

|i32.sub|从`var1`中减去4（仅在堆栈上，而不是在局部变量内存中）|

|call 1|调用排序函数|

|drop|丢弃排序调用中的返回值|

|get\_local 2|将`var1`压入堆栈|

|i32.const 4|将4压入堆栈|

|i32.add|`var1`加4(仅在堆栈上，而不是在局部变量内存中)|

|get\_local 1|将`param2`放入堆栈|

|call 1|调用`sort`函数|

|drop|丢弃排序调用中的返回值|

|get\_local 2|返回`var1`(堆栈中最后剩下的项将是返回值)|

用高级伪代码表示：

```
call sort(var1 - 4, param1);
call sort(param2, var1 + 4)
return var1;
```

总而言之，我们的最终结果是：

```
func sort(int param1, int param2) {
int var1;
if (param1 >= param2)
return param2;
var1 = (param1 + param2) / 4 / 2 * 4;
var1 = call partition(*var1,param2,param1);
call sort(var1 - 4, param1);
call sort(param2, var1 + 4)
return var1;
}
```

我们发现这确实是一个QuickSort实现。如果您愿意，可以通过将上述伪代码与您在Internet上找到的某些现有实现进行比较验证。逆向 `partition`函数留给读者练习。

## 总结

我们现在成功地逆向了一个完整的Wasm函数。首先，我们使用`wasm2wat`程序将Wasm二进制格式转换为其文本格式，并通过分析文本格式，我们能够创建算法的高级伪代码。

存在用于执行自动反编译的工具，一种比我们今天所做的更有效的逆向工程方法。虽然自动反编译可以节省时间，但它通常不完美，对手动分析的理解使我们能够解决这些不完善之处。

## 参考文献

+ <mark class="encrypted">97eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0m8M7%4y4W2L8h3u0D9P5g2)9J5c8W2g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4y4h3k6@1K9r3g2Q4y4h3k6@1k6i4S2@1i4K6g2X3k6X3!0J5L8h3q4@1</mark>

+ <mark class="encrypted">e26K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1M7%4c8J5N6h3y4@1K9h3!0F1M7H3`.`.</mark>

+ <mark class="encrypted">314K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3L8X3k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2N6X3g2J5M7$3W2F1k6#2)9J5k6s2N6S2M7$3#2Q4x3X3g2H3k6r3j5`.</mark>

+ <mark class="encrypted">509K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4m8Z5L8%4y4Q4x3X3g2X3K9h3I4W2M7#2)9J5k6i4N6G2M7X3c8H3M7X3g2K6M7#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3z5q4)9J5c8Y4y4G2M7r3S2G2M7#2)9J5k6s2g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4x3X3c8%4k6h3u0Q4x3X3c8S2M7%4y4W2L8h3u0D9P5g2)9J5k6i4m8V1k6R3`.`.</mark>

翻译：看雪翻译小组 欢歌笑语

校对：看雪翻译小组 Nxe

原文链接：<mark class="encrypted">24dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8X3#2S2L8Y4g2S2L8q4)9J5k6s2u0W2N6X3g2J5M7$3g2Q4x3X3c8W2L8X3N6A6L8X3g2W2M7X3W2F1k6#2)9J5k6s2N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1M7%4c8S2N6r3W2U0i4K6u0V1j5$3!0V1k6g2)9J5k6r3q4F1j5h3I4&6M7$3W2K6</mark>

在我们上一篇关于WebAssembly（Wasm）的[博客](https://bbs.kanxue.com/elink@c6eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8X3q4F1j5h3I4&6P5X3W2F1k6#2)9J5k6s2N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1j5X3W2F1j5i4u0A6k6i4y4Q4x3X3c8A6L8X3W2@1K9h3q4D9i4K6u0V1k6X3g2W2L8q4)9J5k6r3q4F1k6q4)9J5k6r3u0W2K9r3q4$3K9h3!0J5j5h3I4Q4x3X3c8S2L8X3q4D9P5i4y4A6M7H3`.`.)中，我们初步了解了一个未知的Wasm二进制文件，并对其进行了一些行为分析。今天我们将继续深入研究相同的Wasm样本。我们将通过研究Wasm文本格式手动分析该样本。

> 为了能够手动分析Wasm文本，我们首先需要学习更多理论。我们之前的博文描述了如何处理内存和数据。在此基础上，我们将介绍一些对逆向Wasm有用的概念，然后应用这些知识来分析Wasm样本。
>
> 注意：这篇文章是系列文章的一部分。该系列的上一篇文章介绍了Wasm内存处理，所以可能需要阅读[这些文章](https://bbs.kanxue.com/elink@d13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8X3q4F1j5h3I4&6P5X3W2F1k6#2)9J5k6s2N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1j5X3W2F1j5i4u0A6k6i4y4Q4x3X3c8A6L8X3W2@1K9h3q4D9i4K6u0V1k6X3g2W2L8q4)9J5k6r3q4F1k6q4)9J5k6r3u0W2K9r3q4$3K9h3!0J5j5h3I4Q4x3X3c8S2L8X3q4D9P5i4y4A6M7H3`.`.)。

## Wasm指令集

正如我们在今年早些时候[讨论](https://bbs.kanxue.com/elink@32cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8Y4N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1M7r3!0@1k6h3&6@1K9h3q4D9M7#2)9J5k6r3q4F1k6q4)9J5k6s2m8A6N6r3k6S2L8r3I4K6)的那样，Wasm本身不能与外界联系。与外部环境的所有通信都需要通过JavaScript API调用。考虑到这一点，我们现在主要讨论那些Wasm中对实际计算一些有用的指令 ，而不是调用JavaScript的指令。

与`x86`或`x64`的指令集相比，Wasm的指令集非常小，有几组不同的功能：

- 算术指令
- 控制流指令
- 内存访问指令
- 比较指令
- 转换指令

以下是常见Wasm指令的几个示例。有关更全面的说明列表，请参阅[参考手册](https://bbs.kanxue.com/elink@dd9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1M7%4c8J5N6h3y4@1K9h3!0F1M7H3`.`.)。

| 指令 | 描述 |
| --- | --- |
| get\_local `<variable>` | 获取本地存储中变量的值，将其压入堆栈使其可用于后续指令 |
| set\_local `<variable>` | 设置本地存储中变量的值，从堆栈中弹出值并将该值分配给相关的局部变量 |
| get\_global `<variable>` | 获取全局变量的值 |
| set\_global `<variable>` | 设置全局变量的值 |
| i32.add | 从堆栈中弹出两个数字相加，并将结果压入堆栈 |
| call<func nbr> | 直接调用指定的函数号 |
| call\_indirect `<var>` | 调用函数号在运行时生成的函数 |
| if/else/end | 条件分支 |
| br | 无条件分支 |
| br\_if | 条件分支 |
| loop | 定义要循环的代码块 |
| block | 定义一个代码块 |
| return | 函数返回 |

## 了解文本格式

上面给出是WebAssembly文本格式的说明。由于Wasm是二进制格式，因此这些文本指令将在编译文件中用字节码表示。

让我们分析一个简单的函数：

```
(func $max (; 0 ;) (param $0 i32) (param $1 i32) (result i32)
  (select
   (get_local $0)
   (get_local $1)
   (i32.gt_s
    (get_local $0)
    (get_local $1)
   )
  )
 )
```

第一行表示一个名为`'max'`的函数，以两个整数作为参数，`$0`和`$1`，并返回一个 `i32` (32-bit integer)类型的整数。

`select`指令有三个参数：第一个操作数（`get_local $0`），第二个操作数（`get_local $1`）和一个条件参数（在这种情况下是`i32.gt_s`指令及其相关的操作数）。如果条件操作数不为零，则`Select`返回第一个操作数，否则返回第二个操作数。
