

# basic\_string<char,std::char\_traits<char>,std::allocator<char> 是什么意思

> 原文：[`www.pediy.com/kssd/pediy10/91120.html`](https://www.pediy.com/kssd/pediy10/91120.html)

msvcp80.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::data  
  
这个是不是一个事件?  
我应该怎样分析出的它的含义...?  
大家帮帮忙.. 

006A1131    68 70C16C00     push asktao.006CC170                 ; ASCII "xx\_kk"  
006A1136    8D4C24 24       lea ecx,dword ptr ss:\[esp+24\]  
006A113A    8BD8            mov ebx,eax  
006A113C    FF15 58136C00   call dword ptr ds:\[<&MSVCP80.std::ba>; msvcp80.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::basic\_string<char,std::char\_traits<char>,std::allocator<char> >  
  
类似这样的代码 应该怎么分析? 

std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::basic\_string<char,std::char\_traits<char>,std::allocator<char> >  
  
首先是最外层:  
std::basic\_string<X,Y,Z >::basic\_string<X,Y,Z >  
其中:  
X = char  
Y = std::char\_traits<char>  
Z = std::allocator<char>  
  
std 是 STL 的 namespace  
显然这个是在调用 basic\_string<X,Y,Z >的构造函数  
  
这里  
Y,Z 是 X 确定定后 basic\_string 的默认模版参数,  
所以实际就是  
basic\_string<char>  
  
这个正是 string  
  
所以这里是调用  
string("xx\_kk") 

iicup 已经解释得很清楚了。  
  
我再多几句嘴，如果你不习惯 C++的 STL 编程方式，别人用某个例子就事论事地讲，你也很难明白。“逆向”的的前提，至少是知道“正向”是如何工作的吧。  
  
简单（但不严格）地说，std::basic\_string<char>就是 STL 提供的的字符串，即 std::string  
  
而 std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >就是 std::basic\_string<char>的缺省模板参数展开形式。  
  
std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::data 写成简单的形式就是 std::string::data  
  
msvcp80.std::basic\_string<char,std::char\_traits<char>,std::allocator<char> >::basic\_string<char,std::char\_traits<char>,std::allocator<char> >这个函数其实就是字符串的构造函数，可以理解为：std::string::string(const char \*)  
  
  
如果要深究 string 类中的 data 成员为何物，就需要研究 STL 源码了。可以去看侯俊杰《STL 源码分析》。