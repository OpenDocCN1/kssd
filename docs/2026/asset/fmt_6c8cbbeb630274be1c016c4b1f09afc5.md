
# [原创]Bluetooth Internals: A Reverse Engineer’s Perspective - Bouffalo Lab

## Bouffalo Lab

射频IP是自研的，采用CEVA蓝牙IP。BL618/BL606P官方文档说明支持经典蓝牙，相关的射频IP寄存器的描述和定义未知，只能不完全分析BL602/BL702的低功耗蓝牙BLE的相关寄存器。

## L1

### Low-IF Transceiver

射频IP自研，是低中频收发器， bl602用模拟锁相环进行频率合成，bl702 用数字锁相环进行频率合成， bl618/bl606p未知，只能收集<a href="elink@8e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9h3&6W2y4U0c8Q4x3V1k6T1L8o6j5H3x3W2)9J5k6s2u0W2i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0D9L8$3u0K6i4K6u0r3M7X3g2Y4i4K6u0r3M7X3g2Y4i4K6g2X3M7X3k6Q4x3X3g2Z5">bl602</a>和<a href="elink@018K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3c8W2L8r3k6Q4x3V1k6T1L8o6M7H3x3W2)9J5k6s2m8S2j5#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6M7X3y4Q4x3V1k6J5k6R3`.`.">bl702</a>射频收发器寄存器定义，bl618/bl606p未知。

### Modem

bl618/bl606p官方文档说明支持经典蓝牙，相关寄存器未知，只能收集到bl602低功耗蓝牙BLE物理层基带寄存器定义<a href="elink@41bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9h3&6W2y4U0c8Q4x3V1k6T1L8o6j5H3x3W2)9J5k6s2u0W2i4K6u0r3j5X3I4G2j5W2)9J5c8Y4y4J5j5#2)9J5c8X3W2F1j5$3I4#2k6r3g2Q4x3V1k6H3K9s2W2Q4x3V1k6T1P5W2)9#2k6Y4m8Z5P5g2)9J5k6h3R3`.`.">`bz_phy`</a>，寄存器各字段含义未知。

## L2

### Version

官方静态库文件`rwble.o`的`rwble_version`函数汇编指令(riscv)如下：

```
00000000 <rwble_version>:
0:47a5 li a5,0x09
2:00f501a3 sb a5,3(a0)
6:478d li a5,0x03
8:00f50123 sb a5,2(a0)
c:02100793 li a5,0x21
10:00f500a3 sb a5,1(a0)
14:00050023 sb zero,0(a0)
18:280007b7 lui a5,0x28000
1c:43d8 lw a4,4(a5)
1e:8361 srli a4,a4,0x18
20:00e581a3 sb a4,3(a1)
24:43d8 lw a4,4(a5)
26:8341 srli a4,a4,0x10
28:00e58123 sb a4,2(a1)
2c:43d8 lw a4,4(a5)
2e:8321 srli a4,a4,0x8
30:00e580a3 sb a4,1(a1)
34:43dc lw a5,4(a5)
36:00f58023 sb a5,0(a1)
3a:8082 ret
```

可能的C源码：

```
/**
 * @brief VERSION register definition
 * <pre>
 * Bits Field Name Reset Value
 * ----- ------------------ -----------
 * 31:24 TYP 0x09
 * 23:16 REL 0x03
 * 15:08 UPG 0x21
 * 07:00 BUILD 0x00
 * </pre>
 */
#define BLE_VERSION_ADDR 0x28000004
#define BLE_VERSION_OFFSET 0x00000004
#define BLE_VERSION_INDEX 0x00000001
#define BLE_VERSION_RESET 0x09032100
__INLINE uint32_t ble_version_get(void)
{
return REG_BLE_RD(BLE_VERSION_ADDR);
}
// field definitions
#define BLE_TYP_MASK ((uint32_t)0xFF000000)
#define BLE_TYP_LSB 24
#define BLE_TYP_WIDTH ((uint32_t)0x00000008)
#define BLE_REL_MASK ((uint32_t)0x00FF0000)
#define BLE_REL_LSB 16
#define BLE_REL_WIDTH ((uint32_t)0x00000008)
#define BLE_UPG_MASK ((uint32_t)0x0000FF00)
#define BLE_UPG_LSB 8
#define BLE_UPG_WIDTH ((uint32_t)0x00000008)
#define BLE_BUILD_MASK ((uint32_t)0x000000FF)
#define BLE_BUILD_LSB 0
#define BLE_BUILD_WIDTH ((uint32_t)0x00000008)
#define BLE_TYP_RST 0x09
#define BLE_REL_RST 0x03
#define BLE_UPG_RST 0x21
#define BLE_BUILD_RST 0x00
__INLINE void ble_version_unpack(uint8_t* typ, uint8_t* rel, uint8_t* upg, uint8_t* build)
{
uint32_t localVal = REG_BLE_RD(BLE_VERSION_ADDR);
*typ = (localVal & ((uint32_t)0xFF000000)) >> 24;
*rel = (localVal & ((uint32_t)0x00FF0000)) >> 16;
*upg = (localVal & ((uint32_t)0x0000FF00)) >> 8;
*build = (localVal & ((uint32_t)0x000000FF)) >> 0;
}
__INLINE uint8_t ble_version_typ_getf(void)
{
uint32_t localVal = REG_BLE_RD(BLE_VERSION_ADDR);
return ((localVal & ((uint32_t)0xFF000000)) >> 24);
}
__INLINE uint8_t ble_version_rel_getf(void)
{
uint32_t localVal = REG_BLE_RD(BLE_VERSION_ADDR);
return ((localVal & ((uint32_t)0x00FF0000)) >> 16);
}
__INLINE uint8_t ble_version_upg_getf(void)
{
uint32_t localVal = REG_BLE_RD(BLE_VERSION_ADDR);
return ((localVal & ((uint32_t)0x0000FF00)) >> 8);
}
__INLINE uint8_t ble_version_build_getf(void)
{
uint32_t localVal = REG_BLE_RD(BLE_VERSION_ADDR);
return ((localVal & ((uint32_t)0x000000FF)) >> 0);
}
#define BT40_VERSION (0x06)
#define BT41_VERSION (0x07)
#define BT42_VERSION (0x08)
#define BT50_VERSION (0x09)
#define BT51_VERSION (0x0A)
#define BT52_VERSION (0x0B)
#define BT53_VERSION (0x0C)
//bl602 or bl702
//ble_link_base:0x28000000
#define RWBT_SW_VERSION_MAJOR BT50_VERSION
//bl618 or bl606p ???
////bt_link_base:0x28000400????
////ble_link_base:0x28000800????
//#define RWBT_SW_VERSION_MAJOR BT52_VERSION
#define RWBT_SW_VERSION_MINOR (0x03)
#define RWBT_SW_VERSION_BUILD (0x21)
#define RWBT_SW_VERSION_SUB_BUILD (0x00)
void rwble_version(uint8_t* fw_version, uint8_t* hw_version)
{
// FW version
*(fw_version+3) = RWBT_SW_VERSION_MAJOR;
*(fw_version+2) = RWBT_SW_VERSION_MINOR;
*(fw_version+1) = RWBT_SW_VERSION_BUILD;
*(fw_version) = RWBT_SW_VERSION_SUB_BUILD;
// HW version
*(hw_version+3) = ble_version_typ_getf();
*(hw_version+2) = ble_version_rel_getf();
*(hw_version+1) = ble_version_upg_getf();
*(hw_version) = ble_version_build_getf();
return;
}
```

bl602/bl702采用版本9的蓝牙IP，BLE链路寄存器基址为`0x28000000`，bl618/bl606p可能采用版本10或版本11或版本12的蓝牙IP，低功耗蓝牙BLE链路寄存器基址可以通过逆向`rwble.o`的`rwble_isr`函数获得，基址应该为`0x28000800`，同理可以获得经典蓝牙BT链路寄存器基址为`0x28000400`。

### Register

bl602/bl702的链路寄存器粗略表示如下(不一定对)：
