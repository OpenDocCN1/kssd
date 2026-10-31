
# [分享] C 语言基础七——结构体

## 结构体的定义

`short`、`int`、`long`、`char`、`float`、`double`这六个关键字代表 C 语言里的六种基本数据类型。C 语言中，还提供了`struct`结构体数据类型，结构体是一种构造类型，由若干成员组成，每个成员可以是一个基本数据类型，也可以又是一个构造类型，因此结构体可以用来存放一组不同类型的数据。例如：

```c
typedef struct _Student
{
    int nId;
    char szName[16];
    double dbHeight;
} Student, *pStudent;
```

我们定义了一种`Student`结构体数据类型和一种`pStudent`结构体指针数据类型，可以使用`Student`、`pStudent`定义变量。

## 结构体的大小

- 结构体对齐方式：自然对齐。
- 结构体数据成员对齐规则：各成员变量存放的起始地址相对于结构体起始地址的偏移量是`sizeof`（基本类型）的整数倍；如果成员为非基本类型，偏移量为其子成员中最大的基本类型长度的整数倍。
- 结构体整体对齐规则：结构体的总大小是其成员中最大基本类型长度的整数倍。

## 结构体变量和结构体指针

```c
typedef struct _Student
{
    int nId;
    char szName[16];
    double dbHeight;
} Student, *pStudent;

/* 变量的定义 */
// 1. 结构体
Student tagStuA;

// 2. 指针
pStudent pTagStuB;
Student* pTagStuC;

/* 变量的初始化 */
// 1. 大括号初始化
Student tagStuC = {202101, "xiaoxin", 100.0};

// 2. 单个成员赋值
tagStuC.dbHeight = 101.0;

// 3. 同结构体变量可以给同结构体变量赋值
Student tagStuD = tagStuC;
pStudent pTagStuE = &tagStuC;

// 变量的访问，. 和 -> 运算符的优先级比 * 运算符高
printf("%s\n", tagStuC.szName);
printf("%s\n", pTagStuE->szName);
```

## 结构体应用

- 结构体数组
- 结构体指针数组
- 结构体作为参数传参
- 结构体指针作为参数传参

## 传值和传地址的区别

我们知道，一个结构体可以有多个成员。当我们定义结构体的时候，底层会根据我们的结构体成员算出对齐值，然后根据对齐值大小申请内存。

```c
typedef struct _Student
{
    int nId;
    char szName[16];
    double dbHeight;
} Student, *pStudent;

void ChangeValueA(Student tagStu)
{
    tagStu.szName = xiaoxin;
}

void ChangeValueB(pStudent pTagStu)
{
    pTagStu->szName = xiaoxin;
}

int main()
{
    Student tagStu;

    // 传值
    ChangeValueA(tagStu);

    // 传地址
    ChangeValueB(&tagStu);
}
```

当传入`tagStu`给函数的时候，函数中的`tagStu`相当于重新申请的变量，拷贝外部的实际参数`tagStu`。我们在函数内部修改`tagStu`不会改变实际参数的值，这就是传值。

如果我们想修改实际参数的值，需要把地址传过去，函数拷贝的是地址，根据地址修改里面的值，就可以改变实参的值，这就是传地址。传值会消耗内存，传地址效率更高。

## 浅拷贝 shallow copy、深拷贝 deep copy

1. 结构体中只有基本类型成员时，结构体变量可以直接赋值给相同类型的结构体变量，浅拷贝没问题。
2. 结构体中包含指针成员时，浅拷贝会造成野指针问题。
3. 深拷贝：为结构体指针成员重新分配内存拷贝数据。
4. C 语言中编译器默认结构体浅拷贝，深拷贝由程序员自己实现。
5. 深拷贝会造成内存浪费的问题。
6. 写时拷贝 copy-on-write：采用引用计数机制，共享内存数据，解决野指针和内存浪费的问题。

### GitHub

`3bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3d9e0l9H3d9f1V1H3x3o6m8u0x3o6m8u0x3p5V1H3`

### Bilibili

`566K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3q4U0k6g2)9J5k6h3u0A6L8r3W2T1K9h3I4A6i4K6u0W2j5$3!0E0i4K6u0r3x3U0R3@1x3o6t1J5y4e0l9$3`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
