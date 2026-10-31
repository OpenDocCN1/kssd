
# HappyIcon注册机TC源码 (1千字)

最新版本V2.53,可将图形或相片转为Icon文件

`http://happyicon.free.fr/happyicone.zip`

```
#include  <stdio.h>

int main(void)

{

const long a=0x13579ace;

int counter=0,i=0,j=0,k=0;

unsigned long zcm=0;

unsigned long b=0;

unsigned long c=0xffffffff;

unsigned char yourname[30];

unsigned char firstname[30];

unsigned char name0[10]={'H','a','p','p','y','I','c','o','n','\0'};

unsigned char name[64];

clrscr();

printf("HappyIcon----keygen by guodong\n");

printf("http://duba.126.com\n");

printf("\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\n");

printf("please enter your name:");

gets(yourname);

printf("please enter your first name:");

gets(firstname);

for(i=0;firstname[i]!='\0';++i)

    name[i]=firstname[i];

for(j=0;name0[j]!='\0';++j)

    name[j+i]=name0[j];

for(k=0;yourname[k]!='\0';++k)

    name[k+j+i]=yourname[k];

    name[i+j+k]='\0';

for(counter=0;counter<=strlen(name)-1;counter++)

{

  b=name[counter]^c;

  c=b^a;

    }

zcm=c^0x2468bdf0;

printf("your register number is: %lu\n",zcm);

printf("bye");

return(0);

}
```
