
```java
private String a(byte[] arg7, Map arg8, String arg9) {

        String v0_1;

        int v0 = 0;

        byte[] v3 = arg9.getBytes();

		//得到UserAgent

        byte[] v2 = c.a(arg8) ? f.a().c().getBytes(Charset.forName("UTF-8")) : "".getBytes(Charset.forName("UTF-8"));	

        if(arg7 != null) {

            try {

                byte[] v4 = new byte[arg7.length + v2.length];	//构造byte数组v4，长度为mzip的长度加user Agent的长度

                int v1;

                for(v1 = 0; v1 < v2.length; ++v1) {

                    v4[v1] = v2[v1];

                }

                while(v0 < arg7.length) {			//拼接两个byte数组

                    v4[v2.length + v0] = arg7[v0];

                    ++v0;

                }

                v0_1 = Coded.getInstance().sign(v4, v3);	//调用sign函数，参数依次为：拼接过的byte数组，aes密钥

            }

            return v0_1;

        }

    }
```

`sign`函数内部调用了`libcoded_jni.so`中的函数`sdbyecbu37x`

`sdbyecbu37x`中调用了`libcoded.so`中的native函数`sign`

`sign`函数起初处有一个检测`TracePid`的反调试

然后是一个`switch`语句，经过分析发现，`xsign`的计算位于`case 1`中，

将参数1与参数2的前8个字节拼接，进行一次`SHA1`运算

```
	loc_6140                ; jumptable 000060F0 case 3

	ADD.W           R4, R11, #8

	MOV             R0, R4  ; size	//申请一块length+8的空间buff

	BLX             malloc	

	MOV             R1, R9

	MOV             R2, R11

	MOV             R5, R0			

	BLX             __aeabi_memcpy	//拷贝参数1至buff

	LDR.W           R0, [R10]

	MOV             R2, R4

	LDR.W           R1, [R10,#4]

	STR.W           R0, [R5,R11]

	ADD.W           R0, R5, R11

	STR             R1, [R0,#4]

	MOV             R0, R5

	MOV             R1, R8

	BLX             j_momoEnc_sha1

	MOV             R0, R5  ; ptr

	BLX             free

	MOVS            R0, #0

	B               loc_6120 ; jumptable 000060F0 cases 2,4
```

至此可得结论，`X-SIGN`的输出形式为：

```
 SHA1( (postData.getBytes+userAgent.getBytes) + secret.subString(0,8).getBytes )
```

一般而言，如果可以模拟程序的登录包，其他的包自然也可以模拟，所以分析到这里，我决定把目标切换为模拟程序登录。

抓包`HTTPS`得到

```
    code_version=2

    map_id=9864824510

    mzip=AgPs+lDzADrqaD/2zXNra9FWpFkFgiF3pwde4c2UeAQ3nfKDjwSLquaW9IqPRol/GQfy6Ap7S0Q...

    X-KV=32712cf9

    ck=AgPs+lDzAOE81bVd6TGkHR2gbX0bsZF9g/FRMmwAzvaogaOIDx/pOh7SRpeSw+mCazBCEX7AIQ...

```

可以看到，登录包中有5个参数，分别为：`code_version`，`map_id`，`mzip`，`X-KV`，`ck`

上面已经分析了`mzip`，`code_version`是一个固定值，所以现在需要分析的有`ck`和`map_id`，`X-KV`

在`X-SIGN`计算的下方，有如下代码：

```
    v0_2 = d.a().e();

    this.i.put("ck", d.a().f());

    this.i.put("code_version", d.a().g());

    this.i.put("map_id", v0_2);

    this.i.put("X-KV", d.a().h());
```

可以看到，程序通过`d.a().e()`获取`mapid`

追踪代码发现`mapid`的生成方式，一个时间戳经过计算后拼接一个随机数

```
	public String i() {

        long v4 = 100000;

        long v0 = System.currentTimeMillis() % 1000000;

        if(v0 < v4) {

            v0 += v4;

        }



        d.e = v0 + "" + ((((int)(Math.random() * 9000))) + 1000);

        return d.e;

    }
```

同理，追踪`d.a().f()`函数，发现`ck`的生成函数：`com.immomo.mmutil.a.a(v2)`

函数的参数`v2`的生成依赖于一个程序第一次启动时生成的公钥`localPublicKey`(生成算法过于复杂，在此不做展示，

读者如果想要验证本文的分析，可以在程序第一次启动时hook `com.immomo.momoenc.d`类中的函数得到这个值)

以公钥作为输入，字符串`"Iu0WKHFy"`作为密钥，进行`aes128`运算(需要注意的是，`Iu0WKHFy`只有8字节，需要在后面补八个字节零)

输出的值即为`com.immomo.mmutil.a.a`函数的参数`v2`

进入函数内部后发现，`com.immomo.mmutil.a.a`为`Base64`的`encode`函数

```
    byte[] v1 = com.immomo.mmutil.a.b(v7.a.getBytes());	//Base64.decode

    byte[] v3 = f.a().g().getBytes();	

    byte[] v5 = new byte[Coded.getInstance().computeOutputLength(v1.length, 1)];

    int v1_1 = Coded.getInstance().aesEncode(v1, v1.length, v3, v3.length, v5);

    byte[] v2 = new byte[v1_1];

    int v0_1;

    for(v0_1 = 0; v0_1 < v1_1; ++v0_1) {

        v2[v0_1] = v5[v0_1];

    }

    d.a().b(com.immomo.mmutil.a.a(v2));		//Base64.encode
```

得到`ck`的输出形式

```
    aesEncode ( Base64.decode( localPublicKey ), "Iu0WKHFy".getBytes+ new byte[8]={0} )
```

登录时`header`中还需要一个`X-KV`字段

追踪`d.a().h()`函数

得到以下代码，其中`j.b`函数为判空函数，`j.a`函数为`MD5`，参数`this.b`为`localPublicKey`

```
	public String d() {

        String v0 = j.b(this.b) ? "" : j.a(this.b).substring(0, 8);

        return v0;

                }
```

即`xkv`的的输出为

```
    MD5( localPublicKey ).substring(0,8);
```

至此，我们已经完成了整个登陆包所需的加密

需要的参数太多，各种手机信息位置信息网络信息硬件检测，具体的参数信息获取在此就不写了，参数获取不是很难，就是有些繁琐

最后稍微写一下`respone`的解密

解密跟`mzip`的加密正好相对，文章开头我们已经看到了，`respone`的格式为`0203`，想必已经有读者能够猜到解密的流程了

将`respone`进行分割：`0203` + 4字节`IV`生成数 + `Data`

`IV`=SHA1(`IV`生......

`data`=.....嗯，应该不用写了

## 结语

本来想附代码的，结果不让发...还是算了

程序获取了几百种的信息，烦人的要死，恨不得把你穿什么颜色的裤衩都上传...

至于程序的脱机，最难的其实是逃不过厂商的风控，不过，我还是逃过了一部分...

这只是程序配置相关`https`层的通讯，有关程序`tcp`层的通讯在此不做讲解...

有兴趣的可以联系我，联系我也不会告诉你的...

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-6-3 11:37

被`kanxue`编辑，原因：
