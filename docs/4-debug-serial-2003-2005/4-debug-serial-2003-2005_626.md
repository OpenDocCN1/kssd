
# freeRes V0.94

> 原文：[`www.pediy.com/kssd/pediy06/pediy6520.htm`](https://www.pediy.com/kssd/pediy06/pediy6520.htm)

写了很久的东西了；今天来学院；没什么东东贴，就上这个吧

中了一次毒，丢了不少东东

最可惜的是一堆关于公开密钥算法快速实现的资料，希望能……

```c
/* freeRes V 0.94 KeyGen */

/* by lq7972 */

/* bruceyu13@sina.com */

#include <stdio.h>

#include <string.h>

#include <wchar.h>

char *ChangeName (char *, int) ;

int ModifyValue (int, int) ;

int main ()

{
  int  name_len, i, counter, temp ;
  unsigned long int temp00, tempNumber = 0x3D6 ;
  char user_name [255] =  "", tempName [255] = "", hard_code [255] = "" ;
  char *new_name, RegCode [41] = "" ;
  wchar_t reg_code01 [20], reg_code02 [20] ;
  FILE *cfPtr ;

  printf ("\t   freeRes V 0.94 KeyGen\n\tby lq7972[bruceyu13@sina.com]\n\n") ;

  printf ("Enter your name : ") ;

  gets (user_name) ;

  printf ("Enter hard code : ") ;

  gets (hard_code) ;

  strcpy (tempName, user_name) ;

  strcat (tempName, hard_code) ;

  name_len = strlen (tempName) ;

  /************************************************************/

  /* change user_name */

  new_name = ChangeName (tempName, name_len) ;

  name_len = strlen (new_name) ;

  /************************************************************/

  /* Calc01 */

  for (counter=0; counter<name_len; counter++) {
    temp00 = tempNumber ;
    temp00 <<= 16 ;
    temp00 >>= 24 ;
    reg_code01 [counter] = new_name [counter] ^ temp00 ;
    tempNumber += reg_code01 [counter] ;
    tempNumber *= 0x3183 ;
    tempNumber += 0x8C34 ;
  }

  /************************************************************/

  /* Calc02 */

  tempNumber = 0x3D6 ;

  for (counter=0; counter<name_len; counter++) {
    temp00 = tempNumber ;
    temp00 <<= 16 ;
    temp00 >>= 24 ;
    reg_code02 [counter] = reg_code01 [counter] ^ temp00 ;
    tempNumber += reg_code02 [counter] ;
    tempNumber *= 0x3183 ;
    tempNumber += 0x8C34 ;
  }

  /************************************************************/

  /* Calc03 */

  for (i=0, counter=0; counter<name_len; counter++, i=i+2) {
    if (reg_code02 [counter]>0x7E) {
      temp = ModifyValue (reg_code02 [counter], 0x7E) ;
      RegCode [i] = reg_code02 [counter] >> temp ;
      RegCode [i+1] = ((temp & reg_code02 [counter]) << 4) | 0x42 ;
    }
    else {
      if (reg_code02 [counter] >=0x21) {
        RegCode [i] = reg_code02 [counter] ;
        RegCode [i+1] = 0x43 ;
      }
      else {
        temp = ModifyValue (reg_code02 [counter], 0x21) ;
        RegCode [i] = reg_code02 [counter] << temp ;
        RegCode [i+1] = temp+0x43 ;
      }
    }
  }

  printf ("Your reg code are : %s\n", RegCode) ;

  printf ("Writing file...\n") ;

  if ((cfPtr = fopen("key.txt", "w")) == NULL)
    printf ("File could not be opened\n") ;
  else
    fprintf (cfPtr, "%s\n%s\n%s%s\n%s%s\n%s\n%s",
        "freeRes V 0.94 KeyGen",
        "by lq7972[bruceyu13@sina.com]",
        "Your user name : ", user_name,
        "Your hard code : ", hard_code,
        "Your reg code are :",
        RegCode) ;

  return 0 ;
}

/************************************************************/

/**/

char *ChangeName (char *uName, int len)

{
  int i=1, j, counter, temp ;
  char nName [20] = "", tempName [19] = "" ;
  if (len <= 0x13) {
    nName [0] = uName [len-1] ;
    nName [1] = len ;
    if (0x13 == len) {
      strncpy (tempName, uName, 18) ;
      strcat (nName, tempName) ;
    }
    else {
      for (counter=(0x12-len+1); counter>1; counter--) {
        nName [counter] = nName [0] + i ;
        i ++ ;
      }
      strcat (nName, uName) ;
    }
  }
  else {
    i=(len-0x13) ; j = 0x13 ;
    temp = uName [0x12] ;
    for (; i>0; i--, j++) {
      temp -= uName [j] ;
      if (temp<0) temp *= -1 ;
    }
    nName [0] = temp ;
    nName [1] = len ;
    strncpy (tempName, uName, 18) ;
    strcat (nName, tempName) ;
  }
  return nName ;
}

/************************************************************/

/**/

int ModifyValue (int in, int select)

{
  register int counter = 0 ;
  switch (select) {
  case 0x7E :
    while (in > 0x7F) {
      counter ++ ;
      in = (in & 0xFF) >> 1 ;
    }
    if (2 == counter) counter = 3 ;
    break ;
  case 0x21 :
    while (in < 0x21) {
      counter ++ ;
      in += in ;
    }
    break ;
  }
  return counter ;
}
```

买一送一

Software : Delphi 学习宝典
教学软件，主要有以下功能:初级教程/开发教程/API 函数/内置函数/错误信息
[`learndelphi.yeah.net/`](http://learndelphi.yeah.net/)

Tools : OllyDbg

Cracker : lq7972 [bruceyu13@sina.com]

很简单的注册机制，不过这个东东的资料搜集还是下了些功夫

主程序是 Microsoft Visual Basic 6.0

### 注册机

```c
/* [请使用 E 文注册名] */

#include <stdio.h>

#include <string.h>

int main ()

{
  register int i ;
  int code = 0 ;
  char name [256] ;

  printf ("\t\tLearnDelphi V 1.2 KeyGen\n"
    "\t    by  lq7972 [bruceyu13@sina.com]\n") ;

  printf ("Enter your name : ") ;

  gets (name) ;

  for (i=0; i < strlen(name); i ++) code += name[i] ;

  printf ("Your code is : %d\n", code*0x12D687) ;

  printf ("Press any key to continue\n");

  getch () ;

  return 0 ;
}
```

4dd466d0e1e0cc123afbe504da2462ca
