
# [原创]识别c++ RTTI的ida脚本

大型的c++程序，往往都开启了RTTI，利用此脚本能很方便的识别类名，极大的提高分析效率。

背景:

`RTTI`(`RuniTime TypeInfo`)是c++的一种运行时类型识别机制

当一个类的引用或者指针的值去进行“类型识别”，叫做动态识别

当一个类的实例去进行"类型识别"，叫做静态识别

动态识别需要编译打开了`/GR`开关才可有效，否则会引起一个运行时错误，而静态的不需要

对于大型工程而言，静态识别并没有多少意义

实现原理:

这里假设你对类的虚表非常熟悉,如果一个类存在虚函数，或者他的基类存在虚函数，

或者`virtual`继承了其他类，或者开启了`RTTI`，那么编译器会为该类生成一个虚表，

该表是一个该类需要虚实现的函数的指针表，一般类的头4个字节为该表的指针

当基类是多重继承的时候,非第一个继承的基类的虚表不是在头4字节

当有`RTTI`在的时候,虚表-4的位置是一个叫做`RTTI Complete Object Locator`的描述体

该描述体+`c`的位置是一个叫做`RTTI Type Descriptor`的描述体，该描述体+8的位置是一段

原始类名的`buf`，我们只要得到了该`buf`，就能得到类的名字.

实现:

`RTTI Type Descriptor`这个描述体在c++中叫做`type_info`，他的定义如下

```class type_info {
public:
    _CRTIMP virtual ~type_info();
    _CRTIMP int operator==(const type_info& rhs) const;
    _CRTIMP int operator!=(const type_info& rhs) const;
    _CRTIMP int before(const type_info& rhs) const;
    _CRTIMP const char* name() const;
    _CRTIMP const char* raw_name() const;
private:
    void *_m_data;
    char _m_d_name[1];
    type_info(const type_info& rhs);
    type_info& operator=(const type_info& rhs);
};
```

`typeid`作为一个c++关键字，给它传递一个任意类型的实例，他能够返回一个`type_info`的引用。

假如有下列类的定义

`Class Base {}`
`Class Derived : public Base {}`

-------------------------

代码片一

`Base instance;`
`typeid(instance);`

c++采用静态识别，对应的伪汇编代码为

```mov ecx,  offset Base_type_info
```

-------------------------

代码片二

`Base instance;`
`Base &rinstance  = instance;`
`typeid(rinstance);`

c++采用动态识别，对应的伪汇编代码为

```mov ecx, rinstance_vtbl
call __RTtypeid
mov ecx, eax
```

-------------------------

代码片三

该代码对于大型工程经常使用，目的就是为了知道传递的类型是不是需要的类型

达到 `RTTI` 的类型识别的初衷

```bool IsBaseClass(Base &base)
{
   return typeid(base) == typeid(Base);
}
```

代码的实质其实就是`strcmp`前面说到的`buf(type_info`中的`_m_d_name`)是不是和`Base`的`_m_d_name`相等

因为代码片三的存在，所以让我们利用这个`ida`脚本反`c++`类名的可能性存在。

--------------------------

前面所说，`buf(_m_d_name)`其实是一个c++类名的内部表达，比如我们的类`Base`的`_m_data`其实

显示为`.?AVBase@@`，则么还原成我们认识的 `class Base`这样的形式呢？

参考下列代码

```printf("%s", typeid(base).name());
```

该函数能正确的显示出`class Base`，所以，我们如果有了`raw_name`(前文的`buf`，又`type_info`的`_m_d_name`)，

可以通过`type_info`的 `const char *name() const`函数来实现

现在怎么调用这个函数？这里采用常用的欺骗方法，构造一个假的`type_info *`，然后去访问他

代码如下

```struct _fake_typeinfo
{
        void *vtbl;
        void *name;
        char raw_name[1024];
        _fake_typeinfo(const char *rname)
                : vtbl(NULL), name(NULL)
        {
                strcpy(raw_name, rname);
        }
};
_fake_typeinfo ti(".?AVBase@@);
type_info * pti = reinterpret_cast<type_info *>(&ti);
printf("%s", pti->name());
```

可以看到正确的显示出`class Base`了.

脚本：

`ida`无法利用c++的欺骗方法调用c++的内部库，但是脚本支持2个指令`exec`, `readstr`,

可以认为他是`idc`跟外界打交道的两个途径.

当需要吧一个`raw_name`转化成一个`name`的时候，`idc`先执行一个外部程序，把`raw_name`作为`commandline`传递

该外部程序把`name`生成一个文件，然后`idc`读取该文件的第一行来得到返回值.

该脚本需要光标在在有`RTTI`的工程的类的虚表的-4的位置（即`RTTI Complete Object Locator`所在的位置)执行，

并且存在一个`c:\111.exe`用来翻译`name`

执行后会给该`RTTI Complete Object Locator`备注上类的显示名字

```// begin copy here
// filename: GetCppRTTI.idc
// author: jjnet
#include <idc.idc>
static EnsureMakeDword(ea)
{
                MakeUnknown(ea, 4, DOUNK_SIMPLE);
                MakeDword(ea);
}
static EnsureMakeStr(ea)
{
  auto ea_end;
  ea_end = ea;
  while(Byte(ea_end)!=0 && ea_end-ea<=255) ++ea_end;
  ++ea_end; // append tial zero.
  MakeUnknown(ea, ea_end-ea, DOUNK_SIMPLE);
  MakeStr(ea, ea_end);
}
static CommentRTTI(ea)
{
        auto ea1;
        ea1 = Dword(ea);
        EnsureMakeDword(ea1);
        EnsureMakeDword(ea1+4);
        EnsureMakeDword(ea1+8);
        EnsureMakeDword(ea1+12);
        ea1 = Dword(ea1+12);
        EnsureMakeDword(ea1);
        EnsureMakeDword(ea1+4);
        EnsureMakeStr(ea1+8);
        return GetString(ea1+8, BADADDR, ASCSTR_C);
}
static main()
{
  auto str_type, raw_name, fp, name;
  str_type = GetLongPrm(INF_STRTYPE);
  SetLongPrm(INF_STRTYPE, ASCSTR_C);
  raw_name = CommentRTTI(ScreenEA());
        Exec("C:\\111.exe " + raw_name);
        fp = fopen("c:\\111.txt", "r");
        name = readstr(fp);
        fclose(fp);
        MakeRptCmt(ScreenEA(), name=="" ? raw_name : name);
        SetLongPrm(INF_STRTYPE, str_type);
}
// end copy
```

`123.exe`的原码

```#include <stdlib.h>
#include <stdio.h>
struct _fake_typeinfo
{
        void *vtbl;
        void *name;
        char raw_name[1024];
        _fake_typeinfo(const char *rname)
                : vtbl(NULL), name(NULL)
        {
                strcpy(raw_name, rname);
        }
};
int _tmain(int argc, _TCHAR* argv[])
{
        FILE *fp = fopen("C:\\111.txt", "w");
        if (argc != 2)  { fclose(fp); return -1; }
        type_info *p_tinfo = reinterpret_cast<type_info *>(new _fake_typeinfo(argv[1]));
        fprintf(fp, "%s", p_tinfo->name());
        fclose(fp);
        return 0;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
