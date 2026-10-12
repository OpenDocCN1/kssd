
```
0xB0C2649Bl,0x26F263ECl,0x9CA36A75l,0x0A936D02l,0xA906099Cl,0x3F360EEBl,0x85670772l,0x13570005l,

0x824ABF95l,0x147AB8E2l,0xAE2BB17Bl,0x381BB60Cl,0x9B8ED292l,0x0DBED5E5l,0xB7EFDC7Cl,0x21DFDB0Bl,

0xD4D2D386l,0x42E2D4F1l,0xF8B3DD68l,0x6E83DA1Fl,0xCD16BE81l,0x5B26B9F6l,0xE177B06Fl,0x7747B718l,

0xE65A0888l,0x706A0FFFl,0xCA3B0666l,0x5C0B0111l,0xFF9E658Fl,0x69AE62F8l,0xD3FF6B61l,0x45CF6C16l,

0x78E20AA0l,0xEED20DD7l,0x5483044El,0xC2B30339l,0x612667A7l,0xF71660D0l,0x4D476949l,0xDB776E3El,

0x4A6AD1AEl,0xDC5AD6D9l,0x660BDF40l,0xF03BD837l,0x53AEBCA9l,0xC59EBBDEl,0x7FCFB247l,0xE9FFB530l,

0x1CF2BDBDl,0x8AC2BACAl,0x3093B353l,0xA6A3B424l,0x0536D0BAl,0x9306D7CDl,0x2957DE54l,0xBF67D923l,

0x2E7A66B3l,0xB84A61C4l,0x021B685Dl,0x942B6F2Al,0x37BE0BB4l,0xA18E0CC3l,0x1BDF055Al,0x8DEF022Dl};
```

```c
int table2[]={0x0A,0x16,0x1A,0x0B,0x07,0x08,0x01,0x17,0x05,0x1C,0x00,0x19,0x13,0x1B,0x03,0x1D,0x0D,
            0x1E,0x02,0x18,0x0F,0x06,0x09,0x10,0x0C,0x0E,0x11,0x15,0x12,0x04,0x14,0x00};

int table3[]={0x32,0x33,0x34,0x35,0x36,0x37,0x38,0x39,0x41,0x42,0x43,0x44,0x45,0x46,0x47,0x48,0x4A,
            0x4B,0x4C,0x4D,0x4E,0x50,0x52,0x53,0x54,0x55,0x56,0x57,0x58,0x59,0x5A,0x00};

char name[30],code[18],c;

char s1[40]="00000001";

int namelen;

unsigned long  eax,ebx,ecx,edx,edi,esi,ebp;

unsigned long temp,a=0,b=0;

int i,j;

for(i=0;i<256;i++)         //Table1 表内的常量顺序颠倒过来。

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

  }while((__asm edi)!=0);  //请教各位大侠，这样用汇编是否合适？我想直接用 edi 的值比较。上面程序都对，可是这里 EAX 与 EDI 相乘，需要高 4 字节，只好用汇编，不知这个循环条件是否可以用汇编代码？如果全部用汇编，就要全部把 C 代码换成汇编代 码了，那样直接采用上面软件里面的即可，当然需要局部修改。

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

用户名：`wanggang`

注册码：`FKXN8-FR9EM-8C4D2`

完稿：`2006` 年 `2` 月 `8` 日

`qduwg`

`qduwg@163.com`

`4dd466d0e1e0cc123afbe504da2462ca`
