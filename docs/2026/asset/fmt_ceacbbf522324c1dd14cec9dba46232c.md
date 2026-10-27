
```
len= 17

key=

00000000: 90 A0 21 50 4F 8F FB FF  85 83 CF 56 41 12 7A F9  ..!PO......VA.z.

00000010: 31                                                1

plain=

00000000: 4D 72 2E 20 53 68 65 72  6C 6F 63 6B 20 48 6F 6C  Mr. Sherlock Hol

00000010: 6D 65 73 2C 20 77 68 6F  20 77 61 73 20 75 73 75  mes, who was usu

00000020: 61 6C 6C 79 20 76 65 72  79 20 6C 61 74 65 20 69  ally very late i

00000030: 6E 20 74 68 65 20 6D 6F  72 6E 69 6E 67 73 2C 20  n the mornings,

00000040: 73 61 76 65 0D 0A 75 70  6F 6E 20 74 68 6F 73 65  save..upon those

00000050: 20 6E 6F 74 20 69 6E 66  72 65 71 75 65 6E 74 20   not infrequent

00000060: 6F 63 63 61 73 69 6F 6E  73 20 77 68 65 6E 20 68  occasions when h

00000070: 65 20 77 61 73 20 75 70  20 61 6C 6C 20 6E 69 67  e was up all nig

00000080: 68 74 2C 20 77 61 73 20  73 65 61 74 65 64 0D 0A  ht, was seated..

00000090: 61 74 20 74 68 65 20 62  72 65 61 6B 66 61 73 74  at the breakfast

000000A0: 20 74 61 62 6C 65 2E 20  49 20 73 74 6F 6F 64 20   table. I stood

000000B0: 75 70 6F 6E 20 74 68 65  20 68 65 61 72 74 68 2D  upon the hearth-

000000C0: 72 75 67 20 61 6E 64 20  70 69 63 6B 65 64 20 75  rug and picked u

000000D0: 70 20 74 68 65 0D 0A 73  74 69 63 6B 20 77 68 69  p the..stick whi

000000E0: 63 68 20 6F 75 72 20 76  69 73 69 74 6F 72 20 68  ch our visitor h

000000F0: 61 64 20 6C 65 66 74 20  62 65 68 69 6E 64 20 68  ad left behind h

00000100: 69 6D 20 74 68 65 20 6E  69 67 68 74 20 62 65 66  im the night bef

00000110: 6F 72 65 2E 20 49 74 20  77 61 73 20 61 0D 0A 66  ore. It was a..f

00000120: 69 6E 65 2C 20 74 68 69  63 6B 20 70 69 65 63 65  ine, thick piece

00000130: 20 6F 66 20 77 6F 6F 64  2C 20 62 75 6C 62 6F 75   of wood, bulbou

00000140: 73 2D 68 65 61 64 65 64  2C 20 6F 66 20 74 68 65  s-headed, of the

00000150: 20 73 6F 72 74 20 77 68  69 63 68 20 69 73         sort which is

```

因此，这就是正确的key。

不用说，输入的数据越大越好。这个350字节的文件实际上是另一个更大加密文件的前戏([`cipher2.txt`](<https://bbs.kanxue.com/elink@663K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6h3&6F1K9i4y4k6N6i4u0A6j5$3S2W2N6W2)9J5c8Y4W2#2M7X3W2U0K9r3g2$3i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6K9x3#2)9J5c8X3y4A6M7r3S2W2M7U0u0Q4x3X3g2@1P5s2b7>), 12903 bytes) ，这个大文件的key可以不用添加那些些启发式条件也能求得。

SMT Solver 可以秒杀这些东西，我曾经很天真的解决了这些问题，这里有一个更快的版本： [4b8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6E0j5i4y4C8i4K6g2X3x3W2)9J5c8R3`.`.](<https://bbs.kanxue.com/elink@bcbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6i4u0A6j5$3S2W2N6W2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6j5e0#2u0Q4y4h3k6E0j5i4y4C8i4K6g2X3x3W2)9J5c8R3`.`.). 尽管如此，这仍然是另一个优化问题的演示。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](<https://www.kanxue.com/book-section_list-227.htm>)
