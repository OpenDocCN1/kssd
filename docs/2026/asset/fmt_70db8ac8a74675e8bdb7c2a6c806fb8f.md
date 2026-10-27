
#### 3.进阶花指令(自定义花指令)

前面几种花指令都是比较老套的，入门花指令还能勉勉强强骗过反编译器，不过有经验的逆向者一眼就能识破，以下几种花指令形式，可以任由自己构造。

##### a.替换`ret`指令

```asm
_asm
{
call LABEL9;
_emit 0xE8;
_emit 0x01;
_emit 0x00;
_emit 0x00;
_emit 0x00;
LABEL9:
push eax;
push ebx;
lea eax, dword ptr ds : [ebp - 0x0];
#将ebp的地址存放于eax
add dword ptr ss : [eax-0x50] , 26;
#该地址存放的值正好是函数返回值，
#不过该地址并不固定，根据调试所得。
#加26正好可以跳到下面的mov指令，该值也是调试计算所得
pop eax;
pop ebx;
pop eax;
jmp eax;
_emit 0xE8;
_emit 0x03;
_emit 0x00;
_emit 0x00;
_emit 0x00;
mov eax,dword ptr ss:[esp-8];
#将原本的eax值返回eax寄存器
}
```

由于：

> `call`指令的本质：`push 函数返回地址`然后`jmp 函数地址`
> `ret`指令的本质：`pop eip`

两者都是对寄存器`eip`中存放的地址的操作。

所以我们可以在`call`指令之后，清楚的明白函数返回地址存放于`esp`，可以将值取出，用跳转指令跳转到该地址，即可代替`ret`指令。

当然，这种构造跳转指令可以变化多样。

##### b.控制标志寄存器跳转

这一部分需要精通标志寄存器，每一个操作码都会对相应的标志寄存器产生相应的影响，如果我们对标志寄存器足够熟练，就可以使用对应的跳转指令**构造永恒跳转**！

##### c.利用函数返回确定值

有些函数返回值是确定的，比如我们自己写的函数，返回值可以是任意非零整数，就可以自己**构造永恒跳转**。

还有些api函数也是如此：

一方面可以传入一些错误的参数，如<a href="elink@fdbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1L8r3!0S2k6r3I4A6j5Y4u0S2M7Y4W2S2">`LoadLibraryA`</a>

```apl
HMODULE LoadLibraryA(
LPCSTR lpLibFileName
);
```

如果我们故意传入一个不存在的模块名称，那么他就会返回一个确定的值`NULL`，我们就可以通过这个**构造永恒跳转**

另一方面，某些api函数，我们既然使用他，肯定就是一定要调用成功的，而这些api函数基本上只要调用成功就会返回一个确定的零或者非零值，如<a href="elink@e45K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8Y4g2K6k6i4u0Q4x3V1k6F1k6W2)9J5k6s2N6A6L8Y4g2K6k6i4u0Q4x3X3c8E0k6i4y4K6j5h3N6W2j5X3!0^5">`MessageBox`</a>

```apl
int MessageBox(
HWND hWnd,
LPCTSTR lpText,
LPCTSTR lpCaption,
UINT uType
);
```

该api只有在其调用失败的时候才能返回零，那么我们也可以通过这一点**构造永恒跳转**

PS：利用`MessageBox`实现花指令也是我在**1.花指令简介**中用到的源码

#### 4.花指令原理另类利用

当我们理解了花指令的原理后，我们可以在将花指令中的垃圾数据替换为一些特定的**特征码**，可以对应的“定位功能”，尤其在**<a href="elink@c71K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3c8G2L8Y4c8T1k6h3y4G2k6r3g2J5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5y4K6f1@1y4K6t1&6i4K6y4r3N6i4c8E0i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5j5i4m8H3">SMC自解码</a>**这个反调试技术中可以运用。例如：

```asm
asm
{
Jz Label
Jnz Label
_emit 'h'
_emit 'E'
_emit 'l'
_emit 'L'
_emit 'e'
_emit 'w'
_emit 'o'
_emit 'R'
_emit 'l'
_emit 'D'
Label:
}
```

将这串特征码`hElLowoRlD`嵌入到代码中，那我们只需要在当前进程中搜索`hElLowoRlD`字符串，就可以定位到当前代码位置，然后对下面的代码进行SMC自解密。

### 花指令 指令小结

#### `jz jnz/jmp`

```asm
__asm {
_emit 075h #jmp $+4
_emit 2h
_emit 0E9h
_emit 0EDh
}
```

![image-20230816110539561](upload/attach/202311/957393_GURY7GR4H2UQBTE.jpg)

9是`jmp`指令对应的机器码，当反汇编器读取到`E9`时，接着会往下读取四个字节的数据作为跳转地址的偏移，所以才会看到错误的汇编代码。

#### `call ret`

> `call+pop/add esp/add [esp] + retn`

```cpp
#include <iostream.h>
#include <windows.h>
void main()
{
DWORD p;
__asm
{
call l1
l1:
pop eax
mov p,eax//确定当前程序段的位置
call f1
_EMIT 0xEA//花指令，此处永远不会执行到
jmp l2//call结束以后执行到这里
f1://这里用F8OD会终止调试，F7跟进的话就正常,why?
pop ebx
inc ebx
push ebx
mov eax,0x11111111
ret
l2:
call f2//用ret指令实现跳转
mov ebx,0x33333333//这里永远不会执行到
jmp e//这里永远不会执行到
f2:
mov ebx,0x11111111
pop ebx//弹出压栈的地址
mov ebx,offset e//要跳转到的地址
push ebx//压入要跳转到的地址
ret//跳转
e:
mov ebx,0x22222222
}
cout<<hex<<p<<endl;
}
```

`call`指令可以理解为`jmp` + `push ip`，因此如果通过`add esp,4`来降低栈顶即可去除`push ip`的影响，从而使`call`等价于`jmp`，但IDA会认为这是函数的分界，从而导致函数的范围识别错误

##### reference

<mark class="encrypted">14bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8q4)9J5k6s2Z5I4x3q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9#2i4K6u0r3x3e0N6Q4x3V1k6X3L8r3!0%4k6i4u0Q4x3X3g2Z5N6r3#2D9</mark>

### 实例

#### 源码

```cpp
#include <stdio.h>
void func1()
{
__asm
{
lea eax, lab1
jmp eax
_emit 0x90
};
lab1:
printf("func1\n");
}
void func2()
{
__asm
{
cmp eax, ecx
jnz lab1
jz lab1
_emit 0xB8
};
lab1:
printf("func2\n");
}
int main()
{
func1();
func2();
return 0;
}
```

![image-20230816093751805](upload/attach/202311/957393_EJ58TG492D9C9US.jpg)

此外还有更多形式的花指令，读者请根据指令含义自行探索。

#### 花指令逆向分析

> IDA有栈跟踪的功能，它在函数内部遇到`ret`(`retn`)指令时会做判断：栈指针的值在函数的开头/结尾是否一致，如果不一致就会在函数的结尾标注 `sp-analysis failed`。一般编程中，不同的函数调用约定（如`stdcall` & `_cdcel` `call`）可能会出现这种情况；另外，为了实现代码保护而加入代码混淆（特指用`push/push+ret`实现函数调用）技术也会出现这种情况。

对上文编写的花指令程序进行逆向分析：

![image-20230816103200617](upload/attach/202311/957393_UGHTCC9HH93QU2Y.jpg)

加入花指令改变了`esp`的空间而实际有效代码的空间并没有这么大，在`esp`恢复时（出栈），会加上比实际值大的数字，从而导致栈指针不平衡。

## 花指令分析实操

### 1. `_emit 0xe8`

以下内容摘自ctfwiki

```cpp
// 正常的函数代码
int add(int a, int b){
int c = 0;
c = a + b;
return c;
}
// 添加花指令的函数代码
int add_with_junk(int a, int b){
int c = 0;
__asm{
jz label;
jnz label;
_emit 0xe8; call 指令，后面加4bytes的地址偏移，因此导致反汇编器不能正常识别
label:
}
c = a + b;
return c;
}
```

使用 ida 的反编译时，添加了花指令的函数不能正常识别，结果如下：

伪代码：

```asm
// 添加了花指令
.text:00401070 loc_401070: ; CODE XREF: sub_401005↑j
.text:00401070 push ebp
.text:00401071 mov ebp, esp
.text:00401073 sub esp, 44h
.text:00401076 push ebx
.text:00401077 push esi
.text:00401078 push edi
.text:00401079 lea edi, [ebp-44h]
.text:0040107C mov ecx, 11h
.text:00401081 mov eax, 0CCCCCCCCh
.text:00401086 rep stosd
.text:00401088 mov dword ptr [ebp-4], 0
.text:0040108F jz short near ptr loc_401093+1
.text:00401091 jnz short near ptr loc_401093+1
.text:00401093
.text:00401093 loc_401093: ; CODE XREF: .text:0040108F↑j
.text:00401093 ; .text:00401091↑j
.text:00401093 call near ptr 3485623h
.text:00401098 inc ebp
.text:00401099 or al, 89h
.text:0040109B inc ebp
.text:0040109C cld
.text:0040109D mov eax, [ebp-4]
.text:004010A0 pop edi
.text:004010A1 pop esi
.text:004010A2 pop ebx
.text:004010A3 add esp, 44h
.text:004010A6 cmp ebp, esp
.text:004010A8 call __chkesp
.text:004010AD mov esp, ebp
.text:004010AF pop ebp
.text:004010B0 retn
```

在上面这个例子中，把混淆视听的花指令 patch 成 `nop` 即可修复，然后正常分析。

值得注意的是，ida 对于栈的判定比较严格，因此 `push`、`ret` 一类的花指令会干扰反汇编器的正常运行，下面给出一个具体的例子，读者可以自己编译复现：

```asm
#include <stdio.h>
// 使用 gcc/g++ 进行编译
int main(){
__asm__(".byte 0x55;"); // push rbp 保存栈
__asm__(".byte 0xe8,0,0,0,0;"); // call $5;
__asm__(".byte 0x5d;"); // pop rbp -> 获取rip的值
__asm__(".byte 0x48,0x83,0xc5,0x08;"); // add rbp, 8
__asm__(".byte 0x55;"); // push rbp -> 相当于将call的返回值修改到下面去
__asm__("ret;");
__asm__(".byte 0xe8;"); // 这是混淆指令不执行
__asm__(".byte 0x5d;"); // pop rbp 还原栈
printf("whoami \n");
return 0;
}
```

### 2. `看雪.TSRC 2017CTF秋季赛`第二题

这里以`看雪.TSRC 2017CTF秋季赛`第二题作为讲解. 题目下载链接: <a href="elink@c7cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3j5%4c8X3i4K6u0V1j5$3S2S2L8r3I4W2L8X3N6W2M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6J5k6i4k6W2M7Y4y4W2i4K6u0r3j5h3&6@1K9g2)9J5k6r3c8W2j5Y4g2Y4i4K6u0r3x3U0l9I4y4#2)9#2k6Y4m8W2k6r3W2&6i4K6u0r3j5%4c8X3x3U0l9I4y4#2)9#2k6V1k6H3j5#2)9J5k6h3g2^5k6b7`.`.">`ctf2017_Fpc.exe`</a>

程序写了几个函数混淆视听, 将关键的验证逻辑加花指令防止了 IDA 的静态分析. 我们用 IDA 打开 `Fpc` 这道题, 程序会先打印一些提示信息, 然后获取用户的输入。

![image-20230816143658327](upload/attach/202311/957393_8APKXNS4T93UTC8.jpg)

这里使用了不安全的`scanf`函数, 用户输入的缓冲区只有`0xCh`长, 我们双击`v1`进入栈帧视图

![stack.png](upload/attach/202311/957393_PCEXUYJGV3DKM26.jpg)

因此我们可以通过溢出数据, 覆盖掉返回地址, 从而转移到任意地址继续执行.

这里我还需要解释一下, 就是`scanf`之前写的几个混淆视听的函数, 是一些简单的方程式但实际上是无解的. 程序将真正的验证逻辑加花混淆, 导致 IDA 无法很好的进行反编译. 所以我们这道题的思路就是, 通过溢出转到真正的验证代码处继续执行.

我们在分析时可以在代码不远处发现以下数据块.

![](upload/attach/202311/957393_TZ7D987ZQ25ZE59.jpg)

因为 IDA 没能很好的识别数据, 因此我们可以将光标移到数据块的起始位置, 然后按下`C`键 (code) 将这块数据反汇编成代码

![image-20230816144445545](upload/attach/202311/957393_FS4PJRJ2M9P6U3G.jpg)

值得注意的是, 这段代码的位置是`0x00413131`, `0x41`是`'A'`的 ascii 码，而`0x31`是`'1'`的 ascii 码. 由于看雪比赛的限制, 用户输入只能是字母和数字, 所以我们也完全可以利用溢出漏洞执行这段代码

用 OD 打开, 然后`Ctrl+G`到达`0x413131`处设下断点, 运行后输入`12345612345611A`回车, 程序成功地到达`0x00413131`处. 然后`右键分析->从模块中删除分析`识别出正确代码

![entry.png](upload/attach/202311/957393_ZWSR3M9N6GUXB3H.jpg)

断在`0x413131`处后, 点击菜单栏的`"查看"`, 选择`"RUN跟踪"`, 然后再点击`"调试"`, 选择`"跟踪步入"`, 程序会记录这段花指令执行的过程, 如下图所示:
