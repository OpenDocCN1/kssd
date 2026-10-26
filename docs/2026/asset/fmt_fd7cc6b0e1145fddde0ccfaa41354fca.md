
```
00421301  |>  C645 84 5A    MOV BYTE PTR SS:[EBP-7C], 5A       ;  连接上5A即字母Z

00421305  |.  C645 85 59    MOV BYTE PTR SS:[EBP-7B], 59       ;  连接上59即字母Y，即“mxaxaxaxaxaxaxaxZY”

00421309  |.  C745 CC 18000>MOV DWORD PTR SS:[EBP-34], 18

00421310  |.  66:C745 EC 08>MOV WORD PTR SS:[EBP-14], 8

00421316  |.  837D CC 28    CMP DWORD PTR SS:[EBP-34], 28

0042131A  |.  7D 4A         JGE SHORT 123.00421366

0042131C  |>  8B55 CC       /MOV EDX, DWORD PTR SS:[EBP-34]    ;  EDX=18

0042131F  |.  0FBE8415 5CFF>|MOVSX EAX, BYTE PTR SS:[EBP+EDX-A>;  上次循环得到的字符串第一位给EAX

00421327  |.  C1E0 04       |SHL EAX, 4                        ;  EAX=EAX左移4位

0042132A  |.  8B55 CC       |MOV EDX, DWORD PTR SS:[EBP-34]

0042132D  |.  0FBE8C15 5DFF>|MOVSX ECX, BYTE PTR SS:[EBP+EDX-A>;  上次循环的字符串的第二位给ECX

00421335  |.  D1F9          |SAR ECX, 1                        ;  ECX=ECX右移1位

00421337  |.  33C1          |XOR EAX, ECX                      ;  EAX=EAX XOR ECX

00421339  |.  50            |PUSH EAX                          ; /Arg1

0042133A  |.  E8 59000000   |CALL 123.00421398                 ; \123.00421398

0042133F  |.  59            |POP ECX

00421340  |.  B9 1A000000   |MOV ECX, 1A                       ;  ECX=1A

00421345  |.  99            |CDQ

00421346  |.  F7F9          |IDIV ECX                          ;  EAX=EAX\1A EDX=MOD(EAX,1A)

00421348  |.  83C2 41       |ADD EDX, 41                       ;  EDX=EDX+41

0042134B  |.  8B45 CC       |MOV EAX, DWORD PTR SS:[EBP-34]

0042134E  |.  0FBE4405 A0   |MOVSX EAX, BYTE PTR SS:[EBP+EAX-6>;  替换后的字符串的第25位给EAX

00421353  |.  3BD0          |CMP EDX, EAX                      ;

这个循环分别比较注册码第25位到40位是不是25E,26S,27U,28S,29U,30S,31R,32U,33U,34C,35N,36B,37K,38I,39M,40V

00421355  |.  74 06         |JE SHORT 123.0042135D

00421357  |.  C645 DB 00    |MOV BYTE PTR SS:[EBP-25], 0

0042135B  |.  EB 09         |JMP SHORT 123.00421366

0042135D  |>  FF45 CC       |INC DWORD PTR SS:[EBP-34]

00421360  |.  837D CC 28    |CMP DWORD PTR SS:[EBP-34], 28

00421364  |.^ 7C B6         \JL SHORT 123.0042131C

00421366  |>  0FBE55 AA     MOVSX EDX, BYTE PTR SS:[EBP-56]    ;  替换后字符串第11位给EAX

0042136A  |.  83FA 59       CMP EDX, 59                        ;  第11位是否为59即大写字母Y

0042136D  |.  74 04         JE SHORT 123.00421373

0042136F  |.  C645 DB 00    MOV BYTE PTR SS:[EBP-25], 0

00421373  |>  8A45 DB       MOV AL, BYTE PTR SS:[EBP-25]

00421376  |.  50            PUSH EAX

00421377  |.  FF4D F8       DEC DWORD PTR SS:[EBP-8]

0042137A  |.  8D45 08       LEA EAX, DWORD PTR SS:[EBP+8]      ;  注册码地址给EAX

0042137D  |.  BA 02000000   MOV EDX, 2                         ;  EDX=2

00421382  |.  E8 ED240D00   CALL 123.004F3874

00421387  |.  58            POP EAX

00421388  |.  8B55 DC       MOV EDX, DWORD PTR SS:[EBP-24]

0042138B  |.  64:8915 00000>MOV DWORD PTR FS:[0], EDX

00421392  |>  5F            POP EDI

00421393  |.  5E            POP ESI

00421394  |.  8BE5          MOV ESP, EBP

00421396  |.  5D            POP EBP

00421397  \.  C3            RETN
```

=========================================================================================

## 算法小结：

1、用户名位数大于3位，未参与注册码计算

2、第2、11、40-44位分别为0、Y、P、2、I、1，24位是0-9的数字

3、先将注册码第2位替换为“#”，然后和字符串“1z1h+2a0n-0g8y\*9a1n|”循环运算，算出第12-19位的注册码

4、替换后的注册码经过16次循环，得到字符串“mxaxaxaxaxaxaxax”，再连接上“ZY”，即“mxaxaxaxaxaxaxaxZY”

5、字符串“mxaxaxaxaxaxaxaxZY”经过16次循环得到25-40位的注册码

6、其他位的注册码参与运算但未进行比较，所以仍是你原来输入的相应数字

算法比较简单，就是循环次数太多，要有耐性

由于注册机在制作过程中出现一些技术问题，（第2组注册码能成功算出，但第一个有些问题）正在调试，成功后再发

发送两组可用注册码：

7074747474YENPHNVRV47474ESUSUSRUUCNBKIMVP2I1

1011111111YGKSSMSKO11111SOOOOOSNNBUTBUGPP2I1

注册信息保存在注册表：`HKEY_CURRENT_USER\Software\XTZY\Pic2Ico`,删除即可重玩

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
