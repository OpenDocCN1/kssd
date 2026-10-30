
```
==98287==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x631000039680 at pc 0x562394639b54 bp 0x7ffee24654e0 sp 0x7ffee24654d0

READ of size 2 at 0x631000039680 thread T0

    #0 0x562394639b53 in ok_jpg_decode_block_subsequent_scan /root/study/ok-file-formats/afl-test/ok_jpg.c:1102

    #1 0x56239463b11f in ok_jpg_decode_scan /root/study/ok-file-formats/afl-test/ok_jpg.c:1238

    #2 0x56239463fc60 in ok_jpg_read_sos /root/study/ok-file-formats/afl-test/ok_jpg.c:1734

    #3 0x562394640d3c in ok_jpg_decode2 /root/study/ok-file-formats/afl-test/ok_jpg.c:1900

    #4 0x562394641605 in ok_jpg_decode /root/study/ok-file-formats/afl-test/ok_jpg.c:1990

    #5 0x5623946308a4 in ok_jpg_read_with_allocator /root/study/ok-file-formats/afl-test/ok_jpg.c:268

    #6 0x56239463071b in ok_jpg_read /root/study/ok-file-formats/afl-test/ok_jpg.c:257

    #7 0x56239462fd5e in main /root/study/ok-file-formats/afl-test/main.c:8

    #8 0x7fe63f9a4b96 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21b96)

    #9 0x56239462fb29 in _start (/root/study/ok-file-formats/afl-test/Asanjpg/Asanjpg+0x2b29)



0x631000039680 is located 113 bytes to the right of 69135-byte region [0x631000028800,0x63100003960f)

allocated by thread T0 here:

    #0 0x7fe63fe52b40 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb40)

    #1 0x56239462ff00 in ok_stdlib_alloc /root/study/ok-file-formats/afl-test/ok_jpg.c:55

    #2 0x56239463eb20 in ok_jpg_read_sof /root/study/ok-file-formats/afl-test/ok_jpg.c:1595

    #3 0x562394640ac2 in ok_jpg_decode2 /root/study/ok-file-formats/afl-test/ok_jpg.c:1884

    #4 0x562394641605 in ok_jpg_decode /root/study/ok-file-formats/afl-test/ok_jpg.c:1990

    #5 0x5623946308a4 in ok_jpg_read_with_allocator /root/study/ok-file-formats/afl-test/ok_jpg.c:268

    #6 0x56239463071b in ok_jpg_read /root/study/ok-file-formats/afl-test/ok_jpg.c:257

    #7 0x56239462fd5e in main /root/study/ok-file-formats/afl-test/main.c:8

    #8 0x7fe63f9a4b96 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x21b96)



SUMMARY: AddressSanitizer: heap-buffer-overflow /root/study/ok-file-formats/afl-test/ok_jpg.c:1102 in ok_jpg_decode_block_subsequent_scan

Shadow bytes around the buggy address:

  0x0c627ffff280: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

  0x0c627ffff290: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

  0x0c627ffff2a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

  0x0c627ffff2b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

  0x0c627ffff2c0: 00 07 fa fa fa fa fa fa fa fa fa fa fa fa fa fa

=>0x0c627ffff2d0:[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

  0x0c627ffff2e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

  0x0c627ffff2f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

  0x0c627ffff300: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

  0x0c627ffff310: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

  0x0c627ffff320: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa

Shadow byte legend (one shadow byte represents 8 application bytes):

  Addressable:           00

  Partially addressable: 01 02 03 04 05 06 07

  Heap left redzone:       fa

  Freed heap region:       fd

  Stack left redzone:      f1

  Stack mid redzone:       f2

  Stack right redzone:     f3

  Stack after return:      f5

  Stack use after scope:   f8

  Global redzone:          f9

  Global init order:       f6

  Poisoned by user:        f7

  Container overflow:      fc

  Array cookie:            ac

  Intra object redzone:    bb

  ASan internal:           fe

  Left alloca redzone:     ca

  Right alloca redzone:    cb

==98287==ABORTING
```

第一、可以从SUMMARY看出这个crash是不是有价值，有时候报一些莫名其妙的没有价值的错还需要用`gdb`进一步调试，当然我一般懒得调试（好吧其实是我技术不到位分析不出啥东西来）；

第二、在第一段中给出了触发位置的详细调用过程，如果想写exp，这一块比较重要，我现在不是很了解这部分，不敢说太多；

第三、并不是所有的crash都可以报这种错误，有时候就是仅仅是文件类型不匹配等等问题，这时候crash其实就没什么用；

第四、**重复crash**，这里着重说一下，我现在在做的一个小工具（正在验证中），请看截图：

![crashes](https://bbs.kanxue.com/upload/attach/202008/873988_C4DPSR52XUY8B6Z.png)

可以看到我开了20个线程，最后生成了2262个crash，经过前十几个crash文件的验证，发现重复性特别高，虽然crash文件内容不一样，但是触发的漏洞位置一模一样，这种crash其实并不unique，而且很多crash并不能触发真正的漏洞，是假crash，所以最近在做这样一个筛选出真正独一无二crash的工具。

### 6.3 漏洞提交issue

当得到6.2中的Asan报告之后，就可以认为触发了堆溢出漏洞，可以在GitHub上给作者提交issue了，最后贴一下我提交issue的大概内容（加注释）

[issue地址：bdaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4U0K9$3g2W2L8W2)9J5c8X3!0C8i4K6u0V1k6X3W2D9k6g2)9J5k6r3k6G2M7X3#2S2N6s2y4Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1j5%4](https://bbs.kanxue.com/elink@e98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4U0K9$3g2W2L8W2)9J5c8X3!0C8i4K6u0V1k6X3W2D9k6g2)9J5k6r3k6G2M7X3#2S2N6s2y4Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1j5%4)

> # Describe 【漏洞描述】
>
> A heap-buffer-overflow was discovered in `ok_file_formats`. The issue is being triggered in function `ok_jpg_decode_block_subsequent_scan()` at `ok_jpg.c:1102`
> **这部分放漏洞的总述，把漏洞类型、漏洞触发函数位置、漏洞所在文件列出来；**
>
> # Reproduce 【复现过程所需内容】
>
> **test program**
>
> ```
> #include <stdio.h>
>
> int main(int _argc, char **_argv) {}
> ```
>
> **我一般是把验证程序放在这里，好让作者可以自己reproduce-复现；**
> Tested in Ubuntu 18.04, 64bit.
> Compile test program with address sanitizer with this command:
>
> ```
> gcc -g -fsanitize=address -fno-omit-frame-pointer -O1 -o Asanjpg main.c ok_jpg.c  ok_jpg.h
> ```
>
> **告诉作者使用的，以及编译的环境和方法；**
> You can get program [here](https://bbs.kanxue.com/elink@db2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4j5i4W2F1k6f1c8W2N6V1#2S2P5X3g2Q4x3V1k6S2k6X3I4Q4y4h3k6@1k6i4y4@1i4K6g2X3N6$3!0J5K9%4y4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3e0@1E0X3K9h3I4W2k6X3!0J5L8h3q4@1i4K6u0r3b7i4y4S2L8X3A6H3k6#2)9J5c8V1q4K6j5h3&6B7M7r3M7`.).
> **为了方便作者验证漏洞，我也会把编译好的程序放出来**
>
> # ASan Reports 【验证程序报的内存报告】
>
> ```
> ./Asanjpg crash/jpg-heap-buffer-overflow-1
> ```
>
> Get ASan reports
>
> ```
> ==98287==ABORTING
> ```
>
> **这里就放`Asanjpg`验证之后报的内容，就是6.2里报的内存错误跟踪信息**
>
> # Poc 【其实就是crash文件】
>
> Poc file is [here](https://bbs.kanxue.com/elink@8b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4j5i4W2F1k6f1c8W2N6V1#2S2P5X3g2Q4x3V1k6S2k6X3I4Q4y4h3k6@1k6i4y4@1i4K6g2X3N6$3!0J5K9%4y4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3e0@1E0X3K9h3I4W2k6X3!0J5L8h3q4@1i4K6u0r3b7i4y4S2L8X3A6H3k6#2)9J5c8X3y4J5j5i4y4Z5i4K6u0r3K9Y4m8Y4i4K6u0V1K9r3g2S2M7q4)9J5k6r3u0#2k6X3k6W2M7W2)9J5k6r3!0$3k6i4u0X3L8r3!0%4i4K6u0V1x3b7`.).).

当然如果Fuzz的程序比较知名，可以尝试申请CVE，我这个尝试申请了，但是没有回信，应该是凉了。

## 结语

总体来说，整个Fuzz过程熟悉一遍之后就变得不那么复杂了，还记得刚过年的时候我还是一头雾水，现在对小型库的fuzz已经比较有信心了，希望大家看完本文能有所帮助。

[GitHub-AFL源码阅读笔记](https://bbs.kanxue.com/elink@bf5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4j5i4W2F1k6f1c8W2N6V1#2S2P5X3g2Q4x3V1k6o6K9r3W2F1k6i4y4W2i4K6g2X3L8X3!0@1k6h3c8Q4y4h3k6m8c8V1H3`.)文中说的想要做的工具，完成后应该会放在这，欢迎star；

[Fuzz学习过程随笔](https://bbs.kanxue.com/elink@2a0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4&6L8X3g2Q4x3X3c8@1j5h3!0Q4x3V1k6@1j5h3N6Q4x3V1k6m8c8V1I4Q4c8e0g2Q4b7f1c8Q4b7e0k6Q4c8e0c8Q4b7U0W2Q4b7e0m8Q4x3V1j5`.)记录自己在学习过程中遇到的坑和一些心得；

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-8-5 13:25 被张公子T40编辑 ，原因：
