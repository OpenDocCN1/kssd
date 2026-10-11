
# MiniPad 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum587.htm`](https://www.pediy.com/kssd/pediy03/forum587.htm)

`TRW2000` `Ver:1.23` 破解 `MiniPad 2000`

`MiniPad 2000` 下载： `ftp://ftp2.download.com.cn/pub/new\_tools/mp160c.exe&name=mp160c.exe`

1. 运行`TRW2000`

2. 运行 `MiniPad 2000`

   来到程序帮助中的注册窗口

   随便输入注册名：你的名字 注册码注册码: `787878`

3. 按`CTRL+M`激活`TRW2000`输入如下命令:

   ```
   BPX HMEMCPY 设置中断条件
   F5 返回 MiniPad 2000
   ```

4. 按`完成注册`后，程序被拦截。清除断点：`BC *`

5. 按`F12` 键`X`次（当然是弹出失败窗的前一次了），开始烦人动作：按`F10` 跟踪一直到：

   ```
   :
   :

   016F:00479A71  MOV  EDX ,[EBP-14]

   016F:00479A74  MOV  EAX ,[EBP-04]    (用 D ESI 可以看到你输入的注册码）

   016F:00479A77  CALL 00403FB8         (用 D ECX 可以看到你的正确注册码）

   016F:00479A7C  JZ   00479A9C          不跳就死！！！
   ```

6. 用笔记下吧！

7. 用刚才记下来的注册码来注册一下试试吧。

作者:`Lihaitao`

信箱:`lihaitao@pub.xaonline.com`

```c
#include "stdio.h"

main()

{

unsigned char string[81];

unsigned long i,a=0,b=0;

  unsigned c;

printf("This keygen is maked by Pretender\nPlease input your name    ");

gets(string);

for(i=0;(c=string[i])!='\0';i++);

printf("Your name is ");

puts(string);

if(string[0]>127) string[0]+=256;

a=i+17+string[0];

if(a>=200) a-=200;

if(a>=100) a-=100;

printf("Your Register code is:      %d",a);

for(b=1;b<i;b++)

{a+=string[b];

  if(a>=200) a-=200;

  if(a>=100) a-=100;

  printf("%d",a);}

printf("\n");

printf("Welcome to WWW.CRACKNOW.COM\n");}
```

`4dd466d0e1e0cc123afbe504da2462ca`
