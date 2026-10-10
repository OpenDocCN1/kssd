
`#define TABLE_SIZE 32`
`#define SERIAL_SIZE 0x1000`
`char name[0x10];`
`char serial[SERIAL_SIZE];`
`unsigned char nametable[10];`
`int namelen = 0, seriallen = 0;`

```
void calc_alpha(unsigned char *xor, char *result_str)
{
    int i;
    unsigned char xor0[10] = {0x25,0x9a,0xf3,0x6f,0x82,0xda,0x72,0xfe,0xc9,0xb7};
    for(i=0; i<10; i++)  result_str[i] = xor0[i]^xor[i];
}
char buf[0x20];
int checkserial( char *name,char *serial)
{
    int i;
    int remainder;
    unsigned namecalc = 0x13572468;
    unsigned adder,adder2;
    unsigned char xorfail[10] = {0x63,0xfb,0x9a,0x03,0xa3,0xda,0x72,0xfe,0xc9,0xb7};  //fail!
    unsigned char xorok[10] = {0x6a,0xd1,0xd2,0x4e,0x82,0xda,0x72,0xfe,0xc9,0xb7};  //OK!!
    memset(nametable,0,0xa);
    for(i=0; i<namelen; i++)
    {
        adder = (name[i]+ namecalc)*0x3721273+0x24681357;
        adder2 = adder<<0x19;
        __asm sar adder,7
        namecalc = adder2|adder;
    }
    for (i=1; i<9; i++)
        nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);
    nametable[9] = 1;
    for (i=0; i<seriallen; i++)
    {
      if(serial[i]<'0' || serial[i]>'9') goto failinput;
      remainder = (serial[i]-0x30+ ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10))%10;
      if(remainder != 1)
      {
        if(nametable[remainder-1]==1)
        {
          int calc = remainder-2;
          while(calc >=1)
              if(nametable[calc--]==1) goto fail;
          nametable[remainder]^=1;
        }
        else goto fail;
      }
      else nametable[1] ^=1;
    }
    for (i=1; i<10; i++)
        if(nametable[i]==1) goto failedcheck;
    goto success;
failinput:
fail:
failedcheck:
    calc_alpha(xorfail,buf); return 1;
success:
    calc_alpha(xorok,buf); return 0;
}
```

---

稍微总结一下验证过程，循环读入序列号，根据序列号对 `nametable` 的对应位置取反（其中位置判断是否序列号有效），最后序列号结束后判断是否 `nametable` 全为 0，否则失败。

另外这个题逆推回去有些不容易。

# 第一阶段第一题注册机思路+完整注册机代码 看雪金山 2007 逆向分析挑战赛

by `aker`

9:10 2007-8-24

上一篇中分析了看雪金山 2007 逆向分析挑战赛第一阶段第一题 crackme 的代码，并实际动手逆向实现了该 crackme。

主要函数为 `int checkserial( char *name,char *serial);` 该函数接受用户名和序列号，通过一系列变换判断是否变换成功。

本篇中分析如何根据关键代码流程写出注册机，附件是注册机及代码。

关键代码的操作过程如下：

循环操作如下：
1. 依次加载序列号的每一个字节。
2. 判断是否为数字字符 `if(serial[i]<'0' || serial[i]>'9')`，不是数字字符就跳转。
3. 循环计算 32 位整数就是刚才计算的用户名特征码的 1/2 模 10 的余数，每次都多计算其 1/2，所以可以看到这个数是以 32 位为循环的。
4. 该数字加上刚刚的数字字符串代表的数字再模 10，以这个余数为索引到开始计算的名字特征值的名字表中作变换。
5. 下面是变换过程
   5.1 如果为 1，则一号位异或 1，见 `004003BE`，`cmp edx,ebx`
   5.2 否则异或该位置，但是需要该位置的前一个位置为 1，再前面所有数据为 0,不满足就报错

循环结束后
6 检查名字特征值的名字表中的值，需要都为 0 才表示序列号为真。

实际代码如下：

```
    // check
    for (i=0; i<seriallen; i++)
    {
      if(serial[i]<'0' || serial[i]>'9') goto failinput;
      remainder = (serial[i]-0x30+ ((namecalc>>(unsigned char)(i%(TABLE_SIZE-1)))%10))%10;
      if(remainder != 1)
      {
        if(nametable[remainder-1]==1)
        {
          int calc = remainder-2;
          while(calc >=1)
              if(nametable[calc--]==1) goto fail;
          nametable[remainder]^=1;
        }
        else goto fail;
      }
      else nametable[1] ^=1;
    }
    for (i=1; i<10; i++)
        if(nametable[i]==1) goto failedcheck;
    goto success;
```

昨天这个地方我首先走了弯路，也介绍下：

从该验证过程可以看到，每次读入一位，然后该位肯定为 `'0' - '9'` 之间的一位数，所以想最简单的就是写一个外壳，对序列号从 1-0x1000 验证，每次 `'0' - '9'` 变换就是了,原形如下，当时将 `getserial` 的返回值作了分类，`fail` 表示错误，不可接受，`failedcheck;` 表示前面都对，但是最后验证错误，`success` 表示成功，这样，我就可以区分是不是需要对序列号数字加一。但是实际上却出问题了，序列号怎么都不能最终使 `nametable` 全为 0，但是做了这么久，又有点不想放弃这个思路。傍晚刚好停了一下电，出去吃了晚饭，回来电来了，网络断了，正好可以不看论坛:)换个思路做做看。

```
char buf[ERRO_SIZE];
int shellgo(unsigned namecalc)
{
    int i;
    enum result = OTHER;;
    while(result)
    {
        for (i=0; i<10 && result!=FAILEDCHECK && result!=SUCCESS;; i++)
        {
            serial[seriallen-1] = (0x30+i);
            result = getserial(namecalc,serial);
        }
        if(result != SUCCESS;; ) result = OTHER;
        seriallen++;
        if(seriallen>=0x1000)
        {
            wsprintf(buf,"exceeds..............\n");
            return -1;
        }
    }
    return 0;
}
```

仔细看了代码，发现一个咚咚，就是 `((namecalc>>(unsigned char)(i%31))%10)` 得变化对于每个用户名都是一样的循环变化。

也就是说可以拿出来，先生成。还有就是名字特征的 1-8 位的数组都是对每个用户名一样的。

```
// 定义一个 32 位的数组
unsigned char table[32];
for (i=0; i<32; i++) 
    table[i] = ((namecalc>>(unsigned char)(i%31))%10);
// 另外这个是名字特征的 1-8 位的数组，加上最后一位固定为 1
for (i=1; i<9; i++)
    nametable[i] = (unsigned char)((unsigned char)(namecalc>>i)&1);
nametable[9] = 1;
```

现在就容易懂了，每次从 `table` 中循环选取一个数字，和当前取到的序列号相加，模 10 就可以得到名字特征表的位置。

然后对该位置异或。

但是该位置选取有很多限制：
见步骤 5.1,5.2
5.1 如果为 1，则 1 号位异或 1，见 `004003BE`，`cmp edx,ebx`
5.2 否则异或该位置，但是需要该位置的前一个位置为 1，而且再前面所有数据为 0,不满足就报错

说简单点就是，如果位置为 1，则 1 号位取反，否则，要保证当前位置前面一位为 1，而且再前面所有数据为 0，都满足就可以对该位取反。

实例：对名字 `aker` 有
`name: aker`
`nametable: *110000111`
`pos: 0123456789`
`table: 89942689999426310005789942631528`

比如第一个序列号为 3，则，可以知道 `(3+8)%10 == 1`,所以对第一位取反。
`nametable` 为 `*010000111`
如果不为 3，假定为 5，则余数为 3，2 号位为 1，但是 1 号位也为 1，所以报错。
如果不为 3，假定为 4，则余数为 2，1 号位为 1，前面是 0 号，所以也可以进去，但是我们可以发现，如果随便选取一个的话，有可能进入死循环，出现很多序列号前后完全重复的情况。

序列号按照上面的步骤如果位置为 1，则 1 号位取反，否则，要保证当前位置前面一位为 1，而且再前面所有数据为 0，都满足就可以对该位取反。使得最后一个序列号完成的时候，`nametable` 中全为 0。这样就表示序列号为真。

一些细节：序列号一次和 `table` 中的数字相加，得到的余数就是在 `nametable` 中的位置。

好了，我们现在应该清楚如何检验序列号正确性的问题了。

稍微总结一下上面所说的：
上面介绍了序列号如何验证，主要就是通过序列号使对应 `nametable` 为 0。
变换可以写成下式：
`nametable[(serial[i]+table[i%31])%10]^=1;` 而且此时 `(serial[i]+table[i%31])%10` 要么为 1，要么在最左边的 1 的下一位，持续该变换过程可使 `nametable` 为 0。

下面就是问题的关键了：
问题是，我们如何通过 `table` 和 `nametable` 找 `serial` 呢？
根据上面的变换过程我们可以知道，每次异或位置只有两个，1，或者是最左边的 1 的右边。
那现在的问题就是判断怎么样选取位置了，估计很多人就卡在这里了:)

其实仔细想一下可以明白，肯定要交互选取，这次 1，下次就是最左边的 1 的右边，为什么呢？
因为如果你不交互选取的话，等于是回退了，比如你这次选了 1，下次还选 1 异或，就变为原来的数字了；或者你这次选了最左边的 1 的右边，那么下次再选，肯定还是这个位置，你再异或不是又回去了不，估计不少看官都恍然大悟了;)

那么，最开始该选 1 好位置还是选最左边的 1 的右边呢？
结论是根据 `nametable` 中 1 的个数而不同，奇数选 1，偶数选最左边 1 的右边。
这个我不能明白的证明给大家看，自己能够理解，但是感觉说不出来，这样吧，我举例子：

比如，最简单的，只有一个 1 的情况，假设 1 在最右边，你说这个时候该怎么选择位置呢？你说白痴才不知道，根据上面的条件，只能选最左边的 1 啊，好，你说对了；再假定 1 在最左边，你说位置该怎么选呢？你说，"当然最左边啦，选 1 号位就直接得结果了，你不急我都要替你急了":P。再假定 1 个 1 在 2-8 的任何位置呢？你说还是一样啊，肯定选最左边的 1，选右边，数字又变大了，什么时候才能变回来啊。要变除非再选哪个位置;)
好了 1 个 1 的情况很清楚了，肯定是选最左边的 1 号位。

再接着说有两个 1 的 `nametable`，还是从最简单的开始，最左边有两个 1 就不用说了，肯定是第二个 1，这样可以直接消掉，最右边两个 1，肯定还是右边;)如果左边一个 1，右边一个 1 呢？;)反证一下，如果你选了 1 号位，下次你还要选这个，肯定是不行的;)如果任意位两个 1 呢？其实是这样的，如果 1 的个数为偶数的话，左边的总要往右边靠，靠到了，就可以把他消掉。所以，一定是选 1 右边的位置。

感觉口都说干了，谁给我买点水吧，呵呵呵呵。其实这是一种感觉。3 个的也类似，反正就两个位置，要不 1，要不最左边 1 的右边，最左边 3 个，你说怎么最快消掉哦？最右边呢，肯定是把最左边的先去掉才能动后面的啊。类推吧。。。。。再说了，要是有 1 被异或掉了，那就变成了两个，这就是叫什么什么证明来着，高中的东西都忘光了。

代码如下，直接从验证的倒过来就是了，有些累了，不想多说了。输入用户名后，直接 `work()`，结束后，序列号存放在 `char serial[SERIAL_SIZE];` 中，可以看到，整体框架和计算序列号是否有效是差不多的。

代码应该比较容易看懂，函数名和变量名就是注释了。
