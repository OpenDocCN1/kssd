
### AMD defined CPU features

```c
/* AMD-defined CPU features, CPUID level 0x80000001, word 1 */
/* Don't duplicate feature flags which are redundant with Intel! */
#define X86_FEATURE_SYSCALL        ( 1*32+11) /* SYSCALL/SYSRET */
#define X86_FEATURE_MP            ( 1*32+19) /* MP Capable */
#define X86_FEATURE_NX            ( 1*32+20) /* Execute Disable */
#define X86_FEATURE_MMXEXT        ( 1*32+22) /* AMD MMX extensions */
#define X86_FEATURE_FXSR_OPT        ( 1*32+25) /* FXSAVE/FXRSTOR optimizations */
#define X86_FEATURE_GBPAGES        ( 1*32+26) /* "pdpe1gb" GB pages */
#define X86_FEATURE_RDTSCP        ( 1*32+27) /* RDTSCP */
#define X86_FEATURE_LM            ( 1*32+29) /* Long Mode (x86-64, 64-bit support) */
#define X86_FEATURE_3DNOWEXT        ( 1*32+30) /* AMD 3DNow extensions */
#define X86_FEATURE_3DNOW        ( 1*32+31) /* 3DNow */
```

`_PAGE_NX`定义：

```c
#if defined(CONFIG_X86_64) || defined(CONFIG_X86_PAE)
#define _PAGE_NX    (_AT(pteval_t, 1) << _PAGE_BIT_NX)
#define _PAGE_DEVMAP    (_AT(u64, 1) << _PAGE_BIT_DEVMAP)
#define __HAVE_ARCH_PTE_DEVMAP
#else
#define _PAGE_NX    (_AT(pteval_t, 0))
#define _PAGE_DEVMAP    (_AT(pteval_t, 0))
#endif
```

`_PAGE_BIT_NX`定义语句为`#define _PAGE_BIT_NX 63`，与Intel SDM中描述一致。`__supported_pte_mask`会在`massage_pgprot`函数中使用：

```c
static inline pgprotval_t massage_pgprot(pgprot_t pgprot)
{
pgprotval_t protval = pgprot_val(pgprot);
    if (protval & _PAGE_PRESENT)
protval &= __supported_pte_mask;

    return protval;
}

static inline pte_t pfn_pte(unsigned long page_nr, pgprot_t pgprot)
{
    return __pte(((phys_addr_t)page_nr << PAGE_SHIFT) |
    massage_pgprot(pgprot));
}

static inline pmd_t pfn_pmd(unsigned long page_nr, pgprot_t pgprot)
{
    return __pmd(((phys_addr_t)page_nr << PAGE_SHIFT) |
    massage_pgprot(pgprot));
}

static inline pud_t pfn_pud(unsigned long page_nr, pgprot_t pgprot)
{
    return __pud(((phys_addr_t)page_nr << PAGE_SHIFT) |
    massage_pgprot(pgprot));
}
```

## 0x03 checksec

2.4版本的checksec对NX的检查一方面是文件：

```bash
# check for NX support
${debug} && echo -e "\n***function filecheck->nx"
if ${readelf} -l "${1}" 2>/dev/null | grep -q 'GNU_STACK'; then
if ${readelf} -l "${1}" 2>/dev/null | grep 'GNU_STACK' | grep -q 'RWE'; then
echo_message '\033[31mNX disabled\033[m   ' 'NX disabled,' ' nx="no"' '"nx":"no",'
else
echo_message '\033[32mNX enabled \033[m   ' 'NX enabled,' ' nx="yes"' '"nx":"yes",'
fi
else
echo_message '\033[31mNX disabled\033[m   ' 'NX disabled,' ' nx="no"' '"nx":"no",'
fi
```

一方面是`/proc/cpuinfo`：

```bash
# check cpu nx flag
nxcheck() {
${debug} && echo -e "\n***function nxcheck"
if grep -qFw nx /proc/cpuinfo; then
echo_message '\033[32mYes\033[m\n\n' ' ' ' ' ' '
else
echo_message '\033[31mNo\033[m\n\n' ' ' ' ' ' '
fi
}
```

## 0x04 参阅链接

- [NX(No-eXecute)的实现分析](https://bbs.kanxue.com/elink@7dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4u0V1k6h3&6W2k6r3I4A6L8Y4g2^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8U0t1H3x3e0k6Q4x3V1j5H3y4W2)9J5c8U0l9I4i4K6u0r3e0W2S2Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6h3S2@1L8h3H3`.)

- [Overall Options](https://bbs.kanxue.com/elink@370K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8Y4g2Q4x3X3g2G2M7X3N6Q4x3V1k6G2L8X3I4A6L8X3g2V1L8$3y4K6i4K6u0r3k6$3y4U0i4K6u0V1z5g2)9J5k6e0y4Q4x3X3f1H3i4K6u0r3k6$3y4U0i4K6u0r3e0%4k6W2M7X3q4D9L8q4)9J5k6p5!0H3N6r3W2G2L8Y4y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6e0%4k6W2M7X3q4D9L8q4)9J5k6p5!0H3N6r3W2G2L8Y4x3`.)

- [Link-Options](https://bbs.kanxue.com/elink@0d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8Y4g2Q4x3X3g2G2M7X3N6Q4x3V1k6G2L8X3I4A6L8X3g2V1L8$3y4K6i4K6u0r3k6$3y4U0i4K6u0V1z5g2)9J5k6e0y4Q4x3X3f1H3i4K6u0r3k6$3y4U0i4K6u0r3e0r3W2F1K9#2)9J5k6p5!0H3N6r3W2G2L8Y4y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6e0r3W2F1K9#2)9J5k6p5!0H3N6r3W2G2L8Y4x3`.)

- [Collect2](https://bbs.kanxue.com/elink@9ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3y4Q4x3X3g2Y4L8Y4g2Q4x3X3g2G2M7X3N6Q4x3V1k6G2L8X3I4A6L8X3g2V1L8$3y4K6i4K6u0r3k6$3y4U0K9h3&6@1i4K6u0r3b7$3!0D9L8r3g2U0N6o6u0Q4x3X3g2Z5N6r3#2D9)

- [getopt_long_only(3) - Linux man page](https://bbs.kanxue.com/elink@367K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6#2P5q4)9J5k6h3c8A6k6g2)9J5k6h3&6W2N6q4)9J5c8X3#2S2L8W2)9J5c8U0y4Q4x3V1k6Y4k6i4c8G2M7s2c8Q4y4h3k6D9L8$3&6Y4i4K6g2X3L8$3&6D9P5b7`.`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-11-29 14:56

被erfze编辑，原因：

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
