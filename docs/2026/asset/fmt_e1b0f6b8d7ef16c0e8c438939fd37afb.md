
# [原创]HexAssistant1.9注册码算法部分的分析

- 【文章标题】: HexAssistant1.9算法分析
- 【文章作者】: fangawxs
- 【作者邮箱】: fangawxs@163.com
- 【软件名称】: HexAssistant1.9
- 【软件大小】: 2.34M
- 【下载地址】: 自己搜索下载
- 【加壳方式】: 无
- 【保护方式】: serial
- 【编写语言】: VC++
- 【使用工具】: 0D,IDA Pro5.0
- 【操作平台】: win32
- 【软件介绍】: HexAssistant is an Internet-ready 32-bit hex edi
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 【详细过程】

运行软件,出现注册对话框,输入注册名:fangawxs,注册码:123456789123456789后,出现错误提示:Invalid User Name or Registration Code!

反汇编很快就找到出错的地方:

```asm
.text:0041224D 50                        push  eax                           ; lpData;指向用户名
.text:0041224E B9 38 17 4E 00            mov   ecx, offset unk_4E1738
.text:00412253 E8 5D E4 03 00            call  sub_4506B5                    ;关键call,进入
.text:00412258 85 C0                     test  eax, eax
.text:0041225A 74 0A                     jz    short loc_412266
.text:0041225C 8B 4D E8                  mov   ecx, [ebp+var_18]
.text:0041225F E8 D8 CD 04 00            call  ?OnOK@CDialog@@MAEXXZ         ; CDialog::OnOK(void)
.text:00412264 EB 17                     jmp   short loc_41227D
.text:00412266                         ; ---------------------------------------------------------------------------
.text:00412266
.text:00412266                         loc_412266:                           ; CODE XREF: sub_4121D3+87j
.text:00412266 6A 00                     push  0
.text:00412268 6A 01                     push  1
.text:0041226A 68 3A F0 00 00            push  0F03Ah                        ; 出错(Invalid User Name
                                                                                 or Registration Code!
.text:0041226F 8B 55 E8                  mov   edx, [ebp-18h]
.text:00412272 52                        push  edx
.text:00412273 B9 88 16 4E 00            mov   ecx, offset unk_4E1688
.text:00412278 E8 63 24 FF FF            call  sub_4046E0
.text:0041227D
.text:0041227D                         loc_41227D:                           ; CODE XREF: sub_4121D3+91j
.text:0041227D C6 45 FC 00               mov   byte ptr [ebp+var_4], 0
.text:00412281 8D 4D F0                  lea   ecx, [ebp+var_10]
```

进入`.text:00412253`处的`call sub_4506B5`

```asm
.text:004506B5 55                        push  ebp                           ; *lpData="用户名"
.text:004506B5                                                               ; *arg_4="注册码"
.text:004506B6 8B EC                     mov   ebp, esp
.text:004506B8 81 EC 1C 01 00 00         sub   esp, 11Ch
.text:004506BE 89 8D E4 FE FF FF         mov   [ebp+var_11C], ecx            ; 一个指针,*ecx=004c7834
.text:004506C4 C6 45 B8 00               mov   byte ptr [ebp+var_48], 0      ; var_48=0
.text:004506C8 33 C0                     xor   eax, eax                      ; set eax to zero
.text:004506CA 89 45 B9                  mov   [ebp+var_48+1], eax           ; var_48+1=var_47
.text:004506CD 89 45 BD                  mov   [ebp+var_43], eax
.text:004506D0 89 45 C1                  mov   [ebp+var_3F], eax
.text:004506D3 66 89 45 C5               mov   [ebp+var_3B], ax
.text:004506D7 88 45 C7                  mov   [ebp+var_39], al
.text:004506DA C6 45 E0 00               mov   byte ptr [ebp+var_20], 0
.text:004506DE 33 C9                     xor   ecx, ecx                      ; set ecx to zero
.text:004506E0 89 4D E1                  mov   [ebp+var_20+1], ecx           ; var_20+1=var_1F
.text:004506E3 89 4D E5                  mov   [ebp+var_1B], ecx
.text:004506E6 89 4D E9                  mov   [ebp+var_17], ecx
.text:004506E9 66 89 4D ED               mov   [ebp+var_13], cx
.text:004506ED 88 4D EF                  mov   [ebp+var_11], cl
.text:004506F0 C6 45 F0 00               mov   byte ptr [ebp+var_10], 0
.text:004506F4 33 D2                     xor   edx, edx                      ; set edx=0
.text:004506F6 89 55 F1                  mov   [ebp+var_10+1], edx           ; var_10+1=var_0F
.text:004506F9 89 55 F5                  mov   [ebp+var_B], edx
.text:004506FC 89 55 F9                  mov   [ebp+var_7], edx
.text:004506FF 66 89 55 FD               mov   [ebp+var_3], dx
.text:00450703 88 55 FF                  mov   [ebp+var_1], dl
.text:00450706 C6 45 CC 00               mov   byte ptr [ebp+var_34], 0
.text:0045070A 33 C0                     xor   eax, eax                      ; eax=0
.text:0045070C 89 45 CD                  mov   [ebp+var_34+1], eax           ; var_34+1=var_33
.text:0045070F 89 45 D1                  mov   [ebp+var_2F], eax
.text:00450712 89 45 D5                  mov   [ebp+var_2B], eax
.text:00450715 66 89 45 D9               mov   [ebp+var_27], ax
.text:00450719 88 45 DB                  mov   [ebp+var_25], al
.text:0045071C 8B 4D 08                  mov   ecx, [ebp+lpData]
.text:0045071F 51                        push  ecx                           ; char *
.text:00450720 E8 6B F0 00 00            call  strlen
.text:00450725 83 C4 04                  add   esp, 4
.text:00450728 83 F8 10                  cmp   eax, 10h                      ; 用户名 > 10h?
.text:00450728                                                               ; 如果大于16位,就取前16位
.text:0045072B 76 14                     jbe   short loc_450741
.text:0045072D 6A 10                     push  10h                           ; size_t
.text:0045072F 8B 55 08                  mov   edx, [ebp+lpData]             ; src
.text:00450732 52                        push  edx                           ; void *
.text:00450733 8D 45 B8                  lea   eax, [ebp+var_48]             ; dest
.text:00450736 50                        push  eax                           ; void *
.text:00450737 E8 60 F0 00 00            call  memcpy                        ; copy src to dest
.text:0045073C 83 C4 0C                  add   esp, 0Ch
.text:0045073F EB 1D                     jmp   short loc_45075E
.text:00450741                         ; ---------------------------------------------------------------------------
.text:00450741
.text:00450741                         loc_450741:                           ; CODE XREF: sub_4506B5+76j
.text:00450741 8B 4D 08                  mov   ecx, [ebp+lpData]
.text:00450744 51                        push  ecx                           ; char *
.text:00450745 E8 46 F0 00 00            call  strlen
.text:0045074A 83 C4 04                  add   esp, 4
.text:0045074D 50                        push  eax                           ; size_t
.text:0045074E 8B 55 08                  mov   edx, [ebp+lpData]             ; src
.text:00450751 52                        push  edx                           ; void *
.text:00450752 8D 45 B8                  lea   eax, [ebp+var_48]             ; dest
.text:00450755 50                        push  eax                           ; void *
.text:00450756 E8 41 F0 00 00            call  memcpy
.text:0045075B 83 C4 0C                  add   esp, 0Ch
.text:0045075E
.text:0045075E                         loc_45075E:                           ; CODE XREF: sub_4506B5+8Aj
.text:0045075E 8D 4D E0                  lea   ecx, [ebp+var_20]
.text:00450761 51                        push  ecx
.text:00450762 8D 55 B8                  lea   edx, [ebp+var_48]             ; *var_48=用户名
.text:00450765 52                        push  edx
.text:00450766 B9 28 1C 4E 00            mov   ecx, offset table1
.text:0045076B E8 EB F6 FE FF            call  sub_43FE5B                    ; 用户名的算法.
.text:0045076B                                                               ; 结果放在var_20处
.text:00450770 C6 45 DC 00               mov   [ebp+var_24], 0

.text:00450774 33 C0                     xor   eax, eax
```
