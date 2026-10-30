
# 字符串提取与 RC4 解密分析

## 初始字符串构造

程序会把两个字符串给局部变量，分别是`Jyu3CJlVDSGQ`和`Pjpeyjk6mmH=`。

```
.text:000028F2                 MOVS            R0, #'J'

.text:000028F4                 MOVS            R1, #'y'

.text:0000295C                 STRH.W          R0, [SP,#0x22]

.text:000029C6                 STRH.W          R1, [SP,#0x24]

.text:000029CA                 MOVS            R1, #'u'

.text:00002A32                 STRH.W          R1, [SP,#0x26]

.text:00002A36                 MOVS            R1, #'3'

.text:00002A9E                 STRH.W          R1, [SP,#0x28]

.text:00002AA2                 MOVS            R1, #'C'

.text:00002B0A                 STRH.W          R1, [SP,#0x2A]

.text:00002B74                 MOVS            R1, #0

.text:00002B76                 STRH.W          R0, [SP,#0x2C]

.text:00002B7A                 MOVS            R0, #'l'

.text:00002BE2                 STRH.W          R0, [SP,#0x2E]

.text:00002BE6                 MOVS            R0, #'V'

.text:00002C4E                 STRH.W          R0, [SP,#0x30]

.text:00002C52                 MOVS            R0, #'D'

.text:00002CBA                 STRH.W          R0, [SP,#0x32]

.text:00002CBE                 MOVS            R0, #'S'

.text:00002D26                 STRH.W          R0, [SP,#0x34]

.text:00002D2A                 MOVS            R0, #'G'

.text:00002D92                 STRH.W          R0, [SP,#0x36]

.text:00002D96                 MOVS            R0, #'Q'

.text:00002DFE                 STRH.W          R0, [SP,#0x38]

;===============================================================

.text:00001A30                 MOVS            R0, #'P'

.text:00001A32                 MOVS            R1, #'p'

.text:00001A34                 STRH.W          R0, [SP,#0x22]

.text:00001A38                 MOVS            R0, #'j'

.text:00001AA0                 STRH.W          R0, [SP,#0x24]

.text:00001B0A                 MOVS            R3, #0

.text:00001B0C                 STRH.W          R1, [SP,#0x26]

.text:00001B10                 MOVS            R1, #'e'

.text:00001B78                 STRH.W          R1, [SP,#0x28]

.text:00001B7C                 MOVS            R1, #'y'

.text:00001BE4                 STRH.W          R1, [SP,#0x2A]

.text:00001C4E                 MOVW            R1, #0xFFFF

.text:00001C52                 STRH.W          R0, [SP,#0x2C]

.text:00001C56                 MOVS            R0, #'k'

.text:00001C58                 LDR.W           R12, =unk_1E216

.text:00001CC2                 MOVS            R2, #0

.text:00001CC4                 STRH.W          R0, [SP,#0x2E]

.text:00001CC8                 MOVS            R0, #'6'

.text:00001D30                 STRH.W          R0, [SP,#0x30]

.text:00001D34                 MOVS            R0, #'m'

.text:00001D9C                 STRH.W          R0, [SP,#0x32]

.text:00001E06                 ADD             R12, PC

.text:00001E08                 STRH.W          R0, [SP,#0x34]

.text:00001E0C                 MOVS            R0, #'H'

.text:00001E74                 STRH.W          R0, [SP,#0x36]

.text:00001E78                 MOVS            R0, #'='

.text:00001EE0                 STRH.W          R0, [SP,#0x38]
```

## 动态调试与流程跟踪

其他是一些计算逻辑，静态看着太费力，上动态调试，直接附加，并没有遇到反调试，再次跟流程 （以下`so`基址均为`0xB3998000`）

发现这两个字符串最终被穿插拼接成一个 `JPyjup3eCyJjlkV6DmSmGHQ=`，使用`base64`解密没有明文。继续跟

```
.text:B399B6B4                 LDR.W           R0, [R9]

.text:B399B6B8                 MOV             R1, R8

.text:B399B6BA                 MOVS            R2, #0

.text:B399B6BC                 MOVS            R4, #0

.text:B399B6BE                 LDR.W           R3, [R0,#0x2A4]

.text:B399B6C2                 MOV             R0, R9

.text:B399B6C4                 BLX             R3                //获取输入的字符串

...

...

...

.text:B39B1DA8                 PUSH.W          {R4-R9,LR}

.text:B39B1DAC                 ADD             R7, SP, #0xC

.text:B39B1DAE                 SUB.W           SP, SP, #0x408

.text:B39B1DB2                 SUB             SP, SP, #4

.text:B39B1DB4                 MOV             R9, R0

.text:B39B1DB6                 LDR.W           R0, =(__stack_chk_guard_ptr - 0xB39B1DBE)

.text:B39B1DBA                 ADD             R0, PC ; __stack_chk_guard_ptr

.text:B39B1DBC                 LDR             R0, [R0] ; __stack_chk_guard

.text:B39B1DBE                 LDR             R0, [R0]

.text:B39B1DC0                 STR.W           R0, [R7,#var_10]

.text:B39B1DC4                 BL              sub_B39B231C       //获取199310124853!

.text:B39B1DC8                 MOV             R8, R0

...

...

...

.text:B39B1EE4 loc_B39B1EE4                                       //循环，计算输入字符串长度

.text:B39B1EE4                 LDRB.W          R1, [R0],#1        

.text:B39B1EE8                 CMP             R1, #0

.text:B39B1EEA                 BNE             loc_B39B1EE4

.text:B39B1EEC                 MVN.W           R1, R9

.text:B39B1EF0                 ADDS            R6, R0, R1

.text:B39B1EF2                 ADDS            R4, R6, #1

...

.text:B39B1F80                 MOV             R0, R4  ; size    //根据输入字符串长度分配空间

.text:B39B1F82                 BLX             malloc

.text:B39B1F86                 MOV             R1, R4

.text:B39B1F88                 MOV             R5, R0

.text:B39B1FF0                 BLX             __aeabi_memclr

.text:B39B1FF4                 MOV             R4, SP           

.text:B39B1FF6                 MOVS            R1, #8             //rc4密钥长度

.text:B39B1FF8                 MOV             R0, R4 

.text:B39B1FFA                 MOV             R2, R8             //R8: 199310124853!

.text:B39B20C8                 BL              sub_B399D5E4       //创建一个[0-FF]的数组，并且根据19931012循环打乱
```

## 解密验证

看到这里，又想到一开始的`rc4`(`int`)，猜测是`rc4`加密后转的`base64`，直接用`python`解密，脚本如下

```
from Crypto.Cipher import ARC4

import base64

if __name__ == '__main__':

    key = '19931012'

    cipher = ARC4.new(key)

    print cipher.decrypt(base64.b64decode('JPyjup3eCyJjlkV6DmSmGHQ='))
```

解出 `sn` 为 `madebyericky94528`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
