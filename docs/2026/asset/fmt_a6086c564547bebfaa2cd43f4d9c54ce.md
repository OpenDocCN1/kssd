
```c
unsigned long tmpCallLng;

tmpCallDbl=log(inDbl)/log(2)*2;     /* 以2为底求对数 */

tmpCallLng=tmpCallDbl+0.5;          /* 四舍五入 */

tmpCallDbl-=tmpCallLng;

tmpCallDbl=pow(2,tmpCallDbl-1+1);

tmpCallDbl=pow(2,tmpCallLng)*tmpCallDbl;

return(tmpCallDbl);

}
```

```c
void main()

{

/*****************************************************/
/* KeyGen By XJTL   747K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6f1y4F1h3p5S2S2j5$3E0W2M7W2)9J5k6h3y4G2L8b7`.`. */
/*****************************************************/

  double tmpDbl;
  unsigned long tmpLng1,tmpLng2,OldLng1=0,OldLng2=0,Key1,Key2,Key3,ecx;
  int j,k,l,eax,esi=0;
  char n[15],sn[15];
  clrscr();
  printf("\n********** KeyGen For BB_038wg **********\n");
  printf("\n*****Powered By XJTL CnXHacker.com*****\n\n");
  printf("Pls Input Your Machine Code:");
  gets(n);
  l=strlen(n);
  printf("\nYour Serial Num is : ");
  for(k=1;k<=l;k++)
  {
    tmpDbl=sqrt(n[k-1])*k+1;
    tmpLng1=n[k-1]*k*k;
    tmpDbl*=tmpLng1;
    OldLng1+=tmpDbl;
    Key1=OldLng1%0x186A0;      /* 密钥1 */
    tmpDbl=Call1(n[k-1]);      /* 调用求对数函数 */
    tmpLng1=tmpDbl*k+0.5;      /* 四舍五入 */
    OldLng2*=k;
    tmpDbl=sqrt(Key1);
    tmpLng1+=OldLng2;
    tmpLng2=tmpLng1;           /* 保存临时变量 */
    OldLng2=tmpLng1%0x186A0;
    Key2=OldLng2;              /* 密钥2 */
    tmpLng1=tmpDbl;            /* 保存临时变量 */
    tmpLng2+=tmpLng1;
    tmpLng2=tmpLng2%0x186A0;
    Key3=tmpLng2;              /* 密钥3 */
  }

/* 根据密钥生成三个表,共15 byte */
  for(eax=0;eax<=4;eax++)
  {
    ecx=eax;
    ecx*=eax;
    ecx*=eax;
    ecx+=Key1+0x1F;
    ecx&=0x8000007F;
    sn[eax]=ecx;            /* 生成表第一部分,5 byte */
  }
  for(eax=5;eax<=9;eax++)
  {
    ecx=eax;
    ecx*=eax;
    ecx*=eax;
    ecx+=Key2+0x1F;
    ecx&=0x8000007F;
    sn[eax]=ecx;            /* 生成表第二部分,5 byte */
  }
  for(eax=10;eax<=14;eax++)
  {
    ecx=eax;
    ecx*=eax;
    ecx*=eax;
    ecx+=Key3+0x1F;
    ecx&=0x8000007F;
    sn[eax]=ecx;            /* 生成表第三部分,5 byte */
  }
  ecx=0;
  while(esi<=14)
  {
     /* 遍历表输出可显字符 */
    if(sn[esi]>='0'&&sn[esi]<='9'||sn[esi]>='a'&&sn[esi]<='z'||sn[esi]>='A'&&sn[esi]<='Z')
    {
      ecx+=7;
      esi+=1;
    }
    else
    {
      eax=sn[esi]+ecx+0x1F;
      eax&=0x8000007F;
      sn[esi]=eax;       /* 生成注册码,15 byte */
    }
   }

/* 注册码添加"-"然后输出 */
for(j=0;j<=4;j++)
    printf("%c",sn[j]);
printf("-");
for(j=5;j<=9;j++)
    printf("%c",sn[j]);
printf("-");
for(j=10;j<=14;j++)
    printf("%c",sn[j]);
printf("\n\n");
getch();
}
```

----------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
