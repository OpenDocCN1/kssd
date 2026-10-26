
# [原创]简简单单找到Open Video Converter v3.01的注册码

## 文章信息
【破文标题】简简单单找到Open Video Converter v3.01的注册码

【破文作者】marsaber

【作者邮箱】marsaber@163.com

【作者主页】[337K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2M7Y4y4S2L8X3N6W2L8q4)9J5k6i4W2K6x3e0j5^5i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@a9aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2S2M7Y4y4S2L8X3N6W2L8q4)9J5k6i4W2K6x3e0j5^5i4K6u0W2j5$3!0E0)

【破解工具】`Peid` 0.93、`W32dasm`、`OllyDbg`

【破解平台】Windows 2003 SP1

【软件名称】Open Video Converter v3.01

【软件大小】541 KB

【原版下载】[119K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4J5M7$3E0&6i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0x3^5x3e0S2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@905K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4J5M7$3E0&6i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0x3^5x3e0S2Q4x3X3g2Z5N6r3#2D9)

【保护方式】不知道

【软件简介】一款易于使用的视频转换,分割和编辑工具.它能转换多个视频格式如MPG, AVI, ASF, WMV到AVI 文件. 它能改变帧尺寸,帧频,视频和音频压缩编码.主要功能有: -转换MPEG,WMV,ASF,MPG,VCD,OGM,DAT,SVCD为AVI. 支持DIVX,XVID编码.

-分割视频文件

-改变帧尺寸,调整视频屏幕高宽比.

-通过选择不同编码率改变视频文件大小

-改变AVI文件的视频和音频压缩编码。

------------------------------------------------------------------------

## 【破解过程】
1.`Peid` 0.93查壳，无壳，Microsoft Visual C++ 7.0编写！

2.运行，出现输入用户名和注册码的提示框，用户名：`marsaber`，注册码：`111111111`，错误提示：Registration failed!

3.`W32dasm3.0`无极版载入，查找字符串参考，没有发现错误提示信息：Registration failed!

4.`OllyDbg`载入，来到：

```
00438A7C >  6A 60           push 60
00438A7E    68 C8524600     push VideoCon.004652C8
00438A83    E8 A4030000     call VideoCon.00438E2C
```

右键-->Ultra字符串参考-->查找ASCII-->Ctrl+F-->输入"`Registration failed`"-->来到"`00424C55  push VideoCon.004603C0` Registration failed!"，双击来到：`00424C55`处！

5.`W32dasm3.0`无极版重新载入，Shift+F12，输入`00424C55`，回车！得到信息并分析：

```
:00424B5D 3BD0                    cmp edx, eax
:00424B5F 753C                    jne 00424B9D
:00424B61 0FB64DF8                movzx ecx, byte ptr [ebp-08]
:00424B65 0FB655FD                movzx edx, byte ptr [ebp-03]
:00424B69 83EA30                  sub edx, 00000030
:00424B6C 3BCA                    cmp ecx, edx
:00424B6E 752D                    jne 00424B9D               //如果ecx!=edx，跳到00424B9D！
:00424B70 0FB645FF                movzx eax, byte ptr [ebp-01]
:00424B74 0FB64DF6                movzx ecx, byte ptr [ebp-0A]
:00424B78 83E930                  sub ecx, 00000030
:00424B7B 3BC1                    cmp eax, ecx
:00424B7D 751E                    jne 00424B9D               //如果eax!=ecx，跳到00424B9D！
:00424B7F 0FB655FA                movzx edx, byte ptr [ebp-06]
:00424B83 0FB645F5                movzx eax, byte ptr [ebp-0B]
:00424B87 83E830                  sub eax, 00000030
:00424B8A 3BD0                    cmp edx, eax
:00424B8C 750F                    jne 00424B9D               //如果edx!=eax，跳到00424B9D！
:00424B8E 0FB64DF4                movzx ecx, byte ptr [ebp-0C]
:00424B92 0FB655F9                movzx edx, byte ptr [ebp-07]
:00424B96 83EA30                  sub edx, 00000030
:00424B99 3BCA                    cmp ecx, edx
:00424B9B 7458                    je 00424BF5               //如果ecx==edx，注册成功！
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00424B5F(C), :00424B6E(C), :00424B7D(C), :00424B8C(C)
|
:00424B9D 0FB645FC                movzx eax, byte ptr [ebp-04]
:00424BA1 83F835                  cmp eax, 00000035
:00424BA4 0F85A7000000            jne 00424C51               //不等于5就跳到00424C51
:00424BAA 0FB64DFD                movzx ecx, byte ptr [ebp-03]
:00424BAE 83F931                  cmp ecx, 00000031
:00424BB1 0F859A000000            jne 00424C51               //不等于1就跳到00424C51
:00424BB7 0FB655F6                movzx edx, byte ptr [ebp-0A]
:00424BBB 83FA38                  cmp edx, 00000038
:00424BBE 0F858D000000            jne 00424C51               //不等于8就跳到00424C51
:00424BC4 0FB645F5                movzx eax, byte ptr [ebp-0B]
:00424BC8 83F839                  cmp eax, 00000039
:00424BCB 0F8580000000            jne 00424C51               //不等于9就跳到00424C51
:00424BD1 0FB64DF9                movzx ecx, byte ptr [ebp-07]
:00424BD5 83F937                  cmp ecx, 00000037
:00424BD8 7577                    jne 00424C51               //不等于7就跳到00424C51
:00424BDA 0FB655F7                movzx edx, byte ptr [ebp-09]
:00424BDE 83FA36                  cmp edx, 00000036
:00424BE1 756E                    jne 00424C51               //不等于6就跳到00424C51
:00424BE3 0FB645FE                movzx eax, byte ptr [ebp-02]
:00424BE7 83F832                  cmp eax, 00000032
:00424BEA 7565                    jne 00424C51               //不等于2就跳到00424C51
:00424BEC 0FB64DFB                movzx ecx, byte ptr [ebp-05]
:00424BF0 83F939                  cmp ecx, 00000039
:00424BF3 755C                    jne 00424C51               //不等于9就跳到00424C51
                                                             //得到-->51897629，事实证明这个就是我这台机子的注册码！
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00424B9B(C)
|
:00424BF5 6A00                    push 00000000
:00424BF7 6A00                    push 00000000
:00424BF9 6874034600              push 00460374               //注册成功！
:00424BFE E86CBF0200              call 00450B6F
:00424C03 8B4DE0                  mov ecx, dword ptr [ebp-20]
:00424C06 83C170                  add ecx, 00000070
:00424C09 E862F4FFFF              call 00424070
:00424C0E 50                      push eax
:00424C0F 6890034600              push 00460390
:00424C14 689C034600              push 0046039C
:00424C19 E8A295FEFF              call 0040E1C0
:00424C1E 8BC8                    mov ecx, eax
:00424C20 E8AABC0200              call 004508CF
:00424C25 8B4DE0                  mov ecx, dword ptr [ebp-20]
:00424C28 83C174                  add ecx, 00000074
:00424C2B E840F4FFFF              call 00424070
:00424C30 50                      push eax
:00424C31 68A4034600              push 004603A4
:00424C36 68B8034600              push 004603B8
:00424C3B E88095FEFF              call 0040E1C0
:00424C40 8BC8                    mov ecx, eax
:00424C42 E888BC0200              call 004508CF
:00424C47 8B4DE0                  mov ecx, dword ptr [ebp-20]
:00424C4A E887620200              call 0044AED6
:00424C4F EB0E                    jmp 00424C5F
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00424BA4(C), :00424BB1(C), :00424BBE(C), :00424BCB(C), :00424BD8(C)
|:00424BE1(C), :00424BEA(C), :00424BF3(C)
|
:00424C51 6A00                    push 00000000
:00424C53 6A00                    push 00000000
:00424C55 68C0034600              push 004603C0               //注册失败！
:00424C5A E810BF0200              call 00450B6F
```

6.运行`VideoConverter.exe`，输入用户名`marsaber`，注册码`51897629`，OK，注册成功！关闭再运行，无注册提示框出现！成功！

7.`HKEY_CURRENT_USER\Software\DigitByteStudio\VideoConverter\Option`下删除键值`username`和`registration_code`，重新运行又需要输入用户名和注册码，输入`110`和注册码：`51897629`，成功

8.再删除键值，输入霏凡提供的用户名：`crsky`和注册码：`98756171`，成功！汗，不知道是怎么回事，反正是稀里糊涂的成功了！

------------------------------------------------------------------------

## 【破解总结】
毫无技术性可言！

纯属娱乐！

------------------------------------------------------------------------

## 【版权声明】
本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
