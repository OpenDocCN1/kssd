
# 用 `FOR_EACH` 学习空空运算

> 原文：[`www.pediy.com/kssd/pediy12/128639.html`](https://www.pediy.com/kssd/pediy12/128639.html)

用 `FOR_EACH` 学习空空运算。

上一次我出了一道题：

`http://bbs.pediy.com/showthread.php?t=127740`

一道给 C++高级程序员的题写 `FOREACH`，居然没有一个人肯认真做。

有一个人回答，用：

```cpp
#define FOREACH BOOST_FOREACH
```

没错，我出这一道题就是因为看了 `BOOST`，发现它居然实现了一个这么奇妙的功能，我根本想不到用Ｃ＋＋居然能这么做！所以才希望大家能研究一下，究竟怎么样用Ｃ＋＋实现这么一个貌似不可能实现的功能。可惜居然没有一个人肯认真研究这个问题。

很多回贴都是说，意义不大。

什么叫意义大？什么叫意义不大？

这是一道题，你不会做，这就是意义。

这道题，会用到大量的模板运算，你做了它，就会学到这些东西，这就是意义。

如果你是高级程序员，你应该知道如何下手，怎么做，给你时间你总能做出来，而不是一筹莫展。这就是意义。

我希望大家踏实一点，多学一点Ｃ＋＋基础，有好处的。

还有人说，这有什么用？单是这个 `FOR_EACH`，已经很方便了。它比 `std::for_each` 方便多了，即使加了 `lambda` 支持，你试试就知道了。

也许会有人说，我把 `BOOST` 代码抄下来不就行了吗？`BOOST` 代码太复杂了，`include` 关系复杂，如果真把实现 `FOR_EACH` 用到的东西都抄下来，会有几十 `KB` 甚至更多。抄下来了你还是没有搞明白。

接下来，就让我们一点点研究它的实现。我是花了多天的时间，把 `BOOST` 代码一点点简化才得到的。不断感叹世间竟有如此聪明之人，想出这么绝妙的主意。

因为 `VC6` 的模板功能问题太多，请使用 `VS2005` 以上编译器测试代码。

## 第一章 如何区分数组类型？

为了研究这个复杂的问题，我们把问题先简单化。第一个小问题，如何知道一个变量的数据类型是不是数组？是不是一筹莫展，毫无思路？

原来，解决方法，是用模板的特例化功能。

模板的编译期运算，我给它起个名字叫“空空运算”。我们先引入空空运算的基础，两个常量 `true_` 和 `false_`：

```cpp
template< bool C_ > struct bool_
{
  enum { value = C_ };
};

typedef bool_<true> true_;
typedef bool_<false> false_;
```

怎么理解这几行代码？定义了一个空的结构，空的模板，实在看不出它有什么用。但这，就是整个空空运算的基石。

附件中的 `1.cpp`，演示了用模板的特例化功能，区分一个数据类型是不是数组。请仔细体会以上代码。

`苏州 WinMount` 继续招 C++程序员，请把简历发到 `support@winmount.com`

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | `1.cpp.txt` |
|---|---|

谢谢 `NeverGone` ，已下载《Modern C++ design》和 `loki`，正在学习。

这便是讨论的收获。

## 第二章 尝试支持数组

废话不多说，请看附件 `2.cpp`，这里用 `define` 实现了基本的 `FOR_EACH`：

```cpp
#define FOR_EACH2(VAR, COL)                 \
  auto_any_t _cur = BAIL::begin11( COL );    \
  auto_any_t _end = BAIL::end11( COL );      \
  for (bool _conti = true;                   \
  _conti && !BAIL::done1( _cur, _end, COL);  \
  _conti ? BAIL::next1( _cur, COL) : (void)0 ) \
    { _conti = false;                        \
  for (VAR = BAIL::deref1(_cur, COL); !_conti; _conti = true)
```

```cpp
void test()
{
  int tbl[] = {1,2,3,4,5,6};
  FOR_EACH2( int i, tbl )
  {
    std::cout << i << ' ';
  }
}
```

说明几点：

* 因为 `FOR_EACH` 后面是一个 `{}`，显示不可能定义为一个 `template` 或函数，只能是 `define`
* `FOR_EACH` 的第一个参数是 `int i`，如果写成通常的 `for`，那么怎么写 `i++` 呢？显然做不到。不得已只好用双 `for`，每次都给 `int i` 赋值
* 加了个 `_conti` 有两个用途，一是 `int i` 所在的 `for` 只能允许它运行一次。二是当它 `break` 时上一个 `for` 也能知道并退出。就这么复杂了
* 我们可以写一个 `begin11` 的模板，返回一个 `iterator1<T>::type` 的数据类型。但一旦从模板出来，用什么样的数据类型去保存它呢？`boost` 的做法，是转为 `auto_any<T>`，然后存到一个 `auto_any_t` 中，使用的时候用 `auto_any_cast` 可以转化回来。这个过程我至今还没有理解。

最后，用这个方法写出来的 `FOR_EACH` 有两个缺点，一是前面不能加 `if`，它虽然看起来是一行，实际却是多行。二是后面必须多写一个 `}` 号。因为用了双层的 `for`。你知道怎么解决这两个问题吗？请看下一章。

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | `2.cpp.txt` |
|---|---|

## 第三章 尝试支持数组之二

为了解决上一章提到的两个问题，增加一个 `set_false` 函数：

```cpp
inline bool set_false(bool &b)
{
    b = false;
    return false;
}
```

使用了一个技巧：

```cpp
if (int i = 0) {} else
```

见附件 `3.cpp`，至此已经完全支持数组了：

```cpp
void test()
{
  int tbl[] = {1,2,3,4,5,6,7};
  FOR_EACH( int i, tbl )
  {
    std::cout << i << ' ';
  }
}
```

且支持 `break`, `continue`, 前 `if`。

接下来，尝试增加对 `std::list`, `"hello"`的支持。

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | `3.cpp.txt` |
|---|---|

## 第四章 支持 `std::list`

稍作修改，支持 `std::list`, `std::vector`, `std::deque`：

```cpp
void test()
{
  std::vector<int> m;
  m.push_back(10);
  m.push_back(20);
  m.push_back(30);
  FOR_EACH( int i, m )
  {
    std::cout << i << ' ';
  }
}
```

接下来，尝试增加对 `"hello"`的支持。

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | `4.cpp.txt` |
|---|---|

## 第五章 支持 C++字串

继续修改，支持 `null-terminated C-style strings`，最后，我们的宏支持所有以下情况：

```cpp
void test()
{
  int tbl[] = {1,2,3,4,5,6,7};
  if (1)
  FOR_EACH( int i, tbl )
  {
    std::cout << i << ' ';
  }
  std::cout << std::endl;

  std::vector<int> m; // also support std::list, std::deque
  m.push_back(10);
  m.push_back(20);
  m.push_back(30);
    FOR_EACH( int i, m )
    {
        std::cout << i << ' ';
    }
  std::cout << std::endl;

  const char * hello = "hello, world\n";
    FOR_EACH( char ch, hello )
    {
        std::cout << ch;
    }
}
```

代码见附件，不知道还能不能更简单地实现。

关注的人太少了，不再更新....

| ![](img/12f2a5fbf8d3fd9a8bf145533ecd3281.jpg) | `5.cpp.txt` |
|---|---|
