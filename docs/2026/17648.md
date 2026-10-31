
# Linux 动态注入（含视频演示）

如果纯粹用文字来描述什么是动态注入，可能还是不太容易理解，所以本篇文章从如下一段代码开始：

```c
// who.c

#include <stdio.h>

#include <unistd.h>



int main()

{

	while (1) {

		printf("who are you ?\n");

		sleep(2);

	}



	return 0;

}
```

这段代码本身没有什么意思，执行`gcc who.c -o who -g -Wall`完成编译并启动`who`程序，每隔 2s 会向终端打印一句"who are you ?"，但有意思的是，可能会出现一种"诡异"的现象，比如屏幕上突然冒出一句"it's me ~"。

## 1. 为什么可以出现这种"诡异"的现象？

- 通过反汇编`who`程序可以看出，程序最终会进入`while(1)`循环体，重复执行`bf24064000`、`e8c5feffff`、`bd0x000000`、`e8ebfeffff`、`ebea`这 5 条机器指令

![](https://bbs.kanxue.com/upload/attach/201809/815036_HNE4YDK5HCWB2S7.png)

- 如果`who`进程执行到某条指令（比如`e8c5feffff`）时暂停了，并且在恢复执行之前，`0x400586`处的指令块被替换成打印"it's me ~"的代码，那么`who`进程下次执行，自然就会打印"it's me ~"

![](https://bbs.kanxue.com/upload/attach/201809/815036_RKRPAK7FY3Q9MBY.png)

## 2. 怎么获取打印"it's me ~"的机器码？

- 32 位系统，编译如下代码并在反汇编结果中提取：

```c
// isme32.c

int main()

{

__asm__(

	"jmp forward\n\t"

	"backward:popl %esi\n\t"

	"movl $4, %eax\n\t"

	"movl $2, %ebx\n\t"

	"movl %esi, %ecx\n\t"

	"movl $12, %edx\n\t"

	"int $0x80\n\t"

	"int3\n\t"

	"forward:call backward\n\t"

	".string \"it's me ~\\n\""

	);



	return 0;

}
```

- 64 位系统，编译如下代码并在反汇编结果中提取：

```c
// isme64.c

int main()

{

__asm__(

	"jmp forward\n\t"

	"backward:popq %rsi\n\t"

	"movq $1, %rax\n\t"

	"movq $2, %rdi\n\t"

	"movq $12, %rdx\n\t"

	"syscall\n\t"

	"int3\n\t"

	"forward:call backward\n\t"

	".string \"it's me ~\\n\""

	);



	return 0;

}
```

- 本篇文章的实验环境是 64 位 ubuntu 系统，所以执行`gcc isme64.c -oisme64 -g -Wall`生成`isme64`可执行文件，并通过反汇编`isme64`文件提取机器码

![](https://bbs.kanxue.com/upload/attach/201809/815036_6XFEAQTXGEGY2HR.png)

用蓝色、绿色标记出来的 16 进制内容，即为打印"it's me ~"的机器码，但有 2 点需要说明：

a. 绿色部分其实是"it's me ~\n\0"这串字符的 ascii 码值，也被`objdump`解释成汇编指令了，这是反汇编工具很难避免的一个问题，用`gdb/disassemble`反汇编的结果也是一样的，因为它们选择的都是线性扫描算法，递归下降算法效果会好一些，但仍然不能保证得到精确的结果；

b. C 程序实现打印"it's me ~"，写一条`printf("it's me ~")`语句不就可以了么，为什么要写成这种"奇奇怪怪"的样子？因为如果用 C 语法来实现的话，"it's me ~"这个字符串会被编译器安排到`isme64`程序的`.data`段，就没办法随着指令块一起注入到目标进程中，而且这段代码中取"it's me ~"地址的技巧，对于汇编初学者是很有意思的，代码不长，建议在大脑里执行一遍，并找到其中的奥妙。

## 3. 怎么实现注入？

本篇文章仅仅用于学习目的，注入的场景和方法都比较简单，思路在文章第 1 节已经介绍过了，就是"暂停被注入进程 -> 替换即将执行的指令块 -> 通知被注入进程继续执行"，由于有些场合还需要保证注入操作的隐蔽性，所以以下程序还在注入指令块完整执行后，将被注入进程恢复到了注入前的状态：

（理解这段代码，至少需要学习`ptrace()`系统调用的作用，如果有内核基础，也可以更深入的学习一下`ptrace()`的内部原理，另外，`CODE`宏对应的内容，即为利用文章第 2 节描述的方法，提取的机器码）

```c
// inject.c

#include <stdio.h>

#include <stdlib.h>

#include <string.h>

#include <sys/ptrace.h>

#include <sys/types.h>

#include <sys/wait.h>

#include <sys/user.h>

#include <errno.h>





// 注入指令块(打印"it's me ~")

#ifdef ENV_I386

#define CODE \

	"\xeb\x15\x5e\xb8\x04\x00\x00\x00" \

	"\xbb\x02\x00\x00\x00\x89\xf1\xba" \

	"\x0c\x00\x00\x00\xcd\x80\xcc\xe8" \

	"\xe6\xff\xff\xff\x69\x74\x27\x73" \

	"\x20\x6d\x65\x20\x7e\x0a\x00"

#define REG_IP  regs.eip

#else   // X_64

#define CODE \

	"\xeb\x19\x5e\x48\xc7\xc0\x01\x00" \

	"\x00\x00\x48\xc7\xc7\x02\x00\x00" \

	"\x00\x48\xc7\xc2\x0c\x00\x00\x00" \

	"\x0f\x05\xcc\xe8\xe2\xff\xff\xff" \

	"\x69\x74\x27\x73\x20\x6d\x65\x20" \

	"\x7e\x0a\x00"

#define REG_IP  regs.rip

#endif



#define CODE_SIZE (sizeof(CODE)-1)





/* 往pid进程的addr地址处写数据 */

void putdata(pid_t pid, unsigned long addr, void *vptr, int len)

{

	int count = 0;

	long word;



	while (count < len)

	{

		memcpy(&word, vptr+count, sizeof(word));

		word = ptrace(PTRACE_POKEDATA, pid, addr+count, word);

		count += sizeof(word);



		if (errno != 0)

			printf("putdata failed: %p\n", (void *)(addr+count));

	}

}



/* 读pid进程addr地址处的数据 */

void getdata(pid_t pid, unsigned long addr, void *vptr, int len)

{

	int i = 0, count = 0;

	long word;

	unsigned long *ptr = (unsigned long*)vptr;



	while (count < len)

	{

		word = ptrace(PTRACE_PEEKDATA, pid, addr+count, NULL);

		count += sizeof(word);

		ptr[i++]  = word;



		if (errno != 0)

			printf("getdata failed: %p\n", (void *)(addr+count));

	}

}





int main(int argc, char *argv[])

{

	pid_t pid;

	struct user_regs_struct regs;

	char backup[CODE_SIZE+1];



	if (argc != 2) {

		printf("Usage: %s {pid}\n", argv[0]);

		return -1;

	}



	// 通过启动参数，获取被注入进程号，并attach

	pid = atoi(argv[1]);

	ptrace(PTRACE_ATTACH, pid, NULL, NULL);  // SIGTRAP

	if (errno != 0)

		printf("attach failed: %s\n", strerror(errno));

	wait(NULL);  // 收到回复信号，保证后续过程在attach完成的情况下执行



	// 获取被注入进程当前寄存器值

	ptrace(PTRACE_GETREGS, pid, NULL, &regs);

	// 备份*ip寄存器指向的指令块

	getdata(pid, REG_IP, backup, CODE_SIZE);

	// 替换为CODE指令块

	putdata(pid, REG_IP, CODE, CODE_SIZE);

	// 恢复被注入进程的执行

	ptrace(PTRACE_CONT, pid, NULL, NULL);    // SIGCONT

	// 注入程序最后一条指令为int3，此为即为等待注入指令块执行完毕

	wait(NULL);



	// 等待用户输入，方便查看结果

	printf("continue to execute the orginal process, press any key ..\n");

	getchar();



	// 将修改的指令块恢复为备份内容

	putdata(pid, REG_IP, backup, CODE_SIZE);

	// 恢复被注入进程寄存器值

	ptrace(PTRACE_SETREGS, pid, NULL, &regs);

	

	// detach,被注入进程恢复正常执行

	ptrace(PTRACE_DETACH, pid, NULL, NULL);



	return 0;

}
```

## 4. 视频演示

a. 执行`gcc inject.c -oinject -g -Wall`，编译生成注入程序

b. 查看`who`进程号 N，并执行`sudo inject N`，进行动态注入操作

c. 观察`who`程序执行窗口是否打印了"it's me ~"

d. 回到`inject`程序执行窗口，按任意键结束`inject`进程，并恢复`who`进程到注入前的状态，继续不停的打印"who are you ?"

视频链接：[ff6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3i4K6u0W2P5h3!0#2K9%4g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3i4K6g2X3M7$3S2G2N6#2)9J5c8X3W2V1i4K6g2X3h3p5#2\*7k6%4A6y4g2p5@1I4e0W2c8g2x3f1&6Y4i4K6y4p5i4K6y4p5i4K6u0W2K9s2c8E0L8q4)9K6c8Y4y4H3L8g2)9K6c8r3p5J5K9o6m8C8i4K6u0W2x3e0p5@1x3e0M7K6y4o6u0Q4x3X3g2K6L8%4u0W2M7%4g2D9N6s2y4Q4x3X3g2V1M7r3!0K6N6r3g2J5](https://bbs.kanxue.com/elink@21cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3i4K6u0W2P5h3!0#2K9%4g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3i4K6g2X3M7$3S2G2N6#2)9J5c8X3W2V1i4K6g2X3h3p5#2*7k6%4A6y4g2p5@1I4e0W2c8g2x3f1&6Y4i4K6y4p5i4K6y4p5i4K6u0W2K9s2c8E0L8q4)9K6c8Y4y4H3L8g2)9K6c8r3p5J5K9o6m8C8i4K6u0W2x3e0p5@1x3e0M7K6y4o6u0Q4x3X3g2K6L8%4u0W2M7%4g2D9N6s2y4Q4x3X3g2V1M7r3!0K6N6r3g2J5)

## 5. 参考

ptrace 实现动态注入：[713K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3h3&6Y4x3q4)9J5c8Y4m8Q4x3V1j5&6y4e0R3#2x3K6f1$3i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@e60K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3h3&6Y4x3q4)9J5c8Y4m8Q4x3V1j5&6y4e0R3#2x3K6f1$3i4K6u0W2K9s2c8E0L8l9`.`.)

gdb 的工作原理：[204K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1$3y4e0R3K6y4o6k6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1I4x3e0f1&6z5e0M7I4](https://bbs.kanxue.com/elink@898K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1$3y4e0R3K6y4o6k6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1I4x3e0f1&6z5e0M7I4)

转载请注明出处：*https://bbs.pediy.com/thread-246948.htm*

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-9-20 19:47

被 `jmpcall` 编辑，原因：
