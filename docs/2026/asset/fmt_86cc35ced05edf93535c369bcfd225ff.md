
## 问题分析

```
0x00401013:8:   s0xfffffff4(0x00401013:8) = #0x1
0x00401024:11:  EAX(0x00401024:11) = s0xfffffff4(0x00401013:8)
```

转换成了

```
0x00401024:11:  EAX(0x00401024:11) = #0x1
```

分析到这里，Ghidra反编译失败的原因就很明显了，在将`*(ram,u0x00002880(0x00401019:e))`识别成堆栈变量`s0xfffffff4`之前就进行了传播优化。

## 解决方案

那么怎么解决这个问题呢，Ghidra的`oppool1`老实说我感觉就像是一个大杂烩，啥规则都往里面扔，里面的优化逻辑就是一直优化，优化到pcode没有发生变化为止，然而优化顺序似乎不怎么能保证，因此我决定自己编写规则。

模仿`oppool2`中的`RuleStoreVarnode`规则，这个规则是尝试将store节点转换为stack变量，我们也写一个`RuleVmpStoreVarnode`规则，在每次遇到`CPUI_STORE`类型的pcode的时候，就尝试去计算写入的地址是否是堆栈。规则代码如下:

```
class RuleVmpStoreVarnode : public Rule {
public:
    RuleVmpStoreVarnode(const string& g) : Rule(g, 0, "vmpstorevarnode") {}
    ///< Constructor
    virtual Rule* clone(const ActionGroupList& grouplist) const {
        if (!grouplist.contains(getGroup())) return (Rule*)0;
        return new RuleVmpStoreVarnode(getGroup());
    }
    virtual void getOpList(vector<uint4>& oplist) const;
    virtual int4 applyOp(PcodeOp* op, Funcdata& data);
};

void RuleVmpStoreVarnode::getOpList(vector<uint4>& oplist) const
{
    oplist.push_back(CPUI_STORE);
}

int4 RuleVmpStoreVarnode::applyOp(PcodeOp* op, Funcdata& data)
{
    VmpStackEvaluator evalCall;
    Varnode* offvn = op->getIn(1);
    int stackOffset = 0x0;
    if (!evalCall.EvaluateStackOffset(data, offvn, stackOffset)) {
        return 0;
    }
    int4 size = op->getIn(2)->getSize();
    Address addr(data.getArch()->getStackSpace(), uint32_t(stackOffset));
    data.newVarnodeOut(size, addr, op);
    op->getOut()->setStackStore();
    // Mark as originally coming from CPUI_STORE
    data.opRemoveInput(op, 1);
    data.opRemoveInput(op, 0);
    data.opSetOpcode(op, CPUI_COPY);
    return 1;
}
```

为了计算堆栈的偏移，写了一个类`VmpStackEvaluator`，核心思路就是不断遍历def节点，判断最终是否为`esp + xxx`这种偏移。

```
class VmpStackEvaluator
{
public:
    bool EvaluateStackOffset(Funcdata& data,Varnode* vn,int& outOffset);
private:
    uintb traceVarnodeStack(Varnode* vn);
    uintb tracePcodeStack(PcodeOp* op);
    uintb eval(PcodeOp* op);
private:
    bool bContainEsp = false;
    bool bError = false;
    VarnodeData espLoc;
};

uintb VmpStackEvaluator::eval(PcodeOp* op)
{
    if (bError) {
        return 0x0;
    }
    uintb val1 = traceVarnodeStack(op->getIn(0));
    uintb val2 = traceVarnodeStack(op->getIn(1));
    uintb calVal = op->getOpcode()->evaluateBinary(op->getOut()->getSize(), op->getIn(0)->getSize(), val1, val2);
    return calVal;
}

uintb VmpStackEvaluator::tracePcodeStack(PcodeOp* op)
{
    if (bError) {
        return 0x0;
    }
    OpCode code = op->code();
    switch (code) {
    case CPUI_INT_ADD:
    case CPUI_INT_SUB:
    case CPUI_PIECE:
    case CPUI_SUBPIECE:
        return eval(op);
    case CPUI_COPY:
        return traceVarnodeStack(op->getIn(0));
    default:
        bError = true;
        break;
    }
    return 0x0;
}

uintb VmpStackEvaluator::traceVarnodeStack(Varnode* vn)
{
    if (bError) {
        return 0x0;
    }
    if (vn->isConstant()) {
        return vn->getOffset();
    }
    if (vn->isInput()) {
        if (vn->getSpace() == espLoc.space && vn->getOffset() == espLoc.offset) {
            bContainEsp = true;
        }
        else {
            bError = true;
        }
        return 0x0;
    }
    PcodeOp* defOp = vn->getDef();
    if (!defOp) {
        bError = true;
        return 0x0;
    }
    return tracePcodeStack(defOp);
}

bool VmpStackEvaluator::EvaluateStackOffset(Funcdata& data, Varnode* vn,int& outOffset)
{
    espLoc = data.getArch()->translate->getRegister("ESP");
    if (!vn->isWritten()) {
        return false;
    }
    PcodeOp* defOp = vn->getDef();
    if (defOp->code() != CPUI_INT_ADD && defOp->code() != CPUI_INT_SUB) {
        return false;
    }
    outOffset = tracePcodeStack(defOp);
    if (bError) {
        return false;
    }
    return bContainEsp;
}
```

不管效率怎么样，往`oppool1`里面扔进去这个规则，执行后最终的代码变成了:

```
0x00401024:11:  EAX(0x00401024:11) = #0x2
0x00401025:15: return(EAX(0x00401024:11))
```

也算是勉强完成了修复吧

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-9-5 11:13 被fjqisba编辑，原因：
