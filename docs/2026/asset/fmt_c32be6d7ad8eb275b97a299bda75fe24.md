
## `addBogusFlow`函数

`copy`后变量名字进行改动。

### 修改跳转指令

```cpp
alteredBB->getTerminator()->eraseFromParent();
basicBlock->getTerminator()->eraseFromParent();
```

这里的话是指把`entry`里面的跳转和拷贝出来的`block`块最后一个跳转也删掉：

```
entry：
originalBBalteredBB:                              ; preds = %originalBB, %entry
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
```

### 生成浮点数比较

```cpp
Value *LHS = ConstantFP::get(Type::getFloatTy(F.getContext()), 1.0);
Value *RHS = ConstantFP::get(Type::getFloatTy(F.getContext()), 1.0);
Twine *var4 = new Twine("condition");
FCmpInst *condition = new FCmpInst(*basicBlock, FCmpInst::FCMP_TRUE , LHS, RHS, *var4);
```

在`entry`里面生成两个浮点数，并进行两个浮点数的比较跳转指令：

```llvm
entry:
  %condition=fcmp true float 1.00000e+00,1.00000e+00
  br i1 %7, label %originalBB, label %originalBBalteredBB
```

`fcmp`后面条件为`true`，它只会一直跳转为前者`originalBB`。

```cpp
BranchInst::Create(originalBB, alteredBB, (Value *)condition, basicBlock);
```

在`originalBBalteredBB`生成一个跳转指令，跳转到`originalBB`:

```llvm
originalBBalteredBB:                              ; preds = %originalBB, %entry
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

### 分割`originalBB`

```cpp
BasicBlock::iterator i = originalBB->end();
// Split at this point (we only want the terminator in the second part)
Twine *var5 = new Twine("originalBBpart2");
BasicBlock *originalBBpart2 = originalBB->splitBasicBlock(--i , *var5);
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
Twine *var6 = new Twine("condition2");
FCmpInst *condition2 = new FCmpInst(*originalBB, CmpInst::FCMP_TRUE , LHS, RHS, *var6);
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

这个函数主要就是创建了`entry`和`originalBB`代码块的最后两行浮点数比较的跳转。

## 尾声

网上很多很多现成资料，我这个也是自己看了再总结的，如果哪里不对的话，还请多多指教。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-8-1 00:12 被寻梦之璐编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
