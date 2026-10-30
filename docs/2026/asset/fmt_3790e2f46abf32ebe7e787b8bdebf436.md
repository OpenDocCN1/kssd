
```c
#define MAX_CALL_FRAMES 8

struct bpf_verifier_state {
	struct bpf_func_state *frame[MAX_CALL_FRAMES]; // 保存调用栈
	struct bpf_verifier_state *parent;

	/*
	 * 'branches' field is the number of branches left to explore:
	 * 0 - all possible paths from this state reached bpf_exit or
	 * were safely pruned
	 * 1 - at least one path is being explored.
	 * This state hasn't reached bpf_exit
	 * 2 - at least two paths are being explored.
	 * This state is an immediate parent of two children.
	 * One is fallthrough branch with branches==1 and another
	 * state is pushed into stack (to be explored later) also with
	 * branches==1. The parent of this state has branches==1.
	 * The verifier state tree connected via 'parent' pointer looks like:
	 * 1
	 * 1
	 * 2 -> 1 (first 'if' pushed into stack)
	 * 1
	 * 2 -> 1 (second 'if' pushed into stack)
	 * 1
	 * 1
	 * 1 bpf_exit.
	 *
	 * Once do_check() reaches bpf_exit, it calls update_branch_counts()
	 * and the verifier state tree will look:
	 * 1
	 * 1
	 * 2 -> 1 (first 'if' pushed into stack)
	 * 1
	 * 1 -> 1 (second 'if' pushed into stack)
	 * 0
	 * 0
	 * 0 bpf_exit.
	 * After pop_stack() the do_check() will resume at second 'if'.
	 *
	 * If is_state_visited() sees a state with branches > 0 it means
	 * there is a loop. If such state is exactly equal to the current
	 * state it's an infinite loop. Note states_equal() checks for states
	 * equvalency, so two states being 'states_equal' does not mean
	 * infinite loop. The exact comparison is provided by
	 * states_maybe_looping() function. It's a stronger pre-check and
	 * much faster than states_equal().
	 *
	 * This algorithm may not find all possible infinite loops or
	 * loop iteration count may be too high.
	 * In such cases BPF_COMPLEXITY_LIMIT_INSNS limit kicks in.
	 */
	u32 branches;
	u32 insn_idx;
	u32 curframe;
	u32 active_spin_lock;
	bool speculative;

	/* first and last insn idx of this verifier state */
	u32 first_insn_idx;
	u32 last_insn_idx;

	/*
	 * jmp history recorded from first to last.
	 * backtracking is using it to go from last to first.
	 * For most states jmp_history_cnt is [0-3].
	 * For loops can go up to ~40.
	 */
	struct bpf_idx_pair *jmp_history;
	u32 jmp_history_cnt;
};
```

- `branch`字段代表的是剩余探索的分支数量。
- `branch = 0`，从这个状态出发的所有可能路径都达到了`bpf_exit`，或者已经被修剪了。
- `branch = 1`，至少有一条路径正在被探索，这个状态还没有达到`bpf_exit`。
- `branch = 2`，至少有两条路径正在被探索，这个状态是两个子节点的直接父节点。

其中一个是`FALLTHROUGH`，也就是顺序执行的状态树边。另一个（稍后将被`explored`的）状态也被压入栈中，也有`branch = 1`。它的父状态结点也有`branch = 1`。

- 举个例子，如果我们通过`if`的一条路径走到`bpf_exit`了，会调用`update_branch_counts()`回溯更新每个状态树节点`branches`的值。`if`的分支节点之间通过`struct bpf_verifier_state *parent;`指针相连。

![](upload/attach/202106/876323_MS7383XC36RMF3Q.png)

## `bpf_reg_state`

维护了 BPF 寄存器的状态。

```c
struct bpf_reg_state {
	/* Ordering of fields matters. See states_equal() */

	enum bpf_reg_type type;

	union {
		/* valid when type == PTR_TO_PACKET */
		u16 range;

		/*
		 * valid when type == CONST_PTR_TO_MAP | PTR_TO_MAP_VALUE |
		 * PTR_TO_MAP_VALUE_OR_NULL
		 */
		struct bpf_map *map_ptr;

		u32 btf_id; /* for PTR_TO_BTF_ID */

		/* Max size from any of the above. */
		unsigned long raw;
	};

	/* Fixed part of pointer offset, pointer types only */
	s32 off;

	/*
	 * For PTR_TO_PACKET, used to find other pointers with the same variable
	 * offset, so they can share range knowledge.
	 * For PTR_TO_MAP_VALUE_OR_NULL this is used to share which map value we
	 * came from, when one is tested for != NULL.
	 * For PTR_TO_SOCKET this is used to share which pointers retain the
	 * same reference to the socket, to determine proper reference freeing.
	 */
	u32 id;

	/*
	 * PTR_TO_SOCKET and PTR_TO_TCP_SOCK could be a ptr returned
	 * from a pointer-cast helper, bpf_sk_fullsock() and
	 * bpf_tcp_sock().
	 *
	 * Consider the following where "sk" is a reference counted
	 * pointer returned from "sk = bpf_sk_lookup_tcp();":
	 *
	 * 1: sk = bpf_sk_lookup_tcp();
	 * 2: if (!sk) { return 0; }
	 * 3: fullsock = bpf_sk_fullsock(sk);
	 * 4: if (!fullsock) { bpf_sk_release(sk); return 0; }
	 * 5: tp = bpf_tcp_sock(fullsock);
	 * 6: if (!tp) { bpf_sk_release(sk); return 0; }
	 * 7: bpf_sk_release(sk);
	 * 8: snd_cwnd = tp->snd_cwnd; // verifier will complain
	 *
	 * After bpf_sk_release(sk) at line 7, both "fullsock" ptr and
	 * "tp" ptr should be invalidated also. In order to do that,
	 * the reg holding "fullsock" and "sk" need to remember
	 * the original refcounted ptr id (i.e. sk_reg->id) in ref_obj_id
	 * such that the verifier can reset all regs which have
	 * ref_obj_id matching the sk_reg->id.
	 *
	 * sk_reg->ref_obj_id is set to sk_reg->id at line 1.
	 * sk_reg->id will stay as NULL-marking purpose only.
	 * After NULL-marking is done, sk_reg->id can be reset to 0.
	 *
	 * After "fullsock = bpf_sk_fullsock(sk);" at line 3,
	 * fullsock_reg->ref_obj_id is set to sk_reg->ref_obj_id.
	 *
	 * After "tp = bpf_tcp_sock(fullsock);" at line 5,
	 * tp_reg->ref_obj_id is set to fullsock_reg->ref_obj_id
	 * which is the same as sk_reg->ref_obj_id.
	 *
	 * From the verifier perspective, if sk, fullsock and tp
	 * are not NULL, they are the same ptr with different
	 * reg->type. In particular, bpf_sk_release(tp) is also
	 * allowed and has the same effect as bpf_sk_release(sk).
	 */
	u32 ref_obj_id;

	/*
	 * For scalar types (SCALAR_VALUE), this represents our knowledge of
	 * the actual value.
	 * For pointer types, this represents the variable part of the offset
	 * from the pointed-to object, and is shared with all bpf_reg_states
	 * with the same id as us.
	 */
	struct tnum var_off;

	/*
	 * Used to determine if any memory access using this register will
	 * result in a bad access.
	 * These refer to the same value as var_off, not necessarily the actual
	 * contents of the register.
	 */
	s64 smin_value; /* minimum possible (s64)value */
	s64 smax_value; /* maximum possible (s64)value */
	u64 umin_value; /* minimum possible (u64)value */
	u64 umax_value; /* maximum possible (u64)value */

	/* parentage chain for liveness checking */
	struct bpf_reg_state *parent;

	/*
	 * Inside the callee two registers can be both PTR_TO_STACK like
	 * R1=fp-8 and R2=fp-8, but one of them points to this function stack
	 * while another to the caller's stack. To differentiate them 'frameno'
	 * is used which is an index in bpf_verifier_state->frame[] array
	 * pointing to bpf_func_state.
	 */
	u32 frameno;

	/*
	 * Tracks subreg definition. The stored value is the insn_idx of the
	 * writing insn. This is safe because subreg_def is used before any insn
	 * patching which only happens after main verification finished.
	 */
	s32 subreg_def;

	enum bpf_reg_liveness live;

	/* if (!precise && SCALAR_VALUE) min/max/tnum don't affect safety */
	bool precise;
};
```

## `bpf_reg_type`

```c
/* types of values stored in eBPF registers */

/*
 * Pointer types represent:
 * pointer
 * pointer + imm
 * pointer + (u16) var
 * pointer + (u16) var + imm
 * if (range > 0) then [ptr, ptr + range - off) is safe to access
 * if (id > 0) means that some 'var' was added
 * if (off > 0) means that 'imm' was added
 */

enum bpf_reg_type {
	NOT_INIT = 0, /* nothing was written into register */
	SCALAR_VALUE, /* reg doesn't contain a valid pointer */
	PTR_TO_CTX, /* reg points to bpf_context */
	CONST_PTR_TO_MAP, /* reg points to struct bpf_map */
	PTR_TO_MAP_VALUE, /* reg points to map element value */
	PTR_TO_MAP_VALUE_OR_NULL, /* points to map elem value or NULL */
	PTR_TO_STACK, /* reg == frame_pointer + offset */
	PTR_TO_PACKET_META, /* skb->data - meta_len */
	PTR_TO_PACKET, /* reg points to skb->data */
	PTR_TO_PACKET_END, /* skb->data + headlen */
	PTR_TO_FLOW_KEYS, /* reg points to bpf_flow_keys */
	PTR_TO_SOCKET, /* reg points to struct bpf_sock */
	PTR_TO_SOCKET_OR_NULL, /* reg points to struct bpf_sock or NULL */
```
