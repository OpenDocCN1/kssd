
## 阶段实验

做个实验回顾一下之前的页机制，以达到熟悉页机制的目的。

实验1：编写驱动读取某个进程的某个虚拟地址内容并打印出物理地址，然后手工修改这个物理地址里的内容来验证找到的是否正确（驱动代码共享内存然后修改内存的方式后面再讲，现在只手工实验）。这里默认大家都学过`x32`的内核知识，不过会在代码注释中将页之外的动作加上对应的说明。

实验代码：

测试应用程序：

```
#include <Windows.h>
#include <stdio.h>

DWORD32 testVal = 111;

int main() {
    printf("pid: %d\r\n", GetCurrentProcessId());
    while (true) {
        system("pause");
        printf("addr: 0x%p\r\n", &testVal);
        printf("val: %d\r\n", testVal);
    }
    return 0;
}
```

根据`pid`和`addr`编写我们的驱动代码。

驱动：`x64Common.h`
