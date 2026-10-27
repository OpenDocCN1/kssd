
```
///////////////////////////////////////////////////////////////

//功能:   伽罗瓦域内的乘法运算  GF(128)

//参数:   Num_L           输入的左参数

//      Num_R           输入的右参数

//返回值:计算结果

char GaloisMultiplication(unsigned char Num_L, unsigned char Num_R)

{

    //定义变量

    unsigned char Result = 0;       //伽罗瓦域内乘法计算的结果



    while (Num_L)

    {

        //如果Num_L最低位是1就异或Num_R，相当于加上Num_R * 1

        if (Num_L & 0x01)

        {

            Result ^= Num_R;

        }



        //Num_L右移一位，相当于除以2

        Num_L = Num_L >> 1;



        //如果Num_R最高位为1

        if (Num_R & 0x80)

        {

            //左移一位相当于乘二

            Num_R = Num_R << 1;     //注：这里会丢失最高位，但是不用担心



            Num_R ^= 0x1B;  //计算伽罗瓦域内除法Num_R = Num_R / (x^8(刚好丢失最高位) + x^4 + x^3 + x^1 + 1)

        }

        else

        {

            //左移一位相当于乘二

            Num_R = Num_R << 1;

        }

    }

    return Result;

}
```

**在解密的逆向列混淆中与正向列混淆的不同之处在于使用的左乘矩阵不同，它与正向列混淆的左乘矩阵互为逆矩阵，也就是说，数据矩阵同时左乘这两个矩阵后，数据矩阵不会发生任何变化。**

**正向列混淆处理**

![正向列混淆](https://bbs.kanxue.com/upload/attach/202002/813468_UES487VAAZPKDFS.png)

**逆向列混淆**

![逆向列混淆](https://bbs.kanxue.com/upload/attach/202002/813468_ZCDXZKFMZA7K9SD.png)

**加解密验证**

![验证](https://bbs.kanxue.com/upload/attach/202002/813468_CY9JWPVZNVZD3ET.png)

**加密部分讲解完毕，最后应该注意要将密文结果从矩阵形式还原成字符串形式输出！**

---

# 第四节：AES密钥生成

![密钥生成流程](https://bbs.kanxue.com/upload/attach/202002/813468_CHWREHFQKJRYJR8.png)

**子密钥的生成是以列为单位进行的，一列是32Bit，四列组成子密钥共128Bit。生成子密钥的数量比AES算法的轮数多一个，因为第一个密钥加法层进行密钥漂白时也需要子密钥。密钥漂白是指在AES的输入盒输出中都使用的子密钥的XOR加法。子密钥在图中都存储在`W[0]`、`W[1]`、...、`W[43]`的扩展密钥数组之中。`k1`-`k16`表示原始密钥对应的字节，而图中子密钥`k0`与原始子密钥相同。在生成的扩展密钥中`W`的下标如果是4的倍数时(从零开始)需要对异或的参数进行`G`函数处理。扩展密钥生成有关公式如下：**

```
1<= i <= 10

1<= j <= 3

w[4i]     = W[4(i-1)] + G(W[4i-1]);

w[4i+j]   = W[4(i-1)+j] + W[4i-1+j];
```

![扩展密钥组](https://bbs.kanxue.com/upload/attach/202002/813468_ESXF84G6FPB4AP6.png)

**函数`G()`首先将4个输入字节进行翻转，并执行一个按字节的`S`盒代换，最后用第一个字节与轮系数`Rcon`进行异或运算。轮系数是一个有10个元素的一维数组，一个元素1个字节。`G()`函数存在的目的有两个，一是增加密钥编排中的非线性；二是消除AES中的对称性。这两种属性都是抵抗某些分组密码攻击必要的。**

![轮系数](https://bbs.kanxue.com/upload/attach/202002/813468_2TE266M7PETUFED.png)

**生成密钥代码:**

```
//用于密钥扩展    Rcon[0]作为填充，没有实际用途

const unsigned int Rcon[11] = { 0x00, 0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80, 0x1B, 0x36 };





int Key_S_Substitution(unsigned char(*ExtendKeyArray)[44], unsigned int nCol)

{

    int ret = 0;



    for (int i = 0; i < 4; i++)

    {

        ExtendKeyArray[i][nCol] = S_Table[(ExtendKeyArray[i][nCol]) >> 4][(ExtendKeyArray[i][nCol]) & 0x0F];

    }



    return ret;

}





int G_Function(unsigned char(*ExtendKeyArray)[44], unsigned int nCol)

{

    int ret = 0;



    //1、将扩展密钥矩阵的nCol-1列复制到nCol列上，并将nCol列第一行的元素移动到最后一行，其他行数上移一行

    for (int i = 0; i < 4; i++)

    {

        ExtendKeyArray[i][nCol] = ExtendKeyArray[(i + 1) % 4][nCol - 1];

    }



    //2、将nCol列进行S盒替换

    Key_S_Substitution(ExtendKeyArray, nCol);



    //3、将该列第一行元素与Rcon进行异或运算

    ExtendKeyArray[0][nCol] ^= Rcon[nCol / 4];



    return ret;

}





int CalculateExtendKeyArray(const unsigned char(*PasswordArray)[4], unsigned char(*ExtendKeyArray)[44])

{

    int ret = 0;



    //1、将密钥数组放入前四列扩展密钥组

    for (int i = 0; i < 16; i++)

    {

        ExtendKeyArray[i & 0x03][i >> 2] = PasswordArray[i & 0x03][i >> 2];

    }



    //2、计算扩展矩阵的后四十列

    for (int i = 1; i < 11; i++)    //进行十轮循环

    {

        //(1)如果列号是4的倍数，这执行G函数  否则将nCol-1列复制到nCol列上

        G_Function(ExtendKeyArray, 4*i);



        //(2)每一轮中，各列进行异或运算

        //      列号是4的倍数

        for (int k = 0; k < 4; k++)//行号

        {

            ExtendKeyArray[k][4 * i] = ExtendKeyArray[k][4 * i] ^ ExtendKeyArray[k][4 * (i - 1)];

        }



        //      其他三列

        for (int j = 1; j < 4; j++)//每一轮的列号

        {

            for (int k = 0; k < 4; k++)//行号

            {

                ExtendKeyArray[k][4 * i + j] = ExtendKeyArray[k][4 * i + j - 1] ^ ExtendKeyArray[k][4 * (i - 1) + j];

            }

        }

    }



    return ret;

}
```

---

# 第五节：AES解密流程图

![解密流程图](https://bbs.kanxue.com/upload/attach/202002/813468_JQP55QA4VDRPSYN.png)

**至此，AES算法基础部分介绍完毕！**

---

# 进阶部分概述：

> - **本节目的：**这一章作为AES算法的进阶部分，目的主要是对AES算法中的`S`盒的建立做一些介绍。
> - **阅读方法：**希望大家在浏览完本章文章后可以自己去实现一下，相信一定会对你的编程技术有所提高。`(附件中提供参考代码)`
> - **具备基础：**
>   (1)熟练掌握C语言
>   (2)相关数学知识
> - **学习环境：**任意C语言开发环境

---

# 第六节：相关的数学知识

**在接触密码学之前我认为数学的主要用途就是考试。。。但是接触密码学后我才发现数学的魅力，虽然数学只是一个工具，但这个工具却异常强大，我也因此吃了以前不认真学习相关知识的亏。好了，不多说了现在学习还不算太迟。**

## 欧几里得算法：

**两个正整数`r0`和`r1`的`gcd`表示为`gcd(r0, r1)`，它指的是可以被`r0`和`r1`同时整除的最大正整数，例如`gcd(21, 6)=3`。对与较小的整数而言`gcd`就是将两个整数进行因式分解，并找出最大的公因子。**

**例：`r0=84`,`r1=30`，因式分解：`r0=2·2·3·7`；`r1=2·3·5`；`gcd`的结果就是：`gcd(30,84)=2·3=6`。**

**`gcd(r0,r1)=gcd(r0-r1,r1)`其中假设`r0>r1`，并且两个数均是正整数。证明：`gcd(r0,r1)=g`，由于`g`可以同时被`r0`、`r1`整除，则可以记作`r0=g·x`、`r1=g·y`，其中`x>y`，并且`x`和`y`互为素数。所以得到：`gcd(r0,r1)=gcd(r0-r1,r1)=gcd(g·(x-y),g·y)=g=gcd(ri,0)`。**

**例：`r0=973`,`r1=301`,`gcd`的计算方式为：**

![例题1](https://bbs.kanxue.com/upload/attach/201908/813468_58VGT48ENBUDVK3.png)

**欧几里得算法代码**

```
//输入两个正整数r0>r1，输出计算结果

int gcd(int r0, int r1)

{

    int r=0;

    while(r1 != 0)

    {

        r = r0 % r1;

        r0 = r1;

        r1 = r;

    }

    return r0;

}
```

## 扩展欧几里得算法：

**扩展欧几里得算法主要的应用不是为例计算最大公因子，它的在密码学中主要的作用是为了计算乘法逆元，乘法逆元在公钥密码学中占有着举足轻重的地位。当然，除了扩展欧几里得算法`EEA`除了可以计算`gcd`，它还可以计算以下形式的线性组合：**

![公式7](https://bbs.kanxue.com/upload/attach/201908/813468_7HZKHAE3B768VZH.png)

**其中`s`和`t`都表示整型系数。关于如何计算这两个系数的推到过程这里就不介绍了，我们只给出最后的公式结论:**

![公式8](https://bbs.kanxue.com/upload/attach/201908/813468_Z6VD8PAP2EJBJ3H.png)

**介绍完这些公式，我们来看看乘法的逆元是怎么计算的吧。假设我们要计算`r1 mod r0`的逆元，其中`r1 < r0`。我们前面提到过乘法的逆元计算公式为`a*b=1 mod p`，`b`就是`a mod p`的乘法逆元，也就是`gcd(p, a)=1`的情况。下才存在乘法逆元。则`s·r0+t·r1=1=gcd(r0,r1)`，将这个等式执行模`r0`计算可得：**

![公式9](https://bbs.kanxue.com/upload/attach/201908/813468_RD73HP2FEVA8N5A.png)

**例题：计算`12 mod 67`，`12`的逆元，即`gcd(67,12)=1`**

![例题2](https://bbs.kanxue.com/upload/attach/201908/813468_4GWA95MP2KQEQR9.png)

**注：通常情况下不需要计算系数`S`,而且实际中一般也用不上它，另外结果`t`可能是一个负数，这种情况下就必须把`t`加是`r0`让人`的结果为正，因为`t=(t+r0) mod r0`。**

**扩展欧几里得算法代码：**

```
int EEA(int r0, int r1)

{

    int mod = r0;

    int r = 0;

    int t0 = 0;

    int t1 = 1;

    int t = t1;

    int q = 0;



    //0不存在乘法逆元

    if (r1 == 0)

    {

        return 0;

    }



    while (r1 != 1)

    {

        q = r0 / r1;



        r = r0 - q * r1;



        t = t0 - q * t1;



        r0 = r1;

        r1 = r;

        t0 = t1;

        t1 = t;

    }



    //结果为负数

    if (t < 0)

    {

        t = t + mod;

    }



    return t;

}
```

**现在相关知识已经学完了，开始进入重点。如果要想计算伽罗瓦域内乘法的逆元，函数的输入`r0`就是`GF(2^8)`的不可约多项式`p(x)`，`r1`就是域元素`a(x)`，然后通过`EEA`计算多项式`t(x)`得到`a(x)`的乘法逆元。只不过在上方给出的`EEA`代码略有不同，因为在伽罗瓦域中多项式都是在`GF(2)`上进行加减运算的，也就是说上面的加号和减号都要换成异或运算符，同时乘法和除法也有要进行适当的调整，转变成多项式乘法和除法。否则结果会出现偏差。**

**伽罗瓦域的扩展欧几里得算法：**
