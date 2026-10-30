
```c++
void Typer::Run(const NodeVector& roots,
LoopVariableOptimizer* induction_vars) {
if (induction_vars != nullptr) {
induction_vars->ChangeToInductionVariablePhis();
}
Visitor visitor(this, induction_vars);
GraphReducer graph_reducer(zone(), graph(), tick_counter_, broker());
graph_reducer.AddReducer(&visitor);
for (Node* const root : roots) graph_reducer.ReduceNode(root);
graph_reducer.ReduceGraph();
···
induction_vars->ChangeToPhisAndInsertGuards();
}
}
```

在 `Typer::Run` 中会调用 `ReduceGraph` 尝试对结点进行缩减，它最终会根据结点的类型来确定运行的函数：

```c++
Type Typer::Visitor::JSCallTyper(Type fun, Typer* t) {
if (!fun.IsHeapConstant() || !fun.AsHeapConstant()->Ref().IsJSFunction()) {
return Type::NonInternal();
}
JSFunctionRef function = fun.AsHeapConstant()->Ref().AsJSFunction();
if (!function.shared().HasBuiltinId()) {
return Type::NonInternal();
}
switch (function.shared().builtin_id()) {
case Builtin::kMathRandom:
return Type::PlainNumber();
case Builtin::kMathFloor:
case Builtin::kMathCeil:
case Builtin::kMathRound:
case Builtin::kMathTrunc:
return t->cache_->kIntegerOrMinusZeroOrNaN;
···
```

这是一个庞大的 switch，对于那些内置函数(buildin)，它能够很快找出对应的类型；而对于一些其他类型的函数，则返回 `NonInternal`。这么做的目的是能够简化一些检查操作，既然判明了这个结点必然会是某个确定的属性，就不再需要对它的输入做其他类型的检查了。

```c++
Type Typer::Visitor::TypeNumberConstant(Node* node) {
double number = OpParameter<double>(node->op());
return Type::Constant(number, zone());
}
```

> 不过这里用到的是 `double` 类型，所以 v8 中的常数最大值肯定小于普通的八字节可表示的常数最大值。

然后再进入 `Type::Constant`：

```c++
Type Type::Constant(double value, Zone* zone) {
if (RangeType::IsInteger(value)) {
return Range(value, value, zone);
} else if (IsMinusZero(value)) {
return Type::MinusZero();
} else if (std::isnan(value)) {
return Type::NaN();
}
DCHECK(OtherNumberConstantType::IsOtherNumberConstant(value));
return OtherNumberConstant(value, zone);
}
```

对于普通整数的返回值自然就是一个 `Range` 了，另外还有两种值被使用 `-0` 和 `NaN`。

而 Speculative 前缀含有推测的意思，这往往意味着这个函数能够根据情况进一步优化，例如`SpeculativeSafeIntegerAdd` 就是如此。在优化以前，它会以这个结点表示所有的加法，而在它通过代码运行时分析，发现其执行数据符合一定的预期时，就能够用更加具体且更加快速的函数来替代了。

```c++
Type OperationTyper::SpeculativeToNumber(Type type) {
return ToNumber(Type::Intersect(type, Type::NumberOrOddball(), zone()));
}
```

`ToNumber` 会继续向下化简，最终根据我们给出的 `Range` 选择一个合适的函数替代，我们以如下的例子说明：

假如我们使用一个稍大一些的数：

```js
let opt_me = (x) => {
return x + 1000000000000;
}
opt_me(42);
for(var i=0;i<0x10000;i++)
{
opt_me(4242);
}
```

就会使用 `SpeculativeNumberAdd` 替代它：

![](upload/attach/202207/924548_Z3UNQ8HA22ERTXT.png)

而如果我们只使用一些较小的数：

```js
let opt_me= (x) => {
let y = x ? 10 : 20;
return y + 100;
}
for(var i=0;i<0x10000;i++)
{
opt_me(false);
}
```

就会生成相当简单的 `Int32Add`：

![](upload/attach/202207/924548_6V9KRQFSYEEQWBS.png)

另外，而如果需要通过索引来读取数组：

```js
function opt() {
var arr = [1.1,2.2];
var x = 1;
return arr[x];
}
for (var i=0;i<0x20000;i++) {
opt();
}
```

![](./img/chapter4/turbolizer-3.png)

有一个特殊的函数是 `CheckBounds`，它会检查输入的索引值是否越界，然后才能够返回对应的数。它的类型也是 `Range`，通过确定的上下界就能够很容易的分析出索引是否越界，因此在旧版的 V8 中会在优化后消除检查；不过，在现在的版本里，这个检查又加回来了：

![](./img/chapter4/turbolizer-4.png)

> 似乎看起来消除检查也没太大问题，因为上下界确定的情况下 Turbofan 认为绝对不可能发生越界了。
>
> 但如果在代码层面和优化层面对数值的计算不一致，优化层计算出的结果表示不会越界，而代码层的计算结果却超出了范围，那么就能够利用优化后取出检查的机制来越界读写了。
>
> 很危险，因此现在又恢复了这个检查。

总结一下目前可能产生的优化：

- `JSCall` 调用内置函数结点被 `PlainNumber` 等已知类型替代
- `NumberConstant` 以 `Range(n,n)` 表示
- `SpeculativeNumberAdd(PlainNumber, PlainNumber)` 则会以 `PlainNumber` 表示，

当然，肯定不只是这些内容，但我们没必要全部展开一一阐明，并且我相信您至少对这种替换有了一定的认识了。

但这只是初步优化，接下来还会做不同阶段的分层优化：

```c++
TypedOptimization typed_optimization(&graph_reducer, data->dependencies(),
data->jsgraph(), data->broker());
AddReducer(data, &graph_reducer, &dead_code_elimination);
AddReducer(data, &graph_reducer, &create_lowering);
AddReducer(data, &graph_reducer, &constant_folding_reducer);
AddReducer(data, &graph_reducer, &typed_lowering);
AddReducer(data, &graph_reducer, &typed_optimization);
AddReducer(data, &graph_reducer, &simple_reducer);
AddReducer(data, &graph_reducer, &checkpoint_elimination);
AddReducer(data, &graph_reducer, &common_reducer);
```

在 `TypedOptimization` 中，会调用各类 `Reduce` 函数对类型进行优化，例如上述的 `SpeculativeNumberAdd`：

```c++
Reduction TypedOptimization::ReduceSpeculativeNumberAdd(Node* node) {
Node* const lhs = NodeProperties::GetValueInput(node, 0);
Node* const rhs = NodeProperties::GetValueInput(node, 1);
Type const lhs_type = NodeProperties::GetType(lhs);
Type const rhs_type = NodeProperties::GetType(rhs);
NumberOperationHint hint = NumberOperationHintOf(node->op());
if ((hint == NumberOperationHint::kNumber ||
hint == NumberOperationHint::kNumberOrOddball) &&
BothAre(lhs_type, rhs_type, Type::PlainPrimitive()) &&
NeitherCanBe(lhs_type, rhs_type, Type::StringOrReceiver())) {
// SpeculativeNumberAdd(x:-string, y:-string) =>
// NumberAdd(ToNumber(x), ToNumber(y))
Node* const toNum_lhs = ConvertPlainPrimitiveToNumber(lhs);
Node* const toNum_rhs = ConvertPlainPrimitiveToNumber(rhs);
Node* const value =
graph()->NewNode(simplified()->NumberAdd(), toNum_lhs, toNum_rhs);
ReplaceWithValue(node, value);
return Replace(value);
}
return NoChange();
}
```

这会尝试通过 `NumberOperationHintOf` 来判别我们的表达式行为：

```c++
NumberOperationHint NumberOperationHintOf(const Operator* op) {
DCHECK(op->opcode() == IrOpcode::kSpeculativeNumberAdd ||
op->opcode() == IrOpcode::kSpeculativeNumberSubtract ||
op->opcode() == IrOpcode::kSpeculativeNumberMultiply ||
op->opcode() == IrOpcode::kSpeculativeNumberPow ||
op->opcode() == IrOpcode::kSpeculativeNumberDivide ||
op->opcode() == IrOpcode::kSpeculativeNumberModulus ||
op->opcode() == IrOpcode::kSpeculativeNumberShiftLeft ||
op->opcode() == IrOpcode::kSpeculativeNumberShiftRight ||
op->opcode() == IrOpcode::kSpeculativeNumberShiftRightLogical ||
op->opcode() == IrOpcode::kSpeculativeNumberBitwiseAnd ||
op->opcode() == IrOpcode::kSpeculativeNumberBitwiseOr ||
op->opcode() == IrOpcode::kSpeculativeNumberBitwiseXor ||
op->opcode() == IrOpcode::kSpeculativeNumberEqual ||
op->opcode() == IrOpcode::kSpeculativeNumberLessThan ||
op->opcode() == IrOpcode::kSpeculativeNumberLessThanOrEqual ||
op->opcode() == IrOpcode::kSpeculativeSafeIntegerAdd ||
op->opcode() == IrOpcode::kSpeculativeSafeIntegerSubtract);
return OpParameter<NumberOperationHint>(op);
}
```

最终它会发现，如果表达式的二值均为 `NumberOperationHint::kNumber` 这类数字而不会是字符串或其他类型，那么就能够将 `SpeculativeNumberAdd` 替换为 `NumberAdd`。

`JSTypedLowering::ReduceJSCall` 也有类似的操作，这里不再展开，读者可以自行尝试对照源代码。

# 实例分析

## GoogleCTF2018-Just In Time

惯例根据一个实际的样本来说明 Turbofan 的利用过程，理解一下这种优化在什么情况下能够被利用。首先我们从资料较多的例题开始。

题目附件给了 diff 文件，我们可以直接阅读代码来确定问题所在：

```diff
@@ -1301,6 +1302,8 @@ struct TypedLoweringPhase {
data->jsgraph()->Dead());
DeadCodeElimination dead_code_elimination(&graph_reducer, data->graph(),
data->common(), temp_zone);
+ DuplicateAdditionReducer duplicate_addition_reducer(&graph_reducer, data->graph(),
+ data->common());
···
@@ -1318,6 +1321,7 @@ struct TypedLoweringPhase {
data->js_heap_broker(), data->common(),
data->machine(), temp_zone);
AddReducer(data, &graph_reducer, &dead_code_elimination);
+ AddReducer(data, &graph_reducer, &duplicate_addition_reducer);
AddReducer(data, &graph_reducer, &create_lowering);
```

可以注意到，在最后的一系列优化中，题目添加了一个额外的优化，向上跟踪可以找到其来自于 `DuplicateAdditionReducer` 再往上找即可发现关键的漏洞代码：
