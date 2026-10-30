
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 | `/* check_stack_read/write functions track spill/fill of registers, stack boundary and alignment are checked in check_mem_access() */ static int check_stack_write(struct bpf_verifier_env *env, struct bpf_func_state *state, /* func where register points to */ int off, int size, int value_regno, int insn_idx) { struct bpf_func_state *cur; /* state of the current function */ int i, slot = -off - 1, spi = slot / BPF_REG_SIZE, err; u32 dst_reg = env->prog->insnsi[insn_idx].dst_reg; struct bpf_reg_state *reg = NULL; // 由于栈扩展导致的需要重新申请空间放state跟踪stack状态 err = realloc_func_state(state, round_up(slot + 1, BPF_REG_SIZE), state->acquired_refs, true); if (err) return err; /* caller checked that off % size == 0 and -MAX_BPF_STACK <= off < 0, * so it's aligned access and [off, off + size) are within stack limits */ // 不允许非特权用户由于部分写破坏栈上合法指针 if (!env->allow_ptr_leaks && state->stack[spi].slot_type[0] == STACK_SPILL && size != BPF_REG_SIZE) { verbose(env, "attempt to corrupt spilled pointer on stack\n"); return -EACCES; } cur = env->cur_state->frame[env->cur_state->curframe]; if (value_regno >= 0) reg = &cur->regs[value_regno]; // 优化回溯 if (reg && size == BPF_REG_SIZE && register_is_const(reg) && !register_is_null(reg) && env->allow_ptr_leaks) { if (dst_reg != BPF_REG_FP) { /* The backtracking logic can only recognize explicit * stack slot address like [fp - 8]. Other spill of * scalar via different register has to be conervative. * Backtrack from here and mark all registers as precise * that contributed into 'reg' being a constant. */ err = mark_chain_precision(env, value_regno); if (err) return err; } save_register_state(state, spi, reg); } else if (reg && is_spillable_regtype(reg->type)) { /* register containing pointer is being spilled into stack */ // 源寄存器是STACK_SPILL，部分写不允许 if (size != BPF_REG_SIZE) { verbose_linfo(env, insn_idx, "; "); verbose(env, "invalid size of register spill\n"); return -EACCES; } if (state != cur && reg->type == PTR_TO_STACK) { verbose(env, "cannot spill pointers to stack into stack frame of the caller\n"); return -EINVAL; } // 引入sanitize，防止stack的重用，防范侧信道攻击？ if (!env->allow_ptr_leaks) { bool sanitize = false; if (state->stack[spi].slot_type[0] == STACK_SPILL && register_is_const(&state->stack[spi].spilled_ptr)) sanitize = true; for (i = 0; i < BPF_REG_SIZE; i++) if (state->stack[spi].slot_type[i] == STACK_MISC) { sanitize = true; break; } if (sanitize) { int *poff = &env->insn_aux_data[insn_idx].sanitize_stack_off; int soff = (-spi - 1) * BPF_REG_SIZE; /* detected reuse of integer stack slot with a pointer * which means either llvm is reusing stack slot or * an attacker is trying to exploit CVE-2018-3639 * (speculative store bypass) * Have to sanitize that slot with preemptive * store of zero. */ if (*poff && *poff != soff) { /* disallow programs where single insn stores * into two different stack slots, since verifier * cannot sanitize them */ verbose(env, "insn %d cannot access two stack slots fp%d and fp%d", insn_idx, *poff, soff); return -EINVAL; } *poff = soff; } } save_register_state(state, spi, reg); } else { // reg是非有效指针 u8 type = STACK_MISC; /* regular write of data into stack destroys any spilled ptr */ state->stack[spi].spilled_ptr.type = NOT_INIT; /* 标记栈为STACK_SPILL， 代表包含非指针、非0变量等 */ /* Mark slots as STACK_MISC if they belonged to spilled ptr. */ if (state->stack[spi].slot_type[0] == STACK_SPILL) for (i = 0; i < BPF_REG_SIZE; i++) state->stack[spi].slot_type[i] = STACK_MISC; /* only mark the slot as written if all 8 bytes were written * otherwise read propagation may incorrectly stop too soon * when stack slots are partially written. * This heuristic means that read propagation will be * conservative, since it will add reg_live_read marks * to stack slots all the way to first state when programs * writes+reads less than 8 bytes */ if (size == BPF_REG_SIZE) state->stack[spi].spilled_ptr.live |= REG_LIVE_WRITTEN; /* when we zero initialize stack slots mark them as such */ // reg为0 if (reg && register_is_null(reg)) { /* backtracking doesn't work for STACK_ZERO yet. */ err = mark_chain_precision(env, value_regno); if (err) return err; type = STACK_ZERO; // 设置为STACK_ZERO } /* Mark slots affected by this stack write. */ // reg为0，标记所写栈的slot类型为设置为STACK_ZERO，否则仍然是一开始的MISC for (i = 0; i < size; i++) state->stack[spi].slot_type[(slot - i) % BPF_REG_SIZE] = type; } return 0; } |
```

- 读操作，检测可读性，这里就不展开了，比可写性简单一些。

### check_max_stack_depth

我们重新回到上层的 `bpf_check` 中。

函数目的：

- 保证函数调用深度不超过 `MAX_BPF_STACK`

- 挑选出code为**BPF_JMP** , **BPF_CALL** ；src_reg为：**BPF_PSEUDO_CALL**，即eBPF2eBPF的函数调用。记录对应的深度与返回地址，调用完返回上层subprog。在这之间如果调用深度超过8或者最大栈消耗超过512字节，abort。

|  |  |

| --- | --- |
