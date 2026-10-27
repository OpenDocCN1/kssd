
```
void CodeObfs::CleanImm(std::vector<cs_insn*> &vIns) {

    if (vIns.empty())

        return;

    std::vector<ContextInfo> v;

    ContextInfo ct;

    for (int i = X86_REG_INVALID; i <= X86_REG_ENDING; i++)

        ct[i] = -1;

    v.push_back(ct);

    for (unsigned int i = 0; i < vIns.size() - 1; i++) {

        for (int x = 0; x < vIns[i]->detail->regs_write_count; x++)

            ct[vIns[i]->detail->regs_write[x]] = i;

        for (int x = 0; x < vIns[i]->detail->x86.op_count; x++) {

            if (vIns[i]->detail->x86.operands[x].type == X86_OP_REG && vIns[i]->detail->x86.operands[x].access & CS_AC_WRITE)

                ct[vIns[i]->detail->x86.operands[x].reg] = i;

        }

        v.push_back(ct);

    }

    //我偷懒了，忽略了一些问题

    /*

    这是我偷懒忽略掉的情况

    mov ax,2010

    mov dl,al

    */

    std::vector<int> vReg;

    for (unsigned int i = 0; i < v.size(); i++) {

        if (!(vIns[i]->detail->x86.op_count != 0 && vIns[i]->detail->x86.operands[0].type == X86_OP_REG && vIns[i]->detail->x86.operands[0].access & CS_AC_WRITE))

            continue;

        if (vIns[i]->detail->x86.op_count != 1 && vIns[i]->detail->x86.operands[1].type != X86_OP_IMM)

            continue;

        int k = v[i][vIns[i]->detail->x86.operands[0].reg];

        if (k == -1 || vIns[k] == NULL)

            continue;

        if (strcmp(vIns[k]->mnemonic, "mov") || vIns[k]->detail->x86.operands[1].type != CS_OP_IMM)

            continue;

        DWORD dwImm1 = DWORD(vIns[k]->detail->x86.operands[1].imm);

        DWORD dwImm2 = DWORD(vIns[i]->detail->x86.operands[1].imm);

        if (!strcmp(vIns[i]->mnemonic, "ror")) {

            __asm {

                push ecx

                mov ecx,dwImm2

                ror dwImm1,cl

                pop ecx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "rol")) {

            __asm {

                push ecx

                mov ecx,dwImm2

                rol dwImm1,cl

                pop ecx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "add")) {

            __asm {

                push edx

                mov edx,dwImm2

                add dwImm1,edx

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "sub")) {

            __asm {

                push edx

                mov edx,dwImm2

                sub dwImm1,edx

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "xor")) {

            __asm {

                push edx

                mov edx,dwImm2

                xor dwImm1,edx

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "not")) {

            __asm {

                push edx

                mov edx,dwImm2

                not dwImm1

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "inc")) {

            __asm {

                push edx

                mov edx,dwImm2

                inc dwImm1

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "dec")) {

            __asm {

                push edx

                mov edx,dwImm2

                dec dwImm1

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "and")) {

            __asm {

                push edx

                mov edx,dwImm2

                and dwImm1,edx

                pop edx

            }

        }

        else if (!strcmp(vIns[i]->mnemonic, "or")) {

            __asm {

                push edx

                mov edx,dwImm2

                or dwImm1,edx

                pop edx

            }

        }

        else

            continue;

        char szIns[255];

        char szFmt[255] = {0};

        strcat_s(szFmt, sizeof(szFmt), "mov ");

        strcat_s(szFmt, sizeof(szFmt), cs_reg_name(_handle, vIns[k]->detail->x86.operands[0].reg));

        strcat_s(szFmt, sizeof(szFmt), ",0x%X");

        wsprintfA(szIns, szFmt, dwImm1);

        BYTE bCode[15];

        Asm(DWORD(vIns[k]->address), szIns, bCode);

        vIns[i] = NULL;

        cs_insn* insn;

        cs_disasm(_handle, bCode, 15, vIns[k]->address, 1, &insn);

        cs_free(vIns[k], 1);

        vIns[k] = insn;

    }

    return;

};
```

## 4）整合结果

```
    CleanOPCode(); //去除花指令

    std::vector<DWORD> vAccess; //未访问的代码块的起始地址

    std::vector<cs_insn*> vIns; //块代码

    DWORD pIns = 0x423002;

    BYTE bCode[15] = {0};

    cs_insn* pDasm;

    while (true) {

        GetData(pIns, bCode, 15);

        cs_disasm(_handle, bCode, 15, pIns, 1, &pDasm);

        pIns += pDasm->size;

        vIns.push_back(pDasm);

        if (IsJxInsn(pDasm)) {

            unsigned int k;

            do {

                k = vIns.size();

                CleanNop(vIns);

                CleanImm(vIns);

                CleanNop(vIns);

                CleanPattern(vIns);

            } while (k != vIns.size());

            break;

        }

    }

    for (unsigned int i = 0; i < vIns.size(); i++)

        printf("%s %s \r\n", vIns[i]->mnemonic, vIns[i]->op_str);

    system("pause");

    return;
```

![](https://bbs.kanxue.com/upload/attach/201908/783210_DQHSR7P6D8E9YXG.jpg)

对比给出的原版CrackMe，去混淆结果较为可观。

发生了灵异事件，附件一直上传不成功，我发到百度网盘了。

链接: dfbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4f1p5S2G2e0h3I4T1f1o6b7%4L8h3M7$3c8r3A6k6z5g2b7$3M7Y4A6Z5b7b7`.`. 提取码: hhqu

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
