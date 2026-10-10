

# 极光多能闹钟 3.2 版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50640.htm`](https://www.pediy.com/kssd/pediy05/pediy50640.htm)

fly 兄：  
          你好！  
   你让看算法的极光多能闹钟 3.2 软件，真的是太老了，我在网上找了半天，也没有下载到这个版本，所以没有办法调试。最新版的注册稍有不同，输入注册码的地方，被加了锁(TEXT1.ENABLE=FALSE),只有从作者处得到注册方法，才能解锁。具体我还没有研究。  
   针对极光多能闹钟 3.2 版(VB 程序)的动态分析，我给你点提示，不足或是错误的地方还请你和其它高手指出：

1、VB 的冗余代码太多，所以在分析的时候，一定要将这段代码从手稿中删除(这些代码对程序也许有用，但对我们分析程序的注册算法没有用)，不要有顾虑，否则会分散注意力! 这里有个小方法，对一个特长的算法 CALL，你先要初步分析，要在纸上记下所有的跳转，特别要注意是的循环结构。第二步就是对每一个循环结构进行分析，只留下与算法有关的。第三步就是从你的分析稿中将所有无关代码删除。(具体可以看看我本论坛发表的的 Advanced MP3 WMA Recorder v3.73 破文，这个 VB 程序里的就有一个天大的算法 CALL，而真正有用的部分不多。)  
象在极光多能闹钟 3.2 程序中，如下代码就可以这样考虑：

:0049C1F6 C745C801000000          mov \[ebp\-38\], 00000001  
:0049C1FD C745C002000000          mov \[ebp\-40\], 00000002  
:0049C204 7206                    jb 0049C20C 

\* Reference To: MSVBVM60.\_\_vbaGenerateBoundsError, Ord:0000h  
                                |  
:0049C206 FF152C114000            Call dword ptr \[0040112C\] 

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0049C204(C)  
|  
:0049C20C 8B4E68                  mov ecx, dword ptr \[esi+68\]  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
直接改为：

:0049C1F6 C745C801000000          mov \[ebp\-38\], 00000001  
:0049C1FD C745C002000000          mov \[ebp\-40\], 00000002  
:0049C20C 8B4E68                  mov ecx, dword ptr \[esi+68\]  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

  
2、对一些 VB 函数有个大致了解：  
VB 程序在调用函数时，它都会先用 PUSH 命令将函数所要用到的参数压入椎栈，例：

:0049C2F7 52                      push edx   <===这里就是第一个字符  
:0049C2F8 50                      push eax   <===这里就是第二个字符

\* Reference To: MSVBVM60.\_\_vbaStrCat, Ord:0000h  
                                |    <===这里是将两个字符串合起来的函数  
:0049C2F9 FF1574104000            Call dword ptr \[00401074\]  
                                  
\-----------------------------------------  
:0049C22A 51                      push ecx  
:0049C22B 52                      push edx  
:0049C22C 50                      push eax 

\* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h  
                                |  
:0049C22D FF1518114000            Call dword ptr \[00401118\]  
  这是 VB 中的 MID 函数，用法 MID("字符串","开始的位置","取几个字符")                           

\------------------------------------------  
:0048BADA 8B45A8                  mov eax, dword ptr \[ebp\-58\]  
:0048BADD 50                      push eax  <===将假码压入椎栈  
:0048BADE 8B4DAC                  mov ecx, dword ptr \[ebp\-54\]  
:0048BAE1 51                      push ecx  <===将真码压入椎栈  
\* Reference To: MSVBVM60.\_\_vbaStrCmp, Ord:0000h  
                                |  <===这是字符串比较函数  
:0048BAE2 FF1534114000            Call dword ptr \[00401134\]  
                                  
\------------------------------------------

  
VB 函数的列表(newlaos 翻译整理)

1) 数据类型转换:   
a) \_\_vbaI2Str    将一个字符串转为 8 位（1 个字节）的数值形式(范围在 0 至 255 之间) 或 2 个字节的数值形式(范围在 -32,768 到 32,767 之间)。  
b)\_\_vbaI4Str   将一个字符串转为长整型(4 个字节)的数值形式(范围从-2,147,483,648 到 2,147,483,647)   
c)\_\_vbar4Str  将一个字符串转为单精度单精度浮点型(4 个字节)的数值形式  
d)\_\_vbar8Str   将一个字符串转为双精度单精度浮点型(8 个字节)的数值形式  
e) VarCyFromStr    (仅 VB6 库. 要调试，则在 WINICE.DAT 里必须有 OLEAUT32.DLL)字符串到变比型数据类型   
f) VarBstrFromI2  (仅 VB6 库. 要调试，则在 WINICE.DAT 里必须有 OLEAUT32.DLL)整型数据到字符串: 

2) 数据移动：   
a) \_\_vbaStrCopy      将一个字符串拷贝到内存，类似于 Windows API HMEMCPY   
b) \_\_vbaVarCopy     将一个变量值串拷贝到内存   
c) \_\_vbaVarMove    变量在内存中移动，或将一个变量值串拷贝到内存

3) 数学运算：  
a)  \_\_vbavaradd      两个变量值相加   
b) \_\_vbavarsub      第一个变量减去第二个变量  
c) \_\_vbavarmul     两个变量值相乘  
d) \_\_vbavaridiv     第一个变量除以第二个变量，得到一个整数商   
e) \_\_vbavarxor       两个变量值做异或运算

4) 程序设计杂项：   
a) \_\_vbavarfornext  这是 VB 程序里的循环结构， For... Next...  (Loop)   
b) \_\_vbafreestr    释放出字符串所占的内存，也就是把内存某个位置的字符串给抹掉  
c) \_\_vbafreeobj   释放出 VB 一个对象(一个窗口，一个对话框)所占的内存，也就是把内存某个位置的一个窗口，一个对话框抹掉  
d) \_\_vbastrvarval  从字符串特点位置上获取其值   
e) multibytetowidechar  将数据转换为宽字符格式，VB 在处理数据之都要这样做，在 TRW2000 显示为 7.8.7.8.7.8.7.8  
f) rtcMsgBox   调用一个消息框，类似于 WINDOWS 里的 messagebox/a/exa,此之前一定有个 PUSH 命令将要在消息框中显示的数据压入椎栈   
g) \_\_vbavarcat    将两个变量值相连，如果是两个字符串，就连在一起   
h) \_\_vbafreevar  释放出变量所占的内存，也就是把内存某个位置的变量给抹掉  
i) \_\_vbaobjset   
j)  \_\_vbaLenBstr   获得一个字符串的长度，注：VB 中一个汉字的长度也为 1  
k) rtcInputBox    显示一个 VB 标准的输入窗口，类似 window's API getwindowtext/a, GetDlgItemtext/a   
l) \_\_vbaNew      调用显示一个对话框，类似 Windows' API Dialogbox   
m) \_\_vbaNew2    调用显示一个对话框，类似 Windows' API Dialogboxparam/a   
n) rtcTrimBstr   将字串左右两边的空格去掉 

5) 比较函数  
a)  \_\_vbastrcomp   比较两个字符串，类似于 Window's API lstrcmp   
b)  \_\_vbastrcmp   比较两个字符串，类似于 Window's API lstrcmp   
c) \_\_vbavartsteq  比较两个变量值是否相等  
d)\_\_vbaFpCmpCy                  - Compares Floating point to currency. sp;            Compares Floating point to currency 

6) 在动态跟踪，分析算法时，尤其要注意的函数：  
rtcMidCharVar  从字符串中取相应字符，VB 中的 MID 函数，用法 MID("字符串","开始的位置","取几个字符")  
rtcLeftCharVar 从字符串左边取相应字符，VB 中的用法：left("字符串","从左边开始取几个字符")  
rtcRightCharVar 从字符串右边取相应字符，VB 中的用法：Right("字符串","从右边开始取几个字符")  
\_\_vbaStrCat  用字符串的操作，就是将两个字符串合起来，在 VB 中只有一个&或+  
\_\_vbaStrCmp  字符串比较，在 VB 中只有一个=或<>  
ASC()函数    取一个字符的 ASC 值，在反汇编时，还是有的 movsx 操作数

7) 在函数中的缩写：  
bool 布尔型数据(TRUE 或  FALSE)   
str  字符串型数据 STRING  
i2   字节型数据或双字节整型数据  BYTE or Integer  
ui2  无符号双字节整型数据    
i4   长整型数据(4 字节)  Long  
r4   单精度浮点型数据(4 字节)  Single   
r8   双精度浮点型数据(8 字节)  Double  
cy 　（8 个字节）整型的数值形式 Currency  
var  变量     Variant  
fp 　浮点数据类型  Float Point  
cmp  比较   compare  
comp 比较   compare

花了半天牛劲，还不知对 fly 有没有用。我毕竟接触 VB 时间也不长，不足或是错误的地方还请你将就的看了：-)

          
                                          newlaos\[CCG\]\[DFCG\]  2003-04-23

 4dd466d0e1e0cc123afbe504da2462ca