
```
import string

import struct

import numpy as np

from pwn import *



base62c = string.digits + string.ascii_uppercase + string.ascii_lowercase

base18c = "0123456789ABCDEFGH"



def basen_e(i, n):

    assert(n <= 62)

    res = ""

    while i > 0:

        res += base62c[i % n]

        i /= n

    return res



def basen_d(s, n):

    sums = 0

    mults = 1

    for i in range(len(s)):

        num = base62c.index(s[i])

        sums += num * mults

        mults *= n

    return sums



def compute(s):

    i = basen_d(s, 62)

    return basen_e(i, 18)



def inverse_compute(s):

    i = basen_d(s, 18)

    return basen_e(i, 62)



def read_array():

    with open("arr.mem", "rb") as f:

        res = f.read()

    assert(len(res) == (48 * 2 + 48 * 48 * 6) * 4)

    nums = len(res) / 4

    res = struct.unpack("<%dL" % nums, res)

    result = np.array(list(res[:48]))

    iv = np.array(list(res[48:96]))

    arrs = list(res[96:])

    arrs = [np.array(arrs[i:i+48*48]).reshape(48, 48) for i in range(0, len(arrs), 48*48)]

    return result, iv, arrs



def array_pow(arrs, iv, s):

    #arr_count = [0 for i in range(6)]

    for i in s:

        i = base18c.index(i)

        a1, a2 = i / 3, (i % 3) + 1

        for j in range(a2):

            iv = np.dot(iv, arrs[a1])

            #arr_count[a1] += 1

    return iv#, arr_count



def dump_bignum(s):

        s = format(s, 'x')

        s = '0' * (len(s) % 2) + s

        print hexdump(s.decode('hex')[::-1])



num = 1773497881723270350100

flag = basen_e(num, 62)

print flag



result, iv, arrs = read_array()

print np.array_equal(iv, result)

s = compute("KanXueCrackMe2017")

iv1 = array_pow(arrs, iv, s)



b18_flag = compute(flag)

iv2 = array_pow(arrs, iv1, b18_flag)



assert(np.array_equal(iv2, result))
```

总的来说这题逆向起来还是挺费劲的……而且还有椭圆曲线密码学……是一道还不错的题……不过当初不逆第一个keygen就好了……完全没卵用……

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
