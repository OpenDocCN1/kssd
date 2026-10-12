
# CD Audio MP3 Converter 3.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/76028.html`](https://www.pediy.com/kssd/pediy10/76028.html)

【文章标题】: `CD Audio MP3 Converter 3.0` 算法分析

【文章作者】: `iouken/de` 神

【作者邮箱】: `sos_ftp@yahoo.com.cn`

【作者主页】: `http://hi.baidu.com/天蝎型男`

【作者 QQ 号】: `250771765`

【软件名称】: `CD Audio MP3 Converter 3.0 Build 20081103`

【软件大小】: `1.56M`

【下载地址】: 自己搜索下载

【加壳方式】: `ASPack 2.12`

【保护方式】: 注册码

【编写语言】: `Delphi 6.0 - 7.0`

【使用工具】: `PEID`, `OD`

【操作平台】: 100%的盗版 `Windows XP2`

【软件介绍】: 一款 CD 音频提取软件。能够将提取出来的音频保存为 `mp3` 格式。同时能够从 `CDDB` 数据库中提取标题信息，归一化输出文件和支持额外的 `LAME` 选项。另外还可以通过麦克风录制声音，并保存为 `Mp3` 格式。

【作者声明】: 仅仅只是一场游戏而已，没有其他目的。失误之处敬请诸位大侠赐教!

【详细过程】

一、用 `PEID` 对程序进行查壳 → `ASPack 2.12` -> `Alexey Solodovnikov`

`ASPack` 的壳很简单的，脱壳我就不多说了，`ESP` 定律很快就到 `OEP` 了。我多数都是带壳调试的，懒得去脱了。嘻嘻。

二、用 `OD` 载入程序进行分析。

载入 `OD` 后运行程序，试注册一下，有错误提示。用 `F12` 暂停法来到下面的地方。

引用：

```
004E19F1    55               PUSH EBP                                   ; F2 在这下断
004E19F2    68 0F1B4E00      PUSH cdextrac.004E1B0F
004E19F7    64:FF30          PUSH DWORD PTR FS:[EAX]
004E19FA    64:8920          MOV DWORD PTR FS:[EAX],ESP
004E19FD    8D55 FC          LEA EDX,DWORD PTR SS:[EBP-4]
004E1A00    8B83 44030000    MOV EAX,DWORD PTR DS:[EBX+344]
004E1A06    E8 4D1CF8FF      CALL cdextrac.00463658                       ; 取试练码长度
004E1A0B    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004E1A0E    E8 FD8DFCFF      CALL cdextrac.004AA810                       ; 关键 CALL，F7
004E1A13    84C0             TEST AL,AL                                   ; 标志位比较，这里 AL 的值如果等于 1 就注册成功，否则就 GAME OVER
004E1A15    75 3E            JNZ SHORT cdextrac.004E1A55                  ; 关键跳
004E1A17    6A 10            PUSH 10
```

跟进关键 `CALL` 后来到以下地方。

引用：

```
..........省略一些代码......

004AA82C    8BD6             MOV EDX,ESI
004AA82E    E8 21A1F5FF      CALL cdextrac.00404954
004AA833    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004AA836    E8 41A3F5FF      CALL cdextrac.00404B7C                       ; 取试练码长度给 EAX
004AA83B    83F8 06          CMP EAX,6                                    ; 试练码长度和 6h 比较
004AA83E    7C 4B            JL SHORT cdextrac.004AA88B                   ; 试练码位数小于 6 位就 GAME OVER
004AA840    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004AA843    33C9             XOR ECX,ECX                                  ; ECX 清零
004AA845    8A08             MOV CL,BYTE PTR DS:[EAX]                     ; 取试练码第 1 位 ASCII 值给 CL
004AA847    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004AA84A    0FB640 04        MOVZX EAX,BYTE PTR DS:[EAX+4]                ; 取试练码第 5 位 ASCII 值给 EAX
004AA84E    03C8             ADD ECX,EAX                                    ; ECX＝ECX+EAX
004AA850    8BC1             MOV EAX,ECX                                    ; 把相加后的结果送给 EAX
004AA852    83E8 60          SUB EAX,60                                    ; EAX＝EAX－60
004AA855    83F8 08          CMP EAX,8                                    ; 相减后的结果和 8h 比较
004AA858    7E 31            JLE SHORT cdextrac.004AA88B                  ; 小于等于 8 就 GAME OVER
004AA85A    8BC1             MOV EAX,ECX                                    ; 把上面相加后的结果送给 EAX
004AA85C    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EDX
004AA85F    0FB652 01        MOVZX EDX,BYTE PTR DS:[EDX+1]                ; 取试练码第 2 位 ASCII 值给 EDX
004AA863    03C2             ADD EAX,EDX                                    ; EAX＝EAX+EDX
004AA865    2D 90000000      SUB EAX,90                                    ; EAX＝EAX－90
004AA86A    B9 0A000000      MOV ECX,0A                                    ; ECX＝0A
004AA86F    99               CDQ                                           ; EDX 清零
004AA870    F7F9             IDIV ECX                                       ; EAX 除以 ECX，商送给 EAX，余数送给 EDX
004AA872    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004AA875    0FB640 03        MOVZX EAX,BYTE PTR DS:[EAX+3]                ; 取试练码第 4 位 ASCII 值给 EAX
004AA879    83E8 30          SUB EAX,30                                    ; EAX＝EAX－30
004AA87C    3BD0             CMP EDX,EAX                                    ; 上面相除后的余数和 EAX 的值比较
004AA87E    75 0B            JNZ SHORT cdextrac.004AA88B                  ; 不相等就 GAME OVER
004AA880    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]                 ; 把试练码送给 EAX
004AA883    8078 02 35       CMP BYTE PTR DS:[EAX+2],35                   ; 试练码第 3 位 ASCII 值和 35h 比较
004AA887    76 02            JBE SHORT cdextrac.004AA88B                  ; 第 3 位 ASCII 值低于等于 35h 就 GAME OVER
004AA889    B3 01            MOV BL,1                                      ; 注册码错误 BL＝0，注册码正解则 BL＝1
004AA88B    33C0             XOR EAX,EAX                                    ; EAX 清零
004AA88D    5A               POP EDX
```

【经验总结】

易语言注册机源码：

引用：

```
.版本 2
.局部变量 容器
.局部变量 容器 2
.局部变量 容器 3
.局部变量 容器 4
.局部变量 容器 5
.局部变量 临时容器
.局部变量 i
.判断开始 (编辑框 2.内容 ＝ “”)
    编辑框 2.内容 ＝ “请输入用户名！”
.默认
    置随机数种子 (取启动时间 ())
    .变量循环首 (1, 1000, 1, i)
        容器 ＝ 取随机数 (49, 57)
        容器 5 ＝ 取随机数 (49, 57)
        临时容器 ＝ 容器 ＋ 容器 5 － 96
        .判断开始 (临时容器 ＞ 8)
            容器 2 ＝ 取随机数 (49, 57)
            临时容器 ＝ 容器 ＋ 容器 5 ＋ 容器 2 － 144
            临时容器 ＝ 临时容器 ％ 10
            容器 4 ＝ 临时容器 ＋ 48
            容器 3 ＝ 取随机数 (54, 57)
            跳出循环 ()
        .默认
        .判断结束
    .变量循环尾 ()
    编辑框 3.内容 ＝ 字符 (容器) ＋ 字符 (容器 2) ＋ 字符 (容器 3) ＋ 字符 (容器 4) ＋ 字符 (容器 5) ＋ 到文本 (
    取随机数 (1, 9999999))
```

【版权声明】: 本文原创于 `de` 神, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 11 月 04 日 14:26:21
