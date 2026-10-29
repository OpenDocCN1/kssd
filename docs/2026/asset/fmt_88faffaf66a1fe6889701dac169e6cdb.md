
# JS 还原C++函数总结

```javascript
function stringToUint8Array(str){
var arr = [];
for (var i=0;i<str.length;i++)
{
arr.push(str.charCodeAt(i));
}
var outputbytes = new Uint8Array(arr);
return outputbytes;
}
```

这么看不太直观。拿刚才的第三题举例

C++中

```c++
char str1[] = "cPN~zk)|Kpehw7zZn;PU6Gg_kj*[Z";
string str(str1);
int n = str.length();
int param2 = 0;
char* v17 = nullptr;
v17 = (char*)param2;
unsigned char* str2 = sub_8B04(int(""), (int)str1, n);
```

这里面`str1`是char数组被强转为整型当成传参进入的`sub_8B04`函数

但在`JS`中，我们进入`sub_8B04`就是传参`str`是字符串，不需要变整型

![image-20231105115216312](upload/attach/202311/940967_C9QHA9QNUNXD6EN.png)

为了方便后面的字节偏移计算，我们再把字符串`str`转为8位无符号整型数组

```javascript
var a2 = stringToUint8Array(str);
```

## C++ 函数里面与字符串类型传参相关的参数 在JS中的还原

```c++
unsigned char* sub_8B04(int a1, int a2, int a3)
{
int v7 = 0; // r5
v9 = a2 + v7; //开始出现a2(传进来的参数)
}
```

在JS中还原C++函数`sub_8B04`里面的参数`V9`思路:

因为`a2`原先是被int强转char数组。`v9`是(`char*`)`a2`数组偏移`v7`个字节后的参数

在前面的步骤中，`a2`在JS中已经被我们设置为8位无符号整型数组，所以`v9 = a2[v7]`

```js
function sub_8B04(str,len){
var a2 = stringToUint8Array(str);
v9 = a2[v7];
}
```

根据这个思路

使用JS还原一下C++代码中`* (unsigned __int8*)(v9 + 1)`和`* (unsigned __int8*)(v9 + 2)`代码如下

```js
var v9_1 = a2[v7+1];
var v9_2 =a2[v7+2];
```

## 字节运算

由于JS语言的特性，不能直接把字符和数字进行直接的运算，所以要把字符转为`Unicode`编码与数字进行运算后，再把得到的结果重新转为字符

需要用js的`charAt`、`charCodeAt`、`fromCharCode`三个函数进行转化，这里面贴一下介绍

> `charAt` 方法用于返回指定索引位置的字符。索引位置从0开始计数。
>
> 语法: `string.charAt(index)`
>
> 示例:
>
> ```javascript
> var str = "Hello World";
> console.log(str.charAt(0)); // 输出 "H"
> console.log(str.charAt(6)); // 输出 "W"
> ```
>
> `charCodeAt` 方法返回指定索引位置的字符的Unicode编码。索引位置从0开始计数。
>
> 语法: `string.charCodeAt(index)`
>
> 示例:
>
> ```javascript
> var str = "Hello World";
> console.log(str.charCodeAt(0)); // 输出 72
> console.log(str.charCodeAt(6)); // 输出 87
> ```
>
> `fromCharCode` 方法从Unicode编码创建一个字符串。
>
> 语法: `String.fromCharCode(number1, number2, ... , numberX)`
>
> 示例:
>
> ```javascript
> console.log(String.fromCharCode(72, 101, 108, 108, 111)); // 输出 "Hello"
> ```

现在分析一下前面第三题贴的C代码

```c++
*v8 = aAyzpq23ijrtffg[*(unsigned __int8*)(a2 + v7) >> 2] ^a3;
```

这段C代码的含义是将变量`a2`和`v7`的和作为地址，取出该地址处的一个字节数据，将其右移两位后转换为无符号8位整数，然后使用这个值作为索引，从数组`aAyzpq23ijrtffg`中取出对应位置的值，与变量`a3`进行异或操作，并将结果赋给变量`v8`。

在JS中`aAyzpq23ijrtffg[*(unsigned __int8*)(a2 + v7) >> 2]`这个字符要和`a3`进行异或，`a3`是整型不能和字符异或，所以我们要先把字符转为`Unicode`码再和`a3`进行异或运算，然后转为字符串如下

```javascript
v8 += String.fromCharCode(aAyzpq23ijrtffg.charCodeAt(a2[v7] >>2 ) ^ a3);
```

# Python 还原C++函数总结

## 字符串篇

### C++字符串型的传参在python中的变化

C++的函数传参如果是char数组，当使用`python`对其进行还原时候，为了方便后面字符串型参数在`python`中计算字符串偏移，最好把字符串转为`bytes`数组（对应是`C++`中的`unsigned char*`）

如下

```python
def sub_8B04(str,len):
a2 = bytes(str,"utf-8")
```

### C++ 函数里面与字符串类型传参相关的参数 在python中的还原

```c++
unsigned char* sub_8B04(int a1, int a2, int a3)
{
int v7 = 0; // r5
v9 = a2 + v7; //开始出现a2(传进来的参数)
}
```

在python中还原C++函数`sub_8B04`里面的参数`V9`思路:

因为`a2`原先是被int强转char数组。`v9`是(`char*`)`a2`数组偏移`v7`个字节后的参数

在前面的步骤中，`a2`在python中已经被我们设置为`bytes`数组，所以`v9 = a2[v7]`

```python
v9 = a2[v7];
```

根据这个思路

使用python还原一下C++代码中`* (unsigned __int8*)(v9 + 1)`和`* (unsigned __int8*)(v9 + 2)`代码如下

```python
v9_1 = a2[v7+1]
v9_2 =a2[v7+2]
```

## 字节运算

由于python语言的特性，不能直接把字符和数字进行直接的运算，所以要把字符转为`Unicode`编码与数字进行运算后，再把得到的结果重新转为字符

> chr和ord是Python中的内置函数，用于字符和对应的Unicode编码之间的转换。
>
> chr函数接受一个整数参数，返回对应的字符。例如：
>
> ```
> print(chr(65)) # 输出A
> print(chr(97)) # 输出a
> print(chr(8364)) # 输出€
> ```
>
> ord函数接受一个字符参数，返回对应的Unicode编码。例如：
>
> ```
> print(ord('A')) # 输出65
> print(ord('a')) # 输出97
> print(ord('€')) # 输出8364
> ```
>
> 通过chr和ord函数，我们可以方便地在字符和对应的Unicode编码之间进行转换。

现在分析一下前面第三题贴的C代码

```c++
*v8 = aAyzpq23ijrtffg[*(unsigned __int8*)(a2 + v7) >> 2] ^a3;
```

这段C代码的含义是将变量`a2`和`v7`的和作为地址，取出该地址处的一个字节数据，将其右移两位后转换为无符号8位整数，然后使用这个值作为索引，从数组`aAyzpq23ijrtffg`中取出对应位置的值，与变量`a3`进行异或操作，并将结果赋给变量`v8`。

在python中`aAyzpq23ijrtffg[*(unsigned __int8*)(a2 + v7) >> 2]`这个字符要和`a3`进行异或，`a3`是整型不能和字符异或，所以我们要先把字符转为`Unicode`码再和`a3`进行异或运算，然后转为字符串如下

```python
v8 += chr(ord(aAyzpq23ijrtffg[a2[v7] >>2]) ^ a3)
```

# 导读

你能从本文中学到apk中的so层简单算法还原以及使用js和python还原C代码的部分细节

题目链接

链接：[0adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9r3u0C8P5V1&6K9M7f1W2c8h3h3Z5$3M7Y4N6p5M7q4y4k6N6q4)9#2k6X3q4c8](<https://bbs.kanxue.com/elink@af0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9r3u0C8P5V1&6K9M7f1W2c8h3h3Z5$3M7Y4N6p5M7q4y4k6N6q4)9#2k6X3q4c8>)

提取码：1234

# 题目一

使用`jadx`打开`algorithmbase_10.apk`

![image-20231101144427292](https://bbs.kanxue.com/upload/attach/202311/940967_8ZVX7SRD3KKT7KX.png)

## JAVA层

使用`Frida`获取先生成的随机字符串

```javascript
// 定义一个名为hook_js的JavaScript函数
function hook_js(){
    // 使用Java.perform()函数来执行JavaScript代码
    Java.perform(function(){
        // 使用Java.use()函数来获取Java类com.kanxue.algorithmbase.MainActivity
        var m_randomAscii = Java.use("com.kanxue.algorithmbase.MainActivity")
        // 检查获取的类对象是否存在
        if(m_randomAscii!=undefined){
            console.log("开始Hook"); // 打印开始Hook的消息
            // 重写函数encodeFromJni_11的实现
            m_randomAscii.encodeFromJni_11.implementation = function(input){
                // 获取输入参数input
                var res = this.encodeFromJni_11(input);
                // 打印输入参数和返回值
                console.log("input:==>"+input);
                console.log("res:==>"+res);
                // 返回结果
                return res;
            }
        }
    })
}
function main(){
    hook_js();
}
setImmediate(main);
```

传入的随机字符串和字符串在**Native**层加密后的结果如下

```
input:==> meUx3DppB%Gj]-2J
res:==> LCllTadbMHYZ0kNnDitri5==
```

- 随机字符串:meUx3DppB%Gj]-2J
- 加密后的结果: LCllTadbMHYZ0kNnDitri5==

![image-20231101145832850](https://bbs.kanxue.com/upload/attach/202311/940967_VWHACV5MC76M9PA.png)

我们进入`Native`层查看加密后的结果

## Native层

使用`unzip`命令解压`apk`获取`so`文件

![image-20231101144703961](https://bbs.kanxue.com/upload/attach/202311/940967_R6NDGMZ7ZN5ZV9D.png)

使用`Ida`打开`so`文件，在`Export`表输入`JNI`查看加密函数的位置

![image-20231101150231637](https://bbs.kanxue.com/upload/attach/202311/940967_DR79857G5B5DP2C.png)

使用快捷键`Y`和`N`修改传入的参数名称，以便我们后续分析

![image-20231101150546220](https://bbs.kanxue.com/upload/attach/202311/940967_PMG3RMQRPWCT4F9.png)

返回值是`v13`

我们从下往上回溯来到`sub_EE38(v9, v10, v11)`函数

使用`Frida`对`sub_EE38`函数进行`hook`，目的是为了查看传入的参数和传出的参数

脚本如下
