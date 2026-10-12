
# 共享迅雷 P2P 通信的加密算法

> 原文：[`www.pediy.com/kssd/pediy12/144482.html`](https://www.pediy.com/kssd/pediy12/144482.html)

来源[`hi.baidu.com/vessial/blog/ite...e35a80f93.html`](http://hi.baidu.com/vessial/blog/item/93080325fed3097e35a80f93.html)

共享一下迅雷 P2P 通信的加密算法，方便那些研究迅雷 P2P 通信协议的同学们：）

算法细节不详细描述了，共享算法代码，只有 python 版，解密出来的结果就是

明文协议了，接下来的事情就是做协议分析了。发一个协议通信的片断。

`192.168.1.103 port 10047` ---> `111.195.202.215 port 10527`

```
0000  32 00 00 00 05 10 00 00 00 30 30 32 32 46 42 38  2........0022FB8
0010  33 41 41 41 34 38 4e 51 51 00 00 3b b4             3AAA48NQQ..;.
```

`111.195.202.215 port 10527` ---> `192.168.1.103 port 10047`

```
0000  32 00 00 00 06 00 00 00 00 3b b4 00 00 61 6a 00  2........;...aj.
0010  02 25 e2 60 24 00 00 00 00 00 00 01 00 00 00      .%.`$..........
```

`192.168.1.103 port 10047` ---> `111.195.202.215 port 10527`

```
0000  32 00 00 00 06 01 00 00 00 00 00 3b b4 61 57 e1  2..........;.aW.
0010  03 63 e1 7a 03 26 e2 60 24 00 00 05 00 00 00      .c.z.&.`$......
```

`111.195.202.215 port 10527` ---> `192.168.1.103 port 10047`

```
0000  32 00 00 00 11 3b b4 00 00 61 6a 00 02 00 00 01  2....;...aj.....
0010  00 26 e2 60 24 64 e1 7a 03 63 e1 7a 03 01 00 00  .&.`$d.z.c.z....
0020  00 00 00 00 00                                   .....
```

算法如下

```python
# Created by vessial

def p2p_udp_encrypt(pkt):
    header=random.randint(0,0xffff)
    t=((header&0x1fff)+0x4000)<<0x10
    t+=random.randint(0,0xffff)
    body=struct.pack('I',t)
    sec2=random.randint(0,0xff)
    body+=struct.pack('B',sec2)
    length=(sec2&0x03)+9
    pos=length
    for i in range(length-5):
        x=random.randint(0,0xff)
        body+=struct.pack('B',x)
    l=pos*7
    t=struct.unpack('B',body[pos-3])[0]
    t*=0x0d
    t&=0xff
    k=l^t
    header=body[:pos-2]+struct.pack('B',k)
    t=struct.unpack('B',header[-1])[0]
    t*=0x0d
    t&=0xff
    k=(l+7)^t
    header=header+struct.pack('B',k)
    out=''
    j=0
    t=array.array('B',header)
    buf=array.array('B',pkt)
    body_len=len(pkt)
    for i in range(body_len):
        j+=1
        if j==pos:
            j=0
        x=t[j]+0x5b
        x&=0xff
        x^=t[j-1]
        t[j-1]=x
        buf[i]=(buf[i]+x)&0xff
    out=header+buf.tostring()
    #print "out buf len is %d"%len(out)
    #print hexdump(out)
    #print hexdump(p2p_udp_decrypt(out))
    return out

def p2p_udp_decrypt(pkt):
    if len(pkt)<=8:
        return False
    header=struct.unpack('<I',pkt[:4])[0]
    header>>=0x1d
    if header>3:
        return False
    elif header ==1:
        pass
    elif header ==2:
        byte5=struct.unpack('B',pkt[4])[0]
        byte5&=0x80000003
        byte5+=9
        if byte5>len(pkt):
            return False
        pos=byte5
        head_check=array.array('B',pkt[:pos])
        t=struct.unpack('B',pkt[pos-2])[0]
        mid=pos*7
        p=7*pos+7
        t*=0x0d
        t&=0xff
        p^=t
        s=struct.unpack('B',pkt[pos-1])[0]
        if p != s:
            return False
        t=struct.unpack('B',pkt[pos-3])[0]
        t*=0x0d
        t&=0xff
        t^=mid
        if t !=struct.unpack('B',pkt[pos-2])[0]:
            return False
        j=0
        body=array.array('B',pkt[pos:])
        body_len=len(pkt)-pos
        for i in range(body_len):
            j+=1
            if j==pos:
                j=0
            x=head_check[j]+0x5b
            x&=0xff
            x^=head_check[j-1]
            head_check[j-1]=x
            body[i]=(body[i]-x)&0xff
        x=body.tostring()
        #print hexdump(x)
        return x
    elif header ==3:
        pass
    return False
```
