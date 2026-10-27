
## 最后我们的文件内容如下

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33 | `#include "BogusControlFlow.h"`  `#include "Flattening.h"`  `#include "SplitBasicBlock.h"`  `#include "Substitution.h"`  `#include "llvm/Passes/PassBuilder.h"`  `#include "llvm/Passes/PassPlugin.h"`    `using` `namespace` `llvm;`    `llvm::PassPluginLibraryInfo getObfuscationPluginInfo() {`  `return` `{`  `LLVM_PLUGIN_API_VERSION,` `"Obfuscation"``, LLVM_VERSION_STRING,`  `[](PassBuilder &PB) {`  `PB.registerPipelineStartEPCallback([](llvm::ModulePassManager &MPM,`  `OptimizationLevel Level) {`  `MPM.addPass(createModuleToFunctionPassAdaptor(SplitBasicBlockPass()));`  `MPM.addPass(`  `createModuleToFunctionPassAdaptor(BogusControlFlowPass()));`  `MPM.addPass(createModuleToFunctionPassAdaptor(FlatteningPass()));`  `});`  `PB.registerOptimizerLastEPCallback([](llvm::ModulePassManager &MPM,`  `OptimizationLevel Level) {`  `MPM.addPass(createModuleToFunctionPassAdaptor(SubstitutionPass()));`  `});`  `}};`  `}`    `#ifndef LLVM_OBFUSCATION_LINK_INTO_TOOLS`  `extern` `"C"` `LLVM_ATTRIBUTE_WEAK ::llvm::PassPluginLibraryInfo`  `llvmGetPassPluginInfo() {`  `return` `getObfuscationPluginInfo();`  `}`  `#endif` |

`llvmGetPassPluginInfo`的意义是，如果项目不是静态链接到LLVM，也就是编译为so/dll进行动态载入，那么使用`llvmGetPassPluginInfo`作为导出函数给LLVM注册Pass。可惜的是Windows不支持LLVM插件，我们只能静态链接。

## 不存在`createLowerSwitchPass`

这个是老PM时，Pass的创建方式。`Flattening.cpp`里面需要这个Pass，我们改成新Pass的调用方式即可。

|  |  |
| --- | --- |
| 1  2 | `LowerSwitchPass lower;`  `lower.run(F, AM);` |

## 修复`/dev/random`打开失败

问题原因是Windows下没有`/dev/random`和`/dev/urandom`，原版的OLLVM并不适配Windows。

修复方法是使用Windows API `CryptGenRandom`。这里我直接从网上抄了一份下来，就不给代码了，可以去文章开头写的GitHub链接上看，在`CryptoUtils.cpp`里。

## 修复`SplitBasicBlock`

这个Pass有个问题，处理基本块指令数量为2的时候，就会出错。问题定位在：

|  |  |
| --- | --- |
| 1  2  3  4 | `// Check splitN and current BB size`  `if` `((``size_t``)splitN > curr->size()) {`  `splitN = curr->size() - 1;`  `}` |

`splitN`默认是2，如果基本块的指令数量也为2，那么分割肯定会下标越界。

所以修复方法是把`>`改成`>=`就行了。

|  |  |
| --- | --- |
| 1  2  3  4 | `// Check splitN and current BB size`  `if` `((``size_t``)splitN >= curr->size()) {`  `splitN = curr->size() - 1;`  `}` |

## 修复`mismatched subprogram`

问题定位在`BogusControlFlow.cpp`的`createAlteredBasicBlock`函数。

`createAlteredBasicBlock`分为三步走，第一步是调用`llvm::CloneBasicBlock`创建克隆的基本块。克隆的基本块是不能直接用的，比如`AllocaInstr`分配的指针是新的，但是`StoreInstr`赋值的指针还是老的，所以需要修复。

那么第二步就是对克隆后的指令进行修复，让指令的操作数映射到克隆的指令上。这里OLLVM使用了`MapValue`函数。

第三步就是创建随机垃圾指令，然后设置一个恒假跳转语句跳转到这个假分支上。

问题出在了第二步上，LLVM有Intrinsic叫`DbgInfoIntrinsic`，在LLVM IR一般都表示为：

|  |  |
| --- | --- |
| 1  2  3  4 | `call void @llvm.dbg.value(metadata ptr null, metadata !``575``, metadata !DIExpression()), !dbg !``585`  `!``575` `=` `!DILocalVariable(name:` `"data"``, scope: !``565``,` `file``: !``3``, line:` `12``,` `type``: !``576``)`  `!``585` `=` `!DILocation(line:` `0``, scope: !``565``)`  `!``565` `=` `distinct !DISubprogram(name:` `"kull_m_rpc_drsr_RpcSecurityCallback"``, scope: !``3``,` `file``: !``3``, line:` `8``,` `type``: !``566``, scopeLine:` `9``, flags: DIFlagPrototyped, spFlags: DISPFlagDefinition | DISPFlagOptimized, unit: !``2``, retainedNodes: !``568``)` |

LLVM要求`DbgInfoIntrinsic`的`DISubprogram`与其附加的`DebugLoc`中的`DISubprogram`一致，也就是上面例子中`metadata !575`（作为Intrinsic的第二个参数）和`!dbg !585`的`DISubprogram`一致。因为`DISubprogram`表示IR对应的C++代码所在的源码文件位置，如果两处不一致显然是不合理的。

OLLVM调用`MapValue`会对`DbgInfoIntrinsic`进行映射，映射结果是正确的，但是第二个参数（`metadata !575`）被进行了一份拷贝，最后的`DebugLoc`（`!dbg !585`）不会进行拷贝，导致第二个参数对应的`DISubprogram`内容是一样的，但是与最后的`DebugLoc`的`DISubprogram`不是同一个实例了。这样比较两个`DISubprogram`就会返回`false`。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `DISubprogram *LabelSP = getSubprogram(Label->getRawScope());`  `DISubprogram *LocSP = getSubprogram(Loc->getRawScope());`  `if` `(!LabelSP || !LocSP)`  `return``;`    `CheckDI(LabelSP == LocSP,`  `"mismatched subprogram between llvm.dbg."` `+ Kind +`  `" label and !dbg attachment"``,`  `&DLI, BB, F, Label, Label->getScope()->getSubprogram(), Loc,`  `Loc->getScope()->getSubprogram());` |

修复方法有两个，一个是更新最后的`DebugLoc`，另一个是把所有`DbgInfoIntrinsic`删了。

显然第二个更简单，本来就是个假的基本块，不会执行的，有调试信息也没什么用。添加下面的代码到第二步的后面，就可以修复这个问题。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `for` `(``auto` `I = alteredBB->begin(), E = alteredBB->end(); I != E;) {`  `Instruction *Instr = &*I++;`  `if` `(isa<DbgInfoIntrinsic>(Instr))`  `Instr->eraseFromParent();`  `}` |

## 修复`CatchPadInst not the first non-PHI instruction`

问题定位在`BogusControlFlow.cpp`的`addBogusFlow`函数。

比如有如下IR，`addBogusFlow`正在处理`147`这个基本块：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `143``:                                              ; preds` `=` `...`  `%``144` `=` `phi i32 ...`  `%``145` `=` `phi i32 ...`  `%``146` `=` `catchswitch within none [label` `%``147``] unwind to caller, !dbg !``783`    `147``:                                              ; preds` `=` `%``143`  `%``148` `=` `catchpad within` `%``146` `[ptr @``"?filt$0@0@xx@@"``], !dbg !``783`  `catchret` `from` `%``148` `to label` `%``149``, !dbg !``783` |

第一条非PHI指令就是`catchpad`了，那么`addBogusFlow`会使用`basicBlock->splitBasicBlock(i1, *var)`把`147`这个基本块分成两份：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `143``:                                              ; preds` `=` `...`  `%``144` `=` `phi i32 ...`  `%``145` `=` `phi i32 ...`  `%``146` `=` `catchswitch within none [label` `%``147``] unwind to caller, !dbg !``783`    `147``:                                              ; preds` `=` `%``143`  `br label` `%``originalBB147`    `originalBB147:                                    ; preds` `=` `%``147`  `%``148` `=` `catchpad within` `%``146` `[ptr @``"?filt$0@0@xx@@"``], !dbg !``783`  `catchret` `from` `%``148` `to label` `%``149``, !dbg !``783` |

这时候问题就出现了，`147`基本块作为`catchswitch`的一个`catch`块，第一条指令不是`catchpad`了，LLVM就会处理不了这种异常表示（LLVM要求处理异常的基本块的EH指令需要是第一条）。

解决方法也很简单，在`addBogusFlow`函数开头判断当前处理的基本块是不是第一条指令为`catchpad`，如果是的话，就不处理：

|  |  |
| --- | --- |
| 1  2 | `if` `(basicBlock->getFirstNonPHI()->isEHPad())`  `return``;` |

这里使用了`isEHPad`来判断，原因是和这个报错同类型的不止一条，比如`"The unwind destination does not have an exception handling instruction!"`。原因都是EH相关指令不在基本块第一条，所以这里为了方便，只要第一条指令是`EHPad`，那就认为这个块是EH块，直接跳过处理。也不需要从`EHPad`后面的指令处理，因为这样有很多可能，比如可能改变支配关系什么的，导致什么奇奇怪怪的问题，这里就不给自己找麻烦了。

## 修复`The unwind destination does not have...`

这个问题和上面的问题是一致的，LLVM要求处理异常的基本块的EH指令需要是第一条，看上面的修复方法就行。

## CMake构建参数

因为用插件形式编写的项目，所以Windows下用VS编译就加上`-DLLVM_OBFUSCATION_LINK_INTO_TOOLS=ON`，保证项目是被静态链接进的LLVM。

最后使用上和原版OLLVM是一模一样的，因为已经适配到了新PM上，所以不需要`-flegacy-pass-manager`了，而且也不支持这个选项了（大概后续版本旧PM会彻底消失在LLVM中端Pipeline吧）。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
