
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

`class` `Base64New:` `RANGE` `=` `255` `Base64ByteToStr` `=` `[``'v'``,` `'w'``,` `'x'``,` `'r'``,` `'s'``,` `'t'``,` `'u'``,` `'o'``,` `'p'``,` `'q'``,` `'3'``,` `'4'``,` `'5'``,` `'6'``,` `'7'``,` `'A'``,` `'B'``,` `'C'``,` `'D'``,` `'E'``,` `'F'``,` `'G'``,` `'H'``,` `'I'``,` `'J'``,` `'y'``,` `'z'``,` `'0'``,` `'1'``,` `'2'``,` `'P'``,` `'Q'``,` `'R'``,` `'S'``,` `'T'``,` `'K'``,` `'L'``,` `'M'``,` `'N'``,` `'O'``,` `'Z'``,` `'a'``,` `'b'``,` `'c'``,` `'d'``,` `'U'``,` `'V'``,` `'W'``,` `'X'``,` `'Y'``,` `'e'``,` `'f'``,` `'g'``,` `'h'``,` `'i'``,` `'j'``,` `'k'``,` `'l'``,` `'m'``,` `'n'``,` `'8'``,` `'9'``,` `'+'``,` `'/'``]` `StrToBase64Byte` `=` `{char: idx` `for` `idx, char` `in` `enumerate``(Base64ByteToStr)}` `def` `Base64Encode(``self``, bytes):` `res` `=` `[]` `for` `i` `in` `range``(``0``,` `len``(bytes),` `3``):` `enBytes` `=` `[``0``]` `*` `4` `tmp` `=` `0` `for` `k` `in` `range``(``3``):` `if` `i` `+` `k <` `len``(bytes):` `enBytes[k]` `=` `((bytes[i` `+` `k] &` `255``) >> ((k` `*` `2``)` `+` `2``)) | tmp` `tmp` `=` `(((bytes[i` `+` `k] &` `255``) << ((``2` `-` `k)` `*` `2` `+` `2``)) &` `255``) >>` `2` `else``:` `enBytes[k]` `=` `tmp` `tmp` `=` `64` `enBytes[``3``]` `=` `tmp` `for` `k2` `in` `range``(``4``):` `if` `enBytes[k2] <``=` `63``:` `res.append(``self``.Base64ByteToStr[enBytes[k2]])` `else``:` `res.append(``'='``)` `return` `''.join(res)` `def` `Base64Decode(``self``, encoded_str):` `decoded_bytes` `=` `bytearray()` `for` `i` `in` `range``(``0``,` `len``(encoded_str),` `4``):` `enChars` `=` `encoded_str[i:i` `+` `4``]` `deBytes` `=` `[``0``]` `*` `4` `for` `k` `in` `range``(``4``):` `if` `enChars[k] !``=` `'='``:` `deBytes[k]` `=` `self``.StrToBase64Byte[enChars[k]]` `decoded_bytes.extend((((deBytes[``0``] <<` `2``) &` `255``) | (deBytes[``1``] >>` `4``)).to_bytes(``1``,` `'big'``))` `if` `enChars[``2``] !``=` `'='` `and` `len``(enChars) >` `2``:` `decoded_bytes.extend((((deBytes[``1``] <<` `4``) &` `255``) | (deBytes[``2``] >>` `2``)).to_bytes(``1``,` `'big'``))` `if` `enChars[``3``] !``=` `'='` `and` `len``(enChars) >` `3``:` `decoded_bytes.extend((((deBytes[``2``] <<` `6``) &` `255``) | deBytes[``3``]).to_bytes(``1``,` `'big'``))` `return` `decoded_bytes`

再将 `5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs=` 值代入其中，解密后得到：`05397c42f9b6da593a3644162d36eb01`

输入到APP 显示验证成功。

### 解法2：

还有一种处理的办法，从他的代码其实可以看出他是一个base64加密的换表算法，因为加密的逻辑是符合base64的原理的，然后我们写代码来验证下：

```
import base64
s1 = "vwxrstuopq34567ABCDEFGHIJyz012PQRSTKLMNOZabcdUVWXYefghijklmn89+/"
s2 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

# abcdefghijklmnopqrstuvwxyz0123456789+/ABCDEFGHIJKLMNOPQRSTUVWXYZ
data = "5rFf7E2K6rqN7Hpiyush7E6S5fJg6rsi5NBf6NGT5rs="
print(str.maketrans(s1, s2))
print(data.translate(str.maketrans(s1, s2)).encode('utf-8'))
key = str(base64.b64decode(data.translate(str.maketrans(s1, s2)).encode('utf-8')), encoding="utf-8")
print(" key = ", key)
#  key =  05397c42f9b6da593a3644162d36eb01
```

结果是一样的，所以算法的底层逻辑是很重要的，必要时候能节省很多时间。

### 0x03 easy_java

这题是基于java层的，算法是层层嵌套，代码逻辑是非常清晰的，将输入的字符串经过层层处理，得到一个新的字符串，满足`sb.toString().equals("wigwrkaugala")`即可破解，我们来分析下：

![图片描述](https://bbs.kanxue.com/upload/attach/202312/961239_W6AWSXE4SSNW2HR.webp)

我们先来分析他正常逻辑的算法：

这段代码看起来是在处理一个形如 `"flag{...}"` 的字符串，通过某种特定的处理和比较，最终确定是否符合特定的条件。

1. `if (str.startsWith("flag{") && str.endsWith("}"))` 可以看出 输入的字符串必须包含`flag{}`

2. `aVar.a(bVar.a(str))` 这里面计算出来的值很关键。

   frida hook `aVar.a(bVar.a(str))`方法：

```
function hook_java() {
Java.perform(function () {
var MainActivity = Java.use("com.a.easyjava.MainActivity");
MainActivity["a"].overload('java.lang.String','com.a.easyjava.b','com.a.easyjava.a').implementation = function (str, bVar, aVar) {
console.log("MainActivity.a is called: str=${str}, bVar=${bVar}, aVar=${aVar}",str,bVar,aVar);
var result = this["a"](str, bVar, aVar);
console.log("MainActivity.a result=${result}", result);
return result;
};
var a = Java.use("com.a.easyjava.a");
a["a"].overload('java.lang.Integer').implementation = function (num) {
console.log("a.a is called: num=${num}", num);
var result = this["a"](num);
console.log("a.a result=${result}", result);
return result;
};

var b = Java.use("com.a.easyjava.b");
b["a"].overload('java.lang.String').implementation = function (str) {
console.log("b.a is called: str=${str}", str);
var result = this["a"](str);
console.log("b.a result=${result}", result);
return result;
};
})
}

function main() {
hook_java()
}

setImmediate(main)
```

![图片描述](https://bbs.kanxue.com/upload/attach/202312/961239_8MZAVJ24A2XHHHA.webp)

拿到值之后，我们先分析下：

输入`abced`

b方法: a 9、b 8、c 7、e 1、d 21

a方法：9 h、8 w、7 x、1 v、21 a

用python 还b方法验证：

```
class B:
a = []
b = "abcdefghijklmnopqrstuvwxyz"
d = 0

c = [8,25,17,23,7,22,1,16,6,9,21,0,15,5,10,18,2,24,4,11,3,14,19,12,20,13]

def __init__(self, num):
for value in self.c[num:]:
self.a.append(value)
for value in self.c[:num]:
self.a.append(value)
self.a = self.a[:len(self.c)-1]
print(self.a)
@staticmethod
def process_input():
global b, d
value = B.a[0]
B.a.pop(0)
B.a.append(value)
b += b[0]
b = b[1:27]
d += 1

def process_string(self, string):
i = 0
if string.lower() in b:
value = b.index(string.lower())
for i2 in range(len(self.a) - 1):
if self.a[i2] == value:
i = i2
else:
i = -10 if " " in string else -1
B.process_input()
return i

def get_d(self):
return d

# 示例用法：
b = "abcdefghijklmnopqrstuvwxyz"
d = 0
q = 2
jiami_str = "abced"
for jstr in jiami_str:
print("jstr ",jstr)
obj = B(q)
result = obj.process_string(jstr)
print(result)
# 示例返回：2
q = q + 1
```

![图片描述](https://bbs.kanxue.com/upload/attach/202312/961239_A8SWXK3GCD5HUVB.webp)

和hook的值一样 没有问题。

再还原a方法进行验证：
