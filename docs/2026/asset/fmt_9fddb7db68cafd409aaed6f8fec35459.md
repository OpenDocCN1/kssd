
```
def CalcPassAndXor(passwd):

    array = bytearray(passwd);

    size = len(array);

    arr = [];

    i = 0;

    while i < size:

        ch1 = array[i];

        if ch1 >= 0x30 and ch1 <= 0x39:

            ch1 -= 0x30;

        elif ch1 >= 0x61 and ch1 <=0x66:

            ch1 -= 0x57;

        elif ch1 >= 0x41 and ch1 <=0x46:

            ch1 -= 0x37;

        # print ch1;

        i+=1;

        if i == size:

            break;

        ch2 = array[i];

        if ch2 >= 0x30 and ch2 <= 0x39:

            ch2 -= 0x30;

        elif ch2 >= 0x61 and ch2 <= 0x66:

            ch2 -= 0x57;

        elif ch2 >= 0x41 and ch2 <= 0x46:

            ch2 -= 0x37;

        ch3 =  ch1 << 4 | ch2;

        ch3 ^= 0x86;

        arr.append(ch3);

        i += 1;

    return arr;
```

换句话说，由于上面的函数只支持0-F的16进制值，可以简化为一张表，这张表对应 0-F 中的值。代码如下：

```
dic = {'0': 'B6', '1': 'B7', '2': 'B4', '3': 'B5', '4': 'B2', '5': 'B3', '6': 'B0', '7': 'B1', '8': 'BE',

               '9': 'BF', 'A': 'BC', 'B': 'BD', 'C': 'BA', 'D': 'BB', 'E': 'B8', 'F': 'B9'}
```

### 9. 完整的注册机代码

```
# -*- coding: utf-8 -*-

__author__="lantie@15PB"

#  将10进制字符串转为16进制
def calc(name,count=10):
    num = 0;
    array = bytearray(name);
    for i in range(0, len(name)):
        by = array[i] - 0x30;
        #print hex(by);
        if num > 0:
            num *= count;
        num += by;
    return num;

# 将10进制字符串转为16进制，或将16进制进行一些运算
def calc2(bytearr,count=0x10):
    num = 0;
    array = bytearray(bytearr);
    for i in range(0, len(bytearr)):
        if num > 0:
            num *= count;
        by = array[i];
        by -= 0x30;
        if by >=80:
            by = 256-by;
            print hex(by);
            if num > by:
                num -= by;
        else:
            num += by;
        if num == 0:
            num += by;
    return num;

# 对两个数进行计算，以4字节为单位
def CalcUser(num1,num2):
    n1 = num2;
    num3 = 0;
    num4 = 0;
    arr = [];
    while n1 != 0:
        n = n1 & 0xffffffff;
        #print hex(n);
        num3 = n * num1;
        num3 += num4;
        #print hex(num3);
        n = num3 & 0xffffffff;
        arr.insert(0, n);
        n1 >>= 32;
        num3 >>= 32;
        if num3 != 0:
            num4 = num3;
    return arr;

# 对密码进行转换。
def CalcPassAndXor(passwd):
    array = bytearray(passwd);
    size = len(array);
    arr = [];
    i = 0;
    while i < size:
        ch1 = array[i];
        if ch1 >= 0x30 and ch1 <= 0x39:
            ch1 -= 0x30;
        elif ch1 >= 0x61 and ch1 <=0x66:
            ch1 -= 0x57;
        elif ch1 >= 0x41 and ch1 <=0x46:
            ch1 -= 0x37;
        # print ch1;
        i+=1;
        if i == size:
            break;
        ch2 = array[i];
        if ch2 >= 0x30 and ch2 <= 0x39:
            ch2 -= 0x30;
        elif ch2 >= 0x61 and ch2 <= 0x66:
            ch2 -= 0x57;
        elif ch2 >= 0x41 and ch2 <= 0x46:
            ch2 -= 0x37;
        ch3 =  ch1 << 4 | ch2;
        ch3 ^= 0x86;
        arr.append(ch3);
        i += 1;
    return arr;

# 注册机
def CalcPass():
    username = raw_input('please input name: ');
    num1 = calc2(username,10);
    #print 'sum:' + hex(num1);
    num2 = calc2("201510261314",10);
    #print 'sum:' + hex(num2);
    arr = CalcUser(num1,num2)
    password = "BEEF";
    for i in range(len(arr)):
        dic = {'0': 'B6', '1': 'B7', '2': 'B4', '3': 'B5', '4': 'B2', '5': 'B3', '6': 'B0', '7': 'B1', '8': 'BE',
               '9': 'BF', 'A': 'BC', 'B': 'BD', 'C': 'BA', 'D': 'BB', 'E': 'B8', 'F': 'B9'}

        string1 = hex(arr[i]).upper();
        for i in range(2, len(string1)):
            ch = string1[i];
            if ch == 'L':
                continue;
            # print dic[ch];
            password += dic[ch];
    print "password: " + password;

if __name__ == '__main__':
    CalcPass();
```

### 10. 一组正确的用户名和序列号

![](https://bbs.kanxue.com/upload/attach/201709/761040_4g7sonyasro5lh8.png)

## 四. 总结

- 题目设计

  这个题目有一个设计的非常巧妙的地方，那就是密码的前4位作为了解密代码的key，如果不对就无法解密，由于需要暴力破解出前4位密码，瞬间增加了这个题目的难度。而后解密代码的算法也比较巧妙，用户名和密码都参加了计算，非常适合有一点强度的练手。

- 分析收获

  经过分析这个题目，从逆向工具OD、IDA的使用，到VS、python的代码编写，方方面面都涉及到了，真是一个综合性的练习！收获大大的！

- 硬道理

  逆向分析就是练！练！练！,只有打不死的小强，没有练不出来的技术！

- 时间与广告

  2017年9月7日，lantie@15PB

  15PB信息安全教育专注于信息安全官网：[b14K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0p5#2M7r3u0Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8R3`.`.](https://bbs.kanxue.com/elink@253K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0p5#2M7r3u0Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8R3`.`.)

- 致谢

  感谢看雪提供平台！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [注册机.py](javascript:void(0);)

  （2.89kb，80次下载）

- [第四题.exe](javascript:void(0);)

  （24.00kb，112次下载）

- [解密程序.cpp](javascript:void(0);)

  （19.28kb，65次下载）
