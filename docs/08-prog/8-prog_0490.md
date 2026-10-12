`

# 递归计算斐波那契数列时间复杂度之彻底分析

> 原文：[`www.pediy.com/kssd/pediy11/123051.html`](https://www.pediy.com/kssd/pediy11/123051.html)

以前看数据结构与算法分析(C 语言描述),作者在讲述算法分析时提到递归计算斐波那契数列，

```c
Fib(int N)
{
  if(N <= 1)
    return 1;
  else
    return Fib(N - 1) + Fib(N-2);
}
```

时间复杂度 `T(N) = T(N-1) + T(N-2);`

作者说归纳法易证得 `T(N) >= Fib(N);`而 `Fib(N) < (5/3)^(N);`类似计算可正得 `Fib(N)>= (3/2)^(N)`

这个地方作者处理得有些不太彻底，不是吗?翻来覆去没把 `Fib(N)`准确求出，今天我翻看另一本资料，

更模糊了，它求得 `2^(N/2) < T(N) < 2^(N)`就完事了。

不过今天我开窍了，`T(N) = T(N-1) + T(N-2);`这是什么?也许你会说这是一个递推式，但它的学名是

二阶齐次差分方程，我今天猛然看出这一点:)!![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

## 什么是差分方程?

以下资料全部来自 `Introduction to Numerical Analysis, second Edition Translate by Bartels,W.Gautsch and C.Witigall,Springer-Verlag`，世界图书出版公司，(英文版)

这是我大三所用的数值分析教材，目前国内好像还没中文版。

书上 464 页说道：

```
By a linear homogeneous difference equation of order r one means an equation of the form

u(j+r) + a(r-1)u(j+4-1) + .... + a(0)u(j) =0;  j=0,1,2...  (7.2.9.1)
```

上面那些括号里的数一般是下标，我不好打出来，用括号代替了。

反正就是说齐次差分方程就是那个式子的形式了。

接着 466 页上讲到

```
Theorem. let the polynomial

Ψ(u) = u^r + a(r-1)u^(r-1) + ... + a(0)

have the k distinct zeros λ(i) , i = 1,2,...,k

with multiplicities σ(i),i = 1,2,...,k

and let a(0) != 0,then for arbitray polynomials p(i)(t) with deg p(i) < σ(i),i = 1...k;

the sequence

u(j) = p(1)(j)λ(1)^(j) + p(2)(j)λ(2)^(j) +...+ p(k)(j)λ(k)^(j)

is a solution of the difference eqution (7.2.9.1).Conversely,every solution of

(7.2.9.1)can be uniquely represented in the form(7.2.9.9).
```

这段话又是什么意思呢？一句话：这是个牛逼的定理它导出了齐次差分方程的解并且说明了齐次差分方程

的所有解的形式。书上证明不是太短。

接下来举例说明。就拿斐波那契数列来说。

书上 561 页提了个问题，问题我就不打了，它问：

`u(j+2) = u(j+1) + u(j)`的通项公式是什么？然后它说 `u(0) = 0,u(1) = 1` 即可得 `Fibonacci sequence`

就是说这个式子从 `u(0) = 0,u(1) = 1` 递推即可得斐波那契数列。从其他初始状态的话可得其他数列。

开始解决问题。

按照那个牛逼定理，先解方程 `x² - x - 1 = 0;`

这个不难，`x1= (1+5^(1/2))/2,x2 = (1-5^(1/2))/2;`

然后套公式 `u(j) = c1 * x1^(j) + c2 * x2^(j);`

`c1,c2` 为待定常数，利用 `u(0) = 0,u(1) = 1`，得出 `c1 = 1/(5^(1/2));c2 = -c1;`

这样斐波那契数列的通项就出来了，递归计算斐波那契数列时间复杂度问题也就迎刃而解了。

`