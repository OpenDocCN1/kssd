
# [原创]C库字符串反汇编分析

## strlen

```asm
.text:00408150                 mov     ecx, [esp+arg_0]
.text:00408154                 test    ecx, 3          ; if (str & 3)  11B检测地址末位是0 4  8  c
.text:00408154                                         ; 即判断字符串的起始地址是否以4对齐 glibc中基本的对齐
.text:00408154                                         ; 策略是2 * sizeof(size_t)
.text:0040815A                 jz      short main_loop ; 如果地址末尾与3与运算不为0
.text:0040815C
.text:0040815C str_misaligned:                         ; CODE XREF: _strlen+19j
.text:0040815C                 mov     al, [ecx]       ; 取一个字符
.text:0040815E                 inc     ecx             ; 增加串指针
.text:0040815F                 test    al, al          ; 比较al是否为0字符
.text:00408161                 jz      short loc_4081A3 ; 是0则跳到lea eax,[ecx-1]可见strlen()不会包括0字符长度的
.text:00408163                 test    ecx, 3          ;再次判断下一个地址是否进行了对齐如果已对齐则直接跳到main_loop中
.text:00408169                 jnz     short str_misaligned ; 取一个字符
.text:0040816B                 add     eax, 0
.text:00408170
.text:00408170 main_loop:                              ; CODE XREF: _strlen+Aj
.text:00408170                                         ; _strlen+36j ...
.text:00408170                 mov     eax, [ecx]      ; 这里判断4字节字符串中是否有0字符
.text:00408172                 mov     edx, 7EFEFEFFh
.text:00408177                 add     edx, eax
.text:00408179                 xor     eax, 0FFFFFFFFh
.text:0040817C                 xor     eax, edx
.text:0040817E                 add     ecx, 4          ; while
.text:0040817E                                         ; {
.text:0040817E                                         ; int x = xxxxx;
.text:0040817E                                         ; int y = x + 0x7efefeff;
.text:0040817E                                         ; x ^= 0xffffffff;
.text:0040817E                                         ; x ^= y;
.text:0040817E                                         ; x & 0x81010100  ==0表示无0字符串，相反则表示发现字符串
.text:00408186                 jz      short main_loop
.text:00408188                 mov     eax, [ecx-4]    ; 比较第一个字符是否为0
.text:0040818B                 test    al, al
.text:0040818D                 jz      short loc_4081C1
.text:0040818F                 test    ah, ah          ; 比较第二个字符是否为0
.text:00408191                 jz      short loc_4081B7
.text:00408193                 test    eax, 0FF0000h   ; 比较第三个字符是否为0
.text:00408198                 jz      short loc_4081AD
.text:0040819A                 test    eax, 0FF000000h ; 比较第4个字符是否为0
.text:0040819F                 jz      short loc_4081A3
.text:004081A1                 jmp     short main_loop
.text:004081A3 ; ---------------------------------------------------------------------------
.text:004081A3
.text:004081A3 loc_4081A3:                             ; CODE XREF: _strlen+11j
.text:004081A3                                         ; _strlen+4Fj
.text:004081A3                 lea     eax, [ecx-1]
.text:004081A6                 mov     ecx, [esp+arg_0]
.text:004081AA                 sub     eax, ecx
.text:004081AC                 retn
.text:004081AD ; ---------------------------------------------------------------------------
.text:004081AD
.text:004081AD loc_4081AD:                             ; CODE XREF: _strlen+48j
.text:004081AD                 lea     eax, [ecx-2]
.text:004081B0                 mov     ecx, [esp+arg_0]
.text:004081B4                 sub     eax, ecx
.text:004081B6                 retn
.text:004081B7 ; ---------------------------------------------------------------------------
.text:004081B7
.text:004081B7 loc_4081B7:                             ; CODE XREF: _strlen+41j
.text:004081B7                 lea     eax, [ecx-3]    ; 得到ecx-4 的地址相减即可
.text:004081BA                 mov     ecx, [esp+arg_0]
.text:004081BE                 sub     eax, ecx
.text:004081C0                 retn
.text:004081C1 ; ---------------------------------------------------------------------------
.text:004081C1
.text:004081C1 loc_4081C1:                             ; CODE XREF: _strlen+3Dj
.text:004081C1                 lea     eax, [ecx-4]    ; 得到ecx-4的地址
.text:004081C4                 mov     ecx, [esp+arg_0] ; 得到字串起始地址
.text:004081C8                 sub     eax, ecx        ; 减去地址
.text:004081CA                 retn                    ; 返回数值
.text:004081CA _strlen         endp
.text:004081CA
```

//vc6.0中的`strlen()`

### 关于对齐的问题

x86如果地址不对齐则影响程序执行速度，在x64中会引发一个异常，在其它的一些硬件平台比如mips则会引发bus error

x86汇编编译器中有两个伪指令用于对齐——even align

even  偶对齐

```
if (addr % 2==0) return addr; else return addr + 1;
```

align num  //num必须是2^n次方

```
if ((num & (num-1))==0)
{
   if (addr %num==0) return addr; else return (addr / num + 1) *num;
}
```

//最后逆向出来的函数如下:

```c
int  mystrlen(char *p)
{
        int str = *(int *)p;
        int y = 0x7efefeff;
        int count = -1;                                //为了方便操作使用计数器，初始为-1与字符数组的序号相对应
        while (1)
        {
                y += str;                        //这里比较复杂，vc6.0的C库中很多地方都用到了这个判断方式
                str ^= 0xffffffff;
                str ^= y;
                if (str &= 0x81010100)                //用于判断整型中是否有0字符
                {
                        if ((str>>24)==0) return count;                        //0号字符为0 没有字符
                        else if ((str & 0x00ff0000)==0) return count+1; //1号字符为0 有1个字符
                        else if ((str & 0x0000ff00)==0) return count+2; //2号字符为0 有2个字符
                        else if ((str & 0x000000ff)==0) return count+3; //3号字符为0 有3个字符
                }
                else
                {
                        (int *)p++;
                        count += 4;
                }
        }
}
```

//逆向扫描0x80来判断0号位置的算法

```c
unsigned ZeroByte(int x)
{
        int y = (x & 0x7f7f7f7f) + 0x7f7f7f7f;                        //1xxxxxxx
        y = ~(y | x | 0x7f7f7f7f);                                //80xxxxxx
        if (y==0) return 4;                                        //如果y==0说明是4个字符串
        else if (y>0x0000ffff) return (y>>31) ^ 1;                //如果y>0x0000ffff即0个或1号位为0字串，然后右移31位就是0字串的值，与1进行二进制相加s
                                                                //如果为0表示0x80在0号位置，即最高位异或得0，如果为1则0x80在2号位置。
        else return (y>>15) ^ 3;                                //同理
}
```

//直接用指针检测

```c
size_t mystrlen(char  const *str)
{
        char const *p = str;
        while (*p++);
        return (p - str - 1);
}
```

//vc6.0的release版程序中对于`strlen()`的调用进行了优化。

```asm
00401080    57                 push edi                                                       ; 保存edi原数据
00401081    BF 20304000        mov edi,api.00403020                                           ; edi = 字符串起始地址
00401086    83C9 FF            or ecx,FFFFFFFF                                                      ;设置ecx = 0xffffffff 计数器从-1开始
00401089    33C0               xor eax,eax                                                      ;eax清0
0040108B    F2:AE              repne scas byte ptr es:[edi]                                      ;ecx!=0 ecx=ecx-1 scas edi与al
0040108D    F7D1               not ecx                                                              ;把ecx求反得出repne操作的次数
0040108F    49                 dec ecx                                                              ;ecx-1减去0号字符串(因为repne先对计数器进行操作再执行scas)
```

这个代码很棒，有一种补码的思想在里面，任何有限重复操作都可以用补码表示，对于计算机中的数来说，存储上无符号，它就是一个太极（模），超过模就减去模来表
示，逻辑上有正数和负数之分，正数为阳 负数为阴，超过模也是减去模进行操作，负数与模相加用正数表示，总的来说一句话——太极 与 阴阳 合称太极。太极为阴为常，
阴阳为阳为动态。不要把阴阳看成太极分出来的，实际太极与阴阳组成另一个太极。

//`strlen`函数

```asm
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

### C++循环控制结构中的前++与后++问题

在for循环中前++与后++是相同的，但是在while循环中关系到条件判断 它们还是区别的

//示例代码

```c
int main()
{
        int n = 3;
        while (n--)
        {
                cout<<n<<endl;
        }
```
