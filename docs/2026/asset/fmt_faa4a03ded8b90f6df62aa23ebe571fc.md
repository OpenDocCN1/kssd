
### BuildChoiceTable 相关说明

`//` 用来获取 `prios[X][Y]` 的优先级，`prios[X][Y]` 表示是否在一个包含 `X` 的 `program` 中插入一个 `call Y` 会出现新的覆盖率
`/*` 函数的返回值是 `ChoiceTable`，作者对 `ChoiceTable` 的原注释如下:
`ChooseTable allows to do a weighted choice of a syscall`
`for` `a given syscall`
`based on call-to-call priorities`
`and` `a` `set` `of enabled syscalls.`
`call-to-call priorities` 就是上面解释的 `prios[X][Y]`；
`enabled syscalls` 是指的在运行的架构系统中可以使用的 `syscall`，代码对 `disable syscall` 进行了筛选 `*/`
```go
func (target *Target) BuildChoiceTable(corpus []*Prog, enabled map[*Syscall]bool) *ChoiceTable {
    // 如果 enabled == nil，把 target.Syscalls 依次赋值给 enabled，并置为 true
    if enabled == nil {
        enabled = make(map[*Syscall]bool)
        for _, c := range target.Syscalls {
            enabled[c] = true
        }
    }
    // 删除 enabled 是不可用(Disabled)的元素
    for call := range enabled {
        if call.Attrs.Disabled {
            delete(enabled, call)
        }
    }
    // 把 enabled 数组赋值到 enabledCalls（省略部分一）；并按 ID 大小进行排序；省略部分二对 corpus 再进行检查，看有没有不可用的 disabled syscall
    ...(省略部分一)
    sort.Slice(enabledCalls, func(i, j int) bool {
        return enabledCalls[i].ID < enabledCalls[j].ID
    })
    ...(省略部分二)
    // *************** 计算优先级 ***************
    // 重点函数，下面会提
    prios := target.CalculatePriorities(corpus)
    // 把优先级赋值到 run[][] 的每一个元素
    // 返回 &ChoiceTable
    return &ChoiceTable{target, run, enabledCalls}
}
```

**重点函数 `CalculatePriorities(corpus []*Prog)`**

在 `BuildChoiceTable()` 中被调用

### CalculatePriorities 相关说明

`//` 用来计算优先级
`//` 先把函数的原始注释粘上
`/` `Calulation of call-to-call priorities.`
`/` `For a given pair of calls X and Y, the priority is our guess as to whether`
`/` `additional of call Y into a program containing call X is likely to give`
`/` `new coverage or not.`
`/` `The current algorithm has two components: static and dynamic.`
`/` `The static component is based on analysis of argument types. For example,`
`/` `if call X and call Y both accept fd[sock], then they are more likely to give`
`/` `new coverage together.`
`/` `The dynamic component is based on frequency of occurrence of a particular`
`/` `pair of syscalls in a single program in corpus. For example, if socket and`
`/` `connect frequently occur in programs together, we give higher priority to`
`/` `this pair of syscalls.`
`/` `Note: the current implementation is very basic, there is no theory behind any`
`/` `constants.`
`*` 对上面内容做一下大致的解释：
这个函数用来计算 `call-to-call` 的优先级。
`call-to-call` 的优先级用 `prios[X][Y]` 来表示，对于一对给定的 `calls X` 和 `Y`，优先级是指
在一个包含 `X` 的 `program` 中，加入 `call Y` 后能不能生成新的覆盖情况。
这个算法有两个部分：静态部分和动态部分，也调用了两个对应的函数，
`calcStaticPriorities()` 和 `calcDynamicPrio(corpus)`。
静态部分 `calcStaticPriorities()` 是对参数类型进行分析。举个例子，如果 `call X` 和 `call Y`
都接受参数 `fd[sock]`，把它们放在一起就更有可能出现新的覆盖情况。
动态部分 `calcDynamicPrio(corpus)` 是指在语料库中的单个程序中一对特定的 `syscalls` 出现的
频率。举个例子，如果 `socket` 和 `connect` 在 `programs` 中经常出现，就把这一对 `syscalls` 给予更高的优
先级。
`*/`
```go
func (target *Target) CalculatePriorities(corpus []*Prog) [][]int32 {
    /*
    calcStaticPriorities() 通过静态的参数分析设置优先级。
    具体实现一定要结合代码一起看，这里为了留核心的东西就不再另贴一个函数了。
    代码在 syzkaller/prog/prio.go 41 行左右。
    第一步，调用 calcStaticPriorities() 对不同的 Type 设置不同的权值 weight，返回 uses map[string]map[int]weights；
    对 uses 的 weights 进行双重遍历，跳过自身（在第三步单独处理），设置 prios 的值，值为:
    prios[w0.call][w1.call] += w0.inout*w1.in*w3/2 + w0.inout*w1.inout
    对这个值源码也有相关的注释，优先级的值是基于参数方向的，c0 产生资源、c1 来使用会有更高的优先级：
    // The static priority is assigned based on the direction of arguments. A higher priority will be
    // assigned when c0 is a call that produces a resource and c1 a call that uses that resource.
    第二步，调用 normalizePrio(prios) 对 prios 进行 normalize 处理，使优先级的值落在区间 [prioLow,prioHigh] 内。默认 10-1000
    第三步，把 prios[c0][c0] 这种自己调用自己的情况赋予一个较高的优先级，但也不能太高（prioHigh * 9 / 10）。
    */
    static := target.calcStaticPriorities()
    if len(corpus) != 0 {
        // calcDynamicPrio(corpus) 用来动态处理优先级，这个就简单一点了。
        // 先统计在每个 prog 中两个 call 一起出现的次数，再进行 normalize 就可以了。
        dynamic := target.calcDynamicPrio(corpus)
        for i, prios := range dynamic {
            dst := static[i]
            for j, p := range prios {
                // 把静态和动态得到的优先级统计到一起。
                dst[j] = dst[j] * p / prioHigh
            }
        }
    }
    return static
}
```

**重点函数 `loop()`**
