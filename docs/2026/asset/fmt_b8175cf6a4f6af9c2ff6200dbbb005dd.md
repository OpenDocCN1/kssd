
## 4 漏洞利用

按如上分析此exploit不利用heap spray 即可bypass DEP&ASLR，是因为获取了mshtml模块地址和shellcode地址。

## 5 Crash info

```
0:005> g

(bc4.468): Access violation - code c0000005 (first chance)

First chance exceptions are reported before any exception handling.

This exception may be expected and handled.

eax=a7c7bb98 ebx=00143990 ecx=01b73254 edx=000e1860 esi=01b7e520 edi=80020003

eip=6a742ce6 esp=026a9ed4 ebp=026a9ee0 iopl=0         ov up ei ng nz na po cy

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010a83

mshtml!`string'+0x6:

6a742ce6 65007200        add     byte ptr gs:[edx],dh            gs:000e1860=03
```

## 6 POC

[2cfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0R3H3N6Y4g2D9i4K6u0W2j5$3!0E0i4K6u0r3K9h3f1^5i4K6u0r3N6$3W2F1y4#2)9J5c8Y4y4U0i4K6u0W2N6s2S2@1](https://bbs.kanxue.com/elink@e53K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0R3H3N6Y4g2D9i4K6u0W2j5$3!0E0i4K6u0r3K9h3f1^5i4K6u0r3N6$3W2F1y4#2)9J5c8Y4y4U0i4K6u0W2N6s2S2@1)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
