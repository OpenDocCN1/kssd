

# [原创] 从三道题目入手入门 Frida

## 写在前面

- 题目来自 2w 班 6 月习题，题目本身都没有加壳、没有混淆，主要逻辑就是输入 flag，程序本身利用 `hash` 加密后与程序已有密文进行对比。细节方面略有不同，但大体来说就是爆破，整个过程逻辑很清晰，都不难，基本上都是 `Frida` 一把梭。题目主要考察了以下知识点：
  1. `frida java hook` 与静态函数的主动调用
  2. `Frida` 遍历 `ClassLoader`，从而 `hook` 动态加载的 `dex` 的函数
  3. `frida native hook` 去反调试

# 第一题

## 程序分析

Jadx 打开程序。

![](upload/attach/202006/715334\_WGVNMNR5NGTF4BV.png)

可知程序逻辑就是将用户名和密码进行拼接，传入关键函数 `VVVVV.VVVV(this, str)`。

继续看 `VVVV` 函数。

![](upload/attach/202006/715334\_THX3UT7YPEQKNS8.png)

可得知信息：

1. 输入长度为 5 位。
2. `frida hook` 这个函数直接强制返回 `true` 就行，这个 check 就这样 pass 了，但是拿不到正确的 flag。

后续的函数继续去观察会发现，实际上就是一个 `sha1` + `salt` 的加密。

又在运行程序后发现提示只能是数字，那么思路就清晰了：爆破！

解题方法根据要求写了两种。

### 1. Frida

直接上 Frida 脚本，关键代码如下：

```javascript
Java.perform(function(){
Java.use('com.kanxue.pediy1.VVVVV').VVVV.implementation = function(listener,input){
console.log('input= ',input);
for (var i = '0'; i <= '9'; i++) {
//System.out.println(i);
for (var j = '0'; j <= '9'; j++) {
for (var k = '0'; k <= '9'; k++) {
for (var t = '0'; t <= '9'; t++) {
for (var y = '0'; y <= '9'; y++) {
var newInput = Java.use('java.lang.String').$new(i.toString()
+ j.toString()
+ k.toString()
+ t.toString()
+ y.toString())
console.log(newInput)
var result = this.VVVV(listener,newInput)
if(result == true){
console.log('flag is ',newInput)
return result;
}
}
}
}
}
}
}
})
```

最终拿到 `flag` 为 `66888`。

![](upload/attach/202006/715334\_ZQ28SFRH6KVKG8J.png)

### 2. 抠代码

将 APK 中函数直接全部 copy 到一个 Java 工程，运行一遍就行，实际上也是爆破，没啥差别。

```java
import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
public class VVVVV {
public static void main(String[] args){
for (int i = '0'; i <= '9'; i++) {
//System.out.println(i);
for (int j = '0'; j <= '9'; j++) {
for (int k = '0'; k <= '9'; k++) {
for (int t = '0'; t <= '9'; t++) {
for (int y = '0'; y <= '9'; y++) {
StringBuilder sb = new StringBuilder();
String input = sb.append((char) i).append((char)j).append((char)k)
.append((char)t).append((char)y).toString();
//System.out.println(input.length());
if(VVVV(input)){
System.out.println("flag is " + input);
return;
}
}
}
}
}
}
}
}
```

这里只展示了 `main` 函数代码，最终拿到 `flag` 为 `66888`。

![](upload/attach/202006/715334\_VH9UXQ4TNCTYCT3.png)

`check` 一下。

![](upload/attach/202006/715334\_ZGRYVWHYEW7EBVC.png)

拿到 flag！！！

# 第二题

## 程序分析

Jadx 打开程序。

![](upload/attach/202006/715334\_FVSTTNGQT39EJNN.jpg)

可以发现实际上就是动态加载 dex 并调用动态加载的 dex 中的 `VVVV` 函数。

Jadx 打开需要动态加载的 `dex`。

![](upload/attach/202006/715334\_K6FXJ4TESEAMU5Q.jpg)

其实会发现算法和第一题是一样的。

直接写 Frida 脚本，遍历 `ClassLoader`，选择正确的 `classLoader`，再对函数进行主动调用爆破得到：

```javascript
function enumerateClassLoader(){
Java.perform(function(){
Java.enumerateClassLoaders({
onMatch: function(loader){
//console.log('classLoader',classLoader.toString());
if(loader.toString().indexOf('dalvik.system.DexClassLoader')>-1){
console.log('find classLoader',loader.toString());
Java.classFactory.loader = loader;
hookVVVV()
return;
}
},onComplete: function(){
console.log('search complete!');
}
})
})
}
function hookVVVV(){
Java.perform(function(){
console.log('loader',Java.classFactory.loader);
for (var i = '0'; i <= '9'; i++) {
for (var j = '0'; j <= '9'; j++) {
for (var k = '0'; k <= '9'; k++) {
for (var t = '0'; t <= '9'; t++) {
for (var y = '0'; y <= '9'; y++) {
var newInput = Java.use('java.lang.String').$new(i.toString()
+ j.toString()
+ k.toString()
+ t.toString()
+ y.toString())
//console.log(newInput)
var result = Java.use('com.kanxue.pediy1.VVVVV').VVVV(newInput)
if(result == true){
console.log('flag is ',newInput)
return;
}
}
}
}
}
}
})
}
setImmediate(enumerateClassLoader)
```

这里需要注意的是，脚本 attach 的时间，要在第一次输入进行 check 后再将脚本 attach 上去，这样才能保证遍历 `ClassLoader` 能够得到 `DexClassLoader`，即校验函数正确的 `ClassLoader`。

最终得到爆破得到的 `flag` 为 `66999`。

![](upload/attach/202006/715334\_RKKDN4UDVSHATYJ.jpg)

但实际测试还不对，继续往回看，会发现有一个 `stringFromJNI` 的一层调用。这个函数是 `native` 函数，使用 IDA 打开。

![](upload/attach/202006/715334\_HWPYTXF5VYEA86U.jpg)

这个函数有点骚……就是将输入的数字字符串转换为 `int`，然后加了一返回了……

那么 flag 就很明显了，应该为 `66999 - 1 = 66998`。

`check` 一下。

![](upload/attach/202006/715334\_PDN2PE4RUB9D9YT.jpg)

正确！

# 第三题

## 写在前面

这一题其实就是第二题的进阶版，在第二题的基础上加了反调试，所以在这里只分析反调试部分，其余不管。

## 程序分析

这一题和第二题不同的地方在于，在 `OnCreate` 函数中调用了 `init` 函数，而这个函数是 `native` 函数。

![](upload/attach/202007/715334\_V89D479EBFPT5G2.jpg)

IDA 打开这个 `so` 文件。

![](upload/attach/202007/715334\_75NKYJFTACVAVQC.jpg)

观察 `JNI_Onload` 函数可以发现，Java 层的 `init` 函数被动态注册到 `init` 函数上去了。详细介绍的话就涉及 `JNINativeMethod` 结构体了，这里不再赘述。

接下来就简单了，跟踪到 `init` 函数。

![](upload/attach/202007/715334\_6R74JGHEQRT5UNB.jpg)

观察函数名，可以清楚地了解到，这个函数实际上就是创建了一个新线程用于检测 `frida`。跟踪这个函数，伪代码如下：

```c
void \_\_fastcall \_\_noreturn detect\_frida\_loop(void \*a1)
{
v9 = a1;
v14 = &v6;
v13 = 16LL;
v12 = 0;
v11 = 16LL;
v10 = 16LL;
v15 = \_\_memset\_chk(&v6, 0LL, 16LL, 16LL);
v6 = 2;
inet\_aton("0.0.0.0", &v8);
while ( 1 )
{
for ( i = 1; i <= 65533; ++i )
{
v5 = socket(2LL, 1LL, 0LL);
v7 = bswap32(i) >> 16;
if ( connect(v5, &v6, 16LL) != -1 )
{
v20 = &v2;
v19 = 7LL;
v18 = 0;
v17 = 7LL;
v16 = 7LL;
v21 = \_\_memset\_chk(&v2, 0LL, 7LL, 7LL);
v26 = v5;
v25 = &unk\_14A2;
v24 = -1LL;
v23 = 1LL;
v22 = 0;
v33 = v5;
v32 = &unk\_14A2;
v31 = -1LL;
v30 = 1LL;
v29 = 0;
v28 = 0LL;
v27 = 0;
sendto(v5, &unk\_14A2, 1LL, 0LL, 0LL, 0LL);
v38 = v5;
v37 = "AUTH\r\n";
v36 = -1LL;
v35 = 6LL;
v34 = 0;
v45 = v5;
v44 = "AUTH\r\n";
v43 = -1LL;
v42 = 6LL;
v41 = 0;
v40 = 0LL;
v39 = 0;
sendto(v5, "AUTH\r\n", 6LL, 0LL, 0LL, 0LL);
usleep(500LL);
v50 = v5;
v49 = &v2;
v48 = 7LL;
v47 = 6LL;
v46 = 64;
v57 = v5;
v56 = &v2;
v55 = 7LL;
v54 = 6LL;
v53 = 64;
v52 = 0LL;
v51 = 0LL;
v3 = recvfrom(v5, &v2, 6LL, 64LL, 0LL, 0LL);
if ( v3 != -1 )
{
if ( strcmp(&v2, "REJECT") )
{
\_\_android\_log\_print(4LL, "pediy", "not FOUND FRIDA SERVER");
}
else
{
v1 = getpid();
kill(v1, 9LL);
}
}
}
close(v5);
}
}
}
```

可以清楚地看到，实际上就是通过建立 socket 连接，遍历端口，检测是否有端口被占用。如果接收到 `"REJECT"`，则表明 `frida-server` 在运行，并退出程序。

好了，理论逻辑分析完毕。接下来进行反反调试，根据 `r0ysue` 老师要求，三种方法。

### 1. 二次打包

使用 `apktool` 将 `apk` 转换为 smali，然后修改调用 `init` 函数的 smali 语句。具体来说：

#### 1.1. `apktool` 命令反编译为 smali

![](upload/attach/202007/715334\_MQEW8X75W55M53F.jpg)

#### 1.2. 删除调用 `init` 函数的 smali 语句

![](upload/attach/202007/715334\_MKBD3JFRJ5E5KQV.jpg)

对应我这，也就是删除 `MainActivity.smali` 的第 397 和 398 行。

#### 1.3. 二次打包

![](upload/attach/202007/715334\_NBX4ZXP6AJAYQDK.jpg)

这样就过了反调试了。

### 2. Frida 去 hook `pthread_create` 函数

这里就不赘述了，直接几乎照抄的 `r0ysue` 大佬星球里的，链接见<a href="elink@c8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1i4K6u0W2P5Y4y4^5M7g2)9J5k6h3y4G2L8g2)9J5c8W2k6B7b7h3q4&6b7U0t1`.">这</a>（r0ysue 大佬，等会记得广告费结一下，溜了溜了），这里也贴上我抄的代码吧（这里其实还有很多种其他更简单的方法，比如 `hook` `strcmp` 函数等等。我这里的代码还有很多的改进之处，仅供参考）：

```js
function hook\_pthread\_create(){
var pt\_create\_func = Module.findExportByName(null,'pthread\_create');
var detect\_frida\_loop\_addr = null;
console.log('pt\_create\_func:',pt\_create\_func);
Interceptor.attach(pt\_create\_func,{
onEnter:function(){
if(detect\_frida\_loop\_addr == null)
{
var base\_addr = Module.getBaseAddress('libnative-lib.so');
if(base\_addr != null){
detect\_frida\_loop\_addr = base\_addr.add(0xe9c)
console.log('this.context.x2: ', detect\_frida\_loop\_addr , this.context.x2);
if(this.context.x2.compare(detect\_frida\_loop\_addr) == 0) {
hook\_anti\_frida\_replace(this.context.x2);
}
}
}
},
onLeave : function(retval){
// console.log('retval',retval);
}
})
}
function hook\_anti\_frida\_replace(addr){
console.log('replace anti\_addr :',addr);
Interceptor.replace(addr,new NativeCallback(function(a1){
console.log('replace success');
return;
},'pointer',[]));
}
```

### 3. 硬编码 so

这里我注意到，在 `detect_frida_loop` 函数的最后，有一个 `strcmp` 函数的调用。

![](upload/attach/202007/715334\_SN8KDC3WZ6EYHUB.jpg)

对应汇编中：

![](upload/attach/202007/715334\_KV4G7D3CGDVRE37.jpg)

可以看到 `loc_1220` 分支和 `loc_1210` 分支分别为没有找到 `frida_server` 和退出进程。毫无疑问我们需要的是 `loc_1220`。这里我选择将偏移为 `0x120C` 处的：

`B loc_1210` 这条汇编改为：

`B loc_1220`

从而达到无论有没有检测到 `frida_server` 都不要退出进程的效果。

![](upload/attach/202007/715334\_WMEN8RDP559W3AF.jpg)

修改之后伪代码变成了：

![](upload/attach/202007/715334\_SKSQBVRC9UBY2WH.jpg)

最终 get Flag，之后的过程就和第二题一致了，脚本都不用变。

最终拿到 `flag` 为 `99998`。

![](upload/attach/202007/715334\_P2GBJERNMSSZCDX.jpg)

## 写在最后

