
通常我们不是从 `PreservedSet` 中移除分析结果，而是声明一些被保留的分析。例如，你知道你的 pass 在函数内不会修改控制流（control flow graph）和循环信息。如下面的代码：

```
#include "llvm/Analysis/LoopInfo.h"

struct MyNewPass : public PassInfoMixin<MyNewPass> {
  PreservedAnalyses run(Function &F, FunctionAnalysisManager &FAM) {
    // ...
    PreservedAnalyses PA = PreservedAnalyses::none();
    PA.preserve<LoopAnalysis>();
    PA.preserveSet<CFGAnalyses>();
    return PA;
  }
};
```

`preserve<...>` 方法声明了单个 analysis 的保留集（通过向其模板类型中传入 analysis 类型）。而 `preserveSet<...>` 有些区别，它会保留一组 analyse，你需要传入一个 analysis set 的类型（不同于 `Pass` 的概念）。有很多可用的 analysis set 的类型，比如此处的 `CFGAnalyses` 表示所有的控制流相关的 analyse。

**PS：**

还有一些重要话题没有讨论，例如：

- 如何写一个 analysis pass
- 如何查询一个 analysis 是否被保留

或许你可以从源码中找到答案 :-)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-2-14 20:08 被微笑明天编辑，原因：
