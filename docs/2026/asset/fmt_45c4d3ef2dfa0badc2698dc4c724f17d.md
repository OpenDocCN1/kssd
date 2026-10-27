
```python
import angr
import logging
import re
import claripy
from keystone import *
import sys
import time

def fill_nops(addr, size):
    offset = addr - base_addr
    content[offset:offset + size] = b'\x90' * size

def fill_jmp(src, dest):
    offset = src - base_addr
    if dest != src + 5:
        content[offset] = 0xE9
        content[offset + 1:offset + 5] = (dest - src - 5).to_bytes(4, 'little', signed=True)
    else:
        fill_nops(src, 5)

def get_jx_opcode(jx_type):
    ks = Ks(KS_ARCH_X86, KS_MODE_64)
    code, count = ks.asm(f'{jx_type} 0xFFFFFFFF')
    return b''.join(map(lambda x: x.to_bytes(1, sys.byteorder), code[0:2]))

def fill_jx(src, dest, cmov_type):
    offset = src - base_addr
    content[offset:offset + 2] = get_jx_opcode(cmov_type.replace('cmov', 'j'))
    content[offset + 2:offset + 6] = (dest - src - 6).to_bytes(4, 'little', signed=True)

class SuperBlock:
    def __init__(self, block_addr):
        self.addr = block_addr
        self.blocks = []
        self.size = 0

    def insert_block(self, block):
        self.blocks.append(block)
        self.size += block.size

    @property
    def terminator(self):
        return self.blocks[-1].capstone.insns[-1]

    @property
    def successor(self):
        terminator = self.terminator
        try:
            return int(terminator.op_str, 16)
        except:
            return None

    @property
    def insns(self):
        insns = []
        for block in self.blocks:
            insns += block.capstone.insns
        return insns

    def __repr__(self):
        return "<SuperBlock %#08x->%#08x>" % (self.addr, self.addr + self.size)

    def __hash__(self):
        return hash(('superblock', self.addr))

class FunctionCFG:
    def __init__(self, proj, func_addr, func_end):
        self.proj = proj
        self.addr = func_addr
        self.size = func_end - func_addr

    def get_super_block(self, block_addr):
        super_block = SuperBlock(block_addr)
        cur_addr = block_addr
        while True:
            block = self.proj.factory.block(cur_addr)
            if block.size == 0:
                return super_block
            insns = block.capstone.insns
            super_block.insert_block(block)
            cur_addr += block.size
            if insns[-1].mnemonic.startswith('j') or insns[-1].mnemonic == 'ret' or insns[-1].address in call_exit:
                return super_block

    def dump_all_blocks(self):
        curr_addr = self.addr
        blocks = []
        while curr_addr - self.addr < self.size:
            super_block = self.get_super_block(curr_addr)
            if super_block.size == 0:
                curr_addr += 1
                continue
            blocks.append(super_block)
            curr_addr += super_block.size
        return blocks

    def anaylse_blocks(self):
        all_blocks = self.dump_all_blocks()
        relevant_blocks = []
        useless_blocks = []
        for block in all_blocks:
            if block.addr != self.addr and \
               (block.successor is not None or block.terminator.mnemonic == 'ret' or block.terminator.mnemonic == 'call' or block.addr in additional_map.values()):
                relevant_blocks.append(block)
            elif block.addr != self.addr:
                useless_blocks.append(block)
        return relevant_blocks, useless_blocks

def skip_call(state):
    proj.hook(state.addr, angr.SIM_PROCEDURES["stubs"]["ReturnUnconstrained"](), replace=True)

# Map node onto key and return a dict
def map_k2n():
    state = proj.factory.call_state(addr=func_addr)
    # Skip call
    state.inspect.b('call', when=angr.BP_BEFORE, action=skip_call)
    simgr = proj.factory.simgr(state)
    simgr.explore(find=dispatcher_node)
    found = simgr.found[0]
    # Symbolize eax
    key = claripy.BVS('key', 32)
    found.regs.eax = key
    simgr = proj.factory.simgr(found)
    visited = []
    k2n_map = {}
    while len(simgr.active):
        # Filter all visited states to prevent from sticking in a loop
        simgr.move(from_stash='active', to_stash='deadended', filter_func=lambda state: state.addr in visited)
        for active in simgr.active:
            visited.append(active.addr)
            if active.addr in relevant_nodes:
                key_val = active.solver.eval(key)
                k2n_map[key_val] = active.addr
                logger.debug('[*] Mapped node(%#x)->key(%#x) [%d/%d]' % (active.addr, key_val, len(k2n_map), len(relevant_nodes)))
        simgr.step()
    return k2n_map

def fix_branch():
    # Nop all useless blocks
    for block in useless_blocks:
        fill_nops(block.addr, resize_map[block.addr] if block.addr in resize_map else block.size)
    relevant_blocks.append(prologue_block)
    for block in relevant_blocks:
        key_reg, sub_key_reg = 'ecx', 'eax'
        key, sub_key = None, None
        cmov_mnemonic = None
        for insn in block.insns:
            mnemonic, op_str = insn.mnemonic, insn.op_str
            if mnemonic == 'mov':
                if re.match(rf'{key_reg}, 0x[0-9a-f]+', op_str):
                    tmp_key = int(op_str.replace(f'{key_reg}, ', ''), 16)
                    if tmp_key in k2n_map:
                        key = tmp_key
                elif re.match(rf'{sub_key_reg}, 0x[0-9a-f]+', op_str):
                    tmp_sub_key = int(op_str.replace(f'{sub_key_reg}, ', ''), 16)
                    if tmp_sub_key in k2n_map:
                        sub_key = tmp_sub_key
            elif mnemonic.startswith('cmov') and op_str == f'{key_reg}, {sub_key_reg}':
                cmov_mnemonic = mnemonic
        terminator = block.terminator
        if block.addr == prologue_block.addr:
            logger.debug('[*] Progolue node(%#x) has one successor(%#x)' % (block.addr, k2n_map[sub_key]))
            fill_jmp(terminator.address, k2n_map[sub_key])
        elif key is not None and sub_key is None:
            logger.debug('[*] Node(%#x) has one successor(%#x)' % (block.addr, k2n_map[key]))
            fill_jmp(terminator.address, k2n_map[key])
        elif key is not None and sub_key is not None:
            cur_block = block
            while cur_block.successor is not None and cmov_mnemonic is None:
                cur_block = cfg.get_super_block(cur_block.successor)
                for insn in cur_block.insns:
                    mnemonic, op_str = insn.mnemonic, insn.op_str
                    if mnemonic.startswith('cmov') and op_str == f'{key_reg}, {sub_key_reg}':
                        cmov_mnemonic = mnemonic
            if cmov_mnemonic is not None:
                logger.debug('[*] Node(%#x) has two successors(%#x, %#x)' % (block.addr, k2n_map[key], k2n_map[sub_key]))
                fill_jx(terminator.address, k2n_map[sub_key], cmov_mnemonic)
                fill_jmp(terminator.address + 6, k2n_map[key])
            else:
                logger.error('[*] Error at block(%#x)' % block.addr)
                exit(0)
        else:
            logger.debug('[*] Node(%#x) has no successor' % block.addr)

if __name__ == '__main__':
    # Initilization
    logging.getLogger('cle').setLevel(logging.ERROR)
    logging.getLogger('angr').setLevel(logging.ERROR)
    logger = logging.getLogger('deobfu')
    logger.setLevel(logging.DEBUG)
    logger.addHandler(logging.FileHandler(filename='%s.log' % time.strftime("%Y-%m-%d-%H-%M-%S", time.localtime())))
    filename = 'crackme-pls.bin.recover'
    proj = angr.Project(filename)
    base_addr = proj.loader.main_object.mapped_base
    reverse_plt = proj.loader.main_object.reverse_plt
    with open(filename, 'rb') as file:
        content = bytearray(file.read())
    # Prepare sigificant arguments
    func_addr, func_end = 0x402B70, 0x403D81
    call_exit = [0x403D3D, 0x403D7E]
    additional_map = {0x326BF726: 0x403243, 0x3311C734: 0x4035A1, 0x703837C4: 0x4037A0}
    resize_map = {0x403230: 19, 0x40358E: 19, 0x403792: 14}
    # Analyse function CFG in a very crude way
    cfg = FunctionCFG(proj, func_addr, func_end)
    # Get prologue block at func addr
    prologue_block = cfg.get_super_block(func_addr)
    # The successor of prologue block is dispatcher block
    dispatcher_node = prologue_block.successor
    relevant_blocks, useless_blocks = cfg.anaylse_blocks()
    for addr in additional_map.values():
        relevant_blocks.append(cfg.get_super_block(addr))
    # Convert block list to addr list
    relevant_nodes = [block.addr for block in relevant_blocks]
    # Map node onto key
    k2n_map = map_k2n()
    k2n_map.update(additional_map)
    # Fix control flow
    fix_branch()
    with open(filename + '.recover_', 'wb') as file:
        file.write(content)
```
