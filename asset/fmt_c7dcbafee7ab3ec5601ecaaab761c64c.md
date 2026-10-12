
# 小小工具盒 1.11-算法分析(最新 1.22 版也一样)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-67.htm`](https://www.pediy.com/kssd/pediy07/pediy7-67.htm)

整理电脑的时候发现的.是我开始学算法的时候写的，花了整晚.

比较经典的每位取值算法，我想对像我这样的小鸟们一定有帮助，所以厚着脸皮发出来.PS:Winter 兄弟一定也破了-.-

最新版的小小工具盒 1.22 也照样能注册成功，所以说他的算法还是没变。看来作者对自己的算法蛮有信心的：）

还记的那时考 DFCG 时我用的就是其中一篇就是这个，但因软件发布时间超过 15 天被踞之门外..想想我蛮傻的。不过也使我想通了，其实只要自己肯学有毅力，我想在哪里都是一样的吧？

\==================================================================================

【工程作者】深海游侠

【作者邮箱】shenhaiyouxia@163.com

\==================================================================================

【软件名称】小小工具盒 1.11

【下载地址】http://www.onlinedown.net/soft/26370.htm

【软件介绍】

一款十分不错的系统辅助工具！

\==================================================================================

【工程平台】WIN ME

【调试工具】TRW2000 系列

\==================================================================================

## 破解过程

直接 TRW2000 载入，下万能断点，程序断下，`PMODULE:`

```
016F:00405B7A 8D94248C000000   LEA      EDX,[ESP+8C]
016F:00405B81 6880000000       PUSH     DWORD 80
016F:00405B86 52               PUSH     EDX
016F:00405B87 6815040000       PUSH     DWORD 0415
016F:00405B8C 53               PUSH     EBX
016F:00405B8D FFD6             CALL     ESI
016F:00405B8F 50               PUSH     EAX
016F:00405B90 FFD7             CALL     EDI
016F:00405B92 8D84248C000000   LEA      EAX,[ESP+8C]         ------>EAX=假码
016F:00405B99 8D4C240C         LEA      ECX,[ESP+0C]         ------>ECX=用户名
016F:00405B9D 50               PUSH     EAX
016F:00405B9E 51               PUSH     ECX
016F:00405B9F E85C050000       CALL     00406100             ------>关键算法 CALL，跟进
016F:00405BA4 83C408           ADD      ESP,BYTE +08
016F:00405BA7 A388194100       MOV      [00411988],EAX
016F:00405BAC 85C0             TEST     EAX,EAX
016F:00405BAE 5F               POP      EDI
016F:00405BAF 0F8498000000     JZ       NEAR 00405C4D        ------>关键跳
016F:00405BB5 8D542408         LEA      EDX,[ESP+08]
016F:00405BB9 52               PUSH     EDX
016F:00405BBA 6808194100       PUSH     DWORD 00411908
016F:00405BBF FF1508B14000     CALL     `KERNEL32!lstrcpyA`
016F:00405BC5 33C9             XOR      ECX,ECX
016F:00405BC7 8A440C08         MOV      AL,[ESP+ECX+08]
016F:00405BCB 84C0             TEST     AL,AL
016F:00405BCD 740F             JZ       00405BDE
016F:00405BCF 2C02             SUB      AL,02
016F:00405BD1 88440C08         MOV      [ESP+ECX+08],AL
016F:00405BD5 41               INC      ECX
016F:00405BD6 81F980000000     CMP      ECX,80
016F:00405BDC 7CE9             JL       00405BC7
016F:00405BDE 33C9             XOR      ECX,ECX
016F:00405BE0 8A840C88000000   MOV      AL,[ESP+ECX+88]
016F:00405BE7 84C0             TEST     AL,AL
016F:00405BE9 7412             JZ       00405BFD
016F:00405BEB FEC8             DEC      AL
016F:00405BED 88840C88000000   MOV      [ESP+ECX+88],AL
016F:00405BF4 41               INC      ECX
016F:00405BF5 81F980000000     CMP      ECX,80
016F:00405BFB 7CE3             JL       00405BE0
016F:00405BFD 8B3528B14000     MOV      ESI,[0040B128]
016F:00405C03 8D442408         LEA      EAX,[ESP+08]
016F:00405C07 68F8164100       PUSH     DWORD 004116F8
016F:00405C0C 50               PUSH     EAX
016F:00405C0D 68ACD54000       PUSH     DWORD 0040D5AC
016F:00405C12 68D8D04000       PUSH     DWORD 0040D0D8
016F:00405C17 FFD6             CALL     ESI
016F:00405C19 8D8C2488000000   LEA      ECX,[ESP+88]
016F:00405C20 68F8164100       PUSH     DWORD 004116F8        ---> FileName = "D:\Program Files\小小工具盒\ToolBox.ini"
016F:00405C25 51               PUSH     ECX
016F:00405C26 68A8D54000       PUSH     DWORD 0040D5A8        --->Key = "bm"(这里我们不难看出他的成功标志,当然这是用 OD 的情况下，如果你用的 TWR 可以下命令看。我用的就是 TRW)
016F:00405C2B 68D8D04000       PUSH     DWORD 0040D0D8
016F:00405C30 FFD6             CALL     ESI
016F:00405C32 6A40             PUSH     BYTE +40
016F:00405C34 6860D44000       PUSH     DWORD 0040D460
016F:00405C39 6870D64000       PUSH     DWORD 0040D670        ---->Text = "注册成功！ 谢谢您的支持！"
016F:00405C3E 53               PUSH     EBX
016F:00405C3F FF1538B24000     CALL     `USER32!MessageBoxA`  ==== >注册成功 CALL
016F:00405C45 6A01             PUSH     BYTE +01
016F:00405C47 53               PUSH     EBX
016F:00405C48 E9E4FeFFFF       JMP      00405B31
016F:00405C4D 6A40             PUSH     BYTE +40
016F:00405C4F 686CD64000       PUSH     DWORD 0040D66C
016F:00405C54 6850D64000       PUSH     DWORD 0040D650        ----->Text = "注册码不正确，请重新输入！"
016F:00405C59 53               PUSH     EBX
016F:00405C5A FF1538B24000     CALL     `USER32!MessageBoxA`  ----->注册失败 CALL
016F:00405C60 6814040000       PUSH     DWORD 0414
016F:00405C65 53               PUSH     EBX
016F:00405C66 FFD6             CALL     ESI
016F:00405C68 50               PUSH     EAX
016F:00405C69 FF15B4B14000     CALL     `USER32!SetFocus`
016F:00405C6F 5E               POP      ESI
016F:00405C70 33C0             XOR      EAX,EAX
016F:00405C72 5B               POP      EBX
016F:00405C73 81C400010000     ADD      ESP,0100
016F:00405C79 C21000           RET      10
```

关键算法 CALL:

```
016F:00405B9F E85C050000       CALL     00406100
|
016F:00406100 83EC2C           SUB      ESP,BYTE +2C
016F:00406103 53               PUSH     EBX
016F:00406104 55               PUSH     EBP
016F:00406105 8B6C2438         MOV      EBP,[ESP+38]
016F:00406109 56               PUSH     ESI
016F:0040610A 57               PUSH     EDI
016F:0040610B 8BFD             MOV      EDI,EBP
016F:0040610D 83C9FF           OR       ECX,BYTE -01          ---->ECX or -1
016F:00406110 33C0             XOR      EAX,EAX
016F:00406112 F2AE             REPNE SCASB                   ---->串命令
016F:00406114 F7D1             NOT      ECX                  ---->求补
016F:00406116 49               DEC      ECX                  ---->ECX=ECX-1(上面几步到这里等于变相把用户名位数入 ECX)
016F:00406117 C744241C00000000 MOV      DWORD [ESP+1C],00
016F:0040611F 83F904           CMP      ECX,BYTE +04          ---->位数和 4 比较
016F:00406122 BE01000000       MOV      ESI,01
016F:00406127 894C2418         MOV      [ESP+18],ECX
016F:0040612B 7D07             JNL      00406134              ---->大于等于就跳
016F:0040612D 8B442444         MOV      EAX,[ESP+44]          ---->EAX 假码
016F:00406131 C60000           MOV      BYTE [EAX],00         ---->上面跳就是要避开这里，这里等于把你注册码第一位赋 00，00 什么都不是了。怎么比较？
016F:00406134 8A5C2444         MOV      BL,[ESP+44]           ---->跳到这里
016F:00406138 33FF             XOR      EDI,EDI
016F:0040613A 85C9             TEST     ECX,ECX
016F:0040613C 0F8E31010000     JNG      NEAR 00406273         ---->是否输入用户名？
016F:00406142 83F908           CMP      ECX,BYTE +08          ---->是否小于等于 8 位
016F:00406145 7E08             JNG      0040614F
016F:00406147 83FF05           CMP      EDI,BYTE +05
016F:0040614A 7E03             JNG      0040614F
016F:0040614C 8D79FE           LEA      EDI,[ECX-02]
016F:0040614F 8D5701           LEA      EDX,[EDI+01]
016F:00406152 33C0             XOR      EAX,EAX
016F:00406154 3BD1             CMP      EDX,ECX
016F:00406156 8D4C2424         LEA      ECX,[ESP+24]
016F:0040615A 0F9CC0           SETL     AL                    ---->这里要看上面计算，在这种情况下 AL 一定会是 1 吧？
016F:0040615D 33D2             XOR      EDX,EDX               ---->EDX 清 0
016F:0040615F 6A24             PUSH     BYTE +24              ---->36 压栈，事后看这个也蛮重要的，后面计算要靠他了。
016F:00406161 8A142F           MOV      DL,[EDI+EBP]          ---->用户名第一位以 ASCII 码形式入 AL
016F:00406164 51               PUSH     ECX
016F:00406165 52               PUSH     EDX
```
