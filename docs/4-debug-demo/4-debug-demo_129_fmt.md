
# 利用程序原来的代码写注册机

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-339.htm`](https://www.pediy.com/kssd/pediy08/pediy8-339.htm)

【文章标题】: 利用程序原来的代码写注册机

【文章作者】: laomms

【软件名称】: serial

【下载地址】: 附件

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

【详细过程】

有时候，懒得写注册机，就想把原来的程序中的部分汇编代码套用到高级语言中，于是找了一个最简单的程序做了测试。事实证明，这也是非常繁琐的事情。这个程序用高级语言写可能就一会儿，可是我用嵌入汇编的方式，却花了不少时间。既然已经做了，也就把步骤贴出来分享一下。

算法核心部分：

```
00401228   .  68 8E214000   PUSH 0040218E                     ;  用户名
0040122D   .  E8 4C010000   CALL 0040137E                     ;  用户名算法
00401232   .  50            PUSH EAX                           ;  返回结果
00401233   .  68 7E214000   PUSH 0040217E                     ;  注册码
00401238   .  E8 9B010000   CALL 004013D8                     ;  注册码算法
0040123D   .  83C4 04       ADD ESP,4                          ;  返回结果在 EBX
00401240   .  58            POP EAX                            ;  弹出用户名计算结果
00401241   .  3BC3          CMP EAX,EBX                        ;  比较是否相等，相等就成功
00401243   .  74 07         JE SHORT 0040124C
```

找到关键算法后我用 `Asm2Clipboard` 插件将 `CALL 0040137E` 里的汇编代码全部复制出来：

用户名的算法 `CALL`，它的过程是将每位注册码的 `ASCII` 码异或 `5678` 后相加，也就是注册码的 `ASCII` 的和异或 `5678`。`Asm2Clipboard` 复制的结果（附算法分析）。

```
MOV ESI,DWORD PTR SS:[ESP+4]           //这里的 ESP+4 我们准备放入自己的用户名 name
PUSH ESI                                //压入注册码
@002:
    MOV AL,BYTE PTR DS:[ESI]           //逐个取用户名 ASCII
    TEST AL,AL                          //判断 AL 是否为 0，即注册码是否取完，这个把它去掉
    JE @014                             //如果取完就跳到@014，去@014 处看看
    CMP AL,41                           //是否大于 41H，即大于“A”，要改为 CMP AL,$41
    JB @019                             //没有就跳到@019 提示，去@019 处看看
    CMP AL,5A                           //是否大于 5AH，即大于“Z”，这个要改成 16 进制形式：CMP AL,$5A
    JNB @011                            //如果大于就跳到@011 进行变化，减 20H 转大写，去@011 处看看
    INC ESI                             //取下一个注册码
    JMP @002                            //循环执行

@011:
    CALL @035                           //呼叫@035 处的代码，即将每位注册码的 ASCII 减 20H，即小写转大写，我们直接把@035 处的代码移过来放这里
    INC ESI                             //取下一个注册码
    JMP @002                            //重新跳到判断注册码的地方

@014:
    POP ESI                             //注册码出栈
    CALL @026                           //执行@026 处代码，是个寄存器清空操作，我们直接把@026 处的代码移上来
    XOR EDI,5678                        //EDI=EDI XOR 5678
    MOV EAX,EDI                         //将结果存入 EAX，这个 EAX 就是算法 CALL 的最终返回值，也就是我们要的结果
    JMP @025                            //跳到@025 处，@025 是个返回操作，就是退出整个算法 CALL，我们去@025 看看

@019:
    POP ESI
    PUSH 30                             //对话框样式。
    PUSH 00402160                       //对话框标题，改为自定义的标题,MOV EAX,DWORD PTR SS:[title],  PUSH EAX
    PUSH 00402169                       //对话框内容，改为自定义的内容,MOV EAX,DWORD PTR SS:[text],  PUSH EAX
    PUSH DWORD PTR SS:[EBP+8]           //对话框句柄，改为 PUSH 0
    CALL <JMP.&USER32.MessageBoxA>      //MessageBox 函数，改为 CALL MessageBoxA

@025:
    RETN                                //这个返回就是整个算法过程完成后退出整个算法 CALL。我们可以利用它，把结果转移到我们要的变量上，比如改为：MOV SN，
                                        //EAX

@026:
    XOR EDI,EDI                         //EDI 清空
    XOR EBX,EBX                         //EBX 清空，EBX 不能用，改用为：XOR ECX,ECX

@028:
    MOV BL,BYTE PTR DS:[ESI]            //逐个取注册码 ASCII 放入 BL，BL 改 CL。
    TEST BL,BL                          //判断 BL 是否为 0，即所有的注册码是否运算完毕，这个把它去掉，BL 改 CL
    JE @034                             //如果取完了就跳到@034，@034 只是一个返回操作，就是退出这个循环
    ADD EDI,EBX                         //EDI=EDI+EBX，EDI 放着最终的结果，EBX 改 ECX
    INC ESI                             //取下一位注册码
    JMP @028                            //循环操作

@034:
    RETN                                //就是 CALL @026 返回操作，原程序是返回到@014 中的 XOR EDI,5678。所以我们可以把 XOR EDI,5678 划一个段落为@034，并把这
                                        //个删除掉

@035:
    SUB AL,20                           //AL-20，AL 放的是大于 5A 的字符
    MOV BYTE PTR DS:[ESI],AL            //将结果放回段寄存器 DS:[ESI]
    RETN                                //这个返回，就是 CALL @035 的返回，原程序是返回到@002 中 INC ESI 处。
```

注册码的算法 `CALL`，将注册码转成 16 进制值后异或 `1234`：

```
XOR EAX,EAX                   //EAX 清空
XOR EDI,EDI                   //EDI 清空
XOR EBX,EBX                   //EBX 清空
MOV ESI,DWORD PTR SS:[ESP+4]  //逐个取注册码 ASCII
@004:
    MOV AL,0A                 //0A 赋值给 AL，这句要改为 MOV AL,$0A
    MOV BL,BYTE PTR DS:[ESI]  //逐个取出注册码 ASCII 放入 BL
    TEST BL,BL                //判断注册码是否取完
    JE @013                   //取完就跳到@013
    SUB BL,30                 //注册码减 30H,改 SUB BL,30H
    IMUL EDI,EAX              //EDI=EDI*EAX，EAX=0A
    ADD EDI,EBX               //EDI=EDI+EBX，EDI 为结果
    INC ESI                   //取下一位注册码
    JMP @004                  //循环
@013:
    XOR EDI,1234              //EDI 异或 1234
    MOV EBX,EDI               //EDI 的值送入 EBX，EBX 为最终的结果
    RETN
```

程序要求这个两个值相等，也就是 `SUM（NAME）XOR 5678=HEX（serial）XOR 1234`，异或是可以逆的，这个用高级语言来写非常简单 `serial=NAME XOR 5678 XOR 1234`，也可以将原来的程序改装成注册机。我这里想利用原来的代码做出注册机，所以只要将用户名的计算过程再加个异或 `1234` 即可，所以我在原先的代码中加入 `XOR EDI,$1234` 即可。将代码稍微改了下，并加入一个对话框以便没有输入用户名时提示一下：

其中 `NAME` 是用户名，`SN` 是算法 `CALL` 的计算结果

```
MOV ESI,DWORD PTR SS:[name]
PUSH ESI
JE @1                     //这里加入没有输入注册码时提示的对话框
XOR EAX,EAX
@002:
    MOV AL,BYTE PTR DS:[ESI]
    TEST AL,AL
    JE @014
    CMP AL,$41
    JB @019
    CMP AL,$5A
    JNB @011
    INC ESI
    JMP @002
@011:
    SUB AL,$20            //这里提前了
    MOV BYTE PTR DS:[ESI],AL
    INC ESI
    JMP @002
@014:
    POP ESI
    XOR EDI,EDI           //这里提前了
    XOR ECX,ECX
@028:
    MOV CL,BYTE PTR DS:[ESI]
    TEST CL,CL
    JE @034
    ADD EDI,ECX
    INC ESI
    JMP @028
@034:
    XOR EDI,$5678
    XOR EDI,$1234
    MOV EAX,EDI
    JMP @025
@019:                    //这里是提示用户名不能为数字
    push 0
    MOV EAX,DWORD PTR SS:[title]
    PUSH EAX
    MOV EAX,DWORD PTR SS:[text]
    PUSH EAX
    PUSH 0
    CALL MessageBox
    POP ESI
    JMP @2
@1:                      //这是没有输入用户名的对话框提示
    push 0
    MOV EAX,DWORD PTR SS:[title]
    PUSH EAX
    MOV EAX,DWORD PTR SS:[text1]
    PUSH EAX
    PUSH 0
    CALL MessageBox
    POP ESI
    JMP @2
@025:
    MOV SN,EAX
@2:
```

简直就是个汇编注册机了，呵呵。

我这里套在 `DELPHI` 中，具体看附件。

我觉得有时候拿程序的部分代码套在汇编中还是有一定的好处，特别是大数计算和位操作。

我也希望有大虾出一篇专门介绍高级语言嵌入汇编的文章。

---

【版权声明】: 没有任何技术, 大家见笑了, 谢谢!

2006 年 08 月 30 日 16:19:32

这文要支持啊。。。。。顶

用 `C` 写了一下；

```c
#include<stdio.h>
#include<string.h>

main()
{
    int len,i,sum=0,sum1;
    char name[20];
    gets(name);
    len=strlen(name);
    for(i=0;i<len;i++)
    {
        sum1=name[i];
        sum1-=32;
        sum+=sum1;
    }
    sum=sum^0x5678^0x1234;
    printf("%d\n",sum);
}
```

引用:  *最初由** 学习 **发布*

顶。

看不懂 `keymake` 当中是如何用汇编写的注册机。这个先学习。

如果利用 `KEYMAKE` 就更简单了.

在数据中定义一些变量:

```
szHomePage db "http://bbs.pediy.com",0
szEmail    db "mailto:langxang@126.com",0
szErrMess  db "输入的序列号不正确！",0
szErrName  db "输入的必须为英文字母！",0
szFormat   db "%d",0  ;将除数的注册码转为十进制字符格式
szBuff     db 50 dup(0) ;定义 50 个字节的内存空间，用于保存注册码。
```

在代码窗口中写入:

```
lea eax,hInput1
XOR ESI,ESI
MOV ESI,EAX
```



## 汇编代码逻辑

```
PUSH ESI

XOR EAX,EAX

  n2:

  MOV AL,BYTE PTR [ESI]
  TEST AL,AL
  JE n4
  CMP AL,41h
  JB n9
  CMP AL,5Ah
  JNB n1
  INC ESI
  JMP n2

  n1:

  SUB AL,20h               
  MOV BYTE PTR [ESI],AL
  INC ESI
  JMP n2

  n4:

  POP ESI
  XOR EDI,EDI               
  XOR ECX,ECX

  n8:

  MOV CL,BYTE PTR [ESI]
  TEST CL,CL
  JE n6
  ADD EDI,ECX
  INC ESI
  JMP n8

  n6:

  XOR EDI,5678h
  XOR EDI,1234h
  MOV EAX,EDI               
  JMP n5     
  
  n9:               
  lea eax,szErrName       ; 提示输入的必须为字母

  n5:

  invoke wsprintf,addr szBuff,addr szFormat,edi  ; 将注册码转 10 进制

  LEA EAX,szBuff          ; EAX 指向注册码的框
```

编译后就是个算法注册机了.

## C/C++ 语言实现优势

其实用 C 和 C++语言写注册机还是蛮爽的。

如楼上所说，不但必要的时候可以嵌入汇编码；而且这 2 种语言可以很有效的模拟汇编算法：

汇编代码中的寄存器是 32 位的，那我们可以定义一个 `unsigned long` 类型的整型变量 `var`，就可以代表该寄存器；另外很多运算符号也可以很方便的使用，例如：

* `AND eax, 5` 就可以写成 `(var | 5)` // `unsigned long var = ax;`
* `MOVSX eax,al` 就可以写成 `var = var1;` // `unsigned long var = ax; int var1 = int(al);`
