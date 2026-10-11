
# 注册机分析与代码实现

> 代码:
> :004011CE FF75C0  push [ebp-40]
> :004011D1 FF75C4  push [ebp-3C]
> :004011D4 FF75BC  push [ebp-44]
> :004011D7 FF75C8  push [ebp-38]
> :004011DA FF75B4  push [ebp-4C]
> :004011DD FF75B8  push [ebp-48]
> :004011E0 FF75AC  push [ebp-54]
> :004011E3 FF75B0  push [ebp-50]
>
> * Possible StringData Ref from Data Obj ->"%lX%lu-%lu%lX-%lu%lu-%lX%lX"
> :004011E6 6838B44000  push 0040B438
> :004011EB 8D857CFEFFFF lea eax, dword ptr [ebp+FFFFFE7C]
> :004011F1 50  push eax ;结果存在[ebp+FFFFFE7C]
> :004011F2 E88D3D0000  call 00404F84 ;这个 CALL 其实是 wsprintf
> :004011F7 83C428  add esp, 00000028

呵呵，一个`"%lX%lu-%lu%lX-%lu%lu-%lX%lX"`。还记得上次的例子吗，那回是一个`"%lX"`，这回复杂了一些哟。别担心，还是很简单的。上次说了，`"%lX"`是十六进制的大写形式，那么`"%lu"`呢，就是普通的十进制形式啦。再看前面 PUSH 进了一堆参数，这些`[ebp-xx]`的形式都是函数里面的局部变量，在这里把它们以不同的形式表示出来再组合好，就是真正的注册码了。下一步的目标，当然就是看这 8 个变量是如何计算出的啦。记住这几个变量都是什么。从头看：

```
代码:
:004010FB 8B55D4  mov edx, dword ptr [ebp-2C]  ;edx=[ebp-2C]是用户名的长度 n
:004010FE 0155C4  add dword ptr [ebp-3C], edx  ;[ebp-3C]可是一个重要变量,
:00401101 0155C4  add dword ptr [ebp-3C], edx  ;[ebp-3C]=2n,看出来没有
:00401104 8BC2  mov eax, edx
:00401106 83C005  add eax, 00000005
:00401109 8945B8  mov dword ptr [ebp-48], eax  ;[ebp-48]=n+5,也是一个重要变量
:0040110C 33C0  xor eax, eax
:0040110E 8BCF  mov ecx, edi ;edi 是一个常数 64F4F0
:00401110 83C104  add ecx, 00000004
:00401113 894DB4  mov dword ptr [ebp-4C], ecx  ;[ebp-4C]=64f4f4
:00401116 33C9  xor ecx, ecx
:00401118 0155BC  add dword ptr [ebp-44], edx
:0040111B 017DBC  add dword ptr [ebp-44], edi  ;[ebp-44]=64f4f0+n
:0040111E 6BFF03  imul edi, 00000003
:00401121 897DC0  mov dword ptr [ebp-40], edi  ;[ebp-40]=64f4f0*3
```

这几个计算都比较简单的，有 5 个变量已经被搞定了。其中那个 `EDI` 我实在没看明白和我们的输入有什么关系，我改动用户名和注册码它也不会变化，因此我认为这是一个常量，如果不对请高手指正。

```
代码:
:00401126 0FBE8C0544FFFFFF  movsx ecx, byte ptr [ebp+eax-BC];[EBP-BC]是用户名,EAX 作为指针
:0040112E 83F961  cmp ecx, 00000061   
:00401131 7C07  jl 0040113A ;如果小于 61 即'a'就跳转
:00401137 83E920  sub ecx, 00000020 ;如果大于就减 20,对于字母来说是小写转大写

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401131(C)
|
:0040113A 8BF1  mov esi, ecx ;ECX 为用户名第一个字符
:0040113C 03DE  add ebx, esi ;EBX=ECX
:0040113E 0FAFD9  imul ebx, ecx ;实际上 EBX=ECX*ECX
:00401141 4A  dec edx ;EDX 为循环变量减 1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401178(C)
|
:00401142 0FBE8C2F44FFFFFF  movsx ecx, byte ptr [edi+ebp-000000BC];前一个字符
:0040114A 0FBEB42F45FFFFFF  movsx esi, byte ptr [edi+ebp-000000BB];后一个字符
;EDI 也是控制取字符的指针,这里相当于每次取出两个字符,前一个放在 ECX,后一个放在 ESI
:00401152 83F961  cmp ecx, 00000061

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040116C(U)
|
:0040115B 83FE61  cmp esi, 00000061
:0040115E 7D0E  jge 0040116E ;这里对字符进行同样的转换
:00401164 EB0B  jmp 00401171

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401155(C)
|
:00401169 83E920  sub ecx, 00000020
:0040116C EDED  jmp 0040115B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040115E(C)
|
:0040116E 83EE20  sub esi, 00000020

:00401171 47  inc edi ;EDI 这个指针+1
:00401172 03DE  add ebx, esi ;EBX 是累加的结果,再加上后一个字符
:00401174 0FAFD9  imul ebx, ecx ;再乘上前一个字符
:00401177 4A  dec edx     
:00401178 75C8  jne 00401142 ;是否取完?
:0040117A 895DC8  mov dword ptr [ebp-38], ebx ;累加结果存在[ebp-38]
```

这段麻烦一些，看不明白的话看我的注册机代码就清楚了。`![img](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)`来看下面一段：

```
代码:
:0040118F 0FBE840D44FFFFFF  movsx eax, byte ptr [ebp+ecx-000000BC];循环取字符
:00401197 03C1  add eax, ecx ;EAX=每位字符+ECX
:00401199 03D8  add ebx, eax ;累加到 EBX
:0040119B 41  inc ecx ;循环变量递增
:0040119C 3B4DD4  cmp ecx, dword ptr [ebp-2C]
:0040119F 75EE  jne 0040118F ;如果未取完则继续
:004011A1 D1C0  rol eax, 1 ;EAX 左移 1 位
:004011A3 3540E20100  xor eax, 0001E240 ;EAX XOR 1E240
:004011A8 8945B0  mov dword ptr [ebp-50], eax   
```

相信你已经对这个形式很熟悉了吧，`[EBP-BC]`这是用户名，然后用一个 `ECX` 循环递增来取每个字符，再看看：`add eax,ecx` / `add ebx,eax` 好像是把每位字符的值再和字符的位置（`ECX`）累加起来的，呵呵，作者开了个小玩笑。看看下面的操作，都是对 `EAX` 进行的，可是累加的结果是放在 `EBX` 中呀，其实 `EAX` 是用户名的最后一个字符加上用户的长度。要说下的是 `ROL`，这个本来是“滚动”，但因为 `EAX` 肯定很小，最高位为 0，所以在注册机中我简单的用左移 `SHL` 代替了。计算结果放在`[ebp-50]`。

```
代码:
:004011B3 0FBE840D44FFFFFF  movsx eax, byte ptr [ebp+ecx-000000BC]
:004011BB 6BD006  imul edx, eax, 00000006
:004011BE 33C2  xor eax, edx
:004011C0 03D8  add ebx, eax
:004011C2 41  inc ecx
:004011C3 3B4DD4  cmp ecx, dword ptr [ebp-2C]
:004011C6 75EB  jne 004011B3
:004011C8 035DB0  add ebx, dword ptr [ebp-50]
:004011CB 895DAC  mov dword ptr [ebp-54], ebx
```

这一段不写说明了，当作测验，应该看懂了吧，算出结果放在`[ebp-54]`。

至此八个变量都出来了，注册机也容易了吧。

```
代码:
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

void main()
{
  int len,i;
  int EBP_40,EBP_3C,EBP_44,EBP_38,EBP_4C,EBP_48,EBP_54,EBP_50;
  int EDI=0x64F4F0;
  char name[50]={0};
  printf("Please input your name:");
  scanf("%s",name);
  len=strlen(name);
  
  EBP_3C=len*2;
  EBP_48=len+5;
  EBP_4C=EDI+4;
  EBP_44=EDI+len;
  EBP_40=EDI*3;

  EBP_50=((len-1+name[len-1])<<1);
  EBP_50^=0x1E240;
  EBP_54=0;
  for (i=0;i<len;i++)
    EBP_54+=((name[i]*6)^name[i]);
  EBP_54+=EBP_50;

  for (i=0;i<len;i++)
    if (name[i]>='a') name[i]-=0x20;
  EBP_38=name[0]*name[0];
  for (i=1;i<len;i++)
    EBP_38=(EBP_38+name[i])*name[i-1];

  printf("Your password is: %lX%lu-%lu%lX-%lu%lu-%lX%lX\n",
    EBP_50,EBP_54,EBP_48,EBP_4C,EBP_38,EBP_44,EBP_3C,EBP_40);
  printf("KeyGen by RoBa Enjoy Cracking!\n");
}
```

一个可用的注册码：

Name: `RoBa`

Serial: `1E288125744-964F4F4-29089574586616308-812EDED0`
`4dd466d0e1e0cc123afbe504da2462ca`
