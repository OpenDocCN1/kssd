
```asm
004C0639  |.  E8 4E45F4FF     |CALL Unpacked.00404B8C

004C063E  |.  8A00            |MOV AL,BYTE PTR DS:[EAX]

004C0640  |.  25 FF000000     |AND EAX,0FF

004C0645  |.  03F0            |ADD ESI,EAX

004C0647  |.  FF45 F4         |INC DWORD PTR SS:[EBP-C]

004C064A  |.  4B              |DEC EBX

004C064B  |.^ 75 D5           \JNZ SHORT Unpacked.004C0622      ;  循环求得机器码各位字符的ASC的和

004C064D  |>  8BC6            MOV EAX,ESI

004C064F  |.  B9 0C000000     MOV ECX,0C

004C0654  |.  99              CDQ

004C0655  |.  F7F9            IDIV ECX                          ;  求得的和除以0xC

004C0657  |.  69C0 DD870C00   IMUL EAX,EAX,0C87DD               ;  商乘以0xC87DD

004C065D  |.  8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]

004C0660  |.  E8 5788F4FF     CALL Unpacked.00408EBC            ;  转化为十进制字符串，得到字符串一

004C0665  |.  FF75 EC         PUSH DWORD PTR SS:[EBP-14]

004C0668  |.  8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]

004C066B  |.  6BC6 13         IMUL EAX,ESI,13                   ;  机器码各个字符的ASC的和乘以0x13

004C066E  |.  E8 4988F4FF     CALL Unpacked.00408EBC            ;  转化为十进制字符串，得到字符串二

004C0673  |.  FF75 E8         PUSH DWORD PTR SS:[EBP-18]

004C0676  |.  8D55 E4         LEA EDX,DWORD PTR SS:[EBP-1C]

004C0679  |.  8D86 DD870C00   LEA EAX,DWORD PTR DS:[ESI+C87DD]  ;  机器码各个字符的ASC的和加上0xC87DD

004C067F  |.  E8 3888F4FF     CALL Unpacked.00408EBC            ;  转化为十进制字符串，得到字符串三

004C0684  |.  FF75 E4         PUSH DWORD PTR SS:[EBP-1C]

004C0687  |.  8D55 E0         LEA EDX,DWORD PTR SS:[EBP-20]

004C068A  |.  69C6 D1020000   IMUL EAX,ESI,2D1                  ;  机器码各个字符的ASC的和乘以0x2D1

004C0690  |.  05 BE070000     ADD EAX,7BE                       ;  再加上0x7BE

004C0695  |.  E8 2288F4FF     CALL Unpacked.00408EBC            ;  转化为十进制字符串，得到字符串四

004C069A  |.  FF75 E0         PUSH DWORD PTR SS:[EBP-20]

004C069D  |.  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]

004C06A0  |.  BA 04000000     MOV EDX,4

004C06A5  |.  E8 AA43F4FF     CALL Unpacked.00404A54            ;  连接上面得到的四个十进制字符串，这里得到的就是注册码了

004C06AA  |.  33C0            XOR EAX,EAX

004C06AC  |.  5A              POP EDX

004C06AD  |.  59              POP ECX

004C06AE  |.  59              POP ECX

004C06AF  |.  64:8910         MOV DWORD PTR FS:[EAX],EDX

004C06B2  |.  68 D4064C00     PUSH Unpacked.004C06D4

004C06B7  |>  8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]

004C06BA  |.  BA 05000000     MOV EDX,5

004C06BF  |.  E8 3C40F4FF     CALL Unpacked.00404700

004C06C4  |.  8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]

004C06C7  |.  E8 1040F4FF     CALL Unpacked.004046DC

004C06CC  \.  C3              RETN
```

应该看到注册码是怎样形成的了；

## 【注册机源码】

```cpp
void CMy001Dlg::OnOK()

{

        // TODO: Add extra validation here

        //CDialog::OnOK();

        UpdateData(true);

        long i,n=0;

        char cMechine[20],SN1[20],SN2[20],SN3[20],SN4[20];

        strcpy(cMechine,m_Edit1);

        //求机器码各个字符的ASC的总和

        for (i=0;i<m_Edit1.GetLength ();i++)        n += cMechine[i];

        //求注册码的第一部分

        i=n/12;

        i *= 0xC87DD;

        _itoa(i,SN1,10);

        //求注册码的第二部分

        i=n*19;

        _itoa(i,SN2,10);

        //求注册码的第三部分

        i=n+0xC87DD;

        _itoa(i,SN3,10);

        //求注册码的第四部分

        i=n*0x2D1+0x7BE;

        _itoa(i,SN4,10);

        //连接得到注册码

        m_Edit2="";

        m_Edit2 = m_Edit2+SN1+SN2+SN3+SN4;

        UpdateData(false);

}
```

---

## ××特效梦工厂 V2.0

`UPX` 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo的外壳，用`UPX`自己脱掉，Borland Delphi 6.0 - 7.0编写

```asm
004E11D4  |.  8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]

004E11D7  |.  8B86 E4090000   MOV EAX,DWORD PTR DS:[ESI+9E4]

004E11DD  |.  E8 7ECDF5FF     CALL JSBuilde.0043DF60            ;  取得注册码

004E11E2  |.  837D F8 00      CMP DWORD PTR SS:[EBP-8],0

004E11E6  |.  75 1F           JNZ SHORT JSBuilde.004E1207

004E11E8  |.  B8 60134E00     MOV EAX,JSBuilde.004E1360

004E11ED  |.  E8 2A60F5FF     CALL JSBuilde.0043721C            ;  对话框：请输入注册码

004E11F2  |.  8B96 E4090000   MOV EDX,DWORD PTR DS:[ESI+9E4]

004E11F8  |.  A1 F8504F00     MOV EAX,DWORD PTR DS:[4F50F8]

004E11FD  |.  E8 127EF7FF     CALL JSBuilde.00459014

004E1202  |.  E9 1E010000     JMP JSBuilde.004E1325

004E1207  |>  8D55 FC         LEA EDX,DWORD PTR SS:[EBP-4]

004E120A  |.  8B86 E4090000   MOV EAX,DWORD PTR DS:[ESI+9E4]

004E1210  |.  E8 4BCDF5FF     CALL JSBuilde.0043DF60

004E1215  |.  8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]

004E1218  |.  8B86 E4090000   MOV EAX,DWORD PTR DS:[ESI+9E4]

004E121E  |.  E8 3DCDF5FF     CALL JSBuilde.0043DF60

004E1223  |.  8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]

004E1226  |.  E8 FD3AF2FF     CALL JSBuilde.00404D28            ;  计算注册码的长度作为循环次数

004E122B  |.  8BD0            MOV EDX,EAX

004E122D  |.  85D2            TEST EDX,EDX

004E122F  |.  7E 3C           JLE SHORT JSBuilde.004E126D

004E1231  |.  B8 01000000     MOV EAX,1

004E1236  |>  8B4D FC         /MOV ECX,DWORD PTR SS:[EBP-4]

004E1239  |.  8A4C01 FF       |MOV CL,BYTE PTR DS:[ECX+EAX-1]

004E123D  |.  80F9 30         |CMP CL,30

004E1240  |.  72 08           |JB SHORT JSBuilde.004E124A

004E1242  |.  8B5D FC         |MOV EBX,DWORD PTR SS:[EBP-4]

004E1245  |.  80F9 39         |CMP CL,39

004E1248  |.  76 1F           |JBE SHORT JSBuilde.004E1269

004E124A  |>  B8 7C134E00     |MOV EAX,JSBuilde.004E137C        ;  对话框：注册码输入有误,请重新输入!

004E124F  |.  E8 C85FF5FF     |CALL JSBuilde.0043721C

004E1254  |.  8B96 E4090000   |MOV EDX,DWORD PTR DS:[ESI+9E4]

004E125A  |.  A1 F8504F00     |MOV EAX,DWORD PTR DS:[4F50F8]

004E125F  |.  E8 B07DF7FF     |CALL JSBuilde.00459014

004E1264  |.  E9 BC000000     |JMP JSBuilde.004E1325

004E1269  |>  40              |INC EAX

004E126A  |.  4A              |DEC EDX

004E126B  |.^ 75 C9           \JNZ SHORT JSBuilde.004E1236      ;  用循环检验注册码是否全部是数字

004E126D  |>  8D55 F0         LEA EDX,DWORD PTR SS:[EBP-10]

004E1270  |.  8B86 E0090000   MOV EAX,DWORD PTR DS:[ESI+9E0]

004E1276  |.  E8 E5CCF5FF     CALL JSBuilde.0043DF60            ;  取得机器码

004E127B  |.  8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]

004E127E  |.  E8 A17EF2FF     CALL JSBuilde.00409124            ;  机器码的字符串转化为整数

004E1283  |.  E8 FC9BFEFF     CALL JSBuilde.004CAE84            ;  计算真注册码

004E1288  |.  8BD8            MOV EBX,EAX

004E128A  |.  8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]

004E128D  |.  E8 927EF2FF     CALL JSBuilde.00409124            ;  假注册码的字符串转化为整数

004E1292  |.  3BD8            CMP EBX,EAX

004E1294  |.  0F85 81000000   JNZ JSBuilde.004E131B             ;  关键跳转

004E129A  |.  B2 01           MOV DL,1

004E129C  |.  A1 B8C64600     MOV EAX,DWORD PTR DS:[46C6B8]

004E12A1  |.  E8 12B5F8FF     CALL JSBuilde.0046C7B8

004E12A6  |.  8BD8            MOV EBX,EAX

004E12A8  |.  BA 02000080     MOV EDX,80000002

004E12AD  |.  8BC3            MOV EAX,EBX

004E12AF  |.  E8 A4B5F8FF     CALL JSBuilde.0046C858

004E12B4  |.  33C9            XOR ECX,ECX

004E12B6  |.  BA A0134E00     MOV EDX,JSBuilde.004E13A0         ;  ASCII "System\CurrentControlSet\Services\Class\knightsoft\JSBuilder"

004E12BB  |.  8BC3            MOV EAX,EBX

004E12BD  |.  E8 D6B6F8FF     CALL JSBuilde.0046C998

004E12C2  |.  8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]

004E12C5  |.  8B86 E0090000   MOV EAX,DWORD PTR DS:[ESI+9E0]

004E12CB  |.  E8 90CCF5FF     CALL JSBuilde.0043DF60

004E12D0  |.  8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]

004E12D3  |.  E8 4C7EF2FF     CALL JSBuilde.00409124

004E12D8  |.  E8 A79BFEFF     CALL JSBuilde.004CAE84

004E12DD  |.  8BC8            MOV ECX,EAX

004E12DF  |.  BA E8134E00     MOV EDX,JSBuilde.004E13E8         ;  ASCII "registecode"

004E12E4  |.  8BC3            MOV EAX,EBX

004E12E6  |.  E8 FDB7F8FF     CALL JSBuilde.0046CAE8

004E12EB  |.  8BC3            MOV EAX,EBX

004E12ED  |.  E8 36B5F8FF     CALL JSBuilde.0046C828

004E12F2  |.  8BC3            MOV EAX,EBX

004E12F4  |.  E8 0B2AF2FF     CALL JSBuilde.00403D04
```
