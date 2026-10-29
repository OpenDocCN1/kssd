
```python
def hook_code(uc, address, size, user_data):
    global real_blocks
    global csel_addrs
    global list_trace
    global startaddr
    global debugflag
    global isSucess
    global distAddr
    global branch_control
    global list_blocks

    ban_ins = ["bl"]

    if isSucess:
        mu.emu_stop()
        # raw_input()
        return

    if address > end:
        uc.emu_stop()
        return

    for ins in md.disasm(bin1[address:address + size], address):
        print(">>> Tracing instruction at 0x%x, instruction size = 0x%x" % (address, size))
        print(">>> 0x%x:\t%s\t%s" % (ins.address, ins.mnemonic, ins.op_str))
        print

        if address in real_blocks:
            if list_trace.has_key(address):
                print("sssssss")
                ch = raw_input(
                    "This maybe a fake block. codesign:%s "
                    % get_code_sign(list_blocks[address])
                )
                uc.emu_stop()
            else:
                list_trace[address] = 1

        if address in real_blocks and address != startaddr:
            isSucess = True
            distAddr = address
            print("find:%x" % address)
            # uc.reg_write(UC_ARM64_REG_PC, 0xffffffff)
            uc.emu_stop()
            return

        flag_pass = False

        for b in ban_ins:
            if ins.mnemonic.find(b) != -1:
                flag_pass = True
                break

        if ins.op_str.find("[") != -1:
            if ins.op_str.find("[sp") == -1:
                flag_pass = True

        for op in ins.operands:
            if op.type == ARM64_OP_MEM:
                addr = 0

                if op.value.mem.base != 0:
                    addr += mu.reg_read(
                        reg_ctou(ins.reg_name(op.value.mem.base))
                    )
                elif op.value.index != 0:
                    addr += mu.reg_read(
                        reg_ctou(ins.reg_name(op.value.mem.index))
                    )
                elif op.value.disp != 0:
                    addr += op.value.disp

                if addr >= 0x80000000 and addr < 0x80000000 + 0x10000 * 8:
                    flag_pass = False

        if flag_pass:
            print(
                "will pass 0x%x:\t%s\t%s"
                % (ins.address, ins.mnemonic, ins.op_str)
            )
            uc.reg_write(UC_ARM64_REG_PC, address + size)
            return

        # breaks 0x31300
        if address in [0xB72EC] or debugflag:
            debugflag = True
            print("0x%x:\t%s\t%s" % (ins.address, ins.mnemonic, ins.op_str))

            while True:
                c = raw_input(">")

                if c == "":
                    break

                if c == "s":
                    uc.emu_stop()
                    return

                if c == "r":
                    debugflag = False
                    break

                if c[0] == "!":
                    reg = reg_ctou(c[1:])
                    print(
                        "%s=%x (%d)"
                        % (c[1:], mu.reg_read(reg), mu.reg_read(reg))
                    )
                    continue

        if ins.mnemonic == "ret":
            uc.reg_write(UC_ARM64_REG_PC, 0)
            isSucess = False
            print("ret ins..")
            mu.emu_stop()

        if ins.mnemonic == "csel":  # ollvm branch
            print(
                "csel 0x%x:\t%s\t%s"
                % (ins.address, ins.mnemonic, ins.op_str)
            )

            regs = [reg_ctou(x) for x in ins.op_str.split(", ")]
            assert len(regs) == 4

            v1 = uc.reg_read(regs[1])
            v2 = uc.reg_read(regs[2])

            if branch_control == 1:
                uc.reg_write(regs[0], v1)
            else:
                uc.reg_write(regs[0], v2)

            uc.reg_write(UC_ARM64_REG_PC, address + size)
```

## 绘制流程图

使用 Graphviz 库绘制流程图。

```python
def draw():
    queue = []
    dot = Digraph(comment="The Round Table")
    dot.attr("node", shape="box")
    queue.append(offset)
    check = {}

    while len(queue) > 0:
        pc = queue.pop()

        if check.has_key(pc):
            continue

        check[pc] = 1
        dot.node(hex(pc), list_blocks[pc]["ins"])

        for i in flow[pc]:
            if i != None:
                dot.edge(hex(pc), hex(i), constraint="true")
                queue.append(i)

    dot.render("test-output/round-table.gv", view=True)
```

## 比较完整的代码
