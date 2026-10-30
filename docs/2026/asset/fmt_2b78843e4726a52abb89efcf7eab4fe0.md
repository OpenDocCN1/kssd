
## 04\_angr\_symbolic\_stack

```python
import angr
project=angr.Project("./04\_angr\_symbolic\_stack",auto\_load\_libs=False)
state=project.factory.blank\_state(addr=0x08048694)
import claripy
input1=claripy.BVS("input1",32)
input2=claripy.BVS("input2",32)
state.regs.ebp=state.regs.esp
state.regs.esp-=0x1c
state.memory.store(state.regs.ebp-0xc,input1)
state.memory.store(state.regs.ebp-0x10,input2)
sim=project.factory.simgr(state)
sim.explore(find=0x080486E4)
if sim.found:
res=sim.found[0]
res=res.solver.eval(input1)
print(res)
res=sim.found[0]
res=res.solver.eval(input2)
print(res)
```

通过 `state.memory.store(addr,value)` 可以对内存进行符号化，从而在路径发现以后进行求解。

## 05\_angr\_symbolic\_memory

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
int i; // [esp+Ch] [ebp-Ch]
memset(&user\_input, 0, 33);
printf("Enter the password: ");
\_\_isoc99\_scanf("%8s %8s %8s %8s", &user\_input, &unk\_A1BA1C8, &unk\_A1BA1D0, &unk\_A1BA1D8);
for ( i = 0; i <= 31; ++i )
\*(i + 169583040) = complex\_function(\*(i + 169583040), i);
if ( !strncmp(&user\_input, "NJPURZPCDYEAXCSJZJMPSOMBFDDLHBVN", 32) )
puts("Good Job.");
else
puts("Try again.");
return 0;
}
```

这道题同样因为现在的 `Angr` 功能强大而不需要以前那样复杂的技巧了：

```python
import angr
project=angr.Project("./05\_angr\_symbolic\_memory",auto\_load\_libs=False)
state=project.factory.entry\_state()
sim=project.factory.simgr(state)
sim.explore(find=0x0804866D)
if sim.found:
res=sim.found[0]
print(res.posix.dumps(0))#b'NAXTHGNR JVSFTPWE LMGAUHWC XMDCPALU'
```

而题目的本意是让我们将内存符号化，其实和上一题一样，直接对内存进行存储就行了：

```python
import angr
project=angr.Project("./05\_angr\_symbolic\_memory",auto\_load\_libs=False)
state=project.factory.blank\_state(addr=0x080485FE)
import claripy
pwd1=claripy.BVS("pwd1",64)
pwd2=claripy.BVS("pwd2",64)
pwd3=claripy.BVS("pwd3",64)
pwd4=claripy.BVS("pwd4",64)
state.memory.store(0x0A1BA1C0,pwd1)
state.memory.store(0x0A1BA1C0+8,pwd2)
state.memory.store(0x0A1BA1C0+8+8,pwd3)
state.memory.store(0x0A1BA1C0+8+8+8,pwd4)
sim=project.factory.simgr(state)
sim.explore(find=0x0804866D)
if sim.found:
res=sim.found[0]
print(res.solver.eval(pwd1))
print(res.solver.eval(pwd2))
print(res.solver.eval(pwd3))
print(res.solver.eval(pwd4))
```

## 06\_angr\_symbolic\_dynamic\_memory

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
\_BYTE \*v3; // ebx
\_BYTE \*v4; // ebx
int v6; // [esp-18h] [ebp-24h]
int v7; // [esp-14h] [ebp-20h]
int v8; // [esp-10h] [ebp-1Ch]
int v9; // [esp-8h] [ebp-14h]
int v10; // [esp-4h] [ebp-10h]
int v11; // [esp+0h] [ebp-Ch]
int i; // [esp+0h] [ebp-Ch]
buffer0 = malloc(9, v6, v7, v8);
buffer1 = malloc(9, v9, v10, v11);
memset(buffer0, 0, 9);
memset(buffer1, 0, 9);
printf("Enter the password: ");
\_\_isoc99\_scanf("%8s %8s", buffer0, buffer1);
for ( i = 0; i <= 7; ++i )
{
v3 = (\_BYTE \*)(buffer0 + i);
\*v3 = complex\_function(\*(char \*)(buffer0 + i), i);
v4 = (\_BYTE \*)(buffer1 + i);
\*v4 = complex\_function(\*(char \*)(buffer1 + i), i + 32);
}
if ( !strncmp(buffer0, "UODXLZBI", 8) && !strncmp(buffer1, "UAORRAYF", 8) )
puts("Good Job.");
else
puts("Try again.");
free(buffer0);
free(buffer1);
return 0;
}
```

和上一题不同的地方在于，这次的存储位置为堆内存，我们不能直接给出一个地址然后去存储。

一把梭还是可行的：

```python
import angr
project=angr.Project("./06\_angr\_symbolic\_dynamic\_memory",auto\_load\_libs=False)
state=project.factory.entry\_state()
sim=project.factory.simgr(state)
sim.explore(find=0x08048759)
if sim.found:
res=sim.found[0]
print(res.posix.dumps(0))
```

而标准做法是：

```python
import angr
project=angr.Project("./06\_angr\_symbolic\_dynamic\_memory",auto\_load\_libs=False)
state=project.factory.blank\_state(addr=0x08048699)
buff0=0x0ABCC8A4
buff1=0x0ABCC8AC
import claripy
pwd1=claripy.BVS("pwd1",64)
pwd2=claripy.BVS("pwd2",64)
state.memory.store(buff0,0xffffff00,endness=project.arch.memory\_endness)
state.memory.store(buff1,0xffffff80,endness=project.arch.memory\_endness)
state.memory.store(0xffffff00,pwd1)
state.memory.store(0xffffff80,pwd2)
sim=project.factory.simgr(state)
sim.explore(find=0x08048759)
if sim.found:
res=sim.found[0]
print(res.solver.eval(pwd1))
print(res.solver.eval(pwd2))
```

通过这题就能够理解符号执行的一个好处了。由于它并不是真的去执行，只是模拟执行代码而已，所以对地址本身没有限制，完全可以随意设定内存的使用方法。

另外 `endness` 参数用于指定储存的端序，而 `project.arch.memory\_endness` 将会反映程序所在平台的默认端序，此处为小端序。

## 07\_angr\_symbolic\_file

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
int result; // eax
int i; // [esp+Ch] [ebp-Ch]
memset(&buffer, 0, 64);
printf("Enter the password: ");
\_\_isoc99\_scanf("%64s", &buffer);
ignore\_me(&buffer, 64);
memset(&buffer, 0, 64);
fp = fopen("OJKSQYDP.txt", "rb");
fread(&buffer, 1, 64, fp);
fclose(fp);
unlink("OJKSQYDP.txt");
for ( i = 0; i <= 7; ++i )
\*(\_BYTE \*)(i + 134520992) = complex\_function(\*(char \*)(i + 134520992), i);
if ( strncmp(&buffer, "AQWLCTXB", 9) )
{
puts("Try again.");
exit(1);
}
puts("Good Job.");
exit(0);
\_libc\_csu\_init();
return result;
}
```

可以发现程序调用了 `fopen` 去打开文件，对于这种情况，`Angr` 也同样提供了模拟文件的系统。

同样的，照旧一把梭也能搞定：

```python
import angr
project=angr.Project("./07\_angr\_symbolic\_file",auto\_load\_libs=False)
state=project.factory.entry\_state()
sim=project.factory.simgr(state)
sim.explore(find=0x080489B0)
if sim.found:
res=sim.found[0]
print(res.posix.dumps(0))
#b'AZOMMMZM\x00@\x04\x00\x01\x01\x01\x01\x01\x00\x00\x00\x02\x00\x01\x00\x80\x04\x80\x00\x02\x01\x04\x00\x02\x80\x08\x01\x00\x02\x01\x01\x01@\x01\x00\x08\x08\x04\x80\x04\x01\x80\x01\x04\x80\x02\x00\x00@\x00\x00\x00\x00\x00\x00'
```

不过还是来看看它的模拟文件系统吧：

```python
import angr
import claripy
project=angr.Project("./07\_angr\_symbolic\_file",auto\_load\_libs=False)
state=project.factory.blank\_state(addr=0x080488EA)
filename = 'OJKSQYDP.txt'
pwd1=claripy.BVS("pwd1",64\*8)
pwdfile=angr.storage.SimFile(filename,content=pwd1,size=64)
state.fs.insert(filename,pwdfile)
sim=project.factory.simgr(state)
sim.explore(find=0x080489B0)
if sim.found:
res=sim.found[0]
print(hex(res.solver.eval(pwd1)))
#0x415a4f4d4d4d5a4d0000000000000000000000000002000020000000000200000000000000008000000000401002000000000000000000000004001000000000
```

前几个还是照旧，但是也有一些新东西：

```python
pwdfile=angr.storage.SimFile(filename,content=pwd1,size=64)
state.fs.insert(filename,pwdfile)
```

`angr.storage.SimFile` 提供了一个模拟文件系统，通过 `state.fs.insert` 可以将该模拟出来的文件插入到 `state` 符号中。这样在模拟执行时就会用该文件替代真实情况下的文件了。

而 `angr.storage.SimFile` 的 `filename` 参数表示文件名，`content` 参数表示文件内容，`size` 参数表示文件大小，单位为字节。

## 08\_angr\_constraints

```c
int \_\_cdecl main(int argc, const char \*\*argv, const char \*\*envp)
{
int i; // [esp+Ch] [ebp-Ch]
qmemcpy(&password, "AUPDNNPROEZRJWKB", 16);
memset(&buffer, 0, 17);
printf("Enter the password: ");
\_\_isoc99\_scanf("%16s", &buffer);
for ( i = 0; i <= 15; ++i )
\*(i + 134520912) = complex\_function(\*(i + 134520912), 15 - i);
if ( check\_equals\_AUPDNNPROEZRJWKB(&buffer, 16) )
puts("Good Job.");
else
puts("Try again.");
return 0;
}
```

```c
BOOL \_\_cdecl check\_equals\_AUPDNNPROEZRJWKB(int a1, unsigned int a2)
{
int v3; // [esp+8h] [ebp-8h]
unsigned int i; // [esp+Ch] [ebp-4h]
v3 = 0;
for ( i = 0; i < a2; ++i )
{
if ( \*(i + a1) == \*(i + 134520896) )
++v3;
}
return v3 == a2;
}
```

在这里就能遇到之前所说的 “路径爆炸” 问题了。

照例试试一把梭：

```python
import angr
project=angr.Project("./08\_angr\_constraints",auto\_load\_libs=False)
state=project.factory.entry\_state()
sim=project.factory.simgr(state)
sim.explore(find=0x08048694)
if sim.found:
print("yes")
```

会发现这次就没办法那么顺利得到答案了，`Angr` 求解了半天却一直没有给出 “yes” 的回答，因此这次我们必须手动去优化求解的过程。

分析 `check\_equals\_AUPDNNPROEZRJWKB` 函数可以发现，该函数实际上是在对输入和 `password` 对比，而 `password` 的值是固定的 `AUPDNNPROEZRJWKB` 。

因此第一种缓解路径爆炸的方法是，只需要探索到进入该路径即可。而此后的求解过程通过人为的方法手动增加。

首先还是创建状态，这里我们跳过了 `scanf` ：

```python
import angr
project=angr.Project("./08\_angr\_constraints",auto\_load\_libs=False)
state=project.factory.blank\_state(addr=0x08048625)
```

接下来我们为 `buffer` 创建符号，并开始探索：

```python
import claripy
pwd=claripy.BVS("pwd",16\*8)
state.memory.store(0x0804A050,pwd)
sim=project.factory.simgr(state)
sim.explore(find=0x08048565)
```
