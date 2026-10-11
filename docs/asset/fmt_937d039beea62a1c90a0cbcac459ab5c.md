
```
00642EDE  |.MOV EAX,DWORD PTR SS:[EBP-24]    ;
00642EE1  |.CALL ICONCRAF.00484104           ;iconcraf.00484104
00642EE6  |.LEA EAX,DWORD PTR SS:[EBP-70]
00642EE9  |.CALL ICONCRAF.006FFB18
00642EEE  |.MOV ECX,DWORD PTR SS:[EBP-70]
00642EF1  |.MOV EDX,ICONCRAF.00643104        ;ASCII "FullProductName"
00642EF6  |.MOV EAX,DWORD PTR SS:[EBP-24]
00642EF9  |.CALL ICONCRAF.00484060
00642EFE  |.LEA EDX,DWORD PTR SS:[EBP-74]
00642F01  |.MOV EAX,DWORD PTR SS:[EBP-4]
00642F04  |.MOV EAX,DWORD PTR DS:[EAX+2F0]
00642F0A  |.CALL ICONCRAF.00437B28
00642F0F  |.MOV EAX,DWORD PTR SS:[EBP-74]
00642F12  |.LEA EDX,DWORD PTR SS:[EBP-28]
00642F15  |.CALL ICONCRAF.004098EC
00642F1A  |.CMP DWORD PTR SS:[EBP-20],1
00642F1E  |.JE SHORT ICONCRAF.00642F52
00642F20  |.PUSH DWORD PTR SS:[EBP-28]
00642F23  |.PUSH ICONCRAF.0064311C           ;ASCII "<BR>"
00642F28  |.PUSH ICONCRAF.0064312C           ;ASCII " - "
00642F2D  |.LEA EDX,DWORD PTR SS:[EBP-78]
00642F30  |.MOV EAX,DWORD PTR SS:[EBP-20]
00642F33  |.CALL ICONCRAF.00409AA0
00642F38  |.PUSH DWORD PTR SS:[EBP-78]
00642F3B  |.PUSH ICONCRAF.006430A4
00642F40  |.PUSH ICONCRAF.00643138           ;ASCII "licenses -"
00642F45  |.LEA EAX,DWORD PTR SS:[EBP-28]
00642F48  |.MOV EDX,6
00642F4D  |.CALL ICONCRAF.0040424C
00642F52  |>MOV ECX,DWORD PTR SS:[EBP-28]
00642F55  |.MOV EDX,ICONCRAF.0064314C        ;ASCII "UserName"
00642F5A  |.MOV EAX,DWORD PTR SS:[EBP-24]
00642F5D  |.CALL ICONCRAF.00484060
00642F62  |>XOR EAX,EAX
00642F64  |.POP EDX
00642F65  |.POP ECX
00642F66  |.POP ECX
00642F67  |.MOV DWORD PTR FS:[EAX],EDX
00642F6A  |.PUSH ICONCRAF.00642F7F
00642F6F  |>MOV EAX,DWORD PTR SS:[EBP-24]
00642F72  |.CALL ICONCRAF.00403194
00642F77  \\.RETN
```

呵呵,一看前面一大段都是在检查 `- KEY BEGIN KEY -` 和 `- KEY END KEY -` 字符串,就不多说了,关键是在 `642E2C` 处,其内容为,

```
00700D10  /$PUSH EBX
00700D11  |.MOV EBX,EAX
00700D13  |.CMP BYTE PTR DS:[EBX],30         ;  注册码第一位与 0 比较
00700D16  |.JNZ SHORT ICONCRAF.00700D2F       ;  一跳就死
00700D18  |.MOV EAX,EBX
00700D1A  |.CALL ICONCRAF.0040418C           ;  计算注册码长度
00700D1F  |.CMP EAX,0AD                      ;  注册码长度与 0xAD(173) 比较
00700D24  |.JNZ SHORT ICONCRAF.00700D2F       ;  一跳就死
00700D26  |.CMP BYTE PTR DS:[EBX+AC],3D      ;  注册码最后一位与 '=' 比较
00700D2D  |.JE SHORT ICONCRAF.00700D45        ;  不跳就死
00700D2F  |>MOV ECX,ICONCRAF.00700D50        ;  ASCII "Invalid key"
00700D34  |.MOV DL,1
00700D36  |.MOV EAX,DWORD PTR DS:[408A60]
00700D3B  |.CALL ICONCRAF.0040D134
00700D40  |.CALL ICONCRAF.0040392C
00700D45  |>POP EBX
00700D46  \\.RETN
```

注册算法竟然这么简单!

其流程为,

1. 首先检查注册码中是否含 `- KEY BEGIN KEY -` 和 `- KEY END KEY -` 字符串,如果有,则取它们之间的字符为注册码.空格会自动删除.可以为中文.
2. 获得注册码后,查找注册码中 `=` 的个数.若 `=` 个数为 0,则调用注册失败对话框.
3. 当 `=` 只有一个时,截取 `=` 以左部分(包括 `=`)作为新注册码,验证其首位是否为 `0`,末尾是否为 `=`,长度是否为 `0xAD`(173d),是则注册成为单用户协议.
4. 当 `=` 多于一个时,逐次将注册码分为 `......=` 格式的若干段,对每一段都进行首位是否为 `0`,末尾是否为 `=`,长度是否为 `0xAD`(173d)的验证,只要有一段符合要求,协议将注册为多用户协议,协议个数是 `=` 的个数.
5. 写入注册表项.

【破解总结】:

该注册算法是俺有史以来,见过的最简单的注册算法,但对 Cracker 新人来讲,多少有点意义,所以写出来,最后给出一个弱※注册码,供大家玩笑,

```
- KEY BEGIN KEY -
0 鲁鲁拉,鲁鲁拉,弱※算法,鲁鲁拉!你笑它,我笑它,哈哈哈哈哈哈.鲁鲁拉,鲁鲁拉,弱※算法,鲁鲁拉!你笑它,我笑它,哈哈哈哈哈哈.鲁鲁拉,鲁鲁拉,弱※算法,鲁鲁拉!你笑它,我笑它,哈哈哈哈哈哈.=
- KEY END KEY -
```

然后,补一个注册机,这个注册机并非完整意义上的注册机,因为不能列出所有可能的注册码,当然如果非要写,也并非不可,只是觉得没必要.为这个简单注册码算法,写一个复杂注册机,不值得.

高手就不要来了.

**注意:该注册算法同样可以注册 `IconoMaker` 3.20 及 `Perfect` `Icon` 2.30**

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `KeyGen.rar`

|
