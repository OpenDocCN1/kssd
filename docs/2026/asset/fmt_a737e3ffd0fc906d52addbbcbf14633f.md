
# 保存指令和寄存器环境类

```java
class InsAndCtx
{
long addr;
Instruction ins;
List<Number> regs;
public long getAddr() {
return addr;
}
public void setAddr(long addr) {
this.addr = addr;
}
public void setIns(Instruction ins) {
this.ins = ins;
}
public Instruction getIns() {
return ins;
}
public void setRegs(List<Number> regs) {
this.regs = regs;
}
public List<Number> getRegs() {
return regs;
}
}
```

# patch类

```java
class PatchIns{
long addr;//patch 地址
String ins;//patch的指令
public long getAddr() {
return addr;
}
public void setAddr(long addr) {
this.addr = addr;
}
public String getIns() {
return ins;
}
public void setIns(String ins) {
this.ins = ins;
}
}
```

# 指令栈

```java
private Stack<InsAndCtx> instructions;
```

# 所有需要patch的指令

```java
private List<PatchIns> patchs;
```

# 保存指令寄存器环境

```java
public List<Number> saveRegs(Backend bk)
{
List<Number> nb = new ArrayList<>();
for(int i=0;i<29;i++)
{
nb.add(bk.reg_read(i+Arm64Const.UC_ARM64_REG_X0));
}
nb.add(bk.reg_read(Arm64Const.UC_ARM64_REG_FP));
nb.add(bk.reg_read(Arm64Const.UC_ARM64_REG_LR));
return nb;
}
```

# 指令hook，每条指令执行前保存环境

```java
public void processBr()
{
emulator.getBackend().hook_add_new(new CodeHook() {
@Override
public void hook(Backend backend, long address, int size, Object user) {
Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM64,Capstone.CS_MODE_ARM);
byte[] bytes = emulator.getBackend().mem_read(address, 4);
Instruction[] disasm = capstone.disasm(bytes, 0);
InsAndCtx iac = new InsAndCtx();
iac.setIns(disasm[0]);
iac.setRegs(saveRegs(backend));
iac.setAddr(address);
instructions.push(iac);
do_processbr();
}
@Override
public void onAttach(UnHook unHook) {
System.out.println("attach");
}
@Override
public void detach() {
System.out.println("detach");
}
},module.base+start, module.base+end,null);
}
```

# 指令栈回溯，根据处理结果，生成patchIns，供最后统一patch

```java
public void do_processbr()
{
Instruction ins = instructions.peek().getIns();
if(ins.getMnemonic().equals("br") && ins.getOpStr().equals("x9"))
{
boolean finish = false;
long base = -1;
long listoffset = -1;
long cond1 = -1;
long cond2 = -1;
String cond = "";
long addinstaddr = -1;
long brinsaddr = instructions.peek().getAddr() - module.base;
long selectaddr = -1;
long ldaaddr = -1;
try {
while (!finish && !instructions.empty())
{
instructions.pop();
ins = instructions.peek().getIns();
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("add"))
{
String[] split = ins.getOpStr().split(",");
if(split.length == 3)
{
if(split[0].toLowerCase(Locale.ROOT).trim().equals("x9") && split[1].toLowerCase(Locale.ROOT).trim().equals("x9"))
{
String reg = split[2].trim().toLowerCase(Locale.ROOT);
base = getRegValue(reg,instructions.peek().getRegs()).longValue();
addinstaddr = instructions.peek().getAddr() - module.base;
}
else {
break;
}
}
else
{
break;
}
}
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("ldr"))
{
String[] sp = ins.getOpStr().toLowerCase().split(",");
if(sp.length == 3)
{
if(sp[0].trim().toLowerCase(Locale.ROOT).equals("x9") && sp[2].trim().toLowerCase(Locale.ROOT).equals("x9]"))
{
String reg = sp[1].toLowerCase(Locale.ROOT).trim().substring(1);
listoffset = getRegValue(reg,instructions.peek().getRegs()).longValue()-module.base;
ldaaddr = instructions.peek().getAddr()- module.base;
}
}
}
if(ins.getMnemonic().trim().toLowerCase(Locale.ROOT).equals("csel"))
{
String[] sp = ins.getOpStr().toLowerCase(Locale.ROOT).split(",");
if(sp.length == 4)
{
cond = sp[3].trim();
if(sp[0].trim().equals("x9"))
{
String reg1 = sp[1].trim();
String reg2 = sp[2].trim();
cond1 = getRegValue(reg1,instructions.peek().getRegs()).longValue();
cond2 = getRegValue(reg2,instructions.peek().getRegs()).longValue();
selectaddr = instructions.peek().getAddr() - module.base;
}
}
}
if(ins.getMnemonic().trim().toLowerCase(Locale.ROOT).equals("cmp"))
{
if(base == -1 || listoffset == -1 || cond1 == -1 || cond2 == -1 || cond.equals("") || addinstaddr == -1 || ldaaddr == -1 || selectaddr == -1)
{
break;
}
else
{
long offset1 = base + readInt64(emulator.getBackend(), module.base+listoffset+cond1) - module.base;
long offset2 = base + readInt64(emulator.getBackend(),module.base+listoffset+cond2) - module.base;
if( brinsaddr - addinstaddr != 4)
{
System.out.println("add ins and br ins gap more than 4 size,may make mistake");
}
String condBr = "b"+cond.toLowerCase(Locale.ROOT) + " 0x"+ Integer.toHexString((int) (offset1 - addinstaddr));
String br = "b 0x" + Integer.toHexString((int)(offset2 - brinsaddr));
PatchIns pi1 = new PatchIns();
pi1.setAddr(addinstaddr);
pi1.setIns(condBr);
patchs.add(pi1);
PatchIns pi2 = new PatchIns();
pi2.setAddr(brinsaddr);
pi2.setIns(br);
patchs.add(pi2);
PatchIns pi3 = new PatchIns();
pi3.setAddr(selectaddr);
pi3.setIns("nop");
patchs.add(pi3);
PatchIns pi4 = new PatchIns();
pi4.setAddr(ldaaddr);
pi4.setIns("nop");
patchs.add(pi4);
finish = true;
}
}
}
}catch (Exception e)
{
e.printStackTrace();
}
}
}
```

# 遍历patch表，执行patch，生成新的so，使用Ketstone将汇编转为机器码

```java
public void patch()
{
try {
File f = new File(inName);
FileInputStream fis = new FileInputStream(f);
byte[] data = new byte[(int) f.length()];
fis.read(data);
fis.close();
for(PatchIns pi:patchs)
{
System.out.println("procrss addr:"+Integer.toHexString((int) pi.addr)+",code:"+pi.getIns());
Keystone ks = new Keystone(KeystoneArchitecture.Arm64, KeystoneMode.LittleEndian);
KeystoneEncoded assemble = ks.assemble(pi.getIns());
for(int i=0;i<assemble.getMachineCode().length;i++)
{
data[(int) pi.addr+i] = assemble.getMachineCode()[i];
}
}
File fo = new File(outName);
FileOutputStream fos = new FileOutputStream(fo);
fos.write(data);
fos.flush();
fos.close();
System.out.println("finish");
}
catch (Exception e)
{
e.printStackTrace();
}
}
```

patch完成后，再次使用ida打开原so，发现`Jni_onLoad`函数已经可以F5了：

![](upload/attach/202305/872365_ZDSR8F5KX84NM62.png)

但是依然有一些jumpout。

原因是因为在模拟执行的是后并没有走遍所有的分支，我们只是将走过的分支处理了。

接下来需要在jumpout的地方手动修改寄存器的值，来控制他的走势，让unidbg最终走完所有分支。

比如对于这一处br，在blt时才会跳转过去，

![](upload/attach/202305/872365_MZ5XZJH5NDC68Y6.png)

可以看到这一处的跳转和w12有关，我们找到给w8赋值为w12的地方，通过修改寄存器来给其赋值：

![](upload/attach/202305/872365_TT8NKKZ3AY7V6P2.png)

![](upload/attach/202305/872365_4JREVVME3BAYZPT.png)

重新跑一边代码，这一处的`br x9`就修复了。将没有走到的分支处理完后，F5结果如下：

![](upload/attach/202305/872365_Y59FJNYRJ7MYKEY.png)

可以看到是标准的ollvm的控制流平坦化。

所以`br x9`的混淆其实是在ollvm控制流平坦化的基础上，将所有的跳转地址都加了一步运算。

## 3.去除控制流平坦化

间接跳转去除完后，需要去除控制流平坦化。根据控制流平坦化的工作原理，原来的代码会被打碎成代码片段，然后装进一个大的switch case里，最外层包一个while true循环，根据索引变量来按照原来的顺序执行代码。

![](upload/attach/202305/872365_T3F226Q4K8FV9VR.png)

考虑到编译的结果要在逻辑上和原来的代码完全一致，所以一个重要的结论是——真实的代码块一定在`cmp eq`之后。`switch case`的一个case

控制流平坦化对于顺序执行的处理比较简单，直接在前一个真实块的末尾，将索引值改为下一个真实块的索引，然后跳转到主分发器即可。

对于分支执行`判断，循环`，在条件的部分会有一个条件选择指令`CSEl`，将索引寄存器的值根据条件结果设置为不同的索引，然后跳转到主分发器。

所以我们可以按照以下算法对控制流平坦化进行还原：

1.建立一个指令栈，每条指令执行前，保留该指令的地址，指令内容，当前所有寄存器的值。然后将当前指令的信息push进指令栈。

2.对指令进行回溯，如果栈顶指令是`b.eq`，则进行3`收集真实块`，如果栈顶指令是直接跳转指令`b`，则进行5`处理分支块`，否则继续执行下一条指令。

3.向上回溯指令栈，找到第一条`cmp`指令，判断是否为与索引寄存器w8比较，如果是，则进行4，否则继续执行下一条指令。

4.获取与w8比较的另一个寄存器的值，获取`b.eq`的目标地址，组成一个`索引，真实块`对。继续执行下一条指令。

5.判断上一条指令是否为`CSEL W8`。如果是，则记录一个`条件成立块索引，条件不成立块索引`对。否则，继续执行下一条指令。

同时，我们在主分发器处hook指令，记录每次经过主分发器时的索引寄存器的值为索引值顺序。

在模拟执行收集完成信息之后，做以下处理：

1.去除索引值顺序中为条件块的索引。

2.获取索引值顺序中的第一个值，找到对应的真实块。将主分发器处patch为跳转向第一个真实块的跳转指令。

3.根据真实块结束时的新的索引值，寻找到对应的真实块，将结尾处patch为跳转到下一个真实块。

4.在`CSEL`指令处，根据条件成立块索引和条件不成立索引找到对应的真实块。将`CSEL`指令和后面的跳转向主分发器的指令patch为两条指令：

第一条为条件成立时的条件跳转，第二条为条件不成立时的直接跳转。

对应的代码如下:
