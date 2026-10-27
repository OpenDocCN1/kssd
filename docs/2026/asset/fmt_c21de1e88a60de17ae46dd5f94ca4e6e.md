
只允许使用位运算`~`和`&`实现按位异或，这个我们在上面有写过，只不过没有合在一起，稍微转换一下即可，反正实现的思想就是要位之间不一样才返回`1`。

#### 答案

```c
int bitXor(int x, int y) {
  return ~(~(~x&y)&~(x&~y));
}
```

### tmin

```c
/*
 * tmin - return minimum two's complement integer
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 2;
}
```

获得整数的最小值，最小值就是我们喜闻乐见的`-2147483648`啦，但是我们稍微转换一下思路，最小就是最高位一个`0`，用个`1`，然后移位运算把它移到最高位即可。

#### 答案

```c
int tmin(void) {
  return 1<<31;
}
```

### isTmax

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
  return 2;
}
```

判断`x`是否为最大值，最大值应该是最小值取反，然后判断是否一样我们使用异或运算，异或只有在两边相等时才返回`0`，我们用非运算符反一下就可以了。

#### 答案

```c
int isTmax(int x) {
  return !((~(1<<31))^x);
}
```

### allOddBits

```c
/*
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  return 2;
}
```

判断`x`是否奇数位上全为`1`。全`1`我们直接用一个立即数好了`0xAAAAAAAA`，然后与屏蔽偶数位之后判断是否相等即可。

#### 答案

```c
int allOddBits(int x) {
  return !((0xAAAAAAAA&x)^0xAAAAAAAA);
}
```

### negate

```c
/*
 * negate - return -x
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return 2;
}
```

实现对一个数取负，取负的话就是取反再加一即可。

```c
int negate(int x) {
  return ~x+1;
}
```

### isAsciiDigit

```c
/*
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x){
  return 2;
}
```

判断一个整数是否在`ASCII`码的数字位置，再转换一下就是满足这个条件：

```c
x-0x30>=0 && x-0x3a<0
```

我们的目标就是判断`x-0x30`是否为非负数，`x-0x3a`是否为负数，也许需要考虑溢出，但是结果发现不需要，因为这个是两边判断的。

#### 答案

```c
int isAsciiDigit(int x){//x-0x30>=0 && x-0x3a<0
  return !((x+(~0x30+1))&(1<<31))&!!((x+(~0x3a+1))&(1<<31));
}
```

### conditional

```c
/*
 * conditional - same as x ? y : z
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  return 2;
}
```

相当于实现运算`x?y:z`，如果`x`为`0`则返回`x`，如果不为`0`则返回`y`。

我们先把`x`取值变为只有`0`和`-1`。用掩码的思路，如过结果为`-1`则说明位模式全为`1`，相与的结果就是另一个数本身，如果是`0`则不论与谁结果都是`0`，我们让`x`与`y`，让`~x`与`z`，最后得到的两个值或起来即可。`x`如果是`-1`则返回`y`，如果是`0`则返回`z`。得到`x`我们只要非两次，再取负即可，就达到了我们的要求了。

```c
int conditional(int x, int y, int z) {
  x=~(!!x)+1;
  return (x&y)|(~x&z);
}
```

### isLessOrEqual

```c
/*
 * isLessOrEqual - if x <= y  then return 1, else return 0
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  return 2;
}
```

实现一个小于等于的功能，这里需要注意的是不能直接相减，会有溢出。

因此我们选择先判断符号位，符号位一致再相减一定不会溢出，符号不一致则强行返回不一致的结果，一致则返回直接相减的结果。首先我们先拿两个变量保存符号的结果，然后再判断符号是否一致，一致则屏蔽符号的结果，不一致则屏蔽相减的结果。

这里好好再理解一下，仔细看`signx^signy`和`!(signx^signy)`如何起到屏蔽答案的效果。

#### 答案

```c
int isLessOrEqual(int x, int y) {//y-x>=0
  int signx=!(x&(1<<31));
  int signy=!(y&(1<<31));
  int sign=(signx^signy)&signy;//若不一致则返回y是否为非负数
  x=x&0x7fffffff;
  y=y&0x7fffffff;
  return sign|(!(((y+(~x+1))&(1<<31))))&(!(signx^signy));
}
```

### logicalNeg

```c
/*
 * logicalNeg - implement the ! operator, using all of
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4
 */
int logicalNeg(int x) {
  return 2;
}
```

实现逻辑非的功能，我们这里需要研究一下`0`的特性，`0`是除了`TMIN`以外的负数值等于自己本身的数了。然后再判断一下最高位即可。

#### 答案

```c
int logicalNeg(int x) {
  return ((x|(~x+1))>>31)+1;
}
```

### howManyBits

```c
/*
 * howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0) = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x) {
  return 2;
}
```

这题挺难的，首先要想到一个二分，然后要注意正数需要多一位，负数不需要。假如我给了你`0x7fffffff`，那么你肯定要`32`位，虽然我只有`31`位有效数字。我给你个`-1`那一位就够了，因为一位的范围就是`-1~0`。

我们先判断高`16`位是否有`1`，若有则判断高`16`位，答案加上`16`，否则判断低`16`位。

再判断的`16`位中，判断高`8`位是否有`1`，若有则继续判断高`8`位，答案加上`8`，否则判断低`8`位。

直到判完。我一开始想到了这个：

```c
int howManyBits(int x) {
  int sign=!(>>31);
  int b16=(!!((x>>16)&0xffff))<<4;
  x=x>>b16;
  x=x&0xffff;
  int b8=(!!((x>>8)&0xff))<<3;
  x=x>>b8;
  int b4=(!!((x>>4)&0xf))<<2;
  x=x>>b4;
  int b2=(!!((x>>2)&0x3))<<1;
  x=x>>b2;
  int b1=(!!((x>>1)&0x1))<<0;
  x=x>>b1;
  printf("%d %d %d %d %d %d %d\n",b16,b8,b4,b2,b1,x,sign);
  return b16+b8+b4+b2+b1+sign+x;
}
```

但是经过多方调试还是输给了`-1`这个测试点。后来想到如果负数则取反即可，然后符号位就默认给它加着上去即可。

#### 答案

```c
int howManyBits(int x) {
  int sign=x>>31;
  x = (sign&~x)|(~sign&x);
  int b16=(!!((x>>16)&0xffff))<<4;
  x=x>>b16;
  x=x&0xffff;
  int b8=(!!((x>>8)&0xff))<<3;
  x=x>>b8;
  int b4=(!!((x>>4)&0xf))<<2;
  x=x>>b4;
  int b2=(!!((x>>2)&0x3))<<1;
  x=x>>b2;
  int b1=(!!((x>>1)&0x1))<<0;
  x=x>>b1;
  return b16+b8+b4+b2+b1+X+1;
}
```

整数阶段到此完结了就。

### floatScale2

```c
```
