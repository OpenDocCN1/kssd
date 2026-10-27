

# [原创]指令壳开源

## 前言

&emsp;&emsp;在把壳子开源之前，我会先对VMProtect1.70.4这个版本做一个简单的分析。在这几天分析过程中，我感受到了VMProtect的威力，并得出一个结论：分析VMProtect非常耗时间，如果没有做好与之长期斗争的准备，很难有实用的成果。另外，由于我第一次分析VMProtect，有分析不对的地方或者术语用得不恰当的地方，希望老萌萌们能指出来，我立即修改，不能误人子弟。分析的样本有三份，我放到了附件里面。

&emsp;&emsp;壳子我是去年写的，当时刚刚看了《加密与解密》第四版的第21章，我估摸了一下，自己可以写出来，然后就写出来了。在写之前，我记得当时还在bilibili观看了哈工大姜守旭教授教授的编译原理这门课程的视频，了解了大概就开干，看视频这个操作，对我写指令壳起了一种壮胆的效果。现在要开源，我又熟悉了下整个项目的流程。代码写得有点乱，我会画一张加壳时的流程图做一个直观的说明，和其他一些要点说明，以减少读代码时遇到的困惑。如果遇到调试或者编译问题，可以留言，我看到会及时回复。指令壳源码我会上传到附件。

## 分析vmp1.70.4

### 01_速度最快

写一个简单程序来测试：(ps:这个程序在vmp样本一文件夹)

```c++
#include <Windows.h>
#include <iostream>
void \_\_declspec(naked) test\_vmp(int a, int b)
{
/\*\_\_asm {
mov eax,dword ptr[esp+4]
mov ecx,dword ptr[esp+8]
add eax,ecx
ret
}\*/
\_\_asm {
xor eax,ebx
xor eax, ebx
ret
}
}
int main()
{
test\_vmp(1, 2);
printf("%d\n", x);
system("pause");
return 0;
}
```

在OD中打开，函数在0x401080这个位置。

![clipboard\_31\_](./upload/attach/202105/877885\_Y6TGA89TMJ2MHFT.png)

打开vmp，开始加壳：

![clipboard\_41\_](./upload/attach/202105/877885\_2QNNBPDDHYK29JR.png)

下拉列表选择最快速度，其他不选择。

![clipboard\_7\_](./upload/attach/202105/877885\_EFSRMQEGSG24NZN.png)

程序加好壳子后，用OD打开，开始分析：

![clipboard\_28\_](./upload/attach/202105/877885\_RM82ECECJ8UFSNF.png)

0x401080这儿已经变为jmp了，跳到`.vmp0`这节里面：

![clipboard\_2\_](./upload/attach/202105/877885\_27SKYSVHY6QSY4Q.png)

```javascript
//入口
00401080| jmp debug\_test2.vmp.4A77D2 | Debug\_test2.cpp:5
004A77D2| push 4A781400 |
004A77D7| call debug\_test2.vmp.4A6B9F |
004A6B9F| jmp debug\_test2.vmp.4A703A |
004A703A| push esi | esi:\_mainCRTStartup
004A703B| jmp debug\_test2.vmp.4A52E8 |
004A52E8| pushfd |
004A52E9| push D3AC6D6A |
004A52EE| mov byte ptr ss:[esp+4],8F |
004A52F3| pushfd |
004A52F4| pop dword ptr ss:[esp+4] | [esp+4]:\_\_\_use\_sse2\_mathfcns+4A78
004A52F8| jmp debug\_test2.vmp.4A64C1 |
004A64C1| call debug\_test2.vmp.4A5FE9 |
004A5FE9| pushad |
004A5FEA| mov dword ptr ss:[esp+24],ebp | [esp+24]:\_mainCRTStartup
004A5FEE| push esp |
004A5FEF| pushfd |
004A5FF0| push 72B57CE7 |
004A5FF5| pushfd |
004A5FF6| mov dword ptr ss:[esp+30],eax | eax:\_\_\_use\_sse2\_mathfcns+2D1
004A5FFA| mov byte ptr ss:[esp],cl |
004A5FFD| call debug\_test2.vmp.4A701F |
004A701F| call debug\_test2.vmp.4A5A25 |
004A5A25| jmp debug\_test2.vmp.4A6BC5 |
004A6BC5| mov dword ptr ss:[esp+34],edx | [esp+34]:\_\_\_use\_sse2\_mathfcns+2D1
004A6BC9| call debug\_test2.vmp.4A5C3D |
004A5C3D| call debug\_test2.vmp.4A6681 |
004A6681| mov dword ptr ss:[esp+38],edx | [esp+38]:\_\_\_use\_sse2\_mathfcns+2D1
004A6685| push dword ptr ss:[esp+8] | [esp+8]:\_\_\_use\_sse2\_mathfcns+42C0
004A6689| mov dword ptr ss:[esp+38],ecx | [esp+38]:\_\_\_use\_sse2\_mathfcns+2D1, ecx:\_\_\_use\_sse2\_mathfcns+2D1
004A668D| push E7FE4EC3 |
004A6692| mov byte ptr ss:[esp],dh |
004A6695| lea esp,dword ptr ss:[esp+3C] |
004A6699| jmp debug\_test2.vmp.4A632E |
004A632E| btr si,6 |
004A6333| push edi |
004A6334| push 20AD5139 |
004A6339| xchg esi,ecx | esi:\_mainCRTStartup, ecx:\_\_\_use\_sse2\_mathfcns+2D1
004A633B| call debug\_test2.vmp.4A6129 |
004A6129| mov dword ptr ss:[esp+4],ebx |
004A612D| movsx esi,cl | esi:\_\_\_use\_sse2\_mathfcns+2D1
004A6130| pop ecx | ecx:"U嬱Q梓\x0E"
004A6131| btc di,cx |
004A6135| pushad |
004A6136| mov dword ptr ss:[esp+1C],0 |
004A613E| stc |
004A613F| jmp debug\_test2.vmp.4A70CF
```

以上操作是把寄存器压到栈里，执行完后如下图：

![clipboard\_38\_](./upload/attach/202105/877885\_X7VZWPNMA7CHQKA.png)

--------------------------------------------------------------------------------

寄存器压栈完成后，会先计算出调度表的地址。

`[esp+48]`这个值就是入口`push 4A781400`这个值，经过计算后得到调度表的地址。调度表地址存放在`esi`寄存器里。

```javascript
004A70CF| mov esi,dword ptr ss:[esp+48] |
004A70D3| btr bp,sp |
004A70D7| jmp debug\_test2.vmp.4A53AF |
004A53AF| movzx bp,bl |
004A53B3| btc bp,si |
004A53B7| rol esi,18 |
004A53BA| push 124E6496 |
004A53BF| inc esi |
004A53C0| jmp debug\_test2.vmp.4A6C85 |
```

如下，内存5里显示的就是经过加密的调度表：

![clipboard](./upload/attach/202105/877885\_VDHXGY5B9CQ23D8.png)

然后按F7，到下面那个位置，箭头指向的三步，`ebp`指向的是真实堆栈，`edi`就是虚拟机的上下文环境（`VMContext`）。`edi`指向的堆栈空间，最大的作用就是存放寄存器。

![clipboard\_5\_](./upload/attach/202105/877885\_A6XBY8JJM9G55HB.png)

![clipboard\_25\_](./upload/attach/202105/877885\_SUY5QWMWQGDBR2Z.png)

在接下来的步骤中，通过在`esi`指向的调度表中取值，然后把`ebp`所指向的寄存器的值，挨个放到`edi`的虚拟环境中。

![clipboard\_39\_](./upload/attach/202105/877885\_HAP5NRTNDKK6GED.png)

至此，虚拟机的环境构建完成，准备工作已经做好。

----------------------------------------------------------------------------------------------------------

接下来进入正题，程序会通过`edi`寄存器取`ebx`的值，取两次，第一次的值压入`[ebp]`，第二次压入`[ebp+4]`，然后进入一个`handler`块进行运算。

这就是那个`handler`块：

```javascript
004A53F7 | rol ah,6 |
004A53FA | sbb edx,esp |
004A53FC | mov eax,dword ptr ss:[ebp] |
004A53FF | jmp debug\_test2.vmp.4A5BC4
004A5BC4 | cmc |
004A5BC5 | mov edx,dword ptr ss:[ebp+4] |
004A5BC8 | push 25584F9E |
004A5BCD | pushad |
004A5BCE | clc |
004A5BCF | bt cx,C |
004A5BD4 | not eax |
004A5BD6 | pushad |
004A5BD7 | bt cx,bx |
004A5BDB | call debug\_test2.vmp.4A5D63 |
004A5D63 | cmc |
004A5D64 | not edx |
004A5D66 | cmp edi,F7A6A772 |
004A5D6C | stc |
004A5D6D | stc |
004A5D6E | stc |
004A5D6F | and eax,edx |
004A5D71 | jmp debug\_test2.vmp.4A5A3F |
004A5A3F | jmp debug\_test2.vmp.4A6E72 |
004A6E73 | pushfd |
004A6E74 | mov dword ptr ss:[ebp+4],eax |
004A6E77 | jmp debug\_test2.vmp.4A564D |
----------------------------------------------------------------------------------------------------------
//化简之后
004A53FC | mov eax,dword ptr ss:[ebp] |
004A5BC5 | mov edx,dword ptr ss:[ebp+4] |
004A5BD4 | not eax |
004A5D64 | not edx |
004A5D6F | and eax,edx |
004A6E74 | mov dword ptr ss:[ebp+4],eax |
004A6E77 | jmp debug\_test2.vmp.4A564D |
```

可以把上面的`handler`块命名为`Handler_NOT_AND`。

---------------------------------------------------------------------------------

那么，可以把以上的运算过程可以表示成这样：`not(ebx) and not(ebx)`

--------------------------------------------------------------------------------

执行`xor eax,ebx`这条指令的时候，虚拟机会多次调用`Handler_NOT_AND`块，整个流程可以记录为如下形式：

`T = not(not(ebx) and not(ebx)) and not(not(eax) and not(eax))`；即为：`T = ebx and eax`；

`S = not(eax) and not(ebx)`；

最终结果：`ret = not(T) and not(S)`。

通过写程序来验证，与虚拟机算出来的0x690035一致，说明流程记录没有问题。那么，`xor eax,ebx`可以用这个表达式来表示：`eax = not(ebx and eax) and not(not(eax) and not(ebx))`。

![clipboard\_43\_](./upload/attach/202105/877885\_EJKYPBTZT4BJNSJ.png)

执行完`xor eax,ebx`之后，`eax`寄存器的位置在`edi`中的会变：

![clipboard\_35\_](./upload/attach/202105/877885\_G64HBEFT29X4QBH.png)

--------------------------------------------------------------------------------

下一个`xor eax,ebx`和上面的操作是一样的，这个操作完了，`eax=0x004A3035`。

`eax`寄存器的位置在`edi`中又变了：

![clipboard\_42\_](./upload/attach/202105/877885\_ZUEQVYTKEBJ6WWA.png)

按F7单步跟，(...省略不重要的部分)，接着，程序把`edi`中所保存的寄存器，再吐出来给`ebp`所指向的堆栈空间，然后`ebp`赋值给`esp`，最后再`pop`到真实寄存器，退出虚拟机。

![clipboard\_24\_](./upload/attach/202105/877885\_Y62CM2M7ZXE758Y.png)

退出虚拟机：

![clipboard\_33\_](./upload/attach/202105/877885\_35R7XNM695NMMFX.png)

--------------------------------------------------------------------------------

### 02_开启检测调试器

测试程序和上面一样。(ps:这个程序在vmp样本二文件夹)

![clipboard\_3\_](./upload/attach/202105/877885\_QNEG2QNHZ7T6XJS.png)

下拉列表选择最快速度，再把调试器勾选上，其他不选择。

![clipboard\_19\_](./upload/attach/202105/877885\_T4AFSRDRX44GV8R.png)

加壳完成后，打开CFF来查看，程序会新增一节`.vmp1`，程序入口也在这节里面。

![clipboard\_11\_](./upload/attach/202105/877885\_XSQMBPS8S2KYRH9.png)

![clipboard\_30\_](./upload/attach/202105/877885\_QG6AT436WS5P3HV.png)

此外，还构建了一个TLS表：(但在这儿作用似乎不大）

![clipboard\_27\_](./upload/attach/202105/877885\_3WU3X9ZFSHQ7FD2.png)

程序执行到入口后，按F7单步跟，步骤和上面“01_速度最快”分析时相差无几，会先构建虚拟机环境。

虚拟环境构建完成后，接着按F7单步跟，我的想法是，很快就能找到一些反调试的线索，但是跟了几个小时，发现不对劲了，和上面“01_速度最快”分析时用手工跟踪，完全不在一个数量级的。天气又大，整个人木在那里。后来，想到在退出虚拟机那个地方下断，方法就是搜索`.vmp1`这节的`ret`或者`ret xx`，最终找到两个地方，一个是调度器`ret xx`，这个不管，另一个就是下图所给出的，在`ret 0x40`处下断。按F9程序跑起来后，会断在这里，能看到右边寄存器窗口的字符串。这个位置，可以作为过掉检测调试器的突破口。

![clipboard\_14\_](./upload/attach/202105/877885\_YWNZ9FRXKJAVZQ9.png)

当然，最好的办法应该是在一些能检测出调试器的API函数下断。

vmp1.70.4这个版本，开启调试器检测后，程序会依次调用以下的API来检测是否有调试器存在：

