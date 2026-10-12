
### 汇编代码分析：注册码处理逻辑

```
0879790E   |.  83>sbb edx,0                         ;  这里必不经过  
08797911   |>  5F pop edi  
08797912   |.  5E pop esi  
08797913   |.  5B pop ebx  
08797914   |.  5D pop ebp  
08797915   |.  C2>retn 8  
08797918   |>  F7>div ebx                     ;  如果进位为 0 把固定值 3F17DC15 除以 2  
0879791A   |.  92 xchg eax,edx                ;  交换 EAX 与 EDX  为了得到余数  
0879791B   |.  33>xor edx,edx                ;  把商清零  
0879791D   \\.^ EB>jmp short Hrms.08797911  

…………………………………………………………………………………………………………………………  

发现处理注册码的函数不可逆！函数出来后到了这里：比较终端数目！  

…………………………………………………………………………………………………………………………  

08A0A416    .  85>test eax,eax  
08A0A418    .  75>jnz short Hrms.08A0A448     ;  如果 EAX 等于 0 就错误  
08A0A41A    .  6A>push 10  
08A0A41C    .  B9>mov ecx,Hrms.08A0A610  
08A0A421    .  BA>mov edx,Hrms.08A0A624  
08A0A426    .  A1>mov eax,dword ptr ds:\[8A19470\]  
08A0A42B    .  8B>mov eax,dword ptr ds:\[eax\]  
08A0A42D    .  E8>call Hrms.087E7400         ;  错误窗口！  
08A0A432    .  8B>mov eax,dword ptr ss:\[ebp-4\]  

……………………省略……………………  

08A0A454    .  E8>call Hrms.087C7A28         ;  得到输入的终端数 返回位数  
08A0A459    .  8B>mov eax,dword ptr ss:\[ebp-30\]  
08A0A45C    .  8D>lea edx,dword ptr ss:\[ebp-2C\]  
08A0A45F    .  E8>call Hrms.08799DE4  
08A0A464    .  83>cmp dword ptr ss:\[ebp-2C\],0  ;  比较看有没有输入数据  
08A0A468    .  74>je short Hrms.08A0A48B  
08A0A46A    .  8D>lea edx,dword ptr ss:\[ebp-34\]  
08A0A46D    .  8B>mov eax,dword ptr ss:\[ebp-4\]  
08A0A470    .  8B>mov eax,dword ptr ds:\[eax+2FC\]  
08A0A476    .  E8>call Hrms.087C7A28         ;  得到输入的终端数 返回位数  
08A0A47B    .  8B>mov eax,dword ptr ss:\[ebp-34\]  
08A0A47E    .  E8>call Hrms.0879A138         ;  将终端数转化为 16 进制数  
08A0A483    .  0F>movzx edx,word ptr ss:\[ebp-E\]    ;  取前面计算出来的结果 2 的 16 进制后 4 位  
08A0A487    .  3B>cmp eax,edx                  ;  与输入的终端数比较  
08A0A489    .  74>je short Hrms.08A0A4B9  

…………………………………………………………………………………………………………………………  

由上面可以知道只能逆转处理序列号的函数了，这里发现可以直接在 `OD` 中逆循环只要改变一下代码  

如果你已经到了 `089F0D80` 处的指令：

```
089F0D80   |.  3B>cmp eax,dword ptr ss:\[ebp-20\]    ;  比较   用注册码双精度数右移后的结果与序列号处理后的结果比较  
```

看到了处理序列号的应该的值，就可以逆转了。修改代码：`089F0B74` 的 `push` 段的代码：

…………………………………………………………………………………………………………………………

### 逆向工程：序列号验证算法逆向

```
08BA0B8D   |> /4A /dec edx                     ;   
08BA0B8E     |2B>sub esi,dword ptr ds:\[8BC8C84\]   ;  注意这里走过这个代码后修改把 ESI 改成 C6EF3720  
08BA0B94     |8B>mov edi,ebx  
08BA0B96   |. |C1>|shl edi,4                  ;   
08BA0B99     |2B>sub eax,edi                  ;   
08BA0B9B   |. |8B>|mov edi,dword ptr ds:\[ecx\]     ;   
08BA0B9D   |. |33>|xor edi,ebx               ;   
08BA0B9F     |2B>sub eax,edi                  ;   
08BA0BA1   |. |8B>|mov edi,ebx  
08BA0BA3   |. |C1>|shr edi,5                  ;   
08BA0BA6   |. |33>|xor edi,esi               ;   
08BA0BA8     |2B>sub eax,edi                  ;   
08BA0BAA     |2B>sub eax,dword ptr ds:\[ecx+4\]    ;   
08BA0BAD   |. |8B>|mov edi,eax  
08BA0BAF   |. |C1>|shl edi,4                  ;   
08BA0BB2     |2B>sub ebx,edi                  ;   
08BA0BB4     |8B>mov edi,dword ptr ds:\[ecx+8\]    ;   
08BA0BB7   |. |33>|xor edi,eax  
08BA0BB9     |2B>sub ebx,edi  
08BA0BBB   |. |8B>|mov edi,eax  
08BA0BBD   |. |C1>|shr edi,5  
08BA0BC0   |. |33>|xor edi,esi  
08BA0BC2     |2B>sub ebx,edi  
08BA0BC4     |2B>sub ebx,dword ptr ds:\[ecx+C\]  
08BA0BC7   |. |85>|test edx,edx  
08BA0BC9   |.^\\77>\\ja short Hrms.08BA0B8D  
08BA0BCB   |.  8B>mov edx,dword ptr ss:\[esp\]  
```

也就是把加改成减，而且把最后要进行比较的值放在 `EBX` 中，`EAX` 中的数据随便填。

并把计算的参数前八位与后 8 位交换变成：

```
59 6F 75 20 61 72 65 20  You are  
48 61 2E 2E 2E 68 61 2C  Ha...ha,  
66 61 74 20 70 69 67 21  fat pig!  
```

当然最后的结果要稍微处理一下。

…………………………………………………………………………………………………………

### 附注册程序代码

```c
unsigned long int xiangx(shuru,b,p)
unsigned long int shuru,b,*p;
{
    unsigned long int jie[8],c,d,e=0xffffffff,jinwei=0,qian8=0,i,g,j,k,h,hou8;
    static long int a[8];
    unsigned static long int carry[8];

    for(i=1;i<=8;i++)
    {
        k=shuru<<(32-i*4);
        k=k>>(32-i*4);
        a[8-i]=k>>((i-1)*4);
    }

    for(i=0;i<8;i++)
    {
        h=b;
        jinwei=0;
        for(j=1;j<a[7-i];j++)
        {
            if((e-h)<=b)
            {
                jinwei++;
                h=h+b;
            }
            else
            {
                h=h+b;
            }
        }
        d=jinwei;
        carry[i]=d;
        jie[i]=b*a[7-i];
    }

    for(i=1;i<8;i++)
    {
        carry[i]=carry[i]<<(i*4);
        g=jie[i]>>(32-i*4);
        carry[i]=carry[i]|g;
    }

    for(i=0;i<8;i++)
    {
        qian8=qian8+carry[i];
    }

    jinwei=0;
    for(i=1;i<8;i++)
    {
        if((e-jie[0])<=(jie[i]<<(4*i)))
        {
            jinwei++;
            jie[0]=jie[0]+(jie[i]<<(4*i));
        }
        else
        {
            jie[0]=jie[0]+(jie[i]<<(4*i));
        }
    }

    qian8=qian8+jinwei;
    hou8=jie[0];
    *p=qian8;
    return(hou8);
}

unsigned long int chuli2(jie,jinwei,gud)
unsigned long int jie,jinwei,gud;
{
    unsigned long int jieguo=0,i,e=0xffffffff,c;

    if(jinwei==0)
    {
        jieguo=jie%gud;
    }
    else
    {
        for(i=64;i>0;i--)
        {
            if((e-jie)<=jie)
            {
                c=1;
                jie=jie+jie;
            }
            else
            {
                c=0;
                jie=jie+jie;
            }

            if((e-jinwei)<=jinwei)
            {
                jinwei=2*jinwei+c;
                c=1;
            }
            else
            {
                jinwei=2*jinwei+c;
                c=0;
            }

            jieguo=2*jieguo+c;

            if(jieguo>gud)
            {
                jieguo=jieguo-gud;
                jie++;
            }
        }
    }
    return(jieguo);
}

unsigned long int fun(x,y,z)
unsigned long int x,y,z;
{
    unsigned long int jie,jinwei,*p=&jinwei,jieguo;
    jie=xiangx(x,y,p);
    jieguo=chuli2(jie,jinwei,z);
    return(jieguo);
}

unsigned long int zong(shuru)
unsigned long int shuru;
{
    unsigned long int xunh=0x3f17dc15,gud=0x758f0581,bchenji=1;

    for(;xunh>0;)
    {
        if((xunh%2)==0)
        {
            xunh=xunh/2;
            shuru=fun(shuru,shuru,gud);
        }
        else
        {
            xunh--;
            bchenji=fun(shuru,bchenji,gud);
        }
    }
    return(bchenji);
}

unsigned long int xlh(qian4,hou4,p)
unsigned long int qian4,hou4,*p;
{
    unsigned long int lei,guding1,guding2,guding3,guding4,guding5;
    int i;

    guding1=0x20756f59;
    guding2=0x20657261;
    guding3=0x2e2e6148;
    guding4=0x2c61682e;
    guding5=0x9e3779b9;

    lei=0xC6EF3720;

    for(i=0;i<0x20;i++)
    {
        hou4=hou4-(qian4<<4);
        hou4=hou4-(qian4^guding1);
        hou4=hou4-((qian4>>5)^lei);
        hou4=hou4-guding2;

        qian4=qian4-(hou4<<4);
        qian4=qian4-(hou4^guding3);
        qian4=qian4-((hou4>>5)^lei);
        qian4=qian4-guding4;

        lei=lei-guding5;
    }
    *p=hou4;
    return(qian4);
}

unsigned long int jianpan(p)
unsigned long int *p;
{
    unsigned long int shuru1=0,shuru2=0;
    int n,i,zj;
    char zifu1[20],zifu2[20],zifu3[20],zifu4[20];

    scanf("%s",zifu1);
    scanf("%s",zifu2);
    scanf("%s",zifu3);
    scanf("%s",zifu4);

    strcat(zifu1,zifu2);
    strcat(zifu3,zifu4);

    n=strlen(zifu1);
    for(i=0;i<n;i++)
    {
        if((zifu1[i]>0x29)&(zifu1[i]<0x40))
        {
            zj=zifu1[i]-0x30;
            shuru1=(shuru1<<4)|zj;
        }
        else if((zifu1[i]>0x40)&(zifu1[i]<0x47))
        {
            zj=zifu1[i]-0x37;
            shuru1=(shuru1<<4)|zj;
        }
        else if((zifu1[i]>0x60)&(zifu1[i]<0x67))
        {
            zj=zifu1[i]-0x57;
            shuru1=(shuru1<<4)|zj;
        }
        else
        {
            shuru1=0;
            break;
        }
    }

    n=strlen(zifu3);
    for(i=0;i<n;i++)
    {
        if((zifu3[i]>0x29)&(zifu3[i]<0x40))
        {
            zj=zifu3[i]-0x30;
            shuru2=(shuru2<<4)|zj;
        }
        else if((zifu3[i]>0x40)&(zifu3[i]<0x47))
        {
            zj=zifu3[i]-0x37;
            shuru2=(shuru2<<4)|zj;
        }
        else if((zifu3[i]>0x60)&(zifu3[i]<0x67))
        {
            zj=zifu3[i]-0x57;
            shuru2=(shuru2<<4)|zj;
        }
        else
        {
            shuru2=0;
            break;
        }
    }
    *p=shuru2;
    return(shuru1);
}

main()
{
    unsigned long int shuru1,shuru2,jieguo1,jieguo2,f=0x0000000f,d,xulihao1,xulihao2,*p=&xulihao2,b;
}
```
