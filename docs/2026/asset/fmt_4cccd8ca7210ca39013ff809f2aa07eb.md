
# [原创] NDK 集成 OLLVM 模块流程记录

> 欢迎关注个人微信公众号《[Android安全工程](<elink@c25K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3S2G2L8X3N6Z5N6h3q4Q4x3X3g2U0L8W2)9J5c8V1q4F1k6s2u0G2K9h3c8Q4c8e0g2Q4b7f1g2Q4z5o6W2Q4c8e0g2Q4z5o6g2Q4b7e0S2Q4c8e0g2Q4b7U0N6Q4b7e0S2Q4c8e0N6Q4b7e0S2Q4z5p5t1`.>)》（可点击进行扫码关注）。个人微信公众号主要围绕 Android 应用的安全防护和逆向分析，分享各种安全攻防手段、Hook 技术、ARM 汇编等 Android 相关的知识。

## 集成流程

> 具体的编译 NDK 的 LLVM 的流程可参考文章：[编译 NDK 特定的 LLVM 版本的流程记录](https://bbs.kanxue.com/thread-277727.htm)

获取 ollvm 混淆部分代码：[8a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7%4u0U0i4K6u0V1j5$3q4K6i4K6u0r3k6X3I4G2N6h3&6V1k6i4t1`.](<elink@dccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7%4u0U0i4K6u0V1j5$3q4K6i4K6u0r3k6X3I4G2N6h3&6V1k6i4t1`.>)

1. 复制混淆部分 pass 到 `llvm12` 中：

```
flounder/llvm/include/llvm/Transforms/Obfuscation -> toolchain/llvm-project/llvm/include/llvm/Transforms/Obfuscation
flounder/llvm/lib/Transforms/Obfuscation -> toolchain/llvm-project/llvm/lib/Transforms/Obfuscation
```

2. 添加下面这行到 `llvm-project\llvm\lib\Transforms\CMakeLists.txt` 的第 13 行：

```
add_subdirectory(Obfuscation)
```

![](upload/attach/202306/928718_6R5KUWAMT5PM6QF.png)

3. 添加下面这行到 `llvm-project\llvm\lib\Transforms\IPO\CMakeLists.txt` 的 73 行：

```
Obfuscation
```

![](upload/attach/202306/928718_Z6EFCCU8DBHBPKE.png)

4. 在 `llvm-project\llvm\lib\Transforms\IPO\PassManagerBuilder.cpp` 导入头文件：

```cpp
#include "llvm/Transforms/Obfuscation/BogusControlFlow.h"
#include "llvm/Transforms/Obfuscation/Flattening.h"
#include "llvm/Transforms/Obfuscation/Split.h"
#include "llvm/Transforms/Obfuscation/Substitution.h"
#include "llvm/Transforms/Obfuscation/CryptoUtils.h"
#include "llvm/Transforms/Obfuscation/StringObfuscation.h"
```

![](upload/attach/202306/928718_UEWW8GA8Y589KNY.png)

在同一个文件的第 93 行：

```cpp
// Flags for obfuscation
static cl::opt<std::string> Seed("seed", cl::init(""), cl::desc("seed for the random"));
static cl::opt<std::string> AesSeed("aesSeed", cl::init(""), cl::desc("seed for the AES-CTR PRNG"));
static cl::opt<bool> StringObf("sobf", cl::init(false), cl::desc("Enable the string obfuscation"));
static cl::opt<bool> Flattening("fla", cl::init(false), cl::desc("Enable the flattening pass"));
static cl::opt<bool> BogusControlFlow("bcf", cl::init(false), cl::desc("Enable bogus control flow"));
static cl::opt<bool> Substitution("sub", cl::init(false), cl::desc("Enable instruction substitutions"));
static cl::opt<bool> Split("split", cl::init(false), cl::desc("Enable basic block splitting"));
// Flags for obfuscation
```

![](upload/attach/202306/928718_DY4X2PEJ7SV27NG.png)

在同一个文件的第 569 行：

```cpp
//obfuscation related pass
MPM.add(createSplitBasicBlockPass(Split));
MPM.add(createBogusPass(BogusControlFlow));
MPM.add(createFlatteningPass(Flattening));
MPM.add(createStringObfuscationPass(StringObf));
MPM.add(createSubstitutionPass(Substitution));
```

![](upload/attach/202306/928718_MJQF7YB38FDRQRY.png)

5. 在 `llvm-project\llvm\lib\Transforms\Obfuscation\StringObfuscation.cpp` 文件中：

在第 10 行把 `#include "llvm/IR/CallSite.h"` 改为 `#include "llvm/IR/AbstractCallSite.h"`

![](upload/attach/202306/928718_U2EEKNG9YASHT7J.png)

在第 15 行加入：

```cpp
#include "llvm/IR/Instructions.h"
```

![](upload/attach/202306/928718_77VU63479MGT7KK.png)

在同一个文件的第 174 处（共三处地方）：

```cpp
LoadInst *ptr_19 = new LoadInst(gvar->getType()->getArrayElementType(),
gvar, "", false, label_for_body);
ptr_19->setAlignment(Align(8));
...
LoadInst* int8_20 = new LoadInst(ptr_arrayidx->getType()->getArrayElementType(), ptr_arrayidx, "", false, label_for_body);
int8_20->setAlignment(Align(1));
...
void_21->setAlignment(Align(1));
```

![](upload/attach/202306/928718_K4FKW2Z2BRNEXJQ.png)

6. 在 `llvm-project\llvm\lib\Transforms\Obfuscation\Substitution.cpp` 文件的第 215 行：

```cpp
// Implementation of a = -(-b + (-c))
void Substitution::addDoubleNeg(BinaryOperator *bo) {
BinaryOperator *op, *op2 = NULL;
UnaryOperator *op3, *op4;
if (bo->getOpcode() == Instruction::Add) {
op = BinaryOperator::CreateNeg(bo->getOperand(0), "", bo);
op2 = BinaryOperator::CreateNeg(bo->getOperand(1), "", bo);
op = BinaryOperator::Create(Instruction::Add, op, op2, "", bo);
op = BinaryOperator::CreateNeg(op, "", bo);
bo->replaceAllUsesWith(op);
// Check signed wrap
//op->setHasNoSignedWrap(bo->hasNoSignedWrap());
//op->setHasNoUnsignedWrap(bo->hasNoUnsignedWrap());
} else {
op3 = UnaryOperator::CreateFNeg(bo->getOperand(0), "", bo);
op4 = UnaryOperator::CreateFNeg(bo->getOperand(1), "", bo);
op = BinaryOperator::Create(Instruction::FAdd, op3, op4, "", bo);
op3 = UnaryOperator::CreateFNeg(op, "", bo);
bo->replaceAllUsesWith(op3);
}
}
```

![](upload/attach/202306/928718_B9YF3BS62YA2N48.png)

在第 319 行（原 299 行）修改 `Substitution::subNeg` 函数：

```cpp
// Implementation of a = b + (-c)
void Substitution::subNeg(BinaryOperator *bo) {
BinaryOperator *op = NULL;
if (bo->getOpcode() == Instruction::Sub) {
op = BinaryOperator::CreateNeg(bo->getOperand(1), "", bo);
op = BinaryOperator::Create(Instruction::Add, bo->getOperand(0), op, "", bo);
// Check signed wrap
//op->setHasNoSignedWrap(bo->hasNoSignedWrap());
//op->setHasNoUnsignedWrap(bo->hasNoUnsignedWrap());
} else {
auto op1 = UnaryOperator::CreateFNeg(bo->getOperand(1), "", bo);
op = BinaryOperator::Create(Instruction::FAdd, bo->getOperand(0), op1, "", bo);
}
bo->replaceAllUsesWith(op);
}
```

![](upload/attach/202306/928718_49Q3QXR8P9YWP57.png)

7. 在 `llvm-project\llvm\lib\Transforms\Obfuscation\BogusControlFlow.cpp` 文件的第 380 行添加：

```cpp
UnaryOperator *op2;
```

![](upload/attach/202306/928718_25GBCKREDGMAZKC.png)

在同一个文件的第 422 行：

```cpp
case 1: op2 = UnaryOperator::CreateFNeg(i->getOperand(0),*var,&*i);
```

![](upload/attach/202306/928718_9Q6XSP7X9JEW3TK.png)

在同一个文件的第 573 行：

```cpp
opX = new LoadInst (x->getType()->getElementType(), (Value *)x, "", (*i));
opY = new LoadInst (x->getType()->getElementType(), (Value *)y, "", (*i));
```

![](upload/attach/202306/928718_35CVDCZ9C5XE3WE.png)

在 `llvm-project\llvm\include\llvm\InitializePasses.h` 的第 453 行添加：

```cpp
void initializeFlatteningPass(PassRegistry&);
```

![](upload/attach/202306/928718_5ZNEP5MWGSJTE5E.png)

在 `llvm-project\llvm\lib\Transforms\Obfuscation\Flattening.cpp` 的第 17 行添加：

```cpp
#include "llvm/InitializePasses.h"
```

![](upload/attach/202306/928718_T33UE55U9DNYPR8.png)

在相同文件的第 123 行修改：

```cpp
load = new LoadInst(switchVar->getType()->getElementType(), switchVar, "switchVar", loopEntry);
```

![](upload/attach/202306/928718_ZTDW4VARSTCEW9R.png)

在相同文件的第 239 行修改：

![](upload/attach/202306/928718_NSTJWJX9T5ZRKYW.png)

## 混淆效果

`Android.mk` 的参数配置（全局）：

```
LOCAL_CFLAGS += -mllvm -bcf -mllvm -bcf_loop=4 -mllvm -bcf_prob=100 -mllvm -sub -mllvm -sub_loop=2 -mllvm -fla -mllvm -sobf -mllvm -split
```

指定函数（局部）：

```
int binaryInsertionSort() __attribute((__annotate__(("bcf"))));
```

`CMakeLists.txt` 的参数配置（全局）：

```
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -mllvm -sub -mllvm -sobf -mllvm -fla ")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -mllvm -sub -mllvm -sobf -mllvm -fla")
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -mllvm -sub -mllvm -sobf -mllvm -fla" )
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -mllvm -sub -mllvm -sobf -mllvm -fla" )
```

指定函数混淆（局部）：

```
__attribute((__annotate__("bcf")))
__attribute((__annotate__("fla")))
__attribute((__annotate__("sub")))
__attribute((__annotate__("split")))
__attribute((__annotate__("sobf")))
void binaryInsertionSort(int arr[], int n) {}
```

**混淆效果：**

![](upload/attach/202306/928718_9H385QQ2AUGK4EB.png)

![](upload/attach/202306/928718_Q9K85CRUE6MMV4R.png)

## 总结

这里只是参考网上的帖子对 ollvm 整个源码下载，到编译集成到特定的 NDK 版本的过程做一个学习记录，但具体每个 pass 的性能、具体的防护效果、以及可能存在的问题在这里暂还没深入考究，这个系列待后续继续补充完善。

同时这块的源码、以及编译产物因体积较大，读者感兴趣可通过关注微信公众号《Android安全工程》回复关键字 `ollvm` 获取相关下载链接。

## 参考资料
