
```python
import struct
import ctypes

algorithm_1 = {
    (0, "dec"): lambda x: (x + 28) & 0xff,
    (1, "dec"): lambda x: x ^ 0xd3,
    (2, "dec"): lambda x: (x + 94) & 0xff,
    (3, "dec"): lambda x: x ^ 0x86
}

def dec_xtea(input_low, input_high):
    v0, v1 = ctypes.c_uint32(input_low), ctypes.c_uint32(input_high)
    key = [0x7b777c63, 0xc56f6bf2, 0x2b670130, 0x76abd7fe]
    delta = 0x21524111
    total1 = ctypes.c_uint32(0xBEEFBEEF - 64 * delta)
    total2 = ctypes.c_uint32(0x9D9D7DDE - 64 * delta)
    for i in range(64):
        total1.value += delta
        total2.value += delta
        v1.value -= (((v0.value << 8) ^ (v0.value >> 7)) - v0.value) ^ (total2.value + key[(total2.value >> 13) & 3])
        v0.value -= (((v1.value << 7) ^ (v1.value >> 8)) + v1.value) ^ (total1.value - key[total1.value & 3])
    return v0.value, v1.value

def dec_vm_low(input):
    byte4, byte3, byte2, byte1 = struct.unpack(">4B", input.to_bytes(4, 'big'))
    byte1 = (byte1 ^ 0) + 27
    byte2 = (byte2 ^ 8) ^ 194
    byte3 = (byte3 ^ 16) - 168
    byte4 = (byte4 ^ 24) ^ 54
    input = struct.pack(">4B", byte1 & 0xff, byte2 & 0xff, byte3 & 0xff, byte4 & 0xff)
    return int.from_bytes(input, 'little')

def dec_vm_high(input):
    byte4, byte3, byte2, byte1 = struct.unpack(">4B", input.to_bytes(4, 'big'))
    byte1 = (byte1 - 0) + 47
    byte2 = (byte2 - 8) ^ 182
    byte3 = (byte3 - 16) - 55
    byte4 = (byte4 - 24) ^ 152
    input = struct.pack(">4B", byte1 & 0xff, byte2 & 0xff, byte3 & 0xff, byte4 & 0xff)
    return int.from_bytes(input, 'little')

def dec_2(input):
    input_byte = bytearray(input.to_bytes(4, 'big'))
    xor_arr = [50, -51, -1, -104, 25, -78, 0x7C, -102]
    for i in range(4):
        input_byte[i] = (input_byte[i] - i) & 0xff
        input_byte[i] = (input_byte[i] ^ xor_arr[i]) & 0xff

    input = int.from_bytes(input_byte, 'big')
    input = (input << 7 | input >> 25) & 0xffffffff
    input = int.from_bytes(input.to_bytes(4, 'little'), 'big') # 字节再次翻转

    return input

def dec_1(input):
    input_byte = bytearray(input.to_bytes(8, 'little'))
    for i in range(len(input_byte)):
        index = i % 4
        input_byte[i] = (algorithm_1[(index, "dec")]((input_byte[i] + 8 * index) & 0xff)) ^ index
    return int.from_bytes(input_byte, 'little')

token = int(input("enter the token plz~:"))
input_low, input_high = token, 0

# xtea
input_low, input_high = dec_xtea(input_low, input_high)

# vm
input_low, input_high = dec_vm_low(input_low), dec_vm_high(input_high)

# bswap32
input = struct.pack('<2I', input_low, input_high)
input_low, input_high = struct.unpack('>2I', input)

# 高/低32位对调
input_low, input_high = input_high, input_low

# blackObfuscator
input_high = dec_2(input_high)
input_low = dec_2(input_low)

# last decrypt
input = struct.pack('>2I', input_low, input_high)
input = int.from_bytes(input, 'little')
input = dec_1(input)

# output
print(input)
```

# 参考资料

- [ELF文件格式的详解](https://bbs.kanxue.com/elink@df3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3e0M7I4x3o6R3$3z5l9`.`.)

- [[原创]2023腾讯游戏安全竞赛初赛题解(安卓)](https://bbs.kanxue.com/thread-276949.htm#msg_header_h2_0)

- [[原创]2023腾讯游戏安全大赛-安卓赛道初赛wp](https://bbs.kanxue.com/thread-276896.htm#msg_header_h2_2)

- [[原创] 腾讯游戏安全技术竞赛2023 安卓客户端初赛WriteUp](https://bbs.kanxue.com/thread-276893.htm)

- [ARMv8(aarch64)指令集特性](https://bbs.kanxue.com/elink@c06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0x3^5y4U0j5&6y4e0j5I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6M7J5y4K6V1#2x3U0R3`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-11-17 18:48 被 oacia 编辑，原因：写了一个IDA插件实现trace so

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)
