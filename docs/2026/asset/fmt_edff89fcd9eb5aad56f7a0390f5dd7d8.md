
`004FF9BB  |.  BA C4FA4F00   |MOV EDX,Xlcs.004FFAC4`

`004FF9C0  |.  E8 CB50F0FF   |CALL Xlcs.00404A90`

`004FF9C5  |.  EB 58         |JMP SHORT Xlcs.004FFA1F`

`004FF9C7  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 34 ('4') of switch 004FF94D`

`004FF9CA  |.  BA D0FA4F00   |MOV EDX,Xlcs.004FFAD0`

`004FF9CF  |.  E8 BC50F0FF   |CALL Xlcs.00404A90`

`004FF9D4  |.  EB 49         |JMP SHORT Xlcs.004FFA1F`

`004FF9D6  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 35 ('5') of switch 004FF94D`

`004FF9D9  |.  BA DCFA4F00   |MOV EDX,Xlcs.004FFADC`

`004FF9DE  |.  E8 AD50F0FF   |CALL Xlcs.00404A90`

`004FF9E3  |.  EB 3A         |JMP SHORT Xlcs.004FFA1F`

`004FF9E5  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 36 ('6') of switch 004FF94D`

`004FF9E8  |.  BA E8FA4F00   |MOV EDX,Xlcs.004FFAE8`

`004FF9ED  |.  E8 9E50F0FF   |CALL Xlcs.00404A90`

`004FF9F2  |.  EB 2B         |JMP SHORT Xlcs.004FFA1F`

`004FF9F4  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 37 ('7') of switch 004FF94D`

`004FF9F7  |.  BA F4FA4F00   |MOV EDX,Xlcs.004FFAF4`

`004FF9FC  |.  E8 8F50F0FF   |CALL Xlcs.00404A90`

`004FFA01  |.  EB 1C         |JMP SHORT Xlcs.004FFA1F`

`004FFA03  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 38 ('8') of switch 004FF94D`

`004FFA06  |.  BA 00FB4F00   |MOV EDX,Xlcs.004FFB00`

`004FFA0B  |.  E8 8050F0FF   |CALL Xlcs.00404A90`

`004FFA10  |.  EB 0D         |JMP SHORT Xlcs.004FFA1F`

`004FFA12  |>  8D45 E4       |LEA EAX,DWORD PTR SS:[EBP-1C]           ;  Case 39 ('9') of switch 004FF94D`

`004FFA15  |.  BA 0CFB4F00   |MOV EDX,Xlcs.004FFB0C`

`004FFA1A  |.  E8 7150F0FF   |CALL Xlcs.00404A90`

`004FFA1F  |>  43            |INC EBX                                 ;  Default case of switch 004FF94D`

`004FFA20  |.  FE4D DF       |DEC BYTE PTR SS:[EBP-21]`

`004FFA23  |.^ 0F85 18FFFFFF \JNZ Xlcs.004FF941`

上面OD注释了那么多的Case，我没有跟进去看，但是每循环一次会出现一位注册码，猜想一下，很可能是在查表，每个Call都压入不同的内存地址，就是到哪里去取数字，到内存窗口察看一下，很容易找到对应关系：

`"0"`、`"1"`、`"2"`、`"3"`、`"4"`、`"5"`、`"6"`、`"7"`、`"8"`、`"9"`

`"4"`、`"0"`、`"5"`、`"9"`、`"6"`、`"1"`、`"7"`、`"3"`、`"8"`、`"2"`

```
004FFA29  |>  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004FFA2C  |.  8B55 E4       MOV EDX,DWORD PTR SS:[EBP-1C]
004FFA2F  |.  E8 F04DF0FF   CALL Xlcs.00404824
004FFA34  |>  33C0          XOR EAX,EAX
004FFA36  |.  5A            POP EDX
004FFA37  |.  59            POP ECX
004FFA38  |>  59            POP ECX
004FFA39  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004FFA3C  |.  68 6BFA4F00   PUSH Xlcs.004FFA6B
004FFA41  |>  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]
004FFA44  |.  BA 05000000   MOV EDX,5
004FFA49  |>  E8 A64DF0FF   CALL Xlcs.004047F4
004FFA4E  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
004FFA51  |.  BA 04000000   MOV EDX,4
004FFA56  |.  E8 994DF0FF   CALL Xlcs.004047F4
004FFA5B  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004FFA5E  |.  E8 6D4DF0FF   CALL Xlcs.004047D0
004FFA63  \.  C3            RETN
004FFA64   .^ E9 AF46F0FF   JMP Xlcs.00404118
004FFA69   .^ EB D6         JMP SHORT Xlcs.004FFA41
004FFA6B   .  5F            POP EDI
004FFA6C   .  5E            POP ESI
004FFA6D   .  5B            POP EBX
004FFA6E   .  8BE5          MOV ESP,EBP
004FFA70   .  5D            POP EBP
004FFA71   .  C3            RETN
```

----------------------------------------------------------------------------------------------

## 【破解心得】

注册码是这样形成的：(计算过程用十进制描述)

取得用户名`"rdsnow[BCG]"`的第一个字符`"r"`和最后一个字符`"]"`的十进制ASC码，分别得到`"114"`、`"93"`，取`"114"`、`"93"`的最后一位`"4"`、`"3"`，然后`1981*4*3=23772`

然后`2002414+ASC(r)*1+ASC(d)*2+ASC(s)*3+ASC(n)*4+ASC(o)*5+ASC(w)+ASC([)+ASC(B)+ASC(C)+ASC(G)+ASC(])=2004575`

上面两个结果连接得到`"237722004575"`

每个数字按下了对应关系查表

`"0"`、`"1"`、`"2"`、`"3"`、`"4"`、`"5"`、`"6"`、`"7"`、`"8"`、`"9"`

`"4"`、`"0"`、`"5"`、`"9"`、`"6"`、`"1"`、`"7"`、`"3"`、`"8"`、`"2"`

得到结果`"593355446131"`这就是注册码了。

用户名：`rdsnow[BCG]`

注册码：`593355446131`

## 【注册机源码】

Microsoft Visual C++ 6.0编写：

```c
void CMy001Dlg::OnOK()
{
        // TODO: Add extra validation here
        //CDialog::OnOK();
        UpdateData(true);
        char cName[50],cSN1[100],cSN2[100];
        int i,j,n,SN1,SN2;
        //检查用户名的长度
        n=m_Edit1.GetLength ();
        if (n>50) {
                MessageBox("用户名的长度不超过50","提示",MB_OK);
                return;
        }
        //生成中间结果1
        strcpy(cName,m_Edit1);
        SN1=cName[0];
        SN2=cName[n-1];
        SN1 %= 10;
        SN2 %= 10;
        SN1 = 1981*SN1*SN2;
        _itoa(SN1,cSN1,10);
        //生成中间结果2
        SN2=2002414;
        for (i=0;i<n;i++){
                if(i<5) j=i+1;
                else j=1;
                SN2 += cName[i] *j;
        }
        _itoa(SN2,cSN2,10);
        //连接两个结果，生成字符串
        strcat(cSN1,cSN2);
        //查表，得到注册码
        n=strlen(cSN1);
        for (i=0;i<n;i++){
                switch(cSN1[i]){
                case '0':cSN1[i]='4';break;
                case '1':cSN1[i]='0';break;
                case '2':cSN1[i]='5';break;
                case '3':cSN1[i]='9';break;
                case '4':cSN1[i]='6';break;
                case '5':cSN1[i]='1';break;
                case '6':cSN1[i]='7';break;
                case '7':cSN1[i]='3';break;
                case '8':cSN1[i]='8';break;
                case '9':cSN1[i]='2';break;
                }
        }
        //输出注册码
        m_Edit2=cSN1;
        UpdateData(false);
}
```

----------------------------------------------------------------------------------------------

## 【破解声明】

我是一只小菜鸟，偶得一点心得，愿与大家分享:)

## 【版权声明】

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

----------------------------------------------------------------------------------------------

文章写于2005-5-16 11:54:09

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
