
# [原创]利用机器学习分析vmp的思路

## 前言

继续使用上一篇文章《利用活跃变量分析来去掉vmp的大部分垃圾指令》的样本。有个小问题先需要说明一下，加壳的样本在wow64环境、32位环境和linux wine环境下，样本的执行流程是不同的，因为vmp会根据环境来调整通过`sysenter`或`syscall`指令来进行系统调用，比如`NtProtectVirtualMemory`函数。如果程序是32位的，`qiling`会设置成32位环境。所以利用`qiling`模拟执行和调试器的执行流程可能会不一样。本篇文章分析vmp的大致思路是通过机器学习对`handle`进行分类，然后分别处理和化简。

## handle的提取

通过机器学习来进行分类，首先要提取出`handle`以及它的特征向量。为了能够方便的提取出`handle`，本文继续使用`qiling`模拟执行记录样本的vmp壳程序每一条指令以及`eax`, `ebx`, `ecx`, `edx`, `esi`, `edi`, `ebp`, `esp`, `eip`寄存器的值，记录这些寄存器的值是为了方便后面的化简。模拟执行的代码在`traceCode.py`文件，保存后的文件大小有1G多，大部分都是循环产生的重复指令。然后对这些指令序列根据`jmp register`和`ret`指令进行分割。分割后的指令序列需要删除一些因循环产生的重复指令。对于还有`call`指令的函数调用，需要把函数体对应的指令删除掉。vmp壳程序也是有一些正常函数的，比如一些字符串的解密函数和散列函数。`handle`的提取、分类和化简都在`analyzeInsn.py`文件中。主函数代码如下：

```
def analyzeInsnTrace():
with open('G:\\vmpAnalysis\\EliminateVmpJunkCode\\traceInsn.bin', 'rb') as f:
s = f.read()
insnExecInfoList = [] #保存待分析指令信息
i=0
point = 0
while(point < len(s)): #007FFFE8和006D0321两处进行某种循环运算
size = s[point]
point += 1
bInsn = s[point:point+size]
point += size
eax,ebx,ecx,edx,esi,edi,ebp,esp,eip = struct.unpack('IIIIIIIII',s[point:point+36])
point += 36
insn = list(md.disasm(bInsn, eip))[0]
if(capstone.x86\_const.X86\_INS\_JMP == insn.id and \
capstone.x86\_const.X86\_OP\_IMM == insn.operands[0].type):#jmp 0xXXXXXXXX;00416BC1: jmp far ;X86\_INS\_LJMP
continue
#elif(capstone.x86\_const.X86\_INS\_SYSENTER == insn.id): #wow64环境下通过天堂门进入NtProtectVirtualMemory
# continue
#if(0x0068304e == eip):
# return
insnInfo = InsnExecInfo(insn, eax,ebx,ecx,edx,esi,edi,ebp,esp)
insnExecInfoList.append(insnInfo)
#jmp reg
if((capstone.x86\_const.X86\_INS\_JMP == insn.id and capstone.x86\_const.X86\_OP\_REG == insn.operands[0].type) or\
capstone.x86\_const.X86\_INS\_RET == insn.id):
insnExecInfoList = HandleCallInsn(insnExecInfoList)
if(capstone.x86\_const.X86\_INS\_CALL != insnExecInfoList[-1].csInsn.id):#call里面又有call,通常handle不以call结尾
insnExecInfoList = DeleteRedundantInsnInfo(insnExecInfoList)
insnExecInfoList = GetUsefulInsnList(insnExecInfoList)
TranslateHandle(insnExecInfoList)
insnExecInfoList.clear()
```

`DeleteRedundantInsnInfo`函数删除掉循环产生的重复指令，`HandleCallInsn`函数用来处理含有`call`的调用。`GetUsefulInsnList`在提取`handle`的特征前需要去掉`handle`一些不需要的指令，比如`handle`地址的计算和一些寄存器轮转相关的指令。这个可以通过一个简单的污点分析实现，保留与写内存操作相关的指令，实现代码在`TaintAnalysis`函数中。

比如有如下`handle`：

```
mov ecx, [esi]
add esi, 4
lea ebp, [ebp-1]
movzx edx, byte ptr [ebp+0]
xor dl, bl
ror dl, 1
add dl, 0B3h
neg dl
not dl
neg dl
xor bl, dl
mov [esp+edx+0], ecx
sub ebp, 4
mov edx, [ebp+0]
xor edx, ebx
dec edx
rol edx, 1
not edx
inc edx
bswap edx
xor ebx, edx
add edi, edx
push edi
retn
```

有用的指令就这么两条：

```
mov ecx, [esi]
mov [esp+edx+0], ecx
```

## handle的特征提取及模型训练

特征可以人工选取或者通过自然语言处理的一些模型得到，比如`word2vec`和`bert`模型，把汇编指令看成一个单词，`handle`看成一个句子。通过自然语言处理提取代码的特征可以参考一些代码相似度检测相关的论文。本文采用人工定义的特征和`kmeans`聚类算法，主要是实现简单，可以快速验证这个思路是否可靠。一个`handle`的特征定义如下：

```
push指令的个数
pop指令的个数
数据转移指令的个数
算术运算指令的个数
位操作指令的个数
串操作指令的个数
call指令的个数
分支指令的个数
其它一些需要分析的指令个数（sysenter、cpuid、jmp far）
立即数的个数
指令总数
```

代码实现如下：

```
def ExtractFeature(insnExecInfoList):
NumOfPushInsn = 0
NumOfPopInsn = 0
NumOfDataTransferInsn = 0 #数据转移指令的个数
NumOfArithmeticInsn = 0 #算术运算指令的个数
NumOfBitManipulationInsn = 0 #位操作指令的个数
NumOfStringInsn = 0 #串操作指令的个数
NumOfCall = 0 #call指令的个数
NumOfBranchInsn = 0 #分支指令的个数
NumOfOtherInsn = 0
NumOfImmOperand = 0 #立即数的个数
NumOfInsn = len(insnExecInfoList)
for insnInfo in insnExecInfoList:
insn = insnInfo.csInsn
#print("%08x %s %s" % (insn.address, insn.mnemonic, insn.op\_str))
if(insn.id in g\_pushInsnId):
NumOfPushInsn += 1
elif(insn.id in g\_popInsnId):
NumOfPopInsn += 1
elif(insn.id in g\_dataTransferInsnId):
NumOfDataTransferInsn += 1
elif(insn.id in g\_arithmeticInsnId):
NumOfArithmeticInsn += 1
elif(insn.id in g\_bitManipulationInsnId):
NumOfBitManipulationInsn += 1
elif(insn.id in g\_stringInsnId):
NumOfStringInsn += 1
elif(capstone.x86\_const.X86\_INS\_CALL == insn.id): #操作数可能为立即数
NumOfCall += 1
continue
elif(insn.group(capstone.x86\_const.X86\_GRP\_BRANCH\_RELATIVE)): #操作数可能为立即数
NumOfBranchInsn += 1
continue
elif(insn.id in g\_otherInsnId):
NumOfOtherInsn += 1
for op in insn.operands:
if(capstone.x86\_const.X86\_OP\_IMM == op.type):
NumOfImmOperand += 1
return [NumOfPushInsn, NumOfPopInsn, NumOfDataTransferInsn, NumOfArithmeticInsn, NumOfBitManipulationInsn, \
NumOfStringInsn, NumOfCall, NumOfBranchInsn, NumOfOtherInsn, NumOfImmOperand, NumOfInsn]
```

壳程序所有`handle`的提取在`ExtractHandleFeature`函数实现。特征提取后，通过`t-sne`对这些数据降维然后可视化，效果如下：

![图片描述](upload/attach/202108/742617\_K6GASTRSZ7WUAWJ.png)

可以看出这数据还是具有可分性的，一个簇一类，大致可以划分为14类。数据的可视化和模型的训练相关的代码在`model.py`。代码如下：

```
g\_dataDict = dict()
with open('handleFeature.txt','r') as f:
dataDictStr = f.read()
g\_dataDict = json.loads(dataDictStr)
f.close()
data = []
for addr in g\_dataDict:
data.append(g\_dataDict[addr])
x\_train = np.array(data)
#model = joblib.load('kmeans.pkl')
#labels = model.labels\_
model = KMeans(n\_clusters=14)
model.fit(x\_train)
joblib.dump(model,'kmeans.pkl')
tsne = TSNE(perplexity=30, n\_components=2, random\_state=0)
x\_2d = tsne.fit\_transform(x\_train)
plt.figure()
color = ['c', 'b', 'g', 'r', 'm', 'y', 'k', 'bisque', 'slategray', 'pink', 'grey', 'chocolate', 'aqua', 'lime', 'gold']
for i in range(len(x\_2d)):
x = x\_2d[i]
plt.scatter(x[0],x[1], c='red')#c=color[labels[i]]
plt.savefig("Figure\_Perp30.png")
plt.show()
```

部分分类出的`handle`如下：
