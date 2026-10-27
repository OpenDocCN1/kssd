
# [原创]OLLVM虚假控制流源码学习笔记

虚假控制流的原理和去除方法我在另一篇文章已经讲过了：[利用angr符号执行去除虚假控制流](https://bbs.pediy.com/thread-266005.htm)，这篇文章简单记录下我阅读OLLVM虚假控制流源码的过程。

OLLVM虚假控制流源码地址：[BogusControlFlow.cpp](https://github.com/obfuscator-llvm/obfuscator/blob/master/lib/Transforms/Obfuscation/BogusControlFlow.cpp)

## 0x00. `runOnFunction` 函数

首先从`runOnFunction`函数开始：

```
virtual bool runOnFunction(Function &F){
// Check if the percentage is correct
if (ObfTimes <= 0) {
errs()<<"BogusControlFlow application number -bcf_loop=x must be x > 0";
return false;
}
// Check if the number of applications is correct
if ( !((ObfProbRate > 0) && (ObfProbRate <= 100)) ) {
errs()<<"BogusControlFlow application basic blocks percentage -bcf_prob=x must be 0 < x <= 100";
return false;
}
```

`ObfTimes`是对函数进行混淆的次数，`ObfProbRate`是对基本块进行混淆的概率，这两个分别是执行`opt`时传入的参数：

```
static cl::opt<int>
ObfProbRate("bcf_prob", cl::desc("Choose the probability [%] each basic blocks will be obfuscated by the -bcf pass"), cl::value_desc("probability rate"), cl::init(defaultObfRate), cl::Optional);
static cl::opt<int>
ObfTimes("bcf_loop", cl::desc("Choose how many time the -bcf pass loop on a function"), cl::value_desc("number of times"), cl::init(defaultObfTime), cl::Optional);
```

`runOnFunction`函数剩下的内容：

```
// If fla annotations
if(toObfuscate(flag,&F,"bcf")) {
bogus(F);
doF(*F.getParent());
return true;
}
return false;
} // end of runOnFunction()
```

`toObfuscate`函数的作用是检查这个函数之前有没有进行过虚假控制流混淆，如果有则返回`false`避免重复混淆，还检查了一些其他东西，感觉不是很重要。要重点分析的是`bogus`和`doF`函数。

## 0x01. `bogus` 函数

OLLVM的作者喜欢在函数开头就声明所有全局变量，先不管它。

```
void bogus(Function &F) {
// For statistics and debug
++NumFunction;
int NumBasicBlocks = 0;
bool firstTime = true; // First time we do the loop in this function
bool hasBeenModified = false;
DEBUG_WITH_TYPE("opt", errs() << "bcf: Started on function " << F.getName() << "\n");
DEBUG_WITH_TYPE("opt", errs() << "bcf: Probability rate: "<< ObfProbRate<< "\n");
if(ObfProbRate < 0 || ObfProbRate > 100){
DEBUG_WITH_TYPE("opt", errs() << "bcf: Incorrect value,"
<< " probability rate set to default value: "
<< defaultObfRate <<" \n");
ObfProbRate = defaultObfRate;
}
DEBUG_WITH_TYPE("opt", errs() << "bcf: How many times: "<< ObfTimes<< "\n");
if(ObfTimes <= 0){
DEBUG_WITH_TYPE("opt", errs() << "bcf: Incorrect value,"
<< " must be greater than 1. Set to default: "
<< defaultObfTime <<" \n");
ObfTimes = defaultObfTime;
}
NumTimesOnFunctions = ObfTimes;
```

进入`bogus`函数后首先检查了一遍`ObfProbRate`和`ObfTimes`（话说之前在`runOnFunction`里不就检查过了吗= =）。

```
NumTimesOnFunctions = ObfTimes;
```

这里的`NumTimesOnFunctions`是调试用的。虚假控制流的代码比控制流平坦化的代码多了几百行，重要的原因就是添加了很多调试语句。为了突出重点，之后的调试语句直接忽略:

```
STATISTIC(NumTimesOnFunctions, "b. Number of times we run on each function");
```

接着是一个超大的循环，直到函数结尾，我们去掉所有调试语句分析，循环总共有两层，第一层的循环开头把所有基本块存入一个list中：

```
do{
// Put all the function's block in a list
std::list<BasicBlock *> basicBlocks;
for (Function::iterator i=F.begin();i!=F.end();++i) {
basicBlocks.push_back(&*i);
}
```

接着进入第二层的循环，每个基本块有`ObfProbRate`的概率被混淆，即对基本块调用了`addBogusFlow`函数：

```
while(!basicBlocks.empty()){
NumBasicBlocks ++;
// Basic Blocks' selection
if((int)llvm::cryptoutils->get_range(100) <= ObfProbRate){
hasBeenModified = true;
BasicBlock *basicBlock = basicBlocks.front();
addBogusFlow(basicBlock, F);
}
// remove the block from the list
basicBlocks.pop_front();
} // end of while(!basicBlocks.empty())
```

后面就都是调试内容了，所以`bogus`函数的作用就是对指定函数的每个基本块以`ObfProbRate`的概率进行混淆，接下来分析`addBogusFlow`函数。

## 0x02. `addBogusFlow` 函数第一部分

首先通过`getFirstNonPHIOrDbgOrLifetime`函数和`splitBasicBlock`函数把`BasicBlock`分成了两部分，第一部分只包含`PHI Node`和一些调试信息，其中`Twine`可以看做字符串，即新产生的`BasicBlock`的名称：

```
/* addBogusFlow
*
* Add bogus flow to a given basic block, according to the header's description
*/
virtual void addBogusFlow(BasicBlock * basicBlock, Function &F){
// Split the block: first part with only the phi nodes and debug info and terminator
// created by splitBasicBlock. (-> No instruction)
// Second part with every instructions from the original block
// We do this way, so we don't have to adjust all the phi nodes, metadatas and so on
// for the first block. We have to let the phi nodes in the first part, because they
// actually are updated in the second part according to them.
BasicBlock::iterator i1 = basicBlock->begin();
if(basicBlock->getFirstNonPHIOrDbgOrLifetime())
i1 = (BasicBlock::iterator)basicBlock->getFirstNonPHIOrDbgOrLifetime();
Twine *var;
var = new Twine("originalBB");
BasicBlock *originalBB = basicBlock->splitBasicBlock(i1, *var);
```

随后调用`createAlteredBasicBlock`函数产生了一个新的基本块：

```
// Creating the altered basic block on which the first basicBlock will jump
Twine * var3 = new Twine("alteredBB");
BasicBlock *alteredBB = createAlteredBasicBlock(originalBB, *var3, &F);
```

## 0x03. `createAlteredBasicBlock` 函数

`createAlteredBasicBlock`函数首先调用了`CloneBasicBlock`函数对传入的基本块进行克隆：

```
virtual BasicBlock* createAlteredBasicBlock(BasicBlock * basicBlock,
const Twine & Name = "gen", Function * F = 0){
// Useful to remap the informations concerning instructions.
ValueToValueMapTy VMap;
BasicBlock * alteredBB = llvm::CloneBasicBlock (basicBlock, VMap, Name, F);
```

但是`CloneBasicBlock`函数进行的克隆并不是完全的克隆，第一他不会对指令的操作数进行替换，比如：

```
orig:
%a = ...
%b = fadd %a, ...
clone:
%a.clone = ...
%b.clone = fadd %a, ... ; Note that this references the old %a and
not %a.clone!
```

在clone出来的基本块中，`fadd`指令的操作数不是`%a.clone`，而是`%a`。

所以之后要通过`VMap`对所有操作数进行映射，使其恢复正常：

```
// Remap operands.
BasicBlock::iterator ji = basicBlock->begin();
for (BasicBlock::iterator i = alteredBB->begin(), e = alteredBB->end() ; i != e; ++i){
// Loop over the operands of the instruction
for(User::op_iterator opi = i->op_begin (), ope = i->op_end(); opi != ope; ++opi){
// get the value for the operand
Value *v = MapValue(*opi, VMap, RF_None, 0);
if (v != 0){
*opi = v;
DEBUG_WITH_TYPE("gen", errs() << "bcf: Value's operand has been setted\n");
}
}
```

第二，它不会对`PHI Node`进行任何处理，`PHI Node`的前驱块仍然是原始基本块的前驱块，但是新克隆出来的基本块并没有任何前驱块，所以我们要对`PHI Node`的前驱块进行`remap`：

```
// Remap phi nodes' incoming blocks.
if (PHINode *pn = dyn_cast<PHINode>(i)) {
for (unsigned j = 0, e = pn->getNumIncomingValues(); j != e; ++j) {
Value *v = MapValue(pn->getIncomingBlock(j), VMap, RF_None, 0);
if (v != 0){
pn->setIncomingBlock(j, cast<BasicBlock>(v));
}
}
}
```

解释一下什么是`PHI Node`，所有 LLVM 指令都使用`SSA` (Static Single Assignment，静态一次性赋值) 方式表示，即所有变量都只能被赋值一次，这样做主要是便于后期的代码优化。如下图，`%temp`的值被赋值成1后就永远是1了：

![图片描述](upload/attach/202102/910514_6RWUREBTATQRJMC.png)

`PHI Node`是一条可以一定程度上绕过`SSA`机制的指令，它可以根据不同的前驱基本块来赋值（有点像三元运算符）。如下图，如果`PHI Node`的前驱基本块是`entry`，则将`current_i`赋值为2，如果是`for_body`，则赋值为`%i_plus_one`：

![图片描述](upload/attach/202102/910514_HS9JP8AC34VYME5.png)

在这里我产生了一个疑问：既然`Clone`出来的基本块不存在任何前驱，那`PHI Node`的前继基本块到底被`remap`到了哪呢？

我翻了一下LLVM的源码，发现`VMap`只会保存新旧指令的映射关系：

![图片描述](upload/attach/202102/910514_SQCA3GYRCMP96RS.png)

然后根据`MapValue`的注释来看应该是映射成了`nullptr`：

![图片描述](upload/attach/202102/910514_GFGUTKXX9DVUHBY.png)

暂且不知道`PHI Node`中的前驱块设为`nullptr`会有什么影响，先继续往下看吧。

之后是恢复`Metadata`和`Debug`信息，个人感觉是可有可无的，不管它：
