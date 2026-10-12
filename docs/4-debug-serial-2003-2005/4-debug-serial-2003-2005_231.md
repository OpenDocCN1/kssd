
# QuickCD V1.0.4

> 原文：[`www.pediy.com/kssd/pediy05/pediy50636.htm`](https://www.pediy.com/kssd/pediy05/pediy50636.htm)

这是我第一次写`pj`文章，希望大家能给点鼓励:)

`QuickCD V1.0.4`

软件大小： 59 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 光驱工具

应用平台： `Win9x/NT/2000/XP`

界面预览： 无

加入时间： 2003-04-18

联系人： `QuickMore@Hotmail.com`

开发商： [`www.websamba.com/morequick`](http://www.websamba.com/morequick)

软件介绍：

`QuickCD`可以帮助你方便快捷的打开关闭光驱门,你可以选择热键或者鼠标点击两种方式:

1.  热键(`F9`:打开光驱门,`F10`:关闭光驱门,`F11`:弹出设置对话框),当然你可以设置新的热键.
2.  鼠标点击(点击托盘图标:打开光驱门,右击托盘图标:关闭光驱门,右双击托盘图标:弹出菜单)

下载地址：`http://www.skycn.com/soft/6381.html`

工具：`softice4.05`

进入软件注册画面：

输入注册码：`12345678`

调出`softice`，下断点`bpx getwindowtexta`，回到软件，点击注册后中断，`F12`返回程序领空，

到这里

```
001B:0040A9C6  8B4C2408                MOV       ECX,[ESP+08]
001B:0040A9CA  6AFF                    PUSH      FF
001B:0040A9CC  E8FA380000              CALL      0040E2CB
001B:0040A9D1  EB0C                    JMP       0040A9DF
001B:0040A9D3  8B01                    MOV       EAX,[ECX]
001B:0040A9D5  FF742408                PUSH      DWORD PTR [ESP+08]
001B:0040A9D9  FF9088000000            CALL      [EAX+00000088]
001B:0040A9DF  5E                      POP       ESI
```

一直按`F10`，此时`Eax`值没有变化，所以一直走下去

到这里

```
001B:00402342 8B442414                mov eax, dword ptr [esp+14]//这里 eax 变化，用 d eax,看到什么?我的机器吗!!!
001B:00402346 50                      push eax//机器码入栈,为下面的 call 做准备
001B:00402347  E84B180000             CALL      00403B97//这个 call 的作用是将机器吗的十进制值转化为十六进制值,怎么知道呢?call 完之后用?eax 就知道了
001B:0040234C
  8B4C2410                  MOV       ECX,[ESP+10]//假注册码给 Ecx
001B:00402350  8BF8                    MOV       EDI,EAX//eax 赋值给 edi,也就是 edi 中放着机器吗
001B:00402352
  51                          PUSH      ECX//ecx(假注册码)入栈
001B:00402353  E83F180000              CALL      00403B97//同上面的 call 一样把假注册码转为十六进制值
001B:00402358
  57                          PUSH      EDI//机器吗入栈,机器吗和假注册码都入栈了,后面有好戏看了:)
001B:00402359
  8BE8                        MOV       EBP,EAX//eax 赋值给 ebp
001B:0040235B  E860FFFFFF               CALL      004022C0//这个 call 不知道是干嘛的,搁着.....
001B:00402360
  83C40C                      ADD       ESP,0C
001B:00402363  3BC5                    CMP       EAX,EBP//关键比较
001B:00402365
  0F8587000000                JNZ       004023F2//关键跳转,怎么知道是关键跳转呢?这里有个技巧,用 r fl z 将 zf 置 1,接着按 F5，看到了什么？注册错误！！！说明这是个关键跳转，不用说上面就是关键跳转了，那么是什么改变了 eax 呢?在 001B:00402359 处 eax 不是赋值给了 ebp 吗？两者应该相等才对！！！
```

呵呵，现在应该知道`0040235B`处的`call`是干什么的吧，他就是关键`call`，计算注册码后，改变了`eax`的值，现在我们`F8`进入到关键`call``004022C0`

```
001B:004022C0  8B442404                MOV       EAX,[ESP+04]//机器吗放入 eax
001B:004022C4  35AC0BBB02              XOR       EAX,02BB0BAC//eax 与 0X02BB0BAC 异或结果放入 eax
001B:004022C9  05FD593202              ADD       EAX,023259FD//eax+0X023259FD
001B:004022CE  7905                    JNS       004022D5//SF=0 则跳走
001B:004022D0  99                      CDQ
001B:004022D1  33C2                    XOR       EAX,EDX
001B:004022D3  2BC2                    SUB       EAX,EDX
001B:004022D5  C3                      RET
```

注册码的计算过程就这么一点，简单吧:)

注册成功后注册码写入同目录下的`QuickCD.ini`文件中

`c++`源码：

```cpp
#include<iostream.h>

void main()
{
int mac\_code,reg\_code;

cout<<"\\n\*\*\*\*\*\*\*\*\*\*\*\*\*KenGen BY NetFox\*\*\*\*\*\*\*\*\*\*\*\*\*"<<endl
                <<"\\n";

cout<<"Machine Code Please:";

cin>>mac\_code;

mac\_code^=0x02bb0bac;

reg\_code=mac\_code+0x023259fd;

cout<<"Your Register Code Is:"<<reg\_code<<endl<<"\\n";

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
