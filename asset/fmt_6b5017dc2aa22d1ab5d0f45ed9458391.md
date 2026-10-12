
# OICQ 图形留言系统

> 原文：[`www.pediy.com/kssd/pediy03/forum597.htm`](https://www.pediy.com/kssd/pediy03/forum597.htm)

```
0167:004BACDE  MOV       EAX,[EBP-0C]       EAX 中是我的用户名

0167:004BACE1  CALL    00403E38

0167:004BACE6  MOV       EDI,EAX

0167:004BACE8  TEST      EDI,EDI

0167:004BACEA  JNG       004BAD24

0167:004BACEC  MOV       EBX,01                这个 1 后来要用到

0167:004BACF1  LEA       EAX,[EBP-10]

0167:004BACF4  PUSH      EAX

0167:004BACF5  MOV       ECX,01

0167:004BACFA  MOV       EDX,EBX

0167:004BACFC  MOV       EAX,[EBP-0C]

0167:004BACFF  CALL    00404040

0167:004BAD04  MOV       EAX,[EBP-10]

0167:004BAD07  CALL    00403FFC

0167:004BAD0C  MOV       AL,[EAX]           将用户名的第一个字符放入 AL

0167:004BAD0E  LEA       EDX,[EBX+17]      EDX=1+23=24

0167:004BAD11  IMUL      EDX,ESI           此时 ESI=0，EDX=0

0167:004BAD14  AND       EAX,FF

0167:004BAD19  IMUL      EAX,EAX,BYTE +17  AL 乘以 23（即第一个字符乘以 23）

0167:004BAD1C  ADD       EDX,EAX           EDX=EDX+EAX

0167:004BAD1E  MOV       ESI,EDX           ESI=EDX

0167:004BAD20  INC       EBX               EBX+1

0167:004BAD21  DEC       EDI               EDI-1（EDI 是用户名的长度）

0167:004BAD22  JNZ       004BACF1          不为零则跳（即还有用户名字符没有计算）

0167:004BAD24  CMP       ESI,[EBP-08]      ESI 中是正确的注册码，EBP-8 中是你输入的注册码
```

示例：用户名为 `123`，注册码输入 `321`

计算：真实注册码=`{[(49x23+24x0)x25+50x23]x26+51x23}=763623`

注册机写起来不难

只要在`HKEY_CURRENT_USER\Software\ABCSoft\Oicqpic`新建字符串值`Name`和`DOWRD`值`Pass`即可完成注册`4dd466d0e1e0cc123afbe504da2462ca`
