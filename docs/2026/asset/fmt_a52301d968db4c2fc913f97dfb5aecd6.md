
# [原创]DynamoRIO源码分析(二)--基本块(Basic Blocks)和跟踪 (trace)

## 前提回顾

上篇我们分析到劫持了目标程序，进行了一系列初始化并且注册了收集覆盖率信息的回调函数，最后以一个干净的堆栈调用`d_r_dispatch`。现在我们还没有运行目标程序的代码，本章将讲述DynamoRIO如何运行目标程序代码。

## 概述

下图演示了DynamoRIO的高级设计。DynamoRIO通过将应用程序代码复制到代码缓存中来执行目标应用程序，每次复制一个基本块。代码缓存是通过从DynamoRIO的调度状态到应用程序的调度状态的上下文切换进入的。

![图片描述](upload/attach/202304/949068_PUUWJJTPQ88W2UC.png)

## d_r_dispatch

现在我们开始，由于我在上篇分析到`d_r_dispatch`的时候打了快照，现在恢复快照：

请注意`dcontext->next_tag`为目标进程主线程的`EIP(RtlUserThreadStart)`

![图片描述](upload/attach/202304/949068_3GHCU2G89D8B9NC.png)

现在我们来分析`d_r_dispatch`，同样的，我们只保留关键的操作，简化后的函数如下：

```
void
d_r_dispatch(dcontext_t *dcontext)
{
fragment_t *targetf;
fragment_t coarse_f;
dispatch_enter_dynamorio(dcontext);
targetf = fragment_lookup_fine_and_coarse(dcontext, dcontext->next_tag, &coarse_f,
dcontext->last_exit);
do {
if (targetf != NULL) {
targetf = monitor_cache_enter(dcontext, targetf);
}
if (targetf != NULL)
break;
SHARED_BB_LOCK();
if (USE_BB_BUILDING_LOCK() || targetf == NULL) {
targetf = fragment_lookup_fine_and_coarse(dcontext, dcontext->next_tag,
&coarse_f, dcontext->last_exit);
}
if (targetf == NULL) {
targetf = build_basic_block_fragment(dcontext, dcontext->next_tag, 0,
true /*link*/, true /*visible*/,
false /*!for_trace*/, NULL);
}
if (targetf == NULL)
break;
} while (true);
if (targetf != NULL) {
if (dispatch_enter_fcache(dcontext, targetf)) {
/* won't reach here: will re-enter d_r_dispatch() with a clean stack */
ASSERT_NOT_REACHED();
} else
targetf = NULL; /* targetf was flushed */
}
}
```

现在得到简化后的`d_r_dispatch`函数，我们以动态跟踪详细分析各个函数的作用。

在执行的过程中发现前面的函数都没有为`targetf`赋值，直到`build_basic_block_fragment`函数：

![图片描述](upload/attach/202304/949068_V93SCY847BEVVG8.png)

因此我们首先分析`build_basic_block_fragment`,之后再分析前面的函数做了哪些操作。

### build_basic_block_fragment

从名字上就能看出此函数创建了基本块。基本块是从入口点开始，直到到达控制转移（控制转移说白了 就是如跳转，call，ret等，这种不按程序的语句流程执行的指令）。下图便是一个基本块：

![图片描述](upload/attach/202304/949068_QE4X7AY6HBTMHEG.png)

首先笔者先带大家了解常见的控制转移指令的缩写：`cti`(Control Transfer Instructions 控制转移指令)，`ubr`(Unconditional Branch Instruction 无条件跳转指令如jmp)，`cbr`（Conditional Branch Instruction 条件分支指令），`mbr`（通过寄存器等的间接分支）

现在我们开始分析`build_basic_block_fragment`此函数，简化后如下：

```
fragment_t *
build_basic_block_fragment(dcontext_t *dcontext, app_pc start, uint initial_flags,
bool link, bool visible, bool for_trace,
instrlist_t **unmangled_ilist)
{
fragment_t *f;
build_bb_t bb;
dr_where_am_i_t wherewasi = dcontext->whereami;
/* 初始化bb */
init_interp_build_bb(dcontext, &bb, start, initial_flags, for_trace, unmangled_ilist);
build_bb_ilist(dcontext, &bb);
f = emit_fragment_ex(dcontext, start, bb.ilist, bb.flags, bb.vmlist, link, visible);
exit_interp_build_bb(dcontext, &bb);
return f;
}
```

让我们来逐个分析各个函数的作用

#### init_interp_build_bb

```
static inline void
init_interp_build_bb(dcontext_t *dcontext, build_bb_t *bb, app_pc start,
uint initial_flags, bool for_trace, instrlist_t **unmangled_ilist)
{
dcontext->bb_build_info = (void *)bb;
init_build_bb(
bb, start, true /*real interp*/, true /*for cache*/, true /*mangle*/,
false /* translation: set below for clients */, INVALID_FILE,
initial_flags |
(INTERNAL_OPTION(store_translations) ? FRAG_HAS_TRANSLATION_INFO : 0),
NULL /*no overlap*/);
if (!TEST(FRAG_TEMP_PRIVATE, initial_flags))
bb->has_bb_building_lock = true;
.....
}
static void
init_build_bb(build_bb_t *bb, app_pc start_pc, bool app_interp, bool for_cache,
bool mangle_ilist, bool record_translation, file_t outf, uint known_flags,
overlap_info_t *overlap_info)
{
memset(bb, 0, sizeof(*bb));
bb->check_vm_area = true;
bb->start_pc = start_pc;
bb->app_interp = app_interp;
bb->for_cache = for_cache;
if (bb->for_cache)
bb->record_vmlist = true;
bb->mangle_ilist = mangle_ilist;
bb->record_translation = record_translation;
bb->outf = outf;
bb->overlap_info = overlap_info;
bb->follow_direct = !TEST(FRAG_SELFMOD_SANDBOXED, known_flags);
bb->flags = known_flags;
bb->ibl_branch_type = IBL_GENERIC; /* initialization only */
}
```

可以看到`init_interp_build_bb`此函数为`bb`分配了空间，并对`bb`进行了初始化操作。执行此函数后`bb`的结构如下：

![图片描述](upload/attach/202304/949068_TK7KTF9W48UVN5A.png)

#### build_bb_ilist

此函数十分庞大，因为它维持着一个基本块的解码操作，我们将之简化后如下：

```
DISABLE_NULL_SANITIZER
static void
build_bb_ilist(dcontext_t *dcontext, build_bb_t *bb)
{
dcontext_t *my_dcontext = get_thread_private_dcontext();
bb->cur_pc = bb->start_pc;
bb->instr_start = bb->cur_pc;
/* 为bb->ilist分配空间 并对其初始化 */
bb->ilist = instrlist_create(dcontext);
bb->instr = NULL;
/* 循环为每条指令解码 */
while (true) {
/* 为bb->instr分配空间并初始化 */
bb->instr = instr_create(dcontext);
non_cti_start_pc = bb->cur_pc;
bb->instr_start = bb->cur_pc;
/* 如果是64位执行状态调用decode_with_ldstex */
/* 如果是32位执行状态调用decode，这里我们使用decode */
/* decode函数会解析一条指令填充到bb->instr 并将下一条指令基址给bb->cur_pc */
bb->cur_pc = IF_AARCH64_ELSE(decode_with_ldstex,
decode)(dcontext, bb->cur_pc, bb->instr);
total_instrs++;
/* 如果当前指令为非条件跳转 */
if (instr_is_near_ubr(bb->instr)) {
/* 一般返回false */
/* 此函数会将instr追加到ilist中 */
/* 同时会设置 exit_target为跳转的地址 */
if (bb_process_ubr(dcontext, bb))
continue;
else {
/* 一般会进入这里 */
if (bb->instr != NULL) /* else, bb_process_ubr() set exit_type */
bb->exit_type |= instr_branch_type(bb->instr);
break;
}
}
/* 如果不是，将当前指令加入到bb->ilist中 */
else
instrlist_append(bb->ilist, bb->instr);
/* 如果是直接调用 */
if (instr_is_near_call_direct(bb->instr)) {
if (!bb_process_call_direct(dcontext, bb)) {
if (bb->instr != NULL)
bb->exit_type |= instr_branch_type(bb->instr);
break;
}
}
/* 之后都是判断是什么控制转移指令 */
......
/* 如果是条件跳转 */
else if (instr_is_cti(bb->instr) &&
(!instr_is_call(bb->instr) || instr_is_cbr(bb->instr))) {
total_branches++;
if (total_branches >= BRANCH_LIMIT) {
/* set type of 1st exit cti for cbr (bb->exit_type is for fall-through) */
/* 设置instr->flags*/
instr_exit_branch_set_type(bb->instr, instr_branch_type(bb->instr));
break;
}
}
/* 但是条件跳转并没有设置exit_target */
} /* while(true)的结尾 */
bb->end_pc = bb->cur_pc;
/* 此函数执行会调用我们注册的回调 */
/* 我们后面再分析他 */
client_process_bb(dcontext, bb);
如果没有设置exit_target，则会将下一条指令地址给exit_target。
if (bb->exit_target == NULL){
bb->exit_target = (cache_pc)bb->cur_pc;
}
下面过程笔者将通过实例呈现
if (bb->mangle_ilist &&
(bb->instr == NULL || !instr_opcode_valid(bb->instr) ||
!instr_is_near_ubr(bb->instr) || instr_is_meta(bb->instr))) {
/* 此宏会创建一个jmp指令，参数二为跳转的目标地址 */
instr_t *exit_instr =
XINST_CREATE_jump(dcontext, opnd_create_pc(bb->exit_target));
/* 将此jmp语句加入到ilist中 */
instrlist_append(bb->ilist, exit_instr);
}
/* bb->instr清空 */
bb->instr = NULL;
/* 此函数会将ilist处理成我们想要的状态 */
/* 比如 如果是以直接调用指令结束的基本块 */
/* 此函数会将call 转化成 push eip和 jmp*/
mangle_bb_ilist(dcontext, bb);
}
```

终于我们完成了对此函数的大致解读，为了让我们更加清晰，笔者将通过动态调试分析此过程

首先让我们执行到`while (true)` ，此时`cur_pc`为`772641e0`：

![图片描述](upload/attach/202304/949068_9XQMQQDQEJ4JVZT.png)

运行`decode`函数，此时应解码成`cmp`指令，将此指令填充到`bb->instr`结构：

![图片描述](upload/attach/202304/949068_7RHJY3U9WU6M7WA.png)

此时`opcode`为`0xe`，

![图片描述](upload/attach/202304/949068_UVJQMHT534R6B2P.png)

随后发现其不是`cti`指令将此`cmp`指令加入到`bb->ilist`：

![图片描述](upload/attach/202304/949068_33CG8RZPR8E23Q2.png)

![图片描述](upload/attach/202304/949068_FQJ8YRYTSEM8F79.png)

接着解码下一条指令，发现其为条件跳转指令，将其加入`ilist`跳出`while`循环，将下一条指令地址给`bb->exit_target`：

![图片描述](upload/attach/202304/949068_E4VCASK2WKSH2DQ.png)

创建`ilist`的过程还没有结束，因为此条件跳转指令带来了两个分支，为了确定采用了哪个分支，DynamoRIO在条件跳转指令后再添加一个`jmp`指令，`jmp`的跳转地址为`bb->exit_target`(也就是条件跳转指令的下一条指令)。将此`jmp`语句加入到`ilist`中。

还没有结束因为还有一个函数我们没有分析，`client_process_bb`这个函数是在`while`循环结束后调用的。还记得上篇中我们注册的收集覆盖率信息的回调吗。`client_process_bb`将调用我们注册的回调。

#### client_process_bb

回调的调用过程如下：
