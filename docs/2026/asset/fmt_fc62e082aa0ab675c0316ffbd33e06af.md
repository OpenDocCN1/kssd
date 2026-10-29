
解析如下：

```
63 00 00 00 //数据包长度

09 00 //代表了index和type，即索引号和类型，下面同理。这儿对比谷歌的protobuf即可理解

    00 //timestamp:0 //凡是值为整数形式的，均采用了压缩整数，但是和谷歌的protobuf的压缩算法不太一样。

11 00 

    02 //device_type:1

19 00 

    D6 F6 AE 9E 08 //appid：1105583531

20 00 

    25 00 00 00 //长度

        20 00 00 00 //长度

            34 44 34 30 43 33 43 45 35 36 45 36 42 32 35 36 37 42 45 32 43 42 31 43 45 32 33 45 37 31 37 36 

            //skey：4D40C3CE56E6B2567BE2CB1CE23E7176

        00 

28 00 

    25 00 00 00 

        20 00 00 00 

            35 30 35 31 30 33 46 33 46 33 33 34 36 42 46 39 31 35 43 45 43 33 31 45 33 34 43 44 42 35 30 44 

            //openid：505103F3F3346BF915CEC31E34CDB50D

    00
```

序列化完成之后，进行加密，加密之后添加包头

```
61 EA 00 00 //clsid，命令号

AF DE AB AC //security_flag，固定

00 00 00 00 //checksum，固定

67 00 00 00 //密文长度

2D 61 BC 00 55 E8 79 63 C9 51 39 50 8F 10 3E BC D3 1E 4A 2E 51 ED 5D F5 A2 D1 0D 05 13 EF 99 CC 0B 4C 42 C6 8D 23 BB 7D 83 06 CF 1A C2 CD 17 28 AC E1 02 11 74 B2 1A E1 AD DD D4 64 D5 1D BA B4 33 97 36 F6 7A E0 72 D6 E4 12 AD CD 82 31 C1 AA C1 B6 69 E5 82 2B 08 7F 4B AB 2C 69 C2 45 10 81 1E 53 6E 6E 1F 21 0C //密文
```

整数型压缩与解压算法和加密与解密算法随后给出

接下来先讨论返回的数据

java同样定义了返回的数据类

```
public class e extends d_INetMessage {

    @n(a=1) public int a;

    @n(a=2) public boolean b;

    @n(a=3) public ArrayList c;

    @n(a=4) public ArrayList d;

    @n(a=5) public int e;

    @n(a=6) public ag f;

    @n(a=7) public long g;

    @n(a=8) public boolean h;



    public e() {

        super();

    }



    public int h() {

        return 60002; //即为62 EA 00 00

    }



    public String toString() {

        return "CEventQueryVideoAccountInfoRes [m_time_stamp=" + this.a + ", m_succ=" + this.b + ", m_room_proxy_infos=" + this.c + ", m_account_infos=" + this.d + ", m_err_code=" + this.e + ", m_last_login_acc=" + this.f + ", m_qq=" + this.g + ", m_is_also_anchor=" + this.h + "]";

    }

}
```

收到数据：

```
62 EA 00 00 AF DE AB AC 00 00 00 00 E4 0C 00 00 AE 6D BC 00 E4 08 8F 95 84 B9 2C 4A 23 F1 C8 26 B7 1E EB F6 4B 67 0D 5E 3B D7 B4 14 C8 3C DB 1C 9A 45 01 72 0C 74 37 ......数据太长，后面不放出来了。
```

与发送数据正好相反

```
62 EA 00 00 //clsid

AF DE AB AC //security_flag

00 00 00 00 //checksum

E4 0C 00 00 //密文长度

AE 6D BC 00 E4 08 8F 95 84 B9 2C 4A 23 F1 C8 26 B7 1E EB F6 4B 67 0D 5E 3B D7 B4 14 C8 3C DB 1C 9A 45 01 72 0C 74 37 //密文
```

客户端拿到数据之后，首先获取clsid号，接着在一个`hashmap`中寻找对应的类，找到之后，通过反射获取类的所有属性，从而完成解析。

解析之后数据如下：

```
CEventQueryVideoAccountInfoRes [m_time_stamp=0, m_succ=true, m_room_proxy_infos=[RoomProxyInfo [name=华南一区, provider=, group=, recommend=false, zoneid=110, addresses=[], channel=0], RoomProxyInfo [name=华南二区......数据太长，截取部分
```

需要注意的是，并不是所有的数据结构类都会重写`toString()`方法帮助我们理解各个属性的真实含义，同样有很多类是没有重写该方法的。因此，我们需要尽可能重命名我们知道真实含义的每个值，方便以后遇到没有重写`toString()`方法时，对其属性的含义完成猜测和验证。

# 四、压缩和解压算法，加密与解密算法

1. 压缩解压算法如下：

```
    //long转为var_int

    private static void a_to_var_int(long arg6, f_NetBuffer arg8) {

        byte v2;

        long v0 = arg6 << 1 ^ arg6 >> 0x3F;

        while(true) {

            v2 = ((byte)(((int)(0x7F & v0))));

            v0 >>>= 7;

            if(v0 == 0) {

                break;

            }



            arg8.a_putByte(((byte)(v2 | 0x80)));

        }



        arg8.a_putByte(v2);

    }
```

```
    //var_int转为long

    private static long a_from_var_int(f_NetBuffer arg8) {

        long v2 = 0;

        int v0 = 0;

        do {

            int v1 = arg8.g_getByte();

            v2 += ((((long)v1)) & 0x7F) << v0;

            v0 += 7;

        }

        while((v1 & 0x80) != 0);



        return v2 >>> 1 ^ -(v2 & 1);

    }
```

2. 加密解密算法：

```
    //加密代码片段，arg4为初始密钥，12345678，arg5为明文，arg6为长度

    static void a(int arg4, byte[] arg5, int arg6) {

        int v0;

        for(v0 = 0; v0 < arg6; v0 += 4) {

            int v1 = v0;

            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)arg4))));

            v1 = v0 + 1;

            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 8)))));

            v1 = v0 + 2;

            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 16)))));

            v1 = v0 + 3;

            arg5[v1] = ((byte)(arg5[v1] ^ (((byte)(arg4 >> 24)))));

            arg4 = l.a_GetCRC32(arg5, v0, 4);

        }

    }
```

```
    //解密代码片段，arg5为初始密钥，12345678，arg6为密文，arg7为长度

    static void b(int arg5, byte[] arg6, int arg7) {

        int v0 = 0;

        while(v0 < arg7) {

            int v1 = l.a_GetCRC32(arg6, v0, 4);

            int v2 = v0;

            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)arg5))));

            v2 = v0 + 1;

            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 8)))));

            v2 = v0 + 2;

            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 16)))));

            v2 = v0 + 3;

            arg6[v2] = ((byte)(arg6[v2] ^ (((byte)(arg5 >> 24)))));

            v0 += 4;

            arg5 = v1;

        }

    }
```

# 五、重要数据包分析

在数据序列化一节已经以`QueryVideoAccountInfo`为例进行了阐述。在抓包之后，想要完成对数据包的详细分析，我采用xposed和jeb分析相结合的方法。我们已经知道，数据包开头即为clsid号，对应了一个java类，我们可以在JEB的disassembly页面直接搜索相关的十六进制或者十进制的值来定位关键类，同时，我们也可以通过xposed，在其序列化和反序列化的入口，拦截这个类，打印其方法名，从而得到其具体位置。

假设我们现在有一个数据包，其clsid为D1 A0 00 00。我们想要找到其实现类。

通过xposed日志，我们搜索数据包发送的clsid: D1 A0 00 00

接着找到`SaveStream_param_0`: `com.h3d.qqx5.model.s.a.c`

进入`com.h3d.qqx5.model.s.a.c`类，我们发现并没有重写`toString()`方法，无法准确得知各个属性的含义：

```
public class c extends d_INetMessage {

    @n(a=1) public String a;

    @n(a=2) public String b;

    @n(a=3) public String c;

    @n(a=4) public String d;

    @n(a=5) public String e;

    @n(a=6) public int f;

    @n(a=7) public int g;

    @n(a=8) public boolean h;



    public c() {

        super();

        this.h = false;

    }



    public int h() {

        return 0xA0D1;

    }

}
```

利用jeb交叉引用功能，以及之前对其他类的各种注释以及猜测，我们尽可能的完成了对属性的含义的解析

```
public class c extends d_INetMessage {

    @n(a=1) public String a_open_id;

    @n(a=2) public String b_open_key;

    @n(a=3) public String c_pay_token;

    @n(a=4) public String d_pf;

    @n(a=5) public String e_pf_key;

    @n(a=6) public int f_save_num;

    @n(a=7) public int g_device_type;

    @n(a=8) public boolean h_0; //其含义并没有分析出来，但是分析发现其固定为false，也就是0



    public c() {

        super();

        this.h_0 = false;

    }



    public int h() {

        return 0xA0D1;

    }

}
```

分析出其含义，我们对于数据包的组包和解包也就有了很大的信息。

下面给出xposed代码：
