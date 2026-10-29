
其实也是调用的拷贝构造函数，`v4`指向的拷贝好的类在栈上的首地址，第一个`*`代表读取`vptr`，第二个`*`代表读取`vtable`的第一个函数（`child2->showLOG();`就是`ChildClass`的第一个虚函数），然后再把自己（`v4`）当成`this`传递给这个虚函数调用

### 4.拷贝构造拷贝父类

- 详见下图

// 由编译器为我们生成的拷贝构造函数

![](upload/attach/202203/868525_5STXM8MASHUYSBT.png)

```
ChildClass(const ChildClass &child){
this->m = child.m;
this->n = 12;
LOGD("called ChildClass拷贝构造函数");
}
```

// 由我们自己编写的拷贝构造函数

![](upload/attach/202203/868525_P9ZCE8CSST24DQV.png)

// 虚函数表

![](upload/attach/202203/868525_A2TY7TG5T34UXU6.png)

由此可见调用子类的拷贝构造函数会先调用父类的构造函数，然后在调用当前类的拷贝构造，这里的`off_85600`就是 `vptr` ，从虚函数表中也可以看见，子类覆盖了父类的虚函数就会指向子类的虚函数，若没有覆盖，表项中依旧是指向父类的函数地址，而且顺序是按照父类的虚函数表顺序排列，子类中父类没有的虚函数会按顺序继续排在后面，不同类的虚函数表其实都是在编译期就已经确定了的，不同类的虚函数表处于临近的内存区域

#### 类的 构造/析构 函数调用时机

- 详见下图（`ChildClass`中新增了一个析构函数）

// 新增析构函数

![](upload/attach/202203/868525_FAQXTCRXEVBNBH3.png)

#### 类继承权限

- 类的继承权限并不会影响子类继承父类子类所拥有的父类的成员变量个数，换句话说，不管父类的成员变量是什么权限，之类都完全拥有一份父类的成员变量的拷贝（这里就不展示）

#### 类型的强转

- 主要是针对 `dynamic_cast` 向下转型的情况

```
BaseClass* baseTmp = dynamic_cast<BaseClass*>(child1);
if (baseTmp!= nullptr){
baseTmp->showLOG1();
}
BaseClass* baseTmp1 = static_cast<BaseClass*>(child1);
if (baseTmp1!= nullptr){
baseTmp1->showLOG1();
}
ChildClass* baseTmp2 = dynamic_cast<ChildClass*>(baseTmp);
if (baseTmp2!= nullptr){
baseTmp2->showLOG1();
}
```

// 向下转型

![](upload/attach/202203/868525_VPMG5M8S7AB9MBG.png)

1. 由上两图可见对于类 向上转型 `dynamic_cast` 和 `static_cast` 本质是一样的，没有做任何处理

2. `dynamic_cast` 向下转型的时候是借助了 RTTI 机制，就是我们前面图中看到的`vptr->vtable` 除了虚函数以后的指针标识该类的类型用于动态类型转换，同样也是`typeid`这个操作符的信息来源，具体可以参考 <a href="elink@874K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Q4x3X3g2T1K9h3q4F1j5$3S2W2L8X3N6Q4x3X3g2F1k6i4c8Q4x3V1k6$3K9h3g2%4i4K6u0r3x3U0x3@1x3#2)9J5k6h3S2@1L8h3H3`.">这篇文章</a>

3. 其实虚表什么的都是在编译期间就已经完全确定了，之前还误解以为动态类型转换中的向上转型可以让该子对象调用已经被子对象覆盖的父对象的方法，想多了想多了... 但是如果真想实现这样的"向上转型"也不是不行，借助指针去操作虚函数表即可 ↓

// 实现所谓的"向上转型"

![](upload/attach/202203/868525_VTNZ7RYY23ECWZ3.png)

// 效果图

![](upload/attach/202203/868525_B5UY68EE2HDS3P2.png)

最后两条日志可见，我们对同一对象调用 `showLOG()` 一个是父函数，一个是子函数，对应代码 815 和 819 行

#### 简介 lambda

- 细节介绍参考 <a href="elink@206K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6h3y4H3M7s2u0W2k6X3g2J5k6h3&6U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4N6Q4x3V1k6U0M7s2m8Q4x3V1k6D9j5h3&6Y4N6h3q4Y4k6g2)9J5c8X3I4S2L8h3u0V1j5b7`.`.">**这个**</a> 和 <a href="elink@8deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3^5y4o6x3I4y4o6b7%4y4l9`.`.">**这篇文章**</a>

引用传递和值传递

![](upload/attach/202203/868525_DKYJADPTMXGF9FN.png)

#### 关于lambda表达式分为三部分解析

##### 1 . 表达式位于类中 （为了看到最原始的实现，不要开编译器优化）

```
#define xASM(x) __asm __volatile__ (x)
using namespace std;
class testA{
private:
int tempInt0 = 123;
int tempInt1 = 321;
int tempInt2 = 110;
public:
testA(){
testFunction();
}
NOINLINE
void testFunction(){
[&]()-> void {
LOGD("testA anonymous_0 -> tempInt2:%d",this->tempInt2);
}();
[]()-> void {
xASM("MOV r4, r0":::"r4");
int** tmp = nullptr;
xASM("MOV %0, r4":"=r"(tmp)::);
LOGD("testA anonymous_1 -> tempInt0:%d",***(tmp + 0x1));
}();
[]()-> void {
xASM("MOV r4, r0":::"r4");
int tmp = 0;
xASM("LDR r4, [r4, #0x8]":::"r4");
xASM("LDR r4, [r4, #0x4]":::"r4");
xASM("MOV %0, r4":"=r"(tmp)::"r4");
LOGD("testA anonymous_2 -> tempInt1:%d",tmp);
}();
}
};
```

构造以及调用testFunction

![](upload/attach/202203/868525_AVSZ34HHJGVBM54.png)

testFunction汇编也可以明显看到被IDA识别为了lambda表达式

![](upload/attach/202203/868525_SQ32FMVFFK8G286.png)

从这里我们可以看到虽然源码后面两个lambda表达式虽然没有捕获参数，但是依旧有一个栈地址的传递(可以理解为一个空 this)

传递栈上地址逐个相差一个指针长度

![](upload/attach/202203/868525_RCV2UE3XV4UCQ9N.png)

0x4*6 = 24(0x18) sp+0x4 sp+0x8 sp+0xc

- 上述源代码中没有表现出来，即便是空 lambda 实现，编译器依旧会传递一个栈上地址过去，这里也不做展示了

- 然后后面两个 lambda 实现主要是为了实践，即使不捕获任何的参数，依旧可以拿到类实例，以及去读取类成员变量

- 在类里面的 lambda 可以理解为对 () 的重载（被IDA也是识别为重载 operator()）

读取类成员变量日志

![](upload/attach/202203/868525_474B2TZDVKC3TDE.png)

##### 2 . 表达式位于类外无捕获参数

```
NOINLINE
void testB(){
auto testNoCatch = [](int s) -> int {
LOGD("testB called lambda function : %d", ++s);
return s;
};
LOGD("testB testNoCatch ---> %p",*testNoCatch);
// using TypeFunc = int(*)(int);
auto testNoCatchNew = *testNoCatch;
testNoCatchNew(5);
testNoCatch(10);
}
```

表达式位于类外无捕获参数

![](upload/attach/202203/868525_N9PPQY2USV7TNGQ.png)

中间函数用来返回lambda函数真实的地址

![](upload/attach/202203/868525_GZ8YV9C53TTCSXQ.png)

中间跳板函数

![](upload/attach/202203/868525_4KC4GDYXQGYKS49.png)

lambda函数的实现,和普通函数没有啥差别

![](upload/attach/202203/868525_M5NGQ63FBT9YGHF.png)

- 类外 lambda 函数，*lambda 都会生产这样的一个跳转逻辑

- 类内 lambda 函数不管有没有捕获参数都是直接理解为匿名类重载 () 运算符

testB 日志

![](upload/attach/202203/868525_K7FPNN6CRG7PBKR.png)

##### 3 . 表达式位于类外有捕获参数

```
NOINLINE
void testC(){
int a = 10;
int* b = new int(11);
auto c = make_unique<int>(12);
auto* d = new float(13);
auto testNoCatch = [](int s,int t) -> void {
LOGD("testC called lambda function : %d", s+t);
};
auto testCatch = [=,&c,&d](int s) -> void {
LOGD("testC called lambda function : {%d} | {%d，%d，%d，%f}", ++s,a,*b,*c,*d);
};
// LOGD("testC testNoCatch typeid:%s | testCatch typeid:%s",typeid(testNoCatch).name(),typeid(testCatch).name());
LOGD("testC ------ ");
testNoCatch(3,4);
testCatch(2);
// &testCatch 拿到的是他准备好的参数在栈区的首地址
LOGD("testC &testCatch %p", &testCatch);
LOGD("testC &testNoCatch %p", &testNoCatch);
void* funcAddress = nullptr;
auto testCatchNew= [=,&c,&d,&funcAddress](int s) -> void {
// xASM("MOV %0, lr":"=r"(funcAddress)::);
// [&funcAddress]()-> void{
// if (funcAddress == nullptr) xASM("MOV %0, lr":"=r"(funcAddress)::);
// }();
LOGD("testC called lambda function : {%d} | {%d，%d，%d，%f}", ++s,a,*b,*c,*d);
};
testCatchNew(3);
LOGD(" %p ", funcAddress);
free(b);
}
```

IDA反汇编

![](upload/attach/202203/868525_6C929UXT9UM9P7Y.png)

testC 日志

![](upload/attach/202203/868525_HQZ4MDZ2Y36RU54.png)

- 从 `sub_319EC(v7, 3, 4)` → `testNoCatch(3, 4)`

可以看出 [] 捕获的参数其实都在第一个参数，lambda的传参在 第二个参数往后，结合把lambda理解为一个重载 () 运算符的类也是自洽的

- 从 `sub_31A4C(v6, 2)` → `testCatch(2);`

再去对应看v6的参数，也就可以更加理解，lambda 表达式引用传递和值传递的区别，源码中的 c为一个类 （理解为→ 构造 : `sub_3198C(v10, &v9);` | 析构 : `sub_31BB4(v10);`），栈传参的时候源码中的引用传递放在最前面，其次按顺序传递参数

- 从 `sub_3198C(v10, &v9);` 和 `sub_31BB4(v10)`

`sub_3198C(v10, &v9)` → `auto c = make_unique<int>(12);`

`sub_31BB4(v10)` → 作用域结束，对unique指针的析构

- 从 上图 29 30 行可见：对带捕获参数的 lambda 表达式取地址得到的只是 匿名类(分配在栈上)的首地址，其实从栈的角度看也是待传参数数组的首地址

- 没有带捕获参数的 lambda 表达式 基本上可以等价于一个普通函数，函数地址通过 * 来获得（编译器针对表达式特殊处理的）；带捕获参数的 lambda 表达式 不能使用 * ，如果使用 & 只能获得该匿名类首地址，而且 匿名lambda类的构造函数可以理解为inline构造

- lambda 表达式可以使用 `[=]` / `[&]` 捕获外部 值传递 / 引用传递，编译器只会把使用到的变量按照对应传递方式传递给匿名lambda类，没用到的变量不会被拷贝

由此上结论我们可以将 dobby hook 稍微封装一下

![](upload/attach/202203/868525_VVKSXRD23KN3ENC.png)

registerHook的重载第三个参数（Callback）本来是想用模板的但是好像不太行

得到一个类似于java函数回调一样的写法

这里srcCall可以用一个变长参数简写一下代码

![](upload/attach/202203/868525_ABAYEYTMBHWEQQ7.png)

##### 函数返回对象

```
class testClass {
public:
int a = 10;
int b = 11;
int c = 12;
int d = 13;
testClass(){
LOGD("testClass");
}
void callTest() {
LOGD("callTest");
}
};
NOINLINE
testClass getTestClass() {
auto tmp = testClass();
LOGD("getTestClass %p %d",&tmp,tmp.c);
return tmp;
}
void testRetClass() {
testClass tt = getTestClass();
LOGD("testRetClass b -> %d",tt.d);
}
```

![](upload/attach/202203/868525_43EGASK5Z22345D.png)

- “SUB SP, SP, #24” 4*6 总计开辟了 6 个位置，最下面的那个位置(fp-0x4) 用于存放栈检查的fp或者说成sp

- 正常情况下 `class()` 构造出来的类就在当前函数栈中，但是这里有一个特例：对于在 函数`getTestClass` 中 创建在栈上的`class(tmp)`，实际上他真实存在的位置是在 函数`testRetClass` 的栈上 位置 {fp-0x8,fp-0x14},最顶上的那个位置（sp）是空的（在这里分配栈最小差值0x8），结合上述描述再去看地址 0x4F5FC 就是logd中的第三个参数 “tt.d”

ps：如果这里的 class 只有三个成员变量，这里的 “SUB SP, SP, #0x18” 将会变成 “SUB SP, SP, #0x10”,刚好用满栈的四个位置

- 对于地址 0x4F5E8 这里的这个函数调用就是对 类`testClass` 的初始化，传递了第一个地址（函数testRetClass栈地址）进去 对 int a，b，c，d 的初始化就放在 “LOGD("testClass");” 之前

##### 模板类的实现

模板类的实现

![](upload/attach/202203/868525_G2A42YZDYBZJ9NQ.png)

实现

![](upload/attach/202203/868525_7J3TNDZ27QPUGDM.png)

- 它的上一层是一个 plt 跳转

- 使用模板对应生成了多个实现方法

该文章作为日常学习理解的记录，理解可能又不准确的地方，欢迎大佬们指出 0.0

#### 单/多继承

- 单继承

测试源码

|  |  |

| --- | --- |
