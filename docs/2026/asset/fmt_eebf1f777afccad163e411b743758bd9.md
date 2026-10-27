
在详细地检查VM指令管理之前，最好花点时间在VM内存管理上面。如前所说，VM内存是大端顺序的。所以，在小端机器上，我们需要回来转换所有移动到这的值。这个VM为这个步骤实现两个函数：`Read_VMMemory_To()`和`Write_VMMemory_From()`。让我们检查它们的实现，为了对它们的工作获得一些想法：

```
.text:00401230 ; int __cdecl Write_VMMemory_From(int VMAddress,int Ptr_ValueToWriteAndSwap,int VMValue_OperandSize,int vm_context_ptr)
.text:00401230 Write_VMMemory_From proc near ; CODE XREF: Write_VMValue_To_Param+CB#p
.text:00401230 ; VM_PUSH+3D#p ...
.text:00401230
.text:00401230 VMAddress = dword ptr 4
.text:00401230 Ptr_ValueToWriteAndSwap= dword ptr 8
.text:00401230 VMValue_OperandSize= dword ptr 0Ch
.text:00401230 vm_context_ptr = dword ptr 10h
.text:00401230
.text:00401230 ebx_param_vmvalue= ebx
.text:00401230
.text:00401230 000 mov eax, [esp+Ptr_ValueToWriteAndSwap]
.text:00401234 000 mov edx, [esp+VMAddress]
.text:00401238 000 push ebx
.text:00401239 004 lea ecx, [esp+4+Ptr_ValueToWriteAndSwap] ; Load Effective Address
.text:0040123D 004 mov ebx_param_vmvalue, [eax]
.text:0040123F 004 mov eax, [esp+4+vm_context_ptr]
.text:00401243 004 push ecx ; Write_RealAddr_To
.text:00401244 008 push edx ; VM_Address
.text:00401245 00C push eax ; vm_context_ptr
.text:00401246 010 call VMAddress2Real ; Call Procedure
.text:00401246
.text:0040124B 010 add esp, 0Ch ; Add
.text:0040124E 004 test eax, eax ; Logical Compare
.text:00401250 004 jnz short loc_401254 ; Jump if Not Zero (ZF=0)
.text:00401250
.text:00401252 004 pop ebx_param_vmvalue
.text:00401253 000 retn ; Return Near from Procedure
.text:00401253
.text:00401254 ; ---------------------------------------------------------------------------
.text:00401254
.text:00401254 loc_401254: ; CODE XREF: Write_VMMemory_From+20#j
.text:00401254 004 mov eax, [esp+4+VMValue_OperandSize]
.text:00401258 004 dec eax ; Decrement by 1
.text:00401259 004 jz short op_byte_no_be_swap ; Jump if Zero (ZF=1)
.text:00401259
.text:0040125B 004 dec eax ; Decrement by 1
.text:0040125C 004 jz short op_word_do_le2be_swap ; Jump if Zero (ZF=1)
.text:0040125C
.text:0040125E 004 sub eax, 2 ; Integer Subtraction
.text:00401261 004 jnz short loc_4012AD ; here below, operandsize is 4
... (the code here was already shown: it is the prior 'swap endian' code)
.text:004012AD 004 mov eax, 1
.text:004012B2 004 pop ebx_param_vmvalue
.text:004012B3 000 retn ; Return Near from Procedure
.text:004012B3
.text:004012B3 Write_VMMemory_From endp
int Write_VMMemory_From(int VMAddress,int *LEValueSource, // Ptr_ValueToWriteAndSwap
int OperandSize, TVMContext* VMContext) // .text:00401230
{
int *DestAddr;
res = VMAddress2Real(VMContext,VMAddress,&DestAddr);
switch(OperandSize) {
case 1: *(byte*)DestAddr = SWAP((byte)LEValueSource);break;
case 2: *(word*)DestAddr = SWAP((word)LEValueSource);break;
case 4: *(dword*)DestAddr = SWAP((dword)LEValueSource);break;
}
return 1;
}
```

正如你所看到的，这个过程的代码简单的翻译一个内存地址到它的x86地址，然后写一个值在上面，交换它的最大的端。很明显它使用于处理诸如虚拟堆栈之类的过程。读过程非常的相似，但是它们的区别在于写的是x86内存，而不是虚拟的。

## VM 指令核心

用于解释VM程序操作码和传递它们到VM指令调度器的缓冲区代表这个VM指令的核心。从这点来看，你可以通过交换VM解码器和调整一些字段写出自己的VM语言。这可能是由于事实上VM解释这一层是在缓冲区里的，缓冲区扮演一个间接的层。但是让我们来仔细检查一下这个缓冲区：

```
struct TparamDecoding{ // used by the decoding array to retrieve i.e. the parameters usage of instructions
int ID,
int Params[3];
}
struct TSubInstr { // represents a parameter's field of the VM Instruction
int AddressType,
int RegisterIdx,
int Decoder_ParamsValue,
int VMValue
}
struct TInstructionBuffer{
int Length,
int InstructionData,
char InstrType,
//char Fillers1[3], // if structure alignment is 1
int Operand_Size,
char InstructionParamsCount,
//char Fillers2[3], // if structure alignment is 1
SubInstr ParamDest,
SubInstr paramSrc,
SubInstr ParamThird,
SubInstr *WorkSubField
};
```

这些结构正是VM使用的。现在让我们检查读写参数的函数：

```
enum TVMAddressType {
vmaRegister = 0,
vmaRegisterAddress = 1,
vmaDirectAddress = 2,
vmaVMValue_orC4__or_displacement = 3
}
int Retrieve_Param_Value(TInstructionBuffer *InstrBuff, SubInstr *Param, // 14h/24h/34h
int *WriteValueTo, TVMContext *VMContext) // .text:00401340
{
int myLEvalue;
 switch(Param->AddressType) {
case vmaRegister:
switch(InstrBuff->OperandSize){
case 4: myLEvalue = (dword)VMContext->Registers[Param->RegisterIdx];break;
case 2: myLEvalue = (word)VMContext->Registers[Param->RegisterIdx];break;
case 1: myLEvalue = (char)VMContext->Registers[Param->RegisterIdx];break;
}
break;
case vmaDirectAddress:
vmaddr = vmAddress;
case vmaRegisterAddres:
if (Param->AddressType!=vmaDirectAddress) {
vmaddr =VMContext->Registers[Param->RegisterIdx];
}
res =Read_VMMemory_To(vmaddr,myLEvalue,InstrBuff->OperandSize)
if (!res) return 0;
break;
case vmaVMValue:
myLEvalue = ParamField->VMAddress;
break:
default:
}
*WriteValueTo=myLEvalue;
}
```

正如我们注意到的，这个读取参数的过程辨别请求的地址类型，从VM寄存器集合中取回的值，从虚拟地址内存中的地址，从直接的值。任何时候它需要访问VM内存，它使用适当的函数读内存然后返回一个交换(小端)值。用来写参数的函数也是非常相似的。

在这点，只有两个主要的函数需要检查，一个关乎于虚拟标志，另一个是VM解码器本身。

`Evaluate_Flags()`函数是在改变标志状态的函数里被调用。例如`VM_CMP`和`VM_TEST`指令调用这个过程来设置适当的VM标志。这个函数在数学操作之后也被调用为了保持一致的内部状态。这个程序让人感兴趣的部分在于它接收一个特殊的参数表明标志测试应该怎样执行：它用于改变OF/CF标志的数学操作。

```
int __cdecl Evaluate_Flags(int ParamAdditional, int ParamEvaluate,
int TestType, TInstructionBuffer* Instruction_ptr, TVMContext* VMContext) // .text:00401340
{
TVMFLAGS *Flags = &VMContext->VMFlags;
int OpSize = Instruction_ptr->OperandSize;
int NegMark;
switch(OpSize) {
case 4: NegMark =0x80000000;Flags->ZF= ParamEvaluate==0; break;
case 2: NegMark =0x8000;Flags->ZF= (word)ParamEvaluate==0; break;
case 1: NegMark =0x80;Flags->ZF= (char)ParamEvaluate==0; break;
default: NegMark = ParamAdditional; // room for BTx instructions expansion.
// custom evaluation of flags based on bit-testing.
}
Flags->SF= (NegMark&ParamEvaluate)!=0;
switch (TestType) {
case 2: Flags->OF = (NegMark&ParamEvaluate)==0&&(NegMark&ParamAdditional)!=0;
Flags->CF = (NegMark&ParamEvaluate)!=0&&(NegMark&ParamAdditional)==0;
break;
case 1: Flags->OF = ! ((NegMark&ParamEvaluate)==0&&(NegMark&ParamAdditional)!=0);
Flags->CF = ! ((NegMark&ParamEvaluate)!=0&&(NegMark&ParamAdditional)==0);
break;
case 0:
default:
}
return;
}
```

旁注：你能实现自己的作用于位(BTS, BTC等)字段的VM指令，然后用一个操作数大小不同于正常值(1-2-4)的参数调用这个函数。这会造成函数接受额外的参数作为位掩码来执行BTX测试在这个位上。

这是解码器函数：
