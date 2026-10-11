
# MagicWin 98 1.30f

> 原文：[`www.pediy.com/kssd/pediy03/forum408.htm`](https://www.pediy.com/kssd/pediy03/forum408.htm)

`MagicWin 98 1.30f` 注册机：

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>

#define BUF_LEN 30
#define HEAD_SIZE 3

/* 前三个字符不作处理，Serial Number 的长度要大于这个值 */

#define MOV_SIZE 4
/* 对 Serial Number 处理时，间隔 4 个字节相互替换 */

void int2str(int n, char *pt);
/* 好久没用 TC20 了，所以不知道怎么将 Int 型的值转换成字符串 */

main () {
 char S_N[BUF_LEN];      /* Serial Number 存放的数组 */
 char User_Name[BUF_LEN]; /* 用户名存放的数组 */
 char *pstr, ch;        
 unsigned int sum = 0;
 pstr = S_N;

 printf("Please Enter your Serials Number: ");
 scanf("%s",S_N);

 if (strlen(S_N) > HEAD_SIZE) {
   printf("\nPlease Enter your Name: ");
   scanf("%s",User_Name);

   if ((sum = strlen(User_Name)) > 0) {
     pstr = User_Name;      /* 将用户名字符的 ASCII */
     sum += pstr;           /* 值累加，并加上用户名 */
     while (*pstr++ != '\0') /* 长度。           */
       sum += *pstr;         
     int2str(sum, User_Name); /* 求出这个 Int 对应的字符串 */
     pstr = S_N + HEAD_SIZE;
     while (strlen(pstr) > MOV_SIZE) { /* 每隔 4 个字符 xchg */
       ch = *pstr;
       *pstr = *(pstr + MOV_SIZE);
       *(pstr + MOV_SIZE) = ch;
       pstr++;
     }

     while (*pstr != '\0' && *(pstr + 1) != '\0') {
       ch = *pstr;                
       *pstr = *(pstr+1);
                       /* 将最后不大于 4 长度的字符循环左移一位 */
       *(pstr+1) = ch;
       pstr++;
     }

     pstr = S_N + HEAD_SIZE;
     sum = HEAD_SIZE;
     while (*pstr != '\0') {/* 从第四个字符起将转换了得  */
       *pstr += sum;  /* 字符串依次加上 0x10 + 在  */
       *pstr += 0x10; /* 字符串中该字符的位置。注：*/
       pstr++;        /* 第一个字符的位置为 0，以此 */
       sum++;         /* 类推                  */
 }

strcat(S_N, User_Name); /* 将 User_Name 字符串（转变后的） */
                     /* 接到 S_N 字符串的后面，此时的  */
                     /* S_N 字符串就是正确的 Code 了    */

 printf ("\nThe MagicWin 98 1.30f software Code is: %s \n",S_N);

   }

   else {
     printf ("Name can't be NULL.");

   }

 }

 else {
   printf ("Serials Number Must be %d char or more.\n",HEAD_SIZE+1);

 }

}

void int2str(int n, char *pt) {

 int k = 0;
 ldiv_t lx;
 lx.quot = n;
 while (lx.quot != 0) {
   lx = ldiv(lx.quot, 10L);
   *(pt+k) = lx.rem + 0x30;
   k++;
 }
 
 *(pt+k) = '\0';
 strrev(pt);
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
