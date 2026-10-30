
```
// Mutate()进行变异操作，跟上面提到的MutateWithHints()完全不一样。MutateWithHints()是对能匹配的 // 的值进行替换，这里的变异和AFL的变异操作基本类似。
func (p *Prog) Mutate(rs rand.Source, ncalls int, ct *ChoiceTable, corpus []*Prog) {
    // 初始化，省略掉了
    ...
    // 所有变异都不是一定发生，都设置了一定的概率。
    for stop, ok := false, false; !stop; stop = ok && len(p.Calls) != 0 && r.oneOf(3) {
        switch {
        case r.oneOf(5):
            // Not all calls have anything squashable,
            // so this has lower priority in reality.

            /* 原注释如下：
            // Picks a random complex pointer and squashes its arguments into an ANY.
            // Subsequently, if the ANY contains blobs, mutates a random blob.
            选择一个随机的复杂的指针，把它的参数压缩进ANY。然后，如果ANY包含blob对象，变异一个随机的blob。

            文中给出下面的例子，这是原始的prog：
            foo$any0(&(0x7f0000000000)={0x11, 0x11223344, 0x2233, 0x1122334455667788, {0x1, 0x7, 0x1, 0x1, 0x1bc, 0x4}, [{@res32=0x0, @i8=0x44, "aabb"}, {@res64=0x1, @i32=0x11223344, "1122334455667788"}]})

            下面就是经过squashed得到的程序：
            foo$any0(&(0x7f0000000000)=ANY=[@ANYBLOB="1100000044332211223300000000000088776655443322117d00bc11", @ANYRES32=0x0, @ANYBLOB="0000000044aabb00", @ANYRES64=0x1, @ANYBLOB="44332211112233445566778800000000"])

            在进行squash时，会调用mutateData()对数据进行变异，所涉及的规则在syzkaller/prog/mutations.go的mutateDataFuncs数组中，包括翻转字节、插入随机的字节、移除字节、添加一些字节、替换随机的字节、加减一个int8/int16/int32/int64、把int8/int16/int32/int64设置为interesting的值。
            */
            ok = ctx.squashAny()
        case r.nOutOf(1, 100):
            /* 原注释如下：
            // This function selects a random other program p0 out of the corpus, and
            // mutates ctx.p as follows: preserve ctx.p's Calls up to a random index i
            // (exclusive) concatenated with p0's calls from index i (inclusive).
            这个方法随机选择一个语料库外的程序p0，并对程序ctx.p进行变异。具体操作如下：选一个随机数i，保留源程序ctx.p的前i个call（不包括第i个），把p0的call从第i个位置往后接（包括i）;
            换句话说就是把p0的所有call从i的位置往后插，画了下面的图，将就看一下。

            ctx.p[0]       ctx.p[i] p0[0] <--插在这里--> p0结束 ctx.p[i+1]
            |------------------|---------------------|---------------------|

            如果call的总长度超过了总长度ncalls，就从最后往前删除，反正后面是ctx.p的call。
            */
            ok = ctx.splice()
        case r.nOutOf(20, 31):
            /* 原注释如下：
            // Inserts a new call at a randomly chosen point (with bias towards the end of
            // existing program). Does not insert a call if program already has ncalls.
            在随机位置插入一个新的call。如果call的数目达到最大值ncalls就不要插入了。
            在选取随机位置时调用了函数biasedRand(n, k int)，此函数会生成一个随机数[0..n)，但是获得n-1的概率是0的概率的k倍。
            这个方法跟上面的方法不一样的是：1、这个是插入一个call，splice()是插入一个prog的所有calls；
            2、这个插入的call是新生成的call，splice()插入的是已经存在的prog的calls。
            */
            ok = ctx.insertCall()
        case r.nOutOf(10, 11):
            // Mutate an argument of a random call.
            // 对一个随机call的参数进行变异。
            // 先判断参数的类型，对不同类型的参数使用不同的变异策略。
            ok = ctx.mutateArg()
        default:
            // Removes a random call from program.
            // 在program中随机移除一个call。
            ok = ctx.removeCall()
        }
    }

    ...
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-6-26 20:58 被wx_左撇子编辑，原因：

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)
[#自动化挖掘](https://bbs.kanxue.com/forum-150-1-155.htm)
[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)

上传的附件：

- [`myFuzz2.md`](javascript:void(0);) （15.22kb，16次下载）
