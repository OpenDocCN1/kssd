
```
.implements Landroid/view/View$OnClickListener;
```

注解描述，如果一个类中使用注解,会用.annotation定义:其格式如下:

```
.annotation [注解的属性] <注解类名>
[注解字段=值]
...
.end
```

示例代码

```
.annotation runtime Ljava/lang/Deprecated;
.end annotation
```

字段描述，Smali中使用.field描述字段，我们知道java中分为静态字段(类属性)和普通字段(实例属性)，它们在smali中的表示如下:

普通字段:
```
#instance fields
.field <访问权限修饰符> [非权限修饰符] <字段名>:<字段类型>
```

访问权限修饰符相比各位已经非常熟了,而此处非权限修饰符则可是final,volidate,transient.

举例说明:
```
.field private TAG:Ljava/lang/String;
```

静态字段，静态字段知识在普通字段的的定义中添加了static,其格式如下:
```
#static fields
.field <访问权限> static [修饰词] <字段名>:<字段类型>
```

需要注意:smali文件还为静态字段,普通字段分别添加#static field和#instan filed注释，举例说明：
```
# static fields
.field private static final pi:F = 3.14f
.class public Lcom/smali/test/Test;
.super Ljava/lang/Object;
.source "Test.java"
.method public constructor <init>()V
.locals 2
.line 8
invoke-direct {p0}, Ljava/lang/Object;-><init>()V
const-string v0, "Smali"
const-string v1, "This is constructor method!"
.line 9
invoke-static {v0, v1}, Landroid/util/Log;->i(Ljava/lang/String;Ljava/lang/String;)I
return-void
.end method
.method static add(II)I
.locals 0
.annotation runtime Ljava/lang/Deprecated;
.end annotation
add-int/2addr p0, p1
add-int/lit8 p0, p0, 0x1
return p0
.end method
```

### 3.1.5 函数调用

函数是整个程序的基石，开始讲Smali函数调用之前，首先介绍Smali汇编语言中的函数构成，Smali中函数定义格式为：
```
.method public/private [static] method()<返回类型>
<.locals>
[.parameter]
[.prologue]
[.line]
<代码逻辑>
.end method
```

使用`.method`标识函数由此开始，`.end method`标识函数到底结束。方法名前使用修饰符对方法进行标识。例如直接方法用private修饰的，虚方法用public或protected。`.locals`标识方法内使用的局部变量的个数。`.parameter`标识了该方法中的参数。`.prologue`标识方法中代码的开始处。

函数示例代码如下
```
# direct methods
.method public constructor ()V
.registers 2
.prologue
.line 8
invoke-direct {p0}, Landroid/app/Activity;->()V
.line 10
const-string v0, "MainActivity"
iput-object v0, p0, Lcom/test/demo/MainActivity;->TAG:Ljava/lang/String;
.line 13
const/4 v0, 0x0
iput-boolean v0, p0, Lcom/test/demo/MainActivity;->running:Z
return-void
.end method
```

Smali汇编中的函数和java语言中的函数一样存在访问控制，根据访问级别不同可分为direct和virtual两类。修饰符direct声明的函数是等同于java代码中的private类型函数，修饰符virtual的函数等同于java代码中protected和public类型函数。按方法类型可以分为三类static、interface和super，即静态方法、接口方法和父方法。

根据函数的访问权限和方法类型进行区分，调用函数时有invoke-direct，invoke-virtual，invoke-static、`invoke-super`以及invoke-interface等几种不同的指令,具体调用格式如下：
```
invoke-指令类型 {参数1, 参数2,...}, L类名;->方法名
```

直接方法调用，即private方法调用。参考示例代码如下：
```
invoke-direct {p0, v0, v1}, Lcom/smali/test/Test;->start(II)I
```

上述代码中start方法是定义在Test类中的一个private函数，可以通过invoke-direct调用。

虚方法调用，即用于调用protected或public函数。参考示例代码如下：
```
invoke-virtual {p1}, Lcom/smali/test/Test;->start()V
```

上述代码中start()就是定义在Test中的一个public函数，可以通过invoke-virtual调用。

静态方法调用。参考示例代码如下：
```
invoke-static {}, Lcom/smali/test/Test;->test()V
```

调用父类方法用的指令invoke-super，一般用于调用onCreate、onDestroy等方法。参考示例代码如下：
```
invoke-super {p0, p1}, Landroidx/appcompat/app/AppCompatActivity;->onCreate(Landroid/os/Bundle;)V
```

通过invoke-interface指令调用接口类方法，参考示例代码如下：
```
invoke-interface {v0}, Ljava/util/List;->size()I
```

函数返回值

在Smali代码中根据函数返回类型可以分为三类：空值、基本数据类型、对象类型。因为返回值类型不同用到的返回指令也各不相同。具体如下表：

| 指令 | 指令含义 |
| ---------------- | -------------------------------------------------- |
| return-void | 表示函数从一个void方法返回 |
| return vA | 表示函数返回一个32位非对象类型的值 |
| return-wide vA | 表示函数返回一个64位非对象类型的值. |
| return-object vA | 表示函数返回一个对象类型的值.返回值为8位的寄存器vA |

在Java代码中调用函数并返回函数执行结果只需一条语句便可完成，而在Smali代码中调用函数和返回函数结果需要分开实现。如果调用的函数返回结果为基本数据类型，需要使用move-result或move-result-wide指令将结果移动到指定的寄存器；如果调用的函数返回结果为对象则需要使用move-result-object指令将结果对象移动到指定的寄存器。

返回空值
```
.method public constructor <init>()V
.locals 0
invoke-direct {p0}, Ljava/lang/Object;-><init>()V
return-void
.end method
```

返回基本数据类型
```
.method public static start()I
.locals 2
const/4 v0, 0x4
const/4 v1, 0x5
invoke-static {v0, v1}, Lcom/smali/test/Test;->test(II)I
move-result v0
return v0
.end method
```

返回对象数据类型
```
.method public test()Ljava/lang/String;
.locals 2
invoke-virtual {p0, v0}, Lcom/smali/test/Test;->test()
Ljava/lang/String;
move-result-object v0
return-object v0
.end method
```

## 3.2 ARM汇编基础

ARM 是 Advanced RISC Machine 的缩写，可以理解为一种处理器的架构，还可以将它作为一套完整的处理器指令集。RISC(Reduced Instruction Set Computing) 精简指令集计算机：一种执行较少类型计算机指令的微处理器。

ARM 处理器是典型的 RISC 处理器，因为它们执行的是加载/存储体系结构。只有加载和存储指令才能访问内存。数据处理指令只操作寄存器的内容。目前市面上绝大多数的手机CPU都是基于ARM架构的，但是也有少数采用了英特尔X86架构的CPU。

### 3.2.1 寄存器

ARM处理器共有37个32位的寄存器，其中31个为通用寄存器，6个为状态寄存器。

- 31个通用寄存器，包括1个程序计数器 （PC）和30 个通用寄存器。
- 6个状态寄存器，包括1个CPSR寄存器和5个SPSR寄存器。虽然这些寄存器都是32位的，但实际只使用了其中的12位。

虽然ARM处理器可用寄存器有37个，但是这些寄存器是无法同时被访问的，具体的哪些寄存器可以访问是由处理器的工作状态和运行模式决定。不同的处理器模式中使用不同的寄存器组。如图所示，在任何处理器模式下通用寄存器(R0~R15)、1或2个状态寄存器都是可以访问的。图中每列展示寄存器的即是该用户模式下所有可见的寄存器。

![图片描述](upload/attach/202307/452101_D6BRM9BH537FVRD.png)

通用寄存器包括R0~R15，可以分为三类：

- 不分组寄存器R0~R7
- 分组寄存器R8~R4
- 程序计数器R15(PC)

不分组寄存器R0-R7，不分组寄存器在所有的处理器模式中均可以访问，是真正的通用寄存器。但有一点需要注意在中断或异常处理进行模式转换时可能会造成寄存器中数据的损坏。

分组寄存器R8~R14，分组寄存器的访问与当前处理器模式相关，如果想不依赖处理器模式访问特定的寄存器则需要使用规定的寄存器名字。

寄存器R13通常用做堆栈指针，因此有时也被称作SP。程序进行异常运行模式如函数调用R13寄存器会在指向异常模式分配的堆栈，异常处理程序可将当前运行环境中其它寄存器地的值存储到堆栈中。当函数执行结束将堆栈中的值重新恢复到原来的寄存器中，这样异常处理前的运行环境恢复继续执行后续流程。

寄存器R14通常用做子程序连接寄存器，就是通常说的LR寄存器。当执行程序跳转指令BL或BLX时，程序计数器R15中的子程序返回地址将被拷贝到R14。待子程序执行结束返回时，R14中存储的返回地址会恢复到R15中。

执行指令如下
```
MOV PC，LR
BX LR
```

子程序入口，执行如下指令把寄存器R14存到堆栈
```
STMFD SP!,{<rsgisters>,LR}
```

使用如下指令把寄存器将堆栈中数据进行恢复
```
LDMFD SP!，{<registers>,PC}
```

程序计数器R15，R15寄存器在分类上属于通用寄存器，但默认其做为程序计数器使用，不能做为通用寄存器使用，因此寄存器R15也被叫做PC寄存器。如果强行将其做为通用寄存器使用则可能会导致程序出现不可预知的行为。

状态寄存器，程序状态寄存器有包含1个当前程序状态寄存器CPSR和5个备份的程序状态寄存器SPSR。当前程序状态器CPSR在任何处理器模式下均可被访问，CPSR用于标记当前程序的运算结果、处理器状态、当前运行模式等。

SPSR寄存器用来备份当前的程序状态寄存器，当程序触发异常中断时，可将CPSR的值存放到SPSR。异常处理程序执行结束返回时，再将SPSR中存放地的当前程序状态值恢复至CPSR。CPSR和SPSR格式是相同，具体如下图所示.

![图片描述](upload/attach/202307/452101_QRAHW994EM98TJM.png)

条件标志位，N(Negative)、Z(Zero)、C(Carry)和V(oVerflow)被通称为条件标志位。条件标志位的值会根据程序中的算数或逻辑指令的执行结果值决定，程序可以根据条件标志位中的值决定程序的执行流程，常见的判断逻辑如if条件语句和switch语句。具体用处如下图所示：

![图片描述](upload/attach/202307/452101_7A4MDQT4JAFAXM3.png)

控制位，CSPR的低8位统称为控制位，当异常发生这些值将会发生变化，也可在特权模式下对这些标志位进行修改。

中断禁止位I、F和T标志位具体功能如下图所示：

![图片描述](upload/attach/202307/452101_4M275GH8MQGSHDW.png)

模式控制位M[4:0]用于标识了处理器的工作模式，同时也标记 了当前处理器模式下可访问的寄存器。具体含义工作模式下可访问的寄存器对照关系如表所示：

![图片描述](upload/attach/202307/452101_ZASPE2X9E347VNF.png)

### 3.2.2 基础指令

本节将简要介绍ARM的指令集和它的基本用法。作为汇编语言的基本单位，了解指令的用法，指令间的如何关联以及将指令进行组合能实现什么功能对于学习汇编语言是至关重要的。

ARM汇编由ARM指令组成。ARM指令通常跟一到两个操作数，具体的语法格式如下所示：
