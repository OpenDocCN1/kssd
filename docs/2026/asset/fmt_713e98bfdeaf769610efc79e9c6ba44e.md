

# [原创]探索C++虚函数在内存中的表现形式及运行机制

# 探索C++虚函数在内存中的表现形式及运行机制

> 本文是我在科锐学习完一阶段的一个总结文章，几个月前就写好了，最近看雪支持了`markdown`，所以就赶紧发出来支持下。同时感谢科锐戚老师的耐心教导！

- 文章概要：从简单地例子来进行探索，但是篇幅较长，建议先仔细阅读下目录结构再阅读文章，便于跳读和回顾。<br>前面一段是简单地铺垫，然后观察单层继承下的虚表指针和虚表，分析了不同情况下它们的表现形式，并手动模拟了虚函数的跳转。在中间插了一段需要注意的间接调用问题和重载、覆盖、隐藏的区别，来指出使用继承和虚函数时需要注意的地方。最后分析了在构造析构、多重继承和菱形继承的下虚表指针和虚表的表现形式。

- 如果仅仅需要了解虚函数的实现机制，不想看这么多的内存数据就直接看这篇文章吧 <mark class="encrypted">32fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3K9r3q4G2k6h3I4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5&6y4o6R3H3y4e0q4Q4x3V1j5`.</mark> （但如果对虚函数实现机制还比较模糊，也建议先花几分钟看下这边文章再回头来看内存吧）

---

编译环境：`win7 32bit vs2013` 平台工具集：`Visual Studio 2013 - Windows Xp (v120_xp)`

---

[toc]

# 引言：虚函数与多态

> 本部分是基本概念的简单表述，可以跳过 ^\_^

在面向对象编程最重要的思想就是多态，而多态是通过虚函数来实现的，虚函数在继承中使用。这里我们就探究探究虚函数的编译器对虚函数的实现及其在内存中的表现形式

## 继承与虚函数

### 什么情况下需要用到继承

在创建多个类时，会出现由于数据的重复而导致接口（方法/函数）的重复，于是就产生了冗余。我们可以通过组合的方式来解决冗余问题，但又产生了很多层不必要的调用。这时就需要继承——一个类可以继承/获取另一个类的部分数据成员和方法——来解决以上问题（这里为什么说是部分数据成员和方法，这和类权限问题相关，不做讨论）

### 一个简单的继承例子

游戏中有各种各样的角色，现在我们需要创建两个角色：枪手和骑兵。我们也只需要他们基本的操作：攻击和血量
于是就需要两个类：枪手类和骑兵类。这时我们发现，他们都需要存储血量的数据成员和攻击的成员方法，这时就可以创建一个具有这两个的士兵类让他们继承

#### 代码

> 为了演示方便就没有分开头文件、声明和实现

```cpp
#include <iostream>

// 士兵类，具有血量数据成员和攻击方法

class CSoldier
{
public:
    CSoldier()
        : m\_nBlood(0x20)
    {
        /\* Nothing to do \*/
    }

    ~CSoldier()
    {
        std::cout << "~CSolder()" << std::endl;
    }

    void attack()
    {
        std::cout << "Soldier Attack!" << std::endl;
    }

protected:
    int m\_nBlood;
};

// 枪手类，继承士兵类，并覆盖士兵类攻击方法

class CGuner
    : public CSoldier
{
public:
    CGuner()
        : CSoldier()
    {
        /\* Nothing to do \*/
    }

    ~CGuner()
    {
        std::cout << "~CGuner()" << std::endl;
    }

    void attack()
    {
        std::cout << "Guner Attack!" << std::endl;
    }
};

// 骑士类，继承士兵类，并覆盖士兵类攻击方法

class CKnight
    : public CSoldier
{
public:
    CKnight()
        : CSoldier()
    {
        /\* Nothing to do \*/
    }

    ~CKnight()
    {
        std::cout << "~CKnight()" << std::endl;
    }

    void attack()
    {
        std::cout << "Knight Attack!" << std::endl;
    }
};

// 测试

int main()
{
    CGuner gunerA;
    CKnight knightA;

    gunerA.attack();
    knightA.attack();

    return 0;
}
```

输出：
![](upload/attach/201709/751258\_9zrm9xaapatejju.png)

正常调用攻击方法和析构

#### 问题

上面的代码执行是正常的，但是如果有很多的兵种并且出现了很多个对象实类再这样一个个调用就很麻烦了，尤其是场景不同需要操作的对象实类也不同。
这时就想到把所有子类对象赋给父类指针再进行操作，现实中也是这样做的
代码：

```cpp
int main()
{
    // 创建多个不同种类的士兵
    CGuner\* pGunerA = new CGuner;
    CGuner\* pGunerB = new CGuner;
    CKnight\* pKnightA = new CKnight;
    CKnight\* pKnightB = new CKnight;

    // 将需要操作的士兵放进一个其父类指针数组里
    const int nSoldierNum = 4;
    CSoldier\* const pAllSoldier[ nSoldierNum ] =
        { pGunerA, pGunerB, pKnightA, pKnightB };

    // 调用攻击方法
    for(int i = 0; i < nSoldierNum; ++i)
    {
        pAllSoldier[ i ]->attack();
    }

    // 释放空间
    for(int i = 0; i < nSoldierNum; ++i)
    {
        delete pAllSoldier[ i ];
    }

    return 0;
}
```

输出：
![](upload/attach/201709/751258\_b036105sjekae1b.png)

此时结果完全不对，并没有调用子类的攻击方法，而是直接调用了父类的攻击方法，而且在释放时也只是调用了父类的析构，并没有调用子类的析构！

#### 通过`virtual`关键字声明虚函数解决问题

对于上面的问题通过虚函数就可以解决
具体方法，在父类的攻击方法和析构声明前面加上`virtual`关键字，子类可加可不加，但为了代码可读性一般都会加上

修改后的士兵类：

```cpp
class CSoldier
{
public:
    CSoldier()
        : m\_nBlood(0x20)
    {
        /\* Nothing to do \*/
    }

    virtual ~CSoldier()
    {
        std::cout << "~CSolder()" << std::endl;
    }

    virtual void attack()
    {
        std::cout << "Soldier Attack!" << std::endl;
    }

protected:
    int m\_nBlood;
};
```

输出：

![](upload/attach/201709/751258\_7rfasx5mqoctgcq.png)

这时调用攻击方法和析构就都正常了

## 纯虚函数与抽象类（接口类）

在上面的三个类中，我们发现士兵类根本不需要有实际的攻击方法，因为它只是一个类别，而具体的攻击方法应交给具体的士兵来实现的。这时就引入一个概念：抽象类。

抽象类：不能实例化出对象的类

实现抽象类的方法：

```cpp
class CSoldier
{
public:
    // 通过在成员方法的声明后面加上 = 0 的方式来声明这是一个抽象类
    // 这样就不能写任何的实现了，一写就报错，用该类实例对象时也报错
    virtual void attact() = 0;

    // VS2013下，抽象类可以写构造的声明和实现，但不能只写构造的声明
    CSoldier()
        : m\_nBlood(0x20)
    {
        /\* Nothing to do \*/
    }

    // VS2013下，抽象类必须要写虚析构的声明和实现
    // 只写声明会报链接错误，不写虚析构则不能正常调用析构
    virtual ~CSoldier()
    {
        std::cout << "~CSolder()" << std::endl;
    }

    // 在VC6下，如果在析构后面加上 = 0 则必须要写析构的实现
    // （VS2013也支持这个写法，但就算不写也必须写实现）
    // virtual ~CSoldier() = 0;

protected:
    int m\_nBlood;
};
```

# 虚函数实现原理探索

## 继承后类的内存情况

### 普通继承

> 基于上面第一段没有加`virtual`关键字的代码

测试代码：

```cpp
// 给 CGuner 和 CKnight 加上下面两个数据成员
int m\_nBlood; // 覆盖父类同名数据成员，构造时初始化为 0x30
int m\_nTest;  // 构造时初始化为 0x10

int main()
{
    CSoldier soldierA;
    CGuner gunerA;
    CKnight knightA;
    return 0;
}
```

内存观察：
![](upload/attach/201709/751258\_g1po1um1hifouq0.png)
![](upload/attach/201709/751258\_c5iiih968gmu6ne.png)
![](upload/attach/201709/751258\_s7hecqr9gb2ytof.png)
![](upload/attach/201709/751258\_29qdsmuf4ggptgo.png)

我们发现，在父类内存中的数据成员虽然被子类覆盖了但还是出现在了子类内存中（如果父类有`private`数据成员也会出现在子类内存中，在此不做演示），如果有了解过组合内存情况的话就会发现继承的内存情况和组合的完全相同

总结：在普通继承后，子类中包含父类所有数据成员，并且父类数据成员出现在子类内存起始位置

### 有虚函数的继承

> 基于上面在士兵类攻击方法和析构声明前加`virtual`关键字的代码（非抽象类）

测试代码：

```cpp
// 给 CGuner 和 CKnight 加上下面两个数据成员
int m\_nBlood; // 构造时初始化为 0x30
int m\_nTest;  // 构造时初始化为 0x10

int main()
{
    CSoldier soldierA;
    CGuner gunerA;
    CKnight knightA;
    return 0;
}
```

内存观察：
![](upload/attach/201709/751258\_ine64n6ewbz90pg.png)
![](upload/attach/201709/751258\_4iq9uw5yxiluk2s.png)
![](upload/attach/201709/751258\_biq07n0784tt7yk.png)
![](upload/attach/201709/751258\_vi0b0j4fovxrgci.png)

上图中绿线框中的内容是与没加`virtual`关键字内存情况的区别之处，同时我们也发现了内存中新增的部分是一个指针，这就是传说中的虚表指针，而这个指针指向的内容就是传中的虚表

## 虚表指针和虚表

### 类内存中的虚表指针

![](upload/attach/201709/751258\_i5a7r7oudzpu8gc.png)

通过监视窗口看到三个类中都出现了一个指针`_vfptr`放在对象内存的前四个字节（父类数据成员的前面），但又不像数据成员，因为它在不同的类中其值也不同
这就是虚表指针，至于它在不同类中值不同、到底是什么时候变化的？将会在后面虚表指针和虚表初始化的时机中阐述

### 虚表指针指向的虚表

先来看一波连续的内存图：
![](upload/attach/201709/751258\_qrfez0lqwdy1286.png)
![](upload/attach/201709/751258\_f7ut6m03g1vqp67.png)
![](upload/attach/201709/751258\_328eug8zebho0lw.png)
![](upload/attach/201709/751258\_8ng7hx35z4rdtbz.png)
![](upload/attach/201709/751258\_0ge2ruekdz6l9yl.png)
![](upload/attach/201709/751258\_je0qoyzhdkb1am0.png)
![](upload/attach/201709/751258\_gh5rrh6kev5hsps.png)
![](upload/attach/201709/751258\_pp5011lzxv93h9s.png)
![](upload/attach/201709/751258\_85s7m89yx00hmp4.png)
![](upload/attach/201709/751258\_qujxi0pltig2w2g.png)
![](upload/attach/201709/751258\_j03x9zd8zsm50rn.png)

仔细分析上面的内存和反汇编图，我们看到每个类的虚表指针分别指向一张虚表，而这几张虚表又分别存储了上面加了`virtual`关键字的虚函数的地址，虚表第一个内容指向的是对应的类的析构，第二个内容指向的是对应的类的`attact()`方法

所以我们发现虚表其实就是一个函数指针数组

### 一个简单地总结

对类继承后的虚表指针和虚表的内存进行分析发现每个类中都有一张存储自己虚函数地址的表，这样通过指针调用时，将先找到这张表，然后在通过这张表中的内容找到对应的方法，所以将避免没有虚函数时通过父类指针操作子类时调用的不是子类方法的问题了

而这一系列寻找对应的虚函数的操作是编译器在编译时生成查找代码，在运行时程序根据这些查找代码来找到对应的函数并调用的。可以称作动态绑定

### 虚表在继承后各种情况下的内存形态

#### 子类覆盖父类所有虚方法

上面对于继承后类的内存分析采用的就是子类覆盖父类所有方法的情况，子类虚表的内容和父类虚表的内容完全不同，也说明了，子类覆盖了父类所有的方法

#### 子类不覆盖父类虚方法

将前面的内存分析代码中`guner`类的析构和`attact()`方法去掉，采用完全继承父类的方法后`guner`虚表的内存形态：
![](upload/attach/201709/751258\_3pfr45kh7cp4g3m.png)
![](upload/attach/201709/751258\_zxulou8xo2gmqfv.png)
![](upload/attach/201709/751258\_4o530u0szapnegn.png)
![](upload/attach/201709/751258\_gu2ec1p9fdsd3o0.png)

我们看到，`gunerA`和`soldierA`中的虚表指针依然不同，而虚表指针指向的虚表的析构也是不同的（因为此时编译器给`CGuner`类了一个默认析构，所以还是相当于覆盖了`CSoldier`的虚析构方法），但`gunerA`虚表中的第二项和`soldierA`虚表中的第二项完全相同、指向同一个方法`CSoldier::attact()`

这就说明了，在子类不覆盖父类虚方法时，子类的虚表中依然存放父类虚方法的地址。这也是合理的，因为子类继承了父类的虚方法，所以要能够通过子类的虚表找到继承过来的方法

#### 子类覆盖部分父类虚方法

> 上一步演示的子类不覆盖父类的方法实质上是子类覆盖父类的部分方法，因为编译器提供了一个默认析构。所以在此就不再进行过多演示。

结论：子类覆盖父类的虚方法地址会放在虚表对应的位置，而不覆盖的虚方法对应的位置存放的是父类虚方法的地址（关于虚表存放的内容的顺序将在后面探究）

#### 子类新增虚方法

在前面内存分析代码的`CKnight`骑士类中增加一个上马的方法：

```cpp
virtual void toHorsel()
{
    std::cout << "Knight to horsel!" << std::endl;
}
```

此时骑士类的内存情况：
![](upload/attach/201709/751258\_ddlwcevl0jiafi9.png)
![](upload/attach/201709/751258\_g2vcpis0w35qhm9.png)
![](upload/attach/201709/751258\_axowl3xl5yec58n.png)
![](upload/attach/201709/751258\_snksdhhzd6z24l5.png)
![](upload/attach/201709/751258\_kykkjc0pt36inpy.png)

我们看到，新增的`toHorsel()`方法添加到了虚表后面的位置

结论：子类新增的虚函数会出现在子类虚表的尾部，这样在用父类指针指向子类对象是依然能够通过虚表找到子类覆盖父类的虚方法地址

### 虚表内容的顺序

> 这个规律很简单，不做演示了^\_^

虚表的顺序是按照虚方法在父类声明的顺序排列的，析构也不例外。如果子类新增了虚函数，在子类的虚表中出现在父类虚方法地址后面的内容是按照新增方法在子类中声明顺序排列的，即使子类新增方法的声明插在覆盖父类的虚方法声明的中间也不影响上面的排列规律

## 虚表指针和虚表初始化的时机

