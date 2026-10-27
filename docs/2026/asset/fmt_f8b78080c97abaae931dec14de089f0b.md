
```cpp
bool VMInstructionDecoder(TInstructionBuffer* InstructionPtr, byte *VMEIP_RealAddr) { // .text:00401000
TInstrTag InstrType;
byte LowNib,HiNib;
AddrSize;
JccIndex;
dword *ExaminedDwords;
TempInstrSize;
dd* TempPtr;
ParamsCount;
Temp;
wTemp;
bTemp;
memset(InstrBuf,0,0x13*4);
InstructionPtr->WorkSubField = &InstructionPtr->ParamDest; // set which is the first decoded param
/* 1. set types */
InstrType = VMEIP_RealAddr[0];//*(byte *)VMEIP_RealAddr
InstructionPtr->InstrType = InstrType.InstrType;//b&0x3F; // ==00111111b
swith(InstrType.AddSize) { //swith(InstrType>>6) { // the sub is needed for setting flags!
case 0: AddrSize = 1; break;
case 1: AddrSize = 2; break;
case 2: AddrSize = 4; break;
default: return 0;
};
InstructionPtr->OperandSize = AddrSize;
ParamIdx= InstructionPtr->InstrType; // InstructionPtr->InstrType<<4; // *structure size
if (ParamTable[ParamIdx].ID==0x33) return 0; // 0x33 entry has no associated instruction
if ( (char)ParamTable[ParamIdx].ParamDest==4 && AddrSize!=4) return 0;
InstructionPtr->InstrID = ParamTable[ParamIdx].ID; // Jump Address!
/* 2. cycle thru instruction parameters as from Instruction Decoder's Table, and fill buffer */
ExaminedParams = 0;
TempInstrSize = 1; //0 was already used for getting here!!
ParamsCount = 0; // decode the first param, so!
while (ParamTable[ParamIdx].Params[ParamsCount]!=0) { // .text:004010B1 param decoding loop
//ParamsValue = ParamTable[ParamIdx].Params[ParamsCount].ID;
LowNib_RegIdx = VMEIP_RealAddr[TempInstrSize]&0x0F;
HiNib_AddrMode = VMEIP_RealAddr[TempInstrSize]>>4;
InstructionPtr->WorkSubField[ParamsCount].AddressType = HiNib;
/* 3. set up instruction sub-type (Jcc Type in this VM) */
InstructionPtr->WorkSubField[ParamsCount].Decoder_ParamsValue = ParamTable[ParamIdx].Params[ParamsCount];
switch (HiNib_AddrMode) { // NOTE: switch on decoded address type!!
case vmaRegister: // 0
case vmaRegisterAddress: // 1
InstructionPtr->WorkSubField[ParaCount].RegisterIdx = LowNib_RegIdx;
TempInstrSize++;
break;
case vmaVMValue_orC4__or_displacement: // 3 .text:00401134
if ( (char)ParamTable[ParamIdx].Params[ParamsCount]==2) return 0;
TempInstrSize++;
switch(InstructionPtr->OperandSize) {
case 1:
bTemp = ((byte *)VMEIP_RealAddr)[TempInstrSize];
InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)bTemp;
TempInstrSize++;
break;
case 2:
// this might be an instrinsic inline function, due to code shape (compiler didnt recon param 2 was 0)
wTemp = ((word *)VMEIP_RealAddr)[TempInstrSize];
wTemp = SWAP(wTemp);
InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)wTemp;
TempInstrSize+=2;
case 4:
break;
default: return 0;
}
case vmaDirectAddress: // 2
if (HiNib_AddrMode==vmaDirectAddress) { //added by me to keep flow
TempInstrSize++;
if (InstructionPtr->OperandSize!=4) return 0;
}
// .text:00401101 common code to case 2 and 3 here...
Temp = ((dword *)VMEIP_RealAddr)[TempInstrSize];
Temp = SWAP(Temp);
InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)Temp;
TempInstrSize+=4;
break;
default:
return 0;
}
ParamsCount++; // next param data!
ExaminedParams++;
if (ParaCount>=3) break; // max 32 bytes fetched this way
};
InstructionPtr->InstructionParamsCount = ExaminedParams;
InstructionPtr->InstrSize = TempInstrSize;
return TempInstrSize;
}
```

它简单的填充常见参数到指令的初始化部分，然后通过使用参数的解码器结构开始循环。这个结构持有指令使用的参数，以及某个指令子类型，看上去和`Jcc`中使用的标志有关。最初我把这个字段叫做“`JccType`”，然后我改成了另一个通用的“`InstrType`”，因为理论上它有一个更通用的用法正如代码所揭露的。

对参数的主循环是执行于`TParamDecoding.Params[]`数组之上。零值就意味着指令没有使用参数。依赖于参数的地址类型，它取到正确的数据并且填充`TInstructionBuffer`结构。作为备注，`TParamDecoding`数组开始于`.data:00407030 ParamTable`正好结束于VM指令表之前。

## VM 指令

指令集包含最常见的x86指令，和一些消息。这是这个机器实现的VM操作码列表：

```
.data:00407430 VM_Opcode_Table dd offset VM_MOV ; DATA XREF: _main+162#r
.data:00407434 dd offset VM_Multiple_op2
.data:00407438 dd offset VM_Multiple_op2
.data:0040743C dd offset VM_Multiple_op2
.data:00407440 dd offset VM_Multiple_op2
.data:00407444 dd offset VM_Multiple_op2
.data:00407448 dd offset VM_PUSH
.data:0040744C dd offset VM_POP
.data:00407450 dd offset VM_JMP
.data:00407454 dd offset VM_CALL
.data:00407458 dd offset VM_LOOP
.data:0040745C dd offset VM_RET
.data:00407460 dd offset VM_Multiple_op2
.data:00407464 dd offset VM_INC
.data:00407468 dd offset VM_DEC
.data:0040746C dd offset VM_NOP
.data:00407470 dd offset VM_Jcc
.data:00407474 dd offset VM_Jcc
.data:00407478 dd offset VM_Jcc
.data:0040747C dd offset VM_Jcc
.data:00407480 dd offset VM_Jcc
.data:00407484 dd offset VM_Jcc
.data:00407488 dd offset VM_Jcc
.data:0040748C dd offset VM_Jcc
.data:00407490 dd offset VM_NOP
.data:00407494 dd offset VM_NOP
.data:00407498 dd offset VM_NOP
.data:0040749C dd offset VM_NOP
.data:004074A0 dd offset VM_NOP
.data:004074A4 dd offset VM_NOP
.data:004074A8 dd offset VM_Multiple_op2
.data:004074AC dd offset VM_Multiple_op2
.data:004074B0 dd offset VM_Multiple_op2
.data:004074B4 dd offset VM_Multiple_op2
.data:004074B8 dd offset VM_CMP
.data:004074BC dd offset VM_TEST
.data:004074C0 dd offset VM_NOT
.data:004074C4 dd offset VM_Multiple_op2
.data:004074C8 dd offset VM_Multiple_op2
.data:004074CC dd offset VM_MOV_MEMADDR_TO
.data:004074D0 dd offset VM_MOV_EIP_TO
.data:004074D4 dd offset VM_SWAP
.data:004074D8 dd offset VM_ADD_TO_ESP
.data:004074DC dd offset VM_SUB_FROM_ESP
.data:004074E0 dd offset VM_MOV_FROM_ESP
.data:004074E4 dd offset VM_MOV_TO_ESP
.data:004074E8 dd offset VM_NOP
.data:004074EC dd offset VM_NOP
.data:004074F0 dd offset VM_NOP
.data:004074F4 dd offset VM_NOP
.data:004074F8 dd offset VM_NOP
.data:004074FC dd offset VM_NOP
.data:00407500 dd offset VM_NOP
.data:00407504 dd offset VM_NOP
.data:00407508 dd offset VM_NOP
.data:0040750C dd offset VM_NOP
.data:00407510 dd offset VM_NOP
.data:00407514 dd offset VM_NOP
.data:00407518 dd offset VM_NOP
.data:0040751C dd offset VM_NOP
.data:00407520 dd offset VM_Status_8
.data:00407524 dd offset VM_SET_RESUME_EIP
.data:00407528 dd offset VM_NOP
.data:0040752C dd offset VM_ALLOW_IO
```

没有什么好说的了，除了可能检查其它的指令实现。

下面的可能很有趣：

```
int __cdecl VM_LOOP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
int VMCycleValue, VMValue;
 Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMCycleValue);
VMCycleValue--;
if (VMCycleValue==0) {
NextInstr(VMContext,DecodedInstr);
return 0;
}
Write_VMValue_To_Param(&DecodedInstr->ParamSrc, &VMCycleValue,4,VMContext);
Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
if (DecodedInstr->AddressType==vmaVMValueOrDisplacement)
VMValue+=VMContext->VM_EIP;
VMContext->VM_EIP = VMValue;
return 0;
}
```

你可以注意到，这个指令取回一个参数值——`ECX`是理想的等价物——并递减它。当它到达零值时结束并跳到下一条指令，就像`LOOPCXZ`，另外它写递减后的值到源操作数/寄存器执行一个跳转到请求的地址，是相对还是绝对依赖于地址类型：

下面是另一个很有趣的指令：

```
int __cdecl VM_CALL(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
int VMValue, VMRetValue;int res;
 VMRetValue = VMContext->VM_EIP+DecodedInstr->Length;
res = Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
if (!res) return 1;
if (DecodedInstr->AddressType==vmaVMValueOrDisplacement)
VMValue+=VMContext->VM_EIP;
VMContext->VM_EIP = VMValue;
VMContext->VM_ESP+=4;
Write_VMMemory_From(&VMContext->VM_ESP, &VMRetValue,4,VMContext);
return 0;
}
```

正如你所看到的，这个指令读子程序地址，设置`VM_EIP`准备开始下一个执行，然后写`VM_ESP`保存下一条指令的`VM_EIP`值（在`VM_CALL`后的哪一个）到堆栈中。如前面所记录的，堆栈地址模式被交换。

还有很多的指令可以检查，但是你可以看附录里的源码如果你正寻找一些特别的东西。

## 总结

这个VM有一些东西丢失了，最明显就是操作单个位，以及调试层支持的指令。二者都只是简单的出现（原文in nuce，网上找到一个定义in a nutshell）中，由于我们排除了在`Evaluate_Flags()`中进行特殊位测试的可能性，除了最明显的一个在机器主循环中的陷阱测试。

好了，我让你烦够了，写这些也烦够我了。希望你能学到些关于虚拟机的东西，并且你会发现这篇短文对于你所需要的无论什么都很有用。

感谢整个RCE社区无偿分享他们所知道的秘诀。我只是希望把从你们中所学的归还出来。

## Appendix

这里你能找到几乎整个T2'06逆向源代码。一些部分丢失了（所有的完整性检查可以肯定），因而它不可能编译得和原来一模一样。然而用它来编码你自己的虚拟机只需要付出很少的努力，也可以学习到T2'06更多的细节。

希望你会欣赏它。
