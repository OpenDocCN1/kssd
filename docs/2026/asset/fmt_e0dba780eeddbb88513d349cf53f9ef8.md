
# [原创]修改某款lora固件库以支持接收错误数据包

## LoRa 固件

目前有一个需求，为了研究 `lora` 在不同干扰情况下数据包字节错误的情况，我必须将错误的数据包也记录下来，即没有通过CRC校验的数据包。但是现有的 `lora` 固件，会直接将没有通过CRC的数据包丢弃，无法查看错误的情况。因此，有了这个记录。

## 针对的芯片

研究的是 `lora` 2.4 的通信，因此针对的芯片是 `sx1280` 的，这里就不把集成了该芯片的厂商说出来了。下面主要介绍一些思路。

整体过程为：
1. 提取共享库
2. 结合网上开源的代码，找到关键位置，思考修改的方案
3. 找到最优的修改方案，执行修改，并重新打包测试。

### 提取共享库

厂商提供的是一个封装起来的 `LIB` 库，利用 `ubuntu` 的 `file` 工具发现是一个 `AR` 格式的打包文件，搜索的解包的方式：（参考<a href="elink@719K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1K9r3g2Y4k6h3g2C8M7%4c8#2k6X3k6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5H3i4K6u0r3x3o6S2Q4x3V1k6S2M7W2)9J5k6r3y4G2L8h3#2S2L8X3c8Q4x3X3c8W2P5r3q4E0M7r3I4W2M7#2)9J5c8R3`.`.">AR解压缩</a>）

![](upload/attach/202001/554506_D72GQYRA24EANEX.jpg)

```
ar x name.lib
```

修改之后，可以用下面的方式打包回来：

```
ar cr dest.lib src1.o src2.o src3.o
```

其中后面的 `src1.o src2.o src3.o` 就是共享库。`dest.lib` 是最终打包的文件名。

之后，我发现其中的 `sx1280.0` 的库比较关键，就尝试专注分析该库。

还有另外两个厂商提供的封装库，提供了一些 `MAC` 层的操作。这里就叫 `manufactory.o`.

### 找到关键位置

我初步分析时，首先尝试分析 `manufactory.o` 里面和 `MAC` 消息有关的操作。

我分析到两个关键的位置，1，正确接收到 `lora` 的消息时的操作

```
else if ( a2 & 0x40 )
clear_event((unsigned __int8)s_phy_layer_task_id, 4);
stop_timerEx((unsigned __int8)s_phy_layer_task_id, 4);
radio_raw_data = (unsigned __int8 *)msg_allocate(255);
if ( radio_raw_data )
{
*(_DWORD *)radio_raw_data_len = 0;
SX1280Hal::SetDirDisable((SX1280Hal *)&Radio);
clear_event((unsigned __int8)s_phy_layer_task_id, 76);
SX1280::GetPayload((SX1280 *)&Radio, radio_raw_data, radio_raw_data_len, 0xFFu);
SX1280::GetPacketStatus(&Radio, &PacketStatus);
SX1280Hal::SetDirEnable((SX1280Hal *)&Radio);
}
```

可以发现几个关键的步骤就是利用 `SX1280` 的库来获取 `payload` 并且存储起来。

另一个方面，我发现了几个处理 `lora` 接收错误情况的流程：

```
else if ( !s_PeriodBaseCount_flag )
{
SetRcMode();
send_cmd(9u); // RX_ERROR_CMD
}
else if ( a2 & 8 ) // SAMPLE_MSG_RX_TIMEOUT_EVNT
{
clear_event((unsigned __int8)s_phy_layer_task_id, 4);
stop_timerEx((unsigned __int8)s_phy_layer_task_id, 4);
if ( s_tx_confirm_flag == 1 )
{
s_tx_confirm_flag = 0;
v24 = msg_allocate(32);
if ( v24 )
{
*(_BYTE *)v24 = 2;
*(_BYTE *)(v24 + 2) = 6; // MAC_EVENT_TX_TIMEOUT_CMD
*(_BYTE *)(v24 + 3) = s_phy_layer_task_id;
*(_BYTE *)(v24 + 4) = Radio_packet_type;
*(_DWORD *)(v24 + 28) = 0;
*(_DWORD *)(v24 + 24) = 0;
msg_send((unsigned __int8)g_sample_task_id, v24);
}
}
else
{
v25 = msg_allocate(32);
if ( v25 )
{
*(_BYTE *)v25 = 2;
*(_BYTE *)(v25 + 2) = 9; // MAC_EVNT_RX_ERROR_CMD
*(_BYTE *)(v25 + 3) = s_phy_layer_task_id;
*(_BYTE *)(v25 + 4) = Radio_packet_type;
*(_DWORD *)(v25 + 28) = 0;
*(_DWORD *)(v25 + 24) = 0;
msg_send((unsigned __int8)g_sample_task_id, v25);
}
}
result = v4 ^ 8;
}
```

那么基本的思路，就是在上述所有反馈了 `lora` `rx error` 的地方，仍然将上述 `SX1280` 获取和拷贝数据的操作添加在逻辑里面。但是操作起来很复杂，拷贝 `payload` 的汇编代码整理一下有这些。而且对于某些条件跳转的地址，还得仔细的修改：

```
MOVS R1, #4
LDR R0, =s_phy_layer_task_id
LDRB R0, [R0]
BL osal_clear_event
MOVS R1, #4
LDR R0, =s_phy_layer_task_id
LDRB R0, [R0]
BL osal_stop_timerEx
MOVS R0, #0xFF
BL osal_msg_allocate
MOV R9, R0
CMP.W R9, #0
BEQ loc_123C
MOVS R0, #0
STR R0, [SP,#0x28+radio_raw_data_len]
LDR R0, =Radio ; this
BL _ZN9SX1280Hal13SetDirDisableEv ; SX1280Hal::SetDirDisable(void)
MOVS R1, #0x4C ; 'L'
LDR R0, =s_phy_layer_task_id
LDRB R0, [R0]
BL osal_clear_event
MOVS R3, #0xFF ; unsigned __int8
ADD R2, SP, #0x28+radio_raw_data_len ; unsigned __int8 *
MOV R1, R9 ; unsigned __int8 *
LDR R0, =Radio ; this
BL _ZN6SX128010GetPayloadEPhS0_h ; SX1280::GetPayload(uchar *,uchar *,uchar)
LDR R1, =PacketStatus
LDR R0, =Radio
BL _ZN6SX128015GetPacketStatusEP14PacketStatus_t ; SX1280::GetPacketStatus(PacketStatus_t *)
LDR R0, =Radio ; this
BL _ZN9SX1280Hal12SetDirEnableEv ; SX1280Hal::SetDirEnable(void)
```

看到上述都是基于 `sx1280` 的操作，那么我想能不能直接在 `sx1280` 找到一些突破口。

然后发现 `sx1280` 的库实现时开源的。<a href="elink@69fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7#2)9J5k6h3#2T1k6h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1k6h3q4E0M7#2)9J5c8W2y4W2L8i4c8W2j5$3S2Q4x3V1k6U0L8$3c8W2i4K6u0r3f1#2R3I4x3U0R3H3e0r3W2T1i4K6u0r3">sx1280</a>

这是非常幸运的。我直接就搜索 `CRC` 有关的位置，发现在 `SX1280::ProcessIrqs( void )` 的这个函数里面，有这样一个逻辑：

```
if( ( irqRegs & IRQ_RX_DONE ) == IRQ_RX_DONE )
{
if( ( irqRegs & IRQ_CRC_ERROR ) == IRQ_CRC_ERROR )// 如果收到底层CRC不通过的ERROR，就向上层汇报IRQ_CRC_ERROR_CODE。那么MAC层就会根据这个error进行相应的处理，也即是上面看到的丢包处理。理论上，我们只要使得该语句永远为False就会一直跳转到下面的rxDone中，始终汇报数据包接收成功即可。
{
if( rxError != NULL )
{
rxError( IRQ_CRC_ERROR_CODE );
}
}
else
{
if( rxDone != NULL ) //这条基本都会true，因为rxDone是一个指针函数。
{
rxDone( );
}
}
}
```

正如代码中分析的，只要 `if( ( irqRegs & IRQ_CRC_ERROR ) == IRQ_CRC_ERROR )` 永远为真，那么就可以上报数据包成功接收消息，那么就可以把错误的数据包保留。

如果收到底层CRC不通过的ERROR，就向上层汇报 `IRQ_CRC_ERROR_CODE`。那么 `MAC` 层就会根据这个 `error` 进行相应的处理，也即是上面看到的丢包处理。理论上，我们只要使得该语句永远为False就会一直跳转到下面的 `rxDone` 中，始终汇报数据包接收成功即可。

### 最优方案的选择，修改测试

我们发现，直接修改 `sx1280` 是最简单的。

![在这里插入图片描述](upload/attach/202001/554506_G3BJ46F72ZPJGAD.jpg)

在 `0x228` 的代码位置，如果上述比较不等于 `0` 才会跳转执行下面的 `0x238` 的操作，即 `rxDone` 操作。因此，只要将 `BNE loc_238` 修改成无条件跳转 `B loc_238` 即可。汇编中 `D1` 是 `BNE` 跳转，`E0` 是无条件跳转即 `B`。因此在对应位置如下修改：

![在这里插入图片描述](upload/attach/202001/554506_YX37ANR8TGGG5NY.jpg)

修改的工具路径是：`edit->patch program->change byte`，（`IDA pro 7.0`）。

修改完之后，再看看程序逻辑已经被修改：

![在这里插入图片描述](upload/attach/202001/554506_QDYBGWRCKVEZRTU.jpg)

因此，我们修改一个字节就能够达到效果了。

保存修改。`edit->patch program-> apply patched to input file` ，就可以保存修改了。

再重新利用 `AR` 打包回来，测试效果。

发现错误的数据包也可以接受了。

本文仅供学习交流，如作他用所承受的法律责任一概与作者无关。
