
在`select`中，指令[`i32.gt_s`](https://bbs.kanxue.com/elink@b38K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1N6r3g2Y4k6i4u0Q4x3X3c8Y4M7X3g2S2N6r3g2J5i4K6u0V1N6r3S2S2L8W2)9J5k6s2y4A6k6$3&6W2k6l9`.`.)检查第一个参数（`get_local $0`）是否大于第二个参数（`get_local $1`）。此检查的结果将保存在`select`运算符的条件操作数中。

因此，如果第一个参数大于第二个参数，则返回第一个参数，否则返回第二个参数。

请注意，不同的工具表示的Wasm文本格式可能会略微不同（就像不同的反汇编器一样）。例如，上面的例子也可以这样表示：

```
(func (;0;) (type 0) (param i32 i32) (result i32)
  get_local 0
  get_local 1
  get_local 0
  get_local 1
  i32.gt_s
  select)
```

无论我们使用何种文本表示，它都对应于以下高级语言：

```
int max（int a，int b）{
  return a> b？a：b;
}
```

有关Wasm文本格式的更多信息，请访问[此处](https://bbs.kanxue.com/elink@a19K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0m8M7%4y4W2L8h3u0D9P5g2)9J5c8W2g2F1k6r3g2J5M7%4c8S2L8X3c8A6L8X3N6Q4y4h3k6@1K9r3g2Q4y4h3k6@1k6i4S2@1i4K6g2X3k6X3!0J5L8h3q4@1)。

## 使用wasm2wat进行静态代码分析

现在，我们理解了堆栈机，局部变量，全局内存，数据存储（如我们以前的[帖子](https://bbs.kanxue.com/elink@a8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8X3q4F1j5h3I4&6P5X3W2F1k6#2)9J5k6s2N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1j5X3W2F1j5i4u0A6k6i4y4Q4x3X3c8A6L8X3W2@1K9h3q4D9i4K6u0V1k6X3g2W2L8q4)9J5k6r3q4F1k6q4)9J5k6r3u0W2K9r3q4$3K9h3!0J5j5h3I4Q4x3X3c8S2L8X3q4D9P5i4y4A6M7H3`.`.)提到的），指令集和Wasm文本格式的概念，接下来用所学知识深入分析之前的博客文章中的[Wasm示例](https://bbs.kanxue.com/elink@317K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3#2A6L8X3W2U0N6r3q4J5M7W2)9J5c8Y4q4#2K9h3y4C8M7$3!0J5N6q4)9J5k6i4N6S2M7$3#2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7i4g2A6j5$3E0K6L8%4u0@1i4K6u0W2N6$3q4K6L8b7`.`.)。

从之前的位置继续：最初的浅层分析和行为分析表明我们正在处理的是排序算法。对于这个样本，分析可能会在此时就完成了，具体取决于您可以在一个样品上花费多少时间。

如果我们处理的是一个功能不那么明显的样本怎么办？您可能经常需要查看源代码，接下来进行这项操作。

为了处理代码，我们将再次使用我们之前的博文中的`wasm2wat`工具。我们已经发现`sort`函数的函数号为1。下面是Wasm文本表示的该函数的第一部分：

```
$ ./wasm2wat quicksort.wasm
[snip]
(func (;1;) (type 1) (param i32 i32) (result i32)
  (local i32)
```

它从函数定义开始，表示函数使用两个整数并返回一个整数。然后定义了一个局部变量`local i32`。用高级伪代码表示：

```
func sort(int param1, int param2) {
  int var1;
```

Wasm代码:

```
    get_local 0
    get_local 1
    i32.ge_s
    if  ;; label = @1
      get_local 1
      return
    end
```

前两个指令`get_local 0/1`分别获取第一个和第二个函数参数的值，并将它们压入堆栈。然后第三条指令[`i32.ge_s`](https://bbs.kanxue.com/elink@200K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1N6r3g2Y4k6i4u0Q4x3X3c8Y4M7X3g2S2N6r3g2J5i4K6u0V1N6r3S2S2L8W2)9J5k6r3!0J5i4K6u0V1k6i4q4#2j5h3I4Q4x3X3c8@1L8#2)9J5k6s2y4A6k6$3&6W2k6l9`.`.)将对堆栈上的这两个值进行操作，通过隐式弹出它们然后判断第一个值是否大于或等于第二个值。比较的结果将被压入堆栈。如果堆栈顶部的值为非零值，则后续`if`语句将为true。换句话说，`if`语句的分支将取决于前面的三条指令。

如前所述，相同的代码可以用不同的方式表示。如果上面的`if`语句难以理解，这里有一个替代表示：

```
  (if
   (i32.ge_s
    (get_local $var$0)
    (get_local $var$1)
   )
```

到目前为止我们所逆向的内容可以用高级伪代码表示，如下所示：

```
func sort(int param1, int param2) {
  int var1;
  if (param1 >= param2)
    return param2;
```

继续使用`wasm2wat`查看Wasm文本格式，发现`sort`函数：

```
    get_local 0
    get_local 1
    i32.add
    i32.const 4
    i32.div_s
    i32.const 2
    i32.div_s
    i32.const 4
    i32.mul
```

这段代码进行数学计算。`get_local 0/1`指令获取传递给函数的两个参数值，随后的[`i32.add`](https://bbs.kanxue.com/elink@f68K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1N6r3g2Y4k6i4u0Q4x3X3c8S2k6r3b7`.)指令对这两个值进行操作，把它们相加并将结果压入栈中。

之后[`i32.const`](https://bbs.kanxue.com/elink@299K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3y4G2L8Y4y4@1j5h3&6@1) 4指令将4压入堆栈。后续指令[`i32.div_s`](https://bbs.kanxue.com/elink@aa8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3y4G2L8Y4y4@1j5h3&6@1)将堆栈顶部第二项的值除以最顶部的值。换句话说，先前`param1 + param2`的值将被除以4。在此之后，我们再次执行相同的模式，但这次除以2。同样，接下来的两条指令是`i32.mul`指令，操作数是4。

最终结果是目前获得的值乘以4。可以更简洁地表示为，代码执行以下计算：`(param1 + param2)/4/2 * 4`。

让我们看看随后的Wasm代码，右侧添加了我们的注释：

| 指令 | 描述 |
| --- | --- |
| `set_local 2` | 局部变量`var1`=堆栈顶部的值：`(param1 + param2)/4/2*4` |
| `get_local 0` | 将`param1`放入堆栈，准备函数调用 |
| `get_local 1` | 将`param2`放入堆栈，准备函数调用 |
| `get_local 2` | 将`var1`放入堆栈，准备函数调用 |
| [`i32.load`](https://bbs.kanxue.com/elink@1bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3I4G2j5h3b7`.) | 弹出栈顶的值，作为指向全局内存的指针，然后获取它指向的数据并将该数据压入堆栈 |
| `call 0` | 使用刚刚设置的参数,调用函数0(名为`partition`) |
| `set_local 2` | 局部变量`var1`=函数调用的返回值 |

再一次，用高级伪代码表达：

```
var1 = (param1 + param2) / 4 / 2 * 4;
var1 = call partition(*var1,param2,param1);
```

如果这很难掌握，请尝试在Chrome中运行示例，单步执行并在调试器中查看（即`DevTools`）堆栈和局部变量值的变化，以及全局内存的构造。一篇[关于Wasm分析的早期博文](https://bbs.kanxue.com/elink@9ddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8%4u0U0k6i4m8G2K9h3&6@1i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1L8r3q4T1M7#2)9J5c8X3q4F1j5h3I4&6P5X3W2F1k6#2)9J5k6s2N6W2j5X3q4K6M7$3g2E0j5X3I4&6i4K6u0V1j5X3W2F1j5i4u0A6k6i4x3`.)介绍了做法。当我们位于`call 0`指令时，在执行之前，局部变量和堆栈将如下所示：

![图片](https://bbs.kanxue.com/upload/attach/202002/793242_N8XDUX3M8AKC5DS.jpg)

继续查看Wasm代码：

| 指令 | 描述 |
| --- | --- |
| `get_local 0` | 将`param1`压入堆栈 |
| `get_local 2` | 将`var1`压入堆栈 |
| `i32.const 4` | 将4压入堆栈 |
| [`i32.sub`](https://bbs.kanxue.com/elink@d06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6h3&6X3K9i4y4Z5j5$3!0V1k6g2)9J5c8Y4N6S2M7$3#2Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8E0j5h3&6#2j5h3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1b7i4y4K6k6h3#2T1L8s2W2Q4x3X3g2E0k6q4)9J5x3$3W2F1N6r3g2Y4k6i4u0Q4x3X3c8K6N6h3u0@1M7X3q4U0N6l9`.`.) | 从`var1`中减去4（仅在堆栈上，而不是在局部变量内存中） |
| `call 1` | 调用排序函数 |
| `drop` | 丢弃排序调用中的返回值 |
| `get_local 2` | 将`var1`压入堆栈 |
| `i32.const 4` | 将4压入堆栈 |
| `i32.add` | `var1`加4(仅在堆栈上，而不是在局部变量内存中) |
| `get_local 1` | 将`param2`放入堆栈 |
| `call 1` | 调用`sort`函数 |
| `drop` | 丢弃排序调用中的返回值 |
| `get_local 2` | 返回`var1`(堆栈中最后剩下的项将是返回值) |

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

我们发现这确实是一个QuickSort实现。如果您愿意，可以通过将上述伪代码与您在Internet上找到的某些现有实现进行比较验证。逆向`partition`函数留给读者练习。

## 总结

现在，我们成功地逆向了一个完整的Wasm函数。首先，我们使用`wasm2wat`程序将Wasm二进制格式转换为其文本格式，并通过分析文本格式，我们能够创建算法的高级伪代码。

存在用于执行自动反编译的工具，一种比我们今天所做的更有效的逆向工程方法。虽然自动反编译可以节省时间，但它通常不完美，对手动分析的理解使我们能够解决这些不完善之处。

## 参考文献
