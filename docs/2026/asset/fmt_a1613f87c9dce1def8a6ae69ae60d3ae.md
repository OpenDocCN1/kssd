
```
08049570 l O .eh_frame 00000000 __EH_FRAME_BEGIN__
080484b4 l F .text 00000000 fini_dummy
0804960c l O .bss 00000018 object.11
080484bc l F .text 00000000 frame_dummy
080484e0 l F .text 00000000 init_dummy
08049570 l O .data 00000000 force_to_data
08049574 l O .ctors 00000000 __CTOR_LIST__
00000000 l df *ABS* 00000000 crtstuff.c
08048520 l .text 00000000 gcc2_compiled.
08048520 l F .text 00000000 __do_global_ctors_aux
08049578 l O .ctors 00000000 __CTOR_END__
08048548 l F .text 00000000 init_dummy
08049570 l O .data 00000000 force_to_data
08049580 l O .dtors 00000000 __DTOR_END__
08049570 l O .eh_frame 00000000 __FRAME_END__
00000000 l df *ABS* 00000000 p10.c
080483ac F *UND* 00000031 printf
0804959c g O *ABS* 00000000 _DYNAMIC
08048550 g O *ABS* 00000000 _etext
08048390 g F .init 00000000 _init
08049624 g O .bss 00000004 environ
00000000 w *UND* 00000000 __deregister_frame_info
08049630 g O *ABS* 00000000 end
08049628 g O .bss 00000004 xx
08049564 g O .data 00000004 __progname
080483dc g F .text 00000083 _start
0804960c g O *ABS* 00000000 __bss_start
080484e8 g F .text 00000038 main
08048550 g F .fini 00000000 _fini
0804962c g O .bss 00000004 yy
080483bc F *UND* 00000070 atexit
0804960c g O *ABS* 00000000 _edata
08049584 g O *ABS* 00000000 _GLOBAL_OFFSET_TABLE_
08049630 g O *ABS* 00000000 _end
080483cc F *UND* 0000005b exit
00000000 w *UND* 00000000 __register_frame_info
```

## 动态符号表

```
DYNAMIC SYMBOL TABLE:
080483ac DF *UND* 00000031 printf
0804959c g DO *ABS* 00000000 _DYNAMIC
08048550 g DO *ABS* 00000000 _etext
08048390 g DF .init 00000000 _init
08049624 g DO .bss 00000004 environ
00000000 w D *UND* 00000000 __deregister_frame_info
08049630 g DO *ABS* 00000000 end
08049564 g DO .data 00000004 __progname
0804960c g DO *ABS* 00000000 __bss_start
08048550 g DF .fini 00000000 _fini
080483bc DF *UND* 00000070 atexit
0804960c g DO *ABS* 00000000 _edata
08049584 g DO *ABS* 00000000 _GLOBAL_OFFSET_TABLE_
08049630 g DO *ABS* 00000000 _end
080483cc DF *UND* 0000005b exit
00000000 w D *UND* 00000000 __register_frame_info
```

## Debugging Information

```
int main ()
{ /* 0x80484e8 */
} /* 0x80484e8 */

int main ()
{ /* 0x80484e8 */
/* file /usr/home/shieyuan/test/p10.c line 3 addr 0x80484ee */
/* file /usr/home/shieyuan/test/p10.c line 5 addr 0x80484ee */
/* file /usr/home/shieyuan/test/p10.c line 6 addr 0x80484f8 */
/* file /usr/home/shieyuan/test/p10.c line 7 addr 0x8048502 */
/* file /usr/home/shieyuan/test/p10.c line 8 addr 0x804851e */
/* file /usr/home/shieyuan/test/p10.c line 8 addr 0x804851e */
} /* 0x8048520 */

int xx /* 0x8049628 */;
int yy /* 0x804962c */;
```

## 动态重定位表

```
DYNAMIC RELOCATION RECORDS
OFFSET TYPE VALUE
08049590 R_386_JUMP_SLOT printf
08049594 R_386_JUMP_SLOT atexit
08049598 R_386_JUMP_SLOT exit
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [elf.doc](javascript:void(0);)（112.00kb，100次下载）
