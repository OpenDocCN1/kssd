
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
function sub_EE38(str,input,nCnt){
var input_bytes = stringToUint8Array(input);
var aAyzabfghz0cmbd = "AYZabFGHz0cmBdefghijklCDE1KLMNTU56789+/VWXnopq23IJrstuvwOPQRSxy4";
var result =0;
var v4 ="";
var v7=0;
var v3 =0;
var v6 =0
var v8 =0
var v9 = ""
while (v3 <nCnt-2){
var v5 = input_bytes[v3];
var v5_1 = input_bytes[v3+1]
var v5_2 = input_bytes[v3+2]
v6 = input_bytes[v3];
v3 +=3;
v4 += aAyzabfghz0cmbd.charAt(v6 >>2)
v4 += aAyzabfghz0cmbd.charAt(((16*v5)& 0x30) | v5_1 >>4);
v4 += aAyzabfghz0cmbd.charAt((4*v5_1)&0x3C |v5_2 >>6 );
v6 = aAyzabfghz0cmbd[v5_2 & 0x3F]
v4 += v6;
}
v4 +=aAyzabfghz0cmbd[(input_bytes[v3]) >> 2];
v8 = 16 * input_bytes[v3] & 0x30
if (v3 >= nCnt -1){
v4 += aAyzabfghz0cmbd[v8]
v9 = "="
}else{
v4 += aAyzabfghz0cmbd[v8 |(v4[v3+1] >>4)]
v9 = aAyzabfghz0cmbd[(4*v4[v3+1] & 0x3C)]
}
v4 += v9
v4 += "="
return v4
}
console.log(sub_EE38("","meUx3DppB%Gj]-2J",0x10))
```

![image-20231101171616527](upload/attach/202311/940967_E3XY59V7BJKX8U2.png)

使用Python还原算法如下

```python
def sub_EE38(strResult,inPut,nCnt):
v3 =0
v4 = bytes(inPut, "utf-8")
aAyzabfghz0cmbd ="AYZabFGHz0cmBdefghijklCDE1KLMNTU56789+/VWXnopq23IJrstuvwOPQRSxy4"
v6=0
result =0
v8=0
output = ""
v7 =0
while True:
v5= v4[v3]
v5_1 = v4[v3+1]
v5_2 = v4[v3+2]
v6 =v4[v3]
v3 +=3
output += aAyzabfghz0cmbd[v6>>2]
output+= aAyzabfghz0cmbd[(16*v5) & 0x30 | (v5_1 >>4)]
output+= aAyzabfghz0cmbd[(4*v5_1)&0x3C | (v5_2>>6)]
v6 = aAyzabfghz0cmbd[v5_2&0x3F]
output +=v6
v7 +=4
if v3>=nCnt-2:
break
output += aAyzabfghz0cmbd[v4[v3] >>2]
v8 = 16 * v4[v3] & 0x30
if(v3 >=nCnt -1):
output += aAyzabfghz0cmbd[v8]
v9 = '='
else:
output += aAyzabfghz0cmbd[v8 | (v4[v3+1]>>4)]
v9 = aAyzabfghz0cmbd[(4*v4[v3+1]&0x3C)]
output += v9
output += '='
return output
# Press the green button in the gutter to run the script.
if __name__ == '__main__':
print(sub_EE38("","meUx3DppB%Gj]-2J",0x10))
```

![image-20231101171651251](upload/attach/202311/940967_B8SY26TNVSGCQ5U.png)

# 题目二

`algorithmbase_12.apk`

![image-20231104100416930](upload/attach/202311/940967_Z3H69ZZRH6ZUECD.png)

使用`apkInfo`打开发现是32位程序

### JAVA层

![image-20231104100119863](upload/attach/202311/940967_NQFRZ58DXDFBYE4.png)

分析如图，先生成随机字符串（我们可以通过`Hook` `encodeFromJni_12`函数来查看生成的随机字符串的值），然后把生成的随机字符串丢到`encodeFromJni_12`里面处理

我们先Hook一下`encodeFromJni_12`函数，查看加密前的字符串和加密后的字符串

```javascript
// 定义一个名为hook_js的函数
function hook_js() {
// 使用Java.perform方法执行函数体
Java.perform(function() {
// 定义变量enrandomcode并使用Java.use方法获取com.kanxue.algorithmbase.MainActivity类
var enrandomcode = Java.use("com.kanxue.algorithmbase.MainActivity")
// 如果enrandomcode已定义
if(enrandomcode != undefined) {
// 打印开始JAVA层Hook
console.log("开始JAVA层Hook");
// 重写encodeFromJni_12方法的实现
enrandomcode.encodeFromJni_12.implementation = function(intput) {
// 调用原始的encodeFromJni_12方法，并将结果赋值给变量res
var res = this.encodeFromJni_12(intput);
// 打印传入参数的值
console.log("传入参数是====>" + intput);
// 打印加密后的参数的值
console.log("加密后的参数是====>" + res);
// 返回加密后的结果
return res;
}
}
})
}
```

结果如下

![image-20231104101314842](upload/attach/202311/940967_CY7PCKEA8CNVT4E.png)

```text
传入参数是====>xVjHx-D&nji8i*rBZ)j
加密后的参数是====>`!^fZFrxI^q~[+\~p<yT#p~F==
```

加密的结果看起来很像是`Base64`，我们用**64**编码后的结果对比一下

![image-20231104101716116](upload/attach/202311/940967_6EJYCURRDNFSYUX.png)

不是64编码，但是结果很像。猜测可能是换了码表或并非完全的`base64`加密

### so

打开`IDA`，来到加密函数`Java_com_kanxue_algorithmbase_MainActivity_encodeFromJni_112(int a1, int a2, int a3, char *a4)`里面进行分析

从下往上回溯，定位到`sub_8B04`函数

```c++
// 定义一个名为sub_8B04的函数，参数为a1、a2和a3，返回值为_BYTE类型的指针
_BYTE *__fastcall sub_8B04(int a1, int a2, int a3)
{
int v3; // r12
int v6; // r6
int v7; // r5
_BYTE *v8; // r3
int v9; // r2
unsigned int v10; // r0
char v11; // r4
int v12; // r6
char v13; // r1
// 计算v3的值
v3 = a3 - 2;
v6 = 0;
v7 = 0;
// 进入while循环，直到v7 >= v3
while ( 1 )
{
// 获取a1+v6处的_BYTE类型指针
v8 = (_BYTE *)(a1 + v6);
if ( v7 >= v3 )
break;
// 计算v9的值
v9 = a2 + v7;
// v6加上4
v6 += 4;
// 将aAyzpq23ijrtffg[*(unsigned __int8 *)(a2 + v7) >> 2]的值赋给v8指向的地址
*v8 = aAyzpq23ijrtffg[*(unsigned __int8 *)(a2 + v7) >> 2];
// 计算v10的值
v10 = *(unsigned __int8 *)(a2 + v7 + 1);
// 获取a2+v7处的值
v11 = *(_BYTE *)(a2 + v7);
// v7加上3
v7 += 3;
// 将aAyzpq23ijrtffg[(v10 >> 4) & 0xFFFFFFCF | (16 * (v11 & 3))]的值赋给v8指向的地址+1处
v8[1] = aAyzpq23ijrtffg[(v10 >> 4) & 0xFFFFFFCF | (16 * (v11 & 3))];
// 将aAyzpq23ijrtffg[(*(unsigned __int8 *)(v9 + 2) >> 6) & 0xFFFFFFC3 | (4 * (*(_BYTE *)(v9 + 1) & 0xF))]的值赋给v8指向的地址+2处
v8[2] = aAyzpq23ijrtffg[(*(unsigned __int8 *)(v9 + 2) >> 6) & 0xFFFFFFC3 | (4 * (*(_BYTE *)(v9 + 1) & 0xF))];
// 将*(_BYTE *)(v9 + 2) & 0x3F的值赋给v8指向的地址+3处
v8[3] = aAyzpq23ijrtffg[*(_BYTE *)(v9 + 2) & 0x3F];
}
// 如果v7 < a3，执行以下代码
if ( v7 < a3 )
{
// 将aAyzpq23ijrtffg[*(unsigned __int8 *)(a2 + v7) >> 2]的值赋给v8指向的地址
*v8 = aAyzpq23ijrtffg[*(unsigned __int8 *)(a2 + v7) >> 2];
// 计算v12的值
v12 = (16 * *(unsigned __int8 *)(a2 + v7)) & 0x30;
// 如果a3 - 1 == v7，执行以下代码
if ( a3 - 1 == v7 )
{
v13 = 61;
// 将aAyzpq23ijrtffg[v12]的值赋给v8指向的地址+1处
v8[1] = aAyzpq23ijrtffg[v12];
}
else
{
// 将aAyzpq23ijrtffg[v12 | (*(unsigned __int8 *)(a2 + v7 + 1) >> 4)]的值赋给v8指向的地址+1处
v8[1] = aAyzpq23ijrtffg[v12 | (*(unsigned __int8 *)(a2 + v7 + 1) >> 4)];
// 将aAyzpq23ijrtffg[4 * (*(_BYTE *)(a2 + v7 + 1) & 0xF)]的值赋给v13
v13 = aAyzpq23ijrtffg[4 * (*(_BYTE *)(a2 + v7 + 1) & 0xF)];
}
// 将61的值赋给v8指向的地址+3处
v8[3] = 61;
// 将v13的值赋给v8指向的地址+2处
v8[2] = v13;
// v8加上4
v8 += 4;
}
// 将0赋给v8指向的地址
*v8 = 0;
// 返回&v8[-a1 + 1]的值
return &v8[-a1 + 1];
}
```

进行hook

这里有个坑，就是在使用拦截器对`sub_8B04`位置进行定位，如果使用`var sub_8B04 = base_address.add(0x8B04);`拦截出错就加一

这样才能进入函数里面

```js
var sub_8B04 = base_address.add(0x8B04+1);
```

hook代码如下

```js
function hook_native(){
Java.perform(function(){
var base_address = Module.getBaseAddress("libnative-lib.so");
if(base_address !=undefined){
console.log("开始native层hook");
//定位拦截位置
var sub_8B04 = base_address.add(0x8B04+1);
console.log("sub_8b04:",sub_8B04)
if(sub_8B04!=undefined){
console.log("进入native层0x8B04");
}else{
console.log("拦截器加载失败");
}
//使用拦截器
Interceptor.attach(sub_8B04,{
//开始进入函数,传入了三个参数依次打印
onEnter:function(args){
//保存参数
this.arg0 = args[0];
this.arg1 = args[1];
this.arg2 = args[2];
console.log("========>ENTER<========");
console.log("sub_8B04未初始化时第一个参数是===>"+this.arg0.readCString());
console.log("sub_8B04未初始化时第二个参数是===>"+this.arg1.readCString());
console.log("sub_8B04未初始化时第三个参数是===>"+this.arg2.readCString());
},
onLeave:function(result){
console.log("========>LEAVE<========");
console.log("sub_8B04已经初始化时第一个参数是===>"+this.arg0.readCString());
}
}
)
}
})
}
```

hook显示的结果如下

```text
sub_8B04未初始化时第一个参数是===>
sub_8B04未初始化时第二个参数是===>xVjHx-D&nji8i*rBZ)j
sub_8B04未初始化时第三个参数是===>0x13(传入参数的字符串大小)
sub_8B04已经初始化时第一个参数是===>`!^fZFrxI^q~[+\~p<yT#p~F==
```

使用`IDA`对`libnative-lib.so`进行[动态调试](https://bbs.kanxue.com/thread-279335-1.htm#msg_header_h1_3)的时候会发现`aAyzpq23ijrtffg`字符串和我们之前记录的字符串（`aAyzpq23ijrtffg DCB "AYZpq23IJrTFfghijklCDE1KLMmBdestU5678GHz0cuvwabN9+/VWXnoOPQRSxy4",0`）并不相同，结合前面的思考，换了码表可能性更大

![image-20231104105122522](upload/attach/202311/940967_9K6WBENU82Y2TPN.png)

我们开始`hook` `aAyzpq23ijrtffg`码表（`0x1B000`），这个有个`hook`技巧：复制使用`readCString`打印字符串可能会造成数据丢失，所以我们使用`hexdump`（16进制）显示码表的内容会更精确
