
# [原创]RSA的数学原理与实现

经常看到各位大佬挥毫泼墨，就算法和密码问题各抒己见，小弟今天斗胆也来上一课。

关于RSA的数学原理和实现，本文根据数论中的知识给出了证明，然后是自己手撸的64位RSA算法。

## 1. 基础知识

辗转相除法：

$$若a = bx + c,则(a,b) = (b,c)$$

证明略。

欧拉函数：

$$ \phi(n) = n\*(1 - 1 /p)(1 - 1/q) = (p - 1)\*(q - 1) $$

证明自行百度。

## 2. RSA加解密的条件

1. `p`和`q`是两个大素数，欧拉函数值`$\phi(n)=n(1 -\frac{1}{p})(1 - \frac{1}{q})=(p-1)(q-1)$`
2. `w`与`$\phi(n)$`互素，`d`是`w`的模欧拉值逆,即`dw=1(mod n)`

## 3. 加解密过程

加密：$$c = E(m) = m ^w mod \quad n $$

解密：$$m = D(c) = c^d mod \quad n$$

## 4. 证明过程：

$$(m^w mod\quad n)^d mod \quad n = m^{wd} mod \quad n\\ [\\ 证明：\\ 假设m^w = a n + b,则(m^w mod \quad n)^d mod \quad n = b ^ d mod \quad n\\ 而m^{wd} mod \quad n = (an+b)^d mod \quad n = (an)^n + \mathrm{C}\_d^{n-1}(an)^{n-1}b + ... + b^d mod \quad n = b^d mod \quad n\\ ] $$

所以，要证明加解密过程的正确性，即证明：

$$ m^{wd} mod \quad n = m$$

因为：

$$wd \equiv 1(mod \quad \phi(n))$$

所以

$$wd = A \phi(n) + 1$$

### 1. 若`m`与`n`互素

根据欧拉定理可得：

$$m^{\phi(n)} \equiv 1(mod \quad n) \\ m^{k \phi(n)+1} \equiv m(mod \quad n)$$

### 2. 若`m`与`n`不互素

因为`m`是合数，`n=pq`, 根据算数基本定理，`m`必包含`p`或者`q`，假设`m`包含`q`,则根据`fermat`定理可得，

$$m ^{p -1} \equiv 1(mod \quad p)$$

等式得证。

屈婉玲等几位老师的《离散数学》证明如下：

![图片描述](upload/attach/202304/653410\_MJG2C2Z5PRKC95N.png)

## 例子程序(仅支持64位以内的密钥长度）

代码是自己根据数学公式一行一行写的，因为对照公式可以很容易识别各个函数的用途，所以没写注释，请大家见谅。

只是为了学习验证，可能有所缺陷，敬请批评指正。

### c++文件：

```
#include "rsa.h"
#include <windows.h>
#include <stdio.h>
myInt64 g\_n = 0;
myInt64 g\_d = 0;
myInt64 g\_w = 0;
int g\_blocksize = 0;
int g\_blocksize\_src = 1;
myInt64 pow\_i\_old(myInt64 a, myInt64 b, myInt64 p) {
myInt64 res = 1;
for (myInt64 i = 0; i < b; i++)
{
res = (res \* a) % p;
if (res == 0) //caution here: avert to overlow
{
break;
}
}
return res;
}
myInt64 pow\_i(myInt64 x, myInt64 n, myInt64 p)
{
if (n == 0)
{
return 1;
}
myInt64 temp = pow\_i((x \* x) % p, n / 2, p); //递归计算（X\*X）^[N/2]
if ((n & 1) != 0) //判断n的奇偶性
{
temp = (temp \* x) % p;
}
return temp;
}
myInt64 pow\_ii(myInt64 a, myInt64 b, myInt64 g\_n) {
myInt64 res = 1;
for (myInt64 i = 0; i < b; i++)
{
res = (res \* a) % g\_n;
if (res == 0) //caution here: avert to overlow
{
//break;
}
}
return res;
}
int isMutualPrime(myInt64 n, myInt64 m) {
return 1;
}
int isPrimeNumber(myInt64 n) {
for (myInt64 i = 2; i < n; i++)
{
myInt64 quotient = n % i;
if (quotient == 0)
{
return FALSE;
}
}
return TRUE;
}
int rsaInit(myInt64 p, myInt64 q)
{
if (p == q || p <= 2 || q <= 2 || isPrimeNumber(p) == 0 || isPrimeNumber(q) == 0)
{
return FALSE;
}
g\_n = p \* q;
if (g\_n < 128)
{
return FALSE;
}
myInt64 mi64 = 0x8000000000000000;
int bh = 63;
for (int i = 63; i >= 0; i--)
{
if (g\_n & mi64)
{
bh = i;
break;
}
mi64 = mi64 >> 1;
}
if (bh >= 32)
{
g\_blocksize = 8;
}
else if (bh >= 16)
{
g\_blocksize = 4;
}
else if (bh >= 8)
{
g\_blocksize = 2;
}
else
{
g\_blocksize = 1;
}
myInt64 fai\_n = (p - 1) \* (q - 1);
myInt64 w = 2;
myInt64 d = 2;
for (w = 2; w < fai\_n; w++)
{
int result = isPrimeNumber(w);
if (result)
{
for (d = 2; d < fai\_n; d++)
{
result = isPrimeNumber(w);
if (result) {
myInt64 m = w \* d;
if ((d != w) && (m % fai\_n == 1))
{
g\_d = d;
g\_w = w;
return TRUE;
}
}
}
}
}
return FALSE;
}
//g\_w is used to encrypt and g\_d used to decrypt,and g\_w must be primer to fai(n)
int rsa\_encrypt(char\* data, int size, char\* dst) {
if (g\_n == 0 || g\_w == 0 || g\_d == 0)
{
return FALSE;
}
int block\_cnt = size / g\_blocksize\_src;
int mod = size % g\_blocksize\_src;
unsigned char\* s = (unsigned char\*)data;
myInt64\* d = (myInt64\*)dst;
for (int i = block\_cnt; i > 0; i--)
{
myInt64 tmp = 0;
if (g\_blocksize == 8)
{
tmp = \*s;
tmp = (tmp & 0xffffffffffffffff);
tmp = (pow\_i\_old(tmp, g\_w, g\_n));
\*d = tmp;
}
else if (g\_blocksize == 4)
{
tmp = \*s;
tmp = tmp & 0xffffffff;
tmp = (pow\_i\_old(tmp, g\_w, g\_n));
\*(DWORD\*)d = \*(DWORD\*)&tmp;
}
else if (g\_blocksize == 2)
{
tmp = \*s;
tmp = tmp & 0xffff;
tmp = (pow\_i\_old(tmp, g\_w, g\_n));
\*(WORD\*)d = \*(WORD\*)&tmp;
}
else if (g\_blocksize == 1)
{
tmp = \*(unsigned char\*)s;
tmp = (tmp & 0xff);
tmp = (pow\_i\_old(tmp, g\_w, g\_n));
\*(unsigned char\*)d = \*(unsigned char\*)&tmp;
}
s = (unsigned char\*)((unsigned char\*)s + g\_blocksize\_src);
d = (myInt64\*)((unsigned char\*)d + g\_blocksize);
}
if (mod)
{
myInt64 tmp = 0;
memcpy(&tmp, s, mod);
if (g\_blocksize\_src == 8)
{
tmp = tmp & 0xffffffffffffffff;
}
else if (g\_blocksize\_src == 4)
{
tmp = tmp & 0xffffffff;
}
else if (g\_blocksize\_src == 2)
{
tmp = tmp & 0xffff;
}
else if (g\_blocksize\_src == 1)
{
tmp = tmp & 0xff;
}
tmp = pow\_i\_old(tmp, g\_w, g\_n);
memcpy(d, &tmp, mod);
}
return size;
}
int rsa\_decrypt(char\* data, int size, char\* dst) {
if (g\_n == 0 || g\_w == 0 || g\_d == 0)
{
return FALSE;
}
int block\_cnt = size / g\_blocksize\_src;
int mod = size % g\_blocksize\_src;
myInt64\* s = (myInt64\*)data;
unsigned char\* d = (unsigned char\*)dst;
for (int i = block\_cnt; i > 0; i--)
{
myInt64 tmp = 0;
if (g\_blocksize == 8)
{
tmp = \*s;
tmp = (tmp & 0xffffffffffffffff);
tmp = (pow\_i\_old(tmp, g\_d, g\_n));
\*d = (unsigned char)tmp;
}
else if (g\_blocksize == 4)
{
tmp = \*(DWORD\*)s;
tmp = tmp & 0xffffffff;
tmp = (pow\_i\_old(tmp, g\_d, g\_n));
\*d = (unsigned char)tmp;
}
else if (g\_blocksize == 2)
{
tmp = \*(WORD\*)s;
tmp = tmp & 0xffff;
tmp = (pow\_i\_old(tmp, g\_d, g\_n));
\*d = (unsigned char)tmp;
}
else if (g\_blocksize == 1)
{
tmp = \*(unsigned char\*)s;
tmp = (tmp & 0xff);
tmp = (pow\_i\_old(tmp, g\_d, g\_n));
\*d = (unsigned char)tmp;
}
s = (myInt64\*)((unsigned char\*)s + g\_blocksize);
d = (unsigned char\*)((unsigned char\*)d + g\_blocksize\_src);
}
if (mod)
{
myInt64 tmp = 0;
memcpy(&tmp, s, mod);
if (g\_blocksize == 8)
{
tmp = tmp & 0xffffffffffffffff;
}
else if (g\_blocksize == 4)
{
tmp = tmp & 0xffffffff;
}
else if (g\_blocksize == 2)
{
tmp = tmp & 0xffff;
}
else if (g\_blocksize == 1)
{
tmp = tmp & 0xff;
}
tmp = pow\_i\_old(tmp, g\_d, g\_n);
memcpy(d, &tmp, mod);
}
return size;
}
```

### 头文件：

```
#pragma once
#define myInt64 unsigned long long
myInt64 pow\_i(myInt64 a, myInt64 b);
int rsaInit(myInt64 p, myInt64 q);
int rsa\_encrypt(char\* data, int size, char\* dst);
int rsa\_decrypt(char\* data, int size, char\* dst);
```

### 调用方式

在`main`函数中执行如下代码，`main`函数就先省略了：

```
rsaInit(11, 23);
const char\* myteststr = "Hello!\r\n Hi! \r\nHow are you ?\r\n Fine,thank you, and you ? \r\nI am fine, too!\r\n";
int teststrlen = lstrlenA(myteststr);
char dst0[1024];
rsa\_encrypt((char\*)myteststr, teststrlen + 1, dst0);
char dst1[1024];
rsa\_decrypt((char\*)dst0, teststrlen + 1, dst1);
```

## 问题和注意事项：

1. `pow_i`函数中为什么要取余数？为了防止溢出？数学知识不扎实。
2. 加密之前与解密之后的`bit`位数应该长度相同，加密之后的`bit`数与解密之前的`bit`位长度应该相同。
3. 下一期将会讲解一下离散素数相关的`DSA`、`ecdh`等加密原理和简化的代码例子。
