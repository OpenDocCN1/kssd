
![trace.png](upload/attach/202311/957393_BPP9PWWSSQFGRZM.jpg)

这段花指令本来很长, 但是使用 OD 的跟踪功能后, 花指令的执行流程就非常清楚. 整个过程中进行了大量的跳转, 我们只要取其中的有效指令拿出来分析即可.

需要注意的是, 在有效指令中, 我们依旧要满足一些条件跳转, 这样程序才能在正确的逻辑上一直执行下去.

比如`0x413420`处的`jnz ctf2017_.00413B03`. 我们就要重新来过, 并在`0x413420`设下断点

![jnz.png](upload/attach/202311/957393_3YQGMC44UJ6MQ8H.jpg)

通过修改标志寄存器来满足跳转. 继续跟踪步入 (之后还有`0041362E jnz ctf2017_.00413B03`需要满足). 保证逻辑正确后, 将有效指令取出继续分析就好了

![register.png](upload/attach/202311/957393_HD7ZYY4RGCVAAUW.jpg)

### 3. `_asm` `_emit` 0E9 + 可执行花指令

#### reference

<mark class="encrypted">5cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3z5o6j5^5x3R3`.`.</mark>

如下有tea加密算法

```
#include <stdio.h>
#include <stdint.h>
void encrypt(uint32_t* v, uint32_t* k) {
uint32_t v0 = v[0], v1 = v[1], sum = 0, i; /* set up */
uint32_t delta = 0x9e3779b9; /* a key schedule constant */
uint32_t k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3]; /* cache key */
for (i = 0; i < 32; i++) { /* basic cycle start */
sum += delta;
v0 += ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1);
v1 += ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3);
} /* end cycle */
v[0] = v0; v[1] = v1;
}
int main() {
int a = 1;
uint32_t flag[] = { 1234,5678 };
uint32_t key[] = { 9,9,9,9 };
encrypt(flag, key);
printf("%d,%d", flag[0], flag[1]);
return 0;
}
```

#### 加入花指令

接下来我们加入两个花指令

第一个花指令在`main`函数中，就是我们上面提供的最简单的花指令编写方法

第二个花指令在`encrypt`函数中，是一个可执行花指令，下面我会具体分析这个花指令以及对应的去除方法

```C
#include <stdio.h>
#include <stdint.h>
#define JUNKCODE __asm{
__asm jmp junk1
__asm __emit 0x12
__asm junk2:
__asm ret
__asm __emit 0x34
__asm junk1:
__asm call junk2
}
void encrypt(uint32_t* v, uint32_t* k) {
uint32_t v0 = v[0], v1 = v[1], sum = 0, i; /* set up */
uint32_t delta = 0x9e3779b9; /* a key schedule constant */
uint32_t k0 = k[0], k1 = k[1], k2 = k[2], k3 = k[3]; /* cache key */
for (i = 0; i < 32; i++) { /* basic cycle start */
JUNKCODE
sum += delta;
v0 += ((v1 << 4) + k0) ^ (v1 + sum) ^ ((v1 >> 5) + k1);
v1 += ((v0 << 4) + k2) ^ (v0 + sum) ^ ((v0 >> 5) + k3);
} /* end cycle */
v[0] = v0; v[1] = v1;
}
int main() {
int a = 1;
uint32_t flag[] = { 1234,5678 };
uint32_t key[] = { 9,9,9,9 };
__asm {
_emit 075h
_emit 2h
_emit 0E9h
_emit 0EDh
}
encrypt(flag, key);
printf("%d,%d", flag[0], flag[1]);
return 0;
}
```

**第一个花指令**

首先ida打开查看`main`函数：

![image-20200616235743442](upload/attach/202311/957393_SQDKQY29ZZU3TWK.jpg)

虽然还是反编译成功了，但是可以看到内容已经完全错误，我们再来看`main`函数的汇编代码，也就是我们加入的第一个花指令

![image-20200616235852756](upload/attach/202311/957393_6RA6G8PPU8RRZNX.jpg)

可以看到这里出现的红色就是我们的第一处花指令，`patch`方法同上，我们主要看第二处花指令。

**第二个花指令**

f5反编译直接报错

![image-20200617000152860](upload/attach/202311/957393_EZ9DRG2RPV49DBK.jpg)

可以发现花指令的混淆作用还是很明显的，那我们继续跟进到花指令的反汇编代码处

![image-20200617000420510](upload/attach/202311/957393_SAAVUSHJMADNW9K.jpg)

这里框出的指令就是我们加入的花指令，逻辑其实特别清晰，就是先跳转到`junk1`，再`call junk2`，`call junk2`的时候会把地址`0x004118D3`压栈，然后进入`junk2`中执行`retn`指令又会把地址`0x004118D3` `pop`到`eip`中，然后接下来程序继续正常执行。

#### **去除方法**

这种连续的可执行花指令的去除方法特别简单，直接整块`nop`掉即可。

但是真正的复杂程序里这种花指令的数量很多，人工`nop`很耗时，同时极容易出错，所以我们真正应该掌握的是自动化的方法，编写脚本匹配花指令模板进行去除。

## 花指令练习

### `jz` `jnz`/`jmp`

> 现在许多的花指令采用生成确定标志位并搭配两个互补的条件跳转指令替代一个强制跳转指令(如`jmp`)以增加汇编难度的方式。

> 其实如果一个程序出现下面互补跳转代码，而且跳转代码前某一个标志位一定是确定值，同时指令下方还出现报错的情况，基本可以断定是花指令（可以作为花指令起始或者存在的标志，是充分不必要条件）。

> 而且运行过程中必然不发生跳转的那一个指令的目标地址或者标号是干扰项。

#### 花指令1

> `E8` --> `call`

形式：

```
xor eax, eax
test eax, eax //产生确定标志位
je LABEL1
jne LABEL2
LABEL2 :
/*干扰项所在*/
LABEL1:
/*正常代码*/
```

含有花指令的代码

```
#include<stdio.h>
#include<windows.h>
int a = 0;
int main()
{
//MessageBox(NULL, "未加花指令", "YQC", 0);
int n, m, result = 1;
printf("请输入n和m:\n");
scanf_s("%d%d", &n, &m);
__asm {
xor eax, eax
test eax, eax
je LABEL1
jne LABEL2
LABEL2 :
_emit 0xe8
LABEL1 :
}
for (int i = 0; i < m; i++)
result *= n;
printf("结果为%d", result);
return 0;
}
```

IDA打开：

![image-20230817112338453](upload/attach/202311/957393_V4DKKS6WVYGHY3T.jpg)

我们分析一下

因为`E8`指令(对应汇编`call`指令)的存在，`E8`之后的汇编指令被错误的识别成了地址

![image-20230817144205603](upload/attach/202311/957393_XTJR8WKTGSUFHX6.jpg)

按D键转换成数据

![image-20230817144238492](upload/attach/202311/957393_4NBYQMPP563FU3E.jpg)

![image-20230817144300841](upload/attach/202311/957393_PJ3R3N7K8XSUGCJ.jpg)

然后把`E8`指令`nop`掉

![image-20230817144351642](upload/attach/202311/957393_TF7HP9KT6Y7NFNJ.jpg)

把整段花指令`nop`掉也可以

![image-20230817145240157](upload/attach/202311/957393_YWZZJ4WQAZQGCUN.jpg)

![image-20230817145248451](upload/attach/202311/957393_D8Z3QQVCA6NKTGW.jpg)

再按C键将错误识别的数据转换成指令

![image-20230817144440327](upload/attach/202311/957393_9KAJ4BXXC5UJMSP.jpg)

最后从函数的结尾到函数的开始重新编译 按P键

`retn` --> `int __cdecl main`

![image-20230817144633355](upload/attach/202311/957393_H68J7R3DZ26F5PS.jpg)

最后Tab或F5

![image-20230817144740904](upload/attach/202311/957393_WS243WW3UBD9WM6.jpg)

#### 花指令2

编写：

```c
#include<stdio.h>
#include<windows.h>
int a = 0;
int main()
{
//MessageBox(NULL, "未加花指令", "YQC", 0);
int n, m, result = 1;
printf("请输入n和m:\n");
scanf_s("%d%d", &n, &m);
__asm {
xor eax, eax
test eax, eax
je LABEL1
jne LABEL2
LABEL2 :
_emit 0x5e //与pop si机器码相同
and eax, ebx
_emit 0x50 //与push ax机器码相同
xor eax, ebx
_emit 0x74 //与汇编助记符 jz 机器码相同
add eax, edx
LABEL1 :
}
for (int i = 0; i < m; i++)
result *= n;
printf("结果为%d", result);
return 0;
}
```

因为`ZF`标志位的存在，`lable2`的花指令部分并不会被执行，会直接跳转到`label1`执行正确的程序流程，所以正确的程序执行并不会出错

分析一下反汇编部分：

![image-20230817094651890](upload/attach/202311/957393_WG5S5N52VHJPZQG.jpg)

可以看到IDA把我们加入的机器码当做了汇编指令来执行。（`0x5e`与`pop si`机器码相同，`0x50`与`push ax`机器码相同，`0x74`与汇编助记符`jz`机器码相同）

```
xor eax, eax //eax寄存器置零
test eax, eax //ZF=1,是确定标志位【test不会将结果放在寄存器上,它只影响ZF的状态，如果EAX == 0,那么ZF = 1】
je LABEL1 //ZF=1时触发跳转到正常指令处LABEL1
jne LABEL2 //ZF=0时触发跳转到干扰项处LABEL2，与上面的指令形成互补跳转
LABEL2 :
/*干扰项所在*/
LABEL1:
/*正常代码*/
```

那么回到当前花指令分析，这个花指令最主要起作用的地方是让IDA将机器码`0x74`翻译成了`jz`指令，致使这个指令的**下一个本该翻译成指令（即`add eax, edx`）的机器码却被翻译成了跳转的地址**，进而出现了后面的指令不正常反汇编的情况。

![image-20230817095410879](upload/attach/202311/957393_SPPH4BAHWPZCC99.jpg)

去花：

只需要把花指令部分`nop`掉即可

![](upload/attach/202311/957393_NXA28HVAJ8FBX2D.jpg)

按D先转换成数据

![image-20230817151540581](upload/attach/202311/957393_AR349JZJTK4NDRV.jpg)

`nop`

![image-20230817151533938](upload/attach/202311/957393_WGHVZ8HMBCHXNEA.jpg)

加上未识别的数据重新反汇编

![image-20230817151618118](upload/attach/202311/957393_8F7VBNETKR76Y7D.jpg)

![image-20230817151632843](upload/attach/202311/957393_R8SWBJCWKF8AX27.jpg)

重新反编译

![image-20230817151653048](upload/attach/202311/957393_BYGZFW5C8QAJY9Y.jpg)

##### IDC去花

先获取`jz short loc_45E421`的字节码如下图，我们只要取`74 03`即可，因为`0x74`代表`jz`助记符，而`add reg16,reg16`的字节码刚好为`0x03`，对应了`add eax,eax`

![image-20230817151924552](upload/attach/202311/957393_ER3ZW49CRDKYUZ6.jpg)

![image-20230817151952590](upload/attach/202311/957393_79SV5SSSB2HY9GA.jpg)

根据`idc`的`API`和语法写出如下`idc脚本`

```
#include<idc.idc>
static main()
{
auto StartVa, StopVa, Size, i;
StartVa=0x00411960;
StopVa=0x00411A27;
Size=StopVa-StartVa;
for (i=0; i<Size; i++){
if (Byte(StartVa)==0x74)
{
if(Byte(StartVa+1)==0x03)
{
PatchByte(StartVa, 0x90);
MakeCode(StartVa);
StartVa++;
Message("Find FakeJmp Opcode!!\n");
continue;
}
}
StartVa++;
}
Message("Clear FakeJmp Opcode Ok\n");
}
```

在IDA里运行

![image-20230817152102477](upload/attach/202311/957393_TASVBYBPC4S8BFK.jpg)

![image-20230817152124837](upload/attach/202311/957393_7ZP7FQYD2PY2H7E.jpg)

![image-20230817152348358](upload/attach/202311/957393_8JVM7SG9EBP534B.jpg)

![image-20230817152410567](upload/attach/202311/957393_NQP6M9S63U6NUWA.jpg)

但不足的是这个脚本没有实现自动反汇编成汇编代码，运行了该脚本之后依旧有一些没有反汇编成正常汇编代码的部分。依旧`需要手动改成数据后再改成代码，最后申明函数`

### `call` `ret`

> 需要先了解一下一些汇编转移指令`call`和`ret`的原理

#### 1、`call` 指令

> 看不懂CS:IP的同学可以先去笔者的<a href="elink@e24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6h3p5H3k6#2)9J5k6i4c8G2M7q4)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3K6t1%4">8086汇编</a>一文中学习，本文不再赘述。

（1）`call + label`

这个指令是先将`call + 标号`的下一条语句的`IP`放入栈中，然后使当前的`IP+16`位位移，相当于

```
push IP
jmp near ptr 标号
```

（2）`call far ptr + label`
