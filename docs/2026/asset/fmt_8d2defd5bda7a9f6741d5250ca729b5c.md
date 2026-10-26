
## 第十一个条件：`0x49 <= S2[3] <= 0x52`

```
00498C32   .  7C 0A         JL SHORT 00498C3E
00498C34   .  3C 5A         CMP AL,5A
00498C36   .  7F 06         JG SHORT 00498C3E
00498C38   .  B8 01000000   MOV EAX,1
00498C3D   .  C3            RETN
00498C3E   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498C40   .  C3            RETN
```

## 第十二个条件：`0x42 <= S2[2] <= 0x4B`

```
00498B9A   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498B9E   .  8A40 02       MOV AL,BYTE PTR [EAX+2]           ;  取 S2[2]
00498BA1   .  3C 42         CMP AL,42
00498BA3   .  7C 39         JL SHORT 00498BDE
00498BA5   .  3C 4B         CMP AL,4B
00498BA7   .  7F 35         JG SHORT 00498BDE                 ;  0x42 <= S2[2] <= 0x4B
00498BA9   .  B8 01000000   MOV EAX,1
00498BAE   .  C3            RETN
00498BAF   >  83F8 05       CMP EAX,5
00498BB2   .  7E 15         JLE SHORT 00498BC9
00498BB4   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498BB8   .  8A41 02       MOV AL,BYTE PTR [ECX+2]           ;  取 S2[2]
00498BBB   .  3C 20         CMP AL,20
00498BBD   .  7E 1F         JLE SHORT 00498BDE
00498BBF   .  3C 4B         CMP AL,4B
00498BC1   .  7F 1B         JG SHORT 00498BDE                 ;  0x20 < S2[2] <= 0x4B
00498BC3   .  B8 01000000   MOV EAX,1
00498BC8   .  C3            RETN
00498BC9   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]
00498BCD   .  8A42 02       MOV AL,BYTE PTR [EDX+2]           ;  取 S2[2]
00498BD0   .  3C 41         CMP AL,41
00498BD2   .  7C 0A         JL SHORT 00498BDE
00498BD4   .  3C 5A         CMP AL,5A
00498BD6   .  7F 06         JG SHORT 00498BDE                 ;  0x41 <= S2[2] <= 0x5A
00498BD8   .  B8 01000000   MOV EAX,1
00498BDD   .  C3            RETN
00498BDE   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498BE0   .  C3            RETN
```

## 第十三个条件：`0x41 < S1[1] < 0x5A`

```
00498B4A   .  8B4424 08     MOV EAX,DWORD PTR [ESP+8]
00498B4E   .  8A40 01       MOV AL,BYTE PTR [EAX+1]           ;  取 S1[1]
00498B51   .  3C 41         CMP AL,41
00498B53   .  7E 04         JLE SHORT 00498B59
00498B55   .  3C 5A         CMP AL,5A
00498B57   .  7C 29         JL SHORT 00498B82                 ;  0x41 < S1[1] < 0x5A
00498B59   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498B5B   .  C3            RETN
00498B5C   >  83F8 04       CMP EAX,4
00498B5F   .  7C 0E         JL SHORT 00498B6F
00498B61   .  8B4C24 08     MOV ECX,DWORD PTR [ESP+8]
00498B65   .  33C0          XOR EAX,EAX
00498B67   .  8079 01 41    CMP BYTE PTR [ECX+1],41           ;  S1[1] > 0x41
00498B6B   .  0F9FC0        SETG AL
00498B6E   .  C3            RETN
00498B6F   >  83F8 02       CMP EAX,2
00498B72   .  7C 0E         JL SHORT 00498B82
00498B74   .  8B5424 08     MOV EDX,DWORD PTR [ESP+8]
00498B78   .  33C0          XOR EAX,EAX
00498B7A   .  807A 01 7A    CMP BYTE PTR [EDX+1],7A           ;  S1[1] < 0x7A
00498B7E   .  0F9CC0        SETL AL
00498B81   .  C3            RETN
00498B82   >  B8 01000000   MOV EAX,1                         ;  到这里就返回 1 了
00498B87   .  C3            RETN
```

## 第十四个条件：`0x4B <= S2[4] <= 0x54`

```
00498C5A   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498C5E   .  8A40 04       MOV AL,BYTE PTR [EAX+4]           ;  取 S2[4]
00498C61   .  3C 4B         CMP AL,4B
00498C63   .  7C 39         JL SHORT 00498C9E
00498C65   .  3C 54         CMP AL,54
00498C67   .  7F 35         JG SHORT 00498C9E                 ;  0x4B <= S2[4] <= 0x54
00498C69   .  B8 01000000   MOV EAX,1
00498C6E   .  C3            RETN
00498C6F   >  83F8 02       CMP EAX,2
00498C72   .  7E 15         JLE SHORT 00498C89
00498C74   .  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498C78   .  8A41 04       MOV AL,BYTE PTR [ECX+4]           ;  取 S2[4]
00498C7B   .  3C 41         CMP AL,41
00498C7D   .  7E 1F         JLE SHORT 00498C9E
00498C7F   .  3C 54         CMP AL,54
00498C81   .  7F 1B         JG SHORT 00498C9E                 ;  0x41 < S2[4] <= 0x54
00498C83   .  B8 01000000   MOV EAX,1
00498C88   .  C3            RETN
00498C89   >  8B5424 0C     MOV EDX,DWORD PTR [ESP+C]
00498C8D   .  8A42 04       MOV AL,BYTE PTR [EDX+4]           ;  取 S2[4]
00498C90   .  3C 41         CMP AL,41
00498C92   .  7C 0A         JL SHORT 00498C9E
00498C94   .  3C 5A         CMP AL,5A
00498C96   .  7F 06         JG SHORT 00498C9E                 ;  0x41 <= S2[4] <= 0x5A
00498C98   .  B8 01000000   MOV EAX,1
00498C9D   .  C3            RETN
00498C9E   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498CA0   .  C3            RETN
```

## 第十五个条件：`0x41 < S1[0] < 0x5A`

```
00498AFA   .  8B4424 08     MOV EAX,DWORD PTR [ESP+8]
00498AFE   .  8A00          MOV AL,BYTE PTR [EAX]             ;  取 S1[0]
00498B00   .  3C 41         CMP AL,41
00498B02   .  7E 04         JLE SHORT 00498B08
00498B04   .  3C 5A         CMP AL,5A                         ;  0x41 < S1[0] < 0x5A
00498B06   .  7C 27         JL SHORT 00498B2F
00498B08   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498B0A   .  C3            RETN
00498B0B   >  83F8 04       CMP EAX,4
00498B0E   .  7C 0D         JL SHORT 00498B1D
00498B10   .  8B4C24 08     MOV ECX,DWORD PTR [ESP+8]
00498B14   .  33C0          XOR EAX,EAX
00498B16   .  8039 41       CMP BYTE PTR [ECX],41             ;  S1[0] > 0x41
00498B19   .  0F9FC0        SETG AL
00498B1C   .  C3            RETN
00498B1D   >  83F8 02       CMP EAX,2
00498B20   .  7C 0D         JL SHORT 00498B2F
00498B22   .  8B5424 08     MOV EDX,DWORD PTR [ESP+8]
00498B26   .  33C0          XOR EAX,EAX
00498B28   .  803A 7A       CMP BYTE PTR [EDX],7A             ;  S1[0] < 0x7A
00498B2B   .  0F9CC0        SETL AL
00498B2E   .  C3            RETN
00498B2F   >  B8 01000000   MOV EAX,1                         ;  到这里就返回 1 了
00498B34   .  C3            RETN
```

## 第十六个条件：`0x4A <= S2[5] <= 0x53`

```
00498CBA   .  8B4424 0C     MOV EAX,DWORD PTR [ESP+C]
00498CBE   .  8A40 05       MOV AL,BYTE PTR [EAX+5]           ;  取 S2[5]
00498CC1   .  3C 4A         CMP AL,4A
00498CC3   .  7C 1F         JL SHORT 00498CE4
00498CC5   .  3C 53         CMP AL,53
00498CC7   .  7F 1B         JG SHORT 00498CE4                 ;  0x4A <= S2[5] <= 0x53
00498CC9   .  B8 01000000   MOV EAX,1
00498CCE   .  C3            RETN
00498CCF   >  8B4C24 0C     MOV ECX,DWORD PTR [ESP+C]
00498CD3   .  8A41 05       MOV AL,BYTE PTR [ECX+5]           ;  取 S2[5]
00498CD6   .  3C 41         CMP AL,41
00498CD8   .  7C 0A         JL SHORT 00498CE4
00498CDA   .  3C 5A         CMP AL,5A
00498CDC   .  7F 06         JG SHORT 00498CE4                 ;  0x41 <= S2[5] <= 0x5A
00498CDE   .  B8 01000000   MOV EAX,1
00498CE3   .  C3            RETN
00498CE4   >  33C0          XOR EAX,EAX                       ;  到这里就返回 0 了
00498CE6   .  C3            RETN
```

---

## 破解心得

把十六个条件整理一下：就可以制作注册机了，另外程序中又很多方法生成随机数，也值得借鉴一下(文中已经略去，可以自己跟一下)

个人感觉作者使用了消磨 Cracker 耐力的办法了阻挡破解，也有一定效果，如果程序被做出 Keygen ，作者只要稍许修改几个数据，就会使得 Cracker 又要花上很多的精力去跟程序。

但是这种方法也有缺陷：

因为注册码跟硬件无关，作者通过频繁更换注册方法阻止算法注册机的出现，但是新版本注册算法的变化会导致正版用户不得不不断的更换注册码来享受新版本。

另外程序在防止暴力破解上做的不够。

建议：

- 某些位置的注册码跟硬件有关，这样特定的电脑上只能使用某些注册码
- 可以将 16 个验证放在程序的不同位置，不一定非要放在按钮的响应代码中，比如程序启动时或使用到某个功能时随机选择不同的代码验证。

## 注册机源码

```
void CKeygenDlg::OnOK()
{
        // TODO: Add extra validation here

        char S0[7],S1[7],S2[8],S3[6];

        //S1

        S0[0]='D';
        S0[5]='-';
        S0[6]=0;

        do{
```
