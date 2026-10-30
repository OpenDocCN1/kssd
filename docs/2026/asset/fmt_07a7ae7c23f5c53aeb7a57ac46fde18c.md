

# [原创]syzkaller源码分析（二） `syz-fuzzer.go`

这一篇主要来说`syz.fuzzer`的内容，它主要负责fuzz的部分，但是由于篇幅原因，`Mutate()`和`Generate()`以及`executeHintSeed()`会放到下一篇中。

写的目的为了删繁就简，只列举了比较重要的函数和主要流程，一些具体的实现要结合源码看。源码的地址：[101K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6K6P5i4A6C8j5h3I4D9k6i4t1](<elink@710K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6K6P5i4A6C8j5h3I4D9k6i4t1>)。

`syz-manager.go`的解析在 [https://bbs.pediy.com/thread-268152.htm](https://bbs.pediy.com/thread-268152.htm)

参考：[cf0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5#2x3U0t1K6](<elink@2ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5#2x3U0t1K6>)。

md文件会附在最后。

## 重点函数 `poll(needCandidates, nil)`

```go
//更新corpus以及candidate队列
func (fuzzer *Fuzzer) poll(needCandidates bool, stats map[string]uint64) bool {
//启动Manager.Poll，不过没找到对应的代码在哪里，猜测可能类似linux的poll()，用来处理队列信息
fuzzer.manager.Call("Manager.Poll", a, r)
//用来获得最大的信号量
//调用了fuzzer.maxSignal.Merge(sign)实现此功能：对已经存在的sign比较优先级，对没有的sign直接添加
fuzzer.addMaxSignal(maxSignal)
for _, inp := range r.NewInputs {
//更新corpusSignal和maxSignal
//主要调用了fuzzer.addInputToCorpus(p, sign, sig)，这个函数又调用了Merge(sign)
fuzzer.addInputFromAnotherFuzzer(inp)
}
for _, candidate := range r.Candidates {
//使candidate进入队列workQueue
//先反序列化获得Prog，再根据candidate.Minimized、candidate.Smashed的值对flags对应的位进行置位
fuzzer.addCandidateInput(candidate)
}
//如果需要Candidates，并且Candidates长度为0，并且triagedCandidates==0，就把triagedCandidates置为1
if needCandidates && len(r.Candidates) == 0 && atomic.LoadUint32(&fuzzer.triagedCandidates) == 0 {
atomic.StoreUint32(&fuzzer.triagedCandidates, 1)
}
//NewInputs、Candidates、maxSignal有一个不为空，就返回true
return len(r.NewInputs) != 0 || len(r.Candidates) != 0 || maxSignal.Len() != 0
}
```

## 重点函数 `BuildChoiceTable(fuzzer.corpus, calls)`

```go
//用来获取prios[X][Y]的优先级，prios[X][Y]表示是否在一个包含X的program中插入一个call Y会出现新的覆盖率
/* 函数的返回值是*ChoiceTable，作者对ChoiceTable的原注释如下:
ChooseTable allows to do a weighted choice of a syscall for a given syscall
based on call-to-call priorities and a set of enabled syscalls.
call-to-call priorities就是上面解释的prios[X][Y]；
enabled syscalls 是指的在运行的架构系统中可以使用的syscall，代码对disable syscall进行了筛选*/
func (target *Target) BuildChoiceTable(corpus []*Prog, enabled map[*Syscall]bool) *ChoiceTable {
//如果enabled == nil，把target.Syscalls依次赋值给enabled，并置为true
if enabled == nil {
enabled = make(map[*Syscall]bool)
for _, c := range target.Syscalls {
enabled[c] = true
}
}
//删除enabled是不可用(Disabled)的元素
for call := range enabled {
if call.Attrs.Disabled {
delete(enabled, call)
}
}
//把enabled数组赋值到enabledCalls（省略部分一）；并按ID大小进行排序；省略部分二对corpus再进行检查，看有没有不可用的disabled syscall
...(省略部分一)
sort.Slice(enabledCalls, func(i, j int) bool {
return enabledCalls[i].ID < enabledCalls[j].ID
})
...(省略部分二)
//******************计算优先级******************
//重点函数，下面会提
prios := target.CalculatePriorities(corpus)
把优先级赋值到run[][]的每一个元素
//返回&ChoiceTable
return &ChoiceTable{target, run, enabledCalls}
}
```

## 重点函数 `CalculatePriorities(corpus []*Prog)`

在`BuildChoiceTable()`中被调用

```go
//用来计算优先级
//先把函数的原始注释粘上
// Calulation of call-to-call priorities.
// For a given pair of calls X and Y, the priority is our guess as to whether
// additional of call Y into a program containing call X is likely to give
// new coverage or not.
// The current algorithm has two components: static and dynamic.
// The static component is based on analysis of argument types. For example,
// if call X and call Y both accept fd[sock], then they are more likely to give
// new coverage together.
// The dynamic component is based on frequency of occurrence of a particular
// pair of syscalls in a single program in corpus. For example, if socket and
// connect frequently occur in programs together, we give higher priority to
// this pair of syscalls.
// Note: the current implementation is very basic, there is no theory behind any
// constants.
/*
对上面内容做一下大致的解释：
这个函数用来计算call-to-call的优先级。
call-to-call的优先级用prios[X][Y]来表示，对于一对给定的calls X和Y，优先级是指
在一个包含X的program中，加入call Y后能不能生成新的覆盖情况。
这个算法有两个部分：静态部分和动态部分，也调用了两个对应的函数，
calcStaticPriorities()和calcDynamicPrio(corpus)。
静态部分calcStaticPriorities()是对参数类型进行分析。举个例子，如果call X和call Y
都接受参数fd[sock]，把它们放在一起就更有可能出现新的覆盖情况。
动态部分calcDynamicPrio(corpus)是指在语料库中的单个程序中一对特定的syscalls出现的
频率。举个例子，如果socket和connect在programs中经常出现，就把这一对syscalls给予更高的优
先级。
*/
func (target *Target) CalculatePriorities(corpus []*Prog) [][]int32 {
/*
calcStaticPriorities()通过静态的参数分析设置优先级。
具体实现一定要结合代码一起看，这里为了留核心的东西就不再另贴一个函数了。
代码在syzkaller/prog/prio.go 41行左右。
第一步，调用calcStaticPriorities()对不同的Type设置不同的权值weight，返回uses map[string]map[int]weights；
对uses的weights进行双重遍历，跳过自身（在第三步单独处理），设置prios的值，值为:
prios[w0.call][w1.call] += w0.inout*w1.in*3/2 + w0.inout*w1.inout
对这个值源码也有相关的注释，优先级的值是基于参数方向的，c0产生资源、c1来使用会有更高的优先级：
// The static priority is assigned based on the direction of arguments. A higher priority will be
// assigned when c0 is a call that produces a resource and c1 a call that uses that resource.
第二步，调用normalizePrio(prios)对prios进行normalize处理，使优先级的值落在区间[prioLow,prioHigh]内。默认10-1000
第三步，把prios[c0][c0]这种自己调用自己的情况赋予一个较高的优先级，但也不能太高（prioHigh * 9 / 10）。
*/
static := target.calcStaticPriorities()
if len(corpus) != 0 {
//calcDynamicPrio(corpus)用来动态处理优先级，这个就简单一点了。
//先统计在每个prog中两个call一起出现的次数，再进行normalize就可以了。
dynamic := target.calcDynamicPrio(corpus)
for i, prios := range dynamic {
dst := static[i]
for j, p := range prios {
//把静态和动态得到的优先级统计到一起。
dst[j] = dst[j] * p / prioHigh
}
}
}
return static
}
```

## 重点函数 `loop()`

