
`bool` `HLLWriter::emitTargetCode(ShPtr<Module> module) {`  `...`  `//` `输出模块静态引用相关信息`  `if` `(emitXXXHeader()) {}`  `....`  `emitFunctions();`  `...`  `}`  
`//` `输出函数主体`  
`bool` `HLLWriter::emitFunctions() {`  
`FuncVector funcs(module->func_definition_begin(), module->func_definition_end());`  
`sortFuncsForEmission(funcs);`  
`bool` `somethingEmitted` `=` `false;`  
`for` `(const auto &func : funcs) {`  
`if` `(somethingEmitted) {`  
`//` `To produce an empty line between functions.`  
`out->newLine();`  
`}`  
`somethingEmitted |=` `emitFunction(func);`  
`}`  
`return` `somethingEmitted;`  
`}`  
`void CHLLWriter::visit(ShPtr<Function> func) {`  
`if` `(func->isDeclaration()) {`  
`emitFunctionPrototype(func);`  
`}`  
`else` `{`  
`emitFunctionDefinition(func);`  
`}`  
`}`  
`void CHLLWriter::emitFunctionDefinition(ShPtr<Function> func) {`  
`PRECONDITION(func->isDefinition(),`  
`"it has to be a definition"` `);`  
`out->addressPush(func->getStartAddress());`  
`emitFunctionHeader(func);`  
`out->space();`  
`emitBlock(func->getBody());`  
`out->addressPop();`  
`out->newLine();`  
`}`  
`//` `输出块主体`  
`void CHLLWriter::emitBlock(ShPtr<Statement> stmt) {`  
`out->punctuation('{');`  
`out->newLine();`  
`increaseIndentLevel();`  
`//` `Emit the block, statement by statement.`  
`do {`  
`out->addressPush(stmt->getAddress());`  
`emitGotoLabelIfNeeded(stmt);`  
`//` `Are there` `any` `metadata?`  
`std::string metadata` `=` `stmt->getMetadata();`  
`if` `(!metadata.empty()) {`  
`emitDebugComment(metadata);`  
`}`  
`stmt->accept(this);`  
`out->addressPop();`  
`//` `递归后继直到没有后继`  
`stmt` `=` `stmt->getSuccessor();`  
`}` `while` `(stmt);`  
`decreaseIndentLevel();`  
`out->space(getCurrentIndent());`  
`out->punctuation('}');`  
`}`

retdec内部构造的一个和LLVM IR中间指令等价的指令结构,这些指令又构成了和`llvm::BasicBlock`类等价的`Statement`等价结构,由`CFGNode`类负责管理每个`Function`的前驱后继和分支边.在`CHLLWriter`这个类中完成所有模块中的结构体和函数生成C语言高级代码的工作.从`Module`输出了模块的依赖结构体后,在`emitFunctions`环节`CHLLWriter`使用一个类`Visitor->visit`重载函数递归解析`Function`及其包含的内部`Statement`分支结构,递归`Statement`的所有后继和和分支边,直到没有后继子分支,在这个过程中输出内部变量retdec指令,至此全部反编译流程完成并输出至目标c文件.

### Klee源码分析

随着现代软件规模的加大、复杂度不断增加,对软件安全性的需求也不断上升,传统的分析测试技术因自身的局限已无法满足当前的软件安全性的需求.而符号执行技术因自身具有的优异特性在软件测试领域中受到了广泛的关注,常见和符号执行工具如[angr](https://bbs.kanxue.com/elink@f34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3N6J5i4K6u0r3j5h3&6Y4M7R3`.`.)(基于python),`klee`(基于llvm),[Triton](https://bbs.kanxue.com/elink@ca8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6v1L8$3&6S2N6r3S2S2L8W2y4S2L8s2N6S2L8W2)9J5c8W2c8J5K9i4c8G2L8R3`.`.)(也是基于llvm)等.`Klee`对约束求解进行优化策略,将自身的`STP`求解器与微软公司开发的`Z3`求解器结合并行化工作,特点是具有自动化测试用例生成,程序路径覆盖率高,无需人工操作等优点,受到了当前众多高校和科技公司的关注,随着许多符号执行工具不断出现,在实际应用中取得了很好的成绩.但是符号执行技术还存在着一些需要亟待解决的关键问题如路径爆炸,指针计算以及约束求解等问题.`Klee`中的符号执行过程称为状态机(`state machine`)机制,通过将`LLVMIR`中间指令解释成`Klee`内建指令,并将代码中指令映射为约束条件的符号变量值(`Symbolic Variable Value`),通过模拟程序执行流程构建单元代码的控制流图,当选择代码路径遇到分支语句时,提取并存储不同分支的约束条件,使用约束求解器求解进入某一分支所要满足的约束,最后根据路径调度算法来对控制流图中的所有路径进行遍历分析,代入满足约束条件的值以进入该分支,并对执行过的程序路径进行记录,下一次执行到该分支时选择未进入的分支执行,这样逐步对程序中所有的可行路径进行遍历.用程序的语言表述是`Klee`在测试用例的入口处将一个配置好的初始状态(`initial state`)放入池中,循环地从状态池里选择状态,并在那个状态下符号化地执行每条指令.直到状态池里没有状态了,或者时间超过了用户设定的.与普通进程不同,状态寄存器、堆栈和堆对象的存储位置指的是`Klee`表达式（树）,而不是原始数据值.表达式的叶子节点是符号变量或常量,内部节点是`LLVM IR`中间指令的操作符（例如算术操作、按位操作、比较和内存访问）.存储常量表达式的存储内容是实际的整形常数值,变量的存储是嵌套的表达式树引用.

由于我们没有去解析要分析的目标二进制相关的依赖文件模块,也在无法创建真实的进程环境而是采用符号执行的方法模拟一个函数执行,这过程必然会丢失或者欠缺真实运行环境实际存在在内存中的一些东西,所以这种方法不能被当作虚拟化环境分析目标二进制程序(emulating execution in a virtual (emulated) environment)而是近似模拟符号执行,简单的说就是没有虚拟层.每当一次`state`迭代结束,`state`的出口条件分支是布尔表达式,`Klee`会去查询约束求解器来确定当前路径上的分支条件是否恒为真或假.如果恒为真或假,就更新指令指针到预计的位置,`fork`出一个新的`state`.反之两种分支都有可能.然后`KLEE`路径调度算法就会去复制当前状态,并从这个状态`fork`出两个新的`state`,这样就可以同时探索两条路径,并且同时更新指令指针和路径条件.

```
bool` `SymbolicTypeReconstructor::runOnModule(llvm::Module& m)`
`{`
`for` `(auto addr : image->getRtti().getVtablesMsvc())`
`{`
`const Interpreter::InterpreterOptions opt;`
`RetdecInterpreterHandler            interpreterHandlerPtr` `=` `new RetdecInterpreterHandler(M);`
`interpreter` `=` `Interpreter::create(ctx, opt, interpreterHandlerPtr);`
`_config` `=` `ConfigProvider::getConfig(M);`
`_module` `=` `_config->_module;`
`interpreter->initSingleModule(M);`
`DataLayout TD` `=` `_module->getDataLayout();`
`Context::initialize(TD.isLittleEndian(),`
`static_cast<Expr::Width>(TD.getPointerSizeInBits()));`
`llvm::Function` `*` `func` `=` `_config->getLlvmFunction(addr);`
`char` `*` `argv` `=` `nullptr;`
`char` `*` `env` `=` `nullptr;`
`//` `构建runFunctionAsMain执行每个要分析的函数`
`interpreter->runFunctionAsMain(func,` `0``, &argv, &env);`
`}`
`}`
`//` `重构函数入口处执行`
`void Executor::runFunctionAsMain(Function` `*``f,` `int` `argc,` `char` `*``*``argv,` `char` `*``*``envp) {`
`std::vector<ref<Expr> > arguments;{`
`ExecutionState` `*``state` `=` `new ExecutionState(kmodule->functionMap[f]);`
`MemoryManager` `*``memory` `=` `new MemoryManager(NULL);`
`//` `构建参数内存对象`
`MemoryObject` `*``argvMO` `=`  
`memory->allocate((argc` `+` `1` `+` `envc` `+` `1` `+` `1)` `*` `NumPtrBytes,`  
`/*isLocal=*/` `false,` `/*isGlobal=*/` `true,`  
`/*allocSite=*/` `first,` `/*alignment=*/` `8` `);`
`//` `初始化全局变量:`
`initializeGlobals(*state);`
`void Executor::run(ExecutionState &state){`
`searcher` `=` `constructUserSearcher(*this);`
`std::vector<ExecutionState` `*``> newStates(states.begin(), states.end());`
`searcher->update(0, newStates, std::vector<ExecutionState` `*``>());`
`while` `(!states.empty() && !haltExecution) {`
`ExecutionState &state` `=` `searcher->selectState();`
`KInstruction` `*``ki` `=` `state.pc;`
`stepInstruction(state);`
`executeInstruction(state, ki);`
`updateStates(&state);`
`}`
```

`ExecutionState`由一个传入的`Function`构造初始状态,然后迭代执行里面的`Instruction`,具体工作由`executeInstruction`函数完成,内部分类模拟处理了所有指令类型,`Function`的自带参数被分配在它的`StackFrame->locals`字段,对应的是`llvm::.Function::.Argument`字段,从`x86`架构来讲32位的函数参数由`esp`传递,64位由`rcx`等寄存器和`rsp`传递,在初始化状态这些寄存器都被映射到对应的全局变量,也就是说是运行时赋值的,尽管`llvm`的`Function`构造时反编译程序已将关联的虚拟寄存器`Value`映射到这些参数.由于我们不知道要分析之前的函数实际运行时的参数值,我们只能构造一个伪造的预先分配`rcx`结构体和`rsp`栈结构体处理这些参数.这里我们要用到`klee`里有个很重要的概念就是内存模型。前面我们讲到`klee`的每个状态由`ExecutionState`解释执行,它的`AddressSpace`字段维护了所有运行时内存分配与调度,`AddressSpace`内部实际上维护的是一对`MemoryObject`和`ObjectState`组成的`ObjectPair`数组`Map`字典,这两个对象成对出现和使用描述的是同一块内存区域.在`Memory.h`定义文件里，我们可以看见有`MemoryObject`的定义,包含以下属性,包括`id`，地址，内存对象的大小等,这些信息仅供描述内存分配信息使用,而实际内存读写操作由`ObjectState`参与完成,它的地址由`MemoryManager->allocate`根据申请的大小分配相对映射地址,这个大小在申请时确定不能重分配.对应64位进程分析32位进程,为了实现等价的32位地址分配,笔者无法在`windows`实现源码中实现`mmap`类似功能在64位系统申请32位长度地址,而是改用`std::map`字典映射32地址和64实际分配地址,具体实现方式参考笔者代码.尽管每对`ObjectPair`的两个对象维护相同大小的内存区域,实际上只有`ObjectState`在内部`concreteStore`字段维护了实际分配的内存数据,当`ExecutionState`需要执行内存操作时先调用符号执行解释器(`solver`)采用符号化的方式求出需要操作`ObjectPair`,然后求出相对偏移量执行符号化的读写操作.
