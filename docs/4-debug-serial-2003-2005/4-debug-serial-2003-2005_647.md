
# TVjukebox2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6895.htm`](https://www.pediy.com/kssd/pediy06/pediy6895.htm)

【软件名称】`TV jukebox2.0`

【下载地址】拿别人的拷贝

【应用平台】`Win9x/NT/2000/XP`

【软件大小】未知

【软件限制】安装序列号

【破解作者】`sword`

【破解声明】破解只是感兴趣，无其它目的。失误之处敬请诸位大侠赐教!

【破解工具】`ollydbg`，`C32Asm`

【软件简介】一款电视卡软件，同学说只能从光盘安装时才会出现输入序列号的地方，否则安装完成后，提示序号错误退出！

--------------------------------------------------------------------------------

## 【分析过程】

### 【前言】
很弱智的软件加密算法，高手别看了。浪费时间

### 【Thinking in Crack】
既然能安装成功，在运行时才提示出错，那么注册码基本上就是写在硬盘或者注册表的某个地方，（1）可以用 `RegOpenKey` 和 `CreateFile` 下断试试！（2）既然会弹出错误提示我们不防反编译一下，找找从哪提示出错的，很遗憾找不到，那样的话，就用下 `MessageBox` 看看有没有收获，呵呵有，下面用第二种方式分析：

### 【算法追踪】

```
0041DC55  .  50                 PUSH EAX                      ; /pBufSize
0041DC56  .  8B4424 14          MOV EAX,DWORD PTR SS:[ESP+14] ; |
0041DC5A  .  8D5424 50         LEA EDX,DWORD PTR SS:[ESP+50]  ; |
0041DC5E  .  51                 PUSH ECX                        ;                       |  Buffer
0041DC5F  .  52                 PUSH EDX                        ; |pValueType
0041DC60  .  53                 PUSH EBX                        ; |Reserved
0041DC61  .  68 98494400        PUSH tvjukebo.00444998          ; |ValueName = serial_number"
0041DC66  .  50                 PUSH EAX                        ; |hKey
0041DC67  .  C74424 60 00010000 MOV DWORD PTR SS:[ESP+60],100   ; |
0041DC6F  .  FF15 00204300      CALL DWORD PTR DS:[<&ADVAPI32.RegQueryValueExA>>; \\取注册表的键值  哈哈看看你的注册表看看有没有这个，没有的话自己建一个，然后给他符一个字符串 我用 sword
0041DC75  .  85C0              TEST EAX,EAX
0041DC77  .  75 25              JNZ SHORT tvjukebo.0041DC9E          ;  如果没有那个键值的话，跳走
0041DC79  .  8B4424 4C          MOV EAX,DWORD PTR SS:[ESP+4C]
0041DC7D  .  BF 01000000         MOV EDI,1
0041DC82  .  3BC7               CMP EAX,EDI
0041DC84  .  75 18              JNZ SHORT tvjukebo.0041DC9E
0041DC86  .  8D8C24 20010000    LEA ECX,DWORD PTR SS:[ESP+120]      ;  传字符串首地址
0041DC8D  .  51                 PUSH ECX
0041DC8E  .  E8 8D070000        CALL tvjukebo.0041E420              ;  很容易发现这里是对输入的注册码进行计算和比较的－－跟进
0041DC93  .  83C4 04            ADD ESP,4
0041DC96  .  85C0               TEST EAX,EAX
0041DC98  .  75 04              JNZ SHORT tvjukebo.0041DC9E
0041DC9A  .  897C24 44          MOV DWORD PTR SS:[ESP+44],EDI
0041DC9E  |> 8B5424 10          MOV EDX,DWORD PTR SS:[ESP+10]
0041DCA2  .  52                 PUSH EDX                        ; /hKey
0041DCA3  .  FF15 10204300      CALL DWORD PTR DS:[<&ADVAPI32.RegCloseKey>]     ; \\RegCloseKey
0041DCA9  .  395C24 44          CMP DWORD PTR SS:[ESP+44],EBX
0041DCAD      75 28             JNZ SHORT tvjukebo.0041DCD7
0041DCAF  .  A1 148D4400        MOV EAX,DWORD PTR DS:[448D14]
0041DCB4  .  53                 PUSH EBX                        ; /Style
0041DCB5  .  68 40DC4300        PUSH tvjukebo.0043DC40          ; |Title = "TV Jukebox 2.0"
0041DCBA  .  8B48 5C            MOV ECX,DWORD PTR DS:[EAX+5C]   ; |
0041DCBD  .  51                 PUSH ECX                        ; |Text
0041DCBE  .  53                 PUSH EBX                        ; |hOwner
0041DCBF  .  FF15 58234300      CALL DWORD PTR DS:[<&USER32.MessageBoxA>]       ;  \\MessageBoxA  弹出错误提示框！
0041DCC5  .  5F                 POP EDI
0041DCC6  .  5E                 POP ESI
```

## 【具体算法】
跟进上面的那个 `call`



## 分析总结

### 算法总结
从上面的分析可以看出，注册码的长度必须为 `>=20` 个字符。根据第四位和第 12 位的和（奇偶性）不同采用不同的变换，但是算法都差不多，然后处理前 4 个字符，将处理的结果转化成长度为 8 的字符串，这 8 个字符就是注册码第 13 位到第 20 位的字符。

```
代码:

0041E420  /$ 83EC 14             SUB ESP,14
0041E423  |. 56                 PUSH ESI
0041E424  |. 8B7424 1C          MOV ESI,DWORD PTR SS:[ESP+1C]
0041E428  |. 57                 PUSH EDI
0041E429  |. 0FBE46 03         MOVSX EAX,BYTE PTR DS:[ESI+3]             ;  注册码的第四位字符
0041E42D  |. 0FBE4E 0B         MOVSX ECX,BYTE PTR DS:[ESI+B]             ;  注册码的第十二位字符，如果注册码长度小于 12 这里是程序中的一个值
0041E431  |. 8D7E 0B           LEA EDI,DWORD PTR DS:[ESI+B]              ;  字符串地址传送指令，将第 12 个字符以后的字符串首地址写入 edi 中
0041E434  |. 03C8              ADD ECX,EAX                               ;  将输入注册码的第四位和第十二位相加，
0041E436  |. 81E1 01000080     AND ECX,80000001                          ;  这里的结果决定下面第二个跳转的分支
0041E43C  |. 79 05             JNS SHORT tvjukebo.0041E443               ;  符号位为 0 时跳
0041E43E  |. 49                DEC ECX
0041E43F  |. 83C9 FE           OR ECX,FFFFFFFE
0041E442  |. 41                INC ECX
0041E443  |> 75 38             JNZ SHORT tvjukebo.0041E47D               ;  根据上面和的奇偶值不同做不同的处理
0041E445  |. 8B4E 08           MOV ECX,DWORD PTR DS:[ESI+8]              ;  取双字节的值
0041E448  |. 8B56 04           MOV EDX,DWORD PTR DS:[ESI+4]
0041E44B  |. 894C24 0C         MOV DWORD PTR SS:[ESP+C],ECX
0041E44F  |. 895424 08         MOV DWORD PTR SS:[ESP+8],EDX
0041E453  |. 0FBE0E            MOVSX ECX,BYTE PTR DS:[ESI]
0041E456  |. C64424 10 00      MOV BYTE PTR SS:[ESP+10],0
0041E45B  |. 8D0C89            LEA ECX,DWORD PTR DS:[ECX+ECX*4]          ;  注册码的第一个字符乘 5
0041E45E  |. 8D1489            LEA EDX,DWORD PTR DS:[ECX+ECX*4]          ;  注册码的第一个字符乘 5 之后结果再乘 5 也就是第一个字符乘 25
0041E461  |. 0FBE4E 01         MOVSX ECX,BYTE PTR DS:[ESI+1]
0041E465  |. 8D0C91            LEA ECX,DWORD PTR DS:[ECX+EDX*4]          ;  注册码的第二个字符和第一个字符计算的结果相加
0041E468  |. 8D0C89            LEA ECX,DWORD PTR DS:[ECX+ECX*4]          ;  结果乘 5
0041E46B  |. 8D1489            LEA EDX,DWORD PTR DS:[ECX+ECX*4]          ;  结果再乘 5
0041E46E  |. 0FBE4E 02         MOVSX ECX,BYTE PTR DS:[ESI+2]            ;  注册码的第三位
0041E472  |. 8D0C91            LEA ECX,DWORD PTR DS:[ECX+EDX*4]
0041E475  |. 8D0C89            LEA ECX,DWORD PTR DS:[ECX+ECX*4]
0041E478  |. 8D1489            LEA EDX,DWORD PTR DS:[ECX+ECX*4]          ;  和上面两位的处理一样
0041E47B  |. EB 37             JMP SHORT tvjukebo.0041E4B4
0041E47D  |> 33C9              XOR ECX,ECX
0041E47F  |> 8A17              /MOV DL,BYTE PTR DS:[EDI]                ;  取注册码的第十二位
0041E481  |. 88540C 08          |MOV BYTE PTR SS:[ESP+ECX+8],DL
0041E485  |. 41                 |INC ECX
0041E486  |. 4F                 |DEC EDI
0041E487  |. 83F9 08            |CMP ECX,8
0041E48A  |. 7C F3              \JL SHORT tvjukebo.0041E47F             ;  将从第 12 位往前的 8 个字符串逆序
0041E48C  |. C6440C 08 00      MOV BYTE PTR SS:[ESP+ECX+8],0
0041E491  |. 8D0480            LEA EAX,DWORD PTR DS:[EAX+EAX*4]          ;  注册码的第 4 位字符
0041E494  |. 0FBE4E 02         MOVSX ECX,BYTE PTR DS:[ESI+2]            ;  注册码的第三位字符
0041E498  |. 8D1480            LEA EDX,DWORD PTR DS:[EAX+EAX*4]          ;  第四位字符乘 25
0041E49B  |. 8D0491            LEA EAX,DWORD PTR DS:[ECX+EDX*4]          ;  第三位和上面的结果相加
0041E49E  |. 0FBE4E 01         MOVSX ECX,BYTE PTR DS:[ESI+1]            ;  注册码的第二位字符
0041E4A2  |. 8D0480            LEA EAX,DWORD PTR DS:[EAX+EAX*4]
0041E4A5  |. 8D0480            LEA EAX,DWORD PTR DS:[EAX+EAX*4]          ;  上面的结果乘 25
0041E4A8  |. 8D0481            LEA EAX,DWORD PTR DS:[ECX+EAX*4]
0041E4AB  |. 8D0480            LEA EAX,DWORD PTR DS:[EAX+EAX*4]
0041E4AE  |. 8D1480            LEA EDX,DWORD PTR DS:[EAX+EAX*4]
0041E4B1  |. 0FBE06            MOVSX EAX,BYTE PTR DS:[ESI]
0041E4B4  |> 8D4C24 08          LEA ECX,DWORD PTR SS:[ESP+8]             ;  从注册码的第 5 位开始
0041E4B8  |. 8D3C90            LEA EDI,DWORD PTR DS:[EAX+EDX*4]          ;  注册码的第四位开始计算
0041E4BB  |. 51                 PUSH ECX
0041E4BC  |. 0FAFFF             IMUL EDI,EDI                              ;  乘方，取双字节 注册码的十六进制
0041E4BF  |. E8 1EA60000       CALL tvjukebo.00428AE2
0041E4C4  |. 8BD0               MOV EDX,EAX
0041E4C6  |. 0FAFD0             IMUL EDX,EAX
0041E4C9  |. 03D7               ADD EDX,EDI
0041E4CB  |. 8D4424 0C          LEA EAX,DWORD PTR SS:[ESP+C]
0041E4CF  |. 52                 PUSH EDX                                    ; /<%08x>
0041E4D0  |. 68 E4494400        PUSH tvjukebo.004449E4                    ; |Format = "%08x"
0041E4D5  |. 50                 PUSH EAX                                    ; |s
0041E4D6  |. FF15 80234300       CALL DWORD PTR DS:[<&USER32.wsprintfA>]     ; \转化为长度为 8 的十六进制字符串
0041E4DC  |. 83C4 10            ADD ESP,10
0041E4DF  |. 33C0               XOR EAX,EAX                                 ;  清空累加器
0041E4E1  |> 8A4C06 0C          /MOV CL,BYTE PTR DS:[ESI+EAX+C]            ;  从注册码的第十三位开始比较
0041E4E5  |. 8A5404 08          |MOV DL,BYTE PTR SS:[ESP+EAX+8]
0041E4E9  |. 3ACA               |CMP CL,DL
0041E4EB  |. 75 0E              |JNZ SHORT tvjukebo.0041E4FB
0041E4ED  |. 40                 |INC EAX
0041E4EE  |. 83F8 08            |CMP EAX,8
0041E4F1  |. 7C EE              \JL SHORT tvjukebo.0041E4E1
0041E4F3  |. 5F                 POP EDI
0041E4F4  |. 33C0               XOR EAX,EAX
0041E4F6  |. 5E                 POP ESI
0041E4F7  |. 83C4 14            ADD ESP,14
0041E4FA  |. C3                 RETN
0041E4FB  |> 5F                 POP EDI
0041E4FC  |. B8 01000000        MOV EAX,1
0041E501  |. 5E                 POP ESI
0041E502  |. 83C4 14            ADD ESP,14
0041E505  \. C3                 RETN
```

### 注册机写法
先让用户输入固定的前 12 位字符然后，根据用户的输入计算出后 8 个字符，两个相连接，就是最后的序号。

```cpp
void CTVjukebox_keygenDlg::OnCalc() 
{
  // TODO: Add your control notification handler code here
  UpdateData(TRUE);
  //MessageBox(m_xuhao);
  if (strlen(m_xuhao)<12) //避免后面计算时内存出错
  {
    MessageBox("您输入的序列号不足 12 位，请重新输入!");
    return ;
  }
  int ad=0;
  ad = (int)m_xuhao[3]+(int)m_xuhao[11];
  if (0 == ad%2) //偶数不跳
  {
    ad = (int)m_xuhao[0]*25;
    ad = ad * 4 + (int)m_xuhao[1];
    ad = ad * 25;
    ad = ad * 4 + (int)m_xuhao[2];
    ad = ad * 25;
    ad = ad * 4 + (int)m_xuhao[3];
    ad = ad * ad;
  }
  else
  {
    ad = (int)m_xuhao[3]*25;
    ad = ad * 4 + (int)m_xuhao[2];
    ad = ad * 25;
    ad = ad * 4 + (int)m_xuhao[1];
    ad = ad * 25;
    ad = ad * 4 + (int)m_xuhao[0];
    ad = ad * ad;
  }
  m_xuhao.Format(m_xuhao+"%08x",ad); //连接字符串
  UpdateData(FALSE);
}
```

## 后记
注册软件我不知道从哪里来得，破解价值不是很大，难度也巨容易，你要是看到这里那你亏大了，我只是聊以此文迎接新年！呵呵

## 版权信息
无

2005-1-4 `4dd466d0e1e0cc123afbe504da2462ca`
