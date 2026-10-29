
```java
Object[] result = new Object[2];
byte[] rc4 = new byte[16];
long local_1 = fix.length, local_4 = 1, local_5 = 0, local_6 = 0;
long local_7 = fix.length << 1, local_8 = 0, local_9, local_A = 0, local_B = 0;
int local_C = 0;
for (local_C = 0; local_C < local_1; local_C++) {
    local_5 = fix[local_C] & 0xFFL;
    local_6 = local_4 << 5;
    local_4 = local_6 - local_4;
    local_6 = local_C & 0xFL;
    local_4 += local_5;
    local_4 &= 0xFFL;
    rc4[(int) local_6] = (byte) (local_5 & 0xFFL);
}
local_C = 0;
local_6 = xor;
local_5 = (local_7 + (local_7 << 1)) & 0xFFL;
local_9 = local_C + local_4 + local_7;
for (local_C = 0; local_C < local_1; local_C++) {
    local_7 = local_C & 0xFL;
    local_8 = rc4[(int) local_7] & 0xFFL;
    local_6 = (local_4 ^ local_6) & 0xFFFFFFFFL;
    local_A = fix[local_C] & 0xFFL;
    local_B = local_8 & 0xFL;
    local_B = (local_4 ^ local_B) & 0xFFFFFFFFL;
    local_8 = (local_4 ^ (local_8 >> 4)) & 0xFFFFFFFFL;
    local_8 = (local_8 | local_B) & 0xFFFFFFFFL;
    local_6 = (local_6 ^ local_8) & 0xFFFFFFFFL;
    rc4[(int) local_7] = (byte) (local_6 & 0xFFL);
    local_8 = xor;
    local_7 = (local_A ^ local_C) & 0xFFFFFFFFL;
    local_7 = (local_7 ^ local_5) & 0xFFFFFFFFL;
    local_7 = local_7 & 0xFFL;
    local_6 = (local_9 + local_C) & 0xFFFFFFFFL;
    local_6 = (local_6 ^ local_8) & 0xFFFFFFFFL;
    local_5 += 3;
    local_6 = (local_6 ^ local_7) & 0xFFFFFFFFL;
    local_5 &= 0xFFL;
    xor = local_6;
}
result[0] = rc4;
result[1] = xor;
return result;
}
```

3. 依次读取每个DEX的数据，详细结构：`dexindex+next+data1size+data1+data2`

解密流程：

```java
result = new byte[(int) size][];
long dex_off = 4;
for (int i = 0; i < size; i++) {
    // next_off, dex_size, dex[]
    long next_off = off + dex_off;
    int size_1 = (int) ByteToDword(dex, next_off + 4);
    long size_2 = ByteToDword(dex, next_off);

    byte[] dex_data = memcpy(dex, next_off + 8, size_1);
    byte[] dex_data2 = memcpy(dex, next_off + 8 + size_1, size_2 - size_1 - 4);

    byte[] key = init_rc4(rc4, rc4.length);
    dex_rc4_decode(key, dex_data, size_1);

    dex_data = LZMA(dex_data);
    if (dex_data == null || dex_data.length == 0)
        continue;

    for (int j = 0; j < 0x70; j++)
        dex_data[j] = (byte) (((dex_data[j] & 0xFFL) ^ (xor & 0xFFL)) & 0xFFL);

    // 解密剩余的部分
    // 2个解密后的byte[]合并成一个行的byte[]
    byte[] dex_3 = new byte[dex_data.length + dex_data2.length];
    System.arraycopy(dex_data, 0, dex_3, 0, dex_data.length);
    System.arraycopy(dex_data2, 0, dex_3, dex_data.length, dex_data2.length);

    result[i] = dex_3;
    dex_off += size_2 + 4;
}
```

用`makekey`计算出来的`rc4key`先解密，然后`LZMA:24`解压，再把前`0x70`个字节和`makekey`的第二个`KEY`异或，最后和`dex_data2`合并就得到完整的`dex`。

其余的代码很好逆，菜鸟代码写的比较挫就不献丑了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
