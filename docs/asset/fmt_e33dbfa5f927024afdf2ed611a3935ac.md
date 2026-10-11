
`005BCA62` `|` `.` `8B>mov eax,dword ptr ds:[edi]`

…………………………………………………………………………………………………………………………

附：

```c
main()  

{  

unsigned long int guding=0x75bcd15,jieguo=0,zj;  

int i,yushu,b;  

static char zuhe[21]="000000TOPTHINK000000",jiqima[12],zhucema[13]="000000000000";  

printf("shuru jiqima\\n");  

scanf("%s",jiqima);  

for(i=0;i<12;i++)  

{  

if((jiqima[i]>='a')&(jiqima[i]<='z'))  

jiqima[i]=jiqima[i]-0x20;  

}  

for(i=0;i<6;i++)  

{zuhe[i]=jiqima[i];}  

for(i=14;i<20;i++)  

{zuhe[i]=jiqima[i-8];}  

for(i=19;i>=0;i--)  

{  

jieguo=jieguo+guding;  

yushu=0x459%zuhe[i];  

jieguo=jieguo-yushu;  

}  

for(i=11,b=1;i>3;i--,b++)  

{  

zj=jieguo<<(32-b*4);  

zj=zj>>28;  

if((zj>=0)&(zj<=9))  

zj=zj+0x30;  

else if((zj>=0xa)&(zj<=0xf))  

zj=zj+0x37;  

zhucema[i]=zj;  

}  

printf("zhucema:\\n");  

printf("%s",zhucema);  

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
