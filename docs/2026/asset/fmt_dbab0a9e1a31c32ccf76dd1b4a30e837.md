
```asm
00466D1A   |.  8BF9 mov edi,ecx
00466D1C   |.  8BF2 mov esi,edx
00466D1E   |.  8BD8 mov ebx,eax
00466D20   |.  8B4D>mov ecx,dword ptr ss:[ebp+14]    ;  固定值表循环（以准备修改开始计算）的第4组数据
00466D23   |.  8BD7 mov edx,edi                      ;  固定值表循环（以准备修改开始计算）的第3组数据
00466D25   |.  8BC6 mov eax,esi                      ;  固定值表循环（以准备修改开始计算）的第2组数据
00466D27   |.  E8 4>call unpacked.00466C78           ;  三个数据XOR 一下即可
00466D2C   |.  0345>add eax,dword ptr ss:[ebp+10]    ;  函数得到的结果加上机器码表的依次第几组数据
00466D2F   |.  0345>add eax,dword ptr ss:[ebp+8]     ;  结果再加上压入的固定值数据
00466D32   |.  0103 add dword ptr ds:[ebx],eax       ;  得到的结果储存的加上固定值表的此次参与运算的数据
00466D34   |.  8BC3 mov eax,ebx                      ;  得到地址存储用
00466D36   |.  8A55>mov dl,byte ptr ss:[ebp+C]       ;  得到压入的固定值参数2位的
00466D39   |.  E8 4>call unpacked.00466C8C           ;  把修改的数据右移20-固定值位，再把修改的数据左移固定值位，最后把两次移动结果进行OR运算直接储存
00466D3E   |.  0133 add dword ptr ds:[ebx],esi       ;  结果还要加上向下循环的第2组数据
00466D40   |.  5F   pop edi
00466D41   |.  5E   pop esi
00466D42   |.  5B   pop ebx
00466D43   |.  5D   pop ebp
00466D44   \.  C2 1>retn 10
```

```asm
00466D47       90   nop
00466D48   /$  55   push ebp
00466D49   |.  8BEC mov ebp,esp
00466D4B   |.  53   push ebx
00466D4C   |.  56   push esi
00466D4D   |.  57   push edi
00466D4E   |.  8BF9 mov edi,ecx
00466D50   |.  8BF2 mov esi,edx
00466D52   |.  8BD8 mov ebx,eax
00466D54   |.  8B4D>mov ecx,dword ptr ss:[ebp+14]    ;  固定值表循环（以准备修改开始计算）的第4组数据
00466D57   |.  8BD7 mov edx,edi                      ;  固定值表循环（以准备修改开始计算）的第3组数据
00466D59   |.  8BC6 mov eax,esi                      ;  固定值表循环（以准备修改开始计算）的第2组数据
00466D5B   |.  E8 2>call unpacked.00466C80           ;  第4位数据取反的结果和第2位数据OR运算的结果和第3位XOR
00466D60   |.  0345>add eax,dword ptr ss:[ebp+10]    ;  函数得到的结果加上机器码表的依次第几组数据
00466D63   |.  0345>add eax,dword ptr ss:[ebp+8]     ;  结果再加上压入的固定值数据
00466D66   |.  0103 add dword ptr ds:[ebx],eax       ;  得到的结果储存的加上固定值表的此次参与运算的数据
00466D68   |.  8BC3 mov eax,ebx                      ;  得到地址存储用
00466D6A   |.  8A55>mov dl,byte ptr ss:[ebp+C]       ;  得到压入的固定值参数2位的
00466D6D   |.  E8 1>call unpacked.00466C8C           ;  把修改的数据右移20-固定值位，再把修改的数据左移固定值位，最后把两次移动结果进行OR运算直接储存
00466D72   |.  0133 add dword ptr ds:[ebx],esi       ;  结果还要加上向下循环的第2组数据
00466D74   |.  5F   pop edi
00466D75   |.  5E   pop esi
00466D76   |.  5B   pop ebx
00466D77   |.  5D   pop ebp
00466D78   \.  C2 1>retn 10
```

```asm
00466C60   /$  23D0 and edx,eax                      ;  第2，3与运算覆盖了第3组数据记为EDX
00466C62   |.  F7D0 not eax                          ;  第2组取反
00466C64   |.  23C8 and ecx,eax                      ;  第2组取反后的数据与第4组与运算记为ECX
00466C66   |.  0BD1 or edx,ecx                       ;  EAX与EDX  OR运算，得到结果
00466C68   |.  8BC2 mov eax,edx
00466C6A   \.  C3   retn
00466C6B       90   nop
00466C6C   /$  23C1 and eax,ecx                      ;  第2，4与运算覆盖了第2组数据记为EAX
00466C6E   |.  F7D1 not ecx                          ;  第4组取反
00466C70   |.  23D1 and edx,ecx                      ;  第4组取反后的数据与第3组与运算记为edx
00466C72   |.  0BC2 or eax,edx                       ;  EAX与EDX  OR运算，得到结果
00466C74   \.  C3   retn
00466C75       8D40>lea eax,dword ptr ds:[eax]
00466C78   /$  33C2 xor eax,edx                      ;  第2，3组数据XOR
00466C7A   |.  33C8 xor ecx,eax                      ;  第2，3组数据XOR的结果再和第4组数据XOR
00466C7C   |.  8BC1 mov eax,ecx
00466C7E   \.  C3   retn
00466C7F       90   nop
00466C80   /$  F7D1 not ecx                          ;  第4位数据取反
00466C82   |.  0BC1 or eax,ecx                       ;  第4位数据取反的结果和第2位数据OR运算
00466C84   |.  33D0 xor edx,eax                      ;  第4位数据取反的结果和第2位数据OR运算的结果和第3位XOR
00466C86   |.  8BC2 mov eax,edx
00466C88   \.  C3   retn
```

还有一个移动的函数不变`00466D6D   |.  E8 1>call unpacked.00466C8C`

```asm
00466C8C   /$  53   push ebx
00466C8D   |.  33C9 xor ecx,ecx
00466C8F   |.  8ACA mov cl,dl                        ;  得到2位固定值
00466C91   |.  51   push ecx
00466C92   |.  B9 2>mov ecx,20
00466C97   |.  5B   pop ebx
00466C98   |.  2BCB sub ecx,ebx                      ;  用20减去2位固定值07
00466C9A   |.  8B18 mov ebx,dword ptr ds:[eax]       ;  取刚改变的固定值表中此次参与值
00466C9C   |.  D3EB shr ebx,cl                       ;  把修改的数据右移20-固定值位
00466C9E   |.  8BCA mov ecx,edx
00466CA0   |.  8B10 mov edx,dword ptr ds:[eax]       ;  又得到数据
00466CA2   |.  D3E2 shl edx,cl                       ;  把修改的数据左移固定值位
00466CA4   |.  0BDA or ebx,edx                       ;  把两次移动后的值进行OR运算
00466CA6   |.  8918 mov dword ptr ds:[eax],ebx       ;  结果直接存储了！！
00466CA8   |.  5B   pop ebx
00466CA9   \.  C3   retn
```

## 算法总结

在固定值表1中有4组数据，分别A、B、C、D，若准备处理A，则用了其他的B、C、D3组数据，先让他们处理（在函数`00466E59   |.  E8 4>call unpacked.00466CAC           ; \unpacked.00466CAC`等）后的结果加上固定值2中的数据，再家上，机器码表中的某个（没有具体的规律）数据，得到的结果再加上固定值表1中的A数据。

再把固定值表1中的数据A右移（32-固定值表3中的值）次，再将固定值表1中的数据A左移固定值表3中的值次，两次移动的结果相OR，这个结果储存在A中，再把A加上固定值表1中的A的向下第2位B，这样完成一次函数调用。

下一次调用，固定值准备处理的输入变成了D了，第2位变成A，第3位变成B，第4位C，也就是转一次。其他，差不多。

一共要有64次函数的调用，里面函数有4个。

得到的数据每2位颠倒，得到一个字符串，（比如一组数据：ABCDEFGH，颠倒之后变成BADCFEHG，也就是由堆栈到内存的过程）

这样一共有4组数据每组8位，共32位，再它转化为字符串，最后把这个字符串整个颠倒过来）

## 注册机原代码

```c
shuchu(jieguo)

unsigned long int jieguo[];

{
static unsigned long int zuizong[4];
unsigned long int zhi=0x0000000f;
int i;
for(i=0;i<4;i++)
{
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(7*4)))>>4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(6*4)))<<4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(5*4)))>>4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(4*4)))<<4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(3*4)))>>4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(2*4)))<<4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(1*4)))>>4);
zuizong[i]=zuizong[i]+((jieguo[i]&(zhi<<(0*4)))<<4);
}
printf("%8lx%8lx%8lx%8lx\n",zuizong[3],zuizong[2],zuizong[1],zuizong[0]);
}

unsigned long int jisuan1(guding,yidong,beijia,jiqi)
unsigned long int guding[],yidong,beijia,jiqi;
{
unsigned long int a,b,c,d,e,f;
a=guding[1]&guding[2];
b=guding[3]&(~guding[1]);
c=a|b;
c=c+jiqi+beijia;
guding[0]=guding[0]+c;
d=guding[0]>>(0x20-yidong);
e=guding[0]<<yidong;
guding[0]=d|e;
guding[0]=guding[0]+guding[1];
f=guding[3];
guding[3]=guding[2];
guding[2]=guding[1];
guding[1]=guding[0];
guding[0]=f;
}

unsigned long int jisuan2(guding,yidong,beijia,jiqi)
unsigned long int guding[],yidong,beijia,jiqi;
{
unsigned long int a,b,c,d,e,f;
a=guding[1]&guding[3];
b=guding[2]&(~guding[3]);
c=a|b;
c=c+jiqi+beijia;
guding[0]=guding[0]+c;
d=guding[0]>>(0x20-yidong);
e=guding[0]<<yidong;
guding[0]=d|e;
guding[0]=guding[0]+guding[1];
f=guding[3];
guding[3]=guding[2];
guding[2]=guding[1];
guding[1]=guding[0];
guding[0]=f;
}

unsigned long int jisuan3(guding,yidong,beijia,jiqi)
unsigned long int guding[],yidong,beijia,jiqi;
{
unsigned long int a,b,c,d,e,f;
a=guding[1]^guding[2];
b=a^guding[3];
c=b;
c=c+jiqi+beijia;
guding[0]=guding[0]+c;
d=guding[0]>>(0x20-yidong);
e=guding[0]<<yidong;
guding[0]=d|e;
guding[0]=guding[0]+guding[1];
f=guding[3];
guding[3]=guding[2];
guding[2]=guding[1];
guding[1]=guding[0];
guding[0]=f;
}

unsigned long int jisuan4(guding,yidong,beijia,jiqi)
unsigned long int guding[],yidong,beijia,jiqi;
{
unsigned long int a,b,c,d,e,f;
a=~guding[3];
b=a|guding[1];
c=b^guding[2];
c=c+jiqi+beijia;
guding[0]=guding[0]+c;
d=guding[0]>>(0x20-yidong);
e=guding[0]<<yidong;
guding[0]=d|e;
guding[0]=guding[0]+guding[1];
f=guding[3];
guding[3]=guding[2];
guding[2]=guding[1];
guding[1]=guding[0];
guding[0]=f;
}

main( )
{
static unsigned long int guding[4]={0x67452301,0xefcdab89,0x98badcfe,0x10325476};
static unsigned long int jieguo[4]={0x67452301,0xefcdab89,0x98badcfe,0x10325476};
static unsigned long int canyu1[64]={
0x07,0x0C,0x11,0x16,
```
