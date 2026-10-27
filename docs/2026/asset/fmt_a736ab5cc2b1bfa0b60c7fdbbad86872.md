
首先我们需要从官网下载LLVM Project中LLVM和Clang部分的源码，将其放在同一个目录，编译。这个过程可以参考知乎上的一个教程——[LLVM Pass入门导引](https://bbs.kanxue.com/elink@416K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0p5J5x3U0f1J5x3U0b7^5y4b7`.`.)以及官方文档，这里不再赘述了。我的编译环境是**Ubuntu 18.04**、**gcc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0**。

官方的教程[Writing an LLVM Pass](https://bbs.kanxue.com/elink@b0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0U0M7#2)9J5c8W2N6J5K9i4c8A6L8X3N6m8L8V1I4x3g2V1#2b7j5i4y4K6i4K6u0W2K9s2c8E0L8l9`.`.)中提供了一个示例，它的作用是打印所有函数的名称：

`test.sh`

```bash
./build/bin/clang -c -emit-llvm test.cpp -o test.bc
./build/bin/opt -load ./build/lib/LLVMHello.so -hello test.bc -o /dev/null
```

`test.cpp`

```cpp
#include <cstdio>

void func1(){}
void func2(){}
int main(){
  puts("Hello!");
}
```

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_6F2V8ZJ7W998WY3.png)

然而示例代码还是有点复杂，可以把它的代码简化一下，方便我们理解：

`Obfu.cpp`

```cpp
#include "llvm/IR/Function.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

namespace{
struct Obfu : public FunctionPass{
  static char ID;
  Obfu() : FunctionPass(ID){}

  bool runOnFunction(Function &F) override{
    outs() << "Function: " << F.getName() << "\n";
    return false;
  }
};
}

char Obfu::ID = 0;
static RegisterPass<Obfu> X("obfu", "My obfuscating pass");
```

写好之后重新编译LLVM，之前编译过的话重新编译的速度会很快，然后运行一下我们写的Pass：

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_K28SES8SURWN6HJ.png)

OK！这样后续的混淆功能只需要在这个框架上添加行了。

# 0x02. 控制流平坦化的基本思想和实现思路

控制流平坦化(Control Flow Flattening)的基本思想主要是通过一个主分发器来控制程序基本块的执行流程，例如下图是正常的执行流程：

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_5P3VP4AEY22BV77.png)

经过控制流平坦化后的执行流程就如下图：

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_FQUNKKACNGX9JAV.png)

可以看到除了基本块1外，其他的基本块都集中到了同一个层次上。不同基本块的调度顺序由主分发器决定（在程序里可以看做一个`switch`，不同的基本块就对应不同的`case`）。这样可以模糊基本块之间的前后关系，增加程序分析的难度。

控制流平坦化的过程相当于把原有程序正常的逻辑改为**一个循环嵌套一个`switch`**的逻辑。

以上图代表的程序为例，执行完基本块1后程序进入主分发器，然后执行基本块2对应的`case`。在原逻辑中基本块2的末尾是一个**条件跳转**，可以跳转到基本块3或者基本块4，在平坦化中基本块2的末尾会根据原有跳转的条件修改`switch`变量的值，使其接下来能执行到基本块3或者基本块4对应的`case`，然后返回主分发器（即进入下一个循环）。

如果是非条件跳转的话，比如基本块5到基本块6，在基本块5的末尾修改`switch`变量的值，使下一个循环中`switch`能到达基本块6对应的`case`即可。

用伪代码表示，未混淆的逻辑是这样：

```text
基本块1
基本块2
if(condition){
  基本块3
}else{
  基本块4
}
基本块5
基本块6
```

控制流平坦化的逻辑是这样：

```text
基本块1
switchVar = 2;
while(true){
  switch(switchVar){
    case 2:
      基本块2
      switchVar = condition ? 3 : 4;
    case 3:
      基本块3
      switchVar = 5
    case 4:
      基本块4
      switchVar = 5
    case 5:
      基本块5
      switchVar = 6
    case 6:
      基本块6
      goto end;
  }
}
end:
```

# 0x03. 基于LLVM Pass实现控制流平坦化

LLVM Pass的所有操作都是基于LLVM IR的，因此你需要对**LLVM IR**有所了解：[LLVM IR Tutorial](https://bbs.kanxue.com/elink@446K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6C8i4K6u0W2P5X3S2A6K9s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3@1k6@1j5i4u0Y4k6i4c8Q4x3@1c8Z5N6s2c8H3M7#2)9J5y4e0y4m8i4K6u0r3i4K6u0r3L8r3I4$3L8g2)9J5k6h3!0J5k6#2)9J5c8X3c8W2N6X3#2@1k6#2)9J5c8U0t1H3x3e0W2Q4x3X3b7H3y4q4)9J5c8Y4y4D9K9h3c8W2M7#2)9J5c8W2c8#2N6r3!0J5K9h3q4D9i4K6u0V1b7Y4u0A6k6r3N6W2M7Y4y4Q4x3X3c8x3e0q4k6y4i4K6g2X3d9g2u0Q4y4h3k6@1N6i4c8G2M7X3W2S2L8q4)9J5k6i4m8V1k6R3`.`.)

LLVM Pass的一些重要API也很有必要看一看：[LLVM Programmer’s Manual](https://bbs.kanxue.com/elink@d5cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0U0M7#2)9J5c8W2m8J5L8$3N6J5j5h3#2E0k6i4u0K6e0h3q4F1N6h3q4D9i4K6u0W2K9s2c8E0L8l9`.`.)

控制流平坦化的实现代码我参考的是：[OLLVM控制流平坦化源代码](https://bbs.kanxue.com/elink@f25K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3k6#2M7$3y4S2N6r3!0J5i4K6u0V1L8r3I4$3L8g2)9J5c8X3!0T1k6Y4g2K6j5$3q4@1L8%4u0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8r3I4$3L8g2)9J5k6o6c8Q4x3X3f1H3i4K6u0r3L8r3W2T1i4K6u0r3g2s2u0S2L8Y4y4X3L8%4u0E0M7#2)9J5c8V1!0T1k6Y4g2K6j5$3q4@1K9h3!0F1i4K6u0r3c8X3I4S2N6s2c8W2L8X3W2F1k6#2)9J5k6h3y4H3M7l9`.`.)

首先把函数的定义移到外面去，让重点更突出一点，现在我们只需要关注`flatten()`函数的实现就可以了：

`Obfu.cpp`

```cpp
#include "llvm/IR/Function.h"
#include "llvm/Pass.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/Instructions.h"
#include <vector>
using namespace llvm;

namespace{
struct Obfu : public FunctionPass{
  static char ID;
  Obfu() : FunctionPass(ID){}

  bool flatten(Function *f);
  bool runOnFunction(Function &F);
};
}

bool Obfu::runOnFunction(Function &F){
  return flatten(&F);
}

bool Obfu::flatten(Function *f){

}

char Obfu::ID = 0;
static RegisterPass<Obfu> X("obfu", "My obfuscating pass");
```

首先遍历函数中所有基本块，将其存到一个`vector`中：

```cpp
// 遍历函数所有基本块，将其存到vector中
vector<BasicBlock*> origBB;
for(BasicBlock &BB: *f){
  origBB.push_back(&BB);
}
// 基本块数量不超过1则无需平坦化
if(origBB.size() <= 1){
  return false;
}
```

根据平坦化的基本思想，第一个基本块是要单独拿出来处理的：

```cpp
// 从vector中去除第一个基本块
origBB.erase(origBB.begin());
BasicBlock *firstBB = &f->front();
```

如果第一个基本块的末尾是一个条件分支，则把条件跳转的**两个IR指令**（类似于汇编里的`cmp`和`jmp`）单独分离出来作为一个基本块，方便与非条件跳转统一处理：

```cpp
// 如果第一个基本块的末尾是条件跳转
if(isa<BranchInst>(firstBB->getTerminator())){
  BranchInst *br = cast<BranchInst>(firstBB->getTerminator());
  if(br->isConditional()){
    CmpInst *cmpInst = cast<CmpInst>(firstBB->getTerminator()->getPrevNode());
    BasicBlock *newBB = firstBB->splitBasicBlock(cmpInst, "newBB");
    origBB.insert(origBB.begin(), newBB);
  }
}
```

这里出现了一个函数`splitBasicBlock()`，如果你想知道这个函数到底做了什么操作，可以直接阅读源码内的注释，其他函数也是一样。简而言之，`splitBasicBlock()`函数在给定位置将一个基本块分为两个，并且在第一个基本块的末尾加上一个非条件跳转：

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_6G7WN3GRP8R8SPV.png)

有关`isa()`和`cast()`两个泛型函数的用法，参考上面提到的重要API文档：

![图片描述](https://bbs.kanxue.com/upload/attach/202102/910514_2YDTJ33BEKAVZ47.png)

接下来创建循环的循环头和循环尾，注意到新创建的基本块是被插入到`firstBB`前面的，所以还需要把`firstBB`移回顶部：

```cpp
// 创建循环
BasicBlock *loopEntry = BasicBlock::Create(f->getContext(), "loopEntry", f, firstBB);
BasicBlock *loopEnd = BasicBlock::Create(f->getContext(), "loopEnd", f, firstBB);
firstBB->moveBefore(loopEntry);
```

对第一个基本块做一些处理，主要包括去除第一个基本块原来的跳转，插入初始化switch on变量的指令，插入新的跳转使其进入循环：

```cpp
// 去除第一个基本块末尾的跳转
firstBB->getTerminator()->eraseFromParent();
// 用随机数初始化switch on变量
srand(time(0));
int randNumCase = rand();
AllocaInst *swVarPtr = new AllocaInst(int32Type, 0, "swVar.ptr", firstBB);
new StoreInst(ConstantInt::get(int32Type, randNumCase), swVarPtr, firstBB);
// 使第一个基本块跳转到loopEntry
BranchInst::Create(loopEntry, firstBB);
```

在`loopEntry`中插入`load`指令，`load`指令类似于C语言里的指针取值：

```cpp
// 在进入loopEntry读取switch on变量
LoadInst *swVar = new LoadInst(int32Type, swVarPtr, "swVar", false, loopEntry);
BranchInst::Create(loopEntry, loopEnd);
```
