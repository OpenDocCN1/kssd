
```
push    ebp

mov     ebp, esp

sub     esp, 5B8h

mov     eax, large gs:14h

mov     [ebp+var_C], eax

xor     eax, eax
```
可以看到这里启用了`canary`保护

将内存`large gs:14h`中的(随机值)入栈

并在程序返回前对`canary`值进行检验：
```
nop

mov     eax, [ebp+var_C]

xor     eax, large gs:14h

jz      short locret_8049432
```
`canary`值在栈中位于返回地址和函数调用参数之间

从而保护了栈内数据，防止我们修改返回地址造成栈溢出

### 0x02 `_bzero`
`canary`入栈后`calc`调用了`_bzero`：
```
mov     dword ptr [esp+4], 400h

lea     eax, [ebp+s]

mov     [esp], eax      ; s

call    _bzero
```
这里从`ebp+s`开始将一段长为`0x400`的空间清零

### 0x03 `get_expr`
开辟一段数据后
`calc`调用了`get_expr`函数
```
mov     dword ptr [esp+4], 400h

lea     eax, [ebp+s]

mov     [esp], eax

call    get_expr
```
跟进`get_expr`后发现一堆判断跳转

大致过程：
```
过滤掉除"[0-9],+,-,×,/,%"外的其他字符

读入我们输入的表达式到_bzero开辟的空间中

当我们成功读入返回值不为0，calc跳转到loc_80493CC处：

test    eax, eax

jnz     short loc_80493CC
```

### 0x04 `init_pool`
接下来`calc`调用`init_pool`：
```
lea     eax, [ebp+var_5A0]

mov     [esp], eax

call    init_pool
```
`init_pool`：
```
.text:08048FF8                 push    ebp

.text:08048FF9                 mov     ebp, esp

.text:08048FFB                 sub     esp, 10h

.text:08048FFE                 mov     eax, [ebp+arg_0]

.text:08049001                 mov     dword ptr [eax], 0

.text:08049007                 mov     [ebp+var_4], 0

.text:0804900E                 jmp     short loc_8049022

.text:08049010 ; ---------------------------------------------------------------------------

.text:08049010

.text:08049010 loc_8049010:                            ; CODE XREF: init_pool+2E↓j

.text:08049010                 mov     eax, [ebp+arg_0]

.text:08049013                 mov     edx, [ebp+var_4]

.text:08049016                 mov     dword ptr [eax+edx*4+4], 0

.text:0804901E                 add     [ebp+var_4], 1

.text:08049022

.text:08049022 loc_8049022:                            ; CODE XREF: init_pool+16↑j

.text:08049022                 cmp     [ebp+var_4], 63h

.text:08049026                 jle     short loc_8049010

.text:08049028                 leave

.text:08049029                 retn
```
很简短的一个过程：
```
从ebp+var_5A0开始

将长度为63h的空间清零
```

### 0x05 `parse_expr`
接下来`calc`调用 `parse_expr`函数：
```
lea     eax, [ebp+var_5A0]

mov     [esp+4], eax

lea     eax, [ebp+s]

mov     [esp], eax

call    parse_expr
```
可以看到其参数：
```
init_pool清零的那段空间的首地址：ebp+var_5A0

对应读入表达式的首地址：ebp+s
```
首先`F5`分析一下`parse_expr`的伪代码（分析在注释处）：
```
signed int __cdecl parse_expr(int a1, _DWORD *a2)

{

  int v2; // ST2C_4

  int v4; // eax

  int v5; // [esp+20h] [ebp-88h]

  int i; // [esp+24h] [ebp-84h]

  int v7; // [esp+28h] [ebp-80h]

  char *s1; // [esp+30h] [ebp-78h]

  int v9; // [esp+34h] [ebp-74h]

  char s[100]; // [esp+38h] [ebp-70h]

  unsigned int v11; // [esp+9Ch] [ebp-Ch]



  v11 = __readgsdword(0x14u);

  v5 = a1;

  v7 = 0;

  bzero(s, 0x64u);

  for ( i = 0; ; ++i )

  {

    if ( (unsigned int)(*(char *)(i + a1) - 48) > 9 )// 比对ascii并转换成unsigned int后，检验是否为运算符

    {

      v2 = i + a1 - v5;                         // 运算符左操作数长度

      s1 = (char *)malloc(v2 + 1);

      memcpy(s1, v5, v2);

      s1[v2] = 0;

      if ( !strcmp(s1, "0") )                   // 判断运算符左边操作数是否为0

      {

        puts("prevent division by zero");

        fflush(stdout);

        return 0;

      }

      v9 = atoi((int)s1);                       // 将读入的操作数由字符串转化为int

      if ( v9 > 0 )

      {

        v4 = (*a2)++;                           // a2[0]保存操作数个数

        a2[v4 + 1] = v9;                        // 将第二个操作数存入第二次开辟的那段空间

      }

      if ( *(_BYTE *)(i + a1) && (unsigned int)(*(char *)(i + 1 + a1) - 48) > 9 )// 判断是否两个运算符连续

      {

        puts("expression error!");

        fflush(stdout);

        return 0;

      }

      v5 = i + 1 + a1;                          // v5指向运算符后一个字符，构造下一个循环

      if ( s[v7] )                              // 判断是否为第一个操作数（对上一个操作符进行判断）

      {

        switch ( *(char *)(i + a1) )

        {

          case 37:

          case 42:

          case 47:

            if ( s[v7] != 43 && s[v7] != 45 )   // 判断运算是否为加减从而确定运算顺序

            {

              eval(a2, s[v7]);

              s[v7] = *(_BYTE *)(i + a1);

            }

            else

            {

              s[++v7] = *(_BYTE *)(i + a1);

            }

            break;

          case 43:

          case 45:

            eval(a2, s[v7]);

            s[v7] = *(_BYTE *)(i + a1);

            break;

          default:

            eval(a2, s[v7--]);                  // 保证了最后while时运算符右边的优先级大于左边

            break;

        }

      }

      else                                      // 若此操作符不是第一个操作符，则读入s[v7]中

      {

        s[v7] = *(_BYTE *)(i + a1);

      }

      if ( !*(_BYTE *)(i + a1) )                // 字符串结尾

        break;

    }

  }

  while ( v7 >= 0 )

    eval(a2, s[v7--]);                          // 将因优先级问题没有计算的运算从右向左依次计算

  return 1;

}
```
除此之外，这里调用了`eval`函数来进行计算：
```
_DWORD *__cdecl eval(_DWORD *a1, char a2)

{

  _DWORD *result; // eax



  if ( a2 == 43 )

  {

    a1[*a1 - 1] += a1[*a1];

  }

  else if ( a2 > 43 )

  {

    if ( a2 == 45 )

    {

      a1[*a1 - 1] -= a1[*a1];

    }

    else if ( a2 == 47 )

    {

      a1[*a1 - 1] /= a1[*a1];

    }

  }

  else if ( a2 == 42 )

  {

    a1[*a1 - 1] *= a1[*a1];

  }

  result = a1;

  --*a1;

  return result;

}
```
可以看到：
```
init_pool中开辟的空间依次保存操作数（即calc中的：var_59C= dword ptr -59Ch）（开始位置保存操作数个数）

parse_expr中新开辟的空间s保存运算符

a2[*a2]处保存表达式最终结果
```

## 0x02 漏洞
在`parse_expr`中分析：

正常情况下最终应该在`a2[1]`处的值为结果

可当考虑到第一个字符即为运算符的情况下：

例如：`+10`
```
*a2=1(一个操作数)

a2[1]=10

s[0]='+'

a2[*a2-1]=a2[*a2-1]+a2[*a2]

即：a2[0]=a2[0]+a2[1]=11

而后--*a2,即：*a2=10

最终输出结果为a2[*a2]=a2[10]

这里注意*a2与 init_pool中开辟的63h长度的地址是连续的，记 init_pool中地址为a3的话

那么如果最后输出a3[*a2-1]=a2[*a2]
```
同样地：
```
如果+10+1

则会使：a2[10]=a2[10]+1

并输出a2[10]

那么当我们选取恰当大小的操作数即可绕过canary修改返回地址，从而实现溢出
```
这里注意：
```
每一次循环都会重新调用前面两个清零的函数，我们修改这里的数据，下一次依然会清零（不过这段地址外数据（包括我们要修改的返回地址）不会清零，可以修改）
```
我们查看一下程序的保护机制：
```
checksec  --file ./calc
```
发现：
 ![](https://bbs.kanxue.com/upload/attach/201902/3_RBAY5YE6VY5BXSX.png)

这里开启了`NX`保护

我们无法在栈上执行`shellcode`拿到`shell`

同时看到这里：
```
objdump -R ./clac
```
![](https://bbs.kanxue.com/upload/attach/201902/3_F2HPYA6NMZWYZPJ.png)

程序是静态链接

我们这里考虑利用`ROP`调用`sys_execve`来获得`shell`

## 0x03 ROP
首先计算出返回地址与`*a2`的距离
```
0x5A0+0x4=1444

1444/4=361
```
故而:
```
输入+361时反回的即时calc的返回地址

我们需要连续修改a2[361]后的一段栈内数据来构造ROP链
```
我们最终需要：
```
ebx="/bin/sh"字符串首地址

ecx=0

eax=0xb
```
我们需要构造一段栈内数据：
```
addr(pop eax;ret)->0xb->addr(pop ecx;popebx,ret)->0->addr"/bin/sh"->addr(int 80h)->"/bin/sh"
```
利用`ROPgadget`找到我们需要指令的地址：
```
ROPgadget --binary ./calc  --ropchain
```
![](https://bbs.kanxue.com/upload/attach/201902/3_KK8KKMM8JV2TEEE.png)

下面：
```
我们需要先通过找到栈中对应位置的值计算出我们需要的差值

利用差值将从返回地址开始的一段栈数据修改成我们需要的值

例如：

我们先修改+361处的值

+361处需要修改为addr(pop eax;ret)（pop eax;ret指令地址）

假设pop eax;ret指令地址为：0x1

我们输入"+361"，返回：0x0

它与我们需要的值差值为0x1-0x0=1

我们输入+361+1

即可修改+361处值为我们需要的0x1
```
注意：
**其中`/bin/sh`字符串我们只知道其在栈中的相对地址，这里需要我们先取得`main`函数的`ebp`地址（我们取得`+360(main函数基地址)`是负数，需要`+0x100000000`转换后运算，再在最后`-0x100000000`修改对应位置值）**

在`main`中：
```
and     esp, 0FFFFFFF0h

sub     esp, 10h
```
故而返回地址即在：
```
addr_re=([ebp]&0xfffffff0)-16   #注意脚本书写时运算优先级"+">"&"
```
**而后再根据我们最后在栈内构造的字符串`"/bin/sh"`与返回地址的相对位置计算出字符串`"/bin/sh"`的地址即可**

## 0x04 EXP
