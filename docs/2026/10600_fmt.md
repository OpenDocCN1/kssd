
# [分享]C++基础八-类和对象

## 什么是类和对象

C++ 中的类（Class）可以看做是C语言中结构体（Struct）的升级。

结构体是一种构造类型，可以包含若干成员变量，每个成员变量的类型可以不同；可以通过结构体来定义结构体变量，每个变量拥有相同的性质。

C++ 中的类也是一种构造类型，但是进行了一些扩展，类的成员不但可以是变量，还可以是函数；通过类定义出来的变量有了新的名称，叫做对象。

## 类和对象的用法

类是创建对象的模板，一个类可以创建多个对象，每个对象都是类类型的一个变量。通过类创建对象的过程，叫做类的实例化，因此也称对象是类的一个实例，拥有类的成员变量和成员函数。

### 创建对象

```cpp
#include <iostream>
using namespace std;

typedef struct _STUDENT
{
    int nID;
    char m_szName[128];
    double nScore;
}STUDENT, * PSTUDENT;

void ShowInfo(PSTUDENT tagStudent)
{
    cout << "ID=" << tagStudent->nID << endl;
    cout << "Name=" << tagStudent->m_szName << endl;
    cout << "Score=" << tagStudent->nScore << endl;
}

class CStudent
{
public:
    CStudent()
    {
        cout << "CStudent" << endl;
    }
    ~CStudent()
    {
        cout << "~CStudent" << endl;
    }
    void ShowInfo()
    {
        cout << "ID=" << m_nID << endl;
        cout << "Name=" << m_szName << endl;
        cout << "Score=" << m_nScore << endl;
    }
public:
    int m_nID = 0;
    char m_szName[128] = { 0 };
    double m_nScore = 0;
};

int main()
{
    //定义结构体变量
    STUDENT tagStudent = { 123,"xiaoxin",99 };
    //传递结构体指针作为参数输出结构体信息
    ShowInfo(&tagStudent);
    //结构体大小
    cout << "sizeof struct:" << sizeof(tagStudent) << endl;

    //创建类对象
    CStudent Student;
    //类对象赋值
    Student.m_nID = 1;
    Student.m_nScore = 9;
    strcpy(Student.m_szName, "xiaoming");
    //直接输出类对象信息
    Student.ShowInfo();
    //类大小
    cout << "sizeof class:" << sizeof(Student) << endl;

    return 0;
}
```

1、`class`是C++ 中新增的关键字，专门用来定义类。

2、`CStudent`是类的名称，类名的首字母一般大写，以和其他的标识符区分开。

3、`{ }`内部是类所包含的成员变量和成员函数，它们统称为类的成员。

4、类的不同对象，数据成员是独享的，成员函数是共享的。

5、排除多态的情况下，类和结构体的成员变量在内存中的布局规划一致。

6、`public`是C++ 新增的关键字，它只能用在类的定义中，表示类的成员变量或成员函数具有公开的访问权限。

## 对象指针

### new 和 delete

在C语言中，动态分配内存用`malloc()`函数，释放内存用`free()`函数。在C++中，这两个函数仍然可以使用，但是C++又新增了两个关键字，`new`和`delete`：`new`用来动态分配内存，`delete`用来释放内存。

```cpp
int* pNumber = new int; //分配一个int数据类型的内存空间
delete pNumber; //释放指针pNumber指向的内存空间

char* szName = new char[128]; //分配128个连续的字符空间
delete[] szName; //释放指针szName指向的内存空间
```

和`malloc`一样，`new`也是在堆区分配内存空间，所以使用完毕后必须手动释放内存空间，否则只能等到程序运行结束时由操作系统回收，可能会造成内存泄漏。为了避免内存泄露，通常`new`和`delete`、`new[]`和`delete[]`操作符应该成对出现，并且不要和C语言中`malloc()`、`free()`一起混用。

### 创建对象指针

```cpp
CStudent* pStudent = new CStudent;
CStudent* pStudentArray = new CStudent[2];

pStudent->m_nID = 2;
pStudent->m_nScore = 8;
strcpy_s(pStudent->m_szName, "xiaobai");
pStudent->ShowInfo();

delete pStudent;
delete[] pStudentArray;
```

1、在栈上创建对象，形式和定义普通变量类似。

2、在堆上使用`new`关键字创建对象，必须要用一个指针指向它，使用完毕后用`delete`释放对象指针。

3、通过对象名字访问成员使用点号`.`，通过对象指针访问成员使用箭头`->`。

### github

- <mark class="encrypted">154K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3d9e0l9H3d9f1V1H3x3o6m8u0x3o6m8u0x3p5V1H3</mark>
- <mark class="encrypted">a22K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3d9e0l9H3d9f1V1H3x3o6m8u0x3o6m8u0x3p5V1H3</mark>

### bilibili

- <mark class="encrypted">a77K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3q4U0k6g2)9J5k6h3u0A6L8r3W2T1K9h3I4A6i4K6u0W2j5$3!0E0i4K6u0r3x3U0R3@1x3o6t1J5y4e0l9$3</mark>
- <mark class="encrypted">efaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3q4U0k6g2)9J5k6h3u0A6L8r3W2T1K9h3I4A6i4K6u0W2j5$3!0E0i4K6u0r3x3U0R3@1x3o6t1J5y4e0l9$3</mark>

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-8-30 17:28 被流照君编辑，原因：[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
