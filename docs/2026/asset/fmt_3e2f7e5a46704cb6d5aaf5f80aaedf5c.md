
# [原创][Googlectf2022]硬件题weather

## 前言

之前尝试挖iot设备的漏洞大多是基于对固件的分析，对硬件层面的东西还不太熟悉，正好开学数电计组连考积累了一点点这方面最基础的东西，于是就找了这道题来做个人了解硬件的一个入门

## 前置知识

当时看着xuanxuan师傅的博客，遇到了很多问题也学到了许多零散的知识，总结如下：

（1）I2C总线

（2）SPI协议

（3）EEPROM

（4）8051的特殊寄存器sfr

## 复现环境搭建

下载下来的赛题环境里有个docker，从dockerfile里可以看出运行在1337端口：

![img](upload/attach/202303/943085_VFFP5AQXS3ZRR36.jpg)

使用如下命令运行docker：

```plain
docker build -t hardware:weather .
docker run --privileged -p xxxx:1337 -d hardware:weather
```

![img](upload/attach/202303/943085_EQKDMFHZU8BJ3NZ.jpg)

然后使用`nc`指令就能连上了：

```plain
nc 127.0.0.1 1337
```

![img](upload/attach/202303/943085_J52HCDRZ65B8RSV.jpg)

## 附件分析

附件通过一个pdf提供了赛题的集成电路图：

![img](upload/attach/202303/943085_ZAAZHZ2NCGR7TKD.jpg)

可以看出：

（1）存在5个传感器，通过IIC总线连接

（2） EEPROM与8051通过SPI协议相连，并且都有IIC总线接口

（3）Flag位于8051的FlagROM

分析题目提供的源码：

```c
#include <stdint.h>
#include <stdbool.h>
#ifndef NULL
#define NULL ((void*)0)
#endif
// Secret ROM controller.
__sfr __at(0xee) FLAGROM_ADDR;
__sfr __at(0xef) FLAGROM_DATA;
// Serial controller.
__sfr __at(0xf2) SERIAL_OUT_DATA;
__sfr __at(0xf3) SERIAL_OUT_READY;
__sfr __at(0xfa) SERIAL_IN_DATA;
__sfr __at(0xfb) SERIAL_IN_READY;
// I2C DMA controller.
__sfr __at(0xe1) I2C_STATUS;
__sfr __at(0xe2) I2C_BUFFER_XRAM_LOW;
__sfr __at(0xe3) I2C_BUFFER_XRAM_HIGH;
__sfr __at(0xe4) I2C_BUFFER_SIZE;
__sfr __at(0xe6) I2C_ADDRESS; // 7-bit address
__sfr __at(0xe7) I2C_READ_WRITE;
// Power controller.
__sfr __at(0xff) POWEROFF;
__sfr __at(0xfe) POWERSAVE;
const char *ALLOWED_I2C[] = { //五个传感器
"101", // Thermometers (4x).
"108", // Atmospheric pressure sensor.
"110", // Light sensor A.
"111", // Light sensor B.
"119", // Humidity sensor.
NULL
};
int8_t i2c_write(int8_t port, uint8_t req_len, __xdata uint8_t *buf) {
while (I2C_STATUS == 1) {
POWERSAVE = 1; // Enter power save mode for a few milliseconds.
}
I2C_BUFFER_XRAM_LOW = (uint8_t)(uint16_t)buf;
I2C_BUFFER_XRAM_HIGH = (uint8_t)((uint16_t)buf >> 8);
I2C_BUFFER_SIZE = req_len;
I2C_ADDRESS = port;
I2C_READ_WRITE = 0; // Start write.
int8_t status;
while ((status = I2C_STATUS) == 1) {
POWERSAVE = 1; // Enter power save mode for a few milliseconds.
}
return status;
}
int8_t i2c_read(int8_t port, uint8_t req_len, __xdata uint8_t *buf) {
while (I2C_STATUS == 1) {
POWERSAVE = 1; // Enter power save mode for a few milliseconds.
}
I2C_BUFFER_XRAM_LOW = (uint8_t)(uint16_t)buf;
I2C_BUFFER_XRAM_HIGH = (uint8_t)((uint16_t)buf >> 8);
I2C_BUFFER_SIZE = req_len;
I2C_ADDRESS = port;
I2C_READ_WRITE = 1; // Start read.
int8_t status;
while ((status = I2C_STATUS) == 1) {
POWERSAVE = 1; // Enter power save mode for a few milliseconds.
}
return status;
}
const char *i2c_status_to_error(int8_t err) {
switch (err) {
case 0: return "i2c status: transaction completed / ready\n";
case 1: return "i2c status: busy\n";
case 2: return "i2c status: error - device not found\n";
case 3: return "i2c status: error - device misbehaved\n";
}
return "i2c status: unknown error\n";
}
void serial_print(const char *s) {
while (*s) {
while (!SERIAL_OUT_READY) {
// Busy wait...
}
SERIAL_OUT_DATA = *s++;
}
}
char serial_read_char(void) {
while (1) {
if (SERIAL_IN_READY) {
return (char)SERIAL_IN_DATA;
}
POWERSAVE = 1; // Enter power save mode for a few milliseconds.
}
}
struct tokenizer_st {
char *ptr;
int replaced;
};
void tokenizer_init(struct tokenizer_st *t, char *str) {
t->ptr = str;
t->replaced = 0x7fff;
}
char *tokenizer_next(struct tokenizer_st *t) {
if (t->replaced != 0x7fff) {
*t->ptr = (char)t->replaced;
}
while (*t->ptr == ' ') {
t->ptr++;
}
if (*t->ptr == '\0') {
return NULL;
}
char *token_start = t->ptr;
for (;;) {
char ch = *t->ptr;
if (ch != ' ' && ch != '\0') {
t->ptr++;
continue;
}
t->replaced = *t->ptr;
*t->ptr = '\0';
return token_start;
}
}
uint8_t str_to_uint8(const char *s) {
uint8_t v = 0;
while (*s) {
uint8_t digit = *s++ - '0';
if (digit >= 10) {
return 0;
}
v = v * 10 + digit;
}
return v;
}
void uint8_to_str(char *buf, uint8_t v) {
if (v >= 100) {
*buf++ = '0' + v / 100;
}
if (v >= 10) {
*buf++ = '0' + (v / 10) % 10;
}
*buf++ = '0' + v % 10;
*buf = '\0';
}
bool is_port_allowed(const char *port) { //前缀匹配
for(const char **allowed = ALLOWED_I2C; *allowed; allowed++) {
const char *pa = *allowed;
const char *pb = port;
bool allowed = true;
while (*pa && *pb) {
if (*pa++ != *pb++) {
allowed = false;
break;
}
}
if (allowed && *pa == '\0') {
return true;
}
}
return false;
}
int8_t port_to_int8(char *port) {
if (!is_port_allowed(port)) {
return -1;
}
return (int8_t)str_to_uint8(port);
}
#define CMD_BUF_SZ 384
#define I2C_BUF_SZ 128
int main(void) {
serial_print("Weather Station\n");
static __xdata char cmd[CMD_BUF_SZ];
static __xdata uint8_t i2c_buf[I2C_BUF_SZ];
while (true) {
serial_print("? ");
int i;
for (i = 0; i < CMD_BUF_SZ; i++) {
char ch = serial_read_char();
if (ch == '\n') {
cmd[i] = '\0';
break;
}
cmd[i] = ch;
}
if (i == CMD_BUF_SZ) {
serial_print("-err: command too long, rejected\n");
continue;
}
struct tokenizer_st t;
tokenizer_init(&t, cmd);
char *p = tokenizer_next(&t);
if (p == NULL) {
serial_print("-err: command format incorrect\n");
continue;
}
bool write;
if (*p == 'r') {
write = false;
} else if (*p == 'w') {
write = true;
} else {
serial_print("-err: unknown command\n");
continue;
} //首先获取操作符
p = tokenizer_next(&t);
if (p == NULL) {
serial_print("-err: command format incorrect\n");
continue;
}
int8_t port = port_to_int8(p); //获取操作端口
if (port == -1) {
serial_print("-err: port invalid or not allowed\n");
continue;
}
p = tokenizer_next(&t);
if (p == NULL) {
serial_print("-err: command format incorrect\n");
continue;
}
uint8_t req_len = str_to_uint8(p);//获取操作长度
if (req_len == 0 || req_len > I2C_BUF_SZ) {
serial_print("-err: I2C request length incorrect\n");
continue;
}
if (write) { //将写入数据从str转换为uint8数组
for (uint8_t i = 0; i < req_len; i++) {
p = tokenizer_next(&t);
if (p == NULL) {
break;
}
i2c_buf[i] = str_to_uint8(p);
}
int8_t ret = i2c_write(port, req_len, i2c_buf);
serial_print(i2c_status_to_error(ret));
} else {
int8_t ret = i2c_read(port, req_len, i2c_buf);
serial_print(i2c_status_to_error(ret));
for (uint8_t i = 0; i < req_len; i++) {
char num[4];
uint8_to_str(num, i2c_buf[i]);
serial_print(num);
if ((i + 1) % 16 == 0 && i +1 != req_len) {
serial_print("\n");
} else {
serial_print(" ");
}
}
serial_print("\n-end\n");
}
}
// Should never reach this place.
}
```

通过标志性的语法`__sfr __at`可以分析出这个是基于SDCC开发的

可以下载一个SDCC的官方编译器，甚至默认安装选项里就会自动添加环境变量（windows环境），非常方便，在windows的cmd中输入如下命令便可编译：

```plain
sdcc firmware.c
```

编译得到如下文件

![img](upload/attach/202303/943085_VQ45VNTP4Z5FHSR.jpg)

其中`ihx`是可逆向的文件，笔者使用的`ida7.7`貌似不能直接识别为8051，需要手工选择：

![img](upload/attach/202303/943085_A3KVA266PZ83X89.jpg)

程序提供了两个形式的输入指令:

```plain
w port size <PageIndex> <4ByteWriteKey> <ClearMask> ... <ClearMask>
r port size
```

其中对`port`有个检查：

![img](upload/attach/202303/943085_5XXBCRV2429FSHQ.jpg)

发现有个明显的漏洞，就是对于匹配的`101`字符，输入`101xxxxxxx`居然也能通过这个匹配

结合后面的`str_to_int8`可以得到最终的端口号为`101xxxx % 256`

又因为：

![img](upload/attach/202303/943085_PYQC8PESDJAT4JS.jpg)

所以：

![img](upload/attach/202303/943085_2XRU6THX39KD68R.jpg)

![img](upload/attach/202303/943085_PB5D48CF5C6V6W5.jpg)

又因为IIc总线最多支持128个设备，于是我们从`101120~101120+128`爆破扫描128次即可爆破出所有端口

爆破脚本：
