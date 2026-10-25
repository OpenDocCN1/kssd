
debug

```c++
#include <stdio.h>
class Person {
public:
Person() {
age = 20;
}
int age;
};
int main(int argc, char\* argv[]) {
Person\* p = new Person;
//为了突出本节讨论的问题，这里没有检查new运算的返回值
p->age = 21;
printf("%d\n", p->age);
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_C4756DVDX58CNQC.png)

总结：

- 使用`new`申请堆空间之后，需要调用构造函数来完成对象数据成员的初始化
- 如果堆空间申请失败，则不调用构造函数
- 如果`new`执行成功，返回值是对象的首地址
- 识别堆对象的构造函数：重点分析`new`的双分支结构，在判定`new`成功的分支迅速定位并得到构造函数

### 10.1.3. 参数对象

当对象作为函数参数时，会调用赋值构造函数

debug

```c++
#define \_CRT\_SECURE\_NO\_WARNINGS
#include <stdio.h>
#include <string.h>
class Person {
public:
Person() {
name = NULL;//无参构造函数，初始化指针
}
Person(const Person& obj) {
// 注：如果在复制构造函数中直接复制指针值，那么对象内的两个成员指针会指向同一个资源，这属于浅拷贝
// this->name = obj.name;
// 为实参对象中的指针所指向的堆空间制作一份副本，这就是深拷贝了
int len = strlen(obj.name);
this->name = new char[len + sizeof(char)]; // 为便于讲解，这里没有检查指针
strcpy(this->name, obj.name);
}
void setName(const char\* name) {
int len = strlen(name);
if (this->name != NULL) {
delete[] this->name;
}
this->name = new char[len + sizeof(char)]; // 为便于讲解，这里没有检查指针
strcpy(this->name, name);
}
public:
char\* name;
};
void show(Person person) { // 参数是对象类型，会触发复制构造函数
printf("name:%s\n", person.name);
}
int main(int argc, char\* argv[]) {
Person person;
person.setName("Hello");
show(person);
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_36CCWJB6D4T3J9U.png)

调用赋值构造函数

![图片描述](upload/attach/202211/939298\_XVVKNBT9GDSV8GZ.png)

### 10.4. 返回对象

返回对象与参数对象类似，都会使用赋值构造函数。但是，两者使用时机不同

- 当对象为参数时，在进入函数前使用赋值构造函数
- 返回对象则在函数返回时使用赋值构造函数

debug

```c++
#define \_CRT\_SECURE\_NO\_WARNINGS
#include <stdio.h>
#include <string.h>
class Person {
public:
Person() {
name = NULL;//无参构造函数，初始化指针
}
Person(const Person& obj) {
// 注：如果在复制构造函数中直接复制指针值，那么对象内的两个成员指针会指向同一个资源，这属于浅拷贝
// this->name = obj.name;
// 为实参对象中的指针所指向的堆空间制作一份副本，这就是深拷贝了
int len = strlen(obj.name);
this->name = new char[len + sizeof(char)]; // 为便于讲解，这里没有检查指针
strcpy(this->name, obj.name);
}
void setName(const char\* name) {
int len = strlen(name);
if (this->name != NULL) {
delete[] this->name;
}
this->name = new char[len + sizeof(char)]; // 为便于讲解，这里没有检查指针
strcpy(this->name, name);
}
public:
char\* name;
};
Person getObject() {
Person person;
person.setName("Hello");
return person; //返回类型为对象
}
int main(int argc, char\* argv[]) {
Person person = getObject();
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_CREENC2SZNSYDGX.png)

调用`getObject`函数

![图片描述](upload/attach/202211/939298\_MDCTBWD3YD6GGF5.png)

## 10.2. 析构对象的出现时机

### 10.2.1. 局部对象

重点考察作用域的结束处，当对象所在作用域结束后，将销毁作用域所有变量的栈空间，此时便是析构函数出现的时机。析构函数同样属于成员函数，因此在调用的过程中也需要传递`this`指针。

debug

```c++
#include <stdio.h>
class Person {
public:
Person() {
age = 1;
}
~Person() {
printf("~Person()\n");
}
private:
int age;
};
int main(int argc, char\* argv[]) {
Person person;
return 0; //退出函数后调用析构函数
}
```

ida

![图片描述](upload/attach/202211/939298\_QZXAD2CCUDYN397.png)

### 10.2.2. 堆对象

用`detele`释放对象所在的空间，`delete`的使用便是找到堆对象调用析构的关键点

debug

```c++
#include <stdio.h>
class Person {
public:
Person() {
age = 20;
}
~Person() {
printf("~Person()\n");
}
int age;
};
int main(int argc, char\* argv[]) {
Person\* person = new Person();
person->age = 21; //为了便于讲解，这里没检查指针
printf("%d\n", person->age);
delete person;
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_BZGX74ZK6C3EXRT.png)

析构代理函数

![图片描述](upload/attach/202211/939298\_6MT8FV7D2GFQVWM.png)

# 十一、虚函数

对于具有虚函数的类而言，构造函数和析构函数的识别过程更加简单。而且，在类中定义虚函数后，如果没有提供构造函数，编译器会生成默认的构造函数。

对象的多态需要通过虚表和虚指针完成，虚表指针被定义在对象首地址处，因此虚函数必须作为成员函数使用。

## 11.1. 虚函数的机制

当类中定义有虚函数，编译器会将给类中所有虚函数的首地址保存在一张地址表，这张表被称为虚函数地址表，简称虚表。同时还会在类中添加一个隐藏数据成员，称为虚表指针，该指针保存虚表的首地址，用于记录和查找虚函数。

### 11.1.1. 默认构造函数初始化虚表指针的过程

- 没有编写构造函数时，编译器默认提供构造函数，以完成虚表指针的初始化
- 虚表中虚函数的地址排列顺序：先声明的虚函数的地址会被排列在虚表靠前的位置
- 第一个被声明的虚函数的地址在虚表的首地址处

debug

```c++
#include <stdio.h>
class Person {
public:
virtual int getAge() { //虚函数定义
return age;
}
virtual void setAge(int age) { //虚函数定义
this->age = age;
}
private:
int age;
};
int main(int argc, char\* argv[]) {
Person person;
//int size = sizeof(Person);
//定义了虚函数后，因为还含有隐藏数据成员虚表指针，所以Person大小为8
//printf("%d",size); 8
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_MKSJJGRK6BC89K5.png)

构造函数

![图片描述](upload/attach/202211/939298\_92YJMRXFE5FTSJC.png)

### 11.1.2. 调用自身类的虚函数

直接通过对象调用自身的成员虚函数，编译器使用了直接调用函数方式，没有访问虚表指针，而是间接获取虚函数地址。

debug

```c++
#include <stdio.h>
class Person {
public:
virtual int getAge() { //虚函数定义
return age;
}
virtual void setAge(int age) { //虚函数定义
this->age = age;
}
private:
int age;
};
int main(int argc, char\* argv[]) {
Person person;
person.setAge(20);
printf("%d\n", person.getAge());
return 0;
}
```

ida

![图片描述](upload/attach/202211/939298\_QNMK574766FBRKF.png)

`setAge`函数

![图片描述](upload/attach/202211/939298\_XXJEAQ8BE5HGT7U.png)

### 11.1.3. 析构函数分析

执行析构函数时，实际上是在还原虚表指针，让其指向自身的虚表首地址，防止在析构函数中调用虚函数时取到非自身虚表，从而导致函数调用错误。

debug

```c++
#include <stdio.h>
class Person {
public:
~Person() {
printf("~Person()\n");
}
public:
virtual int getAge() { //虚函数定义
return age;
}
virtual void setAge(int age) { //虚函数定义
this->age = age;
}
private:
int age;
};
int main(int argc, char\* argv[]) {
Person person;
person.setAge(20);
printf("%d\n", person.getAge());
return 0;
}
```

ida析构函数分析

![图片描述](upload/attach/202211/939298\_JY584QJ4PVH99H6.png)

## 11.2. 虚函数的识别

判断是否为虚函数

- 类中隐式定义了一个数据成员
- 该数据成员在首地址处，占一个指针大小
- 构造函数会将此数据成员初始化为某个数组的首地址
- 这个地址属于数据区，是相当固定的地址
- 在这个数组中，每个元素都是函数地址
- 这些函数被调用时，第一个参数是`this`指针
- 在这些函数内部，很有可能堆`this`指针使用间接的访问方式

# 十二、从内存角度看继承和多重继承

## 12.1. 识别类与类之间的关系

debug

```c++
#include <stdio.h>
class Base { //基类定义
public:
Base() {
printf("Base\n");
}
~Base() {
printf("~Base\n");
}
void setNumber(int n) {
base = n;
}
int getNumber() {
return base;
}
public:
int base;
};
class Derive : public Base { //派生类定义
public:
void showNumber(int n) {
setNumber(n);
derive = n + 1;
printf("%d\n", getNumber());
printf("%d\n", derive);
}
public:
int derive;
};
int main(int argc, char\* argv[]) {
Derive derive;
derive.showNumber(argc);
return 0;
}
```

ida

![图片描述](upload/attach/202212/939298\_2ZNZMPFWEJ9E32G.png)

子类`Derive`构造函数

![图片描述](upload/attach/202212/939298\_KTKEGENG5W9QC3P.png)

子类`Derive`析构函数

![图片描述](upload/attach/202212/939298\_4SJYY3W9EA45E6W.png)

**人类说话方法的多态模拟类结构**

debug

```c++
#include <stdio.h>
class Person { // 基类——“人”类
public:
Person() {}
virtual ~Person() {}
virtual void showSpeak() {} // 这里用纯虚函数更好，相关的知识点后面会讲到
};
class Chinese : public Person { // 中国人：继承自人类
public:
Chinese() {}
virtual ~Chinese() {}
virtual void showSpeak() { // 覆盖基类虚函数
printf("Speak Chinese\r\n");
}
};
class American : public Person { //美国人：继承自人类
public:
American() {}
virtual ~American() {}
virtual void showSpeak() { //覆盖基类虚函数
printf("Speak American\r\n");
}
};
class German : public Person { //德国人：继承自人类
public:
German() {}
virtual ~German() {}
virtual void showSpeak() { //覆盖基类虚函数
printf("Speak German\r\n");
}
};
void speak(Person\* person) { //根据虚表信息获取虚函数首地址并调用
person->showSpeak();
}
int main(int argc, char\* argv[]) {
Chinese chinese;
American american;
German german;
speak(&chinese);
speak(&american);
speak(&german);
return 0;
}
```

`speak`函数分析，虚函数的调用过程是间接寻址方式

![图片描述](upload/attach/202212/939298\_SNMH76XD66QF9F8.png)

## 12.2. 多重继承

debug
