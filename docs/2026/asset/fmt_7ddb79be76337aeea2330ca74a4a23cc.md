
```c
unsigned long  eax,ebx,ecx,edx,edi,esi,ebp;

unsigned long temp,a=0,b=0;

int i,j;

for(i=0;i<256;i++)         //Table1表内的常量顺序颠倒过来。

{ temp=table1[i];

  for(j=0;j<4;j++)

{

  a=temp%0x100;

  temp=temp>>8;

  b=b*0x100;

  b+=a;

  }

  table1[i]=b;

  b=0;

}

printf("please input your name\n");

scanf("%s",name);

strcat(s1,name);

namelen=strlen(s1);

eax1=-1;

edx=0;

ecx=0;

for(i=0;i<namelen;i++)

{ecx=s1[i];

  edi=eax1;

  edi=edi&0xff;

  eax1=eax1>>8;

  ecx=ecx&0xff;

  ecx=ecx^edi;

  ecx=table1[ecx];

  eax1=eax1^ecx;

  ecx=0;

  }

  eax1=~eax1;

esi=eax1;

eax1=eax1>>7;

esi=esi<<0x19;

esi=esi|eax1;

edi=esi;

edx=esi;

edi=edi>>0x1b;

edx=edx<<5;

edi=edi|edx;

ebp=esi;

eax1=esi;

ebp=ebp<<0x14;

eax1=eax1>>0xc;

ebp=ebp|eax1;

ecx=0;

do

{

edx=0;

eax1=edi;

ebx=0x1f;

edx=eax1%ebx;

eax1=table2[edx];

c=table3[eax1];

if(ecx==5)

code[ecx++]=0x2D;

else

code[ecx++]=c;

   __asm{

         mov eax,8421085

         mul edi

         sub edi,edx

         shr edi,1

         add edi,edx

         shr edi,4

         }

}while((__asm edi)!=0);  //请教各位大侠，这样用汇编是否合适？我想直接用edi的值比较。上面程序都对，可是这里EAX

与EDI相乘，需要高4字节，只好用汇编，不知这个循环条件是否可以用汇编代码？如果全部用汇编，就要全部把C代码换成汇编代

码了，那样直接采用上面软件里面的即可，当然需要局部修改。

if(ecx<7)

{edi=esi;

   edi=edi*ecx;

   do{

     edx=0;

     eax1=edi;

     ebx=0x1f;

     edx=eax1%ebx;

     edi=edi+esi;

     edx=table2[edx];

     c=table3[edx];

     if(ecx==5)

       code[ecx++]=0x2D;

     else

       code[ecx++]=c;

     }while(ecx<7);

  }

else

{

  ebx=0;

  do{

    edx=0;

    eax1=ebp;

    edi=0x1f;

    edx=eax1%edi;

    eax1=table2[edx];

    c=table3[eax1];

    if(ecx==0xC)

      code[ecx++]=0x2D;

    else

      code[ecx++]=c;

    __asm{

        mov eax,8421085

        mul ebp

        sub ebp,edx

        shr ebp,1

        add ebp,edx

        shr ebp,4

        }

    }while((__asm ebp)!=0);

}

if(ecx<0xf)

  {edi=esi;

   edi=edi*ecx;

   do{

      edx=0;

      eax1=edi;

      ebp=0x1f;

      edx=eax1%ebp;

      edi=edi+esi;

      edx=table2[edx];

      c=table3[edx];

      if(ecx==0xC)

        code[ecx++]=0x2D;

      else

        code[ecx++]=c;

     }while(ecx<0xf);

   }

  printf("The code is %s",code);

}
```

结论：

用户名：wanggang

注册码：FKXN8-FR9EM-8C4D2

完稿：2006年2月8日

qduwg

qduwg@163.com

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
