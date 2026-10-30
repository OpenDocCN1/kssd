
```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66 | `union`  `{`  `uint32_t R;`  `struct`  `{`  `reg32 PACKET_EXTRA_PATTERN_WORD_LEN:` `2`;  `reg32 PACKET_EXTRA_PATTERN_MAX_ERR:` `2`;  `reg32 PACKET_EXTRA_PKT_INFO_PRE_NPOST:` `1`;  `reg32 PACKET_EXTRA_EN_STOP_WORD:` `1`;  `reg32 PACKET_EXTRA_STOP_WORD_LEN:` `2`;  `reg32 CONV_CODES_CONF_EN_CONV_CODE:` `1`;  `reg32 CONV_CODES_CONF_CC_EN_TX_STOP:` `1`;  `reg32 CONV_CODES_CONF_CC_VITERBI_LEN:` `2`;  `reg32 reserved_12_15:` `4`;  `reg32 CONV_CODES_POLY_CC_POLY_0:` `5`;  `//` `bit[` `16` `:` `20` `].polynomial of the first convolutional code`  `reg32 CONV_CODES_POLY_CC_POLY_1:` `5`;  `//` `bit[` `20` `:` `25` `].polynomial of the second convolutional code`  `reg32 CONV_CODES_POLY_CC_POLY_2:` `5`;  `} B;`  `} RF_REG0C;`  `//` `RF_REG0C.CONV_CODES_POLY_CC_POLY_0` `=` `0b1111` `(x^` `3` `+` `x^` `2` `+` `x^` `1` `+` `x^` `0` `)`  `//` `RF_REG0C.CONV_CODES_POLY_CC_POLY_1` `=` `0b1101` `(x^` `3` `+` `x^` `2`  `+` `x^` `0` `)`  `//` `S` `=` `2`  `//` `S` `=` `8`    `union`  `{`  `uint32_t R;`  `struct`  `{`  `reg32 CONV_CODES_PUNCT_CC_PUNCT_0:` `5`;  `//` `bit[` `0` `:` `4` `].puncture of the first convolutional code`  `reg32 CONV_CODES_PUNCT_CC_PUNCT_1:` `5`;  `//` `bit[` `5` `:` `9` `].puncture of the second convolutional code`  `reg32 CONV_CODES_PUNCT_CC_PUNCT_2:` `5`;  `reg32 reserved_15_15:` `1`;  `reg32 FRAC_CONF_RX_EN_FRAC:` `1`;  `reg32 FRAC_CONF_TX_EN_FRAC:` `1`;  `reg32 FRAC_CONF_RX_FRAC_GAIN:` `1`;  `reg32 FRAC_CONF_TX_FRAC_GAIN:` `1`;  `reg32 reserved_20_23:` `4`;  `reg32 RX_FRAC_CONF_RX_FRAC_NUM:` `4`;  `reg32 RX_FRAC_CONF_RX_FRAC_DEN:` `4`;  `} B;`  `} RF_REG0F;`  `//` `RF_REG0F.CONV_CODES_PUNCT_CC_PUNCT_0` `=` `0b111` `;???`  `//` `RF_REG0F.CONV_CODES_PUNCT_CC_PUNCT_1` `=` `0b001` `;???`    `union`  `{`  `uint32_t R;`  `struct`  `{`  `reg32 DECISION_EN_VITERBI_GFSK:` `1`;  `//` `bit[` `0` `:` `0` `].` `0` `:disable;` `1` `:enable`    `reg32 DECISION_VITERBI_POW_NLIN:` `1`;  `reg32 DECISION_VITERBI_LEN:` `2`;  `reg32 DECISION_USE_VIT_SOFT:` `1`;  `reg32 reserved_05_07:` `3`;  `reg32 RSSI_BANK_TAU_RSSI_FILTERING:` `4`;  `reg32 RSSI_BANK_EN_FAST_PRE_SYNC:` `1`;  `reg32 RSSI_BANK_FAST_RSSI:` `1`;  `reg32 RSSI_BANK_RSSI_TRI_CK_DIV:` `2`;  `reg32 CORRECT_CFREQ_IF_CORRECT_CFREQ_IF:` `16`;  `} B;`  `} RF_REG18;`  `//` `RF_REG18.DECISION_EN_VITERBI_GFSK` `=` `1` `;`  `//` `S` `=` `2`  `//` `S` `=` `8` |
```

## L4

对应OSI L6，通常表现成蓝牙Bluetooth HCI，类比无线通信IP的RIL(Radio Interface Layer)，未关注。

## L5

对应OSI L7，通常表现成蓝牙Bluetooth Host，类比无线通信IP的AP(Application Processor)。如果某蓝牙芯片能够直接通过总线访问射频RF IP寄存器和蓝牙Bluetooth IP链路层寄存器，该寄存器部分有详细的定义和描述，并且此部分在用户手册中有上千页文档，那用这芯片结合蓝牙官方标准文档来实现蓝牙渗透工具就非常完美，可惜不大可能，其次是在该芯片的Bluetooth Controller实现蓝牙渗透功能。

推荐相对比较好实现蓝牙渗透工具的蓝牙芯片:

### BLE

#### ON Semiconductor

`RSL10`/`RSL15`的蓝牙功能不知道是不是基于CEVA 蓝牙IP的参考设计改进实现，`RSL15`可能是为了支持3个BLE ISO Channel，使用3个射频IP，刚好对应了BLE的3个广播信道，挺不错。相关文档比较详细。

#### NXP

`MKW39`/`K32W1`使用收购来的Freescale射频IP，使用Mindtree蓝牙IP，该芯片文档应该是唯一最详细的；`QN9090`/`K32W0`不知道是不是使用Catena的射频收发器IP，物理层基带基于收购而来的Quintic物理层基带改进，使用CEVA蓝牙IP，相关文档不太详细。

### BTDM

#### Bouffalo Lab

`BL618`/`BL606P`的射频IP应该是自研的，官方文档说明支持经典蓝牙的EDR 2和EDR 3，相关的RF IP寄存器的描述和定义未知，采用CEVA蓝牙IP，蓝牙 Controller实现用闪存存储。

#### Beken Corporation

`BK3288`和`BK7256`的射频IP应该是自研的，官方文档说明支持经典蓝牙的EDR 2和EDR 3，相关的RF IP寄存器的描述和定义未知，采用CEVA蓝牙IP，蓝牙 Controller实现用闪存存储。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2023-4-13 06:38

被vasthao编辑，原因：

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#无线电安全](https://bbs.kanxue.com/forum-128-1-177.htm)
