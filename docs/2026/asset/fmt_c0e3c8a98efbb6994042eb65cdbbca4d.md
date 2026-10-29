
![](https://bbs.kanxue.com/upload/attach/202210/905443_DA7WCH27PWZGTYH.png)

我们前面提到了`Pass`，而`llvm`通过编写`Pass`将中间语言`IR`进行优化，而`Ollvm`则是通过编写更加复杂的`Pass`，将代码复杂化，这样就达到了混淆的目的。

### 1.2 ollvm分类

前面我们已经知晓了`ollvm`的原理，那么这里我们理解最原始的`ollvm`的分类就变得更加容易。官方的`Ollvm`更新到`llvm`的`4.0`，目前主要分为：指令替换、虚假控制流、控制流平坦化。当然后面越来越多的人编写了更加复杂的`Pass`，这里我们就不具体深究了。

![image-20220921201852889](https://bbs.kanxue.com/upload/attach/202210/905443_DS26AAV57YZHKRQ.png)

下面我们简单的介绍每个`Pass`的实现原理

#### 1.2.1 指令替换（Sub）

指令替换，将一条运算指令，替换为多条等价的运算指令。例如：`y=x+1`变为`y=x+1-1`

#### 1.2.2 虚假控制流（bcf）

虚假控制流混淆主要通过加入包含不透明谓词的条件跳转和不可达的基本块，来干扰`IDA`的控制流分析和`F5`反汇编

![image-20220921202830658](https://bbs.kanxue.com/upload/attach/202210/905443_B2NNVQ2MJJHRWBA.png)

不透明谓词：在跳转前就已经确定的不等式，但是`IDA`无法分析，例如`y > 10 || x * (x + 1) % 2 == 0`这个不等式，大家都知道`x * (x + 1) % 2 == 0`这个式子是恒成立的，因此刚才这个不等式是恒成立的，当然这是我们知道，而`IDA`就不知道了，`IDA`不确定`x`，`y`的值，因此无法识别出来，因此就可以增加虚假控制流。

不透明谓词是研究反虚假控制流的重点，因此不透明谓词除了永真/永假型、还有可真可假型，而现在的研究都在此基础上进行了深入研究，所以不透明谓词的研究也成为难点。

不可达基本块，是指在虚假控制流中一些基本块是永远不可能执行的，这也是进一步去增加代码的复杂程度

#### 1.2.3 控制流平坦化（Fla）

控制流平坦化，主要通过一个主分发器来控制程序基本块的执行流程。该方法将所有基本代码放到控制流最底部，然后删除原理基本块之间跳转关系，添加次分发器来控制分发逻辑，然后过新的复杂分发逻辑还原原来程序块之间的逻辑关系。

![image-20220921210110215](https://bbs.kanxue.com/upload/attach/202210/905443_SVP3V9ADVNE5MA6.png)

我们在阅读源码后，会发现逻辑十分清晰，`Ollvm`的基本流程：

- 添加一个随机数种子 `blockID`
- 保存所有的基本块
- 将代码中含有`switch`改为`if`
- 删除第一个基本块，第一个需要特殊处理
- 识别`main`中的`if`，并且删除跳转指令
- 插入一个`switch`指令
- 第一个块跳转到`loopEntry`块
- 把所有的`block`保存到`switch`语句
- 重新计算`switch`变量的值
- 处理不是条件跳转 直接删除`jump` 跳转到`loopEnd` 进行下一轮循环
- 处理条件跳转 对真分支和假分支进行相应处理 真则选择真的`ID`

|  |  |
| --- | --- |
| 1 | `先将里面含switch的改为if-else，再将所有的if-else变为Switch的结果，所以多次进行控制流平坦化就会变得越来越复杂` |

#### 1.2.4 字符串加密

字符串加密的原理很简单，编写一个`pass`将其中的字符串信息使用一些加密算法进行加密，然后特定的时间进行还原。一般含有字符串混淆、函数名混淆、不在`init_array`解密等

## 2.Ollvm反混淆

`ollvm`反混淆主要思路基本上就是静态分析和动态分析，基本的流程大体都是：

|  |  |
| --- | --- |
| 1  2  3  4  5 | `（1）找到所有基本块（特征匹配或机器学习） 控制流平坦化中区分虚假块和真实块是难点`  `（2）动静态分析找到真实块的联系`  `静态分析：符号执行/反编译器提供的IL的API`  `动态分析：模拟指令/IDA trace`  `（3）编写相应的patch脚本，进行还原原程序` |

### 2.1 指令替换

指令替换一般可以使用`llvm`的`pass`进行优化，或者使用`Miasm`框架进行匹配，然后优化处理即可，由于指令替换一般不会影响程序整体逻辑，这里我们不进行深究。

### 2.2 反字符串加密

字符串加密的的常规解决方式：

**（1）特征搜索**

一般在`so`中可以直接搜索`datadiv_decode`，一般很多编写解密函数进行操作是这个函数，针对这种情况，一般可以通过`frida hook`就可以拿到解密后的值，然后进行`patch`

**（2）init_array中解密**

字符串解密操作在`init_arrray`中进行，一般可以通过模拟执行`init_array`，然后将解密后的字符串全部保存下来

**（3）jni_onload解密**

在`jni_onload`函数中进行解密操作，这时候就要进行`inlinehook`拿到解密后寄存器的值，也可以进行`hook`，也可以使用`unicorn`进行操作

### 2.3 反虚假控制流

虚假控制流去除的思路一般为除去不可达块和不透明谓词。但是难点在于不透明谓词，现在不透明谓词的研究不断发展，有永真/永假型不透明谓词，也有可真可假型不透明谓词。当然针对复杂的虚假控制流，在反混淆过程中还需要考虑死循环等问题

**不透明谓词：**

|  |  |
| --- | --- |
| 1  2 | `永真/假型：插入的后续基本块中必有一个不被执行`  `可真可假型：插入的两个后继基本块的语义应相同` |

针对简单的控制流混淆，去不透明谓词的思想主要是：

|  |  |
| --- | --- |
| 1  2  3 | `（1）不直接处理不透明谓词，通过让不透明谓词的变量地址可读，则IDA便可以优化`  `（2）直接将不透明谓词赋值为0或者将不透明谓词中变量x，y赋值为0`  `（3）编译器优化去干掉不透明谓词` |

**不可达块：**

不可达块是指控制流永远无法到达的基本块，一般我们可以使用符号执行或模拟执行来除去不可达基本块

### 2.4 反控制流平坦化

一般通用的反控制流平坦化思路：

|  |  |
| --- | --- |
| 1  2  3  4 | `（1）先保存所有的基本块`  `（2）区分真实块和分发器（虚假块） 一般通过规则匹配来做，但是并无法使用所有情况 （难点）`  `（3）连接真实块的顺序  一般静态可以通过IDA trace然后编写IDApython脚本，动态可以通过符号执行、模拟执行`  `（4）编写patch修复  对目标函数进行修复、恢复原始逻辑` |

**（1）保存所有的基本块**

控制流平坦化本质逻辑是把原始的基本块都碎片化，再通过`switch-case`语句，对函数的执行流进行重建。那么反混淆的时候，可以尝试根据主分发器将这些执行链给一条条的拆解出来，具体划分为三类：

|  |  |
| --- | --- |
| 1  2  3 | `入口链：原始函数代码的入口逻辑链，为序言到主分发器的执行路径`  `循环链：入口及出口均为主分发器的流程链，对应混淆过程中的循环体`  `Return链：指代入口为主分发器，出口为目标函数结束地址的流程链` |

**（2）区分真实块和分发器**

其中入口链没有分发器这样的控制块，所以代码全是真实指令。所以主要是找出循环链及`Return`链中的真实块，一般有几种思路：

|  |  |
| --- | --- |
| 1  2  3 | `（1）通过特征匹配来找真实块`  `（2）真实块的入口跳转地址必须为绝对的比较指令，如beq、bne，首次匹配到这种绝对的跳转指令，就能定位对应流程链中的真实入口地址，然后识别真是快`  `（3）凡是有内存操作的以及有bl/blx函数调用的都是真实块，其中有一些指向主分发器的虚拟块也会包含内存操作` |

主分发器的确定：直接遍历目标函数下的所有基本块，并计算每一个`Block`的引用次数，数量最多的那个就是主分发器

**（3）连接真实块的顺序**

通过判断`movwne/movtne r1`指令的地址是否大于`mov r1`，如果大于就说明此真实块会有两条路径去指向两个基本块。对于有两条路径的真实块就需要寻找两次分别去寻找两条路径下对应的真实块，而对于没有两条路径的真实块就直接寻找一次路径就`ok`了

一般我们使用模拟执行能解决该问题，但遇到上面情况并需要相应的手动修改

**（4）编写patch**

|  |  |
| --- | --- |
| 1  2 | `1.打patch，即通过jump指令或者一些条件跳转指令试图将它们重新连接起来`  `2.提取出所有真实块的指令，并根据它们之间的关系，计算相对偏移，据此对函数进行重构` |

前者简单，但是适用性不强，后者复杂，但是工作量大。这里我们主要介绍第一种`patch`

|  |  |
| --- | --- |
| 1  2  3 | `(1)把无用块都改成nop指令`  `(2)针对没有产生分支的真实块把最后一条指令改成jmp指令跳转到下一真实块`  `(3)产生分支的真实块把CMOV指令改成相应的条件跳转指令跳向符合条件的分支，例如CMOVZ 改成JZ ，再在这条之后添加JMP 指令跳向另一分支` |

## 3.Ollvm反混淆实操

这里收集了一些网上开源的脚本和样例，便于大家学习

### 3.1反字符串加密

首先我们打开一个字符串加密的样本

![image-20220922172044918](https://bbs.kanxue.com/upload/attach/202210/905443_UJVFE2ZA8DZYHRU.png)

我们可以发现导出函数中有`datadiv_decode`字段，我们初步判定就是在该函数中完成对字符串的加密

![image-20220922172239875](https://bbs.kanxue.com/upload/attach/202210/905443_V8BR59NKR29X5F8.png)

![image-20220922172318658](https://bbs.kanxue.com/upload/attach/202210/905443_DGT5NY4U4DVC9UZ.png)

很明显发现这些函数进行了字符串加密

我们在查看加密函数`datadiv_decode`的交叉引用，可以很明显的发现在`init_array`中完成了对字符串的解密

![image-20220922172516725](https://bbs.kanxue.com/upload/attach/202210/905443_J6GAZT7DBP5NMSV.png)

下面我们使用模拟执行的方法来去除`ollvm` 字符串加密

#### 3.1.1 AndroidNativeEmu

`AndroidNativeEmu`是基于`Unicorn`的框架，主要解决`Unicorn`不支持第三方库、`JNI_Onload`调用等问题

反混淆思路：

|  |  |
| --- | --- |
| 1 | `我们分析得知，字符串解密一定在init_array运行完结束，将字符串读取到内存，因此我们运行init_array，然后将内存中解密的字符串保存下来即可` |

运行环境：

|  |  |
| --- | --- |
| 1  2 | `（1）不使用AndroidNativeEmu项目，可以直接安装 pip install androidemu`  `（2）也可以去原仓库进行下载` |

这里我采用第一种环境，解密脚本：

```
import logging
import sys
from unicorn import *
import struct
from androidemu.emulator import Emulator
logging.basicConfig(
stream=sys.stdout,
level=logging.DEBUG,
format='%(asctime)s %(levelname)7s %(name)34s | %(message)s'
)
logger = logging.getLogger(__name__)
emulator = Emulator(vfp_inst_set=True, vfs_root='vfs')
str_datas = {}
def hook_mem_write(uc,type,address,size,value,userdata):
try:
curdata = struct.pack("I", value)[:size]
str_datas[address] = curdata
except:
print(size)
emulator.mu.hook_add(UC_HOOK_MEM_WRITE,hook_mem_write)
lib_module = emulator.load_library('obf.so',do_init=True)
base_addr = lib_module.base
sodata = open('obf.so','rb').read()
for address,value in str_datas.items():
if base_addr < address < base_addr + lib_module.size:
offset = address - base_addr -0x1000
print('address:0x%x data:%s offset:0x%x ' % (address, value, offset+0x1000))
sodata = sodata[:(offset)] + value + sodata[offset+len(value):]
with open('obf_new.so','wb') as file:
file.write(sodata)
file.close()
```

|  |  |
| --- | --- |
| 1  2  3  4 | `上面脚本中：`  `obf.so为目标样本`  `如果在框架中运行，需要导入libc.so`  `最重要需要注意由于IDA偏移和绝对地址不对 需要offset-0x1000` |

![image-20220922173658071](https://bbs.kanxue.com/upload/attach/202210/905443_EWBADQJC22W2Z5F.png)

运行脚本，得到结果

![image-20220922173829174](https://bbs.kanxue.com/upload/attach/202210/905443_VETAD3VBDUCVVDG.png)

修复前后的`so`对比：

原`so`:

![image-20220922173934780](https://bbs.kanxue.com/upload/attach/202210/905443_VUNDB6YWVQKTUAR.png)

修复`so`：

![image-20220922174035699](https://bbs.kanxue.com/upload/attach/202210/905443_5RDG778YSUJWAJ9.png)

#### 3.1.2 IDA插件

上面我们使用`Unicorn`进行了模拟调用，很显然我们还可以制作成相应的`IDApython`脚本，通过插件来进行动态的查看

参考链接：[使用unicorn对ollvm字符串进行解密](https://bbs.pediy.com/thread-268108.htm)

环境支持：

|  |  |
| --- | --- |
| 1  2  3  4 | `pip install unicorn  IDApython安装unicorn`  `Ida 7.x`  `python3`  `uEmu.py` |
