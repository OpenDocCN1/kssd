
# C 库字符串反汇编分析

> 原文：[`www.pediy.com/kssd/pediy12/127475.html`](https://www.pediy.com/kssd/pediy12/127475.html)

## 1、strlen

```
.text:00408150             mov     ecx, [esp+arg_0]
.text:00408154             test    ecx, 3          ; if (str & 3)  11B 检测地址末位是 0 4  8  c
                                                   ; 即判断字符串的起始地址是否以 4 对齐 glibc 中基本的对齐
                                                   ; 策略是 2 * sizeof(size_t)
.text:0040815A             jz      short main_loop ; 如果地址末尾与 3 与运算不为 0
.text:0040815C

.text:0040815C str_misaligned:                     ; CODE XREF: _strlen+19j
.text:0040815C             mov     al, [ecx]       ; 取一个字符
.text:0040815E             inc     ecx             ; 增加串指针
.text:0040815F             test    al, al          ; 比较 al 是否为 0 字符
.text:00408161             jz      short loc_4081A3 ; 是 0 则跳到 lea eax,[ecx-1]可见 strlen()不会包括 0 字符长度的
.text:00408163             test    ecx, 3          ;再次判断下一个地址是否进行了对齐如果已对齐则直接跳到 main_loop 中
.text:00408169             jnz     short str_misaligned ; 取一个字符
.text:0040816B             add     eax, 0

.text:00408170 main_loop:                          ; CODE XREF: _strlen+Aj
                                                   ; _strlen+36j ...
.text:00408170             mov     eax, [ecx]      ; 这里判断 4 字节字符串中是否有 0 字符
.text:00408172             mov     edx, 7EFEFEFFh
.text:00408177             add     edx, eax
.text:00408179             xor     eax, 0FFFFFFFFh
.text:0040817C             xor     eax, edx
.text:0040817E             add     ecx, 4          ; while
                                                   ; {
                                                   ; int x = xxxxx;
                                                   ; int y = x + 0x7efefeff;
                                                   ; x ^= 0xffffffff;
                                                   ; x ^= y;
                                                   ; x & 0x81010100  ==0 表示无 0 字符串，相反则表示发现字符串
.text:00408186             jz      short main_loop
.text:00408188             mov     eax, [ecx-4]    ; 比较第一个字符是否为 0
.text:0040818B             test    al, al
.text:0040818D             jz      short loc_4081C1
.text:0040818F             test    ah, ah          ; 比较第二个字符是否为 0
.text:00408191             jz      short loc_4081B7
.text:00408193             test    eax, 0FF0000h   ; 比较第三个字符是否为 0
.text:00408198             jz      short loc_4081AD
.text:0040819A             test    eax, 0FF000000h ; 比较第 4 个字符是否为 0
.text:0040819F             jz      short loc_4081A3
.text:004081A1             jmp     short main_loop

.text:004081A3 ; ---------------------------------------------------------------------------

.text:004081A3 loc_4081A3:                         ; CODE XREF: _strlen+11j
                                                   ; _strlen+4Fj
.text:004081A3             lea     eax, [ecx-1]
.text:004081A6             mov     ecx, [esp+arg_0]
.text:004081AA             sub     eax, ecx
.text:004081AC             retn

.text:004081AD ; ---------------------------------------------------------------------------

.text:004081AD loc_4081AD:                         ; CODE XREF: _strlen+48j
.text:004081AD             lea     eax, [ecx-2]
.text:004081B0             mov     ecx, [esp+arg_0]
.text:004081B4             sub     eax, ecx
.text:004081B6             retn

.text:004081B7 ; ---------------------------------------------------------------------------

.text:004081B7 loc_4081B7:                         ; CODE XREF: _strlen+41j
.text:004081B7             lea     eax, [ecx-3]    ; 得到 ecx-4 的地址相减即可
.text:004081BA             mov     ecx, [esp+arg_0]
.text:004081BE             sub     eax, ecx
.text:004081C0             retn

.text:004081C1 ; ---------------------------------------------------------------------------

.text:004081C1 loc_4081C1:                         ; CODE XREF: _strlen+3Dj
.text:004081C1             lea     eax, [ecx-4]    ; 得到 ecx-4 的地址
.text:004081C4             mov     ecx, [esp+arg_0] ; 得到字串起始地址
.text:004081C8             sub     eax, ecx        ; 减去地址
.text:004081CA             retn                   ; 返回数值
.text:004081CA _strlen          endp
```

//vc6.0 中的 `strlen()`

关于对齐的问题：

`x86` 如果地址不对齐则影响程序执行速度，在 `x64` 中会引发一个异常，在其它的一些硬件平台比如 `mips` 则会引发 `bus error`

`x86` 汇编编译器中有两个伪指令用于对齐 `even` `align`

`even` 偶对齐

```
if (addr % 2==0) return addr; else return addr + 1;
```

`align num` //num 必须是 2^n 次方

```
if ((num & (num-1))==0)
{
   if (addr %num==0) return addr; else return (addr / num + 1) *num;
}
```

//最后逆向出来的函数如下:

```
int  mystrlen(char *p)
{
  int str = *(int *)p;
  int y = 0x7efefeff;
  int count = -1;        //为了方便操作使用计数器，初始为-1 与字符数组的序号相对应
  while (1)
  {
    y += str;     //这里比较复杂，vc6.0 的 C 库中很多地方都用到了这个判断方式
    str ^= 0xffffffff;
    str ^= y;
    if (str &= 0x81010100)    //用于判断整型中是否有 0 字符
    {
      if ((str>>24)==0) return count;      //0 号字符为 0 没有字符
      else if ((str & 0x00ff0000)==0) return count+1; //1 号字符为 0 有 1 个字符
      else if ((str & 0x0000ff00)==0) return count+2; //2 号字符为 0 有 2 个字符
      else if ((str & 0x000000ff)==0) return count+3; //3 号字符为 0 有 3 个字符
    }
    else
    {
      (int *)p++;
      count += 4;
    }
  }
}
```

//逆向扫描 `0x80` 来判断 `0` 号位置的算法

```
unsigned ZeroByte(int x)
{
  int y = (x & 0x7f7f7f7f) + 0x7f7f7f7f;      //1xxxxxxx
  y = ~(y | x | 0x7f7f7f7f);        //80xxxxxx
  if (y==0) return 4;          //如果 y==0 说明是 4 个字符串
  else if (y>0x0000ffff) return (y>>31) ^ 1;    //如果 y>0x0000ffff 即 0 个或 1 号位为 0 字串，然后右移 31 位就是 0 字串的值，与 1 进行二进制相加 s
                 //如果为 0 表示 0x80 在 0 号位置，即最高位异或得 0，如果为 1 则 0x80 在 2 号位置。
  else return (y>>15) ^ 3;        //同理
}
```

//直接用指针检测

```
size_t mystrlen(char  const *str)
{
  char const *p = str;
  while (*p++);
  return (p - str - 1);
}
```

//vc6.0 的 release 版程序中对于 `strlen()`的调用进行了优化。

```
00401080    57           push edi                 ; 保存 edi 原数据
00401081    BF 20304000  mov edi,api.00403020     ; edi = 字符串起始地址
00401086    83C9 FF      or ecx,FFFFFFFF          ;设置 ecx = 0xffffffff 计数器从-1 开始
00401089    33C0         xor eax,eax            ;eax 清 0
0040108B    F2:AE        repne scas byte ptr es:[edi]      ;ecx!=0 ecx=ecx-1 scas edi 与 al
0040108D    F7D1         not ecx                ;把 ecx 求反得出 repne 操作的次数
0040108F    49           dec ecx                ;ecx-1 减去 0 号字符串(因为 repne 先对计数器进行操作再执行 scas)
```

这个代码很棒，有一种补码的思想在里面，任何有限重复操作都可以用补码表示，对于计算机中的数来说，存储上无符号，它就是一个太极（模），超过模就减去模来表示，逻辑上有正数和负数之分，正数为阳 负数为阴，超过模也是减去模进行操作，负数与模相加用正数表示，总的来说一句话太极 与 阴阳 合称太极。太极为阴为常，阴阳为阳为动态。不要把阴阳看成太极分出来的，实际太极与阴阳组成另一个太极。

//strlen 函数

```
__strlen proc uses ebx esi edi,pstr
   mov edi,pstr
   cld
   mov ecx,0ffffffffh
   xor eax,eax
   repnz scasb
   not ecx
   dec ecx
   mov eax,ecx
   ret
__strlen endp
```

//C++循环控制结构中的前++与后++问题

在 `for` 循环中前++与后++是相同的，但是在 `while` 循环中关系到条件判断 它们还是区别的

//示例代码

```
int main()
{
  int n = 3;
  while (n--)
  {
    cout<<n<<endl;
  }
```

```
/*
               ; CODE XREF: _main+49j
.text:0040158F            mov     eax, [ebp+var_4] ; eax=3
.text:00401592            mov     ecx, [ebp+var_4] ; ecx=3
```



```asm
.text:00401595             sub     ecx, 1           ; ecx=2  

.text:00401598             mov     [ebp+var_4], ecx ; n=2  

.text:0040159B             test    eax, eax         ; eax=3 用 3 作为条件判断 (n--)格式  

.text:0040159D             jz      short loc_4015BB ; eax!=0 zf!=1 跳走  

.text:0040159F             push    offset loc_4010C8  

.text:004015A4             mov     edx, [ebp+var_4]  

.text:004015A7             push    edx  

.text:004015A8             mov     ecx, offset ?cout@std@@3V?$basic_ostream@DU?$char_traits@D@std@@@1@A ; std::basic_ostream<char,std::char_traits<char>> std::cout  

.text:004015AD             call    j_??6?$basic_ostream@DU?$char_traits@D@std@@@std@@QAEAAV01@H@Z ; std::basic_ostream<char,std::char_traits<char>>::operator<<(int)  

.text:004015B2             mov     ecx, eax  

.text:004015B4             call    j_??6?$basic_ostream@DU?$char_traits@D@std@@@std@@QAEAAV01@P6AAAV01@AAV01@@Z@Z ; std::basic_ostream<char,std::char_traits<char>>::operator<<(std::basic_ostream<char,std::char_traits<char>> & ( *)(std::basic_ostream<char,std::char_traits<char>> &))  

.text:004015B9             jmp     short loc_40158F ; 回跳的方式在 c++主要是 goto 和循环结构  
```

```cpp
*/ 

  cout<<"|-------------|"<<endl;  

  n = 3;  

  while (--n)  

  {  

    cout<<n<<endl;  

  }  
```

```asm
/*  

.text:004015E0             mov     eax, [ebp+var_4] ; eax=3  

.text:004015E3             sub     eax, 1           ; eax -= 1;  

.text:004015E6             mov     [ebp+var_4], eax ; 变量 -= 1;  

.text:004015E9             cmp     [ebp+var_4], 0   ; 从 2 开始比较  

.text:004015ED             jz      short loc_40160B  

.text:004015EF             push    offset loc_4010C8  

.text:004015F4             mov     ecx, [ebp+var_4]  

.text:004015F7             push    ecx  

.text:004015F8             mov     ecx, offset ?cout@std@@3V?$basic_ostream@DU?$char_traits@D@std@@@1@A ; std::basic_ostream<char,std::char_traits<char>> std::cout  

.text:004015FD             call    j_??6?$basic_ostream@DU?$char_traits@D@std@@@std@@QAEAAV01@H@Z ; std::basic_ostream<char,std::char_traits<char>>::operator<<(int)  

.text:00401602             mov     ecx, eax  

.text:00401604             call    j_??6?$basic_ostream@DU?$char_traits@D@std@@@std@@QAEAAV01@P6AAAV01@AAV01@@Z@Z ; std::basic_ostream<char,std::char_traits<char>>::operator<<(std::basic_ostream<char,std::char_traits<char>> & ( *)(std::basic_ostream<char,std::char_traits<char>> &))  

.text:00401609             jmp     short loc_4015E0 ; eax=3  

*/  
```

```cpp
  return 0;  

}  
```

`i++`;判断 `i` 的值是否有效，然后再给 `i+1`  

`++i`;判断 `i+1` 后的值  

```cpp
int i = 0;  

while (i++)  

{  

  //不会执行  

}  

while (++i)  

{  

  //会执行  

}  
```

//关于整数表示

  整数表示的方法常用的是原码 反码 补码，以 `char` 类型为例

  `char` 1 byte ，因此有 2⁸ = 256 = 0x100 种表示方法，即 `mod = 0x100`，表示为整数范围即为 `0--255`

`unsigned char` : 0-255 的范围，对于任意 `int` 如果超出 其最大值 255 则执行 `x % mod` 即

```cpp
if (x<0) return;  

else (x>255) return x % 256;  
```

`single char`: 机器并无正负数的概念，正负数只是编码规则中的一条，在机器码中，人为的规定 0 表示正数 1 表示负数。

  1 111 1111 //负数中最大的值 0xff = -1

  1 000 0000 //负数中最小的值 0x80 = -128

  0 111 1111 //正数中最大的值 0x7f = 127

  0 000 0000 //正数中最小的值 0

  反码：符号位不变 各位求反

  对于任意长度 `x` 的整数 `n` 求反为 `n = 2^x - 1 - n`

  `2^x` //`mod`

  `2^x - 1` //`max` 最大值

  `2^x - 1 - n` //即为其反码

  根据上面的原理`_strlen()`中的 `not ecx` `dec ecx` 都是一样的

```cpp
int a = 1;  //mov a,1  

~a;        //mov a,0xffffffff -1 即为 0xfffffffe  
```

  补码：符号位不变各位求反后 + 1，对于任意长度 `x` 的整数 `n` 来说，求补即以最右侧的 1 开始，前面的所有位求反。因此很容易求得一个数最右侧的 1 的位置

```cpp
  unsigned x;  

  cin>>x;  

  bitset<32> bit;  

  bit = x & (-x);  

  cout<<bit<<endl;  

  for (size_t inx=0; inx!=32; ++inx)  

  {  

    if (bit.test(inx))  

    {  

      cout<<inx<<" 为最右侧 1 的位置!"<<endl;  

    }  

  }  
```

  对于任意整数 `x` 其补码求得方式有多种。

  `x = -x = ~x + 1 = ~(x - 1)` IA32 中对应的汇编

  `neg x` //求补

  `not x`

  `inc x` //求反+1

  `dec x`

  `not x` //-1 求反

//C++中的位运算

`>>`//减小 2^n

`<<`//增大 2^n

`~`//求反 同上

`&`//模为 2 的乘法，即遇 0 得 0

`|`//有 1 得 1

`xor` //模为 2 的加法。

2、`strupr`

//vc6.0 中的反汇编代码

```asm
            push    ebp  

.text:004180F1           mov     ebp, esp  

.text:004180F3           sub     esp, 0Ch  

.text:004180F6           mov     [ebp+lpDestStr], 0  

.text:004180FD           cmp     dword ptr ?$S1@?1?_Nomemory@std@@YAXXZ@4EA+90h, 0  

.text:00418104           jnz     short loc_41814D ; 这里应该是一个判断使用哪种方式 LC_TYPE  

.text:00418104                               ; if (xxxxx)  

.text:00418104                               ; {  

.text:00418104                               ;   通过-0x20 或者+0xe0 来实现转换  

.text:00418104                               ; }  

.text:00418104                               ; else  

.text:00418104                               ; {  

.text:00418104                               ;   通过 crtLCMapString 实现转换  

.text:00418104                               ;  

.text:00418104                               ; }  

.text:00418106           mov     eax, [ebp+lpMultiByteStr]  

.text:00418109           mov     [ebp+var_C], eax  

.text:0041810C           jmp     short loc_418117  

.text:0041810E ; -------------------------------------------------------------------------  

.text:0041810E  

.text:0041810E loc_41810E:                   ; CODE XREF: _strupr:loc_418143j  

.text:0041810E           mov     ecx, [ebp+var_C]  

.text:00418111           add     ecx, 1  

.text:00418114           mov     [ebp+var_C], ecx  

.text:00418117  

.text:00418117 loc_418117:                   ; CODE XREF: _strupr+1Cj  

.text:00418117           mov     edx, [ebp+var_C]  

.text:0041811A           movsx   eax, byte ptr [edx]  

.text:0041811D           test    eax, eax  

.text:0041811F           jz      short loc_418145 ; 发现 0 字符即跳转到函数结束处  

.text:00418121           mov     ecx, [ebp+var_C]  

.text:00418124           movsx   edx, byte ptr [ecx]  

.text:00418127           cmp     edx, 61h         ; 小于 0x61 即为不可打印或大写字母继续下一个  

.text:0041812A           jl      short loc_418143  

.text:0041812C           mov     eax, [ebp+var_C]  

.text:0041812F           movsx   ecx, byte ptr [eax]  

.text:00418132           cmp     ecx, 7Ah  

.text:00418135           jg      short loc_418143 ; 大于 0x7a 即为不可打印字符或者小写字母，继续下一个字符  

.text:00418137           mov     edx, [ebp+var_C]  

.text:0041813A           mov     al, [edx]        ; 小写字母 0x61--0x7a 0110 0001B---0111 1010B  

.text:0041813A                               ; 大写字母 0x41--0x5a 0100 0001B---0101 1010B  

.text:0041813A                               ; 即第 5 位大写的为 0 小写的为 1，加 0xe0 1110 0000B 即可将第 5  

.text:0041813A                               ; 位转换成 0  

.text:0041813C           add     al, 0E0h         ; 加 0xe0 与-0x20 是一样的效果  

.text:0041813E           mov     ecx, [ebp+var_C]  

.text:00418141           mov     [ecx], al  

.text:00418143  

.text:00418143 loc_418143:                   ; CODE XREF: _strupr+3Aj  

.text:00418143                               ; _strupr+45j  

.text:00418143           jmp     short loc_41810E  

.text:00418145 ; -------------------------------------------------------------------------  

.text:00418145  

.text:00418145 loc_418145:                   ; CODE XREF: _strupr+2Fj  

.text:00418145           mov     eax, [ebp+lpMultiByteStr]  

.text:00418148           jmp     loc_4181E6  

.text:0041814D ; -------------------------------------------------------------------------  

.text:0041814D  

.text:0041814D loc_41814D:                   ; CODE XREF: _strupr+14j  
```



```
.text:0041814D                     push    1                 ; 直接调用 crtLCMapString 将其转换成大写的

.text:0041814F                     push    0                 ; CodePage

.text:00418151                     push    0                 ; cchDest

.text:00418153                     push    0                 ; lpDestStr

.text:00418155                     push    0FFFFFFFFh        ; cbMultiByte

.text:00418157                     mov     edx, [ebp+lpMultiByteStr]

.text:0041815A                     push    edx               ; lpMultiByteStr

.text:0041815B                     push    200h              ; dwMapFlags

.text:00418160                     mov     eax, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h

.text:00418165                     push    eax               ; Locale

.text:00418166                     call    ___crtLCMapStringA ; int LCMapString(
.text:00418166                                         ;   LCID Locale,       // locale identifier
.text:00418166                                         ;   DWORD dwMapFlags,  // mapping transformation type
.text:00418166                                         ;   LPCTSTR lpSrcStr,  // source string
.text:00418166                                         ;   int cchSrc,        // number of characters in source string
.text:00418166                                         ;   LPTSTR lpDestStr,  // destination buffer
.text:00418166                                         ;   int cchDest        // size of destination buffer
.text:00418166                                         ; );

.text:0041816B                     add     esp, 20h          ; 回守栈帧，这个函数应该是 __cdecl 调用方式

.text:0041816E                     mov     [ebp+cchDest], eax ; 保存返回缓冲区中字符数

.text:00418171                     cmp     [ebp+cchDest], 0

.text:00418175                     jnz     short loc_418179 ; 缓冲区中字符数不为 0 跳走 如果是 0 则跳到缓冲区释放代码中

.text:00418177                     jmp     short loc_4181D5

.text:00418179 ; ---------------------------------------------------------------
.text:00418179
.text:00418179 loc_418179:                      ; CODE XREF: _strupr+85j
.text:00418179                     push    62h

.text:0041817B                     push    offset aStrupr_c ; "strupr.c"

.text:00418180                     push    2

.text:00418182                     mov     ecx, [ebp+cchDest]

.text:00418185                     push    ecx

.text:00418186                     call    __malloc_dbg    ; 分配堆

.text:0041818B                     add     esp, 10h

.text:0041818E                     mov     [ebp+lpDestStr], eax

.text:00418191                     cmp     [ebp+lpDestStr], 0 ; 分配成功继续执行

.text:00418195                     jnz     short loc_418199

.text:00418197                     jmp     short loc_4181D5

.text:00418199 ; ---------------------------------------------------------------
.text:00418199
.text:00418199 loc_418199:                      ; CODE XREF: _strupr+A5j
.text:00418199                     push    1                 ; int

.text:0041819B                     push    0                 ; CodePage

.text:0041819D                     mov     edx, [ebp+cchDest]

.text:004181A0                     push    edx               ; cchDest

.text:004181A1                     mov     eax, [ebp+lpDestStr]

.text:004181A4                     push    eax               ; lpDestStr

.text:004181A5                     push    0FFFFFFFFh        ; cbMultiByte

.text:004181A7                     mov     ecx, [ebp+lpMultiByteStr]

.text:004181AA                     push    ecx               ; lpMultiByteStr

.text:004181AB                     push    200h              ; dwMapFlags

.text:004181B0                     mov     edx, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h

.text:004181B6                     push    edx               ; Locale

.text:004181B7                     call    ___crtLCMapStringA

.text:004181BC                     add     esp, 20h

.text:004181BF                     test    eax, eax

.text:004181C1                     jnz     short loc_4181C5

.text:004181C3                     jmp     short loc_4181D5

.text:004181C5 ; ---------------------------------------------------------------
.text:004181C5
.text:004181C5 loc_4181C5:                      ; CODE XREF: _strupr+D1j
.text:004181C5                     mov     eax, [ebp+lpDestStr]

.text:004181C8                     push    eax               ; Source

.text:004181C9                     mov     ecx, [ebp+lpMultiByteStr]

.text:004181CC                     push    ecx               ; Dest

.text:004181CD                     call    _strcpy

.text:004181D2                     add     esp, 8

.text:004181D5

.text:004181D5 loc_4181D5:                      ; CODE XREF: _strupr+87j
.text:004181D5                                         ; _strupr+A7j ...
.text:004181D5                     push    2

.text:004181D7                     mov     edx, [ebp+lpDestStr] ; 调用 free_dbg 释放堆空间

.text:004181DA                     push    edx

.text:004181DB                     call    __free_dbg

.text:004181E0                     add     esp, 8

.text:004181E3                     mov     eax, [ebp+lpMultiByteStr]

.text:004181E6

.text:004181E6 loc_4181E6:                      ; CODE XREF: _strupr+58j
.text:004181E6                     mov     esp, ebp

.text:004181E8                     pop     ebp

.text:004181E9                     retn

.text:004181E9 _strupr          endp
```

关于一开始的条件判断 `MSDN` 的注释：

`The _strupr function converts, in place, each lowercase letter in string to uppercase. The conversion is determined by the LC_CTYPE`

`category setting of the current locale. Other characters are not affected. For more information on LC_CTYPE, see setlocale.`

`LC_TYPE` 的描述

`The character-handling functions (except isdigit, isxdigit, mbstowcs, and mbtowc, which are unaffected)`

即该函数会根据 `LC_TYPE` 设置的值来判断使用哪种方法。在 `VC6.0` 中，中文的默认使用 `+0xe0` 的方式。

如果使用如下代码：

```
setlocale(LC_CTYPE, "English");
printf("%s\n",strupr("hacker"));    //它将使用第二种 LCMapString 的方式，函数调用受地域影响
```

小结：

`LCMapString()`
`setlocale()`

`//vc6.0 中生成的 debug 版的程序，一般调用该函数都会引发异常，原因是 debug 版程序将字符串保存到了.rdata 区段，而该区段默认为只读，而生成 release 版后，编译器将数据保存到了.data 区段，故问题解除。`

关于字符大小写转换的方法

`//加减法`
`//-0x20`

```
char *mystrupr(char *str)
{
  char *ptemp = str;
  while (*ptemp)
  {
    *ptemp -= 0x20;
    ++ptemp;
  }
  return str;
}
```

`//+0xe0`

```
char *mystrupr(char *str)
{
  char *ptemp = str;
  while (*ptemp)
  {
    *ptemp += 0xe0;
    ++ptemp;
  }
  return str;
}
```

`//and or 运算`
`//小变大`

```
char *mystrupr(char *str)
{
  char *ptemp = str;
  while (*ptemp)
  {
    *ptemp &= 0xdf;
    ++ptemp;
  }
  return str;
}
```

`//大变小`

```
char *mystrupr(char *str)
{
  char *ptemp = str;
  while (*ptemp)
  {
    *ptemp |= 0x20;
    ++ptemp;
  }
  return str;
}
```

这种运算是根据 `ASCII` 的特点来的，小写字母的第 5 位为 1 而大写字母的第 5 位为 0，因此让其第 5 位变 0 即为小变大，相反则大变小。

`//直接用 LCMapString`

3、`strcpy`

```
; char __cdecl strcpy(char *Dest, const char *Source)
.text:00409020     _strcpy          proc near               ; CODE XREF: _main+31p
.text:00409020                                         ; exception::exception(char const * const &)+4Ep ...
.text:00409020
.text:00409020     Dest             = dword ptr  4
.text:00409020     Source           = dword ptr  8
.text:00409020
.text:00409020 000                   push    edi
.text:00409021 004                   mov     edi, [esp+4+Dest] ; 缓冲区地址在 edi 中
.text:00409025 004                   jmp     short copy_start ; 跳转到 strcat()函数的代码中
.text:00409025     _strcpy          endp
.text:00409025
copy_start:                      ; CODE XREF: _strcpy+5j
.text:00409091                                         ; _strcat+52j ...
.text:00409091 004                   mov     ecx, [esp+4+Source]
```



## `.text:00409095` 004  test  ecx, 3  ; 判断 4 字节对齐

```
.text:00409095 004                     test    ecx, 3          ; 判断 4 字节对齐  
.text:0040909B 004                     jz      short main_loop_entrance  
.text:0040909D  
.text:0040909D      src_misaligned:                     ; CODE XREF: _strcat+7Dj  
.text:0040909D 004                     mov     dl, [ecx]  
.text:0040909F 004                     inc     ecx  
.text:004090A0 004                     test    dl, dl  
.text:004090A2 004                     jz      short loc_409108  
.text:004090A4 004                     mov     [edi], dl  
.text:004090A6 004                     inc     edi  
.text:004090A7 004                     test    ecx, 3          ; 继续判断 4 字节对齐  
.text:004090AD 004                     jnz     short src_misaligned  
.text:004090AF 004                     jmp     short main_loop_entrance  
.text:004090B1      ; ---------------------------------------------------------------------------  
.text:004090B1  
.text:004090B1      main_loop:                      ; CODE XREF: _strcat+9Ej  
.text:004090B1                                    ; _strcat+B8j  
.text:004090B1 004                     mov     [edi], edx      ; 这里实现字符复制  
.text:004090B3 004                     add     edi, 4  
.text:004090B6  
.text:004090B6      main_loop_entrance:                ; CODE XREF: _strcat+6Bj  
.text:004090B6                                    ; _strcat+7Fj  
.text:004090B6 004                     mov     edx, 7EFEFEFFh  
.text:004090BB 004                     mov     eax, [ecx]  
.text:004090BD 004                     add     edx, eax  
.text:004090BF 004                     xor     eax, 0FFFFFFFFh  
.text:004090C2 004                     xor     eax, edx  
.text:004090C4 004                     mov     edx, [ecx]  
.text:004090C6 004                     add     ecx, 4  
.text:004090C9 004                     test    eax, 81010100h  ; 快速判断 0 字符  
.text:004090CE 004                     jz      short main_loop ; 如果没有 0 字符跳到复制代码处  
.text:004090D0 004                     test    dl, dl  
.text:004090D2 004                     jz      short loc_409108 ; 判断 0 字符的位置并写入复制缓冲区的末尾  
.text:004090D4 004                     test    dh, dh  
.text:004090D6 004                     jz      short loc_4090FF ; 与 strlen()的算法类似  
.text:004090D8 004                     test    edx, 0FF0000h  
.text:004090DE 004                     jz      short loc_4090F2  
.text:004090E0 004                     test    edx, 0FF000000h  
.text:004090E6 004                     jz      short loc_4090EA  
.text:004090E8 004                     jmp     short main_loop  
.text:004090EA      ; ---------------------------------------------------------------------------  
.text:004090EA  
.text:004090EA      loc_4090EA:                     ; CODE XREF: _strcat+B6j  
.text:004090EA 004                     mov     [edi], edx  
.text:004090EC 004                     mov     eax, [esp+4+Dest]  
.text:004090F0 004                     pop     edi  
.text:004090F1 000                     retn  
.text:004090F2      ; ---------------------------------------------------------------------------  
.text:004090F2  
.text:004090F2      loc_4090F2:                     ; CODE XREF: _strcat+AEj  
.text:004090F2 004                     mov     [edi], dx  
.text:004090F5 004                     mov     eax, [esp+4+Dest]  
.text:004090F9 004                     mov     byte ptr [edi+2], 0  
.text:004090FD 004                     pop     edi  
.text:004090FE 000                     retn  
.text:004090FF      ; ---------------------------------------------------------------------------  
.text:004090FF  
.text:004090FF      loc_4090FF:                     ; CODE XREF: _strcat+A6j  
.text:004090FF 004                     mov     [edi], dx  
.text:00409102 004                     mov     eax, [esp+4+Dest]  
.text:00409106 004                     pop     edi  
.text:00409107 000                     retn  
.text:00409108      ; ---------------------------------------------------------------------------  
.text:00409108  
.text:00409108      loc_409108:                     ; CODE XREF: _strcat+72j  
.text:00409108                                    ; _strcat+A2j  
.text:00409108 004                     mov     [edi], dl       ; edi 末字符串处理  
.text:0040910A 004                     mov     eax, [esp+4+Dest] ; 返回缓冲区地址  
.text:0040910E 004                     pop     edi                 ; 恢复 edi  
.text:0040910F 000                     retn  
.text:0040910F      _strcat         endp  
```

`strcpy()` 跳到了 `strcat()` 函数中的代码。算法也比较简单。

```c++
//c++源码实现(高质量 C++编程中的测试题目 原来看的时候没有写出来.....)

char *mystrcpy(char *dest,char *src,unsigned count)

{

  char *p = dest;

  while (count--)

  {

    *dest++ = *src++;

  }

  return  p;

}
```

## 4、`strcat`

```
.text:00409040 000 8B 4C 24 04                         mov     ecx, [esp+Dest] ; 目标缓冲区  
.text:00409044 000 57                                   push    edi             ; 保存 edi  
.text:00409045 004 F7 C1 03 00 00 00                   test    ecx, 3          ; 判断 4 字节对齐  
.text:0040904B 004 74 0F                                jz      short find_end_of_dest_string_loop ; 取目标缓冲区的 4 字节  
.text:0040904D  
.text:0040904D                 dest_misaligned:                  ; CODE XREF: _strcat+1Aj  
.text:0040904D 004 8A 01                                mov     al, [ecx]  
.text:0040904F 004 41                                   inc     ecx           ; 未对齐处理一个递增地址继续判断是否对齐  
.text:00409050 004 84 C0                                test    al, al  
.text:00409052 004 74 3B                                jz      short start_byte_3  
.text:00409054 004 F7 C1 03 00 00 00                   test    ecx, 3  
.text:0040905A 004 75 F1                                jnz     short dest_misaligned  
.text:0040905C  
.text:0040905C                 find_end_of_dest_string_loop:     ; CODE XREF: _strcat+Bj  
.text:0040905C                                                                  ; _strcat+32j ...  
.text:0040905C 004 8B 01                                mov     eax, [ecx]      ; 取目标缓冲区的 4 字节  
.text:0040905E 004 BA FF FE FE 7E                      mov     edx, 7EFEFEFFh  
.text:00409063 004 03 D0                                add     edx, eax  
.text:00409065 004 83 F0 FF                              xor     eax, 0FFFFFFFFh  
.text:00409068 004 33 C2                                 xor     eax, edx  
.text:0040906A 004 83 C1 04                               add     ecx, 4  
.text:0040906D 004 A9 00 01 01 81                          test    eax, 81010100h  ; 寻找目标缓冲区中的 0 字符位置  
.text:00409072 004 74 E8                                  jz      short find_end_of_dest_string_loop ; 取目标缓冲区的 4 字节  
.text:00409074 004 8B 41 FC                               mov     eax, [ecx-4]    ; 发现 0 字符将字符串地址返回到原始地址  
.text:00409077 004 84 C0                                  test    al, al  
.text:00409079 004 74 23                                  jz      short start_byte_0 ; 定位 0 字符在 4 字节中的位置  
.text:0040907B 004 84 E4                                  test    ah, ah  
.text:0040907D 004 74 1A                                  jz      short start_byte_1  
.text:0040907F 004 A9 00 00 FF 00                          test    eax, 0FF0000h  
.text:00409084 004 74 0E                                  jz      short start_byte_2  
.text:00409086 004 A9 00 00 00 FF                         test    eax, 0FF000000h  
.text:0040908B 004 74 02                                  jz      short start_byte_3  
.text:0040908D 004 EB CD                                  jmp     short find_end_of_dest_string_loop ; 取目标缓冲区的 4 字节
```



```
.text:0040908F                                                          ; ---------------------------------------------------------------------------  

.text:0040908F  

.text:0040908F                                                          start\_byte\_3:                      ; CODE XREF: \_strcat+12j  

.text:0040908F                                                          ; \_strcat+4Bj  

.text:0040908F 004 8D 79 FF                      lea     edi, [ecx-1]  

.text:00409092 004 EB 0D                          jmp     short copy\_start  

.text:00409094                                                          ; ---------------------------------------------------------------------------  

.text:00409094  

.text:00409094                                                          start\_byte\_2:                      ; CODE XREF: \_strcat+44j  

.text:00409094 004 8D 79 FE                      lea     edi, [ecx-2]  

.text:00409097 004 EB 08                          jmp     short copy\_start  

.text:00409099                                                          ; ---------------------------------------------------------------------------  

.text:00409099  

.text:00409099                                                          start\_byte\_1:                      ; CODE XREF: \_strcat+3Dj  

.text:00409099 004 8D 79 FD                      lea     edi, [ecx-3]    ; 纠正复制地址跳到复制起始位置  

.text:0040909C 004 EB 03                          jmp     short copy\_start  

.text:0040909E                                                          ; ---------------------------------------------------------------------------  

.text:0040909E  

.text:0040909E                                                          start\_byte\_0:                      ; CODE XREF: \_strcat+39j  

.text:0040909E 004 8D 79 FC                      lea     edi, [ecx-4]    ; 纠正复制地址到 edi 中  

.text:004090A1  

.text:004090A1                                                          copy\_start:                         ; CODE XREF: \_strcpy+5j  

.text:004090A1                                                          ; \_strcat+52j ...  

.text:004090A1 004 8B 4C 24 0C                   mov     ecx, [esp+4+Source] ; 取待复制字串  

.text:004090A5 004 F7 C1 03 00 00 00             test    ecx, 3          ; 判断对齐  

.text:004090AB 004 74 19                          jz      short main\_loop\_entrance  

.text:004090AD  

.text:004090AD                                                          src\_misaligned:                    ; CODE XREF: \_strcat+7Dj  

.text:004090AD 004 8A 11                             mov     dl, [ecx]  

.text:004090AF 004 41                                inc     ecx  

.text:004090B0 004 84 D2                             test    dl, dl  

.text:004090B2 004 74 64                             jz      short loc\_409118  

.text:004090B4 004 88 17                             mov     [edi], dl  

.text:004090B6 004 47                                inc     edi  

.text:004090B7 004 F7 C1 03 00 00 00             test    ecx, 3  

.text:004090BD 004 75 EE                             jnz     short src\_misaligned  

.text:004090BF 004 EB 05                          jmp     short main\_loop\_entrance  

.text:004090C1                                                          ; ---------------------------------------------------------------------------  

.text:004090C1  

.text:004090C1                                                          main\_loop:                          ; CODE XREF: \_strcat+9Ej  

.text:004090C1                                                          ; \_strcat+B8j  

.text:004090C1 004 89 17                             mov     [edi], edx  

.text:004090C3 004 83 C7 04                          add     edi, 4  

.text:004090C6  

.text:004090C6                                                          main\_loop\_entrance:                  ; CODE XREF: \_strcat+6Bj  

.text:004090C6                                                          ; \_strcat+7Fj  

.text:004090C6 004 BA FF FE FE 7E                    mov     edx, 7EFEFEFFh  ; 与 strcpy 的代码一致  

.text:004090CB 004 8B 01                             mov     eax, [ecx]  

.text:004090CD 004 03 D0                             add     edx, eax  

.text:004090CF 004 83 F0 FF                          xor     eax, 0FFFFFFFFh  

.text:004090D2 004 33 C2                             xor     eax, edx  

.text:004090D4 004 8B 11                             mov     edx, [ecx]  

.text:004090D6 004 83 C1 04                          add     ecx, 4  

.text:004090D9 004 A9 00 01 01 81                    test    eax, 81010100h  

.text:004090DE 004 74 E1                             jz      short main\_loop  

.text:004090E0 004 84 D2                             test    dl, dl  

.text:004090E2 004 74 34                             jz      short loc\_409118  

.text:004090E4 004 84 F6                             test    dh, dh  

.text:004090E6 004 74 27                             jz      short loc\_40910F  

.text:004090E8 004 F7 C2 00 00 FF 00                 test    edx, 0FF0000h  

.text:004090EE 004 74 12                             jz      short loc\_409102  

.text:004090F0 004 F7 C2 00 00 00 FF                 test    edx, 0FF000000h  

.text:004090F6 004 74 02                             jz      short loc\_4090FA  

.text:004090F8 004 EB C7                          jmp     short main\_loop  

.text:004090FA                                                          ; ---------------------------------------------------------------------------  

.text:004090FA  

.text:004090FA                                                          loc\_4090FA:                         ; CODE XREF: \_strcat+B6j  

.text:004090FA 004 89 17                             mov     [edi], edx  

.text:004090FC 004 8B 44 24 08                   mov     eax, [esp+4+Dest]  

.text:00409100 004 5F                                pop     edi  

.text:00409101 000 C3                               retn  

.text:00409102                                                          ; ---------------------------------------------------------------------------  

.text:00409102  

.text:00409102                                                          loc\_409102:                         ; CODE XREF: \_strcat+AEj  

.text:00409102 004 66 89 17                          mov     [edi], dx  

.text:00409105 004 8B 44 24 08                   mov     eax, [esp+4+Dest]  

.text:00409109 004 C6 47 02 00                       mov     byte ptr [edi+2], 0  

.text:0040910D 004 5F                                pop     edi  

.text:0040910E 000 C3                               retn  

.text:0040910F                                                          ; ---------------------------------------------------------------------------  

.text:0040910F  

.text:0040910F                                                          loc\_40910F:                         ; CODE XREF: \_strcat+A6j  

.text:0040910F 004 66 89 17                          mov     [edi], dx  

.text:00409112 004 8B 44 24 08                   mov     eax, [esp+4+Dest]  

.text:00409116 004 5F                                pop     edi  

.text:00409117 000 C3                               retn  

.text:00409118                                                          ; ---------------------------------------------------------------------------  

.text:00409118  

.text:00409118                                                          loc\_409118:                         ; CODE XREF: \_strcat+72j  
```



## _strcat

```
.text:00409118                                                           ; _strcat+A2j  

.text:00409118 004 88 17                                                 mov     [edi], dl  

.text:0040911A 004 8B 44 24 08                                           mov     eax, [esp+4+Dest]  

.text:0040911E 004 5F                                                   pop     edi  

.text:0040911F 000 C3                                                   retn  

.text:0040911F                   _strcat          endp  
```

**小结**：第一步定位要复制的起始位置，第二步定位要复制的字符串并依次复制

**//c++源码**

```c
char *mystrcat(char *dest,char *src,unsigned count)  

{  

  char *p = dest;  

  while (*p!=NULL) p++;  

  while (count--) *p++ = *src++;  

  return dest;  

}  
```

## 5、strchr

```
.text:00420671 CC CC CC CC CC CC CC CC CC CC+                 align 10h  

.text:00420680  

.text:00420680               ; =============== S U B R O U T I N E =======================================  

.text:00420680  

.text:00420680               ; 因为前面的指令 inc edx 所以这里需要-1  

.text:00420680  

.text:00420680               found_bx        proc near                ; CODE XREF: _strchr+1Dj  

.text:00420680 8D 42 FF                              lea     eax, [edx-1]  

.text:00420683 5B                                pop     ebx  

.text:00420684 C3                                retn  

.text:00420684               ; ---------------------------------------------------------------------------  

.text:00420685 8D A4 24 00 00 00 00 8D 64 24+    align 10h  

.text:00420685 00                          found_bx        endp ; sp-analysis failed  

.text:00420685  

.text:00420690  

.text:00420690               ; =============== S U B R O U T I N E =======================================  

.text:00420690  

.text:00420690  

.text:00420690               ; char *__cdecl strchr(const char *Str, int Val)  

.text:00420690               _strchr          proc near                ; CODE XREF: _main+43p  

.text:00420690                                                           ; ___heap_select+161p ...  

.text:00420690  

.text:00420690               Str             = dword ptr  4  

.text:00420690               Val             = dword ptr  8  

.text:00420690  

.text:00420690 33 C0                                xor     eax, eax  

.text:00420692 8A 44 24 08                          mov     al, byte ptr [esp+Val] ; al 为要查询的字符  

.text:00420696  

.text:00420696               ___from_strstr_to_strchr:                ; CODE XREF: _strstr+6Ej  

.text:00420696 53                                push    ebx  

.text:00420697 8B D8                                mov     ebx, eax        ; 保存 eax 中待查询的字符  

.text:00420699 C1 E0 08                             shl     eax, 8          ; 使得 ax=要查询的字符  

.text:0042069C 8B 54 24 08                          mov     edx, [esp+4+Str] ; edx = 要查询的字符串地址  

.text:004206A0 F7 C2 03 00 00 00                     test    edx, 3  

.text:004206A6 74 13                                jz      short main_loop_start ; 判断对齐  

.text:004206A8  

.text:004206A8               str_misaligned:          ; CODE XREF: _strchr+29j  

.text:004206A8 8A 0A                                mov     cl, [edx]  

.text:004206AA 42                                inc     edx  

.text:004206AB 38 D9                                cmp     cl, bl          ; bl 为待查询的字符  

.text:004206AD 74 D1                                jz      short found_bx  ; 因为前面的指令 inc edx 所以这里需要-1  

.text:004206AF 84 C9                                test    cl, cl  

.text:004206B1 74 51                                jz      short retnull_bx ; 跳到结束  

.text:004206B3 F7 C2 03 00 00 00                     test    edx, 3          ; 如果地址+1 后还未对齐则继续循环否则跳到主循环起始位置  

.text:004206B9 75 ED                                jnz     short str_misaligned  

.text:004206BB  

.text:004206BB               main_loop_start:          ; CODE XREF: _strchr+16j  

.text:004206BB 0B D8                                or      ebx, eax        ; bh bl 都等于要查询的字符  

.text:004206BD 57                                push    edi  

.text:004206BE 8B C3                                mov     eax, ebx  

.text:004206C0 C1 E3 10                             shl     ebx, 10h        ; 以查询 a 为例，此时 eax = 61610000  

.text:004206C3 56                                push    esi  

.text:004206C4 0B D8                                or      ebx, eax        ; ebx = 61616161  

.text:004206C6  

.text:004206C6               main_loop:          ; CODE XREF: _strchr+61j  

.text:004206C6                                                           ; _strchr+70j ...  

.text:004206C6 8B 0A                                mov     ecx, [edx]      ; 这个算法好像很神奇取 4 个字符  

.text:004206C8 BF FF FE FE 7E                         mov     edi, 7EFEFEFFh  

.text:004206CD 8B C1                                mov     eax, ecx        ; eax = 4 个字节  

.text:004206CF 8B F7                                mov     esi, edi        ; esi=0x7efefeff  

.text:004206D1 33 CB                                xor     ecx, ebx        ; *(int *)&str[0] ^= 0x61616161(假设查询 a 字符) 表示为 xor_1  

.text:004206D3 03 F0                                add     esi, eax        ; *(int *)&str[0] += 0x7efefeff  

.text:004206D5 03 F9                                add     edi, ecx        ; edi += xor_1  

.text:004206D7 83 F1 FF                              xor     ecx, 0FFFFFFFFh ; xor_1 ^= 0xffffffff  

.text:004206DA 83 F0 FF                              xor     eax, 0FFFFFFFFh ; 0x61616161 ^= 0xffffffff  

.text:004206DD 33 CF                                xor     ecx, edi  

.text:004206DF 33 C6                                xor     eax, esi  

.text:004206E1 83 C2 04                              add     edx, 4  

.text:004206E4 81 E1 00 01 01 81                         and     ecx, 81010100h  ; 根据 ecx 的值来判断是否发现字符  

.text:004206EA 75 1C                                jnz     short chr_is_found  

.text:004206EC 25 00 01 01 81                           and     eax, 81010100h  

.text:004206F1 74 D3                                jz      short main_loop  

.text:004206F3 25 00 01 01 01                           and     eax, 1010100h   ; 结束  

.text:004206F8 75 08                                jnz     short retnull  

.text:004206FA 81 E6 00 00 00 80                         and     esi, 80000000h  

.text:00420700 75 C4                                jnz     short main_loop ; 继续循环  

.text:00420702  

.text:00420702               retnull:          ; CODE XREF: _strchr+68j  

.text:00420702                                                           ; _strchr+81j ...  

.text:00420702 5E                                pop     esi  

.text:00420703 5F                                pop     edi  

.text:00420704  

.text:00420704               retnull_bx:          ; CODE XREF: _strchr+21j  

.text:00420704 5B                                pop     ebx  

.text:00420705 33 C0                                xor     eax, eax  

.text:00420707 C3                                retn  
```



### `chr_is_found`

```
.text:00420708                         ; ---------------------------------------------------------------------------
.text:00420708
.text:00420708                         chr_is_found:                     ; CODE XREF: _strchr+5Aj
.text:00420708 8B 42 FC                   mov     eax, [edx-4]    ; 恢复到 4 字节起始位置
.text:0042070B 38 D8                   cmp     al, bl          ; bl=要查询字符 al 为取出的 4 个符的开始
.text:0042070D 74 36                   jz      short loc_420745 ; 跳到返回
.text:0042070F 84 C0                   test    al, al          ; 比较 al 是否为 0 字符
.text:00420711 74 EF                   jz      short retnull
.text:00420713 38 DC                   cmp     ah, bl          ; 比较 ah 和 al 是否匹配
.text:00420715 74 27                   jz      short loc_42073E
.text:00420717 84 E4                   test    ah, ah          ; 判断 ah 是否为 0 字符
.text:00420719 74 E7                   jz      short retnull
.text:0042071B C1 E8 10                 shr     eax, 10h        ; 右移 16 bit
.text:0042071E 38 D8                   cmp     al, bl          ; 重复上面的操作
.text:00420720 74 15                   jz      short loc_420737
.text:00420722 84 C0                   test    al, al
.text:00420724 74 DC                   jz      short retnull
.text:00420726 38 DC                   cmp     ah, bl
.text:00420728 74 06                   jz      short loc_420730
.text:0042072A 84 E4                   test    ah, ah
.text:0042072C 74 D4                   jz      short retnull
.text:0042072E EB 96                   jmp     short main_loop

.text:00420730                         ; ---------------------------------------------------------------------------
.text:00420730
.text:00420730                         loc_420730:                      ; CODE XREF: _strchr+98j
.text:00420730 5E                       pop     esi
.text:00420731 5F                       pop     edi
.text:00420732 8D 42 FF                 lea     eax, [edx-1]
.text:00420735 5B                       pop     ebx
.text:00420736 C3                       retn

.text:00420737                         ; ---------------------------------------------------------------------------
.text:00420737
.text:00420737                         loc_420737:                      ; CODE XREF: _strchr+90j
.text:00420737 8D 42 FE                 lea     eax, [edx-2]
.text:0042073A 5E                       pop     esi
.text:0042073B 5F                       pop     edi
.text:0042073C 5B                       pop     ebx
.text:0042073D C3                       retn

.text:0042073E                         ; ---------------------------------------------------------------------------
.text:0042073E
.text:0042073E                         loc_42073E:                      ; CODE XREF: _strchr+85j
.text:0042073E 8D 42 FD                 lea     eax, [edx-3]
.text:00420741 5E                       pop     esi
.text:00420742 5F                       pop     edi
.text:00420743 5B                       pop     ebx
.text:00420744 C3                       retn

.text:00420745                         ; ---------------------------------------------------------------------------
.text:00420745
.text:00420745                         loc_420745:                      ; CODE XREF: _strchr+7Dj
.text:00420745 8D 42 FC                 lea     eax, [edx-4]
.text:00420748 5E                       pop     esi
.text:00420749 5F                       pop     edi
.text:0042074A 5B                       pop     ebx
.text:0042074B C3                       retn

.text:0042074B                         _strchr         endp
.text:0042074B
```

`strchr`函数中的单字扩充到4字的方式，因为接下来的运算需要使用该值，所以在反汇编代码中比较复杂。

类似于如下C++代码：

```
int str = 0x61;
int str_ex4 = str | str<<8 | str<<16 | str<<24;
```

在`main_loop_start`部分的代码相当乱。

```
//C++源码
char *mystrchr(char *str, unsigned me)
{
  while (*str!=NULL && *str!=(char)me) ++str;
  if (*str==(char)me) return str;
  else return NULL;
}
```

## 6、`strcmp`

```
.text:00409030 ; int __cdecl strcmp(const char *str1, const char *Str2)
.text:00409030 _strcmp         proc near                 ; CODE XREF: _main+3Dp
.text:00409030                     ; type_info::operator==(type_info const &)+15p ...
.text:00409030
.text:00409030 str1              = dword ptr  4
.text:00409030 Str2              = dword ptr  8
.text:00409030
.text:00409030                     mov     edx, [esp+str1]
.text:00409034                     mov     ecx, [esp+Str2]
.text:00409038                     test    edx, 3        ; 地址对齐判断
.text:0040903E                     jnz     short dopartial ; 未对齐则取 word 进行逐一检测
.text:00409040

.text:00409040 dodwords:                       ; CODE XREF: _strcmp+3Cj
.text:00409040                     ; _strcmp+66j ...
.text:00409040                     mov     eax, [edx]      ; 取 4 个字符
.text:00409042                     cmp     al, [ecx]       ; 比较 al 与 str2 中的字符
.text:00409044                     jnz     short donene    ; 不相等
.text:00409046                     or      al, al
.text:00409048                     jz      short retn_equ  ; 相等
.text:0040904A                     cmp     ah, [ecx+1]     ; ah 与 ecx+1 比较
.text:0040904D                     jnz     short donene    ; cf＝0 或 cf=1 继续比较
.text:0040904F                     or      ah, ah
.text:00409051                     jz      short retn_equ  ; 相等
.text:00409053                     shr     eax, 10h        ; edx 中的 4 个字符的高字节移到低字节中，CF＝末字节的 bit 位
.text:00409056                     cmp     al, [ecx+2]     ; 与上面代码类似比较方式
.text:00409059                     jnz     short donene
.text:0040905B                     or      al, al
.text:0040905D                     jz      short retn_equ
.text:0040905F                     cmp     ah, [ecx+3]
.text:00409062                     jnz     short donene    ; 不相等跳走
.text:00409064                     add     ecx, 4
.text:00409067                     add     edx, 4
.text:0040906A                     or      ah, ah          ; 如果字符串中有 0 字符执行 or 后 zf=1
.text:0040906C                     jnz     short dodwords  ; zf=1 继续比较下一个 dword，如果结束则返回
.text:0040906E                     mov     edi, edi
.text:00409070

.text:00409070 retn_equ:                       ; CODE XREF: _strcmp+18j
.text:00409070                     ; _strcmp+21j ...
.text:00409070                     xor     eax, eax        ; eax 清 0 作为返回值此时字符相等
.text:00409072                     retn

.text:00409072 ; ---------------------------------------------------------------------------
.text:00409073                     align 4
.text:00409074

.text:00409074 donene:                         ; CODE XREF: _strcmp+14j
.text:00409074                     ; _strcmp+1Dj ...
.text:00409074                     sbb     eax, eax        ; 减去 cf 标志位
```



## 汇编代码分析

### `strcmp` 核心逻辑片段

```
.text:00409076                 shl     eax, 1            ; 左移 1 位
.text:00409078                 inc     eax               ; +1 不影响 CF 标志位
.text:00409079                 retn                    ; 如果 cf=1 则 sbb eax,eax 后 eax = ffffffff shl eax,1 后
.text:00409079                                 ; cf = 1 ffffffe inc eax -1 即小于 0 小于字符
.text:00409079                                 ;
.text:00409079                                 ; 如果 cf=0 则 sbb eax,eax 后 eax=0 shal eax,1 后
.text:00409079                                 ; cf = 0 0  inc eax 1 即大于 0
.text:00409079 ; ---------------------------------------------------------------------------
.text:0040907A                 align 4
.text:0040907C
.text:0040907C dopartial:                          ; CODE XREF: _strcmp+Ej
.text:0040907C                 test    edx, 1            ; 只要是偶地址都是 ZF=1
.text:00409082                 jz      short doword
.text:00409084                 mov     al, [edx]
.text:00409086                 inc     edx
.text:00409087                 cmp     al, [ecx]
.text:00409089                 jnz     short donene
.text:0040908B                 inc     ecx
.text:0040908C                 or      al, al
.text:0040908E                 jz      short retn_equ
.text:00409090                 test    edx, 2
.text:00409096                 jz      short dodwords
.text:00409098
.text:00409098 doword:                             ; CODE XREF: _strcmp+52j
.text:00409098                 mov     ax, [edx]         ; 一次取两个字符
.text:0040909B                 add     edx, 2            ; 字符串地址 p+2
.text:0040909E                 cmp     al, [ecx]         ; 比较 al 和 ecx
.text:004090A0                 jnz     short donene    ; 不相等就是大于或小于，CF=1 或 cf=0
.text:004090A2                 or      al, al
.text:004090A4                 jz      short retn_equ  ; 相等
.text:004090A6                 cmp     ah, [ecx+1]     ; 比较 ah 和 ecx+1 即第二个字符
.text:004090A9                 jnz     short donene
.text:004090AB                 or      ah, ah
.text:004090AD                 jz      short retn_equ
.text:004090AF                 add     ecx, 2            ; str2+2;指针递增
.text:004090B2                 jmp     short dodwords
.text:004090B2 _strcmp         endp
```

`strcmp` 的算法第一步看是否对齐，第二步：`word` 处理 已对齐 依次比较 `dword`，涉及两个小问题三值比较 和 0 字符扫描，在 `dword` 比较中，通过 `or` 运算来设置 `ZF` 标志位，三值比较一句话天下大事必做细，天下难事必做于易。

三值比较，始终是两种情况，等于和不等于，当是不等于的情况又是两种情况，大于小于，所以说再复杂的逻辑细化后不过阴阳二字。

```
if ((x>y) - (x<y))   //典型的三值比较
```

### 7、`strcmpi`

```
.text:004180F0 src               = dword ptr  0Ch
.text:004180F0
.text:004180F0                 push    ebp               ; 保存 ebp
.text:004180F1                 mov     ebp, esp          ; 开辟新栈帧
.text:004180F3                 push    edi
.text:004180F4                 push    esi
.text:004180F5                 push    ebx               ; windows 使用的三个寄存器入栈保存
.text:004180F6                 mov     esi, [ebp+src]    ; esi 为 src  edi 为 dst
.text:004180F9                 mov     edi, [ebp+dst]
.text:004180FC                 lea     eax, ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+88h
.text:00418102                 cmp     dword ptr [eax+8], 0 ;  这里与 strupr 类似用于检测 locale 可以通过 setlocale 设置
.text:00418106                 jnz     short notclocale ; 如果为 0 表示 noclocale setlocale(LC_TYPE,"English")
.text:00418108                 mov     al, 0FFh          ; al=0xff
.text:0041810A                 mov     edi, edi
.text:0041810C
.text:0041810C chk_null:                           ; CODE XREF: __strcmpi+28j
.text:0041810C                                 ; __strcmpi+48j
.text:0041810C                 or      al, al              ; 判断 al 是否为 0 字符如果是 zf=1 相反 zf=0
.text:0041810E                 jz      short done_0
.text:00418110                 mov     al, [esi]           ; 取一个字符
.text:00418112                 inc     esi                 ; 地址递增，CF 不变
.text:00418113                 mov     ah, [edi]           ; 取一个字符到 ah
.text:00418115                 inc     edi                 ; 地址递增 CF 不变
.text:00418116                 cmp     ah, al              ; 比较取得两个字符
.text:00418118                 jz      short chk_null  ; 相等则继续值得取到 0 字符跳转到 done_0
.text:0041811A                 sub     al, 41h             ; al -= 0x41;0x41 是大写字符 A 的 ASCII
.text:0041811C                 cmp     al, 1Ah             ; 如果差值比 0x1a 大则为小写相反则为大写
.text:0041811E                 sbb     cl, cl              ; CF=1 则 cl=0xff CF=0 则 cl=0
.text:00418120                 and     cl, 20h             ; cl=0 或 1111 1111 & 0010 0000 = 0x20
.text:00418123                 add     al, cl              ; 如果是小写则加 0 如果是大写则加 0x20
.text:00418125                 add     al, 41h             ; 加上原来的减数即为原始的字符值
.text:00418127                 xchg    ah, al              ; src 与 dst 的字符调换
.text:00418129                 sub     al, 41h
.text:0041812B                 cmp     al, 1Ah
.text:0041812D                 sbb     cl, cl
.text:0041812F                 and     cl, 20h
.text:00418132                 add     al, cl
.text:00418134                 add     al, 41h
.text:00418136                 cmp     al, ah              ; 重复上面的操作
.text:00418138                 jz      short chk_null  ; 相等则继续执行
.text:0041813A                 sbb     al, al              ; cf=0 al = 0  cf=1 al=0xff
.text:0041813C                 sbb     al, 0FFh            ; 设置返回值
.text:0041813E
.text:0041813E done_0:                             ; CODE XREF: __strcmpi+1Ej
.text:0041813E                 movsx   eax, al             ; 如果是 0 字符则执行符号扩展将 al 扩展到 eax
.text:00418141                 jmp     short doret
.text:00418143 ; ---------------------------------------------------------------------------
.text:00418143
.text:00418143 notclocale:                         ; CODE XREF: __strcmpi+16j
.text:00418143                 mov     eax, 0FFh         ; eax = 0xff
.text:00418148                 xor     ebx, ebx          ; ebx 清 0
.text:0041814A                 mov     edi, edi
.text:0041814C
.text:0041814C chk_null2:                          ; CODE XREF: __strcmpi+68j
.text:0041814C                                 ; __strcmpi+80j
.text:0041814C                 or      al, al              ; 判断 0 字符
.text:0041814E                 jz      short doret     ; 这是 0 字符判断，上面如果 al=NULL 则上面的 or 会设置 ZF=1
.text:00418150                 mov     al, [esi]           ; 取一个字符
.text:00418152                 inc     esi                 ; 地址递增 CF 不变
.text:00418153                 mov     bl, [edi]           ; 取 dst 的一个字符
.text:00418155                 inc     edi                 ; dst 地址递增，cf 不变
.text:00418156                 cmp     al, bl              ; 比较取出的两个字符
.text:00418158                 jz      short chk_null2 ; 相等则继续比较
.text:0041815A                 push    eax               ; 保存 al 中的字符
.text:0041815B                 push    ebx               ; bl 作参数调用下面的函数
.text:0041815C                 call    _tolower          ; 调用 tolower 来将字符转换成小写
.text:00418161                 mov     ebx, eax          ; 保存转换后的字符
.text:00418163                 add     esp, 4            ; __cdecl 调用母函数回收栈空间，此时 esp->eax
.text:00418166                 call    _tolower          ; 转换 al 中的字符
.text:0041816B                 add     esp, 4            ; __cdelc 调用回收栈帧
.text:0041816E                 cmp     bl, al              ; 比较转换后的字符
.text:00418170                 jz      short chk_null2 ; 相等则继续比较下一个字符
.text:00418172                 sbb     eax, eax          ; cf = 1 eax=0xffffffff cf = 0 eax = 0
.text:00418174                 sbb     eax, 0FFFFFFFFh ; cf=1 eax=0 cf=0 eax=-1
.text:00418177
.text:00418177 doret:                              ; CODE XREF: __strcmpi+51j
.text:00418177                                 ; __strcmpi+5Ej
.text:00418177                 pop     ebx
```



`pop esi`

`pop edi`

`leave ; 相等则 eax=0 不相等则 eax=-1 即函数返回值`

`retn`

`__strcmpi endp`

### 8、`strcspn`

`; bts 设置 control 中的位，bt 位用于检测到设置的位并保存到 cf`

`; 从而确定找到字符串，用 ecx 作计数器，jae 跳转执行时即为`

`; 寻找到位置时，此时 ECX 即为字符串的位置`

`; Attributes: bp-based frame`

`; void __cdecl strcspn(char *string, char *control)`

`_strcspn proc near ; CODE XREF: _main+27p`
` ; std::num_put<char,std::ostreambuf_iterator<char,std::char_traits<char>>>::_Iput(std::ostreambuf_iterator<char,std::char_traits<char>>,std::ios_base &,char,char *,uint)+318p ...`

`var_24 = dword ptr -24h`
`string = dword ptr 8`
`control = dword ptr 0Ch`

```
push    ebp         ; 保存前栈帧栈基址
mov     ebp, esp    ; 开辟新栈帧
push    esi         ; 保存 esi
xor     eax, eax    ; eax 清 0
push    eax         ; 0x20h 个字节 + push esi 为 0x24 字节
push    eax
push    eax
push    eax
push    eax
push    eax
push    eax         ; 初始化栈空间为 0 用于后面的 bts 指令
mov     edx, [ebp+control] ; edx 为待查字符串地址
lea     ecx, [ecx+0]    ; [ecx+0]->ecx

listnext: ; CODE XREF: _strcspn+1Fj
mov     al, [edx]
or      al, al          ; 判断 0 字符
jz      short listdone  ; 跳到 0 字符处理分支
inc     edx             ; 不相等取下一个字符
bts     [esp+24h+var_24], eax ; cf=va24 的 eax 中的位，然后将 va24 的 eax 表示的 bit 位设置为 1
jmp     short listnext  ; 循环继续
; -------------------------------------------------------------------------

listdone: ; CODE XREF: _strcspn+18j
mov     esi, [ebp+string] ; string 地址入 esi
or      ecx, 0FFFFFFFFh ; ecx=0xffffffff
nop

dstnext: ; CODE XREF: _strcspn+34j
inc     ecx             ; ecx=0 CF 不变
mov     al, [esi]       ; 取一个字符
or      al, al          ; 判断所取字符是否为 0
jz      short dstdone
inc     esi
bt      [esp+24h+var_24], eax ; 把 eax 指定的位放到 CF 里如果 CF=1 说明找到要寻找的字符
jnb     short dstnext

dstdone: ; CODE XREF: _strcspn+2Dj
mov     eax, ecx        ; ecx 即为发现 0 字符的位置
add     esp, 20h        ; 回收栈帧
pop     esi
leave                   ; 返回
retn
_strcspn endp
```

#### 位检测指令整理如下：

指令的格式：`BT/BTC/BTR/BTS Reg/Mem, Reg/Imm ; 80386+`

受影响的标志位：`CF`

`BT`：把指定的位传送给 `CF`；

`BTC`：把指定的位传送给 `CF` 后，还使该位变反；

`BTR`：把指定的位传送给 `CF` 后，还使该位变为 0； `bit test reset`

`BTS`：把指定的位传送给 `CF` 后，还使该位变为 1； `bit test set`

假设 `(AX)=1234H`，分别执行下面指令。

`BT AX, 2 ; 指令执行后，CF=1，(AX)=1234h`

`BTC AX, 6 ; 指令执行后，CF=0，(AX)=1274h`

`BTR AX, 10 ; 指令执行后，CF=0，(AX)=1234h`

`BTS AX, 14 ; 指令执行后，CF=0，(AX)=5234h`

位检测指令类似于线性表操作，第一个操作数为线性表的起始地址，第二个操作数为其索引，指令执行两步，第一步把索引位置的值设置为 `CF`，第二步把索引位置的值进行相应的求反、置 0、置 1 操作。

### 9、`strdup`

```
push    ebp         ; 保存前栈帧的栈基址
mov     ebp, esp    ; 开辟新栈帧
push    ecx         ; 保存 ecx
cmp     [ebp+Source], 0 ; 判断 src 是否为空
jnz     short loc_418FBE
xor     eax, eax    ; 如果 src 为空则直接将 eax 清 0 即返回值为 NULL
jmp     short loc_418FF3
; -------------------------------------------------------------------------

loc_418FBE: ; CODE XREF: __strdup+8j
mov     eax, [ebp+Source] ; eax=str
push    eax             ; Str
call    _strlen         ; 求字符的个数
add     esp, 4          ; __Cdecl 调用母函数回收栈帧,因为是 add 4 所以判断其参数为 1
                        ; 这种方法可以用于判断参数的个数
add     eax, 1          ; strlen()不包括 NULL 字符，所以还得+1
push    eax             ; Size
call    _malloc         ; 动态分配空间
add     esp, 4
mov     [ebp+Dest], eax ; eax 为分配的内存空间保存到 dest 中
cmp     [ebp+Dest], 0   ; 判断是否分配空间成功
jz      short loc_418FF1
mov     ecx, [ebp+Source] ; ecx=src
push    ecx             ; Source
mov     edx, [ebp+Dest]
push    edx             ; Dest
call    _strcpy         ; 调用 strcpy 复制数据
add     esp, 8
jmp     short loc_418FF3
; -------------------------------------------------------------------------

loc_418FF1: ; CODE XREF: __strdup+2Dj
xor     eax, eax

loc_418FF3: ; CODE XREF: __strdup+Cj
            ; __strdup+3Fj
mov     esp, ebp
pop     ebp
retn
__strdup endp
```

### 10、`strlwr`

只转换字符串中的大写字母为小写

```
push    ebp         ; 栈帧操作
mov     ebp, esp
sub     esp, 0Ch    ; 里面应该有至多 3 个变量
mov     [ebp+lpDestStr], 0
cmp     dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h, 0
jnz     short loc_41814D ; 判断 LC_TYPE 不为 0 则通过 LCMapString 实现字符转换与 strupr 类似
mov     eax, [ebp+lpMultiByteStr] ; eax=string
mov     [ebp+var_C], eax ; 地址保存到 var_c 中
jmp     short loc_418117
; -------------------------------------------------------------------------
```



```assembly
.text:0041810E loc_41810E:                         ; CODE XREF: _strlwr:loc_418143j  

.text:0041810E                 mov     ecx, [ebp+var_C] ; ecx 保存地址  

.text:00418111                 add     ecx, 1           ; 地址递增 CF 受影响  

.text:00418114                 mov     [ebp+var_C], ecx ; 将地址再次保存到 var_c 中  

.text:00418117  

.text:00418117 loc_418117:                         ; CODE XREF: _strlwr+1Cj  

.text:00418117                 mov     edx, [ebp+var_C] ; 字符串地址到 edx 中  

.text:0041811A                 movsx   eax, byte ptr [edx] ; 取一个字符符号扩展到 eax 中  

.text:0041811D                 test    eax, eax         ; 比较 eax 是否为 0 字符  

.text:0041811F                 jz      short loc_418145 ; 是 0 字符跳到结束，将 0 字符作为函数返回值结束函数  

.text:00418121                 mov     ecx, [ebp+var_C] ; string 的地址保存到 ecx 中  

.text:00418124                 movsx   edx, byte ptr [ecx] ; 取一个字符 符号扩展到 edx 中  

.text:00418127                 cmp     edx, 41h         ; 与大写字母 0x41 比较  

.text:0041812A                 jl      short loc_418143 ; 小于 0x41 即为非字母  

.text:0041812C                 mov     eax, [ebp+var_C] ; eax 为地址  

.text:0041812F                 movsx   ecx, byte ptr [eax] ; 取一个字符，符号扩展到 ecx 中  

.text:00418132                 cmp     ecx, 5Ah         ; 与大写字母最大值 Z 0x5A 比较  

.text:00418135                 jg      short loc_418143 ; 大于表示为小写字母不用再进行转换  

.text:00418137                 mov     edx, [ebp+var_C] ; edx 为字符串地址  

.text:0041813A                 mov     al, [edx]        ; 小于则表示为大写字母  

.text:0041813C                 add     al, 20h          ; +0x20 转换成小写  

.text:0041813E                 mov     ecx, [ebp+var_C] ; 将转换后的字符替换原来的大写字母  

.text:00418141                 mov     [ecx], al  

.text:00418143  

.text:00418143 loc_418143:                         ; CODE XREF: _strlwr+3Aj  

.text:00418143                                   ; _strlwr+45j  

.text:00418143                 jmp     short loc_41810E  

.text:00418145 ; ---------------------------------------------------------------------------  

.text:00418145  

.text:00418145 loc_418145:                         ; CODE XREF: _strlwr+2Fj  

.text:00418145                 mov     eax, [ebp+lpMultiByteStr]  

.text:00418148                 jmp     loc_4181E6  

.text:0041814D ; ---------------------------------------------------------------------------  

.text:0041814D  

.text:0041814D loc_41814D:                         ; CODE XREF: _strlwr+14j  

.text:0041814D                 push    1            ; int  

.text:0041814F                 push    0            ; CodePage  

.text:00418151                 push    0            ; cchDest  

.text:00418153                 push    0            ; lpDestStr  

.text:00418155                 push    0FFFFFFFFh     ; cbMultiByte  

.text:00418157                 mov     edx, [ebp+lpMultiByteStr]  

.text:0041815A                 push    edx          ; lpMultiByteStr  

.text:0041815B                 push    100h           ; dwMapFlags  

.text:00418160                 mov     eax, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h  

.text:00418165                 push    eax          ; Locale  

.text:00418166                 call    ___crtLCMapStringA  

.text:0041816B                 add     esp, 20h  

.text:0041816E                 mov     [ebp+cchDest], eax  

.text:00418171                 cmp     [ebp+cchDest], 0  

.text:00418175                 jnz     short loc_418179  

.text:00418177                 jmp     short loc_4181D5  

.text:00418179 ; ---------------------------------------------------------------------------  

.text:00418179  

.text:00418179 loc_418179:                         ; CODE XREF: _strlwr+85j  

.text:00418179                 push    64h  

.text:0041817B                 push    offset aStrlwr_c ; "strlwr.c"  

.text:00418180                 push    2  

.text:00418182                 mov     ecx, [ebp+cchDest]  

.text:00418185                 push    ecx  

.text:00418186                 call    __malloc_dbg  

.text:0041818B                 add     esp, 10h  

.text:0041818E                 mov     [ebp+lpDestStr], eax  

.text:00418191                 cmp     [ebp+lpDestStr], 0  

.text:00418195                 jnz     short loc_418199  

.text:00418197                 jmp     short loc_4181D5  

.text:00418199 ; ---------------------------------------------------------------------------  

.text:00418199  

.text:00418199 loc_418199:                         ; CODE XREF: _strlwr+A5j  

.text:00418199                 push    1            ; int  

.text:0041819B                 push    0            ; CodePage  

.text:0041819D                 mov     edx, [ebp+cchDest]  

.text:004181A0                 push    edx          ; cchDest  

.text:004181A1                 mov     eax, [ebp+lpDestStr]  

.text:004181A4                 push    eax          ; lpDestStr  

.text:004181A5                 push    0FFFFFFFFh     ; cbMultiByte  

.text:004181A7                 mov     ecx, [ebp+lpMultiByteStr]  

.text:004181AA                 push    ecx          ; lpMultiByteStr  

.text:004181AB                 push    100h           ; dwMapFlags  

.text:004181B0                 mov     edx, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+90h  

.text:004181B6                 push    edx          ; Locale  

.text:004181B7                 call    ___crtLCMapStringA  

.text:004181BC                 add     esp, 20h  

.text:004181BF                 test    eax, eax  

.text:004181C1                 jnz     short loc_4181C5  

.text:004181C3                 jmp     short loc_4181D5  

.text:004181C5 ; ---------------------------------------------------------------------------  

.text:004181C5  

.text:004181C5 loc_4181C5:                         ; CODE XREF: _strlwr+D1j  

.text:004181C5                 mov     eax, [ebp+lpDestStr]  

.text:004181C8                 push    eax          ; Source  

.text:004181C9                 mov     ecx, [ebp+lpMultiByteStr]  

.text:004181CC                 push    ecx          ; Dest  

.text:004181CD                 call    _strcpy  

.text:004181D2                 add     esp, 8  

.text:004181D5  

.text:004181D5 loc_4181D5:                         ; CODE XREF: _strlwr+87j  

.text:004181D5                                   ; _strlwr+A7j ...  

.text:004181D5                 push    2  

.text:004181D7                 mov     edx, [ebp+lpDestStr]  

.text:004181DA                 push    edx  

.text:004181DB                 call    __free_dbg  

.text:004181E0                 add     esp, 8  

.text:004181E3                 mov     eax, [ebp+lpMultiByteStr]  

.text:004181E6  

.text:004181E6 loc_4181E6:                         ; CODE XREF: _strlwr+58j  

.text:004181E6                 mov     esp, ebp  

.text:004181E8                 pop     ebp  

.text:004181E9                 retn  

.text:004181E9 _strlwr          endp  
```

`strlwr()` 与 `strupr()` 的处理方式类似，用两种方式进行字符串转换。一种通过 `LCMapString`（它传递的标志为 `0x100` 而 `strupr()` 传递的为 `0x200h`）。

11、`strnx` 系列函数类似只是将条件转换成计数器（略）。

12、`strpbrk`

```assembly
.text:00408D90                 push    ebp          ; 保存前栈帧栈基  

.text:00408D91                 mov     ebp, esp     ; 开辟新栈帧  

.text:00408D93                 push    esi          ; 保存 esi  

.text:00408D94                 xor     eax, eax     ; eax 清 0  

.text:00408D96                 push    eax  

.text:00408D97                 push    eax  

.text:00408D98                 push    eax  

.text:00408D99                 push    eax  

.text:00408D9A                 push    eax  

.text:00408D9B                 push    eax  

.text:00408D9C                 push    eax  

.text:00408D9D                 push    eax          ; 初始化栈空间，用于 bt 指令  

.text:00408D9E                 mov     edx, [ebp+control] ; 要查询的字符串地址保存到 edx 中  
```



```
.text:00408DA1             lea     ecx, [ecx+0]    ; [ecx+0] ->Ecx 未知

.text:00408DA4 listnext:                               ; CODE XREF: _strpbrk+1Fj
.text:00408DA4             mov     al, [edx]       ; 取一个字符
.text:00408DA6             or      al, al          ; 如果为 0 字符
.text:00408DA8             jz      short listdone
.text:00408DAA             inc     edx             ; 地址递增 CF 不变
.text:00408DAB             bts     [esp+24h+var_24], eax ; eax 为序号，var_24 为顺序表起始地埴，将 eax 所指的 bit 位设置为 CF
.text:00408DAB                             ; 同时将该位设置为 1
.text:00408DAF             jmp     short listnext  ; 继续下一个字符

.text:00408DB1 listdone:                              ; CODE XREF: _strpbrk+18j
.text:00408DB1             mov     esi, [ebp+string] ; esi=string
.text:00408DB4 dstnext:                                ; CODE XREF: _strpbrk+2Fj
.text:00408DB4             mov     al, [esi]       ; 取第一个字符
.text:00408DB6             or      al, al          ; 判断是否为 0 字符
.text:00408DB8             jz      short dstdone   ; 是 0 则跳到结束位置
.text:00408DBA             inc     esi             ; 递增地址，CF 不变
.text:00408DBB             bt      [esp+24h+var_24], eax ; var_24 为线性表起始位置，eax 为 inx 序号将该序号的值保存到 cf 中
.text:00408DBF             jnb     short dstnext   ; 小于则表示 cf=0 继续循环
.text:00408DC1             lea     eax, [esi-1]    ; 因为上面 inc esi 了，所以如果 CF=1 即表示找到，还需要-1 定位其地址
.text:00408DC4 dstdone:                                ; CODE XREF: _strpbrk+28j
.text:00408DC4             add     esp, 20h        ; 回收栈帧
.text:00408DC7             pop     esi
.text:00408DC8             leave
.text:00408DC9             retn
.text:00408DC9 _strpbrk        endp
```

类似于上面的 `strcspn` 函数

### 13、`strrev`

```
.text:00418FA0             push    ebp             ; 保存前栈帧栈基
.text:00418FA1             mov     ebp, esp        ; 开辟新栈帧
.text:00418FA3             push    edi             ; 保存 esi edi
.text:00418FA4             push    esi
.text:00418FA5             mov     edi, [ebp+string] ; edi =string 地址
.text:00418FA8             mov     edx, edi        ; edx=string 地址
.text:00418FAA             mov     esi, edi        ; esi=string 地址
.text:00418FAC             xor     eax, eax        ; eax 清 0
.text:00418FAE             or      ecx, 0FFFFFFFFh ; ecx=0xffffffff
.text:00418FB1             repne scasb             ; ecx!=0 ecx=ecx-1 scasb esi->al
.text:00418FB3             cmp     ecx, 0FFFFFFFEh ; 只有一个字符且字符为 0 即 string 为 NULL
.text:00418FB6             jz      short done_0
.text:00418FB8             dec     edi             ; edi-1 CF 不变
.text:00418FB9             dec     edi             ; edi-1 cf 不变
.text:00418FBA lupe:                              ; CODE XREF: _strrev+28j
.text:00418FBA             cmp     esi, edi        ; 双向循环
.text:00418FBC             jnb     short done_0    ; 小于的时候就表示已经对调完毕
.text:00418FBE             mov     ah, [esi]       ; 对调字符串
.text:00418FC0             mov     al, [edi]
.text:00418FC2             mov     [esi], al
.text:00418FC4             mov     [edi], ah
.text:00418FC6             inc     esi             ; 递增地址
.text:00418FC7             dec     edi             ; 递减地址
.text:00418FC8             jmp     short lupe

.text:00418FCA done_0:                             ; CODE XREF: _strrev+16j
.text:00418FCA                             ; _strrev+1Cj
.text:00418FCA             mov     eax, edx        ; edx 保存到 aex 中作为函数返回值
.text:00418FCC             pop     esi
.text:00418FCD             pop     edi
.text:00418FCE             leave
.text:00418FCF             retn
.text:00418FCF _strrev         endp
```

字符串是顺序表，实际上就是一个顺序表的逆序排法。

### 14、`strset`

```
.text:00418FB0             push    ebp
.text:00418FB1             mov     ebp, esp
.text:00418FB3             push    edi
.text:00418FB4             mov     edi, [ebp+string] ; edi=string 地址
.text:00418FB7             mov     edx, edi        ; edx=strin 地址
.text:00418FB9             xor     eax, eax        ; eax 清 0
.text:00418FBB             or      ecx, 0FFFFFFFFh ; ecx 为 0xffffffff
.text:00418FBE             repne scasb             ; ecx!=0 ecx=ecx-1 scasb esi,al
.text:00418FC0             inc     ecx             ; 以 hacker 字串为例 ，repne scasb 后 ecx=0xffffff f8
.text:00418FC0                             ; not ecx dec ecx 即为字符个数
.text:00418FC0                             ; inc ecx f9  inc ecx fa not ecx 5 inc eax 6
.text:00418FC0                             ; inc ecx f9  not ecx 1001 0110 即三种算法一样
.text:00418FC0                             ;
.text:00418FC0                             ; not ecx
.text:00418FC0                             ; dec ecx
.text:00418FC0                             ;
.text:00418FC0                             ; inc ecx
.text:00418FC0                             ; not ecx
.text:00418FC0                             ;
.text:00418FC0                             ; inc ecx
.text:00418FC0                             ; inc ecx
.text:00418FC0                             ; neg ecx
.text:00418FC1             inc     ecx
.text:00418FC2             neg     ecx
.text:00418FC4             mov     al, [ebp+val]   ; 要设置的字符到 al
.text:00418FC7             mov     edi, edx        ; edi 保存地址
.text:00418FC9             rep stosb               ; 重复 ecx 的次数将 al->edi 所指向的地址 此时 DF 需=1
.text:00418FCB             mov     eax, edx
.text:00418FCD             pop     edi
.text:00418FCE             leave
.text:00418FCF             retn
.text:00418FCF __strset        endp
```

### 15、`strstr`

```
.text:00409010
.text:00409010             mov     ecx, [esp+SubStr] ; 需要判断的字串到 ecx
.text:00409014             push    edi             ; 保存 edi esi ebx 三个 windows 使用的寄存器
.text:00409015             push    ebx
.text:00409016             push    esi
.text:00409017             mov     dl, [ecx]       ; 取一个字符
.text:00409019             mov     edi, [esp+0Ch+Str] ; 要查询的字符地址->edi
.text:0040901D             test    dl, dl          ; 判断 dl 是否为 0 字符
.text:0040901F             jz      short empty_str2 ; 如果 substr==NULL 则直接返回 str 的地址
.text:00409021             mov     dh, [ecx+1]     ; 取下一个字符
.text:00409024             test    dh, dh          ; 比较字符是否 NULL
.text:00409026             jz      short strchr_call ; 如果为 null 表示 substr 只有一个字符，直接使用 strchr 函数的代码即可
.text:00409028 findnext:                              ; CODE XREF: _strstr+52j
.text:00409028                             ; _strstr+65j
.text:00409028             mov     esi, edi        ; 要查询的字符串地址入 esi
.text:0040902A             mov     ecx, [esp+0Ch+SubStr] ; ecx=待查询的字符串
.text:0040902E             mov     al, [edi]       ; 取 str 中的字符
.text:00409030             inc     esi             ; 地址递增
.text:00409031             cmp     al, dl          ; 比较第一个字符如果相等则 str 中存在 substr 中的第一个字符
.text:00409033             jz      short first_char_found
.text:00409035             test    al, al          ; str 为空时返回 0
```



```
.text:00409037             jz      short not_found

.text:00409039

.text:00409039 loop_start:                       ; CODE XREF: _strstr+32j

.text:00409039             mov     al, [esi]

.text:0040903B             inc     esi

.text:0040903C

.text:0040903C in_loop:                           ; CODE XREF: _strstr+3Fj

.text:0040903C             cmp     al, dl

.text:0040903E             jz      short first_char_found

.text:00409040             test    al, al

.text:00409042             jnz     short loop_start

.text:00409044

.text:00409044 not_found:                         ; CODE XREF: _strstr+27j

.text:00409044             pop     esi

.text:00409045             pop     ebx

.text:00409046             pop     edi

.text:00409047             xor     eax, eax        ; eax=0 作为返回地址

.text:00409049             retn

.text:0040904A ; ---------------------------------------------------------------------------

.text:0040904A

.text:0040904A first_char_found:                  ; CODE XREF: _strstr+23j

.text:0040904A                                             ; _strstr+2Ej

.text:0040904A             mov     al, [esi]         ; 取 str 的第二个字符到 al

.text:0040904C             inc     esi             ; 继续增加 str 的地址 CF 不变

.text:0040904D             cmp     al, dh            ; 比较 str 中的第二个字符和 substr 中的第二个字符

.text:0040904F             jnz     short in_loop    ; 相等则将 str 第一个字符的地址保存到 edi 中

.text:00409051

.text:00409051 two_first_chars_equal:             ; str 第一个字符的地址保存到 edi 中

.text:00409051             lea     edi, [esi-1]

.text:00409054

.text:00409054 compare_loop:                      ; CODE XREF: _strstr+63j

.text:00409054             mov     ah, [ecx+2]     ; substr+2 的字符即第三个字符保存到 ah 中

.text:00409057             test    ah, ah          ; 判断是否为 null 字符

.text:00409059             jz      short match     ; 如果第三个字符为 null 则表示已经找到匹配的字符地址

.text:0040905B             mov     al, [esi]       ; 取 str 中的第三个字符

.text:0040905D             add     esi, 2          ; 地址+2 影响 CF 标志位

.text:00409060             cmp     al, ah          ; 比较 al 和 ah 是否相等

.text:00409062             jnz     short findnext  ; 不相等则继续循环查找

.text:00409064             mov     al, [ecx+3]     ; 取第 4 个字符

.text:00409067             test    al, al          ; 判断是否为 0 字符

.text:00409069             jz      short match     ; 是 0 字符表示定位到了返回地址

.text:0040906B             mov     ah, [esi-1]     ; 不是空字符取 str 中的第 4 个字符

.text:0040906E             add     ecx, 2          ; 将 substr +2

.text:00409071             cmp     al, ah          ; 再次比较

.text:00409073             jz      short compare_loop ; 相等则继续

.text:00409075             jmp     short findnext  ; 不相等则表示 substr 和 str 不匹配跳回继续查询 str

.text:00409077 ; ---------------------------------------------------------------------------

.text:00409077

.text:00409077 strchr_call:                       ; CODE XREF: _strstr+16j

.text:00409077             xor     eax, eax

.text:00409079             pop     esi

.text:0040907A             pop     ebx

.text:0040907B             pop     edi

.text:0040907C             mov     al, dl

.text:0040907E             jmp     ___from_strstr_to_strchr ; strchr()函数的代码中

.text:00409083 ; ---------------------------------------------------------------------------

.text:00409083

.text:00409083 match:                             ; CODE XREF: _strstr+49j

.text:00409083                                             ; _strstr+59j

.text:00409083             lea     eax, [edi-1]    ; edi-1 为比较的 str 的起始地址，作为函数返回值

.text:00409086             pop     esi

.text:00409087             pop     ebx

.text:00409088             pop     edi             ; 恢复栈帧

.text:00409089             retn                    ; 返回

.text:0040908A ; ---------------------------------------------------------------------------

.text:0040908A

.text:0040908A empty_str2:                        ; CODE XREF: _strstr+Fj

.text:0040908A             mov     eax, edi        ; edi->eax 作返回值结束函数

.text:0040908C             pop     esi

.text:0040908D             pop     ebx

.text:0040908E             pop     edi

.text:0040908F             retn

.text:0040908F _strstr         endp
```

## 16、`strtok`

```
; //测试源码

.text:00409070 ;         char str[] = "i will fuck u!";

.text:00409070 ;         char delim[] = " ";

.text:00409070 ;         char *p = strtok(str,delim);

.text:00409070 ;

.text:00409070 ;        while( p != NULL )

.text:00409070 ;         {

.text:00409070 ;           cout<<p<<endl;

.text:00409070 ;           p = strtok( NULL, delim);

.text:00409070 ;         }

.text:00409070 ;

.text:00409070 ;

.text:00409070 ; Attributes: bp-based frame

.text:00409070

.text:00409070 ; char **__cdecl strtok(char *Str, const char *Delim)

.text:00409070 _strtok         proc near                   ; CODE XREF: _main+58p

.text:00409070                                             ; _main+8Cp

.text:00409070

.text:00409070 var_Delim_addr  = dword ptr -2Ch

.text:00409070 sqlist_0x20     = byte ptr -28h

.text:00409070 var_8           = dword ptr -8

.text:00409070 var_4           = dword ptr -4

.text:00409070 Str             = dword ptr  8

.text:00409070 Delim           = dword ptr  0Ch

.text:00409070

.text:00409070             push    ebp

.text:00409071             mov     ebp, esp

.text:00409073             sub     esp, 2Ch        ; 0x2c / 4 = 0xb 个栈空间 var_28 -var_8 - 0x20 个空间（这里应该是一个顺序表）

.text:00409076             mov     eax, [ebp+Delim] ; delim->eax

.text:00409079             mov     [ebp+var_Delim_addr], eax ; delim->var_2c

.text:0040907C             mov     [ebp+var_8], 0  ; var_8=0

.text:00409083             jmp     short loc_40908E ; 跳到初始化 var_28 的位置

.text:00409085 ; ---------------------------------------------------------------------------

.text:00409085

.text:00409085 loc_409085:                         ; CODE XREF: _strtok+2Cj

.text:00409085             mov     ecx, [ebp+var_8]

.text:00409088             add     ecx, 1          ; 递增操作数 var_8

.text:0040908B             mov     [ebp+var_8], ecx

.text:0040908E

.text:0040908E loc_40908E:                         ; CODE XREF: _strtok+13j

.text:0040908E             cmp     [ebp+var_8], 20h ; //整体源码

.text:0040908E                                             ; for (size_t inx=0; inx!=0x20; ++inx)

.text:0040908E                                             ; {

.text:0040908E                                             ;      var_28[inx] = 0;

.text:0040908E                                             ; }

.text:00409092             jge     short loc_40909E ; 初始化 0x20 byte

.text:00409094             mov     edx, [ebp+var_8]

.text:00409097             mov     [ebp+edx+sqlist_0x20], 0 ; 注意寻址 ebp + edx + var_28 并没有 edx * x 所以应该是一个 byte 类型的变量

.text:0040909C             jmp     short loc_409085 ; 回跳这里是个循环结构

.text:0040909E ; ---------------------------------------------------------------------------

.text:0040909E

.text:0040909E loc_40909E:                         ; CODE XREF: _strtok+22j

.text:0040909E                                             ; _strtok+71j

.text:0040909E             mov     eax, [ebp+var_Delim_addr] ; delim 地址入 eax

.text:004090A1             xor     ecx, ecx        ; ecx=0

.text:004090A3             mov     cl, [eax]        ; 取 delim 串中的字符

.text:004090A5             mov     edx, ecx        ; delim 字符->edx
```



```
.text:004090A7                 sar     edx, 3          ; *delim = *delim>>3 缩小 2³=8 倍  
.text:004090AA                 mov     eax, [ebp+var_Delim_addr] ; eax = delim 地址  
.text:004090AD                 xor     ecx, ecx        ; ecx = 0  
.text:004090AF                 mov     cl, [eax]       ; 取 delim 字符  
.text:004090B1                 and     ecx, 7          ; *delim & 7   7 = 0111B 即取字符的低 3 位值  
.text:004090B4                 mov     eax, 1          ; eax=1  
.text:004090B9                 shl     eax, cl         ; 1<< (*delim &7) 取 delim 字符的低 3 位并左移 1 位扩展位半字节  
.text:004090BB                 mov     cl, [ebp+edx+sqlist_0x20] ; edx = *delim>>3 作为寻址索引  
.text:004090BF                 or      cl, al          ; 即为表达式：sqlist_0x20[*delim>>3] |= 1<< (*delim &7);  
.text:004090C1                 mov     edx, [ebp+var_Delim_addr] ; 取 delim 字符地址到 edx  
.text:004090C4                 xor     eax, eax        ; eax=0  
.text:004090C6                 mov     al, [edx]       ; al=delim 字符  
.text:004090C8                 sar     eax, 3          ; delim 字符缩小 2³ = 8 倍  
.text:004090CB                 mov     [ebp+eax+sqlist_0x20], cl ; 保存 1<<(*delim & 7)的字串值  
.text:004090CF                 mov     ecx, [ebp+var_Delim_addr] ; ecx = delim 字串的地址  
.text:004090D2                 xor     edx, edx        ; edx = 0  
.text:004090D4                 mov     dl, [ecx]       ; dl=delim 字符  
.text:004090D6                 mov     eax, [ebp+var_Delim_addr] ; eax=delim 字串地址  
.text:004090D9                 add     eax, 1          ; delim 字串地址加 1 此指令影响 CF 标志位  
.text:004090DC                 mov     [ebp+var_Delim_addr], eax ; +1 后的 delim 地址保存 var_delim_addr 变量中  
.text:004090DF                 test    edx, edx        ; 判断 delim 字符是否为空  
.text:004090E1                 jnz     short loc_40909E ; 不为空则继续循环设置 sqlist_0x20（这应该是一个字节域，用于检测标志）  
.text:004090E3                 cmp     [ebp+Str], 0    ; 检测参数 str 是否为 0  
.text:004090E7                 jz      short loc_4090F1 ; 如果 str 传递的是 NULL 则跳走  
.text:004090E9                 mov     ecx, [ebp+Str]  ; ecx = str 地址  
.text:004090EC                 mov     [ebp+var_4], ecx ; str 地址保存到 var_4 中  
.text:004090EF                 jmp     short loc_4090FA  

.text:004090F1 ; ---------------------------------------------------------------------------  

.text:004090F1  

.text:004090F1 loc_4090F1:                          ; CODE XREF: _strtok+77j  
.text:004090F1                 mov     edx, dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+20h  
.text:004090F7                 mov     [ebp+var_4], edx  

.text:004090FA  

.text:004090FA loc_4090FA:                          ; CODE XREF: _strtok+7Fj  
.text:004090FA                              ; _strtok+C5j  
.text:004090FA                 mov     eax, [ebp+var_4] ; eax=str 地址  
.text:004090FD                 xor     ecx, ecx        ; ecx=0  
.text:004090FF                 mov     cl, [eax]       ; cl=str 中的字符  
.text:00409101                 sar     ecx, 3          ; *str >>3  
.text:00409104                 xor     edx, edx        ; edx=0  
.text:00409106                 mov     dl, [ebp+ecx+sqlist_0x20] ; 用*str>>3 作索引搜索 sqlist_0x20 即如果*str 与*delim 中的字符  
.text:00409106                              ; 相同，那么所访问的索引是致的  
.text:0040910A                 mov     eax, [ebp+var_4] ; eax=str 地址  
.text:0040910D                 xor     ecx, ecx        ; ecx=0  
.text:0040910F                 mov     cl, [eax]       ; *str 中的字符  
.text:00409111                 and     ecx, 7          ; 1<< (*str & 7); 这里的代码与上面对 delim 字符的处理是一样的  
.text:00409114                 mov     eax, 1  
.text:00409119                 shl     eax, cl  
.text:0040911B                 and     edx, eax        ; 到这里大体上的思路可以理解了。  
.text:0040911B                              ; 1：建立一个 byte 域，全部初始化为 0  
.text:0040911B                              ; 2: 然后用 *delim>>3 缩小 8 倍计算索引，char 字符为 8bit 即模为  
.text:0040911B                              ; 2⁸ / 0x20 = 0x8 要使用 0x20 个标志只能将其缩小 8 倍，通过  
.text:0040911B                              ; 1<<(*delim & 7)，对于 ascii 其低 3 位一定不会全为 0（NULL 除  
.text:0040911B                              ; 外），用它设置 byte 域可以保证标志为 1。  
.text:0040911B                              ; 3: 然后对待查询的串*str 进行相同的算法操作来对比 byte 域  
.text:0040911D                 test    edx, edx        ; 如果不是 delim 则查询的标志为 0（这里还有一点）  
.text:0040911D                              ; 0x00---0x08 //这里的 byte 域所引是一样的。  
.text:0040911D                              ; 0x08---0x16 //byte 域是一样的所以下面还得进行判断  
.text:0040911F                 jz      short loc_409137 ; 如果是 0 则表示不是 delim 中的字符则跳走，如果不是 0 说明是 delim 中的字符继续循环  
.text:0040911F                              ; 直到不是 delim 中的字符  
.text:00409121                 mov     ecx, [ebp+var_4] ; ecx=str 地址  
.text:00409124                 xor     edx, edx        ; edx=0  
.text:00409126                 mov     dl, [ecx]       ; dl=str 字符  
.text:00409128                 test    edx, edx        ; 判断*str 是否为空字符  
.text:0040912A                 jz      short loc_409137 ; 是也跳走与上面的跳转是同一地址说明应该是一个  
.text:0040912A                              ; (xxxx & xxxx)的 C 语言条件判断  
.text:0040912C                 mov     eax, [ebp+var_4]  
.text:0040912F                 add     eax, 1          ; ++str;  
.text:00409132                 mov     [ebp+var_4], eax ; 保存 str 的地址  
.text:00409135                 jmp     short loc_4090FA ; 回跳说明是一个 while (xxxx & xxxxx)的条件判断  

.text:00409137 ; ---------------------------------------------------------------------------  

.text:00409137  

.text:00409137 loc_409137:                          ; CODE XREF: _strtok+AFj  
.text:00409137                              ; _strtok+BAj  
.text:00409137                 mov     ecx, [ebp+var_4] ; ecx=str 地址  
.text:0040913A                 mov     [ebp+Str], ecx  ; 判断后的 str 地址 保存到 str 中  
.text:0040913D                 jmp     short loc_409148  

.text:0040913F ; ---------------------------------------------------------------------------  

.text:0040913F  

.text:0040913F loc_40913F:                          ; CODE XREF: _strtok:loc_40918Dj  
.text:0040913F                 mov     edx, [ebp+var_4]  
.text:00409142                 add     edx, 1  
.text:00409145                 mov     [ebp+var_4], edx  

.text:00409148  

.text:00409148 loc_409148:                          ; CODE XREF: _strtok+CDj  
.text:00409148                 mov     eax, [ebp+var_4] ; 取 str 判断后的地址  
.text:0040914B                 xor     ecx, ecx        ; ecx=0  
.text:0040914D                 mov     cl, [eax]       ; 取 str 字符  
.text:0040914F                 test    ecx, ecx        ; 判断是否为空字符  
.text:00409151                 jz      short loc_40918F ; 如果为 str 中的空字符跳走  
.text:00409153                 mov     edx, [ebp+var_4] ; str 不是空字符继续执行 edx=操作后的 str 地址  
.text:00409156                 xor     eax, eax        ; eax=0  
.text:00409158                 mov     al, [edx]       ; 取一个字符  
.text:0040915A                 sar     eax, 3          ; *str >> 3  
.text:0040915D                 xor     ecx, ecx        ; ecx=0  
.text:0040915F                 mov     cl, [ebp+eax+sqlist_0x20] ; eax 作为索引即还是*str>>3 作为索引  
.text:00409163                 mov     edx, ecx        ; edx=ecx  
.text:00409165                 mov     eax, [ebp+var_4] ; eax=str 地址  
.text:00409168                 xor     ecx, ecx        ; ecx=0  
.text:0040916A                 mov     cl, [eax]       ; 相同的操作 1<<(*str & 7)  
.text:0040916C                 and     ecx, 7  
.text:0040916F                 mov     eax, 1  
.text:00409174                 shl     eax, cl  
.text:00409176                 and     edx, eax  
```



# 字符串处理汇编分析

```
.text:00409178                      test    edx, edx        ; 判断 byte 标志域  
.text:0040917A                      jz      short loc_40918D ; 相等说明不是 delim 继续循环  
.text:0040917C                      mov     ecx, [ebp+var_4] ; 不相等说明找到 delim 字符  
.text:0040917F                      mov     byte ptr [ecx], 0 ; 将与 delim 字符相同的位置设置为空字符  
.text:00409182                      mov     edx, [ebp+var_4] ; 把地址保存到 edx 中  
.text:00409185                      add     edx, 1           ; edx+1  
.text:00409188                      mov     [ebp+var_4], edx ; var_4 += 1;  
.text:0040918B                      jmp     short loc_40918F ; 跳转到返回代码处  
.text:0040918D ; ---------------------------------------------------------------------------  
.text:0040918D  
.text:0040918D loc_40918D:                            ; CODE XREF: _strtok+10Aj  
.text:0040918D                      jmp     short loc_40913F  
.text:0040918F ; ---------------------------------------------------------------------------  
.text:0040918F  
.text:0040918F loc_40918F:                            ; CODE XREF: _strtok+E1j  
.text:0040918F                                      ; _strtok+11Bj  
.text:0040918F                      mov     eax, [ebp+var_4] ; eax= str 查询完后的地址  
.text:00409192                      mov     dword ptr ?$S1@?1??_Nomemory@std@@YAXXZ@4EA+20h, eax ; 保存 eax，变量位置（？）  
.text:00409197                      mov     ecx, [ebp+Str]  ; ecx = str  
.text:0040919A                      cmp     ecx, [ebp+var_4] ; 判断 ecx 和 str 的地址  
.text:0040919D                      jnz     short loc_4091A3 ; 不相等则跳走  
.text:0040919F                      xor     eax, eax  
.text:004091A1                      jmp     short loc_4091A6 ; 如果相等则 eax=NULL 作为返回值返回  
.text:004091A3 ; ---------------------------------------------------------------------------  
.text:004091A3  
.text:004091A3 loc_004091A3:                           ; CODE XREF: _strtok+12Dj  
.text:004091A3                      mov     eax, [ebp+Str]  ; str->eax 作为返回值  
.text:004091A6  
.text:004091A6 loc_4091A6:                            ; CODE XREF: _strtok+131j  
.text:004091A6                      mov     esp, ebp  
.text:004091A8                      pop     ebp  
.text:004091A9                      retn  
.text:004091A9 _strtok          endp  
```

这个函数可能是所有 C 库字符串函数中最复杂的一个，如下几点值得学习：

A 取位 `&7` 对于二进制位的设置  
`and` 用于置 0，`or` 用于置 1 比较方便

B 位域是用位作为条件标志，该函数是将字节作为条件标志

C `*str>>3` 缩小 8 倍的方法比较有意思。根据 `2⁸` `char` 类型的排列组合数结合 32 位计算机来使用顺序表。

## 17. setmem

原型：`extern void setmem(void *buf, unsigned int count, char ch);`

用法：`#include <string.h>`

功能：把 `buf` 所指内存区域前 `count` 个字节设置成字符 `ch`。

在 `vc` 环境中好像没有，`msdn` 中也没有找到该函数。

## 18. movmem

`char` 类型本身主要考虑到内存的基本单位字节而引入，因此对于字符串函数中有一些内存操作函数。

在 `vc` 中没有这个函数，`msdn` 中有一个：

```
VOID MoveMemory (
  PVOID Destination,   // move destination
  CONST VOID *Source,  // block to move
  SIZE_T Length        // size of block to move
);
```

## 19. memset

```
.text:00408160 ; 函数逆向要解决的问题：
.text:00408160 ; 1:返回值  进入函数后寻找 retn 指令及 eax 中的值
.text:00408160 ; 2:参数个数的判断 IDA 中参数以 arg_ 为前缘标出，如果是
.text:00408160 ; __cdecl 则可以根据 call 后 add 指令判断栈中参数的个数，
.text:00408160 ; __stdcall 可以参考 retn 上面的回收栈帧的代码来判断
.text:00408160 ; 3:参数的类型判断，先看所占内存大小，如果为 4 字节再根据
.text:00408160 ; 汇编代码中的[]寻址方式判断是指针还是数值。
.text:00408160 ; 4:参数的功能 分析其反汇编代码可知，系统层
.text:00408160 ; 程序 ＝ API + 数据结构，通过分析 api 可以了解函数功能，
.text:00408160 ; 对于系统级函数则只能分析代码
.text:00408160

.text:00408160 ; void __cdecl memset(void *, int, size_t)
.text:00408160 _memset        proc near          ; CODE XREF: _main+2Cp
.text:00408160                                    ; std::char_traits<char>::assign(char *,uint,char const &)+12p ...
.text:00408160

.text:00408160 arg_dst         = dword ptr  4
.text:00408160 arg_value       = byte ptr  8
.text:00408160 arg_size        = dword ptr  0Ch
.text:00408160

.text:00408160                  mov     edx, [esp+arg_size] ; arg_ 有三，即参数有 3 个，offset 4 8 c 所以其大小都为 4
.text:00408160                                    ; 然后再结合下面的代码分析是指针还是 int 类型
.text:00408164                  mov     ecx, [esp+arg_dst] ; 目标地址保存到 ecx
.text:00408168                  test    edx, edx        ; 判断要初始化的大小是否为 0
.text:0040816A                  jz      short toend     ; 如果要初始化的大小为 0 表示不用操作直接跳到返回代码 k
.text:0040816C                  xor     eax, eax        ; eax=0
.text:0040816E                  mov     al, [esp+arg_value] ; al = 要初始化的数据
.text:00408172                  push    edi             ; 保存 edi
.text:00408173                  mov     edi, ecx        ; ecx 地址保存到 edi(目标操作数寄存器)
.text:00408175                  cmp     edx, 4          ; 比较要初始化的大小是否小于 4
.text:00408178                  jb      short tail      ; 小于 4 时跳到 tail 中执行初始化
.text:0040817A                  neg     ecx             ; 地址求补
.text:0040817C                  and     ecx, 3          ; 判断字符串的存储地址是否对齐
.text:0040817F                  jz      short dwords    ; ecx=eax=value
.text:00408181                  sub     edx, ecx
.text:00408183

.text:00408183 adjust_loop:                          ; CODE XREF: _memset+27j
.text:00408183                  mov     [edi], al
.text:00408185                  inc     edi
.text:00408186                  dec     ecx
.text:00408187                  jnz     short adjust_loop
.text:00408189

.text:00408189 dwords:                               ; CODE XREF: _memset+1Fj
.text:00408189                  mov     ecx, eax        ; ecx=eax=value
.text:0040818B                  shl     eax, 8          ; ah = value
.text:0040818E                  add     eax, ecx        ; ax = value value
.text:00408190                  mov     ecx, eax        ; 保存 eax
.text:00408192                  shl     eax, 10h        ; eax 左移 16 位
.text:00408195                  add     eax, ecx        ; 扩充 value 值为 dword 类型
.text:00408195                                    ; value |= value<<8 | value<<16 |value<<24;
.text:00408197                  mov     ecx, edx        ; size->ecx
.text:00408199                  and     edx, 3          ; size & 3  3 = 0011B 如果是 0 则只能说明 size 的尾数位为
.text:00408199                                    ; 0000 0100  //0 4
.text:00408199                                    ; 1000 1100  //8 c
.text:00408199                                    ;  即判断初始的大小是否为 4 对齐
.text:00408199                                    ;
.text:0040819C                  shr     ecx, 2          ; 大小缩小 4 倍，因为它是以 dword 的形式进行初始化
.text:0040819F                  jz      short tail      ; 如果未对齐则去执行未对齐的操作
.text:004081A1                  rep stosd               ; ecx!=0 ecx=ecx-1 stod eax->edi ++edi;
.text:004081A3

.text:004081A3 main_loop_tail:                      ; 如果 edx=0
.text:004081A3                  test    edx, edx
.text:004081A5                  jz      short finish  ; 直接跳转到 finish
.text:004081A7

.text:004081A7 tail:                                 ; CODE XREF: _memset+18j
.text:004081A7                                    ; _memset+3Fj ...
.text:004081A7                  mov     [edi], al       ; 初始化
.text:004081A9                  inc     edi             ; 地址递增
.text:004081AA                  dec     edx             ; edx -= 1; inc dec 指令不影响 CF 标志位但是会影响 ZF 标志位
.text:004081AB                  jnz     short tail      ; 继续循环
.text:004081AD

.text:004081AD finish:                               ; CODE XREF: _memset+45j
.text:004081AD                  mov     eax, [esp+4+arg_dst]
.text:004081B1                  pop     edi
.text:004081B2                  retn                  ; 返回
.text:004081B3 ; ---------------------------------------------------------------------------
.text:004081B3
.text:004081B3 toend:                                 ; CODE XREF: _memset+Aj
.text:004081B3                  mov     eax, [esp+arg_dst]
.text:004081B7                  retn
```



# 20. `memmove`

在使用 C 库函数时，对齐是一个相当重要的问题。

## 函数原型

```asm
.text:00409020 ; void __cdecl memmove(char *dst, char *src, unsigned int count)
```

## 汇编代码

```asm
.text:00409020 _memmove        proc near               ; CODE XREF: _main+32p
.text:00409020                          ; std::char_traits<char>::move(char *,char const *,uint)+Fp ...
.text:00409020
.text:00409020 dst                 = dword ptr  8
.text:00409020 src                 = dword ptr  0Ch
.text:00409020 count               = dword ptr  10h
.text:00409020
.text:00409020                     push    ebp
.text:00409021                     mov     ebp, esp
.text:00409023                     push    edi
.text:00409024                     push    esi
.text:00409025                     mov     esi, [ebp+src]  ; src->esi
.text:00409028                     mov     ecx, [ebp+count] ; cont->ecx
.text:0040902B                     mov     edi, [ebp+dst]  ; dst->edi
.text:0040902E                     mov     eax, ecx        ; eax = edx = ecx = count
.text:00409030                     mov     edx, ecx
.text:00409032                     add     eax, esi        ; eax = src + count
.text:00409034                     cmp     edi, esi        ; if (dst <= src)
.text:00409036                     jbe     short CopyUp
.text:00409038                     cmp     edi, eax        ; if (dst <= src || dst >= src + count)
.text:00409038                                         ; {
.text:00409038                                         ;   copyup
.text:00409038                                         ; }
.text:00409038                                         ; else
.text:00409038                                         ; {
.text:00409038                                         ;   copydwon
.text:00409038                                         ; }
.text:0040903A                     jb      CopyDown
.text:00409040
.text:00409040 CopyUp:                              ; CODE XREF: _memmove+16j
.text:00409040                     test    edi, 3           ; 判断 dst 是否为 4 字节对齐
.text:00409046                     jnz     short CopyLeadUp ; 未对齐跳走
.text:00409048                     shr     ecx, 2
.text:0040904B                     and     edx, 3
.text:0040904E                     cmp     ecx, 8
.text:00409051                     jb      short CopyUnwindUp
.text:00409053                     rep movsd
.text:00409055                     jmp     ds:TrailUpVec[edx*4]
.text:0040905C ; ---------------------------------------------------------------------------
.text:0040905C
.text:0040905C CopyLeadUp:                          ; CODE XREF: _memmove+26j
.text:0040905C                     mov     eax, edi         ; eax = dst
.text:0040905E                     mov     edx, 3           ; edx = 3
.text:00409063                     sub     ecx, 4           ; cont -= 4
.text:00409066                     jb      short ByteCopyUp ; 如果 count 小于 4 跳到 byte 复制处 根据 count 选择判断条件
.text:00409068                     and     eax, 3           ; dst 地址的末尾 2 bit 与 0011b 进行与运算
.text:0040906B                     add     ecx, eax         ; 复制的大小进行对齐处理
.text:0040906D                     jmp     dword ptr ds:(CopyUnwindUp+4)[eax*4] ; 复制展开代码处
.text:00409074 ; ---------------------------------------------------------------------------
.text:00409074
.text:00409074 ByteCopyUp:                          ; CODE XREF: _memmove+46j
.text:00409074                     jmp     dword ptr ds:TrailUp0[ecx*4]
.text:00409074 ; ---------------------------------------------------------------------------
.text:0040907B                     align 4
.text:0040907C
.text:0040907C CopyUnwindUp:                        ; CODE XREF: _memmove+31j
.text:0040907C                         ; _memmove+8Ej ...
.text:0040907C                     jmp     ds:UnwindUpVec[ecx*4] ; 跳到展开位置
.text:0040907C ; ---------------------------------------------------------------------------
.text:00409083                     align 4
.text:00409084 ; unsigned int LeadUpVec
.text:00409084 LeadUpVec        dd offset LeadUp1
.text:00409088                    dd offset LeadUp2
.text:0040908C                    dd offset LeadUp3
.text:00409090 ; ---------------------------------------------------------------------------
.text:00409090
.text:00409090 LeadUp1:                              ; DATA XREF: _memmove:LeadUpVeco
.text:00409090                     and     edx, ecx
.text:00409092                     mov     al, [esi]
.text:00409094                     mov     [edi], al
.text:00409096                     mov     al, [esi+1]
.text:00409099                     mov     [edi+1], al
.text:0040909C                     mov     al, [esi+2]
.text:0040909F                     shr     ecx, 2
.text:004090A2                     mov     [edi+2], al
.text:004090A5                     add     esi, 3
.text:004090A8                     add     edi, 3
.text:004090AB                     cmp     ecx, 8
.text:004090AE                     jb      short CopyUnwindUp
.text:004090B0                     rep movsd
.text:004090B2                     jmp     ds:TrailUpVec[edx*4]
.text:004090B2 ; ---------------------------------------------------------------------------
.text:004090B9                     align 4
.text:004090BC
.text:004090BC LeadUp2:                              ; DATA XREF: _memmove+68o
.text:004090BC                     and     edx, ecx
.text:004090BE                     mov     al, [esi]
.text:004090C0                     mov     [edi], al
.text:004090C2                     mov     al, [esi+1]
.text:004090C5                     shr     ecx, 2
.text:004090C8                     mov     [edi+1], al
.text:004090CB                     add     esi, 2
.text:004090CE                     add     edi, 2
.text:004090D1                     cmp     ecx, 8
.text:004090D4                     jb      short CopyUnwindUp
.text:004090D6                     rep movsd
.text:004090D8                     jmp     ds:TrailUpVec[edx*4]
.text:004090D8 ; ---------------------------------------------------------------------------
.text:004090DF                     align 10h
.text:004090E0
.text:004090E0 LeadUp3:                              ; DATA XREF: _memmove+6Co
.text:004090E0                     and     edx, ecx
.text:004090E2                     mov     al, [esi]
.text:004090E4                     mov     [edi], al
.text:004090E6                     inc     esi
.text:004090E7                     shr     ecx, 2
.text:004090EA                     inc     edi
.text:004090EB                     cmp     ecx, 8
.text:004090EE                     jb      short CopyUnwindUp
.text:004090F0                     rep movsd
.text:004090F2                     jmp     ds:TrailUpVec[edx*4]
.text:004090F2 ; ---------------------------------------------------------------------------
.text:004090F9                     align 4
.text:004090FC ; unsigned int UnwindUpVec
.text:004090FC UnwindUpVec      dd offset UnwindUp0     ; DATA XREF: _memmove:CopyUnwindUpr
.text:00409100                    dd offset UnwindUp1
.text:00409104                    dd offset UnwindUp2
.text:00409108                    dd offset UnwindUp3
.text:0040910C                    dd offset UnwindUp4
.text:00409110                    dd offset UnwindUp5
.text:00409114                    dd offset UnwindUp6
.text:00409118                    dd offset UnwindUp7
.text:0040911C ; ---------------------------------------------------------------------------
.text:0040911C
.text:0040911C UnwindUp7:                          ; CODE XREF: _memmove:CopyUnwindUpj
.text:0040911C                         ; DATA XREF: _memmove+F8o
.text:0040911C                     mov     eax, [esi+ecx*4-1Ch]
.text:00409120                     mov     [edi+ecx*4-1Ch], eax
.text:00409124
```



```
.text:00409124 UnwindUp6:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409124                                         ; DATA XREF: _memmove+F4o

.text:00409124         mov     eax, [esi+ecx*4-18h]

.text:00409128         mov     [edi+ecx*4-18h], eax

.text:0040912C

.text:0040912C UnwindUp5:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040912C                                         ; DATA XREF: _memmove+F0o

.text:0040912C         mov     eax, [esi+ecx*4-14h]

.text:00409130         mov     [edi+ecx*4-14h], eax

.text:00409134

.text:00409134 UnwindUp4:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409134                                         ; DATA XREF: _memmove+ECo

.text:00409134         mov     eax, [esi+ecx*4-10h]

.text:00409138         mov     [edi+ecx*4-10h], eax

.text:0040913C

.text:0040913C UnwindUp3:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040913C                                         ; DATA XREF: _memmove+E8o

.text:0040913C         mov     eax, [esi+ecx*4-0Ch]

.text:00409140         mov     [edi+ecx*4-0Ch], eax

.text:00409144

.text:00409144 UnwindUp2:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:00409144                                         ; DATA XREF: _memmove+E4o

.text:00409144         mov     eax, [esi+ecx*4-8]

.text:00409148         mov     [edi+ecx*4-8], eax

.text:0040914C

.text:0040914C UnwindUp1:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040914C                                         ; DATA XREF: _memmove+E0o

.text:0040914C         mov     eax, [esi+ecx*4-4]

.text:00409150         mov     [edi+ecx*4-4], eax

.text:00409154         lea     eax, ds:0[ecx*4]

.text:0040915B         add     esi, eax

.text:0040915D         add     edi, eax

.text:0040915F

.text:0040915F UnwindUp0:                             ; CODE XREF: _memmove:CopyUnwindUpj

.text:0040915F                                         ; DATA XREF: _memmove:UnwindUpVeco

.text:0040915F         jmp     ds:TrailUpVec[edx*4]

.text:0040915F ; ---------------------------------------------------------------------------

.text:00409166         align 4

.text:00409168 ; unsigned int TrailUpVec

.text:00409168 TrailUpVec    dd offset TrailUp0      ; DATA XREF: _memmove+35r

.text:00409168                                         ; _memmove+92r ...

.text:0040916C         dd offset TrailUp1

.text:00409170         dd offset TrailUp2

.text:00409174         dd offset TrailUp3

.text:00409178 ; ---------------------------------------------------------------------------

.text:00409178

.text:00409178 TrailUp0:                              ; CODE XREF: _memmove+35j

.text:00409178                                         ; DATA XREF: _memmove:ByteCopyUpr ...

.text:00409178         mov     eax, [ebp+dst]  ; 0 字节时直接将 dst 作为返回值返回

.text:0040917B         pop     esi

.text:0040917C         pop     edi

.text:0040917D         leave

.text:0040917E         retn

.text:0040917E ; ---------------------------------------------------------------------------

.text:0040917F         align 10h

.text:00409180

.text:00409180 TrailUp1:                              ; CODE XREF: _memmove+35j

.text:00409180                                         ; DATA XREF: _memmove+14Co

.text:00409180         mov     al, [esi]       ; 复制 1 字节

.text:00409182         mov     [edi], al

.text:00409184         mov     eax, [ebp+dst]

.text:00409187         pop     esi

.text:00409188         pop     edi

.text:00409189         leave

.text:0040918A         retn

.text:0040918A ; ---------------------------------------------------------------------------

.text:0040918B         align 4

.text:0040918C

.text:0040918C TrailUp2:                              ; CODE XREF: _memmove+35j

.text:0040918C                                         ; DATA XREF: _memmove+150o

.text:0040918C         mov     al, [esi]       ; 复制 2 字节

.text:0040918E         mov     [edi], al

.text:00409190         mov     al, [esi+1]

.text:00409193         mov     [edi+1], al

.text:00409196         mov     eax, [ebp+dst]

.text:00409199         pop     esi

.text:0040919A         pop     edi

.text:0040919B         leave

.text:0040919C         retn

.text:0040919C ; ---------------------------------------------------------------------------

.text:0040919D         align 10h

.text:004091A0

.text:004091A0 TrailUp3:                              ; CODE XREF: _memmove+35j

.text:004091A0                                         ; DATA XREF: _memmove+154o

.text:004091A0         mov     al, [esi]       ; 复制 3 字节

.text:004091A2         mov     [edi], al

.text:004091A4         mov     al, [esi+1]

.text:004091A7         mov     [edi+1], al

.text:004091AA         mov     al, [esi+2]

.text:004091AD         mov     [edi+2], al

.text:004091B0         mov     eax, [ebp+dst]

.text:004091B3         pop     esi

.text:004091B4         pop     edi

.text:004091B5         leave

.text:004091B6         retn

.text:004091B6 ; ---------------------------------------------------------------------------

.text:004091B7         align 4

.text:004091B8

.text:004091B8 CopyDown:                               ; CODE XREF: _memmove+1Aj

.text:004091B8         lea     esi, [ecx+esi-4] ; esi = src + count - 1

.text:004091BC         lea     edi, [ecx+edi-4] ; edi = dst + count - 1

.text:004091C0         test    edi, 3           ; 对齐检测

.text:004091C6         jnz     short CopyLeadDown

.text:004091C8         shr     ecx, 2           ; 缩小 4 倍

.text:004091CB         and     edx, 3

.text:004091CE         cmp     ecx, 8

.text:004091D1         jb      short CopyUnwindDown

.text:004091D3         std

.text:004091D4         rep movsd

.text:004091D6         cld

.text:004091D7         jmp     ds:TrailDownVec[edx*4]

.text:004091D7 ; ---------------------------------------------------------------------------

.text:004091DE         align 10h

.text:004091E0

.text:004091E0 CopyUnwindDown:                         ; CODE XREF: _memmove+1B1j

.text:004091E0                                         ; _memmove+208j ...

.text:004091E0         neg     ecx

.text:004091E2         jmp     ds:off_4092B0[ecx*4]

.text:004091E2 ; ---------------------------------------------------------------------------

.text:004091E9         align 4

.text:004091EC

.text:004091EC CopyLeadDown:                           ; CODE XREF: _memmove+1A6j

.text:004091EC         mov     eax, edi        ; 对齐处理

.text:004091EE         mov     edx, 3

.text:004091F3         cmp     ecx, 4

.text:004091F6         jb      short ByteCopyDown

.text:004091F8         and     eax, 3

.text:004091FB         sub     ecx, eax

.text:004091FD         jmp     dword ptr ds:(ByteCopyDown+4)[eax*4]

.text:00409204 ; ---------------------------------------------------------------------------

.text:00409204
```



```
.text:00409204 ByteCopyDown: ; CODE XREF: _memmove+1D6j

.text:00409204 ; DATA XREF: _memmove+1DDr

.text:00409204 jmp ds:TrailDownVec[ecx*4]

.text:00409204 ; ---------------------------------------------------------------------------

.text:0040920B align 4

.text:0040920C ; unsigned int LeadDownVec

.text:0040920C LeadDownVec dd offset LeadDown1

.text:00409210 dd offset LeadDown2

.text:00409214 dd offset LeadDown3

.text:00409218 ; ---------------------------------------------------------------------------

.text:00409218

.text:00409218 LeadDown1: ; DATA XREF: _memmove:LeadDownVeco

.text:00409218 mov al, [esi+3]

.text:0040921B and edx, ecx

.text:0040921D mov [edi+3], al

.text:00409220 dec esi

.text:00409221 shr ecx, 2

.text:00409224 dec edi

.text:00409225 cmp ecx, 8

.text:00409228 jb short CopyUnwindDown

.text:0040922A std

.text:0040922B rep movsd

.text:0040922D cld

.text:0040922E jmp ds:TrailDownVec[edx*4]

.text:0040922E ; ---------------------------------------------------------------------------

.text:00409235 align 4

.text:00409238

.text:00409238 LeadDown2: ; DATA XREF: _memmove+1F0o

.text:00409238 mov al, [esi+3]

.text:0040923B and edx, ecx

.text:0040923D mov [edi+3], al

.text:00409240 mov al, [esi+2]

.text:00409243 shr ecx, 2

.text:00409246 mov [edi+2], al

.text:00409249 sub esi, 2

.text:0040924C sub edi, 2

.text:0040924F cmp ecx, 8

.text:00409252 jb short CopyUnwindDown

.text:00409254 std

.text:00409255 rep movsd

.text:00409257 cld

.text:00409258 jmp ds:TrailDownVec[edx*4]

.text:00409258 ; ---------------------------------------------------------------------------

.text:0040925F align 10h

.text:00409260

.text:00409260 LeadDown3: ; DATA XREF: _memmove+1F4o

.text:00409260 mov al, [esi+3]

.text:00409263 and edx, ecx

.text:00409265 mov [edi+3], al

.text:00409268 mov al, [esi+2]

.text:0040926B mov [edi+2], al

.text:0040926E mov al, [esi+1]

.text:00409271 shr ecx, 2

.text:00409274 mov [edi+1], al

.text:00409277 sub esi, 3

.text:0040927A sub edi, 3

.text:0040927D cmp ecx, 8

.text:00409280 jb CopyUnwindDown

.text:00409286 std

.text:00409287 rep movsd

.text:00409289 cld

.text:0040928A jmp ds:TrailDownVec[edx*4]

.text:0040928A ; ---------------------------------------------------------------------------

.text:00409291 align 4

.text:00409294 ; unsigned int UnwindDownVec

.text:00409294 UnwindDownVec dd offset UnwindDown7

.text:00409298 dd offset UnwindDown6

.text:0040929C dd offset UnwindDown5

.text:004092A0 dd offset UnwindDown4

.text:004092A4 dd offset UnwindDown3

.text:004092A8 dd offset UnwindDown2

.text:004092AC dd offset UnwindDown1

.text:004092B0 off_4092B0 dd offset UnwindDown0 ; DATA XREF: _memmove+1C2r

.text:004092B4 ; ---------------------------------------------------------------------------

.text:004092B4

.text:004092B4 UnwindDown7: ; DATA XREF: _memmove:UnwindDownVeco

.text:004092B4 mov eax, [esi+ecx*4+1Ch]

.text:004092B8 mov [edi+ecx*4+1Ch], eax

.text:004092BC

.text:004092BC UnwindDown6: ; DATA XREF: _memmove+278o

.text:004092BC mov eax, [esi+ecx*4+18h]

.text:004092C0 mov [edi+ecx*4+18h], eax

.text:004092C4

.text:004092C4 UnwindDown5: ; DATA XREF: _memmove+27Co

.text:004092C4 mov eax, [esi+ecx*4+14h]

.text:004092C8 mov [edi+ecx*4+14h], eax

.text:004092CC

.text:004092CC UnwindDown4: ; DATA XREF: _memmove+280o

.text:004092CC mov eax, [esi+ecx*4+10h]

.text:004092D0 mov [edi+ecx*4+10h], eax

.text:004092D4

.text:004092D4 UnwindDown3: ; DATA XREF: _memmove+284o

.text:004092D4 mov eax, [esi+ecx*4+0Ch]

.text:004092D8 mov [edi+ecx*4+0Ch], eax

.text:004092DC

.text:004092DC UnwindDown2: ; DATA XREF: _memmove+288o

.text:004092DC mov eax, [esi+ecx*4+8]

.text:004092E0 mov [edi+ecx*4+8], eax

.text:004092E4

.text:004092E4 UnwindDown1: ; DATA XREF: _memmove+28Co

.text:004092E4 mov eax, [esi+ecx*4+4]

.text:004092E8 mov [edi+ecx*4+4], eax

.text:004092EC lea eax, ds:0[ecx*4]

.text:004092F3 add esi, eax

.text:004092F5 add edi, eax

.text:004092F7

.text:004092F7 UnwindDown0: ; CODE XREF: _memmove+1C2j

.text:004092F7 ; DATA XREF: _memmove:off_4092B0o

.text:004092F7 jmp ds:TrailDownVec[edx*4]

.text:004092F7 ; ---------------------------------------------------------------------------

.text:004092FE align 10h

.text:00409300 ; unsigned int TrailDownVec

.text:00409300 TrailDownVec dd offset TrailDown0 ; DATA XREF: _memmove+1B7r

.text:00409300 ; _memmove:ByteCopyDownr ...

.text:00409304 dd offset TrailDown1

.text:00409308 dd offset TrailDown2

.text:0040930C dd offset TrailDown3

.text:00409310 ; ---------------------------------------------------------------------------

.text:00409310

.text:00409310 TrailDown0: ; CODE XREF: _memmove+1B7j

.text:00409310 ; _memmove:ByteCopyDownj ...

.text:00409310 mov eax, [ebp+dst]

.text:00409313 pop esi

.text:00409314 pop edi

.text:00409315 leave

.text:00409316 retn

.text:00409316 ; ---------------------------------------------------------------------------

.text:00409317 align 4

.text:00409318

.text:00409318 TrailDown1: ; CODE XREF: _memmove+1B7j

.text:00409318 ; _memmove:ByteCopyDownj ...

.text:00409318 mov al, [esi+3]

.text:0040931B mov [edi+3], al

.text:0040931E mov eax, [ebp+dst]

.text:00409321 pop esi

.text:00409322 pop edi

.text:00409323 leave

.text:00409324 retn
```



### `.text:00409324`

```
.text:00409324  ; ---------------------------------------------------------------------------
.text:00409325              align 4
.text:00409328
.text:00409328 TrailDown2:                    ; CODE XREF: _memmove+1B7j
.text:00409328                                 ; _memmove:ByteCopyDownj ...
.text:00409328              mov     al, [esi+3]
.text:0040932B              mov     [edi+3], al
.text:0040932E              mov     al, [esi+2]
.text:00409331              mov     [edi+2], al
.text:00409334              mov     eax, [ebp+dst]
.text:00409337              pop     esi
.text:00409338              pop     edi
.text:00409339              leave
.text:0040933A              retn
.text:0040933A ; ---------------------------------------------------------------------------
.text:0040933B              align 4
.text:0040933C
.text:0040933C TrailDown3:                    ; CODE XREF: _memmove+1B7j
.text:0040933C                                 ; _memmove:ByteCopyDownj ...
.text:0040933C              mov     al, [esi+3]
.text:0040933F              mov     [edi+3], al
.text:00409342              mov     al, [esi+2]
.text:00409345              mov     [edi+2], al
.text:00409348              mov     al, [esi+1]
.text:0040934B              mov     [edi+1], al
.text:0040934E              mov     eax, [ebp+dst]
.text:00409351              pop     esi
.text:00409352              pop     edi
.text:00409353              leave
.text:00409354              retn
.text:00409354 _memmove        endp
```

有些东西跑到数据段了，`IDA` 分析出现问题，不过可以直接按 `C` 将数据转换成代码，不影响分析，这个函数的实现着实复杂了不少....没学到东东....以后使用时，最好不要使用这个函数。

## 21、`bcmp`

`msdn` 中无此函数

## 22、`bcopy`

`msdn` 中无此函数

## 23、`bzero`

`msdn` 无此函数

## 24、`memccpy`

```
.text:00418FE0 000 8B 4C 24 10                              mov     ecx, [esp+Size]        ; ecx = size
.text:00418FE4 000 53                                       push    ebx              ; 保存 ebx
.text:00418FE5 004 85 C9                                     test    ecx, ecx         ; ecx = size
.text:00418FE7 004 74 40                                     jz      short ret_zero_len ; 判断要复制的长度是否为 0 则跳到函数返回
.text:00418FE9 004 8A 7C 24 10                               mov     bh, byte ptr [esp+4+Val] ; bh = value 中断字符
.text:00418FED 004 56                                       push    esi              ; 保存 esi
.text:00418FEE 008 F7 C1 01 00 00 00                         test    ecx, 1           ; ecx 的末 bit 位是否为 0 0001B
.text:00418FEE                                               ; 末 bit 位为 0 说明是一个偶地址
.text:00418FEE                                               ; 即此处为判断偶对齐
.text:00418FF4 008 8B 44 24 0C                               mov     eax, [esp+8+Dst]    ; eax = dst
.text:00418FF8 008 8B 74 24 10                               mov     esi, [esp+8+Src]    ; esi = src
.text:00418FFC 008 74 0D                                      jz      short lupe2      ; 如果 size 的末 bit 位为 0 跳走
.text:00418FFE 008 8A 1E                                      mov     bl, [esi]         ; src->esi
.text:00419000 008 46                                       inc     esi              ; esi+1
.text:00419001 008 88 18                                     mov     [eax], bl        ; 保存到 dst
.text:00419003 008 40                                       inc     eax              ; dst 地址+1
.text:00419004 008 38 FB                                     cmp     bl, bh           ; 判断是否为中断字符
.text:00419006 008 74 28                                     jz      short toend_1
.text:00419008 008 49                                       dec     ecx              ; 计数器-1
.text:00419009 008 74 1D                                     jz      short retnull_1
.text:0041900B

.text:0041900B                        lupe2:                 ; CODE XREF: _memccpy+1Cj
.text:0041900B                                             ; _memccpy+46j
.text:0041900B 008 8A 1E                                     mov     bl, [esi]         ; 取 scr->bl
.text:0041900D 008 83 C6 02                                  add     esi, 2           ; esi+2
.text:00419010 008 38 FB                                     cmp     bl, bh           ; 判断是否为中断字符 bh = value 中断字符
.text:00419012 008 74 19                                     jz      short toend_mov_inc ; 是则跳走
.text:00419014 008 88 18                                     mov     [eax], bl        ; bl -> dst
.text:00419016 008 8A 5E FF                                  mov     bl, [esi-1]      ; esi-1（上面 add esi)
.text:00419019 008 88 58 01                                  mov     [eax+1], bl      ; 保存到 dst+1 的位置
.text:0041901C 008 83 C0 02                                  add     eax, 2           ; dst 地址+2
.text:0041901F 008 38 FB                                     cmp     bl, bh           ; 比较是否为中断字符
.text:00419021 008 74 0D                                     jz      short toend_1    ; 相等则跳到结束位置
.text:00419023 008 83 E9 02                                  sub     ecx, 2           ; 计数器减 2
.text:00419026 008 75 E3                                     jnz     short lupe2      ; 继续循环
.text:00419028

.text:00419028                        retnull_1:             ; CODE XREF: _memccpy+29j
.text:00419028 008 5E                                       pop     esi
.text:00419029

.text:00419029                        ret_zero_len:          ; CODE XREF: _memccpy+7j
.text:00419029 004 33 C0                                     xor     eax, eax         ; 返回 0
.text:0041902B 004 5B                                       pop     ebx
.text:0041902C 000 C3                                       retn
.text:0041902D                        ; ---------------------------------------------------------------------------
.text:0041902D

.text:0041902D                        toend_mov_inc:         ; CODE XREF: _memccpy+32j
.text:0041902D 008 88 18                                     mov     [eax], bl        ; src 中的字符保存到 dst 中
.text:0041902F 008 40                                       inc     eax              ; eax 地址+1
.text:00419030

.text:00419030                        toend_1:               ; CODE XREF: _memccpy+26j
.text:00419030                                             ; _memccpy+41j
.text:00419030 008 5E                                       pop     esi
.text:00419031 004 5B                                       pop     ebx
.text:00419032 000 C3                                       retn
.text:00419032                        _memccpy        endp
.text:00419032
```

### 判断偶对齐

```
if (addr & 1) cout<<"unaligned!"<<endl;

else cout<<"aligned!"<<endl;
```

以 `x` 字节对齐可以借用公式 `addr & (x - 1)` 来判断

对齐分为地址对齐和大小对齐，上面说的对齐指地址对齐，对于大小对齐则可以

```
if (size % align==0) cout<<"aligned!"<<endl;

else cout <<"unaligned!"<<endl;
```

## 25、`memchr`

`; void **__cdecl memchr(const void *Buf, int Val, size_t MaxCount)`

```
.text:004081A0 _memchr         proc near                ; CODE XREF: _main+5Ep
.text:004081A0

.text:004081A0 Buf             = dword ptr  4
.text:004081A0 Val             = dword ptr  8
.text:004081A0 MaxCount        = dword ptr  0Ch
.text:004081A0

.text:004081A0                mov     eax, [esp+MaxCount] ; eax = 查询计数
.text:004081A4                push    ebx             ; 保存 ebx
```



### 2. 汇编代码分析

```
.text:004081A5                   test    eax, eax        ; 判断计数是否为 0

.text:004081A7                   jz      short retnull   ; 计数为 0 则直接返回

.text:004081A9                   mov     edx, [esp+4+Buf] ; edx=buffer 地址

.text:004081AD                   xor     ebx, ebx        ; ebx=0

.text:004081AF                   mov     bl, byte ptr [esp+4+Val] ; bl = value 要查询的字符

.text:004081B3                   test    edx, 3          ; buffer 地址是否为 4 字节对齐

.text:004081B9                   jz      short main_loop_start

.text:004081BB

.text:004081BB str_misaligned:                      ; CODE XREF: _memchr+2Bj

.text:004081BB                   mov     cl, [edx]       ; 取 buff 中的字符

.text:004081BD                   inc     edx             ; 地址递增

.text:004081BE                   xor     cl, bl          ; cl==bl 则跳走

.text:004081C0                   jz      short found

.text:004081C2                   dec     eax             ; if (--count) 如果 count=1 则跳到返回位置

.text:004081C3                   jz      short retnull   ; 没有找到则直接返回

.text:004081C5                   test    edx, 3          ; 判断是否对齐，如果已 4 字节对齐则跳到主循环中，否则继续

.text:004081C5                                       ; 循环调整地址

.text:004081CB                   jnz     short str_misaligned

.text:004081CD

.text:004081CD main_loop_start:                       ; CODE XREF: _memchr+19j

.text:004081CD                   sub     eax, 4        ; eax -= 4; 计数减去 4

.text:004081D0                   jb      short tail_less_then_4 ; 小于 4 则跳到比对位置

.text:004081D2                   push    edi          ; 计数大于等于 4 的情况

.text:004081D3                   mov     edi, ebx        ; edi = value

.text:004081D5                   shl     ebx, 8

.text:004081D8                   add     ebx, edi

.text:004081DA                   mov     edi, ebx

.text:004081DC                   shl     ebx, 10h

.text:004081DF                   add     ebx, edi        ; 将 value 扩展成 4 字节

.text:004081E1                   jmp     short main_loop_entry

.text:004081E3 ; ------------------------------------------------------------

.text:004081E3

.text:004081E3 return_from_main:                      ; CODE XREF: _memchr+58j

.text:004081E3                   pop     edi

.text:004081E4

.text:004081E4 tail_less_then_4:                       ; CODE XREF: _memchr+30j

.text:004081E4                   add     eax, 4        ; count -= 4;

.text:004081E4                                       ; count += 4; 如果 zf=1 即表示 count=0 此时函数返回

.text:004081E7                   jz      short retnull

.text:004081E9

.text:004081E9 tail_loop:                             ; CODE XREF: _memchr+51j

.text:004081E9                   mov     cl, [edx]       ; 取字符到 cl

.text:004081EB                   inc     edx             ; 地址递增 CF 不变

.text:004081EC                   xor     cl, bl          ; 判断是否相等

.text:004081EE                   jz      short found

.text:004081F0                   dec     eax             ; 计数递减

.text:004081F1                   jnz     short tail_loop

.text:004081F3

.text:004081F3 retnull:                               ; CODE XREF: _memchr+7j

.text:004081F3                                       ; _memchr+23j ...

.text:004081F3                   pop     ebx           ; 直接返回

.text:004081F4                   retn

.text:004081F5 ; ------------------------------------------------------------

.text:004081F5

.text:004081F5 main_loop:                              ; CODE XREF: _memchr+73j

.text:004081F5                                       ; _memchr+8Bj

.text:004081F5                   sub     eax, 4        ; 判断计数

.text:004081F8                   jb      short return_from_main

.text:004081FA

.text:004081FA main_loop_entry:                        ; CODE XREF: _memchr+41j

.text:004081FA                   mov     ecx, [edx]     ; 取 4 字符到 ecx

.text:004081FC                   xor     ecx, ebx        ; 判断是否为要查询的字符

.text:004081FE                   mov     edi, 7EFEFEFFh  ; 快速定位 0 字符的代码 strlen()中就是这样的

.text:00408203                   add     edi, ecx

.text:00408205                   xor     ecx, 0FFFFFFFFh

.text:00408208                   xor     ecx, edi

.text:0040820A                   add     edx, 4

.text:0040820D                   and     ecx, 81010100h

.text:00408213                   jz      short main_loop ; 发现 0 字符跳走，调用计数小于 4 时的代码

.text:00408215

.text:00408215 char_is_found:                          ; 发现字符的处理

.text:00408215                   mov     ecx, [edx-4]

.text:00408218                   xor     cl, bl

.text:0040821A                   jz      short loc_40823F

.text:0040821C                   xor     ch, bl

.text:0040821E                   jz      short loc_408239

.text:00408220                   shr     ecx, 10h

.text:00408223                   xor     cl, bl

.text:00408225                   jz      short loc_408233

.text:00408227                   xor     ch, bl

.text:00408229                   jz      short loc_40822D

.text:0040822B                   jmp     short main_loop

.text:0040822D ; ------------------------------------------------------------

.text:0040822D

.text:0040822D loc_40822D:                             ; CODE XREF: _memchr+89j

.text:0040822D                   pop     edi

.text:0040822E

.text:0040822E found:                                  ; CODE XREF: _memchr+20j

.text:0040822E                                       ; _memchr+4Ej

.text:0040822E                   lea     eax, [edx-1]    ; eax = buffer - 1 即返回寻找到的地址

.text:00408231                   pop     ebx

.text:00408232                   retn

.text:00408233 ; ------------------------------------------------------------

.text:00408233

.text:00408233 loc_408233:                             ; CODE XREF: _memchr+85j

.text:00408233                   lea     eax, [edx-2]

.text:00408236                   pop     edi

.text:00408237                   pop     ebx

.text:00408238                   retn

.text:00408239 ; ------------------------------------------------------------

.text:00408239

.text:00408239 loc_408239:                             ; CODE XREF: _memchr+7Ej

.text:00408239                   lea     eax, [edx-3]

.text:0040823C                   pop     edi

.text:0040823D                   pop     ebx

.text:0040823E                   retn

.text:0040823F ; ------------------------------------------------------------

.text:0040823F

.text:0040823F loc_40823F:                             ; CODE XREF: _memchr+7Aj

.text:0040823F                   lea     eax, [edx-4]

.text:00408242                   pop     edi

.text:00408243                   pop     ebx

.text:00408244                   retn

.text:00408244 _memchr            endp
```

与前面的类似

### 2. 2. memcmp (memicmp 略)

```
.text:004081E0 arg_buf1         = dword ptr  4
.text:004081E0 arg_buf2         = dword ptr  8
.text:004081E0 arg_size         = dword ptr  0Ch

.text:004081E0

.text:004081E0                   mov     eax, [esp+arg_size] ; arg_ 开头 函数参数与栈帧中的位置对应是一个 90 度顺时针旋转

.text:004081E4                   test    eax, eax        ; 判断计数是否为 0

.text:004081E6                   jz      short retnull   ; 函数直接返回

.text:004081E8                   mov     edx, [esp+arg_buf1] ; edx = buf1

.text:004081EC                   push    esi

.text:004081ED                   push    edi

.text:004081EE                   mov     esi, edx        ; esi = buf1

.text:004081F0                   mov     edi, [esp+8+arg_buf2] ; edi = buf2

.text:004081F4                   or      edx, edi

.text:004081F6                   and     edx, 3          ; 判断 4 字节对齐

.text:004081F9                   jz      short dwords    ; 对齐跳到 dwords 处理
```



```
.text:004081FB                 test    eax, 1           ; 0001B 只能与以 0 末尾的数才能得 0 即偶数  

.text:00408200                 jz      short main\_loop ; size=1 则跳到 main\_loop 中  

.text:00408202                 mov     cl, \[esi\]  

.text:00408204                 cmp     cl, \[edi\]  

.text:00408206                 jnz     short not\_equal  

.text:00408208                 inc     esi  

.text:00408209                 inc     edi  

.text:0040820A                 dec     eax  

.text:0040820B                 jz      short done  

.text:0040820D  

.text:0040820D main\_loop:                                ; CODE XREF: \_memcmp+20j  

.text:0040820D                                   ; \_memcmp+48j  

.text:0040820D                 mov     cl, \[esi\]       ; 取 buf1  

.text:0040820F                 mov     dl, \[edi\]       ; 取 buf2  

.text:00408211                 cmp     cl, dl           ; 比较字符  

.text:00408213                 jnz     short not\_equal ; 相等则继续+1 比较不相等则跳到不相等  

.text:00408215                 mov     cl, \[esi+1\]  

.text:00408218                 mov     dl, \[edi+1\]  

.text:0040821B                 cmp     cl, dl  

.text:0040821D                 jnz     short not\_equal ; 不相等则跳到不相等的位置  

.text:0040821F                 add     edi, 2  

.text:00408222                 add     esi, 2  

.text:00408225                 sub     eax, 2  

.text:00408228                 jnz     short main\_loop  

.text:0040822A  

.text:0040822A done:                                    ; CODE XREF: \_memcmp+2Bj  

.text:0040822A                                   ; \_memcmp+84j  

.text:0040822A                 pop     edi  

.text:0040822B                 pop     esi  

.text:0040822C  

.text:0040822C retnull:                                ; CODE XREF: \_memcmp+6j  

.text:0040822C                 retn  

.text:0040822D ; ---------------------------------------------------------------------------  

.text:0040822D  

.text:0040822D dwords:                                  ; CODE XREF: \_memcmp+19j  

.text:0040822D                 mov     ecx, eax        ; 计数->ecx  

.text:0040822F                 and     eax, 3          ; 末两 bit 是否为 0 计数是 4 的倍数  

.text:00408232                 shr     ecx, 2          ; ecx 缩小 4 倍 如果是 0 则表示此时计数为 4  

.text:00408235                 jz      short tail\_loop\_start ; 跳到处理末尾的代码  

.text:00408237                 repe cmpsd            ; ecx!= && zf=1 esi edi，因为上面如果 ZF=1 则跳走，所以这里应该只比较 4 字节的字符  

.text:00408239                 jz      short tail\_loop\_start ; 此时的 ZF 标志位有 cmpsd 指令来设置  

.text:0040823B                 mov     ecx, \[esi-4\]    ; 不相等则取原起始地址的 4 字符分别存入 ecx edx  

.text:0040823E                 mov     edx, \[edi-4\]  

.text:00408241                 cmp     cl, dl          ; 比较低字节  

.text:00408243                 jnz     short difference\_in\_tail  

.text:00408245                 cmp     ch, dh          ; 比较高字节  

.text:00408247                 jnz     short difference\_in\_tail  

.text:00408249                 shr     ecx, 10h        ; 缩小 2¹⁶ 次方 将原来 ecx edx 的高字节保存到低字节中  

.text:0040824C                 shr     edx, 10h  

.text:0040824F                 cmp     cl, dl  

.text:00408251                 jnz     short difference\_in\_tail ; 继续比较  

.text:00408253                 cmp     ch, dh  

.text:00408255  

.text:00408255 difference\_in\_tail:                   ; CODE XREF: \_memcmp+63j  

.text:00408255                                   ; \_memcmp+67j ...  

.text:00408255                 mov     eax, 0  

.text:0040825A  

.text:0040825A not\_equal:                               ; CODE XREF: \_memcmp+26j  

.text:0040825A                                   ; \_memcmp+33j ...  

.text:0040825A                 sbb     eax, eax  

.text:0040825C                 pop     edi  

.text:0040825D                 sbb     eax, 0FFFFFFFFh  

.text:00408260                 pop     esi  

.text:00408261                 retn  

.text:00408262 ; ---------------------------------------------------------------------------  

.text:00408262  

.text:00408262 tail\_loop\_start:                         ; CODE XREF: \_memcmp+55j  

.text:00408262                                   ; \_memcmp+59j  

.text:00408262                 test    eax, eax        ; 判断 eax 是否为 0  

.text:00408264                 jz      short done      ; 是 0 则返回  

.text:00408266                 mov     edx, \[esi\]      ; edx = buf1  

.text:00408268                 mov     ecx, \[edi\]      ; ecx = buf2  

.text:0040826A                 cmp     dl, cl          ; 比较 dl 和 cl  

.text:0040826C                 jnz     short difference\_in\_tail ; 不相等  

.text:0040826E                 dec     eax             ; 计数-1  

.text:0040826F                 jz      short tail\_done ; 减 1 后如果为 0 表示完成返回  

.text:00408271                 cmp     dh, ch          ; 否则比较第二个字符  

.text:00408273                 jnz     short difference\_in\_tail ; 不相等跳走  

.text:00408275                 dec     eax             ; 继续计数-1  

.text:00408276                 jz      short tail\_done ; 如果为 0 表示结束返回  

.text:00408278                 and     ecx, 0FF0000h   ; 否则比较第三个字符  

.text:0040827E                 and     edx, 0FF0000h  

.text:00408284                 cmp     edx, ecx  

.text:00408286                 jnz     short difference\_in\_tail  

.text:00408288                 dec     eax             ; 这部分代码还是一个 0 字符扫描算法与 strln()中的类似  

.text:00408289  

.text:00408289 tail\_done:                               ; CODE XREF: \_memcmp+8Fj  

.text:00408289                                   ; \_memcmp+96j  

.text:00408289                 pop     edi  

.text:0040828A                 pop     esi  

.text:0040828B                 retn  

.text:0040828B \_memcmp         endp
```

## 27、`memcpy`（参照 `memccpy`）

小结：字符串及内存处理函数主要有如下几点内容：

1：顺序表的内容

顺序表是一个非常常用的数据结构，下面是一个数据结构模板入两个利用顺序表模板的具体实例（`PE` 结构和面向对象程序设计模型）。

```cpp
//target:顺序表类模板  
//author:by Reduta  
//descriptor: IA32 + xp Sp3 + Vc6.0  

#include <iostream>  

using namespace std;  

#define DefaultSize 100  

template <typename Type> SqList  
{  
  Type *pList;  
  const unsigned MaxSize;  
  int CurrentSize;  

public:  
  SqList(int Size = DefaultSize):CurrentSize(-1)  
  {  
    if (Size > 0)  
    {  
      pList = new Type\[Size\];  
    }  
    MaxSize = Size;  
  }  
  ~Sqlist()  
  {  
    delete \[\] pList;  
    pList = NULL;  
  }  
  bool IsEmpty(){return CurrentSize==-1;}  
  bool IsFull(){return MaxSize==CurrentSize + 1;}  
  Type FindByInx(int position)  
  {  
    if (position < 0 || position >=CurrentSize)  
    {  
      cout<<"can't find element!"<<endl;  
      return -1;  
    }  
    return pList\[position\];  
  }  
  void ShowList()const;  
  bool Remove(int position);  
  bool Insert(int position,Type data);  
  bool Set(int con);  
  void MaxAndMin();  
};  

//显示所有的元素  
template<typename Type> void SqList<Type>::ShowList()const  
{  
  if (!IsEmpty())  
  {  
    for (size\_t inx=0; inx<=CurrentSize; ++inx)  
    {  
      if (inx!=0 && inx % 8==0)  
        cout<<endl;  
      cout<<pList\[inx\];  
    }  
  }  
}  

//删除元素  
template<typename Type> bool SqList<Type>::Remove(int position)  
{  
  if (position<0 || position>CurrentSize) return false;  
  for (size\_t inx=position; inx!=CurrentSize; ++inx)  
  {  
    pList\[inx\] = pList\[inx+1\];  
  }  
  --CurrentSize;  
  return true;  
}  

//插入元素  
template<typename Type> bool SqList<Type>::Insert(int position,Type data)  
{  
  if (IsFull() || position<0 || position>CurrentSize+1)  
  {  
    cout<<"can't insert data";  
    return false;  
  }  
  for (size\_t inx=CurrentSize; inx!=position; --inx)  
  {  
    pList\[inx+1\] = pList\[inx\];  
  }  
  ++CurrentSize;  
  pList\[position\] = data;  
  return true;  
}  

//根据条件　进行　从小到大　从大到小　逆向　删除重复元素的冒泡排序  
template<typename Type> bool SqList<Type>::Set(int con)  
{  
  if (con==1)  
  {
```



```cpp
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i] > pList[j])
        {
          pList[i] = pList[i] ^ pList[j];
          pList[j] = pList[i] ^ pList[j];
          pList[i] = pList[i] ^ pList[j];
        }
      }
    }
    return true;
  }
  if (con==2)
  {
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i] < pList[j])
        {
          pList[i] = pList[i] ^ pList[j];
          pList[j] = pList[i] ^ pList[j];
          pList[i] = pList[i] ^ pList[j];
        }
      }
    }
    return true;
  }
  if (con==3)
  {
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i]==pList[j])
        {
          for (size_t k=i; k!=CurrentSize+1; ++k)
          {
            pList[k] = pList[k+1];
          }
          --j;
          --i;
          --CurrentSize;
        }
      }
    }
    return true;
  }
  if (con==4)
  {
    for (size_t i=0,j=CurrentSize; i!=(CurrentSize+1)/2; ++i,--j)
    {
      pList[i] = pList[i] ^ pList[j];
      pList[j] = pList[i] ^ pList[j];
      pList[i] = pList[i] ^ pList[j];
    }
  }
}
```

//假设法求最大值与最小值
```cpp
template<typename Type> void SqList<Type>::MaxAndMin()
{
  Type Max = pList[0];
  Type Min = pList[0];
  for (size_t inx=0; inx!=CurrentSize; ++inx)
  {
    if (pList[inx]>Max)
    {
      Max = pList[inx];
    }
    if (pList[inx]<Min)
    {
      Min = pList[inx];
    }
  }
  cout<<"Maximum:"<<Max<<" "<<Minmum:"<<Min<<endl;
}
```

//PE 区段表就是一个典型的顺序表。只不过它的数据元素为 `IMAGE_SECTION_HEADER`
```cpp
void ShowPeSecInfo(LPVOID lpBase)
{
  PIMAGE_DOS_HEADER pdos = (PIMAGE_DOS_HEADER)lpBase;
  PIMAGE_NT_HEADERS pnt = (PIMAGE_NT_HEADERS)((PBYTE)lpBase + pdos->e_lfanew);
  PIMAGE_SECTION_HEADER pfirst = (PIMAGE_SECTION_HEADER)((PBYTE)lpBase + pdos->e_lfanew + sizeof(IMAGE_NT_SIGNATURE) +
sizeof(IMAGE_FILE_HEADER) + pnt->FileHeader->SizeOfOptionalHeader);
  for (size_t inx=0; inx!=pnt->FileHeader.NumberOfSections; ++inx)
  {
    cout<<pfirst[inx].Name<<endl;
    //cout<<(pfirst+inx)->Name<<endl;
  }
}
```

对于 `PE` 区段的添加、删除等操作也是一样的类似。

整个 `PE` 结构可以理解为一链式顺序表，即通过顺序表存储地址的方式。

//`oop` 中的对象数据也是一个顺序表，有 `this` 指针指明对象数据成员的起始地址，通过 `this + inx` 的方式访问
```cpp
#include <iostream>
using namespace std;

class test
{
  int a,b;
public:
  test(int a=0,int b=0);
  void show()const;  //类外定义排除 `inline` 内联
};

test::test(int a/* =0 */,int b/* =0 */)
{
  this->a = a;
  this->b = b;
}

void test::show()const
{
  cout<<a<<" "<<b<<endl;
}

int wmain()
{
  test fk(1,1);    //构造函数在汇编级返回当前对象的 `this` 指针，所以下面使用的 `eax` 而非 `ecx`
  __asm mov [eax],10
  __asm mov [eax+4],10
  fk.show();
  return 0;
}
```

//多维数组也是顺序表
```cpp
int wmain()
{
  int a[2][2][2][2] = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16};
  int *p = NULL;
  __asm lea eax,a
  __asm mov p,eax
  for (size_t inx=0; inx!=16; ++inx)
  {
    cout<<p[inx]<<endl;
  }
  return 0;
}
```

//位操作指令也是一个顺序表
`bt/bts/btr/btc reg/mem reg/imm`

第一个操作数为 `pstart` 第二个操作数 `inx`

`bt` -> `pstart[inx]->cf`
`bts` -> `pstart[inx]->cf` `pstart[inx] = 1`
`btc` -> `pstart[inx]->cf` `pstart[inx] = ~pstart[inx]`
`btr` -> `pstart[inx]->cf` `pstart[inx] = 0`

### 2、基本的整数表示

原码：

反码：最大值 - 当前值

补码：以最右侧的 1 为起始，将其左边的二进制位求反。

### 3、对齐问题

地址对齐 `if (addr & align - 1)`
大小对齐 `if (dwsize % align)`

### 4、逻辑运算

### 5、熟悉位运算

除了上面的 `bt/btc/btr/bts` 还需要理解 `bsf/bsr` (可以快速析出程序中的第一个 1 的位置)、`test` 指令。
