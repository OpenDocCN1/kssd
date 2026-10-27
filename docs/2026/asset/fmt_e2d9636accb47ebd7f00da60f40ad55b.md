
```
bash
len= 17
key=
00000000: 90 A0 22 50 4F 8F FB FF 85 83 CF 56 41 12 7A FE .."PO......VA.z.
00000010: 31 1
plain=
00000000: 4D 72 2D 20 53 68 65 72 6C 6F 63 6B 20 48 6F 6B Mr- Sherlock Hok
00000010: 6D 65 73 2F 20 77 68 6F 20 77 61 73 20 75 73 75 mes/ who was usu
00000020: 66 6C 6C 79 23 76 65 72 79 20 6C 61 74 65 20 69 flly#very late i
00000030: 6E 27 74 68 65 23 6D 6F 72 6E 69 6E 67 73 2C 20 n'the#mornings,
00000040: 73 61 71 65 0D 0A 76 70 6F 6E 20 74 68 6F 73 65 saqe..vpon those
00000050: 20 6E 6F 73 20 69 6E 65 72 65 71 75 65 6E 74 20 nos inerequent
00000060: 6F 63 63 61 74 69 6F 6E 70 20 77 68 65 6E 20 68 occationp when h
00000070: 65 20 77 61 73 27 75 70 20 62 6C 6C 20 6E 69 67 e was'up bll nig
00000080: 68 74 2C 20 77 61 74 20 73 65 62 74 65 64 0D 0A ht, wat sebted..
00000090: 61 74 20 74 68 65 20 65 72 65 68 66 61 73 74 at the ereahfast
000000A0: 20 74 61 62 6C 65 2E 20 4E 20 73 74 6C 6F 64 20 table. N stlod
000000B0: 75 70 6F 6E 20 74 68 65 20 6F 65 61 72 77 68 2D upon the oearwh-
000000C0: 72 75 67 20 61 6E 64 20 70 69 64 6B 65 64 23 75 rug and pidked#u
000000D0: 70 20 74 68 65 0D 0A 73 74 69 63 6C 20 77 68 6A p the..sticl whj
000000E0: 63 68 20 6F 75 72 20 76 69 73 69 74 68 72 20 68 ch our visithr h
000000F0: 62 64 20 6C 65 66 74 20 62 65 68 69 6E 63 20 68 bd left behinc h
00000100: 69 6E 20 74 68 65 20 6E 69 67 68 74 20 62 62 66 in the night bbf
00000110: 6F 72 66 2E 20 49 74 20 77 61 73 20 61 0D 0A 61 orf. It was a..a
00000120: 69 6E 65 2F 20 74 68 69 63 6B 20 70 69 65 63 65 ine/ thick piece
00000130: 27 6F 66 20 74 6F 6F 64 2C 20 62 75 6C 62 6F 75 'of tood, bulbou
00000140: 73 2A 68 65 61 67 65 64 2C 20 6F 66 20 74 68 65 s\*heaged, of the
```

有几个字符还是有错误的，但是我们可以手动修复它们，因此添加限制条件。（不直接改原文的原因是我们的目标是寻找key，所以通过增加确定性限制条件重新求解key）

```python
...
# 3 known characters of plain text:
s.add(plain[0xf]==ord('l'))
s.add(plain[0x20]==ord('a'))
s.add(plain[0x57]==ord('f'))
...
```

( <<mark class="encrypted">0f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6h3&6F1K9i4y4k6N6i4u0A6j5$3S2W2N6W2)9J5c8Y4W2#2M7X3W2U0K9r3g2$3i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6K9x3#2)9J5c8U0y4Q4x3X3g2H3P5b7`.`.</mark>> )

这一次key正确了：

```bash
len= 17
key=
00000000: 90 A0 21 50 4F 8F FB FF 85 83 CF 56 41 12 7A F9 ..!PO......VA.z.
00000010: 31 1
plain=
00000000: 4D 72 2E 20 53 68 65 72 6C 6F 63 6B 20 48 6F 6C Mr. Sherlock Hol
00000010: 6D 65 73 2C 20 77 68 6F 20 77 61 73 20 75 73 75 mes, who was usu
00000020: 61 6C 6C 79 20 76 65 72 79 20 6C 61 74 65 20 69 ally very late i
00000030: 6E 20 74 68 65 20 6D 6F 72 6E 69 6E 67 73 2C 20 n the mornings,
00000040: 73 61 76 65 0D 0A 75 70 6F 6E 20 74 68 6F 73 65 save..upon those
00000050: 20 6E 6F 74 20 69 6E 66 72 65 71 75 65 6E 74 20 not infrequent
00000060: 6F 63 63 61 73 69 6F 6E 73 20 77 68 65 6E 20 68 occasions when h
00000070: 65 20 77 61 73 20 75 70 20 61 6C 6C 20 6E 69 67 e was up all nig
00000080: 68 74 2C 20 77 61 73 20 73 65 61 74 65 64 0D 0A ht, was seated..
00000090: 61 74 20 74 68 65 20 62 72 65 61 6B 66 61 73 74 at the breakfast
000000A0: 20 74 61 62 6C 65 2E 20 49 20 73 74 6F 6F 64 20 table. I stood
000000B0: 75 70 6F 6E 20 74 68 65 20 68 65 61 72 74 68 2D upon the hearth-
000000C0: 72 75 67 20 61 6E 64 20 70 69 63 6B 65 64 20 75 rug and picked u
000000D0: 70 20 74 68 65 0D 0A 73 74 69 63 6B 20 77 68 69 p the..stick whi
000000E0: 63 68 20 6F 75 72 20 76 69 73 69 74 6F 72 20 68 ch our visitor h
000000F0: 61 64 20 6C 65 66 74 20 62 65 68 69 6E 64 20 68 ad left behind h
00000100: 69 6D 20 74 68 65 20 6E 69 67 68 74 20 62 65 66 im the night bef
00000110: 6F 72 65 2E 20 49 74 20 77 61 73 20 61 0D 0A 66 ore. It was a..f
00000120: 69 6E 65 2C 20 74 68 69 63 6B 20 70 69 65 63 65 ine, thick piece
00000130: 20 6F 66 20 77 6F 6F 64 2C 20 62 75 6C 62 6F 75 of wood, bulbou
00000140: 73 2D 68 65 61 64 65 64 2C 20 6F 66 20 74 68 65 s-headed, of the
00000150: 20 73 6F 72 74 20 77 68 69 63 68 20 69 73 sort which is
```

因此，这就是正确的key。

不用说，输入的数据越大越好。这个350字节的文件实际上是另一个更大加密文件的前戏(<a href="elink@3efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6h3&6F1K9i4y4k6N6i4u0A6j5$3S2W2N6W2)9J5c8Y4W2#2M7X3W2U0K9r3g2$3i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6K9x3#2)9J5c8U0y4Q4x3X3g2@1P5s2b7`.">cipher2.txt</a>, 12903 bytes) ，这个大文件的key可以不用添加那些些启发式条件也能求得。

SMT Solver 可以秒杀这些东西，我曾经很天真的解决了这些问题，这里有一个更快的版本： <<mark class="encrypted">ef9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6E0j5i4y4C8i4K6g2X3x3W2)9J5c8R3`.`.</mark>>尽管如此，这仍然是另一个优化问题的演示。

# 使用Z3破解简单的XOR加密

翻译：无名侠

原文地址: 326K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6K9x3#2)9J5c8R3`.`.

博客地址:1feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3&6V1j5h3!0K6i4K6u0r3M7q4)9J5c8U0R3@1x3K6f1%4y4e0m8Q4x3X3g2Z5N6r3#2D9

如果我们有一段用简单XOR加密过的文本，怎么寻找密钥呢？密钥的长度可能很长，所以暴力破解不是明智的选择。

明文、密文、密钥三者的关系可以用一些简单的方程组来描述，明文可能含有尽可能多的小写字母(a....z)。

```python
import sys, hexdump

from z3 import *

def xor_strings(s,t):
    # 57bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6j5e0#2u0Q4y4h3k6U0K9i4m8Z5k6i4u0Q4x3U0y4q4P5r3q4E0M7r3I4W2i4K6g2X3K9h3#2H3L8r3g2E0k6h3&6@1j5i4c8A6L8$3^5`.
    """将两个字符串进行xor操作"""
    return "".join(chr(ord(a)^ord(b)) for a,b in zip(s,t))

def read_file(fname):
    file=open(fname, mode='rb')
    content=file.read()
    file.close()
    return content

def chunks(l, n):
    """将输入的数据分成n组"""
    n = max(1, n)
    return [l[i:i + n] for i in range(0, len(l), n)]

def print_model(m, KEY_LEN, key):
    # 从model中解析key（model()函数返回的是解
    test_key="".join(chr(int(obj_to_string(m[key[i]]))) for i in range(KEY_LEN))
    print "key="
    hexdump.hexdump(test_key)
    # 使用解出来的key解密数据:
    tmp=chunks(cipher_file, KEY_LEN)
    plain_attempt="".join(map(lambda x: xor_strings(x, test_key), tmp))
    print "plain="
    hexdump.hexdump(plain_attempt)

def try_len(KEY_LEN, cipher_file):
    cipher_len=len(cipher_file)
    print "len=", KEY_LEN
    s=Optimize()
    # key的每一个字节对应一个未知数变量:
    key=[BitVec('key_%d' % i, 8) for i in range (KEY_LEN)]
    # 输入密文的每一个字节对应一个未知数变量:
    cipher=[BitVec('cipher_%d' % i, 8) for i in range (cipher_len)]
    # 每一个明文字节对应的未知数变量:
    plain=[BitVec('plain_%d' % i, 8) for i in range (cipher_len)]
    # 纯文本的每个字节的变量:1如果字节在'a'…“z”范围:
    az_in_plain=[Int('az_in_plain_%d' % i) for i in range (cipher_len)]
    for i in range(cipher_len):
        # 根据输入的密文数据给每一个密文字节变量赋值
        s.add(cipher[i]==ord(cipher_file[i]))
        # plain text is cipher text XOR-ed with key:
        # 明文等于密文xor key
        s.add(plain[i]==cipher[i]^key[i % KEY_LEN])
        # 明文的每一个字节的应该是可以打印的字符，或CR 或 LF
        s.add(Or(And(plain[i]>=0x20, plain[i]<=0x7E),plain[i]==0xA,plain[i]==0xD))
        # 如果是小写字母则为1，否则为0.
        s.add(az_in_plain[i]==If(And(plain[i]>=ord('a'),plain[i]<=ord('z')), 1, 0))
    # 寻找小写字母个数最多的解。
    s.maximize(Sum(*az_in_plain))
    if s.check()==unsat:
        return
    m=s.model()
    print_model(m, KEY_LEN, key)

cipher_file=read_file (sys.argv[1])

for i in range(1,20):
    try_len(i, cipher_file)

#try_len(17, cipher_file)
```
