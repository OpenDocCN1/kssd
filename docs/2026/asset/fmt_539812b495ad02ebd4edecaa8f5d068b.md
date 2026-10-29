
```
Fr = function(a, b) {
var test;
for (var c = 0; c < b.length - 2; c += 3) {
var d = b.charAt(c + 2);
d = "a" <= d ? d.charCodeAt(0) - 87 : Number(d);
test=a >>> d;
test=a << d;
d = "+" == b.charAt(c + 1) ? a >>> d : a << d;
a = "+" == b.charAt(c) ? a + d & 4294967295 : a ^ d
}
return a
}
var Er = function(a) {
return function() {
return a
}
}
var TKK="428257.4099441762";
Hr = function(a) {
var b=TKK;
var d = Er(String.fromCharCode(116));//t
c = Er(String.fromCharCode(107));//k
d = [d(), d()];
d[1] = c();//tk
c = "&" + d.join("") + "=";//&tk=
d = b.split(".");
b = Number(d[0]) || 0;
for (var e = [], f = 0, g = 0; g < a.length; g++) {
var l = a.charCodeAt(g);
128 > l ? e[f++] = l : (2048 > l ? e[f++] = l >> 6 | 192 : (55296 == (l & 64512) && g + 1 < a.length && 56320 == (a.charCodeAt(g + 1) & 64512) ? (l = 65536 + ((l & 1023) << 10) + (a.charCodeAt(++g) & 1023),
e[f++] = l >> 18 | 240,
e[f++] = l >> 12 & 63 | 128) : e[f++] = l >> 12 | 224,
e[f++] = l >> 6 & 63 | 128),
e[f++] = l & 63 | 128)
}
a = b;
for (f = 0; f < e.length; f++) {
a += e[f];
a = Fr(a, "+-a^+6");
}
a = Fr(a, "+-3^+b+-f");
a ^= Number(d[1]) || 0;
0 > a && (a = (a & 2147483647) + 2147483648);
a %= 1E6;
return c + (a.toString() + "." + (a ^ b))
};
tk=Hr("hello world")
console.log(tk);//输出&tk=904423.738310
```

还原为 Python 代码:

```
TKK="428257.4099441762"
def UShiftRight(a,b):
if b!=0:
if(a&0x80000000):
a&=0x7fffffff;
a>>=1;
b-=1
a|=0x40000000;
return a>>b;
def Fr(a,b):
a&=0xffffffff;
for c in range(0,b.__len__(),3):
d=b[c+2]
if("a" <= d):
d=ord(d)-87;
else:
d=int(d)
if("+" == b[c+1]):
d=UShiftRight(a,d)
else:
d=a<<d
d&=0xffffffff
if("+"==b[c]):
a=a+d&4294967295
else:
a=a^d
a&=0xffffffff;
return a
def Hr(a):
b=TKK;
d=b.split('.')
b=int(d[0])
e={}
f=0;
aLen=a.__len__()
for g in range(aLen):
l=ord(a[g])
if(128>l):
e[f]=l;f+=1;
elif 2048>l:
e[f]=l>>6|192;f+=1;
elif 55296 == (l & 64512) and g + 1 < aLen and 56320 == (ord(a[g + 1]) & 64512):
l = 65536 + ((l & 1023) << 10) + (ord(a[++g]) & 1023)
e[f]=l>>18|240;f+=1;
e[f]=l>>12&63 |128;
else:
e[f] = l >> 12 | 224;f+=1;
e[f] = l >> 6 & 63 | 128;f+=1;
e[f] = l & 63 | 128;f+=1;
a=b;
for f in range(e.__len__()):
a+=e[f];
a = Fr(a, "+-a^+6");
a= Fr(a, "+-3^+b+-f");
a^=int((d[1]))
if(0>a):
a=(a&0x7fffffff)+0x80000000;
a%=1E6;
a=int(a);
s1=str(a);
return (str(a)+"."+str(a^b))
tk=Hr("我")
print(tk)
```

通过`上面函数`算出来的 `tk` 就可以`正确返回内容`了.

### 协议

URL:`8d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7X3q4F1M7$3I4S2N6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3!0Y4L8r3g2Q4x3X3g2U0L8W2)9J5c8Y4c8J5j5h3&6K6L8r3q4@1k6g2)9#2k6X3q4Q4x3V1k6K6K9h3&6Y4L8r3g2Q4x3@1k6U0L8r3W2W2L8Y4c8Q4x3@1c8@1i4K6t1$3M7$3I4Q4x3@1c8W2L8W2)9J5y4Y4c8D9i4K6y4p5P5X3S2Q4x3X3c8o6e0W2)9J5y4X3S2D9i4K6y4p5P5X3S2Q4x3X3c8o6e0W2)9J5y4X3c8@1i4K6y4p5j5i4c8Q4x3U0k6V1N6q4)9K6c8r3u0V1i4K6t1$3k6s2c8Q4x3@1c8W2P5q4)9J5y4X3c8@1i4K6y4p5L8r3c8Q4x3U0k6V1N6q4)9K6c8r3#2V1i4K6t1$3k6s2c8Q4x3@1c8I4j5$3q4Q4x3U0k6V1N6q4)9K6c8s2u0%4i4K6t1$3k6s2c8Q4x3@1c8J5L8g2)9J5y4X3c8@1i4K6y4p5M7%4y4Q4x3U0k6V1N6q4)9K6c8s2c8Q4x3U0k6A6k6g2)9K6c8q4g2f1c8W2)9J5k6o6S2Q4x3U0k6G2k6g2)9K6c8q4g2f1c8W2)9J5k6o6S2Q4x3U0k6H3j5#2)9K6c8o6q4Q4x3U0k6G2N6r3k6Q4x3@1b7I4i4K6t1$3M7%4y4W2L8q4)9K6c8o6m8Q4x3U0k6@1M7$3g2D9i4K6y4p5x3q4)9J5y4X3E0U0i4K6y4p5x3g2)9J5y4Y4c8C8i4K6y4p5x3U0j5@1z5e0t1&6i4K6u0W2x3e0j5@1z5o6l9^5i4K6t1$3M7g2)9K6c8r3S2W2L8r3I4G2`

| 参数 | 含义 |
| ---- | --------------------------------------- |
| sl | 需要转换的语言 |
| tl | 转换到的语言 |
| tk | 对输入内容的签名(算法看前面) |
| q | 输入的内容 |
| 其他 | 无关紧要,可以去掉 是指示谷歌返回的内容. |

Header:不需要

Method:`GET`

### 总结

技术难度:无.

加密强度:低.

混淆强度:低.

### 工具

1. 谷歌浏览器

2. 谷歌浏览器自带开发者工具(快捷键`F12`)

### 谷歌翻译协议分析

1. 浏览器打开`5e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7X3q4F1M7$3I4S2N6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8R3``.

2. 打开开发者工具进行抓包

3. 分析 js 脚本

4. 还原算法

### 抓包

输入 hello 实用进行抓包,抓取的内容如下

请求 URL:

```
Request URL: cddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7X3q4F1M7$3I4S2N6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8W2)9J5c8Y4c8J5j5h3&6K6L8r3q4@1k6g2)9#2k6X3q4Q4x3V1k6K6K9h3&6Y4L8r3g2Q4x3@1k6U0L8r3W2W2L8Y4c8Q4x3@1c8@1i4K6t1$3j5h3#2H3i4K6y4n7M7$3I4Q4x3@1c8W2L8W2)9J5y4X3q4E0M7q4)9K6b7Y4c8D9i4K6y4p5P5X3S2Q4x3X3c8o6e0W2)9J5y4X3q4E0M7q4)9K6b7X3S2D9i4K6y4p5P5X3S2Q4x3X3c8o6e0W2)9J5y4X3q4E0M7q4)9K6b7X3c8@1i4K6y4p5j5i4c8Q4x3U0k6S2L8i4m8Q4x3@1u0V1N6q4)9K6c8r3u0V1i4K6t1$3j5h3#2H3i4K6y4n7k6s2c8Q4x3@1c8W2P5q4)9J5y4X3q4E0M7q4)9K6b7X3c8@1i4K6y4p5L8r3c8Q4x3U0k6S2L8i4m8Q4x3@1u0V1N6q4)9K6c8r3#2V1i4K6t1$3j5h3#2H3i4K6y4n7k6s2c8Q4x3@1c8I4j5$3q4Q4x3U0k6S2L8i4m8Q4x3@1u0V1N6q4)9K6c8s2u0%4i4K6t1$3j5h3#2H3i4K6y4n7k6s2c8Q4x3@1c8J5L8g2)9J5y4X3q4E0M7q4)9K6b7X3c8@1i4K6y4p5M7%4y4Q4x3U0k6S2L8i4m8Q4x3@1u0V1N6q4)9K6c8s2c8Q4x3U0k6S2L8i4m8Q4x3@1u0A6k6g2)9K6c8q4g2f1c8W2)9J5k6o6S2Q4x3U0k6S2L8i4m8Q4x3@1u0G2k6g2)9K6c8q4g2f1c8W2)9J5k6o6S2Q4x3U0k6S2L8i4m8Q4x3@1u0H3j5#2)9K6c8o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0G2N6r3k6Q4x3@1b7I4i4K6t1$3j5h3#2H3i4K6y4n7M7%4y4W2L8q4)9K6c8o6m8Q4x3U0k6S2L8i4m8Q4x3@1u0@1M7$3g2D9i4K6y4p5x3q4)9J5y4X3q4E0M7q4)9K6b7X3E0U0i4K6y4p5x3g2)9J5y4X3q4E0M7q4)9K6b7Y4c8C8i4K6y4p5x3U0j5@1z5e0t1&6i4K6u0W2x3e0j5@1z5o6l9^5i4K6t1$3j5h3#2H3i4K6y4n7M7g2)9K6c8r3S2W2L8r3I4G2
```

Request Method: `GET`

Status Code: `200`

Remote Address: `127.0.0.1:8888`

Referrer Policy: `no-referrer-when-downgrade`

返回内容:
