
# 文件斧头 1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum997.htm`](https://www.pediy.com/kssd/pediy03/forum997.htm)

标题: 我来贴他的破解。国产软件怎么啦？破就破呗，没什么大不了的。我就不信他的 `win98` 或者是 `VC++`使用正版的。哼，五十步笑百步而已！

内容:

我来贴他的破解。国产软件怎么啦？破就破呗，没什么大不了的。我就不信他的 `win98` 或者是 `VC++`使用正版的。哼，五十步笑百步而已！

## 破解手记之四

作者：`Crack007`

主页：`http://www.crack007.com` ( ^-^ )

破解日期：2001-5-18

破解工具：`TRW2000` `1.22` `W32dASM` `8.93` `Procdump` `v1.62`

难度：`☆`

软件下载地址：`ftp://211.100.8.153/download/99/10261_axefb.zip`

### 软件简介

这个软件可以方便快捷的将一个大文件分割成若干份小文件。分割的文件大小可以任意指定（以 `KB` 为单位，大于 `0` `kb`）。还可以指定生成“还原批处理文件”，可以直接执行这个 `reset.bat` 文件,将分割的文件自动还原。当然，您也可以通过“文件斧头”还原。

先用 `Fi.exe` 侦测出该程序是用 `Aspack1.07b` 加的壳，所以先用 `Procdump` 进行脱壳。

然后启动 `TRW2000`，随便输入注册信息：

`用户名`:`Crack007`

`注册码`:`78787878`

下 `Bpx` `hmemcpy` ,被拦截后用 `Pmodule` 来到程序核心

不停地按 `F10`，来到此处：

* `Referenced` by a `(U)nconditional` or `(C)onditional` `Jump` at `Address`:

`|` `:00466A8F(C)`

`|`

`|` `:00466ACF` `837DF800` `cmp` `dword` `ptr` `\[ebp\-08\]`, `00000000`

`|` `:00466AD3` `0F846F020000` `je` `00466D48`

`|` `:00466AD9` `8B45F8` `mov` `eax`, `dword` `ptr` `\[ebp\-08\]` `<----` `过此处下` `d` `edx` `可显示正确注册码`

`|` `:00466ADC` `8B55F0` `mov` `edx`, `dword` `ptr` `\[ebp\-10\]` `<----` `显示输入的注册码：`

`|` `:00466ADF` `E8A0D2F9FF` `call` `00403D84`

`|` `:00466AE4` `0F855E020000` `jne` `00466D48` `<---` `经过` `W32dasm` `反汇编后可以看到此处将跳转到` `GameOver` `处。`

`|` `:00466AEA` `8D9504FEFFFF` `lea` `edx`, `dword` `ptr` `\[ebp+FFFFFE04\]`

`|` `:00466AF0` `33C0` `xor` `eax`, `eax`

`|` `:00466AF2` `E8CDBDF9FF` `call` `004028C4`

`|` `:00466AF7` `8B8504FEFFFF` `mov` `eax`, `dword` `ptr` `\[ebp+FFFFFE04\]`

`|` `:00466AFD` `8D55EC` `lea` `edx`, `dword` `ptr` `\[ebp\-14\]`

`|` `:00466B00` `E8AB16FAFF` `call` `004081B0`

`|` `:00466B05` `8D8500FEFFFF` `lea` `eax`, `dword` `ptr` `\[ebp+FFFFFE00\]`

`|`

* `Possible` `StringData` `Ref` `from` `Code` `Obj` `->"错误的序列号！"`

`|` `:00466D48` `B8E86D4600` `mov` `eax`, `00466DE8`

`|` `:00466D4D` `E80255FEFF` `call` `0044C254` `<----` `呼叫出错误注册码的对话框`

整理一下：

`Name`: `Crack007`

`Code`: `12385513832171413`

### 文件斧头 1.0 注册机,简单

`http://zangqi.home.chinaren.com`

```
==============CUT Begin==============

#include "stdio.h"

main()

{

char string[81];

int d=0,c;

int i,e;

clrscr();

printf("******File-Axe1.0 文件斧头******\n 风飘雪[CCG]制作\n");

printf("http://fpxfpx.yeah.net\nPlease input your name:");

gets(string);

d=strlen(string);

printf("Your name is           ");

puts(string);

printf("Your Register code is ");

for(i=0;(c=string[i])!='\0';i++)

{e=(7*d+c)/(i+1);

printf("%d",e);

}

return(0);

}

============CUT End===================
```

```c
#include "stdio.h"

main()

{

unsigned char string[80];

int i,c,d,e;

printf(" ******FILE-AXE******\nThis keygen is maked by Pretender\nPlease input your name :   ");

gets(string);

d=strlen(string)*7;

  printf("Your Register code is  :  ");

for(i=0;(c=string[i])!='\0';i++)

{e=c+d;

e/=(1+i);

printf("%d",e);

  }

printf("\n ***************   ---     ---     ---\n *Welcome to WWW.CRACKNOW.COM*      /      /      / --\n ***************   ---     ---     --/\n");}
```
`4dd466d0e1e0cc123afbe504da2462ca`
