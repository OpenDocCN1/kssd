
```
0187:004BB870  MOV       ECX,EAX

0187:004BB872  SHL       BL,CL            <---假注册码的奇数位

0187:004BB874  LEA       EAX,[EBP-14]

0187:004BB877  MOV       EDX,EBX

0187:004BB879  CALL      00403E34

0187:004BB87E  MOV       EDX,[EBP-14]

0187:004BB881  MOV       EAX,ESI

0187:004BB883  CALL      00403F14

0187:004BB888  LEA       EAX,[EBP-18]

0187:004BB88B  MOV       DL,[EBP-06]

0187:004BB88E  CALL      00403E34

0187:004BB893  MOV       EDX,[EBP-18]

0187:004BB896  MOV       EAX,ESI

0187:004BB898  CALL      00403F14

0187:004BB89D  JMP       SHORT 004BB8C4   <---保存注册码后跳到后面再循环
```

如果 `Code1` 的数据是小于 `7E` 而且大于 `21`,也就是可见的字符串的话就会跳到这里来,这时 `BL` 就是假注册码的奇数位了,看看下面的 `Call` 都是保存注册码的,那这时假注册码的偶数位在哪呢?请看 `4BB8BA` 处

```
0187:004BB89F  LEA       EAX,[EBP-1C]

0187:004BB8A2  MOV       EDX,[EBP-04]

0187:004BB8A5  MOV       DL,[EDX+EDI-01]

0187:004BB8A9  CALL      00403E34

0187:004BB8AE  MOV       EDX,[EBP-1C]

0187:004BB8B1  MOV       EAX,ESI

0187:004BB8B3  CALL      00403F14

0187:004BB8B8  MOV       EAX,ESI

0187:004BB8BA  MOV       EDX,004BB90C     <---试试 D 4BB90C,是否看到字符"C",偶数位就是它了,不用怀疑,因为这个"C"是在程序的数据段里的,而且整个系列的都是如此

0187:004BB8BF  CALL      00403F14

0187:004BB8C4  INC       BYTE [EBP-05]

0187:004BB8C7  DEC       BYTE [EBP-08]

0187:004BB8CA  JNZ       NEAR 004BB7E8

0187:004BB8D0  XOR       EAX,EAX

0187:004BB8D2  POP       EDX

0187:004BB8D3  POP       ECX

0187:004BB8D4  POP       ECX

0187:004BB8D5  MOV       [FS:EAX],EDX

0187:004BB8D8  PUSH      DWORD 004BB8FA

0187:004BB8DD  LEA       EAX,[EBP-1C]

0187:004BB8E0  MOV       EDX,05

0187:004BB8E5  CALL      00403CB0

0187:004BB8EA  LEA       EAX,[EBP-04]

0187:004BB8ED  CALL      00403C8C

0187:004BB8F2  RET
```

OK~~~~~~~~我们的任务算是完成了,至于 Y 逆算法我就不贴出来了,有兴趣的就自己看看,在 `4BB63F` 处开始,因为要写注册机根本就不用两个逆算法.我们现在来讨论一下其它几个程序的不同.

其实 `FreeRes0.94`、`HexEdit0.20`、`GetVBRes0.51` 的算法都是一样的,`HexEdit0.20` 甚至连机器码都相同,不同在哪里呢?其实区别就是算法 X 中的 3 个重要数据

`FreeRes0.94` 中的是(16 进制): `3D6`、`3183`、`8C34`

`HexEdit0.20` 中的是(16 进制): `3D7`、`3184`、`8C35`

`GetVBRes0.51` 中的是(16 进制): `3D8`、`3185`、`8C36`

另外 `GetVBRes0.51` 的 `Name+HardCode` 的补位有了变化,而 `FreeRes0.94`、`HexEdit0.20` 的注册名不能大于 `10` 位,这个我还不能完全肯定.

下面是 `GetVBRes0.51` 的补位

```
============================================
```

`GetVBRes0.51` 应该没有注册名位数限制,`40` 位也没问题,但最后也是只取 `20` 位.如果 `Name+HardCode` 的长度小于等于 `18` 位或等于 `19`、`20` 位的话,补位和 `FreeRes` 一样,如果大于 `20` 位的话就要....

```
0187:0049AA63  CMP       ESI,BYTE +12     <----esi 是 Name+HardCode 的长度

0187:0049AA66  JNG       0049AAD4

0187:0049AA68  MOV       EAX,[EBP-04]

0187:0049AA6B  MOV       BL,[EAX+12]      <----取出 Name+HardCode 的第 18 位

0187:0049AA6E  MOV       EAX,ESI

0187:0049AA70  SUB       AL,14             <----算出多出多少位

0187:0049AA72  JC        0049AAA5

0187:0049AA74  INC       EAX

0187:0049AA75  MOV       [EBP-15],AL      <---加 1 作为计数器

0187:0049AA78  MOV       AL,14

0187:0049AA7A  XOR       EDX,EDX

0187:0049AA7C  MOV       DL,AL

0187:0049AA7E  MOV       ECX,[EBP-04]

0187:0049AA81  MOV       DL,[ECX+EDX-01]  <---取出第 20 位

0187:0049AA85  CMP       BL,DL

0187:0049AA87  JNA       0049AA94

0187:0049AA89  XOR       ECX,ECX

0187:0049AA8B  MOV       CL,AL

0187:0049AA8D  MOV       EDI,[EBP-04]

0187:0049AA90  SUB       BL,DL

0187:0049AA92  JMP       SHORT 0049AA9F

0187:0049AA94  XOR       ECX,ECX

0187:0049AA96  MOV       CL,AL

0187:0049AA98  MOV       EDI,[EBP-04]

0187:0049AA9B  SUB       DL,BL

0187:0049AA9D  MOV       EBX,EDX          <---用大数减去小数保存在 ebx

0187:0049AA9F  INC       EAX

0187:0049AAA0  DEC       BYTE [EBP-15]

0187:0049AAA3  JNZ       0049AA7A         <---当循环完后 BL 就是 Code1 的第一位了,然后再取

                                           Name+HardCode 的前 18 位补上
```

`GetVBRes0.51` 注册后是否有些人看不到自己的注册名,`ResTools` 系列的注册信息保存在 `HKEY_LOCAL_MACHINE\SOFTWARE\RESTOOLS` 下,只要你注册 `ResFree` 后就可在 `GetVBRes` 看到 `ResFree` 的注册名了,注意在 `GetVBRes` 中用任意名注册都只会显示 `HKEY_LOCAL_MACHINE\SOFTWARE\RESTOOLS\freeRes` 下 `reguser` 里的字串,注册不了的可以自己建立一个.

再讲讲 `ResScope1.35`、`FreeRes0.60`

两个注册算法一样,只用 A、B 两步,注册名不能大于 `18` 位,补位规则同上一样,要注意的是,调试的时候,填好注册名和注册码后按确定,程序并不会比较注册码,只是简单的将名和码写到注册表中,当你再次点击帮助菜单时,程序才会进行运算比较.它们的重要数据:

`ResScope1.35` 中的是(16 进制): `3D5`、`3182`、`8C33`

`FreeRes0.60` 中的是(16 进制): `3D6`、`3183`、`8C34

-=End=-

```
      _/_/_/
    _/          _/_/_/  _/_/_/  _/_/
    _/_/    _/    _/  _/    _/    _/
        _/  _/    _/  _/    _/    _/
_/_/_/      _/_/_/  _/    _/    _/
                                                    Sam.com
                                                    6:04 2002-3-3  4dd466d0e1e0cc123afbe504da2462ca
```
