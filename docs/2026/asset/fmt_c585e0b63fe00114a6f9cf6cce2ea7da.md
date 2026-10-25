
both operands are converted to numbers;

- all other operations: // 其它所有

  operand(s) are converted to `longs`;

If any of the `long` operands is 64bit, the other operand is converted to 64bit too.

如果任何一个 `long` 型操作数是 64 位的，其它的操作数也会转换成 64 位的。

There is one notable exception concerning type conversions: if one operand is a string and the other is zero (`0`), the

n a string operation is performed. Zero is converted to an empty string in this case.

这里有一个类型转换中值得注意的例外：如果一个操作数是 `string` 类型，并且其它的都是 `0`，则 `string` 转换会被执行，在这种情
况下，零会被转换为空字符串。

The `&` operator is used to take a reference to a variable. References themselves can not be modified once created. Any

 assignment to them will modify the target variable. For example:

操作符“`&`”用于获取一个变量的引用。引用他们自己不能被一次创建。任何对他们的赋值将会修改目标变量。例子：

``` 
        [IDC 代码]

        auto x, r;

        r = &x;

        r = 1;   // x is equal to 1 now
```

References to references are immediately resolved:

引用到引用是“直接”决定的：

```
        [IDC 代码]

        auto x, r1, r2;

        r1 = &x;

        r2 = &r1; // r2 points to x
```

Since all non-object arguments are passed to functions by value, references are a good way to pass arguments by refer
ence.

所有的非对象参数是以值传递到函数，引用是一个引用传递参数的好方法！

------------------------------------------------------------------------

### IDC: Predefined symbols

IDC: 预定义符号

The following symbols are predefined in the IDC preprocessor:

下面的符号是 IDC 预处理中预定义的符号：

[IDA 预定义符号列表]

`\_NT\_`           IDA is running under MS Windows

`\_LINUX\_`        IDA is running under Linux

`\_MAC\_`          IDA is running under Mac OS X

`\_UNIX\_`         IDA is running under Unix (linux or mac)

`\_EA64\_`         64-bit version IDA

`\_GUI\_`          GUI version of IDA

`\_TXT\_`          Text version of IDA

`\_IDA\_VERSION\_`  The current IDA version, for example: "5.1"

These symbols are also defined when parsing C header files.

分析 C 头文件时，这些符号也被定义。

------------------------------------------------------------------------

### IDC: Slices

The slice operator can be applied IDC objects are strings.

操作符 `slice` 可以应用在 IDC 的字符串对象上。

For strings, the slice operator denotes a substring:

对于字符串来说，操作符 `slice` 指示一个子字符串：

`str[i1:i2]` - substring from i1 to i2. i2 is excluded
             - 子字符串从 i1 到 i2，i2 是不被包含的

`str[idx]`   - one character substring at 'idx'.
               this is equivalent to `str[idx:idx+1]`
             - 在“idx”的单一字符的子字符串。
             - 相当于 str[idx:idx+1]。

`str[:idx]`  - substring from the beginning of the string to idx
               this is equivalent to `str[0:idx]`
             - 子字符串从开始到 idx，相当于 str[0:idx]。

`str[idx:]`  - substring from idx to the end of the string
               this is equivalent to `str[idx:0x7fffffff]`
             - 子字符串从 idx 到结尾，相当于 str[idx:0x7fffffff]。

Any indexes that are out of bounds are silently adjusted to correct values. If i1 >= i2, empty string is returned. Ne
gative indexes are used to denote positions counting from the end of the string.

对于任何超出界限的索引将会寂静地调整为正确的值。如果 i1 大于等于 i2，空字符串将会被返回。负索引被用于指示从字符串后
面计算。

String slices can be used on the right side of an assignment. For example:

字符串 `slices` 可以被用在赋值语句的右面，例子：（← 翻译不确定）

```
  [IDC 代码]

  str[0:2] = "abc";
```

will replace 2 characters at the beginning of the string by "abc".

将会替换在字符串 "abc" 中的前两个字符。

For objects, the slice operator denotes a subset of attributes. It can be used to emulate arrays:

对于对象，操作符 `slice` 指示属性的一个子集。它可以用来模仿数组：

```
  [IDC 代码]

  auto x = object();

  x[0] = value1;

  x[1] = "value2";
```

`x[i1:i2]` denotes all attributes with numeric values between i1 and i2 (i2 is excluded).

`x[i1:i2]` 指示所有在 i1 和 i2 之间（不包含 i2）带有数字的属性。

Any non-numeric attributes are ignored by the slice operator.

所有非数字的属性用操作符 `slice` 时会被忽略。

------------------------------------------------------------------------

### IDC: Exceptions

IDC: 异常

Any runtime error generates an exception. Exceptions terminate the execution. It is possible to catch an exception in
stead of terminating the execution:

任何运行时错误将会产生一个异常。异常停止执行。可以捕捉异常来代替停止执行：

```
        [IDC 代码]

        auto e;

        try

        {

          ... some statements that cause a runtime error...

        }

        catch ( e )

        {

          // e holds the exception information
          // it is an instance of the exception class
        }
```

The `try`/`catch` blocks can be nested. If the current function has no `try`/`catch` blocks, the calling function will be exa
mined, and so on, until we find a `try`/`catch` block or exit the main function. If no `try`/`catch` block is found, an unhan
dled exception is reported.

`try`/`catch` 块可以嵌套。如果当前的函数没有 `try`/`catch` 块，将会检查调用者，直到找到了一个 `try`/`catch` 块或者退出了 `main` 函
数。如果没有 `try`/`catch` 块被找到，将会报告一个未处理的异常。

It is also possible to throw an exception explicitly. Any object can be thrown. For example:

也可以显式地抛出一个异常。任何对象都可以抛。例子：

```
        [IDC 代码]

        throw 5;
```

will throw value '5'.

将会抛出值 “5”。

------------------------------------------------------------------------

翻译：`hmlky`   日期时间：15:08 2013年7月26日星期五

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
