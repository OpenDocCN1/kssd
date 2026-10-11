
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
