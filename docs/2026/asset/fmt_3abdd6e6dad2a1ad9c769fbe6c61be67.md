
```
for (v0 = 0; true; ++v0) {
    int[] v1 = a.b;
    if (v0 >= v1.length) {
        break;
    }
    v1[v0] ^= 0xAB;
}
....
while (v4 < a.a.length) {
    try {
        v0_1 = MessageDigest.getInstance("MD5");
    } catch(NoSuchAlgorithmException v1_1) {
        v1_1.printStackTrace();
    }
    v0_1.update(a.a[v4]);
    a.a[v4] = v0_1.digest();
    ++v4;
}
```

对密钥进行`MD5`。

当`b`数组的值`<8`时，不使用密钥，`hash`加密的方式不同，`b`数组`>=8`时，相同的`hash`加密`HmacSha512`，不同密钥。

这样就直接开始爆破，8组数据，范围为4个`0-0x7f`字节。直接抄程序反编译代码去爆破。

`HmacSha512`: 将密钥和密文对应就行。

```
import java.nio.charset.StandardCharsets;
import java.math.BigInteger;
import java.security.GeneralSecurityException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class demo {
    public static final String v0 = "HmacSha512";
    public static void main(String[] args) {
        MessageDigest v0_1 = null;
        StringBuilder v1_2;
        byte[] miwen = new byte[4];
        String v6_1 = "%02x";
        try {
            for (int i=0x0; i<=0x7e; i++) {
                miwen[0] = (byte)i;
                for (int j = 0x0; j <= 0x7e; j++) {
                    miwen[1] = (byte)j;
                    for (int k = 0x0; k <= 0x7e; k++) {
                        miwen[2] = (byte)k;
                        for (int l = 0x0; l <= 0x7e; l++) {
                            miwen[3] = (byte)l;
                            try {
                                v0_1 = MessageDigest.getInstance("MD5");
                            } catch(NoSuchAlgorithmException v1_1) {
                                v1_1.printStackTrace();
                            }
                            byte[] tmp = "ALFjcgztxnUaC89v".getBytes();
                            v0_1.update(tmp);
                            byte[] miwen2 = v0_1.digest();
                            SecretKeySpec v1 = new SecretKeySpec(miwen2, v0);
                            Mac v3 = Mac.getInstance(v0);
                            v3.init(((Key)v1));
                            v3.update(miwen);
                            String v2_2;
                            String s = "78b0be39e63b6837";
                            for (v2_2 = new BigInteger(1, v3.doFinal()).toString(16); v2_2.length() < 0x20; v2_2 = "0" + v2_2) {
                            }
                            if (v2_2.substring(0,16).equals(s)) {
                                System.out.printf("%x ", i);
                                System.out.printf("%x ", j);
                                System.out.printf("%x ", k);
                                System.out.printf("%x ", l);
                                System.out.printf("%n");
                                return;
                            }
                        }
                    }
                }
            }
        } catch(InvalidKeyException | NoSuchAlgorithmException v2) {
            return;
        }
    }
}
```

无密钥`hash`: 将`hash`方法和密文对应。

```
import java.nio.charset.StandardCharsets;
import java.math.BigInteger;
import java.security.GeneralSecurityException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public class demo5 {
    public static final String v0 = "HmacSha512";
    public static void main(String[] args) {
        MessageDigest v1_11;
        byte[] miwen2;
        byte[] miwen = new byte[4];
        String v6_1 = "%02x";
        StringBuilder v1_2;
        for (int i=0x0; i<=0x7e; i++) {
            miwen[0] = (byte)i;
            for (int j = 0x0; j <= 0x7e; j++) {
                miwen[1] = (byte)j;
                for (int k = 0x0; k <= 0x7e; k++) {
                    miwen[2] = (byte)k;
                    for (int l = 0x0; l <= 0x7e; l++) {
                        miwen[3] = (byte)l;
                        String s = "f2dda5fc021fe2bf";
                        try {
                            v1_11 = MessageDigest.getInstance("SHA-384");
                            v1_11.update(miwen);
                            miwen2 = v1_11.digest();
                            v1_2 = new StringBuilder();
                            for (int v4 = 0; v4 < miwen2.length; ++v4) {
                                v1_2.append(String.format(v6_1, Byte.valueOf(miwen2[v4])));
                            }
                            String tmp = v1_2.toString().substring(0,16);
                            if (tmp.equals(s)){
                                System.out.printf("%x ", i);
                                System.out.printf("%x ", j);
                                System.out.printf("%x ", k);
                                System.out.printf("%x ", l);
                                System.out.printf("%n");
                                return;
                            }
                        } catch(NoSuchAlgorithmException v0_2) {
                            v0_2.printStackTrace();
                        }
                    }
                }
            }
        }
    }
}
```

这里我没有再去考虑大于`7F`的输入，我直接分析flag的输出：

```
if ((v10 >>> 3 & 1) == 0) {
    v6_1[v10 & 7] = new String(v8_1);
} else {
    for (v9 = 0; v9 < v8_1.length / v5; ++v9) {
        byte v11 = v8_1[v9];
        v8_1[v9] = v8_1[v8_1.length - 1 - v9];
        v8_1[v8_1.length - 1 - v9] = v11;
    }
    v6_1[v10 & 7] = new String(v8_1); // 输出顺序和加密方式有关
}
```

这里可以看到，如果使用了`HmacSha512`，需要反序。

最后爆破了半天得到flag。

```
FLAG{H@5H_15_7H3_8@PP1N355_C11P}
```

#### Studydesk

输入处理：

```
byte[] v0 = a.a;
.....
if (v6 < v5) {
    v9 = in.indexOf(v0[v6]);
    if (v9 == -1) {
        v0 = ((byte[])v7);
    } else {
        v4.append(Integer.toBinaryString((v9 & 0x3F) + v2).substring(1));
        ++v6;
        continue;
    }
}
```

输入为32字节，然后依次查询`a`类的`a`数组再输入中的位置，将位置`+32`后转化为2进制去掉首位保存字符串。

保存的范围就是`00000 - 11111`，一共是16个。

查看`a`类的`a`数组，可以编写脚本处理发现只有32种。要求的是每个数都再输入中能找到，不能重复。

之后就是2进制连起来，依次取8位，转化为`byte`数组。

密文的生成直接抄反编译代码，然后调试可以发现，是圆周率。长度是360位。

校验：

```
while (true) {
    if (v3_3 < v2_2.length()) {
        v4_1 = v3_3 + 2;
        if ((((byte)(Integer.parseInt(v2_2.substring(v3_3, v4_1), 10) & 0xFF))) != v0[v3_3 / 2]) {
            v0_1 = 0;
        } else {
            v3_3 = v4_1;
            continue;
        }
    } else {
        break;
    }
    goto label_120;
}
```

要求的是依次取圆周率字符串2位，视为10进制数和5位2进制数生成的`byte`数组一一比较，一共180次。

这里不妨算一下，`a`数组的长度为288位，转化出来的2进制长度为`288 * 5 == 1440`，密文长度是`180 * 8 == 1440`，刚好对应。
