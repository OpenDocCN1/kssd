
# RtlCompressBuffer声明

<mark class="encrypted">191K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3e0s2W2e0K9r3q4J5K9#2)9J5c8Y4m8Q4x3V1j5I4y4K6R3@1z5o6M7J5y4q4)9J5k6h3S2@1L8h3H3`.</mark>

## 动态调试

在如下两个位置下断点

```
bp cldflt!HsmpRpCommitNoLock
bp cldflt!HsmpRpCommitNoLock+0x13de
```

运行poc，可以看到已经进入`HsmpRpCommitNoLock`函数

```
1: kd> g
Breakpoint 0 hit
cldflt!HsmpRpCommitNoLock:
fffff804`6f6a1e88 48895c2420 mov qword ptr [rsp+20h],rbx
```

继续运行，触发第二个断点

```
0: kd> g
Breakpoint 1 hit
cldflt!HsmpRpCommitNoLock+0x13de:
fffff804`6f6a3266 e81571faff call cldflt!memcpy (fffff804`6f64a380)
```

此时`memmove`已经被优化为`memcpy`，而要拷贝的长度为`0x3f94`，`dst`所在的堆大小为`0x4000`，`dst`指向偏移`0x74`处，最多有`0x3f8c`大小，所以`memcpy`拷贝时会越界写入8个字节，造成堆溢出。

```
1: kd> rr8
r8=0000000000003f94
1: kd> !pool rcx
Pool page ffffd980717f7074 region is Paged pool
*ffffd980717f7000 : large page allocation, tag is HsRp, size is 0x4000 bytes
Owning component : Unknown (update pooltag.txt)
```

继续运行，则在`memcpy`内部触发异常，因为尝试往未分配的内存里面写入00

```
0: kd> u
cldflt!memcpy+0x165:
fffff800`8186a4e5 0f2941f0 movaps xmmword ptr [rcx-10h],xmm0
0: kd> !pool rcx - 0x10
Pool page ffffe5028e4fa000 region is Paged pool
ffffe5028e4fa000 is not a valid large pool allocation, checking large session pool...
ffffe5028e4fa000 is not valid pool. Checking for freed (or corrupt) pool
Address ffffe5028e4fa000 could not be read. It may be a freed, invalid or paged out page
0: kd> rxmm0
mm0=0000000000000000
```

对应代码为

```c
if ( v25 )
*(\_OWORD \*)(v15 + v25 - 16) = *(\_OWORD \*)(v15 + v25 - 16 + v13);
*(\_\_m128 \*)(v15 - 0x10) = v14;
```

以下为调用栈

```
1: kd> k
# Child-SP RetAddr Call Site
00 fffffb8a`8a45e4f8 fffff804`63717f82 nt!DbgBreakPointWithStatus
01 fffffb8a`8a45e500 fffff804`63717566 nt!KiBugCheckDebugBreak+0x12
02 fffffb8a`8a45e560 fffff804`635fd747 nt!KeBugCheck2+0x946
03 fffffb8a`8a45ec70 fffff804`63638f6f nt!KeBugCheckEx+0x107
04 fffffb8a`8a45ecb0 fffff804`63430730 nt!MiSystemFault+0x1de5ff
05 fffffb8a`8a45edb0 fffff804`6360d1d8 nt!MmAccessFault+0x400
06 fffffb8a`8a45ef50 fffff804`6f64a4e1 nt!KiPageFault+0x358
07 fffffb8a`8a45f0e8 fffff804`6f6a326b cldflt!memcpy+0x161
08 fffffb8a`8a45f0f0 fffff804`6f6a983b cldflt!HsmpRpCommitNoLock+0x13e3
09 fffffb8a`8a45f230 fffff804`6f66f0d7 cldflt!HsmiOpUpdatePlaceholderDirectory+0x57f
0a fffffb8a`8a45f320 fffff804`6f674b65 cldflt!HsmFltProcessUpdatePlaceholder+0x443
0b fffffb8a`8a45f3d0 fffff804`6f6a4504 cldflt!HsmFltProcessHSMControl+0x3d5
0c fffffb8a`8a45f500 fffff804`647264cc cldflt!HsmFltPreFILE_SYSTEM_CONTROL+0x6a4
0d fffffb8a`8a45f5a0 fffff804`64725f7a FLTMGR!FltpPerformPreCallbacksWorker+0x36c
0e fffffb8a`8a45f6c0 fffff804`64725021 FLTMGR!FltpPassThroughInternal+0xca
0f fffffb8a`8a45f710 fffff804`6475ae2f FLTMGR!FltpPassThrough+0x541
10 fffffb8a`8a45f7a0 fffff804`63410665 FLTMGR!FltpFsControl+0xbf
11 fffffb8a`8a45f800 fffff804`6380142c nt!IofCallDriver+0x55
12 fffffb8a`8a45f840 fffff804`63801081 nt!IopSynchronousServiceTail+0x34c
13 fffffb8a`8a45f8e0 fffff804`638d9ed6 nt!IopXxxControlFile+0xc71
14 fffffb8a`8a45fa20 fffff804`63610ef5 nt!NtFsControlFile+0x56
15 fffffb8a`8a45fa90 00007ff9`c648d704 nt!KiSystemServiceCopyEnd+0x25
16 00000056`01aff5b8 00007ff6`5e59167f ntdll!NtFsControlFile+0x14
17 00000056`01aff5c0 00000000`000001bc 0x00007ff6`5e59167f
18 00000056`01aff5c8 00000000`00000000 0x1bc
```

![](<mark class="encrypted">8d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0K9o6x3#2N6r3&6#2N6q4)9J5k6i4y4A6N6r3g2Q4x3V1k6A6L8h3q4Y4k6i4y4Q4x3V1k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6x3$3x3o6x3$3i4K6u0V1g2$3W2F1k6r3!0%4M7#2)9J5c8U0y4Q4x3X3g2H3L8X3N6Q4x3U0V1`.</mark>

这里调用`HsmpRpValidateBuffer`

```
cldflt!HsmpRpCommitNoLock+0x573
fffff804`155823f6 e81539fdff call cldflt!HsmpRpValidateBuffer (fffff80415555d10)
fffff804`155823fb 8bc8 mov ecx, eax
```

这里调用`ExAllocatePoolWithTag`

```
cldflt!HsmpRpCommitNoLock+0x93D
PAGE:00000001C00727BE 48 FF 15 B3 61 FB FF call cs:\_\_imp\_ExAllocatePoolWithTag
PAGE:00000001C00727BE
PAGE:00000001C00727C5 ; 601: P = PoolWithTag;
PAGE:00000001C00727C5 0F 1F 44 00 00 nop dword ptr [rax+rax+00h] // 0x942
PAGE:00000001C00727CA 33 FF xor edi, edi
```

PoC会在过几天上传到GitHub

```
https://github.com/Chestnuts4/POC
```

### 小结

本次漏洞分析离不开业内前辈逆向得出的`_HSM_REPARSE_DATA`结构体信息，这个结构体微软没有公开的文档，相关资料也很少。目前只有这一个仓库有相关信息，向前辈致敬。

![[../../../images/vulneribility/CVE-2023-36036/7.png]]

这里引用一下前辈的主页。

><a href="elink@bb2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2*7k6i4A6#2L8r3q4Q4x3X3g2F1k6i4c8Q4x3V1k6W2L8W2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3L8h3q4A6L8W2)9J5k6h3S2@1L8h3H3`."><mark class="encrypted">cc8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2*7k6i4A6#2L8r3q4Q4x3X3g2F1k6i4c8Q4x3V1k6W2L8W2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3L8h3q4A6L8W2)9J5k6h3S2@1L8h3H3`.</mark></a>

整体来看，这个漏洞原理和触发方式较为简单，在使用`memcpy`之前没有校验长度，而修复也简单，再解压之前验证长度是否超过`0x4000`，超过则认为数据有错，进入到错误逻辑，从而在源头阻止了触发漏洞逻辑。

在漏洞修复处在修复上个整数下溢的漏洞时，开发人员只修复当时的整数下溢漏洞，没有去考虑长度会不会过长，某些程度来说这也是开发的粗心大意导致了这个漏洞留到现在。

在编写PoC参考了其他安全研究员已有的分析。

## 参考链接

><a href="elink@4f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3N6i4m8V1j5i4c8W2i4K6u0V1k6%4g2A6k6r3g2Q4x3V1k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6x3$3x3o6x3$3"><mark class="encrypted">6b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3N6i4m8V1j5i4c8W2i4K6u0V1k6%4g2A6k6r3g2Q4x3V1k6$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6x3$3x3o6x3$3</mark></a>

><a href="elink@cfaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3&6x3U0p5&6y4o6b7$3y4l9`.`."><mark class="encrypted">9fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3&6x3U0p5&6y4o6b7$3y4l9`.`.</mark></a>

><a href="elink@368K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3V1k6i4K9h3&6V1L8%4N6K6i4K6u0V1j5$3I4S2M7%4y4A6j5#2)9J5k6s2y4S2L8i4m8D9k6i4y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8W2y4S2L8i4m8D9k6i4y4Q4x3V1k6o6L8r3!0#2k6p5#2A6M7Y4u0G2M7R3`.`."><mark class="encrypted">7faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3V1k6i4K9h3&6V1L8%4N6K6i4K6u0V1j5$3I4S2M7%4y4A6j5#2)9J5k6s2y4S2L8i4m8D9k6i4y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8W2y4S2L8i4m8D9k6i4y4Q4x3V1k6o6L8r3!0#2k6p5#2A6M7Y4u0G2M7R3`.`.</mark></a>

><a href="elink@fd1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6U0k6X3q4H3K9g2)9J5c8X3y4D9L8%4g2V1i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1M7X3g2X3k6i4u0W2L8X3y4W2"><mark class="encrypted">aa5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6U0k6X3q4H3K9g2)9J5c8X3y4D9L8%4g2V1i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1M7X3g2X3k6i4u0W2L8X3y4W2</mark></a>

><a href="elink@4e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6U0k6X3q4H3K9g2)9J5c8X3y4D9L8%4g2V1i4K6u0V1k6X3W2D9k6i4y4Q4x3X3c8X3N6h3&6U0N6r3W2G2L8Y4x3`."><mark class="encrypted">6c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6U0k6X3q4H3K9g2)9J5c8X3y4D9L8%4g2V1i4K6u0V1k6X3W2D9k6i4y4Q4x3X3c8X3N6h3&6U0N6r3W2G2L8Y4x3`.</mark></a>
