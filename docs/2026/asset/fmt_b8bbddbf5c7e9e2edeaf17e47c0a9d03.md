
# CrackMe 分析笔记

## 初始汇编片段

```
00401645 . 897C24 04          mov dword ptr ss:[esp+0x4],edi ; |offset = 21DE1 (138721.)

00401649 . C74424 08 00000000 mov dword ptr ss:[esp+0x8],0x0 ; |whence = SEEK_SET

00401651 . 8B03               mov eax,dword ptr ds:[ebx] ;     |

00401653 . 890424             mov dword ptr ss:[esp],eax ;     |stream = msvcrt.77C2FCE0

00401656 . E8 2D950000        call <jmp.&msvcrt.fseek> ;       \fseek

……

……

0040168D . C74424 08 01000000 mov dword ptr ss:[esp+0x8],0x1 ; |n = 0x1

00401695 . 894424 04          mov dword ptr ss:[esp+0x4],eax ; |size = 675 (1653.)

00401699 . 893C24             mov dword ptr ss:[esp],edi ;     |ptr = 003E82A8

0040169C . 895424 0C          mov dword ptr ss:[esp+0xC],edx ; |stream = msvcrt.77C2FCE0

004016A0 . E8 F3940000        call <jmp.&msvcrt.fread> ;       \fread
```

## 解压说明

解压使用的是`zlib`库，版本1.2.8，可根据代码中的错误提示字符串搜索得到。

C代码的执行的逻辑到这就结束了，因为程序运行起来看到的界面都是在python中执行的，后面就说下怎么获取并反编译pyc

## 提取 pyc

前面说到了一个表，这个表在原始的cm文件的末尾，可以直接拷贝出来单独保存为`dataInfo`，然后用下面的脚本解出所有的pyc

```
#-*- coding:utf-8 -*-

import struct

import zlib

if __name__ == "__main__":

    fp = open('dataInfo', 'rb')

    while True:

        data = fp.read(16);

        if data == '':

            break;

        blockSize, dataOffset, dataLen, unknow = struct.unpack('>IIII', data)

        print 'blockSize:\t0x%08x\ndataOffset:\tx%08x\ndataLen:\t0x%08x\nunknow: 0x%08x' % (blockSize, dataOffset, dataLen, unknow)

        fp.read(1)

        dataName = fp.read(blockSize - 17)

        name = dataName.strip('\x00').replace('\\','_')

        print 'dataName: %s\n' % dataName

        if dataLen == 0:

            continue

        fout = open(name, 'wb')

        fexe = open('CrackMe_CTF_Demo.exe', 'rb')

        fexe.seek(0x1F000 + dataOffset)

        fout.write(fexe.read(dataLen))

        fout.close()

        fexe.close()

        if dataLen > 0x11f000:

            continue

        decompress(name, name + '.pyc')
```

解压出来的pyc中主要的是`sKERNEL32.dll.pyc` 下载 `Easy Python Decompiler v1.3.2` 参考这篇[7aaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3)9I4L8K6q4G2x3h3)9I4L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2K9r3E0Q4x3V1j5J5x3o6p5$3i4K6u0r3x3e0q4Q4x3V1k6H3P5i4c8Z5L8$3&6Q4x3X3c8H3P5h3W2F1M7%4c8S2L8r3I4W2M7W2)9J5k6s2u0W2N6X3g2J5M7$3g2Q4x3X3c8W2L8X3N6A6L8X3g2W2M7W2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@d76K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3)9I4L8K6q4G2x3h3)9I4L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2K9r3E0Q4x3V1j5J5x3o6p5$3i4K6u0r3x3e0q4Q4x3V1k6H3P5i4c8Z5L8$3&6Q4x3X3c8H3P5h3W2F1M7%4c8S2L8r3I4W2M7W2)9J5k6s2u0W2N6X3g2J5M7$3g2Q4x3X3c8W2L8X3N6A6L8X3g2W2M7W2)9J5k6h3S2@1L8h3H3`.) 中的说法

> 下面這包是用 python 3.4，一樣可以查表  
> 或是利用 `archive_viewer.py` 顯示前幾包的內容，開頭的 4 bytes 就是 `magic number`  
> `'EE 0C 0D 0A'`

在`sKERNEL32.dll.pyc`最前面添加　`EE 0C 0D 0A XX XX XX XX XX XX XX XX` 十二个字节，`注意是添加不是修改` 我一开始是修改了开头的12字节，结果导致Easy Python Decompiler无法还原，后来在这个帖子里

[c09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3U0p5H3y4U0M7K6x3e0y4Q4x3V1k6Z5L8%4N6Q4x3X3c8U0L8$3&6$3k6i4u0@1i4K6u0V1j5h3&6V1i4K6u0V1M7$3q4$3k6g2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6r3#2G2k6s2g2D9k6g2)9J5k6r3k6J5L8$3#2Q4x3X3c8H3P5h3!0T1K9X3g2U0N6q4)9J5k6r3q4K6i4K6u0V1j5X3W2F1j5i4u0&6i4K6u0V1k6r3q4@1j5g2)9J5k6s2c8G2i4K6u0V1N6i4y4W2i4K6u0V1K9i4c8Q4x3X3c8D9j5i4b7`.](https://bbs.kanxue.com/elink@3f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3U0p5H3y4U0M7K6x3e0y4Q4x3V1k6Z5L8%4N6Q4x3X3c8U0L8$3&6$3k6i4u0@1i4K6u0V1j5h3&6V1i4K6u0V1M7$3q4$3k6g2)9J5k6s2m8&6N6r3S2G2L8W2)9J5k6r3#2G2k6s2g2D9k6g2)9J5k6r3k6J5L8$3#2Q4x3X3c8H3P5h3!0T1K9X3g2U0N6q4)9J5k6r3q4K6i4K6u0V1j5X3W2F1j5i4u0&6i4K6u0V1k6r3q4@1j5g2)9J5k6s2c8G2i4K6u0V1N6i4y4W2i4K6u0V1K9i4c8Q4x3X3c8D9j5i4b7`.) 看到一段代码

```
codeobj = PyMarshal_ReadObjectFromString(python_code+8, size-8);
```

这里减8 是因为是python2.7 只有前8字节是`magic value`, 而3.4是前12字节

补上12字节的magic value后直接拖到Easy Python Decompiler就能看到python代码了，如下

## 反编译后 Python 代码

```
# File: s (Python 3.4)

import os

import hashlib

def z1(z2):

    for n in range(0, len(z2)):

        if ord(z2[n]) not in range(48, 58) and ord(z2[n]) not in range(97, 103):

            return 0

    return 1

def b(e2, c_2):

    g = ''

    for f2 in range(0, len(e2)):

        g += chr(e2[f2] ^ c_2 ^ f2 + 1)

    return g

m1 = [244,208,209,214,215,212,213,218,219,216,217,

        222,223,209,179,157,143,142,135,134,143,201,

        197,202,203,200,201,206,207,204,205,242,243,240]

        

m2 = [244,169,148,158,218,159,148,150,145,213,157,128,

        210,136,159,154,156,205,143,132,152,155,141,132,

        146,197,148,130,145,146,151,176,172,185,214]

        

m3 = [142,145,153,154,137,156,216,158,152,133,129,

        135,210,129,145,156,157,154,131,153,142,211,200]

        

m4 = [250,161,151,146,157,132,130,139,153,143,147,146,146,

        223,141,148,131,128,135,150,151,129,147,133,201,225]

        

m5 = [ 219,223,216,137,217,209,140,129,209,131,129,215,214,133,130,198,

        152,196,158,205,204,203,201,145,199,193,193,198,194,148,197,250]

print(b(m1, 255))

print(b(m2, 255))

while None:

    try:

        z4 = input(b(m3, 255))                        #获取输入

        if len(z4) == 10 and z1(z4) == 1:             #检查输入

            z3 = str(int(z4, 16))                     #16进制转10进制字符串

            if hashlib.md5(z3.encode('utf-8')).hexdigest() == b(m5, 239):    #求md5验证

                print(b(m4, int(z4[8:10], 16)))

                break

    continue     

    continue

    continue

    return None

    

#调整一下while使他能直接运行

#while True:

#    z4 = input(b(m3, 255))                        #获取输入

#    if len(z4) == 10 and z1(z4) == 1:             #检查输入

#        z3 = str(int(z4, 16))                     #16进制转10进制字符串

#        if hashlib.md5(z3.encode('utf-8')).hexdigest() == b(m5, 239):    #求md5验证

#            print(b(m4, int(z4[8:10], 16)))

#            break

#    continue
```

## 代码分析

代码看起来有些问题，先注释掉while后面的用python运行下看看，注意运行环境是python3.x

```
———— Crackme ————

The flag is your correct password

Press any key to continue …
```

看到了这个界面，看来这就是验证的地方，仔细阅读下while循环中的代码，得知`z4`是输入且长度为10，`z1`用来验证输入的有效性，

可知`key`的长度为10且由0-9,a-f组成，`str(int(z4, 16))`用来把输入的16进制串转为10进制串。

关键的判断点是`if hashlib.md5(z3.encode('utf-8')).hexdigest() == b(m5, 239)`

所以算法就是判断输入的值转10进制字符串后的md5是否与预设值相等，将`b(m5, 239)`放在前面直接print得到md5为524b38df7fe44db9f9b6621f14550e55

## 但是这个md5值与实际执行时的并不一样。

---

## SMC 修改分析

代码中其实还有smc， 最终运行的python脚本与文件里保存的有一点区别，修改代码的函数是`sub_407CC0`

```
00401773 . 892C24          mov dword ptr ss:[esp],ebp

00401776 . E8 45650000     call CrackMe_.00407CC0
```

这里有个有趣的事情，下面这段代码如果单步运行，那么最后不会修改python代码，直接走free，如果在这段代码之后下断点直接run过去，就会多走修改python代码的处理，经过几次单步感觉是`pop ss`影响了`push fd`的执行，`nop`掉`push ss`和`pop ss`就能单步跟到smc处

```
0040AD45 > 60        pushad

0040AD46 . 16        push ss

0040AD47 . EB 01     jmp short CrackMe_.0040AD4A

0040AD49   18 db 18

0040AD4A > 17        pop ss

0040AD4B . 9C        pushfd

0040AD4C . EB 01     jmp short CrackMe_.0040AD4F
```

smc代码

```
0040ADA2 . A1 F0BF4000   mov eax,dword ptr ds:[0x40BFF0]      ;读取解压后的代码长度

0040ADA7 . EB 01         jmp short CrackMe_.0040ADAA

0040ADA9 05 db 05

0040ADAA > 3D 2C070000   cmp eax,0x72C                        ;比较代码长度是否为0x72C

0040ADAF . EB 01         jmp short CrackMe_.0040ADB2

0040ADB1 0F db 0F

0040ADB2 >^ 75 D3        jnz short CrackMe_.0040AD87          ;不相等不修改，跳走

0040ADB4 . EB 01         jmp short CrackMe_.0040ADB7          ;相等准备修改

0040ADB6 1D db 1D

0040ADB7 > A1 F4BF4000   mov eax,dword ptr ds:[0x40BFF4]      ;获取解压后代码起始位置

0040ADBC . EB 01         jmp short CrackMe_.0040ADBF

0040ADBE C7 db C7

0040ADBF > 05 C5010000   add eax,0x1C5                        ;偏移0x1c5

0040ADC4 . EB 01         jmp short CrackMe_.0040ADC7

0040ADC6 C8 db C8

0040ADC7 > FE08          dec byte ptr ds:[eax]                ;减去1
```
