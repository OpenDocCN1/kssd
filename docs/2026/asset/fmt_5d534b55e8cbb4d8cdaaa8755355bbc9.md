
```python
import idaapi
import idc
import idautils
class MyDbgHook(idaapi.DBG\_Hooks):
""" Own debug hook class that implementd the callback functions """
def dbg\_process\_start(self, pid, tid, ea, name, base, size):
print("MyDbgHook Process started, pid=%d tid=%d name=%s" % (pid, tid, name))
# add breakpoint at entry
idc.add\_bpt(StartEA(), 0, BPT\_ENABLED | BPT\_UPDMEM)
# IDA creates a segment named "TIB[XXXXXXXX]", which points to
# wow\_peb64 antually. We can get peb from wow\_peb64 with 0x1000 offset.
# peb\_addr = wow\_peb64\_addr + 0x1000
# Note: IDA has not created segment "TIB[XXXXXXXX]" at this point.
# tid = get\_current\_thread()
# tib\_segm\_name = "TIB[%08X]" % tid
# print tib\_segm\_name
# tib\_segm = get\_segm\_by\_name(tib\_segm\_name)
# wow\_peb64 = tib\_segm.start\_ea
# peb = tib\_segm.start\_ea + 0x1000
# on debugging start, ebx points to peb
# get addrs of peb and wow\_peb64
ebx = get\_reg\_value("ebx")
peb = ebx
wow\_peb64 = peb - 0x1000
# patch peb->BeingDebugged
# solving peb->NtGlobalFlag and "Heap Magic" anti-debug method
# at the same time.
patch\_dbg\_byte(peb + 2, 0)
patch\_dbg\_byte(wow\_peb64 + 2, 0)
# patching peb process paramters
peb\_process\_parameters = idaapi.get\_dword(peb + 0x10)
flag = idaapi.get\_dword(peb\_process\_parameters + 0x8)
patch\_dword(peb\_process\_parameters + 0x8, flag | 0x4000)
# patching peb64 process paramters
peb64\_process\_parameters = idaapi.get\_qword(wow\_peb64 + 0x20)
flag = idaapi.get\_dword(peb64\_process\_parameters + 0x8)
patch\_dword(peb64\_process\_parameters + 0x8, flag | 0x4000)
def dbg\_process\_exit(self, pid, tid, ea, code):
print("MyDbgHook Process exited pid=%d tid=%d ea=0x%x code=%d" % (pid, tid, ea, code))
if self.nt\_query\_infomation\_process != BADADDR:
idc.del\_bpt(self.nt\_query\_infomation\_process)
if self.nt\_close != BADADDR:
idc.del\_bpt(self.nt\_close)
if self.process32\_next != BADADDR:
idc.del\_bpt(self.process32\_next)
def dbg\_library\_unload(self, pid, tid, ea, info):
print("MyDbgHook Library unloaded: pid=%d tid=%d ea=0x%x info=%s" % (pid, tid, ea, info))
return 0
def dbg\_process\_attach(self, pid, tid, ea, name, base, size):
print("MyDbgHook Process attach pid=%d tid=%d ea=0x%x name=%s base=%x size=%x" % (pid, tid, ea, name, base, size))
def dbg\_process\_detach(self, pid, tid, ea):
print("MyDbgHook Process detached, pid=%d tid=%d ea=0x%x" % (pid, tid, ea))
return 0
def dbg\_library\_load(self, pid, tid, ea, name, base, size):
print "MyDbgHook Library loaded: pid=%d tid=%d name=%s base=%x" % (pid, tid, name, base)
def dbg\_suspend\_process(self):
print "MyDbgHook Process suspended"
ev\_ea = get\_event\_ea()
if ev\_ea == StartEA():
self.nt\_query\_infomation\_process = get\_name\_ea\_simple("ntdll\_NtQueryInformationProcess")
if self.nt\_query\_infomation\_process != BADADDR:
print("adding conditional breakpoint at NtQueryInformationProcess 0x%x" % self.nt\_query\_infomation\_process)
idc.add\_bpt(self.nt\_query\_infomation\_process)
cond = """
import idautils
import idaapi
process\_information\_class = idaapi.get\_dword(idautils.cpu.esp + 8)
if process\_information\_class == 7 or process\_information\_class == 30 \
or process\_information\_class == 31 or process\_information\_class ==35:
return True
else:
return False
"""
idc.set\_bpt\_cond(self.nt\_query\_infomation\_process, cond)
self.nt\_close = get\_name\_ea\_simple("ntdll\_NtClose")
if self.nt\_close != BADADDR:
print("adding breakpoint at NtClose 0x%x" % self.nt\_close)
cond = """
import idautils
import idaapi
handle = idaapi.get\_dword(idautils.cpu.esp + 4)
if handle > 0x1000:
return True
else:
return False
"""
idc.add\_bpt(self.nt\_close)
idc.set\_bpt\_cond(self.nt\_close, cond)
self.process32\_next = get\_name\_ea\_simple("kernel32\_Process32Next")
if self.process32\_next != BADADDR:
idc.add\_bpt(self.process32\_next)
idc.del\_bpt(StartEA())
idaapi.continue\_process()
elif ev\_ea == self.nt\_query\_infomation\_process:
print "NtQueryInformationProcess"
ret\_addr = idaapi.get\_dword(cpu.esp)
process\_handle = idaapi.get\_dword(cpu.esp + 4)
process\_information\_class = idaapi.get\_dword(cpu.esp + 8)
process\_information = idaapi.get\_dword(cpu.esp + 12)
process\_information\_length = idaapi.get\_dword(cpu.esp + 16)
return\_length = idaapi.get\_dword(cpu.esp + 20)
if process\_information\_class == 7 or process\_information\_class ==30:
print("NtQueryInformationProcess(ProcessHandle=%x, ProcessInformationClass=%x, ProcessInformation=%x, ProcessInformationLength=%d, ReturnLength=%d)" \
% (process\_handle, process\_information\_class, process\_information, process\_information\_length, return\_length))
patch\_dword(process\_information, 0)
cpu.eax = 0
cpu.esp = cpu.esp + 24
cpu.eip = ret\_addr
elif process\_information\_class == 31:
print("NtQueryInformationProcess(ProcessHandle=%x, ProcessInformationClass=%x, ProcessInformation=%x, ProcessInformationLength=%d, ReturnLength=%d)" \
% (process\_handle, process\_information\_class, process\_information, process\_information\_length, return\_length))
patch\_dword(process\_information, 1)
cpu.eax = 0
cpu.esp = cpu.esp + 24
cpu.eip = ret\_addr
elif process\_information\_class == 35:
print("NtQueryInformationProcess(ProcessHandle=%x, ProcessInformationClass=%x, ProcessInformation=%x, ProcessInformationLength=%d, ReturnLength=%d)" \
% (process\_handle, process\_information\_class, process\_information, process\_information\_length, return\_length))
patch\_word(process\_information, 0)
cpu.eax = 0
cpu.esp = cpu.esp + 24
cpu.eip = ret\_addr
idaapi.continue\_process()
elif ev\_ea == self.nt\_close:
ret\_addr = idaapi.get\_dword(cpu.esp)
handle = idaapi.get\_dword(cpu.esp + 4)
print("NtClose = %x" % self.nt\_close)
print("NtClose(Handle=0x%x)" % handle)
cpu.eax = 0
cpu.esp = cpu.esp + 8
cpu.eip = ret\_addr
idaapi.continue\_process()
elif ev\_ea == self.process32\_next:
ret\_addr = idaapi.get\_dword(cpu.esp)
h\_snapshot = idaapi.get\_dword(cpu.esp + 4)
lppe = idaapi.get\_dword(cpu.esp + 8)
print("Process32Next(hSnapshot=0x%x, lppe=0x%x)" % (h\_snapshot, lppe))
cpu.eax = 0
cpu.esp = cpu.esp + 12
cpu.eip = ret\_addr
idaapi.continue\_process()
# Remove an existing debug hook
try:
if debughook:
print("Removing previous hook ...")
debughook.unhook()
except:
pass
# Install the debug hook
debughook = MyDbgHook()
debughook.hook()
load\_and\_run\_plugin('python', 3)
```

## 测试结果

使用了`ScyllaHide`中的`ScyllaTest_x86.exe`和`

看雪.TSRC 2017CTF秋季赛中的第三题CrackMe`做了测试，结果如下。

`ScyllaTest_x86.exe`

![](upload/attach/201805/677218\_2SNBCSR4J68NYM5.png)

`CrackMe`，其特点是用了很多反调试手段，但是由于作者的失误，导致出现了多解，并且其中一个解还会在调试过程中以明文的形式作为`memcmp`的参数。

![](upload/attach/201805/677218\_VWZEK9BMCQC3XTP.png)

![](upload/attach/201805/677218\_BSJCX28TPFTPJVD.png)

测试`Crackme`时对异常的设置

![](upload/attach/201805/677218\_S4NBNR3CFTTJ38Y.png)

## 下一步的改进

- 目前，`Hook`掉的函数还只有两个，可以添加更多的函数`Hook`
- 现在所有的`Hook`都是在程序的入口点做的，可以利用`dbg_library_load`在`dll`加载时进行`Hook`
- 利用软断点（`0xCC`）进行`Hook`可能被检测，可以改为利用动态插桩技术。

## 总结

反调试的方法很多，一个脚本肯定是解决不完的，只有搞清楚了反调试原理才能更好的对付反调试。写这个小脚本，是为了学习`windows`平台下各种反调试技术，同时学习一下`IDAPython`

## 更新

加一个`github`地址[`DBGHider`](https://bbs.kanxue.com/elink@adaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6N6$3g2A6P5X3W2E0k6g2)9J5c8V1c8n7c8@1S2A6k6r3g2J5)，里面加了`inline hook`。

## 背景

许多人喜欢用`IDA`进行静态分析，用`OllyDbg`进行动态分析，似乎很少人喜欢使用`IDA`进行调试。然而我这个菜鸟不会用`OD`，所以没有体会它的强大之处，我更喜欢用`IDA`的调试器。

`IDA`的调试器功能也很强大。

- 支持`x64`位程序。
- 可以充分利用静态分析的结果。
- 支持多种平台，`Windows`版本的`IDA`支持本地`Windows`调试，同时也支持远程`Windows`、`远程Linux`、`远程Android`、`远程Mac`调试。
- 支持多种架构，远程`Android`调试自然要用到`ARM`架构，配合`QEMU`模拟器还可以调试`MIPS`等架构。

由于`IDA 7.x`更改的`SDK`的`API`，就带来了一个很尴尬的事情，没有使用于`IDA 7.x`的反反调试插件，所以利用空闲时间写了一个小脚本，实现了对部分反调试方法的绕过。

## 反调试方法的分类

就我个人的理解，反调试的方法可大致分为以下3类：

- `PEB->BeingDebugged`类，包括调用`IsDebuggerPresent()`或手动检测`BeingDebugged`标志。以及由于设置了`BeingDebugged`标志所引起的连锁反应，包括`NtGlobalFlag`, `Heap Flags`等。这种反调试方法可以通过在程序运行前清除标记位来绕过反调试。
- 函数调用类，包括调用`CheckRemoteDebuggerPresent()`，`NtQueryInformationProcess()`函数等。这类方法无法通过清楚标志位来绕过，但可以通过`Hook`相应函数来绕过。
