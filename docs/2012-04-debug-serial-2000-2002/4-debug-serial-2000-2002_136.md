
# Wintility Lite

> 原文：[`www.pediy.com/kssd/pediy02/forum139.htm`](https://www.pediy.com/kssd/pediy02/forum139.htm)

`Wintility(R) Lite 2000 - version 5.3.00`

`Copyright (C) 1999-2000 PX Technologies, Inc.`

`http://www.wintility.com`

`All Rights Reserved`

*** Brief Description ***

`Wintility Lite` is an indispensable tool for any type of user who manages significant volumes of documents and needs to recover them promptly.

`Wintility` is a software utility to organize personnal documents and selected e-mail messages in your computer or network. With a simple and intuitive interface you will be able to browse through your documents such as texts, projects, worksheets, presentations, e-mails, images, and any other work that can be saved in your computer.

You will also be allowed to easily manage your documents without activating the program that originated them, viewing their contents and verifying if they are what you are looking for.

With `Wintility` you see only the documents and messages of your interest, controlling all of your work environment, avoiding the exhibition of files which you don't know the use. Each new document or message received or sent is automatically identified and catalogued.

`Wintility` creates a logical catalog, fully personalized, that doesn't interfere with the original structure of your files in the computer, allowing you to add additional features to the documents such as descriptions, passwords, expiration dates and versions.

These new features, together with the usual properties of the document, will allow you to easily identify and retrieve any document stored on your computer.

开始破解时,想找出软件的注册过程,但似乎运算过程比较复杂.

在跟踪过程中,找到:

```
          call 0043DBA0
          add esp, 00000008
          jz ********
```

若将 `jz` 改为 `nop`,则在注册时,输入任何注册码都可注册,但下次运行时,仍有提示注册的画面.

继续跟踪又发现:

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043DC84(C)

|

```
:0043DC62 8A10                 mov dl, byte ptr [eax]
:0043DC64 8A1E                 mov bl, byte ptr [esi]
:0043DC66 8ACA                 mov cl, dl
:0043DC68 3AD3                 cmp dl, bl
:0043DC6A 90                   nop<----***
:0043DC6B 90                   nop
:0043DC6C 84C9                 test cl, cl
:0043DC6E 7416                 je 0043DC86
:0043DC70 8A5001               mov dl, byte ptr [eax+01]
:0043DC73 8A5E01               mov bl, byte ptr [esi+01]
:0043DC76 8ACA                 mov cl, dl
:0043DC78 3AD3                 cmp dl, bl
:0043DC7A 90                   nop<----***
:0043DC7B 90                   nop
:0043DC7C 83C002               add eax, 00000002
:0043DC7F 83C602               add esi, 00000002
:0043DC82 84C9                 test cl, cl
:0043DC84 75DC                 jne 0043DC62
```

原来***处为 `jnz` 指令，将其改为 `nop`，就不会再有提示注册的画面了．

注意：在用 `HEX` 编辑器修改程序时,会搜索到两处上面的代码,两处都要改.

用户名:`LiuTong`

注册码:`87654321`

跟踪过程中找到了一个码(`3879573111`),上面的程序就是比较 `7654` 和 `3111` 的.

另外软件将用户名和注册码,按下面的方法进行了运算:

`m*n=i*600937` 的 `1` 次方=`j`

`m`--用户名或注册码的 `HEX` 码

`n`--序号

将用户名(或注册码)每个字符的运算结果 `j` 累加

用户名运算后得到 `E73d9677`

注册码运算后得到 `308606D8`

然后我就跟不下去了

若有哪位找出了注册码的生成过程,烦请 `E-mail` 给我一份.

注册机

```c
#include <stdio.h>

#include <string.h>

#include <windows.h>

long GenerateNumber(char *s);

void main(void)

{

   char Name[64];

   long number;

   char Serial[9];

   char buf[16];

   printf("Input your name: ");

   gets(Name);

   number = GenerateNumber(Name);

   sprintf(buf, "%010lu", number);

   Serial[1] = buf[6];

   Serial[2] = buf[7];

   Serial[3] = buf[8];

   Serial[4] = buf[9];

   //Serial[0]、Serial[5]、Serial[6]是任意的，这里随机生成它们

   //Serial[0]不能为 0

   //并非真正的随机数

   number = GetTickCount( ) % 1000;

   Serial[0] = (number % 9) + '1';

   number /= 10;

   Serial[5] = (number % 10) + '0';

   number /= 10;

   Serial[6] = (number % 10) + '0';

   Serial[7] = '\0';

   number = GenerateNumber(Serial);

   sprintf(buf, "%010lu", number);

   Serial[7] = buf[9];

   Serial[8] = '\0';

   printf("Your registration code is: %s\n", Serial);

}

long GenerateNumber(char *s)

{

   long ECX;

   long sum;

   long k, length;

   length = strlen(s);

   if (length <= 0)  return 0;

   sum  = 0;

   ECX = 0x00600937;

   for(k = 0; k < length; k++)

   {

      sum += (s[k] & 0xFF) * (k+1) * ECX;

      ECX *= 0x00600937;

   }

   return sum;

}
```
`4dd466d0e1e0cc123afbe504da2462ca`
