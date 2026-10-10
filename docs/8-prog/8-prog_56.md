

# 跑在 main 之前

> 原文：[`www.pediy.com/kssd/pediy12/134576.html`](https://www.pediy.com/kssd/pediy12/134576.html)

通常这是一个面试题，如何让一段代码跑在 main()函数前面。一个熟练的 C++开发者能够轻易给出答案，即 [**静态初始化**](http://user.qzone.qq.com/31731705/blog/1302767807)。  
  
在 C 里面又如何呢，有没有办法做到这点？下面讲一个 VC 里的方法。  
  
int initBeforeMain()  
{  
 printf( "initBeforeMain (%p)\\r\\n", initBeforeMain );  
 return 0;  
}  
  
int init2BeforeMain()  
{  
 printf( "init2BeforeMain (%p)\\r\\n", init2BeforeMain );  
 return 0;  
}  
  
int initBreak()  
{  
 DebugBreak();  
 return 0;  
}  
  
typedef int (\*pInit)();  
  
// for C  
  
// put following table between \_\_xi\_a and \_\_xi\_z  
  
#pragma data\_seg( ".CRT$XIU" )  
pInit start\[\] = {initBeforeMain, init2BeforeMain, initBreak, };  
#pragma data\_seg()  
  
// for C++  
  
// put following table between \_\_xc\_a and \_\_xc\_z  
  
#pragma data\_seg( ".CRT$XCU" )  
pInit start2\[\] = {initBeforeMain, init2BeforeMain, initBreak, };  
#pragma data\_seg()  
  
// for C  
  
// put following pointer between \_\_xi\_a and \_\_xi\_z but after start  
  
#pragma data\_seg( ".CRT$XIU" )  
pInit start3 = initBreak;  
#pragma data\_seg()  
  
int main( int argc, char\* argv\[\] ) {}  
  
  
在调试器里执行一下，看看效果如何，有没有给你 surprise。  
  
解释上面代码之前先说一些背景知识，众所周知，开发者所写的入口函数一般都不是真正的入口，编译器会提供自己的入口函数，再调用开发者的。逻辑很简单，如下：  
  
CRTEntry() { {\_\_CRTInit}; UserEntry(); {\_\_CRTUninit}; }  
  
\_\_CRTInit 中做一些初始化工作，包括 C 库，C 的初始化函数，C++库，C++的初始化函数等。C 和 C++分别有一张表来保存初始化函数指针，每个表又使用 2 个指针来明确范围，\_\_CRTInit 会依次调用这 2 个表中的函数。  
  
C 初始化函数表：\[\_\_xi\_a, \_\_xi\_z)  
  
C++初始化函数表: \[\_\_xc\_a, \_\_xc\_z)  
  
现在对照代码注释，就会明白上述那段代码的作用。通过特殊的段名称“.CRT$XIU”，“.CRT$XCU”，链接器会把 start 表放在“C 初始化函数表”中，类似这样，\[\_\_xi\_a, ..., start, ..., \_\_xi\_z)，同理，start2 表会被链接器放在“C++初始化函数表”中，象这样，\[\_\_xc\_a, ..., start2, ..., \_\_xc\_z)，而最后一个 start3 变量所对应的指针还是会被放在“C 初始化函数表”中，在上面例子中，会放在 start 之后，\[\_\_xi\_a, ..., start, ..., start3, ..., \_\_xi\_z)，这里的 start3 的位置即函数指针 initBreak 的位置。  
  
最初发布在这：[**http://user.qzone.qq.com/31731705/blog/1302771059**](http://user.qzone.qq.com/31731705/blog/1302771059)，包含一些贴图的结果，这贴图不方便，因此只写文字了，想看图的可以访问链接。