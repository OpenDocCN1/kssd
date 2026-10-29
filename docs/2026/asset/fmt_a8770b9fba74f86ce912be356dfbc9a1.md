
## 注册Native方法：

### 解密Native方法名

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 | `.data.rel.ro:CB215C28 ;` `"_genericNt"` `.data.rel.ro:CB215C2C` `5E` `31` `22` `CB DCD aLjavaLangStrin ;` `"(Ljava/lang/String;I)I"` `.data.rel.ro:CB215C30 B5` `42` `14` `CB DCD _genericNt_sub_CDDFF2B4`+`1` `.data.rel.ro:CB215C34` `4B` `30` `22` `CB DCD aGenericnt2 ;` `"_genericNt2"` `.data.rel.ro:CB215C38` `33` `30` `22` `CB DCD aILjavaLangStri ;` `"(I[Ljava/lang/String;)I"` `.data.rel.ro:CB215C3C` `09` `43` `14` `CB DCD _genericNt2_sub_CDDFF308`+`1` `.data.rel.ro:CB215C40` `27` `30` `22` `CB off_CB215C40 DCD aGenericnt3 ; DATA XREF: RegNative_sub_CDB0C110`+`72`↑o` `.data.rel.ro:CB215C40 ;` `"_genericNt3"` `.data.rel.ro:CB215C44` `0F` `30` `22` `CB DCD aIBLjavaLangStr ;` `"(I[B)Ljava/lang/String;"` `.data.rel.ro:CB215C48` `01` `45` `14` `CB DCD _genericNt3_sub_CDDFF500`+`1` `.data.rel.ro:CB215C4C` `0A` `30` `22` `CB DCD aOnsc ;` `"onSc"` `.data.rel.ro:CB215C50 AF` `2F` `22` `CB DCD aLandroidHardwa ;` `"(Landroid/hardware/SensorManager;Lcom/a"`...` `.data.rel.ro:CB215C54` `19` `46` `14` `CB DCD onSc_sub_CDDFF618`+`1` |

#### 注册方法

|  |  |
| --- | --- |
| 1 2 3 4 5 6 | `.text:CB1381D8 F1 F7 AA FD BL DecString_loc_CEE73D30 ; com/aliyun/TigerTally/TigerTallyAPI` `.text:CB1381DC 03 99 LDR R1, [SP,#0x68+var_5C]` `.text:CB1381DE 04 22 MOVS R2,#4` `.text:CB1381E0 0B F0 3A FF BL RegNative_loc_CEE8E058 // RegisterNatives` `.text:CB1381E4 0C 49 LDR R1, =( __stack_chk_guard_ptr - 0xCB1381EC )` `.text:CB1381E6 12 9A LDR R2, [SP,#0x68+var_20]` |

### 3.2、解密APPKEY

#### `native int _genericNt2(int arg0, String[] arg1)`

获取`java`层传入的`sdk appkey`

|  |  |
| --- | --- |
