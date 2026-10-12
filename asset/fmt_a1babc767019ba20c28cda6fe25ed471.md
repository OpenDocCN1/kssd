
# 变形的 MD5

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-7.htm`](https://www.pediy.com/kssd/pediy08/pediy8-7.htm)

【破文作者】 rdsnow\[BCG\]\[PYG\]\[D.4s\]
【作者主页】 [`rdsnow.ys168.com`](http://rdsnow.ys168.com)
【 E-mail 】 rdsnow@163.com
【 作者 QQ 】 83757177
【文章题目】 变形的 MD5
【软件名称】 AutoRunPro Enterprise
【软件版本】 V4.0.0.32
【下载地址】 [`www.longtion.com`](http://www.longtion.com)

---

【加密方式】 序列号
【破解工具】 ODbyDYK v1.10\[05.09\]
【软件限制】 功能限制
【破解平台】 Microsoft Windows XP Professional
【平台版本】 5.1.2600 Service Pack 2 内部版本号 2600

---

【软件简介】

AutoRun Pro Enterprise can create, edit professional autorun interface and generate autorun files for CD/DVDs in a WYSIWYG environment.

Display a professional and beautiful interface for users to open or execute files, print documents, send e-mail, visit Web sites, browse CDs,

play sound, music and so on, when your CD is inserted. It's fast and easy to use. Anyone can quickly create autorun CD-ROMs within minutes.

【文章简介】

我编程只有 5、6 年前学习的一点 `turbo c` 的基础，为了学习编程，一般的程序都是尽可能的学出 `keygen` ，这下遇到了个采用变形 `MD5` 算法的软件，也来试了下。

给变形的 `MD5` 的程序要写出 `keygen` ，必须找到变形的地方，然后在标准 `MD5` 源码中作出相应的修改。

---

【破解过程】

程序注册窗口要输入三个编辑框：`username`，`serial` 和 `key`，从后面代码看出 `serial` 的前四位必须是 `"0018"`

所以输入：

```
username＝ rdsnow[BCG][PYG][D.4s]
serial  ＝ 001812345678cdef
key     ＝ 9898989832323232
```

因为有注册码错误的对话框，所以很容易找到关键的地方。

```
00562BD2 .  8B83 14030000 MOV EAX,DWORD PTR [EBX+314]
00562BD8 .  E8 3F78F1FF   CALL AutoRunP.0047A41C         ;  取得 username
00562BDD .  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
00562BE0 .  8D55 FC       LEA EDX,DWORD PTR [EBP-4]
00562BE3 .  E8 8460EAFF   CALL AutoRunP.00408C6C
00562BE8 .  8D55 E0       LEA EDX,DWORD PTR [EBP-20]
00562BEB .  8B83 F8020000 MOV EAX,DWORD PTR [EBX+2F8]
00562BF1 .  E8 2678F1FF   CALL AutoRunP.0047A41C         ;  取得 serial
00562BF6 .  8B45 E0       MOV EAX,DWORD PTR [EBP-20]
00562BF9 .  8D55 F8       LEA EDX,DWORD PTR [EBP-8]
00562BFC .  E8 6B60EAFF   CALL AutoRunP.00408C6C
00562C01 .  8D55 DC       LEA EDX,DWORD PTR [EBP-24]
00562C04 .  8B83 FC020000 MOV EAX,DWORD PTR [EBX+2FC]
00562C0A .  E8 0D78F1FF   CALL AutoRunP.0047A41C         ;  取得 key
00562C0F .  8B45 DC       MOV EAX,DWORD PTR [EBP-24]
00562C12 .  8D55 F4       LEA EDX,DWORD PTR [EBP-C]
00562C15 .  E8 5260EAFF   CALL AutoRunP.00408C6C
00562C1A .  8B45 F4       MOV EAX,DWORD PTR [EBP-C]
00562C1D .  50            PUSH EAX                       ;  压入 key
00562C1E .  A1 709E5600   MOV EAX,DWORD PTR [569E70]
00562C23 .  8B00          MOV EAX,DWORD PTR [EAX]
00562C25 .  8B4D F8       MOV ECX,DWORD PTR [EBP-8]      ;  serial
00562C28 .  8B55 FC       MOV EDX,DWORD PTR [EBP-4]      ;  username
00562C2B .  E8 5466FFFF   CALL AutoRunP.00559284         ;  关键 CALL
00562C30 .  84C0          TEST AL,AL
00562C32 .  75 1D         JNZ SHORT AutoRunP.00562C51    ;  关键跳转
00562C34 .  6A 10         PUSH 10
00562C36 .  B9 602D5600   MOV ECX,AutoRunP.00562D60      ;  ASCII "Register"
00562C3B .  BA 6C2D5600   MOV EDX,AutoRunP.00562D6C      ;  ASCII "Incomplete or incorrect information."
00562C40 .  A1 5CA15600   MOV EAX,DWORD PTR [56A15C]
00562C45 .  8B00          MOV EAX,DWORD PTR [EAX]
00562C47 .  E8 788DF3FF   CALL AutoRunP.0049B9C4         ;  错误的对话框
00562C4C .  E9 C8000000   JMP AutoRunP.00562D19
00562C51 >  33C0          XOR EAX,EAX
```

如果是爆破，不建议在 `00562C32` 关键跳转处改跳转，应该跟进关键 `CALL` ， 修改注册标志

---

下面就跟进吧：

```
005592F5 .  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
005592F8 .  E8 F7B3EAFF   CALL AutoRunP.004046F4         ;  取 Serial 的长度
005592FD .  8BD8          MOV EBX,EAX
005592FF .  83FB 01       CMP EBX,1
00559302 .  7C 1E         JL SHORT AutoRunP.00559322      ;  Serial 为空就跳走
00559304 >  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
00559307 .  807C18 FF 20  CMP BYTE PTR [EAX+EBX-1],20
0055930C .  75 0F         JNZ SHORT AutoRunP.0055931D
0055930E .  8D45 E4       LEA EAX,DWORD PTR [EBP-1C]
00559311 .  B9 01000000   MOV ECX,1
00559316 .  8BD3          MOV EDX,EBX
00559318 .  E8 77B6EAFF   CALL AutoRunP.00404994         ;  去掉 Serial 中的空格
0055931D >  4B            DEC EBX
0055931E .  85DB          TEST EBX,EBX
00559320 .^ 75 E2         JNZ SHORT AutoRunP.00559304
00559322 >  8B45 E0       MOV EAX,DWORD PTR [EBP-20]
00559325 .  E8 CAB3EAFF   CALL AutoRunP.004046F4         ;  取 key 的长度
0055932A .  8BD8          MOV EBX,EAX
0055932C .  83FB 01       CMP EBX,1
0055932F .  7C 1E         JL SHORT AutoRunP.0055934F      ;  key 为空就跳走
00559331 >  8B45 E0       MOV EAX,DWORD PTR [EBP-20]
00559334 .  807C18 FF 20  CMP BYTE PTR [EAX+EBX-1],20
00559339 .  75 0F         JNZ SHORT AutoRunP.0055934A
0055933B .  8D45 E0       LEA EAX,DWORD PTR [EBP-20]
0055933E .  B9 01000000   MOV ECX,1
00559343 .  8BD3          MOV EDX,EBX
00559345 .  E8 4AB6EAFF   CALL AutoRunP.00404994         ;  去掉 key 中的空格
0055934A >  4B            DEC EBX
0055934B .  85DB          TEST EBX,EBX
0055934D .^ 75 E2         JNZ SHORT AutoRunP.00559331
0055934F >  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
00559352 .  E8 9DB3EAFF   CALL AutoRunP.004046F4         ;  取 Serial 的长度
00559357 .  83F8 10       CMP EAX,10
0055935A .  0F85 98010000 JNZ AutoRunP.005594F8          ;  Serial 的长度不等于 16 就跳走
00559360 .  8B45 E0       MOV EAX,DWORD PTR [EBP-20]
00559363 .  E8 8CB3EAFF   CALL AutoRunP.004046F4         ;  取 key 的长度
00559368 .  83F8 10       CMP EAX,10
0055936B .  0F85 87010000 JNZ AutoRunP.005594F8          ;  key 的长度不等于 16 就跳走
00559371 .  33C0          XOR EAX,EAX
00559373 .  55            PUSH EBP
00559374 .  68 4B945500   PUSH AutoRunP.0055944B
00559379 .  64:FF30       PUSH DWORD PTR FS:[EAX]
0055937C .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0055937F .  8D45 D4       LEA EAX,DWORD PTR [EBP-2C]
00559382 .  50            PUSH EAX
00559383 .  B9 08000000   MOV ECX,8
00559388 .  BA 01000000   MOV EDX,1
0055938D .  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
00559390 .  E8 BFB5EAFF   CALL AutoRunP.00404954         ;  取 Serial 的前 8 个字符
00559395 .  8B4D D4       MOV ECX,DWORD PTR [EBP-2C]
00559398 .  8D45 D8       LEA EAX,DWORD PTR [EBP-28]
0055939B .  BA 3C955500   MOV EDX,AutoRunP.0055953C
005593A0 .  E8 9BB3EAFF   CALL AutoRunP.00404740         ;  准备转换
005593A5 .  8B45 D8       MOV EAX,DWORD PTR [EBP-28]
005593A8 .  E8 63FCEAFF   CALL AutoRunP.00409010         ;  将 8 个字符组成的 Hex 文本转为数值 a
005593AD .  8945 F8       MOV DWORD PTR [EBP-8],EAX      ;  保存 a 进 inbuff
005593B0 .  8D45 CC       LEA EAX,DWORD PTR [EBP-34]
005593B3 .  50            PUSH EAX
005593B4 .  B9 08000000   MOV ECX,8
005593B9 .  BA 09000000   MOV EDX,9
005593BE .  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
005593C1 .  E8 8EB5EAFF   CALL AutoRunP.00404954         ;  取 Serial 的后 8 个字符
005593C6 .  8B4D CC       MOV ECX,DWORD PTR [EBP-34]
005593C9 .  8D45 D0       LEA EAX,DWORD PTR [EBP-30]
005593CC .  BA 3C955500   MOV EDX,AutoRunP.0055953C
005593D1 .  E8 6AB3EAFF   CALL AutoRunP.00404740         ;  准备转换
005593D6 .  8B45 D0       MOV EAX,DWORD PTR [EBP-30]
005593D9 .  E8 32FCEAFF   CALL AutoRunP.00409010         ;  将 8 个字符组成的 Hex 文本转为数值 b
005593DE .  8945 F4       MOV DWORD PTR [EBP-C],EAX      ;  保存 b 进入 inbuff
005593E1 .  8D45 C4       LEA EAX,DWORD PTR [EBP-3C]
005593E4 .  50            PUSH EAX
005593E5 .  B9 08000000   MOV ECX,8
005593EA .  BA 01000000   MOV EDX,1
```
