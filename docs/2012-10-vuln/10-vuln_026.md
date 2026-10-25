
# 缓冲区溢出的简单例子，适合初学者

> 原文：[`www.pediy.com/kssd/pediy11/110187.html`](https://www.pediy.com/kssd/pediy11/110187.html)

前几天在 PYG 里面看到了一段代码，很有意思，差点就把我给迷惑住了，在经过 Think 之后，想通了其中的猫腻，为了验证一下想法，就把代码编译运行，然后用 OD 跟了一下，确认了一下自己的想法。个人认为这是一个新手理解缓冲区溢出的很不错的例子，同时也可以锻炼一下自己的逆向分析能力，其实很简单，大牛们请飘过，新手可以参考学习一下。希望版主能发个邀请码，如果能加精的话那就再好不过了，小弟感激涕零。嘻嘻

首先请看那段代码(C 语言)：

```
#include<stdio.h>
void main()
{
  int i=0;
  int a[]={1,2,3,4,5,6,7,8,9,10};

  for(i=0;i<=10;i++)
  {
    a[i]=0;
    printf("Hello World!\\n");
  }
}
```

大家请先不要看下面的内容，可以看下代码，推算一下这段程序的运行结果。

(使用环境 `windows XP+VC6.0`)

这段代码经过 `VC 6.0` 编译后，运行之，我们看到了程序的结果，在控制台无限制输出了“HelloWorld!”,这是为什么呢？

下面我们用 OD 调试一下，即可发现其中的原因

OD 加载程序，一步步跟，程序在经过初始化运行环境之后，我们来到了这里：

```
00401270  |.  8B15 8C>mov edx,dword ptr ds:[427C8C]
00401276  |.  52      push edx
00401277  |.  A1 847C>mov eax,dword ptr ds:[427C84]
0040127C  |.  50      push eax
0040127D  |.  8B0D 80>mov ecx,dword ptr ds:[427C80]
00401283  |.  51      push ecx
00401284  |.  E8 7CFD>call Text.00401005             ;/////////注意，这是 Main 函数，F7 跟进
```

跟进后我们来到了这里：

```
00401005  /$ /E9 0600>jmp Text.00401010          ;Main 函数入口
0040100A  |  |CC      int3
0040100B  |  |CC      int3
0040100C  |  |CC      int3
0040100D  |  |CC      int3
0040100E  |  |CC      int3
0040100F  |  |CC      int3
00401010  |>  \\55      push ebp                   ;Main 函数开始
00401011  |.  8BEC    mov ebp,esp
00401013  |.  83EC 6C  sub esp,6C                  ;在栈中分配局部变量空间
00401016  |.  53      push ebx
00401017  |.  56      push esi
00401018  |.  57      push edi
00401019  |.  8D7D 94  lea edi,dword ptr ss:[ebp-6C]
0040101C  |.  B9 1B00>mov ecx,1B
00401021  |.  B8 CCCC>mov eax,CCCCCCCC
00401026  |.  F3:AB   rep stos dword ptr es:[edi]
00401028  |.  C745 FC>mov dword ptr ss:[ebp-4],0                     ;  i=0;
0040102F  |.  C745 D4>mov dword ptr ss:[ebp-2C],1                    ;  a[0]=1;   \\
00401036  |.  C745 D8>mov dword ptr ss:[ebp-28],2                    ;  a[1]=2;   |
0040103D  |.  C745 DC>mov dword ptr ss:[ebp-24],3                    ;  a[2]=3;   |
00401044  |.  C745 E0>mov dword ptr ss:[ebp-20],4                    ;  a[3]=4;   |
0040104B  |.  C745 E4>mov dword ptr ss:[ebp-1C],5                    ;  a[4]=5;   |初始化 a 数组
00401052  |.  C745 E8>mov dword ptr ss:[ebp-18],6                    ;  a[5]=6;   |
00401059  |.  C745 EC>mov dword ptr ss:[ebp-14],7                    ;  a[6]=7;   |
00401060  |.  C745 F0>mov dword ptr ss:[ebp-10],8                    ;  a[7]=8;   |
00401067  |.  C745 F4>mov dword ptr ss:[ebp-C],9                     ;  a[8]=9;   |
0040106E  |.  C745 F8>mov dword ptr ss:[ebp-8],0A                    ;  a[9]=10;  /
00401075  |.  C745 FC>mov dword ptr ss:[ebp-4],0                     ;  for(i=0;
0040107C  |.  EB 09   jmp short Text.00401087
0040107E  |>  8B45 FC  /mov eax,dword ptr ss:[ebp-4]                  ;  把 i 放入 EAX
00401081  |.  83C0 01  |add eax,1                                 ;  i++;
00401084  |.  8945 FC  |mov dword ptr ss:[ebp-4],eax
00401087  |>  837D FC>  cmp dword ptr ss:[ebp-4],0A                   ;  i<=10
0040108B  |.  7F 1A   |jg short Text.004010A7
0040108D  |.  8B4D FC  |mov ecx,dword ptr ss:[ebp-4]
00401090  |.  C7448D  >|mov dword ptr ss:[ebp+ecx*4-2C],0
00401098  |.  68 A42F>|push Text.00422FA4                            ; /Arg1 = 00422FA4 ASCII "Hello World!
"
0040109D  |.  E8 3E00>|call Text.004010E0                            ; \\printf("Hello World!\\n);
004010A2  |.  83C4 04  |add esp,4
004010A5  |.^ EB D7   \\jmp short Text.0040107E                       ;  当 i<=10 时  继续循环
004010A7  |>  5F      pop edi
004010A8  |.  5E      pop esi
004010A9  |.  5B      pop ebx
004010AA  |.  83C4 6C  add esp,6C
004010AD  |.  3BEC    cmp ebp,esp
004010AF  |.  E8 AC00>call Text.00401160
004010B4  |.  8BE5    mov esp,ebp
004010B6  |.  5D      pop ebp
004010B7  \\.  C3      retn
```

我们知道，函数的参数和局部变量是在栈中存放的，所以我们在程序中所定义的局部变量 `i` 和数组 `a`,我们可以在栈内存中查看，不难发现情况如下：

```
0012FF54   00000001         \\
0012FF58   00000002         |
0012FF5C   00000003         |数组 a 在栈中存放
0012FF60   00000004         |
0012FF64   00000005         |
0012FF68   00000006         |
0012FF6C   00000007         |
0012FF70   00000008         |
0012FF74   00000009         |
0012FF78   0000000A        /
0012FF7C   00000000           在数组 a 下一个地址，存放的是我们在程序中的计数器 i
```

到这里，我们就可以看出其中的猫腻了，程序中 `for` 循环一共要执行 11 次，可是我们所定义的数组 `a` 只有 10 个元素，在执行第 11 次循环时，发生了溢出，程序把存放在 `a` 下面的计数器 `i` 也赋值为 0 了，此时 `i` 又小于等于 10 了，这样就会无限制的执行下去，`i` 永远也不可能超过 10，程序就成了无限循环。

总结：这是一个很简单很简单的例子，不过对于这个例子相信对于初学者理解缓冲区溢出，会有帮助。由于在对数组 `a` 进行赋值操作时，没有检查数组边界，造成了把下一块内存中的数据覆盖，导致程序的死循环。但是当有心人发现这一点后，就可以加以利用，这里是覆盖了计数器 `i` 的值，可是在经过精心的计算后，可以把函数的返回地址覆盖(我们知道，在调用子函数时，会把子函数要返回的指令地址压入堆栈，在子函数执行完后，会把存入栈中的母函数地址 `pop` 给 `EIP`,从而返回母函数继续运行)，那样就可以得到程序的控制权，使程序跳转到自己想要的地方，有时这也是破解的一种法方。因此，我们在今后的写程序中也要注意这一点。

到这里就结束了，希望对于初学者们能有一下帮助，写的不好，请多包含。(附上程序源码)

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || Text.rar  |
