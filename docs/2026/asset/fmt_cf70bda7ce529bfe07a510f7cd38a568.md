
# 安装 python2 以及 pwntools

参考：[88dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0j5J5y4U0t1I4x3o6p5#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3K6l9I4y4e0j5K6x3e0t1`.](https://bbs.kanxue.com/elink@623K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0j5J5y4U0t1I4x3o6p5#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3K6l9I4y4e0j5K6x3e0t1`.)

因为好多好工具都是基于 python2 开发的，使用 python3 会有很多问题。

## 安装 python2

`apt-get install python2`

## 安装 python2 的 pip

```
apt install python-dev
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2 get-pip.py
```

## 安装 pwntools

```
apt-get install libssl-dev libffi-dev build-essential
python2 -m pip install --upgrade pwntools
```

以上如果有问题可以参考：[886K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0j5J5y4U0t1I4x3o6p5#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3K6l9I4y4e0j5K6x3e0t1`.](https://bbs.kanxue.com/elink@5e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0j5J5y4U0t1I4x3o6p5#2i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3K6l9I4y4e0j5K6x3e0t1`.)

下载 `pattern.py`

```
6feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0N6X3g2F1K9i4c8G2i4K6u0r3k6i4S2H3L8r3!0A6N6q4)9J5k6s2m8S2N6s2c8W2M7X3^5`.
```

# 0x02 简单的缓冲区溢出

## 编写有漏洞的 C 代码

测试代码来自郑敏老师的 C 代码：

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void vulnerable_function() {
    char buf[128];
    read(STDIN_FILENO, buf, 256);
}

int main(int argc, char** argv) {
    vulnerable_function();
    write(STDOUT_FILENO, "Hello, World\n", 13);
}
```

保存代码到 `level1.c` 文件。

## 编译代码

对于初学者来说，这里将关闭地址随机化（ASLR）以及 canary 和 NX。

### 关闭地址随机化

```
sudo -s
echo 0 > /proc/sys/kernel/randomize_va_space
exit
```

还可以使用：

`sysctl -w kernel.randomize_va_space=0`

默认情况下 `randomize_va_space` 中的值为 2：

`cat /proc/sys/kernel/randomize_va_space`

输出：`2`

关闭地址随机化后的输出为：`0`

### 关闭 canary 和 NX

栈帧内的栈保护用到的 canary 和 NX 是 GCC 编译器加入的完整栈检测代码和栈属性设置。这里用到 GCC 的一些选项：

```c
// 禁用 canary 保护
-fno-stack-protector

// 栈可执行
-z execstack
```

具体 GCC 命令为：

`gcc -m32 -fno-stack-protector -z execstack -o level1 level1.c`

# 0x03 缓冲区溢出和任意代码执行

## 分析

从代码可以看到 `vulnerable_function` 函数有明显的缺陷，如下图编译器已经发出了警告！当向 `buf` 读入数据时，读入的数据明显比 `buf` 能存储的 128 字节多，这就会造成缓冲区溢出。当栈上的返回地址被淹没（控制）时，函数返回时就会被控制去执行二进制代码。

```
└─# gcc -m32 -fno-stack-protector -o level1 level1.c
level1.c: In function ‘vulnerable_function’:
level1.c:9:5: warning: ‘read’ writing 256 bytes into a region of size 128 overflows the destination [-Wstringop-overflow=]
    9 | read(STDIN_FILENO, buf, 256);
      |     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~
level1.c:7:10: note: destination object ‘buf’ of size 128
    7 |     char buf[128];
      |          ^~~
In file included from level1.c:3:
/usr/include/unistd.h:371:16: note: in a call to function ‘read’ declared with attribute ‘access (write_only, 2, 3)’
  371 | extern ssize_t read (int __fd, void *__buf, size_t __nbytes) __wur
```

## 成因

缓冲区漏洞的成因，和最早 PC 框架设计有关。

首先就是函数 `call`，当执行 `vulnerable_function` 时的汇编代码：

```
0x5655621e <main+20>:  add    ebx,0x2db2
=> 0x56556224 <main+26>:   call   0x565561bd <vulnerable_function>
0x56556229 <main+31>:  sub    esp,0x4
```

`call` 会把地址 `0x56556229` 压入栈中（用来返回后继续执行）。然后执行 `vulnerable_function` 函数：

```
0x56556218 <main+14>:  push   ecx
0x56556219 <main+15>:  call   0x565560c0 <__x86.get_pc_thunk.bx>
0x5655621e <main+20>:  add    ebx,0x2db2
=> 0x56556224 <main+26>:   call   0x565561bd <vulnerable_function>
0x56556229 <main+31>:  sub    esp,0x4
0x5655622c <main+34>:  push   0xd
0x5655622e <main+36>:  lea    eax,[ebx-0x1fc4]
0x56556234 <main+42>:  push   eax
No argument
[------------------------------------stack-------------------------------------]
0000| 0xffffd3d0 --> 0xffffd3f0 --> 0x1
0004| 0xffffd3d4 --> 0xf7e2a000 --> 0x229dac
0008| 0xffffd3d8 --> 0xf7ffd020 --> 0xf7ffda40 --> 0x56555000 --> 0x464c457f
0012| 0xffffd3dc --> 0xf7c21519 (<__libc_start_call_main+121>: add    esp,0x10)
0016| 0xffffd3e0 --> 0xffffd624 ("/home/dbg/Desktop/rop/level1/level1")
0020| 0xffffd3e4 --> 0x70 ('p')
0024| 0xffffd3e8 --> 0xf7ffd000 --> 0x36f2c
0028| 0xffffd3ec --> 0xf7c21519 (<__libc_start_call_main+121>: add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x56556224 in main ()
gdb-peda$
```

执行进入 `vulnerable_function`：

```
0x565561b4 <frame_dummy+4>:    jmp    0x56556110 <register_tm_clones>
0x565561b9 <__x86.get_pc_thunk.dx>:    mov    edx,DWORD PTR [esp]
0x565561bc <__x86.get_pc_thunk.dx+3>:  ret
=> 0x565561bd <vulnerable_function>:   push   ebp
0x565561be <vulnerable_function+1>:    mov    ebp,esp
0x565561c0 <vulnerable_function+3>:    push   ebx
0x565561c1 <vulnerable_function+4>:    sub    esp,0x84
0x565561c7 <vulnerable_function+10>:   call   0x565560c0 <__x86.get_pc_thunk.bx>
[------------------------------------stack-------------------------------------]
0000| 0xffffd3cc (")bUV\360\323\377\377")
0004| 0xffffd3d0 --> 0xffffd3f0 --> 0x1
0008| 0xffffd3d4 --> 0xf7e2a000 --> 0x229dac
0012| 0xffffd3d8 --> 0xf7ffd020 --> 0xf7ffda40 --> 0x56555000 --> 0x464c457f
0016| 0xffffd3dc --> 0xf7c21519 (<__libc_start_call_main+121>: add    esp,0x10)
0020| 0xffffd3e0 --> 0xffffd624 ("/home/dbg/Desktop/rop/level1/level1")
0024| 0xffffd3e4 --> 0x70 ('p')
0028| 0xffffd3e8 --> 0xf7ffd000 --> 0x36f2c
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x565561bd in vulnerable_function ()
gdb-peda$
```

其中 `0xffffd3cc` 就存放了返回地址：

`0000| 0xffffd3cc (")bUV\360\323\377\377")`

再来看看 `vulnerable_function` 返回时的栈：

```
[-------------------------------------code-------------------------------------]
0x56556204 <vulnerable_function+71>:   nop
0x56556205 <vulnerable_function+72>:   mov    ebx,DWORD PTR [ebp-0x4]
0x56556208 <vulnerable_function+75>:   leave
=> 0x56556209 <vulnerable_function+76>:    ret
0x5655620a <main>: lea    ecx,[esp+0x4]
0x5655620e <main+4>:   and    esp,0xfffffff0
0x56556211 <main+7>:   push   DWORD PTR [ecx-0x4]
0x56556214 <main+10>:  push   ebp
[------------------------------------stack-------------------------------------]
0000| 0xffffd3cc (")bUV\360\323\377\377")
0004| 0xffffd3d0 --> 0xffffd3f0 --> 0x1
0008| 0xffffd3d4 --> 0xf7e2a000 --> 0x229dac
0012| 0xffffd3d8 --> 0xf7ffd020 --> 0xf7ffda40 --> 0x56555000 --> 0x464c457f
0016| 0xffffd3dc --> 0xf7c21519 (<__libc_start_call_main+121>: add    esp,0x10)
0020| 0xffffd3e0 --> 0xffffd624 ("/home/dbg/Desktop/rop/level1/level1")
0024| 0xffffd3e4 --> 0x70 ('p')
0028| 0xffffd3e8 --> 0xf7ffd000 --> 0x36f2c
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
0x56556209 in vulnerable_function ()
gdb-peda$
```

其中 `0xffffd3cc` 就存放了返回地址：

`0000| 0xffffd3cc (")bUV\360\323\377\377")`

通过命令行 `set *(0xffffd3cc) = 0xcccccccc` 修改 `ret` 返回地址为 `0xcccccccc`。
