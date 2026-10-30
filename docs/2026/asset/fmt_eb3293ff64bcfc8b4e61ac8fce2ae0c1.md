
对应`OSI L1`，通常表现成射频`RF IP`。该射频`IP`采用零中频`Zero-IF`收发器或低中频`Low-IF`收发器，用`PLL`锁相环进行频率合成，`PLL`锁相环又分为模拟锁相环、数字锁相环和全数字锁相环。难点在于蓝牙双模的经典蓝牙的解调，经典蓝牙有79个信道，信道分为若干个625μs时隙，跳频时如果锁相环只有单一子频段，频率合成锁定时通过二分法搜索进行在线频率校准标定则至少需要7次，则锁相环的频率锁定时间+通信时间至少需要625us/7，大概90us，单一子频段的数字锁相环不知道能不能满足。为了支持经典蓝牙的`EDR 2`和`EDR 3`解调，物理层基带解调需要支持相干解调、星座解映射和差分解码等。射频`RF IP`最好是低中频收发器，灵活性高。

`CEVA`本来自己是有蓝牙`IP`的，射频`IP`方面应该是采用`Orca Systems`的，不知道有没有官方实现，收购了`RivieraWaves`后，不仅带来了新的蓝牙`Bluetooth IP`，而且射频`RF IP`方面有官方实现，也有射频`RF IP`官方合作厂商：`CSEM`、`Aura Semi`和`Sirius Wireless`，当然也可以用其它第三方射频`RF IP`厂商，比如`Catena`的`Bluetooth`双模`RF IP`与`Wi-Fi`双频`RF IP`配套搭配使用。一些射频`RF IP`架构图如下：

- `CSEM ICYTRX`:

   ![CSEM icytrx](https://bbs.kanxue.com/upload/attach/202303/422740_4TMAGMFB2Y5BF78.png)

- `Aura Semi AU50xx`

   ![Aura Semi](https://bbs.kanxue.com/upload/attach/202303/422740_V9G8MKP33FK2TUE.png)

- `NXP QN9090`

   ![](https://bbs.kanxue.com/upload/attach/202303/422740_ZMJ2RMPP2V488Q5.jpg)

### Physical Layer Register

重点关注`CSEM`，射频`RF IP`有`ICYTRX`和`ICYTRXDM`两种：`ICYTRX IP`是用65nm/55nm的台积电或格芯半导体工艺制作，`BLE`单模，有多个版本；`ICYTRXDM IP`是用40nm/22nm的台积电或格芯半导体工艺制作，`BT`双模，可配置成`BLE`单模。

#### ICYTRX GCS1

`CSEM`射频`RF IP`寄存器描述粗略表示如下(详情请查看`RSL10`的`SVD`文件和用户手册)：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70 | ```// 基于RSL10 svd2rust(rsl10-pac) typedef struct icytrx_gcs1_type { REG32 reg00; //0x00000000 REG32 reg01; //0x00000004 REG32 reg02; //0x00000008 REG32 reg03; //0x0000000C REG32 reg04; //0x00000010 REG32 reg05; //0x00000014 REG32 center_freq; //0x00000018 REG32 reg07; //0x0000001C REG32 reg08; //0x00000020 REG32 reg09; //0x00000024 REG32 reg0A; //0x00000028 REG32 sync_pattern; //0x0000002C REG32 reg0C; //0x00000030 REG32 crc_polynomial; //0x00000034 REG32 crc_rst; //0x00000038 REG32 reg0F; //0x0000003C REG32 reg10; //0x00000040 REG32 tx_pulse0; //0x00000044 REG32 tx_pulse1; //0x00000048 REG32 tx_pulse2; //0x0000004C REG32 tx_pulse3; //0x00000050 REG32 rx_pulse; //0x00000054 REG32 reg16; //0x00000058 REG32 reg17; //0x0000005C REG32 reg18; //0x00000060 REG32 reg19; //0x00000064 REG32 reg1A; //0x00000068 REG32 reg1B; //0x0000006C REG32 agc_lut1; //0x00000070 REG32 agc_lut2; //0x00000074 REG32 agc_lut3; //0x00000078 REG32 agc_lut4; //0x0000007C REG32 reg20; //0x00000080 REG32 agc_att1; //0x00000084 REG32 reg22; //0x00000088 REG32 reg23; //0x0000008C REG32 reg24; //0x00000090 REG32 reg25; //0x00000094 REG32 reg26; //0x00000098 REG32 reg27; //0x0000009C REG32 reg28; //0x000000A0 REG32 pll_ctrl; //0x000000A4 REG32 reg2A; //0x000000A8 REG32 xtal_ctrl; //0x000000AC REG32 reg2C; //0x000000B0 REG32 reg2D; //0x000000B4 REG32 reg2E; //0x000000B8 REG32 reg2F; //0x000000BC REG32 reg30; //0x000000C0 REG32 reg31; //0x000000C4 REG32 reg32; //0x000000C8 REG32 txfifo; //0x000000CC REG32 rxfifo; //0x000000D0 REG32 deser_status; //0x000000D4 REG32 irq_status; //0x000000D8 REG32 reg37; //0x000000DC REG32 reg38; //0x000000E0 REG32 reg39; //0x000000E4 REG32 resevered_000000E8; //0x000000E8 REG32 resevered_000000EC; //0x000000EC REG32 resevered_000000F0; //0x000000F0 REG32 resevered_000000F4; //0x000000F4 REG32 resevered_000000F8; //0x000000F8 REG32 revision; //0x000000FC } ICYTRX_GCS1_TypeDef;``` |

#### ICYTRX GCS2

`CSEM`射频`RF IP`寄存器描述粗略表示如下(详情请查看`RSL15`的`SVD`文件和用户手册):

|  |  |
| --- | --- |
