
```
; 该代码是 aarch64-linux-android-gcc + IDA PRO 生成的反汇编代码

.text:00000000004006A0                 MOV             W0, #8

.text:00000000004006A4                 STR             W0, [SP] ; int

.text:00000000004006A8                 MOV             W0, #9

.text:00000000004006AC                 STR             W0, [SP,#8] ; int

.text:00000000004006B0                 MOV             W0, #10

.text:00000000004006B4                 STR             W0, [SP,#16] ; int

.text:00000000004006B8                 MOV             X0, X1  ; this

.text:00000000004006BC                 MOV             W1, #1  ; int

.text:00000000004006C0                 MOV             W2, #2  ; int

.text:00000000004006C4                 MOV             W3, #3  ; int

.text:00000000004006C8                 MOV             W4, #4  ; int

.text:00000000004006CC                 MOV             W5, #5  ; int

.text:00000000004006D0                 MOV             W6, #6  ; int

.text:00000000004006D4                 MOV             W7, #7  ; int

.text:00000000004006D8                 BL              _ZN4Calc12thiscall_addEiiiiiiiiii ; Calc::thiscall_add(int,int,int,int,int,int,int,int,int,int)

.text:00000000004006DC                 STR             W0, [X29,#0x1C]



.text:0000000000400628                 EXPORT _ZN4Calc12thiscall_addEiiiiiiiiii

.text:0000000000400628

.text:0000000000400628                 SUB             SP, SP, #0x40

.text:000000000040062C                 STR             X0, [SP,#40]

.text:0000000000400630                 STR             W1, [SP,#36]

.text:0000000000400634                 STR             W2, [SP,#32]

.text:0000000000400638                 STR             W3, [SP,#28]

.text:000000000040063C                 STR             W4, [SP,#24]

.text:0000000000400640                 STR             W5, [SP,#20]

.text:0000000000400644                 STR             W6, [SP,#16]

.text:0000000000400648                 STR             W7, [SP,#12]

.text:000000000040064C                 LDR             W1, [SP,#36]

.text:0000000000400650                 LDR             W0, [SP,#32]

.text:0000000000400654                 ADD             W1, W1, W0

.text:0000000000400658                 LDR             W0, [SP,#28]

.text:000000000040065C                 ADD             W1, W1, W0

.text:0000000000400660                 LDR             W0, [SP,#24]

.text:0000000000400664                 ADD             W1, W1, W0

.text:0000000000400668                 LDR             W0, [SP,#20]

.text:000000000040066C                 ADD             W1, W1, W0

.text:0000000000400670                 LDR             W0, [SP,#16]

.text:0000000000400674                 ADD             W1, W1, W0

.text:0000000000400678                 LDR             W0, [SP,#12]

.text:000000000040067C                 ADD             W0, W1, W0

.text:0000000000400680                 STR             W0, [SP,#60]

.text:0000000000400684                 LDR             W0, [SP,#60]        # 返回值

.text:0000000000400688                 ADD             SP, SP, #0x40

.text:000000000040068C                 RET
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-10-26 12:19

被有影编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
