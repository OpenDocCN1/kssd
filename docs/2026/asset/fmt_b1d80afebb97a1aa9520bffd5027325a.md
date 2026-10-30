
如上图，是`sk_buff`的内存结构图，`data`的数据区是可以存储，`frags[*]`里面的这些也是可以存储的，`data`的数据区很小，应该不是很大，如果一次发送很大的数据，应该是存储到`frags`里面的。`sk_buff`和`skb_shared_info`虽然不是一个结构体，但是内存是相邻的。

`skb_copy_and_csum_bits()`函数，是将数据拷贝到新申请的`skb`的`data`数据区了，而这个`data`，跟`skb_shared_info`数据是相邻的，这就意味着，如果数据超过`data`的大小，就能将数据覆盖到`skb_shared_info`的位置。`skb_shared_info`就是那个我们要溢出的指针函数所在的内存位置。(这里不具体写了，相关资料里有，这里比较简单)

## 3、找到发送溢出的位置了，如何溢出到指定的位置

我们直接看`poc`，发送了3484个字节，而`skb_shared_info`的位置，在3165，也就是说，需要3164个字节都是来覆盖`data`数据，`mtu=1500,udphead = 8,iphead = 20,3164-1500+8+20=1692`，第一个`skb`填满数据后，开始在新分配的`skb`进行溢出`1692+20 = 1712`（不需要`udp`头了），从这里可以推测出`data`大小= 1712

我们也可以按照代码里的算，传入数据`3484 + 8+20 = 3512`，第一个`skb`所有数据长度。`3512+1500=2012` `2012-（3484-3164）+20 = 1712`。

实际过程中，我们申请了1531个字节，但是返回1728个字节（头部16个字节，从`head`开始算的），`tail=1516`，`end = 1728`。

所以，1712刚好能覆盖`data`所有地址，后续溢出的就是`skb_shared_info`

# 三、环境搭建

vm虚拟机+ubuntu，参考这个

<mark class="encrypted">54cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0V1J5y4K6f1#2i4K6t1K6K9o6u0Q4x3X3b7$3</mark>

<mark class="encrypted">a7eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0x3&6y4U0p5#2y4b7`.`.</mark>

我使用的是linux mint主机，里面装了vm虚拟机，没有使用双机调试，但是`gdb`附加远程主机的时候，使用桥接的虚拟机的IP不能连接。

然后我试了试本机的IP，然后成功了

`target remote localhost:8864`

也可以使用`qemu`来调试，你可以使用上面的虚拟机安装的源码，或者自己下载对应的linux版本。不过根文件系统不能太简陋，必须具备一些功能。

本来是想用这个漏洞学习`syzkaller`的，但是最后出了点问题，不过，我发现他制作文件系统的工具`debootstrap`，有兴趣的可以了解下，我用他提供的教程制作了镜像，用于`syzkaller`测试，但是竟然，连接都失败了，后来解决了一大堆问题，还是无法连接，后来我发现，linux内核和文件系统不兼容，`debootstrap`是Debian提供的，制作的是根文件系统有版本的，后来我找到了这个，然后就一直用这个了

在这里下载镜像`wheezy.img`

<mark class="encrypted">854K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3V1k6K6P5i4A6C8j5h3I4D9k6i4u0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6r3!0U0M7#2)9J5c8Y4y4&6P5X3u0G2N6q4)9J5k6h3#2V1</mark>

# 四、漏洞利用技术

## 1、rop

>学术解释：ROP的全称为Return-oriented programming（返回导向编程），这是一种高级的内存攻击技术可以用来绕过现代操作系统的各种通用防御。

个人技术理解解释：

>搜索内存中后缀有`ret`指令的代码片段，构成一段payload，将它们的地址`push`到一个自定义的栈中，当能控制`ip`时，进行栈迁移，利用`ret`指令的特性，连续地执行代码片段

rop条件：

1. 后缀有`ret`的代码片段

2. `ip`

3. 栈迁移

找了个小demo学习了一下

源码地址：<mark class="encrypted">3d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3L8X3W2C8y4e0t1^5y4#2)9J5c8X3E0W2M7X3&6W2L8q4)9#2k6Y4u0G2M7l9`.`.</mark>

在这个漏洞中，或者在这个demo中，`ret`代码段，我们可以搜索出来，`ip`我们能拿到，那栈迁移那

### 栈迁移

我们有`ip`，但是我们应该只能控制调用到什么地址，让他执行，但是我们如何给`sp`设置值，又设置成什么值，我们可以构造一个栈进行rop，但是我们只能控制一个被调用地址

如果想要进行栈迁移，先用我们的猪脑子，想一想，自己该怎么办

```
mov rsp,xxx 栈迁移
ret 执行rop
```

想想，好像，不太可能有类似的，即使是有条转接指令`mov rxx, 0xfffxx`，然后`mov rsp, rxx`，这样也不行，你这个地址，得多幸运，才能找到啊。

看看具体实例，大神们如何做哪：

1. 先找`xchg eax, esp ; ret`

2. 通过这条指令地址算出栈的地址

3. 然后`mmap`算出来的地址，在这个地址上构造栈。

4. 控制的地址是`xchg eax, esp ; ret`的地址

开始看，我直接懵了，栈迁移那，`xchg eax, esp`，开始没想通，`eax`的值，怎么就变成你自己栈的位置了

看我们控制的位置的汇编，`call rax` 想了想，跳转地址，也就是`rax`的地址，不就是我控制的吗

`rax = (xchg eax, esp ; ret)`的地址；`esp = rax`低32位，我们申请的栈的位置就是`rax`低32位的地址

这就已经很明白了，我们跳转到`xchg eax, esp ; ret`，`rax`是可以控制的，也就是说，第一条指令进行了栈迁移，`ret`进入了rop

这个我们要考虑`call`的寄存器，最好调用的是`call rax`，我们就`xchg eax, esp`；如果是`call rbx`，我们就需要`xchg ebx, esp`

总结一下rop利用过程。

首先，找到我们控制`ip`的位置，反汇编，看看是通过什么方式过来的，是`call xxx`比如`rax`

因为我们是可以控制`rax`的，知道`rax`的值，当`call`过去以后，`rax`的值是不变的，所以我们可以让`rax = xchg eax, esp ; ret`这个片段的内存地址，这样，`esp = rax`低32位了。我们是事先知道`rax`的数值的，所以我们可以在`rax`低32位的地址申请内存，作为rop的栈。

## 2、namespace

这个网上查一查。资料基本一大堆，而这个漏洞，需要`mtu = 1500`，也可以设置成别的数，不过剩下的数据可能要自己计算。同时为了漏洞能够触发，必须找到一个能够建立连接，可以让系统进入发送udp包逻辑的网络地址，这无疑，`lo`最合适，别的地址，IP我们基本没法百分比保证，但是，`lo`的`mtu`基本都是65535，设置它的`mtu`是需要root权限的。是不是很尴尬。在`poc`里面，巧妙地利用了`namespace`。

我手工测试过`namespace`，`namespace`本身也有root一说的，而且，必须用root创建的`namespace`，才是root的`namespace`，才能设置网卡，用户创建的，是没有权限的，那么`poc`中是如何做到的

```
if (!write_file("/proc/self/uid_map", "0 %d 1\n", real_uid)) {
perror("[-] write_file(/proc/self/uid_map)");
exit(EXIT_FAILURE);
}
```

我测试了，这行代码，就可以在不使用root创建`namespace`的情况下，将`namespace`的权限变成root的，可以设置网卡了。

# 五、`gdb`自动化脚本

`gdb`也算是老朋友的，经常一调试就是一天，但是，问题在于，敲命令，键盘都敲烂了，各种数据也要自己记录，跑错一次，本次调试全部作废，时间浪费了很多，于是就研究了一下脚本，然后，真香。

官方文档

<mark class="encrypted">309K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3k6$3c8T1i4K6u0r3N6$3W2C8K9g2)9J5c8W2m8&6N6r3S2G2L8V1N6V1j5W2c8#2N6r3!0J5K9h3q4D9</mark>

没找到中文的，自己看了三天。。。

下面是多线程调试的例子

```
#include<stdio.h>
#include <pthread.h>
#include <sys/syscall.h>
void br(int a6){
printf("%s",a6);
printf("main thread lwpid = %u\n", syscall(SYS_gettid));
}
void *nbi(void *arg)
{
while(1){
sleep(1);
br(6);
}
}
int main(){
pthread_t tid,tid2,tid3;
printf("feafea");
printf("main thread lwpid = %u\n", syscall(SYS_gettid));
pthread_create(&tid, NULL, nbi, NULL);
pthread_create(&tid2, NULL, nbi, NULL);
pthread_create(&tid3, NULL, nbi, NULL);
pthread_join(tid, NULL);
pthread_join(tid2, NULL);
pthread_join(tid3, NULL);
return 0;
}
```

```
gdb.execute('file a.out', to_string=True)
class MyBreakpoint(gdb.Breakpoint):
def __init__(self, spec, command=""):
super(MyBreakpoint, self).__init__(spec, gdb.BP_BREAKPOINT, internal = False) #internal 断点，用户可见与否
self.command = command # not used
#self.thread = 3
print(spec)
def stop (self):
gdb.write('MyBreakpoint\n')
print(self.location)
print(self.number) #断点编号
print(self.hit_count) #触发次数
print(self.thread) #触发的线程号，gdb给线程的编号
t = gdb.selected_thread()
print(t.name)
print(t.num)
print(t.global_num)
print(t.ptid)
print(gdb.newest_frame() == gdb.selected_frame ())
top = gdb.selected_frame()
someVector = top.read_var('a6')
print(someVector)
print(top.name())
print(top.function())
print(hex(top.pc()))
print(top.older())
print(gdb.invalidate_cached_frames())
print(top.find_sal())
print(hex(top.read_register("rcx")))
# Continue automatically.
# return False
# Actually stop.
return True
gdb.execute("start")
MyBreakpoint('pygdbtst.c:9 thread 3')
gdb.execute('run')
```

放几个入门例子

```
bp1 = gdb.Breakpoint("myFunc()")
# Define handler routines
def stopHandler(stopEvent):
for b in stopEvent.breakpoints:
if b == bp1:
print "myFunc() breakpoint"
else:
print "Unknown breakpoint"
//gdb.execute("continue")
# Register event handlers
gdb.events.stop.connect (stopHandler) //注册gdb 调试器stop事件的函数
gdb.execute("run")
```

```
gdb.execute('file a.out', to_string=True) //直接gdb进入，然后选择要执行的文件
class MyBreakpoint(gdb.Breakpoint): //继承断掉类
def stop (self):
print('MyBreakpoint\n')
gdb.write('MyBreakpoint\n')
return False //继续执行
return True //停下来，等待用户交互
MyBreakpoint('br') //给br函数下断点
gdb.execute('run') //执行gdb的命令
```

```
//添加一个gdb命令,这个是gdb的demo，官方文档中有
class _LPrintfBreakpoint(gdb.Breakpoint):
def __init__(self, spec, command):
super(_LPrintfBreakpoint, self).__init__(spec, gdb.BP_BREAKPOINT,
internal = False)
self.command = command
print("init")
def stop(self):
print("stop")
gdb.execute(self.command)
return False
class _LPrintfCommand(gdb.Command):
def __init__(self):
super(_LPrintfCommand, self).__init__('main', //main命令，直接在gdb终端输入
gdb.COMMAND_DATA,
# Not really the correct
# completer, but ok-ish.
gdb.COMPLETE_SYMBOL)
def invoke(self, arg, from_tty): //mian命令执行函数
print("invoke")
(remaining, locations) = gdb.decode_line(arg)
if remaining is None:
raise gdb.GdbError('printf format missing')
remaining = remaining.strip(',')
if locations is None:
raise gdb.GdbError('no matching locations found')
spec = arg[0:- len(remaining)]
_LPrintfBreakpoint(spec, 'printf ' + remaining)
_LPrintfCommand()
```

这个是cve调试的时候使用的，效果很好
