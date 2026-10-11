
# Cabinet Manager 2001 Ver3.7.0.87

> 原文：[`www.pediy.com/kssd/pediy05/pediy50200.htm`](https://www.pediy.com/kssd/pediy05/pediy50200.htm)

```c
/* ---Turbo C 2.0---Cabinet Manager 2001 Ver3.7.0.87 的注册机*/

#include <stdio.h>

#include <string.h>

#include <dos.h>

main()

{

struct date d;

struct time t;

long int y1,c,i,j,total,e=0;

static char string[60];

static int m[]={-1,30,58,89,119,150,180,211,242,273,303,333};

gettime(&t);

getdate(&d);y1=d.da_year;

/*计算 1900 年至今的时间(秒)*/

total=(((d.da_day+m[d.da_mon-1]+((y1-1900-1)>>2)+((y1-1900)*365))*24+t.ti_hour)*60+t.ti_min)*60+t.ti_sec;

total+=0x7c551100;        /*和 0x7c551100 运算*/

total=total&0xffff0000;   /*低四为取零,留高四位*/

start:

clrscr();

printf("\nCabinet Manager 2001 Ver3.7.0.87  http://www.microlognet.com\n");

printf("\nPlease enter your name : ");

gets(string);j=strlen(string);

for(i=0;i<j;i++){c=string[i];e+=c;}   /*计算用户名 ASCII 码之和，作为参与运算的低位*/

e+=total;        /*高位＋低位，构成程序所显示的十进制 Cert*/

if(j>=5){

    printf("\nYour    Cert      is  : %d\n",9067);   /*9067 为用户所输入的 Cert*/

    printf("Program display Number : %ld",e);

    e=e^0x90b96501;e=2*e^0x6967;            /*程序显示的 Cert,运算得到注册码的数字部分*/

    printf("\nYour Register code is  : 3RE%ld",e);}   /*前三位 3RE＋计算所得注册码数字部分*/

else {  printf("\nYour username must >5 Character!\n");

    e=0;

    getch();

    goto start;}

printf("\n\n===== Thanks! E_Mail:oakyong@21cn.com =====");

getch();

}  4dd466d0e1e0cc123afbe504da2462ca
```
