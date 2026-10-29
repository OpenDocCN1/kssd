
```
void T32_LDR_LIT::decode(T32_STRUCT_LDR_LIT *inst) {

    DECODE_OP;

    DECODE_RT(Reg);

    s = S(inst->S);

    offset = getImmPCOffset();

}



void T32_LDR_LIT::assembler() {

    SET_OPCODE(LDR_LIT);

    ENCODE_OP;

    ENCODE_RT;

    get()->S = s;

    if (offset >= 0) {

        get()->U = add;

        get()->imm12 = static_cast<InstT32>(offset);

    } else {

        get()->U = cmp;

        get()->imm12 = static_cast<InstT32>(-offset);

    }

}
```

汇编器：基本是对每个指令封装的调用

```
void AssemblerA32::Mov(RegisterA32 &rd, U16 imm16) {

    Emit(reinterpret_cast<Unit<Base>*>(new INST_T32(MOV_MOVT_IMM)(INST_T32(MOV_MOVT_IMM)::MOV, rd, imm16)));

}



void AssemblerA32::Movt(RegisterA32 &rd, U16 imm16) {

    Emit(reinterpret_cast<Unit<Base>*>(new INST_T32(MOV_MOVT_IMM)(INST_T32(MOV_MOVT_IMM)::MOVT, rd, imm16)));

}



void AssemblerA32::Mov(RegisterA32 &rd, U32 imm32) {

    U16 immL = BITS16L(imm32);

    U16 immH = BITS16H(imm32);

    Mov(rd, immL);

    Movt(rd, immH);

}
```

解析器：

```
void Arm64Decoder::decode(void *codeStart, Addr codeLen, InstVisitor &visitor, bool onlyPcRelInst) {

    InstA64 *pc = reinterpret_cast<InstA64 *>(codeStart);

    Addr endAddr = (Addr) codeStart + codeLen;

    Unit<Base>* unit = nullptr;

    while((Addr) pc < endAddr) {

        // pc relate insts

        CASE(B_BL)

        CASE(B_COND)

        CASE(CBZ_CBNZ)

        CASE(TBZ_TBNZ)

        CASE(LDR_LIT)

        CASE(ADR_ADRP)

        if (onlyPcRelInst)

            goto label_matched;

        CASE(MOV_WIDE)

        CASE(MOV_REG)

        CASE(LDR_IMM)

        CASE(LDR_UIMM)

        CASE(LDRSW_IMM)

        CASE(LDRSW_UIMM)

        CASE(STR_UIMM)

        CASE(STR_IMM)

        CASE(BR_BLR_RET)

        CASE(SUB_EXT_REG)

        CASE(SVC)

        CASE(EXCEPTION_GEN)

        label_matched:

        if (unit == nullptr) {

            unit = reinterpret_cast<Unit<Base> *>(new INST_A64(UNKNOW)(*reinterpret_cast<STRUCT_A64(UNKNOW) *>(pc)));

        }

        if (!visitor.visit(unit, pc)) {

            break;

        }

        pc = reinterpret_cast<InstA64 *>((Addr)pc + unit->size());

        unit = nullptr;

    }

}
```

指令修复

```
void* CodeRelocateA64::relocate(Instruction<Base> *instruction, void *toPc) throw(ErrorCodeException) {

    void* curPc = __ getPC();



    //insert later bind labels

    __ Emit(getLaterBindLabel(curOffset));



    if (!instruction->pcRelate()) {

        __ Emit(instruction);

        instruction->ref();

        return curPc;

    }

    switch (instruction->instCode()) {

        CASE(B_BL)

        CASE(B_COND)

        CASE(TBZ_TBNZ)

        CASE(CBZ_CBNZ)

        CASE(LDR_LIT)

        CASE(ADR_ADRP)

        default:

            __ Emit(instruction);

            instruction->ref();

    }

    return curPc;

}





IMPL_RELOCATE(B_BL) {



    if (inRelocateRange(inst->offset, sizeof(InstA64))) {

        inst->ref();

        inst->bindLabel(*getLaterBindLabel(inst->offset + curOffset));

        __ Emit(reinterpret_cast<Instruction<Base>*>(inst));

        return;

    }



    Addr targetAddr = inst->getImmPCOffsetTarget();



    if (inst->op == inst->BL) {

        Addr lr = reinterpret_cast<Addr>(toPc);

        lr += 4 * 4; // MovWide * 4;

        lr += 4 * 4; // MovWide * 4;

        lr += 4; // Br

        __ Mov(LR, lr);

    }

    __ Mov(IP1, targetAddr);

    __ Br(IP1);

}
```

# 结尾

其实 Native Inline Hook 与 ART Hook 还是有很大不同的，其本身并没有什么深度，只要你多看看手册 ==，当然 native hook 目前测试的不多，欢迎测试一起完善。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
