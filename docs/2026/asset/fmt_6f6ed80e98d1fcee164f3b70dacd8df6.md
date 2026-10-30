
### 基于RSL15 SVD(montana.svd)

```c
// 基于RSL15 SVD(montana.svd)
typedef struct ble_link_10.x_type
{
    // 0x00
    // General Purpose Registers
    REG32  rw_ble_cntl;
    REG32  version;
    REG32  rw_ble_conf;
    REG32  int_cntl0;

    // 0x10
    REG32  int_stat0;
    REG32  int_ack0;
    REG32  int_cntl1;
    REG32  int_stat1;

    // 0x20
    REG32 int_ack1;
    REG32 act_fifo_stat;
    REG32  current_rx_descrptr;
    REG32  eptr;

    // 0x40
    // Deep Sleep Registers
    REG32  deep_sleep_cntl;
    REG32  deep_sleep_wakeup;
    REG32  deep_sleep_stat;
    REG32  enbp_reset;

    // 0x40
    REG32  fine_cnt_corr;
    REG32  clk_cnt_corr;
    REG32  reserved_48;
    REG32  reserved_4C;

    // 0x50
    // Validation Registers
    REG32  diag_cntl;
    REG32  diag_stat;
    REG32  debug_add_max;
    REG32  debug_add_min;

    // 0x60
    REG32  error_type_stat;
    REG32  sw_profiling;
    REG32  reserved_68;
    REG32  reserved_6C;

    // 0x70
    // Radio Registers
    REG32  radio_cntl0;
    REG32  radio_cntl1;
    REG32  radio_cntl2;
    REG32  radio_cntl3;

    // 0x80
    REG32  radio_pwr_up_down0;
    REG32  radio_pwr_up_down1;
    REG32  radio_pwr_up_down2;
    REG32  radio_pwr_up_down3;

    // 0x90
    REG32  radio_tx_rx_tim0;
    REG32  radio_tx_rx_tim1;
    REG32  radio_tx_rx_tim2;
    REG32  radio_tx_rx_tim3;

    // 0xA0
    REG32  spi_ptr_cntl0;
    REG32  spi_ptr_cntl1;
    REG32  spi_ptr_cntl2;
    REG32  spi_ptr_cntl3;

    // 0xB0
    // Encryption and Decryption Registers
    REG32  aes_cntl;
    REG32  aes_key_31_0;
    REG32  aes_key_63_32;
    REG32  aes_key_95_64;

    // 0xC0
    REG32  aes_key_127_96;
    REG32  aes_ptr;
    REG32  tx_mic_value;
    REG32  rx_mic_value;

    // 0xD0
    // Regulatory Body and RF Testing Registers
    REG32  rf_test_cntl;
    REG32  rf_test_tx_stat;
    REG32  rf_test_rx_stat;
    REG32  reserved_DC;

    // 0xE0
    // Timing Generator Registers
    REG32  time_gen_cntl;
    REG32  gross_time_target;
    REG32  fine_time_target;
    REG32  clk_current_target1;

    // 0xF0
    REG32  half_micro_second_target1;
    REG32  clk_current_target2;
    REG32  half_micro_second_target2;
    REG32  slot_clock;

    // 0x100
    // Event Scheduler Registers
    REG32  ble_scheduler_cntl;
    REG32  start_event_clkn_count;
    REG32  start_event_fine_count;
    REG32  end_event_clkn_count;

    // 0x110
    REG32  end_event_fine_count;
    REG32  skip_event_clkn_count;
    REG32  skip_event_fine_count;
    REG32  reserved_11C;

    // 0x120
    // Advertising Timer and Scanning Timer Registers
    REG32  advertising_time;
    REG32  scanning_time;
    REG32  reserved_128;
    REG32  reserved_12C;

    // 0x130
    // Device Filtering Registers
    REG32  white_list_cntl;
    REG32  white_list_current_ptr;
    REG32  search_time_out;
    REG32  reserved_13C;

    // 0x140
    // WLAN Coexistence and WiMax Coexistence Registers
    REG32  coexistence_cntl0;
    REG32  coexistence_cntl1;
    REG32  coexistence_cntl2;
    REG32  ble_priority0;

    // 0x150
    REG32  ble_priority1;
    REG32  ble_priority2;
    REG32  reserved_158;
    REG32  reserved_15C;

    // 0x160
    // Resolving Address List Registers
    REG32  resolving_addr_list_cntl;
    REG32  resolving_addr_list_current_ptr;
    REG32  resolving_addr_list_local_random;
    REG32  resolving_addr_list_peer_random;

    // 0x170
    // AoA Controls and AoD Controls Registers
    REG32 dfc_cntl0_1us;
    REG32 dfc_cntl0_2us;
    REG32 dfc_cntl1_1us;
    REG32 dfc_cntl1_2us;

    // 0x180
    REG32  dfc_current_ptr;
    REG32  df_antenna_cntl;
    REG32  df_interface_cntl;
    REG32  reserved_18C;

    // 0x190
    // ISO Channel Registers ?????
} BLE_LINK_10.X_TypeDef;
```

#### BT v10.X

经典蓝牙BT IP链路层寄存器描述粗略表示如下:

|  |  |

| --- | --- |
