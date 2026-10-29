
```
define dso_local i32 @main(i32 %argc, i8** %argv) #0 {
entry:
%.reg2mem = alloca i32
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
%call = call i32 @atoi(i8* %1) #3
store i32 %call, i32* %a, align 4
%2 = load i32, i32* %a, align 4
store i32 %2, i32* %.reg2mem
%switchVar = alloca i32
store i32 157301900, i32* %switchVar
br label %loopEntry
loopEntry: ; preds = %entry, %loopEnd
%switchVar10 = load i32, i32* %switchVar
switch i32 %switchVar10, label %switchDefault [
i32 157301900, label %NodeBlock8
i32 241816174, label %NodeBlock
i32 1003739776, label %LeafBlock6
i32 -1519555718, label %LeafBlock
i32 -749093422, label %sw.bb
i32 1599617141, label %sw.bb2
i32 1815329037, label %sw.bb4
i32 1738940479, label %NewDefault
i32 -282945350, label %sw.default
i32 387774014, label %sw.epilog
i32 1681741611, label %if.then
i32 347219667, label %if.else
i32 -618048859, label %return
]
switchDefault: ; preds = %loopEntry
br label %loopEnd
NodeBlock8: ; preds = %entry
%Pivot9 = icmp slt i32 %2, 2
%3 = select i1 %Pivot9, i32 -1519555718, i32 241816174
store i32 %3, i32* %switchVar
br label %loopEnd
NodeBlock: ; preds = %NodeBlock8
%Pivot = icmp slt i32 %2, 3
%4 = select i1 %Pivot, i32 1599617141, i32 1003739776
store i32 %4, i32* %switchVar
br label %loopEnd
LeafBlock6: ; preds = %NodeBlock
%SwitchLeaf7 = icmp eq i32 %2, 3
%5 = select i1 %SwitchLeaf7, i32 1815329037, i32 1738940479
store i32 %5, i32* %switchVar
br label %loopEnd
LeafBlock: ; preds = %NodeBlock8
%SwitchLeaf = icmp eq i32 %2, 1
%6 = select i1 %SwitchLeaf, i32 -749093422, i32 1738940479
store i32 %6, i32* %switchVar
br label %loopEnd
sw.bb: ; preds = %LeafBlock
%call1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([5 x i8], [5 x i8]* @.str, i64 0, i64 0))
store i32 387774014, i32* %switchVar
br label %loopEnd
sw.bb2: ; preds = %NodeBlock
%call3 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([5 x i8], [5 x i8]* @.str.1, i64 0, i64 0))
br label %sw.epilog
sw.bb4: ; preds = %LeafBlock6
%call5 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([5 x i8], [5 x i8]* @.str.2, i64 0, i64 0))
store i32 387774014, i32* %switchVar
br label %loopEnd
NewDefault: ; preds = %LeafBlock6, %LeafBlock
br label %sw.default
sw.default: ; preds = %NewDefault
store i32 387774014, i32* %switchVar
br label %loopEnd
sw.epilog: ; preds = %sw.default, %sw.bb4, %sw.bb2, %sw.bb
%3 = load i32, i32* %a, align 4
%cmp = icmp eq i32 %3, 0
%8 = select i1 %cmp, i32 1681741611, i32 347219667
store i32 %8, i32* %switchVar
br label %loopEnd
if.then: ; preds = %sw.epilog
store i32 1, i32* %retval, align 4
store i32 -618048859, i32* %switchVar
br label %loopEnd
if.else: ; preds = %sw.epilog
store i32 10, i32* %retval, align 4
store i32 -618048859, i32* %switchVar
br label %loopEnd
return: ; preds = %if.else, %if.then
%4 = load i32, i32* %retval, align 4
ret i32 %4
loopEnd: ; preds = %if.else, %if.then, %sw.epilog, %sw.default, %NewDefault, %sw.bb4, %sw.bb2, %sw.bb, %LeafBlock, %LeafBlock6, %NodeBlock, %NodeBlock8, %switchDefault
br label %loopEntry
}
```

附带一句，进行控制流平坦化之前刚开始会把`switch`语句给它全部进行改为`if else`目的主要是为了进行多次平坦化做准备（进行平坦化时里面是可以填次数）

# 魔改平坦化

正常流程就是首先进入`entry`，`entry`会给一个`case`常量值，然后进入`loopentry`，`loopentry`根据这个常量值进行`switch`分跳转到`case`常量对应的基本块，基本块执行完又会赋值一个`case`常量值，跳转到`loopend`，`loopend`又会跳转到`loopentry`进行下一次分发。（感觉特征点的话就是`entry`和`loopend`都会跳转到`loopentry`）。

大部分的方法都是基于定位`loopend`基本块，这种简单的`IR`结构编译成汇编代码也是这个样子，很容易定位到`loopend`基本块。在这里，我们的魔改方法就直接在`loopend`基本块后面加一个新的`switchInst`，然后跳转到基本块，相当于从开始的`switch`转移到后面来了。即抹除原来的跳转，插入`loadInst`加载`swichvar`，再用于`swichInst`跳转。

# 尾声

网上很多很多现成资料，我这个也是自己看了再总结的，如果哪里不对的话，还请多多指教。

目录

- [前置知识](https://bbs.kanxue.com/#前置知识)
- [Flattening::flatten(Function *f)](https://bbs.kanxue.com/#flatteningflattenfunction-f)
- [源代码](https://bbs.kanxue.com/#源代码)
- [目前代码](https://bbs.kanxue.com/#目前代码)
- [创建switch](https://bbs.kanxue.com/#创建switch)
- [目前代码：](https://bbs.kanxue.com/#目前代码：)
- [创建case](https://bbs.kanxue.com/#创建case)
- [目前代码](https://bbs.kanxue.com/#目前代码-1)
- [枚举更改各个case block块](https://bbs.kanxue.com/#枚举更改各个case-block块)
- [return block](https://bbs.kanxue.com/#return-block)
- [非条件跳转block](https://bbs.kanxue.com/#非条件跳转block)
- [条件跳转 block](https://bbs.kanxue.com/#条件跳转-block)
- [目前代码](https://bbs.kanxue.com/#目前代码-2)
- [魔改平坦化](https://bbs.kanxue.com/#魔改平坦化)
- [尾声](https://bbs.kanxue.com/#尾声)

# 前置知识

`Module`是指模块，`Function`模块下的函数，`BasicBlock`函数下的基本块，`Instruction` 基本块下的`IR`指令

# `Flattening::flatten(Function *f)`

```
for (Function::iterator i = f->begin(); i != f->end(); ++i) {
    BasicBlock *tmp = &*i;
    origBB.push_back(tmp);

    BasicBlock *bb = &*i;
    if (isa[previous internal execution text redacted]>(bb->getTerminator())) {
        return false;
    }
}
```

把函数分成很多个基本块，并且push到`vector`类型的`origBB`中。

```
if (origBB.size() <= 1) {
    return false;
}
```

判断里面基本块是否大于1，不大于1的话就没有意义去进行混淆：

```
origBB.erase(origBB.begin());
// Get a pointer on the first BB
Function::iterator tmp = f->begin();
// ++tmp;
BasicBlock *insert = &*tmp;

// If main begin with an if
BranchInst *br = NULL;
if (isa<BranchInst>(insert->getTerminator())) {
    br = cast<BranchInst>(insert->getTerminator());
}

if ((br != NULL && br->isConditional()) || insert->getTerminator()->getNumSuccessors() > 1) {
    BasicBlock::iterator i = insert->end();
    --i;

    if (insert->size() > 1) {
        --i;
    }

    BasicBlock *tmpBB = insert->splitBasicBlock(i, "first");
    origBB.insert(origBB.begin(), tmpBB);
}
```

如果是条件跳转的话这里是把上面自动添加那个跳转指令给删除，如果不是的话，那么也是需要把它删除，因为跳转点目标还不能确定：

```
// Remove jump
insert->getTerminator()->eraseFromParent();
```

# 源代码

```
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
%call = call i32 @atoi(i8* %1) #3
store i32 %call, i32* %a, align 4
%2 = load i32, i32* %a, align 4
br label %NodeBlock8
```

# 目前代码
