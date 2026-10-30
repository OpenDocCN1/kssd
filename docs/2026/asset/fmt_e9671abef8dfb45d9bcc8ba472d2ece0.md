
```c
// 基于RSL15 SVD(montana.svd)
typedef struct icytrx_gcs2_type
{
    REG32 reg00; // 0x00000000
    REG32 reg01; // 0x00000004
    REG32 reg02; // 0x00000008
    REG32 reg03; // 0x0000000C
    REG32 pads_03; // 0x00000010
    REG32 pads_47; // 0x00000014
    REG32 center_freq; // 0x00000018
    REG32 pads_89; // 0x0000001C
    REG32 reg08; // 0x00000020
    REG32 coding; // 0x00000024
    REG32 packet_handling; // 0x00000028
    REG32 sync_pattern; // 0x0000002C
    REG32 reg0C; // 0x00000030
    REG32 packet_extra; // 0x00000034
    REG32 crc_polynomial; // 0x00000038
    REG32 crc_rst; // 0x0000003C
    REG32 reg10; // 0x00000040
    REG32 reg11; // 0x00000044
    REG32 tx_pulse_shape_1; // 0x00000048
    REG32 tx_pulse_shape_2; // 0x0000004C
    REG32 tx_pulse_shape_3; // 0x00000050
    REG32 tx_pulse_shape_4; // 0x00000054
    REG32 frontend; // 0x00000058
    REG32 rx_pulse; // 0x0000005C
    REG32 reg18; // 0x00000060
    REG32 reg19; // 0x00000064
    REG32 reg1A; // 0x00000068
    REG32 reg1B; // 0x0000006C
    REG32 rssi_ctrl; // 0x00000070
    REG32 reg1D; // 0x00000074
    REG32 agc_lut1; // 0x00000078
    REG32 agc_lut2; // 0x0000007C
    REG32 agc_lut3; // 0x00000080
    REG32 agc_lut4; // 0x00000084
    REG32 agc_lut5; // 0x00000088
    REG32 agc_att1; // 0x0000008C
    REG32 agc_att2; // 0x00000090
    REG32 reg25; // 0x00000094
    REG32 bias_0_2; // 0x00000098
    REG32 bias_3_6; // 0x0000009C
    REG32 bias_7_9; // 0x000000A0
    REG32 bias_10_12; // 0x000000A4
    REG32 reg2A; // 0x000000A8
    REG32 pll_ctrl; // 0x000000AC
    REG32 dll_ctrl; // 0x000000B0
    REG32 reg2D; // 0x000000B4
    REG32 reg2E; // 0x000000B8
    REG32 xtal_ctrl; // 0x000000BC
    REG32 subband; // 0x000000C0
    REG32 reg31; // 0x000000C4
    REG32 demod_ctrl; // 0x000000C8
    REG32 reg33; // 0x000000CC
    REG32 reg34; // 0x000000D0
    REG32 ble_lr; // 0x000000D4
    REG32 reg36; // 0x000000D8
    REG32 prot_timer; // 0x000000DC
    REG32 cte_opts; // 0x000000E0
    REG32 pt_delta_0; // 0x000000E4
    REG32 pt_delta_1; // 0x000000E8
    REG32 cte_if; // 0x000000EC
    REG32 cte_ctrl; // 0x000000F0
    REG32 agc_advanced; // 0x000000F4
    REG32 data_streaming; // 0x000000F8
    REG32 revision; // 0x000000FC
    REG32 fsm_ctrl; // 0x00000100
    REG32 iofifo_status; // 0x00000104
    REG32 txfifo; // 0x00000108
    REG32 rxfifo; // 0x0000010C
    REG32 iofifo; // 0x00000110
    REG32 reg45; // 0x00000114
    REG32 deser_status; // 0x00000118
    REG32 ble_aes_ccm; // 0x0000011C
    REG32 irq_status; // 0x00000120
    REG32 rssi_min_max; // 0x00000124
    REG32 reg4A; // 0x00000128
    REG32 fei; // 0x0000012C
    REG32 reg4C; // 0x00000130
    REG32 analog_info; // 0x00000134
    REG32 sample_rssi; // 0x00000138
    REG32 rssi_therm; // 0x0000013C
    REG32 resevered_16[16]; // 0x00000140
    REG32 lut_antenna_array_1; // 0x00000180
    REG32 lut_antenna_array_2; // 0x00000184
    REG32 lut_antenna_array_3; // 0x00000188
    REG32 lut_antenna_array_4; // 0x0000018C
    REG32 resevered_12[12]; // 0x00000190
    REG32 reg50; // 0x000001C0
    REG32 resevered_000001C4; // 0x000001C4
    REG32 resevered_000001C8; // 0x000001C8
    REG32 resevered_000001CC; // 0x000001CC
    REG32 reg51; // 0x000001E0
    REG32 resevered_000001E4; // 0x000001E4
    REG32 reg52; // 0x000001E8
    REG32 resevered_000001EC; // 0x000001EC
    REG32 reg53; // 0x000001F0
    REG32 reg54; // 0x000001F4
    REG32 reg55; // 0x000001F8
    REG32 reg56; // 0x000001FC
} ICYTRX_GCS2_TypeDef;
```

#### BTDM RF

一些蓝牙芯片厂商采用CEVA蓝牙IP时如果支持蓝牙双模，则会用官方的蓝牙双模射频IP或自研射频IP，官方双模射频IP有Ripple和Thetis两种：Ripple不支持BLE 2M PHY，BLE Coded PHY由蓝牙IP提供BLE FEC支持；Thetis完美支持双模。另外还有Catena的蓝牙双模射频和CSEM的蓝牙双模射频ICYTRXDM，Ripple用的比较多，Thetis和ICYTRXDM现在应该很少使用，Catena的蓝牙双模射频IP会和Wi-Fi双频射频IP搭配一起使用，自研射频IP不大了解。当前只能收集到ICYTRXDM寄存器命名的文件，简单关注。

## L2

对应OSI L2-L4，通常表现成蓝牙Bluetooth IP，类比Wi-Fi IP的LMAC，也类比无线通信IP的BP( Baseband Processor)。重点关注收购RivieraWaves而来的蓝牙Bluetooth IP，可以收集到7.X、8. X、9. X和10.X四个版本。由于有些厂商的射频RF IP并没有实现BLE FEC编解码模块，CEVA从9.X版本开始好像在蓝牙Bluetooth IP实现了此模块。

一些版本蓝牙Bluetooth IP架构图如下所示：

- RSL10(v8.X)

  ![RSL10](https://bbs.kanxue.com/upload/attach/202304/422740_YMEXT5T589CVYPF.jpg)

- RSL15(v10.X)

  ![RSL15](https://bbs.kanxue.com/upload/attach/202304/422740_YSP6VDNGURVMEV9.png)

- QN9090(v8.X)

  ![QN9090](https://bbs.kanxue.com/upload/attach/202304/422740_H2GHTHWV7QCMMX8.jpg)

蓝牙Bluetooth IP有可能用CEVA的DSP实现，比如Teaklite/CEVA-X1/CEVA-X2/CEVA-BX1/CEVA-BX2等，也有可能是Cortus APS3，并且ROM固化，逆向很难搞，只能退而求其次，关注蓝牙Bluetooth IP链路层寄存器定义和描述。

### Link Layer Register

#### BLE v8.X

低功耗蓝牙BLE IP链路层寄存器描述粗略表示如下(详情请查看RSL10的SVD文件和用户手册)：

|  |  |

| --- | --- |
