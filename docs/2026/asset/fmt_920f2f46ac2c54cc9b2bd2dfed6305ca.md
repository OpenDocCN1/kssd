
# [原创]浅析和介绍如何在delphi中定位要分析的函数

我喜欢废话，所以先废话下。

![](https://bbs.kanxue.com/view/img/face/1.gif) 因为今天有重要的事情要做，所以就一直忙到现在了。不过因为一直在干活，头脑还是很兴奋，想了想，最近在delphi应用的分析上面有了一点点的经验，今天和大家分享下，希望可以帮到你。

但是呢，时间有点晚，而且文章也是从我自己逆向时候做的笔记里修改的，可能会有些难理解，甚至有错误的地方，这个万望指正，我肯定改的。

因为是笔记，所以里面也有摘的其他人的东西的，而且也引用了之前自己的一篇文章，希望见到了熟悉的段子，理解下。毕竟是介绍和笔记嘛，也要从低到高的 :)...

只要程序员还在用delphi，我们就还需要熟悉borland delphi的结构。

而且相当有价值的一个部分是，现在有大量的不人道的病毒程序员啊，人家写个病毒也不怕体积大，10KB的程序用delphi的控件来写，成了200KB。坏的是分析人员的事，想简单分析下流程都只能看字符串和导入表了。我们这就开始研究如何定位delphi的函数吧。

//----------

## 一、从 `rsrc` 段入手

一开始看一个delphi程序，重要的第一要看的肯定是`rsrc`段，因为windows的API处理资源时候使用了硬编码的`.rsrc`字符串，所以delphi程序中不管怎么样做加壳，`rsrc`段肯定是存在的。

而delphi的程序的源代码是分为pascal源代码(`.PAS`, `.DPR`, `.INC`)和资源文件 (`.RES`, `.RC`, `.DFM`, `.DCR`)二种的，而资源文件在编译之后，就是几乎完全源代码形式的储存在`rsrc`段中的`rcdata`类型的资源中。

为什么这个`rcdata`重要，原因是在delphi里，form中在运行时(进程中)对用户代码的调用，都是通过保存在`rcdata`中的函数符号名称来进行的。这样，我们不仅仅可以通过`rcdata`定位特定事件的处理函数，也可以通过任意一个用户事件Handler，向上跟踪找到这个包含字符串的结构，来定位其他自己感兴趣的函数。

通常的delphi小程序，这样的分析就足够了，只是，这还不够浅析咧。

想要分析好目标，我们要从PE的入口点开始分析。

## 二、PE入口点与初始化表

首先delphi的可执行文件无论是`exe`还是`dll`类型的，都有一个`InitRouterTable`，其中保存了在初始化时挨个调用到的`routines`。

```
struct tagInitTable
{
    DWORD dwNum;  //初始化函数的个数(是初始化，结束化不包括在里面，也就是说只是FunTable元素个素的一半)
    PFUNTABLE pFunTable;
} * InitRoutineTable;
typedef struct tagFunTable
{
    void (* Initialization1)(void);
    void (* Finalization1)(void);
    ...
}FUNTABLE, *PFUNTABLE;
```

`PFUNTABLE`中的初始化函数和结束化函数是成对出现的，在调用这个表中的函数时候，delphi的循环每次是+0x8的偏移，于是Initialization时总会跳过Finalization的函数。

但是一个很沮丧的消息是，在这个初始化，还有完成初始化的列表的函数中，基本是找不到用户自定义的函数的，至少在我遇到的程序中没有。

### 典型的带form的delphi exe入口

```
push    ebp
mov     ebp, esp
add     esp, 0FFFFFFF4h
mov     eax, offset InitRoutineTable
call    @@InitExe//
mov     eax, ds:off_442C20
mov     eax, [eax]
call    unknown_libname_291
mov     ecx, ds:off_442AB4
mov     eax, ds:off_442C20
mov     eax, [eax]
mov     edx, off_441498
call    @TApplication@CreateForm
mov     eax, ds:off_442C20
mov     eax, [eax]
call    @TApplication@Run
call    @@Halt0
```

`InitExe`，`unknown_libname_291`，`TApplication::CreateForm`，`TApplication::Run` 这些都是delphi的库函数，可以在VCL的`system.pas`中找到。

`InitExe`会调用一个叫做`StartExe`的库函数，来从`InitRoutineTable`读取出所有的`FunTable`，并且挨个执行Init的Routine。

`TApplication::CreateForm`则顾名思义，创建程序的主Form，这个就是delphi初始化的主要流程。传递给它的第一个参数是名称为`TApplication`的类的指针，而`edx`则是类似于c++的类成员函数的第一个参数，其指向的是

`TCustomForm`类的`TMetaClass`结构的指针，会被用来构造程序的CostumForm类型的主form。这个函数，其实也是delphi中一个普通的类为自己重载的`ClassCreate`构造函数，特别之处只是构造函数的是inline在`TApplication`的

成员函数中的，只是在入口点，所以被特别命名了。

`TApplication::Run`中，通常要先用`AddExitProc`把from的析构函数添加到退出时要执行的流程里，然后就是标准的进入`TApplication::HandleMessage` delphi的消息循环了。

`Halt0`则是delphi退出入口点的流程，当没有保存DLL方式进入口点的`dwReason`，即`exe`类型时的其值为0，在`halt`的最后就会调用`ExitProcess`结束进程的执行。

### 标准的delphi dll入口

```
push ebp
mov ebp,esp
add esp,-3C
mov eax, InitRouterTable
call InitLib  //
call @@Halt0
```

`InitLib`是Delphi的一个库函数，从它的参数`InitRouterTable` 可以定位到`DllMain`中会被执行的流程的数组。和`InitExe`的功能是类似的

一般的，如果DLL中也有窗体，需要进行消息循环的话，也就是说，在DLL里也调用了`TApplication::CreateForm`来创建`CostumForm`主form的程序中，在入口点会先创建一个新线程，然后，再在新线程中执行

`TApplation_CreateClass`进行`TApplation`类的构造，以及执行`TApplation::CreateForm`和进入`TApplication::Run`消息循环。以避免DLL载入窗口的过程阻塞了原应用程序的执行。

在`Halt0`中，是DLL的时候，会判断一个全局结构里保存的进入`DllMain`的`Reason`，这个值不为0时，就意味着是DLL的入口点，会调用一个专为`DllMain`返回准备的库函数`System::_16618`，它是以

```
leavel
retn    0Ch
```

结尾的。

//----------

上面的就是delphi的默认入口点。相信你也跟我一样不知其所谓，这是因为delphi的实现，全部都是用类来实现的!

而且delphi的入口点，功能其实只是调用类的构造函数，区别只是怎么样的调用类的构造函数。

而具体的程序如何初始化的，完全是由类的构造来决定的。这个类，就是`CostumForm`，没错，delphi中的form也完全就是一个类。

关于delphi的class逆向的一个实例性质循环渐进的说明，可以看下面的文章

`ef2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3j5X3u0K6K9s2c8E0L8q4)9J5c8V1u0n7f1K6k6Q4x3V1k6H3k6h3c8A6P5e0j5&6x3K6g2Q4x3X3g2Z5N6r3@1`.

或者同一篇文章在`http://bbs.pediy.com/showthread.php?t=5476` #7 英文原版，以及我自己的杨白劳翻译。这篇文章中有一个笔误的地方是`TmetaClass`被写成`ClassInit`了。

在这儿，只简单叙述下delphi中的class的构造函数，以方便明白下面介绍的`TMetaClass`

在delphi中，一个类的构造函数都是单独存在的一个函数，不像c++中总是inline初始化过程到构造类对象的代码位置。同时，除非是自定义的构造函数，这个构造函数的功能一般都很简单。

在里面只有一个功能，以`TMetaClass`中保存的虚函数表作为参数，调用一个叫做`System::ClassCreate`的库函数。在这个`ClassCreate`里面会根据类的大小分配内存空间，以及调用类的初始化对象的函数，此函数的偏移是在虚函数表的 `-0xc`位置。

对于一个类而言，它在`constructor`中，首先要调用`System::ClassCreate`，它其中会使用函数表的 地址- 0xc位置的类特例的`InitInstance`函数，在其中调用`System::TObject::InitInstance`来做必备的初始化实例，此时通常会调用`System::__linkproc__ GetMem(int)`来申请到对象占用的内存。

如你所见，一个类的初始化函数完全没有我们想要的信息。它包含了什么东西，就像c++的初始化函数中不会有这些一样，delphi的也没有。那delphi的窗口、控件、函数是如何组织起来的，就完全要看`TMetaClass`了。

//---------------------------------------------

## 三、`TMetaClass` 结构

delphi里定义类的信息的`TMetaClass`结构，其结构格式是:

```
struct TMetaClass
{
+0x00:SefPtr            指向虚函数表的指针
+0x04:IntfTable         指向接口表指针 ?
+0x08:AutoTable         指向自动化信息表的指针 ?
+0x0c:InitTable         指向实例初始化表的指针 ?
+0x10:TypeInfo          指向本类的RTTI信息的指针
+0x14:FieldTable        指向field表的指针（published field）
+0x18:MethodTable       指向method表的指针（published method）
+0x1c:PrivateMethodTable指向pravite method表的指针
+0x20:ClassName         指向类名字符串的指针
+0x24:InstanceSize      对象实例的大小
+0x28:Parent            指向父类的TMetaClass的指针
+0x2c:SafeCallException ~~从这儿开始，和以下的都是虚函数的指针
+0x30:AfterConstruction
+0x34:BeforeDestruction
+0x38:Dispatch
+0x3c:DefaultHandler
+0x40:NewInstance
+0x44:FreeInstance
+0x48:Destroy
+0x4c:VirtualFunction0 //虚函数0
+0x50:VirtualFunction1 //虚函数1
...
+0x54:VirtualFunctionn //虚函数n
+0x58:ClassNameStrAry     //class类名，当然是pascal格式的字符串
};
```

其中delphi固定使用的结构大小有`0x4C`，也就是`VTBL`之前的`0x4c`字节。

各个元素的简略含义在上面修改自李战大师的结构中已经标明清楚了。

一个`TMetaClass`的实例

```
CODE:00413BF4 THook_ClassInit dd offset FuncTable 
CODE:00413BF8                 dd 2 dup(0)
CODE:00413C00                 dd 5 dup(0)
CODE:00413C14                 ClassName_Ptr           ; "THook"
CODE:00413C18                 dd 18h
CODE:00413C1C                 dd offset _cls_System_TObject_SubClassInit
CODE:00413C20                 System::TObject::SafeCallException(System::TObject *,void *) ; 
CODE:00413C24                 nullsub
CODE:00413C28                 nullsub
CODE:00413C2C                 System::TObject::Dispatch(void *) ;
CODE:00413C30                 nullsub
CODE:00413C34                 Comctrls::TTreeNodes::GetFirstNode(void) ; 
CODE:00413C38                 Destructor_THook
CODE:00413C3C                 System::TObject::~TObject(void) ;
CODE:00413C40 FuncTable:
CODE:00413C40 ClassName_Ptr:  db 5,'THook'
```

当分析delphi的类的时候，在这个结构中有几个重要的数据是定位目标函数要使用到的。它们的指针位于虚函数表的之前，也就是`TMetaClass`结构中的数据，摘入下

```
+0x10:TypeInfo 指向本类的RTTI信息的指针，
+0x14:FieldTable  指向field表的指针（Published Field）
+0x18:MethodTable 指向publick method表的指针
+0x1c:PrivateMethodTable 指向private表的指针

+0x20:ClassName         指向类名字符串的指针
+0x24:InstanceSize      指向对象实例的大小
+0x28:Parent            指向父类的指针
```

相对于类实例中`0x00000000`偏移处储存的类函数表的指针，其偏移分别是

```
-0x3c: TypeInfo
-0x38: FieldTable 
-0x34: MethodTable
-0x30: PrivateMethodTable

-0x2c: ClassName
-0x28: InstanceSize 
-0x24: Parent
```

~~~~~~~~~~~~~~~~~~~~~~~

-0x3c: TypeInfo

这儿的`TypeInfo`，储存的是这个`TMetaClass`自身的`TTypeInfo`，即这个`TMetaClass`代表的是一个什么类型的类。

它是一个`PTTypeInfo`类型的指针，在`TTypeInfo`里面储存的数据，是delphi实现RTTI功能重要数据。

在这个结构中的数据，也用于delphi的控件和组件的实现中，比如窗口之类的组件在delphi程序启动时如何进行初始化，就部分源自`TTypeInfo`中的`property`的设定。

`PTTypeInfo`指向的结构`TTypeInfo`可以认为是delphi中的最基础的类型，比方`integer`、`float`等，包括`class`的类型，都是由它来定义出来的类型。其结构如下

```
struct DECLSPEC_DRECORD TTypeInfo{
public:
    TTypeKind Kind;         // BYTE
    System::ShortString Name;   // Pascal Ascii String
       {TypeData: TTypeData}
};
```

`TTypeKind`是一个枚举类型，其值决定了后面跟着的那个`TTypeData`中的数据会被如何组织和使用。注意，枚举的计数是从0开始计数的

在Borland Delphi中，这个枚举的值以及特定的含义为，
