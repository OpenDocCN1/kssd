
# 加密精灵 v2.61

> 原文：[`www.pediy.com/kssd/pediy05/pediy50631.htm`](https://www.pediy.com/kssd/pediy05/pediy50631.htm)

加密精灵注册机算法分析：

请各位大侠多多指点!谢谢!

**软件**：加密精灵 (EncryptGenie) 2.61 版

**工具**：`trw1.22` 娃娃修正版，`VC++6.0`，`fi2.5`。

### 软件简介

`加密精灵` (`EncryptGenie`) 2.61 版是一个功能强大非常方便和容易使用的加密文件和邮件的软件。

安装后的软件为未注册版本，有两个功能限制：

1. 未注册版最多只可以输入 10 个字符长的密码。注册版最多可以输入 128 个字符长的密码。
2. 未注册版软件不能保存编辑的加密规则。

首先用 `fi` 查看有无壳，结果无壳，并且是用 `VC++6.0` 编写的。

运行加密精灵，弹出注册对话框，输入姓名：`crackerboy`，注册码：`7878787878`

运行 `trw1.22`

`Ctrl+N` 呼出 `trw`

下断点 `bpx hmemcpy`

`bc *`

`pmodule`

来到下面的代码：

```
:004018C1 6A1E                  push 0000001E
:004018C3 68E84E4400            push 00444EE8
:004018C8 6815040000            push 00000415
:004018CD 53                    push ebx
:004018CE FFD6                  call esi /*得到注册名*/
:004018D0 50                   push eax
:004018D1 FFD7                  call edi /*得到注册码*/
:004018D3 6A02                  push 00000002
:004018D5 6A00                  push 00000000
:004018D7 E844CE0100            call 0041E720
:004018DC 6A00                  push 00000000
:004018DE 68E84E4400            push 00444EE8
:004018E3 68EC594400            push 004459EC

* Reference To: Rule.CheckRegister, Ord:0000h
                     |
:004018E8 E82B060300            Call 00431F18 /*检查注册码*/
:004018ED 83C414                add esp, 00000014
:004018F0 F7D8                  neg eax
:004018F2 1BC0                  sbb eax, eax
```

`Rule.CheckRegister` 顾名思义是检查注册，这是关键 `call`，按 `F8` 跟进去。

```
004734B0 > 8B4424 04       MOV EAX,DWORD PTR SS:[ESP+4]
004734B4   83EC 28         SUB ESP,28
004734B7   85C0            TEST EAX,EAX
004734B9   57              PUSH EDI
004734BA   74 5A          JE SHORT RULE.00473516
004734BC   8B7C24 34      MOV EDI,DWORD PTR SS:[ESP+34]
004734C0   85FF           TEST EDI,EDI
004734C2   74 52          JE SHORT RULE.00473516
004734C4   8D4C24 04      LEA ECX,DWORD PTR SS:[ESP+4]
004734C8   51              PUSH ECX
004734C9   50              PUSH EAX
004734CA   E8 51000000     CALL RULE.CreateRegisterSerial /*产生注册码*/
004734CF   83C4 08         ADD ESP,8
004734D2   85C0           TEST EAX,EAX
004734D4   74 40          JE SHORT RULE.00473516
004734D6   53             PUSH EBX
004734D7   56             PUSH ESI
004734D8   8D7424 0C      LEA ESI,DWORD PTR SS:[ESP+C]
004734DC   8BC7           MOV EAX,EDI
004734DE   8A10           MOV DL,BYTE PTR DS:[EAX]
004734E0   8A1E           MOV BL,BYTE PTR DS:[ESI]
004734E2   8ACA           MOV CL,DL
004734E4   3AD3          CMP DL,BL
004734E6   75 1E          JNZ SHORT RULE.00473506
004734E8   84C9          TEST CL,CL
004734EA   74 16          JE SHORT RULE.00473502
004734EC   8A50 01        MOV DL,BYTE PTR DS:[EAX+1]
004734EF   8A5E 01        MOV BL,BYTE PTR DS:[ESI+1]
004734F2   8ACA          MOV CL,DL
004734F4   3AD3          CMP DL,BL
004734F6   75 0E          JNZ SHORT RULE.00473506
004734F8   83C0 02        ADD EAX,2
004734FB   83C6 02        ADD ESI,2
004734FE   84C9          TEST CL,CL
00473500   75 DC          JNZ SHORT RULE.004734DE
00473502   33C0          XOR EAX,EAX
00473504   EB 05          JMP SHORT RULE.0047350B
00473506   1BC0          SBB EAX,EAX
00473508   83D8 FF       SBB EAX,-1
0047350B   5E             POP ESI
0047350C   5B             POP EBX
0047350D   85C0          TEST EAX,EAX
0047350F   75 05          JNZ SHORT RULE.00473516
00473511   5F             POP EDI
00473512   83C4 28        ADD ESP,28
00473515   C3             RETN
00473516   B8 01000000    MOV EAX,1
0047351B   5F             POP EDI
0047351C   83C4 28        ADD ESP,28
0047351F   C3             RETN
```

走到 `004734CA` 时按 `F8` 跟进去，它是产生注册码的子程序。

按 `F8` 后就来到这里：

```
00473520 > 8B5424 04       MOV EDX,DWORD PTR SS:[ESP+4]
00473524   83EC 2C         SUB ESP,2C
00473527   85D2            TEST EDX,EDX
00473529   57              PUSH EDI
0047352A   0F84 B6000000   JE RULE.004735E6
00473530   8B4424 38      MOV EAX,DWORD PTR SS:[ESP+38]
00473534   85C0           TEST EAX,EAX
00473536   0F84 AA000000  JE RULE.004735E6
0047353C   8BFA           MOV EDI,EDX
0047353E   83C9 FF        OR ECX,FFFFFFFF
00473541   33C0           XOR EAX,EAX
00473543   F2:AE          REPNE SCAS BYTE PTR ES:[EDI]
00473545   F7D1           NOT ECX
00473547   49             DEC ECX    /*得到用户名长度*/
00473548   83F9 03        CMP ECX,3
0047354B   0F8C 95000000  JL RULE.004735E6 /*小于 3 则跳走*/

00473551   53             PUSH EBX
00473552   55             PUSH EBP
00473553   56             PUSH ESI
00473554   B3 41          MOV BL,41                       /*
00473556   885C04 14      MOV BYTE PTR SS:[ESP+EAX+14],BL /*
0047355A   FEC3           INC BL                           /*
0047355C   40             INC EAX
0047355D   83F8 1A        CMP EAX,1A                       /*
00473560   75 F4          JNZ SHORT RULE.00473556          /*
这里产生密码表：从 A 到 Z

00473562   B3 30          MOV BL,30                       /*
00473564   885C04 14      MOV BYTE PTR SS:[ESP+EAX+14],BL /*
00473568   FEC3           INC BL                           /*
0047356A   40             INC EAX                          /*
0047356B   83F8 25        CMP EAX,25                       /*
0047356E   75 F4          JNZ SHORT RULE.00473564          /*
产生 0 到 9

00473570   33ED           XOR EBP,EBP /*清 ebp*/
00473572   33C0           XOR EAX,EAX /*清 eax*/
00473574   85C9           TEST ECX,ECX
00473576   896C24 10      MOV DWORD PTR SS:[ESP+10],EBP
0047357A   7E 10          JLE SHORT RULE.0047358C /*长度小于零则跳走*/
0047357C   33DB           XOR EBX,EBX /*ebx 清零*/
0047357E   8A1C10         MOV BL,BYTE PTR DS:[EAX+EDX] /*把注册码的第一个字符送给 Bl*/
00473581   03EB           ADD EBP,EBX /*
00473583   40             INC EAX /*
00473584   3BC1           CMP EAX,ECX /*
00473586   7C F4          JL SHORT RULE.0047357C /*
从 47357c 到 473586 是这样的：它把注册名的所有注册名累加，并送入 ebp 保存。

00473588   896C24 10      MOV DWORD PTR SS:[ESP+10],EBP
0047358C   33FF           XOR EDI,EDI
0047358E   33DB           XOR EBX,EBX
00473590   BE 08184800    MOV ESI,RULE.00481808
00473595   EB 08          JMP SHORT RULE.0047359F
00473597   8B6C24 10      MOV EBP,DWORD PTR SS:[ESP+10]
0047359B   8B5424 40      MOV EDX,DWORD PTR SS:[ESP+40]
0047359F   33C0           XOR EAX,EAX
004735A1   8A0417         MOV AL,BYTE PTR DS:[EDI+EDX] /*送注册名的字符给 al*/
004735A4   03C5           ADD EAX,EBP /*与所有注册名的 ascii 值相加*/
004735A6   8B2E           MOV EBP,DWORD PTR DS:[ESI] /*送常数数组给 esi*/
004735A8   03C1           ADD EAX,ECX
004735AA   03C5           ADD EAX,EBP
004735AC   3BF9           CMP EDI,ECX /*比较是否超过了注册名长度*/
004735AE   7C 04          JL SHORT RULE.004735B4 /*没超过就跳*/
004735B0   33FF           XOR EDI,EDI
004735B2   EB 01          JMP SHORT RULE.004735B5 /*跳*/
004735B4   47             INC EDI /*指向下一个注册名*/
004735B5   99             CDQ
004735B6   BD 24000000    MOV EBP,24
004735BB   83C6 04        ADD ESI,4
004735BE   F7FD           IDIV EBP /*除以 0x24 即 36*/
004735C0   8B6C24 44      MOV EBP,DWORD PTR SS:[ESP+44]
004735C4   43             INC EBX
004735C5   81FE 48184800  CMP ESI,RULE.00481848  /*是否超过了常数数组*/
004735CB   8A4414 14      MOV AL,BYTE PTR SS:[ESP+EDX+14] /*dl 是除数，这样就在密码表中定位了
注册码*/
004735CF   88442B FF      MOV BYTE PTR DS:[EBX+EBP-1],AL /*存注册码*/
004735D3   7C C2          JL SHORT RULE.00473597 /*没超过就继续*/
```



```asm
004735D5   C6042B 00        MOV BYTE PTR DS:[EBX+EBP],0  
004735D9   5E              POP ESI  
004735DA   5D              POP EBP  
004735DB   5B              POP EBX  
004735DC   B8 01000000     MOV EAX,1  
004735E1   5F              POP EDI  
004735E2   83C4 2C         ADD ESP,2C  
004735E5   C3              RETN  
```

注册码的长度等于常数数组的长度，也是 16 位。

注册机的 VC 源程序如下（关键部分）：

```cpp
void CPatch0Dlg::OnOK()
{
  UpdateData();

  int length = m_hstr0.GetLength();

  m_hcode = "";

  //  TODO: Add extra validation here

  GetDlgItem(IDC_EDIT1)->SetFocus();

  char password[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";//密码表
  int count[] = {0x16, 0x1e, 0x13, 0x0c, 0x09, 0x04, 0x19, 0x1a, 0x24, 0x20, 0x13, 0x07, 0x36, 0x0e, 0x32, 0x3c};//常数数组
  int name1[17], flag0 = 0, flag1 = 1, flag2 = 0, countnum = 0, i, j, k = 0, l = 0;

  if ((length < 0) || (length < 4))
  {
    MessageBox("输入名字不能是空或者名字少于 4 个字符!!", 0, MB_OK);
    flag2 = 1;
  }
  else
  {
    for (i = 0; i < length; i++)
    {
      name1[i] = (int)m_hstr0[i];
      flag1 = isupper(m_hstr0[i]) | isdigit(m_hstr0[i]) | islower(m_hstr0[i]);
      if (!flag1)
      {
        flag0 = 1;
        flag2 = 1;
      }
    }

    if (flag0)
      MessageBox("你输入的字符有非 ASCII 字符，请重新输入!!", 0, MB_OK);
    else if (!flag2)
    {
      for (i = length; i < 17; i++)
        name1[i] = 0;
      for (i = 0; i < length; i++)
        countnum += name1[i];
      i = 0;
      CString temp;
      m_hcode += temp;
      for (j = 0; j < 16; j++)
      {
        if (k > length) //k 是当前的注册名长度
          k = 0;
        i = countnum + length + name1[k++] + count[j];
        l = i % 0x24;
        temp.format("%c", password[l]); //定位注册码
        m_hcode += temp;
        i = 0;
      }
      GetDlgItem(IDC_EDIT2)->SetWindowText(m_hcode);
    }
  }

  if (flag2)
  {
    GetDlgItem(IDC_EDIT2)->SetWindowText("");
    GetDlgItem(IDC_EDIT1)->SetWindowText("");
  }

  //CDialog::OnOK();
}
```
