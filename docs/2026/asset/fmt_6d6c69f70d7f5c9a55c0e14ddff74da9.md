
```
/* Scan the shellcode for suitable encode byte */

for (i = 1; i < 255; i++) {
    /* Mark the candidate as found */
    found = 1;

    /* Try out every byte in encode_bytes array */
    for (j = 0; j < size; j++) {
        /* XOR candidate with encode_bytes */
        try = i^encode_bytes[j];

        /* Now try every byte in shellcode */
        for (k = 0; k < len; k++) {
            c = 0xff&filebuf[k];

            /* If match, mark the candidate "not found" */
            if (c == try) {
                found = 0;
                break;
            }
        }
    }

    /* Break out if byte found */
    if (found == 1) break;
}
```

创建新的shellcode，只取jump传统解码方式的来解读一下

```
/* Create the new shellcode */

/* jump/call decoder */

if (decoder == DECODER_JUMP) {
    /* Copy 1st header */
    memcpy(&shellbuf[0], decoder_jump_header,
           sizeof(decoder_jump_header)); //拷贝解码字节

    /* Copy the actual shellcode */
    for (i = 0; i < len; i++)
        filebuf[i] = filebuf[i]^encode_byte;
    memcpy(&shellbuf[sizeof(decoder_jump_header)],
           filebuf, len); //拷贝经过xor处理后的实际shellcode

    /* High len byte: */ //处理字节数
    shellbuf[9] = ((0x10000-len)>>8)&0xff;

    /* Low len byte: */
    shellbuf[8] = (0x10000-len)&0xff;
    /* Adjust, if needed.. */
    if (shellbuf[8] == 0)
        shellbuf[8]++;

    /* XOR byte */
    shellbuf[12] = encode_byte; //将0x99修改为xor使用的字节

    /* Decoder register */ //使用不同寄存器的情况下，修改相应的字节，例如ebx的话第三个字节0x5b，第11个字节为0x33，第13个字节为0x43，这些在反汇编下很容易看出来
    switch (reg) {
        case REG_EAX:
            shellbuf[2] = 0x58;
            shellbuf[11] = 0x30;
            shellbuf[13] = 0x40;
            break;
        case REG_EDX:
            shellbuf[2] = 0x5a;
            shellbuf[11] = 0x32;
            shellbuf[13] = 0x42;
            break;
        case REG_EBX:
            shellbuf[2] = 0x5b;
            shellbuf[11] = 0x33;
            shellbuf[13] = 0x43;
            break;
    }
}
```

到此为此基本了解了这个xor编码解码的原理。这是一种防止限制shellcode个别字符使用的，以及防止IDS/IPS检测等等的方法。

## 05 参考

`878K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3E0D9j5h3E0W2i4K6u0W2L8%4u0Y4i4K6u0r3i4K6N6q4K9Y4c8Q4x3V1k6W2L8X3y4G2k6r3g2J5i4K6u0r3`

`f1dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3P5h3y4Z5i4K6g2X3L8h3q4^5i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5^5x3e0f1^5x3K6t1`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
