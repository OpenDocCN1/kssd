
# 汇编代码功能分析

> 原文：[`www.pediy.com/kssd/pediy12/133251.html`](https://www.pediy.com/kssd/pediy12/133251.html)

几年不整汇编，完全遗忘，重新拿起来看下，不知道以下代码的具体含义，只知道大概意思是每 4 字节做操作，但是具体操作方式不清楚。请帮忙解释下

```
00415659  |> /83BE 2C000200 > cmp     dword ptr [esi+2002C], 4
00415660  |. |75 1F           | jnz     short 00415681
00415662  |. |8B86 28000200   | mov     eax, dword ptr [esi+20028]
00415668  |. |D1E8            | shr     eax, 1
0041566A  |. |69C0 BF070000   | imul    eax, eax, 7BF
00415670  |. |05 FD030000     | add     eax, 3FD
00415675  |. |899E 2C000200   | mov     dword ptr [esi+2002C], ebx
0041567B  |. |8986 28000200   | mov     dword ptr [esi+20028], eax
00415681  |> |8B86 2C000200   | mov     eax, dword ptr [esi+2002C]
00415687  |. |8A9430 280002>  | mov     dl, byte ptr [eax+esi+20028]
0041568E  |. |30540C 10       | xor     byte ptr [esp+ecx+10], dl
00415692  |. |83C0 01         | add     eax, 1
00415695  |. |83C1 01         | add     ecx, 1
00415698  |. |3BCF            | cmp     ecx, edi
0041569A  |. |8986 2C000200   | mov     dword ptr [esi+2002C], eax
004156A0  |.\75 B7           \ jnz     short 00415659
004156A2  |>  8B4E 0C         mov     ecx, dword ptr [esi+C]
```

```c
if(*(esi+200ch) == 4)
{
   eax = *(esi+20028h);
   eax = eax / 2;
   eax = eax * 7bfh;
   eax += 3fdh;
   *(esi+2002ch) = ebx;
   *(esi+20028h) = eax;
}
else {
  eax = *(esi + 2002h);
  dl =*(eax+esi+20028h); //这里取一个字节
  dl ^= *(esp+ecx+10h);//这里异或一个字节
  eax++;
  ecx++;
   if( ecx == edi)
   {
    ecx = *(esi+ch);
   } esle {
    ecx++;
    *(esi+2002ch) = eax;     
   }    
}
```

![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")

```c
DWORD dEsi20028, dEsi2002c=0, dEbx;
byte bEsp10[edi]={..};//定义初始化 3 个变量，一个数组
for(int i=0;i<edi;i++)
{
  if(esi2002c==4)
  {dEsi20028>>1;
  dEsi20028*=0x7bf;
  dEsi20028+=0x3fd;
  dEsi2002c=dEbx;   
  }
  byte bDl=(dEsi20028+dEsi2002c)%0x100;
  bEsp10[i]=bEsp10[i]^bDl;
  dEsi2002c=dEsi20028; 
}
```
