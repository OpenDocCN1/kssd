
```c
static int jit_subprogs(struct bpf_verifier_env *env)
{
struct bpf_prog *prog = env->prog, **func, *tmp;
int i, j, subprog_start, subprog_end = 0, len, subprog;
struct bpf_insn *insn;
void *old_bpf_func;
int err;
//判断subprog数量
if (env->subprog_cnt <= 1)
return 0;
//扫描指令，找到bpf2bpf的调用
for (i = 0, insn = prog->insnsi; i < prog->len; i++, insn++) {
if (insn->code != (BPF_JMP | BPF_CALL) ||
insn->src_reg != BPF_PSEUDO_CALL)
continue;
/* Upon error here we cannot fall back to interpreter but
* need a hard reject of the program. Thus -EFAULT is
* propagated in any case.
*/
//根据调用地址找到被调用subprog的索引，保存索引到insn->off中。
subprog = find_subprog(env, i + insn->imm + 1);
if (subprog < 0) {
WARN_ONCE(1, "verifier bug. No program starts at insn %d\n",
i + insn->imm + 1);
return -EFAULT;
}
/* temporarily remember subprog id inside insn instead of
* aux_data, since next loop will split up all insns into funcs
*/
insn->off = subprog;
/* remember original imm in case JIT fails and fallback
* to interpreter will be needed
*/
env->insn_aux_data[i].call_imm = insn->imm;
/* point imm to __bpf_call_base+1 from JITs point of view */
insn->imm = 1;
}
//分配空间
err = bpf_prog_alloc_jited_linfo(prog);
if (err)
goto out_undo_insn;
err = -ENOMEM;
func = kcalloc(env->subprog_cnt, sizeof(prog), GFP_KERNEL);
if (!func)
goto out_undo_insn;
//为了每个subprog申请空间，然后根据prog来做初始化
for (i = 0; i < env->subprog_cnt; i++) {
subprog_start = subprog_end;
subprog_end = env->subprog_info[i + 1].start;
len = subprog_end - subprog_start;
/* BPF_PROG_RUN doesn't call subprogs directly,
* hence main prog stats include the runtime of subprogs.
* subprogs don't have IDs and not reachable via prog_get_next_id
* func[i]->aux->stats will never be accessed and stays NULL
*/
func[i] = bpf_prog_alloc_no_stats(bpf_prog_size(len), GFP_USER);
if (!func[i])
goto out_free;
memcpy(func[i]->insnsi, &prog->insnsi[subprog_start],
len * sizeof(struct bpf_insn));
func[i]->type = prog->type;
func[i]->len = len;
if (bpf_prog_calc_tag(func[i]))
goto out_free;
func[i]->is_func = 1;
func[i]->aux->func_idx = i;
/* the btf and func_info will be freed only at prog->aux */
func[i]->aux->btf = prog->aux->btf;
func[i]->aux->func_info = prog->aux->func_info;
/* Use bpf_prog_F_tag to indicate functions in stack traces.
* Long term would need debug info to populate names
*/
func[i]->aux->name[0] = 'F';
func[i]->aux->stack_depth = env->subprog_info[i].stack_depth;
func[i]->jit_requested = 1; //开启jit
func[i]->aux->linfo = prog->aux->linfo;
func[i]->aux->nr_linfo = prog->aux->nr_linfo;
func[i]->aux->jited_linfo = prog->aux->jited_linfo;
func[i]->aux->linfo_idx = env->subprog_info[i].linfo_idx;
func[i] = bpf_int_jit_compile(func[i]); //对该subprog做jit
if (!func[i]->jited) { //如果没有jit成功
err = -ENOTSUPP;
goto out_free;
}
cond_resched();
}
/* at this point all bpf functions were successfully JITed
* now populate all bpf_calls with correct addresses and
* run last pass of JIT
*/
//修正bpf2bpf调用的函数距离
for (i = 0; i < env->subprog_cnt; i++) {
insn = func[i]->insnsi;
for (j = 0; j < func[i]->len; j++, insn++) {
if (insn->code != (BPF_JMP | BPF_CALL) ||
insn->src_reg != BPF_PSEUDO_CALL)
continue;
subprog = insn->off;
//这里也只是部分修复，最后真正的修复在bpf_int_jit_compile中
insn->imm = BPF_CAST_CALL(func[subprog]->bpf_func) -
__bpf_call_base;
}
/* we use the aux data to keep a list of the start addresses
* of the JITed images for each function in the program
*
* for some architectures, such as powerpc64, the imm field
* might not be large enough to hold the offset of the start
* address of the callee's JITed image from __bpf_call_base
*
* in such cases, we can lookup the start address of a callee
* by using its subprog id, available from the off field of
* the call instruction, as an index for this list
*/
func[i]->aux->func = func;
func[i]->aux->func_cnt = env->subprog_cnt;
}
for (i = 0; i < env->subprog_cnt; i++) {
old_bpf_func = func[i]->bpf_func;
tmp = bpf_int_jit_compile(func[i]); //真正的修复bpf2bpf函数的调用距离
if (tmp != func[i] || func[i]->bpf_func != old_bpf_func) {
verbose(env, "JIT doesn't support bpf-to-bpf calls\n");
err = -ENOTSUPP;
goto out_free;
}
cond_resched();
}
/* finally lock prog and jit images for all functions and
* populate kallsysm
*/
for (i = 0; i < env->subprog_cnt; i++) {
bpf_prog_lock_ro(func[i]); //jit之后子函数内存只读
bpf_prog_kallsyms_add(func[i]); //添加到kallsyms
}
/* Last step: make now unused interpreter insns from main
* prog consistent for later dump requests, so they can
* later look the same as if they were interpreted only.
*/
for (i = 0, insn = prog->insnsi; i < prog->len; i++, insn++) {
if (insn->code != (BPF_JMP | BPF_CALL) ||
insn->src_reg != BPF_PSEUDO_CALL)
continue;
insn->off = env->insn_aux_data[i].call_imm;
subprog = find_subprog(env, i + insn->off + 1);
insn->imm = subprog;
}
prog->jited = 1; //设置jit完成
prog->bpf_func = func[0]->bpf_func; //prog在内核里的可执行地址
prog->aux->func = func;
prog->aux->func_cnt = env->subprog_cnt;
bpf_prog_free_unused_jited_linfo(prog);
return 0;
out_free:
for (i = 0; i < env->subprog_cnt; i++)
if (func[i])
bpf_jit_free(func[i]);
kfree(func);
out_undo_insn:
/* cleanup main prog to be interpreted */
prog->jit_requested = 0;
for (i = 0, insn = prog->insnsi; i < prog->len; i++, insn++) {
if (insn->code != (BPF_JMP | BPF_CALL) ||
insn->src_reg != BPF_PSEUDO_CALL)
continue;
insn->off = 0;
insn->imm = env->insn_aux_data[i].call_imm;
}
bpf_prog_free_jited_linfo(prog);
return err;
}
```

### do_jit / bpf_int_jit_compile

这是最主要的 `jit` 函数。

内核中实际上有两条可以到达这里的有效 `jit` 路径：

![](upload/attach/202106/876323_GHMVA6RC9TXV89G.png)

第一条是 `jit_subprogs`（这里涉及到 `bpf2bpf` 函数调用等，所以是多函数的 `bpf jit` 路径）。

另一条是通过 `bpf_prog_select_runtime`，即当 `bpf_check` 结束后，调用 `bpf_prog_select_runtime`。

### do_jit

调用： `static int do_jit(struct bpf_prog *bpf_prog, int *addrs, u8 *image,int oldproglen, struct jit_context *ctx)`

1. 首先通过 `emit_prologue` 构建栈初始化环境。为了符合 `x64` 的 `abi`，主要涉及 `rsp`、`rbp` 的操作等。

```c
/*
* Emit x86-64 prologue code for BPF program and check its size.
* bpf_tail_call helper will skip it while jumping into another program
*/
static void emit_prologue(u8 **pprog, u32 stack_depth, bool ebpf_from_cbpf)
{
u8 *prog = *pprog;
int cnt = X86_PATCH_SIZE;
/* BPF trampoline can be made to work without these nops,
* but let's waste 5 bytes for now and optimize later
*/
memcpy(prog, ideal_nops[NOP_ATOMIC5], cnt);
prog += cnt;
EMIT1(0x55); /* push rbp */
EMIT3(0x48, 0x89, 0xE5); /* mov rbp, rsp */
/* sub rsp, rounded_stack_depth */
EMIT3_off32(0x48, 0x81, 0xEC, round_up(stack_depth, 8));
EMIT1(0x53); /* push rbx */
EMIT2(0x41, 0x55); /* push r13 */
EMIT2(0x41, 0x56); /* push r14 */
EMIT2(0x41, 0x57); /* push r15 */
if (!ebpf_from_cbpf) {
/* zero init tail_call_cnt */
EMIT2(0x6a, 0x00);
BUILD_BUG_ON(cnt != PROLOGUE_SIZE);
}
*pprog = prog;
}
```

2. 针对类似的普通指令，可以一对一的进行翻译，`opcode`、`eBPF`、`寄存器`相匹配：

```c
/* ALU */
case BPF_ALU | BPF_ADD | BPF_X:
case BPF_ALU | BPF_SUB | BPF_X:
case BPF_ALU | BPF_AND | BPF_X:
case BPF_ALU | BPF_OR | BPF_X:
case BPF_ALU | BPF_XOR | BPF_X:
case BPF_ALU64 | BPF_ADD | BPF_X:
case BPF_ALU64 | BPF_SUB | BPF_X:
case BPF_ALU64 | BPF_AND | BPF_X:
case BPF_ALU64 | BPF_OR | BPF_X:
case BPF_ALU64 | BPF_XOR | BPF_X:
switch (BPF_OP(insn->code)) {
case BPF_ADD: b2 = 0x01; break;
case BPF_SUB: b2 = 0x29; break;
case BPF_AND: b2 = 0x21; break;
case BPF_OR: b2 = 0x09; break;
case BPF_XOR: b2 = 0x31; break;
}
if (BPF_CLASS(insn->code) == BPF_ALU64)
EMIT1(add_2mod(0x48, dst_reg, src_reg));
else if (is_ereg(dst_reg) || is_ereg(src_reg))
EMIT1(add_2mod(0x40, dst_reg, src_reg));
EMIT2(b2, add_2reg(0xC0, dst_reg, src_reg));
break;
```

3. 修正 `BPF_CALL`，当前的 `imm32 = func_addr-__bpf_call_base`
