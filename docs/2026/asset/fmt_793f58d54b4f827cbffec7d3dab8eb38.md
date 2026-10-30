
```
static int check_cond_jmp_op(struct verifier_env *env,

                 struct bpf_insn *insn, int *insn_idx)

{

    struct reg_state *regs = env->cur_state.regs;

    struct verifier_state *other_branch;

    u8 opcode = BPF_OP(insn->code);

    int err;



    if (opcode > BPF_EXIT) {

        verbose("invalid BPF_JMP opcode %x\n", opcode);

        return -EINVAL;

    }



    if (BPF_SRC(insn->code) == BPF_X) {

        if (insn->imm != 0) {

            verbose("BPF_JMP uses reserved fields\n");

            return -EINVAL;

        }



        /* check src1 operand */

        err = check_reg_arg(regs, insn->src_reg, SRC_OP);

        if (err)

            return err;



        if (is_pointer_value(env, insn->src_reg)) {

            verbose("R%d pointer comparison prohibited\n",

                insn->src_reg);

            return -EACCES;

        }

    } else {

        if (insn->src_reg != BPF_REG_0) {

            verbose("BPF_JMP uses reserved fields\n");

            return -EINVAL;

        }

    }



    /* check src2 operand */

    err = check_reg_arg(regs, insn->dst_reg, SRC_OP);

    if (err)

        return err;



    /* detect if R == 0 where R was initialized to zero earlier */

[1]    if (BPF_SRC(insn->code) == BPF_K &&

        (opcode == BPF_JEQ || opcode == BPF_JNE) &&

        regs[insn->dst_reg].type == CONST_IMM &&

        regs[insn->dst_reg].imm == insn->imm) {

        if (opcode == BPF_JEQ) {

            /* if (imm == imm) goto pc+off;

             * only follow the goto, ignore fall-through

             */

            *insn_idx += insn->off;

            return 0;

        } else {

            /* if (imm != imm) goto pc+off;

             * only follow fall-through branch, since

             * that's where the program will go

             */

            return 0;

        }

    }



[2]    other_branch = push_stack(env, *insn_idx + insn->off + 1, *insn_idx);

    if (!other_branch)

        return -EFAULT;



    /* detect if R == 0 where R is returned value from bpf_map_lookup_elem() */

    if (BPF_SRC(insn->code) == BPF_K &&

        insn->imm == 0 && (opcode == BPF_JEQ ||

                   opcode == BPF_JNE) &&

        regs[insn->dst_reg].type == PTR_TO_MAP_VALUE_OR_NULL) {

        if (opcode == BPF_JEQ) {

            /* next fallthrough insn can access memory via

             * this register

             */

            regs[insn->dst_reg].type = PTR_TO_MAP_VALUE;

            /* branch targer cannot access it, since reg == 0 */

            other_branch->regs[insn->dst_reg].type = CONST_IMM;

            other_branch->regs[insn->dst_reg].imm = 0;

        } else {

            other_branch->regs[insn->dst_reg].type = PTR_TO_MAP_VALUE;

            regs[insn->dst_reg].type = CONST_IMM;

            regs[insn->dst_reg].imm = 0;

        }

    } else if (is_pointer_value(env, insn->dst_reg)) {

        verbose("R%d pointer comparison prohibited\n", insn->dst_reg);

        return -EACCES;

    } else if (BPF_SRC(insn->code) == BPF_K &&

           (opcode == BPF_JEQ || opcode == BPF_JNE)) {



        if (opcode == BPF_JEQ) {

            /* detect if (R == imm) goto

             * and in the target state recognize that R = imm

             */

            other_branch->regs[insn->dst_reg].type = CONST_IMM;

            other_branch->regs[insn->dst_reg].imm = insn->imm;

        } else {

            /* detect if (R != imm) goto

             * and in the fall-through state recognize that R = imm

             */

            regs[insn->dst_reg].type = CONST_IMM;

            regs[insn->dst_reg].imm = insn->imm;

        }

    }

    if (log_level)

        print_verifier_state(env);

    return 0;

}
```

## Q&A2：能否进行直接的内存读写？

内存读写需要用到的指令主要是`BPF_LDX_MEM`或者`BPF_STX_MEM`两类。如下，当`r7`和`r8`的值可控就可以达到内存任意写，类似于`mov dword ptr[r7],r8`这样的操作。

```
 STX_MEM_DW(8,7,0x0,0x0)
```

接下来分析一下`ST`和`LD`有哪些限制，`check_reg_arg`[1]处检查寄存器是否访问寄存器的序号是否超过最大值10，如果是`SRC_OP`检查是否是未初始化的值。否则检查是否要写的地方是`rbp`，并将要写的寄存器值置为`UNKOWN`。然后是`check_mem_access`检查，该函数会根据读写类型检查`dst`或`src`的值是否为栈指针、数据包指针、map指针，否则不允许读写：

```
 else if (class == BPF_LDX) {

            enum bpf_reg_type src_reg_type;



            /* check for reserved fields is already done */



            /* check src operand */

[1]            err = check_reg_arg(regs, insn->src_reg, SRC_OP);

            if (err)

                return err;



[1]            err = check_reg_arg(regs, insn->dst_reg, DST_OP_NO_MARK);

            if (err)

                return err;



            src_reg_type = regs[insn->src_reg].type;



            /* check that memory (src_reg + off) is readable,

             * the state of dst_reg will be updated by this func

             */

[2]            err = check_mem_access(env, insn->src_reg, insn->off,

                           BPF_SIZE(insn->code), BPF_READ,

                           insn->dst_reg);

            if (err)

                return err;



            if (BPF_SIZE(insn->code) != BPF_W) {

                insn_idx++;

                continue;

            }



            if (insn->imm == 0) {

                /* saw a valid insn

                 * dst_reg = *(u32 *)(src_reg + off)

                 * use reserved 'imm' field to mark this insn

                 */

                insn->imm = src_reg_type;



            } else if (src_reg_type != insn->imm &&

                   (src_reg_type == PTR_TO_CTX ||

                    insn->imm == PTR_TO_CTX)) {

                /* ABuser program is trying to use the same insn

                 * dst_reg = *(u32*) (src_reg + off)

                 * with different pointer types:

                 * src_reg == ctx in one branch and

                 * src_reg == stack|map in some other branch.

                 * Reject it.

                 */

                verbose("same insn cannot be used with different pointers\n");

                return -EINVAL;

            }



        } else if (class == BPF_STX) {

            enum bpf_reg_type dst_reg_type;



            if (BPF_MODE(insn->code) == BPF_XADD) {

                err = check_xadd(env, insn);

                if (err)

                    return err;

                insn_idx++;

                continue;

            }



            /* check src1 operand */

[1]            err = check_reg_arg(regs, insn->src_reg, SRC_OP);

            if (err)

                return err;

            /* check src2 operand */

[1]            err = check_reg_arg(regs, insn->dst_reg, SRC_OP);

            if (err)

                return err;



            dst_reg_type = regs[insn->dst_reg].type;



            /* check that memory (dst_reg + off) is writeable */

[2]            err = check_mem_access(env, insn->dst_reg, insn->off,

                           BPF_SIZE(insn->code), BPF_WRITE,

                           insn->src_reg);

            if (err)

                return err;



            if (insn->imm == 0) {

                insn->imm = dst_reg_type;

            } else if (dst_reg_type != insn->imm &&

                   (dst_reg_type == PTR_TO_CTX ||

                    insn->imm == PTR_TO_CTX)) {

                verbose("same insn cannot be used with different pointers\n");

                return -EINVAL;

            }



        }
```

以上情况，如果采用`MOV`这样的赋值指令去读写的话，寄存器类型会判定为`IMM`，而拒绝。另外一种是用`BPF_FUNC_map_lookup_elem`这样的函数调用返回，再赋给某个寄存器，然后再进行读写。而这种方法会在赋值时被设定为`UNKNOWN`而拒绝读写。

## `__bpf_prog_run`

以上就是对于加载指令的全部检查，可以看到我们能想到的内存读写方法都是会被检测出来的。真正执行的时候代码在[`__bpf_prog_run`](<https://bbs.kanxue.com/elink@3fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4q4)9J5k6e0c8Q4x3X3f1I4x3e0m8Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3V1k6T1M7r3k6Q4x3V1k6U0L8%4u0W2i4K6u0W2j5#2)9J5x3@1H3I4z5e0f1>)中，其中可以看到所谓的各个寄存器和栈只是这个函数的局部变量：
