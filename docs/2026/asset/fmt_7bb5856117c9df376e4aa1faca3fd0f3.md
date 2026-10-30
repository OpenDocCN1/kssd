
```
00413131    83C4 F0         ADD ESP,-0x10

00413150    33C0            XOR EAX,EAX                              ; EAX清零

00413184    A3 34B04100     MOV DWORD PTR DS:[0x41B034],EAX          ; 全局变量[0x41B034]清零

004131BA    58              POP EAX                                  ; EAX=输入的1-4位，例如7473754A"Just"，记为a

004131EB    8BC8            MOV ECX,EAX                              ; ECX=EAX

0041321F    58              POP EAX                                  ; EAX=输入的5-8位，例如726F6630"0for"，记为b

00413254    8BD8            MOV EBX,EAX                              ; EBX=EAX

00413289    58              POP EAX                                  ; EAX=输入的第9-12位，例如6E756630 "0fun"，记为c

004132B5    8BD0            MOV EDX,EAX                              ; EDX=EAX

004132AD    8BD0            MOV EDX,EAX                              ; EDX=EAX

004132E2    8BC1            MOV EAX,ECX                              ; EAX=ECX

00413316    2BC3            SUB EAX,EBX                              ; EAX=EAX-EBX=a-b

00413349    C1E0 02         SHL EAX,0x2                              ; EAX=EAX*4=4*(a-b)

00413380    03C1            ADD EAX,ECX                              ; EAX=EAX+ECX=4*(a-b)+a

004133B5    03C2            ADD EAX,EDX                              ; EAX=EAX+EDX=4*(a-b)+a+c

004133E9    2D E217F9EA     SUB EAX,0xEAF917E2                       ; EAX=EAX-0xEAF917E2=4*(a-b)+a0xEAF917E2==0……⑴

00413455    03C1            ADD EAX,ECX                              ; EAX=ECX=a

00413489    2BC3            SUB EAX,EBX                              ; EAX=EAX-EBX=a-b

004134BF    8BD8            MOV EBX,EAX                              ; EBX=EAX

004134F3    D1E0            SHL EAX,1                                ; EAX=EAX*2=2*(a-b)

00413525    03C3            ADD EAX,EBX                              ; EAX=EAX+EBX=3*(a-b)

00413559    03C1            ADD EAX,ECX                              ; EAX=EAX+ECX=3*(a-b)+a

0041358F    8BC8            MOV ECX,EAX                              ; ECX=EAX

004135C3    03C2            ADD EAX,EDX                              ; EAX=EAX+EDX=3*(a-b)+a+c

004135F7    2D C808F5E8     SUB EAX,0xE8F508C8                       ; EAX=EAX-0xE8F508C8=3*(a-b)+a0xE8F508C8==0……⑵

00413665    8BC1            MOV EAX,ECX                              ; EAX=ECX=3*(a-b)+a

0041365D    8BC1            MOV EAX,ECX                              ; EAX=ECX

004136A7    2BC2            SUB EAX,EDX                              ; EAX=EAX-EDX=3*(a-b)c

004136D8    2D 683C0A0C     SUB EAX,0xC0A3C68                        ; EAX=EAX-0xC0A3C68=3*(a-b)c-0xC0A3C68==0……⑶

00413747    58              POP EAX                                  ; EAX=0x413131

00413777    35 01810000     XOR EAX,0x8101                           ; EAX=EAX^0x8101=0x41B030

004137A9    8BF8            MOV EDI,EAX                              ; EDI=EAX

004137E2    33C0            XOR EAX,EAX                              ; EAX=0

00413817    AB              STOS DWORD PTR ES:[EDI]

0041385C    58              POP EAX                                  ; EAX=0x413835

0041388E    50              PUSH EAX                                 ; ctf2017_.00413835

004138BA    8BF8            MOV EDI,EAX                              ; ctf2017_.00413835

004138B2    8BF8            MOV EDI,EAX                              ; ctf2017_.00413835

004138E6    68 6909004E     PUSH 0x4E000969

0041391F    58              POP EAX                                  ; EAX=x4E000969

00413950    33C2            XOR EAX,EDX                              ; EAX=EAX^EDX=20756F59

00413983    AB              STOS DWORD PTR ES:[EDI]

004139B5    35 3E0A0100     XOR EAX,0x10A3E                          ; EAX=EAX^0x10A3E=20746567

004139EB    AB              STOS DWORD PTR ES:[EDI]

00413A1C    33C3            XOR EAX,EBX                              ; EAX=EAX^EBX=22706A7D

00413A4D    35 141E5122     XOR EAX,0x22511E14                       ; EAX=EAX^0x22511E14=00217469

00413A82    AB              STOS DWORD PTR ES:[EDI]

00413AB6    35 2D646100     XOR EAX,0x61642D                         ; EAX=EAX^0x61642D=00401044

00413B83    3305 34B04100   XOR EAX,DWORD PTR DS:[0x41B034]          ; EAX=EAX^0=0x401044

00413BBB  ^\FFE0            JMP EAX                                  ; JMP 0x401044 飞向光明顶

esp:

0018FF48   00413835  ASCII "You get it!"
```

这里得到了三组方程：

```
⑴：4*(a-b)+a0xEAF917E2==0

⑵：3*(a-b)+a0xE8F508C8==0

⑶：3*(a-b)c-0xC0A3C68==0
```

三个未知数，三组方程可以求解了，化简、笔算也可以，如果图省事那就：

```
#!/usr/bin/env python3

# -*- coding: utf-8 -*-

# ctf.pediy.com TSRC 2017CTF crackme 2 keygen

# python 3.6.0 64bit

# written by 爱琴海

# 2017/10/26



from sympy import *



def Main():

	a=Symbol('a')

	b=Symbol('b')

	c=Symbol('c')

	print (solve([4*(a-b)+a0xEAF917E2,3*(a-b)+a0xE8F508C8,3*(a-b)c-0xC0A3C68],[a,b,c]))



if __name__=="__main__":

	Main()
```

解出：{`a`: 1953723722, `b`: 1919903280, `c`: 1853187632} 转十六进制：{`a`: 0x7473754A, `b`: 0x726F6630, `c`: 0x6E756630} 得到答案：“Just0for0fun11A”

### 五、总结

作者通过显性算法坑、溢出隐藏算法流程、花指令干扰分析、加密成功出口等手段对抗破解，有意思。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
