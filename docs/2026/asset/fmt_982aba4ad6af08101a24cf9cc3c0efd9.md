
```
def GetUnVar(info):

	z = []

	for x in range(0, MaxPtr + 1):

		if not info[x]:

			z.append(x)

	return z

#默认在尾部时，所有栈变量都是活跃的

for i in range(0, MaxPtr + 1):

	AnalyzeStackVar(LstStackInfo, i)

LstNewAsm = LstAsm[:]

LstNewStackInfo = LstStackInfo[:]

LstAsmInfo = []

for x, k in zip(LstNewAsm, LstNewStackInfo):

	y = {}

	y['Asm'] = x

	y['Stack'] = k

	LstAsmInfo.append(y)

LstNewAsmInfo = LstAsmInfo[:]

for asmi in LstAsmInfo:

	#不涉及到栈指针调整的指令可以直接删除

	if (len(asmi['Stack']['write']) != 0) and (IsListIn(asmi['Stack']['write'], GetUnVar(asmi['Stack']))) and (asmi['Asm'].mnemonic not in ['push', 'pushal', 'pushfd', 'pushad', 'pop', 'popfd', 'popal', 'popad']):

			print('位于%X处的指令被清除了' % asmi['Asm'].address)

			LstNewAsmInfo.remove(asmi)
```

输出情况并不是非常理想，仅删除了3条语句。

![](https://bbs.kanxue.com/upload/attach/201902/783210_9CV56YGSUKMNRGQ.jpg)

这3条指令如下。

```
mov word ptr ss:[esp],dx

mov byte ptr ss:[esp+0x8],al

mov byte ptr ss:[esp],bl
```

当然，我们还可以继续化简。

当入栈与出栈所写入的栈变量为死变量时，我们可以将其修改为一条修改栈指针的指令。如下图所示。

![](https://bbs.kanxue.com/upload/attach/201902/783210_8PMASYDD4CE7MT8.png)

`0x406194`处的代码可以直接化简为`lea esp,[esp - 4]`，因为写入的第0号栈变量是死变量。但我们不能直接删除这行代码，否则会导致栈不平衡。

具体代码如下。

```
#清理堆栈混淆

def CleanStackObfs(LstStackInfo, LstAsm):

	def AnalyzeStackVar(LstStackInfo, StackVar):

		k = True

		LstStackInfo.reverse()

		for info in LstStackInfo:

			info[StackVar] = k

			if StackVar in info['write']:

				k = False

			if StackVar in info['read']:

				k = True

		LstStackInfo.reverse()

		return

	def GetUnVar(info):

		z = []

		for x in range(0, MaxPtr + 1):

			if not info[x]:

				z.append(x)

		return z

	#默认在尾部时，所有栈变量都是活跃的

	for i in range(0, MaxPtr + 1):

		AnalyzeStackVar(LstStackInfo, i)

	LstNewAsm = LstAsm[:]

	LstNewStackInfo = LstStackInfo[:]

	LstAsmInfo = []

	for x, k in zip(LstNewAsm, LstNewStackInfo):

		y = {}

		y['Asm'] = x

		y['Stack'] = k

		LstAsmInfo.append(y)

	LstNewAsmInfo = LstAsmInfo[:]

	ObDis = Cs(CS_ARCH_X86, CS_MODE_32)

	ObDis.detail = True

	for insn in ObDis.disasm(b'\x8D\x64\x24\xFC', 401000):

		vPush = insn

	for insn in ObDis.disasm(b'\x8D\x64\x24\xE0', 401000):

		vPushal = insn

	for insn in ObDis.disasm(b'\x8D\x64\x24\x04', 401000):

		vPop = insn

	for insn in ObDis.disasm(b'\x8D\x64\x24\x20', 401000):

		vPopal = insn

	for asmi in LstAsmInfo:

		if (len(asmi['Stack']['write']) != 0) and (IsListIn(asmi['Stack']['write'], GetUnVar(asmi['Stack']))):

			if asmi['Asm'].mnemonic not in ['push', 'pushal', 'pushfd', 'pushad', 'pop', 'popfd', 'popal', 'popad']: #不涉及到栈指针调整的指令可以直接删除

				print('位于%X处的指令被清除了' % asmi['Asm'].address)

				LstNewAsmInfo.remove(asmi)

			if asmi['Asm'].mnemonic in ['push', 'pushfd']:

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Asm'] = vPush

				print('位于%X处的指令被清除了' % asmi['Asm'].address)

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['write'] = []

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['read'] = []

			if asmi['Asm'].mnemonic in ['pushal', 'pushad']:

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Asm'] = vPushal

				print('位于%X处的指令被清除了' % asmi['Asm'].address)

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['write'] = []

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['read'] = []	

			if asmi['Asm'].mnemonic in ['pop', 'popfd']:

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Asm'] = vPop

				print('位于%X处的指令被清除了' % asmi['Asm'].address)

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['write'] = []

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['read'] = []

			if asmi['Asm'].mnemonic in ['popad', 'popal']:

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Asm'] = vPopal

				print('位于%X处的指令被清除了' % asmi['Asm'].address)

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['write'] = []

				LstNewAsmInfo[ListFind(LstNewAsmInfo, asmi)]['Stack']['read'] = []				

	LstAsmInfo = LstNewAsmInfo[:]

	return LstNewAsmInfo
```

输出结果如下。

![](https://bbs.kanxue.com/upload/attach/201902/783210_AZX87YH3KWUWX5J.png)

两次化简的情况看起来不是非常理想，未化简前有123条代码，化简后仍有120条代码。但其中大部分代码都是调整栈指针的无意义代码。

我们仍然运用消除死代码的思想，但这次我们的对象换为寄存器。

寄存器的活跃分析原则与栈变量的活跃分析原则相同，在此不做赘述。

需要注意的是，我将所有对某一32位寄存器的低16位，高8位，低8位寄存器的操作均视为对该32位寄存器的操作。

具体代码如下。

```
def AnalyeRegObfs(LstAsm):

	def IsUseful(AsmInfo):

		insn = AsmInfo['Asm']

		(regs_read, regs_write) = insn.regs_access()

		if len(regs_write) == 0:

			return True

		if ModifyReg('esp', insn):

			return True

		for r in AsmInfo.keys():

			if r != 'Asm' and AsmInfo[r]: #如果影响了任意一个活跃寄存器即视为有效

				if ModifyRegExact(r, insn):

					return True 

		return False

	def AnalyzeReg(LstAsm, 	Reg):

		k = True

		LstAsm.reverse()

		LstRegInfo = []

		for insn in LstAsm:

			LstRegInfo.insert(0, k)

			if ModifyRegExact(Reg, insn):

				k = False

			if ReadRegExact(Reg, insn):

				k = True

		LstAsm.reverse()

		return LstRegInfo

	LstAsmInfo = []

	LstReg = ['eax', 'ebx', 'ecx', 'edx', 'edi', 'esi', 'ebp']

	LstRegInfo = {}

	for r in LstReg:

		LstRegInfo[r] = AnalyzeReg(LstAsm, r)

	for i in range(0, len(LstAsm)):

		insn = LstAsm[i]

		asmi = {}

		asmi['Asm'] = insn

		for k in LstRegInfo.keys():

			asmi[k] = LstRegInfo[k][i]

		LstAsmInfo.append(asmi)

	LstNewAsmInfo = LstAsmInfo[:]

	LstDel = []

	for i in LstAsmInfo:

		if not IsUseful(i):

			print('%X %s %s' % (i['Asm'].address, i['Asm'].mnemonic, i['Asm'].op_str))

	return LstAsmInfo
```

删除了20条指令。

![](https://bbs.kanxue.com/upload/attach/201902/783210_QRSYWS2FK2NZ5S7.jpg)

注意，我没有分析标志位的活跃状态，因为标志位的活跃状态分析比较扯淡，因为有的指令对标志位既访问又写入，如`cmc`、`sbb`、`adc`等指令，而反汇编引擎无法解析明确。

注意，删除了一次死代码后，可能又会有新的死代码出现。如下。

```
int x; //1

x = 1; //2

x += 5; //3

x += 6; //4

x = 10; //5
```

所以我们需要不断重复这个过程，直到无法再删除任何指令。当然如果你和我一样懒，也可以多重复调用几次清理过程……

最终代码化简到85条，其中包含29条调整栈指针的代码，基本可以认为最终生成的代码为56条。

代码如下。
