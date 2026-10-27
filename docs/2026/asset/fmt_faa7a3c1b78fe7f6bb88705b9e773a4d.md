
```cpp
Function *VariableRotation::createRotateFunc(Module *m, PointerType *ptrType, Twine &name) {
    std::vector<Type *> params;
    params.push_back(ptrType);
    params.push_back(Type::getInt32Ty(m->getContext()));
    params.push_back(Type::getInt32Ty(m->getContext()));
    Type *rawType = ptrType->getPointerElementType();
    FunctionType *funcType = FunctionType::get(Type::getVoidTy(m->getContext()), params, false);
    Function *func = Function::Create(funcType, GlobalValue::PrivateLinkage, name, m);
    BasicBlock *entry1 = BasicBlock::Create(m->getContext(), "entry1", func);
    BasicBlock *cmp1 = BasicBlock::Create(m->getContext(), "cmp1", func);
    BasicBlock *entry2 = BasicBlock::Create(m->getContext(), "entry2", func);
    BasicBlock *cmp2 = BasicBlock::Create(m->getContext(), "cmp2", func);
    BasicBlock *shift = BasicBlock::Create(m->getContext(), "shift", func);
    BasicBlock *end2 = BasicBlock::Create(m->getContext(), "end2", func);
    BasicBlock *end1 = BasicBlock::Create(m->getContext(), "end1", func);
    Function::arg_iterator iter = func->arg_begin();
    Value *ptr = iter;
    Value *len = ++iter;
    Value *times = ++iter;
    IRBuilder<> irb(entry1);
    Value *zero = ConstantInt::get(irb.getInt32Ty(), 0);
    Value *one = ConstantInt::get(irb.getInt32Ty(), 1);
    AllocaInst *i = irb.CreateAlloca(irb.getInt32Ty());
    AllocaInst *j = irb.CreateAlloca(irb.getInt32Ty());
    AllocaInst *tmp = irb.CreateAlloca(rawType);
    AllocaInst *array = irb.CreateAlloca(ptrType);
    irb.CreateStore(zero, j);
    irb.CreateStore(ptr, array);
    irb.CreateBr(cmp1);

    irb.SetInsertPoint(cmp1);
    irb.CreateCondBr(irb.CreateICmpSLT(irb.CreateLoad(j), times), entry2, end1);

    irb.SetInsertPoint(entry2);
    irb.CreateStore(zero, i);
    irb.CreateStore(irb.CreateLoad(irb.CreateInBoundsGEP(irb.CreateLoad(array), {zero})), tmp);
    irb.CreateBr(cmp2);

    irb.SetInsertPoint(cmp2);
    irb.CreateCondBr(irb.CreateICmpSLT(irb.CreateLoad(i), irb.CreateSub(len, one)), shift, end2);

    irb.SetInsertPoint(shift);
    Value *ival = irb.CreateLoad(i);
    Value *arr = irb.CreateLoad(array);
    irb.CreateStore(irb.CreateLoad(irb.CreateInBoundsGEP(arr, {irb.CreateAdd(ival, one)})), irb.CreateInBoundsGEP(rawType, arr, {ival}));
    irb.CreateStore(irb.CreateAdd(ival, one), i);
    irb.CreateBr(cmp2);

    irb.SetInsertPoint(end2);
    irb.CreateStore(irb.CreateAdd(irb.CreateLoad(j), one), j);
    irb.CreateStore(irb.CreateLoad(tmp), irb.CreateInBoundsGEP(irb.CreateLoad(array), {irb.CreateSub(len, one)}));
    irb.CreateBr(cmp1);

    irb.SetInsertPoint(end1);
    irb.CreateRetVoid();
    return func;
}
```

这里并不是重点，熟悉llvm的ir的很快就能写出来。

### 处理函数的AllocaInst

- 函数第一个参数代表着要处理的llvm函数，第二个代表着统计出的函数的`AllocaInst`集合，第三个代表之前生成的移位函数。

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_H65TT7R8HWGSZTQ.png)

- 然后开始处理，先遍历`AllocaInst`，然后使用`getTypeAllocSize`分别获取该指令分配的空间的大小，用于计算`value_map`，其实就是表示这`AllocaInst`到buffer的偏移地址的一个映射。

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_GV5CSD696WEBD9F.png)

- 计算完成各个变量的偏移地址后，同时也得到了buffer应该有的大小，保存下来，并且使用`AllocaInst`分配空间。

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_YATD3GBRT6ANTKX.png)

- 开始遍历每个基本块。

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_2ERSUUQUXXQXF6C.png)

- 对于每个基本块，开始进行处理每个引用了原来变量的指令，替换成`GEP` buffer生成的指针，并将指针类型转换成原`AllocaInst`的指针类型。然后在基本块内部维护一个`int`，保存到当前指令的时候，已经轮转了多少位了，然后通过随机数进行插入轮转函数，更新`int`。

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_NC7FPQ7WY532NGH.png)

- 处理完基本块之后，查看轮转了多少位，如果不是0的话则需要修正一下，让轮转的位数为0(在取模意义下)

  ![](https://bbs.kanxue.com/upload/attach/202203/948449_N8C44NZXT5H5FZH.png)

### 完整代码
