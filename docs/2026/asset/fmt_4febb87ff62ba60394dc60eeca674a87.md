
4. 要求新case不能被bitfilp和arith生成过

|  |  |
| --- | --- |
| 1 2 | `if` `(could_be_bitflip(orig ^ (u8)interesting_8[j]) ||` `could_be_arith(orig, (u8)interesting_8[j],` `1``))` |

5. 朴实无华的执行并恢复原case

|  |  |
| --- | --- |
| 1 2 3 4 | `stage_cur_val` `=` `interesting_8[j];` `out_buf[i]` `=` `interesting_8[j];` `if` `(common_fuzz_stuff(argv, out_buf,` `len``)) goto abandon_entry;` `out_buf[i]` `=` `orig;` |

### interest 16/8

每次对16bit进行替换变异，步长8bit

### interest 32/8

每次对32bit进行替换变异，步长8bit

## **DICTIONARY STUFF阶段**

用户提供的字典里有token，用来替换要进行变异的文件内容，如果用户没提供就使用 bitflip 自动生成的 token

### user extras (over)

以8bit为步长，标记起始位置开始，替换为token

1. 每个字节都替换一遍

|  |  |
| --- | --- |
| 1 | `for` `(i` `=` `0``; i <` `len``; i``+``+``)` |

2. 遍历用户字典

|  |  |
| --- | --- |
| 1 | `for` `(j` `=` `0``; j < extras_cnt; j``+``+``)` |

3. 出现以下情况，直接下一条token

|  |  |
| --- | --- |
| 1 2 3 4 | `if` `((extras_cnt > MAX_DET_EXTRAS && UR(extras_cnt) >``=` `MAX_DET_EXTRAS) ||` `extras[j].``len` `>` `len` `-` `i ||` `!memcmp(extras[j].data, out_buf` `+` `i, extras[j].``len``) ||` `!memchr(eff_map` `+` `EFF_APOS(i),` `1``, EFF_SPAN_ALEN(i, extras[j].``len``)))` |

- 字典token数>200，随机生成一个小于字典token数，仍>=200
- 替换token后长度超过case原大小
- case中数据与token一致
- `eff_map`为0

4. 替换，执行

|  |  |
| --- | --- |
| 1 2 3 | `last_len` `=` `extras[j].``len``;` `memcpy(out_buf` `+` `i, extras[j].data, last_len);` `if` `(common_fuzz_stuff(argv, out_buf,` `len``))` |

5. 所有token结束后恢复，跳回步骤1

|  |  |
| --- | --- |
| 1 | `memcpy(out_buf` `+` `i, in_buf` `+` `i, last_len);` |

### user extras (insert)

以8bit为步长，标记起始位置插入token

### auto extras (over)

以8bit为步长，标记起始位置开始，替换为在bitflip阶段生成的token

这是*deterministic steps*的最后一步

|  |  |
| --- | --- |
| 1 | `if` `(!queue_cur``-``>passed_det) mark_as_det_done(queue_cur);` |

我们可以在这里设置完成状态

## **RANDOM HAVOC阶段**

进行很大程度的杂乱破坏，随机组合，规则比较杂，但目的一致

## **SPLICING阶段**

通过将两个case按一定规则进行拼接，得到一个新case

HAVOC和SPLICING是相结合的，拼接case后会回到havoc进行随机变异

参考文章：

1. AFL源码阅读笔记之gcc与fuzz部分https://bbs.pediy.com/thread-265936.htm

2. AFL 源码分析43aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2L8X3N6Q4y4h3k6D9k6h3g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3z5o6t1@1y4o6j5J5y4H3`.

3. 漏洞挖掘技术之 AFL 项目分析https://bbs.pediy.com/thread-249912.htm

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-7-25 18:27

被Azyka编辑，原因： 增加图片引用

[#Fuzz](https://bbs.kanxue.com/forum-150-1-157.htm)

[#Linux](https://bbs.kanxue.com/forum-150-1-161.htm)
