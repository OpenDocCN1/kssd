
```
.text:0046657F mov [ebp+var_4], 7Bh
.text:00466586 lea eax, [ebp+var_50]
.text:00466589 push eax
.text:0046658A mov cx, [ebp+var_FC]
.text:00466591 add cx, 1
.text:00466595 jo loc_466832
.text:0046659B movsx edx, cx
.text:0046659E push edx
.text:0046659F push 1
.text:004665A1 push offset asc_40D030 ; "-"
.text:004665A6 push 0
.text:004665A8 call ds:__vbaMidStmtBstr ; mid(var_50,var_fc+1,1) = "-"
.text:004665AE
.text:004665AE loc_4665AE: ; CODE XREF: sub_464EF0+1664j
.text:004665AE ; sub_464EF0+166Ej
.text:004665AE mov [ebp+var_4], 7Dh
.text:004665B5 mov ax, [ebp+var_FC]
.text:004665BC add ax, 2
.text:004665C0 jo loc_466832
.text:004665C6 mov [ebp+var_FC], ax ; var_fc 加 2
.text:004665CD mov [ebp+var_4], 7Eh
.text:004665D4 mov cx, [ebp+var_F0]
.text:004665DB add cx, 1
.text:004665DF jo loc_466832
.text:004665E5 mov [ebp+var_F0], cx ; var_f0 加 1
.text:004665EC mov [ebp+var_4], 7Fh
.text:004665F3 cmp [ebp+var_F0], 3
.text:004665FB jnz short loc_46660D
.text:004665FD mov [ebp+var_4], 80h
.text:00466604 mov [ebp+var_F0], 1 ; var_f0 等于 3 则赋值 1
.text:0046660D
.text:0046660D loc_46660D: ; CODE XREF: sub_464EF0+170Bj
.text:0046660D jmp loc_4666AF
.text:00466612 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466612
.text:00466612 loc_466612: ; CODE XREF: sub_464EF0+150Cj
.text:00466612 mov [ebp+var_4], 83h
.text:00466619 movsx edx, word ptr [ebp+var_E4] ; 取绝对值
.text:00466620 mov [ebp+var_160], edx ; 保存到这里
.text:00466626 cmp [ebp+var_160], 201h
.text:00466630 jnb short loc_46663E
.text:00466632 mov [ebp+var_280], 0
.text:0046663C jmp short loc_46664A
.text:0046663E ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046663E
.text:0046663E loc_46663E: ; CODE XREF: sub_464EF0+1740j
.text:0046663E call ds:__vbaGenerateBoundsError
.text:00466644 mov [ebp+var_280], eax
.text:0046664A
.text:0046664A loc_46664A: ; CODE XREF: sub_464EF0+174Cj
.text:0046664A mov eax, [ebp+var_160] ; 绝对值
.text:00466650 mov ecx, [ebp+var_64] ; 数组 2 地址
.text:00466653 movsx edx, word ptr [ecx+eax*2] ; 取数组 2 一个元素
.text:00466657 push edx
.text:00466658 lea eax, [ebp+var_114]
.text:0046665E push eax
.text:0046665F call ds:rtcVarBstrFromAnsi ; 转成字符
.text:00466665 lea ecx, [ebp+var_50]
.text:00466668 push ecx
.text:00466669 movsx edx, [ebp+var_B8] ; 当前索引
.text:00466670 push edx
.text:00466671 push 1
.text:00466673 lea eax, [ebp+var_114]
.text:00466679 push eax
.text:0046667A call ds:__vbaStrVarMove
.text:00466680 mov edx, eax
.text:00466682 lea ecx, [ebp+var_104]
.text:00466688 call ds:__vbaStrMove
.text:0046668E push eax
.text:0046668F push 0
.text:00466691 call ds:__vbaMidStmtBstr ; mid(var_50,当前索引,1) = chr(数组 2 一个元素)
.text:00466697 lea ecx, [ebp+var_104]
.text:0046669D call ds:__vbaFreeStr
.text:004666A3 lea ecx, [ebp+var_114]
.text:004666A9 call ds:__vbaFreeVar
.text:004666AF
.text:004666AF loc_4666AF: ; CODE XREF: sub_464EF0+171Dj
.text:004666AF mov [ebp+var_4], 85h
.text:004666B6 jmp loc_466259
.text:004666BB ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:004666BB
.text:004666BB loc_4666BB: ; CODE XREF: sub_464EF0+1392j
.text:004666BB mov [ebp+var_4], 86h
.text:004666C2 mov ecx, [ebp+var_50] ; var_50
.text:004666C5 push ecx
.text:004666C6 mov edx, [ebp+arg_8] ; 假注册码
.text:004666C9 mov eax, [edx]
.text:004666CB push eax
.text:004666CC call ds:__vbaStrCmp ; strcomp
.text:004666D2 test eax, eax
.text:004666D4 jnz short loc_466742 ; 不等则跳
.text:004666D6 mov [ebp+var_4], 87h
.text:004666DD call ds:rtcGetTimer
.text:004666E3 fild [ebp+time_EC]
.text:004666E9 fstp [ebp+var_288]
.text:004666EF fsub [ebp+var_288]
.text:004666F5 fnstsw ax
.text:004666F7 test al, 0Dh
.text:004666F9 jnz loc_46682D
.text:004666FF call ds:__vbaFpR8
.text:00466705 fcomp ds:dbl_401D90
.text:0046670B fnstsw ax
.text:0046670D test ah, 1
.text:00466710 jnz short loc_466714
.text:00466712 jmp short loc_466742
.text:00466714 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:00466714
.text:00466714 loc_466714: ; CODE XREF: sub_464EF0+1820j
.text:00466714 mov [ebp+var_4], 8Ah
.text:0046671B mov ecx, [ebp+var_F8] ; 第一次变换的字符串
.text:00466721 push ecx
.text:00466722 mov edx, [ebp+arg_4] ; 用户名
.text:00466725 mov eax, [edx]
.text:00466727 push eax
.text:00466728 call ds:__vbaStrCmp ; strcomp
.text:0046672E test eax, eax
.text:00466730 jnz short loc_466742 ; 不等则跳
.text:00466732 mov [ebp+var_4], 8Bh
.text:00466739 mov [ebp+var_100], 0FFFFh ; 相等则结果为真
.text:00466742
.text:00466742 loc_466742: ; CODE XREF: sub_464EF0+5D9j
.text:00466742 ; sub_464EF0+6C4j
.text:00466742 ; sub_464EF0+8B7j
.text:00466742 ; sub_464EF0+E56j
.text:00466742 ; sub_464EF0+10ABj ...
.text:00466742 wait
.text:00466743 push offset loc_466813
.text:00466748 jmp short loc_466770
.text:0046674A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0046674A lea ecx, [ebp+var_104]
.text:00466750 call ds:__vbaFreeStr
.text:00466756 lea ecx, [ebp+var_124]
.text:0046675C push ecx
.text:0046675D lea edx, [ebp+var_114]
```
