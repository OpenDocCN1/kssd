
# [原创]ollvm算法还原案例分享

样本来自看雪3W班5月题

本题主要是还原so的算法函数，关键算法函数很好定位，主要还原方法是`frida`+`ida调试`+`ida trace`

## 1、找到对应产生结果的函数`public native byte[] e(byte[] arg1)`

```java
public class MainActivity extends AppCompatActivity {
static {
System.loadLibrary("native-lib");
}
public native byte[] d(byte[] arg1) {
}
public native byte[] e(byte[] arg1) {
}
@Override // androidx.appcompat.app.AppCompatActivity
protected void onCreate(Bundle arg3) {
super.onCreate(arg3);
this.setContentView(0x7F09001C); // layout:activity_main
TextView v3 = (TextView)this.findViewById(0x7F070061); // id:sample_text
StringBuilder v0 = new StringBuilder();
v0.append("pediy_imyang_");
v0.append(RandomStringUtils.randomAlphabetic(10));
v3.setText(ByteString.of(this.e(v0.toString().getBytes())).hex());
}
}
```

## 2、固定入参，打印参数

```js
function hook_java(){
Java.perform(function () {
//org.apache.commons.lang3.RandomStringUtils.randomAlphabetic(int): java.lang.String
var RandomStringUtils = Java.use("org.apache.commons.lang3.RandomStringUtils");
RandomStringUtils.randomAlphabetic.overload('int').implementation = function(arg){
var result = this.randomAlphabetic(arg);
// console.log("org.apache.commons.lang3.RandomStringUtils.randomAlphabetic:", result);
result = "lXcaTALmow";
return result;
}
//com.kanxue.ollvm5.MainActivity.encryt(byte[]): byte[]
var ByteString = Java.use("com.android.okhttp.okio.ByteString");
var MainActivity = Java.use("com.kanxue.ollvm5.MainActivity");
MainActivity.e.implementation = function(arg){
var result = this.e(arg);
console.log("com.kanxue.ollvm5.MainActivity arg:", ByteString.of(arg).hex());
console.log("com.kanxue.ollvm5.MainActivity result:", ByteString.of(result).hex());
return result;
}
});
}
```

得到结果：

```
com.kanxue.ollvm5.MainActivity arg: 70656469795f696d79616e675f6c58636154414c6d6f77
com.kanxue.ollvm5.MainActivity result: 797903090001a4b8b6c89eb47f4f29b44d47c7382f851ad57618f9b820c5d55298cb5f941c8c
```

## 3、ida打开对应so，找到关键函数

ida打开对应so，找到`jbyteArray __fastcall Java_com_kanxue_ollvm5_MainActivity_e(JNIEnv *env_, __int64 a2, jbyteArray ary)`函数，hook发现`sub_184EC`调用三次，其中第三次的arg1为结果函数：

```
sub_184EC(&arg0, arg1, arg2);
```

查找`sub_184EC`引用，找到函数`sub_13CE4`：

```
__int64 __fastcall sub_13CE4(__int64 result, unsigned int a2, __int64 a3, unsigned int a4, __int64 output)
```

## 4、hook `sub_184EC`, 根据结果可知参数类型, 第2个参数是input，第5个参数是ouput, 即`sub_184EC`为关键算法函数

```js
//13CE4
var addr_13CE4 = base.add(0x13CE4);
var arg13CE4_0;
var arg13CE4_1;
var arg13CE4_4;
Interceptor.attach(addr_13CE4, {
onEnter: function (args) {
//var r2 = this.context.r2;
arg13CE4_0 = args[0];
arg13CE4_1 = args[1];
arg13CE4_4 = args[4];
console.log("addr_13CE4 onEnter args0", hexdump(args[0])); //input
console.log("addr_13CE4 onEnter args1", args[1]); //inputlen
console.log("addr_13CE4 onEnter args4", hexdump(args[4]));
},
onLeave: function (retval) {
console.log("addr_13CE4 onLeave args0", hexdump(arg13CE4_0));
console.log("addr_13CE4 onLeave args1", arg13CE4_1);
console.log("addr_13CE4 onLeave args4", hexdump(arg13CE4_4));
// console.log("addr_13CE4 retval", hexdump(arg1));
}
});
```

frida写一个主动调用，trace一下

```js
function call_e() {
//主动调用函数
//com.kanxue.ollvm5.MainActivity.e(byte[]): byte[]
Java.perform(function () {
var MainActivity = Java.use("com.kanxue.ollvm5.MainActivity");
Java.choose("com.kanxue.ollvm5.MainActivity", {
onMatch: function (instance) {
//pediy_imyang_lXcaTALmow
var buffer = Java.array('byte', [0x70, 0x65, 0x64, 0x69, 0x79, 0x5f, 0x69, 0x6d, 0x79, 0x61, 0x6e, 0x67, 0x5f, 0x6c, 0x58, 0x63, 0x61, 0x54, 0x41, 0x4c, 0x6d, 0x6f, 0x77])
var result = instance.e(buffer);
var ByteString = Java.use("com.android.okhttp.okio.ByteString");
console.log(ByteString.of(result).hex())
},
onComplete: function () {
}
});
});
}
```

从trace的结果来看只看到input没看到output

再返回仔细看，尝试hook `sub_13808`试试
