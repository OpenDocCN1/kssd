
### 修复方法

```cpp
DISubprogram *LabelSP = getSubprogram(Label->getRawScope());
DISubprogram *LocSP = getSubprogram(Loc->getRawScope());
if (!LabelSP || !LocSP)
return;
CheckDI(LabelSP == LocSP,
"mismatched subprogram between llvm.dbg." + Kind +
" label and !dbg attachment",
&DLI, BB, F, Label, Label->getScope()->getSubprogram(), Loc,
Loc->getScope()->getSubprogram());
```

修复方法有两个，一个是更新最后的`DebugLoc`，另一个是把所有`DbgInfoIntrinsic`删了。

显然第二个更简单，本来就是个假的基本块，不会执行的，有调试信息也没什么用。添加下面的代码到第二步的后面，就可以修复这个问题。

```cpp
for (auto I = alteredBB->begin(), E = alteredBB->end(); I != E;) {
Instruction *Instr = &*I++;
if (isa<DbgInfoIntrinsic>(Instr))
Instr->eraseFromParent();
}
```

## 修复 `CatchPadInst` not the first non-PHI instruction

问题定位在`BogusControlFlow.cpp`的`addBogusFlow`函数。

比如有如下`IR`，`addBogusFlow`正在处理`147`这个基本块：

```
143: ; preds = ...
%144 = phi i32 ...
%145 = phi i32 ...
%146 = catchswitch within none [label %147] unwind to caller, !dbg !783
147: ; preds = %143
%148 = catchpad within %146 [ptr @"?filt$0@0@xx@@"], !dbg !783
catchret from %148 to label %149, !dbg !783
```

第一条非`PHI`指令就是`catchpad`了，那么`addBogusFlow`会使用`basicBlock->splitBasicBlock(i1, *var)`把`147`这个基本块分成两份：

```
143: ; preds = ...
%144 = phi i32 ...
%145 = phi i32 ...
%146 = catchswitch within none [label %147] unwind to caller, !dbg !783
147: ; preds = %143
br label %originalBB147
originalBB147: ; preds = %147
%148 = catchpad within %146 [ptr @"?filt$0@0@xx@@"], !dbg !783
catchret from %148 to label %149, !dbg !783
```

这时候问题就出现了，`147`基本块作为`catchswitch`的一个`catch`块，第一条指令不是`catchpad`了，`LLVM`就会处理不了这种异常表示（`LLVM`要求处理异常的基本块的`EH`指令需要是第一条）。

解决方法也很简单，在`addBogusFlow`函数开头判断当前处理的基本块是不是第一条指令为`catchpad`，如果是的话，就不处理：

```cpp
if (basicBlock->getFirstNonPHI()->isEHPad())
return;
```

这里使用了`isEHPad`来判断，原因是和这个报错同类型的不止一条，比如`The unwind destination does not have an exception handling instruction!`。原因都是`EH`相关指令不在基本块第一条，所以这里为了方便，只要第一条指令是`EHPad`，那就认为这个块是`EH`块，直接跳过处理。也不需要从`EHPad`后面的指令处理，因为这样有很多可能，比如可能改变支配关系什么的，导致什么奇奇怪怪的问题，这里就不给自己找麻烦了。

## 修复 `The unwind destination does not have...`

这个问题和上面的问题是一致的，`LLVM`要求处理异常的基本块的`EH`指令需要是第一条，看上面的修复方法就行。

## `CMake`构建参数

因为用插件形式编写的项目，所以`Windows`下用`VS`编译就加上`-DLLVM_OBFUSCATION_LINK_INTO_TOOLS=ON`，保证项目是被静态链接进的`LLVM`。

最后使用上和原版`OLLVM`是一模一样的，因为已经适配到了新`PM`上，所以不需要`-flegacy-pass-manager`了，而且也不支持这个选项了（大概后续版本旧`PM`会彻底消失在`LLVM`中端`Pipeline`吧）。

本文介绍了将`OLLVM`从官方的`LLVM4`分支移植到最新的`LLVM16`，包含`LLVM API`更新和`OLLVM`的`BUG`修复。

# 完整源码

我不想知道怎么适配的，给我源码！这是`GitHub`链接：[wwh1004/ollvm-16](https://bbs.kanxue.com/elink@22eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4N6$3R3I4x3o6l9@1i4K6u0r3L8$3I4D9N6X3#2Q4x3X3b7I4y4R3`.`.)，编译教程在`README.md`，编译好的`clang-cl.exe`在`Release`页面下载。

# 适配教程

`LLVM`从`4.0`到`16.0`包含了一些`API`更新，还有行为的改变，所以`OLLVM`的代码直接编译是过不了的，而且也不适配最新的`LLVM 16.0`。所以这篇移植教程包括了新`API`的适配，和对`LLVM`新行为的适配，还有对`OLLVM`本身`bug`的修复。

## `CMakeLists.txt`更新

老的`CMakeLists.txt`如下：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `add_llvm_library(LLVMObfuscation` `CryptoUtils.cpp` `Substitution.cpp` `BogusControlFlow.cpp` `Utils.cpp` `SplitBasicBlocks.cpp` `Flattening.cpp` `)` `add_dependencies(LLVMObfuscation intrinsics_gen)` |

`LLVMObfuscation`表示项目名称，根据`LLVM`的项目命名规则，因为是直接嵌入到`LLVM`内部的，`LLVM`就是一个必须的前缀。

我们用新的插件方式创建项目，可以降低日后更新工作量，`CMakeLists.txt`这样写：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 | `add_llvm_pass_plugin(Obfuscation` `CryptoUtils.cpp` `Substitution.cpp` `BogusControlFlow.cpp` `Utils.cpp` `SplitBasicBlock.cpp` `Flattening.cpp` `Plugin.cpp` `ADDITIONAL_HEADER_DIRS` `${PROJECT_SOURCE_DIR}` `DEPENDS` `intrinsics_gen` `)` |

因为是作为插件形式（可以静态链接进`LLVM`，也可以外挂），所以`LLVM`前缀是不需要的了，项目名就叫`Obfuscation`。

## `API`更新

首先是`BinaryOperator`中一些关于浮点数的运算移动到了`UnaryOperator`中，比如：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 | `-void Substitution::addDoubleNeg(BinaryOperator *bo) {` `- BinaryOperator *op, *op2 = NULL;` `+void addDoubleNeg(BinaryOperator *bo) {` `+ Instruction *op, *op2 = NULL;` `if (bo->getOpcode() == Instruction::Add) {` `op = BinaryOperator::CreateNeg(bo->getOperand(0), "", bo);` `// op->setHasNoSignedWrap(bo->hasNoSignedWrap());` `// op->setHasNoUnsignedWrap(bo->hasNoUnsignedWrap());` `} else {` `- op = BinaryOperator::CreateFNeg(bo->getOperand(0), "", bo);` `- op2 = BinaryOperator::CreateFNeg(bo->getOperand(1), "", bo);` `+ op = UnaryOperator::CreateFNeg(bo->getOperand(0), "", bo);` `+ op2 = UnaryOperator::CreateFNeg(bo->getOperand(1), "", bo);` `op = BinaryOperator::Create(Instruction::FAdd, op, op2, "", bo);` `- op = BinaryOperator::CreateFNeg(op, "", bo);` `+ op = UnaryOperator::CreateFNeg(op, "", bo);` `}` `bo->replaceAllUsesWith(op);` `}` |

然后许多`Instruction`的创建需要指定类型了，构造器第一个参数是类型，比如：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 | `+ Type *I32Ty = Type::getInt32Ty(F.getContext());` `// Create switch variable and set as it` `- switchVar =` `- new AllocaInst(Type::getInt32Ty(f->getContext()), 0, "switchVar", insert);` `+ switchVar = new AllocaInst(I32Ty, 0, "switchVar", insert);` `new StoreInst(` `ConstantInt::get(I32Ty, llvm::cryptoutils->scramble32(0, scrambling_key)),` `switchVar, insert);` `...` `- load = new LoadInst(switchVar, "switchVar", loopEntry);` `+ load = new LoadInst(I32Ty, switchVar, "switchVar", loopEntry);` |

## 新`PM`适配

新`Pass Manager`的适配不是很麻烦，大致有`2`个关注点。一个是`Pass`类的定义需要更改，`runOnFunction`变成了`run`，基类型变了。另一个是`Pass`的注册和插入时机。

首先看`Pass`类的变化，以`Flattening`为例，这个是原来的定义方法：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 | `struct` `Flattening :` `public` `FunctionPass {` `static` `char` `ID;` `Flattening() : FunctionPass(ID) {}` `bool` `runOnFunction(Function &F) {` `Function *tmp = &F;` `// Do we obfuscate` `if` `(toObfuscate(flag, tmp,` `"fla"``)) {` `if` `(flatten(tmp)) {` `++Flattened;` `}` `}` `return` `false``;` `}` `};` |

然后我们需要更改成新的：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 | `class` `FlatteningPass :` `public` `PassInfoMixin<FlatteningPass> {` `public``:` `PreservedAnalyses run(Function &F, FunctionAnalysisManager &AM) {` `// Do we obfuscate` `if` `(toObfuscate(Flattening, &F,` `"fla"``)) {` `if` `(flatten(F)) {` `++Flattened;` `}` `return` `PreservedAnalyses::none();` `}` `return` `PreservedAnalyses::all();` `}` `static` `bool` `isRequired() {` `return` `true``; }` `};` |

`isRequired`返回`true`的作用是让`O0`优化级别的时候，也就是带有`optnone`标识的时候，`Pass`依然运行。

`runOnFunction`函数返回一个`bool`类型，新的`run`函数返回一个`PreservedAnalyses`结构表示哪些`Analysis Pass`的结果有变化。比如没有任何改变，可以返回`PreservedAnalyses::all()`表示保留全部分析结果；如果变化了，可以返回`PreservedAnalyses::none()`表示分析结果全部失效，需要重新计算。

其它`Pass`的修改方式相同，难度不高。

接下来是新`Pass Manager`的`Pass`注册方式变化。

老的注册方式是定义一个全局变量，调用`RegisterPass`类进行注册：

|  |  |
| --- | --- |
| 1 | `static` `RegisterPass<BogusControlFlow> X(``"boguscf"``,` `"inserting bogus control flow"``);` |

新的注册方式有两种，一是修改`PassRegistry.def`和`PassBuilder.cpp`文件，直接追加`Pass`定义进去，但是这种比较麻烦。我们使用第二种，用插件接口进行注册。

我们创建一个新的`C++`源码文件`Plugin.cpp`，名字可以随便。因为我们`CMakeLists.txt`里面的项目名称就是`Obfuscation`，那么添加函数`getObfuscationPluginInfo`。如果`CMakeLists.txt`

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 | `llvm::PassPluginLibraryInfo getObfuscationPluginInfo() {` `return` `{` `LLVM_PLUGIN_API_VERSION,` `"Obfuscation"``, LLVM_VERSION_STRING,` `[](PassBuilder &PB) {` `...` `}};` `}` |

我们不需要显式注册`Pass`到`LLVM`内部，我们只需要通过回调函数把`Pass`注入到`PassBuilder`的`Pipeline`里面就行。

根据`OLLVM`原始代码看，`bcf`、`fla`、`split`这三个`Pass`是在`Pipeline`最开始时运行的，那么我们使用`registerPipelineStartEPCallback`进行注入：

|  |  |
| --- | --- |
| 1 2 3 4 5 | `PB.registerPipelineStartEPCallback([](llvm::ModulePassManager &MPM, OptimizationLevel Level) {` `MPM.addPass(createModuleToFunctionPassAdaptor(SplitBasicBlockPass()));` `MPM.addPass(createModuleToFunctionPassAdaptor(BogusControlFlowPass()));` `MPM.addPass(createModuleToFunctionPassAdaptor(FlatteningPass()));` `});` |

`sub`这个`Pass`是在优化完成后运行的，那么我们用`registerOptimizerLastEPCallback`进行注入：

|  |  |
| --- | --- |
| 1 2 3 | `PB.registerOptimizerLastEPCallback([](llvm::ModulePassManager &MPM, OptimizationLevel Level) {` `MPM.addPass(createModuleToFunctionPassAdaptor(SubstitutionPass()));` `});` |
