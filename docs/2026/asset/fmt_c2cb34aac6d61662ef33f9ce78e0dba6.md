
```python
while todo:
# Prepare a solution to try, based on the clean state
file_content = todo.pop()
print "CUR: %r" % file_content
open(TEMP_FILE.name, "w").write(file_content)
dse.restore_snapshot(snapshot, keep_known_solutions=True)
FILE_to_info.clear()
FILE_to_info_symb.clear()
# Play the current file
try:
sb.run()
except FinishOn as finish_info:
print finish_info.string
if finish_info.string == "OK":
# Stop if the expected result is found
found = True
break
# Get new candidates
finfo = FILE_to_info_symb[FILE_stream]
for sol_ident, model in dse.new_solutions.iteritems():
# Build the file corresponding to solution in 'model'
...
todo.add(out)
```

启动这个最终的脚本，得到下列输出：

```
CUR: ''
BAD
CUR: '\x00'
BAD
CUR: '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
BAD
CUR: 'M\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
BAD
CUR: 'M\x00\x00\x00\x00\x00\x00\x07\x00\x00\x00\x00\x00\x00\x00'
BAD
CUR: 'M\x00\x00\x00\x00\x00\x00\x87\x00\x00\x00\x00\x00\x00\x00'
BAD
CUR: 'M!@$M\x00\x00\x87\x00\x00\x00\x00\x00\x00\x00'
BAD
CUR: 'M!@$MN\x0e\x87\xb1}\nN\xc60\xe7'
OK
FOUND !
```

我们可以观察到上面这些检查点，分别是：

- 检查文件大小

- 检查第一个字节（等于`'M'`）

- 分别在两个测试用例（`x!=7,2*x==14`）中检查溢出

- 检查`magic`值

- 检查`CRC16`的值（必须等于`1337`）

我们可以用已得到的结果来检查结果是否正确：

```
$ hexdump -C test.txt
00000000 4d 21 40 24 4d 4e 0e 87 b1 7d 0a 4e c6 30 e7 |M!@$MN...}.N.0.|
$ gcc dse_crackme.c -o dse_crackme
$ ./dse_crackme test.txt
OK
```

最终我们得到了正确的结果。

在这里，代码覆盖率的策略已经足够了；但并不总是这样。如果这一最简单的策略不够好，那就使用更强力，当然也更麻烦，的策略，比如说分支和路径覆盖策略。要使用新的策略，只需要修改`DSEPathConstraint`的`produce_solution`参数。

边注：对于那些对内部实现感兴趣的人，你可能注意到了，`CRC16`是基于表实现的。最后，一些待求解的约束如下所示：

`@16[table_offset + symbolic_element] == CST`

我们可以用`z3`查找符号，解出这个约束，要做到这点，`z3`需要知道表的值。

为了达到这一目的，我们可以使用`Miasm` `expr_range`工具来取得可到达的地址区间。如果区间不是太大的话，我们可以在内存内容中添加一个约束给`z3`（`@16[table_offset + 2] == 0x1021`等）.

因为地址是用符号保存的，所以这也是可以做到的。因为性能的原因，其它的一些框架没有选择这种方法.要在`Miasm`中模仿这一过程，寻址过程可以由它们的`DSE.handle`方法中的具体值来完成。

# 最后的想法

这篇文章对`DSE`能完成的工作做了一个大体的介绍。

`Miasm`的实现方式非常灵活；它可以满足大部人的需要。此外，只需要几行代码，就可以将`DSE`添加到现有脚本中。事实证明它是一种强大的逆向工具。

# 相关资源

相关的脚本：

- <a href="elink@141K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5i4y4E0i4K6u0W2M7X3g2Q4x3V1k6T1L8r3!0Y4i4K6u0r3i4K6g2X3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0r3N6r3W2Y4M7X3g2K6M7#2)9J5k6i4m8&6">tigress.py</a>

- <a href="elink@e9cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5i4y4E0i4K6u0W2M7X3g2Q4x3V1k6T1L8r3!0Y4i4K6u0r3i4K6g2X3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0r3M7X3g2H3j5h3y4C8i4K6u0W2M7s2V1`.">repack.py</a>

相关链接：

- <a href="elink@225K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5M7#2)9J5k6h3g2U0k6g2)9J5k6h3y4E0N6g2)9J5k6h3g2V1N6g2)9J5c8W2)9%4c8h3c8T1M7Y4g2E0L8r3g2&6i4K6u0r3M7r3c8X3i4K6u0r3b7$3S2S2i4K6t1#2x3U0m8W2N6q4)9J5y4e0t1H3j5h3I4Q4x3X3g2Q4y4h3j5J5x3o6p5J5i4K6g2X3g2h3&6D9k6h3q4K6K9r3W2F1k6#2)9J5y4e0t1H3e0h3q4&6K9r3g2E0i4K6t1#2x3U0m8G2L8W2)9J5y4e0t1H3b7X3W2F1j5i4u0&6i4K6t1#2x3U0m8o6L8$3c8W2i4K6u0W2M7r3c8X3">Unleashing MAYHEM on Binary Code</a>

- <a href="elink@215K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6s2u0S2K9h3I4G2k6X3u0A6N6s2y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6g2Q4x3V1j5I4y4g2)9J5c8X3#2S2k6$3W2U0i4K6u0V1N6$3W2@1K9q4)9J5k6r3#2S2L8Y4c8A6j5$3!0J5k6g2)9J5c8R3`.`.">Manticore - Defcon 200</a>

- <a href="elink@525K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2P5r3q4E0M7r3I4W2M7#2)9J5c8X3c8W2k6X3y4G2L8U0t1H3x3e0N6I4N6h3q4D9M7#2)9#2k6X3y4J5j5h3y4C8L8h3f1J5x3o6l9H3i4K6u0r3L8h3q4Y4K9h3y4Q4x3X3g2H3P5b7`.`.">Angr - Defcon magic2000</a>

- <a href="elink@38bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4N6h3g2V1L8%4g2Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8Y4c8S2L8r3E0K6i4K6u0r3x3U0l9I4y4#2)9#2k6V1u0W2k6g2u0#2L8i4m8Q4x3V1k6K6L8r3W2V1k6i4y4Q4x3X3g2H3k6r3j5`.">Using Miasm to fuzz binaries with AFL (+ DSE)</a>

- <a href="elink@582K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5M7#2)9J5k6h3g2U0k6g2)9J5k6h3y4E0N6g2)9J5k6h3g2V1N6g2)9J5c8W2)9%4c8h3q4S2N6X3N6W2M7X3W2F1i4K6u0r3M7r3q4H3k6i4u0K6i4K6u0r3N6X3g2J5K9i4c8W2M7%4c8A6L8X3N6Q4x3X3c8A6j5%4y4W2i4K6u0V1x3U0l9I4y4q4)9J5k6i4m8V1k6R3`.`.">Enhancing Symbolic Execution with Veritesting</a>

- <a href="elink@d55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6k6h3g2W2i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2G2M7X3N6Q4x3V1k6f1b7#2)9J5c8W2y4b7x3U0l9I4y4#2)9J5c8Y4m8S2M7r3g2J5M7#2)9J5c8U0f1%4z5g2)9J5k6i4m8V1k6R3`.`.">Your Exploit is Mine: Automatic Shellcode - Transplant for Remote Exploits</a>

- <a href="elink@71bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8Y4c8W2M7X3&6W2N6s2y4G2j5$3W2W2N6s2W2Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9i4c8W2M7#2)9J5c8X3c8W2k6X3q4#2L8s2c8Q4x3V1k6X3K9h3I4W2M7#2)9J5c8X3u0D9L8$3N6K6i4K6u0V1L8h3g2V1K9h3q4Q4x3V1k6V1M7X3W2D9L8r3g2J5i4K6u0V1j5i4g2Y4L8h3g2F1N6r3W2F1k6#2)9J5k6r3k6#2P5Y4A6A6L8X3N6Q4x3X3c8@1K9s2u0G2N6h3N6Z5i4K6u0V1M7$3g2D9k6h3y4@1K9i4k6W2i4K6u0V1M7%4W2E0j5X3!0D9K9h3y4Q4x3X3c8W2P5r3g2U0N6i4c8A6L8$3&6Q4x3X3g2H3k6r3j5`.">Driller: Augmenting Fuzzing Through Selective - Symbolic Execution</a>

原文链接：<mark class="encrypted">d47K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5i4y4E0i4K6u0W2M7X3g2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9I4y4#2)9J5c8U0p5H3i4K6u0r3x3o6g2Q4x3V1k6H3L8r3q4&6K9h3&6Y4i4K6g2X3N6$3W2@1K9q4)9#2k6X3c8&6L8X3q4E0K9h3y4Q4y4h3k6K6P5h3#2T1L8$3I4A6j5#2)9#2k6X3g2^5k6h3y4#2N6r3W2G2L8W2)9J5k6h3S2@1L8h3I4Q4x3U0y4A6L8Y4k6W2M7Y4y4A6L8X3N6Q4x3X3c8S2i4K6u0V1M7$3S2W2L8r3I4U0L8$3c8W2i4K6u0V1M7r3q4U0K9$3g2J5</mark>

编译：看雪翻译小组 梦野间

校对：看雪翻译小组 lumou

# 玩转动态符号执行

动态符号执行（Dynamic symbolic execution，DSE）是一种目前较流行的，强大的分析方法，它通常用在下面几种情况中，如：

- [代码覆盖率](https://bbs.kanxue.com/elink@ab6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7X3W2@1L8$3&6Q4x3X3g2I4N6h3q4J5K9%4y4D9j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3b7$3!0V1k6g2)9J5k6r3y4G2N6X3g2J5j5h3N6W2i4K6u0V1N6i4y4A6L8X3N6Q4x3X3c8V1P5h3&6S2L8h3W2U0i4K6u0V1M7%4W2E0j5X3!0D9K9h3y4Q4x3X3c8W2P5r3g2U0N6i4c8A6L8$3&6Q4x3V1j5`.)

- 生成输入和测试用例

- [生成Exploit](https://bbs.kanxue.com/elink@297K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3N6J5i4K6u0r3j5h3&6Y4M7W2)9J5k6r3c8G2j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6V1L8$3y4K6i4K6u0r3k6i4S2S2L8i4m8D9k6i4y4Q4x3X3g2E0k6q4)9J5x3$3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8R3`.`.)

- [直接fuzzing](https://bbs.kanxue.com/elink@dafK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3k6s2u0A6L8r3I4W2M7R3`.`.)

- 检查越界访问

- 自动破解`crackme`

- [算法重构](https://bbs.kanxue.com/elink@eeaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6v1L8$3&6S2N6r3S2S2L8W2y4S2L8s2N6S2L8W2)9J5c8W2c8A6k6%4u0W2M7%4y4Q4y4h3k6H3M7X3!0@1k6h3y4@1K9h3!0F1)

- [反混淆](https://bbs.kanxue.com/elink@404K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4S2A6N6W2)9J5k6h3!0J5k6#2)9J5c8Y4m8V1k6W2)9J5c8U0p5$3x3e0u0Q4x3X3f1H3y4e0j5%4y4g2)9J5k6i4m8V1k6R3`.`.)

- ...

`Miasm`并不是第一个实现了这一功能的工具，但是，因为它已经满足所有实现`DSE`的必要条件,所以将其他功能添加到[main branch](https://bbs.kanxue.com/elink@6feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6h3q4Q4x3X3c8K6k6h3y4Q4x3V1k6E0K9h3q4K6L8g2)9J5c8Y4m8#2L8r3I4Q4x3V1j5#2x3e0R3`.)只是时间问题。

本文用几个例子和一个简单的`API`来教你怎么在脚本中使用`DSE`。

- 从被混淆的程序中恢复出它的算法

- 在不进行逆向的情况下重用别人的壳

- 自动破解`crackme`

概述：

- 使用`DSE`的破解教程

- `TigressVM`

  - 运行程序

  - 添加`DSE`

  - 深入研究

- 逆向一个`shellcode`

- 提高代码覆盖率-破解`crackme`

  - 概述

  - 构建新的输入

  - 自动探测

- 最后的想法

- 相关资源

这篇分析基于[1fb3326](https://bbs.kanxue.com/elink@9edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6h3q4Q4x3X3c8K6k6h3y4Q4x3V1k6E0K9h3q4K6L8g2)9J5c8X3y4G2L8h3#2A6N6q4)9J5c8U0q4X3j5U0x3K6x3U0j5J5y4X3x3&6j5K6f1&6x3U0f1K6x3o6c8T1x3r3p5H3y4o6y4V1x3U0q4T1z5h3p5^5j5h3q4T1y4$3k6W2z5e0j5`.)版`Miasm`

# 使用DSE的破解教程

动态符号执行（`DSE`，也叫做`concolic execution`)使用一个具体的值进行符号执行。这种符号执行可以避免由循环引起的路径爆炸，和在表达式太复杂或系统调用会生成值时给符号一个具体的值。

在这种方法中，我们每次只考虑一条路径，输入约束会随着程序路径的不断延伸而不断增长，当路径走到尽头时，将其中某一个约束条件取反，以到达新的代码路径。

接下来我们通过一个例子来介绍`DSE`。

思考下列代码：
