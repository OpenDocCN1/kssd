
```cpp
FunctionNode X86Analysis::AnalysisFunction(DWORD64 begin, DWORD bufferSize, map<DWORD64, FunctionNode>* functionMap, map<DWORD64, FunctionNode>* excludeMap, DWORD64 pc)
{
    ud_t ud;
    ud_init(&ud);
    ud_set_mode(&ud, 32);
    ud_set_syntax(&ud, UD_SYN_INTEL);
    ud_set_input_buffer(&ud, (uint8_t*)begin, bufferSize);
    ud_set_pc(&ud, pc);

    InstrFlowNode jcc_max, jcc_flow;
    memset(&jcc_max, 0, sizeof(jcc_max));

    while (ud_disassemble(&ud))
    {
        jcc_flow = GetInstrNode(&ud);

        if (jcc_flow.isInvalid())
        {
            //遇到无效指令(可能花指令)，则停止分析该函数，返回前面分析完成的部分,可能函数长度为0
            FunctionNode function;
            function.memoryFileAddress = begin;
            function.loadImageAddress = pc;
            function.size = begin - (jcc_flow.memoryFileAddress - jcc_flow.insnLen);
            return function;
        }

        if (jcc_flow.isJcc && (jcc_flow.operatorType == UD_OP_JIMM))
        {
            if (jcc_flow.jmpMemoryFileAddress > jcc_max.jmpMemoryFileAddress)
            {
                //记录CFG流图中跳转目标地址最大的跳转指令
                jcc_max = jcc_flow;
            }
        }

        switch (jcc_flow.type)
        {
        case UD_Ijmp:
        {
            //如果无条件跳转目标地址小于函数起始或者当前跳转指令是函数第一条指令则视为结束
            //并把目标地址加入预分析函数节点
            if (jcc_flow.operatorType == UD_OP_JIMM)
            {
                if ((jcc_flow.jmpMemoryFileAddress < begin) || (jcc_flow.memoryFileAddress == begin))
                {
                    if (functionMap != nullptr)
                    {
                        //如果排除map中不存在已经分析过的函数则插入节点
                        if ((excludeMap != nullptr) && (!excludeMap->empty()))
                        {
                            if (excludeMap->find(jcc_flow.jmpMemoryFileAddress) == excludeMap->end())
                            {
                                FunctionNode node(jcc_flow.jmpMemoryFileAddress, jcc_flow.jmpLoadImageAddress);
                                functionMap->operator[](node.memoryFileAddress) = node;
                            }
                        }
                        else
                        {
                            FunctionNode node(jcc_flow.jmpMemoryFileAddress, jcc_flow.jmpLoadImageAddress);
                            functionMap->operator[](node.memoryFileAddress) = node;
                        }
                    }

                    return FunctionNode
                    (
                        begin,
                        pc,
                        jcc_flow.memoryFileAddress + jcc_flow.insnLen - begin
                    );
                }
            }
            break;
        }
        case UD_Icall:
        {
            if (functionMap != nullptr)
            {
                if (jcc_flow.operatorType == UD_OP_JIMM)
                {
                    if ((excludeMap != nullptr) && (!excludeMap->empty()))
                    {
                        //如果排除map中不存在已经分析过的函数则插入节点
                        if (excludeMap->find(jcc_flow.jmpMemoryFileAddress) == excludeMap->end())
                        {
                            //4字节立即数call
                            FunctionNode node(jcc_flow.jmpMemoryFileAddress, jcc_flow.jmpLoadImageAddress);
                            functionMap->operator[](node.memoryFileAddress) = node;
                        }
                    }
                    else
                    {
                        //4字节立即数call
                        FunctionNode node(jcc_flow.jmpMemoryFileAddress, jcc_flow.jmpLoadImageAddress);
                        functionMap->operator[](node.memoryFileAddress) = node;
                    }
                }
            }
            break;
        }

        case UD_Iret:
        {
            if (jcc_max.jmpMemoryFileAddress > jcc_flow.memoryFileAddress)
            {
                //ret之后还有CFG流
                DWORD skip_bytes = jcc_max.jmpLoadImageAddress - ud.pc;
                ud_input_skip(&ud, skip_bytes);
                ud_set_pc(&ud, jcc_max.jmpLoadImageAddress);
            }
            else
            {
                return FunctionNode
                (
                    begin,
                    pc,
                    jcc_flow.memoryFileAddress + jcc_flow.insnLen - begin
                );
            }
            break;
        }

        case UD_Inop:
        case UD_Iint3:
        {
            if (ud_insn_mnemonic(&ud) == UD_Iint3)
            {
                //必须至少出现连续两个CC指令才说明识别到函数末尾
                if (*((ud_insn_ptr(&ud) + 1)) != 0xCC)
                {
                    break;
                }
            }
            return FunctionNode
            (
                begin,
                pc,
                jcc_flow.memoryFileAddress - begin
            );
        }
        case UD_Iadd:
        {
            if (!memcmp(ud_insn_hex(&ud), "0000", 4))
            {
                return FunctionNode
                (
                    begin,
                    pc,
                    jcc_flow.memoryFileAddress - begin
                );
            }
            break;
        }

        default:
        {
        }
        }

        //如果接下来的代码是函数的开始特征
        DWORD64 ptr = jcc_flow.memoryFileAddress + jcc_flow.insnLen;
        if (LookNextBegin(ptr))
        {
            //如果排除map中不存在已经分析过的函数则插入节点
            if ((excludeMap != nullptr) && (!excludeMap->empty()))
            {
                if (excludeMap->find(ptr) == excludeMap->end())
                {
                    FunctionNode node(ptr, jcc_flow.loadImageAddress + jcc_flow.insnLen);
                    functionMap->operator[](ptr) = node;
                }
            }
            else
            {
                FunctionNode node(ptr, jcc_flow.loadImageAddress + jcc_flow.insnLen);
                functionMap->operator[](ptr) = node;
            }

            return FunctionNode
            (
                begin,
                pc,
                jcc_flow.memoryFileAddress + jcc_flow.insnLen - begin
            );
        }
    }

    return FunctionNode
    (
        begin,
        pc,
        bufferSize
    );
}
```

分析`procmon.exe`的`winmain`函数和IDA对比的效果如图

![](https://bbs.kanxue.com/upload/attach/202001/784599_B7CPYKQHTF94HZS.jpg)
![](https://bbs.kanxue.com/upload/attach/202001/784599_YDASSH7HVG2HE78.jpg)

根据IDA的识别计算一下函数大小`0x0045E6B4-0x0045D840=3700`，和自己程序的识别结果一致

![](https://bbs.kanxue.com/upload/attach/202001/784599_FBVVKW9JDGD6M69.jpg)

---

# 函数粉碎

有了上面得到的信息就能进行粉碎了，用前面讲的方法将函数进行混淆得到新的代码块。再**找到`reloc`段的前面一个段**，**向下合并`reloc`段**，**创建一个新的`text`段**，把混淆代码放进去。最后**在新`text`段后面创建`reloc`段**，修复重定位信息。

**混淆之前必须先扫描记录当前函数的所有重定位信息**，混淆过程中将原始重定位信息和新的重定位信息联系在一起，以便后面进行重定位修复。

**部分代码实现：**
