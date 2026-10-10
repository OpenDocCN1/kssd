

# 反字符参考全接触

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-110.htm`](https://www.pediy.com/kssd/pediy09/pediy09-110.htm)

> 这个本来是打算投稿看雪的杂志，所以简单的东西也说得很详细。前两部分可以忽略，文章真正有价值的是第三部分，从指针欺骗到字符欺骗，可以骗过 Ultra String Reference 和 IDA 的字符参考窗口。避免加密后的关键字符仍然以乱码的方式出现在字符参考窗口中，导致仍然可以通过加密字符找到关键代码的情况。毕竟最好的加密是让对方根本不知道其存在。  
>     本来还有其他的想法，例如 Ultra String Reference 可能只搜索 EXE 所在内存范围，而不扫描 DLL 中的参考信息，又或者是象壳那样将解密的代码复制到申请的内存空间中运行。但是前面的方法过于简单，而且有效，以致我觉得完全没有必要用复杂的方法实现，就取消了对复杂方法的探索。  
>     另外，一个明显的注册信息，和一堆明显的无意义字符，你会去看哪一个参考呢？假如把加密的字符隐藏了，现在只有唯一的注册信息，就可以引诱 CRACKER 去看一堆可能根本不可能运行的代码。一两句话就能说清楚的东西，也没必要写示例代码。  
>     文章就砍剩这么多了，大家凑合看。发上来看看能不能混个精华 

> 感谢笨笨雄的文章，不知是用什么编译编的，我用 Visual c++   
> 修改头文件后编译了一下，发现的确有效。  
> 修改版：  
> #include "windows.h"  
>   
> class defMsgBox  
> {  
> public:  
>     defMsgBox()  
>     {  
>       m\_text = (char \*)"你能找到我吗？";  
>       m\_caption = (char \*) "隐藏字符参考";  
>     }  
>     void myShow()  
>     {  
>       MessageBox(0,m\_text,m\_caption,0);  
>     }  
>     char \*m\_text;  
>     char \*m\_caption;  
> };  
>   
>   char \*myText = "测试";  
>   
> int APIENTRY WinMain(HINSTANCE hInstance,  
>                      HINSTANCE hPrevInstance,  
>                      LPSTR     lpCmdLine,  
>                      int       nCmdShow)  
> {  
>   defMsgBox myMsgBox;  
>   char \*myCaption = "\\x1f""Hello Ultra String Reference";  
>   MessageBox(0,myText,myCaption,0);  
>   myMsgBox.myShow();  
> //字符指针 myCaption 由于第一个字节加入了无效字符而被 Ultra String Reference 的确被忽略  
>   return 0;  
> } 

 4dd466d0e1e0cc123afbe504da2462ca