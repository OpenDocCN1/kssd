
`[0, -16]` `[0, -15]` `[0, 15]` `[0, 16]` `[1, -27]` `[1, -4]` `[1, 4]` `[1, 27]` `[3, -26]` `[3, -5]` `[3, 5]` `[3, 26]` `[4, -21]` `[4, -10]` `[4, 10]` `[4, 21]` `[6, -24]` `[6, -7]` `[6, 7]` `[6, 24]` `[7, -20]` `[7, -11]` `[7, 11]` `[7, 20]` `[8, -24]` `[8, -7]` `[8, 7]` `[8, 24]` `[9, -26]` `[9, -5]` `[9, 5]` `[9, 26]` `[13, -23]` `[13, -8]` `[13, 8]` `[13, 23]` `[15, -27]` `[15, -4]` `[15, 4]` `[15, 27]` `[16, 0]` `[17, -24]` `[17, -7]` `[17, 7]` `[17, 24]` `[18, -18]` `[18, -13]` `[18, 13]` `[18, 18]` `[19, -26]` `[19, -5]` `[19, 5]` `[19, 26]` `[24, -22]` `[24, -9]` `[24, 9]` `[24, 22]` `[27, -28]` `[27, -3]` `[27, 3]` `[27, 28]` `[30, 0]`

`MOD41`

```
(16:32) gp > for(x=0,41, for(y=0,41, if((y²-(x³+7*x+8))%41==0, print([x,y]))));
```

`[0, 7]` `[0, 34]` `[1, 4]` `[1, 37]` `[4, 10]` `[4, 31]` `[5, 2]` `[5, 39]` `[6, 15]` `[6, 26]` `[7, 20]` `[7, 21]` `[8, 17]` `[8, 24]` `[9, 12]` `[9, 29]` `[12, 4]` `[12, 37]` `[13, 0]` `[13, 41]` `[14, 12]` `[14, 29]` `[18, 12]` `[18, 29]` `[23, 6]` `[23, 35]` `[25, 8]` `[25, 33]` `[27, 6]` `[27, 35]` `[28, 4]` `[28, 37]` `[29, 0]` `[29, 41]` `[31, 2]` `[31, 39]` `[32, 6]` `[32, 35]` `[35, 18]` `[35, 23]` `[37, 11]` `[37, 30]` `[38, 1]` `[38, 40]` `[40, 0]` `[40, 41]` `[41, 7]` `[41, 34]`

`elltaniyama(E)`

`elltaniyama(e)`: modular parametrization of elliptic curve `e`.

```
(15:50) gp > elltaniyama(E)
```

`%28 = [x^-2 - x² + 2*x⁶ + 4*x⁸ + 5*x¹⁰ + O(x¹⁵), -x^-3 - 2*x - 4*x³ + x⁵ + 2*x⁷ - 2*x⁹ - 10*x¹¹ - 15*x¹³ + O(x¹⁴)]`

谷山丰的模型式      ------ 谷山丰志村五郎都知道

`ellan`、`ellap`    求 `MODP` 的偏移，试了 `5` 和 `17`，可验证 `HEESE` 定理，`mod5` 是条超奇异，不符 `HEESE`

`E = ellinit([0,0,0,7,8]);`

`ellap(E,5)`

`%86 = -1`

`E1 = ellinit([0,0,0,7,8]*Mod(1,5))`

`for(x=0,4, for(y=0,4, if(ellisoncurve(E1,[x,y]),print([x,y]))))`

`[1, 4]`

`ellap(E,17)`

```
(16:31) gp >  for(x=0,16, for(y=0,16, if(ellisoncurve(E2,[x,y]),print([x,y]))))
```

`[0, 5]`
`[0, 12]`
`[1, 4]`
`[1, 13]`
`[2, 8]`
`[2, 9]`
`[4, 7]`
`[4, 10]`
`[5, 7]`
`[5, 10]`
`[7, 3]`
`[7, 14]`
`[8, 7]`
`[8, 10]`
`[9, 1]`
`[9, 16]`
`[12, 1]`
`[12, 16]`
`[13, 1]`
`[13, 16]`
`[16, 0]`

`gp > 5+1-2`

`4`

`17+1-22`

`-4`

`) gp > ellan(E,18)`

`[1, 0, 0, 0, -1, 0, -4, 0, -3, 0, -2, 0, -6, 0, 0, 0, -4, 0]`

`ellpow` 可以找出循环群的生成元点，试了几个，`[6, 9]` 不是，`[7, 17]` 是

`[1, 4]` `[1, 33]` `[2, 17]` `[2, 20]` `[4, 10]` `[4, 27]` `[6, 9]` `[6, 28]` `[7, 17]` `[7, 20]` `[8, 13]` `[8, 24]` `[11, 11]` `[11, 26]` `[12, 9]` `[12, 28]` `[14, 1]` `[14, 36]` `[15, 11]` `[15, 26]` `[18, 3]` `[18, 34]` `[19, 9]` `[19, 28]` `[25, 3]` `[25, 34]` `[27, 14]` `[27, 23]` `[28, 17]` `[28, 20]` `[31, 3]` `[31, 34]` `[32, 12]` `[32, 25]` `[33, 8]` `[33, 29]` `[34, ...]`

```
(17:40) gp > for(n=1,37,print(n,"*([6,9] =  ",lift(ellpow(e,x,n))))
(17:41) gp > x=[7, 17];
(17:41) gp > for(n=1,37,print(n,"*[7, 17]=  ",lift(ellpow(e,x,n))))
```

`1*[7, 17] = [7, 17]`
`2*[7, 17] = [27, 23]`
`3*[7, 17] = [19, 9]`
`4*[7, 17] = [32, 12]`
`5*[7, 17] = [1, 4]`
`6*[7, 17] = [8, 24]`
`7*[7, 17] = [34, 16]`
`8*[7, 17] = [6, 9]`
`9*[7, 17] = [14, 1]`
`10*[7, 17] = [25, 3]`
`11*[7, 17] = [12, 28]`
`12*[7, 17] = [11, 26]`
`13*[7, 17] = [31, 3]`
`14*[7, 17] = [33, 29]`
`15*[7, 17] = [4, 10]`
`16*[7, 17] = [15, 26]`
`17*[7, 17] = [18, 3]`
`18*[7, 17] = [2, 17]`
`19*[7, 17] = [28, 20]`
`20*[7, 17] = [36, 0]`
`21*[7, 17] = [28, 17]`
`22*[7, 17] = [2, 20]`
`23*[7, 17] = [18, 34]`
`24*[7, 17] = [15, 11]`
`25*[7, 17] = [4, 27]`
`26*[7, 17] = [33, 8]`
`27*[7, 17] = [31, 34]`
`28*[7, 17] = [11, 11]`
`29*[7, 17] = [12, 9]`
`30*[7, 17] = [25, 34]`
`31*[7, 17] = [14, 36]`
`32*[7, 17] = [6, 28]`
`33*[7, 17] = [34, 21]`
`34*[7, 17] = [8, 13]`
`35*[7, 17] = [1, 33]`
`36*[7, 17] = [32, 25]`
`37*[7, 17] = [19, 28]`

`ellorder` `PARI/GP` 不能用，但 `MAXMA` 里很好用的

`PARI/GP` 绘图：能感到黑白年代的图像分辨率。。。。。

`PARI/GP` 里对环和理想，类群都很强，不光是多项式环 阿廷环 诺特环 。。。。 可试试?6，水平有限，只提提有关 `ECC-2m` 多项式，没 `MAXMA` 对 `ECC-2m` 多项式直接，`MAXMA` 有 `ECC-2m` 初始化函数的

`PARI/GP` 可下个包，是 `EC` 关于神秘的 `RANK` 问题的，`RANK` 能懂代数数论代数几何就入门了.....我还在找路，没发现门.......

Elliptic Curve Data

`http://www.warwick.ac.uk/staff/J.E.Cremona//ftp/data/`

`MAXMA`

代数方面强捍，可看看下面，集合论同伦李群同调都有函数。。。。，也有拓朴方面的，不开源不过上面我贴了 `magma` 的 `DOWNLOAD` 地址，有心人可用 `IDA` 看看函数参数。。。。`PARI/GP` 是 `ANSI C`，`magma` 应该是 `UNICODE`

维护人多，超过 `PARI/GP`，收钱，不过有个在线 `magma`

`http://magma.maths.usyd.edu.au/magma/handbook/`

关于 `ECC` 方面下面文档，很详细

`RATIONAL CURVES AND CONICS`
`ELLIPTIC CURVES`
`ELLIPTIC CURVES OVER FINITE FIELDS`
`ELLIPTIC CURVES OVER FUNCTION FIELDS`
`MODELS OF GENUS ONE CURVES`
`HYPERELLIPTIC CURVES`
`L-FUNCTIONS`

`RATIONAL CURVES AND CONICS` --------有理圆锥曲线，2 次的椭双抛，之间的同态同构自同构，中学生就懂

`ELLIPTIC CURVES OVER FINITE FIELDS`，`HYPERELLIPTIC CURVES` 有限域椭圆曲线，`ECC` 就建在他上

`FUNCTION FIELDS` 函数域椭圆曲线，可能泛函学过之后才行

`ODELS OF GENUS ONE CURVES` 亏格一曲线的模式

`L-FUNCTIONS` L 函数，很多方面都爱往 L 函数上凑。。。。。。

初始化用 `EllipticCurve`，超椭圆，非超奇异，超奇异都用他

`EllipticCurve(C) : Sch -> CrvEll, MapSch`
`SetVerbose("EllModel", n): Maximum: 3Given a scheme C describing a curve of genus 1 with an easily recognised rational point, the function returns an elliptic curve E, together with a birational map from C to E. If there is no "obvious" rational point then this routine will fail. C must belong to one of the following classes:`

(i)`Hyperelliptic curves of genus 1 of the form C: y2 + h(x)y=f(x) with f of degree 3 or 4 and h of degree at most 1. If the function x on C has a rational branch point then that point is sent to the origin on E. Otherwise, if C has a rational point at x=∞ then that point is used.`

(ii)`Nonsingular plane curves of degree 3. If the curve is already in general Weierstrass form up to a permutation of the variables, then this is recognised and used as a model for the elliptic curve. Otherwise the base field of the curve must have characteristic different from 2 and 3; in this case, the curve is tested for having a rational flex. If it has, then a linear transformation suffices to get the curve into general Weierstrass form, and this is used.`

(iii)`Singular plane curves of degree 4 over a base field of characteristic different from 2 with a unique cusp, with the tangent cone meeting the curve only at that point. Up to linear transformation, these are curves of type y2=f(x), with f of degree 4. Such curves are brought into the standard form above. If either a rational point exists with x=0 or the curve intersects the line at infinity in a rational point, then that point is used to put the curve in general Weierstrass form`

`EllipticCurve([a1, a2, a3, a4, a6]) :`

三种方式都可以

```
E:=EllipticCurve([7, 8]);
E;
jInvariant(E);
```

先定义环

```
Qx<x> := PolynomialRing(Rationals());
EllipticCurve(x³ + 7*x+8);
```

先定 J 不变量

```
E1:=EllipticCurveWithjInvariant(592704/(775));
E1;
```

Elliptic Curve defined by `y² = x³ + 7*x + 8` over Rational Field
`592704/775`

Elliptic Curve defined by `y² = x³ + 7*x + 8` over Rational Field

复域：

```
Qx<x> := PolynomialRing(ComplexField());
EllipticCurve(x³ + 7*x+8);
```

Elliptic Curve defined by `y² = x³ + 7.00000000000000000000000000000*x + 8.00000000000000000000000000000` over Complex field of precision 30

不变量。。。。？？

参数对应，还能省略，自动认超奇异

```
E:=EllipticCurve([0,0,0,7, 8]);
E;
E1:=EllipticCurve([7, 8]);
E1;
E2 := EllipticCurve([GF(17) | 7, 8]);
E2;
E3 := EllipticCurve([GF(19) | 7, 8]);
E3;
E4 := EllipticCurve([GF(31) | 7, 8]);
E4;
```

Elliptic Curve defined by `y² = x³ + 7*x + 8` over Rational Field
Elliptic Curve defined by `y² = x³ + 7*x + 8` over Rational Field
Elliptic Curve defined by `y² = x³ + 7*x + 8` over `GF(17)`
