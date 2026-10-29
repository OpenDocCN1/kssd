
# [原创]零基础算法还原01以及使用python和JS还原C++部分细节

## 导读

你能从本文中学到apk中的so层简单算法还原以及使用js和python还原C代码的部分细节

题目链接

链接：<mark class="encrypted">1c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4d9r3u0C8P5V1&6K9M7f1W2c8h3h3Z5$3M7Y4N6p5M7q4y4k6N6q4)9#2k6X3q4c8</mark>

提取码：1234

## 题目一

使用`jadx` 打开`algorithmbase_10.apk`

![image-20231101144427292](upload/attach/202311/940967_8ZVX7SRD3KKT7KX.png)

### JAVA层

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

* 随机字符串:meUx3DppB%Gj]-2J

* 加密后的结果: LCllTadbMHYZ0kNnDitri5==

![image-20231101145832850](upload/attach/202311/940967_VWHACV5MC76M9PA.png)

我们进入`Native`层查看加密后的结果

### Native层

使用`unzip`命令解压`apk`获取`so`文件

`unzip algorithmbase_10.apk -d fileso10`

![image-20231101144703961](upload/attach/202311/940967_R6NDGMZ7ZN5ZV9D.png)

使用`Ida`打开`so`文件，在`Export`表输入`JNI`查看加密函数的位置

![image-20231101150231637](upload/attach/202311/940967_DR79857G5B5DP2C.png)

使用快捷键`Y`和`N`修改传入的参数名称，以便我们方便我们后续分析

![image-20231101150546220](upload/attach/202311/940967_PMG3RMQRPWCT4F9.png)

返回值是**v13**

我们从下往上回溯来到 `sub_EE38(v9, v10, v11)` 函数

使用`Frida` 对`sub_EE38` 函数进行`hook`，目的是为了查看传入的参数和传出的参数

脚本如下

```javascript
function hook_js(){
Java.perform(function(){
var m_randomAscii = Java.use("com.kanxue.algorithmbase.MainActivity") //获取MainActivity类
if(m_randomAscii!=undefined){
console.log("开始Hook");
m_randomAscii.encodeFromJni_11.implementation = function(input){ //找到encodeFromJni_11方法，并拦截调用
var res = this.encodeFromJni_11(input); //调用原始方法
console.log("input:==>"+input); //打印输入参数
console.log("res:==>"+res); //打印返回结果
return res; //返回结果
}
}
})
}
function hook_native(){
Java.perform(function(){
//找到基地址
var base_address = Module.getBaseAddress("libnative-lib.so") //获取libnative-lib.so的基地址
var sub_EE38 = base_address.add(0xEE38); //计算函数偏移地址
//开启拦截器
Interceptor.attach(sub_EE38,{
//进入函数
onEnter:function(args){
this.arg0 = args[0]; //保存第一个参数
console.log("======>onENter<==========");
console.log("第一个参数未处理前===>"+args[0].readCString()); //打印第一个参数内容
console.log("第二个参数未处理前===>"+args[1].readCString()); //打印第二个参数内容
console.log("第三个参数未处理前===>"+args[2]); //打印第三个参数内容
},
onLeave:function(nresult){
console.log("======>onLeave<==========");
console.log("第一个参数处理后======>"+this.arg0.readCString()); //打印处理后的第一个参数
}
})
})
}
function main(){
hook_js();
hook_native();
}
setImmediate(main);
```

点击app按钮，Hook的代码如下所示

![image-20231101151718613](upload/attach/202311/940967_K7ZVX5SVAT8M3GU.png)

我们获取到如下参数

| sub_EE38(_BYTE *a1, __int64 a2, int a3) | 第一个参数 | 第二个参数 | 第三个参数 |
| --------------------------------------- | ------------------------ | ---------------- | ---------- |
| 未处理前 | "" | meUx3DppB%Gj]-2J | 0x10 |
| 处理后后 | LCllTadbMHYZ0kNnDitri5== | | |

```
第一个参数未处理前===>
第二个参数未处理前===>meUx3DppB%Gj]-2J
第三个参数未处理前===>0x10
======>onLeave<==========
第一个参数处理后======>LCllTadbMHYZ0kNnDitri5==
input:==>meUx3DppB%Gj]-2J
res:==>LCllTadbMHYZ0kNnDitri5==
```

也就是说 `sub_EE38` 就是我们找的加密call

我们不妨进入`sub_EE38` 查看加密细节

```c++
__int64 __fastcall sub_EE38(_BYTE *a1, __int64 a2, int a3)
{
__int64 v3; // x9，循环计数器
_BYTE *v4; // x12，用于存储结果的指针
unsigned __int8 *v5; // x10，用于指向输入数据的指针
unsigned __int64 v6; // x13，临时变量
_BYTE *v7; // x10，下一个结果的存储位置
__int64 v8; // x11，临时变量
char v9; // w8，临时变量
__int64 v10; // x9，临时变量
__int64 result; // x0，函数返回值
if (a3 - 2 < 1) // 当数据长度小于等于2时
{
LODWORD(v3) = 0;
v7 = a1;
if (a3 <= 0)
goto LABEL_11;
}
else
{
v3 = 0LL;
v4 = a1;
do
{
v5 = (unsigned __int8 *)(a2 + v3); // 从输入数据中取出三个字节进行处理
v6 = *(unsigned __int8 *)(a2 + v3);
v3 += 3LL;
*v4 = aAyzabfghz0cmbd[v6 >> 2]; // 取出第一个字节的前6位对应的字符
v4[1] = aAyzabfghz0cmbd[(16 * (unsigned int)*v5) & 0x30LL | ((unsigned __int64)v5[1] >> 4)]; // 取出第二个字节的前4位和第一个字节的后2位所对应的字符
v4[2] = aAyzabfghz0cmbd[(4 * (unsigned int)v5[1]) & 0x3CLL | ((unsigned __int64)v5[2] >> 6)]; // 取出第三个字节的前2位和第二个字节的后4位所对应的字符
LOBYTE(v6) = aAyzabfghz0cmbd[v5[2] & 0x3F]; // 取出第三个字节的后6位对应的字符
v7 = v4 + 4; // 指向下一个结果的存储位置
v4[3] = v6; // 存储刚才取出的字符
v4 += 4; // 指向下一个处理位置
}
while (v3 < a3 - 2); // 处理到倒数第三个字节为止
if ((int)v3 >= a3) // 如果处理到倒数第二个字节或最后一个字节
goto LABEL_11;
}
*v7 = aAyzabfghz0cmbd[(unsigned __int64)*(unsigned __int8 *)(a2 + (unsigned int)v3) >> 2]; // 取出最后一个字节的前6位对应的字符
v8 = (16 * (unsigned int)*(unsigned __int8 *)(a2 + (unsigned int)v3)) & 0x30LL; // 取出最后一个字节的前4位
if ((_DWORD)v3 == a3 - 1) // 如果只有最后一个字节
{
v7[1] = aAyzabfghz0cmbd[v8]; // 存储最后一个字节的前4位对应的字符
v9 = 61; // 存储'='字符
}
else
{
v10 = a2 + (unsigned int)v3; // 最后两个字节的指针
v7[1] = aAyzabfghz0cmbd[v8 | ((unsigned __int64)*(unsigned __int8 *)(v10 + 1) >> 4)]; // 存储最后一个字节的前4位和倒数第二个字节的后2位对应的字符
v9 = aAyzabfghz0cmbd[(4 * (unsigned int)*(unsigned __int8 *)(v10 + 1)) & 0x3CLL]; // 存储倒数第二个字节的后4位对应的字符
}
v7[2] = v9; // 存储倒数第二个字节的后4位或'='字符
v7[3] = 61; // 存储'='字符
v7 += 4; // 指向下一个结果的存储位置
LABEL_11:
result = (unsigned int)((_DWORD)v7 - (_DWORD)a1 + 1); // 计算结果的长度
*v7 = 0; // 结果字符串结尾添加NULL字符
return result; // 返回结果长度
}
```

根据call，还原加密算法

用C代码还原加密算法如下

```c++
// 01.cpp : 此文件包含 "main" 函数。程序执行将在此处开始并结束。
//
#include <iostream>
#include <memory>
#include <Windows.h>
#include <minwindef.h>
#include <rpcndr.h>
using namespace std;
unsigned char* sub_EE38(char* strResult, long long inPut, signed int nCnt)
{
long long v3 = 0;
unsigned char* v4 = new unsigned char[100];
unsigned char* v5 = nullptr;
unsigned long long v6 = 0;
char aAyzabfghz0cmbd[] = "AYZabFGHz0cmBdefghijklCDE1KLMNTU56789+/VWXnopq23IJrstuvwOPQRSxy4";
long long result = 0;
unsigned long long v8 = 0;
char v9;
unsigned char* v7;
long long v10;
//记录v4初始
unsigned char* vCount = v4;
do
{
*v4 = 0;
v5 = (unsigned char*)(inPut + v3);
v6 = *(unsigned char*)(inPut + v3);
v3 += 3;
*v4 = aAyzabfghz0cmbd[v6 >> 2];
v4[1] = aAyzabfghz0cmbd[(16 * (unsigned int)*v5) & 0x30 | ((unsigned long long)v5[1] >> 4)];
v4[2] = aAyzabfghz0cmbd[(4 * (unsigned int)v5[1]) & 0x3C | ((unsigned long long)v5[2] >> 6)];
unsigned char lowByte = aAyzabfghz0cmbd[v5[2] & 0x3F];
v6 = (v6 & ~(0xFFull)) | lowByte;
v7 = v4 + 4;
v4[3] = v6;
v4 += 4;
} while (v3 < nCnt - 2);
if ((int)v3 >= nCnt)
goto LABEL_11;
*v7 = aAyzabfghz0cmbd[(unsigned long long) * (unsigned char*)(inPut + (unsigned int)v3) >> 2];
v8 = (16 * (unsigned int)*(unsigned __int8*)(inPut + (unsigned int)v3)) & 0x30LL;
if (v3 == nCnt - 1)
{
v7[1] = aAyzabfghz0cmbd[v8];
v9 = 61;
}
else
{
v10 = inPut + (unsigned int)v3;
v7[1] = aAyzabfghz0cmbd[v8 | ((unsigned long long) * (unsigned char*)(v10 + 1) >> 4)];
v9 = aAyzabfghz0cmbd[(4 * (unsigned int)*(unsigned char*)(v10 + 1)) & 0x3CLL];
}
v7[2] = v9;
v7[3] = 61;
v7 += 4;
std::cout << (unsigned char*)vCount;
return (unsigned char*)vCount;
LABEL_11:
result = (unsigned int)((DWORD)v7 - (DWORD)strResult + 1);
*v7 = 0;
delete[] v4;
std::cout << result;
return (unsigned char*)result;
}
int main()
{
char str[] = "";
unsigned char* str2= sub_EE38(str, (__int64)"meUx3DppB%Gj]-2J", 0x10);
std::cout << str2;
}
```

![image-20231101171459516](upload/attach/202311/940967_AH2AS8MHV8JXVS8.png)

还原得到加密结果

```
LCllTadbMHYZ0kNnDitri5==
```

使用JS还原加密算法如下
