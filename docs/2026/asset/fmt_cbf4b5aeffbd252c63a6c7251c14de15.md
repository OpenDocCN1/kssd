
```python
# 检测大整数是否是素数,如果是素数,就返回True,否则返回False
# miller_rabin算法
import random

def rabin_miller(num):
    s = num - 1
    t = 0
    while s % 2 == 0:
        s = s // 2
        t += 1

    for trials in range(5):
        a = random.randrange(2, num - 1)
        v = pow(a, s, num)
        if v != 1:
            i = 0
            while v != (num - 1):
                if i == t - 1:
                    return False
                else:
                    i = i + 1
                    v = (v ** 2) % num
    return True

def is_prime(num):
    # 排除0,1和负数
    if num < 2:
        return False

    # 创建小素数的列表,可以大幅加快速度
    # 如果是小素数,那么直接返回true
    small_primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373, 379, 383, 389, 397, 401, 409, 419, 421, 431, 433, 439, 443, 449, 457, 461, 463, 467, 479, 487, 491, 499, 503, 509, 521, 523, 541, 547, 557, 563, 569, 571, 577, 587, 593, 599, 601, 607, 613, 617, 619, 631, 641, 643, 647, 653, 659, 661, 673, 677, 683, 691, 701, 709, 719, 727, 733, 739, 743, 751, 757, 761, 769, 773, 787, 797, 809, 811, 821, 823, 827, 829, 839, 853, 857, 859, 863, 877, 881, 883, 887, 907, 911, 919, 929, 937, 941, 947, 953, 967, 971, 977, 983, 991, 997]
    if num in small_primes:
        return True

    # 如果大数是这些小素数的倍数,那么就是合数,返回false
    for prime in small_primes:
        if num % prime == 0:
            return False

    # 如果这样没有分辨出来,就一定是大整数,那么就调用rabin算法
    return rabin_miller(num)

# 得到大整数,默认位数为1024
def get_prime(key_size=1024):
    while True:
        num = random.randrange(2**(key_size-1), 2**key_size)
        if is_prime(num):
            return num

print(get_prime(50))
```

```text
602912217591119
```

#### `libnum`

```python
import libnum
libnum.generate_prime(30)
```

```text
936552131
```

```python
libnum.generate_prime_from_string("abc")
```

```text
418262526581
```

#### `Crypto.Util.number`

```python
from Crypto.Util.number import getPrime
getPrime(30)
```

```text
930767861
```

### rsa库简单使用

#### 密钥生成

```python
import rsa
(pubkey, privkey) = rsa.newkeys(128)
pubkey,privkey
```

```text
(PublicKey(210654150686773160921155565886246123127, 65537),
 PrivateKey(210654150686773160921155565886246123127, 65537, 45430608142070156598272456648718438625, 245259021963773848463, 858904797874827929))
```

#### rsa加密

```python
m = "hello".encode('utf-8')
rsa.encrypt(m,pubkey)
```

```text
b']\xd6\xb2w\xc4\x89[\xfcu`\x0b&\xa0\xc9`\xd2'
```

#### rsa解密

```python
rsa.decrypt(b']\xd6\xb2w\xc4\x89[\xfcu`\x0b&\xa0\xc9`\xd2', privkey)
```

```text
b'hello'
```

### 公钥文件查看方式

#### `openssl`

`openssl rsa -pubin -in pubkey.pem -text -modulus`

#### `rsa` 库

```python
import rsa
with open('publickey.pem', mode='rb') as f:
    keydata = f.read()
    pubckey = rsa.PublicKey.load_pkcs1_openssl_pem(keydata)
    pubckey.n
    pubckey.e
```

### 私钥文件生成方式

#### `rsatool.py`

`python rsatool.py -f PEM -o prvkey.pem -p 4184799299 -q 3303891593`

#### `from Crypto.PublicKey import RSA`

```python
# coding=utf-8
import math
import sys
from Crypto.PublicKey import RSA

rsa_components=(n,e,int(d),p,q)

keypair=RSA.construct(rsa_components)

private = open('private.pem', 'wb')
private.write(keypair.exportKey())
private.close()
```

### 根据私钥文件读取公钥加密后密文方式

#### 利用 `rsa` 库直接读

```python
import rsa
prikey = rsa.PrivateKey(n , e , d , p , q)
with open("test.enc" , "rb") as fp:
    print(rsa.decrypt(fp.read(), prikey).decode())
```

#### 利用 `openssl`

```text
OpenSSL> rsautl -decrypt -in test.enc -inkey private.pem
```

### 字符串异或

#### `Crypto.Util import strxor`

```python
from Crypto.Util import strxor
strxor(b"hhh",b"aaa")
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
