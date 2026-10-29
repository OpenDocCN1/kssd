
```llvm
originalBBalteredBB: ; preds = %originalBB, %entry
%retvalalteredBB = alloca i32, align 4
%argc.addralteredBB = alloca i32, align 4
%argv.addralteredBB = alloca i8**, align 8
%aalteredBB = alloca i32, align 4
store i32 0, i32* %retvalalteredBB, align 4
store i32 %argc, i32* %argc.addralteredBB, align 4
store i8** %argv, i8*** %argv.addralteredBB, align 8
%70 = load i8**, i8*** %argv.addralteredBB, align 8
%arrayidxalteredBB = getelementptr inbounds i8*, i8** %70, i64 1
%71 = load i8*, i8** %arrayidxalteredBB, align 8
%callalteredBB = call i32 @atoi(i8* %71) #3
store i32 %callalteredBB, i32* %aalteredBB, align 4
%72 = load i32, i32* %aalteredBB, align 4
%cmpalteredBB = icmp eq i32 %72, 0
br label %originalBB
```

```cpp
BasicBlock::iterator i = originalBB->end();
// Split at this point (we only want the terminator in the second part)
Twine * var5 = new Twine("originalBBpart2");
BasicBlock * originalBBpart2 = originalBB->splitBasicBlock(--i , *var5);
```

查找到`originBB`最后一条指令进行`split`，然后创建一个`originalBBpart2`基本块

```llvm
originalBBpart2：
br i1 %cmp, label %if.then, label %if.else
```

切割后`originBB`最后就变成了无条件的跳转：

```llvm
originalBB：
%retval = alloca i32, align 4
%argc.addr = alloca i32, align 4
%argv.addr = alloca i8**, align 8
%a = alloca i32, align 4
store i32 0, i32* %retval, align 4
store i32 %argc, i32* %argc.addr, align 4
store i8** %argv, i8*** %argv.addr, align 8
%0 = load i8**, i8*** %argv.addr, align 8
%arrayidx = getelementptr inbounds i8*, i8** %0, i64 1
%1 = load i8*, i8** %arrayidx, align 8
%call = call i32 @atoi(i8* %1) #2
store i32 %call, i32* %a, align 4
%2 = load i32, i32* %a, align 4
%cmp = icmp eq i32 %2, 0
br label %originalBBpart2
```

紧接着把`originBB`最后一行给删掉，创建一个`fcmp`的条件跳转：

```cpp
originalBB->getTerminator()->eraseFromParent();
Twine * var6 = new Twine("condition2");
FCmpInst * condition2 = new FCmpInst(*originalBB, CmpInst::FCMP_TRUE , LHS, RHS, *var6);
BranchInst::Create(originalBBpart2, alteredBB, (Value *)condition2, originalBB);
```

如下：

```llvm
originalBB：
%retval = alloca i32, align 4
%argc.addr = alloca i32, align 4
%argv.addr = alloca i8**, align 8
%a = alloca i32, align 4
store i32 0, i32* %retval, align 4
store i32 %argc, i32* %argc.addr, align 4
store i8** %argv, i8*** %argv.addr, align 8
%0 = load i8**, i8*** %argv.addr, align 8
%arrayidx = getelementptr inbounds i8*, i8** %0, i64 1
%1 = load i8*, i8** %arrayidx, align 8
%call = call i32 @atoi(i8* %1) #2
store i32 %call, i32* %a, align 4
%2 = load i32, i32* %a, align 4
%cmp = icmp eq i32 %2, 0
%condition2=fcmp true float 1.00000e+00,1.00000e+00
br i1 %condition2, label %originalBBpart2, label %originalBBalteredBB
```

这个函数主要就是创建了`entry`和`originalBB`代码块的最后两行浮点数比较的跳转

# 尾声

网上很多很多现成资料，我这个也是自己看了再总结的，如果哪里不对的话，还请多多指教。

目录

- [runOnFunction函数](https://bbs.kanxue.com/#runonfunction函数)
- [bogus函数](https://bbs.kanxue.com/#bogus函数)
- [目前源码：](https://bbs.kanxue.com/#目前源码：)
- [addBogusFlow函数1](https://bbs.kanxue.com/#addbogusflow函数1)
- [createAlteredBasicBlock函数](https://bbs.kanxue.com/#createalteredbasicblock函数)
- [原基本块：](https://bbs.kanxue.com/#原基本块：)
- [copy的基本块：](https://bbs.kanxue.com/#copy的基本块：)
- [addBogusFlow函数2](https://bbs.kanxue.com/#addbogusflow函数2)
- [尾声](https://bbs.kanxue.com/#尾声)

# `runOnFunction`函数

```cpp
if (ObfTimes <= 0) {
  errs() << "BogusControlFlow application number -bcf_loop=x must be x > 0";
  return false;
}
if ( !((ObfProbRate > 0) && (ObfProbRate <= 100)) ) {
  errs() << "BogusControlFlow application basic blocks percentage -bcf_prob=x must be 0 < x <= 100";
  return false;
}
```

```cpp
const int defaultObfRate = 30, defaultObfTime = 1;

static cl::opt<int> ObfProbRate("bcf_prob", cl::desc("Choose the probability [%] each basic blocks will be obfuscated by the -bcf pass"), cl::value_desc("probability rate"), cl::init(defaultObfRate), cl::Optional);

static cl::opt<int> ObfTimes("bcf_loop", cl::desc("Choose how many time the -bcf pass loop on a function"), cl::value_desc("number of times"), cl::init(defaultObfTime), cl::Optional);
```

这里的`ObfTimes`对应过来的默认值就是1，对函数进行混淆的次数，`ObfProbRate`就是30，对基本块进行混淆的概率，分别是`opt`时传入的参数。

```cpp
// check for compatible
for (BasicBlock &bb : F.getBasicBlockList()) {
  if ([previous internal execution text redacted]Inst>(bb.getTerminator())) {
    return false;
  }
}
```

枚举这个函数的所有基本块，如果这个函数后面基本块中含有`invoke`（调用了某个函数），它就不执行了，就退出这个基本块。

```cpp
if (toObfuscate(flag, &F, "bcf")) {
  bogus(F);
  doF(*F.getParent());
  return true;
}
```

判断有没有`bcf`也就是虚假控制流，有的话就进入。

# `bogus`函数

```cpp
if (ObfProbRate < 0 || ObfProbRate > 100) {
  DEBUG_WITH_TYPE("opt", errs() << "bcf: Incorrect value," << " probability rate set to default value: " << defaultObfRate << " \n");
  ObfProbRate = defaultObfRate;
}
if (ObfTimes <= 0) {
  DEBUG_WITH_TYPE("opt", errs() << "bcf: Incorrect value," << " must be greater than 1. Set to default: " << defaultObfTime << " \n");
  ObfTimes = defaultObfTime;
}
```

首先进行判断这个次数和概率，是否符合条件，不符合的话会进行设置默认值。

紧接着就是一个大型的do while循环里面包含着的代码：

```cpp
std::list<BasicBlock *> basicBlocks;
for (Function::iterator i = F.begin(); i != F.end(); ++i) {
  basicBlocks.push_back(&*i);
}
```

把所有的基本块放在`basicBlocks`列表里面。

获取一个随机值，如果符合的话就进入:

```cpp
if ((int)llvm::cryptoutils->get_range(100) <= ObfProbRate) {
  DEBUG_WITH_TYPE("opt", errs() << "bcf: Block " << NumBasicBlocks << " selected. \n");
  hasBeenModified = true;
  ++NumModifiedBasicBlocks;
  NumAddedBasicBlocks += 3;
  FinalNumBasicBlocks += 3;
  // Add bogus flow to the given Basic Block (see description)
  BasicBlock * basicBlock = basicBlocks.front();
  addBogusFlow(basicBlock, F);
} else {
  DEBUG_WITH_TYPE("opt", errs() << "bcf: Block " << NumBasicBlocks << " not selected.\n");
}
```

每个基本块都有`ObfProbRate`的概率被混淆，即基本块调用了`addBogusFlow`函数。

这个函数的作用就是对指定函数的每个基本块以`ObfProbRate`的概率去进行调用函数混淆。

# 目前源码：

```llvm
define dso_local i32 @main(i32 %argc, i8** %argv) #0 {
entry:
  %retval = alloca i32, align 4
  %argc.addr = alloca i32, align 4
  %argv.addr = alloca i8**, align 8
  %a = alloca i32, align 4
  store i32 0, i32* %retval, align 4
  store i32 %argc, i32* %argc.addr, align 4
  store i8** %argv, i8*** %argv.addr, align 8
  %0 = load i8**, i8*** %argv.addr, align 8
  %arrayidx = getelementptr inbounds i8*, i8** %0, i64 1
  %1 = load i8*, i8** %arrayidx, align 8
  %call = call i32 @atoi(i8* %1) #2
  store i32 %call, i32* %a, align 4
  %2 = load i32, i32* %a, align 4
  %cmp = icmp eq i32 %2, 0
  br i1 %cmp, label %if.then, label %if.else

if.then:                                          ; preds = %entry
  store i32 1, i32* %retval, align 4
  br label %return

if.else:                                          ; preds = %entry
  store i32 10, i32* %retval, align 4
  br label %return

return:                                           ; preds = %if.else, %if.then
  %3 = load i32, i32* %retval, align 4
  ret i32 %3
}
```

# `addBogusFlow`函数1

