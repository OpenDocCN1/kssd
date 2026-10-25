
# [分享]一个菜鸟初步分析C++类的虚函数的实现机制，请大牛们指点。

## 一、前言

作为菜鸟的我，发现学习逆向的道路真是太难了，用小沈阳的话说：“老难了！”。需要掌握的东西太多，而我只是会一点点C语言。咳，不说这些了。

之前逆向一些C++写的程序的时候，发现了好多类似 `CALL [EAX]; CALL [EAX + 20]` 之类的语句，开始以为可能是函数指针的调用，但是对我来说，那些直接地址调用的函数还搞不明白呢，这样的语句就更不清楚了。后来搜索我们的论坛，发现不只是函数指针调用这样的简单，竟然可能是另外一个我根本没有听过的名字“虚函数”，然后我发现论坛上有很多大牛写的关于虚函数的实现原理，可是作为一个对C++一窍不通的我，简直就是天书。既然这样还是先学学C++吧，于是乎再次搜索论坛，发现很多网友推荐学`<<C++ primer>>`这本书，下载完这本书，竟然有1000多页，马上就想到放弃，算了，这要看到啥时候啊，况且我还要工作。但是我随便翻目录的时候，发现有很多是C语言的东西，而且有四五百页呢，既然这样那就开始学习吧。囫囵吞枣，总算看完了，啥多态、虚函数，根本没弄清楚，既然这样，还是先实践吧，毕竟这1000多页书，一段程序都没编，主要是因为我就没用过VC，为了实际动手，还是再学习学习VC吧。于是再次搜索论坛，发现很多人推荐"VS 2005 C++ 红皮书"，再次下载，晕，又是1000多页，想要我命啊，再次翻目录，发现竟然又有500多页介绍C语言和C++的，难道写书为了多挣money，都这样干？（开个玩笑啊，没有侮辱权威的意思），于是又重现开始学习，从控制台程序学习开始，这次发现有些C++基本的概念逐渐弄懂了些。

当学到虚函数的时候，我本想再次查找之前大牛写的文章来看，但是为了自己能够真正的掌握，还是先尝试自己分析吧，当分析完后再去找答案。对我的理解更有好处。

以下的内容只是个人的理解，而且肯定有很多问题，我只是把我学习的过程记录下来。

## 二、类的数据成员

当定义类对象的时候，每个类对象的数据成员都有一份拷贝，比如：

```
class CPix
{
    public:
        int x;
        int y;
};

CPix personPix;
CPix roomPix;

//显示personPix的大小
cout<< "personPix's size is << sizeof(personPix) <<endl;

//显示personPix.x的地址
cout<< "personPix.x's address  is << &personPix.x <<endl;

//显示roomPix的大小
cout<< "roomPix's size is << sizeof(roomPix) <<endl;

//roomPix.x的地址
cout<< "roomPix.x's address  is << &roomPix.x <<endl;
```

输出为：

```
personPix size is 8
personPix.x's address  is 12FF30
roomPix's size is 8
roomPix.x's address  is 12FF38
```

很明显 `personPix.x` 与 `roomPix.x` 的地址是不同的。而且通过这个例子我也知道，定义一个类对象，编译器会根据类的数据成员的大小，为其分配一段内存。比如 `personPix` 和 `roomPix` 的大小都为8。

## 三、类的成员函数

在开始学习C++的时候对成员函数的定义有一些疑问，是否是每定义一个类对象就会定义一个成员函数，如果这样，程序岂不是很大，如果不存在多分成员函数的拷贝，又是通过那种方式来找到类的成员函数的呢，后来知道自己的无知，由于类的成员函数的操作对象就是类的数据成员，而类定义不同的类对象的时候，每个类都有自己的数据成员，因此当调用类的成员函数的时候，只要将类对象的数据成员地址传递给类的成员函数就可以了。为此C++引入了`this`指针，当C++编译的时候会为每个成员函数多增加一个参数，就是`this`指针，这个指针的实参就是每个类对象的数据成员的首地址。

猜测：反汇编的时候很多函数都有`ECX`这个参数，可能这个`ECX`就是`this`指针。

## 三、虚函数

先将例子写出来吧，程序很丑。

```
/****************************************************
文件名   ：VirtualFunction.cpp
文件说明 ：用于学习虚函数的实现机制
日期     ：2009-05-06
*****************************************************/

#include <iostream>

//========================================================
//                             名字空间声明
//========================================================
using std::cout;
using std::endl;

//========================================================
//类  名 ：CBox
//类说明 ：是基类，用于学习虚函数的实现机制
//========================================================
class CBox
{
    public:
        //默认构造函数
        CBox(int lv = 1, int wv = 1, int hv = 1):
            m_Length(lv), m_Width(wv), m_Height(hv)
        {
            cout << "CBox constructor called"<<endl;
        }

        //求“box”的体积，定义为虚函数
        virtual int Volume()
        {
            return m_Length * m_Width * m_Height;
        }

        //显示“box”的体积
        void ShowVolume()
        {
            cout <<"CBox volume is "<<Volume()<<endl;
        }

    protected:
        int m_Length;
        int m_Width;
        int m_Height;
};

//========================================================
//类  名 ：CCandyBox
//类说明 ：基类CBox的派生类，用于学习虚函数的实现机制
//========================================================
class CCandyBox : public CBox
{
    public:
        //默认构造函数
        CCandyBox(int lv, int wv, int hv):CBox(lv, wv, hv){}

        //求“CCandyBox”的体积，定义为虚函数
        virtual int Volume()
        {
            return m_Length * m_Width * m_Height * 2;
        }
};

//========================================================
//函数名   ：main
//函数说明 ：主函数用于学习虚函数的实现机制
//函数参数 ：无
//返回值   ：0
//========================================================
int main()
{
    CBox myBox(1, 2, 3);                                //基类对象定义
    CCandyBox myMinBox(1,2,3);                //派生类对象定义

    //显示基类对象的体积
    myBox.ShowVolume();
    //显示派生类对象的体积
    myMinBox.ShowVolume();

    //显示“myMinBox”的大小
    cout << "myMinBox's size is " << sizeof(myMinBox) << endl;
    return 0;
}
```

### 3.1 代码说明

上面的例子定义了一个基类：`CBox`，它的派生类：`CCandyBox`。其中：

1. `virtual int Volume()` 被定义为虚函数，用于显示Box的体积。我需要通过反汇编查看这个函数的调用过程。
2. `void ShowVolume()` 这个函数是显示体积的大小。类 `CCandyBox` 没有定义，从它父亲那里继承了这个函数。
3. `m_Length`、`m_Width`、`m_Height` 在基类中访问属性被声明为`protected`，因此 `CCandyBox` 同样也从他父亲那里继承过来。
4. `CBox(int lv = 1, int wv = 1, int hv = 1)` 为基类的默认构造函数，在声明基类的时候，编译器会自动的调用这个函数，来初始化基类的对象。通过反汇编可以清楚的看到这部分的代码。
5. `CCandyBox(int lv, int wv, int hv)` 为派生类的默认构造函数，它只是简单的调用基类的默认构造函数。

### 3.2 输出结果

执行上面的代码输入如下：

```
Volume is 6
Volume is 12
```

正如我所希望的那样，`myBox.ShowVolume()` 调用的是 `CBox` 的虚函数 `Volume()`。`myMinBox.ShowVolume()` 调用的是 `CCandyBox` 的虚函数 `Volume()`。

### 3.4 反汇编分析

好了源程序的分析就到这里了，现在需要反汇编了。通过VS 2005生成`irtualFunction.exe`执行文件。使用OD打开这个程序。同时定位到`main`函数入口，代码如下：

```
;========================================================
00411CD0 push    ebp                    ;main 入口函数
00411CD1 mov     ebp, esp
00411CD3 sub      esp, 0F0
00411CD9 push    ebx
00411CDA push    esi
00411CDB push    edi
00411CDC lea       edi, dword ptr [ebp-F0]
00411CE2 mov     ecx, 3C
00411CE7 mov     eax, CCCCCCCC
00411CEC rep      stos dword ptr es:[edi]
00411CEE push    3
00411CF0 push    2
00411CF2 push    1                                                                                             ;将myBox的初始化参数1，2，3压栈
00411CF4 lea     ecx, dword ptr [ebp-14]     ;this指针，指向myBox的首地址
00411CF7 call    00411023                                ;这里是调用CBox的默认构造函数
00411CFC push    3
00411CFE push    2
00411D00 push    1                                            ;将myMinBox的初始化参数1，2，3压栈
00411D02 lea     ecx, dword ptr [ebp-2C]     ;this指针，指向myMinBox的首地址
00411D05 call    004111AE                                ;这里是调用CBox的默认构造函数
00411D0A lea     ecx, dword ptr [ebp-14]    ;this指针，指向myBox的首地址
00411D0D call    004110A0                                ;调用成员函数ShowVolume
00411D12 lea     ecx, dword ptr [ebp-2C]    ;this指针，指向myMinBox的首地址
00411D15 call    004110A0                                ;调用成员函数ShowVolume
00411D1A xor     eax, eax
00411D1C push    edx
00411D1D mov     ecx, ebp
00411D1F push    eax
00411D20 lea     edx, dword ptr [411D44]
00411D26 call    004110CD
00411D2B pop     eax
00411D2C pop     edx
00411D2D pop     edi
00411D2E pop     esi
00411D2F pop     ebx
00411D30 add     esp, 0F0
00411D36 cmp     ebp, esp
00411D38 call    004111D1
00411D3D mov     esp, ebp
00411D3F pop     ebp
00411D40 retn；
;========================================================
```

从这里我证实了之前的猜测：反汇编的时候很多函数都有`ECX`这个参数，可能这个`ECX`就是`this`指针。同时我也看到了在声明类的实例的时候，编译器是通过怎样的汇编指令实现初始化的。

但是，分析`main`函数还没有看出虚函数的调用过程，因为虚函数是在成员函数`ShowVolume`中掉用的，因此需要进入`ShowVolume`函数中查看。而`ShowVolume`函数的地址为：`004110A0`，跟进去后，代码如下：

```
;========================================================
00411700 push    ebp                         ;ShowVolume函数入口
00411701 mov     ebp, esp
00411703 sub     esp, 0CC
00411709 push    ebx
0041170A push    esi
0041170B push    edi
0041170C push    ecx
0041170D lea     edi, dword ptr [ebp-CC]
00411713 mov     ecx, 33
00411718 mov     eax, CCCCCCCC
0041171D rep     stos dword ptr es:[edi]
0041171F pop     ecx
00411720 mov     dword ptr [ebp-8], ecx         ;将this指针保存到dword ptr [ebp-8]中
00411723 mov     esi, esp
00411725 mov     eax, dword ptr [<&MSVCP80D.std::
0041172A push    eax
0041172B mov     ecx, dword ptr [ebp-8]
0041172E mov     edx, dword ptr [ecx]                 ;将this指针的所指向的内容送给edx
00411730 mov     edi, esp
00411732 mov     ecx, dword ptr [ebp-8]
00411735 mov     eax, dword ptr [edx]                 ; 又将edx给eax
00411737 call    eax
```
