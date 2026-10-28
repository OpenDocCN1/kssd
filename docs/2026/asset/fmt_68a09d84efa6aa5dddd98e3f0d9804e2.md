
# [原创]启明X辰样本分析报告

## 写在前面

这是在面试启明X辰时，公司要求分析的 CreakMe，这里做一份记录。

## 样本分析报告

| 分析报告 | 分析环境 |
| :------- | :------------ |
| 分析人 | 刘XX |
| 时间 | 2023年2月28日 |
| 平台 | x86 模拟器 |

### 样本信息

| 样本 | 分析环境 |
| :------- | :--------------------------------------- |
| 样本名称 | `20230221.demo.apk` |
| MD5值 | 2A006D76B461ECE94596D46B3C9B3072 |
| SHA1值 | CF7810FA84DCCFA7F4F21847188E133A4C80CBA7 |
| CRC32 | 6D81EAF7 |

### 分析目标

1. 分析密码生成(`generate1`)过程以及check校验过程。
2. 分析密码生成(`generate2`)过程以及check校验过程。
3. 分析密码生成(`generate3`)过程以及check校验过程。
4. 总结在分析过程中哪些难题, 最后通过什么方式解决的。

### 信息整理

整理一下各个按钮的 id 与对应的linstener

| text | id | button | linstener |
| --------- | ---------- | ---------------- | --------- |
| `generate1` | `btn_login` | `this.s = button` | `g` |
| `GENERATE2` | `btn_gen2` | `this.t = button2` | `e` |
| `generate3` | `btn_gen3` | `this.u = button3` | `f` |
| `check1` | `btn_check` | `this.v = button4` | `d` |
| `check2` | `btn_check2` | `this.w = button5` | `b` |
| `check3` | `btn_check3` | `this.x = button6` | `c` |

## 按钮 generate1

1. 分析 linstener `g`，其主要就是第16行的内容的`MainActivity.this.G`方法和`com.test.pac.demo.a.a.a`方法

```java
class g implements View.OnClickListener {
g() {
}
@Override // android.view.View.OnClickListener
public void onClick(View view) {
String obj = MainActivity.this.y.getText().toString();
MainActivity.this.z.getText().toString();
if (obj.length() != 16) {
Toast.makeText(MainActivity.this.getApplicationContext(), "用户名长度必须为16字节", 0).show();
return;
}
MainActivity.this.v.setEnabled(true);
MainActivity.this.w.setEnabled(false);
MainActivity.this.x.setEnabled(false);
MainActivity.this.z.setText(com.test.pac.demo.a.a.a(MainActivity.this.G(obj.getBytes())));
}
}
```

2. 分析方法`G`

```java
public byte[] G(byte[] bArr) {
try {
byte[] bArr2 = new byte[16];
// 随机生成16个字节
new Random().nextBytes(bArr2);
// 实例化一个加密对象
Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
// 传入密钥和随机数
cipher.init(1, new SecretKeySpec("6SvMO4msTk1OqA8n".getBytes(), "AES"), new IvParameterSpec(bArr2));
// 传入明文加密并得到返回的密文
byte[] doFinal = cipher.doFinal(bArr);
// AES 加密后的数据长度是不变的，这里其实就是创建了一个长度32的字节数组
byte[] bArr3 = new byte[doFinal.length + 16];
// 从最开始申请的随机数的下标8位开始拷贝，拷贝8个长度，放到 barr3 的起始位置
System.arraycopy(bArr2, 8, bArr3, 0, 8);
// 把 AES 加密后的全部数据拷贝到 barr3，从barr3的下标8开始，其实就是接着上边结束位置
System.arraycopy(doFinal, 0, bArr3, 8, doFinal.length);
// 从最开始申请的随机数的下标0位开始拷贝，拷贝8个长度，从barr3的下标 doFinal.length+8 开始，其实就是接着上边结束位置
System.arraycopy(bArr2, 0, bArr3, doFinal.length + 8, 8);
// 小结一下，bArr3 的组成如下所示：
// bArr3 = bArr2[8-15] + aes(明文) + bArr2[0-7]
return bArr3;
} catch (Exception e2) {
e2.printStackTrace();
return null;
}
}
```

3. 分析方法`com.test.pac.demo.a.a.a()`，这个方法较为简单，就是取绝对值后转成16进制，然后再拼接成新的字符串

```java
public static String a(byte[] bArr) {
StringBuffer stringBuffer = new StringBuffer();
for (byte b : bArr) {
// 取绝对值后转成16进制
String hexString = Integer.toHexString(b & 255);
if (hexString.length() == 1) {
hexString = '0' + hexString;
}
stringBuffer.append(hexString.toUpperCase());
}
return stringBuffer.toString();
}
```

4. 总结一下，`generate1`按钮绑定 linstener `g()`，随机生成16个字节，将该16个字节 AES 加密，随后随机数再与密文重新组合一下：`bArr3 = bArr2[8-15] + aes(明文) + bArr2[0-7]`，最后将每个字节取绝对值并转为16进制字符，输出到屏幕。

## 按钮 check1

1. 分析 linstener `d`，可以轻易判断出 `obj` 为 username，`obj2` 为 password，需要重点分析的就是第15行的内容的`com.test.pac.demo.a.a.b`方法和`MainActivity.this.D`方法

```java
class d implements View.OnClickListener {
d() {
}
@Override // android.view.View.OnClickListener
public void onClick(View view) {
Context applicationContext;
String str;
String obj = MainActivity.this.y.getText().toString(); // username
String obj2 = MainActivity.this.z.getText().toString(); // password
if (obj.length() != 16) {
Toast.makeText(MainActivity.this.getApplicationContext(), "用户名长度必须为16字节", 0).show();
return;
}
if (MainActivity.this.D(com.test.pac.demo.a.a.b(obj2), obj.getBytes()) > 0) {
applicationContext = MainActivity.this.getApplicationContext();
str = "verify success!";
} else {
applicationContext = MainActivity.this.getApplicationContext();
str = "verify failed!";
}
Toast.makeText(applicationContext, str, 0).show();
}
}
```

2. 分析方法`com.test.pac.demo.a.a.b`

这个方法起初看起来有点复杂，一直以为是个加密的方法，但仔细观察，这其实就是个十六进制字符串转十进制的方法，将传入的十六进制字符串转换成了对应的十进制字节数组

```java
public static byte[] b(String str) {
if (str.length() < 1) {
return null;
}
byte[] bArr = new byte[str.length() / 2];
for (int i = 0; i < str.length() / 2; i++) {
int i2 = i * 2; // i2 = 0 2 4 6 8 10 ...
int i3 = i2 + 1; // i3 = 1 3 5 7 9 11 ...
// 一个16位的十六进制数由两个字符表示，高位字符转成十进制后乘以16 + 低位字符转成十进制
bArr[i] = (byte) ((Integer.parseInt(str.substring(i2, i3), 16) * 16) + Integer.parseInt(str.substring(i3, i2 + 2), 16));
}
return bArr;
}
```

3. 分析方法`MainActivity.this.D`

方法`D`由两个方法组成，分别是方法`F`和方法`E`，而方法`D`只是简单的判断方法`E`的返回值真假

```java
public int D(byte[] bArr, byte[] bArr2) {
return E(F(bArr), bArr2) ? 1 : 0;
}
```

4. 分析方法`F`

根据之前分析过的方法`G`，其实可以直接判断这显然是方法`G`的逆运算，也就是AES的解密方法，该方法返回了解密后的明文字节数组

```java
private byte[] F(byte[] bArr) {
try {
byte[] bArr2 = new byte[16];
// 把密文的0-7位拷贝到bArr2的8-15位
System.arraycopy(bArr, 0, bArr2, 8, 8);
// 把密文的末端8位拷贝到bArr2的0-7位
System.arraycopy(bArr, bArr.length - 8, bArr2, 0, 8);
// 实例化一个加密对象
Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
// 设置模式为解密，同时传入密钥和随机数的密文
cipher.init(2, new SecretKeySpec("6SvMO4msTk1OqA8n".getBytes(), "AES"), new IvParameterSpec(bArr2));
// 申请16个字节的空间
byte[] bArr3 = new byte[bArr.length - 16];
// 密文的下标8开始拷贝，拷贝16个，也就是将密文的[7-23]位拷贝到bArr3
System.arraycopy(bArr, 8, bArr3, 0, bArr.length - 16);
// 返回解密后的明文
return cipher.doFinal(bArr3);
} catch (Exception e2) {
e2.printStackTrace();
return null;
}
}
```

5. 分析方法`E`，其实都可以直接猜出来，`E`方法一定是在比较解密出来的明文和从控件获得的password

```java
private boolean E(byte[] bArr, byte[] bArr2) {
if (bArr.length == 0 || bArr2.length == 0 || bArr.length != bArr2.length) {
return false;
}
for (int i = 0; i < bArr.length && i < bArr2.length; i++) {
if (bArr[i] != bArr2[i]) {
System.out.println("different");
return false;
}
}
return true;
}
```

6. 总结一下，`check1`按钮完全就是`generate1`的逆向操作，只要是`generate1`设置的 password，`check1`是必然成功的。

## 按钮 generate2

按钮 `generate2` 的分析较为困难，主要问题在于 OpenSSL 库函数较为生疏，而OpenSSL的所以这里先补充一下本次分析中遇到的数据结构和函数知识。

### OpenSSL常见数据结构与函数

| 数据结构与函数 | 功能说明 |
| -------------- | ------------------------------------------------------------ |
| `HMAC_CTX_init` | 初始化一个 `HMAC_CTX` |
| `HMAC_CTX` | 是一个上下文，保存状态数据和中间计算 |
| `EVP_sha256` | 返回 sha256 的 `EVP_MD` |
| `EVP_MD` | 用来存放摘要算法信息以及各种计算函数。 |
| `HMAC_Init_ex` | 初始化HAMC_CTX上下文结构，`key`为秘钥，`len`为秘钥长度，`md`为计算hash |
| `HMAC_Update` | 向HMAC上下文输入字节流 |
| `HMAC_Final` | 生成最终的HMAC串，成功时`len`更新为HMAC的长度 |

### 分析函数 `MainActivity_M1`

这个函数较为简单，通过函数`sub_46660`向`dest`中写入数据，回传到屏幕作为密码

```cpp
jbyteArray __cdecl Java_com_test_pac_demo_MainActivity_M1(JNIEnv *env, jobject a2, BYTE *bytes_)
{
jbyteArray v4; // esi
int v6; // [esp-14h] [ebp-1030h]
unsigned int user_name_length; // [esp+0h] [ebp-101Ch]
jbyte *byte_user_name; // [esp+4h] [ebp-1018h]
char dest[4096]; // [esp+8h] [ebp-1014h] BYREF
unsigned int v10; // [esp+1008h] [ebp-14h]
v10 = __readgsdword(0x14u);
user_name_length = (*env)->GetArrayLength(env, bytes_);
byte_user_name = (*env)->GetByteArrayElements(env, bytes_, 0);
memset(dest, 0, sizeof(dest));
v6 = sub_46660((int)byte_user_name, user_name_length, dest);
v4 = (*env)->NewByteArray(env, v6);
(*env)->SetByteArrayRegion(env, v4, 0, v6, dest);
(*env)->ReleaseByteArrayElements(env, bytes_, byte_user_name, 0);
return v4;
}
```

### 分析函数 `sub_46660`

这个函数就是 `generate2` 按钮的主体逻辑了，首先将该函数分为四个部分依次分析：

- `hmac_ctx_hash`（函数一）
- `evp_encrypt`（函数二）
- 二次加密 `encrypt_data`（函数三）
- `encrypt_4`（函数四）
