
`00549236   8B0C81           MOV ECX,DWORD PTR DS:[ECX+EAX*4]         ; 又是查表（不同的数组）`

这里的表是`{0x55147626,0x8d0bf107,0xF9492A40,0x2874514A}`

```
00549239   8B44DE 04        MOV EAX,DWORD PTR DS:[ESI+EBX*8+4]       ; 查表
0054923D   8B3C24           MOV EDI,DWORD PTR SS:[ESP]
00549240   8B0487           MOV EAX,DWORD PTR DS:[EDI+EAX*4]         ; 查表
00549243   8B5CDE 08        MOV EBX,DWORD PTR DS:[ESI+EBX*8+8]       ; 查表
00549247   8B3C24           MOV EDI,DWORD PTR SS:[ESP]
0054924A   8B1C9F           MOV EBX,DWORD PTR DS:[EDI+EBX*4]         ; 查表
0054924D   03D3             ADD EDX,EBX                              ; EDX=EDX+EBX
0054924F   03DA             ADD EBX,EDX                              ; EBX=EBX+EDX
00549251   8BFA             MOV EDI,EDX                              ; EDI<=EDX
00549253   C1EF 07          SHR EDI,7                                ; EDI右移7位
00549256   33D7             XOR EDX,EDI                              ; EDX=EDX XOR EDI
00549258   03CA             ADD ECX,EDX                              ; ECX=ECX+EDX
0054925A   03D1             ADD EDX,ECX                              ; EDX=EDX+ECX
0054925C   8BF9             MOV EDI,ECX                              ; EDI<=ECX
0054925E   C1E7 0D          SHL EDI,0D                               ; 左移0XD位
00549261   33CF             XOR ECX,EDI                              ; ECX=ECX XOR EDI
00549263   03C1             ADD EAX,ECX                              ; EAX=EAX+ECX
00549265   03C8             ADD ECX,EAX                              ; ECX=ECX+EAX
00549267   8BF8             MOV EDI,EAX                              ; EDI<=EAX
00549269   C1EF 11          SHR EDI,11                               ; EDI右移0X11位
0054926C   33C7             XOR EAX,EDI                              ; EAX=EAX XOR EDI
0054926E   03D8             ADD EBX,EAX                              ; EBX=EBX+EAX
00549270   03C3             ADD EAX,EBX                              ; EAX=EAX+EBX
00549272   8BFB             MOV EDI,EBX                              ; EDI<=EBX
00549274   C1E7 09          SHL EDI,9                                ; EDI左移9位
00549277   33DF             XOR EBX,EDI                              ; EBX=EBX XOR EDI
00549279   03D3             ADD EDX,EBX                              ; EDX=EDX+EBX
0054927B   03DA             ADD EBX,EDX                              ; EBX=EBX+EDX
0054927D   8BFA             MOV EDI,EDX                              ; EDI<=EDX
0054927F   C1EF 03          SHR EDI,3                                ; EDI右移3位
00549282   33D7             XOR EDX,EDI                              ; EDX=EDX XOR EDI
00549284   03CA             ADD ECX,EDX                              ; ECX=ECX+EDX
00549286   8BD1             MOV EDX,ECX                              ; EDX<=ECX
00549288   C1E2 07          SHL EDX,7                                ; EDX左移7位
0054928B   33CA             XOR ECX,EDX                              ; ECX=ECX XOR EDX
0054928D   03C1             ADD EAX,ECX                              ; EAX=EAX+ECX
0054928F   8BD3             MOV EDX,EBX                              ; EDX<=EBX
00549291   C1EA 0F          SHR EDX,0F                               ; 右移0XF位
00549294   33C2             XOR EAX,EDX                              ; EAX=EAX XOR EDX
00549296   03D8             ADD EBX,EAX                              ; EBX=EBX+EAX
00549298   8BC3             MOV EAX,EBX                              ; EAX<=EBX
0054929A   C1E0 0B          SHL EAX,0B                               ; 左移0XB位
0054929D   33D8             XOR EBX,EAX                              ; EBX=EBX XOR EAX
0054929F   8B4424 10        MOV EAX,DWORD PTR SS:[ESP+10]            ; 取4个字节到EAX
005492A3   33C3             XOR EAX,EBX                              ; EAX=EAX XOR EBX
005492A5   8B5424 0C        MOV EDX,DWORD PTR SS:[ESP+C]             ; 取4个字节到EDX
005492A9   895424 10        MOV DWORD PTR SS:[ESP+10],EDX            ; 填充
005492AD   894424 0C        MOV DWORD PTR SS:[ESP+C],EAX             ; 填充
005492B1   83C6 0C          ADD ESI,0C                               ; ESI=ESI+0XC（这个是把数组初始地址加高）
005492B4   FF4C24 14        DEC DWORD PTR SS:[ESP+14]                ; 计数器--
005492B8  ^0F85 61FFFFFF    JNZ MyNotesK.0054921F
005492BE   8B4424 04        MOV EAX,DWORD PTR SS:[ESP+4]
005492C2   8B5424 10        MOV EDX,DWORD PTR SS:[ESP+10]
005492C6   8910             MOV DWORD PTR DS:[EAX],EDX               ; 填充
005492C8   8B4424 04        MOV EAX,DWORD PTR SS:[ESP+4]
005492CC   8B5424 0C        MOV EDX,DWORD PTR SS:[ESP+C]
005492D0   8950 04          MOV DWORD PTR DS:[EAX+4],EDX             ; 填充
005492D3   83C4 18          ADD ESP,18
005492D6   5F               POP EDI
005492D7   5E               POP ESI
005492D8   5B               POP EBX
005492D9   C3               RETN
```

这里的运算，我认为就是变换数组里的值，第一次填充的是常数`0XD9F6`，注册中间值，然后进行复杂运算，变换数组里的值，里面
还用到了2个表第一个是`{0,3,1,2,1,3,1,0,2,3,2,0,3,2,0,1,0,2,2,1,3,0,3,1}`，第2个是`{0x55147626,0x8d0bf107,0xF9492A40,0x2874514A}`
不知道为什么复杂的运算后是明码比较，悲。

## 算法小结

先运算得到注册中间值，然后根据`注册中间值`，和常数`0XD9F6`，进行运算，得到最后的注册码。

`KeyGen` 代码如下`C语言`

```
//为了省事没有对中文注册名进行处理,所以只能输入英文注册名
#include <stdio.h>
#include <math.h>
#include <string.h>
unsigned long  temp[8];
int regcode[8];
unsigned long reg(char name[])    //根据注册名运算得到一个注册中间值,后面运算将用到
{
    int len;
    unsigned long ecx,edx,esi,c;
    len=strlen(name);
    ecx=0;
    edx=0;
    esi=0;
    for(int b=0;b<len;b++)
    {
        c=name[b];
        if(c>=97 && c<=122)
          {c=c-32;             //把注册名中小写的转化为大写
           name[b]=c;}
    }
    for(int a=0;a<len;a++)     //对转化后的注册名进行运算
    {
        ecx=ecx << 4;
        edx=name[a];
        ecx=ecx + edx;
        edx=ecx;
        edx=edx & 0xf0000000;
        if(edx==0)
           {edx=~edx;
            ecx=ecx & edx;}
        else
            {esi=edx;
            esi=esi >> 0x18;
            ecx=ecx ^ esi;
            edx=~edx;
            ecx=ecx & edx;}
    }
   return ecx;                  //返回运算值
}
void DecToHex(unsigned long t)  //dectohex后变换一下字符并输出
{
    unsigned long a;
    long c;
    int i=0;
    int p;
    do
    {
        c=t%16;
        temp[ i]=c;
        i++;
    }while((t/=16)!=0);
    for(i--;i>=0;i--)
    {
        if(temp[ i]>=10&&temp[ i]<=15)//是字母？
            temp[i]=(temp[ i]+55);    //转化为ascii
    }
    for (int w=0;w<8;w++)
    {
        if(w%2==0)                    //根据情况变换
         regcode[w]=temp[w+1];
        else
         regcode[w]=temp[w-1];
    }
    for(int o=0;o<=7;o++)
    {
       if(regcode[o]>=10)              //是字母?
        printf("%c",regcode[o]);
       else
        printf("%d",regcode[o]);
    }
}
main()
{
   char name[100];
   unsigned long a,eax,ebx,edi,ecx,edx;
   unsigned long cs[4]={0x55147626,0x8d0bf107,0xF9492A40,0x2874514A};
   //程序内定16进制值
   unsigned long hh[4]={0x1,0xd9f6,0x0,0x4};
   //自己根据汇编代码定义的数组,第2个元素待填充(先设为0)
   unsigned long zz[]={0,3,1,2,1,3,1,0,2,3,2,0,3,2,0,1,0,2,2,1,3,0,3,1};
   //程序中定义的表
   int b,pp;
   printf("\t\t\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\n");
   printf("\t\t\*          KeyGen For MyNotesKeeper V1.4          \*\n");
   printf("\t\t\*                  Coded By lnn1123               \*\n");
   printf("\t\t\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\n");
   printf("Please input your name:\n");
   gets(name);
   a=reg(name);
   b=1;
   hh[2]=a;
   pp=4;
   edx=0xd9f6;
   int t=0;
   for(int s=0;s<4;s++)    //循环4次
   {
    edx=hh[1];
    eax=hh[0];
    ebx=eax;
    ebx=ebx+ebx;
    ebx=ebx*3;
    if(t==0)               //根据t值对数组访问,可能效率比较低
      eax=zz[ebx*2];
    if(t==3)
      eax=zz[ebx*2+3];
    if(t==6)
      eax=zz[ebx*2+6];
    if(t==9)
      eax=zz[ebx*2+9];
    ecx=cs[eax];
    if(t==0)               //根据t值对数组访问,可能效率比较低
      eax=zz[ebx*2+1];
    if(t==3)
```
