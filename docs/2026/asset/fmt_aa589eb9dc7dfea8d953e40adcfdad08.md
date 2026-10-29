
```javascript
function Kn() {
var r = arguments[t[31]];
if (!r)
return t[33];
for (var a = e[32], o = n[25], i = n[29]; i < r.length; i++) {
var u = r.charCodeAt(i)
, c = u ^ o;
o = u,
a += n[27].fromCharCode(c)
}
return a
}
```

运行报错`Y`未定义。

![](upload/attach/202308/944376_JMADHGAUTZTJN3M.png)

浏览器查看`Y`是字符串“1”，直接在代码中写固定就行

![](upload/attach/202308/944376_9R3BXH6VU39VSR7.png)

报这个错的原因是因为没有调用`b`函数，在`R`函数的开头调用运行一下`b`函数。

![](upload/attach/202308/944376_35N77F75M3TW2NR.png)

对象实例化`Un`函数是未定义的。去浏览器扣。

![](upload/attach/202308/944376_DGPYZH5CPB3KE48.png)

浏览器扣下来的`Un`函数

```javascript
var Un = function() {
var a = r[44]
, o = r[45];
function i(a) {
this[e[53]] = a;
for (var o = r[40], i = a[t[59]]; o < i; o++)
this[o] = n[29]
}
return i[r[46]][r[47]] = function() {
for (var o = t[60], i = this[n[47]], u = [], c = -n[30], s = n[29], f = i[Kn(r[48], t[61])]; s < f; s++)
for (var v = this[s], l = i[s], p = c += l; u[p] = v & parseInt(a + o, t[62]),
--l != e[27]; )
--p,
v >>= t[62];
return u
}
,
i[A + E][Xn(e[54], r[49], x)] = function(a) {
for (var i = t[63], u = this[T + o + i], c = t[35], s = n[29], f = u[Xn(n[48], r[50])]; s < f; s++) {
var v = u[s]
, l = r[40];
do {
l = (l << r[51]) + a[c++]
} while (--v > n[29]);this[s] = l >>> e[27]
}
}
,
i
}()
```

运行报`Xn`未定义，分析代码的作用是把函数赋给对象，那么`i`后面跟着的肯定是字符串，把函数赋给某某对象原型链下面的某个属性。

![](upload/attach/202308/944376_WYPDAEHV2RGA4Q6.png)

替换即可。

![](upload/attach/202308/944376_TPWR2MTEHK4Q6F6.png)

运行报`Gn`未定义，直接进入`Gn.serverTimeNow()`函数内部查看写了什么。

![](upload/attach/202308/944376_3U7ZBT44S9RVDD3.png)

浏览器扣下来的代码，看看这段代码有什么作用。

```javascript
function cn() {
var n = (s,
u,
new v[84]);
return typeof TOKEN_SERVER_TIME == x + T + C ? I[27] : (time = parseInt(TOKEN_SERVER_TIME),
time)
}
```

`cn`函数有一个三元表达式，`TOKEN_SERVER_TIME`是数值，`x + T + C`是"undefined"，`TOKEN_SERVER_TIME == x + T + C` 是false

所以执行`(time = parseInt(TOKEN_SERVER_TIME),time)`

这段函数的整体意思是将`TOKEN_SERVER_TIME`取整赋值给`time`，再把`time`值返回。

![](upload/attach/202308/944376_U7RVTMA89SZNM66.png)

`E[i] = Gn.serverTimeNow()`直接改写成`E[i] = parseInt(TOKEN_SERVER_TIME)`

运行代码报错`TOKEN_SERVER_TIME`未定义，在浏览器搜索`TOKEN_SERVER_TIME`。

![](upload/attach/202308/944376_X5VHBZ6HAJHC5QK.png)

在第一行就可以看到`TOKEN_SERVER_TIME`。直接扣下来

![](upload/attach/202308/944376_BNRP6YSHTUW2UG3.png)

运行报错`Wn`未定义。这段`C`函数代码要是直接去扣就入坑了，先看看这个`C`函数有什么作用。

![](upload/attach/202308/944376_Y9E4REXEBKEPVDE.png)

第一行代码就是取`cookie`名为`v`的值，如果没取到`cookie`的`v`值就去取`headers`的"hexin-v"值。

```javascript
var r = Wn.getCookie(On) || $n.get(Pn);
```

![](upload/attach/202308/944376_ZRAVR9S8MM9ATZ6.png)

一般情况下都不是第一次运行，那么他就会取得到前面`cookie`的`v`值。那么下面的`if`判断就会成立，就会执行。但是我们总不可能每次运行代码都去取他前面的`cookie`，那实在是太复杂了。假设，我是说假设。假设我每一次进去这个网站都是第一次，都取不到`cookie`的`v`值和`headers`的`hexin-v`。那么这段代码又是有什么样的作用呢。

![](upload/attach/202308/944376_QR89SVFERKUFHVN.png)

`cookie`的`v`和`headers`的`hexin-v`都取不到值的话那么`if`里面的就是`null`。就直接执行后面的代码

![](upload/attach/202308/944376_A2PN46UQ5WGK7TB.png)

最后`C`函数的代码就是直接改写成`E[o] = Gn.random()`

运行报错,直接扣`Gn.random()`

![](upload/attach/202308/944376_QB5Y4FEC9HX8UZ6.png)

```javascript
function rn() {
var n = sn;
return n = G,
Math.random() * parseInt("11111111111111111111111111111111", 2) >>> 0
}
```

`C`函数可以值改写成

```javascript
E[o] =Math.random() * parseInt("11111111111111111111111111111111", 2) >>> 0
```

`Dn`报错

![](upload/attach/202308/944376_6H2GRMFYF283DUK.png)

`Dn`是数值`0`

![](upload/attach/202308/944376_3E8BMTQDVQXU7ZS.png)

`xn`未定义

![](upload/attach/202308/944376_EX93GR4DHUV9CYD.png)

`xn`是数值`3`

![](upload/attach/202308/944376_2YABESTTB7ZK8RU.png)

报了`Gn`的错。分析这段代码就是把浏览器指纹`ua`传进了`strhash`函数。可以直接取最后的值写固定。

![](upload/attach/202308/944376_6HQD6WGGGHZ4FXY.png)

```javascript
E[c] = 自己浏览器头strhash后的值
```

后面的报错都是这几个值未定义。

![](upload/attach/202308/944376_EEDJTV83SYR3SXT.png)

`getBrowserFeature`—>获得浏览器特性

`getPlatform`—>获取浏览器平台

`getBrowserIndex`—>获取当前网页所在浏览器的索引

`getPluginNum`—>获取当前网页或浏览器环境中已安装或可用的插件数量。

至于改写的话直接取最后的值就行了

`b`函数都改完之后到`R`函数报错了。`Gn.serverTimeNow()`在`b`函数就已经扣过了

`Gn.serverTimeNow()`—>`parseInt(TOKEN_SERVER_TIME)`

![](upload/attach/202308/944376_ESF8ZGNPYZJXZYX.png)

`Gn.timeNow()`未定义

![](upload/attach/202308/944376_F4BT9P3RTRHNWWF.png)

进入`Gn.timeNow()`函数内部是个`on`函数

![](upload/attach/202308/944376_KFZ2G5KKE3YSHP9.png)

`on`函数的作用就是取当前时间戳除以`1000`再无符号右移`0`位

改写后的代码

```javascript
E[u] = Date.now()/ 1000
```

后面的代码都是未定义的

![](upload/attach/202308/944376_SWXBA5733PHGACT.png)

`Dn`是数值`0`，`n`是字符串"DOMMous"

![](upload/attach/202308/944376_TK8Y4XFNYBPZ99J.png)

`getMouseMove`—>获取鼠标当前的横坐标和纵坐标

`getMouseClick`—>获取鼠标点击时的横坐标和纵坐标，以及所点击的按钮

`getMouseWhell`—>获取滚轮事件的信息

`getKeyDown`—>获取按键事件的信息

`getClickPos`—>获取鼠标点击的坐标，然后进行相应的操作，比如记录点击位置或者根据点击位置执行特定的逻辑。

直接取最后运行完的值

运行`Qn.encode`未定义

![](upload/attach/202308/944376_FXQDMXWUVQDFWCF.png)

进入函数内部`Qn`对象总共有`4`个方法，直接把`Qn`对象补齐.

![](upload/attach/202308/944376_PNU8H6R5GD6UGNY.png)

```javascript
var Qn;
!function(a) {
var o = e[55]
, i = n[49]
, u = n[50]
, c = r[52];
function s(a) {
var o, i, u;
o = i = u = t;
var c, s, f;
c = s = f = r;
var v, p, d;
v = p = d = e;
var h, g, m;
h = g = m = n;
for (var w = g[29], I = (en,
on,
a[d[56]]), _ = []; w < I; ) {
var y = l[a.charAt(w++)] << parseInt(p[57], v[58]) | l[a.charAt(w++)] << d[59] | l[a.charAt(w++)] << v[60] | l[a.charAt(w++)];
_.push(y >> parseInt(Vn(f[53], Tn), d[61]), y >> c[51] & parseInt(Xn(X, p[62]), g[51]), y & parseInt(u[64], h[52]))
}
return _
}
function f(n, t, a, i, u) {
for (var c = n[o + C]; t < c; )
a[i++] = n[t++] ^ u & parseInt(r[54], e[61]),
u = ~(u * parseInt(e[63], r[55]))
}
for (var v = e[64], l = {}, p = n[29]; p < parseInt(Vn(t[65], r[56]), r[51]); p++)
l[v.charAt(p)] = p;
function d(n) {
for (var a = r[40], o = t[35], i = n[t[59]]; o < i; o++)
a = (a << r[57]) - a + n[o];
return a & parseInt(Xn(q, t[66], In), e[61])
}
function h(r) {
var e = d(r)
, a = [c, e];
return f(r, +n[29], a, +t[67], e),
g(a)
}
function g(a) {
for (var o = r[40], u = (G,
i,
a[e[56]]), c = []; o < u; ) {
var s = a[o++] << parseInt(b + R, r[51]) | a[o++] << r[51] | a[o++];
c.push(v.charAt(s >> parseInt(e[65], n[53])), v.charAt(s >> parseInt(n[54], t[62]) & parseInt(Xn(e[66], r[58], S), e[61])), v.charAt(s >> r[59] & parseInt(B + i, t[46])), v.charAt(s & parseInt(n[55], r[55])))
}
return c.join(Vn(r[26], n[56]))
}
function m(n) {
var t, a, o;
t = a = o = r;
var i, u, v;
i = u = v = e;
var l = s(n);
if (l[u[27]] != c)
return error = i[67],
void 0;
var p = l[t[37]]
, h = [];
return f(l, +a[60], h, +i[27], p),
d(h) == p ? h : void 0
}
a[Vn(r[61])] = g,
a[Hn($n, n[57], t[68])] = s,
a[k + u] = h,
a[O + S] = m
}(Qn || (Qn = {}));
```

运行报错

![](upload/attach/202308/944376_9E4TV9PFFC45AHU.png)

在浏览器打印`o+C`是字符串"length"，直接替换

![](upload/attach/202308/944376_S2KHF7RW3KD22QU.png)

![](upload/attach/202308/944376_MWHBPJX92VX6XJ5.png)

在浏览器打印`Vn(t[65],r[56])`是字符串"100"，直接替换

![](upload/attach/202308/944376_QBU2Q6RU8FUBAM7.png)

后面几个都是未定义的，在浏览器找到对应的值替换

![](upload/attach/202308/944376_X4QGTCEH4E5QHZ2.png)

![](upload/attach/202308/944376_E67GGGK6U5EUAED.png)

```javascript
a["base64Encode"] = g,
a["base64Decode"] = s,
a["encode"] = h,
a["decode"] = m
```

直接拿运行之后的结果进行替换

![](upload/attach/202308/944376_PUE569HPJ8H8WJY.png)

`Xn(q, t[66], In)`—>"ff"

![](upload/attach/202308/944376_CWNYMACSWN6MQMB.png)

![](upload/attach/202308/944376_DBTRMZUK2QDU5U5.png)

`G`—>"w"

![](upload/attach/202308/944376_QZ2EPNRP6DC3WXD.png)

![](upload/attach/202308/944376_PKEYA86QQ96GXCK.png)

`Xn(e[66], r[58], S)`—>"3f"

![](upload/attach/202308/944376_BPQYBT3V9NCKZEV.png)

![](upload/attach/202308/944376_HVZTHN79R4YAMPW.png)

`B + i`—>"3f"

![](upload/attach/202308/944376_BM2MXCMH5PZH48W.png)

![](upload/attach/202308/944376_9HEMHTZJRWEXSHZ.png)

`Vn(r[26], n[56])`—>""

![](upload/attach/202308/944376_C42D69TY7GMSZNA.png)

代码再次执行，值便出来了。

![](upload/attach/202308/944376_AHGU2PQHVVNV8AK.png)

带上值去请求一遍。手机验证码发送成功

![](upload/attach/202308/944376_Z7MUPBBCZZSWJSG.png)

# hexin-v(js-dom补环境)

在网上看到一堆文章都又说直接用`js-dom`包补`document`和`window`两个环境。

直接把整个`js`文件复制下来

![](upload/attach/202308/944376_2DS9EAJR85ZVPSR.png)

定义一个全局变量接受`v`的值,方便后面导出。

运行代码报`window`未定义

![](upload/attach/202308/944376_K7EPRKWYZFH8AH5.png)

导包

```javascript
const jsdom = require('jsdom')
const {JSDOM} = jsdom;
const {window} = new JSDOM(``);
```
