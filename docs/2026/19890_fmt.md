
# 菜鸟之2015MSC第一题解题思路（适合入门级别的超详细版）

第一次发表文章，有些小紧张，怕被大牛笑话……超详细版见附件（图文并茂）。

这里添加图片、设置字体颜色等好费劲，咱们这个论坛有没有直接展示`doc`文件的插件呢？要是没有，迫切希望`kanxue`坛主能加个这个功能，这样上传文件就会好看好多，也非常方便。

下面是详细步骤（可惜就是没图片）：

### 第一步：软件运行后的界面和错误提示

下面是软件运行后的界面和点击按钮后的错误提示。

### 第二步：反编译源码，分析MainActivity类

下面我们从代码角度加以分析，反编译源文件，对软件运行流程有个大体了解，下面是`MainActivity`类中的方法。

我们主要关注`oncreate()`方法中的按钮点击事件部分代码，因为猜测这里有逻辑判断及错误提示框，下面我们来找错误提示框。

查找错误提示框的方法：
1. 通过提示框API及其跳转逻辑定位大体位置。
2. 通过显示内容验证（`R`文件`String`类中定位资源`0x7f0a0011`，找到`dialog_error_tips = 0x7f0a0011`，再在`String.xml`中查找`dialog_error_tips`对应的字符串）。

猜测上面的`showDialog()`即为正确的提示框，验证方法同上，进一步将线索聚焦到`if`的判断条件中，而条件中起决定作用的是`s2.equals(s3)`，现在需要理清`s1`、`s2`和`s3`。

### 第三步：分析s1、s2和s3的来源

要了解`s1`、`s2`必须分析`getTableFromPic()`和`getPwdFromPic()`，而`s3`的分析必须看`MainActivity.bytesToAliSmsCode(s1, s.getBytes("utf-8"))`。

#### `s1 = getTableFromPic()`的分析

`s1`实质就是读取图片资源到`abyte0`数组中，从位置`0x15d81`取`768`个字节到数组`abyte1`中，用`utf-8`编码成的字符串（`s3`从这个集合中选取非连续数据）。

#### `s2 = getPwdFromPic()`的分析

这个方法和上面的`s1 = getTableFromPic()`差不多，`s2`实质就是读取图片资源到`byte`数组中，从`0x16481`取`18`个字节到新数组，再用`utf-8`编码成的字符串。

#### `s3 = MainActivity.bytesToAliSmsCode(s1, s.getBytes("utf-8"))`的分析

`s3`实质就是`s1`字符串中指定下标（下标是用用户输入的数据转化来的）的字符连接成的字符串。

首先分析参数`s`为用户的输入`String s = edit.getText().toString();`。

`s.getBytes("utf-8")`：这行代码的意思是把用户输入的编码成字节数组作为参数，传递给`bytesToAliSmsCode(string, byte[])`处理，下面分析这个方法。

通过分析可以看出，用户输入的`s`字符串转换成数组，传到`bytesToAliSmsCode(string, byte[])`数组中，用`i`做下标取数组`abyte0[]`中的数据，再把取出来的数据做`0xff & abyte0[i]`运算后，把结果作为下标，用`s1.charAt(结果)`累加到`StringBuilder`中生成字符串。实质就是用用户输入的数据转化成下标，提取`s1`中的字符。

### 第四步：验证判断逻辑

`MainActivity`中`oncreate()`中的`if`判断就是决定成功失败的地方，主要就是判断`s2`和`s3`是否相等，通过前面我们知道`s2`是：

修改添加输入测试下面两个方法，结合输出会发现结果如下：

用用户输入的数字的ASCII值的十进制作为下标，`table`（`s3`）中取出数据和`s2`比较。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [第一题 解题思路.doc](javascript:void(0);)（925.50kb，144次下载）
