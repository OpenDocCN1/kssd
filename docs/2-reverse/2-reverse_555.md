

# 从汇编角度探讨 C++

> 原文：[`www.pediy.com/kssd/pediy05/pediy50384.htm`](https://www.pediy.com/kssd/pediy05/pediy50384.htm)

要求：   
1、给出程式的输出结果（这个太简单了，试试就知道了~~~）   
2、用专业术语解释为什么会有这样的输出结果（如果方便的话，越详细越好……）   
  
  
#include   
  
class boy//男人类   
{   
public://他敞开胸怀   
void LoveYou(float result)   
{   
cout << "Preety, I love you, marry me!" << endl;   
}   
void KissYou(float result)   
{   
cout << "Pretty, you are too sexy, I want to kiss you..." << endl;   
}   
};   
  
class girl : public boy//上帝说女人是……   
{   
public://她也敞开胸怀……我什么都没看见：）   
void LoveYou(int result)   
{   
cout << "I love you too...But I am too shy to say..." << endl;   
}   
void KissYou(float result)   
{   
cout << "You do what you want..." << endl;   
}   
};   
  
void main(void)   
{   
girl pretty;//一个女孩叫 pretty   
boy handsome;//一个男孩叫 handsome   
  
girl \*the\_pretty\_home = &pretty;//女孩说：我不要离开家……不嫁给你（真的？）！   
boy \*the\_handsome\_home = &pretty;//男孩说：来我家吧，我爱你！   
  
////////////////////////////////////////   
//----------？？？？？他们在干什么呀？请写出结果-------------   
  
the\_handsome\_home->LoveYou(52.1f);   
the\_pretty\_home ->LoveYou(52.1f);   
  
the\_handsome\_home->KissYou(52.1f);   
the\_pretty\_home ->KissYou(52.1f);   
  
////////////////////////////////////////   
} 

\=====hjiaming 说:   
so easy!!!很有趣哦！   
你的程序好像有错误哦！   
girl \*the\_pretty\_home = &pretty；   
boy \*the\_handsome\_home = &pretty;－－>???????   
下面这个 pretty 应该是 handsome 吧！   
结果：   
Preety, I love you, marry me!   
I love you too...But I am too shy to say...   
Pretty, you are too sexy, I want to kiss you...   
You do what you want..   
  
程序本身构建两个类，分别是 Boy 和 Girl，每个类中各有两个成员函数，LoveYou(float)和 KissYou(float)。主函数先构造两个对象，pretty 和 handsome，声明\*the\_pretty\_home 和\*the\_handsome\_home 两个指针变量分别指向两个对象。接下来就是调用相应的成员函数实现程序的输出了！   
  
不过，stuman，我请教一个问题，我不知道成员函数中的参数具体有什么作用啊，能否告知！！   
  
\=====stuman 说：   
几点申明：   
1、程序没有错误的，否则无法编译成功，也就不能运行和打印出结果了。   
2、程序是用标准的 C++写的。   
3、关于 hjianming 朋友提出的问题（的确非常仔细！普通人非常容易忽略这个问题）：   
  
girl \*the\_pretty\_home = &pretty；   
boy \*the\_handsome\_home = &pretty;－－>???????   
下面这个 pretty 应该是 handsome 吧！   
  
这个 pretty 是 pretty 没错（不是我写错了），不是 handsome！   
另外参数的秘密嘛~~~~~~不能说，你再想想：）   
  
\=====leoq 说：   
哈哈，函数重载，调用基类函数，有点意思   
Preety, I love you, marry me!   
Preety, I love you, marry me!   
You do what you want...   
You do what you want...   
这是结果？   
  
\=====hjiaming 说：   
我发现不管是 pretty 还是 handsome，运行结果是一样的，但我搞不明白为什么会这样啊，哎，C++功底不够啊！stuman 快告诉我啊   
  
\=====amakusa 说：   
Preety, I love you, marry me!"   
Preety, I love you, marry me!"   
Pretty, you are too sexy, I want to kiss you..."   
You do what you want..."   
  
\=====amakusa 说：   
555 答案是错的啊 唉 还需努力啊   
最后一个输出 我搞不清是 call 继承来的成员函数 还是自身的   
c++ 不好 见笑了 从学习到工作 一直都是 object pascal   
  
\=====stuman 说：   
我把 Object Pascal 代码写出来。（略）   
  
\=====amakusa 说：   
Delphi 中   
成员函数被重载了   
the\_pretty\_home.LoveYou(50); //这里也必须是整型参数了   
c++中   
the\_pretty\_home ->LoveYou(52.1f);   
c++编译器不会报告参数类型错误吗   
  
\=====stuman 说：   
the\_pretty\_home ->LoveYou(52.1f);   
c++编译器是不会报告参数类型错误的，它用隐含的强制类型转换，如果设置上严格一点，编译器也大不了给个警告出来，编译完全能通过的。所以题目也就利用了这点来迷惑人，嘻嘻~~   
不过 Delphi 就不同了，它对参数类型的要求十分严格，我只好用   
the\_pretty\_home.LoveYou(52);   
使用整数参数了（泄露天机！），55555   
  
现在我就公布答案了：）   
这只是一个概念问题吧。girl 的成员函数对 boy 的成员函数非常类似于重载(reload)，但在概念上叫做“隐藏”(hide)。具体说，girl 的 LoveYou 隐藏了 boy 的 LoveYou，girl 的 KissYou 隐藏了 boy 的 KissYou，所以，对 girl 的派生类来说，boy 的两个成员函数都是存在的。至于说参数为什么不一样，只是为了演示参数类型对“隐藏”的影响和与“重载”的重要区别：   
girl 和 boy 两个类中两个 LoveYou()函数可能用“近似重载"来勉强解释，但是两个 KissYou(float result)函数的形式一模一样，能用“重载”来解释吗？重载的函数必须参数不同的，对吧？重载也不可能发生在两个类中。   
最让人迷惑的就是为什么 the\_pretty\_home 和 the\_handsome\_home 被赋予同样的地址（&pretty），按道理说 the\_pretty\_home.LoveYou 和 the\_handsome\_home.LoveYou（还有两个 KissYou）的结果应该完全一样，但竟然结果不同！那么更深层次的理解需要用类的内存构象来解释，涉及到编译器的处理，我想我们没必要探讨那么深入了。   
这里，我们只要明白在隐藏的情况下，boy.LoveYou 是能通过派生类 girl 调出来的。这在实践上可能没什么用处，不过在程序调试上万一出现这种错误，发现结果不对，百思不解，那么要当心这类问题了！   
  
用调侃的解释来说：男孩娶了女孩以后，如果女孩管教不严，那么男孩可能会隐藏一些私房钱（好危险哦，小心他花心哦），这些私房钱是他自己的，但是没有上交给女孩，所以适当的时候他还是能花掉这些钱的。要彻底拿掉这些私房钱，女孩必须用强力手段“overload”（覆盖）来镇压！   
  
正确的输出结果是：   
  
Preety, I love you, marry me!   
I love you too...But I am too shy to say...   
Pretty, you are too sexy, I want to kiss you...   
You do what you want..   
  
我祝福天下有情人终成眷属！   
  
\=====liuxing 说：   
对于 C＋＋来说，最强的功能是指针，最容易犯错的也是指针。本例中，关键是二个类的结构是完全相同的。这时候 C＋＋的编译器会根据最适合的方法来解释:   
  
girl \*the\_pretty\_home = &pretty;//女孩说：我不要离开家……不嫁给你（真的？）！   
boy \*the\_handsome\_home = &pretty;//男孩说：来我家吧，我爱你！   
  
换句话说，这时，编译器并不知道 the\_handsome\_home，the\_pretty\_home 到底是 boy 还是 girl。必须根据具体的方法调用时才能确定。   
  
。上例的结果实际上，严格来说并不准确，只能说对 VC 是正确的，对不同的编译器会有不同的处理方法，其结果也不同。   
  
\=====hjiaming 说：   
我试过了，在 C++Builder 的编译器中，结果是一样的。   
  
\======stuman 说：   
我们需要更深层次来讨论这个问题，我反汇编了相关代码，这里是 main()段：   
  
:00401060 55                      push ebp   
:00401061 8BEC                    mov ebp, esp   
:00401063 83EC10                  sub esp, 00000010   
  
:00401066 8D45FC                  lea eax, dword ptr \[ebp-04\]                    ;\[ebp-04\]为&pretty   
:00401069 8945F8                  mov dword ptr \[ebp-08\], eax                  ;\[ebp-08\]为\[the\_pretty\_home\]   
;girl  \*the\_pretty\_home  = &pretty;   
  
:0040106C 8D4DFC                  lea ecx, dword ptr \[ebp-04\]   
:0040106F 894DF0                  mov dword ptr \[ebp-10\], ecx                  ;\[ebp-10\]为\[the\_handsome\_home\]   
;boy  \*the\_handsome\_home = &pretty;   
  
:00401072 6866665042              push 42506666   
:00401077 8B4DF0                  mov ecx, dword ptr \[ebp-10\]   
:0040107A E890FFFFFF              call 0040100F   
;the\_handsome\_home->LoveYou(52.1f);   
  
:0040107F 6A34                    push 00000034   
:00401081 8B4DF8                  mov ecx, dword ptr \[ebp-08\]   
:00401084 E890FFFFFF              call 00401019   
;the\_pretty\_home  ->LoveYou(52.1f);   
  
:00401089 6866665042              push 42506666   
:0040108E 8B4DF0                  mov ecx, dword ptr \[ebp-10\]   
:00401091 E86FFFFFFF              call 00401005   
;the\_handsome\_home->KissYou(52.1f);   
  
:00401096 6866665042              push 42506666   
:0040109B 8B4DF8                  mov ecx, dword ptr \[ebp-08\]   
:0040109E E87BFFFFFF              call 0040101E   
;the\_pretty\_home  ->KissYou(52.1f);   
  
:004010A3 8BE5                    mov esp, ebp   
:004010A5 5D                      pop ebp   
:004010A6 C3                      ret   
  
可以看到，the\_pretty\_home 和 the\_handsome\_home 指针的数值的确是相同的，都指向了&pretty   
但是为什么   
the\_handsome\_home->LoveYou(52.1f);和   
the\_pretty\_home->LoveYou(52.1f);   
却得到了不同的结果。汇编代码提示出，在调用成员函数的时候已经被编译成不同的地址：一个是 call 0040100F，一个是 call 00401019。   
经过函数跳转表后，call 0040100F 是下面的代码：   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040100F(U)   
|   
:004010C0 55                      push ebp   
:004010C1 8BEC                    mov ebp, esp   
:004010C3 51                      push ecx   
:004010C4 894DFC                  mov dword ptr \[ebp-04\], ecx   
:004010C7 6814104000              push 00401014   
  
\* Possible StringData Ref from Data Obj ->"Preety, I love you, marry me!"   
                                  |   
:004010CC 6850BD4100              push 0041BD50   
:004010D1 B9E0F44100              mov ecx, 0041F4E0   
:004010D6 E875030000              call 00401450   
:004010DB 8BC8                    mov ecx, eax   
:004010DD E81EFFFFFF              call 00401000   
:004010E2 8BE5                    mov esp, ebp   
:004010E4 5D                      pop ebp   
:004010E5 C20400                  ret 0004   
  
非常明显，这就是 boy 类的成员函数 LoveYou();而 call 00401019 同样可以证明是 girl 类的成员函数 LoveYou();   
  
最后我的结论是：上面的隐藏规则在编译的时候就确定和实现了，也即是“静态绑定”。C++类的指针不仅和它的值有关系，还和它的类别有关系。也就是说，编译器一定要知道 the\_handsome\_home，the\_pretty\_home 到底是 boy 还是 girl，才能确定具体的方法调用。这和 C 具有重大的区别。  4dd466d0e1e0cc123afbe504da2462ca