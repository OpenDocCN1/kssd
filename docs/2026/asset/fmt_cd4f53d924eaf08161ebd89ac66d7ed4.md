
## `emit_indirect_branch_lookup` 函数分析

下面是 `emit_indirect_branch_lookup()` 函数的骨架代码。它的核心功能是生成一段用于间接分支查找（IBL）的指令序列，主要负责在分支目标缓存中查找目标 fragment，并决定后续的执行流。

```
byte *emit_indirect_branch_lookup(dcontext_t *dcontext, generated_code_t *code, byte *pc,
                                  byte *fcache_return_pc, bool target_trace_table,
                                  bool inline_ibl_head, ibl_code_t *ibl_code /* IN/OUT */)
{
    fragment_not_found = XINST_CREATE_load(dcontext, opnd_create_reg(SCRATCH_REG2),
                                           opnd_create_reg(SCRATCH_REG1));
    instr_t *next_fragment_nochasing =
        INSTR_CREATE_cmp(dcontext, OPND_CREATE_MEMPTR(SCRATCH_REG2, HASHLOOKUP_TAG_OFFS),
                         OPND_CREATE_INT8(0));
    if (!inline_ibl_head) {
        // 比较fragment头部，如果找到fragment跳转至下个fragment执行，如果没找到跳转至`next_fragment_nochasing`
        append_ibl_head(dcontext, &ilist, ibl_code, patch, &fragment_found, &compare_tag,
                        IF_X64_ELSE(&trace_cmp_entry, NULL),
                        opnd_create_instr(next_fragment_nochasing),
                        true /*miss can have 8-bit offs*/, target_trace_table,
                        inline_ibl_head);
    }
    // 再次比较fragment头部
    APP(&ilist, next_fragment_nochasing);
    if (INTERNAL_OPTION(ibl_sentinel_check)) {
        // 再次比较fragment头部`HASHLOOKUP_START_PC` 附加检查
        sentinel_check = INSTR_CREATE_cmp(
            dcontext, OPND_CREATE_MEMPTR(SCRATCH_REG2, HASHLOOKUP_START_PC_OFFS),
            OPND_CREATE_INT8((int)(ptr_int_t)HASHLOOKUP_SENTINEL_START_PC));
    } else {
        // 绕过sentinel检查，回到`fcache_return` 返回`DynamoRIO`
        sentinel_check = fragment_not_found;
    }
    APP(&ilist,
        INSTR_CREATE_jcc(dcontext, OP_je_short, opnd_create_instr(sentinel_check)));
    if (inline_ibl_head) {
        // 里面的逻辑和`append_ibl_head` 等价
        compare_tag = INSTR_CREATE_cmp(
            dcontext, OPND_CREATE_MEMPTR(SCRATCH_REG2, HASHLOOKUP_TAG_OFFS),
            opnd_create_reg(SCRATCH_REG1));
        APP(&ilist, compare_tag); // 同样比较fragment头部，如果找到fragment跳转下个fragment入口执行，如果没找到跳转至`next_fragment_nochasing`
        APP(&ilist,
            INSTR_CREATE_jcc(dcontext, OP_jne_short,
                             opnd_create_instr(next_fragment_nochasing)));
        // 增加fragment引用计数，跳转下个fragment入口执行
        append_ibl_found(dcontext, &ilist, ibl_code, patch, HASHLOOKUP_START_PC_OFFS,
                         true, only_spill_state_in_tls,
                         target_trace_table ? DYNAMO_OPTION(trace_single_restore_prefix)
                                            : DYNAMO_OPTION(bb_single_restore_prefix),
                         NULL);
    } else {
        // 继续进入下个fragment检查入口
        APP(&ilist, INSTR_CREATE_jmp_smart(dcontext, opnd_create_instr(compare_tag)));
    }
    if (INTERNAL_OPTION(ibl_sentinel_check)) {
        APP(&ilist, sentinel_check);
        // 如果sentinel检查失败同样回到`fcache_return` 返回`DynamoRIO`，成功继续进入下个fragment检查入口
        APP(&ilist,
            INSTR_CREATE_jcc(dcontext, OP_jne_short,
                             opnd_create_instr(fragment_not_found)));
        APP(&ilist, INSTR_CREATE_jmp(dcontext, opnd_create_instr(compare_tag)));
    }
    APP(&ilist, fragment_not_found);
    // 返回`DynamoRIO` 或者跳转到当前函数参数`fcache_return_pc` 地址（也就是返回`d_r_dispatch`），这个后面接下来的代码永远不会到达
    append_fcache_return_common(dcontext, code, &ilist, true /*ibl end*/, absolute,
                                false /*!shared*/, NULL, false /*no coarse info*/);
}
```

## 函数间调用分析

`DynamoRIO` 提供了一系列 `dr_insert_call_xxx` 用于将 `client` 中的函数代码以 `instrlist` 形式注入到目标代码中。除了借助 `DynamoRIO` 自身的编码和解码引擎外，另一种使用的技术就是寄存器预留。假设汇编指令执行到某个位置的时候，所有的寄存器值和栈空间的数据都是对当前程序有直接联系的，如果直接注入汇编代码就会破坏这个逻辑。采取的一种等价的 `Hook` 方案是，先把需要用到的寄存器标记位脏寄存器的值存入 `TLS`（线程本地储存）或者 `dcontext` 中，当前也可以用汇编指令原生的 `PUSH`/`POP` 方式，对应栈空间也是在当前栈底 `XBP` 位置暂存或开辟一段固定大小的栈空间，等执行完 `Hook` 代码再还原（恢复栈空间 `XBP`）。这样就构成了一个环，以递归的方式预留寄存器等，在用完的时候再还原回去，如果再次用到就放到另外一个寄存器去，用完了再递归还原，这样除了对源程序内存直接修改是全局的之外，退出 `Hook` 时还原现场，对原有的程序运行理论上是没有影响的。`DynamoRIO` 正是采用的这种方法，先将 `client` 中代码编码成 `instrlist` 插入源程序指令缓存，解析时候可以内联执行还是递归执行。

```
DR_API void
dr_insert_mbr_instrumentation(void *drcontext, instrlist_t *ilist, instr_t *instr,
                              void *callee, dr_spill_slot_t scratch_slot)
{
    if (instr_is_return(instr)) {
        opnd_t reg_target = instr_get_src(instr, instr_num_srcs(instr) - 1);
    } else {
        opnd_t reg_target = instr_get_src(instr, 0);
    }
    dr_insert_clean_call(drcontext, ilist, instr, callee, false /*no fpstate*/, 2,
                         /* address of mbr is 1st param */
                         OPND_CREATE_INTPTR(address),
                         /* indirect target (in tls, xchg-d from ecx) is 2nd param */
                         tls_opnd);
}

DR_API void
dr_insert_call_instrumentation(void *drcontext, instrlist_t *ilist, instr_t *instr,
                               void *callee)
{
    ptr_uint_t address = (ptr_uint_t)instr_get_translation(instr);
    ptr_uint_t target = (ptr_uint_t)opnd_get_pc(instr_get_target(instr));
    dr_insert_clean_call(drcontext, ilist, instr, callee, false /*no fpstate*/, 2,
                         /* address of call is 1st parameter */
                         OPND_CREATE_INTPTR(address),
                         /* call target is 2nd parameter */
                         OPND_CREATE_INTPTR(target));
}

// dr_insert_clean_call内部实现
void dr_insert_clean_call_ex_varg(void *drcontext, instrlist_t *ilist, instr_t *where,
                                  void *callee, dr_cleancall_save_t save_flags, uint num_args,
                                  opnd_t *args)
{
    // 如果调用内部不包含远程call，系统调用之类的分支就内联执行
    if (analyze_clean_call(dcontext, &cci, where, callee, save_fpstate,
                           TEST(DR_CLEANCALL_ALWAYS_OUT_OF_LINE, save_flags), num_args,
                           args) {
        return insert_inline_clean_call(dcontext, &cci, ilist, where, args);
    }
    insert_meta_call_vargs(dcontext, ilist, where, call_flags, encode_pc, callee,
                           num_args, args) {
        uint stack_for_params = insert_parameter_preparation(
            dcontext, ilist, instr, TEST(META_CALL_CLEAN, flags), num_args, args) {
            // 循环call的所有参数
            for (i = 0; i < num_args; i++) {
                if (clean_call && opnd_uses_reg(arg, REG_XSP)) {
                    // push模式顺序：先替换`XSP` 为`eax`，保存`eax`->tls，读取`mc`->`eax`，读取`esp`->`eax`，保存`arg`->`eax`（间接通过`xsp`），`push eax`，还原`restore eax`
                    reg_id_t scratch = REG_XAX;
                    if (opnd_uses_reg(arg, scratch)) {
                        scratch = REG_XCX;
                        ASSERT(!opnd_uses_reg(arg, scratch)); /* can't use 3 regs */
                    }
                    opnd_replace_reg(&arg, REG_XSP, scratch);
                    POST(ilist, mark,
                         instr_create_restore_from_tls(dcontext, scratch,
                                                       TLS_XAX_SLOT));
                    POST(ilist, mark, INSTR_CREATE_push(dcontext, arg));
                    POST(ilist, mark,
                         instr_create_restore_from_dc_via_reg(dcontext, scratch,
                                                              scratch, XSP_OFFSET));
                    // 内部是`PRE` 即`instrlist_meta_preinsert`
                    insert_get_mcontext_base(dcontext, ilist, instr_get_next(mark),
                                             scratch);
                    POST(ilist, mark,
                         instr_create_save_to_tls(dcontext, scratch, TLS_XAX_SLOT));
                }
            }
        }
        PRE(ilist, where, INSTR_CREATE_call(dcontext, opnd_create_pc(target)));
    }
}
```

常见的用于追踪分支指令的 `dr_insert_mbr_instrumentation`，`dr_insert_call_instrumentation`，`dr_insert_cbr_instrumentation` 等内部都通过 `dr_insert_clean_call` 将分支指令的地址和目的地址作为参数传入调用实现的。为了保证尽可能的减少使用栈空间来包装插入的 `call`，如果 `xsp` 寄存器没有在 `call` 内部被占用，则采用直接往栈 `push` 参数的函数完成参数布局。如果在 `xsp` 占用的情况下，`DynamoRIO` 采用的一种很巧妙的方法解决了这个问题。这里简单分析一下，在上面的代码段中 `POST` 宏代表往指定指令后插入指令，`PRE` 宏代表往指定指令之前插入指令，所以最后的调用顺序并不是代码中看到的从上至下的顺序，详见注释。`DynamoRIO` 先将 `xsp` 寄存器替换为 `xax` 寄存器，这样往栈 `push` 数据实际上就是以 `xax` 为栈指针 `push` 数据，最终的解为暂存并预留 `xax` 后，读取 `mcontext` 中的 `xsp` 至 `xax`，然后 `push` 参数 `arg`，最后还原 `xax`，至此就实现了在 `xsp` 被占用的情况下也能 `push` 参数的功能。接下来就是序列化 `call` 函数然后直接调用了，这里不再赘述。

## 笔者demo分析
