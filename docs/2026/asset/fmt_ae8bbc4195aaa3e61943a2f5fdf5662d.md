
### ReadHook

`/* ReadHook注册类型为UC_HOOK_MEM_READ的Hook，只要在地址范围起始地址<=read_addr<=结束地址内执行内存读取，就会调用已注册的回调函数。当结束地址 < 起始地址时，将为所有内存读取调用回调 */`
`/* 注意：这个内存读取必须由指令进行，如果使用Unicorn的api mem_read进行读取内存，是不会进行回调的 */`
```java
unicorn.hook_add(new ReadHook() {
@Override
public void hook(Unicorn u,
long address,
int size,
Object user) {
byte[] bytes = u.mem_read(address, size);
System.out.print(String.format(">>> Memory read at 0x%x, block size = 0x%x, value is = 0x%s\n", address, size, Integer.toHexString(bytes[0] & 0xff)));
}
},
0,
-1, null);
```

我们执行上面的汇编指令，输出结果。就说明在`0x1200`地址上，有对内存的读取操作

```
>>> Memory read at 0x1200, block size = 0x4
the emulate finished result is ==> 9
```

### WriteHook

`/* WriteHook注册类型为UC_HOOK_MEM_WRITE的Hook，每当在地址范围起始地址<=write_addr<=结束地址内执行内存写入时，将调用已注册的回调函数。当结束地址 < 起始地址时，将为所有内存写入调用回调。 */`
`/* 注意：这个内存写入操作必须由指令进行，同ReadHook */`
```java
unicorn.hook_add(new WriteHook() {
@Override
public void hook(Unicorn u,
long address,
int size,
long value,
Object user) {
System.out.print(String.format(">>> Memory write at 0x%x, block size = 0x%x, value is = 0x%x\n", address, size, value));
}
},
0,
-1, null);
```

```
>>> Memory write at 0x1200, block size = 0x4, value is = 0x100b
the emulate finished result is ==> 9
```

### MemHook

`MemHook`就是`ReadHook`跟`WriteHook`的结合体，就不单独介绍了

### InterruptHook

`/* InterruptHook注册类型为UC_HOOK_INTR的Hook，每当执行中断指令时，将调用已注册的回调函数。 */`
`/* 在Linux的Arm架构中，所有的系统调用都是通过中断进入，且只有这一条入口，所以我们可以通过InterruptHook来处理系统调用 */`
```java
unicorn.hook_add(new InterruptHook() {
@Override
public void hook(Unicorn u,
int intno,
Object user) {
System.out.print(String.format(">>> Interrup occur, intno = %d\n",intno));
}
}, null);
```

加一条 `svc #0` 指令，产生异常

```
>>> Interrup occur, intno = 2
the emulate finished result is ==> 9
```

### EventMemHook

`/* 注册类型为UC_HOOK_MEM_XXX_UNMAPPED 或者 UC_HOOK_MEM_XXX_PROT的Hook，每当尝试从无效或受保护的内存地址进行读取或写入时，将调用注册的回调函数。 */`
```java
unicorn.hook_add(new EventMemHook() {
@Override
public boolean hook(Unicorn u,
long address,
int size,
long value,
Object user) {
System.out.println(String.format("UC_HOOK_MEM_READ_UNMAPPED at 0x%x, value = 0x%x",address,value));
return true;
}
}, UC_HOOK_MEM_READ_UNMAPPED, null);
```

我们加段指令，来读取`0x2000`（此处未map）的内容

```
movs r2, 0x2000
ldr r3, [r2]
```

```
UC_HOOK_MEM_READ_UNMAPPED at 0x2000, value = 0x0
unicorn.UnicornException: Invalid memory read (UC_ERR_READ_UNMAPPED)
```

## Keystone

接下来我们介绍下Keystone跟Capstone这对兄弟，先来看Keystone。

上面我们使用在线网站进行汇编代码与机器码的相互转换，使用起来非常的麻烦，我们就可以借助`Keystone`这个项目来帮助我们对汇编代码，直接转换成机器码

### 引入

```xml
<dependency>
<groupId>com.github.zhkl0228</groupId>
<artifactId>keystone</artifactId>
<version>0.9.5</version>
</dependency>
```

### 使用

```java
Keystone keystone = new Keystone(KeystoneArchitecture.Arm, KeystoneMode.ArmThumb);
String assembly =
"movs r0, #3\n" +
"movs r1, #2\n" +
"add r0,r1";
byte[] code = keystone.assemble(assembly).getMachineCode();
// code = [0x03, 0x20, 0x02, 0x21, 0x08, 0x44]
```

## Capstone

`Capstone`跟`Keystone`是相反的

### 引入

```xml
<dependency>
<groupId>com.github.zhkl0228</groupId>
<artifactId>capstone</artifactId>
<version>3.0.11</version>
</dependency>
```

### 使用

我们跟`CodeHook`结合来看下`Capstone`如何使用，也顺便看下它们的结合使用

```java
unicorn.hook_add(new CodeHook() {
@Override
public void hook(Unicorn u,
long address,
int size,
Object user) {
Capstone capstone = new Capstone(Capstone.CS_ARCH_ARM, Capstone.CS_MODE_THUMB);
Capstone.CsInsn[] disasm = capstone.disasm(u.mem_read(address, size), address);
for (Capstone.CsInsn ins : disasm) {
System.out.println("0x"+Long.toHexString(ins.address)+": "+ins.mnemonic+" "+ins.opStr );
}
}
},
0,
-1, null);
```

```
0x1000: movs r0, #3
0x1002: movs r1, #2
0x1004: add r0, r1
0x1006: bl #0x1100
0x1100: add r0, r1
0x1102: movs.w r2, #0x1200
0x1106: str.w lr, [r2]
0x110a: ldr.w pc, [r2]
0x100a: svc #0
0x100c: add r0, r1
the emulate finished result is ==> 9
```

## 总结

上面我们介绍了Unicorn的详细使用，有了Unicorn的加持，我们就可以模拟Arm架构的指令集，从而执行So中的各种指令，当然多条指令组合成的函数都是可以的。我们可以看到Unicorn本身提供了丰富的API供我们使用，它本身提供的Hook，覆盖了我们需要实现的所有场景。最后介绍了Keystone跟Capstone这对兄弟，那么本次的分享就结束啦，对本块内容有兴趣的朋友可以加个VX一起学习呀: roy5ue

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-10-25 18:48

被r0ysue编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
