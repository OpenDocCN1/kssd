
### 0x06 双重释放漏洞原理

Double Free漏洞是由于对同一块内存进行二次释放导致的，利用漏洞可以执行任意代码，编译成release示例代码如下：

```c
#include <stdio.h>
#include "windows.h"
int main (int argc, char \*argv[])
{
void \*p1,\*p2,\*p3;
p1 = malloc(100);
printf("Alloc p1：%p\n",p1);
p2 = malloc(100);
printf("Alloc p2：%p\n",p2);
p3 = malloc(100);
printf("Alloc p3：%p\n",p3);
printf("Free p1\n");
free(p1);
printf("Free p3\n");
free(p3);
printf("Free p2\n");
free(p2);
printf("Double Free p2\n"); //二次释放
free(p2);
return 0;
}
```

在二次释放`p2`的时候就会发生程序崩溃，但是并不是每次出现Double Free都会发生崩溃，要有堆块合并的动作发生才会发生崩溃

```c
#include <stdio.h>
#include "windows.h"
int main (int argc, char \*argv[])
{
void \*p1,\*p2,\*p3;
p1 = malloc(100);
printf("Alloc p1：%p\n",p1);
p2 = malloc(100);
printf("Alloc p2：%p\n",p2);
p3 = malloc(100);
printf("Alloc p3：%p\n",p3);
printf("Free p2\n");
free(p2);
printf("Double Free p2\n");
free(p2);
printf("Free p1\n");
free(p1);
printf("Free p3\n");
free(p3);
return 0;
}
```

![](upload/attach/201907/823237\_NDJY7Y3DN2JNGJR.png)

![](upload/attach/201907/823237\_ZWRBMXBMCJF5MTN.png)

双重释放原理图

在释放过程中，邻近的已经释放的堆块存在合并操作，这会改变原有堆头信息，之后再对其地址引用释放就会发生访问异常。

### 0x07 释放后重引用漏洞原理

通过以下代码理解UAF漏洞原理：

```c
#include <stdio.h>
#define size 32
int main(int argc, char \*\*argv) {
char \*buf1;
char \*buf2;
buf1 = (char \*) malloc(size);
printf("buf1：0x%p\n", buf1);
free(buf1);
// 分配 buf2 去“占坑”buf1 的内存位置
buf2 = (char \*) malloc(size);
printf("buf2：0x%p\n\n", buf2);
// 对buf2进行内存清零
memset(buf2, 0, size);
printf("buf2：%d\n", \*buf2);
// 重引用已释放的buf1指针，但却导致buf2值被篡改
printf("==== Use After Free ===\n");
strncpy(buf1, "hack", 5);
printf("buf2：%s\n\n", buf2);
free(buf2);
}
```

![](upload/attach/201907/823237\_NKBGPT28K3B4Y82.png)

`buf2`“占坑”了`buf1`的内存位置，经过UAF后，`buf2`被成功篡改了

程序通过分配和`buf1`大小相同的堆块`buf2`实现占坑，似的`buf2`分配到已经释放的`buf1`内存位置，但由于`buf1`指针依然有效，并且指向的内存数据是不可预测的，可能被堆管理器回收，也可能被其他数据占用填充，`buf1`指针称为悬挂指针，借助悬挂指针`buf1`将内存赋值为`hack`，导致`buf2`也被篡改为`hack`。

如果原有的漏洞程序引用到悬挂指针指向的数据用于执行指令，就会导致任意代码执行。

在通常的浏览器UAF漏洞中，都是某个C++对象被释放后重引用，假设程序存在UAF的漏洞，有个悬挂指针指向`test`对象，要实现漏洞利用，通过占坑方式覆盖`test`对象的虚表指针，虚表指针指向虚函数存放地址，现在让其指向恶意构造的`shellcode`，当程序再次引用到`test`对象就会导致任意代码执行。

![](upload/attach/201907/823237\_3PMDQ7RBF3SWTMT.png)

UAF漏洞利用原理图

### 0x08 数组越界访问漏洞

先区分一下数组越界漏洞和溢出漏洞：数组越界访问包含读写类型，溢出属于数据写入；部分溢出漏洞本质确实就是数组越界漏洞。

数组越界就像是倒水的时候倒错了杯子，溢出就像是水从杯子里溢出来。

下面代码为例分析数组越界访问漏洞：

```c
#include "stdio.h"
int main(){
int index;
int array[3] = {0x11, 0x22, 0x33};
printf("输入数组索引下标：");
scanf("%d", &index);
printf("输出数组元素：array[%d] = 0x%x\n", index, array[index]); //数组越界读操作
//array[index] = 1 ; //数组越界写操作
return 0;
}
```

![](upload/attach/201907/823237\_6QPEMQK29DJJKUC.png)

执行生成的程序，然后分别输入12345，输出结果如上，当输入的数组下标分别是12的时候，会得到正常数值，但是从索引3开始就超出了原来的数组`array`的范围，比如输入5，就会数组越界访问`array`数组，导致读取不在程序控制范围内的数值。

使用`ollydbg`调试发现`array[5]`就是从`array`开始的第六个数据`0x4012A9`，已经读取到了`array`之外的数据，如果越界访问距离过大，就会访问到不可访问的内存空间，导致程序崩溃。

### 0x09 类型混淆漏洞原理

类型混淆漏洞（Type Confusion）一般是将数据类型A当做数据类型B来解析引用，这就可能导致非法访问数据从而执行任意代码，比如将Unit转成了String，将类对象转成数据结构。

类型混淆漏洞是现在浏览器漏洞挖掘的主流漏洞，这类漏洞在`java`，`js`等弱类型语言中非常常见。

下面的代码，A类被混淆成B类，就可能导致私有域被外部访问到：

```c++
class A {
private int value;
};
class B {
public int value;
};
B attack = AcastB(var); //将A类型混淆转成B类型
attack.value = 1; //导致可以访问私有域
```

以IE/Edge类型混淆漏洞（CVE-2017-0037）为例讲解，漏洞原因是函数处理时，没有对对象类型进行严格检查，导致类型混淆。

PoC如下：在PoC中定义了一个`table`，标签中定义了表id为`th1`，在`boom()`中引用，然后是`setInterval`设定事件。

![](upload/attach/201907/823237\_UN8JVAMG8AV4YJH.png)

运行PoC，用`Windbg`附加并加载运行出现崩溃

![](upload/attach/201907/823237\_WU27V2Z6WXKG7N5.png)

从崩溃点可以看到`eax`作为指针，引用了一个无效地址，导致崩溃，而上一条指令是一个`call`，这个无效的返回值来自这个`call`，在这个`call`处下断点，`ecx`作为参数，存放的对象是一个`Layout::FlowItem::`vftable`虚表

![](upload/attach/201907/823237\_HK3RZNC3NMXRJFD.png)

这里读取虚表中+4的值，为0时`this`指针赋值`v1`，随后`v1+16`后返回，因此，`Layout::FlowItem::`vftable`所属指针的这个情况是正常的，函数会正常返回进入后续处理逻辑

![](upload/attach/201907/823237\_9Q29F845WBECJ39.png)

让程序继续运行，会再次调用该函数，此时`ecx`并不是一个虚表对象，而是一个`int Array`对象，这里我们可以通过条件断点来跟踪两个对象的创建过程，重点关注两个对象创建的函数，一个是`FlowItem::`vftable`对应的虚表对象，另一个是引发崩溃的`int Array`对象。这两个函数的返回值，也就是`eax`寄存器中存放的指向这两个创建对象的指针。

![](upload/attach/201907/823237\_U7KYVVNCMDEFWHV.png)

通过跟踪可以看到第一次调用`Readable`函数时`ecx`是一个正常的`FlowItem`对象，而第二次调用的时候`ecx`是一个`int Array Object`。`Layout::Patchable >::Readable`函数是处理虚表对象的函数，由于`boom()`函数中引用`th1.align`导致`Readable`函数得到第二次引用，由于没有进行对象属性检查，导致第二次调用时将`table`对象传入，最终发生类型混淆崩溃。

### 0x10 竞争条件漏洞原理

竞争条件（Race Condition）是由于多个线程/对象/进程同时操作同一资源，导致系统执行违背原有逻辑设定的行为，这类漏洞在`linux`，内核层面非常多见，在`windows`和`web`层面也存在。

互斥锁的出现就是为了解决此类漏洞问题，保证某一对象在特定资源访问时，其他对象不能操作此资源。

```
竞争条件”发生在多个线程同时访问同一个共享代码、变量、文件等没有进行锁操作或者同步操作的场景中。 ——Wikipedia-computer_science
```

比如如下代码：

```python
# -*-coding:utf-8-*- 
import threading
COUNT = 0
def Run(threads_name):
global COUNT
read_value = COUNT
print "COUNT in Thread-%s is %d" % (str(threads_name), read_value)
COUNT = read_value + 1
def main():
threads = []
for j in range(10):
t = threading.Thread(target=Run,args=(j,))
threads.append(t)
t.start()
for i in range(len(threads)):
threads[i].join()
print("Finally, The COUNT is %d" % (COUNT,))
if __name__ == '__main__':
main()
```

执行结果如下：

![](upload/attach/201907/823237\_9WYEPCUCWAAZNAT.png)

按照我们的预想，结果应该都是10，但是发现结果可能存在非预期解，原因就在于我们没有对变量`COUNT`做同步制约，导致可能`Thread-7`在读`COUNT`,还没来得及更改`COUNT`,Thread-8抢夺资源，也来读`COUNT`,并且将`COUNT`修改为它读的结果+1，由此出现非预期。

### 参考信息

《漏洞战争》

<mark class="encrypted">c0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3u0T1x3$3q4S2k6U0y4X3y4e0R3&6x3l9`.`.</mark>

<mark class="encrypted">12aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4j5H3N6#2)9J5k6i4c8G2M7q4)9J5c8U0t1H3x3e0S2Q4x3V1j5H3z5q4)9J5c8U0p5$3i4K6u0r3i4@1f1$3i4K6W2p5i4@1p5I4i4@1f1@1i4@1u0n7i4@1t1$3i4@1f1%4i4@1q4n7i4K6W2q4i4@1f1@1i4@1u0m8i4K6R3&6i4K6u0r3</mark>

个人`github`博客地址：

<mark class="encrypted">ab2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2u0W2k6i4c8D9k6h3q4Y4N6h3g2Q4x3V1j5H3P5r3u0A6M7X3c8Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8R3`.`.</mark>

个人安全研究公众号平台：

![](upload/attach/201907/823237\_P95QDFE7ZF9BXHK.jpg)

目录

- [0x01栈溢出漏洞原理](https://bbs.kanxue.com/#0x01栈溢出漏洞原理)
- 0x02 堆溢出漏洞原理
- 0x03 堆调试技巧
- [堆尾检查](https://bbs.kanxue.com/#堆尾检查)
- [页堆](https://bbs.kanxue.com/#页堆)
- [0x04整数溢出漏洞原理](https://bbs.kanxue.com/#0x04整数溢出漏洞原理)
- [基于栈的整数溢出](https://bbs.kanxue.com/#基于栈的整数溢出)
- [基于堆的整数溢出](https://bbs.kanxue.com/#基于堆的整数溢出)
- 0x05 格式化字符串漏洞原理
- 0x06 双重释放漏洞原理
- [0x07释放后重引用漏洞原理](https://bbs.kanxue.com/#0x07释放后重引用漏洞原理)
- 0x08 数组越界访问漏洞
- [0x09类型混淆漏洞原理](https://bbs.kanxue.com/#0x09类型混淆漏洞原理)
- [0x10竞争条件漏洞原理](https://bbs.kanxue.com/#0x10竞争条件漏洞原理)
- [参考信息](https://bbs.kanxue.com/#参考信息)

本部分将对常见的二进制漏洞做系统分析，方便在漏洞挖掘过程中定位识别是什么类型漏洞，工欲善其事，必先利其器。

### 0x01 栈溢出漏洞原理

栈溢出漏洞属于缓冲区漏洞的一种，实例如下：

```
#include <stdio.h>
#include <string.h>
int main()
{
    char *str = "AAAAAAAAAAAAAAAAAAAAAAAA";
    vulnfun(str);
    return;
}
int vulnfun(char *str)
{
    char stack[10];
    strcpy(stack,str);        // 这里造成溢出！    
}
```

编译后使用`windbg`运行

![](https://bbs.kanxue.com/upload/attach/201907/823237_F4PWZT2F7BS7UZ2.png)

直接运行到了地址`0x41414141`，这个就是字符串`AAAA`，就是变量`str`里面的字符串通过`strcpy`拷贝到栈空间时，没有对字符串长度做限制，导致了栈溢出，最后覆盖到了返回地址，造成程序崩溃。

溢出后的栈空间布局如下：

![](https://bbs.kanxue.com/upload/attach/201907/823237_FXWFF95ZPFEECE4.png)

栈溢出原理图

### 0x02 堆溢出漏洞原理

使用以下代码演示堆溢出漏洞

```
#include <windows.h>
#include <stdio.h>
int main ( )
{
    HANDLE hHeap;
    char *heap;
    char str[] = "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA";
    hHeap = HeapCreate(HEAP_GENERATE_EXCEPTIONS, 0x1000, 0xffff);
    getchar();    // 用于暂停程序，便于调试器加载
    heap = HeapAlloc(hHeap, 0, 0x10);
    printf("heap addr:0x%08x\n",heap);
    strcpy(heap,str);    // 导致堆溢出
    HeapFree(hHeap, 0, heap);    // 触发崩溃
    HeapDestroy(hHeap);
    return 0;
}
```

由于调试堆和常态堆的结构不同，在演示代码中加入`getchar`函数，用于暂停进程，方便运行`heapoverflowexe`后用调试器附加进程。`debug`版本和`Release`版本实际运行的进程中各个内存结构和分配过程也不同，因此测试的时候应该编译成`release`版本。

运行程序，使用`windbg`附加调试（一定要附加调试），`g`运行后程序崩溃

![](https://bbs.kanxue.com/upload/attach/201907/823237_FYUAWTW38PV3MWA.png)

上面的`ecx`已经被`AAAA`字符串覆盖掉了，最后在引用该地址的时候导致崩溃，通过前面的栈回溯定位到了`main`函数入口，找到复制字符串的函数下断点

![](https://bbs.kanxue.com/upload/attach/201907/823237_TRUUHRA69MRMA7B.png)
