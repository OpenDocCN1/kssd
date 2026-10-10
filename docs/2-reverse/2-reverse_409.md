

# 识别 c++ RTTI 的 ida 脚本

> 原文：[`www.pediy.com/kssd/pediy10/61873.html`](https://www.pediy.com/kssd/pediy10/61873.html)

大型的 c++程序， 往往都开启了 RTTI， 利用此脚本能很方便的识别类名， 极大的提高分析效率.  
  
  
  
背景:  
RTTI(RuniTime TypeInfo)是 c++的一种运行时类型识别机制  
当一个类的引用或者指针的值去进行“类型识别",  叫做动态识别  
当一个类的实例去进行"类型识别", 叫做静态识别  
动态识别需要编译打开了/GR 开关才可有效， 否则会引起一个运行时错误， 而静态的不需要  
对于大型工程而言， 静态识别并没有多少意义  
实现原理:  
这里假设你对类的虚表非常熟悉, 如果一个类存在虚函数, 或者他的基类存在虚函数,   
或者 virtual 继承了其他类，或者开启了 RTTI, 那么编译器会为该类生成一个虚表，  
 该表是一个该类需要虚实现的函数的指针表，一般类的头 4 个字节为该表的指针  
(当基类是多重继承的时候, 非第一个继承的基类的虚表不是在头 4 字节)  
当有 RTTI 在的时候,  虚表-4 的位置是一个叫做 RTTI Complete Object Locator 的描述体  
该描述体+c 的位置是一个叫做 RTTI Type Descriptor 的描述体, 该描述体+8 的位置是一段  
原始类名的 buf, 我们只要得到了该 buf, 就能得到类的名字.  
  
  
  
  
实现:  
RTTI Type Descriptor 这个描述体在 c++中叫做 type\_info, 他的定义如下  
class type\_info {  
public:  
    \_CRTIMP virtual ~type\_info();  
    \_CRTIMP int operator==(const type\_info& rhs) const;  
    \_CRTIMP int operator!=(const type\_info& rhs) const;  
    \_CRTIMP int before(const type\_info& rhs) const;  
    \_CRTIMP const char\* name() const;  
    \_CRTIMP const char\* raw\_name() const;  
private:  
    void \*\_m\_data;  
    char \_m\_d\_name\[1\];  
    type\_info(const type\_info& rhs);  
    type\_info& operator=(const type\_info& rhs);  
};  
typeid 作为一个 c++关键字， 给它传递一个任意类型的实例， 他能够返回一个 type\_info 的引用。  
假如有下列类的定义  
Class Base {}  
Class Derived : public Base {}  
\-------------------------  
代码片一  
Base instance;  
typeid(instance);  
c++采用静态识别， 对应的伪汇编代码为  
mov ecx,  offset Base\_type\_info  
\-------------------------  
代码片二  
Base instance;  
Base &rinstance  = instance;  
typeid(rinstance);  
c++采用动态识别， 对应的伪汇编代码为  
mov ecx, rinstance\_vtbl  
call \_\_\_RTtypeid  
mov ecx, eax  
\-------------------------  
代码片三  
该代码对于大型工程经常使用, 目的就是为了知道传递的类型是不是需要的类型  
达到 RTTI 的类型识别的初衷  
bool IsBaseClass(Base &base)  
{  
   return typeid(base) == typeid(Base);  
}  
代码的实质其实就是 strcmp 前面说到的 buf(type\_info 中的\_m\_d\_name）是不是和 Base 的\_m\_d\_name 相等  
因为代码片三的存在， 所以让我们利用这个 ida 脚本反 c++类名的可能性存在。  
\--------------------------  
  
前面所说, buf（\_m\_d\_name）其实是一个 c++类名的内部表达， 比如我们的类 Base 的\_m\_data 其实  
显示为.?AVBase@@, 则么还原成我们认识的 class Base 这样的形式呢？  
参考下列代码  
printf("%s", typeid(base).name());  
该函数能正确的显示出 class Base， 所以， 我们如果有了 raw\_name(前文的 buf， 又 type\_info 的\_m\_d\_name)，  
可以通过 type\_info 的 const char \*name() const 函数来实现  
现在怎么调用这个函数？ 这里采用常用的欺骗方法， 构造一个假的 type\_info \*， 然后去访问他  
代码如下  
struct \_fake\_typeinfo  
{  
  void \*vtbl;  
  void \*name;  
  char raw\_name\[1024\];  
  \_fake\_typeinfo(const char \*rname)   
    : vtbl(NULL), name(NULL)  
  {  
    strcpy(raw\_name, rname);  
  }  
};  
\_fake\_typeinfo ti(".?AVBase@@);  
type\_info \* pti = reinterpret\_cast<type\_info \*>(&ti);  
printf("%s", pti->name());  
可以看到正确的显示出 class Base 了.  
  
  
  
脚本：  
ida 无法利用 c++的欺骗方法调用 c++的内部库， 但是脚本支持 2 个指令 exec, readstr,  
可以认为他是 idc 跟外界打交道的两个途径.  
当需要吧一个 raw\_name 转化成一个 name 的时候， idc 先执行一个外部程序, 把 raw\_name 作为 commandline 传递  
该外部程序把 name 生成一个文件， 然后 idc 读取该文件的第一行来得到返回值.  
该脚本需要光标在在有 RTTI 的工程的类的虚表的-4 的位置（即 RTTI Complete Object Locator 所在的位置)执行，  
并且存在一个 c:\\111.exe 用来翻译 name  
执行后会给该 RTTI Complete Object Locator 备注上类的显示名字  
  
  
// begin copy here  
// filename: GetCppRTTI.idc  
// author: jjnet  
  
#include <idc.idc>  
  
static EnsureMakeDword(ea)  
{  
    MakeUnknown(ea, 4, DOUNK\_SIMPLE);  
    MakeDword(ea);  
}  
static EnsureMakeStr(ea)  
{  
  auto ea\_end;  
  ea\_end = ea;  
  while(Byte(ea\_end)!=0 && ea\_end-ea<=255) ++ea\_end;  
  ++ea\_end; // append tial zero.  
  MakeUnknown(ea, ea\_end-ea, DOUNK\_SIMPLE);  
  MakeStr(ea, ea\_end);  
}  
  
static CommentRTTI(ea)  
{  
  auto ea1;  
    
  ea1 = Dword(ea);  
  EnsureMakeDword(ea1);  
  EnsureMakeDword(ea1+4);  
  EnsureMakeDword(ea1+8);  
  EnsureMakeDword(ea1+12);  
    
  ea1 = Dword(ea1+12);  
  EnsureMakeDword(ea1);  
  EnsureMakeDword(ea1+4);  
  EnsureMakeStr(ea1+8);  
  
  return GetString(ea1+8, BADADDR, ASCSTR\_C);  
}  
  
static main()  
{  
  auto str\_type, raw\_name, fp, name;  
  str\_type = GetLongPrm(INF\_STRTYPE);  
  SetLongPrm(INF\_STRTYPE, ASCSTR\_C);  
    
  raw\_name = CommentRTTI(ScreenEA());  
  Exec("C:\\\\111.exe " + raw\_name);  
  fp = fopen("c:\\\\111.txt", "r");  
  name = readstr(fp);  
  fclose(fp);  
  MakeRptCmt(ScreenEA(), name=="" ? raw\_name : name);  
    
  SetLongPrm(INF\_STRTYPE, str\_type);  
}  
// end copy  
  
  
// 123.exe 的原码  
  
#include <stdlib.h>  
#include <stdio.h>  
  
struct \_fake\_typeinfo  
{  
  void \*vtbl;  
  void \*name;  
  char raw\_name\[1024\];  
  \_fake\_typeinfo(const char \*rname)   
    : vtbl(NULL), name(NULL)  
  {  
    strcpy(raw\_name, rname);  
  }  
};  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{    
  FILE \*fp = fopen("C:\\\\111.txt", "w");  
  if (argc != 2)  { fclose(fp); return -1; }  
   type\_info \*p\_tinfo = reinterpret\_cast<type\_info \*>(new \_fake\_typeinfo(argv\[1\]));  
  fprintf(fp, "%s", p\_tinfo->name());  
  fclose(fp);      
  return 0;  
} 

// 加了点东西， 更容易知道基类拥有的派生类  
  
// author jjnet  
  
#include <idc.idc>  
  
static EnsureMakeDword(ea)  
{  
    MakeUnknown(ea, 4, DOUNK\_SIMPLE);  
    MakeDword(ea);  
}  
static EnsureMakeStr(ea)  
{  
  auto ea\_end;  
  ea\_end = ea;  
  while(Byte(ea\_end)!=0 && ea\_end-ea<=255) ++ea\_end;  
  ++ea\_end; // append tial zero.  
  MakeUnknown(ea, ea\_end-ea, DOUNK\_SIMPLE);  
  MakeStr(ea, ea\_end);  
}  
  
static CommentRTTI(ea)  
{  
  auto ea1, ea2, ea3, ea4, ea5, count;  
  EnsureMakeDword(ea);  
  ea1 = Dword(ea);  
  EnsureMakeDword(ea1);  
  EnsureMakeDword(ea1+4);  
  EnsureMakeDword(ea1+8);  
  EnsureMakeDword(ea1+12);  
  EnsureMakeDword(ea1+16);  
    
  ea2 = Dword(ea1+12);  
  EnsureMakeDword(ea2);  
  EnsureMakeDword(ea2+4);  
  EnsureMakeStr(ea2+8);  
  
  ea3 = Dword(ea1+16);  
  EnsureMakeDword(ea3);  
  EnsureMakeDword(ea3+4);  
  EnsureMakeDword(ea3+8);  
  EnsureMakeDword(ea3+12);  
    
  count = Dword(ea3+8);  
  ea4 = Dword(ea3+12);  
  while(count>0)  
  {  
    EnsureMakeDword(ea4);  
    ea5 = Dword(ea4);  
    EnsureMakeDword(ea5);  
    EnsureMakeDword(ea5+4);  
    EnsureMakeDword(ea5+8);  
    EnsureMakeDword(ea5+12);  
    EnsureMakeDword(ea5+16);  
    EnsureMakeDword(ea5+20);      
    ea4 = ea4 + 4;  
    count = count -1;  
  }  
  EnsureMakeDword(ea4);  
  
  
  return GetString(ea2+8, BADADDR, ASCSTR\_C);  
}  
  
static main()  
{  
  auto str\_type, raw\_name, fp, name;  
  str\_type = GetLongPrm(INF\_STRTYPE);  
  SetLongPrm(INF\_STRTYPE, ASCSTR\_C);  
    
  raw\_name = CommentRTTI(ScreenEA());  
  Exec("C:\\\\111.exe " + raw\_name);  
  fp = fopen("c:\\\\111.txt", "r");  
  name = readstr(fp);  
  fclose(fp);  
  MakeRptCmt(ScreenEA(), name=="" ? raw\_name : name);  
  ExtLinA(ScreenEA(), 0, "-----------------------------------------------------");  
    
  SetLongPrm(INF\_STRTYPE, str\_type);  
}