
# [分享]C++基础四-`const`关键字

C++的`const`关键字允许指定一个语义约束，编译器会强制实施这个约束，允许程序员告诉编译器某值是保持不变的。如果在编程中确实有某个值保持不变，就应该明确使用`const`，这样可以获得编译器的帮助。

## 修饰普通类型的变量

对于基础数据类型的 `const`：

- 当遇到 `const` 声明时编译器在符号表放入常量。
- 编译过程中若发现使用常量则直接以符号表中的值进行替换。
- 编译过程中若发现下述情况则给对应的常量分配存储空间：

    1. 对 `const` 常量使用了 `extern`。
    2. 对 `const` 常量使用了 `&` 操作符。

- 其中即使对应分配了空间，但仍然不会使用存储空间中的值。

    1. 对于分配了空间的局部变量，在栈分配空间。
    2. 对于分配了空间的全局变量或静态局部变量，在静态存储区分配空间，修改时会导致运行崩溃。

```
/*
修改局部空间的const常量，赋值时生效，使用时依然使用符号表中的常量，输出结果不变。
*/
const int nVal = 9;
*(int*)&nVal = 8;
cout << "nVal=" << nVal << endl;
```

## 修饰指针变量

`const`修饰指针变量有以下三种情况：

- `const` 修饰指针，则指针为不可变量。
- `const` 修饰指针指向的内容，则内容为不可变量。
- `const` 修饰指针和指针指向的内容，则指针和内容都为不可变量。

```
int nValA = 8;
int nValB = 9;
const int* pConstVal = &nValA;
pConstVal = &nValB;
//*pConstVal = nValB 不行
int* const constpVal = &nValA;
*constpVal = nValB;
//constpVal = &nValB 不行
const int* const constpConstVal = &nValA;
//*constpConstVal = nValB 不行
//constpConstVal = &nValB 不行
```

## 修饰函数参数和返回值

修饰形参时，用相应的变量初始化`const`常量，在函数体内，按照`const`所修饰的部分进行常量化。

```
void function(const int Var);
//传递过来的参数在函数内不可变（无意义，因为Var本身就是形参）
void function(const char* Var);
//参数指针所指内容为常量不可变
void function(char* const Var);
//参数指针本身为常量不可变（无意义，因为char* Var是形参）
```

修饰函数返回值时，一般情况下，`const`修饰返回值多用于操作符的重载。

## 修饰类对象

`const`修饰类对象时，其对象中的任何成员都不能被修改，对于对象指针和对象引用也是一样。`const`修饰的对象，该对象的任何非`const`成员函数都不能调用该对象，因为任何非`const`成员函数都会有修改成员变量的可能。

## 修饰成员变量

`const`修饰的成员变量不能被修改，同时只能在初始化列表中被初始化，因为常量只能被初始化，不能被赋值。

### 初始化列表

赋值是使用新值覆盖旧值，构造函数先为成员变量开辟空间然后为其赋值，不是初始化。而初始化列表开辟空间和初始化是同时完成的，直接给予一个值，所以`const`成员变量一定要在初始化列表中完成。

## 修饰成员函数

`const`成员函数表示该成员函数不能修改类对象中的任何非`const`成员变量。一般`const`写在函数的后面：

```
class CTest
{
public:
CTest();
~CTest();
void func() const;
private:
int m_nVal;
}
```

如果某个成员函数不会修改成员变量，那么最好将其声明为`const`，因为`const`成员函数不会对数据进行修改，如果修改，编译器将会报错。

对于`const`类对象，只能调用类中的`const`成员函数，所以`const`修饰成员函数的作用主要就是限制对`const`对象的使用。

## `const`常量与`define`宏定义的区别

### 编译器处理方式不同

`define`宏是在预处理阶段展开，`define`常量从未被编译器看见，因为在预处理阶段就已经被替换了，而`const`常量在编译阶段使用。

### 类型和安全检查不同

`define`宏没有类型，不做类型安全检查，仅仅是字符替换，并且在字符替换时可能会产生意料不到的错误。`const`常量有明确的类型，在编译阶段会进行类型检查。

### 存储方式不同

`define`宏是字符替换，有多少地方使用，就会替换多少次，不会分配内存。

编译器通常不会为`const`常量分配空间，只是将它们保存在符号表内，使他们成为一个编译期间的一个常量，没有读取内存的操作，效率高。

### github：18eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3d9e0l9H3d9f1V1H3x3o6m8u0x3o6m8u0x3p5V1H3

### bilibili：647K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3q4U0k6g2)9J5k6h3u0A6L8r3W2T1K9h3I4A6i4K6u0W2j5$3!0E0i4K6u0r3x3U0R3@1x3o6t1J5y4e0l9$3

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-8-24 11:30 被流照君编辑，原因：更改了关于修饰普通类型变量的部分情况

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
