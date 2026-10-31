
# [原创][修改版]c/c++语言实现堆栈修改,通过`ret`跳转到自定义函数,高手飘过

仅是个示例代码,今天上午写代码突然发现堆栈可以在高级语言中通过函数参数指针修改,
所以丢了这个东西出来.如有雷同,纯属巧合,也请你告诉我,让我学习;)

好处是,编译后没有`jmp`指令,通过`ret`跳转到需要的代码,另外在调试时,某些代码会被当作数据,可以增加调试难度.

缺点,需要调用函数的堆栈有至少4个字节的空间,否则堆栈返回出错.
但是这四个字节空间不会被摧毁.

可能我有些东西还没有照顾到,如果有错误,大家告诉我;)

编译环境: vc6 vc7

原来的代码在编译器优化的情况下无法跳回调用函数,改了下;)

```
#include <stdio.h>

int  somefunc( void *ptr)
{
    printf("in somefunc...\n");
    return 0;
}

void stackbuild( void *ptr)
{
    printf("in stackbuild...\n");
    *(unsigned int*)(&ptr-1) ^= *(unsigned int*) &ptr;
    *(unsigned int*)&ptr ^= *(unsigned int*) (&ptr-1);	//注意此处对堆栈操作
    *(unsigned int*)(&ptr-1) ^= *(unsigned int*) &ptr;
}

int main(int argc, char *argv[])
{
    // 还是嵌入了一句汇编，平衡堆栈;)哪位高人改改，看能不能把嵌入汇编去掉
    __asm{push 0} //预留4字节空间,平衡堆栈，注意此句和下面的句子要一起用，没有下面的调用，必须没有该语句
    stackbuild(somefunc);

    printf("exit main...\n");
    return 0;
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
