
```asm
004275C1  |.  75 1E         |JNZ SHORT MA2_6.004275E1

004275C3  |.  EB 1B         |JMP SHORT MA2_6.004275E0

004275C5  |>  3BCF          |CMP ECX,EDI

004275C7  |.  75 0A         |JNZ SHORT MA2_6.004275D3

004275C9  |.  3C 6F         |CMP AL,6F                               ;  是 o 吗？

004275CB  |.  74 13         |JE SHORT MA2_6.004275E0

004275CD  |.  3C 4F         |CMP AL,4F                               ;  是 O 吗?

004275CF  |.  75 10         |JNZ SHORT MA2_6.004275E1

004275D1  |.  EB 0D         |JMP SHORT MA2_6.004275E0

004275D3  |>  83F9 03       |CMP ECX,3

004275D6  |.  75 09         |JNZ SHORT MA2_6.004275E1

004275D8  |.  3C 65         |CMP AL,65                               ;  是 e 吗?

004275DA  |.  74 04         |JE SHORT MA2_6.004275E0

004275DC  |.  3C 45         |CMP AL,45                               ;  是 E 吗？

004275DE  |.  75 01         |JNZ SHORT MA2_6.004275E1

004275E0  |>  42            |INC EDX

004275E1  |>  41            |INC ECX

004275E2  |.  83F9 05       |CMP ECX,5

004275E5  |.^ 7C AC         \JL SHORT MA2_6.00427593

004275E7  |.  83FA 05       CMP EDX,5

004275EA  |.  0F85 25010000 JNZ MA2_6.00427715                       ;  关键跳，跳就死
```

`004275EA`这里跳不跳都无所谓了。经过分析后得到注册码是20个字符,只要前5个字符是`COREK`后15个字符随意，但不能是空格字符就可以成功注册。

给出一个注册码:`COREKxxxxxxxxxxxxxxx`

[文件下载](<https://bbs.kanxue.com/elink@f0eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6e0R3$3M7%4N6Q4x3X3g2U0L8$3#2Q4x3V1k6g2M7r3I4G2j5h3c8r3K9h3I4W2i4K6u0r3x3U0l9H3y4W2)9J5k6o6p5J5i4K6u0r3e0h3q4J5K9h3&6W2i4K6t1#2x3U0m8m8M7i4g2S2M7X3W2#2L8g2)9J5y4e0t1H3N6U0u0Q4x3X3f1$3i4K6t1#2x3U0S2Q4x3U0g2o6b7W2)9J5y4f1q4q4i4K6t1#2c8o6N6Q4x3U0g2q4y4g2)9J5y4f1t1&6i4K6t1#2c8p5c8Q4x3U0g2o6y4W2)9J5y4f1x3I4i4K6t1#2b7U0q4Q4x3U0g2m8x3#2)9J5y4e0t1&6i4K6u0W2M7X3q4J5i4K6g2X3i4K6g2X3i4K6g2X3x3U0l9H3y4U0p5J5x3K6p5I4z5o6p5K6x3K6V1#2x3U0M7#2y4#2)9J5k6i4u0S2M7R3..>)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
