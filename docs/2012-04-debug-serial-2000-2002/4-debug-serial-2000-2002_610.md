
# 东胜算码机

> 原文：[`www.pediy.com/kssd/pediy03/forum1227.htm`](https://www.pediy.com/kssd/pediy03/forum1227.htm)

```c
#include <stdio.h>
#include <conio.h>

unsigned long f0bfb(unsigned long a,unsigned long b);
unsigned long f1345(unsigned long a);
unsigned long f115a(unsigned long a);
unsigned long f1011(unsigned long a);
unsigned long f0f28(unsigned long a,unsigned long *b,unsigned long *c);
unsigned long f1256(unsigned long a);
unsigned long f0d8a(unsigned long b);
unsigned long cacu(unsigned long a,unsigned long b);
unsigned long u1011(unsigned long a);

main()
{
    unsigned long a,b,d;
    char str[10];

    printf("东胜算码机  by  STKMAN \n");
    printf("Please Input Cardid:");
    scanf("%lx",&a);

    printf("Please Input End Date:");
    scanf("%lx",&d);

    d=((((d>>0x10)-0x2000))<<9) |
      ((((d & 0xf000)>>0xc)*0xa+((d & 0x0f00)>>0x8))<<5) |
      ((((d & 0xf0)>>4)*0xa)+(d & 0xf));

    b=0x3838;

    printf("Password is %lX !\n",cacu(a,d));
    printf("Super Password is %lX ! \n",cacu(a,b));
}

unsigned long cacu(unsigned long a,unsigned long b)
{
    unsigned long x,y,z,m;
    int i;
    char str[10];

    x=f1256(f115a(a) ^ f1345(f115a(f0d8a(b))));
    x=((x / 0x10000) ^ x) & 0xffff;
    x= x ^ 0x3838;
    x=f0f28(0,&b,&x);
    x=x ^ f1345(a);
    z=f1345(a ^ 0xffffffff);
    y=(f1345(f115a(a)) % 0x138)+0x38;

    for(i=0;i<y;i++)
        x=u1011(x);

    m=z^x;
    return(m);
}

unsigned long u1011(unsigned long a)
{
    unsigned long x,y,z,n;
    int m;

    x= a & 0x0f;
    y=x;
    z=0;m=4;

    while(m<0x20)
    {
        y=y | (((unsigned long) 1) <<m);
        z=z | (((unsigned long) 1) <<m);

        n=(y+(y<<4)+(y<<5)+(y<<6)+(y<<8)+(y<<9)
           +(y<<0x0a)+(y<<0x0c)+(y<<0x0e)) & z;

        if((a & (((unsigned long)1)<<m)) ==0)
        {
            x=(x^(x&(((unsigned long)1)<<m))) | (n^(((unsigned long)1)<<m));
            y=y^(((unsigned long)1)<<m);
        }
        else
            x=(x^(x&(((unsigned long)1)<<m))) | (n&(((unsigned long)1)<<m));

        m++;
    }

    x=x+0x3838;
    return(x);
}

unsigned long f0bfb(unsigned long a,unsigned long b)
{
    unsigned long c,x,y,z,m;
    int i;

    x=b ^ f1345(a ^0xffffffff);
    y=(f1345(f115a(a)) % 0x138)+0x38;

    for(i=0;i<y;i++)
        x=f1011(x);

    x=x ^ f1345(a);
    f0f28(x,&y,&x);

    z=f1256(f115a(a) ^ f1345(f115a(f0d8a(y))));
    z=((z / 0x10000) ^ z) & 0xffff;
    b= x ^ z;

    if(b==0x3838)
        return(y);
    else
        return(0x4949);
}

unsigned long f1345(unsigned long a)
{
    unsigned char x[4]={0,0,0,0},p[4]={0x69,0x94,0x93,0x70};
    unsigned char q,m,y,n;
    unsigned long r;
    int z,i,j;

    for(z=0;z<4;z++)
    {
        x[z]=(unsigned char)(a >> (0x18-z*8));
        p[3-z]=p[3-z] ^ x[z];
    }

    p[0]=p[0]^0x5a;
    if(p[0]==0) p[0]=0x5a;

    for(z=(p[0] & 0xf) + 0x20;z>0;z--)
    {
        q=0;
        for(i=0;i<4;i++)
        {
            n=0;
            y=x[i] & p[i];
            for(j=0;j<8;j++)
                n=n ^ ((y>>j) & 0x1);
            q=q^n;
        }

        for(i=3;i>=0;i--)
        {
            m=0;
            if(x[i] & 0x80) m=1;
            x[i]=(x[i]<<1) | q;
            q=m;
        }
    }

    r=0;
    for(i=0;i<4;i++)
        r=r+((unsigned long)(x[i])<<(0x18-i*8));

    return(r);
}

unsigned long f115a(unsigned long a)
{
    unsigned long x,y;

    x=((a>>1) & 0x55555555) | ((a<<1) & 0xaaaaaaaa);
    x=((x>>2) & 0x33333333) | ((x<<2) & 0xcccccccc);
    x=((x>>4) & 0x0f0f0f0f) | ((x<<4) & 0xf0f0f0f0);
    x=((x>>8) & 0x00ff00ff) | ((x<<8) & 0xff00ff00);
    x=(x>>0x10) | (x<<0x10);

    return(x);
}

unsigned long f1011(unsigned long a)
{
    unsigned long x,y,z,n;
    int m;

    x=a-0x3838;
    y=x & 0xf;
    z=0;m=4;

    while(m<0x20)
    {
        y=y | (((unsigned long)1)<<m);
        z=z | (((unsigned long)1)<<m);

        n=(y+(y<<4)+(y<<5)+(y<<6)+(y<<8)+(y<<9)
           +(y<<0x0a)+(y<<0x0c)+(y<<0x0e)) & z;

        if((x & z)!=n)
            y=y ^ (((unsigned long )1)<<m);

        m++;
    }

    return(y);
}

unsigned long f0f28(unsigned long a,unsigned long *b,unsigned long *c)
{
    char x;

    if(a!=0)
    {
        *b=0;*c=0;
        x=0;

        while(x<0x10)
        {
            *c= (*c) | (((a>>(x<<1)) & 1)  << x);
            *b= (*b) | (((a>>((x<<1)+1)) & 1) << x);
            x++;
        }
    }
    else
    {
        x=0;

        while(x<0x10)
        {
            a=a+  (((*c >> x) & 1) << (x<<1));
            a=a+  (((*b >> x) & 1) << ((x<<1)+1));
            x++;
        }
    }

    return(a);
}

unsigned long f0d8a(unsigned long b)
{
    unsigned long x,y;

    y=(((b>>9)/0x64)<<8) | ((((b>>9)%0x64)/0x0a) <<4) | (((b>>9)%0x0a) & 0x0f);
    x=(y+0x2000) << 0x10;
    y=(b>>5) & 0xf;

    if(y>0x0a)  y=y+6;
    x=x | (y<<8);

    y=b & 0x1f;
    x=x | ((y/0x0a)<<4) | (y%0x0a);

    return(x);
}

unsigned long f1256(unsigned long a)
{
    unsigned long x,y,z,m,n;
    int i;

    x=a ^ 0xffffffff;y=0;i=0;

    while(i<0x78)
    {
        z=x;m=0;

        while(m<0x20)
        {
            /*  n=z;  */
            if(z & 1)
                if((m%3)==0)
                    y=y ^ 1;
            z=z>>1;
            m++;
        }

        x=x+x;
        m=y & 1;
        y=m;
        x=x | m;
        i++;
    }

    return(x);
}
```
