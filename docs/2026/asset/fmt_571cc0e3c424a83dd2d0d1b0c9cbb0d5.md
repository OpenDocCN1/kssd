
```java
//用于记录条件块中，条件成立，条件不成立时对应的索引值，同时记录条件。
class selectBr
{
long insaddr;
long trueindex;
long falseindex;
String cond;
public String getCond() {
return cond;
}
public void setCond(String cond) {
this.cond = cond;
}
public long getInsaddr() {
return insaddr;
}
public void setInsaddr(long insaddr) {
this.insaddr = insaddr;
}
public long getTrueindex() {
return trueindex;
}
public void setTrueindex(long trueindex) {
this.trueindex = trueindex;
}
public long getFalseindex() {
return falseindex;
}
public void setFalseindex(long falseindex) {
this.falseindex = falseindex;
}
}
//真实块，索引值和开始地址
class TrueBlock{
long index;
long startAddr;
public TrueBlock(){}
public TrueBlock(long l,long s)
{
index = l;
startAddr = s;
}
public long getIndex() {
return index;
}
public void setIndex(long index) {
this.index = index;
}
public long getStartAddr() {
return startAddr;
}
public void setStartAddr(long startAddr) {
this.startAddr = startAddr;
}
}
//记录真实块
private List<TrueBlock>tbs;
//记录条件块
private List<selectBr> sbs ;
//记录索引顺序
private List<Long> indexOrder;
public void processFlt()
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
instructions.add(iac);
do_processflt();
}
@Override
public void onAttach(UnHook unHook) {
System.out.println("attach");
}
@Override
public void detach() {
System.out.println("detach");
}
},module.base+start, module.base+end, null);
}
public void do_processflt()
{
if(instructions.empty())
{
return;
}
Instruction ins = instructions.peek().getIns();
if(instructions.peek().getAddr() - module.base == dispatcher)
{
indexOrder.add(getRegValue("x8",instructions.peek().getRegs()).longValue());
}
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("b.eq")) {
InsAndCtx beq = instructions.peek();
//等于跳转，检查是否为cmp x8,
while (true)
{
if(instructions.empty())
{
break;
}
instructions.pop();
ins = instructions.peek().getIns();
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("cmp"))
{
String[] sp = ins.getOpStr().toLowerCase(Locale.ROOT).split(",");
if(sp[0].equals("w8"))
{
//找到一个真实块
TrueBlock tb = new TrueBlock();
long regValue = getRegValue(sp[1].trim(), instructions.peek().getRegs()).longValue();
long targetAddr = 0;
String offset = beq.getIns().getOpStr().toLowerCase(Locale.ROOT);
long offsetvalue = getLongFromOpConst(offset);
targetAddr = beq.getAddr() + offsetvalue - module.base;
tb.setIndex(regValue);
tb.setStartAddr(targetAddr);
tbs.add(tb);
break;
}
}
}
}
//处理分支块
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("b"))
{
long offset = getLongFromOpConst(ins.getOpStr());
if(offset != 0)
{
long target = offset + instructions.peek().getAddr() - module.base;
//直接跳向主发生器
if(target == dispatcher)
{
instructions.pop();
ins = instructions.peek().getIns();
if(ins.getMnemonic().toLowerCase(Locale.ROOT).equals("csel"))
{
String[] sp = ins.getOpStr().toLowerCase(Locale.ROOT).split(",");
if(sp[0].trim().equals("w8"))
{
String cond = sp[3].trim();
String reg1 = sp[1].trim();
String reg2 = sp[2].trim();
selectBr sb = new selectBr();
sb.setInsaddr(instructions.peek().getAddr() - module.base);
sb.setCond(cond);
sb.setTrueindex(getRegValue(reg1,instructions.peek().getRegs()).longValue());
sb.setFalseindex(getRegValue(reg2,instructions.peek().getRegs()).longValue());
sbs.add(sb);
}
}
}
}
}
}
```

在模拟执行完后，发现有一些索引值没有找到对应的块：

![](upload/attach/202305/872365_FM4PT5VNQ9WQGVY.png)

查看他们的索引，发现刚好是条件块中的索引。这个也好理解，因为在模拟执行的时候我们只走了条件的一个分支，没有走另一个分支，所以就没有记录下对应的真实块。这里我们手动根据`b.eq`和寄存器的值，添加对应的真实块：

![](upload/attach/202305/872365_F3NFE72AA9UGAUW.png)

添加完成后，再次运行程序，发现已经`patch`成功了。打开`ida`查看`Jni_onload`:

![](upload/attach/202305/872365_FCZ6TP6324NSFSQ.png)

发现函数已经被完美的还原了。

# 4.总结

借着五一假期简单研究了一下模拟执行和去混淆，感慨`unidbg`是真的强。我认为我使用的方法应该是没有任何逻辑漏洞，可以完全等效的还原的。

间接跳转的还原比较简单，控制流平坦化的还原则比较难。

控制流平坦化还原的最大难点在与找到真实块与索引之间的关系。由于我这次使用的样本对于跳转做了混淆，虽然难住了`ida`，但是也可能难住了编译器，导致对控制流平坦化部分没有做复杂的编译优化——所有的真实块都在`b.eq`后面，并且只有一个主分发器，没有次分发器。同时真实块执行结束都很乖巧的跳转到了分发器（有的编译优化会直接将两个真实块连在一起）。

总体来说，本次反混淆不是特别完美，有手工的部分，同时也不能通杀所有混淆，只能说提供一种反混淆思路和经验吧。

`ollvm`反混淆感觉是永无止境的，目前还没有见到有什么非常强大，通杀，可以完美还原的方案，只能针对目标混淆器定制反混淆策略。

安全攻防不息，吾辈仍需努力！

# 0.前言

代码混淆是逆向分析中最令人头疼的问题之一。

面对混淆，通常的做法是直接硬着头皮调试，因为混淆只能混淆代码的形，但是代码的神——执行逻辑是不会变的，单步调试下去往往能梳理出程序本身的执行逻辑。但是面对非常复杂的混淆，小弟我也是经常调试数十小时最终招架不住选择放弃。。。有位伟人曾经说过“安全保护的目的不在于让代码固若金汤，无法攻破，而是在于让破解者望而生畏，主动放弃”，正所谓“不战而屈人之兵，善之善也~”，小弟我深以为然。

面对混淆更好的做法是去混淆，或者说反混淆，就是将混淆过的代码还原回去。还原后的代码通常逻辑清晰，通俗易懂，可以很简单的进行分析。但是反混淆往往比较难，抛开各种定制的混淆不说，即使是最基本的`ollvm`混淆，还原起来也不是那么容易。因为还原混淆通常要依赖于符号执行，模拟执行，指令`trace`等一系列高级的逆向手法。小弟我最近刚好遇到一个混淆的样本，趁着五一放假，研究了一下反混淆的方法，最终成功还原出了真实逻辑，故借此机会和看雪各位安全爱好者做一分享。

![](https://bbs.kanxue.com/upload/attach/202305/872365_A99C7H8CJ84NXC5.png)

混淆前的程序，可以看到在 `br x9` 后`ida`无法分析了，因为`x9`是一个寄存器，`ida`并不知道里面是什么值，所以无法继续向下分析。`F5`后可以看到，函数调用`jni->GetEnv`函数后就`jumpout`了。

![](https://bbs.kanxue.com/upload/attach/202305/872365_B42EXRHYUWVR3U2.png)

往下查看汇编代码，可以看到函数里有许多根据寄存器跳转的指令：

![](https://bbs.kanxue.com/upload/attach/202305/872365_ARURQ4UENRGUXF6.png)

# 1.unidbg环境搭建

去混淆通常要对代码进行模拟执行。

模拟执行届扛把子的是`unicorn`引擎。不过`unicorn`引擎相对比较底层，需要手动的开辟内存，将指令写入，设置堆栈的位置，设置寄存器初始值，然后执行。`unidbg`是基于`unicorn`引擎的模拟执行框架，使用`java`语言编写，并且可以直接加载`so`，进行调用（`unidbg`在加载动态库时自动帮你完成了文件解析，映射，重定位，写入`unicorn`引擎等一系列底层操作），同时`unidbg`内置了`Jni`相关的环境，而`unicorn`本身没有相关环境，需要手动补充。同时，`unidbg`还内置了`hookZz`，`xHook`等一系列`hook`框架，可以很方便的进行函数级，指令级的各种`hook`。总之，`unidbg`大大方便了逆向人员进行模拟执行的流程，所以我们选用`unidbg`来进行模拟执行。

## 1.1 下载源码

`unidbg`的工作模式简单粗暴，直接`git clone unidbg`的项目，然后在项目里添加自己的模拟执行类，最终运行的时候是带着整个项目的源码一起运行的，这样做的好处是方便了定制化修改和源码调试。

![](https://bbs.kanxue.com/upload/attach/202305/872365_AZUAZCW4GHCMWAY.png)

下载好后使用`idea`进行打开。

## 1.2 模拟执行

直接在项目的`unidbg-android/src/test/java`目录下建立我们的模拟执行类：`AntiOllvm`

在进行模拟执行之前，需要做以下准备工作：

1.创建一个模拟器。

2.设置`android`系统库的版本

3.创建`android`虚拟机

4.加载动态库

```java
public AntiOllvm()
{
/创建模拟器
emulator = AndroidEmulatorBuilder
.for64Bit()
.addBackendFactory(new Unicorn2Factory(true))
.setProcessName("com.example.antiollvm")
.build();
Memory memory = emulator.getMemory();
/设置andorid系统库版本
memory.setLibraryResolver(new AndroidResolver(26));
/创建虚拟机
vm = emulator.createDalvikVM();
vm.setVerbose(true);
/加载动态库
dm = vm.loadLibrary(new File("f:\\kshs\\libtprt.so"), false);
module = dm.getModule();
}
```

这样，动态库就被`unidbg`加载了。

加载后需要先执行`jni_onload`，而`DalvikModule`这个类已经实现了`callJNI_OnLoad`方法，我们直接调用即可。

```java
public static void main(String[] args) {
AntiOllvm ao = new AntiOllvm();
ao.callJniOnload();
}
public void callJniOnload()
{
dm.callJNI_OnLoad(emulator);
}
```

调用后，出现了报错和许多`warning`：

![](https://bbs.kanxue.com/upload/attach/202305/872365_VEP2WYTHRF9DU6E.png)

一方面，有许多`libc`的函数没有找到，另一方面，`JNI`返回了`-1`。

我们先补充所需的`libc`函数，只需要把手机里`/system/lib64/libc.so`等系统库交给`unidbg`加载即可。

将下列代码添加到加载目标`so`之前：

```java
vm.loadLibrary(new File("f:\\androidlib\\libc.so"),false);
vm.loadLibrary(new File("f:\\androidlib\\libm.so"),false);
vm.loadLibrary(new File("f:\\androidlib\\libstdc++.so"),false);
vm.loadLibrary(new File("f:\\androidlib\\ld-android.so"),false);
vm.loadLibrary(new File("f:\\androidlib\\libdl.so"),false);
```

再次执行，这次运行成功了，并且`unidbg` hook 了所有的`jni`掉用，也将他们打印了出来：

![](https://bbs.kanxue.com/upload/attach/202305/872365_DZ6EH66SXZXUE4C.png)

可以看到，在`0x5e900`处调用了`registernative`，注册的函数名叫：`initialize`，签名是：`(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;[Ljava/lang/String;Ljava/lang/String;)I`，函数的地址在`0x5e1e8`处。

至此，我们成功使用`unidbg`加载了`android`的动态库，并且正常运行了动态库的`jni_onload`函数。

## 1.3 基本的hook

我们已经成功运行了目标动态库，接下来可以看看还能做些什么。我们做一个最基本的操作，使用指令`hook`，在每条指令执行之前将其打印出来。
