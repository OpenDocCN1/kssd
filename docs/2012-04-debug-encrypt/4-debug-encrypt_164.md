
# Magma, PARI/GP 中文文档 ECC 初步

> 原文：`www.pediy.com/kssd/pediy12/129091.html`

先亮亮我的 EC 工具箱，最好都下载了（有 MAlab 和 MATHMATICA，QEMU/WINDOWS 2003 ISO）-------怕快到期了![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

`http://uploading.com/files/manager/0/`

======================

`GOOGLE.COM` `magma`/`PARI/GP` 好像还没中文文档，`SAGE` 有，但 `elliptic curve` 方面没译。

都是统代数系，但比 `MAPLE` 和 `MATHMATICA` 强在专，对学 `ECC` 来说，`elliptic curve in` 在无限域的 `Q/R/C/p` 进数，有限域的 `ZP` 都有函数实现，`PARI/GP` 有些代码是汇编语言写的，其它大都能看 `C` 代码，`GUN` 嘛

`PARI/GP` 作者 `Henri Cohen`，就不多讲了，和 `FREY` 共著 `Handbook of Elliptic and Hyperelliptic Curve Cryptography`，我国 `ECC` 书抄这书的一点点，就能买钱，`FREY` 曲线和怀尔斯，都听过地

`MAGMA` 岩浆？还有个同名的，不要混了，在线 `http://magma.maths.usyd.edu.au/calc/` 没绘图功能 `PLOT` 之类的，可以肯定比单独下个收费的功能少，不过对 `ECC` 初学者从 `C` 库 `MSDN` 那是方便多了，搜了 N 天，有人放这了，快下载收好了，`http://sourceforge.net/projects/maxima/files/Maxima-Windows/5.23.2-Windows/maxima-5.23.2.exe/download`

`SPRINGER` 出了岩浆书 `Discovering Mathematics with Magma`，`GOOGLE.COM` 下就能下

`SAGEMATH.ORG` 在线，能绘图，是个数学工具大集合，常见统代数系的大都综合到这了，注册后页面能看到，十几种吧，可以只选用其中一种，国人很多，因为常见汉字，文档众多，数学家+程序员，天下无敌，基于 `Python` 语言，珍珠红宝石巨蟒，这里是巨蟒密林

下面是关于几个数论和 `ECC` 和绘图的函数用法，水瓶晃荡。。。。

`PARI/GP` 下好安装后，有个 `GP.EXE`，双击

```
(13:23) gp > ?

Help topics: for a list of relevant subtopics, type ?n for n in

  0: user-defined identifiers (variable, alias, function)
  1: Standard monadic or dyadic OPERATORS
  2: CONVERSIONS and similar elementary functions
  3: TRANSCENDENTAL functions
  4: NUMBER THEORETICAL functions
  5: Functions related to ELLIPTIC CURVES  6: Functions related to general NUMBER FIELDS
  7: POLYNOMIALS and power series
  8: Vectors, matrices, LINEAR ALGEBRA and sets
  9: SUMS, products, integrals and similar functions
 10: GRAPHIC functions
 11: PROGRAMMING under GP
 12: The PARI community
```

第四项是数论的

```
(14:12) gp > ?4

addprimes     bestappr      bezout        bezoutres     bigomega

binomial      chinese       content       contfrac      contfracpnqn

core          discdir       dirdiv        direuler      dirmul

divisors      eulerphi      factor        factorback    factorcantor

factorff      factorial     factorint     factormod     ffinite

fibonacci     gcd           hilbert       isfundamental ispower

isprime       ispseudoprime issquare      issquarefree  kronecker

lcm           moebius       nextprime     numbpart      numdiv

omega         precprime     prime         primepi       primes

qfbclassno    qfbcompraw    qfbhclassno   qfbnucomp     qfbnupow

qfbpowraw     qfbprimeform  qfbred        qfbsolve      quadclassunit

quaddisc      quadgen       quadhilbert   quadpoly      quadray

quadregulator quadunit      removeprimes  sigma         sqrtint

zncoppersmith znlog         znorder       znprimroot    znstar
```

看名字就差不离多少，不行就 `?`

```
(14:15) gp > ?bezout

bezout(x,y): gives a 3-dimensional row vector [u,v,d] such that d=gcd(x,y) and
u*x+v*y=d.
```

```
(14:25) gp > bezout(12345,54321)

%14 = [3617, -822, 3]
```

`x= 12345 = 3 × 5 × 823`

`y= 54321 = 3 × 19 × 953`

最大公因式 `(x,y) = 3 = 3`

最小公倍数 `(x,y) = 223530915 = 3 × 5 × 19 × 823 × 953`

`Bezout` 关系: `3617x + (-822) y = 3`

以 `y` 为除数的 `x` 的辗转相除的过程序列:

被除数 商 除数 余数

`12345 = 0 × 54321 + 12345`

`54321 = 4 × 12345 + 4941`

`12345 = 2 × 4941 + 2463`

`4941 = 2 × 2463 + 15`

`2463 = 164 × 15 + 3`

`15 = 5 × 3 + 0`

多项式的：连商带余都给了

```
(14:33) gp > ?bezoutres

bezoutres(x,y): gives a 3-dimensional row vector [u,v,d] such that
d=resultant(x,y) and u*x+v*y=d, where x and y are polynomials.
```

```
(14:35) gp > x=p⁴+p³+3

%1 = p⁴ + p³ + 3
```

```
(14:36) gp > y=p+1

%2 = p + 1
```

```
(14:36) gp > bezoutres(x,y)

%3 = [1, -p³, 3]
```

孙子：

```
(16:10) gp > chinese(Mod(18,115),Mod(21,71))

%13 = Mod(7263, 8165)
```

求 `X^NMOD(P)`

```
(16:13) gp > Mod(217,389)⁵⁰²⁰⁰

%15 = Mod(36, 389)
```

第五项 `ELLIPTIC curve` 的，无限的 `Q/R/C/p` 进数域，有限域的 `ZP` 都可以

```
(14:08) gp > ?5

elladd          ellak             ellan             ellap

ellbil          ellchangecurve    ellchangepoint    ellconvertname

elleisnum       elleta            ellgenerators     ellglobalred

ellheight       ellheightmatrix   ellidentify       ellinit

ellisoncurve    ellj              elllocalred       lllseries

ellminimalmodel ellorder          ellordinate       ellpointtoz

ellpow          ellrootno         ellsearch         ellsigma

ellsub          elltaniyama       elltors           ellwp

ellzeta         ellztopoint
```

`EC` 初始化用 `ellinit`：

```
(14:41) gp > ? ellinit

ellinit(x,{flag=0}): x being the vector [a1,a2,a3,a4,a6] defining the curve
Y² + a1.XY + a3.Y = X³ + a2.X² + a4.X + a6, gives the vector:
[a1,a2,a3,a4,a6,b2,b4,b6,b8,c4,c6,disc,j,[e1,e2,e3],,eta1,etw1,w2a2,area]. If
the curve is defined over a p-adic field, the last six components are replaced
by root,u²,u,q,w,0. If optional flag is 1, omit them altogether. x can also
be a string, in this case the coefficients of the curve with matching name are
looked in the elldata database if available.
```

21 个参数，前五个

`a1,a2,a3,a4,a6`，这六项就是正规表式的 `EC` 曲线 `Y² + a1.XY + a3.Y = X³ + a2.X² + a4.X + a6` 系数,

`b2,b4,b6,b8,c4,c6,disc,j` 这七项和 `EC` 曲线变换有关，和曲线划分有关，奇异非奇异等等，在 `R` 域上图像就是有没尖点---光滑

`[e1,e2,e3],,eta1,etw1,w2a2,area` 这七项应该和 `C` 域有关，`C` 域上 `EC` 曲线是个轮胎，正在学这个轮胎

`flag=1`，`X` 还可以是字符串

找条曲线试试，`E.tate`，`E.w` 这两看来是在 `P` 进数域才能用的

`Y² + a1.XY + a3.Y = X³ + a2.X² + a4.X + a6` 都可变成 `Y²=x³+ax+b` 形式

`Y²=X³+7X+8`

```
(15:20) gp > E=ellinit([0,0,0,7,8]);
```

```
(15:20) gp > E.a1

%9 = 0
```

```
(15:20) gp > E.a2

%10 = 0
```

```
(15:20) gp > E.a3

%11 = 0
```

```
(15:20) gp > E.a4

%12 = 7
```

```
(15:20) gp > E.a6

%13 = 8
```

```
(15:20) gp > E.b2

%14 = 0
```

```
(15:21) gp > E.b4

%15 = 14
```

```
(15:21) gp > E.b6

%16 = 32
```

```
(15:21) gp > E.b6

%17 = 32
```

```
(15:21) gp > E.c4

%18 = -336
```

```
(15:21) gp > E.c6

%19 = -6912
```

```
(15:21) gp > E.disc

%20 = -49600
```

```
(15:26) gp > E.roots

%21 = [-1.000000000000000000000000000, 0.5000000000000000000000000000 - 2.783

181415010961059735649*I, 0.5000000000000000000000000000 + 2.78388218141501096

9735649*I]~
```

```
(15:31) gp > E.omega[1]

%22 = 2.400226835016717000625368437
```

```
(15:32) gp > E.area

%23 = 2.284807014575569154497352277
```

```
(15:33) gp > E.tate

  ***   curve not defined over a p-adic field.
```

```
(15:33) gp > E.w

  ***   curve not defined over a p-adic field.
```

```
(15:34) gp > E.eta1
```

这两个是以前版本的命令：

```
(15:37) gp > initell([0,0,0,7,8]);

  ***   obsolete function: initell([0,0,0,7,8])
                    ^--------------------

For full compatibility with GP 1.39.15, type "default(compatible,3)", or set
"compatible = 3" in your GPRC file.

New syntax: initell(x) ==/> ellinit(x)
```

```
(15:37) gp > smallinitell([0,0,0,7,8]);

  ***   obsolete function: smallinitell([0,0,0,
                    ^--------------------

For full compatibility with GP 1.39.15, type "default(compatible,3)", or set
"compatible = 3" in your GPRC file.

New syntax: smallinitell(x) ==/> ellinit(x,1)
```

求个切线自加点，

`z1=[4,10];`

`z2=[4,10];`

`elladd(E,z1,z2)`

`12 = [-7/16, 141/64]`

```
(16:42) gp > ?ellisoncurve

ellisoncurve(e,x): true(1) if x is on elliptic curve e, false(0) if not.
```

```
16:42) gp > ellisoncurve(E,[-7/16, 141/64])

13 = 1
```

求挠点，`y²=x³+7*x+8`，表明从`[1, 0]`可生成所有有理点，两个一组，这曲线不好。。。

```
(16:23) gp >  elltors(E)%2 = [2, [2], [[-1, 0]]]
```

上面是在 `R` 域，现在 `P` 域求 `MOD31`：`y` 还都可以为负的

```
(16:20) gp > for(x=0,30, for(y=0,30, if((y²-(x³+7*x+8))%31==0, print1([x,y]
[0, 15]  [0, 16]  [1, 4]  [1, 27]  [3, 5]  [3, 26]  [4, 10]  [4, 21]  [6, 7]  [6
, 24]  [7, 11]  [7, 20]  [8, 7]  [8, 24]  [9, 5]  [9, 26]  [13, 8]  [13, 23]  [1
5, 4]  [15, 27]  [16, 0]  [17, 7]  [17, 24]  [18, 13]  [18, 18]  [19, 5]  [19, 2
6]  [24, 9]  [24, 22]  [27, 3]  [27, 28]  [30, 0]
```

```
(17:00) gp > for(x=0,30, for(y=-30,30, if((y²-(x³+7*x+8))%31==0, print1([x,y]
```



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



### 椭圆曲线定义与错误处理

Elliptic Curve defined by `y² = x³ + 7*x + 8` over `GF(19)`

```
>> E4 := EllipticCurve([GF(31) | 7, 8]);
                   ^
Runtime error in 'EllipticCurve': Curve is singular

>> E4;
   ^
User error: Identifier 'E4' has not been declared or assigned
```

### 化系数为整

```
E5 := EllipticCurve([1/12, 1/22, 1, 1/13, 41]);

IE := IntegralModel(E5);

IE;
```

`Elliptic Curve defined by y² + 1/12*x*y + y = x³ + 1/22*x² + 1/13*x + 41 over Rational Field`

`Elliptic Curve defined by y² + 143*x*y + 5053029696*y = x³ + 133848*x² + 666999919872*x + 1046857473454971949056 over Rational Field`

### 最小化模型

```
> ME := MinimalModel(IE);
> ME;
```

`Elliptic Curve defined by y² + x*y = x³ + 1021854892776*x + 1053193319031442674240 over Rational Field`

### 常规式（Weierstrass 模型）

就最常见的了

```
WeierstrassModel(E2);
```

`Elliptic Curve defined by y² = x³ + 7*x + 8 over GF(17)`

### 同构性检查

```
IsIsomorphic(E, E1);
true
```

### 生成 2 次扭曲线

```
QuadraticTwist(E3, 10);

QuadraticTwist(E2, 5);

S1 := QuadraticTwists(E3);

S2 := QuadraticTwists(E2);

S1;

S2;

[ IsIsomorphic(E3,E) : E in S1 ];

[ IsIsomorphic(E2,E) : E in S2 ];
```

`Elliptic Curve defined by y² = x³ + 16*x + 1 over GF(19)`

`Elliptic Curve defined by y² = x³ + 5*x + 14 over GF(17)`

```
[
    Elliptic Curve defined by y² = x³ + 7*x + 8 over GF(19),
    Elliptic Curve defined by y² = x³ + 7*x + 11 over GF(19)
]

[
    Elliptic Curve defined by y² = x³ + 7*x + 8 over GF(17),
    Elliptic Curve defined by y² = x³ + 12*x + 5 over GF(17)
]

[ true, false ]
[ true, false ]
```

### j 不变量与扭曲线

```
jInvariant(E3);

jInvariant(E2);

T3 := Twists(E3);

> T3;

T2 := Twists(E2);

> T2;
```

### 判别式和各种不变量

公式，初中就明白部分

```
b_2 = a_1² + 4*a_2
b_4 = a_1*a_3 + 2*a_4
b_6 = a_3² + 4*a_6
b_8 = a_1²*a_6 + 4*a_2*a_6 - a_1*a_3*a_4 + a_2*a_3² - a_4².

c_4 = b_2² - 24*b_4
c_6 = -b_2³ + 36*b_2*b_4 - 216*b_6.
d = -b_2²*b_8 - 8*b_4³ - 27*b_6² + 9*b_2*b_4*b_6
```

### 求整点和挠点系（辅助理解 Mordell-Weil 定理）

```
Qx<x> := PolynomialRing(Rationals());

EllipticCurve(x³ + 7*x + 8);

Q, reps := IntegralPoints(E);

Q;

reps;
```

```
[ (-1 : 0 : 1), (1 : 4 : 1), (4 : -10 : 1), (7 : 20 : 1), (8 : -24 : 1), (89 : -840 : 1), (1921 : 84196 : 1) ]

[ <(-1 : 0 : 1), 1>, <(1 : 4 : 1), 1>, <(4 : -10 : 1), 1>, <(7 : 20 : 1), 1>, <(8 : -24 : 1), 1>, <(89 : -840 : 1), 1>, <(1921 : 84196 : 1), 1> ]
```

### 非超奇异与超奇异判断

```
E3 := EllipticCurve([GF(17) | 7, 8]);

E3;

IsSupersingular(E3)
IsProbablySupersingular(E3)  
ordinary(E3)
```

`Elliptic Curve defined by y² = x³ + 7*x + 8 over GF(17)`

```
false
false
true
```

好像特征 `P` 的超奇异多项式表达式有关?

```
SupersingularPolynomial(p)

SupersingularPolynomial(31)

$.1³ + 2*$.1² + 22*$.1 + 2
```

### 生成 CHAR2/3 的 ECC 及时间

```
K := FiniteField(2,192); // finite field of size 2¹⁶⁰

> E := EllipticCurve([K!1,0,0,0,K.1]);

E;

> time #E;
```

`Elliptic Curve defined by y² + x*y = x³ + K.1 over GF(2¹⁹²)`

```
6277101735386680763835789423275520405152076905076979576832
Time: 0.100
```

```
K := FiniteField(3,260); // finite field of size 2¹⁶⁰

E3 := EllipticCurve([K!1,0,0,0,K.1]);

E3;

time #E3;
```

`Elliptic Curve defined by y² + x*y = x³ + K.1 over GF(3²⁶⁰)`

```
11259684642548723353960122729962565552251726915490860177319990746860187759589070448135666259186955978522508673000384073458387
Time: 1.800
```

### 找点阶 CODE，想找何种曲线方便多多

```
p := NextPrime(2¹⁵);

p;

K := GF(p);

for k in [1..p] do
    E := EllipticCurve([K | k, 31]);
    n := SEA(E : MaxSmooth := 1);
    if IsPrime(n) and n eq NextPrime(2¹⁵) then
        printf "Found curve of prime order %o for k = %o\n", n, k;
        break;
    end if;
end for;

E;
```

### 验证扭曲线阶之和

2 进制域还不行？。。。

```
P:=NextPrime(10000000000000000000000000000);

P;

E3 := EllipticCurve([GF(P) | 7, 8]);

E3;

E4:=Twists(E3);

E4;

E5 := EllipticCurve([GF(P) | 4651483222866113843238986811, 4220774469207810384173128581]);

IsSupersingular(E3);

IsSupersingular(E5);

O3:=Order(E3);

O5:=Order(E5);

O3+O5;

2*P+2;
```

```
Qx<x> := PolynomialRing(Rationals());

K := FiniteField(13,12);

E2 := EllipticCurve([K | 7, 8]);

E2;

O2:=Order(E2);

Twists(E2);

E1 := EllipticCurve(x³ + (3*K.1¹¹ + K.1¹⁰ + 11*K.1⁹ +
            3*K.1⁸ + 3*K.1⁶ + 7*K.1⁵ + 8*K.1⁴ + 6*K.1³ + 3*K.1² + 5*K.1 + 7)*x
    + (12*K.1¹¹ + 8*K.1¹⁰ + 5*K.1⁹ + 9*K.1⁸ + 7*K.1⁶ + 11*K.1⁴ + 7*K.1³ +
            K.1² + 12*K.1 + 4))
```

### 求挠子群点

`E1 := EllipticCurve([K1|0,-4,0,0,16])` 是 `SILVERMAN` 书上的，对照看看

```
K := FiniteField(47); 

E := EllipticCurve([K|0,0,0,7,8]);

E;

G, h := TorsionSubgroup(E);

torsion_pts_E := [ h(g) : g in G ];

torsion_pts_E;
```

`Elliptic Curve defined by y² = x³ + 7*x + 8 over GF(47)`

```
[ (0 : 1 : 0), (22 : 0 : 1), (5 : 36 : 1), (44 : 30 : 1), (7 : 27 : 1), (25 : 43 : 1), (20 : 8 : 1), (21 : 43 : 1), (0 : 33 : 1), (39 : 2 : 1), (1 : 4 : 1), (4 : 10 : 1), (11 : 10 : 1), (9 : 1 : 1), (8 : 24 : 1), (42 : 41 : 1), (3 : 3 : 1), (12 : 38 : 1), (41 : 28 : 1), (32 : 10 : 1), (26 : 0 : 1), (46 : 0 : 1), (41 : 19 : 1), (32 : 37 : 1), (3 : 44 : 1), (12 : 9 : 1), (8 : 23 : 1), (42 : 6 : 1), (11 : 37 : 1), (9 : 46 : 1), (1 : 43 : 1), (4 : 37 : 1), (0 : 14 : 1), (39 : 45 : 1), (20 : 39 : 1), (21 : 4 : 1), (7 : 20 : 1), (25 : 4 : 1), (5 : 11 : 1), (44 : 17 : 1) ]
```

```
K1 := FiniteField(17); 

E1 := EllipticCurve([K1|0,-4,0,0,16]);

E1;

G, h := TorsionSubgroup(E1);

torsion_pts_E1 := [ h(g) : g in G ];

torsion_pts_E1;
```

`Elliptic Curve defined by y² = x³ + 13*x² + 16 over GF(17)`

```
[ (0 : 1 : 0), (2 : 12 : 1), (9 : 9 : 1), (14 : 15 : 1), (4 : 13 : 1), (11 : 9 : 1), (10 : 2 : 1), (1 : 8 : 1), (0 : 13 : 1), (15 : 3 : 1), (8 : 0 : 1), (15 : 14 : 1), (0 : 4 : 1), (1 : 9 : 1), (10 : 15 : 1), (11 : 8 : 1), (4 : 4 : 1), (14 : 2 : 1), (9 : 8 : 1), (2 : 5 : 1) ]
```

```
MW, f := MordellWeilGroup(E);

MW, f := MordellWeilGroup(E1);
```

`Elliptic Curve defined by y² = x³ - 4*x² + 16 over Rational Field`

```
[ (0 : 1 : 0), (0 : 4 : 1), (4 : -4 : 1), (4 : 4 : 1), (0 : -4 : 1) ]
```

`Elliptic Curve defined by y² = x³ - 4*x² + 16 over Rational Field`

```
[ (0 : 1 : 0), (0 : 4 : 1), (4 : -4 : 1), (4 : 4 : 1), (0 : -4 : 1) ]
```

```
Abelian Group isomorphic to Z/5

Defined on 1 generator

Relations:

    5*MW.1 = 0
```

`Elliptic Curve defined by y² = x³ + 7*x + 8 over Rational Field`

```
[ (0 : 1 : 0), (-1 : 0 : 1) ]
```

```
Abelian Group isomorphic to Z/2 + Z + Z

Defined on 3 generators

Relations:

    2*MW.1 = 0
```

### 示例：Diffie-Hellman 密钥交换

```
q := RandomPrime (160: Proof := false);

q;

Ilog2 (q);                 // 2 进制位数

repeat
    got_it, p := RandomPrime (1024, 1, q, 1000 : Proof := false);
until got_it;

> p;

Ilog2 (p);                 // 2 进制位数

tmp := Random(p);

exp := (p - 1) div q;

> g := Modexp (tmp, exp, p);

> g;

Modexp (g, q, p);

A_priv := Random (p-1);

> A_pub := Modexp (g, A_priv, p);

> B_priv := Random (p-1);

> B_pub := Modexp (g, B_priv, p);

> A_sec := Modexp (B_pub, A_priv, p);

> B_sec := Modexp (A_pub, B_priv, p);

A_sec eq B_sec;

A_sec;
```

```
1442887065756557081811008885496150418808858816401
159
165162635946958987310283525297021062377410801210331676012348379049179592779237209111633454053865690842165269034855688505197872983769684564394428259252932666895321313697674085713119066460654757154900942213051543220536646001952121896818924040862028079007349903890347825426890204440412990981184175301600052642601
1023
83965317079455365903884386025209098411994412252894053725433285527947307311404932359964273401131497951715591859147928675735938983332715282809702511549282280553725755173696743906890758309398859062732475840898068185735680665689570386265522621592741188383919546746662841766547810852182714292042706175121823048920
1
true
```



# 5144176322605153756984581571117138785000992400326941258925591946874878623433746\  
0604279696565409179426210808910383200849384907133032735704943374046999690951643\  
2436947000024050385014162242822642019010522453269701147220601715739683999992133\  
82708798156652799536437768737718826497886878455615884529095508002441544

## Hastad’s broadcast attack on RSA

一种广播攻击 RSA，原理是孙子定理，`M` 是明文

`1024` 位的 `3` 个不同素因子密匙加密，密文只要三次出现在广播中，就能被还原为明文

函数都有，方便啊

孙子谁知道叫啥名？

```
N1 := RSAModulus (1024, 3);
N1;
N2 := RSAModulus (1024, 3);
> N2;
N3 := RSAModulus (1024, 3);
> N3;
N1 lt N2;
N2 lt N3;
M := Random (N3);
M;
C1 := Modexp (M, 3, N1);
C2 := Modexp (M, 3, N2);
> C3 := Modexp (M, 3, N3);
> C1;
C2;
C3;
ciphertexts := [C1, C2, C3];
moduli := [N1, N2, N3];
> Cprime := CRT (ciphertexts, moduli);
> Cprime;
Iroot (Cprime, 3);
```

`1191394973498639734758356886734022915243239449328167951285523220116137226774353\`  
`9176019504000834314079698783061932470536725623631410436123478027064344268960243\`  
`7364284793780727456801948423015783019014512753812345243506372098537224435020854\`  
`359848856524899561390988681729562607944000515285564642890387968646827499`

`1284091640527210186131874393840544095987650982828046236855841544838591571499173\`  
`1866298541100429948705032672190648692663519160856065108275130184865262172800857\`  
`3386565091923148415442237752544661929267855180513383588291450048199800697918748\`  
`507167493425580851595456232294309693917376008568189561641242801970999677`

`1079659736287944257378805435089043111076737452648956984975669952706815978980298\`  
`6622188200179949052343959127642578022293418583950295309545564738766859693242769\`  
`4693380883751647379784344459796750774988626755269852596427483690743738731140884\`  
`348140421109007063610908180910131454760258796787470341628217212597348237`

`true`

`false`

`1065483725312210764607948804191105051984095256035964212419006647799252157478385\`  
`6334603416677964537633362076010548745454399657943033796697306916247944219685014\`  
`0939319424884142058105500815238082740397639628813948210482176247050158934106124\`  
`746653542454869790679475333470275013517239162467216886783299705473412909`

`1924084294474065804173347971493633606471329611790262546576596759680624764052803\`  
`5603855496570716546643207106535414986292485635347396779535480273771479179167217\`  
`3693328315426059122287765033711613884956437027782317188733396126117328636526178\`  
`56045629675048923525525556790596952847575213284810439009623226349241181`

`2376889508979652383834732793528970881037787915415560763427681345964168211539753\`  
`9459811216757749141242641330948276279731548487493134449049621818003288666629642\`  
`7107106879554253057484348013589960001869058474748052966098867953029654679461620\`  
`23532932526818702053588818851843636547054094394130454387535970722272602`

`9829945919241924969246733746114350726954659414719450874216585166522394276356794\`  
`2155581726235090790726244419011366315696252177358211842671261671704146797693032\`  
`4127009226220296583554431629646178702563947023823646933477546640724280367260704\`  
`60397221620150214727998193463203577893197619215531865820102008407048363`

`1209596332738531401730087906166277185149003439550896493434779023414176254956043\`  
`2315223272998309692796071291550238017300406144851952116784618699424318350163383\`  
`0623537152218766553695974915925794283159542108168794504904334776472147979805682\`  
`6758534588794554448343265778153818561455153300516738560092882314196842832629908\`  
`0893739204365221773895249374020378141139662196321124225421875676893136477652571\`  
`3686545436735346949304011400137889800182213384268514452639716065687280905474249\`  
`3802438205591679005814273361702058789755016475321682260206347613282853189661176\`  
`9660825254532364209370935104037356711766047967220791151295375750619273595873632\`  
`4374572156522931220632061230921514554158001662097184542577633122532256834871889\`  
`8076147371585186855569055341508199943243209040147941375606027322104397902862026\`  
`8383567305221626770449237160147609033402505962655670767253025758997906062878114\`  
`56219237300141333155757438706370370416335624953661405429`

`1065483725312210764607948804191105051984095256035964212419006647799252157478385\`  
`6334603416677964537633362076010548745454399657943033796697306916247944219685014\`  
`0939319424884142058105500815238082740397639628813948210482176247050158934106124\`  
`746653542454869790679475333470275013517239162467216886783299705473412909`

---

`http://magma.maths.usyd.edu.au/magma/pdf/examples.pdf`

## Chapter 28

### Elliptic Curve Cryptography

#### 28.5. EXAMPLE: ECDSA

例子很易懂，但选安全曲线就难了

随机选一次系数 `a`，常数 `b`，`X96-2` 荐 `a=-3`，因为在常规表示和雅可比式变换中，有个表达式 `a=-3` 最方便，硬件计算快

```
p := RandomPrime (100);
> K := GF(p);
E := EllipticCurve ([Random(K), Random(K)]);
E;
p := RandomPrime (100);
> K := GF(p);
E := EllipticCurve ([-3, Random(K)]);
E;
```

为了选安全曲线， `repeat` 重复选：

```
p := RandomPrime (100);
> K := GF(p);
> repeat
repeat> E := EllipticCurve ([-3, Random(K)]);
repeat> fo := FactoredOrder (E);---------------要大素数阶
repeat> n := fo[#fo][1];
repeat> until Ilog2 (n) ge 88; ------------------2 进制位比域小
> Ilog2 (n);
IsSupersingular(E);-------------------------------要非超奇异，
```

`P=100` 位，`a` 随机选签名过程

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 69611886555621063506166355926 over GF(157135449112378660707313000637)`

`[ <2, 2>, <3, 1>, <7, 2>, <17, 1>, <181, 1>, <2099, 1>, <596447989, 1>, <69372018557, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 65388471706181964183230162768 over GF(157135449112378660707313000637)`

`[ <73, 1>, <55079, 1>, <8304629, 1>, <4705925295986567, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 126592445929249859850032581477 over GF(157135449112378660707313000637)`

`[ <5431, 1>, <7687, 1>, <3763894893517636036879, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 111827571730976256692740142243 over GF(157135449112378660707313000637)`

`[ <2, 3>, <3, 1>, <17, 1>, <36583, 1>, <10527728854582845299309, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 147021577696058357454344453248 over GF(157135449112378660707313000637)`

`[ <2, 1>, <3, 3>, <406169, 1>, <3777283, 1>, <1896680213262977, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 59756192161956097905575248378 over GF(157135449112378660707313000637)`

`[ <2, 1>, <3, 1>, <151, 1>, <263, 1>, <617, 1>, <9941, 1>, <107516467533934661, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 55629840512052832330840836298 over GF(157135449112378660707313000637)`

`[ <2, 2>, <7, 1>, <151, 1>, <37165432618821789909906481, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 151057113700827850188583571502 over GF(157135449112378660707313000637)`

`[ <2, 2>, <11, 1>, <17, 1>, <29, 1>, <640740839, 1>, <11305562359133503, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 76416094973710013389958980325 over GF(157135449112378660707313000637)`

`[ <2, 1>, <3, 2>, <7, 1>, <11093, 1>, <112422855763737340508257, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 139000244472820858413576632031 over GF(157135449112378660707313000637)`

`[ <2, 2>, <3, 1>, <5, 1>, <37, 1>, <607, 1>, <12746273, 1>, <9148487302593389, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x + 66914405844156577851178843582 over GF(157135449112378660707313000637)`

`[ <3, 1>, <7, 1>, <23, 1>, <8423, 1>, <38624266031016342385927, 1> ]`

`Elliptic Curve defined by y² = x³ + 157135449112378660707313000634*x +`



# 安全曲线与基点阶

143325094039473088283650924294 over GF(157135449112378660707313000637)

\[ <2, 1>, <3, 1>, <5, 2>, <1047569660749190309106427747, 1> \]

89

false

安全曲线重多，100 位下就有上面这些，选好了安全曲线，下面就找了个基点，并求基点的阶，基点更多，安全曲线数\*基点数\*离散指数------大 O，小 O，`ECC` 的安全指数可能真是指数级。。。

用的是下面曲线：

y² = x³ + 157135449112378660707313000634\*x +

143325094039473088283650924294 over GF(157135449112378660707313000637)

\[ <2, 1>, <3, 1>, <5, 2>, <1047569660749190309106427747, 1> \]

## 基点选择与签名验证流程

```
n;----------------------------------------------------E 中 n 阶子循环群这

P := Random(E) * (Order(E) div n);        -----找了个基点

P;

Order(P) eq n; -------------------------------------基点的阶

d := Random(n);                              

Q := d*P;                                  
Q;

M := Random(n);                               甲方 要发的明文

> M;

k := Random(n);                                甲方随机选 k<p

kp_seq := ElementToSequence (k*P);           计算点乘 k*P，表示成点

kp_seq;

r := (IntegerRing()!kp_seq[1]) mod n;       不管 k*P 多大 都要 mod n

s := (Modinv (k, n) * (M + d*r)) mod n;     经过求模逆，模乘，模加，签好了

> s;

r in [1..n-1];

s in [1..n-1];

w := Modinv (s, n);                          下面是乙方验证，也要经过求点乘，模逆，模乘，模加

u1 := M*w mod n; 

u2 := r*w mod n;

temp := u1*P + u2*Q;

temp;

temp_seq := ElementToSequence (temp);

> v := (IntegerRing()!temp_seq[1]) mod n;

> v;

v eq r;                                       验证 EQUAL,OK
```

Elliptic Curve defined by y² = x³ + 546406580005247203836151279129\*x +

230151935666323598335959051729 over GF(966371258750148727567627105897)

\[ <3, 2>, <149, 1>, <720634793997127170564126541, 1> \]三个循环子群，，经过上面 `REPEAT` 选，已知攻击都被除，阶 720634793997127170564126541 的中的点安全,循环子群点阶如过恰=曲线阶，会被多项式级的同构攻击。不过恰=可能性太小了

89

false

720634793997127170564126541

(156817043070046010480672869099 : 138406249392994102180675488148 : 1)基点座标

true

(823452749814859867769709264217 : 409839153103190558971036321686 : 1)

314368839904612579301554984 计算点乘 d\*P;

\[ 750750203409670127154353456215, 274010688382591384982046792082, 1 \]

337341060791448293483425930 明文，已被表示成曲线的点，明文，域元素，曲线点可互相转化。`X9-62` 里有，有三种形式，不太难懂，这 `ElementToSequence` 函数一定包括了这功能

true

true

(750750203409670127154353456215 : 274010688382591384982046792082 : 1)

569382858660742597097727034 计算点乘 k\*P

true

## P=100 位,a=-3 签名过程

Elliptic Curve defined by y² = x³ + 344566048772404937900222995696\*x +

122613265914087061833345770776 over GF(344566048772404937900222995699)

\[ <2, 4>, <173, 1>, <70009, 1>, <1778085018158714891237, 1> \]

Elliptic Curve defined by y² = x³ + 344566048772404937900222995696\*x +

44333872090216421338353867400 over GF(344566048772404937900222995699)

\[ <108421, 1>, <189401, 1>, <16779414654143966807, 1> \]

Elliptic Curve defined by y² = x³ + 344566048772404937900222995696\*x +

168372939479581669368060393010 over GF(344566048772404937900222995699)

\[ <2, 2>, <5, 1>, <41, 1>, <420202498502932494553451531, 1> \]

88

false

420202498502932494553451531

(19629684444028095682059018614 : 313332333218081159316329285802 : 1)

true

(185904318135579502698063049655 : 337526345404850851784130027470 : 1)

210595421815090177972701173

\[ 71071577049370184377915299896, 30419639182367625537958244039, 1 \]

178411782669416518303895082

true

true

(71071577049370184377915299896 : 30419639182367625537958244039 : 1)

57354802374592798381991157

true

估记最少得把序，扩域多项式环搞透才能下面

`http://magma.maths.usyd.edu.au/magma/pdf/examples.pdf`

格攻击

Chapter 29

`LLL` and `Lattice` Based Ciphers

格密码

Chapter 33

Miscellaneous

`NTRU`

## 超椭圆曲线

有理域下定义超椭圆曲线：

```
P<x> := PolynomialRing(RationalField());

> C := HyperellipticCurve(x⁶+x²+1);

C;
```

复域下定义超椭圆曲线：

```
P<x> := PolynomialRing(ComplexField());

> C := HyperellipticCurve(x⁶+x²+10);

C;
```

```
Hyperelliptic Curve defined by y² = x⁶ + x² + 10.0000000000000000000000000000

over Complex field of precision 30
```

有限狱模 17:

```
P1<x> := PolynomialRing(GF(17));

C17 := HyperellipticCurve(x⁶+x²+1);

C17;

P2<x> := PolynomialRing(GF(17));

C1717 := HyperellipticCurve(x¹⁶+x⁵+x²+1);

C1717;
```

扭曲线:

```
Twists(C17);
```

同构吗?:

```
IsIsomorphic(C17, C1717);
```

```
tws,auts := Twists(C17);

tws;
```

是椭圆曲线吗?

```
IsEllipticCurve(C)
```

亏格数:

```
Genus(C);

Genus(C17);

Genus(C1717);
```

阶:

```
Degree(C);

Degree(C17);

Degree(C1717);
```

判别式:

```
Discriminant(C);

Discriminant(C17);

Discriminant(C1717);
```

```
P3<x> := PolynomialRing(GF(17));

> C2 := HyperellipticCurve(x⁶+x²+x+1);
```

J 不变量:

```
JInvariants(C2);
```

绝对不变量:

```
AbsoluteInvariants(C2);
```

无限远点座标:

```
PointsAtInfinity(C);

PointsAtInfinity(C2);

PointsAtInfinity(C17);
```

求点:

```
Points(C : Bound := 1);

Points(C17 : Bound := 1);

Points(C2: Bound := 1);
```

自同构:

```
Aut(C); 

Aut(C2); 

Aut(C17) ;

Iso(C17, C2); 
```

变为雅格布式

```
J1:=Jacobian(C);

J1;

J2:=Jacobian(C2);

J2;

J17:=Jacobian(C17);

J17;
```

变为雅格布式后求阶

```
Order(J2);

Order(J17);

Curve(C17)
```

外尔下降算法分解:

```
PP<x>:=PolynomialRing(GF(2));

> h := PP!1;

> f := x⁷ + x⁴ + x³ + 1;

> J := Jacobian(HyperellipticCurve(f,h));  // a supersingular curve

> Jext := BaseExtend(J, 41);

> Factorization(#Jext);

> m := 177722253954175633;                 // some big subgroup order

> cofact := 3887047*7;

> P := cofact*Random(Jext);

> Q := 876213876263897634*P;               // Q in <P>
```

## SAGE

几个数学家椭圆曲线的各种数据全包里了，

`cremona`，`Stein-Watkins` 的有理域，`John Jones` 数域多项式

`http://www.warwick.ac.uk/staff/J.E.Cremona/book/fulltext/index.html`

前导子 `N` 分类的，默认前 10000，可加载大的到 130000

```
from sage.databases.cremona import LargeCremonaDatabase
```

随机选一条：

```
CremonaDatabase().random() 
```

```
Elliptic Curve defined by y² + x*y = x³ - x² + 324*x - 18068 over

Rational FieldElliptic Curve defined by y² + x*y = x³ - x² + 324*x - 18068 over Rational Field
```

新老版 `cremona` 字母转换：

```
old_cremona_letter_code(5221)
```

```
'UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU\
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU\
UUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUUU'
```

标签分离：

```
c = CremonaDatabase()

parse_cremona_label('37a2')

import sage.databases.cremona as cremona

cremona.split_code('37a2')
```

求前导子 21 的所有曲线：

```
c = CremonaDatabase()

c.allcurves(21)

{'a1': [[1, 0, 0, -4, -1], 0, 8], 'a3': [[1, 0, 0, -39, 90], 0, 8], 

'a2': [[1, 0, 0, -49, -136], 0, 4], 'a5': [[1, 0, 0, -784, -8515], 0, 

2], 'a4': [[1, 0, 0, 1, 0], 0, 4], 'a6': [[1, 0, 0, -34, -217], 0, 2]}{'a1': [[1, 0, 0, -4, -1], 0, 8], 'a3': [[1, 0, 0, -39, 90], 0, 8], 'a2': [[1, 0, 0, -49, -136], 0, 4], 'a5': [[1, 0, 0, -784, -8515], 0, 2], 'a4': [[1, 0, 0, 1, 0], 0, 4], 'a6': [[1, 0, 0, -34, -217], 0, 2]}
```

从 5 个不变量定义曲线:要 `CremonaData` 理的才行：

```
c = CremonaDatabase()

sage: c.elliptic_curve_from_ainvs(11, [0, -1, 1, -10, -20])
```

列出给定前导子的所有曲线：

```
c = CremonaDatabase()

CremonaDatabase().list([3171])

[Elliptic Curve defined by y² + y = x³ - x² - 7*x - 9 over Rational

Field, Elliptic Curve defined by y² + x*y + y = x³ + x² - 49*x - 58 

over Rational Field, Elliptic Curve defined by y² + x*y + y = x³ + x² 

\- 454*x + 3506 over Rational Field, Elliptic Curve defined by y² + x*y 

+ y = x³ + x² - 7249*x + 234536 over Rational Field, Elliptic Curve 

defined by y² + x*y + y = x³ + x² - 139*x + 8672 over Rational Field,

Elliptic Curve defined by y² + y = x³ + x² - 14*x + 3677 over
```



`Rational Field][Elliptic Curve defined by y² + y = x³ - x² - 7*x - 9 over Rational Field, Elliptic Curve defined by y² + x*y + y = x³ + x² - 49*x - 58 over Rational Field, Elliptic Curve defined by y² + x*y + y = x³ + x² - 454*x + 3506 over Rational Field, Elliptic Curve defined by y² + x*y + y = x³ + x² - 7249*x + 234536 over Rational Field, Elliptic Curve defined by y² + x*y + y = x³ + x² - 139*x + 8672 over Rational Field, Elliptic Curve defined by y² + y = x³ + x² - 14*x + 3677 over Rational Field]`

列出给定前导子的所有曲线数：

```
c = CremonaDatabase()

c.number_of_curves(9920)

70
```

FSD 猜想。千年问题：

```
c = CremonaDatabase()

c.allbsd(11)

{'a1': ['5', '1.2692093042795534217', '0.25384186085591068434', '1',
'1.00000000000000000000'], 'a3': ['1', '6.3460465213977671084',
'0.25384186085591068434', '1', '1'], 'a2': ['1',
'0.25384186085591068434', '0.25384186085591068434', '1',
'1.00000000000000000000']}{'a1': ['5', '1.2692093042795534217', '0.25384186085591068434', '1', '1.00000000000000000000'], 'a3': ['1', '6.3460465213977671084', '0.25384186085591068434', '1', '1'], 'a2': ['1', '0.25384186085591068434', '0.25384186085591068434', '1', '1.00000000000000000000']}
```

`SAGE` 连千年问题黎曼零点都整合了，10 万个，精度 10⁹，图灵，冯诺曼依都算过黎曼零点，《美丽心灵》纳什就是想解决黎曼零点问题发疯的

```
zz = zeta_zeros()

zz

zz[100]  --------第一百个

[14.134725142000001, 21.022039638999999, 25.01085758,
30.424876126000001, 32.935061588000003, 37.586178158999999,
40.918719011999997, 43.327073280999997, 48.005150880999999,
49.773832478000003, 52.970321478000002, 56.446247696999997,
59.347044003000001, 60.831778524999997, 65.112544048000004,
67.079810529, 69.546401711000001, 72.067157674000001,
75.704690698999997, 77.144840068999997, 79.337375019999996,
82.910380853999996, 84.735492980999993, 87.425274612999999,
88.809111208000004, 92.491899270999994, 94.651344041000002,
95.870634228, 98.831194217999993,
```

`SAGE` 把在线整数列也包了

`http://oeis.org/`

按编号找数列：

```
SloaneEncyclopedia[6083]

[1, 2, 1, 3, 1, 1, 1, 3, 3, 3, 1, 3, 1, 3, 5, 3, 1, 5, 1, 3, 7, 3, 1, 7,
1, 3, 9, 3, 1, 9, 1, 3, 11, 3, 1, 11, 1, 3, 13, 3, 1, 13, 1, 3, 15, 3,
1, 15, 1, 3, 17, 3, 1, 17, 1, 3, 19, 3, 1, 19, 1, 3, 21, 3, 1, 21, 1, 3,
23, 3, 1, 23, 1, 3, 25, 3, 1, 25, 1, 3, 27, 3, 1, 27, 1, 3, 29, 3, 1,
29, 1, 3, 31, 3][1, 2, 1, 3, 1, 1, 1, 3, 3, 3, 1, 3, 1, 3, 5, 3, 1, 5, 1, 3, 7, 3, 1, 7, 1, 3, 9, 3, 1, 9, 1, 3, 11, 3, 1, 11, 1, 3, 13, 3, 1, 13, 1, 3, 15, 3, 1, 15, 1, 3, 17, 3, 1, 17, 1, 3, 19, 3, 1, 19, 1, 3, 21, 3, 1, 21, 1, 3, 23, 3, 1, 23, 1, 3, 25, 3, 1, 25, 1, 3, 27, 3, 1, 27, 1, 3, 29, 3, 1, 29, 1, 3, 31, 3]
```

按数列找：

```
SloaneEncyclopedia.find([1,2,3,5,8], 1)

Searching Sloane's online database...

[[45,
  'Fibonacci numbers: F(n) = F(n-1) + F(n-2), F(0) = 0, F(1) = 1, F(2) = 1, ...',
  [0,
   1,
   1,
   2,
   3,
   5,
```

按编号找名称：

```
sequence_name(12)
```

`SAGE`：

康威多项式。。。。。还活着就命名多项式了。。。。

```
c = ConwayPolynomials()

 c.polynomial(3, 21)
```

约翰何顿康威，26 个散单群里有板有 3 个他找的

`http://www.hudong.com/wiki/%E7%BA%A6%E7%BF%B0%C2%B7%E4%BD%95%E9%A1%BF%C2%B7%E5%BA%B7%E5%A8%81`
