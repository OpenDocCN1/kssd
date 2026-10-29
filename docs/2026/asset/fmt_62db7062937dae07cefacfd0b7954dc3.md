
`void d_r_dispatch(dcontext_t *dcontext){`
`while(true){`
`...`
`//先检测是否生成trace缓存块`
`fragment_t * targetf = monitor_cache_enter(dcontext, targetf);`
`if(targetf==NULL)`
`{`
`//直到循环构建fragment_t完成`
`targetf = build_basic_block_fragment(`
`dcontext, dcontext->next_tag, 0, true /*link*/,`
`true /*visible*/`
`_IF_CLIENT(false /*!for_trace*/) _IF_CLIENT(NULL));`
`}`
`if (targetf != NULL) {`
`//执行当前构建好的fragment_t,见下文路由逻辑分析`
`dispatch_enter_fcache(dcontext, targetf);`
`}`
`fragment_t * build_basic_block_fragment(dcontext_t *dcontext, app_pc start, uint initial_flags,`
`bool link,`
`bool visible _IF_CLIENT(bool for_trace)`
`_IF_CLIENT(instrlist_t **unmangled_ilist))`
`{`
`build_bb_ilist(dcontext, &bb);`
`return f = emit_fragment_ex(dcontext, start, bb.ilist, bb.flags, bb.vmlist, link, visible);{`
`return emit_fragment_common(dcontext, tag, ilist, flags, vmlist, link, visible,`
`NULL /* not replacing */);`
`}`
`}`
```

`build_basic_block_fragment`内部调用了`build_bb_ilist`用于decode解码当前指令,DynamoRIO内部实现了一套编码与解码对相对于汇编语言的操作库用于将每一条汇编指令翻译成`instr_t`结构,其中包含多个源操作数和目的操作数`opnd_t`结构和当前指令的实际解析地址`app_pc` `translation`和操作码`opcode`等字段,DynamoRIO的api提供了对于`instr`可以调用`opnd_create_xxx`创建操作数后将其和操作码`opcode`作为参数调用`instr_create_xxx`系列函数创建`instr_t`结构,常用基于寄存器类操作数通常可表示为`base_reg + index_reg*scale + disp`类型通用处理.

`build_bb_ilist`中循环每个decode后的`instr`直到确定当前`instr`存在分支指令,根据分支类型计算出`direct_stubs`和`indirect_stubs`的数量,根据这些分支数量在`fragment`尾部申请若干个`linkstub`用于保存分支类型和偏移量,完成`fragment`的构建初始化.在`set_linkstub_fields`把`linkstub`对应分支相对于当前`fragment`起始地址的偏移量字段赋值,将其中需要退出插桩的`linkstub`设置出口目标为对应`ibl_routine`表中的回调项,最后调用默认编码函数`instr_encode_arch`序列化出程序实际运行的汇编代码和`linkstub`的分支路由代码完成`fragment`的构建.

```
static void build_bb_ilist(dcontext_t *dcontext, build_bb_t *bb){
 //最终构建好的块起始地址start_pc就是解码出来的第一条指令
bb->cur_pc = bb->start_pc;
 //在这里使用2中模式解码
if (bb->full_decode) {
 //默认解码函数decode_with_ldstex
bb->cur_pc = IF_AARCH64_ELSE(decode_with_ldstex,
decode)(dcontext, bb->cur_pc, bb->instr);
if (bb->record_translation)
instr_set_translation(bb->instr, bb->instr_start);
} else {
 //分支结构的解码函数decode_cti_with_ldstex
instr_reset(dcontext, bb->instr);
bb->cur_pc = IF_AARCH64_ELSE(decode_cti_with_ldstex, decode_cti)(dcontext, bb->cur_pc, bb->instr);
}
//笔者简化了分支生成代码方便读者理解
if (instr_is_cti(bb->instr))
{
 //根据最后一个instr的类型确定当前块的分支类型
bb->ibl_branch_type= instr_branch_type(bb->instr)
bb->exit_type |= instr_branch_type(bb->instr);
 //处理当前分支结束后需要继续处理的路由,这个由会见对应的fragment加入不同的ibl表中
bb->exit_target = get_ibl_routine(dcontext, get_ibl_entry_type(bb->exit_type), DEFAULT_IBL_BB(), bb->ibl_branch_type);
}
//处理当前块解析对应留给DynamoRIO客户端的回调
if (!client_process_bb(dcontext, bb)) {
 //里面使用这个宏触发回调bb_callbacks,详见插桩API回调分析小节
call_all_ret(ret, |=, , bb_callbacks,
int (*)(void *, void *, instrlist_t *, bool, bool), (void *)dcontext,(void *)tag, bb, for_trace, translating);
}}
static fragment_t * emit_fragment_common(dcontext_t *dcontext, app_pc tag, instrlist_t *ilist, uint flags,void *vmlist, bool link_fragment, bool add_to_htable,fragment_t *replace_fragment)
{
 //根据当前块分支数量在fragment尾部申请若干个linkstub
fragment_t * f = fragment_create(dcontext, tag, offset + extra_jmp_padding_body, num_direct_stubs,
num_indirect_stubs, stub_size_total + extra_jmp_padding_stubs,
flags);
 //编码fragment,链接所有direct_stubs和indirect_stubs
cache_pc pc = set_linkstub_fields(dcontext, f, ilist, num_direct_stubs, num_indirect_stubs,
true /*encode each instr*/);
for (l = FRAGMENT_EXIT_STUBS(f); l; l = LINKSTUB_NEXT_EXIT(l)) {
 //内联模式直接复制对应ibl_routine表中的回调项代码
if (indirect && can_inline) {
ibl_code_t *ibl_code = get_ibl_routine_code(dcontext, extract_branchtype(l->flags), f->flags);
memcpy(pc, ibl_code->inline_ibl_stub_template, ibl_code->inline_stub_length);
} else {
 //非内联模式设置跳转
cache_pc exit_target=get_ibl_routine_ex(dcontext, get_unlinked_type(ibl_type.link_state),
ibl_type.source_fragment_type,
ibl_type.branch_type _IF_X86_64(mode));
pc = insert_relative_jump(pc, exit_target, NOT_HOT_PATCHABLE);
}
}
return f;
}
/*
cache_pc
set_linkstub_fields(dcontext_t *dcontext, fragment_t *f, instrlist_t *ilist,
uint num_direct_stubs, uint num_indirect_stubs, bool emit)
{
cache_pc pc=f->start_pc;
linkstub_t * l = FRAGMENT_EXIT_STUBS(f);
for (inst = instrlist_first(ilist); inst; inst = instr_get_next(inst)) {
if (instr_is_exit_cti(inst)) {
 /*
if (LINKSTUB_CBR_FALLTHROUGH(l->flags)) {
 /* target is indicated via cti_offset */
ASSERT_TRUNCATE(l->cti_offset, short, target - f->tag);
l->cti_offset = (ushort) /* really a short */ (target - f->tag);
} else {
ASSERT_TRUNCATE(l->cti_offset, ushort, pc - f->start_pc);
l->cti_offset = (ushort)(pc - f->start_pc);
}
 //对于indirect类型分支,里面实际上是调用get_ibl_routine_ex获取ibl表中emit_indirect_branch_lookup,其他类型同理
if (!EXIT_HAS_STUB(l->flags, f->flags)) {
instr_set_branch_target_pc(inst, get_unlinked_entry(dcontext, target));
} else {
 //暂时先设为入口pc,会在以后的链接fragment的函数里面再次计算
instr_set_branch_target_pc(inst, pc);
}
}
pc = instr_encode_to_copy(dcontext, inst, vmcode_get_writable_addr(pc),
pc);
{
 //调用默认编码函数instr_encode_arch序列化出程序实际运行的汇编代码
instr_encode_arch(dcontext, instr, copy_pc, final_pc, true,NULL _IF_DEBUG(true));
}
}
}
*/
```

## 插桩API回调分析

DynamoRIO提供了`drmgr_register_bb_instrumentation_ex_event`等api实现了hook每个插桩过程的回调,里面又包装了4种回调类型,这些回调的真正入口由上文的`call_all_ret`宏将参数传给由`dr_register_bb_event`注册的`bb_callbacks`这个回调数组分发回调.初学者可能对这些不同的回调类型不知道如何使用,DynamoRIO的几个demo中由例子可以参考,借助简化后还原代码我们来分析下这几种回调分别对应什么功能
