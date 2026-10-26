
# 掀开高级游戏黑客的面纱，教你打造游戏修改器 (10千字)

标 题:掀开高级游戏黑客的面纱，教你打造游戏修改器 (10千字)  
发信人:CrackYY  
时 间:2003-2-10 19:48:36  
详细信息:掀开高级游戏黑客的面纱，教你打造游戏修改器

工具:SoftICE、金山游侠2002、VC++7.0、PE查看器、SPY++  
测试平台：Window2000 Professional SP2

大家好！我先给大家拜个晚年，时间过真快工，一年又过去了，我也和大家分开一年了，真是感慨万分呀，不知道大还记得不记得YY了，不过我是不会忘记大家的。

这一年真是太忙了，根本就没有时间再像从前那样在深夜里一边听着音乐一边写文章了，今天就着假期，再给大家充充电：D

今天YY给大家带来些什么呢？呵呵，看题目就知道了，看起来很酷吧，“高级游戏黑客”,什么？你说你就游戏黑客？看好了，是“高级”的！什么是高级的？等你看完这篇文章就知道了：D

## 工具准备

首先我介绍一下将会用到的工具：

1. SoftICE（不用多说了吧，我想你应该会用）
2. 金山游侠2002（这个你也应该会用）
3. VC++7.0（不要求你一定会用，但至少应该会一种编程工具）
4. PE查看器（你可以随意找一个，没有也没关系，我会教你用SoftICE查看）
5. SPY++（VC里的一个查看程序信息的工具，你可以和别的，比如Delphi和C++Builder的WinSight32）

## 预备知识

然后就是你应该会的知识:

1. 汇编基础
2. 一些编程基础，至少应该看懂我介绍的几个API函数
3. PE文件结构的基础，不会也没关系，我会解释给你

以上几点你都具备了的话我们就可以开始了。

## 修改器原理

我来介绍一下我要教给你的东西。想必大家都玩过PC游吧，那么也一定用过一些专用的游戏修改器吧，比如暗黑，红警，大富翁这些经典的游戏都有它们专用的修改器，注意，我说的不是FPE之类的通用修改工具。

你试没试过用金山游侠修改红警二的金钱？如果有的话你应该知道每玩一次就要改一次，因为这个游戏是动态分配内存的，每次重新开始都会改变。所以你会选择到网上去下载一个专用的修改器，那么你有没有想过自己做一上呢？想过？那你为什么不做？什么不会？那就好办了，看了这篇教程你就会了：D费话少说，我来讲一下原理。

有一些经常修改游的朋友一定会知道，不论游戏中“物品”的内存地址是否是动态的，物品与物品之间相隔的距离都是不变的，我拿“楚留香新传”为例，我先用金山游侠查找内力值的内存地址，找到的结果是:`79F695C`，再查找物品“金创药”的地址是：`328D1DC`，现在我用`79F695C`减去`328D1DC`，得到：`4769780`，这个数就是内力值与金创药的偏移值，没看懂？接着看呀，我还没说完呢，现在重新再运行游戏，查找内力值的地址，得到：`798695C`再查找金创药得到的地址是：`321D1DC`，两个值的内存地址都改变了，但是用你内力值的地址减去金创药的地址得到的结果是什么？没错，还是`4769780`，也就是说，无论这两个值的内存地址变成多少，它们之间的距离是永远不变的，不光是这个游戏，一般的游戏都是，至少我没见过不是的：D

上面讲的东西总结出一个结论，那就是我们只要得到这两个地址中的任何一个，就可以得到另外一个，只要你知道它们之间的偏移量是多少。

我们第一步要做的就是得到这个地址，但是内存中的地址是动态改变的，得到也没有用，这里我就教你把它变成静态的，叫它永远都不变！我继续拿“楚留香新传”为例，如果你有这个游的话就跟我一起做，没有的也没关系，只要看懂这几个步骤就行了。开工！

## 实战：将动态地址转为静态地址

首先进入游戏，查找内值的地址，得到的是：`798695C`（不知道为什么这上游并不是每次重起都改变内存地址），按`Ctrl+D`打开SoftICE，下命令：`BPM 798695C W`（写这个地址时则中断），回到游戏中，打开人物属性面板，游戏中断了，在SofitICE中你会看到这条指令:

```asm
0047EB17  MOV  EAX [EDX+000003F4]
```

下命令：`D EDX+3F4`将看到内力值

```asm
0047EB1D  PUSH  EAX
………………………………
………………………………
```

从上面可看出`0047EB17`处的指令是将内力值的指针送到`EAX`寄存器中，这是一个典型的寻址方式，设想一下，我们是到了`EDX`中的基址，那么无论什么时候只要用`EDX+3F4`就可以轻松的得到内力值的地址，因为`000003F4`是一个常量，它是不会改变的，改变的只是`EDX`中的地址，所以只要有办法得到`EDX`中的值就什么都好办了，你明白了没有？如果还是不懂，那么请再看一遍。现在要做的就是如何得到这个值，下面我教给你如何做:

我的办法就是设计一段代码，把`EDX`中的值存放到一个地址中，然后运行这段代码，再返回游戏的原有指令继续执行，什么？补丁技术？SMC？随你怎么说啦，只要运行正常就一切OK啦：D

### 实际操作

首先在程序中找一段空白处来存放我们设计的代码，很简单，只要懂得一些PE文件结构的朋友都会知道，一般在EXE文件的数据段（`.data`段）的结尾都会有一段缓冲区，我们可以在这段区域中写任何东西，当然你也可以用“90大法”找一段空白区，但我还是推荐你用我教给你的方法。上同我提到，如果你没有PE文件查看工具我可以教你用SoftICE查看，而且很简单，只要一个命令：`MAP32 “模块名”`，看一下我是怎么做的你就知道了。

`Ctrl+D`呼收出SoftICE，然后下命令：`MAP32 CrhChs`,这时你应该看到EXE各个段的信息，我们要注意的只是`.data`段，既然要找的是数据段的结尾，那么我们就从下一个段开始向上找，如下：

```asm
.data  004FB000
.rsrc  00507000
```

`.data`的下一个段是`.rsrc`段，它是从`00507000`开始的，也就是说以`00507000`为基础向上一个字节就是数据段的结尾，我所择从`00506950`处开始写代码，说了这么半天那么我们的代码到底是什么样子呢？修改后的指令又是什么样的呢？别急，请看下面:

修改`0047EB17`后代码：

```asm
0047EB17  JMP    00506950 //跳到我们的代码中去执行
0047EB1C  NOP  //由于这条指令原来的长度是6字节，而修改后的长度是5个字节，所以用一个空指令补上
0047EB1D  PUSH  EAX
```

我们的代码：

```asm
00506950  MOV  DWORD PTR EAX，[EDX+00003F4] //恢复我们破坏的指令
00506956  MOV  DWORD PTR  [00506961],EDX //把EDX保存以00506961中去
0050695C  JMP  0047EB1D //返回原来的指令去执行
```

把上面的代码用SoftICE的`A`命令写入，OK！

现在我们试一下运行的效果，你现在用金山游侠搜索一下内力址的地址，什么又变了？那就地啦，它要是不变我们还用费这么大劲儿吗？记下这个地址返回到游戏中去，`Ctrl+D`呼出SoftICE，下命令：`D \*[00506961]+000003F4`，在数据窗口看到什么了？呵呵，没错，看到了你刚才记住的那个地址，里面的数值正是内力的值，试着改一下，回到游戏中，呵呵，内力值变了吧：D

讲到这里，我们的工作已经完成了%90，但别高兴的太早，后面的%10要远比前的%90花的时间长，因为我们要用编程实现这一切，因为你不能每次都像刚才那样做一次吧！

## 编程实现修改器

现在我来说一下编程的步骤：

首先用`FindWindow`函数得到窗口句柄，然后用`GetWindowThreadID`函数从窗口句柄得到这个进程的ID，接着用`OpenProcess`得到进程的读写权限，最后用`WriteProcessMemory`和`ReadProcessMemory`读写内存，然后。。。。呵呵，你的修改器就做成啦:D

下面是我抄写以前写的修改器源程序片断，第一部分是动态写入刚才的代码，第二部分是读取并修改内力值，由于我没有时间整理和测试，所以不能保证没有错误，如果大家发现有遗漏的话，可以在QQ上给我留言或写信给我，代码如下:

有几点请大家注意：

1. 写机器码时要一个字节一个字节的写
2. 注意要先写入自己的代码，然后再修改游中的指令(下面的代码没有这样做，因为不影响，但是你应该注意这个问题)

```cpp
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

             //动态写入代码

//0047EB17

 #define MY\_CODE1  0xE9

 #define MY\_CODE2  0x34

#define MY\_CODE3  0x7E

 #define MY\_CODE4  0x08

 #define MY\_CODE5

0x00

 #define MY\_CODE6  0x90

       //00506950

#define MY2\_CODE1  0x8B

 #define MY2\_CODE2  0x82

  //这部分是要写入的机器码的常量定义

 #define MY2\_CODE3  0xF4

 #define MY2\_CODE4

0x03

 #define MY2\_CODE5  0x00

 #define MY2\_CODE6  0x00

 #define MY3\_CODE1  0x89

 #define MY3\_CODE2  0x15

 #define

MY3\_CODE3  0x61

 #define MY3\_CODE4  0x69

 #define MY3\_CODE5

0x50

 #define MY3\_CODE6  0x00

 #define MY4\_CODE1  0xE9

 #define MY4\_CODE2  0xBC

 #define MY4\_CODE3  0x81

 #define

MY4\_CODE4  0xF7

 #define MY4\_CODE5  0xFF

 //-----------------------------------------------------------------------------//

 DWORD A1 =MY\_CODE1;

     DWORD A2 =MY\_CODE2;

  DWORD A3 =MY\_CODE3;

     DWORD A4 =MY\_CODE4;

    DWORD A5 =MY\_CODE5;

     DWORD A6 =MY\_CODE6;

     DWORD B1 =MY2\_CODE1;

     DWORD

B2 =MY2\_CODE2;

     DWORD B3 =MY2\_CODE3;  //这部分是变量的定义

     DWORD B4 =MY2\_CODE4;

     DWORD B5

=MY2\_CODE5;

     DWORD B6 =MY2\_CODE6;

     DWORD

C1 =MY3\_CODE1;

     DWORD C2 =MY3\_CODE2;

     DWORD

C3 =MY3\_CODE3;

     DWORD C4 =MY3\_CODE4;

     DWORD

C5 =MY3\_CODE5;

     DWORD C6 =MY3\_CODE6;

     DWORD

D1 =MY4\_CODE1;

     DWORD D2 =MY4\_CODE2;

     DWORD

D3 =MY4\_CODE3;

     DWORD D4 =MY4\_CODE4;

     DWORD

D5 =MY4\_CODE5;

 //--------------------------------------------------------------------------//

     HWND hWnd =::FindWindow("CRHClass",NULL); //得到窗口句柄

    if(hWnd ==FALSE)

         MessageBox("游戏没有运行！");

     else

     {

GetWindowThreadProcessId(hWnd,&hProcId); // 从窗口句柄得到进程ID

HANDLE nOK =OpenProcess(PROCESS\_ALL\_ACCESS|PROCESS\_TERMINATE|PROCESS\_VM\_OPERATION|PROCESS\_VM\_READ|

          PROCESS\_VM\_WRITE,FALSE,hProcId);

//打开进程并得到读与权限

       if(nOK ==NULL)

    MessageBox("打开进程时出错");

       else

       {

              //0047EB17

    WriteProcessMemory(nOK,(LPVOID)0x0047EB17,&A1,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0047EB18,&A2,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0047EB19,&A3,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0047EB1A,&A4,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0047EB1B,&A5,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0047EB1C,&A6,1,NULL);

        //00506950

WriteProcessMemory(nOK,(LPVOID)0x00506950,&B1,1,NULL);

    WriteProcessMemory(nOK,(LPVOID)0x00506951,&B2,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506952,&B3,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506953,&B4,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506954,&B5,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506955,&B6,1,NULL);

    //第二句

           WriteProcessMemory(nOK,(LPVOID)0x00506956,&C1,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506957,&C2,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506958,&C3,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506959,&C4,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0050695A,&C5,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0050695B,&C6,1,NULL);

        //最后一句

           WriteProcessMemory(nOK,(LPVOID)0x0050695C,&D1,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0050695D,&D2,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0050695E,&D3,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x0050695F,&D4,1,NULL);

           WriteProcessMemory(nOK,(LPVOID)0x00506960,&D5,1,NULL);

           CloseHandle(nOK); //关闭进程句柄

       }

     }

 }

 /////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

                     //读取并修改内力值

 DWORD  hProcId;

     HWND hWnd =::FindWindow("CRHClass",NULL);

     if(hWnd ==FALSE)

         MessageBox("No");

     else

     {

GetWindowThreadProcessId(hWnd,&hProcId);

       HANDLE

nOK =OpenProcess(PROCESS\_ALL\_ACCESS|PROCESS\_TERMINATE|PROCESS\_VM\_OPERATION|PROCESS\_VM\_READ|

          PROCESS\_VM\_WRITE,FALSE,hProcId);

       if(nOK ==NULL)

MessageBox("ProcNo!");

       else

{
```
