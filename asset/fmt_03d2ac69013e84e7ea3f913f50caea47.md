
# DOS 应用程序

> 原文：[`www.pediy.com/kssd/pediy03/forum486.htm`](https://www.pediy.com/kssd/pediy03/forum486.htm)

DOS 应用程序中用 CD-ROM,主要用到 `INT 2FH` 的 `1510H`（`Send device driver request`） 功能。

要用到的几个中断说明如下：

1. `IOCTL input` 的格式：

   偏移量 类型 内容
   
   `00H` 字节 `1AH`
   
   `01H` 字节 `00H`
   
   `02H` 字节 `03H`
   
   `03H` 字节 `11 DUP`（？）
   
   `0EH` 双字 缓冲区地址
   
   `12H` 字 缓冲区长度
   
   `14H` 字节 `6 DUP`（？）

2. `IOCTL output` 的格式：

   偏移量 类型 内容
   
   `00H` 字节 `1AH`
   
   `01H` 字节 `00H`
   
   `02H` 字节 `0CH`
   
   `03H` 字节 `11 DUP`（？）
   
   `0EH` 双字 缓冲区地址
   
   `12H` 字 缓冲区长度
   
   `14H` 字节 `6 DUP`（？）

3. 在缓冲区中填入命令再用 `INT 2fH` 的 `1510H` 便可以实现 CD-ROM 的各种功能，缓冲的一些命令格式如下：

   作用 长度 格式
   
   解锁 2 `01,00`
   
   加锁 2 `01,01`
   
   入盒 1 `05`
   
   取 CD-ROM 状态 5 `06,0,0,0,0`
   
   返回：`06,xx,xx,0,0`
   
   `xx,xx` 为状态字
   
   位 0：`1` 门开，`0` 门关
   
   位 1：`1` 未锁，`0` 门锁
   
   位 B：`1` 无盘，`0` 有盘

4. 检测 CD-ROM 驱动程序的存在：

   ```
   AX = 1500H
   BX = 0
   INT 2fH
   
   返回：BX = CD-ROM 个数
   
          CX = 起始 CD-ROM 盘符，0=A：，1=B：等等
   ```

5. 发送 Device Driver Request：

   ```
   AX = 1510H
   ES：BX=请求头地址
   
   出盒 1 00
   
   CX = CD-ROM 的盘号
   ```
