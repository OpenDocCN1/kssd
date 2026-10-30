
# [原创]基于pwntools和seccomp-tools的awd pwn通防小工具

最初想法是想找到一种不影响程序本身运行，但是能够给程序禁用一些系统调用（`execve`、`open`等）来实现通防的方法。看了网上很多资料，感觉没有找到比较理想的解决方案，有一些是环境都搭不起来，有一些工具对文件本身内容改动太大了（加段），而`awd`中假如`check`的脚本是本地运行的话是有可能`check`文件修改字节数的。那就只能自己动手，丰衣足食了。

## 工具说明

### github地址

<a href="elink@c70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1g2q4W2Q4x3X3c8X3L8r3q4Y4i4K6u0r3k6i4k6A6L8q4m8S2N6r3y4Z5k6i4t1`."><mark class="encrypted">3f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1g2q4W2Q4x3X3c8X3L8r3q4Y4i4K6u0r3k6i4k6A6L8q4m8S2N6r3y4Z5k6i4t1`.</mark></a>

觉得还可以的话给个`star`~

### 特点

- 修改的字节数少，`0x100`字节以下
- 不修改文件头信息
- 沙箱规则可以自己定义
- 支持`i386`和`amd64`
- 支持`pie`开启和未开启情况

注：已在`ubuntu16`、`18`、`20`上测试过默认版本`gcc`编译出的程序，通过；暂不支持`g++`编译出的程序。

### 环境需求

运行需要依赖`seccomps-tools`和`pwntools`

`pwntools` <mark class="encrypted">d6fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2j5h3I4D9L8%4m8K6L8r3g2V1i4K6u0r3M7s2N6F1N6r3!0G2L8s2x3`.</mark>

`seccomps-tools` <mark class="encrypted">255K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2x3`.</mark>

### 目录说明

`sandboxs`文件夹

`sandboxs`里面存放了禁用的规则，这里根据自己的需要进行修改和选择。

实例（`sandbox1.asm`）

```
A = sys_number
A >= 0x40000000 ? dead : next
A == execve ? dead : next
A == open ? dead : next
return ALLOW
dead:
return KILL
```

`test`文件夹

执行`./complie.sh`生成四个可执行文件可供测试，分别是64/32位和`pie`开启/关闭的排列组合。

![](upload/attach/202206/913160_M8YTKPBMBZDXB8C.png)

### 运行脚本命令

第一个参数是想要`patch`的`elf`文件，第二个参数是沙箱规则文件，可以从`sandboxs`文件夹里选，假如想输出更多的中间过程可以在最后参数加上一个`1`。（已自动识别32位和64位）

```
Usage: python evil_patcher.py elfFile sandboxFile
python evil_patcher.py elfFile sandboxFile 1 (more detailed process message)
```

**运行**

![](upload/attach/202206/913160_JKTF4Q8D7FSN8CA.png)

结果输出一个`patch`过后的文件，文件名为原来文件加上`.patch`后缀，再改回原来的名字就可以开开心心传到靶机上了。

![](upload/attach/202206/913160_ENFTVBJ7Y8VKX83.png)

**验证**

![](upload/attach/202206/913160_WYDH2C7B7W58SFZ.png)

## 禁用系统调用的两种方式

### 1.seccomp库实现

这种方法需要安装一些库文件

```bash
sudo apt install libseccomp-dev libseccomp2 seccomp
```

实例

```c
#include <unistd.h>
#include <seccomp.h>
#include <linux/seccomp.h>
int main(void){
scmp_filter_ctx ctx;
ctx = seccomp_init(SCMP_ACT_ALLOW);
seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);
seccomp_load(ctx);
char * str = "/bin/sh";
write(1,"hello worldn",12);
syscall(59,str,NULL,NULL);//execve
return 0;
}
```

`scmp_filter_ctx`是过滤器的结构体

`seccomp_init`对结构体进行初始化，若参数为`SCMP_ACT_ALLOW`，则过滤为黑名单模式；若为`SCMP_ACT_KILL`，则为白名单模式，即没有匹配到规则的系统调用都会杀死进程，默认不允许所有的`syscall`。

`seccomp_rule_add`用来添加一条规则，`arg_cnt`为0,表示我们直接限制`execve`,不管参数是什么，如果`arg_cnt`不为0,那`arg_cnt`表示后面限制的参数的个数,也就是只有调用`execve`,且参数满足要求时,才会拦截

`seccomp_load`是应用过滤器,如果不调用`seccomp_load`则上面所有的过滤都不会生效

注意：编译的时候要在最后面加 `-lseccomp`

这该种方式实现的话会涉及到堆的操作，会留下堆使用痕迹，出题一般不用这种，假如栈题倒是可以用。正常用倒是方便，直接调库就行。

![](upload/attach/202206/913160_3APH6YH5U8DPGVY.png)

### 2.prctl实现

实例

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <time.h>
#include <string.h>
#include <stdlib.h>
#include <malloc.h>
#include <sys/prctl.h>
#include <seccomp.h>
#include <linux/seccomp.h>
#include <linux/filter.h>
int main() {
setvbuf(stdin, 0LL, 2, 0LL);
setvbuf(stdout, 0LL, 2, 0LL);
setvbuf(stderr, 0LL, 2, 0LL);
struct sock_filter filter[] = {
BPF_STMT(BPF_LD+BPF_W+BPF_ABS,4),
BPF_JUMP(BPF_JMP+BPF_JEQ,0xc000003e,0,2),
BPF_STMT(BPF_LD+BPF_W+BPF_ABS,0),
BPF_JUMP(BPF_JMP+BPF_JEQ,59,0,1),
BPF_STMT(BPF_RET+BPF_K,SECCOMP_RET_KILL),
BPF_STMT(BPF_RET+BPF_K,SECCOMP_RET_ALLOW),
};
struct sock_fprog prog = {
.len = (unsigned short)(sizeof(filter)/sizeof(filter[0])),
.filter = filter,
};
prctl(PR_SET_NO_NEW_PRIVS,1,0,0,0);
prctl(PR_SET_SECCOMP,SECCOMP_MODE_FILTER,&prog);
printf("start!\n");
system("id");
return 0;
}
```

主要用到两个结构体`sock_filter`和`sock_fprog`，`prog`会作为参数传入`prctl`,这里禁用了`execve`，`x64`的情况下其系统调用号为`59`。

效果图

![](upload/attach/202206/913160_X876RPMH6WDMRUK.png)

ps：一种更快捷的方式是通过`seccomp-tools`，可以直接生成代码，不过得提前写好沙箱规则的文件，格式和`sandboxs`文件夹中的文件一样。

![](upload/attach/202206/913160_DU5M9QAT45E9MVM.png)

## 实现思路

### 1.修改`_start`代码块

ELF执行流程图

![](upload/attach/202206/913160_9B2YBDZ7DVJ8Q4K.png)

程序的入口默认情况下都是`_start`函数，这里主要就是调用了`__libc_start_main`。

函数原型

```c
int __libc_start_main(int *(main) (int, char * *, char * *), int argc, char * * ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end));
```

`main`函数的地址就是第一个参数，那么我们可以修改这个参数为我们提前布置好的代码的地址，然后最后再跳转回`main`去执行正常流程，类似于病毒？不过这里藏得更深一点，病毒一般会修改程序入口点。

#### 32位情况

`pie`未开启

情况比较简单，直接修改`mov`的第二个操作数为我们要跳转的地址就行。

![](upload/attach/202206/913160_PCV8NUEZEC2X4FB.png)

`pie`开启

这里情况稍微特殊一点，`push ds:(main_ptr - 3FC4)[ebx]`指令长度为6个字节，`lea ecx, (unk_2060 - 3FC4h)[ebx];push ecx;`两条指令长度起来为7，所以后面的`call ___libc_start_main`指令其实往后移了一个字节，把`hlt`覆盖了。

![](upload/attach/202206/913160_38R69DJPY87WMTU.png)

#### 64位情况

64位情况比较简单，都是修改`mov`或者`lea`的第二个参数就行，因为64位情况下是通过寄存器来传递参数的。

`pie`未开启

![](upload/attach/202206/913160_RXARZ8BYBDNXETQ.png)

`pie`开启

![](upload/attach/202206/913160_8JH52JHZ6RHHHBP.png)

### 2.prctl汇编实现

我们需要实现的其实是以下两个函数完成的事情，只不过我们现在使用系统调用来完成。

```c
prctl(PR_SET_NO_NEW_PRIVS,1,0,0,0);
prctl(PR_SET_SECCOMP,SECCOMP_MODE_FILTER,&prog);
```

`PR_SET_NO_NEW_PRIVS`、`PR_SET_SECCOMP`、`SECCOMP_MODE_FILTER`这三个都是常数，那么只要解决`prog`和`prctl`系统调用汇编实现就行了。幸运的是`pwntools`对于系统调用汇编实现已经有了相应的模块，好耶，懒狗狂喜。

![](upload/attach/202206/913160_ZHGWDCMZ72ZFBJ6.png)

对于`sock_filter`和`sock_fprog`两个结构体可以当一维数组来看就行了，无脑`push`进栈就行了，不过最后还是需要进行堆栈平衡操作的，主要怕程序结束的时候异常。

**注入代码放在哪？**

`.eh_frame`节

正常编译处理的程序一般都会包含这个节区，里面主要记载一些异常处理的信息，一般是只读权限的，但是加载到内存的时候是有可能和`.text`节在同一个页，具有执行权限（之后发现`ubuntu18.04`编译出来的是如下图，但`ubuntu16.04`编译出来的`.eh_frame`并没有和`.text`节在一个段里）

选择把`shellcode`注入到这个节上，这个节的长度也一般会>0x100个字节。

实现代码如下，不同情况汇编代码稍有区别，但思路一致。

![](upload/attach/202206/913160_UVXGPQ98HAMCC82.png)

### 3.修改段权限

编译出的文件`.eh_frame`节对应的那个段不一定有可执行权限，和编译器版本相关，那么我们只需要把`.eh_frame`节对应的那个段加上可执行权限就行。

![](upload/attach/202206/913160_MK4TPZ4V5NFD9DS.png)

ps：假如想手动修改段权限可以用`010Editor`直接改，非常好用，学习ELF文件格式用它非常好用。

![](upload/attach/202206/913160_3D778C7YAKQHMEP.png)

最初想法是想找到一种不影响程序本身运行，但是能够给程序禁用一些系统调用（`execve`、`open`等）来实现通防的方法。看了网上很多资料，感觉没有找到比较理想的解决方案，有一些是环境都搭不起来，有一些工具对文件本身内容改动太大了（加段），而`awd`中假如`check`的脚本是本地运行的话是有可能`check`文件修改字节数的。那就只能自己动手，丰衣足食了。

## 工具说明

### github地址

[442K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1g2q4W2Q4x3X3c8X3L8r3q4Y4i4K6u0r3k6i4k6A6L8q4m8S2N6r3y4Z5k6i4t1`.](https://bbs.kanxue.com/elink@144K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1g2q4W2Q4x3X3c8X3L8r3q4Y4i4K6u0r3k6i4k6A6L8q4m8S2N6r3y4Z5k6i4t1`.)

觉得还可以的话给个`star`~

### 特点

- 修改的字节数少，`0x100`字节以下
- 不修改文件头信息
- 沙箱规则可以自己定义
- 支持`i386`和`amd64`
- 支持`pie`开启和未开启情况

注：已在`ubuntu16`、`18`、`20`上测试过默认版本`gcc`编译出的程序，通过；暂不支持`g++`编译出的程序。

### 环境需求

运行需要依赖`seccomps-tools`和`pwntools`

`pwntools` [74eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2j5h3I4D9L8%4m8K6L8r3g2V1i4K6u0r3M7s2N6F1N6r3!0G2L8s2x3`.](https://bbs.kanxue.com/elink@862K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2j5h3I4D9L8%4m8K6L8r3g2V1i4K6u0r3M7s2N6F1N6r3!0G2L8s2x3`.)

`seccomps-tools` [381K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2x3`.](https://bbs.kanxue.com/elink@e17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2x3`.)

### 目录说明

**`sandboxs`文件夹**

`sandboxs`里面存放了禁用的规则，这里根据自己的需要进行修改和选择。

实例（`sandbox1.asm`）

```
A = sys_number
A >= 0x40000000 ? dead : next
A == execve ? dead : next
A == open ? dead : next
return ALLOW
dead:
return KILL
```

**`test`文件夹**

执行`./complie.sh`生成四个可执行文件可供测试，分别是64/32位和`pie`开启/关闭的排列组合。
