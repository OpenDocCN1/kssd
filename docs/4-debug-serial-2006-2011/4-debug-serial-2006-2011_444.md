
# RegistryEasy 4.9 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/79608.html`](https://www.pediy.com/kssd/pediy10/79608.html)

【破文标题】RegistryEasy 4.9 简单分析

【破文作者】tianxj

【作者邮箱】tianxj_2007@126.com

【作者主页】WwW.ChiNaPYG.CoM

【破解工具】`PEiD`, `OD`

【破解平台】Windows XP sp3(雨林木风)

【软件名称】RegistryEasy 4.9

【软件大小】2480KB

【软件类别】国外软件/卸载工具

【软件授权】共享版

【软件语言】英文

【运行环境】Win9x/Me/NT/2000/XP/2003

【更新时间】2008-12-28

【原版下载】华军软件园

【保护方式】注册码

【软件简介】清洁扫描您的 PC 并且安全清除由于错误及无效的操作造成系统速度变慢，死机和崩溃的文件。registryeasy 修理注册表的问题，使您的计算机快速运行！registryeasy 使用了一个高性能的检测算法，将迅速查明在您的 Windows 注册表中消失和无效的参数。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

--------------------------------------------------------------------------------

一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

"Invalid serial number! Please try again..."

--------------------------------------------------------------------------------

二、用 `PEiD` 对 `RE.exe` 查壳，为 Borland Delphi 6.0 - 7.0

--------------------------------------------------------------------------------

三、运行 `OD`，打开 `RE.exe`，右键超级字串参考查找 `ASCII`.

==============================================================

代码:



```
004D4E1C  /$  55              PUSH EBP
004D4E1D  |.  8BEC            MOV EBP,ESP
004D4E1F  |.  B9 0A000000     MOV ECX,0A
004D4E24  |>  6A 00           /PUSH 0
004D4E26  |.  6A 00           |PUSH 0
004D4E28  |.  49              |DEC ECX
004D4E29  |.^ 75 F9           \JNZ SHORT RE.004D4E24
004D4E2B  |.  53              PUSH EBX
004D4E2C  |.  56              PUSH ESI
004D4E2D  |.  8BD8            MOV EBX,EAX
004D4E2F  |.  33C0            XOR EAX,EAX
004D4E31  |.  55              PUSH EBP
004D4E32  |.  68 84504D00     PUSH RE.004D5084
004D4E37  |.  64:FF30         PUSH DWORD PTR FS:[EAX]
004D4E3A  |.  64:8920         MOV DWORD PTR FS:[EAX],ESP
004D4E3D  |.  8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]
004D4E40  |.  8B83 10030000   MOV EAX,DWORD PTR DS:[EBX+310]
004D4E46  |.  E8 91C8F9FF     CALL RE.004716DC
004D4E4B  |.  8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]              ; //试炼码
004D4E4E  |.  8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]
004D4E51  |.  E8 FE42F3FF     CALL RE.00409154
004D4E56  |.  8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]              ; //试炼码
004D4E59  |.  8D55 FC         LEA EDX,DWORD PTR SS:[EBP-4]
004D4E5C  |.  E8 DF40F3FF     CALL RE.00408F40                            ; //将试炼码转小写
004D4E61  |.  A0 94504D00     MOV AL,BYTE PTR DS:[4D5094]
004D4E66  |.  50              PUSH EAX
004D4E67  |.  8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]
004D4E6A  |.  50              PUSH EAX
004D4E6B  |.  33C9            XOR ECX,ECX
004D4E6D  |.  BA A0504D00     MOV EDX,RE.004D50A0                         ; -
004D4E72  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]               ; //小写试炼码
004D4E75  |.  E8 DE9CF3FF     CALL RE.0040EB58                            ; //将小写试炼码中的"-"去掉,设为字符串 A
004D4E7A  |.  8B55 E4         MOV EDX,DWORD PTR SS:[EBP-1C]              ; //字符串 A
004D4E7D  |.  8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
004D4E80  |.  E8 DFFCF2FF     CALL RE.00404B64
004D4E85  |.  8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]
004D4E88  |.  50              PUSH EAX
004D4E89  |.  B9 08000000     MOV ECX,8
004D4E8E  |.  BA 09000000     MOV EDX,9
004D4E93  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]               ; //字符串 A
004D4E96  |.  E8 5101F3FF     CALL RE.00404FEC                            ; //取字符串 A 的 9-16 位
004D4E9B  |.  8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]              ; //字符串 A 的 9-16 位
004D4E9E  |.  8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]
004D4EA1  |.  E8 5E40F3FF     CALL RE.00408F04                            ; //字符串 A 的 9-16 位转大写
004D4EA6  |.  8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
004D4EA9  |.  50              PUSH EAX
004D4EAA  |.  B9 01000000     MOV ECX,1
004D4EAF  |.  BA 03000000     MOV EDX,3
004D4EB4  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]               ; //字符串 A
004D4EB7  |.  E8 3001F3FF     CALL RE.00404FEC                            ; //取字符串 A 的第 3 位
004D4EBC  |.  8D45 DC         LEA EAX,DWORD PTR SS:[EBP-24]
004D4EBF  |.  50              PUSH EAX
004D4EC0  |.  B9 01000000     MOV ECX,1
004D4EC5  |.  BA 01000000     MOV EDX,1
004D4ECA  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]               ; //字符串 A
004D4ECD  |.  E8 1A01F3FF     CALL RE.00404FEC                            ; //取字符串 A 的第 1 位
004D4ED2  |.  8B45 DC         MOV EAX,DWORD PTR SS:[EBP-24]              ; //字符串 A 的第 1 位的地址
004D4ED5  |.  E8 AA48F3FF     CALL RE.00409784                            ; //将字符串 A 的第 1 位送入 EAX
004D4EDA  |.  8BF0            MOV ESI,EAX                                  ; //ESI=EAX
004D4EDC  |.  46              INC ESI                                      ; //ESI=ESI+1
004D4EDD  |.  8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]
004D4EE0  |.  50              PUSH EAX
004D4EE1  |.  8D55 D8         LEA EDX,DWORD PTR SS:[EBP-28]
004D4EE4  |.  B8 AC504D00     MOV EAX,RE.004D50AC                          ; Easy Gang
004D4EE9  |.  E8 EE8FFEFF     CALL RE.004BDEDC                             ; //将"Easy Gang"作 MD5 运算转小写,设为字符串 B
004D4EEE  |.  8B45 D8         MOV EAX,DWORD PTR SS:[EBP-28]               ; //字符串 B
004D4EF1  |.  B9 0A000000     MOV ECX,0A
004D4EF6  |.  8BD6            MOV EDX,ESI
004D4EF8  |.  E8 EF00F3FF     CALL RE.00404FEC                             ; //取字符串 B 的 ESI 位后的 10 位
004D4EFD  |.  8D45 D4         LEA EAX,DWORD PTR SS:[EBP-2C]
004D4F00  |.  50              PUSH EAX
004D4F01  |.  B9 01000000     MOV ECX,1
004D4F06  |.  BA 02000000     MOV EDX,2
004D4F0B  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ; //字符串 A
004D4F0E  |.  E8 D900F3FF     CALL RE.00404FEC                             ; //取字符串 A 的第 2 位
004D4F13  |.  8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]               ; //字符串 A 的第 2 位的地址
004D4F16  |.  E8 6948F3FF     CALL RE.00409784                             ; //将字符串 A 的第 2 位送入 EAX
004D4F1B  |.  8BF0            MOV ESI,EAX                                  ; //ESI=EAX
004D4F1D  |.  46              INC ESI                                      ; //ESI=ESI+1
004D4F1E  |.  8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
004D4F21  |.  50              PUSH EAX
004D4F22  |.  B9 05000000     MOV ECX,5
004D4F27  |.  BA 04000000     MOV EDX,4
004D4F2C  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ; //字符串 A
004D4F2F  |.  E8 B800F3FF     CALL RE.00404FEC                             ; //取字符串 A 的 4-8 位
004D4F34  |.  8D55 D0         LEA EDX,DWORD PTR SS:[EBP-30]
004D4F37  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ; //字符串 A 的 4-8 位
004D4F3A  |.  E8 1542F3FF     CALL RE.00409154
004D4F3F  |.  837D D0 00      CMP DWORD PTR SS:[EBP-30],0
004D4F43  |.  0F84 DD000000   JE RE.004D5026                               ; //字符串 A 的 4-8 位若为空则跳
004D4F49  |.  8D55 CC         LEA EDX,DWORD PTR SS:[EBP-34]
004D4F4C  |.  8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
004D4F4F  |.  E8 0042F3FF     CALL RE.00409154
004D4F54  |.  837D CC 00      CMP DWORD PTR SS:[EBP-34],0
004D4F58  |.  0F84 C8000000   JE RE.004D5026                               ; //字符串 B 的 ESI 位后的 10 位若为空则跳
004D4F5E  |.  8D55 C8         LEA EDX,DWORD PTR SS:[EBP-38]
004D4F61  |.  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004D4F64  |.  E8 EB41F3FF     CALL RE.00409154
004D4F69  |.  837D C8 00      CMP DWORD PTR SS:[EBP-38],0
004D4F6D  |.  0F84 B3000000   JE RE.004D5026                               ; //大写字符串 B 的 9-16 位若为空则跳
004D4F73  |.  8D55 C4         LEA EDX,DWORD PTR SS:[EBP-3C]
004D4F76  |.  8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
004D4F79  |.  E8 D641F3FF     CALL RE.00409154
004D4F7E  |.  837D C4 00      CMP DWORD PTR SS:[EBP-3C],0
004D4F82  |.  0F84 9E000000   JE RE.004D5026                               ; //字符串 A 的第 3 位若为空则跳
004D4F88  |.  8D45 C0         LEA EAX,DWORD PTR SS:[EBP-40]
004D4F8B  |.  50              PUSH EAX
004D4F8C  |.  FF75 F8         PUSH DWORD PTR SS:[EBP-8]                    ; //大写字符串 A 的 9-16 位
004D4F8F  |.  FF75 F0         PUSH DWORD PTR SS:[EBP-10]                   ; //字符串 B 的 ESI 位后的 10 位
004D4F92  |.  FF75 F4         PUSH DWORD PTR SS:[EBP-C]                    ; //字符串 A 的第 3 位
004D4F95  |.  8D45 B8         LEA EAX,DWORD PTR SS:[EBP-48]
004D4F98  |.  BA 03000000     MOV EDX,3
004D4F9D  |.  E8 AAFEF2FF     CALL RE.00404E4C                             ; //将上面三个字符串相连,设为字符串 C
004D4FA2  |.  8B45 B8         MOV EAX,DWORD PTR SS:[EBP-48]               ; //字符串 C
004D4FA5  |.  8D55 BC         LEA EDX,DWORD PTR SS:[EBP-44]
004D4FA8  |.  E8 2F8FFEFF     CALL RE.004BDEDC                             ; //将字符串 C 作 MD5 运算转小写,设为字符串 D
004D4FAD  |.  8B45 BC         MOV EAX,DWORD PTR SS:[EBP-44]               ; //字符串 D
004D4FB0  |.  B9 05000000     MOV ECX,5
004D4FB5  |.  8BD6            MOV EDX,ESI
004D4FB7  |.  E8 3000F3FF     CALL RE.00404FEC                             ; //取字符串 D 的 ESI 位后的 5 位,设为字符串 D
004D4FBC  |.  8B55 C0         MOV EDX,DWORD PTR SS:[EBP-40]               ; //字符串 D
004D4FBF  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]                ; //字符串 A 的 4-8 位
004D4FC2  |.  E8 11FFF2FF     CALL RE.00404ED8                             ; //比较 CALL
004D4FC7  |.  75 35           JNZ SHORT RE.004D4FFE                        ; //关键跳转
004D4FC9  |.  8D55 B0         LEA EDX,DWORD PTR SS:[EBP-50]
004D4FCC  |.  8B83 10030000   MOV EAX,DWORD PTR DS:[EBX+310]
004D4FD2  |.  E8 05C7F9FF     CALL RE.004716DC
004D4FD7  |.  8B45 B0         MOV EAX,DWORD PTR SS:[EBP-50]
004D4FDA  |.  8D55 B4         LEA EDX,DWORD PTR SS:[EBP-4C]
004D4FDD  |.  E8 7241F3FF     CALL RE.00409154
004D4FE2  |.  8B4D B4         MOV ECX,DWORD PTR SS:[EBP-4C]
004D4FE5  |.  33D2            XOR EDX,EDX
004D4FE7  |.  8BC3            MOV EAX,EBX
004D4FE9  |.  E8 06010000     CALL RE.004D50F4
004D4FEE  |.  8BC3            MOV EAX,EBX
004D4FF0  |.  E8 E7010000     CALL RE.004D51DC
004D4FF5  |.  8BC3            MOV EAX,EBX
004D4FF7  |.  E8 B09AFBFF     CALL RE.0048EAAC
004D4FFC  |.  EB 4E           JMP SHORT RE.004D504C
004D4FFE  |>  6A 30           PUSH 30
004D5000  |.  B9 B8504D00     MOV ECX,RE.004D50B8                          ; Registry Easy
004D5005  |.  BA C8504D00     MOV EDX,RE.004D50C8                          ; Invalid serial number! Please try again...
004D500A  |.  A1 1C955800     MOV EAX,DWORD PTR DS:[58951C]
004D500F  |.  8B00            MOV EAX,DWORD PTR DS:[EAX]
004D5011  |.  E8 F6D3FBFF     CALL RE.0049240C
004D5016  |.  8B83 10030000   MOV EAX,DWORD PTR DS:[EBX+310]
004D501C  |.  8B10            MOV EDX,DWORD PTR DS:[EAX]
004D501E  |.  FF92 C4000000   CALL DWORD PTR DS:[EDX+C4]
004D5024  |.  EB 26           JMP SHORT RE.004D504C
004D5026  |>  6A 30           PUSH 30
004D5028  |.  B9 B8504D00     MOV ECX,RE.004D50B8                          ; Registry Easy
004D502D  |.  BA C8504D00     MOV EDX,RE.004D50C8                          ; Invalid serial number! Please try again...
004D5032  |.  A1 1C955800     MOV EAX,DWORD PTR DS:[58951C]
004D5037  |.  8B00            MOV EAX,DWORD PTR DS:[EAX]
004D5039  |.  E8 CED3FBFF     CALL RE.0049240C
004D503E  |.  8B83 10030000   MOV EAX,DWORD PTR DS:[EBX+310]
004D5044  |.  8B10            MOV EDX,DWORD PTR DS:[EAX]
004D5046  |.  FF92 C4000000   CALL DWORD PTR DS:[EDX+C4]
004D504C  |>  33C0            XOR EAX,EAX
004D504E  |.  5A              POP EDX
004D504F  |.  59              POP ECX
004D5050  |.  59              POP ECX
004D5051  |.  64:8910         MOV DWORD PTR FS:[EAX],EDX
004D5054  |.  68 8B504D00     PUSH RE.004D508B
004D5059  |>  8D45 B0         LEA EAX,DWORD PTR SS:[EBP-50]
004D505C  |.  E8 6BFAF2FF     CALL RE.00404ACC
004D5061  |.  8D45 B4         LEA EAX,DWORD PTR SS:[EBP-4C]
004D5064  |.  BA 0D000000     MOV EDX,0D
004D5069  |.  E8 82FAF2FF     CALL RE.00404AF0
004D506E  |.  8D45 E8         LEA EAX,DWORD PTR SS:[EBP-18]
004D5071  |.  E8 56FAF2FF     CALL RE.00404ACC
004D5076  |.  8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]
004D5079  |.  BA 05000000     MOV EDX,5
004D507E  |.  E8 6DFAF2FF     CALL RE.00404AF0
004D5083  \\.  C3              RETN
004D5084   .^ E9 A3F3F2FF     JMP RE.0040442C
004D5089   .^ EB CE           JMP SHORT RE.004D5059
004D508B    .  5E              POP ESI
004D508C    .  5B              POP EBX
004D508D    .  8BE5            MOV ESP,EBP
004D508F    .  5D              POP EBP
004D5090    .  C3              RETN
```



# 破解总结

运用了 `MD5` 加密运算

# 算法总结

举例说明：

1. 根据软件的要求输入的试炼码为 `"1234-5678-9abc-def0"`
2. 去掉试炼码中间的 `"-"` 后为 `"123456789abcdef0"`
3. 取 `"123456789abcdef0"` 的 `9-16` 位为 `"9abcdef0"`，转大写为 `"9ABCDEF0"`
4. `"Easy Gang"` 作 `MD5` 运算转小写为 `"ec7235e71546232da9f7cc02f32c9d0e"`
5. 根据试炼码的第 `1` 位上的字符 `"1"`，`1+1=2`，取 `"ec7235e71546232da9f7cc02f32c9d0e"` 的第 `2` 位后的 `10` 位，即 `"c7235e7154"`
6. 将字符串 `"9ABCDEF0"`、`"c7235e7154"` 和试炼码的第 `3` 位上字符 `"3"` 相连得 `"9ABCDEF0c7235e71543"`
7. `"9ABCDEF0c7235e71543"` 作 `MD5` 运算转小写为 `"4fc224905dcd47d77c3727b723602dc2"`
8. 根据试炼码的第 `2` 位上的字符 `"2"`，`2+1=3`，取 `"4fc224905dcd47d77c3727b723602dc2"` 的第 `3` 位后的 `5` 位，即 `"c2249"`
9. 最后将试炼码去掉 `"-"` 后的 `4-8` 位，即 `"45678"` 与运算所得的 `"c2249"` 比较，相同则注册成功

经分析可知 `"123c-2249-9abc-def0"` 为一组正确的注册码

# 算法注册机

易语言代码

```
.版本 2
.支持库 dp1

.子程序 \_ 按钮 1\_ 被单击
.局部变量 A, 文本型, , , 第 9-16 位
.局部变量 B, 文本型, , , 第 1 位
.局部变量 C, 文本型, , , 第 2 位
.局部变量 D, 文本型, , , 第 3 位
.局部变量 E, 文本型
.局部变量 F, 文本型

A ＝ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9)) ＋ 到文本 (取随机数 (0, 9))
B ＝ 到文本 (取随机数 (0, 9))
C ＝ 到文本 (取随机数 (0, 9))
D ＝ 到文本 (取随机数 (0, 9))
E ＝ 取文本中间 (“ec7235e71546232da9f7cc02f32c9d0e”, 到整数 (B) ＋ 1, 10)
F ＝ 取文本中间 (取数据摘要 (到字节集 (A ＋ E ＋ D)), 到整数 (C) ＋ 1, 5)
编辑框 1.内容 ＝ 取文本中间 (B ＋ C ＋ D ＋ F ＋ A, 1, 4) ＋ “-” ＋ 取文本中间(B ＋ C ＋ D ＋ F ＋ A, 5, 4) ＋ “-” ＋ 取文本中间 (B ＋ C ＋ D ＋ F ＋ A, 9, 4) ＋ “-” ＋ 取文本中间 (B ＋ C ＋ D ＋ F ＋ A, 13, 4)
```

# 注册信息

送几组靓号：

`6660-7f99-6666-6666`

`777a-612f-7777-7777`

`888b-f603-8888-8888`

保存在

`[HKEY_CURRENT_USER\\Software\\RegistryEasy\]`

感谢飘云老大、猫老大、`Nisy` 老大以及很多前辈们的学习教程以及徐超等所有帮助过我的论坛兄弟姐妹们！谢谢

元旦快到了，祝朋友们在新年里：事业如日中天，心情阳光灿烂，工资地覆天翻，未来风光无限，爱情浪漫依然，快乐游戏人间。

# 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！
