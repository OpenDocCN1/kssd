
# LLVM PASS PWN 总结

## 前言

很久没写文章了，本来想像之前发的`glibc`那篇总结一样也整理整理`musl pwn`，`Linux Kernel`和`QEMU-Escape`相关笔记的，但是由于自己太懒了，加上又去搞了搞`IoT`相关的实战并尝试了下安全开发，就咕咕咕了（后面有空一定补QAQ）。

近期的比赛里出现了两次`LLVM PASS PWN`类的题目（`CISCN`初赛和强网杯），但是自己又一直摆烂没去研究。最近刚开学还算比较闲，就看了一下`LLVM PASS PWN`类的题目，这篇文章就浅浅记录一下自己对此类题目不成熟的总结吧。

## LLVM PASS 简介

`LLVM`是`C++`编写的构架编译器的框架系统，可用于优化以任意程序语言编写的程序。

`LLVM Pass`可用于对代码进行优化或者对代码插桩（插入新代码），`LLVM`的核心库中提供了一些`Pass`类可以继承，通过实现它的一些方法，可以对传入的`LLVM IR`进行遍历并操作。

`LLVM IR`即代码的中间表示，有三种形式：

1. `.ll` 格式：人类可以阅读的文本
2. `.bc` 格式：适合机器存储的二进制文件
3. 内存表示

下面给出`.ll`格式和`.bc`格式生成及相互转换的常用指令清单：

```text
.c -> .ll：clang -emit-llvm -S a.c -o a.ll
.c -> .bc: clang -emit-llvm -c a.c -o a.bc
.ll -> .bc: llvm-as a.ll -o a.bc
.bc -> .ll: llvm-dis a.bc -o a.ll
.bc -> .s: llc a.bc -o a.s
```

## 准备工作及注意点

需要安装`CTF`题目中常用的三个版本的`clang`及`LLVM`：

```bash
sudo apt install clang-8
sudo apt install llvm-8

sudo apt install clang-10
sudo apt install llvm-10

sudo apt install clang-12
sudo apt install llvm-12
```

`opt`是`LLVM`的优化器和分析器，可加载指定的模块，对输入的`LLVM IR`或者`LLVM`字节码进行优化或分析。`CTF`题目一般会给出所需版本的`opt`文件（可用`./opt --version`查看版本）或者在`README`文档中告知`opt`版本。安装好`llvm`后，可在`/usr/lib/llvm-xx/bin/opt`路径下找到对应`llvm`版本的`opt`文件（一般不开`PIE`保护）。

![](https://bbs.kanxue.com/upload/attach/202209/949925_XZ7NWZ3M2CEJKY3.png)

需要注意的是，最好使用题目所给`opt`同版本的`clang`生成`ll`或`bc`文件。如：题目所给的文件是`opt-8`，就最好使用`clang-8 -emit-llvm -S exp.c -o exp.ll`命令。

`LLVM PASS`类题目都会给出一个`xxx.so`，即自定义的`LLVM PASS`模块，漏洞点就自然会出现在其中。我们可以使用`opt -load ./xxx.so -xxx ./exp.{ll/bc}`命令加载模块并启动`LLVM`的优化分析（其中`-xxx`是`xxx.so`中注册的`PASS`的名称，`README`文档中一般会给出，也可以通过逆向`PASS`模块得到）。需要注意的是，若题目给了`opt`文件，就用题目指定的`opt`文件启动`LLVM`并调试（如命令`./opt-8 ...`），直接使用`opt-8 ...`命令是用的系统安装的`opt`，可能会和题目所给的有不同。

在打远程的时候，与内核和`QEMU`逃逸的题类似：将`exp.ll`或`exp.bc`通过`base64`加密传输到远程服务器，远程服务器会解码，并将得到的`LLVM IR`传给`LLVM`运行。

## 尝试编写第一个LLVM PASS

参考官方文档：afcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3k6r3!0U0M7#2)9J5c8W2N6J5K9i4c8A6L8X3N6m8L8V1I4x3g2V1#2b7j5i4y4K6i4K6u0W2K9s2c8E0L8l9.`.

这里魔改了一下官方文档中给出的`Hello Pass`，加入了一些`CTF`题中常见的`LLVM`语法：

```cpp
// Hello.cpp

#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/BasicBlock.h"
#include "llvm/IR/Instructions.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"

using namespace llvm;

namespace {
struct Hello : public FunctionPass {
  static char ID;
  Hello() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override {
    errs() << "Hello: ";
    errs().write_escaped(F.getName()) << '\n';
    SymbolTableList<BasicBlock>::const_iterator bbEnd = F.end();
    for (SymbolTableList<BasicBlock>::const_iterator bbIter = F.begin(); bbIter != bbEnd; ++bbIter) {
      SymbolTableList<Instruction>::const_iterator instIter = bbIter->begin();
      SymbolTableList<Instruction>::const_iterator instEnd = bbIter->end();
      for (; instIter != instEnd; ++instIter) {
        errs() << "OpcodeName = " << instIter->getOpcodeName() << " NumOperands = " << instIter->getNumOperands() << "\n";
        if (instIter->getOpcode() == 56) {
          if (const CallInst *call_inst = dyn_cast<CallInst>(instIter)) {
            errs() << call_inst->getCalledFunction()->getName() << "\n";
            for (int i = 0; i < instIter->getNumOperands() - 1; i++) {
              if (isa<ConstantInt>(call_inst->getOperand(i))) {
                errs() << "Operand " << i << " = " << dyn_cast<ConstantInt>(call_inst->getArgOperand(i))->getZExtValue() << "\n";
              }
            }
          }
        }
      }
    }
    return false;
  }
};
}

char Hello::ID = 0;

// Register for opt
static RegisterPass<Hello> X("Hello", "Hello World Pass");

// Register for clang
static RegisterStandardPasses Y(PassManagerBuilder::EP_EarlyAsPossible,
  [](const PassManagerBuilder &Builder, legacy::PassManagerBase &PM) {
    PM.add(new Hello());
  });
```

通过如下命令，可将其编译为`LLVMHello.so`模块：
