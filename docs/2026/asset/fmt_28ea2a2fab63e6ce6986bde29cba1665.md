
# [原创]利用编译器优化干掉控制流平坦化`flatten`

上一篇在这里<https://bbs.pediy.com/thread-265335.htm利用编译器优化干掉虚假控制流>

上一篇我们通过优化去除了类似`x * (x + 1) % 2 == 0`的虚假控制流，这里的混淆去除`bcf`之后如下图，很明显的平坦化混淆

图上已经比较清晰了，黄色圈住的块是进入平坦化，离开平坦化`return`，选择控制流的`switch`混淆块，

蓝色的两块如果执行到的话就是真实块，其余基本都是`case`混淆块![](https://bbs.kanxue.com/upload/attach/202103/877073_DYXADTRSQE3TZXT.jpg)

具体算法过程如下，这里参考了大佬的思路，看到大佬又发了一篇`vmp`的，下面必须抽空继续学习

一种通过后端编译优化脱混淆壳的方法<https://bbs.pediy.com/thread-260626.htm>

0.把`ir`中所有`switch`指令转换为`cmp+br`指令

1.遍历所有`block`，如果某个`block`的`Predecessor >6`,可能是混淆节点，也就是`switch`块，标记为`ob`

2.获取`ob`的`Successor`后续块的`Terminator`指令，如果是条件跳转`cmp+br`，`getOperand(0)`就得到`switchvar`

3.根据`switchvar`这个`Value`类的实例，遍历所有使用这个值的`Users`，挑出其中指令为`cmp`且`switchvar`为`op0`的`Users`，标记为`case`分发块

4.紧随`case`分发块之后一直到`ob`混淆块或者`return`的为真实块。这里大概有这几种情况

4.1赋值->多个`case`分发块->1个或多个真实块->`ob`块->赋值->下一次循环 这种是包含了真实块的有用执行路径

4.2赋值->多个`case`分发块->赋值->`ob`块->下一次循环 这种是只改变`switchvar`的无用路径

4.3赋值->多个`case`分发块->`ob`块->下一次循环 这种在`ob`块的`phi`节点上没有赋值，是永远不可能执行的路径，在分析`ob`块的`phi`节点时可以直接删掉

5.根据所有分发块建立二叉树，根节点为`ob`节点，往下遍历`block`寻找结尾`cmp+br`指令，左节点为`true`，右为`false`，直到没有`br`指令或者`cmp`不为`switchvar`，就到达真实块开头，它正对着的混淆`phi`节点前驱块为真实块结尾

6.从混淆`phi`节点开始，根据`switchvar`的`Use`往上追踪所有赋值，遇到`phi`继续追踪直到定值，建立混淆`phi`节点前驱块与`switchvar`赋值一一对应，没有`switchvar`赋值的`phi`路径肯定不会执行

7.模拟执行，第一次`switchvar`的值在进入`switch`时赋予，也就是`ob`块的后继块的`phi`左值，其余的在到达`ob`混淆节点后由`phi`指令赋予，最后一次必然到达`return`指令，理清楚整个路径的顺序

8.把真实块通过`br`连接起来，优化，这个函数其实只有一个真实块

## 1.runOnFunction函数

``` 
    bool runOnFunction(Function &F) override {

      Function *tmp=&F;

      BasicBlock *ob=NULL;

      bool flatOrNot=hasFlatten(*tmp,&ob);

      // errs() << "混淆block:" << *ob << '\n';

      if(flatOrNot){

        Instruction *var=getSwitchVar(*ob);

        //errs() << "getSwitchVar:" << *var << '\n';

        match(*tmp,*var);

      }

      return false;

    }
```

## 2.1是否有平坦化并获得switch混淆块

遍历所有`block`，如果某个`block`的`Predecessor >6`,可能是混淆节点，也就是`switch`块，标记为`ob`

```
    bool hasFlatten(Function &F,BasicBlock **ob){

      Function *tmp=&F;

      bool flat=false;

      for (Function::iterator bb = tmp->begin(); bb != tmp->end(); ++bb) {

          BasicBlock *tmp=&*bb;        

          int numPre=0;

          pred_iterator PI = pred_begin(tmp), E = pred_end(tmp);

          for (;PI != E; ++PI) {

            numPre+=1;

          }

          if(numPre>=6){

            flat=true;

            static BasicBlock *obb=&*tmp;

            *ob=&*tmp; 

            // errs() << "混淆blockterm:" << *obb<< '\n';

          }

      }

      return flat;

    }
```

## 2.2获得switchvar

获取`ob`混淆块的`Successor`后续块的`Terminator`指令，如果是条件跳转`cmp+br`，`getOperand(0)`就得到`switchvar`

```
    Instruction* getSwitchVar(BasicBlock &ob){

        BasicBlock *tmp=&ob;

        BasicBlock *succ = tmp->getTerminator()->getSuccessor(0);

        // errs() << "混淆block succe:" << *succ << '\n';   

        BranchInst *br = NULL;

        br = cast<BranchInst>(succ->getTerminator());  

        if(br->isConditional()){  //收集所有case和phi控制点

            ICmpInst *ins=NULL;

            ins = cast<ICmpInst>(br->getOperand(0));

            Instruction *op0=NULL;

            op0 = cast<Instruction>(ins->getOperand(0));//op0就是switchvar

          return op0;

        }

        else{

          return NULL;

        }



    }
```

## 3.获得case分发块

根据`switchvar`这个`Value`类的实例，遍历所有使用这个值的`Users`，挑出其中指令为`cmp`且`switchvar`为`op0`的`Users`，标记为`case`分发块

```
      for (User *U : op0->users()) {//这里保存所有switch控制块

        if (Instruction *Inst = dyn_cast<Instruction>(U)) {

          // errs() << Inst->getParent()->getName()<< "\n";

          if(ICmpInst *Inst = dyn_cast<ICmpInst>(U)){

            inst.push_back(Inst);

            blk.push_back(Inst->getParent());

            num.push_back(dyn_cast<Constant>(Inst->getOperand(1)));

            predicate.push_back(Inst->getPredicate());

            }

          else if(PHINode *Inst = dyn_cast<PHINode>(U)){

            phi=Inst;

            // errs() <<"phi:"<< phi->getParent()<< '\n';

            }

          }

        }
```

## 4.获得真实块

紧随`case`分发块之后一直到`ob`混淆块或者`return`的为真实块

```
      for(int i=0;i<blk.size();i++){//这里从控制块往下走，直到找到真实块

          succ_iterator PI = succ_begin(blk[i]), E = succ_end(blk[i]);

          for (;PI != E; ++PI) {

            // errs() << *blk[i]<< '\n';

            // errs() << **PI<< '\n';//iterator指针指向block指针指向block

            // BasicBlock* blk_now=cast<BasicBlock*>(&(**PI));

            vector<BasicBlock*>::iterator it=find(blk.begin(), blk.end(), *PI);

            bool switch_=(*PI!=dyn_cast<BasicBlock>(phi->getParent()));

            if(switch_&&(it == blk.end())){

               blk2.push_back(*PI);

            }

            // if(it == blk.end()){

            //   if(*PI!=dyn_cast<BasicBlock>(phi->getParent())){

            //     blk2.push_back(*PI);

            //   }

            // }

          }

        }
```

## 5.获得混淆phi节点前驱块与switchvar赋值的一一对应

遍历`ob`块`phi`节点获取所有`switchvar`，这里有一些`retdec`识别错误的`ConstantExpr`我们把它替换成`Constant`

```
vector<Constant*>phinum;

        vector<BasicBlock*>blk3;

        int a=phi->getNumIncomingValues();

        for(int i=0;i<phi->getNumIncomingValues();i++){//这里根据phi节点确定每条路径的SwitchVar值

          // blk3.push_back(phi->getIncomingBlock(i));

          Value* a=phi->getIncomingValue(i);

          // errs() << *a<< '\n';

          int j=0;

          while((!isa<Constant>(a))&&(j<5)){

            j+=1;

            a=cast<Instruction>(a)->getOperand(1);

          }

          if(dyn_cast<Constant>(a)){

            phinum.push_back(dyn_cast<Constant>(a));

            blk3.push_back(phi->getIncomingBlock(i));

            }

        }

          for(int i=0;i<phinum.size();i++)

          {

            // errs() << "blk3:" <<blk3[i]->getName()<< '\n';

            if(dyn_cast<ConstantExpr>(phinum[i])){

              ConstantExpr* exp=cast<ConstantExpr>(phinum[i]);

              ConstantInt *b=(ConstantInt *)ConstantInt::get(exp->getType(),0);

              Constant* cons=exp->getWithOperandReplaced(0,b);

              // errs() << "getAsInstruction:" <<*cons<< '\n';

              phinum[i]=cons;

            }

            // errs() << "num:" <<*phinum[i]<< '\n';

          }
```

## 6.模拟执行

本来想学着用`klee`的，破单位网速太差一直下一半断掉，没办法只能自己试着写

`blk`向量存储了所有判断状态变量`switchvar`的的`block`，`blk2`向量存储了所有`ob`混淆`block`的前驱块，`excute`向量存储了所有执行过的`block`，

很明显每次执行首先获得`switchvar`，然后沿着路径根据`blk`的`br`一直执行到`blk2`也就是`ob`混淆`block`的前驱块，

通过`ob`混淆块的`phi`节点更改`switchvar`，执行下一条路径

这么执行几次后，最后一条路径到达`return`，混淆结束，是一个单进单出的大循环

遍历得到所有路径之后，直接连接所有的真实块，把`blk`和`ob`都忽略掉，就可以干掉平坦化了
