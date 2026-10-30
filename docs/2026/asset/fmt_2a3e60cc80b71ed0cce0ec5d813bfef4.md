
## L3

对应OSI L5，通常表现成蓝牙Bluetooth Controller，类比Wi-Fi IP的UMAC，也类比无线通信IP的NP(Network Processor)。该实现可能ROM固化，也可能是Flash闪存实现，实现此功能的MCU指令集通常是ARM和RISC-V，最好是Flash闪存方式的实现，这样逆向蓝牙厂商官方以静态库的方式实现比较容易。

### BLE Coded PHY

CSEM ICYTRX GCS1 + CEVA BLE Link v8如何实现BLE Coded PHY，Controller用Exchange Memory与CEVA蓝牙BP通信，8.X版本的BP并不支持BLE FEC，如何ROM Patch未知，只能在射频BLE 1M PHY配置的基础上再配置相关寄存器。

#### RF Selection

链路寄存器粗略配置如下：

```
typedef struct bt_link_10.x_type
{
    //0x00
    //General Purpose Registers
    REG32  rw_bt_cntl;
    REG32  version;
    REG32  rw_bt_conf;
    REG32  int_cntl0;

    //0x10
    REG32  int_stat0;
    REG32  int_ack0;
    REG32  int_cntl1;
    REG32  int_stat1;

    //0x20
    REG32 int_ack1;
    REG32 act_fifo_stat;
    REG32  current_rx_descrptr;
    REG32  eptr;

    //0x40
    //Deep Sleep Registers
    REG32  deep_sleep_cntl;
    REG32  deep_sleep_wakeup;
    REG32  deep_sleep_stat;
    REG32  enbp_reset;

    //0x40
    REG32  fine_cnt_corr;
    REG32  clk_cnt_corr;
    REG32  reserved_48;
    REG32  reserved_4C;

    //0x50
    //Validation Registers
    REG32  diag_cntl;
    REG32  diag_stat;
    REG32  debug_add_max;
    REG32  debug_add_min;

    //0x60
    REG32  error_type_stat;
    REG32  sw_profiling;
    REG32  reserved_68;
    REG32  reserved_6C;

    //0x70
    //Radio Registers
    REG32  radio_cntl0;
    REG32  radio_cntl1;
    REG32  radio_cntl2;
    REG32  radio_cntl3;

    //0x80
    REG32  reserved_80;
    REG32  reserved_84;
    REG32  reserved_88;
    REG32  radio_pwr_up_down;

    //0x90
    REG32  radio_tx_rx_tim;
    REG32  reserved_94;
    REG32  reserved_98;
    REG32  reserved_9C;

    //0xA0
    REG32  spi_ptr_cntl0;
    REG32  spi_ptr_cntl1;
    REG32  spi_ptr_cntl2;
    REG32  spi_ptr_cntl3;

    //0xB0
    //Encryption and Decryption Registers
    REG32  aes_cntl;
    REG32  aes_key_31_0;
    REG32  aes_key_63_32;
    REG32  aes_key_95_64;

    //0xC0
    REG32  aes_key_127_96;
    REG32  aes_ptr;
    REG32  tx_mic_value;
    REG32  rx_mic_value;

    //0xD0
    //Regulatory Body and RF Testing Registers
    REG32  rf_test_cntl;
    REG32  rf_test_freq;
    REG32  rf_test_tx_stat;
    REG32  rf_test_rx_stat;

    //0xE0
    //Timing Generator Registers
    REG32  time_gen_cntl;
    REG32  fine_time_target;
    REG32  clk_current_target1;
    REG32  half_micro_second_target1;

    //0xF0
    REG32  clk_current_target2;
    REG32  half_micro_second_target2;
    REG32  slot_clock;
    REG32  fine_time_cnt;

    //0x100
    //Event Scheduler Registers
    REG32  bt_scheduler_cntl;
    REG32  start_event_clkn_count;
    REG32  start_event_fine_count;
    REG32  end_event_clkn_count;

    //0x110
    REG32  end_event_fine_count;
    REG32  skip_event_clkn_count;
    REG32  skip_event_fine_count;
    REG32  reserved_11C;

    //0x120
    //Inquiry/Page and EDR Control Registers
    REG32  abtrain_cntl;
    REG32  edr_cntl;
    REG32  reserved_128;
    REG32  reserved_12C;

    //0x130-0x140
    //MWS PCA Coexistence Registers
    //ignore

    //0x140-0x1F0
    //WLAN/MWS Coexistence Registers
    //ignore

    //0x1F0-0x200
    //SAM(Local Map) Registers
    //ignore

    //0x200-0x260
    //Audio Channel Registers
    //ignore

    //0x260-0x270
    //Audio Codec Registers
    //ignore

    //0x270
    //ignore
}BT_LINK_10.X_TypeDef;
```

#### RF Config

射频寄存器粗略配置如下：

```
union {
    uint32_t R;
    struct {
        reg32 SUBVERSION:4; /* bit[0:3].CSEM RF Sub-version selection. 0: IcyTRx 65nm (v0x12, v0x20, v0x22) 1: IcyTRx 55nm CS1 (v0x30);2: IcyTRx 55nm CS2 (v0x31);3: IcyTRx 55nm GCS1 (v0x40);4: IcyTRx 55nm CS553(v0x62)??? */
        reg32 XRFSEL:5; /* bit[4:8].Extended radio selection field.0: No radio selected;1: Ripple radio (BT4.0) 2: External radio controller;3: ICYTRX radio (BLE) */
        reg32 reserved_09_11:3;
        reg32 JEF_SELECT:1;
        reg32 DPCORR_EN:1;
        reg32 reserved_14_14:1;
        reg32 SYNC_PULSE_MODE:1;
        reg32 FORCEAGC_LENGTH:12;
        reg32 reserved_28_29:2;
        reg32 FORCEBLEIQ:1;
        reg32 FORCEAGC_EN:1;
    } B;
} BB_RADIOCNTL1;
//BB_RADIOCNTL1.SUBVERSION=3;
//BB_RADIOCNTL1.XRFSEL=3;

union {
    uint32_t R;
    struct {
        reg32 ADD_WIDTH:5;
        reg32 DATA_WIDTH:1;
        reg32 BUSTYPE:1;
        reg32 INTMODE:1;
        reg32 CLK_SEL:6;
        reg32 USECRYPT:1;
        reg32 USEDBG:1;
        reg32 RFIF:5; /* bit[16:20].Support of the RF front-end. 0: Ripple RF;1: External radio controller support; 2: Atlas radio;3: IcyTRx radio */
        reg32 COEX:1;
        reg32 reserved_22_22:1;
        reg32 DECIPHER:1;
        reg32 ISOPORTNB:2;
        reg32 reserved_26_30:5;
        reg32 DMMODE:1;
    } B;
} BB_RWBLEBCONF;
//BB_RWBLEBCONF.RFIF=3;
```
